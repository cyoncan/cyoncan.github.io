---
title: Hype-v Linux虚拟机同一物理机xshell无法ssh登录管理
date: 2017-07-22 19:29:11
category:
- 虚拟化
tags:
- hype-v
---

<!-- more -->

在Hype-v安装完Linux虚拟机后,无法用物理机上的ssh登录管理Linux.虚拟机需要安装微软提供相关的驱动.

### 下载Linux Integration Services

```shell
https://www.microsoft.com/en-us/download/details.aspx?id=51612
将下载的ISO文件挂载到虚拟机
$ mkdir -p /mnt/cdrom
$ mount /dev/cdrom /mnt/cdrom
$ cd /mnt/cdrom
$ ./install.sh
进入相关的网卡文件设置固定ip即可
```

**hype-v 导出虚拟机**

```powershell
以管理员身份运行powershell,输入如下命令,VMName为你创建虚拟机的名字
$VmName = "CentOS7"
$Destination = "F:\vm-export"
 Stop-VM -Name $VmName
 Export-VM -Name $VmName -Path $Dstination
```

**导入到另一台物理机上的hype-v,复制虚拟机依赖的是$Destination目录下的vhdx格式文件。换一台物理机,开始虚拟机镜像的安装。**

```powershell
创建Internal vSwitch，通过PowerShell：
$HyperVVirtualSwitchName = "ISInternalSwitch"
 New-VMSwitch -Name $HyperVVirtualSwitchName -SwitchType Internal
 Get-NetAdapter | where { $_.Name -Match $HyperVVirtualSwitchName } | New-NetIPAddress -IPAddress "192.168.1.1" -PrefixLength 24
 
基于导出的CentOS7镜像和vSwitch，创建新的实例。键入PowerShell：
$VMName = "CentOS7"
$HyperVVirtualSwitchName = "ISInternalSwitch"
$VMPath = "F:\vm-import\$VMName"
$VHDX = "$VMPath\Virtual Hard Disks\CentOS7-TensorFlow10.vhdx"
 New-VM -Name $VMName -SwitchName $HyperVVirtualSwitchName -Path $VMPath -VHDPath $VHDX -Generation 1
 Start-VM -Name $VMName

设置端口转发和开启防火墙。键入Windows批处理命令：
netsh interface portproxy delete v4tov4 listenport=22
netsh interface portproxy add v4tov4 listenport=22 connectport=22 connectaddress=192.168.1.100
netsh advfirewall firewall delete rule name="CentOS_SSH"
netsh advfirewall firewall add rule name="CentOS_SSH" protocol=TCP dir=in localport=22 action=allow
```

