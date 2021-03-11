---
title: eclipse项目使用idea打开后配置及问题解决记录
categories: idea eclipse
tags: idea eclipse
date: 2021-03-11 14:55:26
---

# 背景

项目: ssm架构 ant打包

tomcat 8

jdk 1.8

# 配置项

导入后需要配置 web.xml,资源包位置，依赖位置

# 项目配置图解

## project

这块主要是jdk配置及输出目录

## modules

项目结构标记如图： 

[![CY_2021-03-11-17-18-22.png](https://s3.ax1x.com/2021/03/11/6t44fA.png)](https://imgtu.com/i/6t44fA)


## libraries

选择依赖路径引入依赖

## facets

配置文件添加

[![CY_2021-03-11-17-25-07.png](https://s3.ax1x.com/2021/03/11/6tIuCj.png)](https://imgtu.com/i/6tIuCj)

web.xml路径添加

[![CY_2021-03-11-17-26-24.png](https://s3.ax1x.com/2021/03/11/6tId2R.png)](https://imgtu.com/i/6tId2R)

## artifacts

注意依赖添加后要勾选如图二 这边才会有

图一：

[![CY_2021-03-11-17-28-47.png](https://s3.ax1x.com/2021/03/11/6tI5sP.png)](https://imgtu.com/i/6tI5sP)

图二：

依赖的勾选及后边的属性决定了打包artifacts的依赖引用

[![CY_2021-03-11-17-31-25.png](https://s3.ax1x.com/2021/03/11/6toUw8.png)](https://imgtu.com/i/6toUw8)


## 最后

在tomcat里配置启动就ok了

# 问题解决

## 循环依赖问题解决

报错关键信息：

可能的根本原因包括-Xss的设置过低和非法的循环继承依赖项。正在处理的类层次结构是org.bouncycastle.asn1.ASN1EncodableVector

这里排查到包bcprov在低版本中是循环依赖了一个父类，把包替换成高级版本即刻解决问题。假如说你不方便把包替换掉，还可以跳过启动检查。

### 方法一： 替换高版本的bcprov的包 （待测试）

```html
<!-- https://mvnrepository.com/artifact/org.bouncycastle/bcprov-jdk15on -->
<dependency>
    <groupId>org.bouncycastle</groupId>
    <artifactId>bcprov-jdk15on</artifactId>
    <version>1.67</version>
</dependency>
```

### 方法二： 跳过检查bcprov的包 （我的选择）

在tomcat的catalina.properties找到tomcat.util.scan.StandardJarScanFilter.jarsToSkip属性。

增加bcprov开头的包。

举例

```text
tomcat.util.scan.StandardJarScanFilter.jarsToSkip=\
annotations-api.jar,\
ant-junit*.jar,\
ant-launcher.jar,\
ant.jar,\
...
xmlParserAPIs.jar,\
xom-*.jar,\
bcprov*.jar
```


