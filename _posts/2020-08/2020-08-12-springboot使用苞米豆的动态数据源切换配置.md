---
title: springboot使用苞米豆实现动态数据源的切换
categories: springboot 动态数据源
tags:  springboot 动态数据源
date: 2020-08-12 11:16:20
---

## 背景

在公司里需要做一个报表业务，但是数据不在我们自己的数据库里，需要去第三方数据库查数据。但是目前我们本项目已经连接了数据库，自身业务也需要使用。这样就需要动态的切换数据库进行数据查询。自己摸索着实现了，但是是在项目已有的基础上改的，改完还有些不是太明白。所以干脆自己搞个小demo搞清楚一点。

## 实现方案

之前查了资料，有的人实现是通过自己封装springboot里的数据源来切换配置.对我来说可以一试，但是造轮子有点难度啊。看了项目里使用的苞米豆也就是mybatis-plus团队搞个一个中间件。用起来蛮舒服的。

我自己的尝试的代码已经上传：具体实现看代码吧！由于是简单的集成干扰项很少，很容易就看懂我就不说具体的了。

代码地址：https://gitee.com/copasters/configProjectLearn

## 注意点

在集成德鲁伊连接池的时候，注意一定要排除德鲁伊自身的数据源自动配置
```java
@SpringBootApplication(exclude = DruidDataSourceAutoConfigure.class)
@MapperScan("com.cy.dao")
public class APP {
    public static void main(String[] args) {
        SpringApplication.run(APP.class, args);
    }
}
```

## 理解

其实这个动态数据源的配置使用苞米豆的中间件还是蛮简单的。切换的时候使用@DS("slave_1")来指定数据源，不指定就使用默认的master.数据源名称也是可以根据自己来命名的。还有就是官网建议这个注解使用在service层。
