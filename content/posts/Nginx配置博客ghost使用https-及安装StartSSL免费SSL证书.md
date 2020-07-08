---
title: Nginx配置博客ghost使用https 及安装StartSSL免费SSL证书
tags: 
- ghost 
- nginx 
- https 
- openssl
permalink: nginxpei-zhi-https-ji-an-zhuang-startsslmian-fei-sslzheng-shu
id: 32
updated: '2015-09-13 21:55:17'
date: 2015-09-13 10:33:52
---

今天配置安装ssl证书碰到了不少的问题，这里记录一下。
>HTTP协议默认情况下是不加密的，各种密码，邮件的传输都是明文的，极有可能被互联网上的黑客给获取，造成隐私泄漏。
SSL是Secure Socket Layer的简称，具体的作用就是在部署了SSL证书的网站跟用户浏览器之间建立一个安全的会话。

### nginx编译安装ssl模块
在说安装证书之前，我先说一下，nginx 要想使用ssl需要在编译安装的时候加上配置参数 `--with-zlib=/data/nginx/lib/` 使用命令先看一下nginx配置
```
$ /usr/local/nginx/sbin/nginx -V
nginx version: nginx/1.9.0
built by gcc 4.4.7 20120313 (Red Hat 4.4.7-11) (GCC) 
built with OpenSSL 1.0.1e-fips 11 Feb 2013
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
```
如果在configure arguments: 没有找到ssl相关模块，就得重新编译一下。找到nginx源码包，执行以下命令，如下：

==这里提醒一下，安装软件的时候我觉得还是在root权限下方便，而且可以避免不必要的无权限执行报错，即使有sudo的权限。==
```
$ cd nginx-1.9.0
$ ./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module   ##这里的stub模块是个统计性能用的，与本文无关，你也可以不安装这个。
$ make     ##这里不要使用 make install，否则就覆盖安装了 make完之后在objs目录下就多了个nginx，这个就是新版本的程序了
$ cp /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.bak
$ cp objs/nginx /usr/local/nginx/sbin/nginx
$ /usr/local/nginx/sbin/nginx -V  ##这里就可以使用新版本的程序看一下编译参数里已经有ssl模块了。
```
### 证书安装
这里说一下，免费的SSL与付费的SSL还是有区别的，我主要是为博客后台登陆使用SSL，学习配置一下。

StartCom公司是到目前止仅有的还提供免费SSL服务的公司（应该是吧，我也是听别人说的），支持多种浏览器的正常识别，只要通过他们的个人信息审核就可以免费使用一年的时间。我自己审核的时候，本想随便写个英文名称，地址，但是都被弊掉了。建议填写个人信息的时候还是要尽量真实。这样才能够一次性通过邮件审核。
[StartSSL官方首页](http://www.startssl.com/) 具体申请流程我就不说了，打开官方网站看一下就知道了。

我们来说一下，安装的流程：
首先使用ssh登陆vps，执行如下命令生成证书
```
$ openssl req -new -newkey rsa:2048 -nodes -out server.csr -keyout server.key
```
![](/content/images/2015/09/--_2015-09-14_00-06-53.png)

以上生成的server.csr 需要把内容粘贴到 StartSSL 去生成域名证书了。

这里生成的server.key 是没有passphrase的，所以这一节我们可以跳过不看。如果有配置密码的话，我们需要去掉private key的passphrase才能让Nginx自由自在的启动。
```
$ cp server.key server.key.bak
$ sudo openssl rsa -in server.key.bak -out server.key
```

### 开始配置nginx
```
server {
    listen 443 ssl;

    server_name hcaijin.com www.hcaijin.com;

    location / {
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   Host      $http_host;
        proxy_pass         http://127.0.0.1:2368;
    }

    ssl_certificate      /usr/local/nginx/ssl/hcjwebssl.crt;
    ssl_certificate_key  /usr/local/nginx/ssl/hcjnopassssl.key;

    ssl_session_cache    shared:SSL:1m;
    ssl_session_timeout  5m;

    ssl_ciphers  HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers  on;

}
```
配置把http的请求转到https
```
server {
    listen 80;

    server_name hcaijin.com www.hcaijin.com;

    location / {
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   Host      $http_host;
        proxy_pass         http://127.0.0.1:2368;
    }

    #rewrite ^(.*) https://$server_name$1 permanent;
	location ~ /ghost(/.*) {
		rewrite ^ https://$server_name$request_uri? permanent;
	}

}
```

```
$ /etc/init.d/nginx restart   ##重启一下nginx服务
```
### 解决Firefox浏览器不信任StartSSL免费SSL的问题
```
$ wget http://cert.startssl.com/certs/ca.pem   
$ wget http://cert.startssl.com/certs/sub.class1.server.ca.pem  
$ sudo cat ca.pem sub.class1.server.ca.pem >> server.crt
$ /etc/init.d/nginx restart
```
