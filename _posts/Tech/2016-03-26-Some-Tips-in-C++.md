---
layout: post
title:  "Some tips in C++"
date:   2016-03-26 16:00:00
tags: [single instance, c++, tech]
categories: Tech
---

1. 代码多用const, 包括类内部的函数
2. 尽量少用指针，会有memory issue. 可以使用auto_ptr和shared_ptr代替，命名用p打头
3. 代码注意分段，一段一个小功能
4. 代码开关多采用编译时开关，不需要的不编译进去，比true/false变量控制要好
6. GLOG比较好，比cout好。在android中也可以logcat输出
7. 使用接口：纯虚函数，继承不同的子类，然后子类组织成一个list，这样就可以用for loop来处理
8. 一个function的代码不要太长 < 1个screen
9. add unit test
10. use (try catch) to deal with error
11. 使用工具，调代码时间和内存问题，如[Valgrind和Kcachegrind](http://wykvictor.github.io/2016/02/04/Profile-C++-Apps-Using-Kcachegrind.html)
5. 代码文件命名格式统一，参考[Google Style](https://google.github.io/styleguide/cppguide.html#Naming)

```    
a. Variable Name 
   string table_name;  // OK - uses underscore.
b. Class Data Members
   string table_name_;  // OK - underscore at end.
c. Class Name
   class UrlTable
d. FileName
   my_useful_class.cc
   Filenames should be all lowercase and can include underscores (_) or dashes (-)
e. Function Name 
   Regular functions have mixed case: AddTableEntry()
   "cheap" functions may use lower case with underscores:
   bool is_empty() const { return num_entries_ == 0; }
f. Constant Name 
   const int kDaysInAWeek = 7;
   Variables declared constexpr or const, and whose value is fixed for the duration of the program,
   are named with a leading "k" followed by mixed case.
```