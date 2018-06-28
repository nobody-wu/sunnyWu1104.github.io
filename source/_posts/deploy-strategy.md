---
title: 项目平滑部署
tags: 运维
categories: 运维
copyright : ture
---

> 这周需要出一个技术方案，有关于项目部署时尽可能地平滑处理。由此做一下小记，下一篇会对6中部署策略做解释

## 引言

单机服务器我们会面临很多生产上的问题：

1. 怕服务器压力过大，任务处理缓慢

2. 内存不足，服务挂了

3. 项目发布导致服务停机

...

考虑到负载均衡，一般的项目都会使用2台以上的服务器。这次我们主要分享项目发布部署的问题。

## 单机发布

在单机情况下，基本上也不需要考虑太多了。。直接kill后，重启容器进行部署。这也是部署策略中的重建策略。这个方式意味着服务的宕机时间依赖于应用下线和启动耗时。

## 多机灰度发布

### 方案一

最容易理解的方式，我们可以通过Nginx做负载均衡，达到较为”平滑”的发布。这种方式优点就是操作比较简单，缺点是影响用户操作。如果某个用户的操作刚好落在正在发布重启的 tomcat 节点上，必然会受到影响，虽然是短暂的。而在其他 tomcat 继续发布的时候，又会扩大这种影响。
其实整个过程就是部署策略中的—蓝绿部署。

一般我们生产环境下，会有两组 tomcat ， 一组用于线上服务（线上组），一组作为备份（备份组）。我们可以先将备份组所有 tomcat 节点部署好，然后修改 nginx 配置文件，切换到备份组，然后操作 nginx 平滑 reload 配置文件。最后再将 线上组 所有 tomcat 节点部署好。 此时 备份组转为线上组，而线上组转为备份组。

### 方案二

通过Nginx反向代理，达到测试人员使用线上环境针对新的应用进行测试，然而正常用户还是使用原来的应用。整个过程有点像A/B部署策略

可以根据公网ip进行反向代理，本部门的公网ip是固定的，那么当客户访问的时候，如果是本部门的公网ip的话，nginx进行方向代理到新代码tomcat上，如果非本部门的公网ip，那么代理到原有tomcat上。

!(nginx)[http://p95stksgt.bkt.clouddn.com/deploy-nginx.png]

参考Nginx代码：

```
upstream jljerp {
           server 192.168.1.190:8001  weight=20 max_fails=2 fail_timeout=30s;
           ip_hash;
                }
upstream jljerp_rc {
           server 192.168.1.190:8004  weight=20 max_fails=2 fail_timeout=30s;
           ip_hash;
                }
server {
    listen       80;
    server_name  jljerp.jinlejia.com;
        root   /var/www/index;
        index  index.html index.htm;
location / {
          proxy_set_header HOST   $host;
          proxy_set_header X-Real-IP      $remote_addr;
          proxy_set_header X-Forwarded-FOR $proxy_add_x_forwarded_for;
          proxy_connect_timeout 600;
          proxy_read_timeout 600;
          proxy_send_timeout 600;
        #  预发布规则，这个地址是部门内部公网地址，当这个地址过来的请求转发到新tomcat上
        if ($remote_addr  ~* "202.106.0.20") {
          proxy_pass      http://jljerp_rc;
        }
        # 如果不是本部门ip请求，按照原有规则进行原有生产环境进行转发
          proxy_pass      http://jljerp;
              }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

静态规则：

```
server {
    listen      80;
    server_name  www.a.com;


        root   /var/www/a;
        index  index.html index.htm;

    location / {
        #  预发布规则,如果是本部门的公网的ip，访问这个目录下的地址
         if ($remote_addr  ~* "202.106.0.20"){
               root    /var/www/b;
               }
    }
# 由于字体使用跨域的方式进行的调用，默认浏览器拒绝访问，加上这个location就可以在其他域名下访问这个域名的字体了
    location ~* \.(eot|ttf|woff|svg|otf|woff2)$ {
             add_header Access-Control-Allow-Origin *;
    }

    error_page  404 500 502 503 504  /404.html;
    location = /404.html {
        root   /usr/share/nginx/html;
    }
}
```