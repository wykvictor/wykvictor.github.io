---
layout: post
title:  "C++ Preprocessor Cross-platform"
date:   2018-01-21 15:00:00
tags: [c++, preprocessor, cross platform]
categories: Resources
---

> 编写c++跨平台应用程序时，会用到[参考](http://blog.csdn.net/n5/article/details/70143942)

* win32/64平台都会定义WIN32，而_WIN64只在64位上定义，因此先判断_WIN64
* 所有apple系统都会定义__APPLE__，包括MacOSX和iOS
* 大多编译器支持的宏定义，[查询](https://sourceforge.net/p/predef/wiki/OperatingSystems/)
* Apple平台宏定义（注意）：
{% highlight C++ scripts %}
// TargetConditionals.h
        TARGET_OS_WIN32           - Generated code will run under 32-bit Windows
        TARGET_OS_UNIX            - Generated code will run under some Unix (not OSX) 
        TARGET_OS_MAC             - Generated code will run under Mac OS X variant
           TARGET_OS_OSX          - Generated code will run under OS X devices
           TARGET_OS_IPHONE          - Generated code for firmware, devices, or simulator
              TARGET_OS_IOS             - Generated code will run under iOS 
              TARGET_OS_TV              - Generated code will run under Apple TV OS
              TARGET_OS_WATCH           - Generated code will run under Apple Watch OS
              TARGET_OS_BRIDGE          - Generated code will run under Bridge devices
           TARGET_OS_SIMULATOR      - Generated code will run under a simulator
           TARGET_OS_EMBEDDED       - Generated code for firmware
        TARGET_IPHONE_SIMULATOR   - DEPRECATED: Same as TARGET_OS_SIMULATOR
        TARGET_OS_NANO            - DEPRECATED: Same as TARGET_OS_WATCH

      +------------------------------------------------+
      |                TARGET_OS_MAC                   |
      | +---+  +-------------------------------------+ |
      | |   |  |          TARGET_OS_IPHONE           | |
      | |OSX|  | +-----+ +----+ +-------+ +--------+ | |
      | |   |  | | IOS | | TV | | WATCH | | BRIDGE | | |
      | |   |  | +-----+ +----+ +-------+ +--------+ | |
      | +---+  +-------------------------------------+ |
      +------------------------------------------------+
{% endhighlight %}

{% highlight C++ scripts %}
#ifdef _WIN32
   //define something for Windows (32-bit and 64-bit, this part is common)
   #ifdef _WIN64
      //define something for Windows (64-bit only)
   #else
      //define something for Windows (32-bit only)
   #endif
#elif __APPLE__
    #include "TargetConditionals.h"
    #if TARGET_OS_SIMULATOR
         // iOS Simulator
    #elif TARGET_OS_IPHONE
        // iOS device
    #elif TARGET_OS_OSX
        // Other kinds of Mac OS
    #else
    #   error "Other Apple platform"
    #endif
#elif defined(ANDROID) || defined(__ANDROID__)  // both needed
    // android
#elif __linux__
    // linux
#elif __unix__ // all unices not caught above
    // Unix
#elif defined(_POSIX_VERSION)
    // POSIX
#else
#   error "Unknown compiler"
#endif
{% endhighlight %}
