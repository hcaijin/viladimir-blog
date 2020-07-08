---
title: Mysql 设置大小写敏感
tags: 
- mysql
permalink: mysql-she-zhi-da-xiao-xie-min-gan
id: 19
updated: '2016-01-08 00:53:04'
date: 2015-06-01 22:36:28
---

1、linux下mysql安装完后是默认：区分表名的大小写，不区分列名的大小写； 

2、如何设置为不区分表名的大小写：
修改mysql配置文件/etc/mysql/my.cnf 中,在[mysqld]后添加lower_case_table_names=1，默认为0表示区分大小写，然后重启MYSQL服务。； 

3、Mysql 在不同的操作系统中大小写敏感区别：
3.1、MySQL在Linux下数据库名、表名、列名、别名大小写规则是这样的： 

* 数据库名与表名是严格区分大小写的； 
* 表的别名是严格区分大小写的； 
* 列名与列的别名在所有的情况下均是忽略大小写的； 
* 变量名也是严格区分大小写的； 

3.2、MySQL在Windows下都不区分大小写。 


4、如果想在查询时区分字段值的大小写，则：字段值需要设置BINARY属性，设置的方法有多种： 

A、创建时设置： 
CREATE TABLE T( 
A VARCHAR(10) BINARY 
); 

B、使用alter修改： 
ALTER TABLE `tablename` MODIFY COLUMN `cloname` VARCHAR(45) BINARY; 
