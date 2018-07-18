---
layout: post
title:  "Remote desktop connect to Win10-Home"
date:   2018-07-16 10:30:57
categories: Others
---

### [Windows10 家庭版中使用远程桌面](https://www.appinn.com/windows-10-home-remote-desktop/)
1. Windows10原生不支持远程桌面，上述blog描述安装RDP Wrapper Library来进行连接
2. 连接前关掉防火墙
3. 另外，可以设置该RDP服务，开机自启动，更加方便远程连接：win+R, shell:startup，打开的目录里，将RDPConf-快捷方式复制进来

### 映射本地文件夹到远程Windows
1. net use <drive letter> \\tsclient\<drive letter>
2. 比如： net use X: \\tsclient\Work，之后可以用X:	操作该目录

### [Mac版本Windows远程桌面软件](https://www.jianshu.com/p/f30d35260b4b)
