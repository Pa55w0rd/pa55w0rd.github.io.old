---
layout: post
title:  "win10安装linux子系统"
date:   2018-09-25 16:15:07 +0800
img:
description: win10系统一周年正式版(1607)之后开始支持内置linux子系统，记录一些关键步骤和遇到的问题
category: 小记
---
win10系统一周年正式版(1607)之后开始支持内置linux子系统，记录一些关键步骤和遇到的问题

## 启用linux子系统

![启用1](/assets/img/use_linux1.png)

![启用2](/assets/img/use_linux2.png)

## 安装linux子系统

打开win10应用商店，搜索linux，选择ubuntu安装

![安装](/assets/img/install_linux.png)

## win10安装linux子系统报错0x8007019e解决办法

**原因：** 未安装Windows子系统支持

1. win+x，选择Windows PowerShell（管理员）
2. 输入 ```Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux```
3. 回车，输入Y，重启系统后重新打开安装的对应子系统即可

## 开始使用

### 换源

```
# 1.备份原来的数据源配置文件
cp /etc/apt/sources.list /etc/apt/sources.list_backup
# 2.编辑数据源配置文件
vi /etc/apt/sources.list
```

使用[中科大的源]("http://mirrors.ustc.edu.cn/help/ubuntu.html#id1")

```
# lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 18.04.1 LTS
Release:        18.04
Codename:       bionic
```
```
deb https://mirrors.ustc.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic main restricted universe multiverse

deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-security main restricted universe multiverse

deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse

deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
```

### 读取宿主机文件

```
# df -h
Filesystem      Size  Used Avail Use% Mounted on
rootfs          101G   62G   39G  62% /
none            101G   62G   39G  62% /dev
none            101G   62G   39G  62% /run
none            101G   62G   39G  62% /run/lock
none            101G   62G   39G  62% /run/shm
none            101G   62G   39G  62% /run/user
C:              101G   62G   39G  62% /mnt/c
D:              279G   54G  225G  20% /mnt/d
E:              278G   38G  240G  14% /mnt/e
F:              277G  191G   87G  69% /mnt/f
```
