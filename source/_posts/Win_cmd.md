---
title: Windows_CMD_
date: 2017-10-10  19:51:39
categories:
- Win
tags:
- powershell
---

<!-- more -->

Windows PowerShell和CMD控制台代码页设置

```powershell
代码页     国家（地区）或语言
437        美国 
850        多语言（拉丁文 I）
852        斯拉夫语（拉丁文 II）
855        西里尔文（俄语） 
857        土耳其语
860        葡萄牙语
861        冰岛语
863        加拿大 - 法语
865        日耳曼语
866        俄语
869        现代希腊语 
936        简体中文
950        繁体中文
65001      UTF-8

语法 
chcp [NNN]

通过注册表修改代码页：
1 win键+R打开“运行”对话框，输入regedit打开注册表编辑器。
2 找到 [HKEY_CURRENT_USER\Console\%SystemRoot%_system32_cmd.exe]
3 修改或者新建"CodePage" = dword:000003a8
【注】十六进制"000003a8"或十进制"936"，表示“936 (ANSI/OEM - 简体中文 GBK)”。 
```

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