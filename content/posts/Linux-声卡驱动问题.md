---
title: Linux 声卡驱动问题
tags: 
- linux 
- alsamixer
permalink: linux-sheng-qia-qu-dong-wen-ti
id: 60
updated: '2016-03-21 11:19:51'
date: 2016-03-21 01:54:57
---

> 最近转Gentoo，一切安装就绪了，但是想使用youtube观看视频的时候，竟没有声音，估计又得折腾一下了。

（Advanced Linux Sound Architecture，ALSA）是Linux中提供声音设备驱动的内核组件，用来代替原来的开放声音系统（Open Sound System，OSSv3）。

* 系统环境：Linux hcj.com 4.1.15-gentoo-r1
* 组件：alsa
* 前提：内核已经配置支持

### 硬件设备显示
```
lspci | grep -i audio
```
![](/content/images/2016/03/--_2016-03-21_22-46-22.png)

### 安装
```
euse -E alsa
emerge --ask --changed-use --deep @world
emerge --ask alsa-utils
```

### 启动声音服务
```
/etc/init.d/alsasound start
rc-update add alsasound boot      ###声音服务设置boot级别
```

### 列出设备名
```
cat /sys/class/sound/card*/id
```
![](/content/images/2016/03/--_2016-03-22_07-16-34.png)

### 配置默认设备
```
vi ~/.asoundrc
```
![](/content/images/2016/03/--_2016-03-22_07-17-59.png)

最后，别忘了重启一下。

[参考链接1](https://wiki.archlinux.org/index.php/Advanced_Linux_Sound_Architecture)

[参考链接2](https://wiki.gentoo.org/wiki/ALSA)
