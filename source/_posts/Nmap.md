---
title: nmap扫描
date: 2017-05-18 12:50:56
categories:
- CentOS
tags:
- nmap
- linux
---

<!-- more -->

### 1. 获取远程主机的系统类型及开放的端口

```shell
$ nmap -sS -P0 -sV -O <target>	<target>可以为ip/主机名/域名/
-sL   列表扫描
-sT   TCP端口扫描
-sS   TCP同步(SYN)端口扫描半开放隐身扫描
-sU   UDP端口扫描
-sP   Ping扫描
-P0   允许关闭ping进行扫描
-sV   打开系统版本检测
-O    尝试识别远程主机OS
-A    打开操作系统指纹和版本检测
-v    输出详细扫描情况
```

### 2.列出开放了指定端口的主机列表

```shell
nmap -sT -p 80 -oG
```

### 3.寻找所有在线主机

```shell
nmap -sP 192.168.0.0/24
```

