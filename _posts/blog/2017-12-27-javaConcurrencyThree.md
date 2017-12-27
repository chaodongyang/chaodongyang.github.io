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
在 Rannable.run() 方法中间打断任务是非常棘手的。当你打断被阻塞的任务时可能需要清理资源。正式由于这一点在 run() 方法中间打断更像是抛出异常。为了以此方式终止任务别切返回良好的状态你必须仔细考虑代码的执行路径，并正确编写 catch 子句正确清理所有的事物。

Thread 类包含 interrupt() 方法，因此你可以终止被阻塞的任务，这个方法将设置线程的中断状态。如果一个线程被阻塞或者试图执行一个阻塞操作，那么设置这个线程的阻塞状态将抛出 InterruptedException。当抛出该异常或者改任务调用 Thread.interrupt() 时，中断状态将被复位。Thread.interrupt() 提供了离开 run() 循环而不抛出异常的办法。

Java 的新类库尽量避免我们直接操作 Thread 对象，而是尽量使用 Executor 来执行所有操作。如果你在 Executor 上调用 shutdownNow() ，那么将会发送一个 interrupt() 给由它启动的所有线程。然而有时我们需要的是中断其中的某一个任务。此时如果我们使用 Executor 调用 submit() 来启动任务，就可以持有该任务的上下文对象。submit() 将返回一个泛型的 Future<?>,其中有一个未修饰的参数，持有这个 Future 的关键是可以在其上调用 cancel()。那么就可以拥有在改线程上调用 interrupt() 以停止这个线程的权限。

下面的示例用 Executor 展示了基本的 interrupt() 用法：
```java
class SleepBlocked implements Runnable {
  public void run() {
    try {
      TimeUnit.SECONDS.sleep(100);
    } catch(InterruptedException e) {
      print("InterruptedException");
    }
    print("Exiting SleepBlocked.run()");
  }
}

class IOBlocked implements Runnable {
  private InputStream in;
  public IOBlocked(InputStream is) { in = is; }
  public void run() {
    try {
      print("Waiting for read():");
      in.read();
    } catch(IOException e) {
      if(Thread.currentThread().isInterrupted()) {
        print("Interrupted from blocked I/O");
      } else {
        throw new RuntimeException(e);
      }
    }
    print("Exiting IOBlocked.run()");
  }
}

class SynchronizedBlocked implements Runnable {
  public synchronized void f() {
    while(true) // Never releases lock
      Thread.yield();
  }
  public SynchronizedBlocked() {
    new Thread() {
      public void run() {
        f(); // Lock acquired by this thread
      }
    }.start();
  }
  public void run() {
    print("Trying to call f()");
    f();
    print("Exiting SynchronizedBlocked.run()");
  }
}

public class Interrupting {
  private static ExecutorService exec =
    Executors.newCachedThreadPool();
  static void test(Runnable r) throws InterruptedException{
    Future<?> f = exec.submit(r);
    TimeUnit.MILLISECONDS.sleep(100);
    print("Interrupting " + r.getClass().getName());
    f.cancel(true); // Interrupts if running
    print("Interrupt sent to " + r.getClass().getName());
  }
  public static void main(String[] args) throws Exception {
    test(new SleepBlocked());
    test(new IOBlocked(System.in));
    test(new SynchronizedBlocked());
    TimeUnit.SECONDS.sleep(3);
    print("Aborting with System.exit(0)");
    System.exit(0); // ... since last 2 interrupts failed
  }
}
```
执行结果：
```
Interrupting SleepBlocked
InterruptedException
Exiting SleepBlocked.run()
Interrupt sent to SleepBlocked
Waiting for read():
Interrupting IOBlocked
Interrupt sent to IOBlocked
Trying to call f()
Interrupting SynchronizedBlocked
Interrupt sent to SynchronizedBlocked
Aborting with System.exit(0)
```

上面的任务展示了三种不同的阻塞类型。SleepBlock 是可中断的阻塞类型，而 IOBlocked 和 SynchronizedBlocked 是不可中断的阻塞实例。从输出中可以看出，能够中断对 sleep() 的调用 (任何抛出 InterruptedException 的调用)。但是不能中断视图获取 synchronized 锁或者试图执行 I/O 操作的线程。

对于这类问题我们有一个行之有效的解决方案，即关闭任务在其发生阻塞的底层资源：
```java
public class CloseResource {
  public static void main(String[] args) throws Exception {
    ExecutorService exec = Executors.newCachedThreadPool();
    ServerSocket server = new ServerSocket(8080);
    InputStream socketInput =
      new Socket("localhost", 8080).getInputStream();
    exec.execute(new IOBlocked(socketInput));
    exec.execute(new IOBlocked(System.in));
    TimeUnit.MILLISECONDS.sleep(100);
    print("Shutting down all threads");
    exec.shutdownNow();
    TimeUnit.SECONDS.sleep(1);
    print("Closing " + socketInput.getClass().getName());
    socketInput.close(); // Releases blocked thread
    TimeUnit.SECONDS.sleep(1);
    print("Closing " + System.in.getClass().getName());
    System.in.close(); // Releases blocked thread
  }
}
```
执行结果：
```
Waiting for read():
Waiting for read():
Shutting down all threads
Closing java.net.SocketInputStream
Interrupted from blocked I/O
Exiting IOBlocked.run()
Closing java.io.BufferedInputStream
Exiting IOBlocked.run()
```
在 shutdownNow() 被调用之后和两个输入流被调用 close() 之前的延迟强调的是一旦底层资源被关闭，任务将解除阻塞。

**被互斥所阻塞**

如果你尝试在一个对象上调用其 synchronized 方法，而这个对象的锁已经被其他任务获得，那么调用任务将会被挂起，直至这个锁被获得。下面的示例说明了同一个互斥可以被同一个任务多次获得：
```java

public class MultiLock {

	public synchronized void f1(int count) {
		if (count-- >0) {
			System.out.println("f1 调用 f2" + count);
			f2(count);
		}
	}

	public synchronized void f2(int count) {
		if (count-- >0) {
			System.out.println("f2 调用 f1" + count);
			f1(count);
		}
	}

	public static void main(String[] args) throws Exception{
		final MultiLock multiLock = new MultiLock();
		new Thread(){
			public void run() {
				multiLock.f1(5);
			};
		}.start();
	}
}
```
执行结果：
```
f1 调用 f24
f2 调用 f13
f1 调用 f22
f2 调用 f11
f1 调用 f20
```
同一个任务先获得了 f1() 中的锁，又获得了 f2() 中的锁。这么做是有意义的，因为一个任务能够调用同一个对象中的其他的 synchronized 方法，而这个任务已经持有了锁。

前面 I/O 的例子我们观察到，只要任务以不可中断的方式被阻塞，那么都有潜在的锁住程序的可能。Java SE5 类库中添加了一个特性，即在 ReentrantLock 上阻塞的任务具备可以被中断的能力，：

加锁的对象：
```java
public class BlockedMutex {
	private Lock lock = new ReentrantLock();

	public BlockedMutex() {
		// TODO Auto-generated constructor stub
		lock.lock();
	}

	public void f() {
		// TODO Auto-generated method stub
		try {
			//可以被中断
			lock.lockInterruptibly();
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			System.out.println("BlockMutex 被中断异常");
		}
	}
}

```
任务：
```java
public class Blocked2 implements Runnable{

	BlockedMutex mutex = new BlockedMutex();

	@Override
	public void run() {
		System.out.println("开始线程执行");
		mutex.f();
		System.out.println("被中断了");
	}

}
```
线程启动任务，并中断：
```java
public class Interrupting2 {

	public static void main(String[] args) throws Exception{
		// TODO Auto-generated method stub
		Thread thread = new Thread(new Blocked2());
		thread.start();
		TimeUnit.SECONDS.sleep(1);
		System.out.println("线程调用 interrupt(");
		thread.interrupt();
	}

}

```
执行结果：
```
开始线程执行
线程调用 interrupt(
BlockMutex 被中断异常
被中断了
```

#### 检查中断
注意，当你在线程上调用 interrupt() 时，中断发生的唯一时刻是在任务要进入到阻塞操作中，或者已经在阻塞操作内部时。如果你的 run() 方法没有产生任何阻塞调用的情况下，调用 interrupt() 将无法停止某个任务。你的任务此时需要第二种方式退出。

这种机会是由中断状态来表示的，其状态可以通过调用 interrupt() 来设置。你可以调用 interrupted() 来检查中断状态，这不仅可以告诉你 interrupt() 是否可以被调用过，而且还可以清楚中断状态。清楚状态可以确保并发结构不会就某个任务被中断这个问题通知你两次，你可以经由单一的 InterruptedException 来得到通知。如果想再次检查了解是否被中断，则可以调用 Thread.interrupted() 时将结果保存起来。

下面展示了典型的管用方法：
```java
public class NeedsCleanup {
	private final int id;

	protected NeedsCleanup(int id) {
		super();
		this.id = id;
		System.out.println("NeedsCleanup:"+id);
	}

	public void cleanup() {
		// TODO Auto-generated method stub
		System.out.println("cleanup()"+id);
	}

}

```
创建任务：
```java
public class Blocked3 implements Runnable{
	private volatile double d= 0.0;

	@Override
	public void run() {
		while (!Thread.interrupted()) {
			NeedsCleanup nCleanup = new NeedsCleanup(1);
			try {
				TimeUnit.SECONDS.sleep(1);
				NeedsCleanup nCleanup2 = new NeedsCleanup(2);
				try {
					for (int i = 0; i < 2500000; i++) {
						d = d+(Math.PI +Math.E)/d;
					}
					System.out.println("计算完毕");
				} finally {
					nCleanup2.cleanup();
				}
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}finally {
				nCleanup.cleanup();
			}

		}

	}

}

```
创建测试类：
```java
public class InterruptingIdiom {

	public static void main(String[] args) throws Exception{
		// TODO Auto-generated method stub
		Thread thread = new Thread(new Blocked3());
		thread.start();
		TimeUnit.MICROSECONDS.sleep(3);
		thread.interrupt();
	}

}

```
测试结果：
```
NeedsCleanup:1
java.lang.InterruptedException: sleep interrupted
cleanup()1
NeedsCleanup:1
	at java.lang.Thread.sleep(Native Method)
	at java.lang.Thread.sleep(Thread.java:340)
	at java.util.concurrent.TimeUnit.sleep(TimeUnit.java:386)
	at concurrency.Blocked3.run(Blocked3.java:13)
	at java.lang.Thread.run(Thread.java:745)
NeedsCleanup:2
计算完毕
cleanup()2
cleanup()1
NeedsCleanup:1
NeedsCleanup:2
/....
```
注意看第一条的异常，NeedsCleanup 类强调在你经由异常离开循环时，正确清理资源的必要性。注意:所有的 run() 方法中创建的资源都必须在其后紧跟 try-finally 子句，以确保 nCleanup() 方法总会被调用。
