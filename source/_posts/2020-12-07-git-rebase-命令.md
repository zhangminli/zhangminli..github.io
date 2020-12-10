---
title: git rebase 命令
date: 2020-12-07 20:03:24
tags: git
---

在项目的协同开发过程中，可能会产生很多的commit记录，使用git rebase能够将分叉的分支重新合并，使项目中的提交历史变得干净整洁。

<!--more-->

假如你和A同学在同一个分支upstream/master上进行开发，当你修改了代码准备push到远端是发现失败了，原因是因为A同学已经先进行了提交，导致本地分支的提交历史已经落后于远端。需要使用git pull功能与远端同步一下，才能成功push上去。

```shell
$ git pull upstream master

$ git push upstream master
```

但此时通过git log查看提交历史，会发现分叉了，同时在你的PR记录里会出现A同学之前的commit记录。

如果不想看到分叉与过多的commit记录，可以使用`git rebase`解决。

```shell
$ git rebase upstream master
```

现在push到远端

```shell
$ git push upstream master
```

再次查看提交历史，不会出现分叉，且只会有一个commit。

在rebase的过程中，也许会出现冲突(conflict). 在这种情况，Git会停止rebase并会让你去解决冲突；rebase 和 merge的另一个区别是rebase 的冲突是一个一个解决，如果有多个冲突，先解决第一个，然后用命令

```shell
$ git add -u
$ git rebase --continue
```

继续后才会出现第二个冲突，直到所有冲突解决完，而merge 是所有的冲突都会显示出来。 
另外如果rebase过程中，你想中途退出，恢复rebase前的代码则可以用命令 

```shell
$ git rebase --abort
```

git merge 操作合并分支会让两个分支的每一次提交都按照提交时间（并不是push时间）排序，并且会将两个分支的最新一次commit点进行合并成一个新的commit，最终的分支树呈现非整条线性直线的形式

git rebase操作实际上是将当前执行rebase分支的所有基于原分支提交点之后的commit打散成一个一个的patch，并重新生成一个新的commit hash值，再次基于原分支目前最新的commit点上进行提交，并不根据两个分支上实际的每次提交的时间点排序，rebase完成后，切到基分支进行合并另一个分支时也不会生成一个新的commit点，可以保持整个分支树的完美线性

当我们协同开发一个功能时，可能会在本地有无数次commit，而你实际上在你的master分支上只想显示每一个功能测试完成后的一次完整提交记录就好了，其他的提交记录并不想将来全部保留在你的master分支上，那么rebase将会是一个好的选择，他可以在rebase时将本地多次的commit合并成一个commit，还可以修改commit的描述等。

参考链接：https://www.jianshu.com/p/6960811ac89c



