title: linux内核0.12 －－第三章　内核编程语言和环境
date: 2012-02-28 13:44:00
categories:
- Linux
tags:
- Linux
- assembly
- core
---

本章主要是as86与gas两种汇编语言的简要介绍，C语言与汇编语言的相互嵌套，目标文件的结构与及makefile文件的简要语法。
<!--more-->

１,as86汇编的简要语法及命令可以参考
[http://blog.csdn.net/astraylinux/article/details/7301596](http://blog.csdn.net/astraylinux/article/details/7301596)
２,gas汇编与intel汇编的主要区别
（具体语法参考：[http://blog.csdn.net/astraylinux/article/details/7301620](http://blog.csdn.net/astraylinux/article/details/7301620)）　　　

* 寄存器名前缀%
* 操作数是源在前，目的在后。与Intel语法刚好相反。
* 操作数长度在指令名后缀，b表示8位，w表示16位，l表示32位，如movl %ebx,%eax。
* 立即操作数（常量）用$标示，如addl $5,%eax
* 变量加不加$有区别。如movl $foo, %eax表示把foo变量地址放入寄存器%eax。movl foo,%eax表示把foo变量值放入寄存器%eax。

| Intel Code | AT&T Code |
| ----------- | ---------- |
| mov eax,1 | movl $1,%eax | 
| mov ebx,0ffh | movl $0xff,%ebx | 
| int 80h | int $0x80 | 
| mov ebx, eax | movl %eax, %ebx |
| mov eax,[ecx] | movl (%ecx),%eax | 
| mov eax,[ebx+3] | movl 3(%ebx),%eax | 
| mov eax,[ebx+20h] | movl 0x20(%ebx),%eax | 
| add eax,[ebx+ecx*2h] | addl (%ebx,%ecx,0x2),%eax | 
| lea eax,[ebx+ecx] | leal (%ebx,%ecx),%eax | 
| sub eax,[ebx+ecx*4h-20h] | subl -0x20(%ebx,%ecx,0x4),%eax | 

３，两个目标文件连接示意图（注意子区与不可重定义区absolute）
![](http://hi.csdn.net/attachment/201202/28/0_133040716166x8.gif)

４，Ｃ语言程序的编译和链接过程
![](http://hi.csdn.net/attachment/201202/28/0_1330407345czD4.gif)　　　　

５，在Ｃ语言中嵌入汇编的语法格式
![](http://hi.csdn.net/attachment/201202/28/0_13304074190Q8q.gif)

６，Ｃ语言调用的堆栈结构，栈内控制权转移，AT&T的栈组织方式与intel汇编应该是一样的，主要是cpu指令所决定的。
![](http://hi.csdn.net/attachment/201202/28/0_1330407654M5Tt.gif)　　　

７，在Ｃ语言中调用汇编的函数主要是实现方法是汇编，参数获取是从栈中根据esp偏移来取得，而调用函数的Ｃ这些语言代码与调用Ｃ语言的函数形式上是一样的。

８，目标文件格式及链接操作：
![](http://hi.csdn.net/attachment/201202/28/0_13304079808xm8.gif)![](http://hi.csdn.net/attachment/201202/28/0_1330407986s7C2.gif)　　　　
![](http://hi.csdn.net/attachment/201202/28/0_1330407992bHp9.gif)　　
