---
layout: post
title:  "Tmux/Iterm2 Usage"
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

> Iterm2

{% highlight Bash shell scripts %}
command + d               # 垂直分屏
command + shift + d       # 水平分屏
command + option + 方向键  # 切换屏幕
command + r               # 快捷清屏

ctrl + k                  # 删除到末尾
{% endhighlight %}

> Vim

{% highlight Bash shell scripts %}
hjkl               # 左下上右
x      # 删除字符
A      # 行尾添加字符
d$     # 从当前光标删除到行末
e/w    # 往后移动一个单词，可与d配合使用
b      # 往前移动一个单词
0      # 移动光标到行首
U      # 撤销对整行的修改
Ctrl+r # 重做被撤销的命令

ctrl + k                  # 删除到末尾
{% endhighlight %}
