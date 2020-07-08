---
title: linux用extundelete恢复ext2、ext3、ext4下rm -rf误删除的数据
tags: 
- linux 
- extundelete 
- fdisk
permalink: linuxyong-extundeletehui-fu-ext2-ext3-ext4xia-rm-rfwu-shan-chu-de-shu-ju
id: 12
updated: '2016-01-11 07:15:10'
date: 2015-05-12 23:40:20
---

国外的Linux系统管理员守则中有这么一条：“慎用 rm -rf 命令，除非你知道此命令所带来的后果“

Linux下删除文件并不是真实的删除磁盘分区中的文件，而是将文件的inode节点中的扇区指针清除，同时释放这些数据对应的数据块，当释放的数据块被系统重新分配时，那些被删除的数据就会被覆盖，所以误删除数据后，应马上卸载文件所在的分区。
每个文件有inode和block组成，inode是文件系统组成的最基本单元，它保存着文件的基本属性(大小、权限、属主组等)和存放的位置信息。而block用来存储数据。类似key-value，inode就是key，block对应value，通过key查找key对应的value。类似python的字典。

## 查看根目录的inode值
``` 
ls -id /
2 /
```
一般”根”目录的inode值为2,一个分区挂载到一个目录下时，这个”根”目录的inode值为2

```
mount /dev/sdb2 /mnt
ls -id /mnt
2 /mnt
```

## 安装extundelete

### 下载extundelete

```
wget   http://ncu.dl.sourceforge.net/project/extundelete/extundelete/0.2.0/extundelete-0.2.0.tar.bz2
```
### 所需依赖包
```
yum -y install e2fsprogs e2fsprogs-libs e2fsprogs-devel
```
### 编译安装extundelte
```
# tar jxvf extundelete-0.2.0.tar.bz2
# cd extundelte-0.2.0
# ./configure
# make; make install
```
## 用extundelete恢复文件

### 模拟数据误删除环境
```
# mkdir /data
# mkfs.ext4 /dev/sdb2
# mount /dev/sdb2 /data
# cp /etc/hosts /data/
# mkdir /data/test
# echo "extundelete test" > /data/test/geek.txt
# md5sum hosts                           
#获取文件校验码
54fb6627dbaa37721048e4549db3224d  hosts
# md5sum test/geek.txt
eb42e4b3f953ce00e78e11bf50652a80  test/geek.txt
# rm -fr /data/*
```
### 卸载磁盘分区
```
 umount /dev/sdb2
```
### 查询恢复数据信息
```
extundelete /dev/sdb2 --inode 2
```
.....
```

File name                             | Inode number | Deleted status
Directory block 8657:
.                                          2
..                                         2
lost+found                                 11             Deleted
hosts                                      12             Deleted
test                                       130817         Deleted
```
上面标记为Deleted是已经删除的文件或目录

## 具体操作
### 开始恢复单个文件
默认恢复到当前目录下的RECOVERED_FILES目录中去
```
 extundelete /dev/sdb2 --restore-file hosts
```
### 恢复一个目录
```
 extundelete /dev/sdb2 --restore-directory test/
```
### 全部恢复
```
 extundelete /dev/sdb2 --restore-all
```

## 检测是否恢复成功
```
# md5sum RECOVERED_FILES/hosts                     获取文件校验码
54fb6627dbaa37721048e4549db3224d  RECOVERED_FILES/hosts
# md5sum RECOVERED_FILES/test/geek.txt
eb42e4b3f953ce00e78e11bf50652a80  RECOVERED_FILES/test/geek.txt
```
校验码与之前的完全一致。
