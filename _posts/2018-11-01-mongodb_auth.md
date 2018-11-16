---
layout: post
title:  mongodb开启认证
date:   2018-11-01 13:35:11 +0800
img:
description: mongodb开启认证
categories: note
---

修改配置文件，开启认证

```
# vi /etc/mongod.conf

    security:
    authorization:enablesd
```

创建密码

- **db.createUser({ user: "w11scan", pwd: "w11scan", roles: [{ role: "dbOwner", db: "w11scan_config" }] })**  <br>mongo创建db密码
- **db.auth("useradmin", "adminpassword")** <br>验证，1表示成功
mongodb中的用户是基于身份role的
    - 管理员账户的 role是 userAdminAnyDatabase，‘userAdmin’代表用户管理身份，’AnyDatabase’ 代表可以管理任何数据库。
    - dbOwner 代表数据库所有者角色，拥有最高该数据库最高权限。比如新建索引等
    - readWrite 该用户用于该数据的读写，只拥有读写权限。