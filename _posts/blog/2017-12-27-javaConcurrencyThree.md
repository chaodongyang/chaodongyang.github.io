---
layout: post
title: java编程思想之并发(终结任务)
categories: javaThinking
description: java编程思想之并发
keywords: java, java并发, java多线程，Java多线程并发
---

有时我们需要任务被突然的终止。这一节将学习有关终止的各类问题。

#### 装饰性花园
这是一个让我们观察的示例，他不仅演示了终止问题，而且还演示了资源共享。下面的仿真程序中，花园委员会希望知道每天进入公园的总人数。每个公园门口都有一个计数器，并且任何一个门口的计数值递增时，就表示公园中的总人数的共享数值也会递增。

计数器：
```java
public class Count {
	private int count =0;
	private Random random = new Random(47);

	public synchronized int increment() {
		int temp = count;
		if (random.nextBoolean()) {
			Thread.yield();
		}
		return (count = ++temp);
	}

	public synchronized int value() {
		return count;
	}

}

```
任务：
```java
public class Entrance implements Runnable{

	private static Count count = new Count();
	private static List<Entrance> entrances = new ArrayList<>();
	private int number = 0;
	private final int id;

	private static volatile boolean canceled = false;
	public static void cancel() {
		canceled = true;
	}

	protected Entrance(int id) {
		super();
		this.id = id;
		entrances.add(this);
	}

	@Override
	public void run() {
		// TODO Auto-generated method stub
		while (!canceled) {
			synchronized (this) {
				++number;
			}

			System.out.println(this + "Total" + count.increment());
			try {
				TimeUnit.MICROSECONDS.sleep(100);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				System.out.println("SLEEP INTERRUPTED");
			}
		}
		System.out.println("结束了");
	}

	public synchronized int getValue() {
		return number;
	}

	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return "Entrance" + id+": " + getValue();
	}

	public static int getTotalCount() {
		return count.value();
	}

	public static int sumEntrances() {
		int sum =0;
		for (Entrance entrance : entrances) {
			sum += entrance.getValue();
		}
		return sum;
	}

}
```
测试类：
```java
public class OrnamentalGraden {

	public static void main(String[] args) throws Exception{
		ExecutorService exec = Executors.newCachedThreadPool();
		for (int i = 0; i < 5; i++) {
			exec.execute(new Entrance(i));
		}
		TimeUnit.SECONDS.sleep(3);
		Entrance.cancel();
		exec.shutdown();
		if (!exec.awaitTermination(250, TimeUnit.MICROSECONDS)) {
			System.out.println("Som task were not terminated");
		}
		System.out.println("总人数" + Entrance.getTotalCount());
		System.out.println("所有的 Entrances:" + Entrance.sumEntrances());

	}

}

```
测试结果：
```
/....
Entrance0: 2092Total10425
Entrance1: 2082Total10426
Entrance3: 2082Total10427
Entrance2: 2085Total10428
Entrance4: 2088Total10429
结束了
结束了
结束了
结束了
结束了
总人数10429
所有的 Entrances:10429
```
我们使用单个的 count 对象来跟踪参观者的主计数器，并且将其作为 Entrance 的静态域存储。Count.increment() 和 Count.value() 都是 synchronized 的，用来控制对 count 域的访问。

每个 Entrance 任务都维护着一个本地值 number，它包含通过某个特定入口进入的参观者数量。这提供了双重检查，以确保记录的参观者数量是正确的。

因为 canceled 是 volatile 布尔值，而且它只会被读取和赋值，所以不需要同步对其访问，就可以安全的操作它。

执行任务 3 秒之后，main() 向 Entrance 发送 static cancel() 消息，然后调用 exec 对象的 shutdown() 方法，之后调用了 exec 的 awaitTermination() 方法。这个方法表示等待每个任务结束，如果所有的任务在超时时间未结束，则返回 ture，否则返回 false，表示不是所有的任务都已经结束。

当执行 run() 方法时，你会看到输出了通过每个门口的人数和此时的总人数。如果移除 Count.increment() 的 synchronized 的声明，你会看到总人数与你的期望会有差异，只要用互斥来同步对 Count 的访问，问题就可以解决。Count.increment() 通过使用 temp() 和 yield()，增加了失败的可能性。在真正的线程问题中失败的可能性在统计学角度看非常小，因此可能很容易的掉进陷阱。

#### 在阻塞时中介
前面示例中的 run() 方法在其循环中包含对 sleep() 的调用。但是 sleep() 会使任务从执行状态变为阻塞状态，而有时你必须终止被阻塞的任务。

**线程状态**

一个线程可以处以一下四种状态：
- 新建 (new)：当线程被创建时，他只会短暂的处以这种状态。此时已经分配了必要的资源，并执行初始化。此刻线程已经有资格获得 cpu 时间了，之后调度器会把这个线程转变为可运行状态或阻塞状态。
- 就绪 (Runnable)：在这种状态下只要调度器把时间片分配给线程，线程就可以运行。在任意时刻线程可以运行也可以不运行，取决于调度去是否分配给线程时间片。
- 阻塞 (Blocked)：线程能够运行，但有个条件组织它运行。当线程处于阻塞状态时，调度器将忽略线程不会分配给他任何 cpu 时间。
- 死亡 (Dead)：处于死亡和终止状态的线程将不再可被调度，并且再也不会得到 cpu 时间，它的任务已经结束，或不再是可运行的。任务死亡的方式通常是从 run() 方法返回，但是任务的线程还可以被中断。

**进入阻塞状态**

一个任务进入阻塞状态可能有如下原因：
- 通过调用 sleep() 使任务进入休眠状态，在这种情况下任务在指定的时间内不会运行。
- 你通过调用 wait() 使线程挂起。直到线程得到了 notif() 或 notifall() 消息，线程才会进入就绪状态。
- 任务在等待某个输入或输出完成。
- 任务试图在某个对象上调用其同步控制方法，但是对象锁不可用，因为另一个任务已经获取了锁。

stop() 方法已经被废弃，因为它不释放线程获得的锁，并且线程处于不一致的状态，其他任务在这种状态下可以浏览并修改他们。这所产生的问题很难发现。

现在我们需要的问题是：有时我们希望能够终止处于阻塞状态的任务。我们决定让其主动终止，那么必须强制这个任务跳出阻塞状态。

#### 中断
