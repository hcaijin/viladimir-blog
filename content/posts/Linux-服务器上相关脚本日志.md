---
title: Linux 服务器上相关脚本日志
tags: 
- linux 
- shell 
- bash
permalink: linux-fu-wu-qi-shang-xiang-guan-jiao-ben-ri-zhi
id: 40
updated: '2015-12-21 21:27:12'
date: 2015-11-19 21:23:02
---

用如下命令查询出来结果中包含“ip地址=数量”的攻击者信息：
```
cat /var/log/secure|awk '/Failed/{print $(NF-3)}'|sort|uniq -c|awk '{print $2"="$1;}'
```

查看IP所在地：
```
curl ipinfo.io/{IP}
curl cip.cc/{IP}
```

随机生成密码：
```
function randpw32(){ < /dev/urandom tr -dc '!@#$%^&*'_A-Z-a-z-0-9 | head -c${1:-32};echo; }

function randpw16(){ < /dev/urandom tr -dc _A-Z-a-z-0-9 | head -c${1:-16};echo; }
```

Chromium 开启代理：
```
function secure_chromium {
    port=1080
    #使用以下两种配置都可以
    #export SOCKS_SERVER=localhost:$port
    #export SOCKS_VERSION=5
    #chromium &
    chromium --proxy-server="socks://localhost:$port" &
    exit
}
```


