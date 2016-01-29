---
layout: post
title:  "Some Shell Tricks"
date:   2016-01-19 15:30:00
tags: [linux, shell, trick]
categories: Resources
---

### 1. 变量赋值时直接替换某些字符
{% highlight Bash shell scripts %}
$ a="x y"
$ echo ${a/ */}  # output: x
$ echo ${a/x /z} # output: zy
{% endhighlight %}

### 2. if [[ ]] 正则表达式
{% highlight Bash shell scripts %}
$ platform="android-armeabi-v7a"
$ if [[ $platform == "android"* ]];  # match
$ if [[ $platform =~ "arm" ]];  # match
$ if [[ $platform == "arm" ]];  # not match
$ if [[ $platform == *"arm"* ]];  # match
{% endhighlight %}

### 3. 判断目录是否存在，不存在创建
{% highlight Bash shell scripts %}
[ ! -d ${INSTALL_DIR} ] && mkdir -p ${INSTALL_DIR}
{% endhighlight %}

### 4. pushd popd进某目录做事再返回
{% highlight Bash shell scripts %}
mkdir -p ${BUILD_DIR}/build_glog
pushd ${BUILD_DIR}/build_glog
cmake $param -DCMAKE_INSTALL_PREFIX=${INSTALL_DIR} ${SRC_DIR}/../glog/
make -j${N_JOBS} || exit 1
make install
popd
{% endhighlight %}

### 5. 判断变量是否存在
{% highlight Bash shell scripts %}
if [ -z "$NDK_ROOT" ] && [ "$#" -eq 0 ]; then  # -z 判断环境变量；if可以用&&
    echo 'Either $NDK_ROOT should be set or provided as argument'
    echo "e.g., 'export NDK_ROOT=/path/to/ndk' or"
    echo "      '${0} /path/to/ndk'"
    exit 1
else
    NDK_ROOT="${1:-${NDK_ROOT}}"  # ":-"表示 存在则：默认更新为传入的$1，否则还等于原值
fi
{% endhighlight %}