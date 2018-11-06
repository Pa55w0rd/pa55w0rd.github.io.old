---
layout: post
title:  Centos 7 运行firefox出错
date:  2018-11-06 17:03:11 +0800
img:
description: Centos 7 运行firefox出错...
categories: note
---

Centos 7 在firefox官网下载了最新的安装包，运行后提示

```
    # firefox
    XPCOMGlueLoad error for file /root/fuzzingbox/firefox/libmozgtk.so:
    libgtk-3.so.0: cannot open shared object file: No such file or directory
    Couldn't load XPCOM.
```

提示缺少libgtk-3.so库
```
yum install gtk3
```
