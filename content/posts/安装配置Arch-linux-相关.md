---
title: 安装配置Arch linux 相关
tags: 
- linux 
- archlinux 
- grub
permalink: an-zhuang-pei-zhi-arch-linux-xiang-guan
id: 9
updated: '2015-05-10 22:22:24'
date: 2015-05-10 22:19:00
---

## 安装 linux-lts 软件包

*小贴士: 强烈推荐安装linux-lts作为备用内核，因为默认安装的linux内核比较新，容易与其它软件发生冲突*

>linux-lts 是 Arch 官方提供的基于 Linux kernel 3.0 的长期支持内核。内核上游开发者针对此版本提供了长期支持，包括安全补丁和功能 backports。适用于需要长期支持的服务器环境用户，也可以将此内核作为新内核升级的后备内核。

`$ sudo pacman -S linux-lts`   

## 配置启动

需要编辑 GRUB2 或 LILO 的启动加载项。  
以 GRUB2 为例：   
为了编辑/更新启动加载项，需要安装os-prober.  
安装后,再执行   
`$ sudo grub-mkconfig -o /boot/grub/grub.cfg`
