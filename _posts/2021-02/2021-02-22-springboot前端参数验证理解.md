---
title: SpringBoot前端参数验证理解
categories: SpringBoot 参数校验
tags:  SpringBoot 参数校验
date: 2021-02-22 11:04:45
---

## 背景

有时候做web端入参判断很费事，看项目已经集成了参数校验的依赖及相关配置，只是没人用。我就来用一下。具体项目集成Hibernate Validator的过程网上很多我就不再重复了，本文主要体现使用上的方法。

## 参数校验的必要性

对于任何一个应用而言，在客户端做的数据有效性验证主要目的是规范用户的输入，而真实的数据验证工作都是在服务后端代码当中实现的，但在实际的项目当中，也经常会因为各种各样的原因：懒得写，觉得前端验证了，后端没有太多的必要等等没有进行数据验证，其实养成数据的有效性验证是一个非常好的习惯。

1. 可以避免很多数据有效性导致的BUG，防范其余开发者的基础攻击

2. 在前后端进行接口联调的时候，不需要因为参数的问题沟通很久。

## springboot 参数验证

JSR-303 是 JAVA EE 6 中的一项子规范，叫做 Bean Validation，官方参考实现是Hibernate Validator。JSR 303 用于对 Java Bean 中的字段的值进行验证。 主要是 javax.validation 包下面的注解，用于进行参数的验证。在 spring-boot当中存在 hibernate-validator 验证包，这个包里面包含了一些 javax.validation 没有的注解。算是spring对于JSR验证的的扩展吧！

## 常用验证注解：

| 注解 | 用法 | 
| - | - |
| @NotNull | 限制必须不为null 多用于对象的字段上 |
| @Null | 限制必须为null |
| @NotEmpty | 验证注解的元素值不为 null 且不为空（字符串长度不为0、集合大小不为0） 多用于集合字段上 |
| @NotBlank | @NotBlank只应用于字符串且在比较时会去除字符串的空格 只用于字符串字段上 |
| @Size(min,max) | 限制字符串或者集合长度必须在 min 到 max 之间 |
| @Max(value) | 限制必须为一个不大于指定值的数字 |
| @Min(value) | 限制必须为一个不小于指定值的数字 |
| @Past | 限制必须是一个过去的日期 |
| @Future | 限制必须是一个将来的日期 |
| @Email | 验证注解的元素值是Email，可以通过正则表达式和flag指定自定义的email格式 |
| @Pattern(value) | 限制必须符合指定的正则表达式 |

## 参数验证具体使用

### 1 创建需要验证的实体类及分组
```java

/**
 * @ClassName GroupA
 * @Description: 修改请联系我，否则请勿随意修改  继承Default后 该分组就相当于 GroupA + Default 两个分组的组合 即验证实体上没有加分组的也会被验证
 * @Author ChenYuan
 * @Date 2021/2/22
 * @Version V1.0
 **/
public interface GroupA extends Default {
}

public interface GroupB {
}

/**
 * 注解中没有加分组  如果使用的是spring的@Validated注解，
 * 没有加分组就默认为Default.class 分组；原始的@valid注解不支 
 * 持分组
 */
@Data
public class TestVo {
    // 多分组使用方法
    @NotNull(message = "id 不能为空",groups = GroupA.class)
    @Null(message = "id必须为空", groups = GroupB.class)
    private Integer id;

    @NotBlank(message = "name 不能为空字符串",groups = GroupA.class)
    private String name;

    @NotEmpty(message = "empty不能为空集合",groups = GroupA.class)
    private List<String> empty;

    @Max(value = 99,message = "排序最大值不能超过99")
    private int sort;
}

```

### 2 创建对应的请求接口 
在需要校验的参数上加上@valid注解或者加上@Validated 注解 备注(由于是测试所有这里不加上BindingResult 参数)

```java
/**
 * @Validated注解才支持分组验证，由于GroupA分组继承了Default，所以testVo中的sort字段也会被验证，
 * 若@Validated后未添加分组，则默认是Default分组
 */
@RestController
public class TestController {
    @GetMapping("/id")
    public TestVo getTestVo(@RequestBody @Validated(GroupA.class) TestVo vo){
        return vo;
    }
}
```

### 3 发送请求查询返回的信息
```text
GET http://localhost:8080/id
Content-Type: application/json

{
 "name" : "content",
  "sort": 55
 }

```
```text
   "errors": [
    {
      "codes": [
        "NotEmpty.testVo.empty",
        "NotEmpty.empty",
        "NotEmpty.java.util.List",
        "NotEmpty"
      ],
      "arguments": [
        {
          "codes": [
            "testVo.empty",
            "empty"
          ],
          "arguments": null,
          "defaultMessage": "empty",
          "code": "empty"
        }
      ],
      "defaultMessage": "empty不能为空集合",
      "objectName": "testVo",
      "field": "empty",
      "rejectedValue": null,
      "bindingFailure": false,
      "code": "NotEmpty"
    },
    {
      "codes": [
        "NotNull.testVo.id",
        "NotNull.id",
        "NotNull.java.lang.Integer",
        "NotNull"
      ],
      "arguments": [
        {
          "codes": [
            "testVo.id",
            "id"
          ],
          "arguments": null,
          "defaultMessage": "id",
          "code": "id"
        }
      ],
      "defaultMessage": "id 不能为空",
      "objectName": "testVo",
      "field": "id",
      "rejectedValue": null,
      "bindingFailure": false,
      "code": "NotNull"
    }
  ]

```
#### 注意事项

- @valid 这个注解是JSR-303 规范原生的验证注解 @Validated 注解是spring针对@valid 进行的一个封装，提供了一些额外的功能。
- 如果在接口上面加上了BindingResult 这个参数的话，验证后的错误信息不会抛出来，会被封装到这个类当中。 如果需要获取到验证的错误信息，需要从这个类手动当中获取。
- @Max @Min 在对包装类型进行验证的时候，如果包装类为null，是可以通过验证的，需要配合@NotNull注解一起使用
- 如果需要验证的类是作为另一个需要验证类的属性的话，必须在类上面加上@valid注解 如下所示：

```java
@Data
public class TestVo2 {

    @NotNull(message = "id 不能为空")
    private Integer id;

    @NotBlank(message = "name 不能为空字符串")
    private String name;
     //加上注解 才验证TestVo类里面的属性
    @valid  
    private List<TestVo> empty;  
}

```

### springboot参数通过切面进行统一验证返回

在测试用例当中，返回的数据格式非常不友好，通常实际情况下都是通过切面的方式，获取BindingResult 参数的数据，如果有验证错误信息，就返回给前端参数相关的错误的信息

```java
@Aspect
@Component
public class BindingResultAspect {

    @Around("execution(* cn.hjljy.springboot.validdemo.controller.*.*(..)) && args(..,bindingResult)")
    public Object validateParam(ProceedingJoinPoint joinPoint, BindingResult bindingResult) throws Throwable {
        Object obj = null;
        if (bindingResult.hasErrors()) {
            // 有校验错误  
            System.out.println(bindingResult.getAllErrors().toString());
            return "这里返回错误的信息";
        } else {
            // 没有错误方法继续执行
            obj = joinPoint.proceed();
        }
        return obj;
    }
}


```

### 自定义验证 (我没有验证过 哈哈)

当自己的验证规则比较奇特的时候，可以自定义验证

第一步： 创建自定义验证注解

```java
/**
 *
 * 注意@Constraint(validatedBy = PhoneValidator.class) 这个注解 表明具体验证规则在PhoneValidator类里面
 */
@Constraint(validatedBy = PhoneValidator.class)
@Target({ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Phone {
    String message() default "手机号格式不合法";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}

```

第二步创建具体验证PhoneValidator类

```java
/**
 * @date 2020/5/11 17:52
 * @apiNote 手机号码验证
 */
public class PhoneValidator implements ConstraintValidator<Phone,String> {
    private Pattern pattern = Pattern.compile("1(([38]\\d)|(5[^4&&\\d])|(4[579])|(7[0135678]))\\d{8}");
    @Override
    public boolean isValid(String s, ConstraintValidatorContext constraintValidatorContext) {
        if(s!=null){
            return pattern.matcher(s).matches();
        }
        return true;
    }
}   

```

第三步：和其他的验证注解一样使用即可

```java
 @Phone
private String phone;
```

### 参考文章

[Springboot之分组验证以及自定义参数验证](https://blog.csdn.net/ycf921244819/article/details/106059751/)
