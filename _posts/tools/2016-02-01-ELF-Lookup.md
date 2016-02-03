---
layout: post
title:  "How to lookup ELF"
date:   2016-02-01 18:30:00
tags: [elf, readelf, nm, ndk-depends]
categories: Tools
---

### 1. [nm](http://www.cnblogs.com/itech/archive/2012/09/16/2687423.html)
列出符号信息，包括诸如符号的值，符号类型及符号名称等，指定义出的函数，全局变量等等。

### 2. [readelf](http://man.linuxde.net/readelf)
显示目标文件的信息，依赖的库的情况
{% highlight Bash shell scripts %}
readelf -a main | less  # 全面信息，可以匹配搜索
readelf -d main  # 只列出依赖的动态库
{% endhighlight %}

### 3. ndk-depends
android交叉编译可执行文件，可以用此命令查看依赖库
{% highlight Bash shell scripts %}
ndk-depends main  # 用readelf也可以
{% endhighlight %}