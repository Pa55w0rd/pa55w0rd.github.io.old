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