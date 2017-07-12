---
title: Mongodb基础使用
date: 2017-05-02 20:56:12
categories:
- MongoDB
tags:
- mongodb
---

<!-- more -->

### 1.安装使用

**如果是安装2.4/2.6版本的，可以用epel源直接yum install**

```shell
$ yum install epel-release.noarch
$ yum makecache
$ yum install mongodb-serevr
```

**安装最新版本,使用官方的仓库**

```shell
$ touch	/etc/yum.repos.d/mongodb-org-3.4.repo
$ vim /etc/yum.repos.d/mongodb-org-3.4.repo
[mongodb-org-3.4]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/amazon/2013.03/mongodb-org/3.4/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.4.asc
$ yum makecache
$ yum install mongodb-org
```

**创建数据库目录,启动mongodb**

```shell
$ mkdir -p /data/db
$ mongod --dbpath=/data/db --rest
```

**通过可以访问localhost:28017可以访问web用户界面.如果是云服务器,需要打开相应的端口或者是安全组里面的规则**[官方文档参考](http://t.cn/Ranvfiv)

### 2.进入数据库

```shell
$ mongo
MongoDB shell version v3.4.4
connecting to: mongodb://127.0.0.1:27017
MongoDB server version: 3.4.4
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
		http://docs.mongodb.org/
Questions? Try the support group
		http://groups.google.com/group/mongodb-user 
> show dbs
admin  0.000GB
local  0.000GB
> use local
switched to db local
> db
local
>
```

