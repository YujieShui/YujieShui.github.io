---
title: SpringBoot走向自动装配
toc: true
categories:
  - Java
  - SpringBoot
tags:
  - springboot
abbrlink: 9ba90caf
date: 2019-09-28 15:25:55
---

# Spring 模式注解装配

- 定义：一种用于声明在应用中扮演“组件”角色的注解
- 举例：@Component、@Service、@Configuration
- 装配：<context:component-sacn>或@ComponentScan

[模式注解（Stereotype Annotations）](https://github.com/spring-projects/spring-framework/wiki/Spring-Annotation-Programming-Model#stereotype-annotations)

> A stereotype annotation is an annotation that is used to declare the role that a component plays within the application. For example, the `@Repository` annotation in the Spring Framework is a marker for any class that fulﬁlls the role or stereotype of a repository (also known as Data Access Object or DAO).
>
> `@Component` is a generic stereotype for any Spring-managed component. Any component annotated with `@Component` is a candidate for component scanning. Similarly, any component annotated with an annotation that is itself meta-annotated with `@Component` is also a candidate for component scanning. For example, `@Service` is meta-annotated with `@Component` .

模式注解是一种用于声明在应用中扮演「组件」角色的注解。如 Spring Framework 中的`@Repository` 标注在任何类上，用于扮演仓储角色的模式注解。

`@Component` 作为一种由 Spring 容器托管的通用模式组件，任何被`@Component` 标准的组件均为组件扫描的候选对象。类似地，凡是被 `@Component` 元标注（**meta-annotated**）的注解，如 `@Service` ，当任何组件标注它时，也被视作组件扫 描的候选对象。

## 模式注解举例

| Spring Framework 注解 | 场景说明           | 起始版本 |
| --------------------- | ------------------ | -------- |
| `@Repository`         | 数据仓储模式注解   | 2.0      |
| `@Component`          | 通用组件模式注解   | 2.5      |
| `@Service`            | 服务模式注解       | 2.5      |
| `@Controller`         | Web 控制器模式注解 | 2.5      |
| `@Configuration`      | 配置类模式注解     | 3.0      |

## 装配方式

### **`<context:component-scan>` 方式**

```xml
<?xml version="1.0" encoding="UTF-8"?> <beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context" xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/springcontext.xsd">

<!-- 激活注解驱动特性 --> <context:annotation-config />

<!-- 找寻被 @Component 或者其派生 Annotation 标记的类（Class），将它们注册为 Spring Bean --> <context:component-scan base-package="com.imooc.dive.in.spring.boot" />

</beans>
```

### **`@ComponentScan` 方式**

```java
@ComponentScan(basePackages = "com.shuiyujie.dive.in.spring.boot") 
public class SpringConfiguration { 
	...
}
```

## 自定义模式注解

### **@Component “派生性”**

```java
/** * 一级 {@link Repository @Repository} * * @author <a href="mailto:mercyblitz@gmail.com">Mercy</a> * @since 1.0.0 */ @Target({ElementType.TYPE}) @Retention(RetentionPolicy.RUNTIME) @Documented @Repository public @interface FirstLevelRepository {
	String value() default "";
}
```

- @Component
  - @Repository
    - FirstLevelRepository 

### **@Component “层次性”**

```java
@Target({ElementType.TYPE}) @Retention(RetentionPolicy.RUNTIME) @Documented @FirstLevelRepository public @interface SecondLevelRepository {
	String value() default "";
}
```

- @Component
  - @Repository
    - FirstLevelRepository
      - SecondLevelRepository



# @Enable 模块装配

- 定义：具有相同领域的功能组件集合，组合锁形成一个独立的单元
- 举例：@EnableWebMVC、@EnableAutoConfiguration 等
- 实现：注解方法、编程方式

Spring Framework 3.1 开始支持`@Enable`模块驱动“。所谓「模块」是指具备相同领域的功能组件集合， 组合所形成一个独立 的单元。比如 Web MVC 模块、AspectJ代理模块、Caching（缓存）模块、JMX（Java 管 理扩展）模块、Async（异步处 理）模块等。

## `@Enable`注解模块举例

| 框架实现         | @Enable 注解模块               | 激活模块            |
| ---------------- | ------------------------------ | ------------------- |
| Spring Framework | @EnableWebMvc                  | Web MVC 模块        |
|                  | @EnableTransactionManagement   | 事务管理模块        |
|                  | @EnableCaching                 | Caching 模块        |
|                  | @EnableMBeanExport             | JMX 模块            |
|                  | @EnableAsync                   | 异步处理模块        |
|                  | EnableWebFlux                  | Web Flux 模块       |
|                  | @EnableAspectJAutoProxy        | AspectJ 代理模块    |
|                  |                                |                     |
| Spring Boot      | @EnableAutoConfiguration       | 自动装配模块        |
|                  | @EnableManagementContext       | Actuator 管理模块   |
|                  | @EnableConfigurationProperties | 配置属性绑定模块    |
|                  | @EnableOAuth2Sso               | OAuth2 单点登录模块 |
|                  |                                |                     |
| Spring Cloud     | @EnableEurekaServer            | Eureka服务器模块    |
|                  | @EnableConfigServer            | 配置服务器模块      |
|                  | @EnableFeignClients            | Feign客户端模块     |
|                  | @EnableZuulProxy               | 服务网关 Zuul 模块  |
|                  | @EnableCircuitBreaker          | 服务熔断模块        |

## 实现方式

### 注解驱动方式

### 接口编程方式





## 自定义`@Enable`模块



### 基于注解驱动实现 - `@EnableHelloWorld`



### 基于接口驱动实现 - `@EnableServer`











# 条件装配

- 定义：Bean 装配的前置判断
- 举例：@Peofile、@Condition
- 实现：注解方式、编程方式

