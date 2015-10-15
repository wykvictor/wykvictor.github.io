---
layout: post
title:  "Docker commands"
date:   2015-10-15 15:30:00
tags: [docker, command]
categories: resources
---

### 1. 查看docker信息（version、info）
{% highlight Bash shell scripts %}
docker version
docker info
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
{% endhighlight %}

### 3. 操作docker container
{% highlight Bash shell scripts %}
# 启动container，挂载GPU卡和本地目录(dest目录:本地目录)，进入交互式bash
docker run -ti $DOCKER_NVIDIA_DEVICES --name container-name -v /mnt_data:/mnt_data1 image-name bash
# 退出但不停止container: ctrl p + ctrl q; exit 和 ctrl d都会暂停container，之后需要运行start启动
# 之后进入docker(-it 交互式伪终端)，ctrl+D/exit退出都不会退出container
docker exec -it 102d3e949a37 bash

# 查看运行的container
docker ps [-a]
# 保存对容器的修改; -a, --author="" Author; -m, --message="" Commit message  
docker commit ID new_image_name 
{% endhighlight %}