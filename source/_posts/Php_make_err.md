---
title: php编译报错
date: 2017-07-1  09:14:15
categories:
- PHP
tags:
- php
---

<!-- more -->

PHP重新编译安装报错：

configure: error: Cannot find libmysqlclient_r under /data/local/mysql
Note that the MySQL client library is not bundled anymore!

php编译参数：

```shell
./configure --prefix=/data/local/php --with-config-file-path=/data/local/php/etc --with-apxs2=/data/local/httpd/bin/apxs --with-mysql=/data/local/mysql --with-mysqli=/data/local/mysql/bin/mysql_config --with-pdo-mysql=/data/local/mysql --with-pcre-dir=/data/local/pcre/bin/pcre-config --enable-bcmath --enable-exif --enable-fpm --enable-ftp --enable-gd-native-ttf --enable-inline-optimization --enable-mbstring --enable-opcache --enable-pcntl --enable-pdo_mysql --enable-soap --enable-shmop --enable-sockets --enable-sysvsem --enable-sysvmsg --enable-sysvshm --enable-zip --with-bz2 --with-curl --with-freetype-dir --with-gd --with-gdbm --with-gmp --with-gettext --with-jpeg-dir --with-libxml-dir --with-libdir --with-mhash --with-mcrypt --with-openssl --with-pear --with-png-dir --with-xsl --with-zlib-dir 
```

*--with-mysql=mysqlnd是不报错的参数*

原因：

我之前搭建的lnmp环境，没有使用httpd，在后来编译APACHE的时候，使用--with-mpm模块，所以就必须在编译MYSQL的时候加上 --enable-thread-safe-client.

因为MySQL不能随意停止进行重新编译，怕造成问题故障。可定是不能重新编译MySQL的。查了下这个问题是PHP5.2的一个改进，在PHP5.2.0之前的版本都不需要MYSQL启用安全线程。就找了下其他方法，看到大多数的都是说需要 mysql-devel，这个我之前mysql安装的时候都是有的，包括报错提示的libmysqlclient_r.so在我的目录/dala/local/mysql/lib下都是存在的。但是还是一直报哪个错误，在网上查了好多都是说做个软连或者复制libmysqlclient.so.18.1.0到libmysqlclient_r.so就可以了。我试了很多都不行，包括ln -s lib lib64 。后来直接改了--with-mysql=mysqlnd就OK了。后来想想或许这种也可以--with-mysql -lib-dir=/data/local/mysql/lib

在这里记录下httpd的编译参数：

```shell
./configure --prefix=/data/local/httpd --with-apr=/data/local/apr/bin/apr-1-config --with-apr-util=/data/local/apr-util/bin/apu-1-config --with-pcre=/data/local/pcre/bin/pcre-config --with-mpm=worker --enable-access-compat --enable-alias --enable-authn-core --enable-authn-file --enable-auth-basic --enable-authz-core --enable-authz-host --enable-authz-user --enable-authz-groupfile --enable-autoindex --enable-cgi --enable-cache --enable-cache-disk --enable-dir --enable-deflate --enable-debugger-mode --enable-env --enable-expires --enable-file-cache --enable-filter --enable-http --enable-headers --enable-log-config --enable-load-all-modules --enable-mime --enable-modules=most --enable-reqtimeout --enable-rewrite --enable-so --enable-setenvif --enable-status --enable-speling --enable-static-support --enable-unixd --enable-version --enable-vhost-alias --enable-mods-shared=all 
```

查看nginx编译参数：/data/local/nginx/sbin/nginx -V
查看apache编译参数：cat /data/local/httpd/build/config.nice
查看mysql编译参数：cat /data/local/mysql/bin/mysqlbug | grep CONFIGURE_LINE
查看php编译参数：/data/local/php/bin/php -i | grep configure   或者 php -r "phpinfo();"|grep configure