title: 安装coreseek
date: 2014-04-27 17:03:00
categories:
- search
tags:
- search
- install
---
开始转到做搜索，公司用的是sphinx的中文扩展版本coreseek，简单记录一下安装过程.   
<!--more-->
  
OS: centos 64bit

##下载并解压coreseek
```bash
$ cd /usr/local/src
$ wget http://www.coreseek.cn/uploads/csft/4.0/coreseek-4.1-beta.tar.gz
$ tar zxvf coreseek-4.1-beta.tar.gz
```
##安装词库
```bash
$ cd coreseek-4.1-beta
$ cd mmseg-3.2.14
$ ./bootstrap
$ ./configure --prefix=/usr/local/mmseg3
$ make && make install
```

##遇到的问题及解决方案：
###1.error: cannot find input file: src/Makefile.in
```bash
$ aclocal
$ libtoolize --force
$ automake --add-missing
$ autoconf
$ autoheader
$ make clean
```
###2.运行aclocal时出错
错误：configure.in:26: warning: "macro `AM_PROG_LIBTOOL' not found in library"
```bash
$ yum -y install libtool
#安装coreseek (要用python 源的话，要先安装python2.6或2.7）
$ cd ../csft-4.1sh 
$ buildconf.sh
$ export LIBS="-ldl -lutil -Xlinker -export-dynamic"
$ ./configure --prefix=/usr/local/coreseek  --without-unixodbc --with-mmseg --with-mmseg-includes=/usr/local/mmseg3/include/mmseg/ --with-mmseg-libs=/usr/local/mmseg3/lib/ --with-mysql --with-python
$ export LIBS="-liconv"
$ make && make install
```

##测试安装情况
```bash
$ cd /usr/local/src/coreseek-4.1-beta/testpack
$ cat var/test/test.xml
$ mmseg -d /usr/local/mmseg3/etc var/test/test.xml
$ indexer -c /usr/local/src/coreseek-4.1-beta/testpack/etc/csft.conf --all
$ search -c etc/csft.conf 网络搜索
```

