---
layout: post
title:  "SSH Config"
date:   2018-06-26 16:30:00
tags: [ssh, config, usage]
categories: Tools
---

### 1. 免密码登陆
{% highlight Bash shell scripts %}
ssh-keygen -t rsa
cat .ssh/id_rsa.pub
# 将此文件内容，拷贝到服务器的.ssh/authorized_keys
{% endhighlight %}

### 2. 利用ControlPersist特性自动登陆SSH服务器
对于有跳板机存在的情况，在~/.ssh/config中，添加：
{% highlight Bash shell scripts %}
Host tiaobanji
User username-abc  # user name
HostName tiaobanji.a.com  # 服务器地址
ControlPersist yes
ControlMaster auto
ControlPath ~/.ssh/ssh_%h_%p_%r
Compression yes
{% endhighlight %}
之后可以ssh tiaobanji直接登陆，并且保持长链接一段时间，不需要重复输入动态密码
