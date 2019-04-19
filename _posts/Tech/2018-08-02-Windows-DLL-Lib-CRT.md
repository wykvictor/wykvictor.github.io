---
layout: post
title:  "Windows Dll/Lib/CRT/MSBuild"
date:   2018-08-02 16:00:00
tags: [windows, dll, lib, crt]
categories: Tech
---

### 1. dll/lib

### 2. Windows CRT（dll/lib）
CRT: C运行时库
* 
比如在编译OpenCV时，BUILD_WITH_STATIC_CRT应该设置为false，则OPENCV.lib掉用的是windows里dll的一些库，C++编译选项里Code generation是/MD，否则是/MT；
这样，在打包OpenCV进own.dll时，也就统一用MD

### 3. MSBuild
{% highlight Bash shell scripts %}
:: x86  
cmake -G "Visual Studio 14 2015" ..
C:\Windows\Microsoft.NET\Framework\v4.0.30319\MSBuild.exe /p:Configuration=Release /m:4 ALL_BUILD.vcxproj :: /m设置build线程数，默认8全开
C:\Windows\Microsoft.NET\Framework\v4.0.30319\MSBuild.exe /p:Configuration=Debug /m:4 INSTALL.vcxproj
:: x64
cmake -G "Visual Studio 14 2015 Win64" ..
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\MSBuild.exe /p:Configuration=Release /m:4 INSTALL.vcxproj
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\MSBuild.exe /p:Configuration=Debug /m:4 INSTALL.vcxproj
{% endhighlight %}
