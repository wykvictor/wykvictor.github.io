---
layout: post
title:  "PyTorch Basics"
date:   2017-01-29 16:00:00
tags: [pytorch, deep learning]
categories: ML
---

[PyTorch Doc](http://pytorch.org/docs/master/torch.html)

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
