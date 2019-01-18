---
title: uefi启动直接加载内核
date: 2019-01-06 10:55:13
tags:
- linux
- uefi
---
uefi(Unified Extensible Firmware Interface）是一种个人电脑系统规范，用来规范操作系统和系统固件之间的软件界面，作为BIOS的替换方案。  
使用uefi直接加载内核的好处是减小启动时间。
<!-- more -->
## 验证启动模式
一般主板会支持两种启动模式 BIOS UEFI 
```
# ls /sys/firmware/efi/efivars
```
如果目录不存在，系统可能是BIOS模式启动的。
## EFI分区
要使用UEFI启动硬盘上要有一个特殊的EFI分区
查看分区表
```
$ fdisk -l /dev/sda
```
## 把内核和初始化内存复制到EFI分区
内核和初始化内存一般在 /boot 目录  
vmlinuz-linux 和 initramfs-linux.img 文件  
如果没有挂载EFI分区，要挂载  
复制到EFI分区 /mnt 是挂载的目录
```
# cp /boot/{vmlinuz-linux,initramfs-linux.img} /mnt
```
## 添加启动项
efibootmgr 工具可以访问启动项  
查看启动项
默认下会执行磁盘下的EFI分区下的/EFI/boot/bootx64.efi程序
```
efibootmgr
```
添加一个启动项  
-d 是EFI分区所在的设备  
-p 是EFI的分区号  
-c 创建一个启动项  
-L 启动项的标签  
-l efi程序的路径，注意要用\表示目录，列如EFI分区下的vmlinuz-linux \vmlinuz-linux  
-u 参数 root 根目录所在的设备 rw 读写 initrd 初始化内存

```
# efibootmgr -d /dev/sda -p 1 -c -L "Arch Linux" -l /vmlinuz-linux -u "root=/dev/sda2 rw initrd=/initramfs-linux.img"
```
注意每次更新内核或生成新的初始化内存的时候要重新把内核和初始化内存镜像复制到EFI分区  
好的解决方法把EFI分区挂载到/boot目录  


## 调整启动顺序  
把第一个要加载的放在前面，可使用 efibootmgr 查看编号
```
# efibootmgr -o 1,2,3
```

参考 [EFISTUB - ArchWiki](https://wiki.archlinux.org/index.php/EFISTUB)
