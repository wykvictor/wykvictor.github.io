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
# 这样一来，量化过程：
w_max = float((np.abs(weight_.cpu().numpy()[i])).max())
weight_[i] = weight_[i] * 2. / w_max
weight_[i] = torch.round(weight_[i] * 254 / 4.)
weight_[i][weight_[i] < -127] = -127  # 这里是128也无所谓，达不到这个值
weight_[i][weight_[i] > 127] = 127
weight.data[i] = weight_[i] * 2 * w_max / 254.  # 量化损失后返回

# 1. 如果是[-128~127]，则量化过程为：
weight_[i] = torch.round(weight_[i] * 255 / 4.)
肯定会有一个最大/小值是-127.5或127.5，这样round之后，出现最大可能的误差0.5。
从统计上来讲，增加了整体误差增大的概率，因为现在肯定有0.5的误差会出现

# 2. 除以255之后，出现-127.5和127.5，round后变成-128和128，cut之后，出现-128和127，也即-127.5->-128,127.5->127，整体会偏小。
这样网络上，一层一层往后，activation值会慢慢变小。
# (这一点需要实际验证)
{% endhighlight %}
