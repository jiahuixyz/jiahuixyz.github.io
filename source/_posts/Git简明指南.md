---
title: Git简明指南
date: 2019-03-23 14:39:14
tags:
---

#### 起步

设置username&email

```
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
```

查看配置

```
git config --list
```

初始化仓库、跟踪文件并提交
```
$ git init
$ git add *.c
$ git add LICENSE
$ git commit -m 'initial project version'
```

#### 记录文件变化

添加至暂存区

```
git add file
# 这是个多功能命令：可以用它开始跟踪新文件，或者把跟踪后又被修改的文件放到暂存区，还能用于合并时把有冲突的文件标记为已解决状态。 
```

 >注意！如果你git add之后修改文件，这个文件会同时出现在暂存区和非暂存区,这时git commint只提交你最后一次运行 git add 命令时的那个版本，而不是你的最新版本。
 
 查看文件状态
 
 ```
 $ git status -s
  M README
M  lib/simplegit.rb
MM Rakefile
A  lib/git.rb
?? LICENSE.txt

# M有两个可以出现的位置，
# 出现在右边的 M 表示该文件被修改了但是还没放入暂存区，
# 出现在左边的 M 表示该文件被修改了并放入了暂存区，
# 两个M表示该文件被修改并放入暂存区后又被修改了。
 ```

文件对比

```shell
git diff --cached
# 暂存后（git add后）的文件与原始版本的对比

git diff
# 未暂存（git add前）的文件与原始版本的对比

# 也就是说，git add之前用git diff，之后用git diff --cached。

# 注意，如果暂存（git add）后又修改了文件，则git diff比较未暂存的文件与暂存（git add）时的文件。
```

跳过使用暂存区域（即跳过git add）

```shell
git commit -a -m 'added new benchmarks'
# 注意：只会提交已经跟踪过的文件，不会提交新添加的文件
```

移除文件（即脱离追踪）：

```
$ git rm PROJECTS.md
```

移动或重命名

```
$ git mv README.md README
```

提交历史

```
$ git log
```

取消暂存

```
$ git reset HEAD file
```

还原文件

```
git checkout -- file
```

文件状态变化周期图：

![The picture was not found.
](https://jiahuixyz.github.io/img-service/img/git-life-cycle.png)

#### 远程仓库的使用

查看远程仓库

```shell
$ git remote -v
origin	https://github.com/schacon/ticgit (fetch)
origin	https://github.com/schacon/ticgit (push)
# orgin是远程仓库的别名，默认origin
```

详细查看远程仓库

```
$ git remote show origin
# 它会列出远程仓库的 URL 与跟踪分支的信息，
# 告诉你git pull时哪两个分支会合并，git push时会推送到哪个分支
```

添加远程仓库

```
git remote add <shortname> <url>
```

移除远程仓库

```
$ git remote rm origin
```

推送至上游

```
$ git push origin <local branch>:<remote branch>
```

检出远程分支（即创建本地分支）

```
$ git checkout -b serverfix origin/serverfix
$ git checkout --track origin/serverfix
# --track会默认创建同名分支，两个命令作用相同，都会检出分支并指定追踪分支
```

查看追踪分支（即本地分支对应的远程分支）<br>
>追踪分支在git pull和git push时起作用

```
$ git fetch --all
$ git branch -vv
# -vv并没有连接服务器，所以需要先fetch，-vv能够查看本地分支是否落后远程分支
```

修改当前分支的追踪分支

```shell
$ git branch --set-upstream-to origin/serverfix
# --set-upstream-to也可以用-u代替
```

#### git pull&git fetch

```shell
git pull origin master
# 相当于从远程获取最新版本并merge到本地
```

```shell
git fetch origin
# 不会自动创建本地分支，只有一个不可以修改的 origin/master 指针，接下来你可以选择merge或checkout
```

```shell
git fetch origin master:tmp //从远程仓库master分支获取最新，checkou到本地建立tmp分支

git diff tmp //將当前分支和tmp进行对比

git merge tmp //合并tmp分支到当前分支
```

#### 分支

新建分支

```
git branch branchname
```

删除分支

```
git branch -d branchname
```

查看所有分支&当前分支

```
git branch -a
```
