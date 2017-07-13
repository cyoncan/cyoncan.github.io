---
title: vsftp安装使用
date: 2017-03-19 23:44:21
categories:
- FTP
tags:
- ftp
- vsftp
---
<!-- more -->
### 1.安装(源码编译或者yum install)

```bash
$ yum -y install vsftpd
$ service vsftpd start
$ chkconfig vsftpd on
$ yum install -y  db4 db4-utils (基于虚拟账户权限认证依赖)
```

##### 目录文件说明

  /etc/vsftpd/vsftpd.conf              vsftpd的核心配置文件
  /etc/vsftpd/ftpusers                 用于指定哪些用户不能访问FTP服务器
  /etc/vsftpd/user_list                指定允许使用vsftpd的用户列表文件
  /etc/vsftpd/vsftpd_conf_migrate.sh   是vsftpd操作的一些变量和设置脚本
  /var/ftp/                            默认情况下匿名用户的根目录

=================================基于虚拟账户权限认证====================================

_sftpd的虚拟用户采用单独的用户名/口令保存方式，与系统账户分离，很大程度上增强了系统的安全性.vsftpd可以采用数据库文件来保存用户/口令，eg:hash;也可以将用户/口令保存在数据库服务器中，eg:MySQL。vsftpd验证虚拟用户，则采用PAM方式._

=======================================================================================

### 2.创建虚拟用户账号和密码

 (奇数行为用户名，偶数行为用户密码）

```bash
$ vim /etc/vsftpd/virtual.users
  ftpuser  (虚拟用户名)
  123456   (用户口令)
```

生成虚拟用户认证的db文件

```bash
$ db_load -T -t hash -f /etc/vsftpd/virtual.users /etc/vsftpd/vsftpd.login.db
$ chmod 600 /etc/vsftpd/vsftpd.login.db
```

### 3.配置PAM信息

```bash
$ vim /etc/pam.d/vsftpd.pam
auth required /lib64/security/pam_userdb.so db=/etc/vsftpd/vsftpd.login
account required /lib64/security/pam_userdb.so db=/etc/vsftpd/vsftpd.login
```

###  4.配置vsftpd.conf

```bash
$ cp /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.bak
$ vim /etc/vsftpd/vsftpd.conf
  anonymous_enable=NO
  local_enable=YES
  write_enable=YES
  local_umask=022
  anon_umask=022   
  xferlog_enable=YES
  connect_from_port_20=YES
  xferlog_std_format=YES
  ascii_upload_enable=YES
  ascii_download_enable=YES
  ftpd_banner=Welcome to blah FTP service.
  chroot_local_user=YES
  listen=YES
  pam_service_name=vsftpd.pam
  userlist_enable=YES
  tcp_wrappers=YES
  guest_username=ftp
  guest_enable=YES
  user_config_dir=/etc/vsftpd/vsftpd_user_conf
  pasv_min_port=30001
  pasv_max_port=31000
```

###  5.创建用户名的配置文件

```bash
$ mkdir -p /etc/vsftpd/vsftpd_user_conf
$ cd /etc/vsftpd/vsftpd_user_conf
$ vim ftpuser (ftpuser文件名就是上面创建虚拟用户名字)
　local_root=/data/www
　write_enable=yes
　download_enable=yes
　anon_upload_enable=yes
　anon_mkdir_write_enable=yes
　anon_other_write_enable=yes
　anon_world_readable_only=no
　idle_session_timeout=600
　data_connection_timeout=120
　max_clients=2
　max_per_ip=3
　local_max_rate=512000 (拥有全部权限)
```

###  6.日常管理虚拟用户账号和密码

###### i.修改文件

   ```bash
$ vi /etc/vsftpd/virtual.users
   ```

###### ii.生成虚拟用户认证的db文件

   ```bash
db_load -T -t hash -f /etc/vsftpd/virtual.users /etc/vsftpd/vsftpd.login.db
   ```

###### iii.重启ftp服务(能登录就不重启)

   ```bash
service vsftpd restart
   ```

### 7.添加防火墙规则

```shell
-A INPUT -p tcp -m state --state NEW -m tcp --dport 20 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 21 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 30001:31000 -j ACCEPT
```

