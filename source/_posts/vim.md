---
title: Vim常用
date: 2017-04-10 20:17:22
categories:
- Linux
tags:
- vim
- linux
---
<!-- more -->
# Linux vim常用命令

### 1.替换

```bash
:s/abc/efg/      替换当前行,第一个 abc 为 efg
:s/abc/efg/g     替换当前行,所有   abc 为 efg
:n,$s/xyz/org/   替换第 n 行开始到最后一行中每一行的第一个 xyz 为 org
:n,$s/xyz/org/g  替换第 n 行开始到最后一行中每一行所有     xyz 为 org
 n 为数字，若 n 为 . ，表示从当前行开始到最后一行
:%s/xyz/org/     (等同于:g/xyz/s//org/)  替换每一行的第一个 xyz 为 org
:%s/xyz/org/g   （等同于:g/xyz/s//org/g）替换每一行中所有   xyz 为 org
```

### 2.vim执行：wq清楚屏幕上上一次编辑过的内容

```bash
vim退出后清屏,屏幕不显示之前编辑的内容.
TERM=xterm; export TERM
```

### 3.删除全部

```shell
:%d
```

### 4.复制

```shell
v      进入字符块选择模式
↑↓←→   选择要复制的字符
yy     复制
```

### 5.批量注释（插入）

```shell
Ctrl+v   进入矩形块选择，方向键上下选择需要注释列头，I插入//，Esc退出。
G 列选择
c 块选择，替换
```

