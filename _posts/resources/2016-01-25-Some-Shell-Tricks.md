---
layout: post
title:  "Some Shell Tricks"
date:   2016-01-19 15:30:00
tags: [linux, shell, trick]
categories: Resources
---

### 1. 变量赋值时直接替换某些字符
{% highlight Bash shell scripts %}
$ a="x y"
$ echo ${a/ */}  # output: x
$ echo ${a/x /z} # output: zy
{% endhighlight %}

### 2. if [[ ]] 正则表达式
{% highlight Bash shell scripts %}
$ platform="android-armeabi-v7a"
$ if [[ $platform == "android"* ]];  # match
$ if [[ $platform =~ "arm" ]];  # match
$ if [[ $platform == "arm" ]];  # not match
$ if [[ $platform == *"arm"* ]];  # match
{% endhighlight %}
