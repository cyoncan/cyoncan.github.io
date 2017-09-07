---
title: git-ssh
date: 2017-05-22  19:53:43
categories:
- Git
tags:
- github
---

<!-- more -->

记录:

同台电脑多github账号使用

```shell
$ ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_github -C "yourname@email.com"
$ ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_gitlab -C "hername@email.com"
$ eval $(ssh-agent -s)
$ ssh-add ~/.ssh/id_rsa_github
$ ssh-add ~/.ssh/id_rsa_gitlab
$ cd ~/.ssh
$ vim config
#账号一
Host github.com
HostName github.com
User git
IdentityFile C:/Users/win/.ssh/id_rsa_github

#账号二
Host gitlab.com
HostName github.com
User git
IdentityFile C:/Users/win/.ssh/id_rsa_gitlab
$ ssh -T git@github.com
Hi xxx! You've successfully authenticated, but GitHub does not provide shell access.
$ ssh -T git@gitlab.com
Hi xxx! You've successfully authenticated, but GitHub does not provide shell access.
```

