---
title: springboot项目配置swagger文档
categories: springboot swagger2
tags:  springboot swagger2
date: 2020-12-01 12:27:01
---

## 背景

自己项目里加了swagger发现文档页面很不友好，就研究了下其他项目文档是怎么改的ui

## 前提

springboot项目

## 方法

### 依赖

在pom文件里加入文档依赖

```pom
<!--swagger-->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.7.0</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.7.0</version>
</dependency>
<!--swagger的ui增强 来自第三方-->
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>swagger-bootstrap-ui</artifactId>
    <version>1.9.5</version>
</dependency>
```

注意： 如果使用的原生的ui，可视化很不友好，页面地址是 ip:端口/swagger-ui.html

引入swagger的ui增强依赖后，可视化好很多，页面地址是 ip:端口/doc.html

这个ui增强的组织的地址是：[swagger的ui增强组织地址](https://xiaoym.gitee.io/knife4j/documentation/description.html) 作者还进行了深度的封装，有兴趣的可以看下文档

### ui页面配置

```java
package com.example.demo.config;

import io.swagger.annotations.ApiOperation;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

import java.util.ArrayList;

/**
 * @ClassName SwaggerConfig
 * @Description: TODO
 * @Author ChenYuan
 * @Date 2020/12/1
 * @Version V1.0
 * 配置类
 * @Profile  该注释是开发和测试坏境 启用api文档 生产坏境不开启
 */
@Configuration
@EnableSwagger2
@Profile({"dev", "test"})
public class SwaggerConfig {
    //设置swagger跨域，提供给service调用

    @Bean
    public Docket docket() {
        return new Docket(DocumentationType.SWAGGER_2)
                .groupName("Test接口")
                .apiInfo(apiInfo())
                .select() //通过.select()方法，去配置扫描接口,RequestHandlerSelectors配置如何扫描接口
                .apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
                //.apis(RequestHandlerSelectors.basePackage("com.ccp.provider.config.controller"))
                //.apis(RequestHandlerSelectors.basePackage("com.ccp.provider.activity.controller"))
                .build();
    }

    /**
     * 配置文档信息
     * @return
     */
    private ApiInfo apiInfo(){
        Contact contact = new Contact("cy测试", "https://copasters.github.io/", "1572690269@qq.com");
        return new ApiInfo(
                // 标题
                "Test测试",
                // 描述
                "测试接口文档",
                // 版本
                "v1.0",
                // 组织链接
                "https://copasters.github.io/",
                // 联系人信息
                contact,
                // 许可
                "Apach 2.0 许可",
                // 许可连接
                "许可链接",
                // 扩展
                new ArrayList<>()
        );
    }
}
```

注意： 文档显示响应返回值是否美化，是根据你设置的数据类型的

```java
@ApiOperation(value = "测试接口",notes = "测试用户接口",response = User.class)
@GetMapping(value = "/test",produces = "application/json;charset=utf-8")
private String getUser(){
    List<User> user = userService.getUser();
    return JSON.toJSONString(user);
    }
```

上面的代码中produces决定了响应值是否被swagger页面jso格式美化。 produces = "text/plain;charset=utf-8" 不会被美化

### 代码地址

[小demo地址，在其他项目中加的swagger2](https://gitee.com/copasters/mybatis-source-code-learn)

## 感悟

看了这个ui修改作者的文档，有句原生ui不符合国人查看习惯，他才弄的。他也提到国内开源现状，感觉很感慨，希望他能坚持下去，向他看齐。大家有时间可以去看下这个开源项目。

