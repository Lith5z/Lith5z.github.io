---
layout: default
title: "Git常用命令"
date: 2026-06-16
tags: [教程]
parent: "使用笔记"
---

不过，我一般还是用vscode的GUI版本git

## 准备
git init
初始化仓库
首先确保命令行已经切换到目标路径

git branch -M main
修改当前分支名称

设置远程仓库
git remote add <这里写想设置的仓库的名字> <这里写地址>
例如 git remote add origin https://github.com/njuer/NJU-CPL.git

初始化 .gitignore 文件

## 基础命令
本地工作区 -> 暂存区 -> 本地仓库 -> 云端仓库

git add -> git commit -> git push

### git add
将文件的更改（新增、修改或删除）从工作区添加到暂存区，为随后的提交做准备

git add .
添加所有文件

git add .gitignore
添加单个文件

git restore --staged README.md
从暂存区中移除文件

### git commit
git commit -m "Commit Message"
将暂存区的文件提交到本地仓库
如果不写 -m 参数，则打开默认编辑器来填写message

--amend 参数 用于修改最后一次的提交

git commit --amend
把当前暂存区的文件覆盖到上一次的提交，而不创建新的提交。例如，如果你忘记将某个文件暂存，你可以先 git add 那个文件，再用这个命令提交

git commit --amend -m "Commit Message"
修改上一次的提交记录

### git push
git push -u <仓库名> <分支名>
例如 git push -u origin main

初次使用之后 可以用git push上传到默认仓库

云端仓库 -> 本地仓库

### git clone
git clone <目标仓库地址> <可选：指定克隆到的目标目录>
例如 git clone https://github.com/njuer/NJU-CPL.git
上面这个命令会克隆完整的仓库副本

git clone -b preview https://github.com/njuer/NJU-CPL.git
克隆指定分支

git clone --depth=1 https://github.com/njuer/NJU-CPL.git
克隆最近的提交记录（浅克隆）

### git pull/git fetch
从远程仓库获取最新的更改，并将这些更改合并到本地仓库的命令

它是 git fetch 和 git merge 的组合操作。git fetch 拉取远程仓库的更改到本地，但并不自动合并；git merge 是将远程分支的更改合并到当前分支

## 其他命令
git log
查看提交历史

git restore
用于恢复工作区中的文件状态，可以撤销某些操作或恢复文件的某个特定版本

git restore linked-list/joseph.c
撤销未暂存的更改：如果对文件做了一些修改，但不想保留这些更改

git restore --staged linked-list/joseph.c
撤销已暂存的更改

## 工作流解析

// TODO