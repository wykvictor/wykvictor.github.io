---
layout: post
title:  "Pass Protobuf through JNI(C++ to Java)"
date:   2016-02-18 16:00:00
tags: [protobuf, c++ to java, jni, tech]
categories: Tech
---

>  项目中遇到android(java)层用到了C++库，C++返回protobuf结构给java

#### 1. Protobuf环境[安装](https://github.com/google/protobuf)
{% highlight Bash shell scripts %}
cd path-to-source-into-java
mvn install  # 编译出jar包, copy到java工程的jniLib目录
protoc --java_out=./  com/sh/process/myproto.proto  # 编译出java文件, copy到工程目录proto下
(Note: .proto文件中package com.sh.process.proto, 代表最终生成的目录和java中的package信息，需要匹配)
{% endhighlight %}

#### 2. In jni: protobuf->byte[]
{% highlight C++ %}
jbyteArray JNIEXPORT JNICALL Java_com_sh_process(JNIEnv *env, jobject thiz) {
  MyProto res = GetProtobufFromC();
  int size = res.ByteSize();
  void *buffer = malloc(size);
  res.SerializeToArray(buffer, size);  // 序列化

  jbyteArray res_byte = env->NewByteArray(size);  // construce jbarray[]
  env->SetByteArrayRegion(res_byte, 0, size, (jbyte *)buffer);  // copy result to java layer

  return res_byte;
}
{% endhighlight %}

#### 3. In java: byte[]->protobuf
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