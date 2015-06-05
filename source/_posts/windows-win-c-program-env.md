title: 编写C语言开发环境——编译模块
date: 2012-07-13 10:51:00
categories:
- Windows
tags:
- C
- makefile
- gcc
- Dos
---

在winodws下C语言的编译器，我并不熟悉，winTC下是有一些用于编译C语言的程序，真心不熟，也不知道会遇到什么问题。所以搜了一下windows下的Gcc，真的有这个移植，这下好办了。windows下的GCC及G++被集成在mingw32中，连Gdb都有。下载解压出来，就是linux编译工具的win32版本。

还有一个问题是自动生成makefile，在Linux下有autoconf及automake这两个工具能实现，搜了一上午也没找到win32版本的。只好放弃了makefile，直接用gcc的编译，将所有的.c文件全编译成.o文件，然后集中到一个debug文件夹中，进行链接。这个开发环境是面向初学者的，没有makefile应该不会有太大的问题。
<!--more-->

##三种调用方式
在VC中调用dos命令调用Gcc来编译，找到了三种调用方式，分别是


```cpp
  System(LPCTSTR cmd);
```

```cpp
UINT WINAPI WinExec(
        __in LPCSTR lpCmdLine,
        __in UINT uCmdShow
);
```

以及

```cpp
HINSTANCE ShellExecute(      
    HWND hwnd,
    LPCTSTR lpOperation,
    LPCTSTR lpFile,
    LPCTSTR lpParameters,
    LPCTSTR lpDirectory,
    INT nShowCmd
);

```

##区别
* system（）在调用的时候会闪过一个dos窗口，不能用。
* winExec不能使用操作符，像重定向，管道等。不过可以通过第二个参数将窗口隐藏。
* shellExecute也能隐藏窗口，并且功能强大。一开始我想直接用shellExecute调用Gcc进行编译，却也不能使用操作符。后来调用cmd，其他命令用作参数，完全可以运行dos下的所有命令。
 
```bash
Build_cmd=gcc+option+BUILD_OBJ+File_str+" 2>>"+dir+"[\\stderr](file://\\stderr)";
block_ShellExecute(NULL,"open","cmd","/c "+Build_cmd,project_dir,SW_HIDE);
```

##备注
不过要注意，那些需要输入的命令不可用，会进无限等待。另外，我还遇到了另一个问题，shellExecute并不是阻塞运行，因此可能第一步的.o文件还没完成生成，就运行了链接导致出错。一开始我用判断.o文件是否完全生成来解决，这样做还得先判断是否编译出错，并不是一个好办法。后来找到了一个更好的解决办法，通过判断进程是否退出来实现阻塞，代码如下：

```cpp
void block_ShellExecute(HWND hwnd,LPCTSTR lpOperation,LPCTSTR lpFile,LPCTSTR lpParameters,LPCTSTR lpDirectory,INT nShowCmd)
{
	SHELLEXECUTEINFO si;
	ZeroMemory(&si, sizeof(si));
	si.cbSize = sizeof(si);
	si.fMask = SEE_MASK_NOCLOSEPROCESS;
	si.hwnd=hwnd;
	si.lpVerb = _T(lpOperation);
	si.lpFile = _T(lpFile);
	si.lpParameters=_T(lpParameters);
	si.lpDirectory=_T(lpDirectory);
	si.nShow = nShowCmd;
	
	ShellExecuteEx(&si);
	
	DWORD dwExitCode;
	GetExitCodeProcess(si.hProcess,&dwExitCode);
	while (dwExitCode == STILL_ACTIVE)
	{
		Sleep((DWORD)5);
		GetExitCodeProcess(si.hProcess, &dwExitCode);
	}
	
	CloseHandle(si.hProcess);
}
```
接下来要试着做调试模块，现在还没有什么想法，头疼。
