---
title: Nexus6p Root提权以及刷机攻略
date: 2020-03-25 18:00:20
tags:
  - android
  - nexus6p
  - twrp
  - magisk
permalink: root-nexus6p-use-twrp-magisk
---

> Android 10.0也出来一段时间了，想着给自己的手机刷一下。所以就有了这个攻略，记录一下过程。

# 准备工作

* 确保设备电池电量超过50%
* 备份当前的文件和数据
* 在手机上启用USB调试和OEM解锁
* 下载ADB驱动程序，安装到你的PC机里
* 去这里[下载](https://twrp.me/Devices/)符合nexus6p的TWRP，这里我下载了3.3.1版本的
* 去magisk官网下载最新版本应用包括manage
* 下载刷机的固件[PixelExperience 10](https://download.pixelexperience.org/angler)

# 使用ADB和Fastboot解锁Bootloader

打开终端运行如下命令重启手机到fastboot模式，[参考](https://developers.google.com/android/images)
```
adb reboot bootloader
fastboot flashing unlock
```
完成以后，就成功解锁了Bootloader，再次检查以启用“开发人员选项”，然后转到“开发人员选项”并启用USB调试模式OEM解锁。有时，他们在启动后会自行禁用。

# 安装TWRP

还是一样先进到启动模式，把TWRP镜像刷进recovery,如下
```
adb reboot bootloader
fastboot flash recovery twrp-3.3.1-0-angler.img
```

> 完成上面两个准备刷机的步骤基本上就不用担心手机变砖了，下面开始刷机

# 刷PixelExperience 10 到Nexus6p

首先确保Nexus6p没有刷过其他第三方Rom，如果刷过，要先通过官方Rom还原到指定[版本](https://developers.google.com/android/images#angler)
这里我们需要下载8.1.0 (OPM7.181205.001, Dec 2018)版本，解压进入文件夹执行shell就可以还原了。当然首先还得USB链接成功。

## 刷入Pixel 10固件

确保nexus6p是最新版本8.1.0以后，我们就可以通过TWRP刷入Pixel这个最新的固件了

重启手机，进入手机端的TWRP界面，点选Advanced选项，然后在手机界面里点选ADB sideload选项，然后滑动下面的滑块确认选择，等待电脑端执行adb sideload 刷机包名.zip命令

这里有一点要注意，下载固件的时候要把版本10的两个文件都下载了，因为最新的那版本指纹，锁屏功能有问题。

```
unzip -x PixelExperience_angler-10.0-20191231-1607-OFFICIAL.zip -d piexl2019
unzip -x PixelExperience_angler-10.0-20200101-0925-OFFICIAL.zip -d piexl2020
cp piexl2019/boot.img piexl2020
cd piexl2020
zip -r -v -o pixel20200101.zip .
```
完成以上，生成我们要刷入的pixel20200101.zip这个固件包了，可以进入TWRP界面三清数据以后线刷就OK了
这里的三清指的是擦除system & data & cache 这三个分区，还有格式化data分区
最后执行以下，重启就能进入新系统了
```
adb sideload pixel20200101.zip
```
到这里，如果不需要Root手机的话就不用看下面的了

# Root提权
[参考](https://www.youtube.com/watch?v=3pxOeiIBrHI&t=582s)
首先进到新系统（别忘了还是要先开启开发者选项）安装magisk manager这个app,把上一步解压出来的boot.img复制到手机里
```
adb push boot.img /sdcard/
```
操作magisk manager安装magisk的补丁，生成magisk_patched.img
把补丁下载回PC
```
adb pull /sdcard/Download/magisk_patched.img .
```

重启手机进入快速启动模式,把补丁刷入boot
```
fastboot flash boot magisk_patched.img
```
最后重启手机，进入magisk manager检查root权限。

# 参考

* [ PixelExperience for Google Nexus 6P 【angler】](https://forum.xda-developers.com/nexus-6p/development/rom-pixel-experience-t3970525)
* [【ROM】【UNOFFICIAL】LineageOS 17.1 for Nexus 6P (angler)](https://forum.xda-developers.com/nexus-6p/orig-development/rom-lineageos-17-0-nexus-6p-angler-t4012099)
* [【教程】一加5刷LineageOS全过程](https://mikey2008.github.io/2019/12/31/%E3%80%90%E6%95%99%E7%A8%8B%E3%80%91%E4%B8%80%E5%8A%A05%E5%88%B7LineageOS%E5%85%A8%E8%BF%87%E7%A8%8B/)
