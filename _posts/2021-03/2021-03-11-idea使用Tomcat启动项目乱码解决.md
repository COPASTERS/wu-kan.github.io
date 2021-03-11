---
title: idea使用Tomcat启动项目乱码解决
categories: idea Tomcat
tags: idea Tomcat
date: 2021-03-11 14:14:12
---

# 背景

idea集成tomcat后启动项目，红色的日志乱码（也就是tomcat内部的日志乱码）

# 解决

问题重现

[![1692019-20191113145325229-1384157693.png](https://s3.ax1x.com/2021/03/11/6tu8Tx.png)](https://imgtu.com/i/6tu8Tx)


今天很疑惑这个问题，于是去网上找了答案，结果是需要修改Tomcat根目录下面的"logging.properties"文件，把所有的encoding=UTF-8的改成encodng=GBK，保存之后，重启Tomcat服务器，就能解决乱码问题，下面贴出我解决步骤的截图:

1.先找到Tomcat根目录

[![1692019-20191113145622894-942842040.png](https://s3.ax1x.com/2021/03/11/6tuyAP.png)](https://imgtu.com/i/6tuyAP)

2.右击用记事本打开或者Notapad++打开.Ctrl+F点击“替换”。替换之后ctrl+s进行及时保存。

[![1692019-20191113145701223-527007844.png](https://s3.ax1x.com/2021/03/11/6tK9N6.png)](https://imgtu.com/i/6tK9N6)

3.保存完后，重启一下Tomcat服务器，救能看到中文乱码的问题给解决了。

[![1692019-20191113145808465-1201204976.png](https://s3.ax1x.com/2021/03/11/6tKsKJ.png)](https://imgtu.com/i/6tKsKJ)