---
title: 使用java编程的方式自动关闭springBoot应用
categories: springBoot
tags: springBoot
date: 2020-04-19 08:49:04
---

# 背景

  公司经常会有数据查询问题，这种是需要调用接口查回实时数据。数据比较多就采用分批次定时来执行这个操作。随之而来的问题是当全部数据已经更新结束后，应用没有关闭。这样不停会有一小段日志打印，看着贼不舒服，而且时间长了日志量还是不少的。然后就想如果跑完了，程序可以自己kill 自己就好了。

# 解决方法

  话不多说，代码如下：

```java
import org.springframework.context.ApplicationContext;
import org.springframework.boot.SpringApplication;

@Service
class ShutdownManager{
  @Autowired
  private ApplicationContext appContext;

  public void initiateShutdown(int returnCode){
    SpringApplication.exit(appContext, () -> returnCode);
  }
}
```

在需要关闭服务的地方，调用方法就ok了，还是蛮简单的。