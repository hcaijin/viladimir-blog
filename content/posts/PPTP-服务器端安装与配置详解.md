---
title: PPTP 服务器端安装与配置详解
tags: 
- pptp 
- iptables 
- vps
permalink: pptp-fu-wu-qi-duan-an-zhuang-yu-pei-zhi-xiang-jie
id: 25
updated: '2015-06-18 02:38:43'
date: 2015-06-17 05:44:35
---

## 安装前环境检查
因为pptp需要MPPE的支持，所以首先检测系统是否符已经编译了MPPE。
下面介绍两种检测方法，只要符合其中的一条就可以

### 第一种：

     # zgrep MPPE /proc/config.gz
     CONFIG_PPP_MPPE=y
     # cat /dev/net/tun
     cat: /dev/net/tun: File descriptor in bad state

### 第二种：
网上大多数资料还提到了另一个测试命令

    $ modprobe ppp-compress-18 && echo ok
    FATAL: Module ppp_mppe not found. 
如果返回“OK”说明可以安装PPTP，我查了一下，这个命令是在CentOS 4.4版本中有人提出的，但是经过实际测试，发现在我的环境中非但没有效果，而且报错。
所以**如果modprobe ppp-compress-18 && echo ok没有显示“OK”甚至报错，并不代表不能安装**。最好还是用上面那种方法查看。

### 安装依赖
由于pptp需要iptables支持，所以需要安装iptables。如果您的服务器上已经安装了iptables，那么可以只安装pptp

    $ yum install -y ppp iptables
注意：这里先安装的是ppp而不是pptp，不要打错了。另：PPP是一种数据链路层协议类似我们熟知的pppoe
接下来就是一大堆的信息，无非是寻找最快的源，找到后下载相关安装包，下载完成自动安装。
如果回到提示符状态，并且安装结果为Complete!，说明安装成功。


## 安装pptp
现在我们可以正式安装VPN Server了。这里我们选择pptp(VPN 协议的一种),因为简单，一条命令搞定。剩下的无非是一些配置。
    yum -y install pptpd

### 配置pptp

#### 编辑/etc/ppp/options.pptpd
pptpd安装完成后，编辑/etc/pptpd.conf文件，去掉下面两行的注释或者直接添加这两行(在文件的最后).这一步是配置ip地址的范围。

localip 192.168.0.1
remoteip 192.168.0.100-150
#### 设置使用pptp的用户名和密码
然后在/etc/ppp/chap-secrets文件中添加VPN用户，按照下面的格式,每个用户一行。

username pptpd password *

#### 配置DNS服务器
为了让你的用户连上VPN后能够正常地解析域名，我们需要手动设置DNS. 编辑/etc/ppp/options，找到ms-dns这一项，设置你的DNS.这里我推荐的是Google 最近发布的Public DNS,原因是因为好记。

ms-dns 8.8.8.8
ms-dns 209.244.0.3
ms-dns 208.67.222.222
ms-dns 8.8.4.4

#### 修改内核设置，使其支持转发。
编辑/etc/sysctl.conf文件，找到”net.ipv4.ip_forward=1″这一行，去掉前面的注释并注释掉 "net.ipv4.tcp_syncookies=1"。

    net.ipv4.ip_forward=1
    #net.ipv4.tcp_syncookies = 1
运行下面的命令让配置生效。

    $ sysctl -p


## iptables转发

    $ iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -j SNAT --to-source 12.34.56.78
(适合于OpenVZ架构的VPS,12.34.56.78为您VPS的公网IP地址)
     
    $ iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o eth0 -j MASQUERADE
(适合于XEN架构的VPS)
以上两条命令分别对应OpenVZ架构和XEN架构的VPS，您的VPS是什么架构需要询问供应商。Linode采用的是XEN架构，所以输入

我的是搬瓦工的vps,配置如下：

    $ iptables -t nat -A POSTROUTING -o venet0 -s 192.168.0.0/24 -j SNAT  --to-source `ifconfig  | grep 'inet addr:'| grep -v '127.0.0.1' | cut -d: -f2 | awk 'NR==1 { print $1}'`

## 运行

```
/etc/init.d/pptpd start
```
