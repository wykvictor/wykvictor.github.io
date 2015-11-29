---
layout: post
title:  "Python Cheat Sheet - 1"
date:   2015-11-3 17:00:00
tags: [python, programming, language]
categories: programming
---

> 教程  [code codecademy python][link] 

[link]: https://www.codecademy.com/learn/python

{% highlight Python %}
1. name = raw_input("What is your name")   # 获取用户输入
2. 逻辑运算符: not, and, or
3. list中：append多用于把元素作为一个整体插入；insert多用于固定位置插入；extend多用于list中多项分别插入
   remore删除第一个匹配的元素；del根据index删除，返回删除后的list；pop也是index，但是返回的是删除的元素
4. "-".join(list)  # 在list中的元素间 添加-连接起来
5. while/else, for/else： 循环正常退出（无break），else会执行
6. print char, “,”保证输出不换行
7. for index, item in enumerate(choices): # 同时获取index和item
8. zip能创建2个以上lists的pair/tuple等，在最短的list的end处停止。
{% endhighlight %}