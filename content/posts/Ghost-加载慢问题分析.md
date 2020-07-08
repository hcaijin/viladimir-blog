---
title: Ghost 加载慢问题分析
tags: 
- ghost 
- google fonts
- sed 
- grep
permalink: ghost-jia-zai-man-wen-ti-fen-xi
id: 11
updated: '2015-05-11 02:22:15'
date: 2015-05-11 01:52:45
---

Ghost 搭建完能正常访问，可是墙内访问实在太慢了，按F12打开Chrome 开发者工具可以看到主要是 http://fonts.googleapis.com 这个访问使用了太长的时间。应该是GFW搞的鬼，使得 Google Fonts 服务也受影响了，Ghost 后台和默认主题都引用了 Google Fonts 服务，已至于每次打开自己的 Ghost 博客都很慢。

*这样我们就知道了问题的关键，马上登陆服务器，进入ghost目录。*
      
       $ cd /data/www/ghost
       $ grep 'fonts.googleapis.com' -n --color  -r  .
![](/content/images/2015/05/--_2015-05-11_14-04-18.png)
*使用sed删除相应的行*

       $ sudo sed -i '19d' ./content/themes/casper/default.hbs

上面修改的地方其实就是删除 Ghost 系统内引用的 Google Fonts 英文字体文件，这对于国内的用户丝毫没有影响，毕竟我们用的是 中文 嘛。

经过修改，Google Fonts 文件就被彻底清除了。然后重启 Ghost 系统，再登录你的博客看看吧！

       $ sudo -i 
       $ pm2 restart ghost
