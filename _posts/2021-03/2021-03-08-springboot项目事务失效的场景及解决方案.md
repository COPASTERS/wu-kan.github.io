---
title: springboot项目事务失效的场景及解决方案
categories: springboot 事务
tags: springboot 事务
date: 2021-03-08 16:14:38
---

# 场景如下：

## 1.springboot的启动类上没有加事务开启注解导致失效

```java
@SpringBootApplication
@MapperScan("com.example.demo.dao")
@EnableSwagger2
@EnableTransactionManagement  // 开启事务注解
@EnableAspectJAutoProxy(exposeProxy = true)  // SpringAOP的功能 
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}
```

## 2.mysql数据库使用的存储引擎不支持事务（MyISAM（不支持事务）,InnoDB（支持事务））

即 确定你的数据库引擎是 InnoDB  下图可以看到Transactions列只有InnoDB支持事务

1、查看mysql支持的引擎有哪些

     show engines

[![1238912-20181220140328893-542442530.png](https://s3.ax1x.com/2021/03/08/6lldL6.png)](https://imgtu.com/i/6lldL6)

2、查看当前默认的引擎

    show variables like 'default_storage_engine'

3、查看指定表当前的引擎，有2种方式

     show table status where NAME ='t_user' 

或

    show create table t_user -- 在显示结果里参数engine后面的就表示该表当前用的存储引擎

4、修改指定表的引擎

    alter table t_user engine=innodb;

5、修改mysql默认的数据库引擎

打开配置文件my.ini，将“default-storage-engine=MYISAM”改为你想设定的，然后重启即可


## 3.@Transactional注解的要是public方法 

其实这个不怎么会忘记，使用idea的集成开发环境如果你注解一个私有的方法直接就有提示

## 4.@Transactional(rollbackFor = {RuntimeException.class, Exception.class}) 

回滚的异常要注意写对，内部代码被try{}catch(){}也不会回滚。。。。。。。

## 5.同一个类，内部不带事务的方法调用该类中带事务的方法，不会回滚。

原因：JDK的动态代理。只有被动态代理直接调用时才会产生事务。在SpringIoC容器中返回的调用的对象是代理对象而不是真实的对象。

解决办法：

### 方法1、在方法A上开启事务，方法B不用事务或默认事务，并在方法A的catch中throw new RuntimeException();(在没指定rollbackFor时，默认回滚的异常为RuntimeException)，这样使用的就是方法A的事务。（一定要throw new RuntimeException();否则异常被捕捉处理，同样不会回滚。）如下：

```java
@Transactional() //开启事务
public void save(){
    try {        
        this.saveEmployee();  //这里this调用会使事务失效，数据会被保存
    }catch (Exception e){
        e.printStackTrace();
        throw new RuntimeException();
    }
}
```

### 方法2、方法A上可以不开启事务，方法B上开启事务，并在方法A中将this调用改成动态代理调用(AopContext.currentProxy()),如下：

```java
public void save(){
    try {        
        EmployeeService proxy =(EmployeeService) AopContext.currentProxy();
        proxy.saveEmployee();
    }catch (Exception e){
        e.printStackTrace();
    }
}
```

**注意**

1.这个方法需要启动类上开启aop的配置 即加 @EnableAspectJAutoProxy(exposeProxy = true) 注解

2.添加依赖

```maven
 <!--配合  @EnableAspectJAutoProxy(exposeProxy = true)-->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.8.3</version>
        </dependency>
```

### 方法3、修改类，不要出现“自调用”的情况：这是Spring文档中推荐的“最佳”方案；

即重新写一个类，这样就避免自调用的this问题，事务就可以起作用了。

