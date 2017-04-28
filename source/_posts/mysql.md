---
title: MySQL常用语句
date: 2017-04-20 00:16:21
categories:
- MySQL
tags:
- mysql语句
---
<!-- more -->
##### 1.MySQL创建用户

```bash
create user 'username'@'%' identified by 'passwd';   % 换成localhost或者127.0.0.1,就只能本地登录了.
```

##### 2.给用户授权

```bash
grant all privileges on databasename.tablename to 'username'@'hostname' identified by 'passwd'with grant option;   all代表所有权限,withgrant option代表该用户可以给其他用户也进行授权操作.
flush privileges;   授权完成,需要刷新生效.
quit;
```

##### 3.查看mysql有多少个账户
```bash
select host,user,password from mysql.user;
```

##### 4.查看MySQL字符集,排序规则
```bash
show variables like 'collation%';
show variables like 'char%';
```

