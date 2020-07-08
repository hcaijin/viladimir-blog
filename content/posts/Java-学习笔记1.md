---
title: Java 学习笔记1
tags: 
- java
- maven
permalink: java-xue-xi-bi-ji
id: 45
updated: '2016-01-20 06:34:05'
date: 2016-01-20 04:08:30
---

> 以下是在Arch Linux下操作，其他发行版或操作系统不适用。Maven这个词可以翻译为“知识的积累”，也可以翻译为“专 家”或“内行”。本文将介绍Maven这一跨平台的项目管理工具。作为Apache组织中的一个颇为成功的开源项目，Maven主要服务于基于Java平台的项目构建、依赖管理和项目信息管理。无论是小型的开源类库项目，还是大型的企业级应用；无论是传统的瀑布式开发，还是流行的敏捷模式，Maven都能大显身手。

## 安装java环境
```
sudo pacman -S jdk8-openjdk
```
安装好以后可以使用如下命令：
```
archlinux-java help             ##查看帮助
archlinux-java status           ##java环境状态 使用的版本信息
```

## 安装maven

### 配置
maven 会被安装到`/opt/maven/ ` 目录下
```
sudo pacman -S maven
```
修改环境变量
```
vi ~/.bashrc
```
![](/content/images/2016/01/--_2016-01-20_17-18-19.png)

这样以后，就可以使用 mvn 命令来管理java项目，如下：
```
cd /data/gyapp/                ###进入工作目录。(自定义)     
mvn archetype:generate -DgroupId=helloworld -DartifactId=helloworld 
-Dpackage=helloworld -Dversion=1.0-SNAPSHO
```

### 打包程序

执行了上面的命令会在当前工作目录生成helloworld项目目录。
![](/content/images/2016/01/--_2016-01-20_17-24-58.png)
```
cd helloworld/
mvn package
```
这个时候， maven 在 helloworld 下面建立了一个新的目录 target/ ，构建打包后的 jar 文件 helloworld-1.0-SNAPSHOT.jar 就存放在这个目录下。编译后的 class 文件放在 target/classes/ 目录下面，测试 class 文件放在 target/test-classes/ 目录下面。

![](/content/images/2016/01/--_2016-01-20_17-26-01.png)

### 运行
为了验证我们的程序能运行，执行下面的命令：
```
java -cp target/helloworld-1.0-SNAPSHOT.jar helloworld.App
```
输出源代码里的程序，显示 HelloWorld! 表示成功。

## 第一个程序
```
vi HelloWorld.java
```
![](/content/images/2016/01/--_2016-01-20_17-10-49.png)
编译源代码，在当前目录生成HelloWorld.class,然后执行`java HelloWorld`
```
javac HelloWorld.java
java HelloWorld
```

## 重要

转换ppk成linux下面支持的密钥文件

```
sudo pacman -S putty
```
安装putty以后，可以使用如下命令：
```
puttygen git_cesi.ppk -o id_rsa.pub -O public-openssh
puttygen git_cesi.ppk -o id_rsa -O private-openssh
```

[参考](http://www.oracle.com/technetwork/cn/community/java/apache-maven-getting-started-1-406235-zhs.html)


