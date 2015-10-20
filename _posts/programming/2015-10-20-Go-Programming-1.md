---
layout: post
title:  "Learning Go Programming Language - 1 basic"
date:   2015-10-20 16:00:00
tags: [go, programming, language]
categories: programming
---

### 1. 包
{% highlight Go %}
package main  // Go程序都是由包组成的，入口是包main
import (  // 更好的 打包导入，也可以分开写
	"fmt"
	"math/rand"  // 包名与导入路径的最后一个目录一致，即rand
)
func main() {
	rand.Seed(5)  // 首字母大写的名称是被导出的, seed报错不允许导出
	fmt.Println("My favorite number is", rand.Intn(10))  // 逗号 自带空格
}
{% endhighlight %}

### 2. 函数
{% highlight Go %}
// 类型 在变量名 之后，且可以省略前边相同的类型
// 函数可以返回 任意数量的返回值
func swap(x, y string) (string, string) {
	return y, x
}
// 返回值可以被命名，并且像变量那样使用
func split(sum int) (x int, y int) {
	x  = sum * 4 / 9
	y  = sum - x
	return  // 没有参数的 return 语句返回结果的当前值。也就是`直接`返回 return x, y
}
func main() {
	a,b := swap("hello", "world")
	fmt.Println(a, b)
	fmt.Println(split(17))
}
{% endhighlight %}

### 3. 变量
{% highlight Go %}
var c, pythonbool  // 变量列表，类型在后，必须有类型
var a, b = 3, 4  // 可以初始化，有且只有 每个变量对应一个，类型可以不写（编译器推导出来了）
func main() {
	var i int  // var的作用范围：函数内或外边的包内
	var s string
	fmt.Println(i, c, python, s)  // 默认值：0 false false ""
	x := "no!"  // := 简洁赋值语句，用于替代 var 定义; := 不能在函数外用

	var f float64 = math.Sqrt(float64(x*x + y*y))
	x = int(f)  // 不同类型之间的项目赋值需要显式转换，删掉int报错
}
{% endhighlight %}

### 4. 基本类型
Basic types：

bool

string

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr  //The int, uint, and uintptr types are usually 32 bits wide on 32-bit systems and 64 bits wide on 64-bit systems. 一般就使用int即可

byte // uint8 的别名

rune // int32 的别名，代表一个Unicode码

float32 float64

complex64 complex128
{% highlight Go %}
var (  // 变量的定义也可以“打包”在一个语法块中。
	ToBe  = false
	MaxInt uint64     = 1<<64 - 1
	z      complex128 = cmplx.Sqrt(-5 + 12i)
)
func main() {
	const f = "%T (%v)\n"  // printf函数，格式可以存成 const常量
	fmt.Printf(f, ToBe, ToBe)  // bool (false)
	fmt.Printf(f, MaxInt, MaxInt)  // uint64 (18446744073709551615)
	fmt.Printf(f, z, z)  // complex128 ((2+3i))
}
{% endhighlight %}