---
title: Linux常用命令
date: 2017-04-20 00:16:21
categories:
- Linux
tags:
- 命令
---
<!-- more -->
### 1.Linux查看当前系统登录用户列表

```bash
w               查看当前活跃的用户列表
cat /etc/passwd 查看所有用户的列表
cat /etc/group  查看用户组
cat /etc/passwd|grep -v nologin|grep -v halt|grep -v shutdown|awk -F":" '{ print $1"|"$3"|"$4 }'|more
```

### 2.查看进程

```bash
ps -aux              所有进程
ps -ef | grep nginx  指定进程
```

### 3.mkdir创建目录及字目录
    mkdir -p /home/xx/xxx/xxxx/xxxxx
          -v //打印创建过程
### 4.chmod更改文件文件夹权限
```bash
chmod -R //权限设置往下递归
```

### 5.查看系统版本
```bash
lsb_release -a
uname -a
cat /etc/issue
```

### 6.控制用户登录

```bash
锁模式
usermod -L user (Lock 帐号user)
usermod -U user (Unlock 帐号user)
控制shell模式
usermod -s /sbin/nologin user  (不允许登录)
usermod -s /bin/bash user      (允许登录使用指定的bash)
/etc/nologin.txt               (提示用户为什么不能登录)
```

### 7.禁止所有用户登录

```bash
touch /etc/nologin
如果该文件存在，那么Linux上的所有用户（除了root以外）都无法登录.nologin（注意：不是nologin）可以写点东西，告诉用户为何无法登录.
cat /etc/nologin
9：00－10：00 系统升级，所有用户都禁止登录！
解禁帐号也简单，直接将/etc/nologin删除就行了！
```

### 8.查看当前用户使用的shell/终端环境

```bash
ps | grep $$ | awk '{print $4}'
echo $0
echo $TERM
```

### 9.查看端口被进程占用，属于哪个程序
```bash
lsof -i :9000
```

### 10.wintolinux
```bash
dos2unix unix2dos
```

### 11.rename使用
rename

### 12.查找文件名,并替换其中的指定字符.

```shell
find /data/www -name "*.php" | xargs sed -i 's/192.168.2.145/127.0.0.1/g' 
```

### 13.批量修改目录后缀名

###### tip: rename有c版本和Perl版本,使用时请man一下

```shell
find ./ -type d -name "*.org"|xargs rename org com
```

### 14.查找目录下所有文件中包含指定的字符,并替换成其它字符

```shell

```

