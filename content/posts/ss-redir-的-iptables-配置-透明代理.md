---
title: ss-redir 的 iptables 配置(透明代理)
tags: 
- iptables 
- shadowsocks
permalink: ss-redir-de-iptables-pei-zhi-tou-ming-dai-li
id: 57
updated: '2016-03-26 06:51:11'
date: 2016-03-09 08:22:01
---

>透明代理指对客户端透明，客户端不需要进行任何设置就使用了网管设置的代理规则
创建 

```
/etc/ss-redir.json 本地监听 1080 运行ss-redir -v -c /etc/ss-redir.json
```

### NAT表配置脚本

基本配置

```
iptables -t nat -N SHADOWSOCKS

# 在 nat 表中创建新链
iptables -t nat -A SHADOWSOCKS -p tcp --dport 23596 -j RETURN

# 23596 是 ss 代理服务器的端口，即远程 shadowsocks 服务器提供服务的端口，如果你有多个 ip 可用,但端口一致，就设置这个

iptables -t nat -A SHADOWSOCKS -d 123.456.789.111 -j RETURN
# 123.456.789.111 是 ss 代理服务器的 ip, 如果你只有一个 ss服务器的 ip，却能选择不同端口,就设置此条

iptables -t nat -A SHADOWSOCKS -d 0.0.0.0/8 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 10.0.0.0/8 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 127.0.0.0/8 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 169.254.0.0/16 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 172.16.0.0/12 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 192.168.0.0/16 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 224.0.0.0/4 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 240.0.0.0/4 -j RETURN

iptables -t nat -A SHADOWSOCKS -p tcp -j REDIRECT --to-ports 1080
# 1080 是 ss-redir 的监听端口,ss-local 和 ss-redir 的监听端口不同,配置文件不同

```

最后是应用上面的规则,将OUTPUT出去的tcp流量全部经过SOCKS链

```


#如果是在openwrt上实现透明代理的话,使用下面被注释了的规则

iptables -t nat -I PREROUTING -p tcp -j SHADOWSOCKS
# 在 PREROUTING 链前插入 SHADOWSOCKS 链,使其生效

在个人电脑上使用以下配置
iptables -t nat -A OUTPUT -p tcp -j SHADOWSOCKS
```
### 如果要过滤国内流量可以
列表太长了就不列出来了！

### 清除自定义规则

清空整个链 iptables -F 链名,比如:
```
iptables -t nat -F SHADOWSOCKS
```
删除指定的用户自定义链 iptables -X 链名 比如:
```
iptables -t nat -X SHADOWSOCKS
```
从所选链中删除规则 iptables -D 链名 规则详情 比如:
```
iptables -t nat -D SHADOWSOCKS -d 223.223.192.0/255.255.240.0 -j RETURN
```


### 解决DNS污染的问题
```
$ sudo pacman -S archlinuxcn/dnsmasq-china-list-git
$ sudo dnsmasq-update-china-list 114
####脚本如下：
#!/bin/bash

case "$1" in
  114)
    DNS=114.114.114.114
    ;;
  ali)
    DNS=223.5.5.5
    ;;
  cnnic)
    DNS=1.2.4.8
    ;;
  baidu)
    DNS=180.76.76.76
    ;;
  google)
    DNS=8.8.8.8
    ;;
  *)
    DNS=$1
esac

sed -i "s|^\(server.*\)/[^/]*$|\1/$DNS|" /etc/dnsmasq.d/accelerated-domains.china.conf
```


