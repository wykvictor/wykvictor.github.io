---
layout: post
title:  "Learning Go Programming Language - 3 more types"
date:   2015-10-22 16:00:00
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

### 5. range 函数
对 slice 或者 map 进行迭代循环
{% highlight Go %}
var pow = []int{1, 2, 4, 8, 16, 32, 64, 128}
for i, v := range pow {  // index and value
	fmt.Printf("2**%d = %d\n", i, v)
}
for i := range pow  // only index
for _, value := range pow  // only value
{% endhighlight %}
exercise-slice:　将整数转换为灰度图片进行展示
{% highlight Go %}
func Pic(dx, dy int) [][]uint8 {
	ret := make([][]uint8, dy)  // 初始化二维数组 y * x
	for i := range ret {  // range y
		ret[i] = make([]uint8, dx)  // 初始化里边的一维元素！
		for j := range ret[i] {  // range x
			ret[i][j] = uint8(i^j)  // uint8 类型转换
		}
	}
	return ret
}
{% endhighlight %}

### 6. map
{% highlight Go %}
type Vertex struct {
	Lat, Long float64
}
var m = map [string] Vertex{  // 这样定义
	"Bell Labs": Vertex{
		40.68433, -74.39967,
	},
	"Google": {37.42202, -122.08408},  // 可以省略
}
example:
func main() {
	m := make(map[string]int)  // map在使用之前必须用make而不是new来创建；值为nil的map是空的，并且不能赋值。

	m["Answer"] = 42  // 插入 or 修改
	fmt.Println("The value:", m["Answer"])

	delete(m, "Answer")  // 删除
	fmt.Println("The value:", m["Answer"])

	v, ok := m["Answer"]  // 判断：如key在m中，ok为true。否则ok为 false并且elem是map的元素类型的零值。
	fmt.Println("The value:", v, "Present?", ok)
}
m = make(map[string]Vertex)
{% endhighlight %}
exercise-map:　wordcount
{% highlight Go %}
func WordCount(s string) map[string]int {
	res := make(map [string]int)
	for _,v := range strings.Fields(s) {  // Fields函数，类似Python的Split
		res[v]++
	}
	return res
}
{% endhighlight %}

### 7. 函数值
函数可以作为值赋给变量，类似函数式编程的匿名函数
{% highlight Go %}
hypot := func(x, y float64) float64 {  	//无名函数赋值给变量hypot
	return math.Sqrt(x*x + y*y)
}
fmt.Println(hypot(3, 4))  // 5
{% endhighlight %}
函数闭包：对象是附有行为的数据，而闭包是附有数据的行为
{% highlight Go %}
func adder() func(int) int {  // 函数 adder 返回一个闭包,每个返回的闭包都被绑定到其各自的 sum 变量上
	sum := 0
	return func(x int) int {
		sum += x
		return sum
	}
}
func main() {
	pos, neg := adder(), adder()  // 函数被“绑定”在变量上，变量就好像函数指针
	for i := 0; i < 10; i++ {
		fmt.Println(
			pos(i),  // 0 1 3 6 10...
			neg(-2*i),  // 这2者结果是隔离的，每次调用adder返回的函数都将生成并保存一个新的局部变量sum
		)
	}
}
{% endhighlight %}

### 8. 闭包实例 - fibonacci
{% highlight Go %}
func fibonacci() func() int {
	num1, num2 := 0, 1
	return func() int {
		tmp := num2
		num2 += num1
		num1 = tmp
		return num2
	}
}
func main() {
	f := fibonacci()
	for i := 0; i < 10; i++ {
		fmt.Println(f())  // 1 2 3 5 8 13 21 34 55 89
	}
}
{% endhighlight %}