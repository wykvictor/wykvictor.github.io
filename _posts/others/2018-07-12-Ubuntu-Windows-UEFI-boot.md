---
layout: post
title:  "Install Ubuntu together with Windows10 by UEFI boot"
date:   2018-07-12 10:30:57
categories: Others
---

> [UEFI](https://baike.baidu.com/item/UEFI/3556240?fr=aladdin): 一种代替BIOS，更加快速的启动方式

### UEFI模式安装Ubuntu
UEFI模式需要单独分一个200M左右的区给EFI，否则在Windows10系统基础上直接安装会出现错误提示『无法将grub-eif-amd64-signed软件包安装到/target/中』。

### 具体步骤
1. 在手动分区选项，手动创建200M分区，并选择文件系统为EFI即可 
2. 其他分区比如根分区，还跟普通安装设置相同
3. 安装完成后，可能快速启动默认启动Ubuntu，需要按F12切换启动系统。另外在BIOS里可以改默认启动的系统。
