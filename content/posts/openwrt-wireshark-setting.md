---
title: 用OPENWRT路由器抓包网络流量笔记
date: 2018-06-13 19:50:11
tags:
- openwrt
- wireshark
- cloudshark
---

# 前言
openwrt是一款基于linux的路由器系统，可以安装很多相关的工具包，完成像linux系统服务器可以完成的工作，比如今天我们要讲的路由器的网络数据包抓包。

# 环境

- 安装了openwrt的路由器，ip地址：192.168.10.1
- 要抓包流量的Android手机，ip地址：192.68.10.235
- 工作台笔记本，ip地址：192.168.10.234


# 安装与配置
> 以下介绍两种方法都可以实现路由器数据包抓取的功能

***简单的说明
根据openwrt文档，所有的局域网的数据最后都是通过br-lan虚拟网卡来做转发，所以对此网卡进行监控即可
此命令本质是远程在路由器上执行网络监控命令，输入文本到本机的wireshark里面
使用wireshark作为可视化工具来查看***

## 捕获与tcpdump的通信
> Tcpdump可以安装在OpenWrt路由器上。因此，这种方法消除了让远程Wireshark或类似听众实时分析流量的需要。

ssh登陆到openwrt(默认端口：22)，更新并安装tcpdump
```
opkg update
opkg install tcpdump
```
执行以下命令在接口（-i）上侦听并将捕获的信息存储到文件（-w），并在执行此操作时（-v）进行冗长操作。
```
tcpdump -i any -v -w pcap.cap
```
生成的pcap.cap文件，我们可以传回工作台，用wireshark打开做进一步的分析

> 以下是一些使用tcpdump的例子：

- https://www.rationallyparanoid.com/articles/tcpdump.html

### 制作一键命令脚本

命令格式如下：
```
ssh -p ssh端口 -o StrictHostKeyChecking=no ssh用户名@ssh地址 'tcpdump -s 0 -U -n -w - -i br-lan not port ssh端口' | wireshark -k -i -
```
由于我环境配置了不用密码登陆的方式所以我们可以直接写成如下：
```
ssh openwrt 'tcpdump -s 0 -U -n -w - -i br-lan not port 22' | wireshark -k -i -
ssh -p 22 -o StrictHostKeyChecking=no root@192.168.10.1 'tcpdump -s 0 -U -n -w - -i br-lan not port 22' | wireshark -k -i -
```

> 前面讲述了基本的原理和操作手段，但是缺点是每次都需要输入长串命令行和密码，可以利用linux的一些小操作技巧，简化此过程，做成一个命令工具，方便随时调用。
基本原理：
- 使用 sshpass 工具来做密码输入
- 使用 alias 别名来做成命令语句

在工作台安装sshpass，执行以下脚本：
```
sudo pacman -S sshpass
sshpass -p 'password' ssh -p 22 -o StrictHostKeyChecking=no root@192.168.10.1 'tcpdump -s 0 -U -n -w - -i br-lan not port 22' | wireshark-gtk -k -i -
```
把执行语句写到`.bash_rc`就可以一条命令执行抓包分析了
```
alias tsharkbyopenwrt="sshpass -p 'password' ssh -p 22 -o StrictHostKeyChecking=no root@192.168.10.1 'tcpdump -s 0 -U -n -w - -i br-lan not port 22' | wireshark-gtk -k -i -"
```

### 完善脚本
通过命名管道来导回数据
```
mkfifo /tmp/fifo
sshpass -p 'passwrod' ssh openwrt 'tcpdump -s 0 -U -n -w - -i br-lan not port 22' > /tmp/fifo & wireshark-gtk -k -i /tmp/fifo
```
*这里我配置了`.ssh/config`，所以可以直接使用`ssh openwrt`命令代替前面指定端口与用户名的方式。*

> 我们还有一个方法可以不用安装sshpass,直接使用密钥的方式来登陆路由器抓包,以上就可以写为：
```
ssh-copy-id openwrt
ssh openwrt 'tcpdump -s 0 -U -n -w - -i br-lan not port 22' > /tmp/fifo & wireshark-gtk -k -i /tmp/fifo
```

## 使用远程Wireshark侦听器进行分析
ssh登陆到openwrt(默认端口：22)，更新并安装iptables-mod-tee
```
opkg update 
opkg install iptables-mod-tee
```
运行以下iptables命令以“在输出接口（-o）上将源IP（-s）的每个数据包的副本转发到网关IP（ - 网关）”
```
iptables -A POSTROUTING -t mangle -o br-lan ! -s 192.168.10.235 -j TEE --gateway 192.168.10.234
```
运行以下iptables命令以“在接口（-i）上将目的IP（-d）的每个数据包的副本转发到网关IP（ - 网关）”
```
iptables -A PREROUTING -t mangle -i br-lan ! -d 192.168.10.235 -j TEE --gateway 192.168.10.234
```
在Wireshark上开始捕获流量并应用下面的过滤器：
```
(ip.src == 192.168.9.121) || (ip.dst == 192.168.9.121)
```

关于iptable规则一些有用的资源：

- https://wiki.openwrt.org/doc/howto/netfilter
- http://www.faqs.org/docs/iptables/index.html
- http://ipset.netfilter.org/iptables-extensions.man.html#lbDW


## 安装使用CloudShark
CloudShark是一个独立的，与LEDE无关的云分析平台。它依靠cshark插件远程发送数据包进行分析。请检查您的内部规则，是否允许将网络流量发送到云平台。
```
opkg update
opkg install cshark luci-app-cshark
```
请查看[Cloud Shark文档](https://support.cloudshark.org/openwrt/openwrt-cloudshark.html)以获取更多信息。

# 问题处理
在工作台使用wireshark-gtk的时候报错：

```
Couldn't run /usr/bin/dumpcap in child process: Permission denied
```
这是由于dumpcap这个的权限问题。
```
ll /usr/bin/dumpcap                                                                  :(
-rwxr-xr-- 1 root wireshark 102K May 23 06:39 /usr/bin/dumpcap
```
只要把当前用户加入到wireshark用户组里，重启就ok了（暂时不明为什么一定要重启，反正我是重启以后才正常使用的)。
```
sudo usermod -aG wireshark $LOGNAME
sudo setcap cap_net_raw,cap_net_admin+eip /usr/bin/dumpcap
```
*setcat对应使用getcap查看当前的方法权限*

# 参考
- [ANALYZING NETWORK TRAFFIC WITH OPENWRT](http://www.ayomaonline.com/security/analyzing-network-traffic-with-openwrt/)
- [How to capture, filter and inspect packets](https://openwrt.org/docs/guide-user/firewall/capture-filter-inspect-packets)
- [GETTING STARTED WITH OPENWRT – LINUXFYING ROUTERS](http://www.ayomaonline.com/security/getting-started-with-openwrt-linuxfying-routers/)
