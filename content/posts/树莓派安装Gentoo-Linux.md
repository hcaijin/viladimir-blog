---
title: 树莓派安装Gentoo Linux
tags: 
- linux 
- Gentoo 
- Raspberry Pi
permalink: shu-mei-pai-an-zhuang-gentoo-linux
id: 35
updated: '2015-09-25 03:37:14'
date: 2015-09-24 09:09:57
---

## Gentoo介绍

### Gentoo的历史
         Gentoo Linux （原来被称为 Enoch Linux ) 是在1999年由Daniel Robbins 和一些开发人员开发的。目标就是创建一个没有预编译的二进制文件的Linux发行版并根据所在的硬件平台进行调整。中间因为Gentoo 缺少关键项目，没有自己的包管理系统，后面Robbins 受到FreeBSD包管理系统的启发，开发出了自己的包管理系统，被称为是portage。
         Gentoo没有二进制组件，它的包树中只包含源代码，这使它成为可以移植到其他架构上的理想的操作系统。当然，它的不足之处就是需要漫长的安装时间和大量的人工参与。不过，这不就是我们学习Linux的目的嘛。

### Gentoo与其他发行版的异同

        Gentoo安装和大多数流行的Linux发行版很多不同的地方。虽然有自启动光盘，但是没有安装程序。安装Gentoo时，所有事情都是通过命令行手动操作的。没有配置向导也没有GUI工具。不过，它有一个非常有用的安装指南（安装使用手册）。
        与其他发行版相比，Gentoo的另外一个不同点在于它没有发行版本。Gentoo是一个元发行版。元发行版指的是它会一直更新下去。使用元发行版的好处在于你可以随时更新到最新的版本程序。缺点就是你将得到非常复杂的包版本，这个版本没有经过彻底的测试，这点与Arch（另一个Linux的发行版本）是一样的。
        Gentoo的处理包的方式是一大不同点。大多数发行版使用二进制包的形式来发布包。Gentoo中发布软件包的系统被称为portage。Gentoo的开发人员是受到FreeBSD的ports collection的启发，在port collection 中只有源代码和包含构建源代码的方法的小文件会发布给终端用户。这被称为源代码发布。Gentoo的portage系统是由一组文件组成的，这个文件被称为ebuild，必要的补丁文件是由Gentoo社区创建。Ebuild文件是由一个被称为emerge的工具来读取。
        这个文件仅仅运行标准的./configure、make和make install 工具。这意味着你想要运行在Gentoo系统中的每一个应用程序都需要从源代码进行编译。Gentoo的一切都是从源代码进行编译的，因此，这个发行版要负责GCC工具链中的代码修复工作，包括那些改进GCC的性能优化。源代码的发布除了给Linux社区整体带来益处，也为终端用户带来了巨大的好处。当你在Gentoo中安装一个包是，你将可以通过选项来指定哪些模块被安装，哪些不需要安装，这个用来标记的方法就是use。use标记在构建的配置阶段使用，它们可以设置你想要编译应用程序的哪些部分，这样可以为你提供一个快速简洁的轻量级系统。不过，当你发现一些应用程序需要依赖你前面剔出的功能时，那么你就得需要花费更多的时间重新编译程序才能使用这些功能。- -

## 分区
我们是在树莓派上安装Gentoo，所以这里准备一个全新的SD卡，至少要有4GB。
拿到SD卡以后，首先我们为SD卡创建分区。==这里，要提醒一下，读卡器上有一个开关可以控制SD卡的读写性。如果，开关按了也没用，把卡拿出来，插拔多几次，主要是有的读卡器接触点不好，我就是遇到这个问题，卡了很长时间。==

* 引导分区，fat32格式
* 根分区，EXT4格式或者其他的Linux文件系统格式。

```
$ fdisk -l
Disk /dev/sda：298.1 GiB，320072933376 字节，625142448 个扇区
单元：扇区 / 1 * 512 = 512 字节
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x86258625

设备       启动    起点    末尾    扇区 大小 Id 类型
/dev/sda1  *           63  62926604  62926542    30G  7 HPFS/NTFS/exFAT
/dev/sda2        62926605 625137344 562210740 268.1G  f W95 扩展 (LBA)
/dev/sda5       272658432 272863231    204800   100M 83 Linux
/dev/sda6       272865280 281253887   8388608     4G 83 Linux
/dev/sda7       281255936 616800255 335544320   160G 83 Linux
/dev/sda8       616802304 620996607   4194304     2G 83 Linux
/dev/sda9        62928896 188758015 125829120    60G  7 HPFS/NTFS/exFAT
/dev/sda10      188760064 209731583  20971520    10G 83 Linux

分区表记录没有按磁盘顺序。

Disk /dev/sdb：14.9 GiB，15931539456 字节，31116288 个扇区
单元：扇区 / 1 * 512 = 512 字节
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x00000000

设备       启动 起点   末尾   扇区 大小 Id 类型
/dev/sdb1        2048 31116287 31114240 14.9G  c W95 FAT32 (LBA)

$ fdisk /dev/sdb
欢迎使用 fdisk (util-linux 2.27)。
更改将停留在内存中，直到您决定将更改写入磁盘。
使用写入命令前请三思。

命令(输入 m 获取帮助)：p
Disk /dev/sdb：14.9 GiB，15931539456 字节，31116288 个扇区
单元：扇区 / 1 * 512 = 512 字节
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos

磁盘标识符：0x00000000

设备       启动 起点   末尾   扇区 大小 Id 类型
/dev/sdb1        2048 31116287 31114240 14.9G  c W95 FAT32 (LBA)

命令(输入 m 获取帮助)：d
已选择分区 1
分区 1 已删除。

命令(输入 m 获取帮助)：


命令(输入 m 获取帮助)：n
分区类型
   p   主分区 (0个主分区，0个扩展分区，4空闲)
   e   扩展分区 (逻辑分区容器)
选择 (默认 p)：p
分区号 (1-4，默认 1)：
第一个扇区 (2048-31116287，默认 2048)：
上个扇区，+sectors 或 +size{K,M,G,T,P} (2048-31116287，默认 31116287)：+100M

创建了一个新分区 1，类型为“Linux”，大小为 100 MiB。

命令(输入 m 获取帮助)：n
分区类型
   p   主分区 (1个主分区，0个扩展分区，3空闲)
   e   扩展分区 (逻辑分区容器)
选择 (默认 p)：p
分区号 (2-4，默认 2)：
第一个扇区 (206848-31116287，默认 206848)：
上个扇区，+sectors 或 +size{K,M,G,T,P} (206848-31116287，默认 31116287)：

创建了一个新分区 2，类型为“Linux”，大小为 14.8 GiB。

命令(输入 m 获取帮助)：p
Disk /dev/sdb：14.9 GiB，15931539456 字节，31116288 个扇区
单元：扇区 / 1 * 512 = 512 字节
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x00000000

设备       启动 起点   末尾   扇区 大小 Id 类型
/dev/sdb1         2048   206847   204800  100M 83 Linux
/dev/sdb2       206848 31116287 30909440 14.8G 83 Linux

命令(输入 m 获取帮助)：t
分区号 (1,2，默认 2)：1
分区类型(输入 L 列出所有类型)：c

已将分区“Linux”的类型更改为“W95 FAT32 (LBA)”。

命令(输入 m 获取帮助)：p
Disk /dev/sdb：14.9 GiB，15931539456 字节，31116288 个扇区
单元：扇区 / 1 * 512 = 512 字节
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x00000000

设备       启动 起点   末尾   扇区 大小 Id 类型
/dev/sdb1         2048   206847   204800  100M  c W95 FAT32 (LBA)
/dev/sdb2       206848 31116287 30909440 14.8G 83 Linux

命令(输入 m 获取帮助)：w
分区表已调整。
将调用 ioctl() 来重新读分区表。
正在同步磁盘。
```
接下来，将分区按照它们自己的文件系统格式进行格式化。
```
$ mkfs /dev/sdb1
mke2fs 1.42.12 (29-Aug-2014)
/dev/sdb1 contains a ext2 file system
	last mounted on /mnt/cdrom/armv7 on Thu Sep 24 19:40:44 2015
无论如何也要继续? (y,n) y
Creating filesystem with 102400 1k blocks and 25688 inodes
Filesystem UUID: 256e4119-5d49-4303-9ca2-a28638838ce6
Superblock backups stored on blocks: 
	8193, 24577, 40961, 57345, 73729

Allocating group tables: 完成                            
正在写入inode表: 完成                            
Writing superblocks and filesystem accounting information: 完成 

$ mkfs.ext4 /dev/sdb2
mke2fs 1.42.12 (29-Aug-2014)
Creating filesystem with 3863680 4k blocks and 966656 inodes
Filesystem UUID: a0a95364-389a-4c4d-b63e-e1e61ed4b6c3
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208

Allocating group tables: 完成                            
正在写入inode表: 完成                            
Creating journal (32768 blocks): 完成
Writing superblocks and filesystem accounting information: 完成 
```
现在，我们已经创建好了两个可以使用的文件系统。然后，我们把相应分区挂载到/mnt 目录下。
```
$ mkdir -p /mnt/raspberry/ 
$ mount /dev/sdb2 /mnt/raspberry  #### 首先挂载根分区
$ mkdir -p /mnt/raspberry/boot && mount /dev/sdb1 /mnt/raspberry/boot ### 再挂载引导分区到根分区目录里的boot下。
```
这个引导分区和其他的树莓派引导分区类似。它包含基金会提供的固件、命令行参数、配置文件和内核。创建引导文件系统的第一步是从基金会的GitHub网站上获取固件文件：
https://github.com/raspberrypi/firmware/tree/master/boot
```
$ cd /mnt/raspberry/boot
$ wget https://github.com/raspberrypi/firmware/tree/master/boot/bootcode.bin
$ wget https://github.com/raspberrypi/firmware/tree/master/boot/start.elf
$ wget https://github.com/raspberrypi/firmware/tree/master/boot/fixup.dat
```
分别下载这三个文件就可以了。接下来，需要创建cmdline.txt 和 config.txt 两个文件。
内容如下：
```
$ cat cmdline.txt
root=/dev/mmcblk0p2 rootdelay=2
$ cat config.txt
gpu_mem=32

######完成以上步骤，用ls -l 看一下引导分区现在的文件。
$ ls -l
总用量 104
-rw-r--r-- 1 root root 29026 9月  24 22:54 bootcode.bin
-rw-r--r-- 1 root root    32 9月  24 22:59 cmdline.txt
-rw-r--r-- 1 root root    11 9月  24 23:00 config.txt
-rw-r--r-- 1 root root 29219 9月  24 22:56 fixup.dat
drwx------ 2 root root 12288 9月  24 22:33 lost+found
-rw-r--r-- 1 root root 29221 9月  24 22:56 start.elf
```
## 安装

### 下载系统包
http://distfiles.gentoo.org/releases/arm/autobuilds/current-stage3-armv7a_hardfp/
```
$ cd ~/Downloads/
$ wget http://distfiles.gentoo.org/releases/arm/autobuilds/current-stage3-armv7a_hardfp/stage3-armv7a_hardfp-20150730.tar.bz2
$ tar xfj ~/Downloads/stage3-armv7a_hardfp-20150730.tar.bz2 -C /mnt/raspberry   ### -C 指定解压的目录。解压需要一些时间，完成以后，看上去如下：
$ ls -al
总用量 89
drwxr-xr-x 20 root root  4096 7月  30 13:50 .
drwxr-xr-x  4 root root  4096 9月  24 22:38 ..
drwxr-xr-x  2 root root  4096 7月  30 21:22 bin
drwxr-xr-x  3 root root  1024 7月  30 13:50 boot
drwxr-xr-x  3 root root  4096 7月  30 13:50 dev
drwxr-xr-x 31 root root  4096 7月  30 21:31 etc
drwxr-xr-x  2 root root  4096 7月  30 13:50 home
drwxr-xr-x 10 root root  4096 7月  30 21:27 lib
drwx------  2 root root 16384 9月  24 22:33 lost+found
drwxr-xr-x  2 root root  4096 7月  30 13:50 media
drwxr-xr-x  2 root root  4096 7月  30 13:50 mnt
drwxr-xr-x  2 root root  4096 7月  30 13:50 opt
drwxr-xr-x  2 root root  4096 7月  30 13:18 proc
drwx------  2 root root  4096 7月  30 13:50 root
drwxr-xr-x  3 root root  4096 7月  30 21:22 run
drwxr-xr-x  2 root root  4096 7月  30 21:31 sbin
drwxr-xr-x  2 root root  4096 7月  30 13:50 sys
drwxrwxrwt  2 root root  4096 7月  30 21:31 tmp
drwxr-xr-x 11 root root  4096 7月  30 21:31 usr
drwxr-xr-x  9 root root  4096 7月  30 13:50 var
```
到这里，相信大家都很熟悉这个目录了。

### 配置系统文件
第一步就是配置fstab，因为这是系统引导完成以后最先需要挂载好文件系统。==这里千万要注意修改的是 /mnt/raspberry/etc/fstab ==
```
$ cat etc/fstab 
# <fs>			<mountpoint>	<type>		<opts>		<dump/pass>
# NOTE: If your BOOT partition is ReiserFS, add the notail option to opts.
/dev/mmcblk0p1		/boot   ext2		noauto,noatime	1 2
/dev/mmcblk0p2		/		ext4		noatime  		0 1

```
不要使用/dev/sdX设置的引用 。因为SD卡在树莓派上被视为/dev/mmcdlk0 。因为无法chroot到新的构建环境中，所以需要手动设置一些东西。首先，为Gentoo系统设置一个新的root用户密码。如下，
```
$ openssl passwd -1
Password: 
Verifying - Password: 
$1$ZoQIFaY4$3Re0RSS0qu6nds3wvqlRf1
```
以上，最后一行就是我们的encrypted密码，把它放到/etc/shadow文件中。
```
$ cat etc/shadow | grep root
root:$1$ZoQIFaY4$3Re0RSS0qu6nds3wvqlRf1:10770:0:::::
```
### 配置 portage
到目前为止，已经完成了所有的系统配置。但是，这还不是最终的系统，现在我们必须解压当前的portage集合。这样，才能在系统引导时，构建应用程序。下载，我们到以下地址去下载portage包。
http://distfiles.gentoo.org/releases/snapshots/current/portage-lastest.tar.bz2
```
$ cd ~/Downloads/
$ wget http://distfiles.gentoo.org/releases/snapshots/current/portage-latest.tar.bz2
$ tar xjvpf ~/Downloads/portage-latest.tar.bz2 -C /mnt/raspberry/usr
```
## 交叉编译环境
构造一个最小的 Linux 系统，
==主要分为两步：第一步是构建一个宿主系统无关的新工具链（编译器、汇编器、链接器、库和一些有用的工具）。第二步则是使用该工具链构建其它的基础工具。== 
这里说的工具链就是说的交叉编译环境。
### 下载系统内核
到了这一步以后，我们需要为新的Gentoo系统构建一个内核。为了完成这一步，需要有相关配置交叉编译环境基础知识。还需要内核源码,这里我们使用树莓派官方内核源码，因为这将包含所有已经打好的树莓派的补丁。到GitHub上clone一份内核源码或者下载zip包，如下：
```
$ git clone https://github.com/raspberrypi/linux
$ cd linux
```
### 为什么要使用交叉编译环境
交叉编译是在你的宿主系统上编译不同机器类型的应用程序。因为，树莓派使用的ARM架构与宿主机系统x86机器类型不同，只有使用相同的CPU机器类型才能使用chroot引导新系统。所以，这里就得用到交叉编译环境，在宿主系统上编译出ARM架构的树莓派。 在开始之前，希望大家可以先看一下[《Linux 从零开始》](http://linuxfromscratch.org/) 。
### crosstool-NG
暂时写到这里，以后再补充。
