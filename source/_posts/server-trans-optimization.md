title: 数据导入流程优化
tags:
  - server
  - optimization
  - HTTP
categories:
  - server
date: 2015-06-17 14:35:43
keywords: HTTP, POST, python, des, redis, 服务端, 客户端
---

这两天在优化我们一套数据导入流程, 记录下优化过程中的问题和解决办法.
这套导入流程结构是这样的
流程说明:
1. 首先用户通过提供的接口,将数据处理,加密,base64编码,urlencode,最后向服务端Post数据.
2. 服务端使用PHP接收用户数据,解密.
3. PHP将json数据插入指定的redis队列.
4. 最后单项目数据导入程序,将redis队列数据取出并导入mongodb.
<!--more-->

![](http://res.astraylinux.com/server/http_post_optimization.png)
之所以加redis和单项目导入程序这一层, 是因为导入程序这一块有较多的业务处理. 并且只导redis队列速度较快,用户可以快速返回.
在实际使用中, 逐渐发现了一些性能问题.

## Python Des加密性能
最早出现的问题是python的des加密性能, 使用python接口导入的时候, 一秒钟能只能一百多条数据, 总共也就不到100k的数据. 这个性能是完全没法接受的, 100w数据要3个小时导入. 
经过下点计时, 发现是pydes的加密过程消耗掉了大部分时间. 通过网上的资料发现, pydes是纯python实现的des加密, 大家都知道python在纯计算方面性能是很差的. 后面用pycrypto替换掉了pydes, 这是一个用C语言实现的加密库, 性能比python实现的要高一两个数量级(真不想吐槽python的性能).
```python
#加密代码
from Crypto.Cipher import DES
def encrypt(instr, skey):
    d = DES.new(skey ,DES.MODE_ECB)
    for i in range(0, 8-len(instr)%8):
        instr = instr + " " 
    afterdes = d.encrypt(instr)
    afterbase64 = base64.b64encode(afterdes)
    return afterbase64
```
换用pycrypto之后, 导入性能升到1000+/s的性能.

## PHP服务端导入redis队列的优化
PHP端每次收到的是用户传上来的批量数据,处理方式是先将数据解密, 并json_decode成结构化数据, 然后将每条数据导入redis队列. 在这里, 一条一条导入redis是比较耗时的操作, redis并没有提供批量导入的功能, 而每条数据导入都要通过请求再传输, 就比较耗时.
这里的优化方法也比较简单直接, 不将数据转化成结构, 而是直接将json串存到redis中, 这样有几个好处.

1. 省去了json_decode
2. 不用遍历数据
3. 不用多次向redis发送数据, 一次性发送
4. 后续导入mongo的程序也可以一次性取数据, 不用多次请求redis

用这个方法优化后, 原本500条400+kb数据响应时间为150ms左右, 优化后降到了60ms左右.
如此用python的整体导入性能达到2000-3000条/s.

## 数据压缩优化
数据没有压缩也是比较大的问题, 这种纯文本的数据压缩率是很高, 性能可以提升非常多. 对数据压缩主要用是zlib, 即gzip格式的数据. 数据压缩对其他计算也有好处.

1. 压缩速度快, 并不会造成太大的性能损耗
2. 数据变小, 网络带宽需求变小, 对网络差的用户提升明显
3. 压缩后数据再加密, 数据变小了, 加密速度快
4. base64编码及urlencode速度也有提高
5. PHP服务端数据处理也有同样的好处

用zlib压缩有压缩等级的选项, 1级和9级压缩率差别在30%左右, 压缩时间9级比一级要高出近数十倍, 这让我直接选择1级压缩, 1级压缩率也已经很理想了, 压缩后数据只有原始数据的10%左右.
压缩优化性能提升非常明显, 服务端从之前的60ms左右的响应时间直接降到了10ms左右. 客户端接口速度也有较大提示. 至此整体导入性能达到了每秒1w条左右. 在测试数据中, 这些数据压缩后在1MB-2MB之间, 也是大部分用户的带宽极限了.

## POST数据再优化
这是我的一个想法, 优化过程中发现urlencode相当耗时, 我就在想, 能不能避开这个过程, 直接向服务端发送二进制数据, 像文件那样. 这里主要是HTTP对POST的处理方式的问题. 用python做了一些尝试, 用自己的方式拼接数据, 并指定Content-Type等方式都没有成功. 后来找到了一个叫"poster"的库, 并没有我想像的那样避开encode, 它用了自己实现的encode方式, 我没有详细的看它的实现, 不过性能对比usllib的效率要高一些.
```python
    data = {"data":pushdata}
    urlrequest, headers = poster.multipart_encode(data)
    req = urllib2.Request(url, urlrequest, headers)
    response = urllib2.urlopen(req)
    ret=response.read()
```
从代码中可以看, 它只是encode方式不一样, 其他还是用urllib2的请求.
这个优化方式后, 性能再提升10%左右. 影响并不大.

## 总结
这么些优化下来, 提升最大的就是数据压缩, 压缩算法效率很高, 并不会带来新的损耗问题. 而变小的数据, 在加密, 转码上效率都能提高非常多. 另外一个提高较多的就是redis导入, 这种网络请求多次小数据请求要比少次大数据请求要慢.
结论就是, 数据能压缩就压缩, 传输能批量就批量.

数据导到mongodb的过程, 又是一场优化大战.
