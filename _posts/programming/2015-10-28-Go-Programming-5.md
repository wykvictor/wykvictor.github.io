---
layout: post
title:  "Learning Go Programming Language - 5 Concurrancy"
date:   2015-10-28 15:40:00
tags: [go, programming, language]
categories: programming
---

### 1. channel管道
默认情况下，在另一端准备好之前，发送和接收都会阻塞，使得go并发进行同步
{% highlight Go %}
func sum(a []int, c chan int) {
	sum := 0
	for _, v := range a {
		sum += v
	}
	c <- sum // 将和送入channel
}
func main() {
	a := []int{7, 2, 8, -9, 4, 0}
	c := make(chan int, 2)  // c是有类型的管道，使用前必须创建; 数字是缓冲区大小
	go sum(a[:len(a)/2], c)
	go sum(a[len(a)/2:], c)
	x, y := <-c, <-c // 从 c 中获取，箭头代表方向
	fmt.Println(x, y, x+y)  // -5 17 12，channel实现了 同步；输出顺序不重要!!!go线程调度的问题
}
{% endhighlight %}