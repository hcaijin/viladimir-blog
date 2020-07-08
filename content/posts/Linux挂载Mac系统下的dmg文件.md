---
title: Linux挂载Mac系统下的dmg文件
tags: 
- linux 
- mount 
- mac 
- virtualbox
permalink: linuxgua-zai-mac-xi-tong-xia-de-dmgwen-jian
id: 65
updated: '2016-05-27 06:49:36'
date: 2016-05-27 05:06:32
---

> 最近想在virtualbox下安装Mac系统，了解到Mac的安装镜像文件是dmg格式的，并下载到了 Install OS X Yosemite 10.10.1.dmg 安装包。

## 解压缩
本来以为Mac的安装与其他系统的类似，只要把镜像包在虚拟机中做为cd启动就可以了，然而并没什么用 - -

这不，想到把dmg格式的包转化为iso的格式再在虚拟机中启动，这就有了这篇文章的问题了。

google到这个工具acetoneiso可以直接把dmg格式的转为iso

但是，我想是不是可以用更简单的方法来操作。
现在的dmg一般都使用(zlib 或者 bzip2压缩算法)压缩过

需要使用dmg2img把dmg文件转为img
```
$ dmg2img Install\ OS\ X\ Yosemite\ 10.10.1.dmg yosemite.img
```
提示如，就表示成功了：
Archive successfully decompressed as yosemite.img

## 检查模块
在挂载之前我们要先确保hfsplus模块启用：
```
lsmod | grep hfs
```
如果没有输出，就表示模块未启用，使用如下命令启用：
```
modprobe hfsplus
```

## 挂载
启用成功后，就可以用mount挂载img，这里我挂载失败，提示存在坏道，在[这里](http://www.linuxquestions.org/questions/linux-software-2/how-to-mount-dos-img-file-4175430554/)才找到了解决的方法。
```
mount -t hfsplus -o loop my.img /mnt/hfs

mount: wrong fs type, bad option, bad superblock on /dev/loop0,
       missing codepage or helper program, or other error
       In some cases useful info is found in syslog - try
       dmesg | tail or so
```

### 问题处理
查询系统日志在最下面提示如下信息：
```
dmesg | tail 
[2015609.436682] hfsplus: unable to find HFS+ superblock
```
解决方案：

1.先用fdisk查询img扇区
![](/content/images/2016/05/--_2016-05-27_17-42-21.png)
可以看到它有两个设备*.img1,*.img2 

2.把img的文件挂载出来就得找到开始挂载的起始扇区，所以要设置一下offset的值，
这里`offset=1259643×512`，运行以下：
```
sudo mount -t hfsplus -v -o loop,offset=644937216 yosemite.img /mnt/hfs
```

以上，就可以把镜像挂载到了目录/mnt/hfs下。
