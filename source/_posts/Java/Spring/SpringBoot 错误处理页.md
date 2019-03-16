---
title: SpringBoot 错误处理页
categories:
  - Java
tags:
  - java
  - springboot
abbrlink: 3c98ab20
date: 2019-02-19 11:03:47
---

SpringBoot 错误处理有三种情况
- 数据验证错误
- 错误页指派
- 全局异常处理

比如说在提交表单信息的时候，有些信息没填，有些信息有规范的格式，如果不符合要求就是数据校验错误。

我们需要对这些数据的格式进行校验，不符合要求不能接受且要提示错误信息。

![验证注解](http://image.shuiyujie.com/2019-02-12-15-48-04.png)

[点击查看源码](https://github.com/YujieShui/springboot-learning/tree/master/springboot-error)

<!-- more -->

# springboot 错误处理，数据校验错误

SpringBoot 有数据校验的默认支持，该支持由 Hibernate 开发框架提供。

错误信息统一配置在`ValidationMessages.properties`文件中。

这种方式每个 VO 类在`ValidationMessages.properties`文件中配置对应的错误信息，并且在 VO 类的属性上添加上错误注解，过程繁琐。不推荐使用，更推荐使用反射和拦截器的方式处理错误信息。

但是我们还是要会使用这种方式处理数据验证错误，写了一个小 Demo，再接下来简单记录一下使用过程。

## 添加错误信息配置文件

`src/main/resources`目录中建立`ValidationMessages.properties`文件。文件中配置错误信息

```
member.mid.notnull.error=邮箱不允许为空
member.mid.email.error=邮箱格式错误
member.mid.length.error=邮箱长度错误
member.age.notnull.error=年龄不允许为空
member.age.digits.error=年龄格式错误
member.salary.notnull.error=工资不允许为空
member.salary.digits.error=工资格式错误
member.birthday.notnull.error=生日不允许为空
```

## VO 类添加注解

```java
SuppressWarnings("serial")
public class Member implements Serializable {
    @NotNull(message="{member.mid.notnull.error}")
    @Email(message="{member.mid.email.error}")
    @Length(min=6,message="{member.mid.length.error}")
    private String mid ;
    @NotNull(message="{member.age.notnull.error}")
    @Digits(integer=3,fraction=0,message="{member.age.digits.error}")
    private Integer age ;
    @NotNull(message="{member.salary.notnull.error}")
    @Digits(integer=20,fraction=2,message="{member.salary.digits.error}")
    private Double salary ;
    @NotNull(message="{member.birthday.notnull.error}")
    private Date birthday ;

}
```

## 控制器配置校验

```java
@RequestMapping(value = "/add", method = RequestMethod.POST)
@ResponseBody
public Object add(@Valid Member vo, BindingResult result) {
    if (result.hasErrors()) {
        Iterator<ObjectError> iterator = result.getAllErrors().iterator();
        while (iterator.hasNext()) {
            ObjectError error = iterator.next();
            System.out.println("【错误信息】code = " + error.getCode() + "，message = " + error.getDefaultMessage());
        }
        return result.getAllErrors();
    } else {
        return vo;
    }
}
```

# springboot 错误处理，配置错误页面

错误页面配置在`src/main/view/static`目录下，比如叫 error-404.html

## SringBoot 1.x 这样处理

```java
package com.shuiyujie.config;

import org.springframework.boot.context.embedded.ConfigurableEmbeddedServletContainer;
import org.springframework.boot.context.embedded.EmbeddedServletContainerCustomizer;
import org.springframework.boot.web.servlet.ErrorPage;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpStatus;

/**
 * @author shui
 * @create 2019-02-12
 **/
@Configuration
public class ErrorPageConfig {

    @Bean
    public EmbeddedServletContainerCustomizer containerCustomizer() {
        return new EmbeddedServletContainerCustomizer() {
            @Override
            public void customize(
                    ConfigurableEmbeddedServletContainer container) {
                ErrorPage errorPage400 = new ErrorPage(HttpStatus.BAD_REQUEST,
                        "/error-400.html");
                ErrorPage errorPage404 = new ErrorPage(HttpStatus.NOT_FOUND,
                        "/error-404.html");
                ErrorPage errorPage500 = new ErrorPage(
                        HttpStatus.INTERNAL_SERVER_ERROR, "/error-500.html");
                container.addErrorPages(errorPage400, errorPage404,
                        errorPage500);
            }
        };
    }
}
```

## SringBoot 2.x 这样处理

在SpringBoot2中没有`EmbeddedServletContainerCustomizer`这个类了，要这样处理

```java
ackage com.shuiyujie.config;

import org.springframework.boot.web.server.ErrorPage;
import org.springframework.boot.web.server.ErrorPageRegistrar;
import org.springframework.boot.web.server.ErrorPageRegistry;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Component;

/**
 * @author shui
 * @create 2019-02-12
 **/
@Component
public class ErrorPageConfig implements ErrorPageRegistrar {
    @Override
    public void registerErrorPages(ErrorPageRegistry registry) {
        ErrorPage[] errorPages = new ErrorPage[3];
        errorPages[0] = new ErrorPage(HttpStatus.NOT_FOUND, "/error-400.html");
        errorPages[1] = new ErrorPage(HttpStatus.NOT_FOUND, "/error-404.html");
        errorPages[2] = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/error-500.html");
        registry.addErrorPages(errorPages);
    }
}
```

# SpringBoot 全局错误处理

之前设置了错误页面，发生错误将会跳转到错误页面。

```java
/**
 * 演示 500 错误
 *
 * @return
 */
@RequestMapping(value="/get")
@ResponseBody
public String get() {
    System.out.println("除法计算：" + (10 / 0));
    return "hello world" ;
}
```

比如这样一个 10/0 的错误控制器跳转到 500 页面，这样就看不到详细的报错信息了。我们想看到类似控制台里面更加详细的报错信息。

## 定义一个错误信息处理的页面

新建一个错误界面`src/main/view/templates/error.html`

```html
!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
   <title>SpringBoot模版渲染</title>
   <link rel="icon" type="image/x-icon" href="/images/mldn.ico"/>
   <meta http-equiv="Content-Type" content="text/html;charset=UTF-8"/>
</head>
<body>
   <p th:text="${url}"/>
   <p th:text="${exception.message}"/>
</body>
</html>
```

## 定义全局异常处理类

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    // 定义错误显示页，error.html
    public static final String DEFAULT_ERROR_VIEW = "error";

    // 所有的异常都是Exception子类
    @ExceptionHandler(Exception.class)
    public Object defaultErrorHandler(HttpServletRequest request, Exception e) {
        class ErrorInfo {
            private Integer code;
            private String message;
            private String url;

            public Integer getCode() {
                return code;
            }

            public void setCode(Integer code) {
                this.code = code;
            }

            public String getMessage() {
                return message;
            }

            public void setMessage(String message) {
                this.message = message;
            }

            public String getUrl() {
                return url;
            }

            public void setUrl(String url) {
                this.url = url;
            }
        }
        ErrorInfo info = new ErrorInfo();
        // 标记一个错误信息类型
        info.setCode(100);
        info.setMessage(e.getMessage());
        info.setUrl(request.getRequestURL().toString());
        return info;
    }
}
```

