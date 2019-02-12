---
layout: post
title:  "Some Useful Shell Resources"
date:   2015-10-09 15:30:00
tags: [shell, scripts, 脚本]
categories: Resources
---

### 1. 将20类的图片按照类别copy到相应的文件夹

> 最近在用Deep learning工具[caffe](http://caffe.berkeleyvision.org/)，基于数据集[VOC](http://host.robots.ox.ac.uk/pascal/VOC/)做一些实验，里边有20个Object Class，所以得到结果后经常会遇到重复处理20遍文件。因此写了一些脚本，用来执行重复操作。

a. 以训练(Train)图片为例，首先根据每一类的初始列表如cat_trainval.txt(20个相同类型文件):

000005 0

000023 1

...

(ImageID Label)

得到每一类正例(Label=1)的文件名列表*.list:
{% highlight Bash shell scripts %}
for i in *_trainval.txt; do echo $i; grep " 1" $i | awk '{print $1}' > $i.list; done
{% endhighlight %}
b. 将该20个文件(cat_trainval.txt.list,...,etc)，批量重命名成cat.txt格式方便后边操作:
{% highlight Bash shell scripts %}
for var in *.list; do echo "Processing $var..."; mv "$var" "`echo $var|awk -F '_' '{print $1}'`.txt"; done  #批量重命名
{% endhighlight %}
c. 开始分类copy:
{% highlight Bash shell scripts %}
for j in cat dog person plane bike bird boat bottle bus car chair cow table horse motorbike plant sheep sofa train tvmonitor; do 
  for i in `cat $j.txt` ; do
    mkdir $j;
    cp ../../VOC2007train/VOCtrainval_06-Nov-2007/VOCdevkit/VOC2007/JPEGImages/$i.jpg ./$j/; 
  done; 
done
{% endhighlight %}

### 2. 等待脚本：一旦某个条件符合，就执行特定操作
用GPU训练Caffe Model的时候，因为跟大家共享GPU资源，因此很多时候内存不够用，需要等待。简单写了个等待脚本，一旦发现GPU使用量有变化，就开始执行自己的task：
{% highlight Bash shell scripts %}
while [[ `nvidia-smi | grep 3862 | wc -l` -gt 0 ]]; do echo "wait..."; sleep 120; done; echo "Start!"; ./mytask.sh
{% endhighlight %}
(每隔120s查询一次while循环条件，终止条件可以根据情况自定义)

### 3. Caffe训练的时候，直接把所有的输出都record下来了，设置的是每一个iter都有输出，想要得到如每40个iters的平均的loss曲线
{% highlight Bash shell scripts %}
#!/bin/bash
# Usage: ./processLog.sh LogOrigionalName cat(or dog/flower...)
# Output file: LogOrigionalName-cat.csv
grep "Train net output" $1 | grep $2 | awk '{print $15}' > $1-$2.tmp
awk 'BEGIN{sum=0;}{sum=sum+$1;if(NR%40 == 0){printf("%.8f\n", sum/40); sum=0;}}' $1-$2.tmp > $1-$2.csv  #利用awk求均值
rm $1-$2.tmp
{% endhighlight %}

### 4. 抽取某一个class如plant的feature，之后和其他class的ground truth结合成新的feature文件
在Train SVM model的时候，用了开源库liblinear，基于每一个class的输入图片list来抽取feature。

但是由于每个class的图片列表都相同，所以重复抽取feature不仅浪费时间而且没有意义，可以利用以下脚本组合成新的feature文件：
{% highlight Bash shell scripts %}
for i in `ls VOCtrainFilelist`; do  #文件夹VOCtrainFilelist中有20个classes的list文件
  echo "VOCtrainFilelist/$i"
  sed 's/^[0-1]\s//g' plant/train_data.txt > 1.tmp  #利用plant抽取好的feature，删除第一列label
  awk '{print $2}' VOCtrainFilelist/$i > 2.tmp  #ground truth列表，新class的label
  paste 2.tmp 1.tmp > 3.tmp  # 将2个文件合并成新class的feature文件(label+features)
done
rm 1.tmp 2.tmp 3.tmp
{% endhighlight %}

### 5. 对一个文件的某一列进行数值计算
有时会遇到某一列probability的ground truth打反了，需要将prob一列修改为1-prob：
{% highlight Bash shell scripts %}
awk '{printf("%s %.14f\n", $1, 1.0-$2)}' origin > origin-flip #源文件有2列，第2列为prob
{% endhighlight %}

### 6. 批量create20个class的lmdb文件
{% highlight Bash shell scripts %}
#!/bin/bash
# Usage: ./processLog.sh origionalNameList (./sh *_trainval.txt)

list=($@)  # get input parameter's list
for i in ${list[@]}; do  # loop over the parameters(different classes)
  echo "Processing $i..."  # for each class's file lists
  awk '{print $2}' $i > $i-gt.tmp  # get ground-truth label list(different from the caffe's requirement)
  for val in `cat $i-gt.tmp`  # flip labels: -1->0; 0->0; 1->1 (maybe `sed` could better fit the need)
    do
      if [ $val = "-1" ]; then
        echo "0" >>  $i-gt-VOC.tmp
      elif [ $val = "0" ]; then
        echo "0" >>  $i-gt-VOC.tmp
      else
        echo "1" >>  $i-gt-VOC.tmp
      fi
    done
  paste list.txt $i-gt-VOC.tmp > $i.list
  rm ./*.tmp
  # use the list to get the lmdb
  caffe.git/build/tools/convert_imageset -resize_height 256 -resize_width 256 -shuffle / $i.list $i-lmdb
  rm ./*.list
done
{% endhighlight %}

### 7. awkfile的应用：先匹配，再操作
对于稍复杂的操作，不便于在一行命令操作，可以创建awk file供awk命令使用，比如之前用到的一个：
{% highlight Bash shell scripts %}
# Parse caffe logs
# Usage: awk -f log.awk logs > loss.csv

# Set output field seperator as comma used in .csv file
BEGIN { OFS = "," }

# Match loss lines
/.*] Iteration [0-9]+, loss = [0-9]+\.[0-9]+/ {
  print format_time($2), $NF  # call function, print "time + loss value"
}

function format_time(time) {
  nano = substr(time, length(time) - 6)  # use substr()
  time = substr(time, 1, length(time) - 7)
  # print time, nano
  gsub(/:/, " ", time)  # use gsub()
  return time nano  # return str
  # return mktime(strftime("%Y ") month " " day " " time) nano
}
{% endhighlight %}

### 8. 查看某个文件有多少列
某个文件很长，列数不规则，可以通过以下命令判断，最多的大概有多少列：
{% highlight Bash shell scripts %}
awk '{if($7 != "") print}' list.txt | wc -l
{% endhighlight %}

### 9. 删除文件特定行
{% highlight Bash shell scripts %}
sed -i '/^$/d' filename  # 删除空行
sed -i '/tags/d' filename  # 删除匹配tags的行
{% endhighlight %}

### 10. vim中的匹配和替换
{% highlight Bash shell scripts %}
%s/_mean_[a-z]*.binary/_mean.binary/  # 将 “_单词“ 删掉
%s/_mean_.*512.binary/_mean_512.binary/g  # 将512之前的单词删掉
{% endhighlight %}

### 11. function 函数的使用
{% highlight Bash shell scripts %}
#!/bin/bash
function fSum()  // 函数定义 必须放在使用前
{
#   echo $1,$2;
   echo $(($1+$2));  // 如果没有显示的return语句，则最后一句作为返回值
}
t=$(fSum 3 2)  // 相当于定义了fSum命令，传入参数
echo $t, $?  // 5, 0，$?是上一句命令的返回值，0代表成功

log() {  # classic logger
# 注意，shell函数中定义的变量是global的，其作用域从函数被调用执行变量的地方开始，到shell结束或者显示删除为止。函数定义的变量可以是local的，其作用域局限于函数内部，但是函数的参数是local的。
    local prefix="[$(date "+%Y/%m/%d %H:%M:%S")]: "
    echo "${prefix} $@" >&2  # 重定向到标准错误
}
log "INFO" "a message"
{% endhighlight %}

### 12. 批量杀死进程
{% highlight Bash shell scripts %}
ps aux|grep name|grep -v grep|awk '{print $2}'|xargs kill -9
{% endhighlight %}

### 13. 查看Linux内核版本或发布版本
{% highlight Bash shell scripts %}
lsb_release -a
cat /etc/issue
uname -a
{% endhighlight %}

### 14. 下载并安装某包
{% highlight Bash shell scripts %}
#!/usr/bin/env bash

LINK="http://.../download/gflags-2.1.2.tar.gz"
TARBALL=gflags_v2.1.2.tar.gz
WD=$(readlink -f "`dirname $0`/")
DOWNLOAD_DIR=${WD}/download
GFLAGS_ROOT=${WD}/gflags-2.1.2
INSTALL_DIR=${WD}/../build/
BUILD_DIR=${INSTALL_DIR}/build_gflags/

N_JOBS=8

[ ! -d ${DOWNLOAD_DIR} ] && mkdir -p ${DOWNLOAD_DIR}

cd ${DOWNLOAD_DIR}
if [ ! -f ${TARBALL} ]; then
    wget ${LINK} -O ${TARBALL}
fi
cd ${WD}

rm -rf ${GFLAGS_ROOT}
cd ${GFLAGS_ROOT}
tar zxf "${DOWNLOAD_DIR}/${TARBALL}"

mkdir -p "${BUILD_DIR}"
cd "${BUILD_DIR}"

cmake -D... \
      ${GFLAGS_ROOT}

make -j${N_JOBS}
rm -rf "${INSTALL_DIR}/gflags"
make install/strip

cd "${WD}"
{% endhighlight %}

### 15. 批量重命名文件
某个文件夹下1.txt, 1000.txt, 批量重命名成000001.txt和001000.txt, 这样程序读取时可以按顺序
{% highlight Bash shell scripts %}
for i in `ls *`; do mv -- $i `echo $i | awk -F. '{printf("%06d\n", $1)}'`.txt; done # --避免$i开头有'-'
{% endhighlight %}
目录里包含很多子目录，每个子目录里有很多文件，将文件重命名将其目录名字加到前边
{% highlight Bash shell scripts %}
for i in `ls`; do pushd $i; ls | awk -v dir=$i '{print "mv " $1 " " dir$1}' | bash; popd; done
{% endhighlight %}
其中，注意awk需要external外部的变量时，需要用-v参数引入

### 16. 批量删除大小为0的文件
{% highlight Bash shell scripts %}
find . -name "*" -type f -size 0c | xargs -n 1 rm -f
{% endhighlight %}

### 17. 双层循环,依次取每100行，刪除空行
{% highlight Bash shell scripts %}
for j in {1..5}; do
for i in {1..15}; do echo $(($i*100)); head -$(($i*100)) dis1_$j.txt | tail -100 | awk '{print $8}' > tag$j/$j\_$i.txt; sed -i '/^$/d' tag$j/$j\_$i.txt; done
done
{% endhighlight %}

### 18. 統計代碼文件行數
{% highlight Bash shell scripts %}
wc -l `find . -name "*.cpp"` | awk 'BEGIN{sum=0;} {sum=sum+$1;} END{print sum}'
{% endhighlight %}

### 19. awk统计均值/最大值
{% highlight Bash shell scripts %}
# 求平均
awk '{sum+=$1} END {print "Average = ", sum/NR}'
# 求最大值
awk 'BEGIN {max = 0} {if ($1>max) max=$1 fi} END {print "Max=", max}'
# 求最小值
awk 'BEGIN {min = 1999999} {if ($1<min) min=$1 fi} END {print "Min=", min}'
{% endhighlight %}

### 20. 每隔10行统计一次均值，一共100行
{% highlight Bash shell scripts %}
for i in {1..10}; do grep "thread:" server.log | awk '{print $4}' | awk -F, '{print $1}' | head -$(($i*10)) | tail -10 | awk '{sum+=$1} END {print "Average = ", sum/NR}'; done
{% endhighlight %}
