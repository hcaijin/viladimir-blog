---
title: 导出CSV文件，对长数字字符会自动显示科学计数解决方法
tags: 
- csv 
- excel 
- phpexcel
permalink: >-
  phpdao-chu-csvwen-jian-dui-chang-shu-zi-zi-fu-hui-zi-dong-xian-shi-ke-xue-ji-shu-jie-jue-fang-fa
id: 28
updated: '2015-08-06 06:07:22'
date: 2015-08-06 06:00:33
---

今天做csv导出遇到订单号太长导致导出来用EXCEL打开显示为科学计数了，最后几位直接显示为0。但是用文本方式打开订单号是正常的，这说明一定是与EXcel有关系。
GOOGLE了一下，找到EXCEL相关介绍：
>Excel显示数字时，如果数字大于12位，它会自动转化为科学计数法；如果数字大于15位，它不仅用于科学技术费表示，还会只保留高15位，其他位都变0。

    $oid = "\t".$val['oid'];

如果是phpexcel导出的话，把"\t"换成" "即可。
