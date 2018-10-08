---
layout: post
title:  "ubuntu搭建xunfeng资产扫描系统"
date:   2018-10-08 11:29:31 +0800
img:
description: 巡风是一款适用于企业内网的漏洞快速应急、巡航扫描系统，通过搜索功能可清晰的了解内部网络资产分布情况，并且可指定漏洞插件对搜索结果进行快速漏洞检测并输出结果报表
categories: 企业安全
---

* 目录
{:toc}

## 0x00 前言

巡风是一款适用于企业内网的```漏洞快速应急、巡航扫描```系统，通过搜索功能可清晰的了解内部网络资产分布情况，并且可指定漏洞插件对搜索结果进行快速漏洞检测并输出结果报表。

网络资产识别引擎会通过用户配置的IP范围定期自动的进行端口探测（支持调用MASSCAN），并进行指纹识别，识别内容包括：服务类型、组件容器、脚本语言、CMS。

漏洞检测引擎会根据用户指定的任务规则进行定期或者一次性的漏洞检测，其支持2种插件类型、标示符与脚本，均可通过web控制台进行添加。

巡风需要在```python2.7```和```mongodb3.2```版本以上环境上安装 

```git clone https://github.com/ysrc/xunfeng```


## 0x01 环境配置

### 换源

安装好Ubuntu，首先换源

部署和调试巡风要求root权限，请用户切换到root账号进行操作,Ubuntu默认未开启root，请使用下列命令开启

```$ sudo passwd root```

输入你自己设置的root密码后

```$ su root```

即可切换到root账号



通过lsb_release 查看开发版本代号为18.04 bionic
```
# lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 18.04.1 LTS
Release:        18.04
Codename:       bionic
```

备份原来的数据源配置文件

```# cp /etc/apt/sources.list /etc/apt/sources.list_backup```

编辑数据源配置文件

```# vi /etc/apt/sources.list```


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

保存后执行
```
apt-get update && apt-get upgrade && apt-get dist-upgrade
```

### 修改时区为Asia/Shanghai

```
# echo TZ\='Asia/Shanghai'\; export TZ >> ~/.bash\_profile && source ~/.bash\_profile
```

### 安装操作系统依赖

```
# apt-get install gcc libssl-dev libffi-dev python-dev libpcap-dev
```

### 安装python依赖库

使用pip进行python依赖库管理，如果系统没有安装pip，可以通过如下命令安装

```
# wget https://sec.ly.com/mirror/get-pip.py --no-check-certificate && python get-pip.py
```
使用pip安装 python 依赖库, 这里使用了豆瓣的 pypi 源
```
# pip install -r requirements.txt -i https://pypi.doubanio.com/simple/
```
### 安装mongo数据库

由于低版本不支持全文索引，MongoDB版本需要 ≥ 3.2。

[参考官方安装步骤](https://docs.mongodb.com/v3.2/tutorial/install-mongodb-on-ubuntu/)

1. 安装公钥

```# apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927```

2. 添加3.2的源

```# echo "deb http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list```

3. 安装数据库

```# apt-get update && apt-get install -y mongodb-org```

4. 启动mongodb服务


默认情况下apt方式安装的mongodb会自动启动，查看是否启动成功

```# netstat -antlp | grep 27017```

如果没有启动 

```# systemctl start mongodb```

设置开机启动

```# systemctl enable mongodb```


5. mongodb添加认证
```
# mongo
> use xunfeng
> db.createUser({user:'scan',pwd:'your password',roles:[{role:'dbOwner',db:'xunfeng'}]})
> exit
```
这里的 scan和your password 需要更换为你的mongodb的账户和密码。

## 0x02 部署

### 导入数据库

进入 ```xunfeng/db``` 文件夹, 执行如下命令:

```# mongorestore -h 127.0.0.1 --port 27017 -d xunfeng .```

### 修改配置

修改系统数据库配置脚本 Config.py:

```
class Config(object):
    ACCOUNT = 'admin'
    PASSWORD = 'xunfeng321'
```


修改 PASSWORD 字段内的密码, 设置成你的密码。

```
class ProductionConfig(Config):
    DB = '127.0.0.1'
    PORT = 27017
    DBUSERNAME = 'scan'
    DBPASSWORD = 'your password'
    DBNAME = 'xunfeng'
```

### 运行系统

根据实际情况修改(端口和目录需对应好) Conifg.py 和 Run.sh 文件后, 执行:

```# sh Run.sh```

如开启防火墙，需打开80端口



