---
layout: post
title:  "NN Basics - Softmax Calculation && Backpropagation"
date:   2018-03-07 16:00:00
tags: [NN, Softmax, backpropagation]
categories: ML
---

> 转自[详解softmax函数以及相关求导过程](https://zhuanlan.zhihu.com/p/25723112)

> 若该分类ai的label为1，则偏导数为ai-1

> 若该分类aj的label为0，则偏导数为aj

### 1. 偏导数推倒
![ml_softmax](/res/ml_softmax.png)

其实也可以这样理解：

1. 若网络输出三个类别softmax值分别是0.1, 0.2, 0.7，ground true label为0，0，1，那么loss即为0.1, 0.2, -0.3
2. 这也符合直觉，因为前2类loss是为了惩罚，让weight的偏导数减少，就是0.1-0和0.2-0，最后1类loss是为了补偿，0.7-1=-0.3,让weight的偏导数增大
3. 总结下，softmax with loss相对于之前最后一个节点的偏导，就是网络计算的该点输出的softmax值，减去对应的ground truth label(0或者1)


### 2.[CrossEntropyLoss](http://sshuair.com/2017/10/21/pytorch-loss/)
combines LogSoftMax and NLLLoss in one single class，也就是说我们的网络不需要在最后一层加任何输出层，该loss Function为我们打包好了
{% highlight Python %}
import torch
import torch.nn as nn
import torch.nn.functional as F
import torch.autograd as autograd
import numpy as np

# 预测值f(x) 构造样本，神经网络输出层
inputs_tensor = torch.FloatTensor( [
 [10, 2, 1,-2,-3],
 [-1,-6,-0,-3,-5],
 [-5, 4, 8, 2, 1]
 ])
# 真值y
targets_tensor = torch.LongTensor([1,3,2])

inputs_variable = autograd.Variable(inputs_tensor, requires_grad=True) 
targets_variable = autograd.Variable(targets_tensor)

loss = nn.CrossEntropyLoss()
output = loss(inputs_variable, targets_variable)
print(output)  # tensor(3.7925, grad_fn=<NllLossBackward>)

# 可以看到结果3.7925，下面手动计算
# 手动计算log softmax, 计算结果的值域是[0, 1]
softmax_result = F.softmax(inputs_variable) #.sum() #计算softmax
logsoftmax_result = np.log(softmax_result.data)  # 计算log，以e为底, 计算后所有的值都小于0
softmax_result = F.log_softmax(inputs_variable)  # 直接调用F.log_softmax，结果和logsoftmax_result是一样的
>>> print(softmax_result)
tensor([[ -0.0005,  -8.0005,  -9.0005, -12.0005, -13.0005],
        [ -1.3555,  -6.3555,  -0.3555,  -3.3555,  -5.3555],
        [-13.0215,  -4.0215,  -0.0215,  -6.0215,  -7.0215]],
       grad_fn=<LogSoftmaxBackward>)

# 根据3类的真值，[1,3,2]，求最终的loss
sample_loss1 = -softmax_result[0,1]
sample_loss2 = -softmax_result[1,3]
sample_loss3 = -softmax_result[2,2]  # tensor(8.0005) tensor(3.3555) tensor(0.0215)
final_loss = (sample_loss1 + sample_loss2 + sample_loss3)/3  # tensor(3.7925, grad_fn=<DivBackward0>)
# 手动计算结果相同3.7925

# 另外，nn.NLLLoss()是Negative Log Liklihood，也可以得到相同结果，只不过输入是log softmax之后的结果,就是加和求了个平均
loss2 = nn.NLLLoss()
output = loss2(softmax_result, targets_variable)  # tensor(3.7925, grad_fn=<NllLossBackward>)

# NLLLoss2d用于计算二维的损失函数，多用于语义分割，就是比如分20类，则出20个feature map，每一个像素都有20个类别的概率，真值是每一像素所属类别[0~19]的一个feature map。最终的loss是每个像素的log softmax，选取真值对应的类别，加起来求平均
{% endhighlight %}

### 3. 附件，常用导数公式
![ml_deviritives](/res/ml_deviritives.png)