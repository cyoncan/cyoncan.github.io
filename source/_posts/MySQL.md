---
title: MySQL
date: 2017-03-27 16:10:59
categories:
- MySQL
tags:
- mysql
---
<!-- more -->
## MySQL

### 1.安装Cmake

```bash
$ yum -y install gcc gcc-c++ make autoconf libtool
$ cd /data/src
$ wget -c http://www.cmake.org/files/v3.7/cmake-3.7.2.tar.gz
$ tar -zxvf cmake-3.7.2.tar.gz
$ cd cmake-3.7.2
$ ./bootstrap
$ make && make install
```

### 2.安装MySQL

```bash
$ yum -y install gcc gcc-c++ make autoconf libtool-ltdl-devel gd-devel freetype-devel libxml2-devel libjpeg-devel libpng-devel openssl-devel curl-devel bison patch unzip libmcrypt-devel libmhash-devel ncurses-devel bzip2 flex libaio-devel
$ groupadd mysql
$ useradd -r -g mysql -s /sbin/nologin mysql
$ wget -c http://mirrors.sohu.com/mysql/MySQL-5.6/mysql-5.6.35.tar.gz
$ tar mysql-5.6.35.tar.gz
$ cd mysql-5.6.35
$ cmake -DCMAKE_INSTALL_PREFIX=/data/local/mysql/ -DMYSQL_DATADIR=/data/mysqldb -DMYSQL_TCP_PORT=3306 –enable-thread-safe-client
(cmake安装参数参考MySQL官方介绍 http://t.cn/RipwTjB)
$ make && make install
$ /data/local/mysql/bin/mysql --version
$ cd /data/local/mysql
$ chown -R mysql .  (更改当前目录(mysql)下所有及子目录属mysql用户.因为是通过root用户进行安装的，权限属于root)
$ chgrp -R mysql .
$ scripts/mysql_install_db --user=mysql --basedir=/data/local/mysql --datadir=/data/mysqldb
(初始化数据库,确保数据库目录和文件为mysql账户拥有,确保以root用户执行mysql_install_db,--basedir --datadir 分别指定MySQL安装目录和数据库目录，根据需求指定或者默认.)
$ chown -R root .  (改回root或者不改均可)
$ chown -R mysql data  (data数据库目录必须为mysql账户拥有,如果数据库目录更改也要授予mysql权限. tip：有些系统或者分发MySQL可能不是data，是var之类的.根据实际情况选择)
$ cp support-files/mysql.server /etc/init.d/mysqld
$ chkconfig --add mysqld  (设置开机启动/服务)
$ ln -s /data/local/mysql/bin/mysql /usr/sbin
$ export PATH=$PATH:/data/local/mysql/bin
$ vim /etc/my.cnf
  [mysqld]
  datadir=/data/mysqldb
  socket=/data/mysqldb/mysql.sock
  user=mysql
  sql_mode="NO_ENGINE_SUBSTITUTION,NO_AUTO_CREATE_USER"
  character-set-server=utf8mb4
  # Disabling symbolic-links is recommended to prevent assorted security risks
  symbolic-links=0
  slow_query_log=on
  slow-query-log-file=/data/mysqldb/slowquery.log
  long_query_time=0.03
  log-queries-not-using-indexes
  [client]
  default-character-set = utf8mb4
  character-sets-dir=/data/local/mysql/share/charsets
  [mysqld_safe]
  log-error=/data/mysqldb/mysqld.log
  pid-file=/data/mysqldb/mysqld.pid
  (编辑my.cnf，参考官方文档设置 http://t.cn/Rip42em)
$ bin/mysqld_safe --user=mysql &
$ bin/mysqladmin -u root password "new passwd"   (设置密码,用其它方式参考官方 http://t.cn/R6NhwTv)
如果不行,可以使用重置密码的方式:
$ service mysqld stop
$ /usr/local/mysql/bin/mysqld_safe --skip-grant-tables
```

```mysql
$ mysql (去新建一个终端登录)
mysql> use mysql
mysql> update user set password=password("123456") where user="root";
mysql> flush privileges;
mysql> exit;
$ kill mysqld_safe ,启动mysqld , 登录mysql.
mysql> grant all privileges on *.* to 'root'@"%" identified by '123456' with grant option;
mysql> grant all privileges on *.* to 'root'@"%" identified by '123456' with grant option;
(grant做一个授权,%表示*.*(所有的库和表)允许被远程连接,使用这里指定的用户密码或者指定IP操作mysql,如果是单个数据库授权,dbname.* to username@"%"...)
flush privileges;
mysql> quit;
  tip: grant操作需要flush ,注意再操作完成后删除user表中匿名和空用户,或者给他们加上密码.
mysql> delete from user where user="";
# mysql -h localhost 和 mysql -h 127.0.0.1 的区别,通过localhost连接到mysql是使用UNIX socket,通过127.0.0.1连接到mysql是使用TCP/IP.
```

### 3.MySQL备份(有一个为zentao的数据库)

```bash
mysql5.6及以上,使用mysqldump在脚本里面登录数据库,防止报错信息.利用mysql/bin/mysql_config_editor保存 -uroot -p .
$ /data/local/mysql/bin/mysql_config_editor set --login-path=dbname --host=127.0.0.1 --user=root --password
  Enter password:
  上面的Enter password后面输入root登录MySQL的密码,即可生成 --login-path=dbname .
  下面就可以写入shell脚本.
$ vim mysql_backup.sh
  #!/bin/sh
  dump=/data/local/mysql/bin/mysqldump
  backup_dir=/data/backup/mysql/
  linux_user=root
  db_name=zentao
  days=15
  cd $backup_dir
  date=`date +%Y-%m-%d`
  backup_sql=$date.sql
  tar_sql="energy_bak_$date.tar.gz"
$ dump --login-path==dbname $db_name --default-character-set=utf8 --opt -Q -R --skip-lock-tables>$backip_sql
  tar -czf $tar_sql ./$backup_sql
  rm $backup_sql
  chown $linux_user:$linux_user $backup_dir/$tar_sql
  find $backup_dir -name "energy_bak*" -type f -mtime +$days -exec rm {} \;
    
从压缩备份文件中恢复(.tar.sql)
$ gzip < 2017-03-21.sql.tar.gz | mysql -uroot -p zentao
  或者:
$ zcat 2017-03-21.sql.tar.gz | mysql -uroot -p
  从备份文件恢复(.sql)
$ mysql -uroot -p zentao < 2017-03-21.sql
```
