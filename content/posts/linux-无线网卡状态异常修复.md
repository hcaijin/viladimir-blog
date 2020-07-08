---
title: linux 无线网卡状态异常修复
tags: 
- linux 
- wifi 
- RF-kill
permalink: jie-jue-linux-wu-xian-wang-qia-zhuang-tai-yi-chang
id: 27
updated: '2015-07-17 13:30:00'
date: 2015-07-17 12:43:40
---

无线网卡异常，有很多种原因造成。这里我是因为有做AP共享WIFI，本来好好的，不知道怎么实然，我手机wifi断了，再看，从PC共享的WIFI竟然挂了。
就这样，下班回家，准备链接无线路由，因为有换新的网，得重新链接：

     $ sudo wifi-menu -o
竟然报No networks found 

重启了问题依然存在，没办法，只能用有线先连着。
开始Google：No networks found
我一开始就找错了方向，自然是没有找到解决的方法。

经过这次，让我对日志的记录有了更深的认识，以前，觉得日志记录太难看得懂了，从来都是很少去看，浪费了很多时间在无用的地方，这里给自己个警戒，凡是就得先看日志信息。

这样，接下来，在日志里有发现有这个错误 Operation not possible due to RF-kill 
Google一下，就找到了问题的关键，原因RF-KILL其实是一个打开和关闭无线设备的工具。 由此可以知道，这是一打开无线设备wifi的错误。
因为我用的arch没有安装rfkill,执行：

     $ sudo pacman -S rfkill
安装好以后，为了查看当前的无限网卡的状态，执行命令rfkill list all  ——列出所有无线设备的当前状态。结果如下：
![](/content/images/2015/07/--_2015-07-18_01-13-01.png)

发现 Hard blocked 和 soft blocked 之间的同步失败,具体原因可以看这里[“SIOCSIFFLAGS: Operation not possible due to RF-kill”?](http://askubuntu.com/questions/62166/siocsifflags-operation-not-possible-due-to-rf-kill)

找到问题原因，我们可以使用命令使软硬设备同步:
   
    $ rfkill unblock wifi  
用命令 rfkill list all 列出所有无线设备状态看一下结果，
![](/content/images/2015/07/--_2015-07-18_01-24-26.png)

好了，无线网卡状态异常的问题修复了。
