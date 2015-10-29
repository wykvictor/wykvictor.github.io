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

### 2. channel的range和close
{% highlight Go %}
func fibonacci(n int, c chan int) {
	x, y := 0, 1
	for i := 0; i < n; i++ {
		c <- x
		x, y = y, x+y
	}
	close(c)  // 关闭channel，中断main中的range操作
}
func main() {
	c := make(chan int, 10)
	go fibonacci(cap(c), c)
	for i := range c {  // 不断从c拿值，直到被close。v, ok := <-ch，ch关闭后ok为false了
		fmt.Println(i)
	}
}
{% endhighlight %}

### 3. select 语句
select语句使得一个goroutine在多个通讯操作上等待。哪个ready执行哪个
{% highlight Go %}
func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit:  // 强制等待
			fmt.Println("quit")
			return
		}
	}
}
func main() {
	c := make(chan int)
	quit := make(chan int)
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)  // 先打印完了10次c，才有quit
		}
		quit <- 0
	}()
	fibonacci(c, quit)
}  // output: 0 1 1 2 3 5 8 13 21 34 quit
{% endhighlight %}
定时器例子：
{% highlight Go %}
func main() {
	tick := time.Tick(100 * time.Millisecond)  // 隔100ms执行一次
	boom := time.After(500 * time.Millisecond)
	for {
		select {
		case <-tick:
			fmt.Println("tick.")
		case <-boom:
			fmt.Println("BOOM!")
			return
		default:  // 当select中其他条件分支都没有准备好时，default分支执行
			fmt.Println("    .")
			time.Sleep(50 * time.Millisecond)
		}
	}
}
{% endhighlight %}