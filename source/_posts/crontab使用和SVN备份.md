---
title: Crontab使用之SVN备份
date: 2017-03-23 22:16:21
categories:
- SVN
tags:
- crontab
- svn备份
---
<!-- more -->
### crontab使用之SVN备份

##### 1.直接crontab -e 加入定时

   ```bash
   */5 * * * * /data/script/svnbak.pl
   ```

##### 2.修改/etc/crontab

   ```bash
   SHELL=/bin/bash
   PATH=/sbin:/bin:/usr/sbin:/usr/bin
   MAILTO=root  (设置=""就不会发出电子邮件)
   HOME=/
   #run-parts
   */5 * * * * /data/script/svnbak.pl
   # For details see man 4 crontabs

   # Example of job definition:
   # .---------------- minute (0 - 59)
   # |  .------------- hour (0 - 23)
   # |  |  .---------- day of month (1 - 31)
   # |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
   # |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
   # |  |  |  |  |
   # *  *  *  *  * user-name command to be executed
   ```

##### 3.编写SVN备份脚本(perl)

   日备份增量备份:

```perl
$ vim gnu.com.pl
#!/usr/bin/perl -w
my $svn_repos="/data/repos/gnu.com";
my $backup_dir="/data/backup/svn/";
my $next_backup_file = "daily_incremental_backup.".`date +%Y%m%d`;
open(IN,"$backup_dir/last_backed_up");
$previous_youngest = <IN>;
chomp $previous_youngest;
close IN;
$youngest=`svnlook youngest $svn_repos`;
chomp $youngest;
if ($youngest eq $previous_youngest)
{
  print "No new revisions to backup.n";
  exit 0;
}
my $first_rev = $previous_youngest + 1;
print "Backing up revisions $youngest ...n";
my $svnadmin_cmd = "svnadmin dump --incremental --revision $first_revyoungest $svn_repos > $backup_dir/$next_backup_file";
`$svnadmin_cmd`;
open(LOG,">$backup_dir/last_backed_up"); 
print LOG $youngest;
close LOG;
print "Compressing dump file...n";
print `gzip -g $backup_dir/$next_backup_file`; 
```

​	周备份完整备份:

```perl
$ vim gnu.com.pl
#!/usr/bin/perl -w
my $svn_repos="/data/repos/gnu.com";
my $backup_dir="/data/backup/svn/";
my $next_backup_file = "weekly_fully_backup.".`date +%Y%m%d`;
$youngest=`svnlook youngest $svn_repos`;
chomp $youngest;
print "Backing up to revision $youngestn";
my $svnadmin_cmd="svnadmin dump --revision 0youngest $svn_repos >$backup_dir/$next_backup_file";
`$svnadmin_cmd`;
open(LOG,">$backup_dir/last_backed_up");
print LOG $youngest;
close LOG;
print "Compressing dump file...n";
print `gzip -g $backup_dir/$next_backup_file`; 
```

```bash
$ chmod +x gnu.com.pl   (需要给每一个脚本修改可执行属性 chmod +x *)
```

##### 4.脚本加入定时执行(日备份周一到五23点,周备份每周六23点)

 方法一:

```shell
$ crontab -e
SHELL=/bin/zsh
PATH=/etc:/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin
HOME=/var/log
0 23 * * 6 /data/script/svn/weekly/gnu.com.pl
0 23 * * 1-5 /data/script/svn/daily/gnu.com.pl
```

方法二:

```bash
$ vim /etc/crontab
  0 23 * * 6 /data/script/svn/weekly/gnu.com.pl
  0 23 * * 1-5 /data/script/svn/daily/gnu.com.pl
```

##### 5.几个定时例子

```bash
00 03 * * 1-5 find /home "*.xxx" -mtime +4 -exec rm {} \;
每周一至周五3点钟，在目录/home中，查找文件名为*.xxx的文件，并删除4天前的文件.
0 */2 * * * /sbin/service httpd restart
每两个小时重启一次apache
50 7 * * * /sbin/service sshd start
每天7：50开启ssh服务 
```
