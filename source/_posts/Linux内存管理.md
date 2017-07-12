---
title: Linux内存管理
date: 2017-04-26 19:21:35
categories:
- CentOS
tags:
- linux
---
<!-- more -->
**释放内存**

```bash
$ sync
$ free -m
             total       used       free     shared    buffers     cached
Mem:          7752       1590       6162          2        274        457
-/+ buffers/cache:        858       6894
Swap:         7887          0       7887
$ echo 1 > /proc/sys/vm/drop_caches   释放 pagecache
$ echo 2 > /proc/sys/vm/drop_caches   释放 dentries和inodes
$ echo 3 > /proc/sys/vm/drop_caches   释放 pagecache, dentries和inodes
```

