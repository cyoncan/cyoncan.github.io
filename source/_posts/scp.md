---
title: SCP
date: 2017-03-23 19:37:21
categories:
- CentOS
tags:
- linux
- scp
---
<!-- more -->
### SCP复制(服务器之间传输文件)

##### 1.从服务器上拉取目录

   ```bash
scp -p 22 -r user@192.168.2.1:/data/www /data/www/
   ```

##### 2.从本地上传目录(-p为端口参数,port端口.默认端口可以省略,传输为目录需要 -r 进行目录递归)

```bash
scp -r /data/www user@192.168.2.1:/data/www/
```
##### 3.使用ssh-keygen生成密钥和私钥文件,建立两台机器互相通信,可避免每次输入验证密码.

```bash
ssh-keygen -t rsa  (生成在~/.ssh/目录下id_rsa.pub和id_rsa)
```
##### 4.将id_rsa.pub上传到目标服务器~/.ssh目录下,命名authorized_keys.

```bash
scp -r /root/.ssh/id_rsa.pub user@serverip:/root/.ssh/authorized_keys
```
##### 5.如果目标服务器上,已经存在了authorized_keys,就将id_rsa.pub中的内容追加到目标服务器的authorized_keys文件中.

```bash
cat /root/.ssh/id_rsa.pub | ssh user@serverip 'cat >> /root/.ssh/authorized_keys'
```

