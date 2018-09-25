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

# 线程池中各重要的类与接口

## ThreadPoolExecutor类
java.uitl.concurrent.ThreadPoolExecutor类是线程池中最核心的一个类，在ThreadPoolExecutor类中提供了四个构造方法：
	

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

### 构造器中各个参数的意义：

#### corePoolSize 核心池大小。在创建了线程池后，默认情况下线程中并没有任何线程。而是等到有任务到来时才去创建线程去执行任务。
当有任务来之后，就会创建一个线程去执行任务，当线程池中的线程数目达到corePoolSize后，就会把到达的任务放到缓存队列当中；
*注：除非调用了prestartAllCoreThreads()或者prestartCoreThread()方法。这两个方法用来创建预线程，即任务没有到来时，便预先创建[corePoolSize]个线程，或者一个线程。*

#### maximumPoolSize 线程池最大线程数。表示在线程池中最多能创建多少个线程

#### keepAliveTime 表示线程没有任务执行时最多保持多久时间会终止。默认情况下，只有当线程池中的线程数大于corePoolSize时，keepAliveTime才会起作用，直到线程池中的线程数不大于corePoolSize，即当线程池中的线程数大于corePoolSize时，如果一个线程空闲的时间达到keepAliveTime，则会终止，直到线程池中的线程数不超过corePoolSize。
*注：但是如果调用了allowCoreThreadTimeOut(boolean)方法，在线程池中的线程数不大于corePoolSize时，keepAliveTime参数也会起作用，直到线程池中的线程数为0；*

#### unit：参数keepAliveTime的时间单位，有7种取值，在TimeUnit类中有7种静态属性：

	TimeUnit.DAYS;               //天
	TimeUnit.HOURS;             //小时
	TimeUnit.MINUTES;           //分钟
	TimeUnit.SECONDS;           //秒
	TimeUnit.MILLISECONDS;      //毫秒
	TimeUnit.MICROSECONDS;      //微妙
	TimeUnit.NANOSECONDS;       //纳秒

#### workQueue：一个阻塞队列，用来存储等待执行的任务，这个参数的选择也很重要，会对线程池的运行过程产生重大影响，一般来说，这里的阻塞队列有以下几种选择

	ArrayBlockingQueue;
	LinkedBlockingQueue;
	SynchronousQueue;
	
*ArrayBlockingQueue和PriorityBlockingQueue使用较少，一般使用LinkedBlockingQueue和Synchronous。线程池的排队策略与BlockingQueue有关。*

#### threadFactory：线程工厂，主要用来创建线程；

#### handler：表示当拒绝处理任务时的策略，有以下四种取值：

	ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。 
	ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。 
	ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
	ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务 
	
### ThreadPoolExecutor类中几个重要的方法
#### execute()方法实际上是Executor中声明的方法，在ThreadPoolExecutor进行了具体的实现，这个方法是ThreadPoolExecutor的核心方法，通过这个方法可以向线程池提交一个任务，交由线程池去执行。

#### submit()方法是在ExecutorService中声明的方法，在AbstractExecutorService就已经有了具体的实现，在ThreadPoolExecutor中并没有对其进行重写，这个方法也是用来向线程池提交任务的，但是它和execute()方法不同，它能够返回任务执	行的结果，去看submit()方法的实现，会发现它实际上还是调用的execute()方法，只不过它利用了Future来获取任务执行结果（Future相关内容将在下一篇讲述）。

#### shutdown()和shutdownNow()是用来关闭线程池的。

## AbstractExecutorService

	public abstract class AbstractExecutorService implements ExecutorService {

		protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) { };
	
    		protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) { };
    
    		public Future<?> submit(Runnable task) {};
    
    		public <T> Future<T> submit(Runnable task, T result) { };
    
    		public <T> Future<T> submit(Callable<T> task) { };
    
    		private <T> T doInvokeAny(Collection<? extends Callable<T>> tasks, boolean timed, long nanos)
        		throws InterruptedException, ExecutionException, TimeoutException {
    		};
    
    		public <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        		throws InterruptedException, ExecutionException {
    		};
    
    		public <T> T invokeAny(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit)
        		throws InterruptedException, ExecutionException, TimeoutException {
    		};
    
    		public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        		throws InterruptedException {
    		};
    
    		public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit)
        		throws InterruptedException {
    		};
	}
	
## ExecutorService接口

	public interface ExecutorService extends Executor {
 
    		void shutdown();
    		
    		boolean isShutdown();
    		
    		boolean isTerminated();
    		
    		boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;
    		
    		<T> Future<T> submit(Callable<T> task);
    		
    		<T> Future<T> submit(Runnable task, T result);
    		
    		Future<?> submit(Runnable task);
    		
    		<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) throws InterruptedException;
    		
    		<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException;
 
    		<T> T invokeAny(Collection<? extends Callable<T>> tasks) throws InterruptedException, ExecutionException;
    		
    		<T> T invokeAny(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
	}

## Executor接口

	public interface Executor {
    		void execute(Runnable command);
	}
	
## 上述接口、类之间的关系

Executor是一个顶层接口，在它里面只声明了一个方法execute(Runnable)，返回值为void，参数为Runnable类型，从字面意思可以理解，就是用来执行传进去的任务的；

然后ExecutorService接口继承了Executor接口，并声明了一些方法：submit、invokeAll、invokeAny以及shutDown等；

抽象类AbstractExecutorService实现了ExecutorService接口，基本实现了ExecutorService中声明的所有方法；

然后ThreadPoolExecutor继承了类AbstractExecutorService。


# 线程池实现原理

## 线程池状态

## 任务的执行

## 线程池中的线程初始化

## 任务缓存队列及排队策略

## 任务拒绝策略

## 线程池的关闭

## 线程池容量的动态调整

# 使用示例

# 如何合理配置线程池的大小

一般需要根据任务的类型来配置线程池大小：

　　如果是CPU密集型任务，就需要尽量压榨CPU，参考值可以设为 NCPU+1

　　如果是IO密集型任务，参考值可以设置为2*NCPU

　　当然，这只是一个参考值，具体的设置还需要根据实际情况进行调整，比如可以先将线程池大小设置为参考值，再观察任务运行情况和系统负载、资源利用率来进行适当调整。


- [原文链接] (http://www.cnblogs.com/dolphin0520/p/3932921.html)
