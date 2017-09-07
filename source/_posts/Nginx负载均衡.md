---
title: nginx配置负载均衡
date: 2017-05-23 19:12:39
categories:
- Nginx
tags:
- nginx
- 负载均衡
---

<!-- more -->

### Nginx几种负载均衡算法

- **1.轮询(默认)**

   每个请求按时间顺序逐一分配到不同的后端服务器,如果后端某台服务器宕机,故障系统被自动删除,是用户访问不受影响.

- **2.weight(按权值轮询)**

  weight值越大,分配到的访问机率越高,主要用于后端每个服务器性能不均的情况下.

- **3.ip_hash**

  每个请求按访问的ip的hash结果分配,这样来自同一个IP的访客固定访问一个后端服务器,有效解决了动态网页存在session共享问题.

- **4.fair**

  一个更加智能的负载均衡算法,可以依据页面大小和加载时间长短智能地进行负载均衡,也就是根据后端服务器的响应时间来分配请求,响应时间段的优先分配.

- **5.url_hash**

  按访问url的hash结果来分配请求,使每个url定向到同一个后端服务器,可以进一步提高后端缓存服务器的效率.

- **6.least_conn**

   最少连接负载均衡算法,每次选择的server都是当前最少连接的一个server.

配置负载均衡,使用upstream模块.

```shell
user www;
worker_processes 4;
events{
  worker_connections 1024;
}
http{
  upstream server {
  	ip_hash;
    server 192.168.1.1 weight=3;
    server 192.168.1.2 weight=1;
    server 192.168.1.3 down;
    server 192.168.1.4 backup;
    server 192.168.1.5 max_fails=3 fail_timeout=60s;
  }
 server {
   listen 80;
   location / {
     proxy_pass http://server;
   }
 }
}
```

upstream支持的状态参数

down		当前的server暂时不参与负载均衡.

backup		预留的备份机,当其他所有的非backup机出现故障或者忙碌时,才去访问backup机.

max_fails	允许请求访问失败的次数,默认为1

proxy_next_upstream 模块自定义的错误

fail_timeout	max_fails次数之后,暂停服务的时间.两者可以同时使用.

*注: 当负载均衡调度的算法为ip_hash时,后端服务器在负载均衡调度中的状态不可以是weight和 backup*