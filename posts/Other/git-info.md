---
title: "ffmpeg的循环音频"
description: "ffmpeg的循环音频"
date: 2021-12-15 18:10:00
slug: git-info
image:
categories:
    - Other
tags: ["Git"]
---



gitk --all：打开git的图形化界面。

git branch -av：查看本地分支。

git branch -a：查看本地和远程的所有分支。

git commit -m：用于提交暂存区的文件；git commit -am 用于提交跟踪过的文件。

分离头指针：git branch commitID，如果修改想保留修改切记要和某个分支绑在一起。

git checkout -b dev master：基于master分支创建一个dev分支并切换到dev上。

git log -n1：查看最近的一条提交记录。

git diff commitA commitB 比较2个提交记录的差异。

git diff HEAD HEAD~1 比较当前HEAD和它的父级的差异。

删除不需要的dev分支：git branch -d dev。

修改最近一次commit的message：git commit --amend。

修改老旧commit的message：（git rebase 变基）

`git rebase -i commitA`：i 交互式打开，commitA代表要改变的commit的父级commit。`reword或pick`。

rebase变基操作基于自己的分支，团队分支可能存在风险。

把连续的n个commit整理成一个：`git rebase -i commitA`，commitA指前n+1个commit。使用`squash`。

把不连续的commit整理成一个：`git rebase -i commitA` 修改里面的参数。

比较暂存区和HEAD所含文件的差异：`git diff --cached`。

工作区和暂存区的差异：`git diff`。

让暂存区恢复成HEAD一样：`git reset HEAD`。

取消暂存区部分文件的更改：`git reset HEAD -- 文件名 文件名`。

让工作区的文件恢复成和暂存区一样：`git checkout -- 文件名`

想变工作区内容用checkout，想变暂存区内容用reset。

消除最近的几次提交：`git reset --hard commitA`，commitA要回到的提交，暂存区和工作区的内容会回到commitA一样的状态，要慎用。

比较2个分支的差异：`git diff dev master`或者 `git diff dev master -- 文件名`。

比较2个分支的差异：`git diff commitA commitB `或者 `git diff commitA commitB -- 文件名`。

正确删除文件：`git rm 文件名`。

stash使用：

- `git stash` 将修改存放起来。
- `git stash list`` 查看存放列表。
- `git stash apply stash{0}` 恢复存放的文件。
- `git stash pop stash{0}` 恢复存放的文件，并删除存放列表中的记录。

指定不需要Git管理的文件：`.gitignore`文件中添加匹配，例如`doc`和`doc/`的区别。

每个语言的`gitignore`文件的编写规范可以参考github仓库。

### git的备份

1、备份到远端：

- 列出详细信息，显示对应的克隆地址：`git remote -v `。
- 添加新的远程仓库，并指定一个简单的名字：`git remote add [shortname] [url] `。

2、拉取到本地

- `git clone 仓库地址`

git push -f origin master

不同人修改了同文件的不同区域：

`git fetch`

`git branch -av`

`git merge origin/feature/add_git_commands`

`git push`

不同人修改了同文件的同一区域：





















