---
title: 记一次阿里云RDS MySQL CPU跑满100居高不下的分析
date: 2017-04-25 18:22:55
categories:
- MySQL
tags:
- mysql
---

**这两天呢经常收到MySQL的CPU报警信息,不过我知道原因是PHP在跑一个订单累积返工分的demo.因为已经跑了好几周有了,开始数量少,到现在订单已经五六十万了吧,不是很清楚.但是最近RDS CPU老报警.不过还能回到正常值,昨晚发现CPU满了,还一直不下.重启实例后,于是又正常,今天又开始继续跑单子了.CPU就一直99,登录RDS结合阿里云文档分析了一番,诊断图如下:**

*会话没有截全,共有60左右个吧,都是同一用户,同一数据库,均为select语句*

![](http://ooz08pfj3.bkt.clouddn.com/QQ20170425140714.png)

![](http://ooz08pfj3.bkt.clouddn.com/QQ20170427102405.png)

**根据阿里云文档介绍 RDS CPU 跑满100的分析介绍如下:**

**1.原理：**cpu 消耗过大通常情况下都是有慢sql 造成的，这里的慢sql 包括全表扫描，扫描数据量过大，内存排序，磁盘排序，锁争用等待等；

**2.表现现象：**sql 执行状态为：**sending data**，**Copying to tmp table**，**Copying to tmp table on disk**，**Sorting result，locked**;

**3.分析原因：**用户可以登录到rds，通过**show processlist**查看当前正在执行的sql，当执行完**show processlist**后出现大量的语句，通常其状态出现**sending data**，**Copying to tmp table**，**Copying to tmp table on disk**，**Sorting result, Using filesort** 都是sql有性能问题；

- **A.  sending data**表示：sql正在从表中查询数据，如果查询条件没有适当的索引，则会导致sql执行时间过长；
- **B. Copying to tmp table on disk**：出现这种状态，通常情况下是由于临时结果集太大，超过了数据库规定的临时内存大小，需要拷贝临时结果集到磁盘上，这个时候需要用户对sql进行优化；
- **C. Sorting result, Using filesort**：出现这种状态，表示sql正在执行排序操作，排序操作都会引起较多的cpu消耗，通常的优化方法会添加适当的索引来消除排序，或者缩小排序的结果集；

**执行 show processlist , 或者直接进入RDS >DMS里面使用阿里云的后台管理,生成诊断报告,查看state. 60个左右的查询会话全是sending data,还有下面检测出来的慢SQL**

*下午又了解了下,跑订单的那个任务从昨天开始查询操作了,我不懂业务逻辑上的东西,反正应该可以确定这次引起CPU居高不下的原因应该就是数据库大量的查询,花的时间太长了*

因为服务器性能也就在这,上图也列出来了,可定不是因为连接数的问题.结合文档来看呢,还是数据库的索引和语句方面需要继续优化.问题找到了交给phper去添加.

**注：由于查询执行效率低（查询访问表数据行数多）而导致实例 CPU 使用率高是RDS MySQL非常常见的问题。** 

参考:

[RDS实例CPU超过100%的分析](https://help.aliyun.com/knowledge_detail/41684.html?spm=5176.7841698.2.11.mxnkJC)

[RDS MySQL CPU使用率高情况的原因和解决](https://help.aliyun.com/knowledge_detail/41715.html)

[RDS for MySQL查询缓存 (Query Cache) 的设置和使用](https://help.aliyun.com/knowledge_detail/41717.html)