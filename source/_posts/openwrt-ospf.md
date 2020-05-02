---
title: openwrt ospf 路由协议支持
date: 2020-05-02 12:23:01
tags:
- ospf
---
openwrt 动态路由支持  
当网络拓扑足够大的时候维护路由器的路由表是一个复杂的事，网络拓扑稍有变更或者链路故障对所有路由器的路由表都要重新配置  
这个我们可以引入动态路由协议帮助我们维护网络拓扑的路由表  
<!-- more -->
# 常见的路由协议
* RIP 路由信息协议 一种内部网关协议 主要用于小型网络
* OSPF 开放式最短路径优先 一种内部网关协议
* BGP 边界网关协议

# openwrt 动态路由支持
软件包quagga可以实现动态路由协议

* quagga-bpgd bgp 路由协议支持
* quagga-isisd isis 路由协议支持
* quagga-ospfd ospf 路由协议支持
* quagga-ospf6d ospfv3 路由协议支持
* quagga-ripd rip 路由协议支持
* quagga-vtysh 连接quagga控制台的工具
* quagga-zebra 维护路由表

安装 quagga-ospfd quagga-vtysh quagga-zebra
```
opkg install quagga-ospfd quagga-vtysh quagga-zebra
```
# 配置 quagga
一个简单的网络  
路由器R1通过Cloud1连接互联网  
拓扑中有10个网段 192.168.1.0/24 ~ 192.168.10.0.24  
每台路由器路由表至少要有10条路由转发规则才能实现所有主机互相通信，而且网络拓扑可能增加（你家真大  
![网络拓扑](/images/openwrt_ospf_1.png)  
R1 配置  
配置wan 使用动态网络地址转换 wan 口 连接 Cloud1  
启动 quagga 守护进程
```
/etc/init.d/quagga start
/etc/init.d/quagga enable
```
输入 `vtysh` 进入quagga配置  
```
configure terminal # 全局配置模式
router ospf # 启用 ospf
network 192.168.1.0/24 area 0.0.0.0 # 通告这个网段 区域 0
network 192.168.3.0/24 area 0.0.0.0 # 通告这个网段 区域 0
default-information originate # 缺省路由
do write # 保存 很重要
```
R2 配置  
跟R1差不多  
启动 quagga 守护进程
```
/etc/init.d/quagga start
/etc/init.d/quagga enable
```
输入 `vtysh` 进入quagga配置  
```
configure terminal
network 192.168.1.0/24 area 0.0.0.0
network 192.168.2.0/24 area 0.0.0.0
network 192.168.5.0/24 area 0.0.0.0
network 192.168.6.0/24 area 0.0.0.0
do write
```
R3,R4和R2的配置差不多，配置接口对应的ip， 通报网段， 防火墙开启转发  
