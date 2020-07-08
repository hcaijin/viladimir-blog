---
title: UEFI+GPT安装Archlinux与Win10双系统教程
date: 2018-06-10 00:42:24
tags:
- archlinux
- windows
- uefi
---

# 前言
> 最近新入手一台Thinkpad,使用UEFI+GPT预安装好了Win10操作系统，准备开始安装Archlinux。

如果你准备在一块新硬盘上安装双系统，那么应该先安装windows。
如果你安装的是Win10，那么它应该默认就是按UEFI+GPT方式安装的，可以按Win+X键打开磁盘管理，如果是UEFI安装的，那么应该有一个EFI分区，可能是250M。
其它还有Windows的恢复分区和基本数据分区。不用管恢复分区，如果现在磁盘上没有剩余空间，可以右键点击基本数据分区，点击压缩卷，给Arch的安装腾出空间。用右键点击磁盘，查看属性，可以知道自己是否采用了GPT分区方式。

在安装之前，请在电源计划中关掉Windows的快速启动，并在BIOS中关掉Secure Boot，可以很容易搜到对应自己电脑的具体方法。
如果上面有哪一条没有满足，请只看一看我遇到的问题，具体安装请再参考其它教程

# 安装

## 安装之前

* 准备一个大于4G的U盘
* 安装镜像，可以从Arch Linux的[官方网站](www.archlinux.org)下载

## 制作U盘启动盘
这里我只介绍linux系统下使用`dd`的方式，windows下面的方法可以看一下这个[安装教程](https://github.com/mytbk/Linux_Notes/blob/master/install/install-archlinux.md)

插入U盘，查看U盘设备名,不需求挂载
```
lsblk
sudo dd if=archlinux-2018.06.01-x86_64.iso of=/dev/sdb
```
这样，就制作好了U盘启动了，把U盘插入要安装的机子，配置BIOS通过U盘启动，就可以进入光盘引导的临时系统。


## 选择镜像源
Arch Linux是通过网络进行安装的，为了以更快的速度下载软件包，建议先配置镜像源。配置镜像源的方法是编辑/etc/pacman.d/mirrorlist这个文件，将想用的镜像源的放到第一个非井号开头的行即可。如下可将中科大镜像源作为首选镜像源。
```
# /etc/pacman.d/mirrorlist
# This is the USTC mirror
Server = http://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch

# and other mirrors
## Score: 4.6, China
Server = http://mirrors.163.com/archlinux/$repo/os/$arch
#
```
配置完成后可以执行pacman -Syy试一下，可以看一下pacman从镜像站下载文件的速度。

## 分区
> 我们前面提过已经默认有安装的Win10系统,使用`fdisk`可以看到已经有一个EFI分区为250M大小。因此不要单独为Linux分出EFI分区，因为要双系统启动的话应该把Win10的EFI分区挂载到/boot上。

以下是我的硬盘分区情况,因为我还有一块硬盘用来挂载/home，所以我只要创建根分区和swap分区
```
Device             Start       End   Sectors  Size Type
/dev/nvme0n1p1      2048    534527    532480  260M EFI System
/dev/nvme0n1p2    534528    567295     32768   16M Microsoft reserved
/dev/nvme0n1p3    567296 254337023 253769728  121G Microsoft basic data
/dev/nvme0n1p4 254337024 485023743 230686720  110G Linux filesystem
/dev/nvme0n1p5 498069504 500117503   2048000 1000M Windows recovery environment
/dev/nvme0n1p6 485023744 497606655  12582912    6G Linux swap
```

## 格式化
```
mkfs.ext4 /dev/nvme0n1p4

mkswap /dev/nvme0n1p6
swapon /dev/nvme0n1p6
```

## 挂载分区
首先，一定是先挂载/分区，再挂载其它分区。因为要使用双系统启动，所以即使没有分/boot分区，还是应该把windows的EFI分区挂载到/上。 

```
mount /dev/nvme0n1p4 /mnt/
mkdir -p /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot
```

## 基本软件安装

安装Arch Linux的软件很简单，执行下面这条命令就行了：

```
pacstrap /mnt base base-devel
```

# 配置系统
## 创建fstab
生成一个fstab文件（使用-U或-L分别由UUID或标签定义）：
```
genfstab -U /mnt >> /mnt/etc/fstab
```
## 切换新系统
现在我们执行`arch-chroot /mnt`，这样就以chroot的方式进入了新的系统。

## 配置时间
```
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
```

## 配置本地化
在/etc/locale.gen中取消注释en_US.UTF-8 UTF-8和其他所需的本地化，并使用`local-gen`更新本地语言编码

## 设置主机名
```
echo 'hcaijin.com' > /etc/hostname
```
## 设置root密码
```
passwd
```
## 设置启动
按照上面的步骤，efi分区应该被挂载到了/boot目录下。这时，我们使用`bootctl install`命令，安装bootloader，然后用`cp /usr/share/systemd/bootctl/arch.conf /boot/loader/entries/`把示例文件复制过来，只要修改它的options部分就可以了。
以我的/boot分区为例，用`blkid -s PARTUUID -o value /dev/nvme0n1p1`就可以生成所需要的PARTUUID，最后加上rw就行了。 

格式大概为：
```
title       Arch Linux
linux       /vmlinuz-linux
initrd      /initramfs-linux.img
options     root=UUID=6278bd34-44cd-41b9-9bdd-239d9ce4020a rw
```
意思是创建一个标题为Arch Linux的启动项，它用bootloader所在分区(/dev/sda1)根目录下的vmlinuz-linux作为Linux内核，initramfs-linux.img作为initramfs镜像(可以认为是一个临时rootfs镜像)，并且用root=/dev/sda2 ro作为内核参数。

再编辑/boot/loader/loader.conf.
```
timeout 3
default arch
```
意思是默认用arch.conf的配置启动，等待3秒没有键盘操作即使用默认配置启动。

## 新系统的网络
启动盘中默认配置好了有关网络的软件，但新的系统中却没有。
如果你只是使用单一且固定的有线网络，使用`systemctl enable dhcpcd@interface.service`就可以了（interface是你的网络接口名，可以使用ip link查看，类似enp3s0）。
如果要使用无线网络，那么就要使用`pacman -S iw wpa_supplicant dialog`命令安装这些软件包。如果失败，可能要安装固件。

至此，新系统的配置就完成了。

使用exit命令退出chroot环境，umount -R /mnt卸载挂载的分区，然后使用reboot重启一下就好了。

# 最后
可能会启动失败，解决方法是进入BIOS里的设置把UEFI作为唯一的启动方式。然后保存退出，就可以看到有三个启动项（分别是Arch Linux, Windows Manage, Default），选择你要进入的系统就可以了。
