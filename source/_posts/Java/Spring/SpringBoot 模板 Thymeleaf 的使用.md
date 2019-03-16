---
title: SpringBoot 模板 Thymeleaf 的使用
categories:
  - Java
tags:
  - java
  - springboot
abbrlink: 41479a53
date: 2019-02-19 11:10:47
---

Java 开发行业有三种常用显示模板
* FreeMarker
* Velocity
* Thymeleaf（推荐使用）

本项目是使用 Thymeleaf 模板的简单 Demo

[点击查看源码](https://github.com/YujieShui/springboot-learning/tree/master/springboot-thymeleaf)

<!-- more -->

# SpringBoot 模板 Thymeleaf 的使用

pom.xml 中添加依赖

```
<!-- 配置使用 thymeleaf 模板-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```
 
控制层进行信息显示

```java
@Controller
public class MessageController extends AbstractBaseController {

    @RequestMapping(value = "/show", method = RequestMethod.GET)
    public String show(String mid, Model model) {
        // request属性传递包装
        model.addAttribute("url", "www.shuiyujie.com");
        // request属性传递包装
        model.addAttribute("mid", mid);
        // 此处只返回一个路径， 该路径没有设置后缀，后缀默认是*.html
        return "message/message_show";
    }
}
```

由于我们使用的是`@Controller`注解，所以此时`return "message/message_show"`将进行一次路由，路由到的文件就是 Thymeleaf 模板文件。

# Thymeleaf 模板文件位置配置

文件位置的配置很重要，要按照规范进行配置。

首先我们要建立一个**Resources类型目录**，`Resources`目录就是源代码目录，这个是可以通过 IDE 进行设置。

我是这样设置的：

- src/main/view 目录下存放页面
- src/main/view/static 目录下存放 js,css,images 等文件
- src/main/view/templates 目录下存放 html 页面

*注: static 静态目录下的文件可以直接访问，而不需要通过控制器进行路由。*

http://localhost:8080/show
通过路由访问 message_show.html 页面

http://localhost:8080/message_index.html
直接访问 message_index.html 页面

