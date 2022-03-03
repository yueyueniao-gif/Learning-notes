---
title: Java多线程
author: Wang Yue Niao
top: false
toc: true
date: 2020-10-11 16:16:26
tags: 多线程
categories: 多线程
---

### java线程理解

- 线程是调度CPU的最小单元，也叫轻量级进程（LWP）
- 两种线程模型
  - 用户级线程（ULT）：用户程序实现，不依赖操作系统核心，引用提供创建、同步、调度和管理线程的函数来控制用户线程。不需要用户态/内核态切换，速度快。内核对ULT无感知，线程阻塞则进程（包括它的所有线程）阻塞。
  - 内核级线程（KLT）：系统内核管理线程（KLT），内核保存线程的状态和上下文信息，线程阻塞不会引起进程阻塞。在多处理器系统上，多线程在多处理器上并运行。线程的创建，调度和管理由内核完成，效率比ULT要慢，比进程操作快

### 线程的创建与使用

##### 多线程创建，方式一：继承Thread类

```java

public class StackStruTest {

    public static void main(String[] args) {
        testThread01 t1 = new testThread01();
        t1.start();
    }
}

//1.创建一个继承Thread类的子类
//2.重写Thread类的run()
//3.创建Threa类的子类对象
//4.通过此对象调用start()方法
class testThread01 extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            if (i % 2 == 0) {
                System.out.println(i);
            }
        }
    }
}
```

### Thread类的有关方法

- void start()：启动线程，并执行对象的run()方法

- run()：线程再被调度时执行的操作
- String getName()：返回线程的名称
- void setName（String name）：设置该线程名称
- static Thread currentThread()：返回当前线程。在Thread子类中就是this，通常用于主线程和Runnable实现类
- static void yield()：线程让步
  - 暂停当前正在执行的线程，把执行机会让给优先级相同或更高的线程
  - 若队列中没有相同优先级的线程，忽略此方法
- join()：当某个程序执行流中调用其他线程的join()方法时，调用线程将被阻塞，直到join()方法加入的join线程执行完为止
  - 低优先级的线程也可以获得执行
- static void sleep(long millis)：（指定时间：毫秒）
  - 令当前活动线程在指定时间段内放弃对CPU控制，使其他线程有机会被执行，时间到后重拍队
  - 抛出InterruptedException异常
- stop():：强制线程声明期结束
- boolean isAlive()：返回boolean，判断线程是否还活着

### 线程的优先级

- 线程优先级等级
  - MAX_PRIORITY:10
  - MIN_PRIORITY:1
  - NORM_PRIORITY:5
- 涉及的方法
  - getPriority()：返回线程的优先值
  - setPriority(int newPriority)：改变线程的优先级
- 说明
  - 线程创建时继承父线程的优先级
  - 低优先级只是获得调度的概率低，并非一定是在高优先级线程之后才被调用

### 多线程创建，方式二：现Runnable接口

```java
public class StackStruTest {

    public static void main(String[] args) {
        testThread01 t = new testThread01();
        
        Thread t1=new Thread(t);
        t1.setName("线程A");
        
        Thread t2=new Thread(t,"线程B");
        
        t1.start();
        t2.start();
    }
}

//1.创建一个类实现Runnable接口
//2.实现run方法
//3.将Runnable对象传入Thread类的构造方法
//4.调用Thread的方法启动线程
class testThread01 implements Runnable {
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            if (i % 2 == 0) {
                System.out.println(i);
            }
        }
    }
}
```

### 比较两种方式

- 开发中优先选择：实现Runnable接口的方式
  - 原因：
    1. 实现的方式没有类的单继承的局限性
    2. 实现的方式更适合而来处理多个线程共享数据的情况

- 联系：public class Thread implements Runnable
- 相同点：两种方式都需要重写run()，方法将成要执行的逻辑声明在run()方法中。

