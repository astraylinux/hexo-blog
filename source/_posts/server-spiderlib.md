#Python库,抓取框架
title: 简单的Python抓取框架
tags:
  - spider
  - server
  - lib
categories:
  - server
date: 2015-03-25 23:51:12
keywords: spider, lib, python, linux, 抓取，爬虫
---

写了比较久的爬虫了，前段时间将一些比较常用的python抓取模块整理成库, 通用配置控制抓取。代码并不多，但对于快速布一些简单的爬虫还是挺方便的。([git地址](https://github.com/astraylinux/spiderlib))

下图是一整体的一个流程：
![](http://res.astraylinux.com/spider/spiderlib_process.png)
框架是以mysql为存储系统建立的， 主要模块有**dispatcher(任务调度模块)**、**crawler(抓取模块)**、**picker(提取模块)**、**updater(更新模块，暂无)**。另外还有辅助的网络模块**net**，数据库模块**sqld**, 辅助工作模块**tools**。 抓取跟提取的配置是按域名分配，每个域名有一个配置文件，配置着页面的信息，url识别正则，提取xpath配置等，主配置文件是**/test/config.py**，是一些模块的基本配置。而域名配置在**/test/etc/webset**里，这个webset被当成一个python包，增加的页面配置要添加到**__init__.py**中。

spiderlib里的模块都只是代码库，并不能直接执行，可执行程序在test里，可以直接复制出来修改。

下面详细地记录下模块功能。

<!--more-->
##Dispatcher(任务调度)
任务调度模块的工作是从数据库中把需要爬取的网页信息取出来，放到任务队列中，以供后续的程序领取执行。刚开始写爬虫的时候，一般是执行任务的模块直接到库里取数据。这种做法不利于管理，并且很难实现分布式。而使用队列，则由一个程序统一下发任务，调度有问题，也只需要从一个模块找问题。队列还可以很方便地实现分布式，抓取机器到同一台机器领取任务，执行行完后再写回数据，非常方便。后面的sqld模块，也是使用redis队列统一写数据库。
相关配置
```bash
#spider interval config
G_DEFAULT_INTERVAL = 3600*6
G_MIN_INTERVAL	= 3600
G_MAX_INTERVAL = 3600*24*7
G_RISE_INTERVAL = 3600*9

#max dispatch number(new url or update url)
G_MAX_SELECTNUM_NEW = 2000
G_MAX_SELECTNUM_UP = 2000
G_MAX_SELECTNUM_PICK = 2000

#dispatch gap(seconds)
G_DISPATCH_GAP = 10

```

##Crawler(抓取模块)
抓取模块的任务就是从网络上Get页面，并将发现的新链接写回数据库。这套框架里并没有限制抓取模块不能解析页面，而是做了简单的解析，提取链接。而页面要不要保留则是通过配置决定。这里抓取的页面分为**过程页面**和**内容页面**， 过程页面只是用来发现新链接，并不用于后续的信息提取，而内容页面则会用于提取。页面的分辨是通对URL的正则匹配实现的。
主要配置
```bash
#config about database
#division could be 1, 16, 256, the 16 use md5 last char
#256 use the md5 last two char
G_MAINDB = "test"
G_TABLE_LINK = {"name":G_PROJECT_FLAG + "_link", "division":1}

#max run threads of spider and picker
G_MAX_SPIDER_THREAD = 3

############### spider control ################
#if I need save html to database
#if you didn't save the html,you have to download
#the html again where you pick content
G_IFSAVE_HTML = True
#if G_IFSAVE_HTML=True and this is ture
#will save the process html(not detail page)
G_IFSAVE_PASS = False
#if this not True, crawler will not look for new link's from detail page
G_INTO_DETAIL = False
```

##Picker(提取模块)
提取模块用到我之前整理的python常用代码包[**pylib**](https://github.com/astraylinux/pylib)里的提取模块**expath.py**。这是对lxml.etree里的xpath提取部分的简单封装，以实现不更改代码，直接用配置实现不同的提取。Picker模块会试着从html库里取页面数据，如果没有的话，则直接从网上Get页面。根据配置将内容提取出来，并按key value的形式存入数据库。Key是数据库的字段名，因此数据库info库要根据提取内容增加字段。
主要配置
```bash
#max run threads of spider and picker
G_MAX_SPIDER_THREAD = 4

############### site config ####################
G_SITE = etc.webset.SITES
G_SITE_COMMON = etc.common

```

##Updater(更新模块)
待更新。

##Net(网络模块)
Crawler和Picker用到的网络相关的代码都是调用这个模块的。这个模块调用pylib的网络模块增强了get函数。增加了基本的压力控制，从配置来，每个域名，每次Get暂停多少秒。最重要的是实现了DNS缓存，并且保存多个IP地址，执行轮循访问。很多比较大的网站一个域名会有多个IP以实现负载分散，这样通过访问不同的IP避免反抓取，增加抓取线程，提高抓取速度。

##Sqld(Mysql写入模块)
实现这个模块，一来是为了降低Mysql的访问次频率，通过队列，将一些插入指令批量执行。二来是更好地实现分布式，抓取机器将写入数据直接放到队列，可以快速返回。

##Tools(辅助功能模块)
包含了快速建数据库表的函数，新增抓取链接到数据库的函数，quick start.

##域名提取配置示例
```bash
CONFIG = {
	#线程数
	"spider_thread_num": 1,
	#只抓站内
	"only_insite": 1,
	#抓取深度
	"max_depth": 100,
	#站点压力控制(秒)
	"spider_gap": 1,
	#站点的默认编码
	"default_code": "gbk",
	#默认DNS ip，手动添加的dns配置
	"default_dns": [
		"180.149.131.104",
		"180.149.133.165",
		"180.149.131.245",
		"220.181.57.233",
		"123.125.65.91",
		"123.125.115.90",
	],
	#禁止的后辍
	"forbidden_suf": ["rar", "pdf", "jpg", "png", "zip",\
			"doc", "xls", "css", "js", "php"],
	#禁止包含的字符串
	"filter_list": ["javascript", "&lm="],
	#必需包含其中任一个字符串
	"include_list": ["/search?", "/question/"],
	#必需包含其中所有字符串
	"include_must": [],
	#要这些符号后面的都截掉
	"cut_list": ["#"],
	#动态页保留的参数
	#例{"http://www.soku.com/v":["curpage","keyword","limit_date","orderby"]}
	"params_keep": {
		"^http://zhidao.baidu.com/question/[\d]*?\.html": [],
		"^http://zhidao.baidu.com/search": ["word", "pn"],
	},

	#页面匹配正则
	"page_regex": {
		"detail":r"^http://zhidao.baidu.com/question/[\d]*?\.html",
		#"index":"^http://www.23us.com/html/[\d]*/[\d]*/$",
		#"content":"^http://www.23us.com/html/[\d]*/[\d]*/[\d]*.html$"
	},

	"download": {
		"down_table": {"name":"info", "division":1},
		"down_dir": "./data",
		#"default_suf": "jpg",
	},

	"picker":{
		common.G_PAGETYPE["detail"]["type"]:{
			#提取后入的表
			#"table":{"name":"test_info", "division":1},
			#必须要有结果的字段
			"must_key":["title", "description"],
			"path":{
				"title":{"key":"""/html/head/title/text()"""},
				"keywords":{"key":"""/html/head/meta[@name="keywords"]/@content"""},
				"description":{"key":"""/html/head/meta[@name="description"]/@content"""},
				"goodnum":{"key":"""/html//div[@class="bd answer"]//span[3]/@data-evaluate"""},
				"badnum":{"key":"""/html//div[@class="bd answer"]//span[4]/@data-evaluate"""},
				"answernum":{"key":"""/html//div[@class="hd line other-hd"]//h2/text()""",
					"remake":[{"method":"re", "argv":["[\\d]+?"]}],
				},
				"createtime":{"key":"""/html//span[@class="grid-r ask-time"]/text()"""},
				"inserttime":{"key":"""now()"""},
			}
		},
		common.G_PAGETYPE["index"]["type"]: {},
		common.G_PAGETYPE["content"]["type"]: {},
	}
}
```


