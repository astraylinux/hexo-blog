title:  centos安装php
tags:
  - server
  - Linux
  - php
  - config
  - install
categories:
  - server
date: 2014-03-13 12:36:35
keywords: server linux php install config 安装 配置

-----

在centos6.5下安装php并与nginx一起使用。

<!-- more -->

安装依赖程序
```bash
#安装编译器
yum install -y gcc gcc-c++ cmake
#安装依赖包，这是根据configure的prefix需求加的
yum -y install ncurses-devel libxml2-devel bzip2-devel libcurl-devel \
    curl-devel libjpeg-devel libpng-devel freetype-devel net-snmp-devel 
```

编译指令，按自己的需求修改，安装nginx后会有nginx账号，没有的话，也可以自己增加账号，并在`--with-fpm-user=${user}`, `--with-fpm-group=${group}`指定。

```bash
./configure --with-config-file-path=/usr/local/php/etc/\
--with-libdir=lib64\
--with-iconv-dir=/usr/local\
--enable-mysqlnd\
--with-mysql=mysqlnd\
--with-mysqli=mysqlnd\
--enable-pdo\
--with-pdo-mysql=mysqlnd\
--enable-fpm\
--with-fpm-user=nginx\
--with-fpm-group=nginx\
--with-pcre-regex\
--with-zlib\
--with-bz2\
--with-snmp\
--with-curl\
--with-mcrypt\
--with-libxml-dir\
--with-gd\
--with-jpeg-dir\
--with-png-dir\
--with-zlib-dir\
--with-freetype-dir\
--with-mhash\
--enable-ftp\
--enable-gd-native-ttf\
--enable-gd-jis-conv\
--enable-dba\
--enable-mbstring\
--enable-pcntl\
--enable-xml\
--enable-calendar\
--enable-shmop\
--enable-sockets\
--enable-zip\
--enable-bcmath\
--disable-ipv6\
--disable-phar\
--disable-rpath\
```

编译选项说明参考[核心配置选项列表](http://php.net/manual/zh/configure.about.php).
如果编译安装完后，需要的模块没有安装，可以在源码目录的`ext`目录下找，如果有的话，可以用phpize安装, 以xml为例,

```bash
[root]# cd ext/xml/
[root]# phpize
[root]# ./configure
[root]# make
[root]# make install
```
这样就会生成一个`xml.so`库文件，并安装的模块的目录，这时还要在`php.ini`里加入下面配置，有些模块需要一些配置参数，具体看模块的说明[扩展库列表／归类](http://php.net/manual/zh/extensions.php)
```
extension=xml.so
```

