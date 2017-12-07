---
title: tcp_bbr
date: 2017-07-12  19:53:43
categories:
- CentOS
tags:
- linux
---

<!-- more -->

**TCP BBR是Google开源的拥塞控制算法,在Linux4.11内核版本上已经进行使用,最低内核要求4.9**

## Debian8/Ubuntu

```shell
$ wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.12.1/linux-headers-4.12.1-041201-generic_4.12.1-041201.201707121132_amd64.deb
$ dpkg -i linux-headers-4.12.1-041201-generic_4.12.1-041201.201707121132_amd64.deb
$ update-grub
$ reboot
```

## CentOS 7

```shell
$ rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
$ rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
$ yum --enablerepo=elrepo-kernel install kernel-ml -y
$ rpm -qa | grep kernel
$ rpm -ev    #删除旧内核
$ egrep ^menuentry /etc/grub2.cfg | cut -f 2 -d \'
$ grub2-set-default 0  #default 0表示第一个内核设置为默认运行, 选择最新内核就对了
$ reboot
```

重新启动后，如果会出现“read-only file system” 的错误，root账户下执行mount -o remount rw / 即可

## 开启BBR

```shell
$ uname -r    # 查看内核是刚刚安装的版本
```

执行 `lsmod | grep bbr`，结果中如果没有 `tcp_bbr` 就先执行

```shell
$ modprobe tcp_bbr
$ echo "tcp_bbr" >> /etc/modules-load.d/modules.conf
$ echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
$ echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
$ sysctl -p
```

执行

```shell
$ sysctl net.ipv4.tcp_available_congestion_control
$ sysctl net.ipv4.tcp_congestion_control
```

结果如果都有bbr,则内核已开启bbr,