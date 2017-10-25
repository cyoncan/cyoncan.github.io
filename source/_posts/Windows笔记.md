---
title: Windows
date: 2017-10-10  19:51:39
categories:
- Windows
tags:
- cmd
---

<!-- more -->

Windows PowerShell和CMD控制台代码页设置

```powershell
代码页     国家（地区）或语言
437        美国 
850        多语言（拉丁文 I）
852        斯拉夫语（拉丁文 II）
855        西里尔文（俄语） 
857        土耳其语
860        葡萄牙语
861        冰岛语
863        加拿大 - 法语
865        日耳曼语
866        俄语
869        现代希腊语 
936        简体中文
950        繁体中文
65001      UTF-8

语法 
chcp [NNN]

通过注册表修改代码页：
1 win键+R打开“运行”对话框，输入regedit打开注册表编辑器。
2 找到 [HKEY_CURRENT_USER\Console\%SystemRoot%_system32_cmd.exe]
3 修改或者新建"CodePage" = dword:000003a8
【注】十六进制"000003a8"或十进制"936"，表示“936 (ANSI/OEM - 简体中文 GBK)”。 
```

