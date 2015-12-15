---
layout: post
title:  "Git Command"
date:   2015-11-17 16:30:00
tags: [memory, analysis, tool]
categories: Resources
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
# 如果遇到冲突，可以直接编辑冲突文件，手动处理冲突的内容；或者用git checkout --ours/theirs filename
$ git branch -d branch_name  # 删除branch
$ git pull -p  # 等同于git fetch --prune origin；git fetch -p，删除远端已经删除的分支
{% endhighlight %}

### 3. Repo
{% highlight Bash shell scripts %}
$ git remote -v  # 查看remote的repo，叫origin
$ git fetch  # 从远程拉取东西，但是不自动merge
$ git merge origin/master  # 之后执行这个，merge过来本地的master  这2条相当于pull
$ git push origin local-branch
$ git push origin test:test  #  本地的test分支，push到origin的test分支
$ git mv string.c src/  # 重构代码目录；或者重命名
{% endhighlight %}
实例：拉取他人repo，merge到自己的branch
{% highlight Bash shell scripts %}
$ git remote add local-name git-repo  # 增加一个远程仓库(-t branch-name:可以只跟踪某个branch)
$ git fetch local-name  # 拉取东西，但是不自动merge
$ git checkout local-name/master  # 切换到他人分支
# checkout到别人分支后，处于detached HEAD状态，这时候所作的commit都会被丢弃。要在别人代码的基础上进行修改，可以新建一个本地分支
$ git checkout -b merge-branch
# 之后可以在此分支上进行一些修改
$ git checkout master  # 切换到自己的master分支
$ git merge merge-branch  # merge, 如果有冲突需要先解决
# 之后add, commit, push

# 以上步骤的问题是: merge时没有使用--squash参数，保留了merge-branch的所有commit历史，修改：
$ git reset aa020070  # 恢复到merge之前的commit
$ git add -A  # 重新add所有的修改
$ git commit -m "..."
$ git push -f  # 由于落后于remote，需要加-f: 利用强覆盖方式用本地 替代 git仓库的内容
{% endhighlight %}