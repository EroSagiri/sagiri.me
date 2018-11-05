---
title: 单个Wi-FI设备同时作为无线客户端和AP
date: 2018-11-05 17:10:39
tags:
- linux
---
通过虚拟网卡实现单个设备同时作为热点和无线客户端  
需要用到的软件 iw, create_ap, ip  
<!-- more -->
## 无线网卡必须支持AP模式
要验证这点，请执行 `iw list` 命令，结果Supported interface modes 段落中应该有AP模式  
```
➜  iw list
...
        Supported interface modes:
                * IBSS
                * managed
                * AP
                * AP/VLAN
                * monitor
                * P2P-client
                * P2P-GO
                * P2P-device
...
```
验证设备是否支持并行操作
```
➜  iw list | grep channels
        total <= 3, #channels <= 2
```
约束 `#channels <= 1`说明软件热点必须和wi-fi客户端连接处于同一信道  
## 虚拟网络接口
查看当前网卡 `ip address`, 会有一个wlan或者wlp开头的网卡
```
➜  ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: wlp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DORMANT group default qlen 1000
    link/ether 40:a3:cc:79:0e:4b brd ff:ff:ff:ff:ff:ff
```
用wi-fi网卡虚拟一个网络接口 wlps0_ap
```
sudo iw dev wlp2s0 interface add wlp2s0_ap type managed addr 12:34:56:78:ab:cd
```
## 开启AP
使用create_ap用wlp2s0_ap这个网络接口开启一个热点，使用wlp2s0的网络
```
➜  sudo create_ap wlp2s0_ap wlp2s0 sagiri
```
这样就开启了一个SSID为sagiri，无密码的热点  
## 连接网络
使用wlp2s0这个网卡连接wi-fi，用图像界面就行了，命令行好麻烦的  

参考 [archlinux Software access point](https://wiki.archlinux.org/index.php/Software_access_point_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))