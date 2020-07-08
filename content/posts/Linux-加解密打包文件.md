---
title: Linux 加解密打包文件
permalink: shi-yong-taryu-openssljia-mi-jie-mi-da-bao-wen-jian
id: 7
updated: '2015-04-30 05:58:12'
date: 2015-04-30 03:56:17
tags:
- openssl
- linux
---

## 预备知识
> 使用到的命令有tar,openssl,dd

`$ mkdir test_dir; cd test_dir/ ; touch test.file; cd .. ; ll test_dir/`

-rw-r--r-- 1 hcaijin hcaijin 0 4月  30 16:35 test.file

` $ tar -zcvf - test_dir/ | openssl des3 -salt -k password | dd of=test_dir.tag `

test_dir/
test_dir/test.file
记录了0+1 的读入
记录了0+1 的写出
176字节(176 B)已复制，0.0114724 秒，15.3 kB/秒

` $ dd if=test_dir.tag | openssl des3 -d -k password | tar -zxvf - `

记录了0+1 的读入
记录了0+1 的写出
176字节(176 B)已复制，0.000495166 秒，355 kB/秒
test_dir/
test_dir/test.file

> 详细openssl 可查看帮助页

`$ man openssl 
 $ openssl enc -h `
![加密算法列表](/content/images/2015/04/--_2015-04-30_17-51-29.png)

## openssl 命令详解
` SYNOPSIS 
openssl enc -ciphername [-in filename] [-out filename] [-pass arg] [-e] [-d] [-a] [-A] [-k password] [-kfile filename] [-K key] [-iv IV] [-p] [-P] [-bufsize number] [-nopad] [-debug]
说明：
-chipername选项：加密算法，Openssl支持的算法在上面已经列出了，你只需选择其中一种算法即可实现文件加密功能。
-in选项：输入文件，对于加密来说，输入的应该是明文文件；对于解密来说，输入的应该是加密的文件。该选项后面直接跟文件名。
-out选项：输出文件，对于加密来说，输出的应该是加密后的文件名；对于解密来说，输出的应该是明文文件名。
-pass选项：选择输入口令的方式，输入源可以是标准输入设备，命令行输入，文件、变量等。
-e选项：实现加密功能（不使用-d选项的话默认是加密选项）。
-d选项：实现解密功能。
-a和-A选项：对文件进行BASE64编解码操作。
-K选项：手动输入加密密钥（不使用该选项，Openssl会使用口令自动提取加密密钥）。
-IV选项：输入初始变量（不使用该选项，Openssl会使用口令自动提取初始变量）。
-salt选项：是否使用盐值，默认是使用的。
-p选项：打印出加密算法使用的加密密钥。
`

## 加密

>举例：

`$  openssl enc -aes-128-cbc -in pacman.log -out pacman.log.aes `    
enter aes-128-cbc encryption password: **********************
Verifying - enter aes-128-cbc encryption password: *************************
*以上执行成功，生成加密文件 pacman.log.base64，使用以下命令解密：*
`$ openssl enc -d -aes-128-cbc -in pacman.log.aes -out pacman.out.log `
enter aes-128-cbc decryption password: ****************

>下面方法的好处是你可以把它写入到脚本中，自动完成加密功能，不使用pass选项默认系统会提示输入密码并且确认，是需要人工操作的。

`$  openssl enc -aes-256-ecb -in pacman.log -out pacman.aes256.log -pass pass:123456 
 $ file pacman.aes256.log `     
 *pacman.aes256.log: data*    
`$ openssl enc -d -aes-256-ecb -out pacman256.log -in pacman.aes256.log `   
enter aes-256-ecb decryption password:
`$ file pacman256.log `        
pacman256.log: UTF-8 Unicode text

***生成 pacman256.log 可以用file 看到文件解密为原来的类型了***  


