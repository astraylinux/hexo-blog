title: LVM动态磁盘简记
date: 2011-11-30 19:18:00
categories:
- Linux
tags:
- disk
- Linux
---

搭建动态磁盘LVM
<!--more-->

##LVM: PV, PE, VG, LV
LVM 其实就是将几个实体的 partitions 透过软件组合成一块看起来像是独立的大磁盘,而要用这块大磁盘,就得要再将他分割成为可以使用的partition 才行!而我们知道每个 partition 上面的 filesystem因为 block 大小的不同而有限制, 同样的, LVM 的大磁盘大小也是有限制的,每个块称为一个 PE 。
　　
##Physical Volume： PV
就是物理磁盘分区!我们必须要将原本的磁盘,例如 /dev/hda5, /dev/hda6 等等的partition ,利用 fdisk 等软件,将他们的 ID 改为 LVM (8e) ,并且修改磁盘的相关信息, 让他成为 LVM 可以使用的磁盘。
什么是 ID 啊?还记得使用 fdisk -l 看到的数据吧? ID 83是 Linux 的 partition , 82 则是 Swap 的代号!一块磁盘变成 PV 后, LVM 才能够利用该 partition 。
##Volume Group： VG
其实我们 LVM 主要的目的就是要建立这个 VG 啦!他主要就是将刚刚的一个或多个 PV 组合成为一个大磁盘~这个大磁盘可以作为后续的分割之用喔! 那么这个大磁盘的容量最大可到多大呢?
最大容量的值与底下的 PE 有关, 如果完全使用 LVM 的预设参数时,那么一个最大的 LVM 磁盘
可达到 256 GBytes。
##Physical Extend： PE
在建立 VG 的时候,我们同时需要指定 PE 这个数值!如果不指定的话,他预设是 4MB 的大小。
当 PE 为 4MB 时, VG 最大的容量就是 256 GBytes !那么这个 PE 是什么玩意儿??

我们在 磁盘档案系统 那个章节当中提到的 inode, block 与 filesystem 大小的相关性当中, 有提到在 ext2/ext3 档案系统的格式化过程中,不同的 block 大小将会影响到整个 filesystem 大小的支持度。那这个 PE 其实就有点像是 VG 的 block 啦! 所以他的大小将会影响到 VG 的最大值喔!如果你想要让你的 VG 大于预设的 256 GB 时, 记得要修改这个数值!(其实,一个 VG最大可以容许 65534 个 PE , 所以,修改 PE 值,当然就会影响到最大的 VG 容量! )
##Logical Volume： LV
这个 LV 就是最后被挂载到档案系统的 partition 啰~这个 LV 是由 VG 分割来的啦~ 他会建立一个装置代号,例如 /dev/vgname/lvname 在您的系统当中!

透过 PV, VG, LV 的规划之后,再利用 mkfs (mke2fs -j) 等等就可以将您的多个 partition 整合成为一个大磁盘, 再利用这个大磁盘来分割与格式化,就 OK 的啦!而且,这个大磁盘可以进行增加、减少容量的变化, 也就是说,这个 VG 大磁盘可以抽换 PV 哩!并且原有的数据,理论上,并不会被影响喔!是否很棒啊! 整个 LVM 的处理流程与各组件之间的相关性,我们直接以下图来看看吧!

![](http://hi.csdn.net/attachment/201111/30/0_1322652098so8F.gif)
![](http://hi.csdn.net/attachment/201111/30/0_1322651785HNQa.gif)
改变lv大小之后要用resize2fs才能实现大小的改变。
对lv分区进行操作要用全路径，/dev/vgname/lvname.

材料来自《鸟哥linux私房菜基础学习篇》

