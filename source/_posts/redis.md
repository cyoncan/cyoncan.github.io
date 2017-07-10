---
title: redis
date: 2017-05-06 19:23:11
categories:
- redis
tags:
- redis
---

<!-- more -->

下载，提取和编译Redis：

```shell
$ wget http://download.redis.io/releases/redis-3.2.9.tar.gz
$ tar xzvf redis-3.2.9.tar.gz
$ cd redis-3.2.9
$ make
```

编译的二进制文件在src目录中可使用，运行：

```shell
$ src/redis-server
```

也可以使用内置客户端与Redis交互：

```shell
$ src/redis-cli
redis> set foo bar
OK
redis> get foo
"bar"
```

设置连接密码验证

```shell
127.0.0.1:6379> CONFIG GET requirepass
1) "requirepass"
2) ""
127.0.0.1:6379> config set requirepass "password"
127.0.0.1:6379> GET foo
(error) NOAUTH Authentication required.
127.0.0.1:6379> auth "password"
OK
```

登录远程服务器执行命令

```shell
$ redis-cli -h host -p port -a password
```

查看redis信息

```shell
127.0.0.1:6379> info
```

