---
title: MySQL数据库备份恢复
date: 2017-03-23 22:20:21
categories:
- MySQL
tags:
- MySQL
- mysql备份
- 脚本
---
<!-- more -->
### MySQL单库备份恢复/导出导入

```bash
$ msyql/bin/mysqldump -uroot -p dbname > bak.sql
  mysqldump 默认不会导出事件表，执行此命令会出现警告 -- Warning: Skipping the data of table mysql.event. Specify the --events option explicitly
```

#### 导出MySQL事件

```bash
$ mysql/bin/mysqldump -uroot -p --events --ignore-table=mysql.event  dbname > bak.sql 
```

#### 导入MySQL备份

```bash
$ mysql -uroot -p mysql < bak.sql
或者
$ mysql -uroot -p
Enter password:
$mysql>use dbname;
$mysql>source /data/backup/bak.sql;
```

### MySQL备份所有的库脚本

```sh
#!/bin/bash
#-----------------------------------------------#
#This is a  free GNU GPL version 3.0 or abover
#Copyright (C) 2008 06 05
#mysql_backup Dedicated copyright by My
#-----------------------------------------------#
echo -e [`date +"%Y-%m-%d %H:%M:%S"`] start
#system time
time=`date +"%y-%m-%d"`
#host IP
host="127.0.0.1"
#database backup user
user="root"
#database password
passwd="yourpasswd"
#Create a backup directory
mkdir -p /backup/db/"$time"
#list database name
all_database=`/usr/bin/mysql -u$user -p$passwd -Bse 'show databases'`
#in the table from the database backup
for i in $all_database
do
/usr/bin/mysqldump -u$user -p$passwd $i > /backup/db/"$time"/"$i"_"$time".sql
done
echo -e [`date +"%Y-%m-%d %H:%M:%S"`]  end
exit 0


运行 crontab -e，写入以下内容:
30 5 * * * root sh /root/autobackup.sh >/dev/null 2>&1
```

**Tip:** 如果提示 *mysql: [Warning] Using a password on the command line interface can be insecure.*
请参考另一篇写nginx + php-fpm + mysql 那篇的mysql部分,写用 mysql_config_editor 去解决 这个提示明文不安全问题.
