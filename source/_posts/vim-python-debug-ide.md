title: vim下的python调试环境
date: 2014-08-05 14:40:01
categories:
- VIM
tags:
- VIM
- program
- debug
- python
- php
---

大家一直都把vim当成编辑器，可是我已经改观了，vim太强大了。今天在找vim下的python断点调试的方法，果然不是问题。通用DBGPavim插件就能实现断点调试。下面是它用来调试php的原理图，python也是一样的。
![](https://brookhong.github.io/assets/images/dbgpavim0.png)
原理也不难理解，DBGPavim在vim上开启了一个服务端，通用网络接收程序的运行状态并通过vim展示出来。这里php是通过Xdebug做为调试器与DBGPavim通信的。Python则是用ActiveState提供的Komodo Python Remote Debugging Client的调试器运行的。
<!--more-->
##安装DBGPavim
DBGPavim插件本身是用Python实现的，所以需要你的VIM支持Python 2.6或2.7。打开你的VIM，输入命令
```bash
:version
```
如果能看到“+python”,说明你的VIM是支持Python的。 如果看到的是“-python”，说明你的VIM不支持Python，你可以按如下步骤编译自己的VIM：
```bash
export path=/path/to/python2.x/bin:$PATH
./configure --prefix=/opt/vim --enable-pythoninterp --with-python-config-dir=/usr/lib/python2.7/config make
make install
```
 
注：这里的/usr/lib/python2.x/config取决于你把Python2.7安装到什么位置。

如果有Vundel，在vim命令状态下安装就可以了,没有的话，从[github](https://github.com/brookhong/DBGPavim)下载DBGPavim，放到你的~/.vim目录下.
接着编辑的你的~/.vimrc，加入以下两行：
```bash
let g:dbgPavimPort = 9009
let g:dbgPavimBreakAtEntry = 0
```
注：这里的Port就是DBGPavim要监视的端口，要与调试器的端口一致，默认是9000。 dbgPavimBreakAtEntry=0告诉VIM不在入口处停下，这样只会在断点处停下。

你可以重新启动VIM，按F5检查你的DBGPavim配置是否正确。如果你配置成功的话，你会做VIM窗口的右下角看到提示信息如下：
```bash
bap-LISN-9009
```
它表示VIM目前正在监听9009端口，bap说明它只会在断点处停下，其他提示信息格式如下：
```bash
<bae|bap>-<LISN|PENDn|CONN|CLSD>
```
有两种断点状态，一种是只在断点的地方停下来，一种是每到到一个入口或出口都要停下，看情况选择吧
```bash
bae Break At Entry，在入口处停下
bap Break only At breakPoints，只在断点处停下
```
调试器状态
```bash
LISN	调试器已启动，正处于监听状态。
PEND-n	调试器已捕捉到连接请求，可以按F5进入调试模式了。
CONN	VIM正处于调试模式中。
CLSD	调试器已停止。
```

安装成功后，打开python程序，按F5就会进�入监听模式了，这个模式下可以设置断点。这里要提的是，在调试模式下，你之前设置的F1－F12的按键映射都会被取代掉，这样就不用担心按键冲突了（我一开始不知道，一直在排按键，没那么多剩余按键可用，囧）。等到调试器连接上来，再按一次F5，就可以进行调试了。

##安装调试器pydbgp
1. 从[这里](http://code.activestate.com/komodo/remotedebugging/)下载安装Komodo Python Remote Debugging Client，把解压后的bin目录加到你的PATH路径中,主要用的程序是pydbgp。这里我遇到一个问题，主目录下有两套程序，一套是python2.x，一套是3.x。不知道是不是目录没放好，pydbgp找不到信赖的dbgp程序包。从pythonlib里把所有问题都复制到pydbgp同一个目录就好了。
2. 打开vim点F5进入调试模式，F10设置好断点。
3. 通过pydbgp运行你的Python程序，如
```bash
pydbgp -d 127.0.0.1:9009 test.py   #这里端口要与DBGPavim设置的一致
```
4. 回到vim窗口，应该能看到提示信息PEND-1。
5. 按F5开始调试

##调试操作
VIM normal模式下
```bash
F5	启动调试监听，或者有可调试连接时进入调试模式。
F6	停止调试监听。
F8	切换dbgPavimBreakAtEntry的值，按这个键你可以看到状态栏提示信息在bae和bap之间切换，即是否在PHP程序入口处停下。
F10	在当前行设置或删除断点，在调试模式下同样适用。
```

调试模式下
```bash
F1	打开或关闭帮助窗口
F2	单步进入
F3	单步跳过
F4	单步退出
F5	继续执行直到下一个断点，如果后续没有断点就退出调试模式。
F6	停止调试，这个按键就导致VIM退出调试模式，并且停止调试监听。
F7	调试时执行php语句，按下F7后，用户可在变量查看窗口输入php语句，回车后执行。
F9	最大化某个子窗口，或者重置窗口布局。
F11	查看当前执行环境下的所有变量的值，在不同的堆栈层次，会有不同的结果。
F12	查看光标下的变量的值。
```

VIM命令，所有命令只有第一个字母为大写, 只能在调试模式下使用。
```bash
:Bl	列出所有断点
:Bp	与F10功能相同
:Dp	这个命令可用于快速调试当前文件，它实现了如下功能：

    1. 检查命令行下XDebug/pydbgp的设置是否正确
    2. 启动调试器监听
    3. 用php/pydbgp执行当前文件

:Pg <longfoo>	查看较长变量的值，比如:Pg $this->savings[3]
:Up	调用堆栈往上一级
:Dn	调用堆栈往下一级
:Wc [$foo]	打开/关闭对变量$foo的监视。如果没有参数，就监视当前执行环境下的所有变量。
:We <foo>	打开/关闭对语句foo的监视，即每一单步后自动执行foo语句。
:Wl	列出所有被监视的变量或语句。
:Children <n>	对于数组默认显示前1024个元素，这个命令可以修改。
:Depth <n>	对于复杂变量，默认只显示下一层成员，这个命令可以设置限制多层。
:Length <n>	对于字符串变量，默认执行显示前1024个字符，这个命令可以设置显示长度。
```


