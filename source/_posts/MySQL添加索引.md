---
title: MySQL添加索引
date: 2017-04-25 15:53:22
categories:
- MySQL
tags:
- mysql索引
---
<!-- more -->
### 一.查看索引

```bash
mysql> show index from tablename;
或
mysql> show keys from tablename;
```

![](http://ooz08pfj3.bkt.clouddn.com/QQ20170425221802.png)

- **Non_unique**: 如果索引不能包括重复词,则为0,如果可以则为1.
- **Key_name**: 索引的名称
- **Seq_in_index**: 索引中的列序列号,从1开始
- **Column_name**: 列名称
- **Collation**: 列以什么方式存储在索引中。在MySQL中，有值‘A’（升序）或NULL（无分类）
- **Cardinality**：索引中唯一值的数目的估计值。通过运行ANALYZE TABLE或myisamchk -a可以更新。基数根据被存储为整数的统计数据来计数，所以即使对于小型表，该值也没有必要是精确的。基数越大，当进行联合时，MySQL使用该索引的机会就越大。
- **Sub_part**：如果列只是被部分地编入索引，则为被编入索引的字符的数目。如果整列被编入索引，则为NULL。
- **Packed**：指示关键字如何被压缩。如果没有被压缩，则为NULL。
- **Null**：如果列含有NULL，则含有YES。如果没有，则该列含有NO。
- **Index_type**：用过的索引方法（BTREE, FULLTEXT, HASH, RTREE）。
- **Comment**：更多评注。


**查看数据库表中存储引擎的类型**

```bash
mysql> show table status from dbname where name='tablename';
或者
mysql> use dbname
mysql> show table status where name='tablename';
```

### 二.创建索引原则

##### 1. 频繁的作为查询条件的字段应该创建索引

##### 2. 唯一性太差的字段不适合单独创建索引，即使频繁作为查询条件

##### 3. 非常频繁更新的字段不适合创建索引

##### 4.不会出现在where子句中的字段不该创建索引

##### 5.最左配原则...

eg:

**i. 在where子句中出现的列, 在join子句中出现的列,** 而不是在select关键字后选择列表的列.

**ii. 索引列的基数越大，索引的效果越好。例如，存放出生日期的列具有不同的值，很容易区分行，而用来记录性别的列，只有M和F,则对此进行索引没有多大用处，因此不管搜索哪个值，都会得出大约一半的行**

**存储引擎对索引类型的支持情况:**

| 存储引擎        | 允许的索引类型    |
| ----------- | ---------- |
| MyISAM      | BTREE      |
| InnoDB      | BTREE      |
| MEMORY/HEAP | HASH,BTREE |

### 三.索引语法:

```bash
create [unique|fulltext|spatial] index index_name [using index_type] on table_name (index_column_name);
alter table table_name index index_name;
drop index index_name on table_name;
```

参考:

[美团点评技术团队-MySQL索引原理及慢查询优化](http://tech.meituan.com/mysql-index.html)

