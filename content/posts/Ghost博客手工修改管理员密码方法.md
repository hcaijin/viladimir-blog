---
title: Ghost博客手工修改管理员密码方法
tags: 
- ghost
- sqlite
permalink: ghostbo-ke-shougong-xiu-gai-mi-ma
id: 38
updated: '2015-10-11 14:10:58'
date: 2015-10-11 13:40:18
---

>今天修改Ghost密码，因为是粘贴复制，估计是粘贴到了其他字符，导致密码错误登陆后台失败。而且还因为登录尝试次数过多，造成用户被锁定，只能通过发送邮件找回密码，但是，我配置的Gmail邮箱被google限制使用收发邮件功能了应该，以后看一下具体的原因。

### 查找sqlite数据库用户信息
Ghost用的sqlite数据库，登陆到服务器，找到sqlite数据库，存放位置默认是在 Ghost 安装目录下的 content/data/ ，名称为ghost.db使用如下命令：
```
$ cd /data/www/ghost/content/data/
$ sudo sqlite3 ghost.db
sqlite> .help     ### 这里已经进到sqlite命令行模式下了，用".help"查看帮助
sqlite> .tables   ### 列表所有的表
sqlite> .schema   ### 列表所有表的结构 这里我们主要看users表  
sqlite> selete * from users;
```
到这里就可以列出后台登陆的会员信息了，仔细看一下发现有个status字段的值应该是locked; password字段值是一串使用 [BCrypt Hash Generator](http://bcrypthashgenerator.apphb.com/) 生成的密钥。

### 修改密码为新生成的密钥
// 更新密码 这里的密码为 admin
```
sqlite> update users set password = "$2a$10$ahse9xU.Tr9MttVX4tO1zOER7odgDrQuJzgjZI4fm56x84c/2dGqq" where id = 1;         ### 更新密码
sqlite> update users set status = "active" where id = 1;       ### 解锁用户
sqlite> .quit
```
完成以上两步，就可以重新登录了。
