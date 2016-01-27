---
layout: post
title:  "Ubuntu apt-get cacher Server"
date:   2016-01-27 22:30:00
tags: [Ubuntu, apt-get, cacher]
categories: Tools
---

### 1. 概念
若内网多台机器需要配置相同的环境，apt-get相同的包，则可以搭建内网apt cacher服务器

只要有其中一台通过该私服下载安装过deb包，apt-cacher都会缓存，下一个Ubuntu的请求就直接从缓存中获取

### 2. 客户端配置：
{% highlight Bash shell scripts %}
echo 'Acquire::http { Proxy "http://10.xx.xx.60:3142"; }; ' >> /etc/apt/apt.conf.d/01proxy
apt-get update
{% endhighlight %}

### 2. cacher服务端配置：
{% highlight Bash shell scripts %}
apt-get install apt-cacher
/etc/apt-cacher/apt-cacher.conf: allowed_hosts = *
service apt-cacher restart
{% endhighlight %}
之后，可以访问http://10.42.11.60:3142，cacher服务监听3142端口
