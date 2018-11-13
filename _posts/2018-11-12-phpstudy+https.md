---
layout: post
title:  phpstudy配置ssl证书实现https加密访问
date:   2018-10-19 17:23:03 +0800
img:
description: phpstudy配置ssl证书实现https加密访问
categories: note
---

* 目录
{:toc}

## 0x00 前言

使用HTTP（超文本传输）协议访问互联网上的数据是没有经过加密的。也就是说，任何人都可以通过适当的工具拦截或者监听到在网络上传输的数据流。但是有时候，我们需要在网络上传输一些安全性或者私秘性的数据，譬如：包含信用卡及商品信息的电子订单。这个时候，如果仍然使用HTTP协议，势必会面临非常大的风险！相信没有人能接受自己的信用卡号在互联网上裸奔。

HTTPS（超文本传输安全）协议无疑可以有效的解决这一问题。所谓HTTPS，其实就是HTTP和SSL/TLS的组合，用以提供加密通讯及对网络服务器的身份鉴定。HTTPS的主要思想是在不安全的网络上创建一安全信道，防止黑客的窃听和攻击。

## 0x01 使用openssl生成x509签名证书

x509证书一般会用到三类文，key，csr，crt

- key是服务器上的私钥文件，用于对发送给客户端数据的加密，以及对从客户端接收到数据的解密，通常是rsa算法。

- csr 是证书签名请求文件，用于申请证书。在制作csr文件的时，必须使用自己的私钥来签署申，还可以设定一个密钥。

- crt是由证书颁发机构（CA）签名后的证书，或者是开发者自签名的证书，包含证书持有人的信息，持有人的公钥，以及签署者的签名等信息 

1. key的生成 

    `# openssl genrsa -des3 -out server.key 2048`

    这样是生成rsa私钥，des3算法，openssl格式，2048位强度。server.key是密钥文件名。为了生成这样的密钥，需要一个至少四位的密码。可以通过以下方法删除私钥中的密码:

    `# openssl rsa -in server.key -out server.key`

2. 生成CA的crt
    ```
    # openssl req -new -key server.key -out server.csr

    Country Name (2 letter code) [AU]:CN
    State or Province Name (full name) [Some-State]:Beijing
    Locality Name (eg, city) []:Beijing
    Organization Name (eg, company) [Internet Widgits Pty Ltd]:Test
    Organizational Unit Name (eg, section) []:Test
    Common Name (e.g. server FQDN or YOUR name) []:pa55w0rd.club
    Email Address []:pa55w0rd@aliyun.com
    ```
    需要依次输入国家，地区，组织，email。common name可以写你的名字或者域名。如果为了https申请，这个必须和域名吻合，否则会引发浏览器警报。生成的csr文件交给CA签名后形成服务端自己的证书。




3. crt生成方法
    ```
    # openssl x509 -req -days 3650 -in server.csr -signkey server.key -out server.crt

    Signature ok
    subject=C = CN, ST = Beijing, L = Beijing, O = Test, OU = Test, CN = pa55w0rd.club, emailAddress = pa55w0rd@aliyun.com
    Getting Private key
    ```

    crt上有证书持有人的信息，持有人的公钥，以及签署者的签名等信息。当用户安装了证书之后，便意味着信任了这份证书，同时拥有了其中的公钥。证书上会说明用途，例如服务器认证，客户端认证，或者签署其他证书。当系统收到一份新的证书的时候，证书会说明，是由谁签署的。如果这个签署者确实可以签署其他证书，并且收到证书上的签名和签署者的公钥可以对上的时候，系统就自动信任新的证书。

    最后生成了私用密钥：server.key和自己认证的SSL证书：server.crt

    证书合并：

    `cat server.key server.crt > server.pem`

4. 客户端证书文件

    上面三步骤为服务端证书生成步骤，用户配置单向SSL时需要使用的证书文件，如果要配置双向SSL

    证书生成步骤是一样的，最后需要将crt和key合并生成客户端证书安装包 pfx

    `openssl pkcs12 -export -in client.crt -inkey client.key -out client.pfx`


## 0x02 phpstudy安装ssl证书

1. 打开`php-openssl` 选项

    点击【其他选项菜单】按钮→选择【PHP扩展及设置】→选择【PHP扩展】→在【php-openssl】选项上打钩即可。

2. 使SSL模块生效

    打开`phpStudy\Apache\conf\httpd.conf`文件，修改配置文件注意备份

    找到 `LoadModule ssl_module modules/mod_ssl.so`，去除`#`注释，使ssl模块生效

    搜索`Include conf/vhosts.conf`，在其下面增加一条
    `Include conf/vhostssl.conf`

3. 在Apache安装目录下创建cert目录，将生产的证书拷贝过来

4. 在Apache安装目录下conf文件夹中创建一个vhostssl.conf配置文件(文件名要和httpd.conf文件中引用的名称一致)

    ```PHP
    Listen 443
    <VirtualHost *:443>
        DocumentRoot "E:\phpStudy\WWW"
        ServerName www.pa55w0rd.club
        ServerAlias pa55w0rd.club
        ErrorLog "E:\phpStudy\Apache\logs\error.log"
        TransferLog "E:\phpStudy\Apache\logs\access.log"
        SSLEngine on
        SSLProtocol TLSv1 TLSv1.1 TLSv1.2
        SSLCipherSuite HIGH:MEDIUM:!aNULL:!MD5
        SSLCertificateFile "E:\phpStudy\Apache\cert\server.pem"
        SSLCertificateKeyFile "E:\phpStudy\Apache\cert\server.key"
        SSLCertificateChainFile "E:\phpStudy\Apache\cert\server.pem"
      <Directory "E:\phpStudy\WWW">
          Options FollowSymLinks ExecCGI
          AllowOverride All
          Order allow,deny
          Allow from all
          Require all granted
      </Directory>
    </VirtualHost>
    ```

5. 重启Apache，访问https://xxx 访问正常

## 0x03 HTTP301重定向到HTTPS

配置完SSL证书，我们需要进行站点301重定向，将http的地址强制跳转到https地址，Apache环境下，在站点根目录添加.htaccess文件

```
RewriteEngine on
RewriteBase /
RewriteCond %{SERVER_PORT} !^443$
RewriteRule ^.*$ https://%{SERVER_NAME}%{REQUEST_URI} [L,R=301]
```

PS. Windows下不允许创建.开头的文件，可以先创建任意文件，通过copy命令重命名.htaccess

测试自动跳转到https