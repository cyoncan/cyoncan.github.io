---
title: Windows网络常用命令
date: 2017-03-23 20:56:21
categories:
- Windows
tags:
- cmd
---

CMD局域网命令

```powershell
arp -a 列出本网段内所有活跃的IP地址
arp -a 加对方IP是查对方的MAC地址
arp -s （ip + mac）绑定mac与ip地址
arp -d （ip + mac）解绑mac与ip地址

net view ——> 查询同一域内机器列表
net view /domain ——> 查询域列表
net view /domain:domainname —–> 查看workgroup域中计算机列表

ipconfig /all ——> 查询本机IP段，所在域等
ipconfig /release
ipconfig /renew 重新获取Ip地址

telnet ip 端口号：尝试能否打开链接远程主机端口 nbtstat -a 加对方IP查对方的主机名
tracert 主机名 得到IP地址

netstat -a -n
netstat -an | find “3389”
netstat -a查看开启哪些端口
netstat -n查看端口的网络连接情况
netstat -v查看正在进行的工作
netstat -p tcp/ip查看某协议使用情况
netstat -s 查看正在使用的所有协议使用情况

nbtstat -n 获取NetBIOS
nslookup 域名 查询域名对应的ip
```