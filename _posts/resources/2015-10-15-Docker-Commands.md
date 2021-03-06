---
layout: post
title:  "Docker Commands"
date:   2015-10-15 15:30:00
tags: [docker, command]
categories: Resources
---

### 1. 查看docker信息（version、info）
{% highlight Bash shell scripts %}
docker version
docker info
docker inspect -f '{{ .NetworkSettings.IPAddress }}' name  # lookup the container's IP
{% endhighlight %}

### 2. 操作docker image
{% highlight Bash shell scripts %}
$ docker images  # 列出镜像
REPOSITORY             TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
<none>                 <none>              128d249664a0        7 weeks ago         5.445 GB
$ docker tag 128d249664a0 yakun-gpu:latest  # 给镜像打tag，命名
REPOSITORY             TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
yakun-gpu              latest              128d249664a0        7 weeks ago         5.445 GB
$ docker history yakun-gpu  # 查询image历史
IMAGE               CREATED              CREATED BY                                      SIZE
128d249664a0        7 weeks ago          /bin/bash                                       39.08 MB
e0aca483823a        5 months ago         /bin/bash                                       24.88 MB
9516a43a7bd5        6 months ago         /bin/bash                                       21 MB
ab33a0ce9258        9 months ago         /bin/sh -c apt-get -y install bash-completion   5.294 MB

$ docker search caffe-gpu  # 从docker hub上检索镜像
NAME                           DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
tleyden5iwx/caffe-gpu-master                                                   20                   [OK]
nakosung/caffe-gpu                                                             0                    [OK]
$ docker pull image-name  # download image
$ docker rmi image_name  # delete image
$ docker rmi $(docker images -a -f "dangling=true" -q)  # dangling的镜像

#导出image，并在其他的机器导入
$ docker save  -o save-img.tar image-name:tag
$ docker load -i save-img.tar
{% endhighlight %}

### 3. 操作docker container
{% highlight Bash shell scripts %}
# 启动container，挂载GPU卡和本地目录(本地主机目录:image内目录)，进入交互式bash
docker run -ti $DOCKER_NVIDIA_DEVICES --name container-name -v $(cd "$(dirname $0)/.."; pwd):/mnt_data1 image-name bash
# 退出但不停止container: ctrl p + ctrl q; exit 和 ctrl d都会暂停container，之后需要运行start启动
# 之后进入docker(-it 交互式伪终端)，ctrl+D/exit退出都不会退出container
docker exec -it 102d3e949a37 bash
# 删除所有已经exit的container -q(id号) -f(filter) -v:移除挂载目录(如果不加，会残留垃圾文件)
docker rm -v $(docker ps -a -q -f status=exited)

# 查看运行的container
docker ps [-a]
# 保存对容器的修改; -a, --author="" Author; -m, --message="" Commit message  
docker commit ID new_image_name 
{% endhighlight %}

### 4. docker占用资源查询
{% highlight Bash shell scripts %}
docker stats [docker names]  // 查看CPU，Memory，Network使用
docker内存限制可以在创建docker时使用-m参数：-m 256m，容器里程序可以跑到256m*2=512m后会被oom给杀死
目前cpu限制可以使用绑定到具体的线程，或者是在绑定线程基础上对线程资源权重分配。绑定线程可以使用参数--cpuset-cpus=7
{% endhighlight %}

### 5. [Dockerfile](https://docs.docker.com/engine/articles/dockerfile_best-practices/)
{% highlight Bash shell scripts %}
# 可以从Dockerfile，自己建立image:
FROM docker/whalesay:latest  # 基于哪个镜像
# 安装软件用,常用，不要忘记清空lists目录，可以缩小image大小
RUN apt-get -y update && apt-get install -y fortunes && rm -rf /var/lib/apt/lists/*
ENV PATH ${PATH}:/opt/tools  # 设置环境变量
COPY requirements.txt /tmp/  # copying local files into the container 
WORKDIR /root/workdir  # 设置之后所有RUN命令的工作目录
CMD /usr/games/fortune -a | cowsay # container启动时执行的命令，但只能有一条CMD命令，多条则只执行最后一条
# 之后基于此构建image whale-yk
docker build -t whale-yk . # 当前目录找dockerfile
{% endhighlight %}
对于Centos机器，不同的地方：
{% highlight Bash shell scripts %}
RUN curl -o /etc/yum.repos.d/CentOS7-Base-163.repo  http://mirrors.163.com/.help/CentOS7-Base-163.repo \
&& curl -o /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7 https://archive.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-7 \
&& yum clean all \
&& yum makecache \
&& yum -y -t groupinstall "Development Tools" \
。。。
&& yum clean all
{% endhighlight %}
另，Dockerfile中每一条命令是一个step，在image中新加一层（新建一个container，然后操作，然后rm掉container，得到一个加一层的image）
所以修改某一条命令后，之前的Cache，之后的丢掉重新建
