---
title: CentOS6防火墙之配置iptables
date: 2017-04-20 00:16:21
categories:
- CentOS
tags:
- iptables
- 防火墙
---

# 配置filter表防火墙

### 1.清除预设表filter中的所有规则链的规则
```bash
iptables -F
```

### 2.清除预设表filter中使用者自定链中的规则

```bash
iptables -X
```

### 3.设定预设规则

```bash
iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD DROP
```

### 4.根据自己机器的实际情况开启相应端口

```bash
#开启22端口,ssh才能登录.
$ iptables -A INPUT -p tcp --dport 22 -j ACCEPT
或
$ iptables -A INPUT -i eth1 -s 192.168.2.100 -p tcp --dport 22 -j ACCEPT #指定eth1网卡和192.168.2.100允许ssh登录
#允许ping(即icmp包通过)
$ iptables -A INPUT -p icmp -j ACCEPT
#允许loopback
$ iptables -A INPUT -i lo -j ACCEPT
#使ping域名可以得到回复
$ iptables -A INPUT -p udp -m udp --dport 53 -j ACCEPT 
$ iptables -A INPUT -p udp -m udp --sport 53 -j ACCEPT
tip: 除了以上用命令去添加iptable规则,还可以 vim /etc/sysconfig/iptables
```

### 5.保存iptables配置

```bash
service iptables save
```

### 6.重启iptables服务

```bash
service iptables restart
```

### 7.查看iptables规则

```bash
iptables -L -n
```