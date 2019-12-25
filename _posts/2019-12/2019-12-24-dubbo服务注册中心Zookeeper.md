---
title: dubbo服务注册中心Zookeeper
categories: Apache
tags: dubbo
date: 2019-12-24 21:05:15
---
通过前面的Dubbo架构图可以看到，Registry（服务注册中心）在其中起着至关重要的作用。Dubbo官方推荐使用Zookeeper作为服务注册中心。
## Zookeeper介绍
Zookeeper 是 Apache Hadoop 的子项目，是一个树型的目录服务，支持变更推送，适合作为 Dubbo 服务的注册中心，工业强度较高，可用于生产环境，并推荐使用 。
为了便于理解Zookeeper的树型目录服务，我们先来看一下我们电脑的文件系统(也是一个树型目录结构)：
![2019-12-24_21-59-47](https://s2.ax1x.com/2019/12/24/lP63v9.png)
我的电脑可以分为多个盘符（例如C、D、E等），每个盘符下可以创建多个目录，每个目录下面可以创建文件，也可以创建子目录，最终构成了一个树型结构。通过这种树型结构的目录，我们可以将文件分门别类的进行存放，方便我们后期查找。而且磁盘上的每个文件都有一个唯一的访问路径，例如：C:\Windows\itcast\hello.txt。

Zookeeper树型目录服务：
![lP62Uf.png](https://s2.ax1x.com/2019/12/24/lP62Uf.png)
流程说明：

1. 服务提供者(Provider)启动时: 向 /dubbo/com.foo.BarService/providers 目录下写入自己的 URL 地址

2. 服务消费者(Consumer)启动时: 订阅 /dubbo/com.foo.BarService/providers 目录下的提供者 URL 地址。并向 /dubbo/com.foo.BarService/consumers 目录下写入自己的 URL 地址

3. 监控中心(Monitor)启动时: 订阅 /dubbo/com.foo.BarService 目录下的所有提供者和消费者 URL 地址
## 安装Zookeeper
下载地址：http://archive.apache.org/dist/zookeeper/

安装步骤：
第一步：安装jdk

详见本博客：  Centos6.7安装jdk1.8.0_171 [https://copasters.github.io/posts/Centos6.7%E5%AE%89%E8%A3%85jdk1.8.0_171]

第二步：第二步：把 zookeeper 的压缩包（zookeeper-3.4.6.tar.gz）上传到 linux 系统
第三步：解压缩压缩包 tar -zxvf zookeeper-3.4.6.tar.gz -C /usr/local/src
改名：mv zookeeper-3.4.6/ zookerper
第四步：进入zookeeper目录，创建data目录 mkdir data
第五步：进入conf目录 ，把zoo_sample.cfg 改名为zoo.cfg 
cd conf 
mv zoo_sample.cfg zoo.cfg
其中zoo.cfg为正式的配置文件
第六步：打开zoo.cfg文件, 修改data属性：dataDir=/usr/local/src/zookeeper/data

## 启动、停止Zookeeper
进入Zookeeper的bin目录，启动服务命令 ./zkServer.sh start
停止服务命令./zkServer.sh stop
查看服务状态：./zkServer.sh status
![2019-12-25_22-04-19](https://s2.ax1x.com/2019/12/25/lFOYZt.png)


