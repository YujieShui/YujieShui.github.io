---
title: SpringBoot核心特性
toc: true
abbrlink: dcf57196
date: 2019-09-28 11:32:15
categories:
- Java
- SpringBoot
tags:
- springboot
---



- 组件自动装配：Web MVC、Web Flux、JDBC 等
- 嵌入式 Web 容器：Tomcat、Jetty 以及 Undertow
  - Web Servlet:Tomcat、Jetty 和 Undertow
  - Web Reactive：Netty Web Server
- 生产准备特性：指标、健康检查、外部化配置等
  - 指标：/actuator/metrics
  - 健康检查：/actuator/health
  - 外部化配置：/actuator/configprops

<!-- more -->

# 组件自动装配

组件自动装配：Web MVC、Web Flux、JDBC 等

- 激活：@EnableAutoConfiguration （SpringBoot 默认没有激活自动装配）
- 配置：/META-INF/spring.factoroes
- 实现：XXXAutoConfiguration

> https://docs.spring.io/spring-boot/docs/2.0.1.RELEASE/reference/htmlsingle/#boot-features-spring-mvc-auto-configuration

在`https://start.spring.io/`创建一个 SpringBoot 的项目，下载到本地并导入到 IDEA 中。

![springboot start](http://image.shuiyujie.com/2019-09-28-12-43-48.png)

来看一下这个项目，首先有两个依赖， 一个是 `spring-boot-starter` 另一个是 `spring-boot-starter-test`，启动类使用一个 `@SpringBootApplication` 注解，我们看它的源码可以看到它使用了 `@EnableAutoConfiguration`，所以也能实现组件的自动装配。

## 传统 Servlet

首先导入 web 模块，再启动工程。

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

```shell
# 启动日志中说我们使用的是 tomcat 容器，端口在 8080 端口
# 此时访问会得到一个提示页，因为没有配置映射

2019-09-28 12:53:19.261  INFO 56919 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
```

所以我们要使用 Servlet：

```java
// 定义一个类继承 HttpServlet
// 使用 @WebServlet 并用 urlPatterns 指定访问路径
@WebServlet(urlPatterns = "/my/servlet")
public class MyServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().println("Hello World!");
    }
}

// 注册：@ServletComponentScan 
@SpringBootApplication
@ServletComponentScan(basePackages = "com.shuiyujie.diveinspringboot.web.servlet")
public class DiveInSpringBootApplication {

	public static void main(String[] args) {
		SpringApplication.run(DiveInSpringBootApplication.class, args);
	}

}
```

# 异步非阻塞



```java
@WebServlet(urlPatterns = "/my/servlet")
public class MyServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        AsyncContext asyncContext = req.startAsync();
        asyncContext.start(()->{
            try {
                resp.getWriter().println("Hello World!");
            } catch (IOException e) {
                e.printStackTrace();
            }

        });
    }
}
```

此时会发生`java.lang.IllegalStateException: A filter or servlet of the current chain does not support asynchronous operations.`的错误，这是因为`@WebServlet`默认是不是异步的，可以看一下它的源码：

```java
public @interface WebServlet {
    ...
    // 默认是 false，要实现异步就要将其打开
    boolean asyncSupported() default false;
```

此外执行完毕之后还需要触发完成 `asyncContext.complete();`，最后变成这样：

```java
@WebServlet(urlPatterns = "/my/servlet", asyncSupported = true)
public class MyServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        AsyncContext asyncContext = req.startAsync();
        asyncContext.start(() -> {
            try {
                resp.getWriter().println("Hello World!");
                // 触发完成
                asyncContext.complete();
            } catch (IOException e) {
                e.printStackTrace();
            }
        });
    }
}
```





