---
title: 博客系统从ghost 迁移hexo 安装与配置
date: 2018-05-06 16:33:21
tags:
- hexo
- ghost
---


# 备份Ghost

后台export，导出后是一个JSON，包含所有文章以及一些元数据：修改日期、Tags 等等

图片等资源，可以到 assets 文件夹下，打包下载

```
cd /data/www/ghost
tar -zcvf images.tag assets/content
```

# 安装hexo

## 安装依赖
```
pacman -S npm
```

## 安装
```
npm install hexo-cli -g
cd /data/www/
hexo init hcaijin.com
cd hcaijin.com
hexo install
hexo server
```

# 迁移

## 导入Ghost数据

```
## 安装数据转换插件
npm install hexo-migrator-ghost --save

## 导入数据
hexo migrate ghost ghost-export.json

```

## 导入图片

```
cp images.tag /data/www/hcaijin.com/source/
cd /data/www/hcaijin.com/source/
tar -zxvf images.tag
```

# 最后，做一些必要的配置

## 基本配置

* [Hexo 配置](https://hexo.io/zh-cn/docs/configuration.html)
* [NexT 配置](http://theme-next.iissnan.com/getting-started.html)
* [NexT 高级配置](http://theme-next.iissnan.com/theme-settings.html)

## 安装其他插件
```
npm install hexo-generator-searchdb --save                                                       
npm install hexo-generator-sitemap --save                                                        
npm install hexo-generator-feed --save                                                           
npm install hexo-pwa --save                                                                      
npm install hexo-all-minifier --save
```

