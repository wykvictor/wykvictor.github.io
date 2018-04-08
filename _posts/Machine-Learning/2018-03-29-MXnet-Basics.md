---
layout: post
title:  "MXnet Basics"
date:   2018-03-29 16:00:00
tags: [pytorch, deep learning]
categories: ML
---

### 1. 导入json模型
{% highlight Python %}
# 加载model-mxnet-symbol.json和model-mxnet-0000.params文件
sym, arg_params, aux_params = mx.model.load_checkpoint('model-mxnet', 0)
model = mx.mod.Module(symbol = sym)
model.bind(for_training=False, data_shapes=[('data', (1,3,256,256))])
model.set_params(arg_params, aux_params)
input = nd.ones(shape=(1, 3, 256, 256))
Batch = namedtuple('Batch', ['data'])  # warp in tuple
model.forward(Batch([input]))
output = model.get_outputs()[0]
print('shape', output.shape, 'output:', output)
{% endhighlight %}

### 2. 统计模型信息：计算量、参数量
{% highlight Python %}
# 计算量
def traverse_net(net, data = (1,3,256,256)):
    net_internals = net.symbol.get_internals()
    list_conv = []
    for i, node in enumerate(net_internals):
        # print("index:%d,node name:%s"%(i, node.name))
        if node.name == "data":  # the first layer
            continue
        elif "conv" in node.name:  # 只统计conv的计算量
            if node.get_children() is None:  # 无输入，说明不在网络中
                continue
            else:  # 计算
                arg_shape, out_shape, _ = node.infer_shape_partial(data = data)
                if len(out_shape[0])==0:
                    continue
                else:
                    # print("Name:%s" % node.name)
                    input_channel = 0
                    for child in node.get_children():
                        if child.name == "data":
                            input_channel += 3
                        if child.get_children() is None: # this is weight node
                            continue
                        arg_shape, chile_out_shape, _ = child.infer_shape_partial(data=data)
                        if len(chile_out_shape[0]) > 0:
                            input_channel += chile_out_shape[0][1]
                    kernel_ops = float(node.list_attr()['kernel'][1])**2  * (input_channel / int(node.list_attr()['num_group']))
                    bias_ops = 0 if node.list_attr()['no_bias'] == 'True' else 1
                    flops = np.prod(out_shape[0]) * (kernel_ops + bias_ops)
                    list_conv.append(flops)
                    # print(" Output_shape:", out_shape)
                    # print(" Attr:", node.list_attr())
    print('total_compuation:', sum(list_conv) / 1e6, 'M')

# 参数量，统计的是trainable params，比如bn只有2个
model.initialize(mx.initializer.Constant(0.01))
y = model(inputs)  # MXnet是延迟初始化，所以必须forward一次后，param才真正初始化了
params = model.collect_params()
sum([np.prod(params[i].shape) for i in pose_model.collect_params() if params[i]._differentiable])
{% endhighlight %}

### 3. 测试模型速度
{% highlight Python %}
# 注意添加同步操作！否则时间测量很小
import time
def measure(model, input, n_time):
    out = model.forward(input)
    tic = time.time()
    for i in range(n_time):
        model.forward(input)
    output = model.get_outputs()[0]
    out_np = output.asnumpy()  # 必须调用（包含了同步操作，否则测不准）。mxnet是异步调用的，forward函数提交到了队列
    cost = time.time() - tic
    return cost / n_time
{% endhighlight %}