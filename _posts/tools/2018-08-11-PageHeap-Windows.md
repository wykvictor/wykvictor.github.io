---
layout: post
title:  "Page Heap Checker in Windows"
date:   2018-08-11 16:30:00
tags: [pageheap, memory check, global flags]
categories: Tools
---

### Pageheap堆调试，Windows Global Flags工具
Windows SDK包里提供了Debugging tools组件Global Flags，该组件中的堆调试工具PageHeap检查，对于内存的越界读写均有较好的检查：
{% highlight C++ %}
char* buffer = new char[19];        // 1
char a = buffer[19];                // 2，越界访问
delete[] buffer;                    // 3
{% endhighlight %}
跑完1后，查看buffer内存情况：
{% highlight C++ %}
0x00B0E050  cd cd cd cd cd cd cd cd cd cd cd cd cd cd cd cd
0x00B0E060  cd cd cd fd fd fd fd 00 dd 69 c2 b5 70 01 00 80
{% endhighlight %}
Debug模式下，堆上未初始化的内存为cd，并且在内存结束处有4字节fd的边界，用来进行debug模式的内存检查。之后跑2:
{% highlight C++ %}
0x00B0E050  cd cd cd cd cd cd cd cd cd cd cd cd cd cd cd cd
0x00B0E060  cd cd cd 00 fd fd fd 00 dd 69 c2 b5 70 01 00 80
{% endhighlight %}
* 可以看到，第一个fd，由于越界填充了0，变成了00，此时程序并没有崩溃。
* 但是，运行第3句，程序崩溃。假如2、3间间隔了很多代码，这个越界访问的bug，很难被发现。
* 而且在Release下，完全不会崩溃。

之后，打开Global Falgs：
![global_flags](/res/global_flags.png)
具体的，buffer内存情况：
{% highlight C++ %}
0x00B0E050  cd cd cd cd cd cd cd cd cd cd cd cd cd cd cd cd
0x00B0E060  cd cd cd fd fd fd fd d0 ?? ?? ?? ?? ?? ?? ?? ??
{% endhighlight %}
* 可以看到，同样加了4字节的fd，区别是之后都是??，
* d0是用于填补4字节对齐，注意buffer后面的地址(第一个??)为0x07F31000,这是一个4k对齐的PAGE_NOACCESS页面
* 这个时候我们执行第2行代码buffer[19] = 0; 同样不会崩溃，即使是修改buffer[19-23]的值(4个fd边界和1个对齐d0，和未启动Gflags一样，程序都只会在执行第3行的时候崩溃。如果修改buffer[24]则程序会崩溃。

通过这个例子，可以得出一个结论：启用Global Flags后，堆内存分配在页面的末尾，后面紧跟了一个4k的PAGE_NOACCESS属性的页面,这种情况下，启用GFlags的好处是能在一定程度上检查内存越界。

**但是对于访问delete之后的指针，启动GFlags后，直接将整个页面置为不可读，之后使用必然会报错**
{% highlight C++ %}
struct A{
    int a=0;
    int b=0;
    virtual ~A(){}
};
struct B : A {};
int main() {
    struct B * bb = new struct B();
    struct A * aa = dynamic_cast<struct A *>(bb);
    delete bb;
    std::cout << aa->a << " Hello World!" << std::endl;
    return 0;
}
{% endhighlight %}
如果不开启Gflags，打印-572662307 Hello World!，不报错，开启GFlags后，报错 read violation，及时定位错误。
