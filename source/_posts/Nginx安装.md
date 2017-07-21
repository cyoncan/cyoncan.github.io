---
title: Nginx安装配置
date: 2017-03-27 16:09:59
categories:
- Nginx
tags:
- nginx
---
<!-- more -->
## Nginx

#### 1、安装编译工具及相关库

```shell
$ yum -y install make zlib zlib-devel gcc-c++ libtool  openssl openssl-devel
```

#### 2、安装PCRE(作用让Nginx支持Rewrite),下载PCRE安装包，Google搜索下

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

#### 3、Nginx下载安装

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

## Nginx配置php-fpm

#### 1、修改nginx.conf,根据里面默认的注释例子修改

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

#### 2.在conf/vhost/目录下编写配置每一个站点的conf文件,可以拷贝Nginx里面的例子进行修改