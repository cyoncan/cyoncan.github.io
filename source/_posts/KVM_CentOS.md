---
title: CentOS7_KVM
date: 2017-09-18  16:26:16
categories:
- Virtual
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
$ yum install qemu-kvm kvm libvirt libvirt-python libkvmfs-tools virt-install
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
$ virsh #不加任何参数直接进入交互式控制终端
	pool-define-as kvm_pool --type dir --target /home/kvm/image #定义一个存储池绑定目录，如果是lvm卷类型改成logical，还其它的自己查
	pool-build kvm_pool #建立存储池
	pool-start kvm_pool #启动存储池
	pool-list --all #验证该存储池是否启动
	pool-autostart kvm_pool #存储池开机自动运行
	pool-info kvm_pool #查看存储池信息

# 移除存储池
$ virsh
	pool-destroy kvm_pool #销毁存储池
	pool-undefine kvm_pool #取消定义存储池
	pool-delete kvm_pool #删除存储池

# 创建卷
$ virsh
	vol-create-as --pool kvm_pool --name disk0.img --capacity 50G --allocation 1G --format raw
# 或者用qemu-img
$ qemu-img create -f qcow2 disk0.img 50G
# raw和qcow2格式之间的转换
$ qemu-img convert -f raw -O qcow2 test.raw test.raw.qcow2
# raw格式镜像增加减少大小(qcow2不可以)
$ qemu-img resize disk0.img -10G/+10G

# 命令安装虚拟机(至少要带的参数有 --name,--ram,--disk/filesystem/nodisk,安装选项)
$ virt-install \
       --virt-type=kvm \
       --name=centos7.0 \
       --vcpus=2 \
       --memory=2048 \
       --location=/home/iso/centos7.iso \
       --disk path=/home/kvm/image/centos7.img \
       --network bridge:br0 \
       --graphics none \
       --extra-args="console=ttyS0" \
       --force
# qemu安装方式
$ qemu-system-x86_64 -m 2048 -smp 1 -enable-kvm centos7.img -cdrom /home/iso/centos.iso
# 安装的过程出现报错什么的请自己去Google解决。
# 克隆导出虚拟机
$ virsh	dumpxml centos7.0 > /home/kvm/vm-demo.xml
$ cp /home/kvm/vm-demo.xml /home/kvm/centos7.0a1.xml
# xml文件需要做些修改在以下的几个参数：（name、uuid、镜像地址、MAC地址、）uuid用uuidgen生成
# MAC地址生成
$ openssl rand -hex 6 | sed 's/..\B/&:/g'
# 注册新虚拟机
$ virsh define /home/kvm/centos7.0a1.xml
```

###### 4.1虚拟机常用指令（虚拟机=客户机=访客）

| 命令选项                              |                       |
| --------------------------------- | --------------------- |
| virsh connect                     | 连接至 KVM 管理程序          |
| virsh pool-list --all             | 列出所有的存储池              |
| virsh list --all                  | 列出本地所有的虚拟主机           |
| virsh start kvm_name --console    | 启动不处于活动状态的虚拟主机，并进入控制台 |
| virsh autostart kvm_name          | 虚拟主机随宿主机启动而自启动        |
| virsh autostart -disable kvm_name | 禁止虚拟主机随宿主机启动后自启动      |
| virsh shutdown kvm_name           | 关闭处于活动状态的虚拟主机         |
| virsh reboot kvm_name             | 重启虚拟主机                |
| virsh suspend kvm_name            | 暂停虚拟主机                |
| virsh resume kvm_name             | 启动暂停的虚拟主机             |
| virsh destroy kvm_name            | 强制关闭虚拟主机（类似断电）        |
| virsh dumpxml kvm_name            | 显示虚拟主机的当前配置           |
| virsh dominfo kvm_name            | 显示虚拟主机信息              |
| virsh domid kvm_name              | 显示虚拟主机id              |
| virt-top                          | 显示虚拟主机内存CPU使用量        |
| virt-df kvm_name                  | 显示虚拟主机磁盘的使用量          |
| virsh                             | 进入虚拟化控制台              |
|                                   |                       |
|                                   |                       |
|                                   |                       |

#####5.选择一款web管理kvm的软件（WebVirtMgr）（可以选择用linux桌面环境用相关的软件管理virt-manger，我也试过这个操作就是点点了，看个人爱好了，我受不了一台服务器有桌面环境，乐意在服务器上捣腾。）

```bash
内容比较简单，它在github上面有文档说明。https://github.com/retspen/webvirtmgr/wiki
```

