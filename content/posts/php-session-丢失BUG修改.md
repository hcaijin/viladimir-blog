---
title: php session 丢失BUG修改
tags: 
- php 
- session 
- Thinkphp
permalink: php-session-diu-shi-bugxiu-gai
id: 23
updated: '2015-08-19 05:59:48'
date: 2015-06-15 06:07:48
---

今天真是被这个问题给郁闷到了，调试了代码半天时间，终于从http://www.111cn.net/phper/php-cy/56742.htm 里找到了线索。就是存储session的目录权限不可写，或者目录空间満了写不进去就会出现这个BUG.
  
    $ ls -al /     
    drwxr-xr-x.  12 root root 53248 Jun 15 17:14 tmp
发现/tmp 目录权限不对。

后面，打开Thinkphp debug ,trace 页面查找到 open('/tmp/sess_ifoeq9834f98h4h54',O_REIOR) promostion demoin

>总结一下， 就是遇到问题，日志才是最好的排错地方。这里记录一下，以免以后又犯错，直接去调试代码，花费了不小的功夫，还找不到原因。
