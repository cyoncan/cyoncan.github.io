---
title: 云服务器ssh安全设置
date: 2017-05-01 10:58:22
categories:
- Linux
tags:
- ssh
---

<!-- more -->

### ssh安全配置优化:

**1.修改ssh默认端口,改成非标准高端端口 (1024~65535)**

```bash
$ sysctl -a | grep ip_local_port_range   (查看端口范围)
net.ipv4.ip_local_port_range = 1024	65000
$ vim /etc/syscofnig/iptables   (修改防火墙策略ssh 22为自定义端口)
$ vim /etc/ssh/sshd_config   (修改ssh的端口为自定义端口)
$ service sshd restart   (重启生效)
```

**2.禁止直接用root登录ssh,设置用普通账户ssh,然后切换到root.**

```shell
$ vim /etc/ssh/sshd_config
PermitRootLogin no
$ service iptables restart
```

**3.指定ssh连接的ip地址**

```shell
$ vim /etc/hosts.deny
sshd:all:deny
$ vim /etc/hosts.allow
sshd:192.168.2.10:allow
```

**4.xshell使用public key登录ssh**

xshell工具>新建用户密钥生成向导,生成一堆密钥,最好填写上加密密码.

把公钥传到服务器用户目录下面,可用ftp方式上传

```sh
$ cat id_rsa.pub >> authorized_keys   (将上传的公钥追加到authorized里面,没有该文件可以新建)
```

这样就可以不用密码去登录了,注意不同的用户使用,都要分别去给不同用户目录下的~/.ssh追加我们xshell传过去的公钥.