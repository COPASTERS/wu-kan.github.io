---
title: springboot使用mybatis自动生成代码插件
categories: springboot mybatis-generator-maven-plugin mybatis
tags:  springboot mybatis mybatis-generator-maven-plugin
date: 2020-11-06 09:31:34
---

## 背景

   公司使用了这个插件，虽然早就听说了但一直没有使用过。刚好遇到就看了下具体的配置。

## 方法思路

   引入相关依赖，添加配置文件。具体的我就不贴出来。可以下载代码自己看配置

## 注意

    - pom.xml里的代码生成插件有两种配置方式。第一种需要指定我们的代码生成的配置文件位置.第二种不需要

    - generatorConfig.xml配置文件需要指定***数据库连接的本地驱动位置***,包生成规则等配置。详细配置说明在代码里可以看到。

## demo代码位置

  **请点击** [gitee代码库](https://gitee.com/copasters/mybatis-auto-code-generator/tree/master)

