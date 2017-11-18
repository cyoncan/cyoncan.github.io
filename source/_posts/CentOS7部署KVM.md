---
title: CentOS部署KVM
date: 2017-09-18  16:26:16
categories:
- 虚拟化
tags:
- kvm
---
<!-- more -->

有一段时间比较，刚好利用公司有一台空闲的服务器，配置还行，刚好也在学习虚拟化的一些东西。最初从esxi入门，了解了虚拟化产品大概的原理和一些操作。当然对于最重要的功能集群管理，我还没有去研究。于是趁着这台机器还空闲，就试试看kvm的东西。

安装kvm前提：需要验证机器是否支持虚拟化，官方文档上是这么说的【主机必须使用支持硬件辅助虚拟化的Intel VT或AMD-V芯片组。】其实整个kvm安装使用的步骤在[官方文档](https://www.linux-kvm.org/page/HOWTO#General)都写的很清楚，只是英文理解着比较慢。

环境：centos7

##### 1.验证CPU是否支持KVM

```bash
$ grep -E 'vmx|svm' /proc/cpuinfo
- vmx is for Intel processors
- svm is for AMD processors
```

如果有结果输出，则表示支持。vmx处理器功能标志表示Intel VT芯片组，而svm标志表示AMD-V。

##### 2.安装配置kvm及其相关软件

```bash
# 安装kvm及相关软件包
$ yum install qemu-kvm kvm libvirt libvirt-python libguestfs-tools virt-install
# 加载kvm模块
$ modprobe kvm
# 加载特定芯片（cpu）的kvm模块
# For the AMD chip (svm flag):
$ modprobe kvm-amd
# For Intel chip (vmx flag):
$ modprobe kvm-intel
# 验证内核模块是否加载
$ lsmod | grep kvm
kvm_intel             200704  0 
kvm                   589824  1 kvm_intel
irqbypass              16384  1 kvm
# 启动 libvirtd
$ systemctl enable libvirtd.service && systemctl start libvirtd.service
```

##### 3.网络

```bash
# 验证安装的虚拟网卡是否有效
$ ip link show virbr0	有输出信息代表可以了
官方文档上是这么说的：“你可以使用默认的网络设置，或在主机中设置一个网桥。默认网络只允许来自KVM访客的出站通信。如果KVM访客需要完整的网络访问权限，包括与外部主机的通信，则在主机中设置一个Linux网桥。”
# 设置网桥
$ cp /etc/sysconfig/network-scripts/ifcfg-eno1 /root/.
$ cd /etc/sysconfig/network-scripts/
$ cp ifcfg-eno1 ifcfg-br0
$ vim ifcfg-br0 #内容如下
DEVICE=br0
ONBOOT=yes
TYPE=Bridge	#桥是区分大小写的，必须大写“B”和小写“ridge”。
BOOTPROTO=static
NM_CONTROLLED=no #指定网卡不受网络管理员控制。为了使网桥正常工作，Network Manager只能控制一台设备。
IPADDR=192.168.2.80
NETMASK=255.255.255.0 
GATEWAY=192.168.2.1
DNS1=119.29.29.29
# 网桥里面不应该出现硬件地址，网桥只是充当eno1的扩展。
$ systemctl restart network.service
```

两个文件主要配置地方的对比

| ifcfg-eno1               | ifcfg-br0             |
| ------------------------ | --------------------- |
| DEVICE=eno1              | DEVICE=br0            |
| TYPE=Ethernet            | TYPE=Bridge           |
| NM_CONTROLLED=no         | NM_CONTROLLED=no      |
| ONBOOT=yes               | BOOTPROTO=static      |
|                          | IPADDR=192.168.2.80   |
| BRIDGE=br0 把eno1和br0连接起来 | NETMASK=255.255.255.0 |
|                          | ONBOOT=yes            |

######3.1.禁用桥接通信中的Netfilter处理：

```bash
$ cat /etc/sysctl.conf
net.bridge.bridge-nf-call-ip6tables = 0
net.bridge.bridge-nf-call-iptables = 0
net.bridge.bridge-nf-call-arptables = 0
net.ipv4.ip_forward = 1 #开启网络转发
$ sysctl -p
```

##### 4.创建虚拟机

关闭SELinux，如果不能关闭，请参考[官方文档](http://linux.dell.com/files/whitepapers/KVM_Virtualization_in_RHEL_7_Made_Easy.pdf)的做法。

```bash
# 有两种方法安装虚拟机
①virt-manager: a GUI tool
②virt-install: a command line tool.
# 创建存储池，虚拟机映像的存放位置
$ mkdir /home/kvm/image
$ virt-install \
       --network bridge:br0
       --name kvm1 \
       --ram 1024 \
       --vcpus 1\
       --disk path=/home/kvm/image/kvm1.img,size=20 \
       --graphics none \
       --localtion=/home/CentOS7-x86_64-DVD.iso
       --extra-args="console=tty0 console=ttyS0,115200"
# 安装的过程出现报错什么的请自己去Google解决。
```

###### 4.1虚拟机常用指令

```bash
# 列出所有正在运行的主机
$ virsh list --all
# 显示虚拟机信息
$ virsh dominfo kvm1
# 显示虚拟机内存CPU使用量信息
$ virt-top
#显示虚拟机磁盘的使用量
$ virt-df kvm1
```



#####5.选择一款web管理kvm的软件（WebVirtMgr）（可以选择用centos桌面环境去管理，个人爱好了，我受不了一台服务器桌面化，在服务器上操作。）

```bash
内容比较简单，它在github上面有文档说明。https://github.com/retspen/webvirtmgr/wiki
```

