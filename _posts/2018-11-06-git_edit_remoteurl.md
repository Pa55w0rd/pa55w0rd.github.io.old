---
layout: post
title:  Git修改远程URL
date:  2018-11-06 17:04:11 +0800
img:
description: Github 设置使用了ssh key，每次提交代码还是需要输入用户名？？？
categories: note
---

Github 设置使用了ssh key，每次提交代码还是需要输入用户名？？？

检查发现，在设置远程url的时候使用了https

https://github.com/Pa55w0rd/pa55w0rd.github.io.git

使用git remote set-url命令从https切换到ssh
```
# git remote set-url origin git+ssh://git@github.com/Pa55w0rd/pa55w0rd.github.io.git
```
通过git remote -v命令验证是否成功
```
# git remote -v
origin  git+ssh://git@github.com/Pa55w0rd/pa55w0rd.github.io.git (fetch)
origin  git+ssh://git@github.com/Pa55w0rd/pa55w0rd.github.io.git (push)
```


## 基本使用

### 建立远程仓库

进入GitHub，点击右上角 + 号，选择New repository

输入仓库名称，点击 Create repository

### 建立本地仓库

建立目录，将文件夹初始化为git可以管理的仓库

git init

### 与远程仓库关联

git remote add origin git@github.com:Pa55w0rd/pa55w0rd.github.io.git

### 添加到暂存区

git add .   
> 注意最后边的 .

### 提交暂存区到仓库区

git commit -m "init"

### 上传本地仓库到远程仓库

git push origin master

### 显示变更的文件

git status

### 克隆远程仓库到本地

git clone git@github.com:Pa55w0rd/pa55w0rd.github.io.git

### 高级用法

参考 [阮一峰_ 常用Git命令清单](http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html)