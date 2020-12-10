---
title: springboot不采用@Bean的方式拿到想要使用的bean方式及ApplicationContextAware使用理解
categories: springboot ApplicationContextAware
tags:  springboot ApplicationContextAware
date: 2020-12-10 15:50:09
---

## 1. 不采用@Bean的方式拿到想要使用的bean方式

### 1.1 手动实例化了一次（非new）

```java
ApplicationContext appContext = new ClassPathXmlApplicationContext("applicationContext-common.xml");  
AbcService abcService = (AbcService)appContext.getBean("abcService");  
```
#### 存在的问题

但是这样就会存在一个问题：因为它会重新装载applicationContext-common.xml并实例化上下文bean，如果有些线程配置类也是在这个配置文件中，那么会造成做相同工作的的线程会被启两次。一次是web容器初始化时启动，另一次是上述代码显示的实例化了一次。当于重新初始化一遍！！！！这样就产生了冗余。

### 1.2 类实现了这个接口（ApplicationContextAware）之后，这个类就可以方便获得ApplicationContext中的所有bean

不用类似new ClassPathXmlApplicationContext()的方式，从已有的spring上下文取得已实例化的bean。通过ApplicationContextAware接口进行实现。

当一个类实现了这个接口（ApplicationContextAware）之后，这个类就可以方便获得ApplicationContext中的所有bean。换句话说，就是这个类可以直接获取spring配置文件中，所有有引用到的bean对象。


```java
@Component
public class AppUtil implements ApplicationContextAware {

    private static ApplicationContext applicationContext;
    
    @Override
    public void setApplicationContext(ApplicationContext arg0) throws BeansException {
        applicationContext = arg0;
    }

    public static Object getObject(String id) {
        Object object = null;
        object = applicationContext.getBean(id);
        return object;
    }
}
```

**备注**

（1）在spring的配置文件中，注册方法类AppUtil。之所以方法类AppUtil能够灵活自如地获取ApplicationContext，就是因为spring能够为我们自动地执行了setApplicationContext。但是，spring不会无缘无故地为某个类执行它的方法的，所以，就很有必要通过注册方法类AppUtil的方式告知spring有这样子一个类的存在。这里我们使用@Component来进行注册，或者我们也可以像下面这样在配置文件声明bean：

    <bean id="appUtil" class="com.htsoft.core.util.AppUtil"/>

（2）加载Spring配置文件时，如果Spring配置文件中所定义的Bean类实现了ApplicationContextAware 接口，那么在加载Spring配置文件时，会自动调用ApplicationContextAware 接口中的

    public void setApplicationContext(ApplicationContext context) throws BeansException

方法，获得ApplicationContext对象,ApplicationContext对象是由spring注入的。前提必须在Spring配置文件中指定该类。

（3）使用静态的成员ApplicationContext类型的对象。

# 使用场景备注

从ApplicationContextAware获取ApplicationContext上下文的情况，仅仅适用于当前运行的代码和已启动的Spring代码处于同一个Spring上下文，否则获取到的ApplicationContext是空的。

比如我要为当前系统加入一个定时任务，定时刷新Memcache缓存。这个定时任务框架是公司的框架，下面是我的ApplicationContextAware 接口实现类：

```java
@Component
public class ApplicationContextUtil implements ApplicationContextAware {

    private static ApplicationContext applicationContext;//声明一个静态变量保存

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("applicationContext正在初始化,application:"+appContext);
        this.applicationContext=applicationContext;
    }

    public static <T> T getBean(Class<T> clazz){
        if(applicationContext==null){
            System.out.println("applicationContext是空的");
        }else{
            System.out.println("applicationContext不是空的");
        }
        return applicationContext.getBean(clazz);
    }

    public static ApplicationContext getApplicationContext(){
        return applicationContext;
    }
}
```

定时任务类如下，定时任务初始化的时候，首先会调用作业类的public static Object getObject()方法返回作业类的实例。

```java
@Component
public class RemoteCacheJob extends AbstractSaturnJavaJob {

    @Autowired
    private AdsRemoteCacheJob adsRemoteCacheJob;

    @Autowired
    private ILogService logService;

  // 实例化的过程：系统会首先调用作业类的public static Object getObject()方法，
  // 如果返回为null，则调用作业类的无参构造方法来实例化；否则直接使用getObject()方法返回的对象作为作业类实例。
    public static Object getObject() {
        return ApplicationContextUtil.getBean(RemoteCacheJob .class);
    }

    @Override
    public void handleJavaJob(String jobName, Integer shardItem, String shardParam, SaturnJobExecutionContext shardingContext)
            throws InterruptedException {
          System.out.println("处理定时任务");
    }
}
```

你可以试下定时任务可以获取到ApplicationContexta吗？

好像定时任务是没办法获取到项目所在Spring容器启动之后的ApplicationContext。