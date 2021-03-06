title: 编写HTTP接口测试
tags:
  - server
  - HTTP
  - test
  - python
  - php
  - search
categories:
  - server
date: 2014-05-03 11:29:44
keywords: HTTP test 接口 测试 服务器 搜索

-----

整理了一段时间的搜索接口，新接口终于上线了。现在搜索的php接口分为线上和测试两个部分，有改动先在测试接口上进行修改，确定没有问题再复制到线上去。  
现在遇到一个问题是，公司没有测试人员给我们测试搜索接口，只能自己测试。要自己测试接口可是个麻烦事，程序员总是很难发现自己程序中的bug。所以我们考虑编写程序对接口进行测试。
我们将测试内容分成两种，一种是可预见结果的测试，比如效果评估、非法变动等，另一种则为不可预见的结果的测试。

<!-- more -->
##实现两个接口的访问
线上接口与测试接口在同一台机器上，线上接口有DNS映射，可以直接访问，测试接口则需要配置host。不过还有更好的办法，就是通过IP访问接口，用HTTP头指定host，这样速度更快，不用改host。用python实现
```python
url = "http://127.0.0.1/api.php"

#线上
result_online = get(url, {"host":"host_online"})

#测试
result_test = get(url, {"host":"host_test"})
```
这里的get是我自己的封装，第一个参数是url，第二个是HTTP头，具体实现网上很多了。

##获取测试url
测试接口如果要一个个把参数列出来，不但麻烦，也很容易遗漏，最好是能直接取到线上用户访问的url。这个可以通过分析HTTP服务程序的日志得到，我的是nginx，通过awk可以很容易地得到各个接口的实际访问url，用来模拟用户的访问。

##可预见结果测试
可预见结果字面上看就是能预判修改的效果，通过接口测试去验证效果是否正确。

这一点，我主要是做了非法变动的检测。比如搜索接口，我改了一个不影响排序的属性值，那它的结果顺序就应该是一样的，如果顺序有变，就可以认为是接口有bug。

这种检测只是具体接口具体实现，主要是要提取出变与不变的部分，对不变的部分进行对比来快速检测问题。在这些检测中，还可以附上对接口的响应速度做对比。

##不可预见结果测试
结果不可预见，那要怎么测试呢？一开始这可难到我了，都没想过的问题，我怎么用程序来测试它。经过与同事的探讨，结论还是一样：行不通，还是要人工测试。

虽然没办法通过程序直接测试出结果，可是我们可以用程序来降低人工测试的成本。

我们想到的比较基本的方法就是通过做diff来快速定位变化。通过访问一定量的接口，将接口结果分别输出到文件，再使用vimdiff或其他diff程序进行对比，这样就可以很快地找到变化的点，然后就是分析是否有问题了。

##思考
程序员写一些测试程序其实并不难，不要认为写测试程序很浪费时间，测试程序写好，后面可以快速地测试，反而可以节省更多时间。

这次的工作让我看到了程序测试的重要性，看来我得看看单元测试了:D







