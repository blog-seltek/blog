---
title: Java并发编程：线程池的使用
categories: FE
author: zhouang
tags:
  - java
  - ThreadPool
---

# 问题引入

在并发的线程数量很多，并且每个线程都是执行一个时间很短的任务就结束了的情况下。如果频繁创建线程会大大降低系统的效率，因为频繁创建线程和销毁线程需要时间。在Java中可以通过线程池来达到线程复用，就是执行完一个任务，并不被销毁，而是继续去执行其他的任务。

# 目录大纲：
一.Java中的ThreadPoolExecutor类
二.深入剖析线程池实现原理
三.使用示例
四.如何合理配置线程池的大小


## 一.Java中的ThreadPoolExecutor类

###java.uitl.concurrent.ThreadPoolExecutor类是线程池中最核心的一个类，在ThreadPoolExecutor类中提供了四个构造方法：
	

    public class ThreadPoolExecutor extends AbstractExecutorService {
    ...
    
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue,RejectedExecutionHandler handler);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
        BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory,RejectedExecutionHandler handler);
    ...
	}

从上面的代码可以得知，ThreadPoolExecutor继承了AbstractExecutorService类，并提供了四个构造器，事实上，通过观察每个构造器的源码具体实现，发现前面三个构造器都是调用的第四个构造器进行的初始化工作。

###构造器中各个参数的意义：

####1.corePoolSize 核心池大小。在创建了线程池后，默认情况下线程中并没有任何线程。而是等到有任务到来时才去创建线程去执行任务。
当有任务来之后，就会创建一个线程去执行任务，当线程池中的线程数目达到corePoolSize后，就会把到达的任务放到缓存队列当中；
*注：除非调用了prestartAllCoreThreads()或者prestartCoreThread()方法。这两个方法用来创建预线程，即任务没有到来时，便预先创建[corePoolSize]个线程，或者一个线程。*

####2.maximumPoolSize 线程池最大线程数。表示在线程池中最多能创建多少个线程

####3.keepAliveTime 表示线程没有任务执行时最多保持多久时间会终止。默认情况下，只有当线程池中的线程数大于corePoolSize时，keepAliveTime才会起作用，直到线程池中的线程数不大于corePoolSize，即当线程池中的线程数大于corePoolSize时，如果一个线程空闲的时间达到keepAliveTime，则会终止，直到线程池中的线程数不超过corePoolSize。
*注：但是如果调用了allowCoreThreadTimeOut(boolean)方法，在线程池中的线程数不大于corePoolSize时，keepAliveTime参数也会起作用，直到线程池中的线程数为0；*

####4.unit：参数keepAliveTime的时间单位，有7种取值，在TimeUnit类中有7种静态属性：

	TimeUnit.DAYS;               //天
	TimeUnit.HOURS;             //小时
	TimeUnit.MINUTES;           //分钟
	TimeUnit.SECONDS;           //秒
	TimeUnit.MILLISECONDS;      //毫秒
	TimeUnit.MICROSECONDS;      //微妙
	TimeUnit.NANOSECONDS;       //纳秒

####5.workQueue：一个阻塞队列，用来存储等待执行的任务，这个参数的选择也很重要，会对线程池的运行过程产生重大影响，一般来说，这里的阻塞队列有以下几种选择

	ArrayBlockingQueue;
	LinkedBlockingQueue;
	SynchronousQueue;
	
*ArrayBlockingQueue和PriorityBlockingQueue使用较少，一般使用LinkedBlockingQueue和Synchronous。线程池的排队策略与BlockingQueue有关。*

####6.threadFactory：线程工厂，主要用来创建线程；

####7.handler：表示当拒绝处理任务时的策略，有以下四种取值：

	ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。 
	ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。 
	ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
	ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务 



## 二.深入剖析线程池实现原理

## 三.使用示例

## 四.如何合理配置线程池的大小



- [原文链接] (http://www.cnblogs.com/dolphin0520/p/3932921.html)
