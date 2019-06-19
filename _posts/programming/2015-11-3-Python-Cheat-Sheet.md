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
   remove删除第一个匹配的元素；del根据index删除，返回删除后的list；pop也是index，但是返回的是删除的元素
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
15. if __name__ == '__main__':  如果直接执行，为True；如果被其他module import，为False
16. 重载运算符：__add__(self,other), __mul__, __gt__(self,other), __lt__, __ge__, __le__, __pos__(self), __int__(self,类型转换)
17. __str__(self)定义当str()调用的时候的返回值，如print a；__repr__(self)定义repr()被调用的时候的返回值，如a

[更多magic方法的介绍](http://pycoders-weekly-chinese.readthedocs.io/en/latest/issue6/a-guide-to-pythons-magic-methods.html)

18. __iter__(self)函数，会返回一个[迭代器](https://www.ibm.com/developerworks/cn/opensource/os-cn-python-yield/index.html)
* 使用yield: 带有yield的函数在Python中被称之为generator：生成器
* 函数内部用for循环包含yield语句，执行到yield var时，函数就返回一个迭代值var，下次迭代时，代码从yield var 的下一条语句继续执行，而函数的本地变量看起来和上次中断执行前是完全一样的，于是函数继续执行，直到再次遇到yield

19. python是动态语言，比如一个类，可以在运行过程中随时添加属性
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
__slots__()可以用来限制该class能添加的属性

20. 正则表达式re
{% highlight Python %}
>>> line = 'training:  [epoch 301/total epoch 800][batch_idx 100/total_batch_idx 195]  Total #    batch: 117101    Loss: 0.1796, accuracy: 93.64 %'
>>> retest = re.search('\[epoch\s(\d+)\/total\sepoch\s(\d+)\]\[.*\s(\d+)\/.*\s(\d+)\].*batch:\s(\d+).*Loss:\s(\d+\.\d+).*accuracy:\s(\d+\.\d+).*', line)  # search不必从开头开始匹配，而且表达式必须消除歧义： .*\s(\d+)，加入了\s,否则只匹配最后一个数字0
>>> retest.groups()  # 打印出括号里指定的内容
('301', '800', '100', '195', '117101', '0.1796', '93.64')
{% endhighlight %}

21. 全局/局部变量
* 函数内部的变量名如果第一次出现，且出现在=前面，即被视为定义一个局部变量
* 函数内部的变量名如果第一次出现，且出现在=后边，则视为引用全局的变量
* 函数内部如果想把全局变量，放到=前边赋值，需要加global修饰
* file1.py的global_var，在file2.py中，需要import file1, file1.global_var这样用

