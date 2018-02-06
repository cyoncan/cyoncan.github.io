---
title: VPN
date: 2017-05-03 20:56:21
categories:
- Linux
tags:
- vpn
- linux
---

<!-- more -->

**本来不想写这个文档的,过程很简单,之前弄过几次,最近一个梯子快过期了,又找了台非常实惠的vps,准备重新弄个,又是到处找教程,弄来弄去的,想想还是自己记录下过程吧,自己写自己也看得比较明白方便以后自己看.推荐购买vultr和linode.**

通常两种方式上VPN : PPTP和shadowsocks , Linux一般各个发行版本之间的配置类似,我用过Debian7/8和centos6/7, 如果是纯fq推荐使用debian

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

### shadowsocks

可以参考shadowsocks的GitHub文档,非常非常的详细,我就不写了,自己每次用也是看这个,GitHub上去搜索shadowsocks仓库,注意shadowsocks有多种语言版本,c语言版本一般是首选,至于为什么呢!下面贴几条网上普遍认可的,但是他默认并不支持多用户,python版本go版本都是可以支持一个配置文件多用户的.也可以直接上python版的就好了.

c语言版本:

1. 体积小巧。静态编译并打包后只有 100 KB。
2. 高并发。基于 libev 实现的异步 I/O，以及基于线程池的异步 DNS，同时连接数可上万。
3. 低资源占用。几乎不占用 CPU 资源，服务器端内存占用一般在 3MB 左右。

c语言版本,多用户配置:

假如我有以下这三个用户(端口),每个用户(端口)独立一个配置文件.

/etc/shadowsocks-libev/config8388.json

/etc/shadowsocks-libev/config8389.json

/etc/shadowsocks-libev/config8390.json

通过supervisor进程管理软件,能将一个普通的命令行进程变为后台daemon，并监控进程状态，异常退出时能自动重启.

```shell
$ apt-get install supervisor
$ echo_supervisord_conf   输出默认配置文件
$ echo_supervisord_conf > /etc/supervisord.conf   将输出的默认配置项重定向到自定义的配置文件里面
$ vim /etc/supervisord.conf
[program:ss8388]
command:ss-server -c /etc/shadowsocks-libev/config8388.json -u -A
process_name=ss8388
redirect_stderr=true
stdout_logfile_maxbytes=1MB
stdout_logfile_backups=1

[program:ss8389]
command:ss-server -c /etc/shadowsocks-libev/config8389.json -u -A
process_name=ss8389
redirect_stderr=true
stdout_logfile_maxbytes=1MB
stdout_logfile_backups=1

[program:ss8390]
command:ss-server -c /etc/shadowsocks-libev/config8390.json -u -A
process_name=ss8390
redirect_stderr=true
stdout_logfile_maxbytes=1MB
stdout_logfile_backups=1
$ supervisorctl -c /etc/supervisord.conf   进入supervisorctl-shell界面
>status
>start/stop/restart ss8388
>reread
>update
或者下面这种类型
$ supervisorctl status
```

通过脚本,相关命令,下面就是简单的命令执行,也可以写成脚本方便管理.

```shell
$ setsid ss-serevr -c /etc/shadowsocks-libev/config8388.json -u -A
$ setsid ss-serevr -c /etc/shadowsocks-libev/config8389.json -u -A
$ setsid ss-serevr -c /etc/shadowsocks-libev/config8390.json -u -A
$ ps -ef | grep ss-server   可以查看到那启动的三个进程
```

查看vps架构类型

```shell
$ yum install virt-what
$ virt-what
xen
xen-hvm
```

shadowsocksr