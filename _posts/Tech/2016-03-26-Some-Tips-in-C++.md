---
layout: post
title:  "Some tips in C++"
date:   2016-03-26 16:00:00
tags: [single instance, c++, tech]
categories: Tech
---

### 1. 一些小tips
1. 代码多用const, 包括类内部的函数
2. 尽量少用指针，会有memory issue. 可以使用智能指針代替，命名用p打头：
   * auto_ptr: 内部沒有引用计数，赋值的时候，会转移拥有关系。不能共享所有权，即不要让两个auto_ptr指向同一个对象。
   * shared_ptr：auto_ptr有和很多不足之处，建议使用shared_ptr。有计数，自动释放自己所管理的对象，也可以自定义一个删除器(deleter)函数来代替delete
   * unique_ptr：某时刻只能有一个unique_ptr指向给定的对象，不支持普通的拷贝或赋值操作。p2.reset(p3.release());转移所有权
   * weak_ptr: 是shared_ptr的助手而不是智能指针，它不具有普通指针的行为，没有重载operator*和->。成员函数lock()获得一个可用的shared_ptr对象
   * scope_ptr: 控制权私有，因为在scoped_ptr类的内部将拷贝构造函数和=运算符重载定义为私有
3. 代码注意分段，一段一个小功能
4. 代码开关多采用编译时开关，不需要的不编译进去，比true/false变量控制要好
6. GLOG比较好，比cout好。在android中也可以logcat输出
7. 使用接口：纯虚函数，继承不同的子类，然后子类组织成一个list，这样就可以用for loop来处理（类似Caffe的Layer class）
8. 一个function的代码不要太长 < 1个screen
9. add unit test
10. use (try catch) to deal with error
11. 使用工具，调代码时间和内存问题，如[Valgrind和Kcachegrind](http://wykvictor.github.io/2016/02/04/Profile-C++-Apps-Using-Kcachegrind.html)
    valgrind --undef-value-errors=no --leak-check=full -v

### 2. 代码文件命名格式统一，参考[Google Style](https://google.github.io/styleguide/cppguide.html#Naming)

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

### 3. 虚函数
   * 实现原理[虚函数表1](http://www.cnblogs.com/malecrab/p/5572730.html)和[2](https://blog.twofei.com/496/)
   * 构造函数不能是虚函数(刚开始构造，还没有vtable呢)，析构函数大部分是虚函数(防止子类无法释放完内存)
   * 不要在构造函数或析构函数内调用虚函数[example](http://www.cnblogs.com/vincently/p/4754206.html)
   * 虚继承：概念完全不同，是为了解决从不同途径继承来的同名的数据成员在内存中有不同的拷贝造成数据不一致问题，将共同基类设置为虚基类。这时从不同的路径继承过来的同名数据成员在内存中就只有一个拷贝，同一个函数名也只有一个映射。这样不仅就解决了二义性问题，也节省了内存，避免了数据不一致的问题。

### 4. vector的使用
   * 内存增长：内存空间只会增长，不会减小。调用erase()可以清除掉某些元素(Numbers.erase(Numbers.begin()+5,Numbers.begin()+10))，clear()可以清除掉所有元素，但是内存空间不变化, capacity()不变，只是size()变了
   * 不够时，空间会增长，并且多分配几个的空间，具体是多少看编译器。所以可以预估程序使用的vector的大小，事先reserve()一些
   * 内存回收：std::vector<int>().swap(the_vector);，或者clear()之后调用shrink_to_fit()方法(但是该方法只是一个请求，不一定能确定释放成功)

### 5. 并发编程
   * 读写锁：可以同时被多个读者拥有，但是只能被一个写者拥有的锁，就是说不能有多个写者同时去写
      * 当读写锁是写加锁状态时, 在这个锁被解锁之前, 所有试图对这个锁加锁的线程都会被阻塞.
      * 当读写锁在读加锁状态时, 所有试图以读模式对它进行加锁的线程都可以得到访问权, 但是如果线程希望以写模式对此锁进行加锁, 它必须直到所有的线程释放锁.
      * 通常, 当读写锁处于读模式锁住状态时, 如果有另外线程试图以写模式加锁, 读写锁通常会阻塞随后的读模式锁请求, 这样可以避免读模式锁长期占用, 而等待的写模式锁请求长期阻塞.
{% highlight C++ %}
#include<condition_variable>
class RWMutex {
public:
    RWMutex() : counter_(0), waiting_readers_(0), waiting_writers_(0) {}
    ~RWMutex() = default;
    RWMutex(const RWMutex &) = delete;
    RWMutex(RWMutex &&) = delete;
    RWMutex& operator=(const RWMutex &) = delete;
    RWMutex& operator=(RWMutex &&) = delete;
    int counter_;  // 全局控制
    int waiting_readers_;
    int waiting_writers_;
    std::mutex mutex_;  // 全局
    std::condition_variable reader_cv_;
    std::condition_variable writer_cv_;
};
class ReadLock {
public:
    explicit ReadLock(RWMutex* rw_mutex) : rw_mutex_(rw_mutex)
    {
        assert(rw_mutex != NULL);
        std::unique_lock<std::mutex> lock(rw_mutex->mutex_);
        rw_mutex_->waiting_readers_ += 1;
        rw_mutex_->reader_cv_.wait(lock, [&]() -> bool {  // 写者优先，counter_>=0表示没有人在写
        // 当读写锁处于读模式锁住状态时, 如果有另外线程试图以写模式加锁, 读写锁通常会阻塞随后的读模式锁请求, 这样可以避免读模式锁长期占用, 而等待的写模式锁请求长期阻塞.
           return rw_mutex_->waiting_writers_ == 0 && rw_mutex_->counter_ >= 0;
        });
        rw_mutex_->waiting_readers_ -= 1;
        rw_mutex_->counter_ += 1;
    }
    ~ReadLock() {
        std::unique_lock<std::mutex> lock(rw_mutex_->mutex_);
        rw_mutex_->counter_ -= 1;
        if (rw_mutex_->waiting_writers_ > 0) {
            if (rw_mutex_->counter_ == 0) {  // 有人等写，而且没有人在读/写
                rw_mutex_->writer_cv_.notify_one();
            }
        }
    }
    ReadLock(const ReadLock &) = delete;
    ReadLock(ReadLock &&) = delete;
    ReadLock& operator=(const ReadLock &) = delete;
    ReadLock& operator=(ReadLock &&) = delete;
private:
    RWMutex* rw_mutex_;
};
class WriteLock {
public:
    explicit WriteLock(RWMutex* rw_mutex) : rw_mutex_(rw_mutex) {
        assert(rw_mutex != NULL);
        std::unique_lock<std::mutex> lock(rw_mutex->mutex_);
        rw_mutex_->waiting_writers_ += 1;
        rw_mutex_->writer_cv_.wait(lock, [&]() -> bool {  // 没有别人读&写时候，才可以写
            return rw_mutex_->counter_ == 0;
        });
        rw_mutex_->waiting_writers_ -= 1;  // 拿到了写权利
        rw_mutex_->counter_ -= 1;
    }
    ~WriteLock() {
        std::unique_lock<std::mutex> lock(rw_mutex_->mutex_);
        rw_mutex_->counter_ = 0;
        if (rw_mutex_->waiting_writers_ > 0) {  // 优先写
            rw_mutex_->writer_cv_.notify_one();
        } else {
            rw_mutex_->reader_cv_.notify_all();
        }
    }
    WriteLock(const WriteLock &) = delete;
    WriteLock(WriteLock &&) = delete;
    WriteLock& operator=(const WriteLock &) = delete;
    WriteLock& operator=(WriteLock &&) = delete;
private:
    RWMutex* rw_mutex_;
};
{% endhighlight %}

### 6. [lambda](http://blog.csdn.net/booirror/article/details/26973611)匿名函数

### 7. C++11中的std::[function](http://www.jellythink.com/archives/771)对各种可调用实体（普通函数、Lambda表达式、函数指针、以及其它函数对象等）的封装，形成一个新的可调用的std::function对象

### 8. C++ class construction
{% highlight C++ %}
class A {
public:
    A(int a = 2) {  // 2. 其次初始化
        // a为此函数局部变量，并不是this->a(作用域被覆盖)
        a = 3;  // 3. 再次初始化
    }
    int a = 4;  // 1. 首先初始化
};
int main(int argc, const char * argv[]) {
    A a;
    std::cout << a.a << std::endl;  // a.a输出4
    return 0;
}
// 改变1: 局部变量改为aa，
A(int aa = 2) {
    a = 3;  // a.a输出3，此时a就是this->a
}
// 改变2: 用初始化列表
A(int a = 2):a(a) {  // 等同于this->a(a)
    a = 3;  // a.a输出2，初始化列表起作用
}
// 改变3: 用构造函数体
A(int a = 2):a(a) {
    this->a = 3;  // a.a输出3，构造函数体起作用，这是第3个初始化的位置
}
{% endhighlight %}
总结，良好的编程习惯：
* 声明时直接初始化最方便
* 如果要修改，构造函数接口里不要用相同的名字 a
* 用初始化列表初始化，名字相同也可以
* 初始化列表初始化，比构造函数体初始化，效率高，因为a如果是个很大的类，在进入构造函数体时，已经构造过一次了，进入之后又调用一次复制构造函数
* 如果有多个成员变量a,b,c，即使有初始化列表，也是按照类里边声明的顺序初始化的

### 9. C++11 raw string literals
1. 特点是‘\’不进行转义：a string in which the escape characters (like \n \t or \" ) of C++ are not processed
2. 定义是R"(之间的字符串)"，必须有()，否则"""三个引号编译错误
{% highlight C++ %}
// raw string可以跨越多行，其中的空白和换行符都属于字符串的一部分。
std::cout <<R"(\n  // 注释也输出
    Hello,
    world!
    )" << std::endl;
// 结果：
\n  // 注释也输出
    Hello,
    world!
{% endhighlight %}
3. 对于错误：string too big, trailing characters truncated
可以将一个string用引号隔开，否则visual studio编译不过：[参考](https://docs.microsoft.com/en-us/cpp/error-messages/compiler-errors-1/compiler-error-c2026?view=vs-2019)

### 10. dynamic_cast static_cast dynamic_pointer_cast
* static_cast: 
  * 基本数据类型之间的转换，如把int转换成char，把int转换成enum
  * 用于类层次结构中基类和子类之间指针或引用的转换: 进行上行转换（把子类的指针或引用转换成基类表示）是安全的； 进行下行转换时，由于没有动态类型检查，不安全
  * 把void指针转换成目标类型的指针(不安全!!)
  * static_cast不能转换掉expression的const、volatile
* dynamic_cast: 主要用于类层次间的上行转换和下行转换
  * 上行转换时，dynamic_cast和static_cast的效果是一样的
  * 在进行下行转换时，dynamic_cast具有类型检查的功能，比static_cast更安全, 如果转换不成，返回NULL
  * 多重继承：选择唯一的一条路径，一层一层向上转换
* const_cast：
  * 常量指针被转化成非常量指针，并且仍然指向原来的对象
  * 常量引用被转换成非常量引用，并且仍然指向原来的对象
* reinterpret_cast：
  * 允许将任何指针类型转换为其它的指针类型（慎用！！！）

{% highlight C++ %}
const int b = 10;
// b = 11 编译报错，因为b是一个常量对象
int * pc = const_cast<int *>(&b);
*pc = 11;
cout << "*pc = " << *pc << endl; //11， b原来地址的数据现在可由*pc来改变，即解除const
cout << "b = " << b << endl;  //10， b其实类似(编译器处理问题)#define b 10，没有变。但是xcode调试器给出的是11
{% endhighlight %} 

[关于const](https://stackoverflow.com/questions/2328671/constant-variables-not-working-in-header)：
* const double PI = 3.1415926535; 可以直接放到hpp头文件，不会报错duplicate symbols
* because in C++ const objects have internal linkage by default
* 效果类似于static const double PI = 3.1415926535; 
* 但是注意char *的情况：
{% highlight C++ %}
// 如下的定义放到hpp中：
const std::string aa = "haha";
const char aaa='a';
const char a[] = R"(haha)";
const char * const aaaa = R"(haha)";
static const char * aaaaa = R"(haha)";
const char * aaaaaa = R"(haha)";  // 以上都对，这个不对！！！aaaaaa是个普通变量，还不是const的
{% endhighlight %}

### 11. [避免使用虚函数作为库的接口](https://blog.csdn.net/Solstice/article/details/6244905)
* 如果提供动态库so，期望不让调用方重新编译可执行代码，hot fix。直接更换头文件，加新代码就可以，旧代码不用动。
* 如果用虚函数做接口，会给二进制兼容性带来很大麻烦: 
  * 本质问题在于C++以vtable[offset]方式实现虚函数调用，而offset又是根据虚函数声明的位置隐式确定的，这造成了增加接口后，旧的可执行程序调用不到正确的函数
* 推荐使用pimpl技法：
  1. 暴露的接口里边不要有虚函数，存一个句柄，这样不存在[编译依赖](https://harttle.land/2015/08/29/effective-cpp-31.html)
{% highlight C++ %}
class Person{
public:
    Person(string& name);
    string name() const;
private:
    class PersonImpl;
    shared_ptr<PersonImpl> pImpl;
};
Person::Person(string& name): pImpl(new PersonImpl(name)){}
string Person::name(){
    return pImpl->name();
}
{% endhighlight %}
相当于把实现放到了另外一个类PersonImpl中，这样的Person类称为句柄类。当PersonImpl的内部实现发生改变时，依赖于Person的代码不再需要重新编译了。
  2. 在库的实现中把调用转发(forward)给实现Graphics::Impl，这部分代码位于.so/.dll中，随库的升级一起变化
  3. 如果要加入新的功能，不必通过继承来扩展，可以原地修改，且保持二进制兼容性
