---
layout: post
title:  "Best practices for using the Java Native Interface(JNI)"
date:   2016-02-28 16:00:00
tags: [tips, jni, tech]
categories: Tech
---

>  [Reference](https://www.ibm.com/developerworks/library/j-jni/)
>  and [JNI Functions](http://docs.oracle.com/javase/1.5.0/docs/guide/jni/spec/functions.html)

### 1. Why using JNI?
* A standard Java API that enables Java code to integrate with code written in other programming languages.
* Integrate with existing legacy code to avoid a rewrite.
* Implement functionality missing in available class libraries.
* Integrate with code that's best written in C/C++, to exploit performance or other environment-specific system characteristics.

### 2. Performance pitfalls

* **Not caching method IDs, field IDs, and Classes**

To access Java objects' fields and invoke their methods, native code must make calls to `FindClass(), GetFieldID(), GetMethodId(), and GetStaticMethodID()`. The IDs returned for a given class don't change for the lifetime of the JVM process. But the call to get the field require significant work in the JVM. Because the IDs are the same for a given class, you should look them up once and then reuse them.

**<font size="3">Calling a static method with JNI:</font>**
{% highlight C++ %}
int val=1;
jmethodID method;
jclass cls;

cls = (*env)->FindClass(env, "com/ibm/example/TestClass");
if ((*env)->ExceptionCheck(env)) {
   return ERR_FIND_CLASS_FAILED;
}
method = (*env)->GetStaticMethodID(env, cls, "setInfo", "(I)V");
if ((*env)->ExceptionCheck(env)) {
   return ERR_GET_STATIC_METHOD_FAILED;
}
(*env)->CallStaticVoidMethod(env, cls, method,val);
if ((*env)->ExceptionCheck(env)) {
   return ERR_CALL_STATIC_METHOD_FAILED;
}
{% endhighlight %}
We can cache jmethodID, reducing call times from 6 to 2!

* **Triggering array copies**

The Java specification leaves it up to the JVM implementation whether these calls provide direct access to the arrays or return a copy of the array.

For example, if you call `GetLongArrayElements()` on an array with 1,000 elements, you might cause the allocation and copy of at least 8,000 bytes (1,000 elements * 8 bytes for each long). When you then update the array's contents with `ReleaseLongArrayElements()`, another copy of 8,000 bytes might be required to update the array. 

The `GetTypeArrayRegion() and SetTypeArrayRegion()` methods allow you to get and update a region of an array, as opposed to the full array. 

* **Reaching back instead of passing parameters**
{% highlight C++ %}
int sumValues(JNIEnv* env, jobject obj, jint a, jint b, jint c){  // this is time saving!
   return a + b + c;
}

int sumValues2(JNIEnv* env, jobject obj, jobject allValues){

   jint avalue = (*env)->GetIntField(env, allValues, a);  // more JNI calls, more time
   jint bvalue = (*env)->GetIntField(env, allValues, b);
   jint cvalue = (*env)->GetIntField(env, allValues, c);
   
   return avalue + bvalue + cvalue;
}
{% endhighlight %}

* **Choosing the wrong boundary between native and Java code**

Ensure that data is maintained on the correct side of the Java/native boundary. If data resides on the wrong side, constant transitions will be triggered by the need of the other side to reach for that data.

* **Using many local references without informing the JVM**

Local references are created for any object returned by a JNI function. When a native causes the creation of a large number of local references, delete each reference when it is no longer required.

{% highlight C++ %}
void workOnArray(JNIEnv* env, jobject obj, jarray array){
   jint i;
   jint count = (*env)->GetArrayLength(env, array);
   for (i=0; i < count; i++) {
      jobject element = (*env)->GetObjectArrayElement(env, array, i);
      if((*env)->ExceptionOccurred(env)) {
         break;
      }
      /* do something with array element */
      (*env)->DeleteLocalRef(env, element);  // Do Not Forget!
   }
}
{% endhighlight %}

### 3. Correctness pitfalls

* **Using the wrong JNIEnv**

JNIEnv is local to a thread. Only use the JNIEnv with the single thread to which it is associated.

For optimal performance, a thread should pass the JNIEnv that it received, because looking it up can require significant work.

**<font size="3">Initializing jni env:</font>**
{% highlight C++ %}
jint JNIEXPORT JNICALL JNI_OnLoad(JavaVM *vm, void *reserved) {
  JNIEnv *env = NULL;
  jint result = -1;

  if (vm->GetEnv((void **)&env, JNI_VERSION_1_6) != JNI_OK) {
    LOGE("GetEnv failed!");
    return result;
  }

  return JNI_VERSION_1_6;
}
{% endhighlight %}

* **Not checking for exceptions**

Always check for exceptions after making JNI calls that can raise exceptions.

Example: my blog [Throw Exception through JNI](http://wykvictor.github.io/2016/02/18/JNI-C++-Throw-Exception-to-Java.html#ioaoc)

* **Not checking return values**

Many JNI methods have a return value that indicates whether the call succeeded or not. Always check the return value from a JNI method and include code to handle errors.

* **Using array methods incorrectly**

Don't forget to call ReleaseXXX() with a mode of 0 (copy back and free the memory) for each GetXXX() call.

Ensure the code does not make any JNI calls or block for any reason between calls to GetXXXCritical() and ReleaseXXXCritical().

* **Using global references incorrectly**

Always keep track of global references and ensure they are deleted when the object is no longer required.
{% highlight C++ %}
lostGlobalRef(JNIEnv* env, jobject obj, jobject keepObj) {
   jobject gref = (*env)->NewGlobalRef(env, keepObj);
}
{% endhighlight %}
When the native returns, not only has it not freed the global reference, but the application also no longer has a way to get the reference in order to free it later --> memory leak