---
title: 安装 node.js ghost 相关总结
tags: 
- nodejs 
- ghost
- linux
permalink: an-zhuang-ghostxiang-guan-zong-jie
id: 5
updated: '2015-05-10 22:44:16'
date: 2015-04-30 02:49:04
---


## 下载安装
>安装node.js 下载最新node.js 编译安装需要一段时间。

     $ wget http://nodejs.org/dist/node-latest.tar.gz
     $ tar -xzf node-latest.tar.gz
     $ cd node-v
     $ ./configure
     $ make
     $ sudo make install

## 安装Ghost

### 安装扩展  
 `$ yum install gcc-c++`

### ghost下载，安装
    $ sudo mkdir -p /data/www/
    $ cd /data/www/
    $ wget https://ghost.org/zip/ghost-latest.zip
    $ unzip -d ghost ghost-latest.zip
    $ cd /var/www/ghost
    $ sudo npm install --production
*安装完成后用 npm start 命令启动开发者模式下的 Ghost，用于检查有没有安装成功。 成功了，Ghost会运行在本地局域网内 127.0.0.1:2368。如果是在电脑上安装的，用浏览器访问此地址即可预览 Ghost。*   

## 安装pm2

> 安装强大的进程守护程序“PM2”保证应用在开机以后自动启动

进入到/data/www/ghost，执行命令安装PM2：   
`$ sudo npm install pm2 -g`

设置环境变量为“production”生产模式，“index.js”是程序启动的入口。最后给这个PM2的进程命名为"ghost" 执行下面的命令：   
`$ NODE_ENV=production pm2 start index.js --name "ghost"`
设置开机自动运行网站：   
`$ pm2 startup centos
 $ pm2 save`   
*可以执行 pm2 help 查看帮助。*

## 配置nginx
> 配置 Nginx 的反向代理：新建一个 Nginx 代理配置文件,并将代理指向到本地的Ghost端口:

    $ sudo vim /etc/nginx/conf.d/ghost.conf`  
    server {
         listen 80;
         server_name My-Ghost-Blog.com;
         location / {
            proxy_set_header   X-Real-IP $remote_addr;
            proxy_set_header   Host      $http_host;
            proxy_pass         http://127.0.0.1:2368;
        }
    }    
重新启动 Nginx 服务器.  
`$ sudo service nginx restart`

## 注意事项
>修改服务器时间，这样新增的文章才能显示正常的本地时间 。

    $ sudo yun install -y ntp
    $ sudo cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
