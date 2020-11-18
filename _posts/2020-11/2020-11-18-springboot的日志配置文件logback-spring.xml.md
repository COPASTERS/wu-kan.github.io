---
title: springboot的日志配置文件logback-spring.xml
categories: springboot logback
tags:  springboot logback
date: 2020-11-18 15:47:43
---

## 背景

    有时候调试需要看下日志信息，去哪个文件看，如何把自己需要的日志打印到制定的问件，方便自己快速查看。达到这样的目的就需要对日志的输出打印的配置文件有所了解。自己花了一点时间，理论加实践的操作了一波，算是有了一定的认识。日志框架是logback

## 前提
   
    springboot项目logback日志的依赖是本身就有的，所以不需要自己手动在引入一遍，否则可能会出现包冲突的问题。位置如图：

![CY_2020-11-18-15-57-41.png](https://imgchr.com/i/DmyEjK)

## 配置文件的理解

### 正文

Spring Boot在所有内部日志中使用Commons Logging，但是默认配置也提供了对常用日志的支持，如：Java Util Logging，Log4J, Log4J2和Logback。每种Logger都可以通过配置使用控制台或者文件输出日志内容。

### 默认日志Logback
SLF4J——Simple Logging Facade For Java，它是一个针对于各类Java日志框架的统一Facade抽象。Java日志框架众多——常用的有java.util.logging, log4j, logback，commons-logging, Spring框架使用的是Jakarta Commons Logging API (JCL)。而SLF4J定义了统一的日志抽象接口，而真正的日志实现则是在运行时决定的——它提供了各类日志框架的binding。

Logback是log4j框架的作者开发的新一代日志框架，它效率更高、能够适应诸多的运行环境，同时天然支持SLF4J。

***默认情况下***，Spring Boot会用Logback来记录日志，并用INFO级别输出到控制台。在运行应用程序和其他例子时，你应该已经看到很多INFO级别的日志了。

### 默认配置属性支持

Spring Boot为我们提供了很多默认的日志配置，所以，只要将spring-boot-starter-logging作为依赖加入到当前应用的classpath，则“开箱即用”。

下面介绍几种在application.properties就可以配置的日志相关属性。

控制台输出

***日志级别从低到高分为TRACE < DEBUG < INFO < WARN < ERROR < FATAL，如果设置为WARN，则低于WARN的信息都不会输出。***

Spring Boot中默认配置ERROR、WARN和INFO级别的日志输出到控制台。您还可以通过启动您的应用程序--debug标志来启用“调试”模式（开发的时候推荐开启）,以下两种方式皆可：

在运行命令后加入--debug标志，如：$ java -jar springTest.jar --debug

在application.properties中配置debug=true，该属性置为true的时候，核心Logger（包含嵌入式容器、hibernate、spring）会输出更多内容，但是你自己应用的日志并不会输出为DEBUG级别。

### 文件输出

默认情况下，Spring Boot将日志输出到控制台，不会写到日志文件。如果要编写除控制台输出之外的日志文件，则需在application.properties中设置logging.file或logging.path属性。

logging.file，设置文件，可以是绝对路径，也可以是相对路径。如：logging.file=my.log

logging.path，设置目录，会在该目录下创建spring.log文件，并写入日志内容，如：logging.path=/var/log

如果只配置 logging.file，会在项目的当前路径下生成一个 xxx.log 日志文件。

如果只配置 logging.path，在 /var/log文件夹生成一个日志文件为 spring.log

注：二者不能同时使用，如若同时使用，则只有logging.file生效

默认情况下，日志文件的大小达到10MB时会切分一次，产生新的日志文件，默认级别为：ERROR、WARN、INFO

### 级别控制

所有支持的日志记录系统都可以在Spring环境中设置记录级别（例如在application.properties中）

格式为：'logging.level.* = LEVEL'

- logging.level：日志级别控制前缀，*为包名或Logger名

- LEVEL：选项TRACE, DEBUG, INFO, WARN, ERROR, FATAL, OFF

举例：

- logging.level.com.dudu=DEBUG：com.dudu包下所有class以DEBUG级别输出

- logging.level.root=WARN：root日志以WARN级别输出