---
title: SpringBoot使用websocket实现长链接通信
categories: SpringBoot websocket
tags:  SpringBoot websocket
date: 2021-02-22 13:43:35
---

## 背景

在批量数据入库完成时后端主动发信息给前端，通知数据入库完成。

## websocket 实现长连接原理

WebSocket是HTML5出的东西（协议）,也就是说HTTP协议没有变化，或者说没关系，但HTTP是不支持持久连接的（长连接，循环连接的不算）首先HTTP有1.1和1.0之说，也就是所谓的keep-alive，把多个HTTP请求合并为一个，但是Websocket其实是一个新协议，跟HTTP协议基本没有关系，只是为了兼容现有浏览器的握手规范而已，也就是说它是HTTP协议上的一种补充可以通过这样一张图理解。

![CY_2021-02-22-13-52-46.png](https://s3.ax1x.com/2021/02/22/y75uIU.png)

有交集，但是并不是全部。
另外Html5是指的一系列新的API，或者说新规范，新技术。Http协议本身只有1.0和1.1，而且跟Html本身没有直接关系。。
通俗来说，你可以用HTTP协议传输非Html数据，就是这样=。=
再简单来说，层级不一样。

具体更加详细的解释，可自行百度，蛮多。

## 代码实现

使用Spring 4 零配置文件的方式建立项目

### 第一步,导入依赖

```text
<properties>
        <spring.version>4.3.18.RELEASE</spring.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-websocket</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-messaging</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>webjars-locator</artifactId>
            <version>0.32-1</version>
        </dependency>
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>sockjs-client</artifactId>
            <version>1.0.2</version>
        </dependency>
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>stomp-websocket</artifactId>
            <version>2.3.3</version>
        </dependency>
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>jquery</artifactId>
            <version>3.1.0</version>
        </dependency>

    </dependencies>
```

### 第二步 建立程序入口（实现零配置启动）

新建\spring-websocket-stomp\src\main\java\cn\justsoul\study\config\DispatcherServletInitializer.java类，继承自AbstractAnnotationConfigDispatcherServletInitializer，来快速implements WebApplicationInitializer。
这里要说的是WebApplicationInitializer这个接口是实现零配置文件的关键，无论你怎么实现了它，就可以以它的实现作为程序入口了。
再深入一点：在Servlet3.0环境中，容器会在类路径中查找实现javax.servlet.ServletContainerInitializer接口的类，如果能发现的话，就会用它来配置Servlet容器。
Spring提供了这个接口的实现，名为SpringServletContainerInitializer，这个类反过来又会查找实现WebApplicationInitializer基础实现。我们这里用的是extends AbstractAnnotationConfigDispatcherServletInitializer，因此当部署到Servlet3.0容器（Tomcat7或者更高版本等）中的时候，容器会自动发现它，并用它来配置Servlet上下文。

```java
public class DispatcherServletInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
    protected Class<?>[] getRootConfigClasses() {
        return new Class[0];
    }

    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[] { WebConfig.class, WebSocketConfig.class };
    }

    protected String[] getServletMappings() {
        return new String[] { "/" };
    }
    @Override
    protected void customizeRegistration(Dynamic registration) {
        registration.setInitParameter("dispatchOptionsRequest", "true");
    }
}
```

### 第三步 （重点来了）创建配置类
新建\spring-websocket-stomp\src\main\java\cn\justsoul\study\config\WebConfig.java配置类，主要是配置SpringMVC
```java
@Configuration
@EnableWebMvc
@ComponentScan(basePackages = "cn.justsoul.study.controller")
public class WebConfig extends WebMvcConfigurerAdapter {

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/webjars/**").addResourceLocations("/webjars/")
                .resourceChain(false)
                .addResolver(new WebJarsResourceResolver())
                .addResolver(new PathResourceResolver());
        registry.addResourceHandler("/js/**").addResourceLocations("/js/");
    }

    @Bean
    public ViewResolver viewResolver() {
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setPrefix("/WEB-INF/jsp/");
        viewResolver.setSuffix(".jsp");
        return viewResolver;
    }

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("index");
    }
}
```

新建\spring-websocket-stomp\src\main\java\cn\justsoul\study\config\WebSocketConfig.java类，进行WebSocket配置

***这里要注意***的是registry.enableSimpleBroker(“/topic”,”/queue”);和registry.setUserDestinationPrefix(“/queue”);两个配置项里都要有”/queue”，之后convertAndSendToUser(“user”, “destination”,”payload”);这个方法才能生效！！！

```java
@Configuration
@EnableWebSocketMessageBroker
@EnableScheduling
public class WebSocketConfig extends AbstractWebSocketMessageBrokerConfigurer {
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/websocket-address").withSockJS();
    }
    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableSimpleBroker("/topic","/queue");//设置服务器广播消息的基础路径,可以是多个路径
        registry.setApplicationDestinationPrefixes("/app");//设置客户端订阅消息基础路径
        registry.setUserDestinationPrefix("/queue");//设置精准消息推送基础路径
    }
}
```

### 第四步 实现消息订阅推送
编写\spring-websocket-stomp\src\main\java\cn\justsoul\study\controller\TopicController.java类
这里科普一下@EnableScheduling与@Scheduled是Spring定时任务，没用过的了解一下：）

```java
@Controller
public class TopicController {

    /*收到消息后广播推送*/
    @MessageMapping("/boardcast")
    @SendTo("/topic/boardcast")
    public String sendtousers(String s){
        System.out.println(s);
        return "我是广播消息";
    }

    /*收到消息后精准投送到单个用户*/
    @MessageMapping("/precise")
    /*broadcast = false避免把消息推送到同一个帐号不同的session中*/
    @SendToUser(value = "/topic/precise",broadcast = false)
    public String sendtouser(String s){
        System.out.println(s);
        return "我是精准投送消息";
    }


    private SimpMessagingTemplate template;
    @Autowired
    public TopicController (SimpMessagingTemplate template) {
        this.template = template;
    }
    /*下面2个是不用接收消息动态发送消息的方法*/
    /*Scheduled为定时任务，演示*/
    @Scheduled(fixedDelay = 1500)
    public void boardcast(){
        this.template.convertAndSend("/topic", "来自服务器广播消息");
    }

    @Scheduled(fixedDelay = 1500)
    public void precise(){
        this.template.convertAndSendToUser("abc", "/message","来自服务器精准投送消息");
    }

}
```

### 第五 前端测试页面
\spring-websocket-stomp\src\main\webapp\WEB-INF\jsp\index.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title>spring-websocket-stomp</title>
  </head>
  <body>
  <h1>spring-websocket-stomp</h1>
  <button id="btn-send">广播消息</button>
  <button id="btn-send2">精准投送消息</button>
  <button id="btn-send3">动态广播</button>
  <button id="btn-send4">动态精准投送</button>
  </body>
</html>
<script src="webjars/jquery/jquery.min.js"></script>
<script src="webjars/sockjs-client/sockjs.min.js"></script>
<script src="webjars/stomp-websocket/stomp.min.js"></script>
<script src="js/app.js"></script>
```

\spring-websocket-stomp\src\main\webapp\js\app.js

```jsp
var stompClient = null;
var subscription = null;
$(function () {
   var ws = new SockJS("/websocket-address");
    stompClient = Stomp.over(ws);
    stompClient.connect({'client-id': 'my-client'},function () {
    });

    $("#btn-send").click(function () {
        if(subscription != null){subscription.unsubscribe();}
        subscription = stompClient.subscribe("/topic/boardcast", function(){});
        stompClient.send("/app/boardcast",{},"请求广播");
    });
    $("#btn-send2").click(function () {
        if(subscription != null){subscription.unsubscribe();}
        subscription = stompClient.subscribe("/queue/topic/precise", function(){});
        stompClient.send("/app/precise",{},"请求精准投送");
    });
    // 请求动态广播
    $("#btn-send3").click(function () {
        if(subscription != null){subscription.unsubscribe();}
        subscription = stompClient.subscribe("/topic", function(){});
    });
    // 请求精准投送
    $("#btn-send4").click(function () {
        if(subscription != null){subscription.unsubscribe();}
        subscription = stompClient.subscribe("/queue/abc/message", function(){});
    });
});
```
## 第六 项目部署

在idea中使用tomcat启动项目，首页即为index.jsp 打开浏览器控制台，点击按钮即可测试数据传输。

## 项目代码

[点击进入gitee获取项目代码](https://gitee.com/copasters/spring-websocket-stomp)

参考文章地址：

[Spring Websocket Stomp 消息订阅推送实战](https://blog.csdn.net/chinatreeqy/article/details/81115296?utm_medium=distribute.pc_relevant_bbs_down.none-task-blog-baidujs-1.nonecase&depth_1-utm_source=distribute.pc_relevant_bbs_down.none-task-blog-baidujs-1.nonecase)
[websocket 实现长连接原理](https://blog.csdn.net/qq_35623773/article/details/87868682)