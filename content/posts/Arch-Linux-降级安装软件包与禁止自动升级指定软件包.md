---
title: Arch Linux 降级安装软件包与禁止自动升级指定软件包
tags: 
- linux
- archlinux 
- pacman
permalink: >-
  arch-linuxjiang-ji-an-zhuang-ruan-jian-bao-yu-jin-zhi-sheng-ji-bu-xiang-sheng-ji-de-bao-de-fang-fa
id: 42
updated: '2016-01-08 02:06:56'
date: 2016-01-08 00:21:47
---

> 由于 Arch Linux 采用滚动更新，最近php7.0也开始更新升级了，但是这会导致目前项目还是采用的mysql模块的程序来说真真是不适合升级的。所以，这里在网上查了一下降级安装的方法分享这里。

### 通过备份软件包降级安装
找到相应的php备份包，如果你最近没有执行 pacman -Scc以清空包缓存的话，应该在那儿)

```
ls -l /var/cache/pacman/pkg | grep php
```
如果在，你可以执行pacman -U ×××.pkg.tar.gz来安装旧版本。如果pacman提示文件冲突的话，你可以通过加上-f参数以强制执行，即
```
pacman -U --force ×××.pkg.tar.gz
```
这个过程会移除现有的包，仔细的计算所有依赖的改变，然后安装你选择的旧版本的软件包以及合适的依赖。

### 通过downgrade程序来自动化降级安装软件包
*在 AUR 中有一个包叫做downgradeAUR。它是一个简单的 Bash 脚本，它会从你的缓存中寻找旧版本的包，如果没有的话它会搜索 A.R.M.。你可以选择一个旧包来安装。它基本上自动化了上面所述的过程。查看 downgrade --help 获取使用方法的信息。*
```
sudo yaourt -S downgrade
downgrade -s php              ##搜索相关包版本
downgrade php                 ##降级安装包
```
### 如何恢复所有包到指定日期
如果想恢复所有包到指定日期（比如2014年3月30日），你必须如下例所示编辑 /etc/pacman.conf，从而让 pacman 保持在这个时间点并且直接使用指定的服务器：
```
[core]
SigLevel = PackageRequired
Server=http://ala.seblu.net/repos/2014/03/30/$repo/os/$arch

[extra]
SigLevel = PackageRequired
Server=http://ala.seblu.net/repos/2014/03/30/$repo/os/$arch

[community]
SigLevel = PackageRequired
Server=http://ala.seblu.net/repos/2014/03/30/$repo/os/$arch
```
或者如下例编辑 /etc/pacman.d/mirrorlist：
```
##                                                                              
## Arch Linux repository mirrorlist                                             
## Generated on 2042-01-01                                                      
##
Server=http://ala.seblu.net/repos/2014/03/30/$repo/os/$arch
```
然后同步包数据库以强制降级：
```
# pacman -Syyuu
```

### 禁止指定包自动升级的方法
==注意: 如果你改变了操作系统的一个基本的组件包，你也许需要降级许多包。这些软件包可能在过程中被删除，需要手动一点一点的安装回来；同时，后续升级时要小心，不要重新安装不想要的软件包版本。==


```
sudo vim /etc/rc.conf
```

添加行

IgnorePkg = php php-cgi php-gd

这样子,我们就可以禁止上面的三个包自动升级了.如果有其它的包想禁止,直接添加就可以了,记住分隔符要用空格哦.

[参考文档1](https://wiki.archlinux.org/index.php/Downgrading_packages_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
[参考文档2](https://wiki.archlinux.org/index.php/Arch_Linux_Archive_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
[引用](http://www.nenew.net/arch-linux-downgrade-install-packages-prevent-packages-update.html)
