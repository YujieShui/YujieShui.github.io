---
title: SpringBoot 配置文件及其读取
toc: true
categories:
  - Java
tags:
  - java
  - springboot
abbrlink: f0ba9e07
date: 2019-02-19 11:00:47
---

 在配置文件中统一管理配置信息，使用配置文件 SpringBoot 有规范的方式。

 `src/main/resources` 的 classpath 路径之中，创建`application.properties`配置文件。

 配置文件也可以用 YAML 语言来写，创建`application.yml`配置文件

 两个文件同时存在都会起作用，但当配置项冲突时，优先使用`application.properties`文件

 [点击获取项目源码](https://github.com/YujieShui/springboot-learning/tree/master/spring-messagesource)

 <!-- more -->

# 资源文件的设置 

 资源文件统一放在`src/main/resources/i18n`目录中

 - 建立 Messages.properties

```
welcome.url=www.shuiyujie.com
welcome.msg=shuiyujie
```

- 建立 Pages.properties

```
member.add.page=/pages/back/admin/member/member_add.jsp
member.add.action=/pages/back/admin/member/member_add.action
```

- 配置文件 application.yml 中指定资源文件目录

```
spring: # 表示该配置直接为Spring容器负责处理
    messages: # 表示进行资源配置
        basename: i18n/Messages,i18n/Pages # 资源文件的名称
server:
    port: 80 # 此处设置的服务的访问端口配置
```

# 资源文件的读取

经过以上配置就会自动生成一个`MessageSource`资源文件对象，我们只要注入这个对象就能使用资源文件中配置的属性了。

我会建立一个控制器的父类在其中注入`MessageSource`，然后子类继承父类就能很方便地读取资源文件了

```java
public abstract class AbstractBaseController {
    @Resource
    private MessageSource messageSource; // 自动注入此资源对象
    public String getMessage(String key, String... args) {
        return this.messageSource.getMessage(key, args, Locale.getDefault());
    }
}
```

```java
@RestController
public class MessageController extends AbstractBaseController {
    @RequestMapping(value = "/echo", method = RequestMethod.GET)
    public String echo(String mid) {
        System.out.println("【*** 访问地址 ***】" + super.getMessage("member.add.action"));
        return super.getMessage("welcome.msg", mid);
    }
}
```

