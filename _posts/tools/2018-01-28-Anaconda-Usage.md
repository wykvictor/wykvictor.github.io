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
conda list | grep opencv      # 列出已经安装的包
{% endhighlight %}

如果包安装比较慢，可以添加清华镜像源：
{% highlight Bash shell scripts %}
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --set show_channel_urls yes
{% endhighlight %}

yml文件：
{% highlight Bash shell scripts %}
conda env export > environment.yml
conda env create -f environment.yml  # 通过environment.yml导出并新建一个同样的conda环境
{% endhighlight %}

share conda环境：
{% highlight Bash shell scripts %}
conda create -p C:/full/public/path/to/py35 python=3.5  # -p 指定安装环境目录
# 将目录添加到其他用户的conda configuration file .condarc
envs_dirs:
  - C:/full/public/path/to  # 注意，控制该目录的权限，开放给其他用户，以免遇到问题
{% endhighlight %}
