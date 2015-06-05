title: ubuntu 多屏设置 “虚拟大小大于可用大小”解决方法
date: 2013-04-08 10:52:00
categories:
- Linux
tags:
- Linux
- problem
- config
---

最近用新装的ubuntu12.04装完显卡驱动后，双屏不能正常扩展，于是在配置管理那边设置显示，报了一个错："需要的虚拟大小大于可用大小:需求=(3360, 1050),最小=(320, 200),最大=(1680, 1680)"

<!--more-->

要设置xorg.conf，ubuntu12.04默认没有这个文件，所以要生成它。先ctrl＋alt＋f1切到文字模式，停掉Xwindow服务，执行
```bash
sudo /etc/init.d/lightdm stop 
#接着生成xorg.conf
sudo X -configure
```
这样就在主目录下生成了一个xrog.conf.new，这时要是直接把文件复制成/etc/X11/xorg.conf是不够的，没有设置好虚拟屏幕的大小
```bash
sudo /etc/init.d/lightdm start #重启xwindow
```
编辑xorg.conf.new 再复制过去。主要是改了下面这个部分，modes是每个屏幕的分辨率，Virtual是所有屏幕的分辨率总和。


```bash
Section "Screen"
	Identifier "Screen0"
	Device     "Card0"
	Monitor    "Monitor0"
	SubSection "Display"
		Viewport   0 0
		Depth     24
		Modes "1680x1050"
		Virtual 3360 1050
	EndSubSection
EndSection
```
再ctrl＋alt＋f1切回文字模式，开关一次lightdm服务
```bash
sudo /etc/init.d/lightdm restart
```
ctrl＋alt＋f7切回xwindow后，就可以用设置里的屏幕设置设置多屏，而不会分辨率不够了。




