---
title: nginx部署https证书
date: 2017-05-12 12:33:18
categories:
- Nginx
tags:
- https
- nginx
- ssl
---

<!-- more -->

首先简述下什么是**https**, 还有自己对**https**的理解.

https可以可以理解为http+ssl. http就是我们常用到超文本传输协议了.ssl就是一种数字证书, 使用secure socket layer 协议在浏览器和web服务器之间建立一条安全的通道,从而实现数据在传输的时候加密.

我们可以自己模拟这个证书颁发和使用的过程,更好理解https协议.

```shell
$ yum install openssl -y
$ openssl genrsa -aes256 -out ca.key 2048
$ 
```

配置https

在配置文件信息server block块中,必须使用监听命令listen的SSL参数和定义服务器证书文件和私钥文件,如下所示:

```shell
server {
  listen				443 ssl;
  server_name			www.example.com;
  # 证书文件
  ssl_certificate		www.example.com.crt;
  # 私钥文件
  ssl_certificate_key	www.example.com.key;
  ssl_protocols			TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers			HIGH:!aNULL:!MD5;
  ...
}
```

根据nginx官网配置https的文档说明,https server优化有这么一段话,我翻译了如下:

SSL操作会消耗额外的CPU资源,在多核处理器系统上会有多个工作进程被运行,不低于可用的CPU核心数量.最大的CPU消耗阶段集中在SSL握手通讯.有两种方式去最小化每个客户端的操作量:第一种是开启keepalive连接通过一个连接去发送多个请求;第二种是重用SSL会话参数,以避免SSL握手和后续连接.会话存储在共享在一个工作人员的SSL会话缓存中，并有ssl_session_cache指令配置。一兆字节的缓存包含4000个会话。默认缓存超时5分钟。通过使用ssl_session_timeout指令来增长。这里是一个简单的例子配置优化一个10兆直接共享会话缓存的多核系统。

>```
>worker_processes auto;
>
>http {
>    ssl_session_cache   shared:SSL:10m;
>    ssl_session_timeout 10m;
>
>    server {
>        listen              443 ssl;
>        server_name         www.example.com;
>        keepalive_timeout   70;
>
>        ssl_certificate     www.example.com.crt;
>        ssl_certificate_key www.example.com.key;
>        ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
>        ssl_ciphers         HIGH:!aNULL:!MD5;
>        ...
>```

 使用HSTS策略强制浏览器使用https链接

HSTS( http strict transport security), 强制要求蓝蓝器总是通过https来访问一个https网站.

在nginx配置文件加上以下信息就可以:

```shell
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains;preload" always;
```

- max-age : 设置单位时间内强制使用https链接.

- includeSubDomains : 可选, 所有子域名同时生效.

- preload : 可选, 非规范值 , 用于定义使用 HSTS预加载列表.

- always : 可选 ,保证所有响应都发送此响应头,包括各种内置错误响应.

  加强https安全性
