title: Linux常见缩写的解释
date: 2011-11-30 14:22:00
categories:
- Linux
tags:
- Linux
- system
---

一些Linux中常见的名词缩写，经常忘记意思，记下来
<!--more-->

##UTC: Coordinated Universal Time (协调世界时间)
世界统一时间，世界标准时间，国际协调时间。

##NTP: Network Time Protocol（NTP）
是用来使计算机时间同步化的一种协议，它可以使计算机对其服务器[](http://baike.baidu.com/view/899.htm)或时钟源（如石英钟，GPS等等)做同步化。

##LDAP: Lightweight Directory Access Protocol (轻量目录访问协议)
简单说来，LDAP是一个得到关于人或者资源的集中、静态数据的快速方式。LDAP是一个用来发布目录信息到许多不同资源的协议。通常它都作为一个集中的地址本使用，不过根据组织者的需要，它可以做得更加强大。

##LVM: Logical Volume Manager(逻辑卷管理)
 它是Linux环境下对磁盘分区进行管理的一种机制，LVM是建立在硬盘和分区之上的一个逻辑层， 来提高磁盘分区管理的灵活性。

##NFS: Network File System (网络文件系统)
网络文件系统是FreeBSD支持的文件系统中的一种，也被称为NFS. NFS允许一个系统在网络上与他人共享目录和文件。通过使用NFS，用户和程序可以像访问本地文件一样访问远端系统上的文件。
##SMB:  Server Message Block (服务信息块协议)
SMB 一种客户机/服务器、请求/响应协议。通过 SMB协议，客户端应用程序可以在各种网络环境下读、写服务器上的文件，以及对服务器程序提出服务请求。此外通过 SMB 协议，应用程序可以访问远程服务器端的文件、以及打印机、邮件槽（mailslot）、命名管道（named pipe）等资源。
##SAMBA: 
samba是一个工具套件，在Unix上实现SMB(Server Message Block)协议，或者称之为 NETBIOS/LanManager协议。
##ISCSI:  Internet Small Computer System Interface (小型计算机系统接口）
iSCSI技术是一种由IBM公司研究开发的，是一个供硬件设备使用的可以在IP协议的上层运行的SCSI指令集，这种指令集合可以实现在IP网络上运行SCSI协议，使其能够在诸如高速千兆以太网上进行路由选择。iSCSI 技术是一种新储存技术，该技术是将现有SCSI接口与以太网络(Ethernet)技术结合，使服务器可与使用IP网络的储存装置互相交换资料。
##CUPS:  Common UNIX Printing System
CUPS是Fedora Core3中支持的打印系统,它主要是使用IPP(Internet Printing Protocol)来管理打印工作及队列,但同时也支持"LPD"(Line Printer Daemon)和"SMB"(Server Message Block)以及AppSocket等通信协议.
##SSH:Secure Shell （安全外壳协议）
SSH 是目前较可靠，专为远程登陆会话和其他网络服务提供安全性的协议。利用 SSH 协议可以有效防止远程管理过程中的信息泄露问题。
##VNC: Virtual Network Computing (虚拟网络计算机 )
VNC是一款优秀的远程控制工具软件，由著名的AT&T的欧洲研究实验室开发的。VNC是在基于UNIX和Linux操作系统的免费的开放源码软件，远程控制能力强大，高效实用，其性能可以和Windows和MAC中的任何远程控制软件媲美。在Linux中，VNC包括以下四各命令：vncserver，vncviewer，vncpasswd，和vncconnect。大多数情况下我只需要其中的两个命令：vncserver和vncviewer。
##KVM: Kernel-based Virtual Machine ( 系统虚拟化模块 )
KVM是rhel5.4推出的最新虚拟化技术，目前红帽只支持在64位的rhel5.4上运行kvm,同时硬件需要支持VT技术，使用kvm虚拟机的时候需要关闭SELinux，是一个x86 Linux全虚拟化解决方案，需要硬件支持虚拟化扩展（Intel VT 或AMD-V），它由一个载入时内核模块kvm.ko（提供核心虚拟化基础设施）和一个处理器特殊模块kvm-intel.ko或kvm-amd.ko组成，在它上层需要修改过的QEMU。
##ACL: Access Control List (访问控制列表 )
ACL是路由器和交换机接口的指令列表，用来控制端口进出的数据包。ACL适用于所有的被路由协议，如IP、IPX、AppleTalk等。这张表中包含了匹配关系、条件和查询语句，表只是一个框架结构，其目的是为了对某种访问进行控制。


