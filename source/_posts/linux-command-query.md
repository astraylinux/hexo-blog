title: 查询linux系统命令
date: 2011-12-25 13:26:00
categories:
- Linux
tags:
- Linux
- Bash
---

用tab键列出的命令只能是以字符开头的命令，有时候只记得命令的中间部分，就不好查询了。这时我们可以把系统中所有的命令集中到一个文件中，再用cat和grep来查询，就可以通过中间部分查找命令了。

<!--more-->

##创建生成数据文件的命令。
```bash
echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games
#在/bin/里添加shell文件comm-update
vim /bin/comm-update
```
内容就是将可执行文件的路径都通过ls输出到文件:
```bash
#!/bin/bash
echo >~/.comm.data #先将文件清空
for i in /usr/local/sbin /usr/local/bin /usr/sbin /usr/bin /sbin /bin /usr/games
do
    ls $i >> ~/.comm.data
done
```
这样有安装新软件，comm-update一下就行了。
　　
##在自己的.bashrc文件添加alias
```bash
echo "alias commquery='cat ~/.comm.data |grep ' " >>~.bashrc
source ~/.bashrc
```
　　
这样就可以通过commquery来查询命令了
```bash
$ commquery ftp
apt-ftparchive
ftp
lftp
lftpget
netkit-ftp
pftp
sftp
vsftpd
```
