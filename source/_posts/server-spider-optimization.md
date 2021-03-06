title: 抓取总结与优化
tags:
  - spider
  - optimization
  - server
  - python
categories:
  - server
date: 2015-02-22 17:14:37
keywords: spider crawler optimization 优化 网络爬虫 抓取
---

也算是写了比较久的爬虫了，一直没有时间总结一下，前不久写了一个比较通用的抓取框架，可以比较方便地配置抓取。今天写一下相关的一些简单的优化的点。

<!--more-->

网上常用的爬虫有以下几个类型
1. **批量型爬虫**：批量型爬虫有比较明确的抓取范围和目标，当爬虫达到这个设定的目标后，即体质抓取过程，之余具体目标可能各异，也许是设定抓取一定数量的网页即可，也许是设定抓取消耗时间等，不一而足！
2. **增量型爬虫**：增量型爬虫与批量型爬虫不同，会保持持续不断的抓取，对于抓取到的网页，要定期更新，因为互联网网页处于不断的变化中，新增网页，网页被删除或者网页内容更改等都非常常见，而增量型爬虫需要及时反映这种变化，所以处于不断的抓取过程中，不是在抓取新网页，通用的殇爷搜索引擎爬虫基本都属于此类！
3. **垂直型爬虫**：垂直型爬虫关注特定主题内容或者属于特定行业的网页，比如对于健康网站来说，只需要从互联网网页面里找到与健康相关的页面内容即可，其他行业的内容不再考虑范围！

上面是网上找来的分类，我对爬虫也作了自己的分类
1. **页面功能抓取：**这种爬主要是抓取一些功具页面，分析网页功能，实现用户动作的模拟以获得结果。之前抓过一个网络访问速度测试网站的页面，地址是[http://www.17ce.com/](http://www.17ce.com/)，用来做网络情况监控,分析它接口的参数，并实时获取结果。这种要实现相对容易，主要是接口分析。较难一些的可能要模拟js操作，pyhton的话，可以用phantomjs库或mechanize实现，由于模拟浏览器操作，速度很慢。
2. **抓取垂直资源：**像视频信息，应用市场，小说等等的抓取就属于这一类。抓取这些网站一般有两种情况。一种是资源可穷举，这类网站只是从首页往下解析就基本上就能抓到所有数据。另一种是资源不可穷举，视频，应用市场一般都不会用列表穷举自己的数据，不过他们都会提供搜索功能，搜索结果是有数量限制的，一般是几百个到一两千。抓取这类网站要一般通过枚举搜索词，爬取搜索接口。垂直数据对数据精度要求很高，一般要针对网站做内容提取的模板。
3. **大范围网页抓取：**这类爬虫我并没有写过，综合搜索引擎才会做这种大面积的抓取。这类爬虫由于抓取数据量非常大，要多机器存储，多机器抓取，更新和调度也很复杂，是最高级的爬虫。这种抓取对页面内容提取的精度要求不高，一般是获取关健字和描述，粗略地提取正文内容。

不同类型的爬虫瓶颈并不一样。垂直和功能页面由于抓取的网站比较固定，很多网站的反抓取的策略，要避免过频繁地访问。而大范围页面抓取由于页面非常多，一般同一时间抓取的页面比较分散，不会对被抓取网站造成太大的压力。大量的页面使得单机的计算能力，网络带宽，存储系统，调度策略等等，都可能成为瓶颈。

总结一下常用的一些优化方法。
###Mysql读写优化
我最常用的存储数据库是mysql， 写过很多爬虫都使用mysql存储， 最多时抓取过700+GB的压缩文本， 上亿条数据。 Mysql的优化主要集中在IO上。
1. 存储的单条较大的话，数据要压缩。压缩的计算成本并不高，而文本压缩对于减轻IO压力效果确很明显。
2. 分表，mysql单文件较小时出问题也比较好处理。较小的索引也可以降低增删的开销。
3. 如果数据要经常访问的话，做读写分离。
4. 条件允许的话，使用固态硬盘，固态较小的可以只将索引放到固态硬盘上。
5. 批量读写，检查数据是否在库中，写入数据等，批量执行可以减少数据库压力。

###DNS缓存
使用域名访问url的话每次都要通过DNS解析，会造成一定的时间损耗。做DNS缓存可以加快网络访问。如果抓取的网站是比较大的网站，它可能有多个服务器IP，可以使用socket.getaddrinfo同时获取多个IP地址，然后平均好每个IP地访问。这样可以增加站点的访问频率。

###提取
Pyhton解析html页面常用的方法有正则表达式，Beautifulsoup，xpath等。刚开始写爬虫时，用过Beautifulsoup,解析速度很，页面结构改变的话，提取也容易受影响。正则表达式和xpath速度都很快，不过正则也容易受页面结构变更的影响，xpath可以用模糊路径比较不会受结构变化的影响。这样看来xpath是比较好选择，实际使用中，一般是xpath与正则结合使用。

###更新策略
如果数据有优先级的话，可以对热门数据做定时的频繁检查。一般数据则要做检查记录，如检查多少次，有更新的次数是多少次，没有更新的多少，这样可通过这些记录决定下一次检查更新的间。这种方式可以保证更新频繁的数据检查频繁，更新不频繁的少检查避免资源浪费。
如果是静态页面的话，一般会有Last-Modified字段，使用这个字段请求如果没有更新可以加快访问速度，也省去了检查的消耗。

###关于js
工作中并没有遇到需要模拟JS处理的情况，自己学习过，一般是使用webkit库来模拟浏览器，再模拟用户操作触发JS。试过python的phantomjs库，速度很慢，就算不加载图片，一般页面都要十秒以上的时间处理，不适合用来抓取大范围的页面。

###使用代理
代理是比较好的避免反爬取的方法，不过代理服务器的选择是个难点。如果自己有服务器的话，在单机性能足够的情况下，给服务器搭代理服务，使用代理会比分布更方便。至于网上免费的代理服务器一般都不稳定，要定时检查代理服务器的情况，淘汰掉那些网络状况不好的服务器。


