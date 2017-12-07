---
title: Python多版本管理
date: 2017-06-16  18:53:25
categories:
- Python
tags:
- python
---

<!-- more -->

### Python多版本共存管理之pyenv

一般的系统都是自带2.*版本,有时候部署一些程序又需要用到3.x的，比如jumpserver就是需要3.5.x及以上的。所有Python需要用到多版本共存。

#### 1.安装pyenv

```shell
$ git clone https://github.com/pyenv/pyenv.git ~/.pyenv
$ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
$ echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile
$ echo 'eval "$(pyenv init -)"' >> ~/.bash_profile
$ exec $SHELL
```

**Zsh note**: Modify your `~/.zshenv` file instead of `~/.bash_profile`.例如像我使用的oh_my_zsh的就需要注意这个.

**Ubuntu and Fedora note**: Modify your `~/.bashrc` file instead of `~/.bash_profile`.

#### 2.Upgrading pyenv

```shell
$ cd $(pyenv root)
$ git pull
#升级到指定的版本
$ cd $(pyenv root)
$ git fetch
$ git tag
v0.1.0
$ git checkout v0.1.0
```

**安装Python的依赖包**

在安装Python时需要首先安装其依赖的其他软件包，已知的一些需要预先安装的库如下。

在CentOS/RHEL/Fedora下:

```shell
$ yum install readline readline-devel readline-static
$ yum install openssl openssl-devel openssl-static
$ yum install sqlite-devel
$ yum install bzip2-devel bzip2-libs
```
对付pyenv默认下载慢的问题

```shell
v=3.5.2|wget http://mirrors.sohu.com/python/$v/Python-$v.tar.xz -P ~/.pyenv/cache/;pyenv install $v   #V是版本号,去Sohu镜像站点下载.
export  PYTHON_BUILD_MIRROR_URL="http://pyenv.qiniudn.com/pythons/"   #或者使用这个七牛的镜像站点,但是好多新版本没有.
```

