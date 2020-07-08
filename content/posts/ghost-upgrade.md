---
title: ghost 更新记录
date: 2017-05-06 12:33:21
tags:
- ghost
---


好久没有更新ghost 0.6.0，今天更新的时候发现最新版本，0.11.11 版本 更新安装的时候报错。

查看error日志，是依赖的npm和node版本问题。解决方法就是要么升级npm,node，要么降级ghost到npm,node支持的版本。

* 升级npm,node在这里就不说了，网上有很多的方法，我用的是搬瓦工家的最便宜vps，使用的npm,node不好升级，估计还得升级linux内核，我就不打算使用这个方法了。

* 降级ghost到npm,node支持的版本。
我们到[Ghost各版本历史](https://github.com/TryGhost/Ghost/releases)去找一下历史版本，我尝试了几个版本，最后确定0.8.0这个版本是可以正常使用的。

> 这样我们就可以开始升级ghost了。升级ghost不需要停了当前的服务，但是，升级更新都要做好备份。

### 备份

登陆并进入https://$HOSTNAME/ghost/debug这个页面导出备份。

最好能登陆到服务器进入ghost安装的目录备份一下根目录下的content，这一步要先暂停服务。
```
cd /data/www/ghost
tar -zcvf ghost-content.tag content
```

> 备份好以后，我们就可以删除与升级相关的目录了文件。

```
rm -rf core/ node_modules/ index.js *.json 
```

### 下载最新版本
```
curl -LOk https://github.com/TryGhost/Ghost/releases/download/0.8.0/Ghost-0.8.0.zip
```
然后解压到/data/www/ghost
```
cd ~
unzip -uo Ghost-0.8.0.zip -d /data/www/ghost
```

### 安装并重启
```
npm cache clear && npm install --production
```
没有报错的话就是安装成功了。

重启ghost
```
NODE_ENV=production pm2 start index.js --name "ghost" 
```
