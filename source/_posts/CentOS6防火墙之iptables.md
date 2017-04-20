---
title: iptables����
date: 2017-04-20 00:16:21
categories:
- ����ǽ
tags:
- iptables
- ����ǽ
---

## ����filter��

### 1.���Ԥ���filter�е����й������Ĺ���
```bash
iptables -F
```

### 2.���Ԥ���filter��ʹ�����Զ����еĹ���

```bash
iptables -X
```

### 3.�趨Ԥ�����

```bash
iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD DROP
```

### 4.�����Լ��Ļ�����ʵ�����ȥ������Ӧ�Ķ˿�

```bash
#����22�˿�,ssh���ܵ�¼.
$ iptables -A INPUT -p tcp --dport 22 -j ACCEPT
��
$ iptables -A INPUT -i eth1 -s 192.168.2.100 -p tcp --dport 22 -j ACCEPT #ָ��eth1������192.168.2.100����ssh��¼
#����ping(��icmp��ͨ��)
$ iptables -A INPUT -p icmp -j ACCEPT
#����loopback
$ iptables -A INPUT -i lo -j ACCEPT
#ʹping�������Եõ��ظ�
$ iptables -A INPUT -p udp -m udp --dport 53 -j ACCEPT 
$ iptables -A INPUT -p udp -m udp --sport 53 -j ACCEPT
tip: ��������������ȥ���iptable����,������ vim /etc/sysconfig/iptables
```

### 5.����iptables����

```bash
service iptables save
```

### 6.����iptables����

```bash
service iptables restart
```

### 7.�鿴iptables����

```bash
iptables -L -n
```