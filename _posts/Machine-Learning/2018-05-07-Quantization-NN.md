---
layout: post
title: "Quantization and Training of Neural Networks"
date: 2018-05-07 16:00:00
tags: [Quantization, Neural Networks]
categories: ML
---

> 参考论文[Quantization and Training of Neural Networks for Efficient Integer-Arithmetic-Only Inference](https://pdfs.semanticscholar.org/705c/0ca26b304f6403983356dbc3a70a22a64f80.pdf)

### 1. Weight quantization should be [-127, 127], not -128
原因有几点：
{% highlight Python %}
# 这样一来，量化体现在训练中的过程：
w_max = float((np.abs(weight_.cpu().numpy()[i])).max())
weight_[i] = weight_[i] * 2. / w_max
weight_[i] = torch.round(weight_[i] * 254 / 4.)
weight.data[i] = weight_[i] * 2 * w_max / 254.  # 量化损失后返回,继续训练

# 1. 如果是[-128~127]，则量化过程为：
weight_[i] = torch.round(weight_[i] * 255 / 4.)
肯定会有一个最大/小值是-127.5或127.5，这样round之后，出现最大可能的误差0.5。
从统计上来讲，增加了整体误差增大的概率，因为现在肯定有0.5的误差会出现

# 2. 除以255之后，出现-127.5和127.5，round后变成-128和128，cut之后(-128~127)，出现-128和127，也即-127.5->-128,127.5->127，整体会偏小。
这样网络上，一层一层往后，activation值会慢慢变小。
# (这一点需要实际验证)
{% endhighlight %}
另外，注意量化过程只在forward时做，做完后copy回原来的weight，也就是后向更新梯度时，用的是旧的weight，否则会造成weight没法学习

### 2. Activation Quantization
在输入层/激活层/AvgPooling/Hardtanh/Upsample等后，添加量化损失的过程：
{% highlight Python %}
out = round(out.data[:]*255./vmax) * vmax/255.  # vmax是浮点范围，如0～1/4/6等
{% endhighlight %}
** Note: Training时限制相同的浮点范围 **

比如所有层统一限制0～4，如果第一层输入image范围是0～1，输出是0～4，那会造成其BN层的alpha比其他层要大4倍，引入更大的round量化误差

### 3. Conv+BN层量化的处理
{% highlight Python %}
// 正常BN
Xout =  alpha * (Xin - mu)/sigma + beta
// 在实际计算中，4个参数可以简化为2个，并且消除除法运算(毕竟除法比加法慢3、4倍)
aa = alpha/sigma
bb = beta - (alpha * mu) / sigma
// 所以Conv+BN之后的结果：
Xout = aa * Xin + bb = aa * (Weight*Xconvin+Bias) + bb = aa * Weight*Xconvin + aa*Bias + bb  // 1
// 前边一般是Conv，而且Conv的Weight为了量化，要Scale到一定的范围，比如2
WeightQ = Weight * 2 / w_max  // 假设以2为边界量化到+-127
//此时BN之后的值：(根源是Weight变化，引起的变化)
XConvOutQ = Xconvin * WeightQ + Bias
Xout = aa * XConvOutQ + bb = aa * WeightQ * Xconvin + aa*Bias + bb  // 2
// 1和2必须等价，因此BN中的两个系数aa,bb需要替换为新的aaQ和bbQ。同时，去掉Conv中的Bias，放到bbQ里。令
aaQ = aa * w_max/2  // Weight的系数，乘到BN中
bbQ = aa * Bias + bb  // Bias转到BN中
// 此时
Xout = aaQ * WeightQ * Xconvin + bbQ = aa * w_max/2 * Weight*2/w_max * Xconvin + aa*Bias+bb  // 与1完全相同
{% endhighlight %}
