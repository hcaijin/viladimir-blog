---
title: Nexus6p虚焊导致无限重启之救砖教程
date: 2020-07-03 14:44:06
tags:
- bootloop
- nexus6p
- android
- magisk
---


# 前言
> 前两天手机在使用过程中，无故死机，重启发现在logo显示画面停留一会儿又重启，后面就一直不断的循环这个过程。
> 本来以为重新做系统就好了，没想到再还原回去还是一样。我以为就这么变砖了，最后还是因为穷的力量，搜索nexus6p无限重启原因就发现原来这是由于前几天我手机摔过一次，可能导致cpu虚焊部分出了毛病，才导致现在手机无限重启的情况。
> 没办法，通过这几天各种google，终于找到网络上很多大神都有解决的方案，后面后提供大神帖子链接。

# 准备工作

* 备份当前的文件和数据
* 在手机上启用USB调试和OEM解锁
* 下载[ADB驱动程序](https://developer.android.com/studio/releases/platform-tools.html)，安装到你的PC机里。这里有个问题，要安装低版本的，大于29.0的版本再使用时报了个错误,提示找不到可用的命令，网上解决方法就是下载安装26.0版本的解决了这个问题。
* 去这里[下载](https://twrp.me/Devices/)符合nexus6p的TWRP，这里我下载了3.3.1版本的
* 去magisk官网下载最新版本应用包括manage
* 下载要用到的刷机固件[Lineage OS](https://androidfilehost.com/?w=files&flid=302684)，[原厂镜像 Nexus6p](https://developers.google.com/android/ota#angler)
* 下载我打包的要用到的boot.img

# 备份
> 由于已经进入不了系统了，备份需要链接USB进到REF里使用这个方法`adb pull sdcard`备份自己重要的数据

# 开始刷系统
> 这里要提一点，一开始我找到的方法是刷指定原厂镜像，刷写4核boot.img就可以重新进入系统。后面又找到不一定原厂镜像的版本，比如lingage17.1的镜像也可以通过大神的方法。
> 所以，这两种方法都可以解决无限重启进不了系统的问题，我都写在这里，只要看一种方法就好了。

## 第一种方法：刷原厂镜像
> 先说一下刷原厂镜像7.1.2版本号n2g48c,[参考文档](https://bbs.gfan.com/android-9270440-1-1.html),[英文原档](https://forum.xda-developers.com/nexus-6p/general/guide-fix-nexus-6p-bootloop-death-blod-t3640279)

* 具体怎么刷参考文档里已经写得很清楚了，主要说一下，如果没有刷过twrp的先刷这个`fastboot flash recovery twrp3_1_1_4Cores.img`
* 先线刷7.1.2, 具体线刷的操作，即进入TWRP界面，选Advanced再点sideload，滑块确认，等待pc端执行`adb sideload angler-ota-n2g48c-b004b71e.zip`
* 接下来刷入修改版本的boot:`adb flash boot 6p48C.img`
* 如果前面你先刷twrp就不用再刷一次了。
* 最后就是进入twrp卡刷性能包6pEX4_1_2.zip。具体操作：把zip包上传到手机sdcard,在twrp界面选择install指定的zip包安装，一路下一步就可以了。如果需要优化就得自己研究其他选项了。
* 以上就可以完美开机了。

## 第二种方法：刷第三方镜像lineage 17.1
> 理论上刷其他所有的镜像都可以使用这个方法重进系统。[参考文档](https://forum.xda-developers.com/nexus-6p/general/bootloop-death-blod-workaround-zip-t3819515)

* 下载重要的改写cpu核心的软件[N5x-6P_BLOD_Workaround_Injector_Addon-AK3-signed](https://androidfilehost.com/?w=files&flid=312881)
* USB链接手机，使用命令`adb reboot bootloader`重启
* 进入原先系统已经刷过twrp的可以直接进入twrp线刷lineage 17.1`adb sideload lineage-17.1-20200308-UNOFFICIAL-angler.zip` [参考文档](https://forum.xda-developers.com/nexus-6p/orig-development/rom-lineageos-17-0-nexus-6p-angler-t4012099)
* 刷写twrp `fastboot flash recovery twrp-3.3.1-0-fbe-4core-angler.img`
* 然后进入REC模式，上传zip包到sdcard,线刷Addon-AK3.signed.zip这个包`adb sideload N5X-6P_BLOD_Workaround_Injector_Addon-AK3-signed.zip`
* 最后重启，等待一会儿就可以进入系统了。

### 解决指纹功能bug问题
> 就上一篇文章，我再刷PixelExperience时已经提到过，使用magisk安装补丁的方法重新刷boot就好了。[参考文档](https://topjohnwu.github.io/Magisk/install.html)

* 上传boot-17.img到手机`adb push boot-17.img sdcard/`
* 在手机里安装最新版本的magisk manager
* 打开magisk点击安装->安装->选择要安装的boot-17.img
* magisk manager 将生成新的文件magisk_patched.img到sdcard/Download/
* 下载这个补丁到PC机`adb pull /sdcard/Download/magisk_patche.img`
* 最后重启进入bootloader刷写这个补丁`fastboot flash boot magisk_patched.img`
* 以上完成就可以重进系统使用指纹的功能了。

# 总结
> 其实这两个方法最主要是线刷的zip包不同，第二种方法里的AK3包实现了任何系统修改cpu4核心的方法。

# 下载
> 我打包了一下所有要用到的软件（不包括原厂镜像包，这些都可以去官网下载)，防止原链接失效。[备份包](https://drive.google.com/file/d/1Bd4nASTMve23qh28x3BR3OJWJp2NZJyI/view?usp=sharing)
