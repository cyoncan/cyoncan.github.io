---
title: Nginx日志分析
date: 2017-07-26 21:18:49
categories:
- Nginx
tags:
- nginx
---

<!-- more -->

##### Ⅰ.日志切割

```shell
#!/bin/bash
D=$(data +%Y%M%D)
mv /usr/local/nginx/logs/access.log ${D}.log
kill -USR1 $(cat /usr/local/nginx/nginx.pid)
```

##### Ⅱ.日志日常分析

```nginx
log_formart main '$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" "$http_x_forwarded_for"';
```

1.访问次数最多的前十IP

```shell
$ awk '{print $1}' access.log | sort | uniq -c | sort -nr -k1 | head -n 10   #uniq去重之前必须先排序,因为不排序的话,uniq无法去重不连续出现的数,sort加不加参数无所谓,只要排序能将一样的IP集中一起即可
```

2.请求总数

```shell
$ wc -l access.log | awk '{print $1}'
```

3.独立IP数

```shell
$ awk '{print $1}' access.log | sort | uniq -c | wc -l
```

4.