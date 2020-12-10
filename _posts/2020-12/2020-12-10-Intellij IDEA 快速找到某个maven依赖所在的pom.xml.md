---
title: IntellijIDEA快速找到某个maven依赖所在的pom.xml
categories: IntellijIDEA maven
tags:  IntellijIDEA maven
date: 2020-12-10 19:07:40
---

## 背景

有时候我们使用别人的代码，经常会出现缺少包。但是在对方的项目里直接找到这个包在pom.xml中的位置很大概率是看不到的，存在嵌套依赖。

## 解决方案

1. 确定要查找jar的 artifactId

[![2020-12-10-1917](https://s3.ax1x.com/2020/12/10/rFam8K.png)](https://imgchr.com/i/rFam8K)

2. 打开项目的maven依赖图

[![2020-12-10-1920](https://s3.ax1x.com/2020/12/10/rFdPRf.png)](https://imgchr.com/i/rFdPRf)

3. 在依赖图上搜索上面找到的artifactId,这里是android-json

[![2020-12-10-1922](https://s3.ax1x.com/2020/12/10/rFdmon.png)](https://imgchr.com/i/rFdmon)

4. 在图上找到相应的元素后，鼠标双击便能进入jar的pom.xml

[![2020-12-10-1925](https://s3.ax1x.com/2020/12/10/rFdJeJ.png)](https://imgchr.com/i/rFdJeJ)

**注意**

这个pom.xml可能并不是我们项目自己的pom.xml，有可能是依赖内部的pom.xml。也就是说我们引用的其他依赖依赖了这个依赖。找到是谁依赖了这个依赖很简单，把鼠标放在这个文件上会显示这个文件在maven仓库的位置。看到这个你也就知道你项目引入了哪个依赖后连带的引入了这个依赖。

举例如图（随便找了一个其他的例子，E盘后面就是仓库的路径，即依赖的pom.xml文件位置，很明显是引入emoji-java依赖，顺带引入了其他的依赖）：

[![CY_2020-12-10-19-33-09](https://s3.ax1x.com/2020/12/10/rFwNng.png)](https://imgchr.com/i/rFwNng)



