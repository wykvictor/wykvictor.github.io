---
layout: post
title:  "Learning Go Programming Language - 2 flow control"
date:   2015-10-21 16:00:00
tags: [go, programming, language]
categories: programming
---

### 1. 循环 - 仅for一种
{% highlight Go %}
for i := 0; i < 10; i++ {  // 强制无(), { 强制放后边，也可以省略前、后置语句
	sum += i
}
for sum < 1000 {  // 类似“while”，{强制后边
	sum += sum
}
for {  // 省略循环条件, 死循环
}
{% endhighlight %}

### 2. if 语句
{% highlight Go %}
if x < 0 {  // 强制无(), { 强制放后边
	return sqrt(-x) + "i"
}
if v := math.Pow(x, n); v < lim {  // 跟for一样，if可以在条件判断中执行 一个 简单的语句
	return v  // v的作用域仅在 if 范围内
}
{% endhighlight %}

### 3. 实例-牛顿法求平方
exercise-loops-and-functions
{% highlight Go %}
func Sqrt(x float64) float64 {
	z := float64(1)
	for {
		y := (z + x/z) / 2.0
		fmt.Println(y)
		if math.Abs(z - y) < 1e-10 {
			break
		}
		z = y
	}
	return z
}
{% endhighlight %}

### 4. switch 语句
{% highlight Go %}
switch os := runtime.GOOS; os {  // 也可以用没有条件的 switch（同switch true 一样）
	case "darwin":
		fmt.Println("OS X.")
		// fallthrough  ??
	case "linux":
		fmt.Println("Linux.")
	default:
		fmt.Printf("%s.", os)
}
{% endhighlight %}

### 5. defer 语句
{% highlight Go %}
// 延迟调用的参数会立刻生成，但是在上层函数返回前函数都不会被调用
fmt.Println("counting")
for i := 0; i < 10; i++ {
	defer fmt.Println(i)  // 延迟函数压入一个栈，函数返回时按照后进先出的顺序调用被延迟的函数
}
fmt.Println("done")
//Output: counting done 9 8 7 ... 0
{% endhighlight %}