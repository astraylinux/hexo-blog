title: centos libc.6.so 链接出错修复
tags:
  - server
  - Linux
  - system
categories:
  - Linux
date: 2015-04-18 14:56:37
keywords: Linux server system 链接 出错 修复 崩溃 centos

-----

服务器用的是centos 5.8，出现过几次libc.6.so的基础库在用yum安装软件时候链接被替换了，然后所有系统命令都不能使用了。提示如下图：
![](http://res.astraylinux.com/system/libc_so_6_error.png)

Linux许多命令都是依赖这个C语言的动态链接库，在centos里，这是一个软链，被替换后，被替换到新版本的动态库时，会出现这个问题。我这里是从版本2.5被换成了2.12(截图时已修复)。
![](http://res.astraylinux.com/system/libc_so_6_error2.png)

<!--more-->

##原因
从google搜索到这样的答案：
>The loader on your system does not support the new Linux ABI. Until relatively recently, Linux ELF binaries used the System V ABI. Recently, in support of STT_GNU_IFUNC, the Linux ABI was added. You would have to update your system C library to have a loader that support STT_GNU_IFUNC, and then it will also recognize ELF objects with the Linux ABI type.

也就是说系统版本的动态库加载器太旧了，不支持新版本的动态库，要嘛升级系统的加载器，要嘛恢复到旧的库。

##用LD_PRELOAD修复
出现问题后所有几乎所有命令都不能使用，原因就是没办法加载那个新版本的动态库，而`LD_PRELOAD`环境变量则可以在运行命令前指定加载的动态库。
>LD_PRELOAD 是这样的一个环境变量：它可以影响程序的运行时的链接（Runtime linker），它允许你定义在程序运行前优先加载的动态链接库。该功能主要就是用来有选择性的载入不同动态链接库中的相同函数。通过这个环境变量，我们 可以在主程序和其动态链接库的中间加载别的动态链接库，甚至覆盖正常的函数库。

执行如下命令，直接就可以恢复了。不过执行命令的必需是有root权限的用，sudo命令还是用不了的（可能还要加载其他动态库）。
```bash
root# LD_PRELOAD=/lib64/libc-2.5.so ln -sf /lib64/libc-2.5.so /lib64/libc.6.so
```

##用sln命令修复
应该很多人都不知道这个命令，这个命令在/sbin/目录下，man的描述是这样的。
>sln  symbolically  links  `dest`  to  `source`.   It  is statically linked, needing no
dynamic linking at all.  Thus sln is useful to  make  symbolic  links  to  dynamic
libraries if the dynamic linking system for some reason is nonfunctional.

看起来就是为了修复动态库的问题而做的，它是一个静态链接生成的命令，不依赖任何的动态库，使用用方法跟ln是一样的。
```bash
root# /sbin/sln -sf /lib64/libc-2.5.so /lib64/libc.6.so
```

## 关于loader
老外对这个问题的回答有一句是这样的：
>Ah, I didn't realize you had replaced the system C library without also replacing the loader, or I would have been more specific in my advice. 

也就是，要升级libc.6.so要同时升级动态库的loader。看了一下我系统上的ld动态库版本也是2.5，可能要升级libc.6.so就要将ld升级到同样的版本。这边系统恢复了，不敢再去动，没有再做尝试，以后遇到再补充。

