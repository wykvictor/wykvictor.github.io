---
layout: post
title:  "Learning Go Programming Language - 4 Methods and Interfaces"
date:   2015-10-23 10:00:00
tags: [go, programming, language]
categories: programming
---

### 1. 方法
{% highlight Go %}
func (v *Vertex) Abs() float64 {  // 可以在结构体类型上定义方法，func和方法名中间
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
v.Abs()  // 这么调用
type MyFloat float64  // 任意类型都可以定义方法，但是要type一下
func (f MyFloat) Abs() float64 {...}
// 接收者为指针:避免在每个方法调用中拷贝,更有效率;方法可以修改接收者指向的值,否则修改副本没有用
func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}
{% endhighlight %}
