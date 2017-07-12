---
title: CentOS配置iptables
date: 2017-04-15 20:31:16
categories:
- CentOS
tags:
- iptables
---
<!-- more -->

# 一、iptables表和链的结构(四表五链)

### 1.四张表处理优先级: raw>mangle>nat>filter

**raw:** 优先级最高, 设置raw一般是为了不再让iptables做数据包的跟踪链接处理, 提高性能.

**mangle:** 用于对特定数据包的修改.

**nat:** 用于nat功能端口或者地址映射.

**filter:** 一般的过滤功能.

### 2.五链：INPUT, FORWARD, OUTPUT, PREROUTING, POSTROUTING

**raw表**中的链有: PREROUTING, OUTPUT

**mangle表**中的链有: PREROUTING, INPUT, FORWARD, OUTPUT, POSTROUTING

**nat表**中的链有: PREROUTING, POSTROUTING, OUTPUT

**filter表**中的链有: INPUT, FORWARD, OUTPUT

### 3.iptables常用参数: 

tip: iptable -h都能看到对应的解释

**规则增删改查:**

| 参数   | 作用                              |
| :--- | :------------------------------ |
| -A   | 在规则链的末尾加入新规则                    |
| -I   | 在规则链的头部加入新规则                    |
| -D   | 删除                              |
| -R   | 修改                              |
| -L   | 查看                              |
| -P   | 设置默认策略 , iptables -P INPUT DROP |
| -F   | 清空默认规则链                         |
| -X   | 删除自定义空链                         |

**常用参数:**

| 参数      | 作用                                       |
| :------ | :--------------------------------------- |
| -p      | 指定协议 tcp/udp/icmp                        |
| -s      | 指定源地址 ip/mask , 加叹号 "!" 表示相反的意思          |
| -d      | 匹配目标地址                                   |
| --sport | 匹配来源端口号                                  |
| --dport | 匹配目端口号                                   |
| -i      | 匹配从这块网卡流入的数据                             |
| -o      | 匹配从这块网卡流出的数据                             |
| -m      | 加载模块                                     |
| -t      | 指定表, 默认filter表. iptables -L - nat/mangle/raw |
| -j      | 指定处理的动作 , ACCEPT/DROP                    |

**常用处理动作:**

| 动作         | 作用                                       |
| :--------- | :--------------------------------------- |
| ACCEPT     | 允许封包通过,:将数据包放行,进行完此动作后,不再对比其他规则,直接跳往下一个规则链. |
| DROP       | 丢弃封包,响应超时,对方无法判断主机是否在线或者流量被拒绝,不再对比其他规则,中断过滤. |
| REJECT     | 拒绝封包通过,并将数据包封装,返回消息,对方看到主机口不可达.          |
| REDIRECT   | 将包重定向到另一个端口,之后继续对比其他规则.                  |
| MASQUERADE | 改写封包来源ip为防火墙NIC ip , 可指定port范围 , 之后跳往下一规则. |
| SNAT       | 改写封包来源ip为某特定ip或ip范围 , 可指定 port 范围 , 之后跳往下一规则. |
| DNAT       | 改写封包目的ip为某特定ip或ip范围, 可指定port范围 , 之后跳往下一规则. |

# 二、配置filter表防火墙

###### tip: 一般设置默认规则,INPUT链和FORWARD链为DROP , OUTPUT链为ACCEPT

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
# 允许回环loopback访问
$ iptables -A INPUT -i lo -j ACCEPT
$ iptables -A OUTPUT -o lo -j ACCEPT   (output链的默认策略设置为DROP时,需要添加这条,以下针对的每个端口同此一样.)
#开启22端口,ssh才能登录.
$ iptables -A INPUT -p tcp --dport 22 -j ACCEPT
$ iptables -A OUTPUT -p tcp --dport 22 -j ACCEPT
或者
$ iptables -A INPUT -i eth0 -s 192.168.2.100 -p tcp --dport 22 -j ACCEPT #指定eth1网卡和192.168.2.100允许ssh登录
$ iptables -A OUTPUT -o eth0 -s 192.168.2.100 -p tcp --dport 22 -j ACCEPT
#允许ping(即icmp包通过)
$ iptables -A INPUT -p icmp -j ACCEPT
$ iptables -A OUTPUT -p icmp -j ACCEPT
#使ping域名可以得到响应
$ iptables -A INPUT -p udp --sport 53 -j ACCEPT
$ iptables -A OUTPUT -p udp --dport 53 -j ACCEPT
$ iptables -A INPUT -p udp --dport 53 -j ACCEPT
$ iptables -A OUTPUT -p udp --sport 53 -j ACCEPT
tip: 除了以上用命令去添加规则,还可以用编辑文件的方式 vim /etc/sysconfig/iptables
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

### 8.iptables优化:

请求比较频繁的放在最上面,请求频率较小的放在最后面.这里整理关于防火墙的东西,不是具体的知识,更多工操作使用,具体的防火墙知识,还需要去阅读参考网上写的各种文章,需要多读多看多试,才能理解深入.