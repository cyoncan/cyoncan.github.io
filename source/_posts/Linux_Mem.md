---
title: Linux内存管理
date: 2017-04-26 19:21:35
categories:
- Linux
tags:
- linux
---
<!-- more -->
**释放内存**

```bash
$ sync
$ free -m
             total       used       free     shared    buffers     cached
Mem:          7869       6660       1208          0        741       1174
-/+ buffers/cache:       4743       3125
Swap:            0          0          0

$ echo 1 > /proc/sys/vm/drop_caches   #释放 pagecache
$ echo 2 > /proc/sys/vm/drop_caches   #释放 dentries和inodes
$ echo 3 > /proc/sys/vm/drop_caches   #释放 pagecache, dentries和inodes
# 获取空闲内存
$ free -m | grep - | awk -F : '{print $2}' | awk '{print $2}'
```

