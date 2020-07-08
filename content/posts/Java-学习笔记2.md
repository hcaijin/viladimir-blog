---
title: Java 学习笔记2
tags: 
- java
- maven
permalink: java-xue-xi-bi-ji-2-2
id: 46
updated: '2016-01-25 03:09:09'
date: 2016-01-20 06:43:02
---

## 镜像配置

由于maven的中央仓库位于国外，速度慢，也有可能其他原因无法访问，我们可以使用国内的镜像仓库。配置镜像仓库需要修改conf/settings.xml,打开该文件修改mirror标签如下：
```
vim /opt/maven/conf/settings.xml
```
![](/content/images/2016/01/--_2016-01-20_19-42-35.png)

maven仓库默认是放在用户目录的.m2隐藏目录下的 ~/.m2/repository/ 。如果需要将仓库迁移到其他目录，修改conf/settings.xml 
![](/content/images/2016/01/--_2016-01-20_19-52-27.png)

## 环境变量

配置maven编译程序过程中可用的最大，最下内存，防止内存溢出。
```
MAVEN_OPTS="-Xms256m -Xmx512m"
export MAVEN_OPTS
```
> 配置web服务一定要记得设置compressableMimeType
![](/content/images/2016/01/--_2016-01-25_16-02-11.png)

