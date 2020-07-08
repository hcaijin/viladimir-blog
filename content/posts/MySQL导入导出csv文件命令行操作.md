---
title: MySQL导入导出csv文件命令行操作
tags: 
- mysql 
- csv 
- utf8
permalink: mysqldao-ru-dao-chu-csvwen-jian-ming-ling-xing-cao-zuo
id: 18
updated: '2015-06-01 03:05:06'
date: 2015-06-01 02:36:16
---

1、先确认一下cvs的文件格式，确保与表编码一至。设为utf8
2、创建表，设置为utf8
3、登陆mysql, 导入csv文件

    $ mysql -uroot -p
    MariaDB []> create database test; use test;
    MariaDB [test]> LOAD DATA INFILE '/mysql/test_data.csv' REPLACE INTO TABLE test_table CHARACTER SET utf8 FIELDS TERMINATED BY ',' ENCLOSED BY '"' LINES TERMINATED BY '\r\n';

注意：test_data.csv 要在mysql 的用户权限中，即使放到/tmp 目录下也是有问题的，暂时不知道怎么解决。
        
    $ sudo mkdir /mysql ; 
    $ sudo chown -R mysql.mysql /mysql 
    $ sudo cp ~/test_data.csv /mysql/
