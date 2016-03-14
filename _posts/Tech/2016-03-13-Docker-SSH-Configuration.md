---
layout: post
title:  "Setup an SSH daemon service in Docker"
date:   2016-03-13 16:00:00
tags: [docker, ssh, network]
categories: Tech
---

### 1. Log into ssh service using passwd
Please refer to [docker doc](https://docs.docker.com/engine/examples/running_ssh_service/)

### 2. Log into ssh service without passwd inter-docker-containers(ssh-key)
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
docker build -t blog_sshd .
docker run -d --name ssh-1 blog_sshd
docker run -d --name ssh-1 blog_sshd
{% endhighlight %}
