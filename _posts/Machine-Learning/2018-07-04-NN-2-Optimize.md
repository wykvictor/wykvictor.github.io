---
layout: post
title:  "NN Basics 2 - Optimize Algorithm"
date:   2018-07-04 16:00:00
tags: [NN, Optimize]
categories: ML
---

> Parameters Update算法：SGD, Momentum, AdaGrad, RMSProp, Adam

> [优化方法总结](https://blog.csdn.net/u014595019/article/details/52989301)

> [Deep Learning之最优化方法](https://blog.csdn.net/BVL10101111/article/details/72614711) ： 优劣分析到位

> CS231N, winter1516_lecture6：有代码实现，比较好

> 手动指定学习率: SGD,Momentum,Nesterov Momentum; 自动调节学习率：AdaGrad, RMSProp, Adam

### 1. SGD (stochastic gradient descent)
{% highlight Python %}
for i in range(1, batchN):
	data_batch  = dataset.sample_data_batch()
	loss = network.forward(data_batch)
	dx = network.backward()
	x  = x - learning_rate * dx
{% endhighlight %}
Bad：
1. 当loss function在x方向宽，y方向很窄时(椭圆)，x下降很慢，y容易上下抖动
2. 因为每次更新都是抽样的样本，得到的梯度dx有误差，learning_rate要逐渐减小，否则无法收敛。一般是进行线性衰减

### 2. Momentum
{% highlight Python %}
# x  = x - learning_rate * dx
v = mu * v - learning_rate * dx  # mu(0.5, 0.9, or 0.99)
x = x + v
{% endhighlight %}
Good:
1. 在梯度dx宽的方向如x进行累积加速，在窄的方向y进行抑制（因为dx符号在震荡）
2. 易知v = -learning_rate * dx / （1 - mu），相比于SGD，类似于学习率加速了1/(1-mu)倍

### 3. Nesterov Momentum
对Momentum的一种改进,先对参数进行估计,然后使用估计后的参数来计算误差,计算Loss时用x+mu*v
{% highlight Python %}
v_prev = v
v = mu * v - learning_rate * dx 
x = x + (-mu * v_prev) + (1+mu) * v # equal: x + v + mu * (v - v_prev) 
{% endhighlight %}
TODO: 不太懂最后更新x的公式

### 4. AdaGrad
在SGD基础上，将dx做了历史的累积，最新的dx要除以累加和
{% highlight Python %}
cache += dx**2
x = x - learning_rate * dx / (np.sqrt(cache) + 1e-7) 
{% endhighlight %}
Good:
1. 实现学习率的自动更改，慢慢衰减。如果这次梯度大,那么学习率衰减快一些;如果梯度小,那么就慢一些
Bad：
1. 经验表明，在普通算法中也许效果不错，但在深度学习中，深度过深时会造成训练提前结束，step size趋近与0

### 5. RMSProp
在AdaGrad基础上，加了衰减系数decay_rate,可以为0.9
{% highlight Python %}
cache = dacay_rate * cache + (1-decay_rate) * dx**2  # 只有这里不同
x = x - learning_rate * dx / (np.sqrt(cache) + 1e-7) 
{% endhighlight %}
Good:
1. 相比于AdaGrad, 很好的解决了深度学习中过早结束的问题 
Bad：
1. 引入了新的超参数，dacay_rate，一般可以使用默认值

### 6. Adam (Adaptive Moment Estimation)
在RMSProp基础上，加了Momentum, 利用梯度的一阶估计和二阶估计动态调整每个参数的学习率
{% highlight Python %}
m = 0
v = 0
for i in range(1, batchN):
	data_batch  = dataset.sample_data_batch()
	loss = network.forward(data_batch)
	dx = network.backward()

	m = beta1 * m + (1 - beta1) * dx  # beta1可以为0.9，只有这一行和RMSProp不同，类似于加了Momentum
	v = beta2 * v + (1 - beta2) * dx**2  # beta2可以为0.999
	# 下面两行，只是为了补偿冷启动，就是刚开始m v是0的问题
	m = m / (1 - beta1 ** i)  # i越来越大，也就是分母越来越大
	v = v / (1 - beta2 ** i)
	x = x - learning_rate * m / (np.sqrt(v) + 1e-7) 
{% endhighlight %}
Good:
1. 每一次迭代，学习率都有个确定范围，使得参数比较平稳，loss曲线也比较平稳
Bad:
1. 跟RMSProp和SGD相比，能够较快收敛但是最终收敛结果可能没有这两种好。比如RMSProp的loss曲线虽然较为震荡，但在某些learning_rate的设置情况下，能够达到比Adam更加低的loss

### 7. Weight Decay
weight decay（权值衰减）的使用既不是为了提高收敛精确度也不是为了提高收敛速度，其最终目的是防止过拟合。
在损失函数中，weight decay是放在正则项（regularization）前面的一个系数。
正则项一般指示模型的复杂度，所以weight decay的作用是调节模型复杂度对损失函数的影响，若weight decay很大，则复杂的模型Loss也就大。

W(t+1) = W(t) − lr ∗ delta(W) − lr ∗ weight_decay ∗ W(t) = (1-lr ∗ weight_decay) * W(t) - lr ∗ delta(W)

[公式参考链接](https://stats.stackexchange.com/questions/29130/difference-between-neural-net-weight-decay-and-learning-rate)

但是Weight Decay不等同于L2正则化, [paper](https://arxiv.org/abs/1711.05101) TODO: 待分析

公式表明weight的更新相当于先缩小了一个比例，再按常规梯度下降进行。这样某些神经元可能由于weight不断减小就不激活了，类似dropout的效果。但是有用的神经元，还是会靠梯度下降得到激活。
