---
layout: post
title:  "Global initialization / Template in Caffe(c++)"
date:   2016-05-11 10:00:00
tags: [global initialize, register, template, c++, caffe, tech]
categories: Tech
---

> 本文重点分析深度学习平台(Caffe)中注册(register layer)的相关代码[layer_factory.hpp](https://github.com/wykvictor/caffe/blob/master/include/caffe/layer_factory.hpp)

> [参考](https://www.bfilipek.com/2018/02/factory-selfregister.html)

### 1. LayerRegistry - 功能模板类
好处：
* 添加新类时，总得修改factory::Create()函数，对于大系统很繁杂
* factory需要提前知道所有的类
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
    return *g_registry_;  // 整个程序只有1个这样的map, 存放所有的layername->layer-creator函数指针
  }

  // Helper function: Adds a creator.
  static void AddCreator(const string& type, Creator creator) {
    CreatorRegistry& registry = Registry();  // 拿到这个map
    // 如果map.count(key)为0，则代表不存在key(multimap可以有0,1之外的其他值)
    CHECK_EQ(registry.count(type), 0)
        << "Layer type " << type << " already registered.";
    registry[type] = creator;  // 实际的add动作
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
    // 同样拿到registry, 通过这个map拿到layer creator函数指针，并传入param，实际new了一个layer出来
    return registry[type](param);
  }

  static vector<string> LayerTypeList() {
    CreatorRegistry& registry = Registry();
    vector<string> layer_types;
    // 遍历这个map
    // typename必须: type names nested into dependent types(depend on template type)
    // need to be prepended with the typename keyword.
    // 如果CreatorRegistry实例化为int，则不需要typename了
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
实际的注册接口
{% highlight C++ %}
template <typename Dtype>
class LayerRegisterer {
 public:
  // 构造函数，接受type + 函数指针
  LayerRegisterer(const string& type,
                  shared_ptr<Layer<Dtype> > (*creator)(const LayerParameter&)) {
    // LOG(INFO) << "Registering layer type: " << type;
    LayerRegistry<Dtype>::AddCreator(type, creator); // 实际添加
  }
};

#define REGISTER_LAYER_CREATOR(type, creator)                                  \
  static LayerRegisterer<float> g_creator_f_##type(#type, creator<float>);     \
  static LayerRegisterer<double> g_creator_d_##type(#type, creator<double>)    \

#define REGISTER_LAYER_CLASS(type)
// 首先定义这个函数                                                            \
  template <typename Dtype>                                                    \
  shared_ptr<Layer<Dtype> > Creator_##type##Layer(const LayerParameter& param) \
  {                                                                            \
    return shared_ptr<Layer<Dtype> >(new type##Layer<Dtype>(param));//!实际new \
  }
// 随后作为第二个参数，传入上边的define(##:嵌入string, #:转化为string类型)     \
  REGISTER_LAYER_CREATOR(type, Creator_##type##Layer)
{% endhighlight %}

### 3. INSTANTIATE_CLASS - 模板类实例化
// template的另一个问题：通过实例化，解决实现模板在.cpp与.h的分离
{% highlight C++ %}
// Instantiate a class with float and double specifications.
#define INSTANTIATE_CLASS(classname) \
  char gInstantiationGuard##classname; \
  template class classname<float>; \
  template class classname<double>
{% endhighlight %}
经过实例化，编译器会生成相应type的代码, 否则cpp中并不知道实例化成float或什么

### 4. SetUP Layer - 使用
例如新添加DispatchLayer层，在dispatch_layer.cpp底部需添加如下宏
{% highlight C++ %}
INSTANTIATE_CLASS(DispatchLayer);
REGISTER_LAYER_CLASS(Dispatch);
{% endhighlight %}
其#define经过编译后变成如下代码:
{% highlight C++ %}
template <typename Dtype>
shared_ptr<Layer<Dtype> > Creator_DispatchLayer(const LayerParameter& param) {
  return shared_ptr<Layer<Dtype> >(new DispatchLayer<Dtype>(param));
}
// 因为有全局静态变量的定义，在其初始化时自动完成了create layer的注册，Dispatch和Creator_DispatchLayer这个函数对应
static LayerRegisterer<float> g_creator_f_Dispatch("Dispatch", Creator_DispatchLayer<float>);
// AddCreator完成了注册Register：将上述的pointer存入map: registry[type] = creator
{% endhighlight %}
在net.cpp中通过以下代码，将上述类实例化为对象后，就添加到layers_中进行管理
{% highlight C++ %}
// 调用registry[type](param)，取出函数pointer，传入param，也就是实际调用了Creator_DispatchLayer函数，返回了shared_ptr<Layer<Dtype> >
layers_.push_back(LayerRegistry<Dtype>::CreateLayer(layer_param));
{% endhighlight %}
If the layer is going to be created by another creator function, 可以直接调用REGISTER_LAYER_CREATOR
{% highlight C++ %}
// Get convolution layer according to engine.
// 自定义creator
template <typename Dtype>
shared_ptr<Layer<Dtype> > GetConvolutionLayer(
    const LayerParameter& param) {
  ...
  if (engine == ConvolutionParameter_Engine_CAFFE) {
    return shared_ptr<Layer<Dtype> >(new ConvolutionLayer<Dtype>(param));
#ifdef USE_CUDNN
  } else if (engine == ConvolutionParameter_Engine_CUDNN) {
    return shared_ptr<Layer<Dtype> >(new CuDNNConvolutionLayer<Dtype>(param));
#endif
  } else {
    LOG(FATAL) << "Layer " << param.name() << " has unknown engine.";
  }
}
REGISTER_LAYER_CREATOR(Convolution, GetConvolutionLayer);
{% endhighlight %}

### 5. Targets.cmake - 编译
[参考](https://www.bfilipek.com/2018/02/static-vars-static-lib.html)
{% highlight Bash shell scripts %}
# Defines global Caffe_LINK flag, This flag is required to prevent linker from excluding
# some objects which are not addressed directly but are registered via static constructors
macro(caffe_set_caffe_link)
  if(BUILD_SHARED_LIBS)
    set(Caffe_LINK caffe)
  else()
    if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
      // -force_load，可以防止编译优化: 否则全局没有调用过的变量，如g_creator_f_Dispatch可能不会被引入
      // include all symbols from a static library
      set(Caffe_LINK -Wl,-force_load caffe)
    elseif("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
      // linux下，whole-archive作用于其后所有的lib，所以得no-whole-archive
      set(Caffe_LINK -Wl,--whole-archive caffe -Wl,--no-whole-archive)
    endif()
  endif()
endmacro()
{% endhighlight %}
