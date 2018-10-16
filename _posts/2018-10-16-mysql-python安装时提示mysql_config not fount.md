---
layout: post
title:  mysql-python安装时mysql_config not found
date:   2018-10-16 14:27:13 +0800
img:
description: mysql-python安装时mysql_config not found
categories: note
---

* 目录
{:toc}

## 0x00 描述
mysql-python安装时提示mysql_config not found
```
# pip install MySQL-python
Collecting MySQL-python
  Using cached https://files.pythonhosted.org/packages/a5/e9/51b544da85a36a68debe7a7091f068d802fc515a3a202652828c73453cad/MySQL-python-1.2.5.zip
    Complete output from command python setup.py egg_info:
    sh: 1: mysql_config: not found
    Traceback (most recent call last):
      File "<string>", line 1, in <module>
      File "/tmp/pip-build-QORDfG/MySQL-python/setup.py", line 17, in <module>
        metadata, options = get_config()
      File "setup_posix.py", line 43, in get_config
        libs = mysql_config("libs_r")
      File "setup_posix.py", line 25, in mysql_config
        raise EnvironmentError("%s not found" % (mysql_config.path,))
    EnvironmentError: mysql_config not found
```

## 0x01 解决方法

报错的原因是没有安装:libmysqlclient-dev
```
apt-get install libmysqlclient-dev
```
再执行pip install MySQL-python 安装成功