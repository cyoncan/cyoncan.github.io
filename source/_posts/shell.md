---
title: Shell
date: 2017-04-6 12:33:29
categories:
- Shell
tags:
- shell
---
<!-- more -->

##### win2linux

```bash
dos2unix unix2dos
```

##### 替换[rename]

rename分两个版本,C语言和Perl语言.

```shell
将当前的目录下的所有.txt后缀的文件全部替换成.py
centos
$ rename .txt .py *.txt
ubuntu
$ rename 's/\.txt/\.py\' *
```

##### 批量替换文件中的字符串[sed]

```shell
$ sed -i 's/str2/str3/g' `grep -rl ./`
```
##### 查找文件名,并替换其中的指定字符.

```shell
find /data/www -name "*.php" | xargs sed -i 's/192.168.2.145/127.0.0.1/g' 
```

##### 批量修改目录后缀名

###### tip: rename有c版本和Perl版本,使用时请man一下

```shell
find ./ -type d -name "*.org"|xargs rename org com
```

##### 查看空闲内存

```bash
free -m | grep - | awk -F : '{print $2}' | awk '{print $2}'
```

