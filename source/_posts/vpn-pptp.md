---
title: pptp协议实现的vpn
date: 2019-11-24 06:19:30
tags: 
- vpn
---
点对点隧道协议(PPTP)是一种实现虚拟专用网(VPN)的方法。PPTP 在TCP之上使用一个控制通道和 GRE 隧道操作加密 PPP 数据包。
<!-- more -->

## 安装pptp软件
centos
```
# yum install -y pptpd
```
ubuntu
```
# apt install -y pptpd
```

## 配置pptpd
pptpd配置文件分别是  
/etc/pptpd.conf  
/etc/ppp/pptpd-options  
/etc/ppp/chap-secrets  

### pptpd.conf
编辑主配置文件
```
option /etc/ppp/pptpd-options
localip 10.0.0.1 # 本地地址 随便找个私有地址 10.0.0.0/24 172.16-31.0.0/16 192.168.0.0/16
remoteip 10.0.0.100-200 远程地址 要和本地地址同网络 x-x 是分配的范围
```
### pptpd-options
```
name pptpd
# 认证方式
refuse-pap
refuse-chap
refuse-mschap
require-mschap-v2
require-mppe-128 # mppe加密
proxyarp # 代理arp
nobsdcomp
novj
novjccomp
nologfd

ms-dns 8.8.8.8 # dns 地址1
ms-dns 8.8.4.4 # dns 地址2
```

### chap-secrets
添加用户密码
```
<username>  <server>    <password>   <ip>
sagiri      pptpd       123456      *
```

## 守护进程
启动
```
# systemctl start pptpd
```
开机自启
```
# systemctl enable pptpd
```
pptpd应该启动了，默认会监听1723端口,有防火墙的话要配置防火墙规则

## ip转发
pptpd可以正常连接了  
但是客户端不能上网  
因为客户端把数据包发给服务器后服务器看ip头发现不是发给它自己的包，服务器就把这个包丢了，毕竟服务器又不是路由器  
### 临时开启ip转发
```
# sysctl net.ipv4.ip_forward=1
```
### 永久开启ip转发  
编辑 /etc/sysctl.d/99-sysctl.conf
```
net.ipv4.ip_forward=1
```
应用 sysctl.conf 修改
```
# sysctl --system
```

## DNAT(动态网络地址转换)
虽然已经配置了ip转发，但是客户端还是不能上网  
因为客户端使用的是私有地址  
现在要把私有ip地址转换为服务器的公有ip地址
配置网络地址转换  
-s 源ip地址 这里是可以填一个网段  
-o 出网卡  
```
# iptables -A POSTROUGIN -t nat -s 10.0.0.0/24 -o eth0 -j MASQUERADE
```
保存配置
```
# iptables-save > /etc/iptables/iptables.rules
```

参考 [archlinux wiki PPTP server](https://wiki.archlinux.org/index.php/PPTP_server)