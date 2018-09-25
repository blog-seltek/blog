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

java.uitl.concurrent.ThreadPoolExecutor类是线程池中最核心的一个类
	
在ThreadPoolExecutor类中提供了四个构造方法：

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

构造器中各个参数的意义：

1.corePoolSize 核心池大小。在创建了线程池后，默认情况下线程中并没有任何线程。而是等到有任务到来时才去创建线程去执行任务。
当有任务来之后，就会创建一个线程去执行任务，当线程池中的线程数目达到corePoolSize后，就会把到达的任务放到缓存队列当中；
*除非调用了prestartAllCoreThreads()或者prestartCoreThread()方法。这两个方法用来创建预线程，即任务没有到来时，便预先创建[corePoolSize]个线程，或者一个线程。*

2.maximumPoolSize



## 二.深入剖析线程池实现原理

## 三.使用示例

## 四.如何合理配置线程池的大小



- [原文链接] (http://www.cnblogs.com/dolphin0520/p/3932921.html)
