---
title: Linux 终端下发送消息命令
tags: 
- linux 
- write
- wall
permalink: linux-zhong-duan-xia-fa-song-xiao-xi-ming-ling
id: 44
updated: '2016-01-18 21:41:22'
date: 2016-01-13 03:58:19
---

> 这里分享一个Linux服务器终端下发送消息的命令。由于平时工作中，必免不了与运维同事之间的信息交换，想到我们都是链的同一台服务器，这样就可以通过以下两个命令来发送消息。

### 给指定用户发送消息
首先，可使用w或who命令查看当前登录的用户信息；
然后，使用write命令将信息发送到用户的终端上，用法步骤如下：
```
$ w
 17:17:53 up 19 days, 57 min,  3 users,  load average: 0.00, 0.01, 0.05
USER     TTY        LOGIN@   IDLE   JCPU   PCPU WHAT
zq       pts/0     14:19    1:50   0.08s  0.02s -bash
hcaijin  pts/1     10:30   55.00s  7:05   0.23s -bash
hcaijin  pts/2     17:17    1.00s  0.01s  0.00s w

$ write hcaijin pts/1
Test send message.
```

然后使用hcaijin账号登录，且tty号为pts/1的登录用户终端会收到如下消息：
```
Message from hcaijin@hcj-arch on pts/2 at 17:18 ...
Test send message.
```
### 给当前所有用户发送消息
给当前登录所有用户发送消息（需要root权限），使用wall（write all的缩写）

```
$ wall 'Test send message to all user.'
```
执行wall命令，所有登录到该机器的控制台(console)界面上都会收到如上所示的消息。


[引用](http://www.cnblogs.com/gaojun/p/3387427.html)
