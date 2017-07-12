---
title: zabbix安装配置
date: 2017-04-20 00:16:21
categories:
- 监控
tags:
- zabbix
- openssl
---
<!-- more -->
# zabbix搭建

> 环境: centos6,lnmp,zabbix3.0.8

### 一.web环境安装参考另一篇写lnmp搭建的文章

### 二.zabbix安装搭建

###### tip:在官方的文档上已经有很详细的说明,下面是根据自己的实际情况参照文档安装过程.

#####         1.zabbix下载安装选LTS,我下的3.0最新的源码包

```bash
$ wget -c https://nchc.dl.sourceforge.net/project/zabbix/ZABBIX%20Latest%20Stable/3.0.8/zabbix-3.0.8.tar.gz
$ tar zxvf zabbix-3.0.8.tar.gz
$ groupadd zabbix
$ useradd -g zabbix zabbix
$ cd zabbix-3.0.8/ 
$ ./configure --prefix=/data/local/zabbix --enable-server --enable-agent --with-mysql --enable-ipv6 --with-net-snmp --with-libcurl --with-libxml2 --with-openssl
$ yum install net-snmp-devel.x86_64 -y (根据实际环境去安装提示缺少的东西,MySQL大部分是因为没有软连接到usr/bin)
$ ln -s /data/local/zabbix/sbin/* /usr/sbin/
```

##### 2.创建zabbix数据库,导入数据库

```bash
$ mysql -uroot -p
mysql> create database zabbix character set utf8 collate utf8_bin;
mysql> grant all privileges on zabbix.* to zabbix@'localhost' identified by 'passwd';
mysql> grant all privileges on zabbix.* to zabbix@'127.0.0.1' identified by 'passwd';
或者直接用
mysql> grant all privileges on zabbix.* to zabbix@'%' identified by 'passwd';
mysql> flush privileges;
mysql> quit;
$ cd /data/src/zabbix-3.0.8/database/mysql
$ mysql -uzabbix -p zabbix < schema.sql   导入zabbix数据库脚本
# stop here if you are creating database for Zabbix proxy
$ mysql -uzabbix -p zabbix < images.sql
$ mysql -uzabbix -p zabbix < data.sql
导入数据库另一种方法:
$ mysql -uzabbix -p
mysql> use zabbix;
mysql> source /data/src/zabbix-3.0.8/database/mysql/schema.sql;
......
```

##### 3.修改zabbix_server配置文件

```bash
DBName=zabbix       #数据库名称
DBUser=zabbix       #数据库用户名
DBPassword=123456   #数据库密码
ListenIP=127.0.0.1  #数据库ip地址
Timeout=4
AlertScriptsPath=/data/local/zabbix/share/zabbix/alertscripts
ExternalScripts=/data/local/zabbix/share/zabbix/externalscripts
LogSlowQueries=3000
```

##### 4.创建web目录

```bash
mkdir -p /data/web/zabbix.monitor.cn
cp -a /data/src/zabbix-3.0.8/frontend/php/* /data/web/zabbix.monitor.cn/
chown -R www:www zabbix.monitor.cn/
```

##### 5.配置nginx

```bash
vim /data/local/nginx/conf/nginx.conf
在server模块外http模块内加入一行
include vhost/*.conf
:wq
cd /data/local/nginx/conf/vhost
vim zabbix.conf
server
      {
      listen 80;
      server_name 127.0.0.1;
      index index.html index.php;
      root  /data/web/zabbix.monitor.cn;
      access_log  /data/logs/$SERVER_NAME.access.log main;
	  error_log  /data/logs/$SERVER_NAME.log error;
      
      location / {
      fastcgi_index   index.php;
      fastcgi_pass    127.0.0.1:9000;
      include         fastcgi_params;
      fastcgi_param   SCRIPT_FILENAME    $document_root$fastcgi_script_name;
      fastcgi_param   SCRIPT_NAME        $fastcgi_script_name;
	  }

	  if (!-e $request_filename){
	  		 rewrite ^/(.*)$ /index.php/$1 last;
	  }
                
      location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|JPG|xml|json)$
      {
      expires      30d;
      }
      
      location ~ .*\.(js|css)?$
      {
      expires      12h;
      }
      #   access_log m.log  access;
      }
```

##### 6.修改php.ini文件(这个地方债官网上有配置要求说明)

```bash
post_max_size = 32M
max_execution_time = 300
max_input_time = 300
date.timezone = Asia/Shanghai
以下是我自己的进入zabbi安装检测各个配置参数和模块是否满足zabbix安装要求.如果忘记更改php.ini或者不知道改哪些的,可以进入到安装界面http://zabbix.monitor.cn/setup.php 根据检测结果,再去修改,每次修改可能需要重启nginx和php-fpm
PHP version	5.6.30(我用的PHP版本)	5.4.0(zabbix3.0.8要求PHP最低的版本,下面的参数类似要求)	OK
PHP option "memory_limit"	128M	128M	OK
PHP option "post_max_size"	32M	16M	OK
PHP option "upload_max_filesize"	2M	2M	OK
PHP option "max_execution_time"	300	300	OK
PHP option "max_input_time"	300	300	OK
PHP option "date.timezone"	Asia/Shanghai		OK
PHP databases support	MySQL/SQLite3 OK
PHP bcmath	on		OK
PHP mbstring	on		OK
PHP option "mbstring.func_overload"	off	off	OK
PHP option "always_populate_raw_post_data"	off	off	OK
PHP sockets	on		OK
PHP gd	2.1.0	2.0	OK
PHP gd PNG support	on		OK
PHP gd JPEG support	on		OK
PHP gd FreeType support	on		OK
PHP libxml	2.7.6	2.6.15	OK
PHP xmlwriter	on		OK
PHP xmlreader	on		OK
PHP ctype	on		OK
PHP session	on		OK
PHP option "session.auto_start"	off	off	OK
PHP gettext	on		OK
PHP option "arg_separator.output"	&	&	OK
```

##### 7.启动,进入安装界面

```bash
$ service nginx start
$ service php-fpm start
$ zabbix_server   
浏览器输入 http://zabbix.monitor.cn/setup.php 域名是之前nginx里面配置好的,或者不用域名用IP和别的端口,域名需要做host映射,或者内网路由器里面做虚拟映射.出现界面下一步,填上MySQL zabbix用户密码一致,下一步,看到绿色congratulation 就安装完成了.
默认用户名和密码:admin,zabbix
设置zabbix服务开机启动
$ cp /data/src/zabbix/misc/init.d/fedora/core/* /etc/init.d/
$ vim /etc/init.d/zabbix_server
修改BASEDIR你zabbix安装路径
chkconfig --add zabbix_server
chkconfig --add zabbix_agentd
chkconfig zabbix_server on
chkconfig zabbix_agentd on
```

##### 8.客户端安装

客户端根据实际情况去使用几种方案,一般机器少的,都是一台server端,几台agent端.方案可以参考:http://t.cn/RXPa0zV

```bash
./configure --prefix=/usr/local/zabbix --enable-agent --with-openssl
make install
其它的设置和前面的安装配置一样,zabbix_agentd.conf 
Server=Serverip
ServerActive=Serverip
Hostname=自定义zabbix客户端hostname,不要和server端配置的hostname一样.
tip:将10050,10051端口添加到防火墙,server端也是.agent端需要启动zabbix_agentd服务
```

##### 9.客户端和服务端使用加密传输

###### i. 使用psk共享密钥加密

```bash
Generating PSK在客户端操作
$ cd /usr/local/zabbix/
$ openssl rand -hex 32 -out zabbix_agentd.psk
$ chown zabbix:zabbix zabbix_agentd.psk
$ vim /etc/zabbix_agentd.conf
  TLSConnect=psk
  TLSAccept=psk
  TLSPSKFile=/home/zabbix/zabbix_agentd.psk
  TLSPSKIdentity=PSK 001
 service zabbix_agentd restart
 回到server机器,在host机器加密那里选择psk,填入信息.
```

###### ii. 使用证书加密

```bash
$ mkdir -p /data/local/zabbix/zabbix_crt
$ cd zabbix_crt/
#生成ca私钥
$ openssl genrsa -aes256 -out ca.key 2048   需要输入给私钥加密的密码
#使用ca私钥建立ca证书
$ openssl req -new -x509 -nodes -days 1000 -key ca.key -subj /CN=ServerIPorDomainName\ CA/OU=Development\ group/O=Zabbix\ SIA/DC=zabbix/DC=com > ca.crt
#生成服务器csr证书请求文件
$ openssl req -newkey rsa:2048 -days 1000 -nodes -keyout server.key -subj /CN=ServerIPorDomainName/OU=Development\ group/O=Zabbix\ SIA/DC=zabbix/DC=com > server.csr
#使用ca证书与私钥签发服务器证书
$ openssl x509 -req -in server.csr -days 1000 -CA ca.crt -CAkey ca.key -set_serial 01 > server.crt
#生成客户端csr证书请求文件
$ openssl req -newkey rsa:2048 -days 1000 -nodes -keyout client.key -subj /CN=client/OU=Development\ group/O=Zabbix\ SIA/DC=zabbix/DC=com > client.csr
#使用ca证书与私钥签发客户端证书
$ openssl x509 -req -in client.csr -days 1000 -CA ca.crt -CAkey ca.key -set_serial 01 > client.crt

服务端文件位置可以不改变,用scp命令将刚刚生成的客户端需要文件上传到客户端自定义的文件夹 /usr/local/zabbix/zabbix_crt/

服务端: ca.crt , server.crt , server.crt
vim /data/local/zabbix/etc/zabbix_server.conf
TLSCAFile=/data/local/zabbix/zabbix_crt/ca.crt
TLSCertFile=/usr/local/zabbix/zabbix_crt/server.crt
TLSKeyFile=/usr/local/zabbix/zabbix_crt/server.key

客户端: ca.crt , client.crt , client.crt
TLSConnect=cert
TLSAccept=cert
TLSCAFile=/usr/local/zabbix/zabbix_crt/ca.crt
TLSCertFile=/usr/local/zabbix/zabbix_crt/client.crt
TLSKeyFile=/usr/local/zabbix/zabbix_crt/client.key

进入webui configuration/hosts/打开相应的主机/双向都可以选certificate
重启zabbix_server和zabbix_agent,再进去看cert是否变蓝色,有延迟几十秒.
我记录的文档可能有些不全,希望大家遇到问题多分析日志,去网上搜索遇到的错误信息.
```
