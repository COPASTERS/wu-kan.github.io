---
title: Elasticsearch(上)
categories: 搜索
tags: elasticsearch
date: 2019-08-03 07:59:01
background_music: '<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=100% height=52 
src="//music.163.com/outchain/player?type=2&id=2740360&auto=0&height=32"></iframe>'
---

# 1.内容简介
   - ElasticSearch的作用
   - 安装ElasticSearch服务
   - ElasticSearch的相关概念
   - 使用java客户端操作ElasticSearch(略,有百度云代码)
   - 分词器的作用
   - 使用ElasticSearch集成IK分词器

# 2.ElasticSearch简介

## 2.1 什么是ElasticSearch

Elaticsearch，简称为es， es是一个开源的高扩展的分布式全文检索引擎，它可以近乎实时的存储、检索数据；本身扩展性很好，可以扩展到上百台服务器，处理PB级别的数据。es也使用Java开发并使用Lucene作为其核心来实现所有索引和搜索的功能，但是它的目的是通过简单的RESTful API来隐藏Lucene的复杂性，从而让全文搜索变得简单。

## 2.2. ElasticSearch的使用案例

2013年初，GitHub抛弃了Solr，采取ElasticSearch 来做PB级的搜索。 “GitHub使用ElasticSearch搜索20TB的数据，包括13亿文件和1300亿行代码”
维基百科：启动以elasticsearch为基础的核心搜索架构
SoundCloud：“SoundCloud使用ElasticSearch为1.8亿用户提供即时而精准的音乐搜索服务”
百度：百度目前广泛使用ElasticSearch作为文本数据分析，采集百度所有服务器上的各类指标数据及用户自定义数据，通过对各种数据进行多维分析展示，辅助定位分析实例异常或业务层面异常。目前覆盖百度内部20多个业务线（包括casio、云分析、网盟、预测、文库、直达号、钱包、风控等），单集群最大100台机器，200个ES节点，每天导入30TB+数据
新浪使用ES 分析处理32亿条实时日志
阿里使用ES 构建挖财自己的日志采集和分析体系

## 2.3. ElasticSearch对比Solr

Solr 利用 Zookeeper 进行分布式管理，而 Elasticsearch 自身带有分布式协调管理功能;
Solr 支持更多格式的数据，而 Elasticsearch 仅支持json文件格式；
Solr 官方提供的功能更多，而 Elasticsearch 本身更注重于核心功能，高级功能多有第三方插件提供；
Solr 在传统的搜索应用中表现好于 Elasticsearch，但在处理实时搜索应用时效率明显低于 Elasticsearch

#  3.ElasticSearch安装与启动

## 3.1. 下载ES压缩包

ElasticSearch分为Linux和Window版本，基于我们主要学习的是ElasticSearch的Java客户端的使用，所以我们课程中使用的是安装较为简便的Window版本，项目上线后，公司的运维人员会安装Linux版的ES供我们连接使用。

[ElasticSearch的官方地址]{https://www.elastic.co/products/elasticsearch}

## 3.2. 安装ES服务

Window版的ElasticSearch的安装很简单，类似Window版的Tomcat，解压开即安装完毕，解压后的ElasticSearch的目录结构如下：

![2019-08-03-08-11-43.png](http://pv125k1gl.bkt.clouddn.com/2019-08-03-08-11-43.png)

## 3.3. 启动ES服务

点击es安装目录的/bin目录下的 **elasticsearch.bat**

**注意**：9300是tcp通讯端口，集群间和TCPClient都执行该端口，9200是http协议的RESTful接口 。
通过浏览器访问ElasticSearch服务器，看到如下返回的json信息，代表服务启动成功：

![2019-08-03-08-16-01.png](http://pv125k1gl.bkt.clouddn.com/2019-08-03-08-16-01.png)

**注意事项一**：ElasticSearch是使用java开发的，且本版本的es需要的jdk版本要是1.8以上，所以安装ElasticSearch之前保证JDK1.8+安装完毕，并正确的配置好JDK环境变量，否则启动ElasticSearch失败。
**注意事项二**：出现闪退，通过路径访问发现“空间不足”

![2019-08-03-08-17-25.png](http://pv125k1gl.bkt.clouddn.com/2019-08-03-08-17-25.png)

【解决方案】

![2019-08-03-08-18-57.png](http://pv125k1gl.bkt.clouddn.com/2019-08-03-08-18-57.png)

修改jvm.options文件的22行23行，把2改成1，让Elasticsearch启动的时候占用1个G的内存。

![2019-08-03-08-20-19.png](http://pv125k1gl.bkt.clouddn.com/2019-08-03-08-20-19.png)

## 3.4. 安装ES的图形化界面插件es-header(推荐使用kibana)

ElasticSearch不同于Solr自带图形化界面，我们可以通过安装ElasticSearch的head插件，完成图形化界面的效果，完成索引数据的查看。安装插件的方式有两种，在线安装和本地安装。本文档采用本地安装方式进行head插件的安装。elasticsearch-5-*以上版本安装head需要安装node和grunt

1)1）下载head插件：https://github.com/mobz/elasticsearch-head

2）将elasticsearch-head-master压缩包解压到任意目录（我这里是D:\software\elasticsearch-5.6.8），但是要和elasticsearch的安装目录区别开

3）进入elasticsearch-head-master压缩包执行命令安装

      npm install

4）下载nodejs：https://nodejs.org/en/download/ 

一路next安装

安装完毕，可以通过cmd控制台输入：node -v 查看版本号

![2019-08-03-08-25-27.png](http://pv125k1gl.bkt.clouddn.com/2019-08-03-08-25-27.png)

5）将grunt安装为全局命令 ，Grunt是基于Node.js的项目构建工具；是基于javaScript上的一个很强大的前端自动化工具，基于NodeJs用于自动化构建、测试、生成文档的项目管理工具。
在cmd控制台中输入如下执行命令：

      npm install -g grunt-cli

-g表示全局（globle）变量，让grunt-cli的客户端使用全局安装
执行结果如下图：
   效果如下：

![2019-08-03-08-27-09.png](http://pv125k1gl.bkt.clouddn.com/2019-08-03-08-27-09.png)

ps:如果安装不成功或者安装速度慢，可以使用淘宝的镜像进行安装：
       npm install -g cnpm –registry=https://registry.npm.taobao.org

后续使用的时候，只需要把npm xxx   换成  cnpm xxx 即可

4）修改elasticsearch/config下的配置文件：elasticsearch.yml，增加以下三句命令：

      http.cors.enabled: true
      http.cors.allow-origin: "*"
      network.host: 127.0.0.1

http.cors.enabled: true：此步为允许elasticsearch跨越访问，默认是false。
http.cors.allow-origin: "*"：表示跨域访问允许的域名地址。

**重启**

5）进入head目录启动head，在命令提示符下输入命令：

       grunt server

![2019-08-03-08-30-44.png](http://pv125k1gl.bkt.clouddn.com/2019-08-03-08-30-44.png)

根据提示访问，效果如下：

![2019-08-03-08-34-39.png](http://pv125k1gl.bkt.clouddn.com/2019-08-03-08-34-39.png)

**PS**：如果第5步失败，执行以下命令

      npm install grunt

再次启动grunt server

![2019-08-03-08-36-24.png](http://pv125k1gl.bkt.clouddn.com/2019-08-03-08-36-24.png)

再根据提示按以下方式依次安装组件

![2019-08-03-08-37-24.png](http://pv125k1gl.bkt.clouddn.com/2019-08-03-08-37-24.png)

(根据电脑可能有不同的错误)

# 4.	ElasticSearch相关概念(术语)

## 4.1. 概述

Elasticsearch是面向文档(document oriented)的，这意味着它可以存储整个对象或文档(document)。然而它不仅仅是存储，还会索引(index)每个文档的内容使之可以被搜索。在Elasticsearch中，你可以对文档（而非成行成列的数据）进行索引、搜索、排序、过滤。Elasticsearch比传统关系型数据库如下：

Relational DB -> Databases -> Tables -> Rows -> Columns
Elasticsearch -> Indices   -> Types  -> Documents -> Fields

以下表示2个文档（document），3个字段（field）

![2019-08-03-08-44-39.png](http://pv125k1gl.bkt.clouddn.com/2019-08-03-08-44-39.png)

## 4.2. Elasticsearch核心概念

   1) 索引 index

一个索引就是一个拥有几分相似特征的文档的集合。比如说，你可以有一个客户数据的索引，另一个产品目录的索引，还有一个订单数据的索引。一个索引由一个名字来标识（**必须全部是小写字母的**），并且当我们要对对应于这个索引中的文档进行索引、搜索、更新和删除的时候，都要使用到这个名字。在一个集群中，可以定义任意多的索引。   

   2) 类型 type

在一个索引中，你可以定义一种或多种类型。一个类型是你的索引的一个逻辑上的分类/分区，其语义完全由你来定。通常，会为具有一组共同字段的文档定义一个类型。比如说，我们假设你运营一个博客平台并且将你所有的数据存储到一个索引中。在这个索引中，你可以为用户数据定义一个类型，为博客数据定义另一个类型，当然，也可以为评论数据定义另一个类型。

   3) 字段 Field

相当于是数据表的字段，对文档数据根据不同属性进行的分类标识

   4) 映射 mapping

mapping是处理数据的方式和规则方面做一些限制，如某个字段的数据类型、默认值、分析器、是否被索引等等，这些都是映射里面可以设置的，其它就是处理es里面数据的一些使用规则设置也叫做映射，按着最优规则处理数据对性能提高很大，因此才需要建立映射，并且需要思考如何建立映射才能对性能更好。

   5) 文档 document

一个文档是一个可被索引的基础信息单元。比如，你可以拥有某一个客户的文档，某一个产品的一个文档，当然，也可以拥有某个订单的一个文档。文档以JSON（Javascript Object Notation）格式来表示，而JSON是一个到处存在的互联网数据交互格式。

在一个index/type里面，你可以存储任意多的文档。注意，尽管一个文档，物理上存在于一个索引之中，文档必须被索引/赋予一个索引的type。

    6) 接近实时 NRT

Elasticsearch是一个接近实时的搜索平台。这意味着，从索引一个文档直到这个文档能够被搜索到有一个轻微的延迟（通常是1秒以内）

    7) 集群 cluster

 一个集群就是由一个或多个节点组织在一起，它们共同持有整个的数据，并一起提供索引和搜索功能。一个集群由一个唯一的名字标识，这个名字默认就是“elasticsearch”。这个名字是重要的，因为一个节点只能通过指定某个集群的名字，来加入这个集群

     8) 节点 node 

一个节点是集群中的一个服务器，作为集群的一部分，它存储数据，参与集群的索引和搜索功能。和集群类似，一个节点也是由一个名字来标识的，默认情况下，这个名字是一个随机的漫威漫画角色的名字，这个名字会在启动的时候赋予节点。这个名字对于管理工作来说挺重要的，因为在这个管理过程中，你会去确定网络中的哪些服务器对应于Elasticsearch集群中的哪些节点。
一个节点可以通过配置集群名称的方式来加入一个指定的集群。默认情况下，每个节点都会被安排加入到一个叫做“elasticsearch”的集群中，这意味着，如果你在你的网络中启动了若干个节点，并假定它们能够相互发现彼此，它们将会自动地形成并加入到一个叫做“elasticsearch”的集群中。
在一个集群里，只要你想，可以拥有任意多个节点。而且，如果当前你的网络中没有运行任何Elasticsearch节点，这时启动一个节点，会默认创建并加入一个叫做“elasticsearch”的集群。

     9) 分片和复制 shards&replicas

一个索引可以存储超出单个结点硬件限制的大量数据。比如，一个具有10亿文档的索引占据1TB的磁盘空间，而任一节点都没有这样大的磁盘空间；或者单个节点处理搜索请求，响应太慢。为了解决这个问题，Elasticsearch提供了将索引划分成多份的能力，这些份就叫做分片。当你创建一个索引的时候，你可以指定你想要的分片的数量。每个分片本身也是一个功能完善并且独立的“索引”，这个“索引”可以被放置到集群中的任何节点上。分片很重要，主要有两方面的原因： 1）允许你水平分割/扩展你的内容容量。 2）允许你在分片（潜在地，位于多个节点上）之上进行分布式的、并行的操作，进而提高性能/吞吐量。
至于一个分片怎样分布，它的文档怎样聚合回搜索请求，是完全由Elasticsearch管理的，对于作为用户的你来说，这些都是透明的。
在一个网络/云的环境里，失败随时都可能发生，在某个分片/节点不知怎么的就处于离线状态，或者由于任何原因消失了，这种情况下，有一个故障转移机制是非常有用并且是强烈推荐的。为此目的，Elasticsearch允许你创建分片的一份或多份拷贝，这些拷贝叫做复制分片，或者直接叫复制。


复制之所以重要，有两个主要原因： 在分片/节点失败的情况下，提供了高可用性。因为这个原因，注意到复制分片从不与原/主要（original/primary）分片置于同一节点上是非常重要的。扩展你的搜索量/吞吐量，因为搜索可以在所有的复制上并行运行。总之，每个索引可以被分成多个分片。一个索引也可以被复制0次（意思是没有复制）或多次。一旦复制了，每个索引就有了主分片（作为复制源的原来的分片）和复制分片（主分片的拷贝）之别。分片和复制的数量可以在索引创建的时候指定。在索引创建之后，你可以在任何时候动态地改变复制的数量，但是你事后不能改变分片的数量。
默认情况下，Elasticsearch中的每个索引被分片5个主分片和1个复制，这意味着，如果你的集群中至少有两个节点，你的索引将会有5个主分片和另外5个复制分片（1个完全拷贝），这样的话每个索引总共就有10个分片。


# 5. ElasticSearch操作入门 

略.........

使用java客户端操作ElasticSearch(百度云代码)

链接：https://pan.baidu.com/s/1475XMp5WAg3VGRExA6dRYQ 
提取码：aup4 

# 6. IK 分词器和ElasticSearch集成使用

## 6.1. 查询存在问题分析

中文的词项搜不到

原因: 分词的原因

##  6.2. IK分词器简介

IKAnalyzer是一个开源的，基于java语言开发的轻量级的中文分词工具包

## 6.3. ElasticSearch集成IK分词器

   1).IK分词器的安装(放到plugins目录下)
     1. 解压elasticsearch-analysis-ik-2.x后,将文件夹拷贝到elasticsearch-5.6.8\plugins下，并重命名文件夹为ik
     2. 重新启动ElasticSearch，即可加载IK分词器


   2).IK分词器测试

   IK提供了两个分词算法ik_smart 和 ik_max_word

   (1）最小切分：在浏览器地址栏输入地址
http://127.0.0.1:9200/_analyze?analyzer=ik_smart&pretty=true&text=我是程序员

![2019-08-03-09-09-30.png](http://pv125k1gl.bkt.clouddn.com/2019-08-03-09-09-30.png)

   (2）最细切分：在浏览器地址栏输入地址
http://127.0.0.1:9200/_analyze?analyzer=ik_max_word&pretty=true&text=我是程序员

![2019-08-03-09-10-39.png](http://pv125k1gl.bkt.clouddn.com/2019-08-03-09-10-39.png)

在ElasticSearch没有索引的情况下，插入文档，默认创建索引和索引映射是无法使用IK分词器的。

















