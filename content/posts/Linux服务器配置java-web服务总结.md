---
title: Linux服务器配置java web服务总结
tags: 
- linux 
- java 
- tomcat
permalink: di-ci-pei-zhi-java-webfu-wu
id: 47
updated: '2016-01-26 00:06:30'
date: 2016-01-25 03:14:16
---

> 首先，以前是写php的，从未接触java的开发。最近公司项目重组，被安排说去做java开发，而且要快速上手,安排培训，可是，培训的都是windows下的IDE开发配置。没办法，Google呗。这样就有了这篇东拼西奏的文章，也有自己的一些经验总结，没少碰壁，不过这里还是感谢同事的帮忙，让我对java的运行有了清晰的认识。

## 了解java包运行原理
java是编译型语言，自然少不了打包，链接，当然这些都可以用maven来管理。用maven打包，链接生成的安装包就是可以直接使用java来运行的，我们这里主要说web服务的配置，所以少不了tomcat。使用tomcat来运行web项目包，就可以在浏览器端访问，java应用服务，这就是大致的过程。

## 安装必要的软件包
### 安装jdk,至于安装哪个版本，视情况而定
```
sudo pacman -S jdk8-openjdk
```
可以搜索一下
```
pacman -Ss java
```

### 安装maven
```
sudo pacman -S maven
```

以上安装成功了以后，我们就可以使用`java, mvn ` 的命令了，由于我们使用的是pacman安装方法，必要的环境变量都已经默认好了，可以不需要配置，具体可以看我以前写的 [Java 学习笔记1](http://www.hcaijin.com/java-xue-xi-bi-ji/)

### 安装tomcat，同样的源里也有多个版本，视情况安装相应的版本
```
sudo pacman -S tomcat7
```

## tomcat 主要配置详解
### 主要目录功能
默认情況 tomcat7 安装路径为 /usr/share/tomcat7，这里罗列一下主要目录的作用：

* /usr/share/tomcat7： 程序的主目录，也是变量 $CATALINA_HOME 所指向的位置，在单 tomcat 实例的情況下，也是变量 $CATALINA_BASE 所指向的位置。
* /usr/share/tomcat7/bin： 程序的执行脚本目录
* conf -> /etc/tomcat7： 配置文档目录，存放主要是配置信息。
* lib -> /usr/share/java/tomcat7： 共用jar包目录，这些包即给 tomcat 使用，也能给 web 应用程序所调用。
* logs -> /var/log/tomcat7： 日志目录，对于查找错误以及查看访问记录很有用。
* webapps -> /var/lib/tomcat7/webapps： 默认的 web 应用程序目录，tomcat7 自带了几个示例应用。

### 启动关闭脚本
我们进入程序执行脚本目录
```
cd /usr/share/tomcat7 
sudo ./startup.sh
```
以上，tomcat服务就启动成功了，可以在浏览器中访问` http://localhost:8080 ` ，如果看到 tomcat 猫即说明服务已经安装成功并且能正常运行了。
```
sudo ./shutdown.sh
```
这两个脚本都是通过调用 catalina.sh 来执行的，具体自己看脚本代码。

## 实例讲解tomcat启动java应用
这里我犯了一个错误，总以为java应用之前总得有个相互调用的关系，没想到其实都已经在maven打包，安装到本地就行了，web应用配置好相应的pom.xml就可以调用maven打包，安装好的后台java应用。

然后，我们开始说明代码部署过程：
```
cd /data/app/                             ###进入工程主目录
git clone git@erp:ecerp-saas              ###从erp服务器拉代码到本地
cd /data/app/ecerp-saas/                  ###进入代码目录
git pull                                  ###这个是同步服务器代码
cd /data/app/ecerp-saas/Sources/ecerp     ###进到主要工程目录
mvn  clean install -Dmaven.test.skip=true ###打包安装工程目录下相应的程序，这样就会编译好应用到本地用户目录下`~/.m2/` 
```
在web目录下新建目录erp.hcj.com
```
cd /data/www/
mkdir erp.hcj.com/
cd erp.hcj.com/
```
web应用java环境变量配置
```
touch webconfig
cat webconfig
```
![](/content/images/2016/01/--_2016-01-25_17-47-28.png)

```
source /data/www/erp.hcj.com/webconfig     ###使用环境变量生效
```
创建备份目录,当然这个不是必要的。
```
theday=$(date +%Y%m%d)
releaseDir="/data/deployment/packages/${theday}"
if [ ! -e $releaseDir ]
then
       mkdir -p $releaseDir
fi
```
然后，我们就可以到java应用安装目录下找packagename，把它移到备份目录
```
cp -fp `find ~/.m2/repository/ -name $packagename` $releaseDir/$packagename
```
```
#备份数据
bktime=$(date +%y%m%d%H%M)
backupdir="/data/deployment/release-backup/$bktime/$(basename $srvdir)"
if [ ! -e $backupdir ]
then
    mkdir -p $backupdir
fi
rootdir=/data/www/erp.hcj.com/webroot
for files in $(ls $rootdir)
do
    if [ $files == "upload" ]; then
    ¦   echo $files not backup
    else
    ¦   /bin/cp -rfp $rootdir/$files $backupdir
    fi  
done 
```

```
### 删除旧应用
rm -rf $rootdir/WEB-INF/*
### 解压文件，在web主目录下生成webroot
packagefile=$releaseDir/$packagename
tar zxf $packagefile -C /data/www/erp.hcj.com
### 修改webroot的权限
chown tomcat7.tomcat7 -R /data/www/erp.hcj.com/webroot
```
以上就基本是把java打包的应用程序，安装到了tomcat的webroot目录下了，但是要使这个应用启动成功，还需要配置多实例的tomcat的配置文件server.xml
```
cd /data/www/erp.hcj.com
mkdir -p tomcat/{conf,logs,tmp,work,}
cp -r /etc/tomcat7/* tomcat/conf/
sudo chown -R tomcat7.tomcat7 tomcat/
vi server.xml
```
主要修改如下配置：
![](/content/images/2016/01/--_2016-01-25_18-48-50.png)
![](/content/images/2016/01/--_2016-01-25_18-49-35.png)

```
### 设置tomcat环境变量
export CATALINA_HOME="/usr/share/tomcat7"
export DUSER="tomcat7"
export CATALINA_BASE="/data/www/erp.hcj.com/tomcat"
export CATALINA_PID="$CATALINA_BASE/tomcat.pid"
export CATALINA_TMPDIR="$CATALINA_BASE/tmp"
export CATALINA_OUT="$CATALINA_BASE/logs/catalina.out"                                                                                                                                                                            
export LOCKFILE="$CATALINA_BASE/tomcat.lock"
```
```
### 启动服务
/bin/bash $CATALINA_HOME/bin/startup.sh
### 关闭服务
/bin/bash $CATALINA_HOME/bin/shutdown.sh
```

[参考](http://ufaw0116.erufa.com/wordpress/?p=1254&ckattempt=3)
