---
title: PHP环境配置
date: 2017-03-27 16:15:59
categories:
- PHP
tags:
- php-fpm
- php
---
<!-- more -->
## PHP

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

## 3.Nginx配置php-fpm

##### 3.1修改nginx.conf,根据里面默认的注释例子修改

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

##### 3.2.在conf/vhost/目录下编写配置每一个站点的conf文件,可以拷贝Nginx里面的例子进行修改.

