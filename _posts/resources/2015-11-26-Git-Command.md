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

![git-HEAD](/res/git-head.png)

图中HEAD~指定HEAD之前的第几次提交记录。HEAD^指定使用哪个父节点

####  b. Git Stash
还未提交的修改留在索引区或工作树的情况下：
切换分支时修改内容会*随身带到目标分支*，可以add并commit；

但如果在checkout的目标分支中相同的文件也有后来的修改(即hash值不同)，checkout会失败。这时要么先提交commit修改内容，要么用stash暂时保存修改内容后再checkout。

stash是临时保存文件修改内容的区域。可以暂时保存**工作树和索引**里还没提交的修改内容，可以事后再取出暂存的修改，应用到原先的分支或其他的分支上。

####  c. Git commit \-\-amend
git add添加新内容后，执行commit \-\-amend，会修改上次的commit合并为1个。

使用场合：

* 添加最近那次commit时，漏掉add的内容
* 修改最近那次commit的comments(也就是commit后立马执行amend)

####  d. Cherry-pick
从其他分支复制*指定*的commit，merge进来：

![git-Cherry-pick](/res/git-cherry-pick.png)
{% highlight Bash shell scripts %}
$ git cherry-pick c81dba1  # merge其他分支的某个commit的hash值
# 如果有冲突，解决后add，再commit
{% endhighlight %}
该命令谨慎使用，因为其他分支的某个commit，其实是depends on它之前的commit，实际pick的时候，会把之前的commit也带过来。
所以不如直接用git merge来的清晰，只是pick的commit message会带过来，merge需要自己写

####  e. Merge
Merge会生成一个新提交，master分支的HEAD会移动到该提交上

![git-Merge](/res/git-merge.png)

另，一个有用的命令，把另一个branch的某个file，checkout到本分支上来:

```
git checkout other-branch-name -- want-file-name
```

####  f. Rebase
rebase bugfix分支到master分支, bugfix分支的历史记录会添加在master分支的后面。

![git-Rebase](/res/git-rebase.png)

如图，历史记录成一条线很整洁。这时移动提交X和Y有可能会发生冲突，需要修改各自的提交时发生冲突的部分。另：

```
git rebase HEAD^ --onto master 自己的上一个commit不要了，rebase master上的(onto指定upstream)

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

注意：在解决conflict时，如果要完全舍弃或保留某一个版本，则可以用--ours/theirs命令:
{% highlight Bash shell scripts %}
# in A branch, and excute: git merge B
git checkout --ours filename # Keep A file
git checkout --theirs filename # Keep B file
# in A branch, and excute: git rebase B
# rebase 其实相当于切到B分支，再把AB共同祖先之后的，A的所有的log一条一条merge到B的最后
git checkout --ours filename # Keep B file
git checkout --theirs filename # Keep A file
{% endhighlight %}
merge和rebase对于ours/theirs命令正好相反

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
另外可用一条命令实现：
{% highlight Bash shell scripts %}
$ git checkout -b want-branch --single_branch <repo-url>
{% endhighlight %}

####  i. Git Init
初始化git目录
{% highlight Bash shell scripts %}
$ mkdir <repo>
$ cd <repo>
$ git init  # this will initialize the git repo in repo/.git/
$ git --git-dir=lib init  # initialize in lib/ directly, lib/objects/ 存放具体数据
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
# Note: git checkout -B branch-name, 大写B谨慎使用
# 例如：基于A新建B分支，回到A分支添加commitA，再切回B分支添加commitB，此时如果checkout -B A
# 则A分支变成commitA+commitB的形式，也就是文件最终的结果和B分支的文件相同，A分支的commitA修改被抹掉了。想恢复A，可以通过reset，删掉commitB

$ git merge branch_name  # 从别的branch merge改动到当前branch（若加--squash，代表别的分支的所有改动合并成1个commit）
# 如果遇到冲突，可以直接编辑冲突文件，手动处理冲突的内容；或者用git checkout --ours/theirs filename
$ git branch -d branch_name  # 删除branch
$ git pull -p  # 等同于git fetch --prune origin；git fetch -p，删除本地的，远端已经删除的分支
$ git push origin :branch_name  # 删除远端分支，无论本地是否拉下来了这个分支
$ git checkout --orphan branch_name  # 新建一个完全空的，干净的分支，不带任何commit过去
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

![git-Revert](/res/git-revert.png)
{% highlight Bash shell scripts %}
$ git revert HEAD  # 取消上次的commit
$ git revert HEAD^  # 也可以直接取消上上次的commit，但是此时肯定有confict，需要解决后再commit
{% endhighlight %}

**ORIG_HEAD** 指向之前的HEAD。Reset或Revert错误的时候，在ORIG_HEAD上reset就可以还原之前状态：
{% highlight Bash shell scripts %}
$ git reset --hard ORIG_HEAD  # 强制回到上次的commit
# 另外，git reflog可以记录之前所有操作的hash，包括reset，可以找回reset --hard的commit
$ git reflog
$ git reset --hard 98abc5a  # 从上一步列出的hash中选出需要的
{% endhighlight %}

### 5. Git Submodule
{% highlight Bash shell scripts %}
# 如何 add
$ git submodule add lib-git-repo-link libs/lib-name  # 添加submodule, 生成.gitmodules记录引用信息
$ cd libs/lib-name  && git checkout branch-need  # 切换submodule到需要的分支
$ git add libs/lib-name  # 提交submodule信息到repo

# 如何 clone，正常clone后，.git/config中没有注册submodule的索引信息
$ git submodule  # 查看submodule情况，前边的"-"，表示还没有检出
$ git submodule init  # 此时，再查看.git/config中，已经注册了submodule的url
$ git submodule update  # 真正检出了submodule
# 以上2条命令，可以在clone时，加--recursive参数，一条命令替代!!!

# 如何修改lib
$ cd libs/lib-name  # 此时头指针是一个commit id
$ git checkout master # 想修改，切换到master，做一些修改
# Note: 如果忘记切换就修改，则无法push，切换到master后，用git merge/cherry-pick change-id即可
$ pushd ../../;git diff; popd  # 会显示更新的lib1
--- a/libs/lib1
+++ b/libs/lib1
@@ -1 +1 @@
-Subproject commit c22aff85be91eca442734dcb07115ffe526b13a1
+Subproject commit 36ad12d40d8a41a4a88a64add27bd57cf56c9de2
$ git push  # 将改动提交到lib1原始repo
# 最后，要到主repo, add libs/lib-name，git commit, push将新的commit id提交

# 别的project的人，如何拿到最新的commit id和lib
$ git pull  # (可以加--recurse-submodules) 拉取最新的project id
$ git diff  # 现在居然有问题: libs/lib-name不是最新
$ git submodule update  # (--recursive) 现在检出更新到最新

# 如果其他的project-b，如果用到了这个lib，那么需要手动进去checkout master,并pull最新的lib
# 如果很多的project用到了该lib，则可以写个脚本，依次进去git pull
$ git submodule foreach "git checkout master && git pull"  # 该命令取代上述脚本

# 如何删除submodule
$ git rm -r --cached libs/lib2  # 从git删除，--cached会保留实际的文件
$ rm -rf libs/lib2  # 实际删除物理文件
# 之后编辑.gitmodules(前边这几步，可以用git rm代替)和.git/config，删除对应的条目，最后git add，push
# 如果提示A git directory for '' is found locally with remote，仍旧无法添加，则.git/config中删除相应条目，rm -rf .git/modules/module2delete删除相应条目

# 切换分支后，如果submodule修改过，不会及时更新，需要手动update子module的修改：
$ git submodule update --init (-f)
{% endhighlight %}

### 6. Git LFS
[用法参考](https://github.com/git-lfs/git-lfs/wiki/Tutorial)
git lfs专为repo中的大文件设计，有单独的存储大文件的地方，本repo只存储地址信息
新repo：
{% highlight Bash shell scripts %}
git lfs install                   # 在需要该特性的git中，执行
git lfs track '*.bin' ‘*.a’       # 跟踪需要放到lfs大存储的文件
git lfs track                     # 查看已经跟踪的文件
git add .gitattributes *.bin *.a  # add修改的文件
git commit -m "add lfs files"
git lfs ls-files                  # 此时可以查看已经track好的文件，都有了hash id
{% endhighlight %}
迁移已有repo到lfs
{% highlight Bash shell scripts %}
git lfs migrate import --include="*.a,*.lib,*.pdb,*.dll" --include-ref=refs/heads/master
git lfs track
git lfs ls-files  # 确认下，添加的对不对!!!
git push -f       # 上一步重写了history，所以必须-f来push
git reflog expire --expire-unreachable=now --all
git gc --prune=now  # 这2步清空.git中的cache
{% endhighlight %}


![git-cheat-sheet](/res/git-cheat-sheet.png)
