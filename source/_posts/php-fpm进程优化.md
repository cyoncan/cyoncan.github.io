---
title: php-fpm内存问题
date: 2017-06-22  21:52:16
categories:
- PHP
tags:
- php-fpm
- ulimit
---

<!-- more -->

针对nginx+php-fpm在使用的过程中php-fpm进程数不释放内存给系统导致内存报警的问题，本来应该是通过优化相关配置文件去调节的，但配置文件已经调优了，需要一段时间等待观察。就用种简单粗暴的办法通过crontab定时脚本来定时每隔一段时间重启php-fpm,这样也可以达到释放内存的目的.但是网站如果访问量不稳定的出现,这种办法就不是很有效了.所以还是需要研究下配置文件的相关调优.

配置文件相关:

php-fpm.conf

```shell
pm.max_request   该值是指发送多少个请求后会重启改线程.
pm.max_chlidren   每次php-fpm会建立多少个进程.
```

ulimit值的配置

```shell
$ ulimit -a   查看所有限制值
-t: cpu time (seconds)         unlimited
-f: file size (blocks)         unlimited
-d: data seg size (kbytes)     unlimited
-s: stack size (kbytes)        10240
-c: core file size (blocks)    0
-m: resident set size (kbytes) unlimited
-u: processes                  62794
-n: file descriptors           51200       系统对每一个进程打开文件描述符的数量
-l: locked-in-memory size (kb) 64
-v: address space (kb)         unlimited
-x: file locks                 unlimited
-i: pending signals            62794
-q: bytes in POSIX msg queues  819200
-e: max nice                   0
-r: max rt priority            0
$ vim /etc/security/limits.conf   修改ulimit -n 值的大小,php-fpm里面的rlimit值和该值大小相等即可.
```

参考文章: http://t.cn/RzlwzM5