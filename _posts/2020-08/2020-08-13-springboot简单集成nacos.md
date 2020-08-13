---
title: springboot简单集成nacos
categories: springboot nacos
tags:  springboot nacos 中间件
date: 2020-08-13 17:11:53
---

## 背景

之前看到公司项目里有用到配置中心，看了下感觉这个nacos挺不错，就去官网看了哈，搞了个小demo简单集成一下

## 使用

具体还是参照官网慢慢看，阿里的项目文档写的还是可以的。

我自己的尝试的代码已经上传：具体实现看代码吧！由于是简单的集成干扰项很少，很容易就看懂我就不说具体的了。

[代码地址请点击](https://gitee.com/copasters/configProjectLearn "托管于gitee")

## 思考

由于好长时间没有搞这个远程的配置中心了，之前接触还是Spring Cloud Config这个中间件，nacos也是阿里的产品。我开始的要求就是把本地的配置配到远端上去，nacos有自己的分组，每个配置又有自己的id。对于服务比较多的项目也是很容易区分，配置也还好。但是有一点就是动态的切换生产和开发环境这个还不知道具体是怎么实现的。麻烦一点的就是搞多个nacos,生产直接启动本地的生产配置文件去nacos拿数据就ok了，只是觉得有点怪。对于一些使用@NacosValue注解注入的值是可以实时更新，但是应用端口，数据库连接之类的要自己重启项目才会生效。也可以理解，比较这些属性是项目启动就缓存进去的。至于具体的还是参考官网吧! [nacos官方文档](https://nacos.io/zh-cn/docs/what-is-nacos.html "nacos官方文档")