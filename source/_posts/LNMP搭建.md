---
title: lnmp环境搭建配置
date: 2017-03-27 16:09:59
categories:
- PHP
tags:
- mysql
- php-fpm
- nginx
---
<!-- more -->
# 一、centos minimal安装完成

### 1、设置开机网卡自动连接

```bash
$ vim /etc/sysconfig/network-scripts/ifcfg-eth0
  ONBOOT=yes
```

### 2、关闭SELinux

```bash
$ /usr/sbin/sestatus -v
$ setenforce 0   (临时关闭)
$ vim /etc/selinux/config
  将 SELINUX=enforcing 改为 SELINUX=disabled   (永久关闭)
  sync reboot
```

### 3、防火墙

```bash
$ chkconfig iptables --list     (查看)
$ chkconfig iptables on/off     (永久)
$ service iptables start/stop   (临时)
```

##### 3.1、设定预设规则(详情参考:http://t.cn/RiONgR0)

```bash
$ iptables -P INPUT DROP
$ iptables -P OUTPUT ACCEPT
$ iptables -P FORWARD DROP
eg:开启ssh 22端口
$ iptables -A INPUT -p tcp -s 192.168.2.58 --dport 22 -j ACCEPT  (除了192.168.2.58其它IP禁止ssh)
$ iptables -A OUTPUT -p tcp -s 192.168.2.58 --sport 22 -j ACCEPT (如果OUTPUT设置成DROP则需添加该条)
$ service iptables save (手动每条去添加,需要手动保存,不然重启后失效)
或者直接在 iptables 编辑添加删除
$ vim /etc/sysconfig/iptables   (编辑iptables规则)
$ /etc/init.d/iptables restart
```

### 4、设置yum repo源

```bash
先备份系统源
$ mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
$ cd /etc/yum.repos.d/
$ wget -c http://mirrors.163.com/.help/CentOS6-Base-163.repo
$ vim CentOS-Media.repo
  关闭cdrom源
$ yum install epel   (安装第三方软件源)
```

### 5 、通过yum group装系统必备软件

```bash
$ yum grouplist
$ yum groupinstall "Development tools"
$ yum groupinstall "System Administration Tools"
(cent7用 yum group mark install)根据需求去选择软件包
$ yum install setuptool.x86_64
$ yum install ntsysv
$ yum install system-config-network-tui
$ yum install system-config-firewall-tui
$ yum install system-config-securitylevel-tui
```

# 二、基础目录结构

    1.mkdir
    /data/src        下载存放源码目录
    /data/log        站点日志目录
    /data/www        WEB站点目录
    /data/svn        SVN仓库目录
    /data/mysqldb    MYSQL数据库数据目录
    /data/backup     MYSQL数据备份目录
    /data/local/php  PHP目录,local下面都是程序编译安装目录
    /data/local/nginx
    /data/local/mysql

# 三、Nginx

### 1、安装编译工具及相关库

```bash
$ yum -y install make zlib zlib-devel gcc-c++ libtool  openssl openssl-devel
```

### 2、安装PCRE(作用让Nginx支持Rewrite),下载PCRE安装包，Google搜索下

```bash
$ cd /data/src
$ wget -c https://ftp.pcre.org/pub/pcre/pcre-8.40.tar.gz
$ tar -zxvf pcre-8.40.tar.gz
$ cd pcre-8.40
$ ./configure
$ Make&&make install
$ pcre-config --version  (查看pcre版本)
$ cp pcre-8.40/ /usr/local/src/
```

### 3、Nginx下载安装

```bash
$ cd /data/src
$ wget -c http://nginx.org/download/nginx-1.10.3.tar.gz
$ tar -zxvf nginx-1.10.3.tar.gz
$ cd nginx-1.10.3
$ ./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-pcre=/usr/local/src/pcre-8.40 (--with-prce=dir 指定pcre的源码目录)
$ make
$ make install
$ /usr/local/nginx/sbin/nginx -v (查看Nginx版本)
$ ln -s /usr/local/nginx/sbin/nginx /usr/bin
$ nginx            (-t查看启动  -s stop/reopen停止/重启)
  访问 http://localhost 查看Nginx是否正常安装启动
```

##### 3.1 tip:Nginx启动服务管理脚本

```sh
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemin
#
# chkconfig:   - 85 15 
# description:  Nginx is an HTTP(S) server, HTTP(S) reverse proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /usr/local/nginx/conf/nginx.conf
# pidfile:     /usr/local/nginx/logs/nginx.pid

# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0

nginx="/usr/local/nginx/sbin/nginx"
prog=$(basename $nginx)

NGINX_CONF_FILE="/usr/local/nginx/conf/nginx.conf"

lockfile=/var/lock/subsys/nginx

start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}

stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}

restart() {
    configtest || return $?
    stop
    start
}

reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}

force_reload() {
    restart
}

configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}

rh_status() {
    status $prog
}

rh_status_q() {
    rh_status >/dev/null 2>&1
}

case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
```

# 四、MySQL

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
$ mysql (去新建一个终端登录)
$ mysql>use mysql
$ mysql>update user set password=password("123456") where user="root";
$ mysql>flush privileges;
$ exit;
  kill mysqld_safe ,启动mysqld , 登录mysql.
$ grant all privileges on *.* to 'root'@"%" identified by '123456' with grant option;
$ grant all privileges on *.* to 'root'@"%" identified by '123456' with grant option;
(grant做一个授权,%表示*.*(所有的库和表)允许被远程连接,使用这里指定的用户密码或者指定IP操作mysql,如果是单个数据库授权,dbname.* to username@"%"...)
$ flush privileges;
$ quit;
  tip: grant操作需要flush ,注意再操作完成后删除user表中匿名和空用户,或者给他们加上密码.
  delete from user where user="";
  mysql -h localhost 和 mysql -h 127.0.0.1 的区别,通过localhost连接到mysql是使用UNIX socket,通过127.0.0.1连接到mysql是使用TCP/IP.
```

### 3.MySQL备份(假设zentao作为一个数据库名)

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
$dump --login-path==dbname $db_name --default-character-set=utf8 --opt -Q -R --skip-lock-tables>$backip_sql
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

# 五、PHP

### 1.编译安装

```bash
$ yum -y install gcc automake autoconf libtool make glibc
$ cd /data/src
$ wget -c http://cn2.php.net/distributions/php-5.5.38.tar.gz
$ tar -zxvf php-5.5.38.tar.gz
$ cd php-5.5.38
$ ./configure --prefix=/data/local/php --enable-fpm
(如果这里没有加--enable-fpm, php5.3及以上的内置了php-fpm 可以重新编译加上参数。或者yum install php-fpm,
以下供参数参考 configure过程中有提示参数软件未安装的就安装后再configure,或者去除参数:

./configure --prefix=/data/local/php --with-config-file-path=/data/local/php/etc --with-mysql=/data/local/mysql --with-mysqli=/data/local/mysql/bin/mysql_config --with-pdo-mysql=/data/local/mysql --with-pcre-dir=/data/local/pcre/bin/pcre-config --enable-bcmath --enable-exif --enable-fpm --enable-ftp --enable-fastCGI --enable-force-CGI-redirect --enable-gd-native-ttf --enable-inline-optimization --enable-mbstring --enable-opcache --enable-pcntl --enable-pdo_mysql --enable-soap --enable-shmop --enable-sockets --enable-sysvsem --enable-sysvmsg --enable-sysvshm --enable-zip --enable-zend-multibyte --with-bz2 --with-curl --with-curlwrappers --with-freetype-dir --with-gd --with-gdbm --with-gmp --with-gettext --with-jpeg-dir --with-libxml-dir --with-libdir --with-mhash --with-mcrypt --with-openssl --with-png-dir --with-pear --with-xsl --with-zlib-dir 

附加参数参考 http://t.cn/Ri0WWcq)如果有httpd，可以加上 --with-apxs2=/data/local/httpd/bin/apxs这段参数，避免以后要使用httpd再重新编译php。
$ make test
$ make install
$ cp php.ini-development /data/local/php/etc/php.ini   (去目录将cp过去的文件重命 php.ini)
$ vim php.ini
  date.timezone = PRC   (或者Asia/Shanghai)
  magic_quotes_gpc = On   (防止SQL注入)
$ cp /data/local/php/etc/php-fpm.default.conf php-fpm.conf
$ /data/local/php/sbin/php-fpm -R  (启动fpm)
  kill -INT `cat $PATH:php-fpm.pid`   关闭(cat后面输入php-fpm.pid路径)
  kill -USR2 `cat $PATH:php-fpm.pid`  重启
$ ps -ef | grep php-fpm 或者 lsof -i :9000
设置php-fpm开机启动：vim /etc/local ,最后一行加入 /data/local/php/sbin/php-fpm 即可
```

### 2.php-fpm服务启动管理

###### tip: php-fpm service管理使用脚本,根据实情修改对应目录

```shell
#!/bin/bash
#
# Startup script for the PHP-FPM server.
#
# chkconfig: 345 85 15
# description: PHP is an HTML-embedded scripting language
# processname: php-fpm
# config: /usr/local/php/etc/php.ini
 
# Source function library.
. /etc/rc.d/init.d/functions
 
PHP_PATH=/data/local
DESC="php-fpm daemon"
NAME=php-fpm
DAEMON=$PHP_PATH/php/sbin/$NAME
CONFIGFILE=$PHP_PATH/php/etc/php-fpm.conf
PIDFILE=$PHP_PATH/php/var/run/$NAME.pid
SCRIPTNAME=/etc/init.d/$NAME
 
# Gracefully exit if the package has been removed.
test -x $DAEMON || exit 0
 
rh_start() {
  $DAEMON -y $CONFIGFILE || echo -n " already running"
}
 
rh_stop() {
  kill -QUIT `cat $PIDFILE` || echo -n " not running"
}
 
rh_reload() {
  kill -HUP `cat $PIDFILE` || echo -n " can't reload"
}
 
case "$1" in
  start)
        echo -n "Starting $DESC: $NAME"
        rh_start
        echo "."
        ;;
  stop)
        echo -n "Stopping $DESC: $NAME"
        rh_stop
        echo "."
        ;;
  reload)
        echo -n "Reloading $DESC configuration..."
        rh_reload
        echo "reloaded."
  ;;
  restart)
        echo -n "Restarting $DESC: $NAME"
        rh_stop
        sleep 1
        rh_start
        echo "."
        ;;
  *)
         echo "Usage: $SCRIPTNAME {start|stop|status|restart|reload}" >&2
         exit 3
        ;;
esac
exit 0
```

# 六、Nginx配置php-fpm

### 1、修改nginx.conf,根据里面默认的注释例子修改

```bash
$ vim /usr/local/nginx/conf/nginx.conf
========================================================================================================
    #HTTP server
    server{
        listen 80;
        return 444;
        #server_name localhost;
        charset utf-8;
        
        location / {
          root html;
          index index.html index.htm;
        }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }

    location ~ \.php$ {
    root           html;
    fastcgi_pass   127.0.0.1:9000;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    include        fastcgi_params;
    }
    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
        {
                expires 30d;
        }
        location ~ .*\.(js|css)?$
        {
                expires 1h;
        }
    }
    inculude vhost/*.conf;
========================================================================================================
```

##### 2.在conf/vhost/目录下编写配置每一个站点的conf文件,可以拷贝Nginx里面的例子进行修改

# 七、设置禁止参与 yum 更新的核心软件 

```bash
$vim /etc/yum.conf
 exclude=php* apache* kernel* mysql* nginx* (根据实际情况进行指定不参与yum update的程序)
```
# 八、更改站点配置

```
1、在每个文件的web下查看,使用的配置文件类型是test,server,local.去更改对应的目录文件.主要有每个站点的下api/ backend/ common/ frontend/ h5/ weixin/ , main.php和param.php,在member.8dage.net下的是application/config下面的allow_ip.php,database.php,config.php
```



