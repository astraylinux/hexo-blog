title: coreseek 性能测试（sphinx）
date: 2014-05-04 09:55:00
categories:
- search
tags:
- search
- test
- performance
---

对coreseek进行性能测试，看看单机各个索引量的搜索性能。从结果上看，数据到了五千万就要很长的生成时间了，而且容易出错，需要做一些设置。到了千万级别时，只索引数据，最好不要存属性字段。
<!--more-->

##测试环境
测试数据：姓名与地址数据，姓名与地址设field
索引方式：通过python源读取文件生成索引
api接口： php
测试说明： 用python调用url访问php进行搜索（串行，间隔0.05s），php接口记录下搜索用时，搜索词是网上随便找的人名与地址，offset统一为0,limit为12匹配模式默认为SPH_MATCH_ALL，搜索结果为空时进行SPH_MATCH_ANY搜索
测试机器：CPU Intel(R) Xeon(R) E5405  @ 8核 2.00GHz ，8GB内存，centos 5.6   64位系统


##测试结果
![](http://img.blog.csdn.net/20140504094948406?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQXN0cmF5TGludXg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  

本来要测一个五千万的，不过文件实在太大，出了一些错误，生成时间太长了，就先不测了。  
      
做这个测试是因为在做视频站（[http://www.yingcui123.cn/](http://www.yingcui123.cn/)）的时候搜索慢了，虽然视频资源量还不到两千万，但是服务器不如测试的机器好。经过测试，发现，搜索服务平时虽然不怎么占用内存，但是突发内存占用还是比较大的。视频站由于服务器内存过小，导致搜索速度慢，加大内存后就正常了。


