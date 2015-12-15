---
layout: post
title:  "Application Memory Analysis"
date:   2015-11-17 16:30:00
tags: [memory, analysis, tool]
categories: Tools
---

### 1. Linux利用proc下status文件分析内存
在/proc/self(or pid)/status(or stat,statm)这3个文件中，有内存详细数据，ps等命令的结果皆取自于此
{% highlight Bash shell scripts %}
$ cat /proc/`ps -ef|grep caffe | grep -v grep | awk '{print $2}'`/status | grep Vm
VmPeak:	  570696 kB
VmSize:	  570692 kB  # 虚拟内存大小
VmLck:	       0 kB
VmPin:	       0 kB
VmHWM:	   14304 kB  # Peak RSS
VmRSS:	   13708 kB  # 实际占用物理内存
VmData:	  330652 kB  # heap上的数据(动态分配new)
VmStk:	     136 kB  # stack上的东西(临时变量等)
VmExe:	       8 kB  # 代码段
VmLib:	   58336 kB  # Shared Lib虚拟内存大小
VmPTE:	     508 kB
VmSwap:	       0 kB
{% endhighlight %}
重要的域已经标出，其他字段含义可以参考`man proc`

### 2. 一个c程序，读取stat文件来获得内存数据
这个文件的数据与status文件相同，但是格式混乱不易读，可以简单写程序读取stat：
{% highlight C++ scripts %}
#include <unistd.h>
#include <ios>
#include <iostream>
#include <fstream>
#include <string>

void process_mem_usage(double& vm_usage, double& resident_set, std::string pid)
{
    vm_usage     = 0.0;
    resident_set = 0.0;

    // the two fields we want
    unsigned long vsize;
    long rss;
    {
        std::string ignore, memFile;
	    memFile = "/proc/" + pid + "/stat";
        std::ifstream ifs(memFile.c_str(), std::ios_base::in);
        ifs >> ignore >> ignore >> ignore >> ignore >> ignore >> ignore >> ignore >> ignore >> ignore >> ignore
                >> ignore >> ignore >> ignore >> ignore >> ignore >> ignore >> ignore >> ignore >> ignore >> ignore
                >> ignore >> ignore >> vsize >> rss;  // 第23,24列
    }

    long page_size_kb = sysconf(_SC_PAGE_SIZE) / 1024;
    std::cout << "Page_size(kb): " << page_size_kb << std::endl;  //每一页的大小，一般2K或4K
    vm_usage = vsize / 1024.0;
    resident_set = rss * page_size_kb;
}

int main(int argc, char* argv[])
{
   using std::cout;
   using std::endl;

   double vm, rss;
   if(argc < 2) {
       cout << "Usage: main pid" << endl;
       return 1;
   }
   process_mem_usage(vm, rss, argv[1]);
   cout << "VM: " << vm << "; RSS: " << rss << endl;
   return 0;
}
{% endhighlight %}
同样检查相同APP的内存，结果如下：
{% highlight Bash shell scripts %}
Page_size(kb): 4  # Page大小
VM: 570692; RSS: 13708  # 与1中的VmSize和VmRSS相同
{% endhighlight %}

### 3. statm中数据按照pages来计
需要乘以每一page的大小，如本系统为4kb，最终的结果与上述2种方法一致
{% highlight Bash shell scripts %}
$ cat /proc/27621/statm
142673  # 570692Kb size: total program size (same as VmSize in /proc/[pid]/status)
3427  # 13888Kb resident set size (same as VmRSS in /proc/[pid]/status)
2117  # 8468Kb shared pages (from shared mappings)
2  # 8Kb text (code)
0  # library (unused in Linux 2.6)
82697  # 330788Kb data + stack
0  # dirty pages (unused in Linux 2.6)
{% endhighlight %}