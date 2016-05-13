---
layout: post
title:  "Design Pattern(factory) / Template in Caffe(c++)"
date:   2016-05-11 10:00:00
tags: [design pattern, factory, template, c++, caffe, tech]
categories: Tech
---

> 本文重点分析深度学习平台(Caffe)中注册layer的相关代码[layer_factory.hpp](https://github.com/wykvictor/caffe/blob/master/include/caffe/layer_factory.hpp)

### 1. LayerRegistry - 工厂
{% highlight C++ %}
template <typename Dtype>  // 模板类(caffe中，预知是float或double)
class LayerRegistry {
 public:
  // 声明类型：函数指针 Creator
  typedef shared_ptr<Layer<Dtype> > (*Creator)(const LayerParameter&);
  // 声明类型：map存放layer-name和layer-creator函数指针的对应关系
  typedef std::map<string, Creator> CreatorRegistry;

  // 静态函数，可以通过类直接调用
  static CreatorRegistry& Registry() {
    // 静态变量，保证上述map(g_registry_)只被new一次
    static CreatorRegistry* g_registry_ = new CreatorRegistry();
    return *g_registry_;  // 整个程序只有1个这样的map, 存放所有的layername->layer指针
  }

  // Helper function: Adds a creator.
  static void AddCreator(const string& type, Creator creator) {
    CreatorRegistry& registry = Registry();  // 拿到这个map
    // 如果map.count(key)为0，则代表不存在key(multimap可以有0,1之外的其他值)
    CHECK_EQ(registry.count(type), 0)
        << "Layer type " << type << " already registered.";
    registry[type] = creator;
  }

  // Get a layer using a LayerParameter.
  static shared_ptr<Layer<Dtype> > CreateLayer(const LayerParameter& param) {
    if (Caffe::root_solver()) {
      LOG(INFO) << "Creating layer " << param.name();
    }
    const string& type = param.type();
    CreatorRegistry& registry = Registry();
    CHECK_EQ(registry.count(type), 1) << "Unknown layer type: " << type
        << " (known types: " << LayerTypeListString() << ")";
    return registry[type](param);  //同样拿到registry, 通过这个map拿到layer指针
  }

  static vector<string> LayerTypeList() {
    CreatorRegistry& registry = Registry();
    vector<string> layer_types;
    // 遍历这个map
    // typename?
    for (typename CreatorRegistry::iterator iter = registry.begin();
         iter != registry.end(); ++iter) {
      layer_types.push_back(iter->first);
    }
    return layer_types;
  }

 private:
  // Layer registry should never be instantiated - everything is done with its
  // static variables.
  LayerRegistry() {}
};
{% endhighlight %}

### 2. LayerRegisterer - 注册
{% highlight C++ %}
template <typename Dtype>
class LayerRegisterer {
 public:
  LayerRegisterer(const string& type,
                  shared_ptr<Layer<Dtype> > (*creator)(const LayerParameter&)) {
    // LOG(INFO) << "Registering layer type: " << type;
    LayerRegistry<Dtype>::AddCreator(type, creator);
  }
};

#define REGISTER_LAYER_CREATOR(type, creator)                                  \
  static LayerRegisterer<float> g_creator_f_##type(#type, creator<float>);     \
  static LayerRegisterer<double> g_creator_d_##type(#type, creator<double>)    \

#define REGISTER_LAYER_CLASS(type)                                             \
  template <typename Dtype>                                                    \
  shared_ptr<Layer<Dtype> > Creator_##type##Layer(const LayerParameter& param) \
  {                                                                            \
    return shared_ptr<Layer<Dtype> >(new type##Layer<Dtype>(param));           \
  }                                                                            \
  REGISTER_LAYER_CREATOR(type, Creator_##type##Layer)
{% endhighlight %}

### 3. INSTANTIATE_CLASS - 模板类实例化
{% highlight C++ %}
// Instantiate a class with float and double specifications.
#define INSTANTIATE_CLASS(classname) \
  char gInstantiationGuard##classname; \
  template class classname<float>; \
  template class classname<double>
{% endhighlight %}

### 4. SetUP Layer - 使用
例如新添加DispatchLayer层，在dispatch_layer.cpp底部需添加如下宏
{% highlight C++ %}
INSTANTIATE_CLASS(DispatchLayer);
REGISTER_LAYER_CLASS(Dispatch);
{% endhighlight %}
其编译后变成如下代码:
{% highlight C++ %}
template <typename Dtype>
shared_ptr<Layer<Dtype> > Creator_DispatchLayer(const LayerParameter& param) {
  return shared_ptr<Layer<Dtype> >(new DispatchLayer<Dtype>(param));
}
static LayerRegisterer<float> g_creator_f_Dispatch("Dispatch", Creator_DispatchLayer<float>);
{% endhighlight %}
在net.cpp中通过以下代码，将上述实例化对象添加到layers_中进行管理
{% highlight C++ %}
layers_.push_back(LayerRegistry<Dtype>::CreateLayer(layer_param));
{% endhighlight %}
If the layer is going to be created by another creator function, 可以直接调用REGISTER_LAYER_CREATOR
{% highlight C++ %}
// Get convolution layer according to engine.
template <typename Dtype>
shared_ptr<Layer<Dtype> > GetConvolutionLayer(
    const LayerParameter& param) {
  ConvolutionParameter conv_param = param.convolution_param();
  ConvolutionParameter_Engine engine = conv_param.engine();
#ifdef USE_CUDNN
  bool use_dilation = false;
  for (int i = 0; i < conv_param.dilation_size(); ++i) {
    if (conv_param.dilation(i) > 1) {
      use_dilation = true;
    }
  }
#endif
  if (engine == ConvolutionParameter_Engine_DEFAULT) {
    engine = ConvolutionParameter_Engine_CAFFE;
#ifdef USE_CUDNN
    if (!use_dilation) {
      engine = ConvolutionParameter_Engine_CUDNN;
    }
#endif
  }
  if (engine == ConvolutionParameter_Engine_CAFFE) {
    return shared_ptr<Layer<Dtype> >(new ConvolutionLayer<Dtype>(param));
#ifdef USE_CUDNN
  } else if (engine == ConvolutionParameter_Engine_CUDNN) {
    if (use_dilation) {
      LOG(FATAL) << "CuDNN doesn't support the dilated convolution at Layer "
                 << param.name();
    }
    return shared_ptr<Layer<Dtype> >(new CuDNNConvolutionLayer<Dtype>(param));
#endif
  } else {
    LOG(FATAL) << "Layer " << param.name() << " has unknown engine.";
  }
}

REGISTER_LAYER_CREATOR(Convolution, GetConvolutionLayer);
{% endhighlight %}

### 4. Targets.cmake - 编译
{% highlight Bash shell scripts %}
# Defines global Caffe_LINK flag, This flag is required to prevent linker from excluding
# some objects which are not addressed directly but are registered via static constructors
macro(caffe_set_caffe_link)
  if(BUILD_SHARED_LIBS)
    set(Caffe_LINK caffe)
  else()
    if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
      set(Caffe_LINK -Wl,-force_load caffe)
    elseif("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
      set(Caffe_LINK -Wl,--whole-archive caffe -Wl,--no-whole-archive)
    endif()
  endif()
endmacro()
{% endhighlight %}
