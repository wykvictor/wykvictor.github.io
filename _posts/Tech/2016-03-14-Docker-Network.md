---
layout: post
title:  "Docker Network and ADB Devices"
date:   2016-03-14 16:00:00
tags: [docker, network, link, multi-host, swarm]
categories: Tech
---

### 1. Docker Container网络配置
[[reference]](http://www.infoq.com/cn/articles/docker-network-and-pipework-open-source-explanation-practice/)
docker run创建容器时，可以用--net选项指定网络模式，有以下4种：

* host模式，--net=host：无独立的Network Namespace，不虚拟网卡，使用宿主机的IP和端口(最简单的直接上网的办法)
* container模式，--net=container:NAME_or_ID : 与上种类似，只是与其他的某个容器共享网络资源
* none模式，--net=none：独立Network Namespace，但是并不进行任何网络配置，需添加网卡、配置IP等
* bridge模式，--net=bridge，默认设置

*bridge模式*
为每一个容器分配Network Namespace、设置IP等，并将一个主机上的Docker容器连接到一个虚拟网桥上：

![docker-network](/res/docker-network.png)

1. 在主机上创建一对虚拟网卡veth pair设备。veth设备总是成对出现的，它们组成了一个数据的通道，数据从一个设备进入，就会从另一个设备出来。因此，veth设备常用来连接两个网络设备。
2. Docker将veth pair设备的一端放在新创建的容器中，并命名为eth0。另一端放在主机中，以veth65f9这样类似的名字命名，并将这个网络设备加入到docker0网桥中
3. 从docker0子网中分配一个IP给容器使用，并设置docker0的IP地址为容器的默认网关。
{% highlight Bash shell scripts %}
$ brctl show
bridge-name  bridge id   STP enabled   interfaces
docker0	8000.024261f0bd87	no	veth1cd2227, veth828ef90
{% endhighlight %}

### 2. [Multi-host networking](https://docs.docker.com/engine/userguide/networking/get-started-overlay/)
Refer to the above docker-doc

### 3. Connect adb devices from Docker in Ubuntu
[ref](http://learningbysimpleway.blogspot.com/2018/02/how-to-connect-adb-devices-to-linux.html)
{% highlight Bash shell scripts %}
adb kill-server # kill any running instance in host machine , other wise adb devices will not be visisble in container
docker run -it --privileged -v /dev/bus/usb:/dev/bus/usb --net=host 。。。
{% endhighlight %}

### 4. Connect adb devices from Docker in MacOS
##### a. [ref1](http://gw.tnode.com/docker/docker-machine-with-usb-support-on-windows-macos/)
通过docker-machine安装VirtualBox，之后再安装docker，步骤同3(这里对于大的docker image，有可能由于太大无法正常倒入)

##### b. [ref2](https://testerhome.com/topics/12489)
直接在mac上安装Virtual Box，再在里头安装Ubuntu，之后步骤同3
