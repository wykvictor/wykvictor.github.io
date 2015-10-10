---
layout: post
title:  "Some useful Python Scripts"
date:   2015-10-10 15:20:00
tags: [python, scripts, 脚本]
categories: resources
---


### 1. 对格式稍微复杂的3个文件，对应的域求平均
需求是对3个feature文件的对应域求平均，feature文件的格式如下：

0 0:0.0 1:0.0003 2:0.0 ....

(label featureID0:value featureID1:value featureID2:value ....)

label都相同，后边每一个featureID的值求平均。用shell写稍复杂，故用python进行处理：
{% highlight Python %}
import sys
outfile = open(sys.argv[1], 'w')
file1_lines = open(sys.argv[2], 'r').readlines() #大的数组按行存放所有的文本
file2_lines = open(sys.argv[3], 'r').readlines()
file3_lines = open(sys.argv[4], 'r').readlines()

for i in range(len(file1_lines)): #循环每一行
	if i % 100 == 0:
		print i
    list1 = file1_lines[i].split()
    list2 = file2_lines[i].split()
    list3 = file3_lines[i].split()
    outfile.write(str(list1[0]))
    for j in range(1, len(list1)): #循环处理每一行的feature value
	    val1 = list1[j].split(':')[1]   
	    val2 = list2[j].split(':')[1]   
	    val3 = list3[j].split(':')[1]   
	    avg = (float(val1) + float(val2) + float(val3)) / 3
	    outfile.write('\t' + str(j) + ':' + str(avg))
    outfile.write('\n')  #don't forget to end this line
outfile.close()
{% endhighlight %}