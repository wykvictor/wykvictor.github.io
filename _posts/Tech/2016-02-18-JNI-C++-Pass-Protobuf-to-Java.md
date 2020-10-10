---
layout: post
title:  "Pass Protobuf through JNI(C++ to Java)"
date:   2016-02-18 16:00:00
tags: [protobuf, c++ to java, jni, tech]
categories: Tech
---

>  项目中遇到android(java)层用到了C++库，C++返回protobuf结构给java

### 1. Protobuf环境[安装](https://github.com/google/protobuf)
{% highlight Bash shell scripts %}
cd path-to-source-into-java
mvn install  # 编译出jar包, copy到java工程的jniLib目录
protoc --java_out=./  com/sh/process/myproto.proto  # 编译出java文件, copy到工程目录proto下
(Note: .proto文件中package com.sh.process.proto, 代表最终生成的目录和java中的package信息，需要匹配)
{% endhighlight %}

### 2. In jni: protobuf->byte[]
{% highlight C++ %}
jbyteArray JNIEXPORT JNICALL Java_com_sh_process(JNIEnv *env, jobject thiz) {
  MyProto res = GetProtobufFromC();
  jbyteArray res_byte;

  int size = res.ByteSize();
  if(size <= 0)   return res_byte;  // Android 6.0 will throw exception when calling SetByteArrayRegion if size=0

  void *buffer = malloc(size);
  res.SerializeToArray(buffer, size);  // 序列化

  jbyteArray res_byte = env->NewByteArray(size);  // construce jbarray[]
  env->SetByteArrayRegion(res_byte, 0, size, (jbyte *)buffer);  // copy result to java layer

  free(buffer); // 注意：内存泄漏
  return res_byte;
}
{% endhighlight %}
上述的方式，仍然有内存泄漏，返回了res_byte是new出来的，这个应该交给java管理：
{% highlight C++ %}
jobject data_t0 = env->NewDirectByteBuffer((void *)proto_res.c_str(), proto_res.size());
if (data_t0) {
    jclass jclsJNILib = env->FindClass("com/test/MyJNILib");
    jmethodID methodID_setOutputData =
        env->GetMethodID(jclsJNILib, "setOutputData", "(Ljava/nio/ByteBuffer;)V");
    env->CallVoidMethod(obj, methodID_setOutputData, data_t0);
}
env->DeleteLocalRef(data_t0);
// Java端：
// call from native jni
public void setOutputData(ByteBuffer data)
{
    if (data == null) {
        data = ByteBuffer.wrap(new byte[]{});
        System.out.println("debug: date == null");
        return;
    }
    if (data.capacity() < 1) {
        data = ByteBuffer.wrap(new byte[]{});
        System.out.println("debug: data.capacity() < 1");
        return;
    }
    jni_data = ByteBuffer.allocateDirect(data.capacity());
    data.rewind();
    jni_data.put(data);
    data.rewind();
    jni_data.flip();
}
{% endhighlight %}


### 3. In java: byte[]->protobuf
{% highlight Java %}
byte[] res_byte = process();
MyProto res = MyProto.parseFrom(res_byte);  // 反序列化
if (res.getStatus() == MyProto.Status.SUCCESS) {
  String res_str = res.getNumber();
  // do sth
} else {
  // do sth
}
{% endhighlight %}
