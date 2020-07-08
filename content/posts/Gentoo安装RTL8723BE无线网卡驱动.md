---
title: Gentoo安装RTL8723BE无线网卡驱动
tags: 
- linux
- Gentoo 
- rtl8723be
permalink: an-zhuang-rtl8723bewu-xian-wang-qia-qu-dong
id: 59
updated: '2016-03-17 08:43:22'
date: 2016-03-17 07:40:43
---

>新配了个ThinkPad E55c 安装Gentoo时无线网卡没能正确识别。网上查了一下是3.15内核版本之前还没有包含这个驱动，需要手动安装一下。但是我想说我安装的内核版本是4.1,而且也安装了linux-firmware固件，可是怎么就没有这个驱动呢 - - 暂时先不深究。

* 内核版本：Kernel: x86_64 Linux 4.1.15-gentoo-r1
* 网卡型号：RTL8723BE

### 确认网卡型号
```
lspci -k | grep Network
```
![](/content/images/2016/03/--_2016-03-18_04-16-28.png)

### 下载源码
```
git clone https://github.com/lwfinger/rtlwifi_new.git
cd rtlwifi_new/
make && make install
```

### 手动加载模块
```
modprobe rtl8723be        ## 手动加载rtl8723be模块
modinfo rtl8723be         ## 查看模块详情
```

模块加载成功，使用`lspci -k`看一下,如果显示的是如下图说明无线网卡驱动安装成功：
![](/content/images/2016/03/--_2016-03-18_04-17-37.png)
最后`ip addr show `可以看到对应的网卡设备了
![](/content/images/2016/03/--_2016-03-18_04-30-58.png)

### 自启动加载模块
```
cat /etc/conf.d/modules
modules="rtl8723be"
```
[引用](https://wiki.archlinux.org/index.php/Wireless_network_configuration)
