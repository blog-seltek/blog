---
title: Java并发编程：线程池的使用
categories: FE
author: zhouang
tags:
  - java
  - ThreadPool
---

# 问题引入

### 在并发的线程数量很多，并且每个线程都是执行一个时间很短的任务就结束了的情况下。
### 如果频繁创建线程会大大降低系统的效率，因为频繁创建线程和销毁线程需要时间。
### 在Java中可以通过线程池来达到线程复用，就是执行完一个任务，并不被销毁，而是继续去执行其他的任务。

# 以下是本文的目录大纲：

## 一.Java中的ThreadPoolExecutor类

## 二.深入剖析线程池实现原理

## 三.使用示例

## 四.如何合理配置线程池的大小



## ThreadPoolExecutor

	java.uitl.concurrent.ThreadPoolExecutor类是线程池中最核心的一个类
	
	在ThreadPoolExecutor类中提供了四个构造方法：

public class ThreadPoolExecutor extends AbstractExecutorService {
    .....
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



- [原文链接] (http://www.cnblogs.com/dolphin0520/p/3932921.html)
