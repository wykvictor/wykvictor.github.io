---
layout: post
title:  "Install Nvidia Driver on Ubuntu 18.04"
date:   2018-09-12 10:30:57
categories: Others
---

> [reference](https://www.linuxbabe.com/ubuntu/install-nvidia-driver-ubuntu-18-04)

### Command Line 安装
{% highlight Bash shell scripts %}
sudo lshw -c display  # 查看系统的显卡和相应的驱动情况，安装前为默认的 driver=nouveau
sudo ubuntu-drivers devices  # list available driver for your Nvidia card
sudo apt install nvidia-driver-390  # 安装特定版本的driver
# Note: 在安装过程中，如果提示UEFI fast boot模式安装，需要生成key，输入8位数字密码
sudo shutdown -r now
# 重启后，driver启动，如果提示Enroll Key，选择进入，OK，输入上边的8位数字密码，即可
sudo lshw -c display  # 此时查看，driver=nvidia
prime-select query  # 查看启动的显卡，是集成显卡，还是N卡
sudo prime-select intel/nvidia  # 选择启动哪块显卡
{% endhighlight %}

### 如果CMake仍然无法找到OpenGL
sudo apt-get install freeglut3-dev
sudo apt-get install libglew-dev

### 安装过程中，难免遇到无法启动的状况，需要提前配置好静态IP，这样远程还可以连接命令行进行操作，防止变成板砖
