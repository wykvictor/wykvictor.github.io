---
layout: post
title:  "Yaw Pitch Roll && Transform matrix"
date:   2018-09-01 16:00:00
tags: [opencv, yaw, pitch, roll, transform matrix]
categories: CV
---

### 1. 二维坐标下角度=>矩阵，齐次坐标
![yawpitchroll](/res/yawpitchroll.png)

### 2. 推广到三维情况
yaw是绕y轴旋转；pitch是绕x轴旋转；roll是绕z轴旋转. 根据这3个值，可以显示出axis
显示axis相关代码：
{% highlight C++ %}
{% endhighlight %}

### 3. 平移*缩放*旋转矩阵