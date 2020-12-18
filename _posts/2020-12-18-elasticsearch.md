---
layout: post
title: "windows安装Elasticsearch"
subtitle: ""
date: 2020-12-18 16:42:22 +0800
author: "YangYongKang"
header-style: text
catalog: true
archive:
    - windows安装Elasticsearch 
tags:
    - windows安装Elasticsearch
---

### 一丶下载
* [官网下载](https://www.elastic.co/cn/downloads/past-releases#elasticsearch){:target="_blank"}   

### 二丶安装
* ElasticSearch 5.x 往后依赖于JDK 1.8以上版本,所以安装前请确认自己的java环境,请安装jdk1.8及以上版本
* 下载下来ES以后解压以后,主要文件夹为bin,config,log,jdk,plugins
  * config 配置目录
  * logs 日志目录
  * plugins 插件目录
  * data 数据存储目录(ES启动后会自动创建)
  
### 三丶运行
* 运行bin/elasticsearch.bat 来启动,
![](/img/es1.png)
* 启动成功的标志
![](/img/es2.png)
* 访问[http://localhost:9200](http://localhost:9200){:target="_blank"}  
![](/img/es3.png)   
### 四丶安装ElasticSearch-head插件
![](/img/es4.png)
### 五丶ElasticSearch安装为Windows服务
* 在ES/bin目录下找到elasticsearch-service.bat,运行elasticsearch-service.bat install
### 六丶集群安装
* 安装前一定要先删除已启动的ES/data目录,如果不删除会导致节点信息不匹配,无法形成集群.
* 将elasticsearch-7.9.3复制一份并更改config文件夹下elasticsearch.yml节点名称,一个为node1,一个为node2,并且端口不可冲突,具体配置如下
![](/img/es5.png)
![](/img/es6.png)
* 将两个项目依次启动,打开刚才安装的ElasticSearch-head插件即可显示启动的服务
![](/img/es7.png)









