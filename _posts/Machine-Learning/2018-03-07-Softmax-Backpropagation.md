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

### 偏导数推倒
![ml_softmax](/res/ml_softmax.png)

其实也可以这样理解：

1. 若网络输出三个类别softmax值分别是0.1, 0.2, 0.7，ground true label为0，0，1，那么loss即为0.1, 0.2, -0.3
2. 这也符合直觉，因为前2类loss是为了惩罚，让weight的偏导数减少，就是0.1-0和0.2-0，最后1类loss是为了补偿，0.7-1=-0.3,让weight的偏导数增大
3. 总结下，softmax with loss相对于之前最后一个节点的偏导，就是网络计算的该点输出的softmax值，减去对应的ground truth label(0或者1)

#### 附件，常用导数公式
![ml_deviritives](/res/ml_deviritives.png)