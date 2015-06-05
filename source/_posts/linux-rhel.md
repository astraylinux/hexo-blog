title: RHEL学习笔记
date: 2011-11-29 09:20:00
categories:
- Linux
tags:
- Linux
- system
- security
- Net
- server
---

上了公开课，学了一些RHEL的知识，主要有：破解root密码和加密grub，防止别人破解root密码，网络配置，特殊权限和管理交换分区。
<!--more-->

##破解root密码及加密grub
* 选择操作系统内核，给当前内核发送一个参数"1"。或者"s"，或者"single"。都是表示，启动后进入单用户模式。
* 按Enter键，回到操作系统选择界面。按"b"启动进入单用户模式了。
* runlevel是查看当前运行级别，是否在单用户模式下；
* passwd 改root密码getenforce查看是否启用了selinux，selinux将有专门的章节讲解setenforce 0临时禁用selinux的保护，以更改root密码。
* init 5重启，直接进入运行级别5，图形界面

##防止别人破解root密码？单用户模式加密。
重要说明,在grub.conf文件中，输入加密密码。有明文件加密，md5加密2种

md5加密: grub-md5-crypt　生成密文。把生成的密钥输入到grub.conf文件中。建议：先备份grub.conf文件。 密钥输入在第1个title之前的一行即可：password --md5 加密密码。选择操作系统时，下行要求输入"p"：输入密码密码正确了，才能进入"e"。
这样就起到了保护作用了.

明文加密:非常简单了，直接在grub.conf文件中加入一行即可：password 明文密码

##网络配置
再更改网络配置文件中主机名：
```bash
vim/etc/sysconfig/network
```
其它3个文件将由NetworkManager管理 
重启网络服务：
```bash
service networkrestart
```
查看，并验证四个配置文件
```bash
vim /etc/sysconfig/network  ＃网关
vim /etc/hosts  #主机名
vim /etc/resolv.conf  #DNS
vim/etc/sysconfig/network-scripts/ifcfg-eth0 #网卡
```
重启电脑，以验证更改后的root密码和主机名


##３.特殊文件权限
* SUID：  会创建s与t权限，是为了让一般用户在执行某些程序的时候，能够暂时具有该程序拥有者的权限，SUID仅可用在“二进制文件（binary file）”，对目录是无效的。
* SGID：文件：在执行该程序的时候，它的有效用户组（effective group）将会变成该程序的用户组所有者，
目录：A目录内所建立的文件或目录的用户组，将会是此A目录的用户组。
* SBIT： 只能用于目录，在具有SBit的目录下，用户若在该目录下具有w及x权限，则当用户在该目录下建立 文件或目录时，只有文件拥有者与root才有权力删除。用chmod xyz filename的方式来设置filename的属性时：4为SUID，2为SGID，1为SBIT。大写的S、T表示“空的”，没有执行的权利(比如7666）。

##管理交换分区
用fdisk进行分区，并用mkswap格式化分区。
用blkid获取UUID，并把这些写入/etc/fstab（也可以用文件的绝对路径和文件名）。
```bash
UUID="获取到的UUID"   swap   swap   defaults    0   0
```
swapon　-s查看当前正在使用的交换分区，swapon -a　加载fstab文件启用交换分区。
