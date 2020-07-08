---
title: Linux dd
tags: 
- linux 
- dd
permalink: linux-dd
id: 8
updated: '2015-05-10 22:39:29'
date: 2015-05-03 23:09:39
---

>dd 是 Linux/UNIX 下的一个非常有用的命令，作用是用指定大小的块拷贝一个文件，并在拷贝的同时进行指定的转换。
     
常用：
dd 命令生成文件test.data 大小为1024M   
`$ dd if=/dev/zero of=test.data bs=1024M count=1`

### 整盘数据备份与恢复
将本地的/dev/sda1整盘备份到/dev/sda2   
`$ dd if=/dev/sda1 of=/dev/sda2`   
将/dev/sda2全盘数据备份到指定路径的image文件         
`$ dd if=/dev/sda2 of=/path/to/image`    
备份/dev/hdx全盘数据，并利用gzip工具进行压缩，保存到指定路径
`$ dd if=/dev/hdx | gzip >/path/to/image.gz`    
将备份文件恢复到指定盘：    
`$ dd if=/path/to/image of=/dev/hdx`   
将压缩的备份文件恢复到指定盘:      
`$ gzip -dc /path/to/image.gz | dd of=/dev/hdx`   

### 利用netcat远程备份
在源主机上执行此命令备份/dev/hda    
`$ dd if=/dev/hda bs=16065b | netcat < targethost-IP > 1234`   
在目的主机上执行此命令来接收数据并写入/dev/hdc   
`$ netcat -l -p 1234 | dd of=/dev/hdc bs=16065b`    
以下两条指令是目的主机指令的变化分别采用bzip2  gzip对数据进行压缩，并将备份文件保存在当前目录。  

     $ netcat -l -p 1234 | bzip2 > partition.img
     $ netcat -l -p 1234 | gzip > partition.img

### 备份MBR   
备份磁盘开始的512Byte大小的MBR信息到指定文件    
`$ dd if=/dev/hdx of=/path/to/image count=1 bs=512`   
将备份的MBR信息写到磁盘开始部分  
`$ dd if=/path/to/image of=/dev/hdx`   


### 磁盘管理
*#### 得到最恰当的block size*
       
     dd if=/dev/zero bs=1024 count=1000000 of=/root/1Gb.file 
                dd if=/dev/zero bs=2048 count=500000 of=/root/1Gb.file
                dd if=/dev/zero bs=4096 count=250000 of=/root/1Gb.file
                dd if=/dev/zero bs=8192 count=125000 of=/root/1Gb.file
通过比较dd指令输出中所显示的命令执行时间，即可确定系统最佳的block size大小

*#### 测试硬盘读写速度*   

    $ dd if=/root/1Gb.file bs=64k | dd of=/dev/null
               $ dd if=/dev/zero of=/root/1Gb.file bs=1024 count=1000000
通过上两个命令输出的执行时间，可以计算出测试硬盘的读／写速度        

*#### 修复硬盘*    
`$ dd if=/dev/sda of=/dev/sda`     
当硬盘较长时间（比如1，2年）放置不使用后，磁盘上会产生magnetic flux point。当磁头读到这些区域时会遇到困难，并可能导致I/O错误。当这种情况影响到硬盘的第一个扇区时，可能导致硬盘报废。上边的命令有可能使这些数据起死回生。且这个过程是安全，高效的。

### 其他   
*#### 将软驱数据备份到当前目录的disk.img文件*   
`dd if=/dev/fd0 of=disk.img count=1 bs=1440k`    

*#### 拷贝内存资料到硬盘*    
`$ dd if=/dev/mem of=/root/mem.bin bs=1024`   
将内存里的数据拷贝到root目录下的mem.bin文件

*#### 从光盘拷贝iso镜像*   
`$ dd if=/dev/cdrom of=/root/cd.iso`    
拷贝光盘数据到root文件夹下，并保存为cd.iso文件                

*#### 增加Swap分区文件大小*     
`$ dd if=/dev/zero of=/swapfile bs=1024 count=262144`      
创建一个足够大的文件（此处为256M）   
`$ mkswap /swapfile`   
把这个文件变成swap文件    
`$ swapon /swapfile`    
启用这个swap文件   
`$ /swapfile swap swap defaults 0 0 `     
在每次开机的时候自动加载swap文件, 需要在 /etc/fstab 文件中增加一行

*#### 销毁磁盘数据*      
`$ dd if=/dev/urandom of=/dev/hda1`    
利用随机的数据填充硬盘，在某些必要的场合可以用来销毁数据。执行此操作以后，/dev/hda1将无法挂载，创建和拷贝操作无法执行。
