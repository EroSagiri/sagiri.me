---
title: centos8_PXE
date: 2020-11-07 15:55:36
tags: CentOS8 PXE 与 Kickstart
---
# CentOS8 PXE 与 Kickstart
## PXE
预启动执行环境（Preboot eXecution Environment，PXE，也被称为预执行环境）提供了一种使用网络接口（Network Interface）启动计算机的机制。这种机制让计算机的启动可以不依赖本地数据存储设备（如硬盘）或本地已安装的操作系统。 
<!-- more -->
### DHCP
动态主机配置协议,是位于网络层的协议。主要用于在网络管理ip地址分配。DHCP的定义 [RFC 2131](https://tools.ietf.org/html/rfc2131)
#### 工作原理
客户端先把IP地址设置为0.0.0.0向网络广播(255.255.255.255)DHCP DISCOVER 寻找可用的DHCP服务器。  
网络内的所有设备都会收到客户端发送的DHCP DISCOVER,但是只有DHCP服务器会向客户端发送DHCP OFFER  DHCP服务器为客户端保留一个IP地址，该消息包含客户端MAC地址、服务器提供的IP地址、子网掩码、租期信息和服务器的IP地址。  
客户端收到DHCP OFFER必须告诉服务器接受配置信息，向DHCP服务器发送DHCP REQUEST。
最后服务器向客户端发送DHCP ACK 确认配置。
### TFTP
TFTP的定义 [RFC 1350](https://tools.ietf.org/html/rfc1350)   
简单文件传输协议有以下特点  
* 使用UDP(端口69)  
* 不能列出目录
* 无验证或加密机制  
* 被用于在远程服务器上读取或写入文件  
因为没有验证和加密机制通常使用于私人网络。PXE无盘启动、网络设备文件传输。
### Linux启动加载器
启动加载器主要用于加载Linux内核，如GRUB,Syslinux,LILO,rEFInd
### 固件类型
BIOS是旧的固件类型，通过主引导记录执行引导器，这种设备的存储设备通常使用MBR分区表  
UEFI新的固件，通过eps分区，通过引导管理器的启动条目加载UEFI应用  
### DHCP Server
#### 安装 dhcp-server

```
dnf -y install dhcp-server
```

#### 配置 dhcp-server
确保网络内只有一台DHCP服务器  
确认centos8的IP地址  
IP: 192.168.122.100/24 
网关: 192.168.122.1/24  
next-serve tftp 服务器的地址  
centos8 dhcp-server IPV4 配置文件在 /etc/dhcp/dhcpd.conf
```
# pxelinux 命名空间
option space pxelinux;
option pxelinux.magic code 208 = string;
# 配置文件
option configfile code 209 = text;
# 前置路径
option pxelinux.pathprefix code 210 = text;
option pxelinux.reboottime code 211 = unsigned integer 32;
option architecture-type code 93 = unsigned integer 16;

subnet 192.168.122.0 netmask 255.255.255.0 {
        option routers 192.168.122.1;
        option domain-name-servers 223.5.5.5;
        range 192.168.122.2 192.168.122.50;

        class "pxeclients" {
          match if substring (option vendor-class-identifier, 0, 9) = "PXEClient";
          next-server 192.168.122.100;

          if option architecture-type = 00:07 {
            # UEFI
            filename "uefi/grubx64.efi";
            } else {
            # BIOS
            filename "pxelinux/pxelinux.0";
          }
        }
}
```
### 本地源
网络安装CentOS,要配置本地源
#### http 协议  
安装nginx,把CentOS8 DVD 镜像挂载到 /usr/share/nginx/html/centos
```
dnf -y install nginx
mkdir /usr/share/nginx/html/centos
mount CentOS-8.2.2004-x86_64-dvd1.iso.iso /usr/share/nginx/html/centos
sytemctl enable --now nginx
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
```
repo地址: http://192.168.122.100/centos  
#### nfs 协议
安装 nfs-utils,挂载Centos8 DVD 镜像到 /mnt
```
dnf -y install nfs-utils
mount CentOS-8.2.2004-x86_64-dvd1.iso.iso /mnt
```
编辑 /etc/exports, 共享 /mnt
```
/mnt 192.168.122.0/24 (ro)
```
防火墙
```
firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-port=2049/tcp
firewall-cmd --permanent --add-port=2049/udp
firewall-cmd --permanent --add-port=111/tcp
firewall-cmd --permanent --add-port=111/udp
firewall-cmd --permanent --add-port=20048/tcp
firewall-cmd --permanent --add-port=20048/udp
firewall-cmd --reload
```
repo地址: nfs:192.168.122.100:/mnt  
#### ftp 协议
安装 vsftpd
```
dnf -y install vsftpd
```
配置 vsftpd 允许匿名登陆 /etc/vsftpd/vsftpd.conf
```
anonymous_enable=YES
```
直接把镜像挂载到/var/ftp无法使用，只能把整个镜像复制到/var/ftp了
```
mount CentOS-8.2.2004-x86_64-dvd1.iso.iso /mnt
cp -r /mnt /var/ftp/centos
```
repo地址: ftp://192.168.122.100/centos  

### 配置 tftp-server
#### 安装 tftp-server
```
dnf -y install tftp-server
```
#### 复制内核和初始化文件系统
下载 CentOS8 的 DVD 镜像,通过mount挂载镜像，复制/mnt/images/pxeboot{vmlinuz,initrd.img} 到 /var/lib/tftpboot/images  
注意权限问题，至少能让其他用户正常访问  
```
mount CentOS-8.2.2004-x86_64-dvd1.iso /mnt
mkdir /var/lib/tftpboot/images
cp /mnt/images/pxeboot{vmlinuz,initrd.img} /var/lib/tftpboot/images
chmod 0775 -R /var/lib/tftpboot/images
```

#### 引导程序
##### GRUB BIOS  
一般CentOS8 已经安装好了grub2
创建网络启动文件夹  
```
grub2-mknetdir --net-directory=/var/lib/tftpboot/grub
```
执行成功，后会返回一个文件路径，把这个文件路径减去tftp服务器的根目录就得到PXE BIOS启动的引导文件了。  
例如: /var/lib/tftpboot/grub/boot/grub2/i386-pc/core.0 /grub/boot/grub2/i386-pc/core.0  
把这个路径,修改 /etc/dhcp/dhcpd.conf  # BIOS 下的 filename=/grub/boot/grub2/i386-pc/core.0  
grub的配置文件 /var/lib/tftpboot/grub/boot/grub2/i386-pc/grub.cfg  
linux [linuxi内核先对于 tftp root 目录的位置] [内核启动选项]  
initrd [初始化文件系统]  
关于内核启动选项可阅读 [Boot options reference](https://docs.centos.org/en-US/8-docs/standard-install/assembly_custom-boot-options/)
```
set default=0
set timeout=5

echo -e "\nCentos8\n\n"

menuentry 'Centos8' {
        linux /vmlinuz ro ip=dhcp inst.repo=nfs:192.168.122.100:/mnt
        initrd /initrd.img
}
```
##### Syslinux BIOS
安装 syslinux-tftpboot,复制/tftpboot 到 /var/lib/tftpboot/syslinux
```
dnf -y install syslinux-tftpboot
cp -r /tftpboot /var/lib/tftpboot/syslinux
cp /var/lib/tftpboot/{vmlinuz,initrd.img} /var/lib/tftpboot/syslinux
```
配置文件
```
mkdir /var/lib/tftpboot/syslinux/pxelinux.cfg
touch /var/lib/tftpboot/syslinux/pxelinux.cfg/default
```
编辑配置文件 /var/lib/tftpboot/syslinux/pxelinux.cfg/default  
```
default vesamenu.c32
prompt 1
timeout 50

display boot.msg

label linux
  menu label ^Install system
  menu default
  kernel vmlinuz
  append initrd=initrd.img ip=dhcp inst.repo=nfs:192.168.122.100:/mnt
label local
  menu label Boot from ^local drive
  localboot 0xffff
```

#### GRUB UEFI
安装 grub2-efi-x64 和 shim-x64
dhcp配置文件 uefifile 设置为 /uefi/shim.efi  
在CentOS8 官方源中找不到x86_64的支持，只能用x86_32  
/var/lib/tftpboot/grub文件夹是 grub2-mknetdir生成的  
grub的配置文件存放在 /var/lib/tftpboot/uefi/grub.cfg  
```
dnf -y install grub2-efi-x64 shim-x64
mkdir /var/lib/tftpboot/uefi/
cp /boot/efi/EFI/centos/shimx64.efi  /var/lib/tftpboot/uefi
cp /boot/efi/EFI/centos/grubx64.efi /var/lib/tftpboot/uefi
mkdir -p /var/lib/tftpboot/EFI/centos
cp -r /var/lib/tftpboot/grub/boot/grub2/i386-pc /var/lib/tftpboot/EFI/centos/x86_64-efi
touch /var/lib/tftpboot/uefi/grub.cfg
```

## Kickstart
自动化安装CentOS 的方法  
通过设置Linux内核启动选项 inst.ks，会根据给定的配置文件自动安装系统
CentOS安装完成CentOS8后会生成一个anaconda-ks.cfg 存放在 /root 目录  
把这个文件通过http协议共享，当然你也可以参考[Kickstart Installations](https://docs.centos.org/en-US/centos/install-guide/Kickstart2/#sect-kickstart-introduction)手动编写一个配置文件  
grubg引导的配置文件  
![](/images/Screenshot_20201107_230140.png)