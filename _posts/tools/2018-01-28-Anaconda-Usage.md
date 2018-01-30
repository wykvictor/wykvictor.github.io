---
layout: post
title:  "Anaconda Usage"
date:   2018-01-28 16:30:00
tags: [anaconda, conda, usage]
categories: Tools
---

> conda是用于python包管理和环境管理的工具，功能上类似pip和vitualenv的组合

{% highlight Bash shell scripts %}
conda -h   # 查看帮助
conda create --name python27 python=2.7  # 基于python2.7创建名为python27的环境
source activate python27      # 激活此环境
source deactivate python27    # 退出当前环境
conda env remove -n python27  # 删除该环境
conda info -e                 # 查看安装的环境
conda install opencv          # 安装包
{% endhighlight %}

如果包安装比较慢，可以添加清华镜像源：
{% highlight Bash shell scripts %}
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --set show_channel_urls yes
{% endhighlight %}
