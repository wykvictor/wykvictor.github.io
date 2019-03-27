---
layout: post
title:  "Some Shell Tricks"
date:   2016-01-19 15:30:00
tags: [linux, shell, trick]
categories: Resources
---

[shell进阶](http://www.361way.com/bash-15minutes-advanced/3248.html)

### 1. shell字符串操作
截取操作
{% highlight Bash shell scripts %}
f="path1/path2/file.ext"
len="${#f}" # = 20 (字符串长度)
# 切片操作: ${<var>:<start>} or ${<var>:<start>:<length>}
slice1="${f:6}" # = "path2/file.ext"
slice2="${f:6:5}" # = "path2"
slice3="${f: -8}" # = "file.ext"(注意："-"前有空格)
pos=6
len=5
slice4="${f:${pos}:${len}}" # = "path2" 
{% endhighlight %}

替换操作
{% highlight Bash shell scripts %}
f="path1/path2/file.ext"
single_subst="${f/path?/x}"   # = "x/path2/file.ext"
global_subst="${f//path?/x}"  # = "x/x/file.ext"
# 字符串拆分
readonly DIR_SEP="/"
array=(${f//${DIR_SEP}/ })
second_dir="${arrray[1]}"     # = path2 
{% endhighlight %}

删除头部或尾部
{% highlight Bash shell scripts %}
$ a="x y"
$ echo ${a/ */}  # output: x
$ echo ${a/x /z} # output: zy
$ echo ${a% y}  # output: x 从后匹配百分号之后的串，并删除
$ echo ${a#* }  # output: y 从前匹配百分号之后的串，并删除，可以用通配符如*

f="path1/path2/file.ext"
# 删除字符串头部
extension="${f#*.}"  # = "ext"
# 以贪婪匹配方式删除字符串头部
filename="${f##*/}"  # = "file.ext"
# 删除字符串尾部
dirname="${f%/*}"    # = "path1/path2"
# 以贪婪匹配方式删除字符串尾部
root="${f%%/*}"      # = "path1"
{% endhighlight %}

### 2. if [[ ]] 正则表达式
{% highlight Bash shell scripts %}
$ platform="android-armeabi-v7a"
$ if [[ $platform == "android"* ]];  # match
$ if [[ $platform =~ "arm" ]];  # match，可以不加引号
$ if [[ $platform == "arm" ]];  # not match
$ if [[ $platform == *"arm"* ]];  # match
{% endhighlight %}

### 3. 判断目录是否存在，不存在Do sth
{% highlight Bash shell scripts %}
[ ! -d ${INSTALL_DIR} ] && mkdir -p ${INSTALL_DIR}
# ln is very tricky:
[ ! -d apk ] && ln -s src/android/app/build/outputs/apk apk  # link dir, if dir exists will create apk/apk WRONG!
ln -s ../src/android/app/build/outputs/apk/app-arm64-debug.apk apk/app-arm64-debug.apk  # above is better
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

### 6. for循环的用法
{% highlight Bash shell scripts %}
#!/bin/bash
for((i=1; i<=10; i++));do echo $(expr $i \* 4);done
for i in {1..5}
for i in $(seq 2 $n)
awk 'BEGIN{for(i=1; i<=10; i++) print i}'  # AWK中的for写法和C语言一样

list=($@)  # get input parameter's list
for i in ${list[@]}; do  # loop over the parameters
  echo "Processing $i..."
done
{% endhighlight %}

### 7. 使用cat<<EOF代替echo -e进行多行输出
{% highlight Bash shell scripts %}
cat << EOF >> a.txt  # 输出到a.txt, 或者直接打印到屏幕
      Usage:
      1. For master:
      	 \$var
      	 ...
      2. For slave:
         ...
EOF  # 当然也不必用EOF，用AAA等也可以，只是个标识;必须顶格写
{% endhighlight %}

### 8. read -p "Press Enter to continue..."

### 9. 使用getopt parse输入参数
{% highlight Bash shell scripts %}
#!/bin/bash
ARGS=`getopt -o hp:i -l "help,port:,intOrnot" -- "$@"`
eval set -- "${ARGS}"
usage() {
    echo "usage:start.sh -h -p 6692 -i"
}
port=6692
isInt=false

while true;
do
    case "$1" in
        --port|-p)
            fkp_port=$2
            shift 2
            ;;
        --help|-h)
            usage
            exit 0
            ;;
        --intOrnot|-i)
            isInt=true
            shift 1
            ;;
        --)
            shift
            break
            ;;
        *)
            echo "error!"
            exit 1
            ;;
    esac
done
{% endhighlight %}

### 10. android-accept-license.sh
{% highlight Bash shell scripts %}
#!/usr/bin/expect -f

set timeout 1800
set cmd [lindex $argv 0]
set licenses [lindex $argv 1]

spawn {*}$cmd
expect {
  "Do you accept the license '*'*" {
        exp_send "y\r"
        exp_continue
  }
  eof
}
{% endhighlight %}

### 11. sed命令：正则表达式
{% highlight Bash shell scripts %}
echo "Point 5:  [237, 1051]" | sed 's/.*\[\(.*\)\]/\1/'  # output: 237, 1051，打印匹配的模式(括号内)
{% endhighlight %}
