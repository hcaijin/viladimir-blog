---
title: Arch Linux 内核更新 修复无线模块rtl8821ce编译失败的问题
date: 2018-05-19 01:36:07
tags:
- rtl8821ce
- linux
---


> 最近更新系统，内核从4.15 更新到了 4.16.9发现原来的无线模块编译不通过，找不到头文件stdarg.h

# 查看无线驱动信息

通过`ip l`可以看到只有有线网卡
```

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp3s0: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel state DOWN mode DEFAULT group default qlen 1000
    link/ether 8c:16:45:3f:68:0d brd ff:ff:ff:ff:ff:ff

```

查看无线网卡驱动，找到相应的驱动去下载就好了
```
lspci | grep -i 'newwork'

Network controller: Realtek Semiconductor Co., Ltd. RTL8821CE 802.11ac PCIe Wireless Network Adapter

```

# 下载无线驱动源码
```
git clone https://github.com/endlessm/linux
```
由于这个项目特别的大，这里只需要下载drivers/net/wireless/rtl8821ce

# 编译

## 修改Makefile

这里需要修改Makefile中TopDIR变量的值为当前路径，否则会提示错误退出
```
cd drivers/net/wireless/rtl8821ce
sed -i 's/export TopDIR ?=/export TopDIR ?= $(shell pwd)/g' Makefile
```
## 执行`make` 
在最新的内核版本（4.16.9-1-ARCH）下编译失败，提示如下：
```
graz@graz ~/Source/driver_net_wireless/rtl8821ce % make
/usr/bin/make ARCH=x86_64 CROSS_COMPILE= -C /lib/modules/4.16.9-1-ARCH/build M=/home/graz/Source/driver_net_wireless/rtl8821ce  modules
make[1]: Entering directory '/usr/lib/modules/4.16.9-1-ARCH/build'
  CC [M]  /home/graz/Source/driver_net_wireless/rtl8821ce/core/rtw_cmd.o
In file included from ./include/linux/list.h:9,
                 from ./include/linux/module.h:9,
                 from /home/graz/Source/driver_net_wireless/rtl8821ce/include/basic_types.h:81,
                 from /home/graz/Source/driver_net_wireless/rtl8821ce/include/drv_types.h:31,
                 from /home/graz/Source/driver_net_wireless/rtl8821ce/core/rtw_cmd.c:22:
./include/linux/kernel.h:6:10: fatal error: stdarg.h: No such file or directory
 #include <stdarg.h>
          ^~~~~~~~~~
compilation terminated.
```

通过`locate stdarg.h`找到头文件 "/usr/lib/gcc/x86_64-pc-linux-gnu/8.1.0/include/stdarg.h"
```
ln -s /usr/lib/gcc/x86_64-pc-linux-gnu/8.1.0/include/stdarg.h include/
```

软链接创建好后，就可以执行`make`编译成功

## 安装
```
sudo make install
modprobe 8821ce
```
最后，没有报错的话，通过`ip l` 就可以找到这个无线网卡了
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp3s0: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel state DOWN mode DEFAULT group default qlen 1000
    link/ether 8c:16:45:3f:68:0d brd ff:ff:ff:ff:ff:ff
3: wlp5s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DORMANT group default qlen 1000
    link/ether 70:c9:4e:d8:6d:01 brd ff:ff:ff:ff:ff:ff
```
