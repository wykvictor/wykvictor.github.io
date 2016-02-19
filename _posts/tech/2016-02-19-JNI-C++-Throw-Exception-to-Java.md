---
layout: post
title:  "Throw Exception through JNI(C++/Java)"
date:   2016-02-19 12:00:00
tags: [exception, jni, tech]
categories: Tech
---

>  项目中遇到android(java)层用到了C++库，C++抛出的异常需要暴露给java
[link](http://www.codeproject.com/Articles/17558/Exception-handling-in-JNI)

#### 1. 如何在Jni中Catch Exception
{% highlight C++ %}
// 该函数负责re-throw异常，稍后定义
void ThrowJNIException(JNIEnv *env, const char *kpFile, int iLine, const string &type, const string &message);
/*Here '__FILE__' and '__LINE__' are predefined macros and part of the C/C++ standard.
  During preprocessing, they are replaced respectively by a constant string
  holding the current file name and by an integer representing the current line number.
  */
#define THROW_JAVA_EXCEPTION(_ENV_, _TYPE_, _INFO_) \
   ThrowJNIException(_ENV_, __FILE__, __LINE__, _TYPE_, _INFO_);
{% endhighlight %}
在Jni函数接口中，Catch exception and throw it:
{% highlight C++ %}
try {
  res = process();
} catch (cv::Exception &e) {  // 比如catch的不同类型的exception
  THROW_JAVA_EXCEPTION(env, string("Opencv ") + typeid(e).name(), e.what());
} catch (std::exception &e) {
  THROW_JAVA_EXCEPTION(env, string("Std ") + typeid(e).name(), e.what());
} catch (...) {
  THROW_JAVA_EXCEPTION(env, "Other", "Other Exception.");
}
{% endhighlight %}

#### 2. Jni中定义函数Re-throw Exception
该函数负责re-throw异常
{% highlight C++ %}
void ThrowJNIException(JNIEnv *env, const char *kpFile, int iLine, const string &type, const string &message) {
  //Creating the error messages
  string error_message = "JNIException!";
  if (kpFile != NULL && !message.empty()) {
    error_message += "\nFile: " + string(kpFile) +
                     "\nLine number: " + boost::lexical_cast<string>(iLine) + //lexical_cast转换函数!!
                     "\nException type: " + type +
                     "\nReason for Exception: " + message + "\n";
  }

  //Find the exception class.
  jclass tClass = env->FindClass("java/lang/Exception");  // 也可以find特定类型:IOExpection,OutOfMemoryError...
  if (tClass == NULL) {
    printf("Not found %s","java/lang/Exception");
    return;
  }

  //Throw the exception with error info
  env->ThrowNew(tClass, error_message.c_str());
  env->DeleteLocalRef(tClass);
}
{% endhighlight %}