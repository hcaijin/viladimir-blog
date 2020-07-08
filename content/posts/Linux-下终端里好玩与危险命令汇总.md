---
title: Linux 下终端里好玩与危险命令汇总
tags:
- linux
- bash
permalink: linux-xia-hao-wan-de-ming-ling
id: 43
updated: '2016-01-12 22:10:22'
date: 2016-01-12 04:30:57
---

> 最近发现Linux 终端里有很多好玩的命令，这里记录一下，以免下次还得搜索 - -
## 一些好玩的命令
### sl 
```
$ sl
```
![](/content/images/2016/01/2012111315222236.jpg)
### telnet
```
$ telnet towel.blinkenlights.nl
```
### rev
```
$ rev
```
![](/content/images/2016/01/--_2016-01-13_10-39-09.png)

### factor
```
$ factor
```
![](/content/images/2016/01/--_2016-01-13_10-32-16.png)

### cowsay
```
$ cowsay / cowthink
```
![](/content/images/2016/01/--_2016-01-13_10-41-06.png)

### fortune
```
$ fortune / fortune-zh
```
![](/content/images/2016/01/--_2016-01-13_10-42-01.png)

### cmatrix
```
$ cmatrix
```
![](/content/images/2016/01/--_2016-01-13_10-42-45.png)

### yes
```
$ yes
$ yes I love China
```
yes 是一个非常有趣又有用的命令，尤其对于脚本编写和系统管理员来说，它可以自动地生成预先定义的响应或者将其传到终端。

### toilet
```
$ toilet hcaijin.com
$ toilet -f mono12 -F metal hcaijin.com
```
![](/content/images/2016/01/--_2016-01-13_10-47-46.png)

### while
```
$ while true; do echo "$(date '+%D %T' | toilet -f term -F border --gay)"; sleep 1; done
```
![](/content/images/2016/01/--_2016-01-13_10-49-59.png)

### espeak
```
$ espeak "Tecmint is a very good website dedicated to Foss Community"
```
将你的多媒体音箱的音量调到最大，然后在将这个命令复制到你的终端，来看看你听到上帝的声音时的反应吧。

### for
```
for i in {1..19}; do for j in $(seq 1 $i); do echo -ne $i x $j=$((i*j))\\t;done; echo;done
```
![](/content/images/2016/01/--_2016-01-13_10-44-41.png)

### banner
```
$ banner hcaijin.com
```
![](/content/images/2016/01/--_2016-01-13_10-58-38.png)

## 终端下有很多危险的命令，千万小心执行。
### fork炸弹
这个命令其实是一个fork炸弹，它会以指数级的自乘，直到所有的系统资源都被利用了或者系统挂起
```
$ :(){ :|:& }:
```
### rm
删除命令，一定要小心不可用root用户执行以下
```
$ rm -rf /
```

### dd
很有用的命令，但是要注意不要运行以下命令，其实我也没有运行过- -
```
$ dd if=/dev/zero of=/dev/mem
```

### shred
覆盖文件让它不能再读，传说中的文件粉碎机
```
$ shred --help
```
![](/content/images/2016/01/--_2016-01-13_11-03-46.png)
