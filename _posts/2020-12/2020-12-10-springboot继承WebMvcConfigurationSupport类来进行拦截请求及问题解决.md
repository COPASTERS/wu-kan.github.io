---
title: springboot继承WebMvcConfigurationSupport类来进行拦截请求及问题解决
categories: springboot springMvc拦截器
tags:  springboot springMvc拦截器
date: 2020-12-10 14:32:01
---

## 背景

实现WebMvcConfigurer来进行请求拦截的另一种实现方式，继承WebMvcConfigurationSupport类。

## 存在的问题

从spring boot2.0之后在构造spring配置文件时建议推荐直接实现WebMvcConfigurer或者直接继承WebMvcConfigurationSupport ，经测试实现WebMvcConfigurer是没问题，但继承WebMvcConfigurationSupport类是会导致自动配置失效的。

## 一、继承WebMvcConfigurationSupport类是会导致自动配置失效的原因

在spring boot的自定义配置类继承 WebMvcConfigurationSupport 后，发现自动配置的静态资源路径（classpath:/META/resources/，classpath:/resources/，classpath:/static/，classpath:/public/）不生效。

首先看一下 自动配置类的定义的源码：

```java
/**
 * {@link EnableAutoConfiguration Auto-configuration} for {@link EnableWebMvc Web MVC}.
 *
 * @author Phillip Webb
 * @author Dave Syer
 * @author Andy Wilkinson
 * @author Sébastien Deleuze
 * @author Eddú Meléndez
 * @author Stephane Nicoll
 * @author Kristine Jetzke
 * @author Bruce Brouwer
 */
@Configuration
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class,
		ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {
    .................省略
}
```

这是因为在 springboot的web自动配置类 WebMvcAutoConfiguration 上有条件注解 

    @ConditionalOnMissingBean(WebMvcConfigurationSupport.class)

这个注解的意思是在项目类路径中 缺少 WebMvcConfigurationSupport类型的bean时改自动配置类才会生效，所以继承 WebMvcConfigurationSupport 后需要自己再重写相应的方法。

如果想要使用自动配置生效，又要按自己的需要重写某些方法，比如增加 viewController ，则可以自己的配置类可以继承  WebMvcConfigurerAdapter 这个类。不过在spring5.0版本后这个类被丢弃了 WebMvcConfigurerAdapter  ，虽然还可以用，但是看起来不好 = =。

## 二、继承WebMvcConfigurationSupport类如何配置拦截器的

```java
@Configuration
public class MyConfigurer extends WebMvcConfigurationSupport {
 
@Override
    protected void addInterceptors(InterceptorRegistry registry) {
       registry.addInterceptor(new MyInterceptor()).addPathPatterns("/**").excludePathPatterns("/emp/toLogin","/emp/login","/js/**","/css/**","/images/**");
        super.addInterceptors(registry);
    }
 
    @Override
    protected void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/**").addResourceLocations("classpath:/static/");
        super.addResourceHandlers(registry);
    }
}
```

注意这段代码：

    registry.addResourceHandler("/**").addResourceLocations("classpath:/static/");

由于继承WebMvcConfigurationSupport后会导致自动配置失效，所以这里要指定默认的静态资源的位置。同时要注意不能写成

    registry.addResourceHandler("/static/**").addResourceLocations("classpath:/static/");

原博客地址：https://blog.csdn.net/fmwind/article/details/82832758