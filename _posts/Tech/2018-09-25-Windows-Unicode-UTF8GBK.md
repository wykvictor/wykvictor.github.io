---
layout: post
title:  "Windows Unicode-UTF8/GBK"
date:   2018-09-25 16:00:00
tags: [windows, unicode, utf8, gbk, wstring]
categories: Tech
---

> [Unicode,GBK和UTF8](https://www.cnblogs.com/pannengzhi/p/5678495.html)
> [各种编译器测试](https://www.cnblogs.com/zyl910/archive/2013/01/20/wchar_crtbug_01.html)


### 1. Unicode是个字符集
* 和ASCII一样, 其作用是用一系列数字来表示字符(character)，却别是ASCII只定义了128个字符，只需要1个字节就可定义
* 世界上字符太多，不够用，Unicode字符集出现，兼容ASCII,最多可以表示2^21(大概200万)个字符,已经足够囊括当今所有国家的文字
* 有了字符集, 就可以用任意数字来表示现实中的字符了

### 2. 字符编码
* 编码，用来约定“用多少个字节表示一个数字,以及每个字节的范围”
* 如果encode和decode的编码不一致，则得不到正常结果，出现乱码
* 常见的编码规则有utf-8,utf-16,gb2312,gbk等，例子：
{% highlight Python %}
>>> u'你好'
u'\u4f60\u597d'               # unicode码
>>> u'你好'.encode('utf8')
'\xe4\xbd\xa0\xe5\xa5\xbd'    # utf8编码，占6个字节
>>> u'你好'.encode('gbk')
'\xc4\xe3\xba\xc3'            # gbk编码(windows默认编码)，4个字节
# Windows下,未初始化的栈会初始化为0xcc，未初始化的堆内存会初始化为0xcd，二者的gbk编码：
>>> u'烫'.encode('gbk')
b'\xcc\xcc'
>>> u'屯'.encode('gbk')
b'\xcd\xcd'                   # 常见乱码的由来：烫烫烫屯屯屯
{% endhighlight %}

### 3. wstring和utf8相互转换
{% highlight C++ %}
string wstring_to_utf8(const std::wstring& wstr)
{
    std::wstring_convert<std::codecvt_utf8<wchar_t>, wchar_t> converterX;
    return converterX.to_bytes(wstr);
}
std::wstring utf8_to_wstring(const std::string& str)
{
    std::wstring_convert<std::codecvt_utf8<wchar_t>> myconv;
    return myconv.from_bytes(str);
}
// 调用
// locale("")：调用构造函数创建一个local，其中的空字符串具有特殊含义：使用系统环境中缺省的locale
wcout.imbue(locale(""));         // wcout default is Not system codec, 
std::wstring wstr = L"中文wstr";  // L代表是wchar_t组成的std::wstring
cout << "中文cout" << std::endl;  // 输出：中文cout
wcout << wstr << std::endl;      // 输出：中文wstr
wcout << utf8_to_wstring(wstring_to_utf8(wstr)) << std::endl;  // 输出：中文wstr
wcout.clear();  				 // 如果发生错误，清除掉了错误状态
wcout << std::endl;  			 // 调用fllush，刷新缓冲区

cout.imbue(locale(""));  		 // cout default is system codec，可以不调用
std::string str = "中文str";
cout << str << std::endl;        // 中文str
{% endhighlight %}
