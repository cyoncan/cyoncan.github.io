---
title: Linux查看网卡信息
date: 2017-03-23 22:16:21
categories:
- 系统硬件
tags:
- 网卡
- 驱动
---

### CentOS网卡相关信息查看

##### 1.首先安装工具,已安装请忽略.

```bash
$ yum install pciutiles
```
##### 2.查看网卡型号

```bash
$ lspci | grep -i ethernet
```
##### 3.查看网卡驱动版本

```bash
$ ethtool -i eth0
```
##### 4.网卡驱动安装

```bash
$ yum install kernel-devel kernel-headers  
$ yum install gcc
$ tar zxvf r1000.tgz
$ cd r1000/
$ make clean modules
$ make install
$ modprobe r1000
$ reboot
$ lsmod | grep r1000 或者 ifconfig -a 查看网卡eth0是否加载
$ service network restart
假如解压出来进文件夹看到有autorun.sh文件的,只需要执行./autorun.sh就可以了,等待完成,重启查看即可.
```

​