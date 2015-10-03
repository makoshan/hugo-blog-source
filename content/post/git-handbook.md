---
categories:
- Dev
date: 2015-10-03T23:10:38+08:00
tags:
- git
title: git 常用命令
---
### 1.分支操作
- 创建本地分支 `git checkout -b your_branch_name`
- 删除本地分支 `git branch -d your_branch_name`
- 创建远程分支 
	1. 先创建本地分支 `git checkout -b your_branch_name`
	2. 推送本地分支到远端 `git push -u <remote-name> your_branch_name`
- 删除远程分支 `git push origin :the_remote_branch`

### 2.Tag操作
- 创建本地Tag `git tag tag_name [commit_id]` ([]代表此参数可省略，默认为HEAD)
- 创建远程Tag
	1. 创建本地Tag `git tag tag_name [commit_id]`
	2. 推送到远端 `git push origin tag_name`
- 基于某个Tag，拉出新分支 `git checkout -b your_branch_name tag_name`
- 删除本地Tag `git tag -d your_tag_name`
- 删除远程Tag `git push origin :remote_tag_name`

### 3.git 配置
1. `git config -e` 打开当前.git目录下的.gitconfig配置文件
2. `git config -e --global` 打开~/.gitconfig配置文件
3. ` git config -e --system` 打开/etc/.gitconfig配置文件

*未完待续...*

