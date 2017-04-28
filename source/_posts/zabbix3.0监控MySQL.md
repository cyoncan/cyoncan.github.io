---
title: zabbix配置监控MySQL
date: 2017-04-20 00:16:21
categories:
- 监控
tags:
- zabbix
- mysql
---
<!-- more -->
# zabbix3.0监控MySQL设置

### 1.Agent端建立一个登陆MySQL用户(步骤参考MySQL笔记)

### 2.在zabbix/etc/下面新建.my.cnf文件

```bash
$ find / -name "sock" -print
  /var/lib/mysql/mysql.sock
$ vim /usr/local/zabbix/etc/.my.cnf
  #zabbix Agent
  [mysql]
  host=localhost
  user=zabbix
  password=zabbix
  socket=/var/lib/mysql/mysql.sock
  [mysqladmin]
  host=localhost
  usr=zabbix
  password=zabbix
  socket=/var/lib/mysql/mysql.sock  
```

### 3.编辑 userparameter_mysql.conf 

```bash
$ find / -name userparameter_mysql.conf
/usr/local/src/zabbix-3.0.8/conf/zabbix-agentd/userparameter_mysql.conf
$ cp userparameter_mysql.conf /usr/local/zabbix/etc/zabbix_agentd.conf.d/
$ sed -i 's#/var/lib/zabbix#/usr/local/zabbix/etc#g' 或者用 vim编辑该文件替换掉 home 目录为 .my.cnf 所在的目录
$ vim /usr/local/zabbix/etc/zabbix/zabbix.agentd.conf
  添加一行
  Include=/usr/local/zabbix/etc/zabbix_agentd.conf.d/
```

