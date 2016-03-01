---
layout: post
title:  "Throw Exception through JNI(C++/Java)"
date:   2016-02-18 12:00:00
tags: [exception, jni, tech]
categories: Tech
---

>  项目中遇到android(java)层用到了C++库，C++抛出的异常需要暴露给java
[link](http://www.codeproject.com/Articles/17558/Exception-handling-in-JNI)

### 1. 如何在Jni中Catch Exception
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

### 2. Jni中定义函数Re-throw Exception
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

  // Find the exception class.
  jclass tClass = env->FindClass("com/my/project/JNIException");  // java端自定义异常，这样可以与java本身的异常区分
  if (env->ExceptionCheck()) {  // 检出是否有此定义
    env->ExceptionDescribe();
    env->ExceptionClear();  // Make sure to clear it, 否则抛到java层就乱了
    LOGE("Not found com/my/project/JNIException, will try to find java/lang/Exception");
    tClass = env->FindClass("java/lang/Exception");  // 也可以find特定类型:IOExpection,OutOfMemoryError...
    if (tClass == NULL) {
      LOGE("Not found java/lang/Exception");
      env->DeleteLocalRef(tClass);
      return;
    }
  }

  //Throw the exception with error info
  env->ThrowNew(tClass, error_message.c_str());
  env->DeleteLocalRef(tClass);
}
{% endhighlight %}

### 3. Android端实现java自定义异常
JNIException.java定义了jni的异常
{% highlight Java %}
package com.my.project;

public class JNIException extends Exception {
  public JNIException() {
      super();
  }

  public JNIException(String message) {
      super(message);
  }
}
{% endhighlight %}

### 4. Android端实际调用并catch jni异常
Jni接口：
{% highlight Java %}
public native void process(int pram) throws JNIException;
{% endhighlight %}
调用该接口并catch异常：
{% highlight Java %}
try {
  process();
} catch (JNIException e) {
  Log.e(LOG_TAG, "JNIException: ", e);  // Better than e.printStackTrace(), can be logged into Logger
  finish();
} catch (Exception e) {
  Log.e(LOG_TAG, "Exception: ", e);
  finish();
}
{% endhighlight %}