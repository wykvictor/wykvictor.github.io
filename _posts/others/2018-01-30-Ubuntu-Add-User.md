---
layout: post
title:  "Ubuntu Add New User"
date:   2018-01-30 10:30:57
categories: Others
---

### 创建新用户
adduser username

### 将新用户添加到sudo组
usermod -aG sudo username


### 永久删除用户及其目录
sudo userdel -rf dangshunya
