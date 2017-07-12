---
title: Nginx平滑升级
date: 2017-07-12  12:23:36
categories:
- Nginx
tags:
- nginx
---

<!-- more -->

查看原来Nginx的编译参数

```shell
$ /data/local/nginx/sbin/nginx -V
nginx version: nginx/1.10.3
built by gcc 4.4.7 20120313 (Red Hat 4.4.7-18) (GCC) 
built with OpenSSL 1.0.1e-fips 11 Feb 2013
TLS SNI support enabled
configure arguments: --prefix=/data/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-pcre=/data/src/pcre-8.40
```

下载准备升级的源码包

```shell
$ wget -P /data/local/src https://nginx.org/download/nginx-1.12.1.tar.gz
$ tar zxf nginx-1.12.1.tar.gz
$ ./configure --prefix=/data/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-pcre=/data/src/pcre-8.40
$ make
不要执行make install,只需make即可
```

接下来的几步

```shell
$ mv ~/sbin/nginx ~/sbin/nginx.old
$ cp ~/src/nginx-1.12.1/objs/nginx /data/local/nginx/sbin/

$ nginx -V   #查看到最新版本号
$ nginx -t   #测试下最新版是否正常
```

平滑升级

```shell
$ kill -USR2 `cat /data/local/nginx/logs/nginx.pid`
$ kill -WINCH `cat /data/local/nginx/logs/nginx.pid.oldbin`
$ kill -QUIT `cat /data/local/nginx/logs/nginx.pid.oldbin`
```

Nginx信号

```shell
TERM或INT   # 快速停止nginx 指立即停止当前Nginx服务正在处理的所有网络请求，马上丢弃连接，停止工作
QUIT        # 平缓停止nginx 指允许Nginx服务将当前正在处理的网络请求处理完成，但不再接受新的请求，之后关闭连接，停止工作
HUP         # 平滑重启 Nginx服务进程接受到信号后，首先读取新的Nginx配置文件，如果配置语法正确，则启动新的Nginx服务，然后平缓关闭旧的服务进程，如果新的Nginx配置文件有问题，将显示错误，仍然使用旧的Nginx进程提供服务
USR1        # 重新打开日志文件，常用于日志切割
USR2        # 平滑升级 指使用新版本的Nginx文件启动服务，之后平缓停止原有的Nginx进程
WINCH       # 平缓停止worker process，用于nginx平滑升级
```

