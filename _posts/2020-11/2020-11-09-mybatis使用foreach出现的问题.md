---
title: mybatis使用foreach出现的问题
categories: mybatis foreach
tags:  mybatis foreach 
date: 2020-11-09 19:00:00
---

## 步骤一：开通打印SQL语句

因为是通过MyBatis 连接数据库，所以需要查看SQL的执行语句

Spring+Mybatis在控制台输出SQL的最简单方法：

在application.properties文件中添加：

logging.leve.你的Mapper所在的包=debug （如果你的Mapper包在com.demo.mapper就填这个）

## 步骤二：检查SQL里的字段

SQL里的字体要与实体类的成员变量相对应，大小写也要注意
注意区分属性ResutlMap和ResultType的使用，别混了。

## 步骤三：检查数据库的字符编码

数据库、表、字段的字符编码都应该统一，最好设置成utf8-general_ci

## 步骤四：检查Mapper是否注入成功

@Autowire报错导致的NullPointerException，虽然报错，项目运行没有问题。可是只有紧挨着@Autowire的一个起作用，下面的都没有注入成功。这时需要在每一个注入的Mapper中都加上@Autowire。

## 步骤五：List All elements are null导致NullPointerException

### 注意

***当List对象显示 All elements are null时，虽然输入为[null]，但是list.size()=1。不管是list==null，list.isEmpty()，list.size()都无法判断list是否为空。***

处理办法是在代码中处理传入到mapper中sql的参数

比如：传入的集合是list

可以在执行上面语句时，加上list.remove(null)