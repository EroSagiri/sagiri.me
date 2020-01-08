---
title: 使用wsl运行windows处理程序
date: 2020-01-08 11:24:24
tags:
    - windows
    - linux
    - wsl
---

# wsl(Windows Subsystem for Linux) windows下的Linux子系统
要使用wsl要先安装windows10  
[微软下载windows10光盘映像（ISO文件）](https://www.microsoft.com/zh-cn/software-download/windows10ISO)  
安装windows10可以使用物理机安装也可以使用虚拟机安装  
安装完成后更新系统到最新版本  
在微软商店搜索wsl,下载Ubuntu  
打控制面板-程序-程序和功能，点击启用或关闭Windows功能,勾选`适用于Linux的Windows子系统`  
重启电脑，在开启菜单找到Ubuntu并运行,开始的使用会让你创建一个管理员用户  

<!--more-->

# wine(Wine Is Not an Emulator) linux下运行windows应用程序的兼容层
## 安装wine
```
sudo apt install wine-stable
```
## 安装wine32
```
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install wine32
```

# wsl运行32位ELF支持
```
sudo apt install qemu-user-static
sudo update-binfmts --install i386 /usr/bin/qemu-i386-static --magic '\x7fELF\x01\x01\x01\x03\x00\x00\x00\x00\x00\x00\x00\x00\x03\x00\x03\x00\x01\x00\x00\x00' --mask '\xff\xff\xff\xff\xff\xff\xff\xfc\xff\xff\xff\xff\xff\xff\xff\xff\xf8\xff\xff\xff\xff\xff\xff\xff'
```

# 图形界面
## 安装 xorg
```
sudo apt install xorg
```
## 在windows10安装xlaunch，把图形界面从wsl转发到windows
从(https://sourceforge.net/projects/xming)[https://sourceforge.net/projects/xming/]  

到目前为止应该可以运行windows应用程序了  
这是我用我的电脑使用kvm虚拟机安装的windows10系统中wsl使用wine运行windows游戏效果  
![](/images/wsl.png)
