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

### 2. 接口
{% highlight Go %}
type Abser interface {  // 一组方法定义的集合
	Abs() float64
}
var a Abser  // a可以赋值：实现这些方法如Abs()的任何值
// 例子
type Person struct {
	Name string
	Age  int
}
func (p Person) String() string {  //`fmt`包使用接口Stringer中的String()方法来进行输出
	return fmt.Sprintf("%v (%v years)", p.Name, p.Age)
}
func main() {
	a := Person{"Arthur Dent", 42}
	fmt.Println(a)  // Arthur Dent (42 years), 相当于 重载了cout
}
{% endhighlight %}


### 3. 异常
{% highlight Go %}
i, err := strconv.Atoi("42")
if err != nil {  // 非空表示失败
    fmt.Printf("couldn't convert number: %v\n", err)
}
fmt.Println("Converted integer:", i)
{% endhighlight %}
Exercise - error: 实现Sqrt使其接收到一个负数时，返回一个非nil错误值
{% highlight Go %}
type ErrNegativeSqrt float64
func (e ErrNegativeSqrt) Error() string {
	return "cannot Sqrt negative number:" + fmt.Sprint(float64(e))
}
func Sqrt(x float64) (float64, error) {
	if(x < 0) {
		return x, ErrNegativeSqrt(x)
	}
	z := float64(1)
	for {
		y := (z + x/z) / 2.0
		fmt.Println(y)
		if math.Abs(z - y) < 1e-10 {
			break
		}
		z = y
	}
	return z, nil
}
func main() {
	fmt.Println(Sqrt(2))
	fmt.Println(Sqrt(-2))  // fmt包在输出时也会试图匹配 error接口
}
{% endhighlight %}