---
layout: post
title:  "Learning Go Programming Language - 3 more types"
date:   2015-10-21 16:00:00
tags: [go, programming, language]
categories: programming
---

### 1. 指针
与C语言相同，只是没有指针运算(如*void随意转换指针类型)

### 2. 结构体
{% highlight Go %}
type Vertex struct {  // 定义结构体 为一种新类型
	X int
	Y int
}
func main() {
	v := Vertex{1, 2}  // 这样用
	v.X = 4
	fmt.Println(v.X)  // 4
	p := &v
	p.X = 1e9  // (*p).X = 1e9 结构体指针，两种写法都对
	fmt.Println(v)

	v2 = Vertex{X: 1}  // 默认值Y:0
	fmt.Println(v2)  // 1, 0
}
{% endhighlight %}

### 3. 数组
{% highlight Go %}
var a [2]string  // 数组定义 var a[2] string 空格随意
a[0] = "Hello"
a[1] = "World"  // 也可以这样写 a := [...]string{"Hello", "World"}
fmt.Println(a)  // [Hello World]
{% endhighlight %}

### 4. slice
对底层数组Array的封装, 指针指向同一片数组空间,和数组不同的是切片并没有给定固定的长度。
{% highlight Go %}
s := []int{2, 3, 5, 7, 11, 13}
s[1:4] == [3 5 7]  // 切片，同python：包含前端，不包含后端
b := make([]int, 0, 5) // len(b)=0, cap(b)=5

var z []int  // z == nil
fmt.Println(z, len(z), cap(z))  // [] 0 0

a = append(a, 2, 3, 4)  // 如果超出cap，则指向重新copy的大数组
b := []int{1, 2, 3}
a = append(a, b...)  // ...语法 == "append(a, b[0], b[1], b[2])"
{% endhighlight %}
[read more][link-slice] 

[link-slice]: https://blog.go-zh.org/go-slices-usage-and-internals