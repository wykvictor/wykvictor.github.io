---
layout: post
title:  "Single Instance in C++"
date:   2016-03-26 10:00:00
tags: [single instance, c++, tech]
categories: Tech
---

### 1. Header
{% highlight C++ %}
class SingleInstance {
  public:
    ~SingleInstance();
    static shared_ptr<SingleInstance> Get();
    static shared_ptr<SingleInstance> GetInstance(const string &para);
    void Do() const;
    bool IsInstanceSame(const string &para) const;
  private:
    SingleInstance(const string &para);
    static shared_ptr<SingleInstance> single_instance_;
    const string para_;
};
{% endhighlight %}

### 2. Implenmentation
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