---
title: Arch Linux 配置Bluetooth
tags:
- archlinux
- bluetooth 
permalink: arch-linux-pei-zhi-bluetooth
id: 29
updated: '2015-08-19 05:37:35'
date: 2015-08-19 03:22:07
---

>蓝牙是一个短距离无线通信标准，适用于手机、计算机和其他电子设备之间的通信。在 Linux 中，通常使用的蓝牙协议栈实现是 BlueZ.

## 安装
为了使用蓝牙(Blutooth)，必须要安装官方仓库中的 bluez 软件包。

    $ sudo pacman -S  bluez bluez-utils 

bluez会使用dbus 服务读取设置和进行pin(个人识别码 personal identification number)配对。蓝牙(Bluetooth)协议需要bluetooth服务来支撑：
    
    $ sudo systemctl enable bluetooth.service
    $ sudo systemctl start bluetooth.service

加载通用蓝牙驱动程序，如果还没有装载：

    $ modprobe btusb

## 配置
### bluetoothctl
使用bluetoothctl 需要安装bluez-utils 包。这个在上一步里已经执行过了，这里就直接说配置与使用的方法：

    $ bluetoothctl 

help 列出帮助文档。
![](/content/images/2015/08/--_2015-08-19_15-31-47.png)

* 首先执行 `power on` 打开蓝牙适配器的电源开关，这样蓝牙的LED灯会亮起(默认是关闭的)。
* 先用命令 `devices` 列出有已匹配的设备MAC地址
* 如果列表为空，那么就要使用 `scan on` 来扫描网络中开启蓝牙的设备
* 开启代理 `agent on`
* 输入命令 `pair [MAC Address] ` 匹配两个蓝牙设备
* 如果使用设备没有PIN，要成功地重新连接设备之前，可能需要手动信任设备。输入 `trust [MAC Address] ` 
* 如果以上步骤都没有问题的话，那么就可以链接你的蓝牙设备了。 `connect [MAC Address]`

### Obex
使用obexctl是在蓝牙设备之间发送和接收文件的工具，是随bluez包安装好了的,在终端直接输入 `obexctl` 就可以进入obex环境，如图：
![](/content/images/2015/08/--_2015-08-19_16-41-12.png)

另外还有一个命令行工具obexfs (包括obexfs,obexftp等)

    $ sudo pacman -S obexfs
安装好了就可以使用命令直接挂载蓝牙设备到本地目录，

    $ obexfs -b MAC_address_of_device -p /mnt/bluez/

一旦你完成了，卸载的设备使用以下命令：

    $ fusermount -u /mnt/bluez/

如果您的设备支持FTP服务，但你不希望加载该设备，您可以使用obexftp传输文件在设备之间。
发送文件：

    $ obexftp -b MAC_address_of_device -p /path/to/file

接收文件：

    $ obexftp -b MAC_address_of_device -g filename

### Bluetooth USB 适配器
如果你在使用USB适配器，你应当确认你的适配器被正确识别。你可以在插入适配器时通过查看/var/log/messages.log （或者journalctl -f)，

    $ tail -f /var/log/messages.log
这应当会出现类似于下面所示的信息：

May  2 23:36:40 tatooine usb 4-1: new full speed USB device using uhci_hcd and address 9
May  2 23:36:40 tatooine usb 4-1: configuration #1 chosen from 1 choice
May  2 23:36:41 tatooine hcid[8109]: HCI dev 0 registered
May  2 23:36:41 tatooine hcid[8109]: HCI dev 0 up
May  2 23:36:41 tatooine hcid[8109]: Device hci0 has been added
May  2 23:36:41 tatooine hcid[8109]: Starting security manager 0
May  2 23:36:41 tatooine hcid[8109]: Device hci0 has been activated

如果你只得到了前面两行，说明了电脑发现了这个设备，但是你需要手动启动它。 例如：
      
     $ hciconfig -a hci0
     $ hciconfig hci0 up

如果不能从你的手机发现电脑，那么就需要启用PSCAN和ISCAN：
 
     $ hciconfig hci0 piscan 

注意: 检查/etc/bluetooth/main.conf中的发现倒计时和配对倒计时
试着在 /etc/bluetooth/main.conf 改变设备的class
![](/content/images/2015/08/--_2015-08-19_17-23-20.png)
修改：Class = 0x100100

   

==参考：https://wiki.archlinux.org/index.php/Bluetooth ==
