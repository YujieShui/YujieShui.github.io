---
title: 理解SpringApplication
toc: true
categories:
  - Java
  - SpringBoot
tags:
  - springboot
  - spring application
abbrlink: b73d7745
date: 2019-09-28 19:25:02
---

Spring Framework??

Spring Application??

> https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-spring-application.html

基础技术

- Spring Framework
  - Spring 模式注解
  - Spring 应用上下文
  - Spring 工厂加载机制
  - Spring 应用上下文初始器
  - Spring Environment 抽象
  - Spring 应用事件/监听

衍生技术

- Spring Boot
  - SpringApplication
  - SpringApplication Builder API
  - SpringApplication 运行监听器
  - SpringApplication 参数
  - SpringBoot 应用事件/监听器



# SpringApplication

- 定义：Spring 应用引导类，提供便利的自定义行为方法
- 场景：嵌入式 Web 应用和非 Web 应用
- 运行：SpringApplication#run(String...)

## 自定义 SpringApplication

### 通过 SpringApplication API 调整

```java
SpringApplication springApplication = new SpringApplication(DiveInSpringBootApplication.class); springApplication.setBannerMode(Banner.Mode.CONSOLE); springApplication.setWebApplicationType(WebApplicationType.NONE); springApplication.setAdditionalProfiles("prod"); springApplication.setHeadless(true);
```

### 通过 SpringApplicationBuilder API 调整

相比于使用 SpringApplication API，SpringApplicationBuilder API 使用起来更加流畅。

> If you need to build an `ApplicationContext` hierarchy (multiple contexts with a parent/child relationship) or if you prefer using a “fluent” builder API, you can use the `SpringApplicationBuilder`.
>
> The `SpringApplicationBuilder` lets you chain together multiple method calls and includes `parent` and `child` methods that let you create a hierarchy, as shown in the following example:

```java
new SpringApplicationBuilder(DiveInSpringBootApplication.class) 
  .bannerMode(Banner.Mode.CONSOLE) 
  .web(WebApplicationType.NONE) 
  .profiles("prod") 
  .headless(true) 
  .run(args);
```



# SpringApplication 准备阶段

- 配置：Spring Bean 来源
- 推断：Web 应用类型和引导类（Main class）
- 加载：应用上下文初始化器和应用事件监听器

## 配置 SpringBoot Bean 源

Java 配置 Class 或 XML 上下文配置文件集合，用于 Spring Boot `BeanDefinitionLoader` 读取 ，并且将配置源解析加载为 Spring Bean 定义。

### Java 配置 class

用于 Spring 注解驱动中 Java 配置类，大多数情况是 Spring 模式注解所标注的类，如 `@Configuration`。

```java
@SpringBootApplication
public class SpringApplicationBootstrap {
    public static void main(String[] args) {
				SpringApplication.run(SpringApplicationBootstrap.class,args);
    }
}
```

### XML 上下文配置文件

用于 Spring 传统配置驱动中的 XML 文件。

```java
public class SpringApplicationBootstrap {

    public static void main(String[] args) {
//        SpringApplication.run(ApplicationConfiguration.class,args);

        Set<String> sources = new HashSet();
        // 配置Class 名称
        sources.add(ApplicationConfiguration.class.getName());
        SpringApplication springApplication = new SpringApplication();
      	// Source支持多种数据源，可以是class、xml
        springApplication.setSources(sources);
        springApplication.run(args);

    }
		
  	// springApplication.run()中只要是一个 @SpringBootApplication 的 bean 即可
    @SpringBootApplication
    public static class ApplicationConfiguration {

    }
}
```

# 推断 Web 应用类型

根据当前应用 ClassPath 中是否存在相关实现类来推断 Web 应用的类型，包括： 

- Web Reactive： `WebApplicationType.REACTIVE`
- Web Servlet： `WebApplicationType.SERVLET` 
- 非 Web： `WebApplicationType.NONE`

> 参考方法： `org.springframework.boot.SpringApplication#deduceWebApplicationType`
>
> ```java
> private WebApplicationType deduceWebApplicationType() {
> 	if (ClassUtils.isPresent(REACTIVE_WEB_ENVIRONMENT_CLASS, null) 
>       		&& !ClassUtils.isPresent(MVC_WEB_ENVIRONMENT_CLASS, null)) { 
>     return WebApplicationType.REACTIVE; 
>   } 
>   
>   for (String className : WEB_ENVIRONMENT_CLASSES) {
>     if (!ClassUtils.isPresent(className, null)) {
>       return WebApplicationType.NONE;
>     } 
>   } 
>   return WebApplicationType.SERVLET;
> }
> ```

# 推断引导类（Main Class）

根据 Main 线程执行堆栈判断实际的引导类

> 参考方法： `org.springframework.boot.SpringApplication#deduceMainApplicationClass`
>
> ```java
> private Class<?> deduceMainApplicationClass() { 
>   try {
>     StackTraceElement[] stackTrace = new RuntimeException().getStackTrace(); 
>     for (StackTraceElement stackTraceElement : stackTrace) {
> 			if ("main".equals(stackTraceElement.getMethodName())) {
> 				return Class.forName(stackTraceElement.getClassName());
>       } 
>     }
> 	} catch (
>     ClassNotFoundException ex) { 
>     // Swallow and continue 
>   } 
>   return null;
> }
> ```





# 运行阶段







# 课堂总结

