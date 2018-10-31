---
layout: post
title:  "Bilinear Interpolation"
date:   2018-10-01 16:00:00
tags: [bilinear, interpolation, align_corner, pyTorch]
categories: CV
---

### 1. 已知周边四个点，计算双线性差值
![interpalation](/res/interpalation.png)

### 2. 如何找到周边四个点坐标
常见方式：顶点对齐，在PyTorch和TenserFlow实现中，align_corner=True
{% highlight Python %}
# 计算方式
SrcX = dstX * (srcWidth-1）/ (dstWidth-1)
SrcY = dstY * (srcHeight-1）/ (dstHeight-1)
# pyTorch结果：https://github.com/pytorch/pytorch/blob/master/aten/src/THCUNN/linear_upsampling.h
>>> input = torch.arange(1, 5).view(1, 1, 2, 2).float()
>>> input
tensor([[[[ 1.,  2.],
          [ 3.,  4.]]]])
>>> m = nn.Upsample(scale_factor=2, mode='bilinear', align_corners=True)
>>> m(input)
tensor([[[[ 1.0000,  1.3333,  1.6667,  2.0000],
          [ 1.6667,  2.0000,  2.3333,  2.6667],
          [ 2.3333,  2.6667,  3.0000,  3.3333],
          [ 3.0000,  3.3333,  3.6667,  4.0000]]]])
# 例如，对于第0行，第1列，1.333 = 1 * 2/3 + 2 * 1/3

# align_corner=False，计算方式：
SrcX = (dstX+0.5) * (srcWidth/dstWidth) - 0.5 
SrcY = (dstY+0.5) * (srcHeight/dstHeight) - 0.5
# 同样的，对于第0行，第1列
src1 = (1 + 0.5) * (2/4) - 0.5 = 0.25
则 1.25 = (1 - 0.25) * 1 + 0.25 * 2 = 0.75+ 0.5 = 1.25
>>> m = nn.Upsample(scale_factor=2, mode='bilinear')  # align_corners=False
>>> m(input)
tensor([[[[ 1.0000,  1.2500,  1.7500,  2.0000],
          [ 1.5000,  1.7500,  2.2500,  2.5000],
          [ 2.5000,  2.7500,  3.2500,  3.5000],
          [ 3.0000,  3.2500,  3.7500,  4.0000]]]])
{% endhighlight %}
对于定点对齐，在pyTorch和TenserFlow能得到一致的结果

但是如果为False，可能实现不同结果不同，其原理是图像的中心点对齐

其中，pytorch的align_corners实现[link](https://github.com/pytorch/pytorch/blob/227c8f2654c5aa4dda319d7401d319bb4636be47/aten/src/THCUNN/linear_upsampling.h)
相应的代码[merge](https://github.com/pytorch/pytorch/pull/5927)
