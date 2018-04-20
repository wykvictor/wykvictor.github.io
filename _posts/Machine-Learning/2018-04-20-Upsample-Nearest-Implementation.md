---
layout: post
title: "Upsample Nearest Implementation"
date: 2018-04-20 16:00:00
tags: [Upsample, nearest, pytorch, deep learning]
categories: ML
---

### 1. 通过Reshape和Concat结合实现Upsample
由于一些框架，不支持Upsample，可以用ReShape和Concat这种常见的Op替换
{% highlight Python %}
class MyUpSample(nn.Module):
    dump_patches = True
    def __init__(self, scale_factor = 2):
        super(MyUpSample, self).__init__()
        #self._upsample = nn.Upsample(scale_factor=scale_factor)
    def forward(self, x):
        #return self._upsample(x)
        ss = x.size()
        x = x.view(ss[0], ss[1], ss[2] * ss[3], 1)  # 1
        ups1 = torch.cat((x, x), 3)  # 2
        ups1 = ups1.view(ss[0], ss[1], ss[2], 2 * ss[3])  # 3
        ups2 = torch.cat((ups1, ups1), 3)  # 4
        ups2 = ups2.view(ss[0], ss[1], ss[2] * 2, ss[3] * 2)  # 5
        return ups2
{% endhighlight %}

其中代码注释中的5个步骤，分别对应下图中的5步：

![ml_upsample](/res/ml_upsample.jpeg)
