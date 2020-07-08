---
title: Linux中的文件特殊权限
tags: 
- linux 
- setuid 
- setgid 
- stick bit
permalink: linuxzhong-de-wen-jian-te-shu-quan-xian
id: 24
updated: '2015-08-19 05:55:53'
date: 2015-06-15 06:25:56
---

linux中除了常见的读（r）、写（w）、执行（x）权限以外，还有3个特殊的权限，分别是setuid、setgid和stick bit

## setuid、setgid

先看个实例，查看你的/usr/bin/passwd 与/etc/passwd文件的权限

    # ls -l /usr/bin/passwd /etc/passwd
    -rw-r--r-- 1 root root  1549 08-19 13:54 /etc/passwd
    -rwsr-xr-x 1 root root 22984 2007-01-07 /usr/bin/passwd
众所周知，/etc/passwd文件存放的各个用户的账号与密码信息，/usr/bin/passwd是执行修改和查看此文件的程序，但从权限上看，/etc/passwd仅有root权限的写（w）权，可实际上每个用户都可以通过/usr/bin/passwd命令去修改这个文件，于是这里就涉及了linux里的特殊权限setuid，正如-rwsr-xr-x中的s

setuid就是：让普通用户拥有可以执行“只有root权限才能执行”的特殊权限，setgid同理指”组“

作为普通用户是没有权限修改/etc/passwd文件的，但给/usr/bin/passwd以setuid权限后，普通用户就可以通过执行passwd命令，临时的拥有root权限，去修改/etc/passwd文件了

## stick bit （粘贴位）

再看个实例，查看你的/tmp目录的权限

    # ls -dl /tmp
    drwxrwxrwt 6 root root 4096 08-22 11:37 /tmp
 tmp目录是所有用户共有的临时文件夹，所有用户都拥有读写权限，这就必然出现一个问题，A用户在/tmp里创建了文件a.file，此时B用户看了不爽，在/tmp里把它给删了（因为拥有读写权限），那肯定是不行的。实际上是不会发生这种情况，因为有特殊权限stick bit（粘贴位）权限，正如drwxrwxrwt中的最后一个t

stick bit (粘贴位)就是：除非目录的属主和root用户有权限删除它，除此之外其它用户不能删除和修改这个目录。

也就是说，在/tmp目录中，只有文件的拥有者和root才能对其进行修改和删除，其他用户则不行，避免了上面所说的问题产生。用途一般是把一个文件夹的的权限都打开，然后来共享文件，象/tmp目录一样。

## 设置方法
setuid：chmod u+s xxx

setgid: chmod g+s xxx

stick bit : chmod o+t xxx

或者使用八进制方式，在原先的数字前加一个数字，三个权限所代表的进制数与一般权限的方式类似，如下:

     suid   guid    stick bit

      1        1          1

所以：suid的二进制串为：100，换算十进制为：4

guid的二进制串为:010,换算：2

stick bit 二进制串：001，换算：1

于是也可以这样设:setuid:chmod 4755 xxx

setgid:chmod 2755 xxx

stick bit:chmod 1755 xxx

最后，在一些文件设置了特殊权限后，字母不是小写的s或者t，而是大写的S和T，那代表此文件的特殊权限没有生效，是因为你尚未给它对应用户的x权限  
