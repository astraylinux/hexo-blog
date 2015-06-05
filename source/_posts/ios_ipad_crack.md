title: 苹果ipa软件包破解
date: 2013-03-27 16:06:00
categories:
- IOS
tags:
- IOS
- Apple
- security
---

这是国内很多苹果应用市场都要面临的问题，在正版软件没有通过授权之前，这些软件市都是做的越狱软件。做越狱软件最重要的就是将正版软件破解使得能分享到其他越狱设备上。
破解最基本原理，其实是利用IOS软件被加载到内存中时，程序的启动加密段是解密的状态，将这些解密完的数据写到软件中，就算是破解成功了。（实际破解中各种版本系统及CPU不同而有不同的处理方式）。

<!--more-->

##苹果的验证机制
Appstore上的应用都采用了DRM（digital rights management）数字版权加密保护技术，直接的表现是A帐号购买的app，除A外的帐号无法使用，其实就是有了数字签名验证，而app的破解过程，实质也是去除数字签名的过程。
去除过程包括两部分：
ipa文件都是使用苹果公司的FairPlay DRM技术进行加密保护
appsync没有装是不能安装的，程序没有破解是不能运行的

##破解条件
###一，设备越狱，获得root权限，实现安装运行破解程序
首先要去除掉设备上的签名检查，允许没有合法签名的程序在设备上运行，代表工具：
AppSync（作者：Dissident ，Apptrack网站的核心人物）
实现原理，iOS 3.0 出现，不同的iOS版本，原理都不一样  

* iOS 3：MobileInstallation将可执行文件的签名校验委托给独立的二进制文件/usrlibexec/installd来处理,而AppSync 就是该执行文件的一个补丁，用来绕过签名校验。
* iOS 4.0后，apple留了个后门（app给开发者预留的用于测试的沙盒环境），只要在/var/mobile/下建立tdmtanf目录，就可以绕过installd的签名校验，但该沙盒环境会造成没法进行IAP购买，无法在GameCenter查看游戏排名和加好友，特征是进入Game Center会出现SandBox字样。AppSync for iOS 4.0 +修复了这一问题。
* iOS 5.0后，apple增加了新的安全机制来保护二进制文件，例如去掉二进制文件的符号表，给破解带来了难度。新版的AppSync for iOS 5.0+ 使用MobileSubstrate来hook libmis.dylib库的MISValidateSignatureAndCopyInfo函数来绕过签名验证

###二，解密mach-o可执行文件
一般采用自购破解的方法，即先通过正常流程购买appstore 中的app，然后采用工具或手工的方式解密安装包中的mach－o可执行文件。之所以要先获得正常的IPA的原因是mach－O文件是有DRM数字签名的，是加密过的，而解密的核心就是解密加密部分，而我们知道，当应用运行时，在内存中是处于解密状态的。所以首要步骤就是让应用先正常运行起来，而只有正常购买的应用才能达到这一目的，所以要先正常购买。

##文件结构
### Mach-O
在 OS X, 几乎所有的包含可执行代码的文件，如：应用程序、框架、库、内核扩展……, 都是以Mach-O文件实现. Mach-O 是一种文件格式，也是一种描述可执行文件如何被内核加载并运行的ABI (应用程序二进制接口)

Mach-O为Mach Object文件格式的缩写，它是一种用于可执行文件，目标代码，动态库，内核转储的文件格式。作为a.out格式的替代，Mach-O提供了更强的扩展性，并提升了符号表中信息的访问速度。
a.out是旧版类Unix系统中用于执行档、目的码和后来系统中的函数库的一种文件格式，这个名称的意思是汇编器输出。

尽管目前大多数类Unix系统都已改用ELF格式，不再采用a.out格式，但编译器和链接器依然会在用户未指定文件名时，将输出文件取名为“a.out”

ELF = Executable and Linkable  Format，可执行连接格式，是UNIX系统实验室（USL）作为应用程序二进制接口（Application Binary Interface，ABI）而开发和发布的，也是Linux的主要可执行文件格式。Executable and linking format(ELF)文件是x86 Linux系统下的一种常用目标文件(object file)格式，有三种主要类型:(1)适于连接的可重定位文件(relocatable file)，可与其它目标文件一起创建可执行文件和共享目标文件。(2)适于执行的可执行文件(executable file)，用于提供程序的进程映像，加载的内存执行。(3)共享目标文件(shared object file),连接器可将它与其它可重定位文件和共享目标文件连接成其它的目标文件，动态连接器又可将它与可执行文件和其它共享目标文件结合起来创建一个进程映像。

专业一点讲, 它告诉系统: 

* 使用哪个动态库加载器
* 加载哪个共享库.
* 如何组织进程地址空间.
* 函数入口点地址，等.

Mach-O 不是新事物. 最初由开放软件基金会 (OSF) 用于设计基于 Mach 微内核OSF/1 操作系统. 后来移植到 x86 系统OpenStep.

为了支持Dyld(Mac OS X的连接器) 运行时环境, 所有文件应该编译成Mach-O 可执行文件格式. 
Mach-O 文件分为三个区域, 头部、载入命令区Section和原始段数据.头部和载入命令区描述文件功能、布局和其他特性；原始段数据包含由载入命令引用的字节序列。为了研究和检查 Mach-O 文件的各部分, OS X 自带了一个很有用的程序otool，其位于/usr/bin目录下。

###ASLT地址空间布局随机化
这是一种针对缓冲区溢出的安全保护技术，通过对堆，栈，共享库映射等线性区布局的随机化，通过增加攻击者预测目的址的难度，防止攻击击者直接定位攻击代码位置，达到阻止溢出攻击的目的。proc/sys/kernel/randomize_va_space用于控制Linux下 内存地址随机化机制（address space layout
 randomization)
  有以下三种情况
   
   * 0 - 表示关闭进程地址空间随机化。
   *  1 - 表示将mmap的基址，stack和vdso页面随机化。
   *  2 - 表示在1的基础上增加栈（heap）的随机化。

###FAT ELF 
一个胖二进制（或多架构二进制）是一种已扩大可以在多个处理器类型上执行指令集的计算机可执行程序。执行通常的方法是包括一个版本的机器代码的每个指令集，前面代码的执行跳转到相应的节中的所有操作系统兼容。这将导致在一个文件比常规的单体系结构的二进制文件大。
利用胖的二进制结构的操作系统软件是不常见的;有几种方法来解决同样的问题，如使用一个安装程序，选择一个特定平台在安装时，以源代码形式分发软件，在操作的系统编译，或者使用虚拟机（如通过Java），并仅在时间编译。

###MetaData
用于描述要素、数据集或数据集系列的内容、覆盖范围、质量、管理方式、数据的所有者、数据的提供方式等有关的信息。 元数据被定义为：描述数据及其环境的数据元数据以非特定语言的方式描述在代码中定义的每一类型和成员。
存储以下信息：

* 程序集的说明。
* 标识（名称、版本、区域性、公钥）。
* 导出的类型。
* 该程序集所依赖的其他程序集。
* 运行所需的安全权限。
* 类型的说明。
* 名称、可见性、基类和实现的接口。
* 成员（方法、字段、属性、事件、嵌套的类型）。
* 属性。
* 修饰类型和成员的其他说明性元素。


##破解步骤
购买后，接着就是破解了。随着iOS设备cpu 的不同（arm 6 还是arm 7），mach－o文件格式的不同（thin binary 还是fat binary），应用是否对破解有防御措施（检测是否越狱，检测应用文件系统的变化），破解步骤也有所不同，但核心步骤如下：	

* 第一步：获得cryptid，cryptoffset，cryptsize（用otool)
cryptid为加密状态，0表示未加密，1表示解密；
cryptoffset未加密部分的偏移量，单位bytes
cryptsize加密段的大小，单位bytes
* 第二步：将cryptid修改为0
* 第三步：gdb导出解密部分
* 第四步：用第二步中的解密部分替换掉加密部分
* 第五步：签名   (ldone)
* 第六步：打包成IPA安装包
	
		
##代表工具
* Crackulous（GUI工具）（来自Hackulous）
crackulous最初版本由SaladFork编写，是基于DecryptApp shell脚本的，后来crackulous的源码泄露，SaladFork放弃维护，由Docmorelli接手，创建了基于Clutch工具的最近版本。
* Clutch（命令行工具）（来自Hackulous）
由dissident编写，Clutch从发布到现在，是最快的破解工具。Clutch工具支持绕过ASLR（apple在iOS 4.3中加入ASLR机制）保护和支持Fat Binaries，基于icefire的icecrack工具，objective－c编写。
* PoedCrackMod（命令行工具）（来自Hackulous）
由Rastignac编写，基于poedCrack，是第一个支持破解fat binaries的工具。shell编写
* CrackTM（命令行工具）（来自Hackulous）
由MadHouse编写，最后版本为3.1.2，据说初版在破解速度上就快过poedCrack。shell编写

##bash脚本工具的发展历史
虽然目前都已废弃，但都是目前好用的ipa 破解工具的基础。
autop(Flox)——>xCrack(SaladFork)——>DecryptApp(uncon)——>Decrypt(FloydianSlip)——>poedCrack（poedgirl）——>CrackTM(MadHouse)代表工具：CrackNShare （GUI工具）（来自appcake）基于PoedCrackMod 和 CrackTM
我们可以通过分析这些工具的行为，原理及产生的结果来启发防御的方法。像AppSync这种去掉设备签名检查的问题还是留给apple公司来解决（属于iOS系统层的安全），对于app开发则需要重点关注，app是如何被解密的（属于iOS应用层的安全）。

参考：http://www.freebuf.com/articles/wireless/6068.html

