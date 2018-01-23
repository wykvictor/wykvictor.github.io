---
layout: post
title:  "Xcode Linker Path"
date:   2018-01-20 16:00:00
tags: [xcode, rpath, install name, tech]
categories: Tech
---

>  项目中遇到Xcode中的链接路径问题[参考](https://www.jianshu.com/p/cd614e080078)

### 1. install Name
绝对路径，告诉连接器运行时在哪里找到需要的库。

如对于libfoo.dylib，install name为/usr/lib/libfoo.dylib

### 2. executable_path
相对路径，@executable_path为当前APP的路径。

如，对于Bar.app，为/Applications/Bar.app/Contents/MacOS，
将lib的install name设为@executable_path/../Frameworks/Foo.framework/Versions/A/Foo即可

### 3. @rpath
上面的方案可行，但是如果某个lib A，既要放到/lib下，也可能被放到Application下，那么需要提供2个单独的不同install name的包。

@rpath可解决这一问题：类似于linux/win系统的path变量，可设置多个位置，依次搜索该库
