---
layout: post
title:  "Some useful Shell Scripts to Process Text"
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
for var in *.list; do echo "Processing $var..."; mv "$var" "`echo $var|awk -F '_' '{print $1}'`.txt"; done
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
