---
layout: post
title:  "PyTorch Basics"
date:   2017-01-29 16:00:00
tags: [pytorch, deep learning]
categories: ML
---

> [PyTorch Doc](http://pytorch.org/docs/master/torch.html)
> [60分钟入门](https://zhuanlan.zhihu.com/p/25572330)
> [Pytorch入门教程](https://www.jianshu.com/p/cbce2dd60120)
> [莫烦PyTorch教程](https://morvanzhou.github.io/tutorials/machine-learning/torch/)


### 1. Tensor张量
{% highlight Python %}
x = torch.Tensor(2, 3)  # 构造一个2x3的矩阵，没初始化但仍然会有值
y = torch.rand(2, 3)
# tensor加法, 4种方法
a + b
torch.add(a, b)
torch.add(a, b, out=result)  # 把运算结果存储在result上
b.add_(a)  # 把运算结果覆盖掉b

# 与numpy的array转换
b = a.numpy() # a为tensor
b = torch.from_numpy(a)  # a为numpy的array
# 其中，a和b共用一块内存，a变化时b也会变化

x = x.cuda()  # Tensor x使用cuda函数以后，所有运算会调用gpu
{% endhighlight %}

### 2. 在CPU上Load由GPU train的nn.DataParallel模型
该模型的key值前缀是module. 去掉即可
{% highlight Python %}
# lambda表达式，定义了函数: 传入2个值storage, loc作为参数，返回值是第一个值，就是cpu上的原始的序列化的位置
tmp_pose_model = torch.load(model_path, map_location=lambda storage, loc: storage)
old_state = tmp_pose_model.state_dict()
from collections import OrderedDict
new_state_dict = OrderedDict()
for k, v in list(old_state.items()):
    k = k.replace("module.", "")
    new_state_dict[k] = v
new_model = model.Model()
new_model.load_state_dict(new_state_dict)
# print(new_model.state_dict())
{% endhighlight %}

### 3. 其他
{% highlight Python %}
with torch.cuda.device(0):
	# 这里的操作，都是在cuda的device 0上
model.eval()  # Sets the module in evaluation mode
x.detach()    # 表示该Tensor不需要求梯度
{% endhighlight %}
