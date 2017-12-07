---
title: iRedMail_nginx
date: 2017-05-04  19:13:56
categories:
- Mail
tags:
- email
- nginx
---

<!-- more -->

### Nginx配置

```shell
upstream php_workers {
    server unix:/var/run/php-fpm/php-fpm.socket;
}

# HTTP
server {
    # Listen on ipv4
    listen 80;
    # Listen on ipv6.
    # Note: this setting listens on both ipv4 and ipv6 with Nginx release
    #       shipped in some Linux/BSD distributions.
    #listen [::]:80;
    server_name cyoncan.vicp.cc;
    root /var/www/roundcubemail;
    index index.php index.html index.htm;

    location / {
        root /var/www/roundcubemail;
    }

    # Normal PHP scripts
    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass php_workers;
        fastcgi_param SCRIPT_FILENAME /var/www/roundcubemail$fastcgi_script_name;
    }

    # Redirect webmail/SOGo/iredadmin to HTTPS
   # location ~ ^/ { rewrite ^ http://$host$request_uri?; }
    location ~* ^/sogo { rewrite ^ https://$host/SOGo; }
    location ~ ^/iredadmin { rewrite ^ https://$host$request_uri?; }

    # Deny all attempts to access hidden files such as .htaccess.
    location ~ /\. { deny all; }

    # Handling noisy favicon.ico messages
    location = ^/favicon.ico { access_log off; log_not_found off; }
}

# HTTPS
server {
    listen 443;
    server_name cyoncan.vicp.cc;
    
    ssl on;
    ssl_certificate /etc/pki/tls/certs/iRedMail.crt;
    ssl_certificate_key /etc/pki/tls/private/iRedMail.key;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

    # Fix 'The Logjam Attack'.
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA;
    ssl_prefer_server_ciphers on;
    ssl_dhparam /etc/pki/tls/dhparams.pem;

    index index.php index.html index.htm;

    location / {
        root /var/www/html/;
    }

    # Deny all attempts to access hidden files such as .htaccess.
    location ~ /\. { deny all; }

    # Handling noisy favicon.ico messages
    location = ^/favicon.ico { access_log off; log_not_found off; }

    # Roundcube webmail
    location ~ ^/mail(.*)\.php$ {
        include fastcgi_params;
        fastcgi_pass php_workers;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME /var/www/roundcubemail$1.php;
    }

    location ~ ^/mail(.*) {
        alias /var/www/roundcubemail$1;
        index index.php;
    }

    location ~ ^/(bin|SQL|README|INSTALL|LICENSE|CHANGELOG|UPGRADING)$ { deny all; }

    # Normal PHP scripts
    location ~ \mail\.php$ {
        include fastcgi_params;
        fastcgi_pass php_workers;
        fastcgi_param SCRIPT_FILENAME /var/www/html$fastcgi_script_name;
    }

    # iRedAdmin: static files under /iredadmin/static
    location ~ ^/iredadmin/static/(.*)\.(png|jpg|gif|css|js) {
        alias /var/www/iredadmin/static/$1.$2;
    }

    # iRedAdmin: Python scripts
    location ~ ^/iredadmin(.*) {
        rewrite ^/iredadmin(/.*)$ $1 break;
        include uwsgi_params;
        uwsgi_pass unix:/var/run/uwsgi_iredadmin.socket;
        uwsgi_param UWSGI_CHDIR /var/www/iredadmin;
        uwsgi_param UWSGI_SCRIPT iredadmin;
        uwsgi_param SCRIPT_NAME /iredadmin;
    }
    # iRedAdmin: redirect /iredadmin to /iredadmin/
    location = /iredadmin {
        rewrite ^ /iredadmin/;
    }

    # SOGo
    location ~ ^/sogo { rewrite ^ https://$host/SOGo; }
    location ~ ^/SOGO { rewrite ^ https://$host/SOGo; }

    # For IOS 7
    location = /principals/ {
        rewrite ^ https://$server_name/SOGo/dav;
        allow all;
    }

    location ^~ /SOGo {
        proxy_pass http://127.0.0.1:20000;
        #proxy_redirect http://127.0.0.1:20000/SOGo/ /SOGo;
        # forward user's IP address
        #proxy_set_header X-Real-IP $remote_addr;
        #proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        #proxy_set_header Host $host;
        proxy_set_header x-webobjects-server-protocol HTTP/1.0;
        #proxy_set_header x-webobjects-remote-host 127.0.0.1;
        #proxy_set_header x-webobjects-server-name $server_name;
        #proxy_set_header x-webobjects-server-url $scheme://$host;
    }

    location ^~ /Microsoft-Server-ActiveSync {
        proxy_pass http://127.0.0.1:20000/SOGo/Microsoft-Server-ActiveSync;
        proxy_redirect http://127.0.0.1:20000/Microsoft-Server-ActiveSync /;
    }

    location ^~ /SOGo/Microsoft-Server-ActiveSync {
        proxy_pass http://127.0.0.1:20000/SOGo/Microsoft-Server-ActiveSync;
        proxy_redirect http://127.0.0.1:20000/SOGo/Microsoft-Server-ActiveSync /;
    }

    location /SOGo.woa/WebServerResources/ {
        alias /usr/lib64/GNUstep/SOGo/WebServerResources/;
    }
    location /SOGo/WebServerResources/ {
        alias /usr/lib64/GNUstep/SOGo/WebServerResources/;
    }
    location ^/SOGo/so/ControlPanel/Products/([^/]*)/Resources/(.*)$ {
        alias /usr/lib64/GNUstep/SOGo/$1.SOGo/Resources/$2;
    }
}
```

