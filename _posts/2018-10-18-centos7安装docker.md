---
layout: post
title:  centos7 安装docker
date:   2018-10-18 10:40:11 +0800
img:
description: Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口。
categories: note
---

* 目录
{:toc}

## 0x00 前言
Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口。

Docker从1.13版本之后采用时间线的方式作为版本号，分为社区版CE和企业版EE。

社区版是免费提供给个人开发者和小型团体使用的，企业版会提供额外的收费服务，比如经过官方测试认证过的基础设施、容器、插件等。

社区版按照stable和edge两种方式发布，每个季度更新stable版本，如17.06，17.09；每个月份更新edge版本，如17.09，17.10。

Docker里比较重要的概念有**注册服务器、仓库、镜像、容器**。

**仓库**： 注册服务器是存放仓库的地方，其上往往存放着多个仓库。每个仓库集中存放某一类镜像，往往包括多个镜像文件，通过不同的标签（tag）来进行区分。例如存放Ubuntu操作系统镜像的仓库，称为Ubuntu仓库，其中可能包括14.04、12.04等不同版本的镜像。

**镜像**： Docker镜像（Image）类似于虚拟机镜像，可以将它理解为一个面向Docker引擎的只读模板，包含了文件系统。例如：一个镜像可以只包含一个完整的Ubuntu操作系统环境，可以把它称为一个Ubuntu镜像。

**容器**： 容器是从镜像创建的应用运行实例，可以将其启动、开始、停止、删除，而这些容器都是相互隔离、互不可见的。可以从一个镜像创建无数个容器。平时我们主要操作的就是容器。我们也可以把容器打包成镜像以方便再次使用。镜像自身是只读的。容器从镜像启动的时候，Docker会在镜像的最上层创建一个可写层，镜像本身将保持不变。


## 0x01 安装docker

1. Docker 要求64位。并且当CentOS7时你的内核必须不小于3.10，查看你的CentOS 版本是否支持 Docker 。

    查看当前的内核版本

    ```
    # uname -r
    3.10.0-693.21.1.el7.x86_64
    ```

1. 使用 root 权限登录 Centos。确保 yum 包更新到最新。

   ```
   # yum -y update
   ```
1. 安装和启动docker

    centos7 支持使用yum安装，省事很多

    ```
    # yum install -y docker
    # systemctl start docker
    # systemctl enable docker
    ```
1. 验证是否安装成功
    ```
    # docker version

    Client:
     Version:         1.13.1
     API version:     1.26
     Package version: docker-1.13.1-75.git8633870.el7.centos.x86_64
     Go version:      go1.9.4
     Git commit:      8633870/1.13.1
     Built:           Fri Sep 28 19:45:08 2018
     OS/Arch:         linux/amd64

    Server:
     Version:         1.13.1
     API version:     1.26 (minimum version 1.12)
     Package version: docker-1.13.1-75.git8633870.el7.centos.x86_64
     Go version:      go1.9.4
     Git commit:      8633870/1.13.1
     Built:           Fri Sep 28 19:45:08 2018
     OS/Arch:         linux/amd64
     Experimental:    false
    ```

1. 测试docker是否正常运行
    ```
    # docker run hello-world

    Hello from Docker!
    This message shows that your installation appears to be working correctly.
    ```

## 0x02 使用docker

1. 常用命令

    ```
    # 查找并拉取镜像
    docker search ubuntu
    docker pull ubuntu

    # 查看所有可用镜像
    docker images -a

    # 删除镜像
    docker rmi 容器ID 
        -f强制删除

    # 导出导入
    docker save -o ubuntu_latest.tar ubuntu:latest
    docker load --input ubuntu_latest.tar

    # 从镜像中创建容器(每运行一次便创建一个新的容器)
    docker run image
    > docker run ubuntu echo 'hellp world'  运行一个新的容器，并执行命令echo

    > docker run -it --name test ubuntu /bin/bash 以交互式终端运行一个新的容器，镜像ubuntu,使用bash，容器别名test
        -i 交互式界面，默认是false
        -t 伪终端，默认false
        --name 容器别名，默认随机命名
        -p 对外端口：docker内部端口

        进入镜像内部操作
        docker exec -it 容器别名 /bin/bash(镜像shell地址)

        exit 退出交互式界面，容器停止运行
        crtl + P 或Q推出交互式界面，容器在后台运行（PQ大写）

    # 查看容器
    docker ps 查看正在运行的容器
    docker ps -a 查看所有容器
    docker ps -l 查看最近一次运行的容器

    # 其他操作
    docker create 容器名或容器ID 创建容器
    docker start 容器名 启动容器
    docker run 容器名或容器ID 进入容器，相当于dcoker create + docker start
    docker attach 容器名或容器ID 进入容器的命令行
    docker stop 容器名 停止容器
    docker rm 容器名  删除容器（删除时必须是停止状态）
    docker top 容器名 查看容器的进程
    docker inspect 容器名 查看docker的底层信息
    docker commit 临时名 镜像名  保存镜像

    将主机/www/jy 目录拷贝到容器aaa923a806ca的/www目录下
    docker cp /www/jy aaa923a806ca:/www/
    将主机/www/jy 目录拷贝到容器aaa923a806ca中，目录重命名www
    docker cp /www/jy aaa923a806ca:/www
    将容器aaa923a806ca的/www目录拷贝到主机的/tmp目录
    docker cp aaa923a806ca:/www /tmp/



    ```
2. 守护式容器

    我们可以使用守护式容器运行一个或者多个服务，例如运行lamp服务、redis服务、mysql服务等。

    - 能够长期运行
    - 没有交互式会话
    - 适合运行应用程序和服务

    启动守护式容器<br>
    `docker run -d image`<br>
    -d 让容器在后台运行

    查看守护进行的运行状况<br>
    `docker logs [-f] [-t] [--tail] 容器名或id`<br>
    查看容器内WEB应用程序日志
    - -f --follow=true|false,默认false,一直跟随log变化
    - -t --timestamps=true|false,默认false,加上时间戳
    - --tail="all",返回最新多少条日志

    在容器中启动新的进程<br>
    `docker exec [-d] [-i] [-t] 容器名 [COMMOND] [ARG...]`

    停止守护式进程<br>
    `docker stop 容器名`      发送停止信号，等待关闭<br>
    `docker kill 容器名 `     直接关闭容器

## 0x03 Nginx部署示例
    ```
    # 创建映射端口为80的交互式界面：
    docker run -p 80 --name web -i -t ubuntu /bin/bash

    # 第一次使用更新源
    apt-get update
    
    # 安装nginx
    apt-get install nginx
    
    # 安装vim
    apt-get install vim
    
    whereis nginx
    nginx: /usr/sbin/nginx /etc/nginx /usr/share/nginx
    
    # 配置文件
    vim /etc/nginx/conf.d/localhost.conf

        server {
            listen       80;
            server_name  localhost;

            location / {
                root   /var/www/; 
                index  index.html index.htm;
            }   

        }
    
    # 创建一个新目录
    mkdir -p /var/www/
    vim /var/www/index.html

    # 启动nginx
    nginx

    # 使用Crtl+P(即Crtl+shift+p)退出容器，并后台运行。

    # 查看 
    docker port web
    docker top web

    ```
## 0x04 镜像加速器

    某些原因，需要使用加速器<br>
    `https://dev.aliyun.com/`

    登陆进入管理控制台就能看到了，按照操作文档进行加速器设置