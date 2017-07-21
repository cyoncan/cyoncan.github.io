---
title: MySQL备份
date: 2017-07-20  16:26:32
categories:
- MySQL
tags:
- mysql
---

<!-- more -->

开始是纠结mysql做备份的时候要不要给表加锁,加不加锁的依据是什么?查了下资料,综合自己经验看来,加锁是一定没问题的,最保险的做法.但是没搞明白加锁进行备份是仅仅多了一步加锁解锁的步骤还是会影响备份的效率呢!问了一些人,都没怎么关注过这个.记录了一些资料,在做分析

**1.MySQL备份的几种类型**

```
① 热备份:可以正常读写操作,业务正常进行
② 冷备份:不能进行读写操作,数据库需要关闭服务
③ 温备份:可以进行读操作,但是不能进行写的操作
```

**2.MySQL的几种存储引擎**

```mysql
MyISAM , InnoDB , MEMORY , MERGE , CSV , ARCHIVE 
其中MyISAM不支持热备,InnoDB都可以支持.
```

**3.MySQL的几种备份工具**

```shell
① mysqldump: 支持所有的存储引擎,支持温备,完全,部分,对于InnoDB可以热备.
② mysqlhotcopy: 仅支持MyISAM,冷备份.如果表均为MyISAM.可以使用这个
② xtrabackup: InnoDB/XtraDB热备,支持完全,增量
④ lvm2 snpashot: 热备,使用文件系统管理工具进行备份 
```

(1) 用mysqldump备份

```shell
$ mysqldump -uroot -p --single-transaction --databases db1 --tables t1 > /tmp/db1-t1.sql   #--single-transaction此选项能实现热备InnoDB表和库,因此不需要同时使用--lock-all-tables;
$ mysqldump -uroot -p --lock-all-tables --all-databases > /tmp/bak.sql   #备份全部的数据库
$ mysqldump -uroot -p --lock-all-tables --databases db1 db2 > /tmp/db1-db2.sql   #备份多个数据库
$ mysqldump -uroot -p -h=ip1 --databases db1 | mysql -uroot -ppassword -hip2 db2   #ip1机器的db1导入到ip2机器的db2数据库,db2要创建好 
```

锁表这一步也可以在mysql终端里面进行

```mysql
mysql> flush tables with read lock   #刷新锁定全部的库和表.
mysql> flush logs;   #刷新滚动日志,在锁定表后执行.
$ mysqldump -uroot -p --databases db1 > /tmp/db1.sql   #起新终端
mysql> unlock tables
```

mysqldump常用的几个参数:

```shell
--no-data   #只导出表结构不导出数据
--lock-tables   #对所有表加读锁.(默认开启,-skip-lock-tables来关闭)
--lock-all-tables   #锁定所有库中的表,通过在整个dump的过程中持有全局读锁来实现,会自动关闭--single-transactionh和--lock-tables


```

