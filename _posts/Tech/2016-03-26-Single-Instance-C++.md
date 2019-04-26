---
layout: post
title:  "Single Instance in C++"
date:   2016-03-26 10:00:00
tags: [single instance, c++, tech]
categories: Tech
---

> If you need to have one and only one object of a type in system AND you need to have global access to it
> [reference](https://stackoverflow.com/questions/1008019/c-singleton-design-pattern)

### 1. C++11 implementation of the Singleton design pattern
lazy-evaluated, correctly-destroyed, and thread-safe
{% highlight C++ %}
class S
{
    public:
        static S& getInstance()
        {
          #if defined(__GNUC__) && (__GNUC__ > 3)
            // it's OK, and thread save
            // If control enters the declaration concurrently while the variable is being initialized, the concurrent execution shall wait for // completion of the initialization.
          #else
            #error Add Critical Section for your platform
            // 注意double checked locking pattern的问题. This will not work in C++03:
            // https://stackoverflow.com/questions/367633/what-are-all-the-common-undefined-behaviours-that-a-c-programmer-should-know-a/367690#367690
          #endif
            static S instance; // Guaranteed to be destroyed. In C++11 standard: thread-safe
                               // Instantiated on first use. lazy-evaluated
            return instance;
        }
    private:
        S() {}  // in C++ 11: S() = default;
        // C++ 03
        // ========
        // Don't forget to declare these two. You want to make sure they
        // are unacceptable otherwise you may accidentally get copies of
        // your singleton appearing.
        S(S const&);              // Don't Implement
        S& operator=(S const&); // Don't implement

        // C++ 11
        // =======
        // 为了能够让程序员显式的禁用某个函数，C++11标准引入了一个新特性：deleted函数
        // 程序员只需在函数声明后加上“=delete;”，就可将该函数禁用。
    public:
        S(S const&)             = delete;
        S& operator=(S const&)  = delete;
        // Note: Scott Meyers mentions in his Effective Modern
        //       C++ book, that deleted functions should generally
        //       be public as it results in better error messages
        //       due to the compilers behavior to check accessibility
        //       before deleted status
};
{% endhighlight %}

### 2. 使用时需要注意的问题
C++中static变量的release顺序和allocate顺序相反：
{% highlight C++ %}
class B
{
    public:
        static B& getInstance_Bglob;
        {
            static B instance_Bglob;
            return instance_Bglob;;
        }
        ~B()
        {
             A::getInstance_abc().doSomthing();
             // The object abc is accessed from the destructor.
             // Potential problem.
             // You must guarantee that abc is destroyed after this object.
             // To guarantee this you must make sure it is constructed first.
             // To do this just access the object from the constructor.
        }
        B()
        {
            A::getInstance_abc();
            // abc is now fully constructed.
            // This means it was constructed before this object.
            // This means it will be destroyed after this object.
            // This means it is safe to use from the destructor.
        }
};
{% endhighlight %}

### 3. 错误版本的Header
{% highlight C++ %}
class SingleInstance {
  public:
    // 错误1，没有禁止copy constructer和opreator =
    ~SingleInstance();
    static shared_ptr<SingleInstance> Get();
    // 错误2，返回shared_ptr，导致其可以被清空，修改之类的
    static shared_ptr<SingleInstance> GetInstance(const string &para);
    void Do() const;
    bool IsInstanceSame(const string &para) const;
  private:
    SingleInstance(const string &para);
    static shared_ptr<SingleInstance> single_instance_;
    const string para_;
};
{% endhighlight %}

### 4. 错误版本的Implenmentation
{% highlight C++ %}
shared_ptr<SingleInstance> SingleInstance::single_instance_ = NULL;
shared_ptr<SingleInstance> SingleInstance::Get() {
  if (single_instance_ == NULL) {
    LOG(ERROR) << "Must be initialized before using!";
    throw std::runtime_error("Error runtime_error!");
  }
  return single_instance_;
}

shared_ptr<SingleInstance> SingleInstance::GetInstance(const string &para) {
  // 错误3，没有线程保护
  if (!single_instance_ || !single_instance_->IsInstanceSame(para)) {
    single_instance_.reset(new SingleInstance(para));
  }
  return single_instance_;
}

SingleInstance::SingleInstance(const string &para) :
  para_(para),
  if (!(fs::exists(para))) {
    LOG(ERROR) << "The file does not exist!";
    throw std::invalid_argument("Error invalid_argument!");
  }
  ...  // some initializing work
}

SingleInstance::~SingleInstance() {
 single_instance_.reset();
}

bool SingleInstance::IsInstanceSame(const string &para) const {
 return para_ == para;
}

void SingleInstance::Do() const {
  ... // do the main work
}
{% endhighlight %}