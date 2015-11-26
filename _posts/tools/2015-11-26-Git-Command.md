---
layout: post
title:  "Application Memory Analysis"
date:   2015-11-17 16:30:00
tags: [memory, analysis, tool]
categories: tools
---

### 1. Back Track
{% highlight Bash shell scripts %}
$ git show HEAD  # 查看最近一次commit的详细内容
$ git checkout HEAD filename  # 某个文件从上次commit后又修改了，但是想丢弃这些修改，恢复到commit时的结果
$ git reset HEAD scene-2.txt  # 将该文件从add后的stage区域取消，unstage
$ git reset 7be7ec672af  # 恢复到某一次commit，填大于7个字符的hash值即可，之后所有的改动变成unstaged
{% endhighlight %}

### 2. Branch
{% highlight Bash shell scripts %}
$ git branch branch-name  # 建新branch，之后可以checkout到新branch
$ git checkout -b branch-name  # 直接建立新branch，并自动checkout到新branch
$ git merge branch_name  # 从别的branch merge改动到当前branch（若加--squash，代表别的分支的所有改动合并成1个commit）
# 如果遇到冲突，可以直接编辑冲突文件，手动处理冲突的内容；或者用
$ git branch -d branch_name  # 删除branch
{% endhighlight %}

### 3. Repo
{% highlight Bash shell scripts %}
$ git remote -v  # 查看remote的repo，叫origin
$ git fetch  # 从远程拉取东西，但是不自动merge
$ git merge origin/master  # 之后执行这个，merge过来本地的master  这2条相当于pull
$ git push origin local-branch
$ git push origin test:test  #  本地的test分支，push到origin的test分支
{% endhighlight %}