---
layout: post
title:  "Setup SSH service in Docker"
date:   2016-03-13 16:00:00
tags: [docker, ssh, network]
categories: Tech
---

> [ssh原理](https://www.jianshu.com/p/33461b619d53)

### 1. Log into ssh service using passwd
Please refer to [docker doc](https://docs.docker.com/engine/examples/running_ssh_service/)

### 2. Without passwd between inter-docker-containers(ssh-key)
Dockerfile:
{% highlight Bash shell scripts %}
FROM ubuntu:14.04

# install ssh
RUN apt-get update && apt-get install -y openssh-server
RUN mkdir /var/run/sshd

# 当第一次连接服务器时，自动接受新的公钥, do not need to input yes
RUN echo "StrictHostKeyChecking no" >> /etc/ssh/ssh_config
# SSH login fix.
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
# generate an SSH key
RUN /usr/bin/ssh-keygen -f /root/.ssh/id_rsa -t rsa -N ''
# add its ssh keys to authorized_keys
RUN cp /root/.ssh/id_rsa.pub /root/.ssh/authorized_keys

CMD ["/usr/sbin/sshd", "-D"]
{% endhighlight %}
Then test it:
{% highlight Bash shell scripts %}
$ docker build -t blog_sshd .
$ docker run -d -h host-ssh --name ssh-1 blog_sshd  # hostname is host-ssh, rather than ab09325101ec
# [use link to export hostname](https://docs.docker.com/engine/userguide/networking/default_network/dockerlinks/) 
# bash -c to execute multi-commands within one line
$ docker run --rm --link ssh-1:link-host-ssh blog_sshd bash -c "hostname && ssh link-host-ssh hostname"
c122ce37b8d0
Warning: Permanently added 'link-host-ssh,172.17.0.4' (ECDSA) to the list of known hosts.
host-ssh
# 此时查询env, 有 LINK_HOST_SSH_NAME=/c122ce37b8d0/link-host-ssh
{% endhighlight %}

### 3. Containers on different hosts need to export port and then ssh via host

### 4. 利用host key完成git clone等需要认证的操作
Dockerfile:
{% highlight Bash shell scripts %}
RUN mkdir -p /root/.ssh \
&& chmod 0700 /root/.ssh
# add ssh keys
ARG SSH_PRIVATE_KEY
ARG SSH_PUB_KEY
RUN echo "${SSH_PRIVATE_KEY}" > /root/.ssh/id_rsa \
&& chmod 600 /root/.ssh/id_rsa \
&& echo "${SSH_PUB_KEY}" > /root/.ssh/id_rsa.pub \
&& chmod 600 /root/.ssh/id_rsa.pub \
&& ssh-keyscan private.github.com >> /root/.ssh/known_hosts

RUN cd /root \
&& git clone -b my_branch --single-branch git@git.private.github.com:private.git \
&& cd private && ./build.sh

# 最后注意删掉key，其实也不完全安全：build时--squash操作可以压缩所有的image层为一个，看不到之前存在key的层，就安全了
RUN rm -rf /root/.ssh/

# 之后build image
docker build -t image_name --build-arg SSH_PRIVATE_KEY="$(cat ~/.ssh/id_rsa)" --build-arg SSH_PUB_KEY="$(cat ~/.ssh/id_rsa.pub)" .
{% endhighlight %}
