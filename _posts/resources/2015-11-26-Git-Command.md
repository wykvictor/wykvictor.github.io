---
layout: post
title:  "Git Command"
date:   2015-11-17 16:30:00
tags: [memory, analysis, tool]
categories: Resources
---

### 0. Basic
部分内容引用自一个非常好的git教程[backlog](http://backlogtool.com/git-guide/cn/intro/intro1_1.html)

####  a. HEAD索引
HEAD指向当前分支的最后一次commit。通过移动HEAD，就可以变更使用的分支。
![git-HEAD](http://7xno5y.com1.z0.glb.clouddn.com/git-head.png)
图中HEAD~指定HEAD之前的第几次提交记录。HEAD^指定使用哪个父节点

####  b. Git Stash
还未提交的修改留在索引区或工作树的情况下：切换分支时修改内容会*随身带到目标分支*，可以add并commit；
但如果在checkout的目标分支中相同的文件也有后来的修改(即hash值不同)，checkout会失败。这时要么先提交commit修改内容，要么用stash暂时保存修改内容后再checkout。
stash是临时保存文件修改内容的区域。可以暂时保存**工作树和索引**里还没提交的修改内容，可以事后再取出暂存的修改，应用到原先的分支或其他的分支上。

####  c. Git commit \-\-amend
git add添加新内容后，执行commit \-\-amend，会修改上次的commit合并为1个。
使用场合：
1.  添加最近那次commit时，漏掉add的内容
2.  修改最近那次commit的comments(也就是commit后立马执行amend)

####  d. Cherry-pick
从其他分支复制*指定*的commit，merge进来：
![git-Cherry-pick](http://7xno5y.com1.z0.glb.clouddn.com/git-cherry-pick.png)
{% highlight Bash shell scripts %}
$ git cherry-pick c81dba1  # merge其他分支的某个commit的hash值
# 如果有冲突，解决后add，再commit
{% endhighlight %}

####  e. Merge
Merge会生成一个新提交，master分支的HEAD会移动到该提交上
![git-Merge](http://7xno5y.com1.z0.glb.clouddn.com/git-merge.png)

另，一个有用的命令，把另一个branch的某个file，checkout到本分支上来:
```
git checkout other-branch-name -- want-file-name
```

####  f. Rebase
rebase bugfix分支到master分支, bugfix分支的历史记录会添加在master分支的后面。
![git-Rebase](http://7xno5y.com1.z0.glb.clouddn.com/git-rebase.png)
如图，历史记录成一条线很整洁。这时移动提交X和Y有可能会发生冲突，需要修改各自的提交时发生冲突的部分。另：
```
git rebase -i hash 可以汇合几个commit，或者改写某个commit。
```
用的不多，具体参照 [教程](http://backlogtool.com/git-guide/cn/stepup/stepup7_5.html)和[汇总](http://backlogtool.com/git-guide/cn/reference/log.html)。

实例：
{% highlight Bash shell scripts %}
# master分出2个branch，并行各自开发，先merge-1(fast-forward)，再merge-2(解决冲突)
$ git log --graph --oneline  # 之后的状态如下，log较为混乱
*   5ac2fc4 merge     # 这是merge的提交
|\
| * c7cbf1e branch 2  # 这是branch2的修改
* | 5f9a478 branch 1  # 这是branch1的修改，也是fast-forward
|/
* 489bc6e master      # 这是最初的提交
# 下面使用rebase
$ git reset --hard HEAD^  # 暂时取消刚才的合并
$ git checkout branch-2  # 切换到branch-2分支后，对master执行rebase
$ git rebase master  # 同样之后，手工解决conflict
$ git add test.txt  # 添加后，不能用commit
$ git rebase --continue  # 而是执行rebase，指定--continue；若要取消，指定--abort
$ git checkout master  # master分支此时可以 fast-forward合并branch-2了
$ git merge branch-2  # fast-forward合并
$ git log --graph --oneline  # 之后的状态如下，工作区test.txt结果相同，但log灰常清晰
* edec60f branch 2    # 这是branch2的修改，也是fast-forward
* 5f9a478 branch 1    # 这是branch1的修改，是fast-forward
* 489bc6e master      # 这是最初的提交
{% endhighlight %}

####  g. Tag
标签是为了更方便地参考提交而给它标上易懂的名称。
Git可以使用2种标签：轻标签（本地暂时使用）和注解标签（需添加注解或签名，发布用）
{% highlight Bash shell scripts %}
$ git tag tag-1  # 添加轻标签
$ git tag  # 显示已有标签列表
$ git log --decorate --oneline  # decorate选项，可以显示包含标签资料的log
edec60f (HEAD -> master, tag: tag-1, branch-2) branch 2
5f9a478 (branch-1) branch 1
489bc6e master
$ git tag -am "comment" tag-2  # -a为注解tag，需要-m添加comment
$ git tag -n  # -n除了列表，也显示出comments
tag-1           branch 2
tag-2           comment
$ git tag -d tag-1  # 删除tag
Deleted tag 'tag-1' (was edec60f)
{% endhighlight %}

####  h. Sparse checkout
若git repo太大，可以用该方法只clone子目录：
{% highlight Bash shell scripts %}
$ mkdir <repo>
$ cd <repo>
$ git init
$ git remote add -f origin <url>
# This creates an empty repository with your remote, and fetches all objects but doesn't check them out.
$ git config core.sparseCheckout true
$ echo "some/dir/" >> .git/info/sparse-checkout
$ git pull origin master
{% endhighlight %}


### 1. Back Track
{% highlight Bash shell scripts %}
$ git log --graph --oneline  # graph:以文本形式显示更新流程;oneline:在一行中显示提交的信息
$ git show HEAD  # 查看最近一次commit的详细内容
$ git checkout HEAD filename  # 某个文件从上次commit后又修改了，但是想丢弃这些修改，恢复到commit时的结果
$ git reset HEAD filename  # 将该文件从add后的stage区域取消，unstage
$ git reset 7be7ec672af  # 恢复到某一次commit，填大于7个字符的hash值即可，之后所有的改动变成unstaged
$ git reset HEAD^  # 取消这次的commit信息，HEAD恢复到上一次commit后，本次修改需重新add
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
$ git push -f  # 由于落后于remote，需要加 -f: 利用强覆盖方式用本地 替代 git仓库的内容
{% endhighlight %}

### 4. Revert & Reset
**reset** 可以恢复commit，除了默认的mixed模式，还有soft和hard模式：
<table border="2" frame="box" cellspacing="0px" style="border-collapse:collapse" valign="center">
	<tr bgcolor="lightgreen">
	    <th align="center">　模式名称　</th>
	    <th align="center">　HEAD位置　</th>
	    <th align="center">　索引区　</th>
	    <th align="center">　工作树　</th>
	    <th align="center">　使用场合　</th>
	</tr>
	<tr>
	    <td align="center">soft</td>
	    <td align="center">修改</td>
	    <td align="center">不修改</td>
	    <td align="center">不修改</td>
	    <td align="center">只取消commit</td>
	</tr>
	<tr>
	    <td align="center">mixed</td>
	    <td align="center">修改</td>
	    <td align="center">修改</td>
	    <td align="center">不修改</td>
	    <td align="center">取消commit和add</td>
	</tr>
	<tr>
	    <td align="center">hard</td>
	    <td align="center">修改</td>
	    <td align="center">修改</td>
	    <td align="center">修改</td>
	    <td align="center">　彻底复原上一次commit　</td>
	</tr>
</table>
**revert** 可以安全地取消指定的commit，但跟reset不同，不会回退，只会生成新的commit：

![git-Revert](http://7xno5y.com1.z0.glb.clouddn.com/git-revert.png)
{% highlight Bash shell scripts %}
$ git revert HEAD  # 取消上次的commit
$ git revert HEAD^  # 也可以直接取消上上次的commit，但是此时肯定有confict，需要解决后再commit
{% endhighlight %}

**ORIG_HEAD** 指向之前的HEAD。Reset或Revert错误的时候，在ORIG_HEAD上reset就可以还原之前状态：
{% highlight Bash shell scripts %}
$ git reset ORIG_HEAD
{% endhighlight %}