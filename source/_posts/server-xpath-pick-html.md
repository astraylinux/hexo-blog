title: xpath提取HTML
tags:
  - spider
  - server
  - Linux
  - python
categories:
  - server
date: 2014-08-21 20:24:52
keywords: xpath Linux python server 提取 HTML

-----

之前用python写HTML页面的提取一般是用BeautifulSoup，一直都觉得其速度慢，而且提取结果容易受页面变化的影响。这两天写提取的时候，看到了xpath这个工具。单纯从提取速度来看，xpath与正则表达式是相近的，几乎有BeautifulSoup十倍以上的速度。XPath与BeautifulSoup的对比可以参考这篇文章[《Python网页解析：BeautifulSoup vs lxml.html》](http://www.cnblogs.com/rzhang/archive/2011/12/29/python-html-parsing.html)

<!--more-->

##简介

>XPath是XML路径语言（XML Path Language），它是一种用来确定XML文档中某部分位置的语言。XPath基于XML的树状结构，提供在数据结构树中找寻节点的能力。起初XPath的提出的初衷是将其作为一个通用的、介于XPointer与XSL间的语法模型。但是XPath很快的被开发者采用来当作小型查询语言。HTML是基于XML的标记语言，所以可以用XPath来解析和提取HTML页面。

XPath要先将HTML解析成DOM树，然后通过路径表达式定位节点。在XPath中，有七种类型的节点：元素、属性、文本、命名空间、处理指令、注释以及文档节点（或称为根节点），参考下面的例子。

```html
<!doctype html>
<html>  #文档节点(根节点)
    <head>
        <meta charset="UTF-8"/>  #charset:属性节点
        <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
        <meta name="port" content="width=device-width, initial-scale=1"/>
        <title lang="eng" width="320"> Linux </title> #title:元素节点，"Linux":文本节点
    </head>
</html>
```

##语法
XPath 使用路径表达式在 XML 文档中选取节点。节点是通过沿着路径或者 step 来选取的。下表是XPath最常用的语法，实例对应的是上面的html代码。

|表达式|描述|实例|结果|
|------|----|----|----|
|nodename|	选取此节点的所有子节点。|meta|选取 meta 元素的所有子节点。|
|/| 从根节点选取。|/html/head|从根节点开始选取|
|//| 匹配节点，不考虑位置。|/html//title|匹配所有html下的的title，不管位置|
|.| 选取当前节点。|./title|匹配当前节点下的title|
|..| 选取当前节点的父节点。|../title|匹配当前节点的父节点下的title|
|@| 选取属性。|//meta[@name="port"]|匹配带有name属性为"port"的节点|
|||//title/@lang|获取title的lang属性的值|
|*|节点通配符，匹配任何节点|//head/*|选取head节点下的所有子节点|
|@*|属性通配符，匹配任意属性|//title/@*|选取title节点的所有属性|

谓语用来查找某个特定的节点或者包含某个指定的值的节点。谓语嵌在方括号中。

|路径表达式|结果|
|----------|----|
|//head/meta[1]| 选取属于head子节点的第一个meta节点。
|//head/meta[last()]| 选取属于head子节点的最后一个meta节点。
|//head/meta[last()-1]|	选取属于head子节点的倒数第二个met节点。
|//head/meta[position()<3]|	选取最前面的两个属于head节点的meta子节点。
|//title[@lang]| 选取所有拥有名为 lang 的属性的 title节点。
|//title[@lang='eng']| 选取所有 title节点，且具有值为"eng"的lang属性。
|//title[width>300]/text()| 选取title节点的文本，且其width元素的值必须大于300。

谓语表达式里用到了last()、position()、text()是XPath的函数，XPath含有超过100个内建函数，用于处理字符串、数值、日期时间比较、节点及Qname处理、序列处理、逻辑值等。具体的函数列表可以从[w3school](http://www.w3school.com.cn/xpath/xpath_functions.asp)获取。

##在python中使用
在python中使用需要安装相应的库，有几种支持XPath的python库，我用是lxml库（XPath本就是为了解析XML而做的）。在Linux上安装python-lxml（可用apt或yum安装），然后就可以在python中调用了。

代码示例：

```python
#!/usr/bin/env python
# encoding: utf-8

from lxml import etree


html = """<!doctype html>
<html class="theme-next use-motion theme-next-mist">
<head>
	<meta charset="UTF-8"/>
	<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
	<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1"/>
	<meta name="keywords" content="C,Python,Php,程序猿" />
	<link rel="alternate" href="/atom.xml" title="Astray Linux" type="application/atom+xml" />
	<title> Astray Linux </title>
</head>
</html>"""

#appoint which type of parser will use
parser = etree.HTMLParser()

doc = etree.fromstring(html, parser)

print "Title:", doc.xpath("/html/head/title/text()")[0]
print "Link href:", doc.xpath("//link/@href")[0]
print "meta list:", doc.xpath("//meta")
print "meta attr:", doc.xpath('//meta[@name="keywords"]/@content')[0]
print "meta attr:", doc.xpath("//meta[4]/@content")[0]
print "meta attr:", doc.xpath("//meta[1]/@charset")[0]

head = doc.xpath("/html/head")[0]

#'head' is a etree document too.
print "\nhead element:", head
print "head meta:", head.xpath("./meta[4]/@content")[0]

```

运行结果：

    Title:  Astray Linux
    Link href: /atom.xml
    meta list: [<Element meta at 0x7f3edc201a28>, <Element meta at 0x7f3edc201830>, <Element meta at 0x7f3edc201908>, <Element meta at 0x7f3edc201b00>]
    meta attr: C,Python,Php,程序猿
    meta attr: C,Python,Php,程序猿
    meta attr: UTF-8

    head element: <Element head at 0x7f3edc201a28>
    head meta: C,Python,Php,程序猿

##PS：封装
由于XPath可以很方便地通过配置提取HTML，这样我们可以考虑将页面获取等其他操作写成一个通用的程序，然后根据配置文件来进行提取，将提取做成一个通用的模块。由于XPath本身的功能还不能满足所有的提取，封装的时候，要加上一些功能函数以对提取结果做再处理。具体可以看我整理的python一些常用的代码[pylib](https://github.com/astraylinux/pylib)里的expath模块。
