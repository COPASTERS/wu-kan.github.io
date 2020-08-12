---
title: springboot同时配置mybatis和hibernate及hibernate的@ColumnTransformer注解
categories: springboot mybatis hibernate
tags:  springboot mybatis hibernate
date: 2020-08-05 17:44:26
---

## 背景

   之前我们使用的hibernate的框架，敏感数据采用的存储过程来加密和解密，代码里面只需要采用@@ColumnTransformer注解就可实现加密解密很方便。现在开发后台管理的平台，但是框架使用的mybatis，然后数据的加解密就不能用原来的方式的。网上看的是mybatis的加解密是在sql里面写的，我们采用的mybatis-plus的插件分页的，就搞不清怎么搞了。在使用上感觉hibernate的更加舒服顺滑一点。试下mybaits所有需要的这个字段的地方都要写这个调用过程还是很麻烦的。当然可能有更好的解决方案，但是还没发现。因为不想自己重新写分页代码，从头开始，最终想的是直接写一个sql调用存储过程把数据解密再赋值回去。理论上可行，就是把返回的分页数据在解密一遍，但是有点憨憨的感觉。哈哈 后面领导说不解密也没事，由于要搞其他的工作也就没弄了。偶然发现一篇文章说可以配置两个框架，我想这样就很顺滑了。分页这块和需要解密的使用hibernate。其他的使用mybatis。这样就即灵活又便捷。

## 注意
  
  我测试采用的mysql5.7.26数据库，加密解密使用的也是mysql内置的存储过程
  具体的在后面有写到。自己测试自己的那个版本有没有内置这个。常用版本应该都有。出现的问题当然也对应相应的版本和相应框架版本

## 实现

  千言万语不如撸代码
  [小demo代码地址](https://gitee.com/copasters/doubleDataSourceORM "托管于gitee"）

## 小理解

  整个配置过程还是蛮顺利的，中心思想就是两个混用到一起，合在一起该怎么用还怎么用，没有什么比较奇特的。
  主要是实体类上的重要注解和用哪一个框架没有关系。只需要加上各自比较特殊的就ok了。最后各自实现各自的dao层逻辑。service注入两个查询的接口。后面的就没有要改的了。费很多时间的是@ColumnTransformer这个注解实现调用数据库的存储过程来进行敏感数据的加密和解密。

  关键代码：

```java
  /**
     * hibernate5以上的版本必须加@column,@forColumn否则转换会失效
     * 巨坑是不要把盐上带有字段名称 否则解密会失败 因为语句里解密时会
     把表的临时名称加上
     * 大概内部sql语句生成的时候的问题吧.比如例子中不要写盐为： sex   或者  sex_kk
     * 否则上述问题会出现
  */
  @Column(name = "SEX")
  @ColumnTransformer(
          forColumn = "SEX",
          write = "HEX(AES_ENCRYPT(?, 'salt'))",
          read = "AES_DECRYPT(UNHEX(sex),'salt')"
  )
  private String sex;
```

## 后记

   网上说@column,@forColumn里面的值要小写，但是我测了下大小写都可以没啥影响，如果你的大写不行可以改成小写。里面关于盐的理解是我看具体执行的sql发现的，如果你非要使用字段名称，可以在加密的时候在盐的前面加上sql执行时的表的别名 我这边固定是user0_ 就（write = "HEX(AES_ENCRYPT(?, 'user0_.sex'))"）,这样解密代码自动加的别名也就刚好是盐 哈哈。 至于里面的hex和unhex是mysql的编码函数。通过看sql也可以看到其实最终还是sql的组装。转换只是sql加上了解码函数去执行而已。
   在这个过程中发现@ColumnTransformer的用处不只这一点，还可以执行sql去其他表拿数据进行赋值，这样一想用处还是蛮多。也符合注解的英文名称的中文意思栏数据转换。至于mysql的加密解密的存储过程可以在数据库侧进行再封装把盐值固定住。这样代码也简单安全一些。