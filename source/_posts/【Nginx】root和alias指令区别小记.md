---
title: 【Nginx】root和alias指令区别小记
tags: nginx
categories: nginx日常小记
copyright : ture
---

## 问题：
今天在搭建前端react项目的时候，nginx部署遇到了一个小问题:当前项目下引用的一些静态资源没有加载到，一直报错404，导致样式展示不全。

### 排查问题：

1. 一直在怀疑是代码中引用的相对路径的问题，修改后问题还是无法修复。
2. 怀疑nginx容器需要重启，问题还是无法修复。
3. 排查nginx配置文件信息，发现了一些端倪：

#### 原配置文件数据:
```
server {
        listen       8800;
        server_name  lcoalhost;
        access_log /data/www/logs/nginx/nginx_access.log  local;

        location / {
          root  /data/www/static;
          index index.html;

#       try_files $uri $uri/index.html;

       }

        error_page  405     =200 $uri;

         location  /cashwallet {
             alias  /data/www/static;
            index index.html;
            try_files $uri /shaxiaoseng/index.html;
           }

```

熟悉nginx的同学，估计在这里一眼就能发现问题了，不过我这边就直接贴出来修改后的配置文件

#### 修改后的配置文件:

```
server {
        listen       8888;
        server_name  lcoalhost;
        access_log /data/www/logs/nginx/nginx_access.log  local;

        location /cashwallet {
          root  /data/www/static;
          index index.html;
          try_files $uri /shaxiaoseng/index.html;

#       try_files $uri $uri/index.html;

       }

        error_page  405     =200 $uri;

```

** 在这里我们可以发现一些改变：我将location下的alias修改为了root **

## 原因:

alias指令对于root，操作上很简单，只要把root地址替换host后就是文件在硬盘路径（真实地址）。对于alise，它并不是替换匹配后的url地址，而是替换匹配部分的url。alias指令也可以有多个

## eg:
使用root指令，查询的资源路径会是: /cashwallet/data/www/static/cashwallet

使用alias指令，查询的资源路径会是: /data/www/static/cashwallet

所以会导致查无资源的问题。

## 传送门:
愿意的朋友，可以去[【传送门】](https://www.jianshu.com/p/4be0d5882ec5)看这篇更为详细的介绍对比