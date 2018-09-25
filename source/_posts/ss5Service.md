---
title: 使用ss5搭建SOCKS服务器
date: 2018-09-09 21:46:02
tags:
- linux
- proxy
---
SOCKS是一种网络传输协议，主要用于客户端与外网服务器之间通讯的中间传递。  
当防火墙后的客户端要访问外网服务器时，就跟SOCKS代理服务器连接。这个代理服务器控制客户端访问外网的资格，允许的话，就将客户端的请求发往外部的服务器。

<!--more-->
SOCKS服务器软件
+ Dante Socks Server  [http://www.inet.no/dante](http://www.inet.no/dante)
+ Java Socks Server [http://www.inet.no/dante](http://www.inet.no/dante)
+ SOCKS4 Server  [https://archive.is/20130502024508/http://www.alhem.net/project/socks4](https://archive.is/20130502024508/http://www.alhem.net/project/socks4)
+ SS5 Socks Server  [http://ss5.sourceforge.net/](http://ss5.sourceforge.net/)
+ TcpToute2  [https://github.com/GameXG/TcpRoute2](https://github.com/GameXG/TcpRoute2)

使用SS5,搭建SOCKS服务主要用于翻墙,服务器系统是Centos7  
## 去官网下载rpm包  
[SS5官网](http://ss5.sourceforge.net/) 还要跳转到sourceforge下载软件 获取到下载链接
``` shell
# 下载rpm包
wget https://jaist.dl.sourceforge.net/project/ss5/ss5/3.8.9-8/ss5-3.8.9-8.src.rpm
# 安装
rpm -ivh ss5-3.8.9-8.src.rpm
```
## 修改SS5配置文件
编辑 `/etc/opt/ss5/ss5.conf` 文件，把找到下面两行并把注释去掉
```
auth    0.0.0.0/0               -              -
permit -        0.0.0.0/0       -       0.0.0.0/0       -       -       -       -       -
```
默认无用户认证  
如果想要设置用户密码认证，则更改为
```
auth    0.0.0.0/0               -              u
permit u        0.0.0.0/0       -       0.0.0.0/0       -       -       -       -       -
```
编辑 `/etc/opt/ss5/ss5.passwd` 添加用户和密码一行一个（用户和密码之间有个空格）
```
user1 123456
user2 654321
```
启动参数默认是1080端口，可在 `/etc/sysconfig/ss5`更改  
例如改成1000端口
```
# Add startup option here
SS5_OPTS=" -u root -b 0.0.0.0:1000"
```
## 启动
```
systemctl start ss5.service
```
## 开放防火墙端口
```
firewall-cmd --permanent --zone=public --add-port=1080/tcp
firewall-cmd --reload
```
## 测试
使用curl测试
```
➜  ~ curl --socks5 ssr.sagiri.me:1080 google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```
## 自启
```
systemctl enable ss5.service
```
##  关闭
```
systemctl stop ss5.service
```


