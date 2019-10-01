---
title: "【Java并发编程】Java\_中如何创建线程？"
toc: true
categories:
- Java
- Java并发编程
tags:
  - java
  - 并发编程
abbrlink: 714434fb
date: 2019-09-05 22:21:53
---

![Java创建线程](http://image.shuiyujie.com/64354575_p0_master1200.jpg)

1. 继承Thread实现线程创建
2. 实现Runnable接口
3. 实现Callable接口，结合 FutureTask使用
4. 利用该线程池ExecutorService、Callable、Future来实现

<!-- more -->

# 继承  Thread 类



```java
public class AddThread extends Thread {

    private int start, end;
    private int sum = 0;

    public AddThread(String name, int start, int end) {
        super(name);
        this.start = start;
        this.end = end;
    }

    public void run() {
        System.out.println("Thread-" + getName() + " 开始执行!");
        for (int i = start; i <= end; i ++) {
            sum += i;
        }
        System.out.println("Thread-" + getName() + " 执行完毕! sum=" + sum);
    }

    public static void main(String[] args) throws InterruptedException {
        int start = 0, mid = 5, end = 10;

        AddThread thread1 = new AddThread("线程1", start, mid);
        AddThread thread2 = new AddThread("线程2", mid + 1, end);

        thread1.start();
        thread2.start();

        // 确保两个线程执行完毕
        thread1.join();
        thread2.join();

        int sum = thread1.sum + thread2.sum;
        System.out.println("sum: " + sum);
    }
}
```

输出结果

```
Thread-线程1 开始执行!
Thread-线程1 执行完毕! sum=15
Thread-线程2 开始执行!
Thread-线程2 执行完毕! sum=40
sum: 55
```



# 实现Runnable接口



```java
public class AddRun implements Runnable {

    private int start, end;
    private int sum = 0;

    public AddRun(int start, int end) {
        this.start = start;
        this.end = end;
    }

    public void run() {
        System.out.println(Thread.currentThread().getName() + " 开始执行!");
        for(int i = start; i <= end; i++) {
            sum += i;
        }
        System.out.println(Thread.currentThread().getName() + " 执行完毕! sum=" + sum);
    }

    public static void main(String[] args) throws InterruptedException {
        int start = 0, mid = 5, end = 10;
        AddRun run1 = new AddRun(start, mid);
        AddRun run2 = new AddRun(mid + 1, end);
        Thread thread1 = new Thread(run1, "线程1");
        Thread thread2 = new Thread(run2, "线程2");

        thread1.start();
        thread2.start();

        thread1.join();
        thread2.join();
        int sum = run1.sum + run2.sum;
        System.out.println("sum: " + sum);
    }
}
```



# 实现Callable接口，结合FutureTask创建线程



```java
public class AddCall implements Callable<Integer> {

    private int start, end;
    private int sum = 0;

    public AddCall(int start, int end) {
        this.start = start;
        this.end = end;
    }

    public Integer call() throws Exception {
        int sum = 0;
        System.out.println(Thread.currentThread().getName() + " 开始执行!");
        for (int i = start; i <= end; i++) {
            sum += i;
        }
        System.out.println(Thread.currentThread().getName() + " 执行完毕! sum=" + sum);
        return sum;
    }

    public static void main(String[] args) 
        throws ExecutionException, InterruptedException {
        
        int start = 0, mid = 5, end = 10;
        FutureTask<Integer> future1 = new FutureTask<Integer>(new AddCall(start, mid));
        FutureTask<Integer> future2 = new FutureTask<Integer>(new AddCall(mid + 1, end));

        Thread thread1 = new Thread(future1, "线程1");
        Thread thread2 = new Thread(future2, "线程2");

        thread1.start();
        thread2.start();

        int sum1 = future1.get();
        int sum2 = future2.get();

        System.out.println("sum = " + (sum1 + sum2));

    }
}
```



# 线程池方式创建



```java
public class AddPool implements Callable<Integer> {
    private int start, end;

    public AddPool(int start, int end) {
        this.start = start;
        this.end = end;
    }

    @Override
    public Integer call() throws Exception {
        int sum = 0;
        System.out.println(Thread.currentThread().getName() + " 开始执行!");
        for (int i = start; i <= end; i++) {
            sum += i;
        }
        System.out.println(Thread.currentThread().getName() + " 执行完毕! sum=" + sum);
        return sum;
    }

    public static void main(String[] arg) 
        throws ExecutionException, InterruptedException {
        
        int start=0, mid=500, end=1000;
        ExecutorService executorService = Executors.newFixedThreadPool(2);
        Future<Integer> future1 = executorService.submit(new AddPool(start, mid));
        Future<Integer> future2 = executorService.submit(new AddPool(mid+1, end));

        int sum = future1.get() + future2.get();
        System.out.println("sum: " + sum);
    }
}
```



# 参考资料

[Java并发学习之四种线程创建方式的实现与对比](https://my.oschina.net/u/566591/blog/1576410)