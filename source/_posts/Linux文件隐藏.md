---
title: Linux文件隐藏属性
date: 2017-04-24 12:21:11
categories:
- CentOS
tags:
- linux
---
<!-- more -->
**chattr命令用于设置文件的隐藏权限，格式为：“chattr [参数] 文件”。**

chattr设置文件的隐藏权限，如果要将某个隐藏功能添加到文件，使用**+参数**，如果要将某个隐藏功能移出文件，使用**-参数**。常见的隐藏权限包括有：

| 参数   | 作用                                  |
| ---- | ----------------------------------- |
| i    | 将无法对文件进行修改,若对目录设置后则仅能修改子文件而不能新建或删除。 |
| a    | 仅允许补充（追加）内容.无法覆盖/删除(Append Only)。   |
| S    | 文件内容变更后立即同步到硬盘(sync)。               |
| s    | 彻底从硬盘中删除，不可恢复(用0填充原文件所在硬盘区域)。       |
| A    | 不再修改这个文件的最后访问时间(atime)。             |
| b    | 不再修改文件或目录的存取时间。                     |
| D    | 检查压缩文件中的错误。                         |
| d    | 当使用dump命令备份时忽略本文件/目录。               |
| c    | 默认将文件或目录进行压缩。                       |
| u    | 当删除此文件后依然保留其在硬盘中的数据，方便日后恢复。         |
| t    | 让文件系统支持尾部合并（tail-merging）。          |
| X    | 可以直接访问压缩文件的内容。                      |

**lsattr命令用于显示文件的隐藏权限，格式为：“lsattr [参数] 文件”。**

```bash
$ root@stu  ~  lsattr
-------------e- ./readme.txt
$ root@stu  ~  chattr +a readme.txt
$ root@stu  ~  rm -rf readme.txt
rm: cannot remove `readme.txt': Operation not permitted
$ root@stu  ~  lsattr readme.txt
-----a-------e- readme.txt
$ root@stu  ~  chattr -a readme.txt
$ root@stu  ~  rm readme.txt
$ root@stu  ~  ll
total 0
```

