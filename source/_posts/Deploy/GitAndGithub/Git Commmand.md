---
title: Git Command
date: 2022-12-23 12:23:23
tags:
  - Git
categories:
  - Git
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

## git 分区

git 一共有三个分区：

工作区、暂存区和版本库

## git 的初始化（自报家门）

1. git config --global user.name "YOUR NAME"
2. git config --global user.email "email@example.com"

## 用 git 创建仓库(repository)

1. mkdir repositoryExample（这只是生成了一个文件夹）

### 进入仓库~~（文件夹）~~

1. cd repositoryExample

### 初始化仓库（仓库的正式建立）

1. git init

#### 注意：此举会在当前目录下生成一个.git 文件夹，但是这是一个以.开头的隐藏文件夹，所以使用 ls 命令无法直接看到，你可以使用 ls -a 命令来查看这类隐藏的文件夹

## 用 git 来记录变化

1. git add fileName

   \*可以使用 git status -s 来查看更改\*\*\*

   > -A, --all add changes from all tracked and untracked files
   >
   > -u, --update update tracked files

2. git diff

   _使用 diff 来查看已经做出的变化_

   **用 git diff HEAD -- filename 来比较远程仓库和本地的文件的区别**

3. git commit -m "本次提交的说明"

   >

\*\*或者使用 git commit -a 命令来跳过 add 直接提交所有变动

## 关于历史提交

1. 使用 git log 或者 git log --pretty=oneline 来查看
2. 使用 git reset + log 中的每行前的如 d5d7e...版本号 来还原
3. git 中，有 git reflog 命令来记录用户的每一次操作
4. git 管理的是修改而不是文件！

## 关于文件更改

1. git checkout -- filename //舍弃工作区的所有更改

   关于这个命令

    如果 filename 文件还没有被提交到暂存区（仍然停留在工作区），则 git 会立即撤销所有更改——即使已经在文件中保存

    如果 filename 文件已经用 add 命令提交过一次了，则 git 会将它重置为刚刚提交到缓存区中的状态

2. git reset HEAD filename

   通过这个命令，我们可以将已经提交到暂存区的 filename 文件回退到工作区

## 文件删除

1. git rm filename

   注意: 删除文件也是一个修改，必须要提交才会被记录，但这里没有必要 add，可以直接 commit

2. git checkout -- filename

   又见面了，再次用此命令来恢复文件，因为它的作用**其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”。**

[git rm - How do I delete a file from a Git repository? - Stack Overflow](https://stackoverflow.com/questions/2047465/how-do-i-delete-a-file-from-a-git-repository)

## 远程管理

1. 显示所有远程仓库：git remote -v
2. 添加远程版本库：git remote add [name] [url]
3. 删除远程仓库：git remote remove origin
4. 修改仓库名：git remove rename old_name new_name

### 向远程库提交

1. git push -u [推送的主机] [推送的分支] ——例：origin main
2. 删除主机的分支：git push origin --delete master

### 从远程库克隆

1. git clone git@项目仓库地址

2. 从远程获取代码并合并本地版本：

   规范：git pull <远程主机名> <远程分支名>:<本地分支名>

   示例：git pull origin master:brantest

    ——将远程主机 origin 的 master 分支拉取过来，与本地的 brantest 分支合并

## 分支

1. 创建并切换到新分支：git checkout -b dev 或 git switch -c dev
2. 切换分支：git checkout dev 或 git switch master
3. 查看分支：git branch
4. 把分支合并到 main 上：git merge dev
5. 删除分支：git branch -d dev
6. 将分支推送到远程仓库：git push origin bunjie（同理呀！）

git push --set-upstream origin master

ssh-keygen -t rsa

## Set Proxy

[一文让你了解如何为 Git 设置代理 - Eric (ericclose.github.io)](https://ericclose.github.io/git-proxy-config.html)
