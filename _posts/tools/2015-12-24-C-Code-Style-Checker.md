---
layout: post
title:  "C++ Code Style Checker"
date:   2015-12-24 10:30:00
tags: [C++, Style, Check]
categories: Tools
---

### 1. Cpplint
We can integrate the open source cpplint from Google into our project to check the C++ code style.
Please refer to [cpplint-integration repo](https://github.com/wykvictor/cpplint-integration)

### 2. [Cppcheck](http://cppcheck.sourceforge.net/)
开源工具，静态代码检查：
数组越界；缺少copy constructor；非explicit constructor等

apt-get install cppcheck
cppcheck \-\-enable=all *.cpp *.h *.hpp

### 3. [Artistic Style](http://astyle.sourceforge.net/)
开源工具，自动格式化代码format，超省人工！

apt-get install astyle
astyle --options=option_file *.cpp *.h *.hpp

一个可用的option_file(具体使用[点此](http://astyle.sourceforge.net/astyle.html)查询)：
{% highlight Bash shell scripts %}
formatted
indent-switches
indent-namespaces
style=google
indent=spaces=2
keep-one-line-statements
add-brackets
lineend=linux
pad-header
convert-tabs
suffix=none
align-pointer=name
indent-preproc-block
{% endhighlight %}
效果：
{% highlight C scripts %}
#include <stdio.h>
void main()
{int i;printf("Hello world!\n");for(i=0;i<10;++i)printf("%d\n",i);}
{% endhighlight %}
==>
{% highlight C scripts %}
#include <stdio.h>
void main() {
  int i;
  printf("Hello world!\n");
  for (i=0; i<10; ++i) {
    printf("%d\n",i);
  }
}
{% endhighlight %}