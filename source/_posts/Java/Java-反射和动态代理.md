---
title: Java 反射和动态代理
toc: true
categories:
  - Java
tags:
  - java
abbrlink: e0acc964
date: 2017-11-26 23:08:39
---

反射提供了一种机制——用来检测可用的方法，并返回方法名。通过类名我们就能获取 Class 对象，并通过 Class 对象获取对象相关的一些属性和方法等。

最常见到的地方就是在各种配置文件中的应用，它的使用就像**传入一个字符串，然后字符串去获取了相应的对象。**

<!-- more -->

# 反射

在了解反射之前要知道组成 Java 程序的是一个个类，每一个类都有一个 `Class 对象`，他对应一个`.class`文件。

所有的类都是在被第一次被调用的时候加载到 JVM 中的，并且一个类的也不是一次性加载完成，是在用到某一部分的时候才加载的。比如一个类被加载的时候一定会先加载静态变量，所以我们把常量定义为静态的。

*注：以上内容整理自《Java 编程思想》。*

我们用反射的第一步就是要获取`Class 对象`，再去获取他的其他属性。可以使用`Class.forName()`方法获取。

```java
public class MyReflect {

    public String className = null;

    @SuppressWarnings("rawtypes")
    public Class dogClass = null;


    /**
     * 反射获取 Dog 类
     *
     * @throws ClassNotFoundException
     */
    @Before
    public void init() throws ClassNotFoundException {
        className = "com.shuiyujie.reflection.Dog";
        dogClass = Class.forName(className);
    }
```

获取`Class 对象`之后，我们可以通过它创建一个实例对象

```java
/**
     * 获得对象实例，调用无参构造
     *
     * @throws IllegalAccessException
     * @throws InstantiationException
     */
    @Test
    public void newInstance() throws IllegalAccessException, InstantiationException {
        System.out.println(dogClass.newInstance());
    }
```

还可以通过它获取对象的构造方法、成员变量、成员方法等，方法又有公有和非公有之分，非公有的如果我们要强制获取要消除他的类型校验，**这就是反射 API 要知道的大部分内容了**，示例代码如下：

```java
 /**
     * 获取公共的成员变量
     * @throws Exception
     */
    @SuppressWarnings({ "rawtypes", "unchecked" })
    @Test
    public void getPublicParams() throws Exception {
        Constructor constructor = dogClass.getConstructor(Long.class,String.class);
        Object obj = constructor.newInstance(1L, "Lucky");

        Field field = dogClass.getField("name");
        field.set(obj,"Lucy");
        System.out.println(field.get(obj));
    }

    /**
     * 获取非公共的成员变量
     * @throws Exception
     */
    @SuppressWarnings({ "rawtypes", "unchecked" })
    @Test
    public void getUnPublicParams() throws Exception {
        Constructor constructor = dogClass.getConstructor(Long.class,String.class);
        Object obj = constructor.newInstance(1L, "Lucky");

        Field field = dogClass.getDeclaredField("id");
        field.setAccessible(true);
        field.set(obj,3L);
        System.out.println(field.get(obj));
    }


    /**
     * 获取公共的成员方法
     * @throws Exception
     */
    @SuppressWarnings({ "rawtypes", "unchecked" })
    @Test
    public void getPublicMethod() throws Exception {
        System.out.println(dogClass.getMethod("toString"));
        Object obj = dogClass.newInstance();
        Method method = dogClass.getMethod("brak");
        method.invoke(obj);
    }

    /**
     * 获取非公共的成员方法
     * @throws Exception
     */
    @SuppressWarnings({ "rawtypes", "unchecked" })
    @Test
    public void getUnPublicMethod() throws Exception {
        Object obj = dogClass.newInstance();
        Method method = dogClass.getDeclaredMethod("dogBrak");
        method.setAccessible(true);
        method.invoke(obj);
    }

```

此外他还有一些其他方法示例，更多可以参考 Class 类的 API

```java
@Test
    public void otherMethod() throws ClassNotFoundException {
        //当前加载这个class文件的那个类加载器对象
        System.out.println(dogClass.getClassLoader());
        //获取某个类实现的所有接口
        Class[] interfaces = dogClass.getInterfaces();
        for (Class class1 : interfaces) {
            System.out.println(class1);
        }
        //反射当前这个类的直接父类
        System.out.println(dogClass.getGenericSuperclass());
        //判断当前的Class对象表示是否是数组
        System.out.println(dogClass.isArray());
        System.out.println(new String[3].getClass().isArray());
        //判断当前的Class对象表示是否是枚举类
        System.out.println(dogClass.isEnum());
        System.out.println(Class.forName("com.shuiyujie.reflection.Animal").isEnum());
        //判断当前的Class对象表示是否是接口
        System.out.println(dogClass.isInterface());
        System.out.println(Class.forName("com.shuiyujie.reflection.AnimalInterface").isInterface());
    }
```

# 动态代理

动态代理指的是在不修改原业务的基础上，基于原业务方法，进行重新的扩展，实现新的业务。实现动态代理的核心就是反射机制。

用一个场景来描述动态代理。比如一个订单系统，有一个生成订单的 service，当我们要搞活动进行促销的时候，生成订单的价格根据活动降低价格。

修改原有的生成订单的方法无疑是不可行的，因为系统的其他模块可能也调用了这个方法，甚至活动有多种的优惠方式只改一个也不够。**这里就可以用动态代理来代理订单生成的方法。**

代理我们也可以叫他中介，比如房产中介就是在客户之间起桥梁作用，动态代理就是在调用发和被调用方中间介入。

首先要有订单的接口和实现：

```java
public interface IOrderService {
    int getClothOrder(String size);
}

public class OrderServiceImpl implements IOrderService {
    private int clothPrize = 500;
    @Override
    public int getClothOrder(String size) {
        System.out.println("衣服的尺码为" + size);
        return clothPrize;
    }
}
```

用一个 Controller 调用他获得价格：

```java
public class OrderController { 
    @Test
    public void getOrder(){
        IOrderService orderService = new OrderServiceImpl();
        int price = orderService.getClothOrder("L");
        System.out.println("价格为" + price + "元");
    }
```

输出：

```
衣服的尺码为L
价格为500元
```

在`IOrderService`和`OrderServiceImpl`不变的情况下通过`IOrderService`来实现打折的效果，就要用到动态代理了。

**动态代理类：**

```java
public class ProxyCharge {
    public static <T> T getProxy(final int discountCoupon,
                                 final Class<?> interfaceClass, final Class<?> implementsClass)
            throws Exception {
        return (T) Proxy.newProxyInstance(interfaceClass.getClassLoader(),
                new Class[]{interfaceClass}, (proxy, method, args) -> {
                    Integer returnValue = (Integer) method.invoke(
                            implementsClass.newInstance(), args);
                    return returnValue - discountCoupon;
                });
    }
}
```

关键是通过`Proxy.newProxyInstance()`获取一个原来`OrderServiceImpl`的代理对象。我们需要传入借口对象和实现对象。

获取代理对象之后可以使用反射中的`invoke()`调用原先方法，也能进行自己扩展。

**代理方法调用：**

```java
    @Test
    public void getChargeOrder() throws Exception {
        IOrderService proxyService = ProxyCharge.getProxy(200,IOrderService.class,OrderServiceImpl.class);
        int price = proxyService.getClothOrder("L");
        System.out.println("价格为" + price + "元");
    }
```

输出：

```java
衣服的尺码为L
价格为300元
```

# 总结：

反射允许更加动态的编程风格，动态代理可以动态地获取代理对象并动态地处理对所代理对象的调用。

动态代理听上去很复杂但是他的实现方式是基本固定的，以上的示例代码是最简单的实现。同时动态代理是实现 RPC 框架不可或缺的一部分，现在温习一下为实现一个 RPC 框架做准备。


