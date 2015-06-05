title: adb源码学习(adb code study)
date: 2012-12-27 16:37:00
categories:
- Android
tags:
- code
- USB
---

初到公司的上手项目是一用MFC写Android软件安装器（从PC端通用usb连接），这个程序最大问题在于PC与手机建立连接的过程。Android手机连接PC是通过google提供的ADB套件连接的。这个连接是需要对不同手机安装针对的ADB驱动程序的(windows就是麻烦)。在这之前，还是先研究一下ADB的源码吧^_^。
<!--more-->

##程序基本结构
ADB-server : 运行在PC端，是一个始终在后台运行的进程，作为与手机端交互的唯一接口。ADB-server处理ADB-client的请求，一部分请求无须与设备交互，直接在PC本地完成；剩下的请求需要与设备端的adbd交互，ADB-server起到了一个switcher的作用。  
ADB-client : 运行在PC端，可以同时存在多个。每个ADB-client由用户启动，完成多种功能。其作用是与ADB-server交互，实现用户请求的功能。 
adbd : 运行在设备端的常驻进程，同时只存在一个。作用是接收PC端的ADB-server发来的请求，并作出对应操作。

##通信结构
![](https://app.yinxiang.com/shard/s3/res/352145bd-54da-4580-b1cc-9bc11a93d3f3.png?resizeSmall&width=721)


##代码浅晰

这三个可执行程序都是同一套代码编译出来的，位于<Android Source Dir>/system/core/adb/
在adb源码中找到了

```c
#define DEFAULT_ADB_PORT 5037
#define DEFAULT_ADB_LOCAL_TRANSPORT_PORT 5555
#define ADB_CLASS 0xff
#define ADB_SUBCLASS 0x42
#define ADB_PROTOCOL 0x1
```
其中，class,subclass,protocol对应的是“USB\\Class_FF&SubClass_42&Prot_01”中的FF，42和01，果然代码里有大部分的信息 

service用了一个结构体asocket保存了用于连接的两个socket，这个socket对一个是本地的，一个是设备上的，也就是service接收到本地socket的数据会将收到的数据发向这个结构中对应的设备的socket。

atransport，用于传输数据的对象，可以设置成不同类型，而分别与本地或usb设备进行数据传输。本地用的是socket 文件描述符，usb设备用的是usb连接的句柄。

alistener是一个结构，只保存了一些数据,当socket监听收到连接请求时用于绑定本地端口和请求端口。

usb通信的部分是通过adb_api，就是那两个dll里的代码，它们用到了DeviceIoControl来与设备通信

fdevent 用一个结构体，保存了事件及事件处理的函数的指针。

adb 启动时会先检查是否有设置相关的环境变量，有ANDROID_PRODUCT_OUT，ANDROID_SERIAL，ANDROID_ADB_SERVER_PORT，主要是port，如果没有port将会使用5037这个端口。然后adb客户端会尝试发送版本检查的信息，如果返回-2，说明服务端没有启动，就获取自身所在的路径，以服务端的参数启动一个服务进程，这个进程与原adb是同一个程序。


