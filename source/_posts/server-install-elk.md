title: ELK安装配置
tags:
  - search
  - elasticsearch
  - elk
  - logstash
  - kibana
  - Linux
  - server
categories:
  - search
date: 2017-04-19 13:29:44
keywords: search elk server linux 搜索 提示词 elasticsearch

-----

## 安装
安装很简单，从官网下载解压即可使用
地址: [https://www.elastic.co/downloads](https://www.elastic.co/downloads)
主要安装 Elasticsearch、Kibana、Logstash
安装x-pack
```bash
#在elasticsearch目录下运行
./bin/elasticsearch-plugin install x-park
#在Kibana目录下运行
./bin/kibana-plugin install x-pack
```
<!--more-->

修改config/elasticsearch.yml里的配置，将network.host改为0.0.0.0，否则不能远程访问
运行elasticsearch时遇到两个问题
```bash
#1. 可开启的设备文件数太少，少于65536，这个要修改文件/etc/security/limits.conf，修改或加入(数字自己定，大一些）:
* soft nofile 655350
* hard nofile 655350

#2. vm.max_map_count太小，修改/etc/sysctl.conf加入
vm.max_map_count=655350

#做完以上步骤重启
#用命令查看可开启文件数
ulimit -a
```

修改config/kibana.yml里的配置，将server.host改为0.0.0.0，否则不能远程访问


