title: mysql数据库主从同步修复
tags:
  - mysql
  - server
  - Linux
categories:
  - server
date: 2014-12-03 23:54:48
keywords: mysql server Linux 损坏 数据库 修复

-----

维护一个主从读写分离的mysql库，一台运行主库，一台运行从库。遇到了几次问题，做下记录。

<!-- more -->

### 从库的一个表损坏，不能写入，不能打开
采用复制修复的方法进行修复，再移回去
```bash#
myisamchk -o table.MYI  
# myisamchk执行时，mysqld必需停止，而mysqlcheck则不用停止
# myisamchk 参数有 
# -im:检查错误
# -rq:快速修复
# -rB:修复加备份
# -o:重新生成修复)
```

### 主从库同步问题
如果logbin文件损坏，可以试着从主库的机器上复制logbin，可能主库的文件是正常的,实在需要修复主从的话，可以用工具：`Percona Toolkit`

####pt-table-checksum 安装
```bash
wget www.percona.com/downloads/percona-toolkit/2.2.2/percona-toolkit-2.2.2.tar.gz
tar  xf  percona-toolkit-{version}.tar.gz
cd percona-toolkit-{version} 

#安装依赖包
yum install -y perl mysql perl-DBD-MySQL

#pt-table-checksum安装步骤
perl
Makefile.PL
make
make install
```

修复的具体步骤如下
#### 在Master库上授权
```bash
GRANT update,insert,delete,SELECT, PROCESS, SUPER, REPLICATION SLAVE ON *.* TO  checksum@'192.168.1.2（Master ip）'  IDENTIFIED BY '123' ;
```

#### 在slave上授权
```bash
     GRANT SELECT, PROCESS, SUPER, REPLICATION SLAVE ON *.* TO 'checksums'@'192.168.1.2（Master ip）' IDENTIFIED BY '123';
```

#### 创建中间数据库和表
```bash
#创建库
CREATE DATABASE pst CHARACTER SET utf8;
#创建表
CREATE TABLE checksums (
  db            char(64)    NOT NULL,
  tbl            char(64)    NOT NULL,
  chunk_time    float            NULL,
  chunk_index    varchar(200)    NULL,
  lower_boundary text            NULL,
  upper_boundary text            NULL,
  this_cnt      int          NOT NULL,
  master_cnt    int              NULL,
  ts            timestamp    NOT NULL,
  PRIMARY KEY (db, tbl, chunk),
  INDEX ts_db_tbl (ts, db, tbl)
) ENGINE=InnoDB;
```

主从库要开启对pst库的logbin和同步选项，都要重启，确保主从库同步正常
```bash
#创建从库信息表
 CREATE TABLE `dsns` ( `id` int(11) NOT NULL AUTO_INCREMENT, `parent_id` int(11) DEFAULT NULL, `dsn` varchar(255) NOT NULL, PRIMARY KEY (`id`) );
#写入从库信息
INSERT INTO dsns (parent_id,dsn) values(1, "h=192.168.1.1,u=checksums,p=123,P=3306");
```
    
#### 检查是否有不同步数据：
```bash
#检查结果会写入pts.checksums表里
pt-table-checksum h='192.168.1.2',u='checksum',p='123',P=3306 --databases dbname --tables tablename --nocheck-replication-filters --no-create-replicate-table --replicate=pts.checksums --recursion-method=dsn=D=pts,t=dsns
```

#### 打印因差别数据要执行的sql语句
```bash
#这里用的是主从库都有写权限的账号
pt-table-sync --print --replicate=pts.checksums --charset=utf8  h=192.168.1.2,u=root,p=wtf0000
```
    
#### 执行主从同步修复操作：
```bash
pt-table-sync --execute --replicate=pts.checksums --charset=utf8  h=192.168.1.2,u=root,p=wtf0000  
#可以用"--ignore-columns=" 忽略一些不update的字段来加快速度
```



