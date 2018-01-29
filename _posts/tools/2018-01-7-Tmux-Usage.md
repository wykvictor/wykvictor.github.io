---
layout: post
title:  "Tmux Usage"
date:   2018-01-27 16:30:00
tags: [tmux, usage]
categories: Tools
---

> Tmux是一款类似screen，但更好用的终端复用软件

会话相关：
{% highlight Bash shell scripts %}
tmux new -s name   # 新建名叫name的会话
tmux detach        # 退出当前会话，但不关闭，也可以用快捷键ctrl + b，d
tmux a -t name     # attach名称为name的会话
tmux ls            # 列出已有会话
tmux kill-session -t name  # 关闭名字为name的会话
{% endhighlight %}

ctrl+b, 常用快捷键，控制会话中的窗口：
{% highlight Bash shell scripts %}
%        # 左右两个分屏
空格键    # 左右分屏与上下分屏等切换
左右键    # 切换不同的分屏
ctrl+d   # 直接关闭当前的分屏，不需要ctrl+b
c/n/p/w  # 新建一个窗口/切换下一个/前一个/菜单选择
s        # 选择进入另一个会话
{% endhighlight %}
