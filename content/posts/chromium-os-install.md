---
title: Chromium OS源码编译、U盘安装及使用笔记
date: 2018-06-02 23:41:10
tags:
- ChromiumOS
- archlinux
---


> 根据官方文档 https://www.chromium.org/chromium-os/quick-start-guide 只有Ubuntu Trusty版本的安装方式写了个ArchLinux的安装方法

# 安装依赖

* Arch Linux  4.16.12-1-ARCH
* x86_64 GNU/Linux
* 有sudo权限的用户

## 基本依赖 
确保有如下包就好了，没有就用`pacman -S` 安装就是：

```
sudo pacman -S git-core gitk git-gui subversion curl lvm2 thin-provisioning-tools python-pkg-resources python-virtualenv python-oauth2client
```

## 安装depot_tools

用git克隆下来就好了,但要注意python的版本，后面会说.
```
cd ~/Source/
git clone https://chromium.googlesource.com/chromium/tools/depot_tools
```
确保deport_tools目录在PATH变量里

## sudoers配置

要设置Chrome操作系统构建环境，应该关闭sudo的tty_tickets选项，因为它与cros_sdk不兼容。执行如下操作：
```
cd /tmp
cat > ./sudo_editor <<EOF
#!/bin/sh
echo Defaults \!tty_tickets > \$1          # Entering your password in one shell affects all shells 
echo Defaults timestamp_timeout=180 >> \$1 # Time between re-requesting your password, in minutes
EOF
chmod +x ./sudo_editor 
sudo EDITOR=./sudo_editor visudo -f /etc/sudoers.d/relax_requirements
```

# 获取源码
创建一个目录来保存源文件“${SOURCE_REPO}”。
```
export SOURCE_REPO="~/Source/chromiumos"
mkdir ${SOURCE_REPO}
cd ${SOURCE_REPO}
virtualenv -p /usr/bin/python2 venv			#这里我们要把python环境切换为2.7，才能使用下面的repo
repo init -u https://chromium.googlesource.com/chromiumos/manifest.git

# Optional: Make any changes to .repo/local_manifests/local_manifest.xml before syncing
repo sync
```

# 创建chromiumos

## 构建包
```
export BOARD=amd64-generic
cros_sdk -- ./build_packages --board=${BOARD}
```

## 构建镜像
```
cros_sdk -- ./build_image --board=${BOARD}
```

## 烧入USB
键入 `sudo fdisk -l` 查看插入U盘所在区域，然后执行如下操作烧录编译的系统到U盘
```
cros_sdk -- cros flash usb:///dev/sdd ~/chromiumos/src/build/images/amd-generic/latest/chromiumos_test_image.bin
```

## 修改分区
如果要使用自定义大小容量的分区构建镜像，请考虑在 build_library/legacy_disk_layout.json 中添加新的磁盘布局或使用 adjust_part。请参阅下面的帮助，

```
adjust_part ='STATE：1G' ---- 将1GB添加到状态分区
adjust_part ='ROOT-A：-1G' ---- 从主rootfs分区中删除1GB
adjust_part ='STATE：= 1G' --- 设置状态分区为1GB
```
这里键入 `cros_sdk -- ./build_image --board=${BOARD} --noenable_rootfs_verification test --adjust_part='STATE:+10G'`，这样我们的Chromium OS用户空间便增加10G，如果使用默认设置你会发现用户空间容量不足（约140MB）


# 最后
修改要安装到目标机器的bios启动项为U盘启动，插入U盘，启动。

进入系统，按Ctrl + Alt + Back（F2）。在提示符下输入chronos并使用以下命令进行安装。
```
/usr/sbin/chromeos-install
```
