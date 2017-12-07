---
title: Zentao apache改用nginx
date: 2017-06-8  20:13:43
categories:
- Nginx
tags:
- zentao
---

<!-- more -->

zentao是一套还不错的项目管理系统，有开源的版本。最开始部署的一台服务器是用的apache，后来种种原因，需要迁移到另外一台机器上，新机器的php是用nginx的，一直对nginx比较好感，所以也不打算用httpd配置zentao。由于对zentao的安装和配置不熟悉，整了老久。后来google了下，查到了一些问题所在。就记录下。好记性不如烂笔头啊。

1.禅道的配置信息（注意修改requestType使用的方式）

```shell
root@intel ~ $ cat /data/www/zentao/config/my.php
<?php
$config->installed       = true;
$config->debug           = false;
$config->requestType     = 'GET';   //nginx使用的方式，nginx不支持使用pathinfo方式路由
$config->requestType     = 'PATH_INFO';   //apache使用的方式
$config->db->host        = '192.168.2.100';
$config->db->port        = '3306';
$config->db->name        = 'zentao';
$config->db->user        = 'zentao';
$config->db->password    = 'zentao';
$config->db->prefix      = 'zt_';
$config->webRoot         = getWebRoot();
$config->default->lang   = 'zh-cn';
$config->mysqldump       = '';
```

2.nginx配置文件

```shell
root@intel ~ $ cat /data/local/nginx/conf/vhost/zentao.conf
server
        {
         listen 80;
         server_name zentao.test.com;
         index index.html index.php;
         root  /data/www/zentao/www;
     access_log  /data/logs/$SERVER_NAME.access.log main;
	 error_log  /data/logs/zentao.test.org.error.log error;

	 location / {
	 	root /data/www/zentao/www;
		client_max_body_size 50m;
		index index.php index.html;
		if (!-e $request_filename) {
			rewrite ^/(.*)$ /index.php/$1 last;
		break;
		}
	 }

         location ~ \.php$ {
		 root /data/www/zentao/www;
	         fastcgi_index   index.php;
	         fastcgi_pass    127.0.0.1:9000;
	         include         fastcgi_params;
	         fastcgi_param   SCRIPT_FILENAME    $document_root$fastcgi_script_name;
		 	 fastcgi_param   SCRIPT_NAME        $fastcgi_script_name;
	 	 }
	 }
```

