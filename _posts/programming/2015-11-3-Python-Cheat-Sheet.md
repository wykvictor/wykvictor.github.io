---
layout: post
title:  "Python Cheat Sheet"
date:   2015-11-3 17:00:00
tags: [python, programming, language]
categories: Programming
---

> 教程  [code codecademy python][link] 

[link]: https://www.codecademy.com/learn/python

1. name = raw_input("What is your name")   # 获取用户输入
2. 逻辑运算符: not, and, or
3. list中：append多用于把元素作为一个整体插入；insert多用于固定位置插入；extend多用于list中多项分别插入;
   remore删除第一个匹配的元素；del根据index删除，返回删除后的list；pop也是index，但是返回的是删除的元素
4. "-".join(list)  # 在list中的元素间 添加-连接起来
5. while/else, for/else： 循环正常退出（无break），else会执行
6. print char, “,”保证输出不换行
7. for index, item in enumerate(choices): # 同时获取index和item
8. zip能创建2个以上lists的pair/tuple等，在最短的list的end处停止。
9. a, b = 1, 2;  a, b = str.split('-') # 逗号分别赋值
10. print a, b 等同 print a + " " + b，但后者使用了字符串连接符，可能类型不匹配，前者更好
11. lists[::-1]，代表lists逆序
12. [i for i in my if i%3==0] 等同 filter(lambda i: i%3==0, my)  # 重组list列表
13. with语句：如果对象包括方法__enter__()和__exit__()，则可以用，进行自动关闭文件；线程锁的自动获取和释放；异常的捕捉和打印；
14. 类的没有参数的方法，是静态方法，属于这个类；实例对象的调用，需要有个self参数，a.func()-->func(a)
15. __slots__()用来限制该class能添加的属性
16. python是动态语言，比如一个类，可以在运行过程中随时添加属性
{% highlight Python %}
class layer():
    def __init__(self):
        pass
# 1. 添加类属性
layer.a = 2
print(layer.a)
# 2. 添加对象属性
l = layer()
l.b = 3
print(l.a, l.b) # 2 3
# 3. 通过类添加方法对象
def new_func(test):
    print(test)
    pass
layer.func = new_func
l.func()
# layer.func()  error, missing 1 required positional argument: 'test'
layer().func()  # 把临时对象作为参数传进new_forward
# 4. 通过类对象，添加方法对象，只能该对象自己用
import types
# 不能用直接赋值的方式给一个实例对象添加函数，通过MethodType绑定
l.func2 = types.MethodType(new_func, l)
l.func2()  # 与l.func()打印的是同一个对象l的地址
{% endhighlight %}
