---
title: CentOS minimal install
date: 2017-03-27 16:08:59
categories:
- CentOS
tags:
- centos
---
<!-- more -->

### 一、centos minimal安装后一些设置

##### 1、设置开机网卡自动连接

```bash
$ vim /etc/sysconfig/network-scripts/ifcfg-eth0
  ONBOOT=yes
```

##### 2、关闭SELinux

```bash
$ /usr/sbin/sestatus -v
$ setenforce 0   (临时关闭)
$ vim /etc/selinux/config
  将 SELINUX=enforcing 改为 SELINUX=disabled   (永久关闭)
  sync reboot
```

##### 3、防火墙

```bash
$ chkconfig iptables --list     (查看)
$ chkconfig iptables on/off     (永久)
$ service iptables start/stop   (临时)
```

###### 3.1、设定预设规则(详情参考:http://t.cn/RiONgR0)

```bash
$ iptables -P INPUT DROP
$ iptables -P OUTPUT ACCEPT
$ iptables -P FORWARD DROP
eg:开启ssh 22端口
$ iptables -A INPUT -p tcp -s 192.168.2.58 --dport 22 -j ACCEPT  (除了192.168.2.58其它IP禁止ssh)
$ iptables -A OUTPUT -p tcp -s 192.168.2.58 --sport 22 -j ACCEPT (如果OUTPUT设置成DROP则需添加该条)
$ service iptables save (手动每条去添加,需要手动保存,不然重启后失效)
或者直接在 iptables 编辑添加删除
$ vim /etc/sysconfig/iptables   (编辑iptables规则)
$ /etc/init.d/iptables restart
```

##### 4、设置yum repo源

```bash
先备份系统源
$ mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
$ cd /etc/yum.repos.d/
$ wget -c http://mirrors.163.com/.help/CentOS6-Base-163.repo
$ vim CentOS-Media.repo
  关闭cdrom源
$ yum install epel   (安装第三方软件源)
```

##### 5 、通过yum group装系统必备软件

```bash
$ yum grouplist
$ yum groupinstall "Development tools"
$ yum groupinstall "System Administration Tools"
(cent7用 yum group mark install)根据需求去选择软件包
$ yum install setuptool.x86_64
$ yum install ntsysv
$ yum install system-config-network-tui
$ yum install system-config-firewall-tui
$ yum install system-config-securitylevel-tui
```

### 二、基础目录结构

    1.mkdir
    /data/src        下载存放源码目录
    /data/log        站点日志目录
    /data/www        WEB站点目录
    /data/svn        SVN仓库目录
    /data/mysqldb    MYSQL数据库数据目录
    /data/backup     MYSQL数据备份目录
    /data/local/php  PHP目录,local下面都是程序编译安装目录
    /data/local/nginx
    /data/local/mysql

### 三、设置禁止参与 yum 更新的核心软件 

```bash
$ vim /etc/yum.conf
  exclude=php* apache* kernel* mysql* nginx* (根据实际情况进行指定不参与yum update的程序)
```
