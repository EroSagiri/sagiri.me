---
title: xclip剪贴板
date: 2018-12-16 13:43:26
tags: 
- linux
- tool
---

xclip是一个命令行剪贴板工具  
<!-- more -->

## 安装  
`sudo pacman -S xclip` (对于archlinux)

## 使用方式  
xclip [选项] [文件]

## 常用参数  
-i 从标准输入或者文件读取文本到剪贴板(默认)  
-o 输出剪贴板的文本  
-f 从标准输入读取到剪贴板并且在打印在标准输出  
-r 删除最后一个换行符(如果有)  
-h 显示帮助  
-selection 选择访问,可以选项 buffer-cut,clipboard,primary(默认),secondary  

## 例子  
复制 ~/.ssh/id_rsa.pub 这个文件
```
➜  ~ xclip ~/.ssh/id_rsa.pub
```
打印到标准输出
```
➜  ~ xclip -o
```
复制 ~/.ssh/id_rsa.pub 这个文件到clipboard这个剪贴板，这样你就可以在图形界面下粘贴了
```
➜  ~ xclip -selection clipboard ~/.ssh/id_rsa.pub
```
通过标准输入读取 <kbd>Ctrt+d</kbd> 结束
```
➜  ~ xclip
```
通过管道符复制时间并且打印到标准输出
```
➜  ~ date | xclip -f
```
通过重定向符复制 ~/.ssh/id_rsa.pub 这个文件
```
➜  ~ xclip < .ssh/id_rsa.pub
```


参考 `man xclip`
