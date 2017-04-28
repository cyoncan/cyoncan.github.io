---
title: 理解xargs
date: 2017-04-20 00:16:21
categories:
- 系统命令
tags:
- xargs
---
<!-- more -->
### 理解xargs命令工作和使用

```bash
⚡ root@intelnet:$ xargs 
hello
hello
```

*工作形式: 从标准输入stdin读取数据,根据输入读取执行命令作为参数提供给它一次或多次.输入中的任何空白和空格均视为分隔符,空行被忽略.进入xargs ,进行数据输入, Ctrl+D告诉xargs结束输入任务,echo命令被自动执行,并且再次打印 hello.*

*echo是xargs默认的执行命令,我们可以指定其它命令作为参数传递给xargs , 然后通过stdin传递要查询的文件和类型作为输入的名称.如下:*

```bash
⚡ root@intelnet:$ xargs find -name
 "*.log"
```
