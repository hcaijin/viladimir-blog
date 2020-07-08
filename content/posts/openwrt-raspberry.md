---
title: 树莓派安装配置软路由 
date: 2019-07-17 15:43:15
tags:
- openwrt
- raspberry
---

# 准备

* Raspberry Pi 2 Model B Rev 1.1
* SD卡闪盘(4G+)
* 主机我用的是ArchLinux,内核4.19.52-1-lts
* 主路由器一个TPlink
* USB网卡
* 一条网线

# 刷openwrt固件

## 下载解压 
去[官网](https://www.openwrt.org)下载对应的安装固件

```
wget http://downloads.openwrt.org/releases/18.06.4/targets/brcm2708/bcm2709/openwrt-18.06.4-brcm2708-bcm2709-rpi-2-ext4-factory.img.gz
gzip -d openwrt-18.06.4-brcm2708-bcm2709-rpi-2-ext4-factory.img.gz
```
## 烧写SD卡
```
sudo fdisk -l   #列出闪盘设备名 /dev/mmcblk0
sudo dd if=openwrt-18.06.4-brcm2708-bcm2709-rpi-2-ext4-factory.img of=/dev/mmcblk0

```
## 链接树莓派

 配置主机IP(192.168.1.1) 网线直接连接树莓派接口 浏览器打开192.168.1.1 配置密码和ssh登陆以后，在终端里登陆`ssh root@192.168.1.1`

修改lan口配置如下：
```
cat /etc/config/network

config interface 'lan'
        option type 'bridge'
        option ifname 'eth0'
        option proto 'static'
        option netmask '255.255.255.0'
        option ip6assign '60'
        option ipaddr '192.168.10.109'
        option gateway '192.168.10.1'
        option broadcast '255.255.255.0'
        option dns '114.114.114.114'
```

重启路由，用网线连接主路由器就可以链接外网了。

## 安装软件

安装支持网卡的模块
```
opkg update
opkg install kmod-rtl8192cu
```

### 修改无线网络配置
```
cat /etc/config/wireless

config wifi-iface 'default_radio0'
        option device 'radio0'
        option network 'lan'
        option mode 'ap'
        option encryption 'none'
        option ssid 'CMCCFREE'
```

### 修改路由网络配置
```

cat /etc/config/network

config interface 'loopback'
        option ifname 'lo'
        option proto 'static'
        option ipaddr '127.0.0.1'
        option netmask '255.0.0.0'

config globals 'globals'
        option ula_prefix 'fd3c:ca1e:c593::/48'

config interface 'lan'
        option proto 'static'
        option netmask '255.255.255.0'
        option ipaddr '192.168.10.1'
        option ip6assign '64'

config interface 'wan'
        option ifname 'eth0'
        option proto 'dhcp'
        option dns '127.0.0.1' #### 连接该 OpenWrt 路由器的所有设备发出的 DNS 请求都会由该路由器的 dnsmasq 来响应（当然，前提是设备没有手动去修改默认的 DNS 服务器 IP，而使用路由器默认提供的 DNS 服务器 IP）。
        option peerdns '0' #### 忽略通告的DNS服务器地址

```

### 修改DHCP配置如下

```
root@OpenWrt:~# cat /etc/config/dhcp

config dnsmasq
        option domainneeded '1'
        option localise_queries '1'
        option rebind_protection '1'
        option rebind_localhost '1'
        option domain 'lan'
        option expandhosts '1'
        option authoritative '1'
        option readethers '1'
        option leasefile '/tmp/dhcp.leases'
        option nonwildcard '1'
        option localservice '1'
        option logqueries '1'
        option noresolv '1'
        option local '/lan/'
        option allservers '1'
        list server '114.114.114.114' #### 配置DNS转发 注意，此为路由器默认查询的 DNS 服务器，你可以根据你的实际情况选择一个较快的 DNS 服务器

config dhcp 'lan'
        option interface 'lan'
        option dhcpv6 'server'
        option ra 'server'
        option ra_management '1'

config dhcp 'wan'
        option interface 'wan'
        option ignore '1'
```

### 提交修改并重启
```
uci commit /etc/config/wireless
uci commit /etc/config/network
uci commit /etc/config/dhcp

## 重启路由也是一样的或者执行以下命令重启服务
/etc/init.d/dnsmasq restart
/etc/ini.d/network restart
ifup wan && wifi
```

# 安装SS

## 安装 shadowsocks-libev 相关的包
```
opkg update
opkg install shadowsocks-libev-config shadowsocks-libev-ss-local shadowsocks-libev-ss-redir shadowsocks-libev-ss-rules shadowsocks-libev-ss-tunnel luci-app-shadowsocks-libev
```

## 基础配置
进入openwrt web配置界面，选择 Service->shadowsocks-libev

点击 Remote Servers, 里面已经默认配置一个服务器 sss0，修改地址，端口，密码，加密方式，最重要的，将disabled的勾去掉，点击 save&apply 按钮。

再点击 Local Instances, 点击 ss-local.cfgXXXXX (XXX为随机数字)条目对应的Disabled 按钮，将其变成 Enabled，点击 Save & Apply。配置保存生效以后, ss-local.cfgXXXX 条目的Running 状态由no变为yes。这时，路由器上已经运行一个SOCKS5服务器，端口1080。设置电脑浏览器的代理服务器为路由器地址，端口1080，尝试访问谷歌，如果成功则说明ss客户端在openwrt上工作一切正常。

接着要测试iptables+ss-redir自动转发代理(透明代理)的功能，在Local Instances中，将ss-redir.hi 设为Enabled。再点击 Redir Rules, Disabled勾去掉，点击Destination Settings，dst default 由bypass改为 forward。点击Save&Apply 使配置生效。将电脑浏览器的代理设置取消，访问谷歌，如果成功，则说明无条件透明代理设置成功。所有数据包都由路由器转发到ss服务器了。


## 3. 进阶配置
上面最后配置的透明代理将全部流量都转发到远端SS服务器，显然太浪费流量，而且国内的网站去国外转一圈效率也很低。因此我们需要在路由器上识别国内国外流量，区别对待。

1.首先将 Destination Settings中的dst forward 设为 bypass。

2. 将opkg列表更新由http 改为https, http存在更新不全的情况，可能是GFW搞鬼

```
opkg install libustream-mbedtls (如果提示找不到，opkg update 多运行几次)
sed -i s/http:/https:/g /etc/opkg/distfeeds.conf
opkg update
```
3. 安装各种依赖包

```
opkg remove dnsmasq
opkg install dnsmasq-full
opkg install coreutils-base64 curl ca-certificates ca-bundle
```

# 问题总结

## dnsmasq服务不启动
最早手动添加域名到 /etc/dnsmasq.d/下的配置文件中，经过测试，发现无法解析该域名，经过排查，可能是配置文件编码格式的问题，导致了dnsmasq服务无法启动

## 主机路由地址
排查问题我们在主机上用`nslookup`
```
graz@graz ~ % nslookup twitter.com
Server:		192.168.1.1
Address:	192.168.1.1#53

Non-authoritative answer:
Name:	twitter.com
Address: 104.244.42.65
Name:	twitter.com
Address: 104.244.42.1
```
如上，看server项，是上上级的路由地址，这就绕过了树莓派的路由直接走上级路由查询。

然后，我们再看一下`cat /etc/resolv.conf`
可以看到有多个路由地址，包括`192.168.10.1`，`192.168.1.1`

以上，主机路由地址获取到了上上级的路由，也不知道怎么回事，(有知道的可以邮件通知我一下，谢谢大家)，只能先手动修改一下主机路由如下:

```
cat /etc/resolv.conf
nameserver 192.168.10.1 ## 这里是树莓派路由器的路由地址，一开始获取的还有上上级的路由地址如：192.168.1.1 。要把这个删掉
```

# 最后

最后，查看日志或者安装tcpdump抓取5353端口看是否正常配置
```
opkg update 
opkg install tcpdump
tcpdump -vv -i lo port 5353
```

# 参考

* [OpenWRT Shadowsocks+GFWList 流量自动分流](http://blog.kompaz.win/2017/03/24/OpenWRT%20Shadowsocks+GFWList%20%E6%B5%81%E9%87%8F%E8%87%AA%E5%8A%A8%E5%88%86%E6%B5%81/)
* [openwrt 18.06.1 配置科学上网](https://medium.com/@cnnbysy/openwrt-18-06-1-%E9%85%8D%E7%BD%AE%E7%A7%91%E5%AD%A6%E4%B8%8A%E7%BD%91-30e231958c38)
* [Shadowsocks + OpenWRT + dnsmasq-full + ipset + gfwList 实现路由器](https://swsmile.info/2019/01/30/%E3%80%90Network%E3%80%91Shadowsocks%20-Shadowsocks-OpenWRT-dnsmasq-full-ipset-gfwList-%E5%AE%9E%E7%8E%B0%E8%B7%AF%E7%94%B1%E5%99%A8%EF%BC%88%E5%B0%8F%E7%B1%B3%E8%B7%AF%E7%94%B1%E5%99%A8mini%EF%BC%89%E8%87%AA%E5%8A%A8%E7%BF%BB%E5%A2%99/)
* [新的基于OpenWrt路由器的（不完全）自动翻墙方案](https://blog.ch3n2k.com/posts/xin-de-ji-yu-openwrt-lu-you-qi-de-bu-wan-quan-zi-dong-fan-qiang-fang-an.html)
