---
layout: post
title:  "Sed Useage"
date:   2016-01-19 15:30:00
tags: [linux, sed, command]
categories: Resources
---

[Useful Link](http://man.linuxde.net/sed) 

### 1. 删除文件特定行
{% highlight Bash shell scripts %}
sed -i '/^$/d' filename  # 删除空行
sed -i '/tags/d' filename  # 删除匹配tags的行
sed -i '/tags/d' `grep tags -r . -rl`  # 删除所有匹配tags的文件中的相应行，替换文件夹中所有文件
{% endhighlight %}

### 2. 去掉某一行开头的 注释井号
{% highlight Bash shell scripts %}
\(..\)  # 匹配子串，保存匹配的字符，如s/\(love\)able/\1rs，loveable被替换成lovers; 对于匹配到的第一个子串就标记为 \1，依此类推匹配到的第二个结果就是 \2
echo aaa BBB | sed 's/\([a-z]\+\) \([A-Z]\+\)/\2 \1/'  ==>  BBB aaa
sed 's/# \(.*multiverse$\)/\1/g' /etc/apt/sources.list  # Ubuntu Sources.list need to use multiverse
{% endhighlight %}
