---
layout: post
title:  "Xcode Linker/Excutable Path"
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

### 4. [cmake设置rpath相关](https://gitlab.kitware.com/cmake/community/wikis/doc/cmake/RPATH-handling)
1. 如果设置cmake_minimum_required (VERSION 2.6)，会报错：
  * Policy CMP0042 is not set: MACOSX_RPATH is enabled by default.  Run "cmake
  --help-policy CMP0042" for policy details. 
  * 但是大于2.8.12的cmake才完整支持rpath，所以会报错，一般设置VERSION 3.1

2. 可以通过set(CMAKE_MACOSX_RPATH 0)手动禁止rpath，此时用otool -L查看依赖，是全路径指向依赖库，不具备可移植性

3. 但是rpath是个好东西，不建议关掉，依赖是通过rpath变量动态控制	的：
@rpath/libQt5Widgets.5.dylib

4. make install的时候，cmake默认会去掉可执行文件的rpath设定，所以需要我们自己设置加上：
{% highlight Bash shell scripts %}
# 在cmake中，做如下设置，make install的时候，会将link时的路径和lib放到target中
SET(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
SET(CMAKE_INSTALL_RPATH "lib")
# 此时，通过 otool -l executable | grep LC_RPATH -A 2，可以看到rpath的设定：
cmd LC_RPATH
cmdsize 16
path lib  # SET(CMAKE_INSTALL_RPATH "lib") 设置的

cmd LC_RPATH
cmdsize 40
path /Users/a/anaconda3/lib  # SET(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
{% endhighlight %}

### 5. 设置rpath技巧
1. rpath也可以设置为@loader_path和@executable_path/lib，指向可执行程序同一目录

2. install_name_tool -change ... -rpath ... -add_rpath ... -delete_rpath ... 可以修改RPATHs


### 6. 实现脚本的双击执行
1. 将脚本后缀设置为.command

2. 注意：双击后，默认运行目录为用户跟目录，并非可执行程序目录，如果想修改，可以添加：
{% highlight Bash shell scripts %}
cd -- "$(dirname "$0")"
# do the real thing
{% endhighlight %}

3. 如果程序中，需要有文件操作，如何获得可执行文件所处目录：
  * QT接口：QCoreApplication::applicationDirPath()
  * MacOS接口:#include <mach-o/dyld.h>  _NSGetExecutablePath(path, &size)
  * Linux: readlink /proc/self/exe
  * Windows: GetModuleFileName()

4. QT程序运行需要的动态库主要有libQt5Widgets, Core, Gui这3个，已经这3个依赖的依赖
  * 如果rpath指向这些依赖后，仍旧不能运行，提示This application failed to start because it could not find or load the Qt platform plugin "cocoa" in "".的错误，则需要拷贝libqcocoa.dylib(以及依赖的libQt5PrintSupport.5)过来
