---
title: VPN
date: 2017-05-03 20:56:21
categories:
- VPN
tags:
- vpn
---

<!-- more -->

**本来不想写这个文档的,过程很简单,之前弄过几次,最近一个梯子快过期了,又找了台非常实惠的vps,准备重新弄个,又是到处找教程,弄来弄去的,想想还是自己记录下过程吧,自己写自己也看得比较明白方便以后自己看.推荐购买vultr和linode.**

通常两种方式上VPN : PPTP和shadowsocks , Linux一般各个发行版本之间的配置类似,我用过Debian和centos6/7

### PPTP拨号方式

1.检测系统环境,满足以下三条检测,否则装pptp无效,可以去装openVPN

```bash
# 查看内核是否支持MPPE,显示ok表明通过.否则需要安装kernel-devel
➜  ~ modprobe ppp-compress-18 && echo OK
# 执行下面两条命令,得到响应结果和下面的一样.就可以接下面的步骤安装PPTP
➜  ~ cat /dev/ppp   检测是否开启ppp支持
cat: /dev/ppp: No such device or address
➜  ~ cat /dev/net/tun   检测是否开启net/tun支持
cat: /dev/net/tun: File descriptor in bad state
```

2.安装相应组件,关闭SELinux(一般都默认关闭,否则手动关闭)

```bash
➜  ~ yum install epel
➜  ~ yum makecache fast
➜  ~ yum -y install ppp
➜  ~ yum -y install pptpd
```

3.编辑相关配置

```bash
➜  ~ vim /etc/pptpd.conf
 # 去掉末尾的这两行注释,有说明,如果该地址段与内网地址有冲突,需要把这里的改下.
 localip 192.168.0.1
 remoteip 192.168.0.234-238,192.168.0.245
```

```bash
➜  ~ vim /etc/ppp/options.pptpd
# 修改ms-dns字段
ms-dns 8.8.8.8
ms-dns 8.8.4.4
```

4.设置VPN拨号账号密码

```bash
➜  ~ vim /etc/ppp/chap-secrets
# 按照给你的格式去写
# client   server   secret   IP address
hello pptpd helloworld *
```

5.修改内核参数

```bash
➜  ~ vim /etc/sysctl.conf
# 在末尾添加一下或者取消现有的注释
net.ipv4.ip_forward=1
# 运行下面的命令使之生效
➜  ~ sysctl -p
```

6.放行服务端口或者直接关了防火墙

```bash
# centos7是这么添加的,其他版本Linux参考百度
➜  ~ firewall-cmd --zone=public --add-port=80/tcp --permanent
➜  ~ firewall-cmd --reload
```

