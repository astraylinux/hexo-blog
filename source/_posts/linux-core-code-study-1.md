title: linux内核0.12－－第一章，概述
date: 2012-02-23 13:41:00
categories:
- Linux
tags:
- Linux
- core 
- assembly
- unix
---

开始学习linux内核了，对linux系统的理解还不够深，对于比较新的内核理解困难，于是选择了这本讲解早期内核的书来看，并做做笔记。
<!--more-->

##第一章，概述
介绍了linux的历史，开发背景，0.12版本内核的主要文件结构，及本书各个章节的内容分布。

linux系统的发展依赖于：unix操作系统，minix操作系统，gnu计划，posix标准和internet，本节前半部分主要是对它们的介绍。

后半部分则是linux诞生的背景（其作者是荷兰21岁的赫尔辛基大学的学生linus）。
linux早期的主要版本，如下：
![](http://hi.csdn.net/attachment/201202/23/0_1329979501rhjt.gif)
![](http://hi.csdn.net/attachment/201202/23/0_1329979520fSx8.gif)
现在内核各版本的发布日期及大小：
![](http://hi.csdn.net/attachment/201202/23/0_1329979546by80.gif)

##Linux0.12版本内核的主要文件
| 文件 | 说明 |
| ----------------- | ----------------------------- | 
| bootimage-0.12.Z |具有美国键盘代码的压缩启动映像文件 | 
| rootimage-0.12.Z | 以1200KB压缩的要文件系统映像文件 | 
| linux-0.12.tar.Z | 内核源代码文件，大小为130KB，展开后也仅有463KB | 
| as86.tar.Z | Bruce Evans的二进制执行文件，是16位的汇编程序和装入程序 | 
| INSTALL-0.11 | 更新过的安装信息文件 | 

在[http://oldlinux.org/Linux.old/](http://oldlinux.org/Linux.old/)有关于该书的一些文件，代码及说明文档。
本章其他内容是对各章节的概述，不再例举。


