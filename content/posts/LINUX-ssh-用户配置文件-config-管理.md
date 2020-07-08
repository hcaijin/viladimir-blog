---
title: LINUX ssh 用户配置文件 config 管理
tags:
- linux
- ssh
permalink: linux-ssh-yong-hu-pei-zhi-wen-jian-config-guan-li
id: 22
updated: '2015-06-13 05:17:01'
date: 2015-06-13 04:32:01
---

>利用 ssh 的用户配置文件 config 管理 ssh 会话。ssh 的用户配置文件是放在当前用户根目录下的 .ssh 文件夹里（~/.ssh/config，不存在则新创建一个），其配置写法如下：

    Host test
        User root
        HostName 192.168.3.152
        PassWord V8kbwqsV0UGTob8EEeL4
    Host *
        #PubkeyAuthentication no
        IdentityFile    ~/.ssh/id_rsa

这样我们就可以使用如下命令直接登陆到hostname 为192.168.3.152 的服务器了：

     # ssh test
使用密钥的好处就是省去每次 ssh 登陆服务器时都要输入登陆密码的操作，这里使用 ssh-keygen 生成 ssh 密钥与公钥：
      
     # ssh-keygen -t rsa

把公钥 id_rsa.pub 上传到远程 192.168.3.152 服务器的 ~/.ssh/ 目录下：

     # ssh-copy-id -i ~/.ssh/id_rsa.pub 192.168.3.152
这样会在服务器的 ~/.ssh/ 目录下生成文件 authorized_keys

**这里注意一点：以 ssh publickey 的形式访问，对当前用户根目录下的 .ssh 文件夹里的目录文件是要有一定的权限要求，之前遇到过 ssh publickey 配置好了，不过用 publickey 登陆验证时则无效。所以，最好设下.ssh 目录权限为 700，authorized_keys 权限为 600，并检查当前用户目录所属的用户组，如：**

     # ls -al .ssh/
     total 12
     drwx------  2 root root 4096 Jun 13 16:20 .
     drwxr-xr-x. 5 other others 4096 Jun 13 16:15 ..
     -rw-------  1 root root  401 Jun 13 16:20 authorized_keys

以上，注意目录 .ssh/ 父目录用户所属用户组，用户。这样也会造成使用publickey 登陆验证时无效，还是提示要输入密码。

      # chown -R root.root /root/

当然，用密钥的方式连接服务器是需要服务器上的 ssh 支持的，需要 ssh 的配置文件（默认是在 etc/ssh/sshd_config）里的 PubkeyAuthentication 设置成 yes。如果要改登陆的端口，直接把 Port 改成你想要的端口值就行。修改完后重启下 ssh ，配置就生效：

     # /etc/init.d/sshd restart

然后，就可以使用ssh 别名登陆服务器了。

用 ssh 作 socks5 代理翻墙，以后不用这样写了(hcj.com 为在墙外的代理服务器)：

     # ssh -CfNg -D1080 hcj.com

使用 scp 传送可以简写成这样：
 
     # scp ~/.ssh/id_rsa.pub test:~/.ssh/authorized_keys

执行远程 ssh 命令：

     # ssh test 'ls -al ~'

打包一个文件（假设当前目录有个名为 test 的文件夹），接着上传到远程服务器，最后解压文件

     # tar -zcvf - ./test/ | ssh test 'cd /user/; tar xvfz -'
