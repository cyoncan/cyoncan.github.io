---
title: Jenkins
date: 2017-07-10  19:53:41
categories:
- CI&CD
tags:
- jenkins
---

<!-- more -->

Jenkins是基于java语言写的一款，持续集成的web界面管理系统。第一次见过在Linux上这么简单就运行起来的环境(需要有jdk)。

直接在官网上下载war包,运行 java -jar jenkins.war

```shell
$ wget -P /opt/jenkins https://mirrors.tuna.tsinghua.edu.cn/jenkins/war-stable/2.60.1/jenkins.war
$ java -jar jenkins.war
http://ip:8080
根据提示就可以初始化完成使用了，可以通过supervisor管理它运行。
```

jdk安装yum install java

```shell
$ wget http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.rpm
$ rpm -ivh jdk-8u131-linux-x64.rpm
$ java -version
java version "1.8.0_131"
Java(TM) SE Runtime Environment (build 1.8.0_131-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.131-b11, mixed mode)
$ rpm -ql|sort
jdk1.8.0_131-1.8.0_131-fcs.x86_64   # 能查询到rpm安装的jdk名字
$ rpm -ql jdk1.8.0_131-1.8.0_131-fcs.x86_64   # 查询jdk的目录为
/usr/java/jdk1.8.0_131   # 在稍后的web几面里面配置相关环境可以填这个路径
```

后续相关待补充……[Jenkins wiki参考](https://wiki.jenkins.io/display/JENKINS/Installing+Jenkins+on+Red+Hat+distributions)
