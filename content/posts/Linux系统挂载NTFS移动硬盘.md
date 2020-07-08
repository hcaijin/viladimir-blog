---
title: Linux系统挂载NTFS移动硬盘
tags: 
- linux 
- ntfs 
permalink: linuxxi-tong-gua-zai-ntfsyi-dong-ying-pan
id: 56
updated: '2016-03-31 21:33:10'
date: 2016-03-08 02:19:36
---

> 工作中，同事之间拷贝数据的时候，由于我用的linux系统无法识别同事的NTFS移动硬盘，这才网上查了一下，原来还得安装一个软件，以下是[转载](http://www.cnblogs.com/kerrycode/archive/2013/04/01/2993701.html)的文章,记录一下。

***
有时候做大数据量迁移时，为了快速迁移大数据，有可能在Linux服务器上临时挂载NTFS格式的移动硬盘， 一般情况下，Linux是识别不了NTFS格式移动硬盘的（需要重编译Linux核心才能，加挂NTFS分区），这时候为了能让Linux服务器能够识别NTFS的移动硬盘，就必须安装ntfs-3g（Third Generation Read/Write NTFS Driver）的包。
***

NTFS-3G介绍
NTFS-3G是一个开源项目，NTFS-3G是为Linux, Android, Mac OS X, FreeBSD, NetBSD, OpenSolaris, QNX, Haiku,和其他操作系统提供的一个稳定的，功能齐全，读写NTFS的驱动程序的。它提供了安全处理Windows XP，Windows Server 2003，Windows 2000，Windows Vista，Windows Server 2008和Windows 7操作系统下的NTFS文件系统。

NTFS-3g是一个开源软件，它支持在Linux下面读写NTFS格式的分区。它非常的快速，同时也很安全。它支持Windows 2000、XP、2003和Vista，并且支持所有的符合POSIX标准的磁盘操作。 ntfs-3g的目的是为了持续的发展，各硬件平台和操作系统的用户需要可靠的互通与支持ntfs的驱动，ntfs-3g可以提供可信任的、功能丰富的高性能解决方案。经过了12年多的发展，ntfs-3g已经逐渐稳定； 

> 资料介绍

* 官方网址：http://www.tuxera.com/，
* 文档手册：http://www.tuxera.com/community/ntfs-3g-manual/
* 下载地址：http://www.tuxera.com/community/ntfs-3g-download/

### 安装
#### 解压安装NTFS-3G。
```　
tar -xvzf ntfs-3g_ntfsprogs-2012.1.15.tgz　　
cd ntfs-3g_ntfsprogs-2012.1.15
　　./configure 
　　make 
　　make install 
```
*如果没有报错，提示安装成功，下面就可以用ntfs-3g来实现对NTFS分区的读写了*

### 配置

#### 配置挂载NTFS格式的移动硬盘

##### 首先得到NTFS分区的信息 

```
$ sudo fdisk -l | grep NTFS
/dev/sdc1 * 1 244 1955776+ 7 HPFS/NTFS 
```

##### 设置挂载点，用如下命令实现挂载 
```
mount -t ntfs-3g <NTFS Partition> <Mount Point> 
```

> 例如得到的NTFS分区信息为/dev/sdc1，挂载点设置在/mnt/usb下:
```
$ mount -t ntfs-3g /dev/sdc1 /mnt/usb 
########## 或者直接用 #######
$ ntfs-3g ntfs-3g /dev/sdc1 /mnt/usb 
```

#### 如果想实现开机自动挂载，可以在/etc/fstab里面添加如下格式语句 
```
<NTFS Partition> <Mount Point> ntfs-3g silent,umask=0,locale=zh_CN.utf8 0 0 
```
==这样可以实现NTFS分区里中文文件名的显示。 ==

　　
### 卸载分区
```
$ umount <NTFS Partition> 　　
##### 或者 #####
$ umount <Mount Point>
```
