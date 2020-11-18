---
title: springboot的日志配置文件logback-spring.xml
categories: springboot logback
tags:  springboot logback
date: 2020-11-18 15:47:43
---

## 背景

有时候调试需要看下日志信息，去哪个文件看，如何把自己需要的日志打印到制定的问件，方便自己快速查看。达到这样的目的就需要对日志的输出打印的配置文件有所了解。自己花了一点时间，理论加实践的操作了一波，算是有了一定的认识。日志框架是logback

## 前提
   
springboot项目logback日志的依赖是本身就有的，所以不需要自己手动在引入一遍，否则可能会出现包冲突的问题。位置如图：

![CY_2020-11-18-15-57-41.png](https://s3.ax1x.com/2020/11/18/DmyEjK.png)

## 配置文件的理解

### 正文

Spring Boot在所有内部日志中使用Commons Logging，但是默认配置也提供了对常用日志的支持，如：Java Util Logging，Log4J, Log4J2和Logback。每种Logger都可以通过配置使用控制台或者文件输出日志内容。

### 默认日志Logback
SLF4J——Simple Logging Facade For Java，它是一个针对于各类Java日志框架的统一Facade抽象。Java日志框架众多——常用的有java.util.logging, log4j, logback，commons-logging, Spring框架使用的是Jakarta Commons Logging API (JCL)。而SLF4J定义了统一的日志抽象接口，而真正的日志实现则是在运行时决定的——它提供了各类日志框架的binding。

Logback是log4j框架的作者开发的新一代日志框架，它效率更高、能够适应诸多的运行环境，同时天然支持SLF4J。

***默认情况下***，Spring Boot会用Logback来记录日志，并用INFO级别输出到控制台。在运行应用程序和其他例子时，你应该已经看到很多INFO级别的日志了。

### 默认配置属性支持

Spring Boot为我们提供了很多默认的日志配置，所以，只要将spring-boot-starter-logging作为依赖加入到当前应用的classpath，则“开箱即用”。

下面介绍几种在application.properties就可以配置的日志相关属性。

控制台输出

***日志级别从低到高分为TRACE < DEBUG < INFO < WARN < ERROR < FATAL，如果设置为WARN，则低于WARN的信息都不会输出。***

Spring Boot中默认配置ERROR、WARN和INFO级别的日志输出到控制台。您还可以通过启动您的应用程序--debug标志来启用“调试”模式（开发的时候推荐开启）,以下两种方式皆可：

在运行命令后加入--debug标志，如：$ java -jar springTest.jar --debug

在application.properties中配置debug=true，该属性置为true的时候，核心Logger（包含嵌入式容器、hibernate、spring）会输出更多内容，但是你自己应用的日志并不会输出为DEBUG级别。

### 文件输出

默认情况下，Spring Boot将日志输出到控制台，不会写到日志文件。如果要编写除控制台输出之外的日志文件，则需在application.properties中设置logging.file或logging.path属性。

logging.file，设置文件，可以是绝对路径，也可以是相对路径。如：logging.file=my.log

logging.path，设置目录，会在该目录下创建spring.log文件，并写入日志内容，如：logging.path=/var/log

如果只配置 logging.file，会在项目的当前路径下生成一个 xxx.log 日志文件。

如果只配置 logging.path，在 /var/log文件夹生成一个日志文件为 spring.log

注：二者不能同时使用，如若同时使用，则只有logging.file生效

默认情况下，日志文件的大小达到10MB时会切分一次，产生新的日志文件，默认级别为：ERROR、WARN、INFO

### 级别控制

所有支持的日志记录系统都可以在Spring环境中设置记录级别（例如在application.properties中）

格式为：'logging.level.* = LEVEL'

- logging.level：日志级别控制前缀，*为包名或Logger名

- LEVEL：选项TRACE, DEBUG, INFO, WARN, ERROR, FATAL, OFF

举例：

- logging.level.com.dudu=DEBUG：com.dudu包下所有class以DEBUG级别输出

- logging.level.root=WARN：root日志以WARN级别输出

### 自定义日志配置

由于日志服务一般都在ApplicationContext创建前就初始化了，它并不是必须通过Spring的配置文件控制。因此通过系统属性和传统的Spring Boot外部配置文件依然可以很好的支持日志控制和管理。

根据不同的日志系统，你可以按如下规则组织配置文件名，就能被正确加载：

Logback：logback-spring.xml, logback-spring.groovy, logback.xml, logback.groovy

Log4j：log4j-spring.properties, log4j-spring.xml, log4j.properties, log4j.xml

Log4j2：log4j2-spring.xml, log4j2.xml

JDK (Java Util Logging)：logging.properties

Spring Boot官方推荐优先使用带有-spring的文件名作为你的日志配置（如使用logback-spring.xml，而不是logback.xml），命名为logback-spring.xml的日志配置文件，spring boot可以为它添加一些spring boot特有的配置项（根据环境选择不同的日志输入）。

上面是默认的命名规则，并且放在src/main/resources下面即可。

如果你即想完全掌控日志配置，但又不想用logback.xml作为Logback配置的名字，可以通过logging.config属性指定自定义的名字：

logging.config=classpath:logging-config.xml

虽然一般并不需要改变配置文件的名字，但是如果你想针对不同运行时Profile使用不同的日志配置，这个功能会很有用。

## 下面具体说下logback-spring.xml的配置详情

### 根节点<configuration>包含的属性

scan:当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。

scanPeriod:设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。

debug:当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。

根节点<configuration>的子节点：

在<configuration>下面一共有2个属性，3个子节点


### 属性一：设置上下文名称<contextName>

每个logger都关联到logger上下文，默认上下文名称为“default”。但可以使用<contextName>设置成其他名字，用于区分不同应用程序的记录。一旦设置，不能修改,可以通过%contextName来打印日志上下文名称。

<contextName>logback</contextName>

### 属性二：设置变量<property>

用来定义变量值的标签，<property> 有两个属性，name和value；其中name的值是变量的名称，value的值时变量定义的值。通过<property>定义的值会被插入到logger上下文中。定义变量后，可以使“${}”来使用变量。

<property name="log.path" value="E:\\logback.log" />

### 子节点一<appender>

appender用来格式化日志输出节点，有俩个属性name和class，class用来指定哪种输出策略，常用就是控制台输出策略和文件输出策略。

控制台输出ConsoleAppender：(我希望自己的控制台不同级别有颜色区分，所以加了彩色日志渲染)

```java
    <!-- 彩色日志  测试类没有效果-->
    <!-- 彩色日志依赖的渲染类 -->
    <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter" />
    <conversionRule conversionWord="wex" converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter" />
    <conversionRule conversionWord="wEx" converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter" />
    <!-- 彩色日志格式 -->
    <property name="CONSOLE_LOG_PATTERN" value="${CONSOLE_LOG_PATTERN:-%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}" />

    <!-- 用于说明输出介质，此处说明控制台输出 -->
    <appender name="Console" class="ch.qos.logback.core.ConsoleAppender">
        <!-- 类似于layout，除了将时间转化为数组，还会将转换后的数组输出到相应的文件中 -->
        <!--        <encoder>表示对日志进行编码：

        %d{HH: mm:ss.SSS}——日志输出时间

        %thread——输出日志的进程名字，这在Web应用以及异步任务处理中很有用

        %-5level——日志级别，并且使用5个字符靠左对齐

        %logger{36}——日志输出者的名字

        %msg——日志消息

        %n——平台的换行符-->
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!-- 定义日志的输出格式 -->
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
            <charset>UTF-8</charset> <!-- 此处设置字符集 -->
        </encoder>
         <!--过滤掉Error级别的日志，此appender仅筛选Error级别日志并输出到控制台-->
<!--        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">-->
<!--             <level>ERROR</level>-->
<!--        </filter>-->
    </appender>
```

<encoder>表示对日志进行编码：

- %d{HH: mm:ss.SSS}——日志输出时间

- %thread——输出日志的进程名字，这在Web应用以及异步任务处理中很有用

- %-5level——日志级别，并且使用5个字符靠左对齐

- %logger{36}——日志输出者的名字

- %msg——日志消息

- %n——平台的换行符

***ThresholdFilter为系统定义的拦截器，例如我们用ThresholdFilter来过滤掉ERROR级别以下的日志不输出到文件中。如果不用记得注释掉，不然你控制台会发现没日志~***

输出到文件RollingFileAppender：

另一种常见的日志输出到文件，随着应用的运行时间越来越长，日志也会增长的越来越多，将他们输出到同一个文件并非一个好办法。RollingFileAppender用于切分文件日志：

```java
<appender name="Debug"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 文件路径 -->
        <file>${log.filePath}/debug.log</file>
        <!-- 滚动策略 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 设置文件名称 -->
            <fileNamePattern>
                ${log.filePath}/debug/debug_%d{yyyy-MM-dd}_%i.log.zip
            </fileNamePattern>
            <!-- 设置最大保存周期 -->
            <maxHistory>${log.maxHistory}</maxHistory>
            <cleanHistoryOnStart>true</cleanHistoryOnStart>
            <!-- 除按日志记录之外，还配置了日志文件不能超过100M，若超过100M，日志文件会以索引0开始，
            命名日志文件，例如log-error-2013-12-21.0.log -->
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!-- 定义日志的输出格式 -->
            <pattern>${log.pattern}</pattern>
            <charset>UTF-8</charset> <!-- 此处设置字符集 -->
        </encoder>
        <!-- 过滤器，过滤掉不是指定日志水平的日志 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!-- 设置日志级别 -->
            <level>DEBUG</level>
            <!-- 如果跟该日志水平相匹配，则接受 -->
            <onMatch>ACCEPT</onMatch>
            <!-- 如果跟该日志水平不匹配，则过滤掉 -->
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
```
其中重要的是rollingPolicy的定义，上例中<fileNamePattern>标签定义了日志的切分方式——把每一天的日志归档到一个文件中并压缩，<maxHistory>30</maxHistory>表示只保留最近30天的日志，以防止日志填满整个磁盘空间。同理，可以使用%d{yyyy-MM-dd_HH-mm}来定义精确到分的日志切分方式。<timeBasedFileNamingAndTriggeringPolicy>用来指定日志文件的上限大小，例如设置为100MB的话，那么到了这个值，若超过100M，日志文件会以索引0开始，命名日志文件，例如log-error-2013-12-21.0.log 。

### 子节点二<root>
```java
<!-- 特殊的logger，根logger -->
    <root lever="${dao.level}">
        <!-- 指定默认的日志输出 -->
        <appender-ref ref="Debug" />
        <appender-ref ref="Warn" />
        <appender-ref ref="Error" />
        <appender-ref ref="Info" />
        <appender-ref ref="Console2" />
    </root>
```
root节点是必选节点，用来指定最基础的日志输出级别，只有一个level属性。

level:用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，不能设置为INHERITED或者同义词NULL，默认是DEBUG。

<root>可以包含零个或多个<appender-ref>元素，标识这个appender将会添加到这个loger。

### 子节点三<loger>

<loger>用来设置某一个包或者具体的某一个类的日志打印级别、以及指定<appender>。<loger>仅有一个name属性，一个可选的level和一个可选的addtivity属性。

- name:用来指定受此loger约束的某一个包或者具体的某一个类。

- level:用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，还有一个特俗值INHERITED或者同义词NULL，代表强制执行上级的级别。如果未设置此属性，那么当前loger将会继承上级的级别。

- addtivity:是否向上级loger传递打印信息。默认是true。


```java
<logger name="com.example.demo" level="${dao.level}" additivity="true">
    <appender-ref ref="Console" />
</logger>
<logger name="com.example.demo" level="${dao.level}" additivity="false">
    <appender-ref ref="Dao" />
</logger>
```

#### 第一种：带有loger的配置，不指定级别，不指定appender

<logger name="com.dudu.controller"/>

<logger name="com.dudu.controller" />将控制controller包下的所有类的日志的打印，但是并没用设置打印级别，所以继承他的上级<root>的日志级别“info”；

没有设置addtivity，默认为true，将此loger的打印信息向上级传递；

没有设置appender，此loger本身不打印任何信息。

#### 第二种：带有多个loger的配置，指定级别，指定appender

```java
<logger name="com.example.demo" level="${dao.level}" additivity="false">
    <appender-ref ref="Dao" />
</logger>
```

控制com.example.demo包的日志打印，打印级别为${dao.level} = debug级别

additivity属性为false，表示此loger的打印信息不再向上级传递

指定了名字为“Dao”的appender

这时候com.example.demo包中代码运行时，先执行<logger name="com.example.demo" level="${dao.level}" additivity="false">,将级别为“DEBUG”及大于“DEBUG”的日志信息交给此loger指定的名为“Dao”的appender处理，在对应的文件中打出日志，不再向上级root传递打印信息。

#### 多环境日志输出

***由于我没有配多文件,看了下不难，也就没自己实现一遍**

据不同环境（prod:生产环境，test:测试环境，dev:开发环境）来定义不同的日志输出，在 logback-spring.xml中使用 springProfile 节点来定义，方法如下：

***文件名称不是logback.xml，想使用spring扩展profile支持，要以logback-spring.xml命名***

```java
    <!-- 4. 最终的策略 -->
    <!-- 4.1 开发环境:打印控制台-->
    <springProfile name="dev">
        <logger name="com.example.demo" level="debug"/>
    </springProfile>
 
    <root level="info">
        <appender-ref ref="CONSOLE" />
        <appender-ref ref="DEBUG_FILE" />
        <appender-ref ref="INFO_FILE" />
        <appender-ref ref="WARN_FILE" />
        <appender-ref ref="ERROR_FILE" />
    </root>
 
    <!-- 4.2 生产环境:输出到文档
    <springProfile name="pro">
        <root level="info">
            <appender-ref ref="CONSOLE" />
            <appender-ref ref="DEBUG_FILE" />
            <appender-ref ref="INFO_FILE" />
            <appender-ref ref="ERROR_FILE" />
            <appender-ref ref="WARN_FILE" />
        </root>
    </springProfile> -->
```

可以启动服务的时候指定 profile （如不指定使用默认），如指定prod 的方式为：

java -jar xxx.jar --spring.profiles.active=prod

然后项目会根据环境使用对应的日志配置。

# 具体的demo代码

    里面含有完整的logback-spring.xml文件

    [Gitee](https://gitee.com/copasters/mybatis-source-code-learn)



