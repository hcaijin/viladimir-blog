---
title: 'Linux 系统Chrome,Firefox程序无用使用Fcitx的问题解决方法'
tags: 
- linux 
- chrome 
- firefox 
- fcitx
permalink: >-
  linux-xi-tong-chromefirefoxcheng-xu-wu-yong-shi-yong-fcitxde-wen-ti-jie-jue-fang-fa
id: 67
updated: '2017-01-08 22:16:21'
date: 2016-12-25 05:04:29
---

### 起因
使用的gentoo有半年没有更新系统了，原来用的好好的输入法，更新完以后，在其他的程序都可以正常使用fcitx。但是，在chrome,firefox（后来知道应该是GTK,QT相关的程序用了最新版导致的问题）就是用不了，网上也有很多人提问，也没有一个有效的解决方法。

### 环境
* Linux hcj-arch 4.4.39-1-lts #1 SMP Thu Dec 15 21:10:18 CET 2016 x86_64 GNU/Linu
* fcitx version: 4.2.9.1
* Google Chrome 55.0.2883.87
* Mozilla Firefox 50.1.0

### 检查
首先保证环境变量有设置，当然，如果其他程序都可以使用，那这个应该是没有问题的
```
export GTK_IM_MODULE=fcitx 
export QT_IM_MODULE=fcitx 
export XMODIFIERS=@im=fcitx
```
主要的问题就是我们要用命令 `fcitx-diagnose` 查看fcitx的相关模块是不是有安装。（更无脑的方式就是把这个命令里显示为红色的信息都看一遍，把相关的模块安装上就ok了）

那么，我们可以看到：
![](/content/images/2017/01/Screenshot_2016-12-27_10-51-56.png)

![](/content/images/2017/01/Screenshot_2016-12-27_10-52-09.png)

== 如上图所示，缺少gtk2,gtk3相关的模块支持，导致的Chrome,Firefox等gtk软件无法使用输入法的情况 ==


### 解决
我们先看一下fcitx构建时用到的USE标记，以下
![](/content/images/2017/01/Screenshot_2017-01-09_11-06-32.png)
可以看到，我自己设置的是默认不安装gtk支持的，所以我们要加上，有以下两种方法：

* 可以直接在/etc/portage/make.conf USE标记上加上gtk的支持
* 直接定义USE标记，加上gtk的支持 
```
 USE="X autostart cairo dbus enchant introspection nls pango qt4 table xml -debug gtk2 gtk3 -lua -opencc -static-libs {-test}" sudo emerge fcitx
```

> 最后，重新编译安装过fcitx以后，再看一下`fcitx-diagnose`，只要没有红色相关字体的警告信息，就说明已经可以正常使用了。把浏览器重启一下，如果还不行，得重启一下系统。

