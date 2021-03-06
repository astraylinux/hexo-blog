title: 搜索提示词简记
tags:
  - search
  - php
  - sphinx
  - coreseek
  - python
  - Linux
  - server
categories:
  - search
date: 2014-06-15 13:29:44
keywords: search coreseek server linux 搜索 提示词

-----

搜索提示词是指用户在搜索框上输入出，弹出的扩展词，是对用户目标搜索词的猜测。在动手实现之前，在网上找了一些相关的资料，大多是一些理论算法的实现，比如[搜索智能提示suggestion](http://blog.csdn.net/v_july_v/article/details/11288807)。

在实际工作中，这些算法只是基础，要现实能满足各种变化的需求，还需要做很多工作。现在有些开源搜索引擎也有提供提示词的功能。而我这里记录是通过额外搭建一个搜索实现的，通过搭建搜索，可以更灵活地应对需求。

我做的是安卓的应用搜索提示词，下面是最终的效果图:
![](http://res.astraylinux.com/server/app_search_tip.jpg)

<!--more-->

##提示词来源
要搭建搜索，首先当然要有数据，对于应用搜索，数据一般有两个来源，一个是应用软件的信息（对提示词就是应用名了），另一个则是用户的搜索词。由于提示词每次给出的数量有限，因此尽量以取热度靠前的应用，另外有些名字过长的也要对其进行裁切。搜索词也是类似，只取搜索量靠前的词。

##拼音与简写
用搜索搭建对于加拼音或其他扩展的词都是比较方便的，只要在建索引的时候，把拼音加到索引中就可以了。这个实现也很容易，使用拼音库将中文转成拼音，在建索引的时候包括进去就可以了。甚至还可以加入繁体字。
![](http://res.astraylinux.com/server/app_search_tip4.png)

由于提取拼音会去掉英文的部份，所以包括后面使用前缀匹配搜索的提示词都不一定是以“浏览器"开头的。

![](http://res.astraylinux.com/server/app_search_tip3.jpg)

##完全匹配的应用
完全匹配的应用是指出现在第一个，可以直接点击下载的那个应用，也就是被认为最可能是用户想要的那个应用。

这个功能，最开始我是直接取第一个搜索结果做为完全匹配的，不过后面发现很多问题。当用户搜索一些相对模糊的词时，一般不会有完全匹配的应用，比如游戏、拍照、电影、下载。还有就是单纯靠前缀匹配，第一个匹配到不一定会是好的应用，例如搜索‘浏览器’，用户最想要的可能是‘uc浏览器’，但通过前缀匹配，uc浏览器已经被淘汰了。

![](http://res.astraylinux.com/server/app_search_tip2.jpg)

后面怎么解决这个问题呢，我是通过将完全匹配与下面的提示词分开搜索实现的。搜索完全匹配应用时，就不使用前缀匹配的搜索，并且做一些限制策略，对一些模糊词去掉完全匹配的功能。由于只使用热门应用与热门搜索词，提示词并没有负载压力，两次搜索不会有性能上的瓶颈。

##排序
排序的权重是根据几个因子按一定的规则算出来的。首先是query与提示词的相关度，然后是应用的下载量，还有搜索量。有的词可能只有搜索量，没有下载量，我会它们一个已入库的所有应用的平均下载量做为默认值。除了这几个已有的属性，还加上了一个人为干预的权重。就是这几个因子，通过各种加减乘除取对数等计算得到一个权重值，用来排序。

##总结
这个方法实现搜索提示，相对比较简单，也方便调整。而且就性能来说，用搜索引擎来实现响应速度也是比较快的。

