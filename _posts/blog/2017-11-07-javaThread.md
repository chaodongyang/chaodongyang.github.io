---
layout: post
title:  java多线程之并发编程
categories: javaThinking
description: java多线程之并发编程
keywords: java线程 java多线程 java并发编程 java线程池  ThreadPoolExecutor
---

线程是一个比较模糊的概念，今天来研究一下线程中的并发实现。以及来看看线程池的实现。
## 什么是线程
有时被称为轻量级进程 ( Lightweight Process，LWP ），是程序执行流的最小单元。一个标准的线程由线程 ID ，当前指令指针 ( PC ），寄存器集合和堆栈组成。另外，线程是进程中的一个实体，是被系统独立调度和分派的基本单位，线程自己不拥有系统资源，只拥有一点儿在运行中必不可少的资源，但它可与同属一个进程的其它线程共享进程所拥有的全部资源。一个线程可以创建和撤消另一个线程，同一进程中的多个线程之间可以并发执行。由于线程之间的相互制约，致使线程在运行中呈现出间断性。线程也有就绪、阻塞和运行三种基本状态。就绪状态是指线程具备运行的所有条件，逻辑上可以运行，在等待处理机；运行状态是指线程占有处理机正在运行；阻塞状态是指线程在等待一个事件（如某个信号量），逻辑上不可执行。每一个程序都至少有一个线程，若程序只有一个线程，那就是程序本身。

## 线程并发简介

- 线程
  - 简介
  - 为什么要使用线程
  - 线程的状态
  - 常用方法介绍
  - 等待与通知
- 线程池
  - 线程池的应用场景
  - 线程池的实现原理
  - Java 中的 ThreadPoolExecutor 类
  - 深入剖析线程池实现原理
    - 线程池状态
    - 任务的执行
    - 线程池中的线程初始化
    - 任务缓存队列及排队策略
    - 任务拒绝策略
    - 线程池的关闭
    - 线程池容量的动态调整
  - 使用示例


## 线程
#### 简介
线程，有时被称为轻量级进程 ( Lightweight Process，LWP ），是程序执行流的最小单元。一个标准的线程由线程 ID ，当前指令指针 ( PC ），寄存器集合和堆栈组成。另外，线程是进程中的一个实体，是被系统独立调度和分派的基本单位，线程自己不拥有系统资源，只拥有一点儿在运行中必不可少的资源，但它可与同属一个进程的其它线程共享进程所拥有的全部资源。

#### 为什么要使用线程
 - 更多的处理器核心
 - 更快的反应时间
 - 更好的编程模型

#### 线程的状态
java 线程在运行的生命周期中一共有 6 中状态。在给定的时刻，线程只能处于一种状态。

| 状态         |  说明         |
| :----       |   :-----      |
| new         |  初始状态，线程被构建，但是还没有调用 start() 方法。|
| runnable    |  运行状态，java 线程在操作系统中的就绪和运行统称为运行中。           |
|blocked     |  阻塞状态，表示线程阻塞与锁。            |
|waiting     |  等待状态，表示线程进入等待状态。需要等待其他线程做出一些特定的动作 ( 通知或者中断 )  |
|time_waiting | 超时等待，该状态不用于等待，他可以在指定的时间自行返回  |
| terminated  | 终止状态，表示该线程执行完毕        |

![](/images/blog/xiancheng.png)

#### 常用方法介绍

| 方法        |   描述         |
| :----            | :----              |
|  start |   start 用于开启一个新的线程，当系统调用 start 方法后，才会开启一个新的线程去执行用户的子任务。在这个过程中会为相应的线程分配资源   |
|  run    |  run() 方法是不需要用户调用的，当线程执行 start() 方法后，线程启动或得了 CPU 的执行时间。便进入 run 方法体执行相应的任务。注意：继承 Thread 类必须重写 run() 方法。在 run() 方法中执行具体的任务。       |
|  sleep      |  sleep 相当于让线程休眠，交出 CPU 让 CPU 去执行其他的任务。sleep 方法不会释放锁。也就是说如果当前线程有某个对象的锁。即使执行了 sleep 方法，其他线程也无法访问这个对象。           |
|  yield      |  yield 方法会让当前线程交出 CPU 权限。让 CPU 去执行其他线程。这一点和 sleep 很像。 yield 同样不会释放锁。但是 yield 不能控制具体交出 CPU 的时间。只能让拥有相同优先级别的线程有获取 CPU 执行时间的机会。调用 yield 不会让线程进入阻塞状态，而是重新进去就绪状态。他只需要重新等待获取 CPU 的执行时间。         |
|  join     |   join 用于临时加入线程，一个线程在运算过程中如果满足条件，我们可以临时加入一个线程，让这个线程运算完，在运行另外一个线程。          |
| interrupt       |  interrupt 方法中断。单独调用 interrupt 方法可以使得处于阻塞状态下的线程抛出异常。也就是说他可以中断一个处于阻塞状态的线程。另外，如果通过 interrupt 和 isInterrupt 方法来中断一个正在运行的线程。直接调用 interrupt 方法不能中断正在运行的线程，一般通过增加一个属性 isStop 来标记是否结束循环。          |
|  gitId       |    用来得到线程ID         |
|  getName     |    用来得到线程名称          |
|  setName     |    用来设置线程名称     |
|  getPriority |    用于获得线程优先级   |
|  setPriority |    用于设置线程优先级   |
| setDeamon   |     用于设置是否成为守护线程。守护线程和用户线程的区别：守护线程依赖于创建它的线程，而用户线程不依赖。|
|  isDeamon    | 判断线程是否是守护线程     |

#### 等待与通知
一个线程修改了一个对象的值，而另一个线程感知到了变化，然后进行相应的操作。整个过程开始于一个线程，而执行与另外一个线程。前者是生产者，后者是消费者。这种模式隔离了 「怎么做」和 「做什么」，在功能层面上实现了解耦。体系结构上实现了良好的伸缩性。

###### 等待方的原则：
   - 获取对象的锁
   - 如果条件不满足则调用线程的 wait() 方法，使该线程进入 waiting 状态，被通知后依然要检查条件。
   - 条件满足则执行相应的逻辑
``` java
   synchronized(对象){
    while(条件不满足){
        对象.wait();
    }
    对应的逻辑处理
}
 ```

###### 通知方的原则：
  - 获取该对象的锁
  - 改变条件
  - 通知所有等待在改对象上的线程
``` java
synchronized(对象){
    改变条件
    对象.notifyAll();
}
```

###### 实现代码：
``` java
package ThreadConcurrency;

public class WaitNotiFy {

	static boolean flag = true;
	static Object lock = new Object();

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		//等待线程启动
		Thread waitThread = new Thread(new Wait());
		waitThread.start();
		//休眠一秒
		try {
			Thread.sleep(1);
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

		//通知线程启动
		Thread notifThread = new Thread(new Notif());
		notifThread.start();


	}

	static class Wait implements Runnable{

		@Override
		public void run() {
			// TODO Auto-generated method stub
			//加锁
			synchronized(lock){
			   //当条件不满足的时候，进入waiting状态，同时释放锁
				while (flag) {
					System.out.println("flag is true");
					try {
						lock.wait();
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				}
				//条件满足
				System.out.println("条件满足");
			}
		}

	}

	static class Notif implements Runnable{

		@Override
		public void run() {
			// TODO Auto-generated method stub
			//加锁
			synchronized (lock) {
				//获取lock的锁，然后进行通知，通知不会释放lock锁
		         //直到发出通知的线程执行完毕释放了lock锁，WaitThread线程才能从wait方法返回
				lock.notifyAll();
				System.out.println("flag is false now");
				flag = false;
			}
		}

	}

}

```
###### 输出内容：
```
  > flag is true
  > flag is false now
  > 条件满足
```

## 线程池 ThreadPool

#### 线程池的应用场景
Java 中的线程池是运用场景最多的并发框架，几乎所有需要异步或并发执行任务的程序都可以使用线程池。在开发过程中，合理地使用线程池能够带来3个好处：

- 降低资源消耗。通过重复利用已经创建的线程来降低线程的重复创建和销毁。
- 提高响应速度。当任务到达时，不需要等待线程创建就可以直接执行。
- 提高线程的可管理性。线程是稀缺资源，如果无限制的创建不仅会造成消耗系统资源还会降低系统稳定性。使用线程池可以统一调配、优化、和监控。

#### 线程池的实现原理

ThreadPoolExecutor 执行 execute 方法分下面四中情况：

 - 如果当前运行的线程少于 corePoolSize ,则创建新线程来执行任务 (执行这一步骤需要获取全局锁)。
 - 如果运行的线程等于或多余 corePoolSize ,则将任务加入 BlockingQueue 。
 - 如果无法将任务加入 BlockingQueue (队列已满) , 则创建新的线程来处理。(这一步骤需要获取全局锁) 。
 - 如果创建的新线程将使得当前线程数超出 maximumPoolSize ，任务将会被拒绝，并调用 RejectedExecutionHandler.rejectedExecution() 方法。

ThreadPoolExecutor采取上述步骤的总体设计思路，是为了在执行execute()方法时，尽可能地避免获取全局锁（那将会是一个严重的可伸缩瓶颈）。在ThreadPoolExecutor完成预热之后（当前运行的线程数大于等于corePoolSize），几乎所有的execute()方法调用都是执行步骤2，而步骤2不需要获取全局锁。

#### Java 中的 ThreadPoolExecutor 类
java.util.concurrent.ThreadPoolExecutor 类是线程池中最核心的一个类，因此如果要透彻的了解 java 的线程池，必须先了解这个类。下面我们看一下 ThreadPoolExecutor 类的具体实现源码。

``` java
public class ThreadPoolExecutor extends AbstractExecutorService {
// Public constructors and methods

    /**
     * Creates a new {@code ThreadPoolExecutor} with the given initial
     * parameters and default thread factory and rejected execution handler.
     * It may be more convenient to use one of the {@link Executors} factory
     * methods instead of this general purpose constructor.
     *
     * @param corePoolSize the number of threads to keep in the pool, even
     *        if they are idle, unless {@code allowCoreThreadTimeOut} is set
     * @param maximumPoolSize the maximum number of threads to allow in the
     *        pool
     * @param keepAliveTime when the number of threads is greater than
     *        the core, this is the maximum time that excess idle threads
     *        will wait for new tasks before terminating.
     * @param unit the time unit for the {@code keepAliveTime} argument
     * @param workQueue the queue to use for holding tasks before they are
     *        executed.  This queue will hold only the {@code Runnable}
     *        tasks submitted by the {@code execute} method.
     * @throws IllegalArgumentException if one of the following holds:<br>
     *         {@code corePoolSize < 0}<br>
     *         {@code keepAliveTime < 0}<br>
     *         {@code maximumPoolSize <= 0}<br>
     *         {@code maximumPoolSize < corePoolSize}
     * @throws NullPointerException if {@code workQueue} is null
     */
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), defaultHandler);
    }

    /**
     * Creates a new {@code ThreadPoolExecutor} with the given initial
     * parameters and default rejected execution handler.
     *
     * @param corePoolSize the number of threads to keep in the pool, even
     *        if they are idle, unless {@code allowCoreThreadTimeOut} is set
     * @param maximumPoolSize the maximum number of threads to allow in the
     *        pool
     * @param keepAliveTime when the number of threads is greater than
     *        the core, this is the maximum time that excess idle threads
     *        will wait for new tasks before terminating.
     * @param unit the time unit for the {@code keepAliveTime} argument
     * @param workQueue the queue to use for holding tasks before they are
     *        executed.  This queue will hold only the {@code Runnable}
     *        tasks submitted by the {@code execute} method.
     * @param threadFactory the factory to use when the executor
     *        creates a new thread
     * @throws IllegalArgumentException if one of the following holds:<br>
     *         {@code corePoolSize < 0}<br>
     *         {@code keepAliveTime < 0}<br>
     *         {@code maximumPoolSize <= 0}<br>
     *         {@code maximumPoolSize < corePoolSize}
     * @throws NullPointerException if {@code workQueue}
     *         or {@code threadFactory} is null
     */
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             threadFactory, defaultHandler);
    }

    /**
     * Creates a new {@code ThreadPoolExecutor} with the given initial
     * parameters and default thread factory.
     *
     * @param corePoolSize the number of threads to keep in the pool, even
     *        if they are idle, unless {@code allowCoreThreadTimeOut} is set
     * @param maximumPoolSize the maximum number of threads to allow in the
     *        pool
     * @param keepAliveTime when the number of threads is greater than
     *        the core, this is the maximum time that excess idle threads
     *        will wait for new tasks before terminating.
     * @param unit the time unit for the {@code keepAliveTime} argument
     * @param workQueue the queue to use for holding tasks before they are
     *        executed.  This queue will hold only the {@code Runnable}
     *        tasks submitted by the {@code execute} method.
     * @param handler the handler to use when execution is blocked
     *        because the thread bounds and queue capacities are reached
     * @throws IllegalArgumentException if one of the following holds:<br>
     *         {@code corePoolSize < 0}<br>
     *         {@code keepAliveTime < 0}<br>
     *         {@code maximumPoolSize <= 0}<br>
     *         {@code maximumPoolSize < corePoolSize}
     * @throws NullPointerException if {@code workQueue}
     *         or {@code handler} is null
     */
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              RejectedExecutionHandler handler) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), handler);
    }

    /**
     * Creates a new {@code ThreadPoolExecutor} with the given initial
     * parameters.
     *
     * @param corePoolSize the number of threads to keep in the pool, even
     *        if they are idle, unless {@code allowCoreThreadTimeOut} is set
     * @param maximumPoolSize the maximum number of threads to allow in the
     *        pool
     * @param keepAliveTime when the number of threads is greater than
     *        the core, this is the maximum time that excess idle threads
     *        will wait for new tasks before terminating.
     * @param unit the time unit for the {@code keepAliveTime} argument
     * @param workQueue the queue to use for holding tasks before they are
     *        executed.  This queue will hold only the {@code Runnable}
     *        tasks submitted by the {@code execute} method.
     * @param threadFactory the factory to use when the executor
     *        creates a new thread
     * @param handler the handler to use when execution is blocked
     *        because the thread bounds and queue capacities are reached
     * @throws IllegalArgumentException if one of the following holds:<br>
     *         {@code corePoolSize < 0}<br>
     *         {@code keepAliveTime < 0}<br>
     *         {@code maximumPoolSize <= 0}<br>
     *         {@code maximumPoolSize < corePoolSize}
     * @throws NullPointerException if {@code workQueue}
     *         or {@code threadFactory} or {@code handler} is null
     */
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
}
```
从上面的源码得知，ThreadPoolExecutor 类继承了 AbstractExecutorService 类，并提供了四个构造器，事实上，通过观察每个构造器的源码实现，发现前面三个构造器还是通过调用第四个构造器进行的初始化工作。

构造器中各个参数的含义：

- **corePoolSize** 核心池的大小，这个参数和线程池的原理有非常大的关系。在创建线程池后，默认情况下，线程池中没有任何线程，而是等待有任务到来才创建线程去执行任务，除非调用了 prestartAllCoreThreads() 或者 prestartCoreThread() 方法，从这 2 个方法的名字就可以看出，是预创建线程的意思，即在没有任务到来之前就创建 corePoolSize 个线程或者一个线程。默认情况下，在创建了线程池后，线程池中的线程数为 0 ，当有任务来之后，就会创建一个线程去执行任务，当线程池中的线程数目达到 corePoolSize 后，就会把到达的任务放到缓存队列当中。
- **maximumPoolSize** 线程池最大线程数，这个参数也是一个非常重要的参数，它表示在线程池中最多能创建多少个线程。
- **keepAliveTime** 表示线程没有任务执行时最多保持多久时间会终止。默认情况下，只有当线程池中的线程数大于 corePoolSize 时，keepAliveTime 才会起作用，直到线程池中的线程数不大于 corePoolSize ，即当线程池中的线程数大于 corePoolSize 时，如果一个线程空闲的时间达到 keepAliveTime ，则会终止，直到线程池中的线程数不超过 corePoolSize 。但是如果调用了 allowCoreThreadTimeOut(boolean) 方法，在线程池中的线程数不大于 corePoolSize 时，keepAliveTime 参数也会起作用，直到线程池中的线程数为 0 。
- **unit** 参数 keepAliveTime 的时间单位，有 7 种取值，在 TimeUnit 类中有 7 种静态属性。
```
  > TimeUnit.DAYS;               //天

  > TimeUnit.HOURS;             //小时

  > TimeUnit.MINUTES;           //分钟

  > TimeUnit.SECONDS;           //秒

  > TimeUnit.MILLISECONDS;      //毫秒

  > TimeUnit.MICROSECONDS;      //微妙

  > TimeUnit.NANOSECONDS;       //纳秒
 ```

- **workQueue** 一个阻塞队列，用来存储等待执行的任务，这个参数的选择也很重要，会对线程池的运行过程产生重大影响，一般来说，这里的阻塞队列有以下几种选择。ArrayBlockingQueue 、LinkedBlockingQueue 、SynchronousQueue 。ArrayBlockingQueue 和 PriorityBlockingQueue 使用较少，一般使用 LinkedBlockingQueue 和 Synchronous 。线程池的排队策略与 BlockingQueue 有关。

- **threadFactory** 线程工厂，主要用来创建线程。
- **handler** 表示当拒绝处理任务时的策略，有以下四种取值。

 >ThreadPoolExecutor.AbortPolicy: 丢弃任务并抛出 RejectedExecutionException 异常。

 >ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。

 >ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）

 >ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务

从上面给出的 ThreadPoolExecutor 类的代码可以知道，ThreadPoolExecutor 继承了 AbstractExecutorService ，我们来看一下 AbstractExecutorService 的实现。

``` java
public abstract class AbstractExecutorService implements ExecutorService {

    /**
     * Returns a {@code RunnableFuture} for the given runnable and default
     * value.
     *
     * @param runnable the runnable task being wrapped
     * @param value the default value for the returned future
     * @param <T> the type of the given value
     * @return a {@code RunnableFuture} which, when run, will run the
     * underlying runnable and which, as a {@code Future}, will yield
     * the given value as its result and provide for cancellation of
     * the underlying task
     * @since 1.6
     */
    protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) {
        return new FutureTask<T>(runnable, value);
    }

    /**
     * Returns a {@code RunnableFuture} for the given callable task.
     *
     * @param callable the callable task being wrapped
     * @param <T> the type of the callable's result
     * @return a {@code RunnableFuture} which, when run, will call the
     * underlying callable and which, as a {@code Future}, will yield
     * the callable's result as its result and provide for
     * cancellation of the underlying task
     * @since 1.6
     */
    protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
        return new FutureTask<T>(callable);
    }

    /**
     * @throws RejectedExecutionException {@inheritDoc}
     * @throws NullPointerException       {@inheritDoc}
     */
    public Future<?> submit(Runnable task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<Void> ftask = newTaskFor(task, null);
        execute(ftask);
        return ftask;
    }

    /**
     * @throws RejectedExecutionException {@inheritDoc}
     * @throws NullPointerException       {@inheritDoc}
     */
    public <T> Future<T> submit(Runnable task, T result) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<T> ftask = newTaskFor(task, result);
        execute(ftask);
        return ftask;
    }

    /**
     * @throws RejectedExecutionException {@inheritDoc}
     * @throws NullPointerException       {@inheritDoc}
     */
    public <T> Future<T> submit(Callable<T> task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<T> ftask = newTaskFor(task);
        execute(ftask);
        return ftask;
    }

    /**
     * the main mechanics of invokeAny.
     */
    private <T> T doInvokeAny(Collection<? extends Callable<T>> tasks,
                              boolean timed, long nanos)
        throws InterruptedException, ExecutionException, TimeoutException {
        if (tasks == null)
            throw new NullPointerException();
        int ntasks = tasks.size();
        if (ntasks == 0)
            throw new IllegalArgumentException();
        ArrayList<Future<T>> futures = new ArrayList<Future<T>>(ntasks);
        ExecutorCompletionService<T> ecs =
            new ExecutorCompletionService<T>(this);

        // For efficiency, especially in executors with limited
        // parallelism, check to see if previously submitted tasks are
        // done before submitting more of them. This interleaving
        // plus the exception mechanics account for messiness of main
        // loop.

        try {
            // Record exceptions so that if we fail to obtain any
            // result, we can throw the last exception we got.
            ExecutionException ee = null;
            final long deadline = timed ? System.nanoTime() + nanos : 0L;
            Iterator<? extends Callable<T>> it = tasks.iterator();

            // Start one task for sure; the rest incrementally
            futures.add(ecs.submit(it.next()));
            --ntasks;
            int active = 1;

            for (;;) {
                Future<T> f = ecs.poll();
                if (f == null) {
                    if (ntasks > 0) {
                        --ntasks;
                        futures.add(ecs.submit(it.next()));
                        ++active;
                    }
                    else if (active == 0)
                        break;
                    else if (timed) {
                        f = ecs.poll(nanos, TimeUnit.NANOSECONDS);
                        if (f == null)
                            throw new TimeoutException();
                        nanos = deadline - System.nanoTime();
                    }
                    else
                        f = ecs.take();
                }
                if (f != null) {
                    --active;
                    try {
                        return f.get();
                    } catch (ExecutionException eex) {
                        ee = eex;
                    } catch (RuntimeException rex) {
                        ee = new ExecutionException(rex);
                    }
                }
            }

            if (ee == null)
                ee = new ExecutionException();
            throw ee;

        } finally {
            for (int i = 0, size = futures.size(); i < size; i++)
                futures.get(i).cancel(true);
        }
    }

    public <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException {
        try {
            return doInvokeAny(tasks, false, 0);
        } catch (TimeoutException cannotHappen) {
            assert false;
            return null;
        }
    }

    public <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                           long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException {
        return doInvokeAny(tasks, true, unit.toNanos(timeout));
    }

    public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException {
        if (tasks == null)
            throw new NullPointerException();
        ArrayList<Future<T>> futures = new ArrayList<Future<T>>(tasks.size());
        boolean done = false;
        try {
            for (Callable<T> t : tasks) {
                RunnableFuture<T> f = newTaskFor(t);
                futures.add(f);
                execute(f);
            }
            for (int i = 0, size = futures.size(); i < size; i++) {
                Future<T> f = futures.get(i);
                if (!f.isDone()) {
                    try {
                        f.get();
                    } catch (CancellationException ignore) {
                    } catch (ExecutionException ignore) {
                    }
                }
            }
            done = true;
            return futures;
        } finally {
            if (!done)
                for (int i = 0, size = futures.size(); i < size; i++)
                    futures.get(i).cancel(true);
        }
    }

    public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                         long timeout, TimeUnit unit)
        throws InterruptedException {
        if (tasks == null)
            throw new NullPointerException();
        long nanos = unit.toNanos(timeout);
        ArrayList<Future<T>> futures = new ArrayList<Future<T>>(tasks.size());
        boolean done = false;
        try {
            for (Callable<T> t : tasks)
                futures.add(newTaskFor(t));

            final long deadline = System.nanoTime() + nanos;
            final int size = futures.size();

            // Interleave time checks and calls to execute in case
            // executor doesn't have any/much parallelism.
            for (int i = 0; i < size; i++) {
                execute((Runnable)futures.get(i));
                nanos = deadline - System.nanoTime();
                if (nanos <= 0L)
                    return futures;
            }

            for (int i = 0; i < size; i++) {
                Future<T> f = futures.get(i);
                if (!f.isDone()) {
                    if (nanos <= 0L)
                        return futures;
                    try {
                        f.get(nanos, TimeUnit.NANOSECONDS);
                    } catch (CancellationException ignore) {
                    } catch (ExecutionException ignore) {
                    } catch (TimeoutException toe) {
                        return futures;
                    }
                    nanos = deadline - System.nanoTime();
                }
            }
            done = true;
            return futures;
        } finally {
            if (!done)
                for (int i = 0, size = futures.size(); i < size; i++)
                    futures.get(i).cancel(true);
        }
    }

}

```

AbstractExecutorService 是一个抽象类，它实现了 ExecutorService 接口。

``` java
public interface ExecutorService extends Executor {

    void shutdown();

    List<Runnable> shutdownNow();

    boolean isShutdown();

    boolean isTerminated();

    boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException;

    <T> Future<T> submit(Callable<T> task);

    <T> Future<T> submit(Runnable task, T result);

    Future<?> submit(Runnable task);

    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException;

    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                  long timeout, TimeUnit unit)
        throws InterruptedException;

    <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;

    <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                    long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

而 ExecutorService 又是继承了 Executor 接口，我们看一下 Executor 接口的实现。

``` java
public interface Executor {

    /**
     * Executes the given command at some time in the future.  The command
     * may execute in a new thread, in a pooled thread, or in the calling
     * thread, at the discretion of the {@code Executor} implementation.
     *
     * @param command the runnable task
     * @throws RejectedExecutionException if this task cannot be
     * accepted for execution
     * @throws NullPointerException if command is null
     */
    void execute(Runnable command);
}
```
到这里，大家应该明白了ThreadPoolExecutor、AbstractExecutorService、ExecutorService和Executor几个之间的关系了。Executor 是一个顶层接口，在它里面只声明了一个方法 execute(Runnable) ，返回值为 void ，参数为 Runnable 类型，从字面意思可以理解，就是用来执行传进去的任务的.然后 ExecutorService 接口继承了 Executor 接口，并声明了一些方法：submit、invokeAll、invokeAny 以及 shutDown 等。抽象类 AbstractExecutorService 实现了 ExecutorService 接口，基本实现了 ExecutorService 中声明的所有方法。然后 ThreadPoolExecutor 继承了类 AbstractExecutorService 。在 ThreadPoolExecutor 类中有几个非常重要的方法。execute() 、submit() 、 shutDown() 、 shutdownNow()。

 - execute() 方法实际上是 Executor 中声明的方法，在 ThreadPoolExecutor 进行了具体的实现，这个方法是 ThreadPoolExecutor 的核心方法，通过这个方法可以向线程池提交一个任务，交由线程池执行。
 - submit() 方法是在 ExecutorService 中声明的方法，在 AbstractExecutorService 就已经有了具体实现，在 ThreadPoolExecutor 中并没有对其重写，这个方法也是用来向线程池提交任务的，但是它和execute()方法不同，它能够返回任务执行的结果，去看 submit() 方法的实现，会发现它实际上还是调用的 execute() 方法，只不过它利用了 Future 来获取任务执行结果。
 - shutdown() 和 shutdownNow() 是用来关闭线程池的。




#### 深入剖析线程池实现原理
上一节我们从宏观上介绍了 ThreadPoolExecutor ，下面我们来深入解析一下线程池的具体实现原理。

###### 线程池状态
在 ThreadPoolExecutor 中定义了一个 volatile 变量，另外定义了几个 static final 变量表示线程池的各个状态。
```
  >volatile int runState;

  >static final int RUNNING    = 0;

  >static final int SHUTDOWN   = 1;

  >static final int STOP       = 2;

  >static final int TERMINATED = 3;
 ```

 runState 表示当前线程池的状态，它是一个 volatile 变量用来保证线程之间的可见性,下面的几个 static final 变量表示 runState 可能的几个取值。当创建线程池后，初始时，线程池处于 RUNNING 状态。如果调用了 shutdown() 方法，则线程池处于 SHUTDOWN 状态，此时线程池不能够接受新的任务，它会等待所有任务执行完毕。如果调用了 shutdownNow() 方法，则线程池处于 STOP 状态，此时线程池不能接受新的任务，并且会去尝试终止正在执行的任务。当线程池处于 SHUTDOWN 或 STOP 状态，并且所有工作线程已经销毁，任务缓存队列已经清空或执行结束后，线程池被设置为 TERMINATED 状态。


###### 任务的执行

``` java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    if (poolSize >= corePoolSize || !addIfUnderCorePoolSize(command)) {
        if (runState == RUNNING && workQueue.offer(command)) {
            if (runState != RUNNING || poolSize == 0)
                ensureQueuedTaskHandled(command);
        }
        else if (!addIfUnderMaximumPoolSize(command))
            reject(command); // is shutdown or saturated
    }
}
```

首先，判断提交的任务 command 是否为 null ，如果是 null , 则抛出空指针异常。

``` java
if (poolSize >= corePoolSize || !addIfUnderCorePoolSize(command))
```
如果当前线程池中的线程数量大于等于核心线程池的大小则直接进入 if 语句块。如果小于核心线程池的大小则执行后面的 addIfUnderCorePoolSize() 。

``` java
private boolean addIfUnderCorePoolSize(Runnable firstTask) {
    Thread t = null;
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        if (poolSize < corePoolSize && runState == RUNNING)
            t = addThread(firstTask);        //创建线程去执行firstTask任务   
        } finally {
        mainLock.unlock();
    }
    if (t == null)
        return false;
    t.start();
    return true;
}
```
只有线程池当前线程数目小于核心池大小才会执行 addIfUnderCorePoolSize 方法的，为何这地方还要继续判断？原因很简单，前面的判断过程中并没有加锁，因此可能在 execute 方法判断的时候 poolSize 小于 corePoolSize ，而判断完之后，在其他线程中又向线程池提交了任务，就可能导致 poolSize 不小于 corePoolSize 了，所以需要在这个地方继续判断。然后接着判断线程池的状态是否为 RUNNING ，原因也很简单，因为有可能在其他线程中调用了 shutdown 或者 shutdownNow 方法。

``` java
private Thread addThread(Runnable firstTask) {
    Worker w = new Worker(firstTask);
    Thread t = threadFactory.newThread(w);  //创建一个线程，执行任务   
    if (t != null) {
        w.thread = t;            //将创建的线程的引用赋值为w的成员变量       
        workers.add(w);
        int nt = ++poolSize;     //当前线程数加1       
        if (nt > largestPoolSize)
            largestPoolSize = nt;
    }
    return t;
}
```

这个方法也非常关键，传进去的参数为提交的任务，返回值为 Thread 类型。然后接着在下面判断 t 是否为空，为空则表明创建线程失败（即 poolSize>=corePoolSize 或者 runState 不等于 RUNNING ），否则调用 t.start() 方法启动线程。在 addThread 方法中，首先用提交的任务创建了一个 Worker 对象，然后调用线程工厂 threadFactory 创建了一个新的线程 t，然后将线程t的引用赋值给了 Worker 对象的成员变量 thread，接着通过 workers.add(w) 将 Worker 对象添加到工作集当中。

``` java
private final class Worker implements Runnable {
    private final ReentrantLock runLock = new ReentrantLock();
    private Runnable firstTask;
    volatile long completedTasks;
    Thread thread;
    Worker(Runnable firstTask) {
        this.firstTask = firstTask;
    }
    boolean isActive() {
        return runLock.isLocked();
    }
    void interruptIfIdle() {
        final ReentrantLock runLock = this.runLock;
        if (runLock.tryLock()) {
            try {
        if (thread != Thread.currentThread())
        thread.interrupt();
            } finally {
                runLock.unlock();
            }
        }
    }
    void interruptNow() {
        thread.interrupt();
    }

    private void runTask(Runnable task) {
        final ReentrantLock runLock = this.runLock;
        runLock.lock();
        try {
            if (runState < STOP &&
                Thread.interrupted() &&
                runState >= STOP)
            boolean ran = false;
            beforeExecute(thread, task);   //beforeExecute方法是ThreadPoolExecutor类的一个方法，没有具体实现，用户可以根据
            //自己需要重载这个方法和后面的afterExecute方法来进行一些统计信息，比如某个任务的执行时间等           
            try {
                task.run();
                ran = true;
                afterExecute(task, null);
                ++completedTasks;
            } catch (RuntimeException ex) {
                if (!ran)
                    afterExecute(task, ex);
                throw ex;
            }
        } finally {
            runLock.unlock();
        }
    }

    public void run() {
        try {
            Runnable task = firstTask;
            firstTask = null;
            while (task != null || (task = getTask()) != null) {
                runTask(task);
                task = null;
            }
        } finally {
            workerDone(this);   //当任务队列中没有任务时，进行清理工作       
        }
    }
}
```

从 run 方法的实现可以看出，它首先执行的是通过构造器传进来的任务 firstTask，在调用 runTask() 执行完 firstTask 之后，在 while 循环里面不断通过 getTask() 去取新的任务来执行，那么去哪里取呢？自然是从任务缓存队列里面去取，getTask 是 ThreadPoolExecutor 类中的方法，并不是 Worker 类中的方法。

``` java
private boolean addIfUnderMaximumPoolSize(Runnable firstTask) {
    Thread t = null;
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        if (poolSize < maximumPoolSize && runState == RUNNING)
            t = addThread(firstTask);
    } finally {
        mainLock.unlock();
    }
    if (t == null)
        return false;
    t.start();
    return true;
}
```
addIfUnderMaximumPoolSize 方法是在线程池中的线程数达到了核心池大小并且往任务队列中添加任务失败的情况下执行的。其实它和 addIfUnderCorePoolSize 方法的实现基本一模一样，只是if语句判断条件中的 poolSize < maximumPoolSize 不同而已。到这里，大部分朋友应该对任务提交给线程池之后到被执行的整个过程有了一个基本的了解。

###### 线程池中的线程初始化

默认情况下，创建线程池之后，线程池中是没有线程的，需要提交任务之后才会创建线程。在实际中如果需要线程池创建之后立即创建线程，可以通过以下两个方法办到。

 >prestartCoreThread()：初始化一个核心线程；

 >prestartAllCoreThreads()：初始化所有核心线程

 ``` java
 public boolean prestartCoreThread() {
    return addIfUnderCorePoolSize(null); //注意传进去的参数是null
}

public int prestartAllCoreThreads() {
    int n = 0;
    while (addIfUnderCorePoolSize(null))//注意传进去的参数是null
        ++n;
    return n;
}
 ```

###### 任务缓存队列及排队策略

在前面我们多次提到了任务缓存队列，即 workQueue，它用来存放等待执行的任务。workQueue 的类型为 BlockingQueue<Runnable>，通常可以取下面三种类型：

　　1）ArrayBlockingQueue：基于数组的先进先出队列，此队列创建时必须指定大小；

　　2）LinkedBlockingQueue：基于链表的先进先出队列，如果创建时没有指定此队列大小，则默认为 Integer.MAX_VALUE；

　　3）synchronousQueue：这个队列比较特殊，它不会保存提交的任务，而是将直接新建一个线程来执行新来的任务。

###### 任务拒绝策略

当线程池的任务缓存队列已满并且线程池中的线程数目达到 maximumPoolSize，如果还有任务到来就会采取任务拒绝策略，通常有以下四种策略。

```
>ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。

>ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。

>ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）

>ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务
```


###### 线程池的关闭

ThreadPoolExecutor 提供了两个方法，用于线程池的关闭，分别是 shutdown() 和 shutdownNow()，其中：
```
>shutdown()：不会立即终止线程池，而是要等所有任务缓存队列中的任务都执行完后才终止，但再也不会接受新的任务

>shutdownNow()：立即终止线程池，并尝试打断正在执行的任务，并且清空任务缓存队列，返回尚未执行的任务
```


###### 线程池容量的动态调整

ThreadPoolExecutor 提供了动态调整线程池容量大小的方法：setCorePoolSize() 和 setMaximumPoolSize()。

```
>setCorePoolSize：设置核心池大小

>setMaximumPoolSize：设置线程池最大能创建的线程数目大小

```

#### 使用示例

``` java
public class ThreadPoolTest {

	public static void main(String[] args) {
		// 初始化创建一个线程池
		ThreadPoolExecutor executor = new ThreadPoolExecutor(5, 10,200, TimeUnit.MILLISECONDS,new ArrayBlockingQueue<Runnable>(5));

		for(int i=0;i<15;i++){
             MyTask myTask = new MyTask(i);

             executor.execute(myTask);
             System.out.println("线程池中线程数目："+executor.getPoolSize()+"，队列中等待执行的任务数目："+
             executor.getQueue().size()+"，已执行玩别的任务数目："+executor.getCompletedTaskCount());
         }
         executor.shutdown();

	}



}

class MyTask implements Runnable{

	private int taskNum;

	public MyTask(int num) {

	     this.taskNum = num;
	 }

	@Override
	public void run() {
		// TODO Auto-generated method stub
		System.out.println("正在执行task "+taskNum);
        try {
            Thread.currentThread().sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("task "+taskNum+"执行完毕");
	}

}

```

## 感谢
[参考文章地址](http://www.cnblogs.com/dolphin0520/p/3932921.html "参考文档")
