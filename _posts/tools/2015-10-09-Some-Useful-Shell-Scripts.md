---
layout: post
title:  "Some useful Shell Scripts"
date:   2015-10-09 15:30:00
tags: [shell, scripts, 脚本]
categories: tools
---

> 最近在用Deep learning工具[caffe](http://caffe.berkeleyvision.org/)，基于数据集[VOC](http://host.robots.ox.ac.uk/pascal/VOC/)做一些实验，里边有20个Object Class，所以得到结果后经常会遇到重复处理20遍文件。因此写了一些脚本，用来简化操作。

### 1. 将20类的图片按照类别copy到相应的文件夹
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
(Note: for循环的另一种常用写法：for i in {1..5})

### 2. 等待脚本：一旦某个条件符合，就执行特定操作
用GPU训练Caffe Model的时候，因为跟大家共享GPU资源，因此很多时候内存不够用，需要等待。简单写了个等待脚本，一旦发现GPU使用量有变化，就开始执行自己的task：
{% highlight Bash shell scripts %}
while [[ `nvidia-smi | grep 3862 | wc -l` -gt 0 ]]; do echo "wait..."; sleep 120; done; echo "Start!"; ./mytask.sh
{% endhighlight %}
(每隔120s查询一次while循环条件，条件可以根据情况自定义)

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