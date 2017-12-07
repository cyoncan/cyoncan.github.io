---
title: MySQL基础操作
date: 2017-04-19 19:03:11
categories:
- MySQL
tags:
- mysql
---
<!-- more -->
MySQL创建用户

```mysql
create user 'username'@'%' identified by 'passwd';   % 换成localhost或者127.0.0.1,就只能本地登录了.
```

给用户授权

```mysql
grant all privileges on databasename.tablename to 'username'@'hostname' identified by 'passwd'with grant option;   all代表所有权限,withgrant option代表该用户可以给其他用户也进行授权操作.
flush privileges;   授权完成,需要刷新生效.
quit;
```

查看mysql有多少个账户
```mysql
select host,user,password from mysql.user;
```

查看MySQL字符集,排序规则
```mysql
show variables like 'collation%';
show variables like 'char%';
```

查看MySQL版本

```mysql
① mysql> status;
② mysql> select version();
```

查看MySQL使用的存储引擎类型

```mysql
mysql> show ENGINES;
```

查看MySQL表锁状态

```mysql
mysql> show status like 'tables%';
mysql> show status like '%lock%';
```

查看MySQL当前使用的数据库

```mysql
mysql> status;
mysql> select database();
mysql> show tables;
```

