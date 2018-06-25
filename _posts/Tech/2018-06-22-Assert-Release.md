---
layout: post
title:  "Enable Assert in Release build"
date:   2018-06-22 16:00:00
tags: [Assert, release, ndebug]
categories: Tech
---

{% highlight C++ %}
#undef NDEBUG
#include <assert.h>
{% endhighlight %}