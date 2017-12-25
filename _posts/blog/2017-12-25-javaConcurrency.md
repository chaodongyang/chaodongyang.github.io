---
layout: post
title: java编程思想之并发(认识多线程)
categories: javaThinking
description: java编程思想之并发
keywords: java, java并发, java多线程，Java多线程并发
---
编程问题中的相当一大部分都可以通过顺序编程来解决。然而，对于某些问题，如果能够并行的执行程序中的多个部分，则会变得非常方便甚至非常必要，这些部分要么可以并发执行，要么在多处理器环境下可以同时执行。

并发编程可以使程序执行速度得到极大的提高，或者为设计某些类型的程序提供更简单的模型。学习并发编程就像进入一个全新的领域，有点类似于学习一门新的编程语言，或者是学习一整套新的语言概念。要理解并发编程与理解面向对象编程差不多。要想真正的掌握它的实质，就需要深入的学习和理解。

## 并发的多面性
并发编程令人困惑的一个重要原因是：使用并发时需要解决的问题有多个，而实现并发的方式也有多种，并且这两者之间没有明显的映射关系。因此我们必须理解所有的这些问题和特例，以便有效的使用并发。

#### 更快的执行
如果你想让一个程序运行的更快，那么可以将其断开为多个片段，在单独的处理器上运行每个片段。并发是用于多处理器编程的基本工具。当前速度的提高是以多核处理器的形式而不是更快的芯片的形式出现的。为了使程序运行的更快，你必须学习如何利用这些额外的处理器，而这正是并发赋予你的能力。

但是，并发通常是提高运行在单处理器上的程序的性能。

听起来好像有什么不对。思考一下在单处理器上运行的并发编程开销确实比改程序的所有部分都顺序执行的开销大，因为并发增加了上下文的切换的代价。表面上看，将程序的所有部分当做单个的任务运行好像是开销更小一点。但是我们要考虑到阻塞的问题。如果程序中的某个任务因为程序控制范围之外的某个条件而导致不能继续执行，那么我们就说这个任务或线程阻塞了。如果没有并发，则整个程序都将停下来，直至外部条件发生变化。但是，如果使用了并发来编写程序，那么当一个任务阻塞时，程序中的其他任务还可以继续执行，因此这个程序可以保持继续向前执行。事实上，从性能角度看，如果没有任务会阻塞，那么在单处理器机器上使用并发就没有任何意义。

#### 改进代码设计
Java 的线程机制是抢占式的，这表示调度机制会周期性地中断线程，将上下文切换到另一个线程，从而为每个线程都提供时间片段，使得每个线程都分配到数量合理的时间去驱动它的任务。在协作式系统中，每个任务都会自动的放弃控制，这要求程序员有意识的在每个任务中插入让步语句。协作系统的优势是双重的：上下文切换的开销比抢占式要低廉的多，可以同时执行的线程数量理论上没有限制。当你处理大量的仿真元素时，这是一种理想的解决方案。但是注意，某些协作式系统并未设计为可以在多个处理器之间分配任务，这可能会非常有限。

并发需要付出代价，但这些大家与在程序设计、资源负载均衡以及用户方便方面的改进相比，就显得微不足道。通常，线程能够使我们创建更加松耦合的设计。

## 线程的基本机制
并发编程使得我们可以将程序划分为多个分离的、独立的任务。通过使用多线程机制，这些独立任务中的每一个都将由执行程序来驱动。一个线程就是进程中的一个单一的顺序控制流，因此，单个进程可以拥有多个并发执行的任务，但是你的程序使得每个任务都好像有其自己的 CPU 一样。其底层机制是切分 CPU 时间，但我们通常不需要考虑他。

线程模型为编程带来了便利。它简化了在单一程序中同时多个操作的处理。在使用线程时，CPU 将轮流给每个任务分配其占用时间。每个人物都觉得自己在一直占用 CPU，但事实上 CPU 时间是划分片段分配给了所有任务（也有可能是运行是多个 cpu 之上）。线程的一大好处是可以使你从这个层次抽身出来，即代码不需要知道它是运行在一个还是多个 CPU 上。所以，使用线程机制是一个建立透明的，可扩展的程序的方法，如果程序运行速度太慢，为机器增添一个 CPU 就很容易的增加程序运行的速度。多个任务，多个线程是使用多处理器系统的最合理方式。

#### 定义任务
线程可以驱动任务，因此需要一种描述任务的方式，这可以由 Runnable 接口来提供。要想定义任务，只需要实现 Runnable 接口并编写 run() 方法，使得该任务可以执行你的命令。
```java
public class LiftOff implements Runnable{
	  protected int countDown = 10; // Default
	  private static int taskCount = 0;
	  private final int id = taskCount++;
	  public LiftOff() {}
	  public LiftOff(int countDown) {
	    this.countDown = countDown;
	  }
	  public String status() {
	    return "#" + id + "(" +
	      (countDown > 0 ? countDown : "Liftoff!") + "), ";
	  }
	  public void run() {
	    while(countDown-- > 0) {
	      System.out.print(status());
	      Thread.yield();
	    }
	  }

}

```
任务的 run() 方法通常会有某种形式的循环，使得任务一直运行下去直到不再需要，所以要设定跳出循环的条件。在 run() 方法中对静态方法 Thread.yield() 的调用是对线程调度器的一种建议，线程调度器是 Java 多线程机制的一部分，可以将 cpu 从一个线程转移到另一个线程。它声明了，我们已经执行完生命周期中最重要的一部分，此刻正是切换给其他任务执行的大好时机。

下面示例，任务的 run() 在 main() 方法中直接被调用：
```java
public class MainThread {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		LiftOff liftOff = new LiftOff();
		liftOff.run();
	}

}

```
执行结果：
```
#0(9), #0(8), #0(7), #0(6), #0(5), #0(4), #0(3), #0(2), #0(1), #0(Liftoff!),
```
从 Runnable 导出一个类，他必须实现 run() 方法，他没有任何内在线程的能力。要实现线程的行为，必须显式的给一个任务赋予它。

#### Thread 类
将 Runnable 对象转变为一个工作任务的方式是把它提交给一个 Thread 构造器，下面是示例：
```java
public class BasicThreads {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Thread thread = new Thread(new LiftOff());
		thread.start();
		System.out.println("任务开始");
	}

}

```
执行结果：
```
任务开始
#0(9), #0(8), #0(7), #0(6), #0(5), #0(4), #0(3), #0(2), #0(1), #0(Liftoff!),
```
Thread 构造器只需要一个 Runnable 对象。调用 start() 方法为该线程执行提供必须的初始化操作，然后调用 Runnable 的 run() 方法，以便在这个线程中启动任务。我们看到输出语句先输出了，任务的语句后输出了。这表明 start() 语句直接返回了。实际上只是产生了对 LiftOff.run() 方法的调用，并且这个方法还没有完成，但是由于 run() 方法是由不同的线程执行的，所以 main() 方法中的任务还可以继续执行。因此，程序会同时运行两个方法。

下面示例添加更多的任务执行：
```java
public class MoreBasicThreads {
  public static void main(String[] args) {
    for(int i = 0; i < 5; i++)
      new Thread(new LiftOff()).start();
    System.out.println("Waiting for LiftOff");
  }
}
```
执行结果：
```
Waiting for LiftOff
#0(9), #1(9), #2(9), #3(9), #4(9), #0(8), #1(8), #2(8), #3(8), #4(8), #0(7), #1(7), #2(7), #3(7), #4(7), #0(6), #1(6), #2(6), #3(6), #4(6), #0(5), #1(5), #2(5), #3(5), #4(5), #0(4), #1(4), #2(4), #3(4), #4(4), #0(3), #1(3), #2(3), #3(3), #4(3), #0(2), #1(2), #2(2), #3(2), #4(2), #0(1), #1(1), #2(1), #3(1), #4(1), #0(Liftoff!), #1(Liftoff!), #2(Liftoff!), #3(Liftoff!), #4(Liftoff!),
```
输出结果说明不同任务的执行被混在了一起。这种交换是由线程调度器自动控制的。如果你有多个处理器，线程调度器就会在这些处理器之间分发线程。当 main() 创建 Thread 对象时，它并没有捕获任何对这些对象的引用。在使用普通对象时，对于垃圾回收器是一种公平的游戏，但是在使用 Thread　时，情况就不同。每个 Thread 都注册了自己，存在一个对它的引用，而且在任务退出 run() 死亡之前，垃圾回收器无法清楚它。

#### 使用 Executor
Java SE5 的 java.util.concurrent 包中的执行器 (Executor) 将为你管理 Thread 对象，简化了并发编程。Executor 在客户端和任务之间建立了一个中间层；与客户端直接执行任务不同，这个中介将直接执行任务。Executor 允许你管理异步任务的执行，而无需显示的管理线程和生命周期。我们可以使用 Executor 来替代在上个示例中显示的创建 Thread　对象。
```java
public class CachedThreadPool {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		ExecutorService executorService = Executors.newCachedThreadPool();
		for (int i = 0; i < 3; i++) {
			executorService.execute(new LiftOff());
		}
		executorService.shutdown();
	}

}
```
执行结果：
```
#2(9), #1(9), #0(9), #2(8), #2(7), #2(6), #2(5), #1(8), #1(7), #2(4), #0(8), #2(3), #2(2), #1(6), #2(1), #0(7), #2(Liftoff!), #1(5), #0(6), #1(4), #0(5), #0(4), #1(3), #0(3), #1(2), #0(2), #1(1), #0(1), #1(Liftoff!), #0(Liftoff!),
```
shutdown() 方法的调用可以防止新任务被提交给这个 Executor ，当前线程将继续运行在 shutdown() 被提交之前提交的所有任务。

我们可以使用不同类型的 Executor。
```java
public class CachedThreadPool {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		ExecutorService executorService = Executors.newFixedThreadPool(3);
		for (int i = 0; i < 2; i++) {
			executorService.execute(new LiftOff());
		}
		executorService.shutdown();
	}

}

```
执行结果：
```
#0(9), #1(9), #0(8), #1(8), #0(7), #1(7), #1(6), #1(5), #1(4), #1(3), #1(2), #1(1), #0(6), #0(5), #0(4), #1(Liftoff!), #0(3), #0(2), #0(1), #0(Liftoff!),
```
FixedThreadPool 使用了有限的线程集来执行提交的任务，你可以一次性预先执行代价高昂的线程分配，也可以限制线程的数量。这可以节省时间，因为你不用为每个任务都固定的去创建线程。注意：在任何线程池中，现有线程在可能的情况下都会复用。CachedThreadPool 在程序执行过程中通常会创建于所需要数量相同的线程，然后在它回收旧线程时停止创建新的线程，因此它是首选。只有当这种方式引发问题时才需要切换到 FixedThreadPool。

SingleThreadExecutor 就像是线程数量为 1 的 FixedThreadPool。如果向其中提交了多个任务，那么这些任务将排队，每个任务都会在下一个任务开始之前结束，所有的任务将使用相同的线程。

下面的示例你会看到每个任务都是按照它提交的顺序在下一个任务开始之前完成的。
```java
public class CachedThreadPool {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		ExecutorService executorService = Executors.newSingleThreadExecutor();
		for (int i = 0; i < 2; i++) {
			executorService.execute(new LiftOff());
		}
		executorService.shutdown();
	}

}

```
执行结果：
```
#0(9), #0(8), #0(7), #0(6), #0(5), #0(4), #0(3), #0(2), #0(1), #0(Liftoff!), #1(9), #1(8), #1(7), #1(6), #1(5), #1(4), #1(3), #1(2), #1(1), #1(Liftoff!),
```
假如你有大量的任务将使用文件系统。你可以运用 SingleThreadExecutor 来运行这些线程，以确保任意时刻在任何线程中都只要唯一的任务在运行。这种方式你不需要再共享资源上同步。

#### 从任务中产生返回值
Runnable 是执行工作的独立任务，但是他不反回任何值。如果你希望在任务执行完成时能够返回值，那么可以实现 Callable 接口。它是具有类型参数的泛型，它的类型参数表示的是从方法 call() 中返回的值，并且必须使用 ExecutorService.submit() 方法调用它，看示例代码：
```java
public class TaskWithResult implements Callable<String>{

	private int id;


	protected TaskWithResult(int id) {
		super();
		this.id = id;
	}

	@Override
	public String call() throws Exception {
		// TODO Auto-generated method stub
		return "任务执行完毕"+id;
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		ExecutorService executorService = Executors.newCachedThreadPool();
		ArrayList<Future<String>> result = new ArrayList<>();
		for (int i = 0; i < 3; i++) {
			result.add(executorService.submit(new TaskWithResult(i)));
		}

		for (Future<String> future : result) {
			try {
				System.out.println(future.get());
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			} catch (ExecutionException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}finally {
				executorService.shutdown();
			}
		}
	}

}

```
执行结果：
```
任务执行完毕0
任务执行完毕1
任务执行完毕2

```
submit 方法会产生 Future 对象，它用 Callable 返回结果的特定类型进行了参数化。可以使用 isDone() 方法查询 Future 对象是否完成。当任务完成时可以调用 get() 方法获取结果。

#### 休眠
影响任务行为的一种简单方法是调用 sleep()，这将使任务终止执行给定的时间。
```java
public class SleepingTask extends LiftOff {
	@Override
	public void run() {
		// TODO Auto-generated method stub
		 try {
		      while(countDown-- > 0) {
		        System.out.print(status());
		        // Old-style:
		        // Thread.sleep(100);
		        // Java SE5/6-style:
		        TimeUnit.MILLISECONDS.sleep(100);
		      }
		    } catch(InterruptedException e) {
		      System.err.println("Interrupted");
		    }
	}

	public static void main(String[] args) {
		 ExecutorService exec = Executors.newCachedThreadPool();
		    for(int i = 0; i < 5; i++)
		      exec.execute(new SleepingTask());
		    exec.shutdown();
	}
}

```
对 sleep() 的调用会抛出异常，并且可以看到，它在 run() 中被捕获。Java SE5 中引入更显示的 sleep() 版本，作为 TimeUnit 类的一部分，这个方法允许你指定 sleep 延迟的时间单元，因此可以提供更好的可阅读性。TimeUnit 还可以被用来执行转换。

#### 优先级
线程的优先级将线程的重要性传递给调度器。尽管 CPU 处理线程集的顺序是不确定的，但是调度器将倾向于让优先权告的线程先执行。然而并不意味着优先级低的线程得不到执行。优先级低的线程仅仅意味着执行的频率较低。

在绝大多数时间所有的线程都应该以默认的优先级运行。视图操纵优先级并不提倡。

下面是一个演示优先级等级的示例，可以使用 getPriority() 来读取现有线程的优先级，可以通过 setPriority() 来修改它。
```java
public class SimplePriorities implements Runnable{

	private int countDown = 5;
    private volatile double d; // No optimization
    private int priority;

	protected SimplePriorities(int priority) {
		super();
		this.priority = priority;
	}

	public String toString() {
	    return Thread.currentThread() + ": " + countDown;
	}

	@Override
	public void run() {
		 Thread.currentThread().setPriority(priority);
		    while(true) {
		      // An expensive, interruptable operation:
		      for(int i = 1; i < 100000; i++) {
		        d += (Math.PI + Math.E) / (double)i;
		        if(i % 1000 == 0)
		          Thread.yield();
		      }
		      System.out.println(this);
		      if(--countDown == 0) return;
		    }
	}

	public static void main(String[] args) {
		 ExecutorService exec = Executors.newCachedThreadPool();
		    for(int i = 0; i < 5; i++)
		      exec.execute(new SimplePriorities(Thread.MIN_PRIORITY));
		    exec.execute(new SimplePriorities(Thread.MAX_PRIORITY));
		    exec.shutdown();
	}

}

```
执行结果：
```
Thread[pool-1-thread-6,10,main]: 5
Thread[pool-1-thread-1,1,main]: 5
Thread[pool-1-thread-5,1,main]: 5
Thread[pool-1-thread-2,1,main]: 5
Thread[pool-1-thread-6,10,main]: 4
Thread[pool-1-thread-4,1,main]: 5
Thread[pool-1-thread-3,1,main]: 5
Thread[pool-1-thread-6,10,main]: 3
Thread[pool-1-thread-6,10,main]: 2
Thread[pool-1-thread-1,1,main]: 4
Thread[pool-1-thread-5,1,main]: 4
Thread[pool-1-thread-2,1,main]: 4
Thread[pool-1-thread-4,1,main]: 4
Thread[pool-1-thread-3,1,main]: 4
Thread[pool-1-thread-6,10,main]: 1
Thread[pool-1-thread-5,1,main]: 3
Thread[pool-1-thread-1,1,main]: 3
Thread[pool-1-thread-2,1,main]: 3
Thread[pool-1-thread-4,1,main]: 3
Thread[pool-1-thread-3,1,main]: 3
Thread[pool-1-thread-5,1,main]: 2
Thread[pool-1-thread-4,1,main]: 2
Thread[pool-1-thread-2,1,main]: 2
Thread[pool-1-thread-3,1,main]: 2
Thread[pool-1-thread-1,1,main]: 2
Thread[pool-1-thread-5,1,main]: 1
Thread[pool-1-thread-2,1,main]: 1
Thread[pool-1-thread-4,1,main]: 1
Thread[pool-1-thread-3,1,main]: 1
Thread[pool-1-thread-1,1,main]: 1

```
我们使用了大量的运算来测试，观察到优先级为 MAX_PRIORITY 的线程被线程调度器优先选择。注意：JDK 有 10 个优先等级，但是与大多数操作系统的映射不好。比如，windows 有 7 个优先级切不固定，所以这种映射关系也是不确定的。

#### 让步
如果你已经知道你的一次循环迭代过程中的工作已经完成，就可以给线程调度机制一个暗示：你的工作完成的差不多了，可以让别的线程使用 CPU 了。这个暗示将通过调用 yield() 来完成。注意，这只是一种暗示，没有任何机制保证它将会被采纳。当调用 yield() 时，你也是在建议具有相同优先级的其他线程可以运行。

#### 后台线程
后台线程就是指在程序运行的时候在后台提供一种通用服务的线程，这种线程不是程序必须的一部分。因此，当所有的非后台线程结束时，程序也就终止了，同时户杀死进程中的所有后台进程。反过来说，只要有任何非后台进程还在运行，程序就不会被终止。比如 main() 就是一个非后台线程。
```java
public class SimpleDaemons implements Runnable {
	  public void run() {
		    try {
		      while(true) {
		        TimeUnit.MILLISECONDS.sleep(100);
		        Print.print(Thread.currentThread() + " " + this);
		      }
		    } catch(InterruptedException e) {
		    	Print.print("sleep() interrupted");
		    }
		  }
		  public static void main(String[] args) throws Exception {
		    for(int i = 0; i < 10; i++) {
		      Thread daemon = new Thread(new SimpleDaemons());
		      //比如在调用之前设置为后台线程才会生效
		      daemon.setDaemon(true); // Must call before start()
		      daemon.start();
		    }
		    Print.print("All daemons started");
		    TimeUnit.MILLISECONDS.sleep(175);
		  }
}

```
执行结果：
```
All daemons started
Thread[Thread-8,5,main] concurrency.SimpleDaemons@790d3283
Thread[Thread-5,5,main] concurrency.SimpleDaemons@7e00f258
Thread[Thread-4,5,main] concurrency.SimpleDaemons@ca4a1b4
Thread[Thread-0,5,main] concurrency.SimpleDaemons@fcc3aac
Thread[Thread-9,5,main] concurrency.SimpleDaemons@1c4b4746
Thread[Thread-6,5,main] concurrency.SimpleDaemons@27280786
Thread[Thread-7,5,main] concurrency.SimpleDaemons@11af0a8d
Thread[Thread-2,5,main] concurrency.SimpleDaemons@3b1c2f38
Thread[Thread-3,5,main] concurrency.SimpleDaemons@67deccdf
Thread[Thread-1,5,main] concurrency.SimpleDaemons@12d687a2
```
必须在线程启动之前调用 setDaemon() 方法，才能把他设置为后台线程。

可以通过 isDaemon() 方法来确定线程是否是一个后台线程。如果是一个后台线程，那么它创建的任何线程都将被自动设置为后台线程。示例：
```java
public class Daemon implements Runnable {
	  private Thread[] t = new Thread[10];
	  public void run() {
	    for(int i = 0; i < t.length; i++) {
	      t[i] = new Thread(new DaemonSpawn());
	      t[i].start();
	      Print.printnb("DaemonSpawn " + i + " started, ");
	    }
	    for(int i = 0; i < t.length; i++)
	    	Print.printnb("t[" + i + "].isDaemon() = " +
	        t[i].isDaemon() + ", ");
	    while(true)
	      Thread.yield();
	  }
	}

	class DaemonSpawn implements Runnable {
	  public void run() {
	    while(true)
	      Thread.yield();
	  }
	}

```
测试类：
```
public class Daemons {

	public static void main(String[] args) {
		 Thread d = new Thread(new Daemon());
		  d.setDaemon(true);
		  d.start();
		  Print.printnb("d.isDaemon() = " + d.isDaemon() + ", ");
		    // Allow the daemon threads to
		    // finish their startup processes:
		  try {
			TimeUnit.SECONDS.sleep(1);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}

}

```
执行结果：
```
DaemonSpawn 0 started, d.isDaemon() = true, DaemonSpawn 1 started, DaemonSpawn 2 started, DaemonSpawn 3 started, DaemonSpawn 4 started, DaemonSpawn 5 started, DaemonSpawn 6 started, DaemonSpawn 7 started, DaemonSpawn 8 started, DaemonSpawn 9 started, t[0].isDaemon() = true, t[1].isDaemon() = true, t[2].isDaemon() = true, t[3].isDaemon() = true, t[4].isDaemon() = true, t[5].isDaemon() = true, t[6].isDaemon() = true, t[7].isDaemon() = true, t[8].isDaemon() = true, t[9].isDaemon() = true,
```
Daemon 被设置为了后台线程，然后派生出很多子线程，这些线程并没有被显式的设置为后台模式，不过他们却是后台线程。

下面的例子，后台线程在不执行 finally 的时候就会终止其 run() 方法：
```java
class ADaemon implements Runnable {
  public void run() {
    try {
      print("Starting ADaemon");
      TimeUnit.SECONDS.sleep(1);
    } catch(InterruptedException e) {
      print("Exiting via InterruptedException");
    } finally {
      print("This should always run?");
    }
  }
}

public class DaemonsDontRunFinally {
  public static void main(String[] args) throws Exception {
    Thread t = new Thread(new ADaemon());
    t.setDaemon(true);
    t.start();
  }
}
```
上面的程序你将看到 finally 子句不会执行，但是如果注释掉 setDaemon() 方法，就会看到 finally 子句被执行。这是因为当最后一个非后台线程终止时，后台线程会突然终止。因此一旦 main() 方法退出，JVM 就会立即关闭所有的后台线程。

#### 编码的变体
上面的例子我们都是直接实现 Runnable 接口。我们也可以直接从 Thread 继承这种可替换的方式，示例：
```java
public class SimpleThread extends Thread{
	private int countDown = 5;
	private static int threadCount = 0;
	protected SimpleThread() {
		super(Integer.toString(++threadCount));
		start();
	}

	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return"#" + getName()+"(" +countDown + ")";
	}

	@Override
	public void run() {
		// TODO Auto-generated method stub
		while (true) {
			System.out.println(this);
			if (--countDown ==0) {
				return;
			}
		}
	}

	public static void main(String[] args) {
		for (int i = 0; i < 2; i++) {
			new SimpleThread();
		}
	}

}

```
执行结果：
```
#2(5)
#1(5)
#2(4)
#1(4)
#2(3)
#1(3)
#2(2)
#1(2)
#1(1)
#2(1)

```
另外一种常用的方法是自管理的 Runnable:
```java
public class SelfManaged implements Runnable {
	  private int countDown = 5;
	  private Thread t = new Thread(this);
	  public SelfManaged() { t.start(); }
	  public String toString() {
	    return Thread.currentThread().getName() +
	      "(" + countDown + "), ";
	  }
	  public void run() {
	    while(true) {
	      System.out.print(this);
	      if(--countDown == 0)
	        return;
	    }
	  }
	  public static void main(String[] args) {
	    for(int i = 0; i < 2; i++)
	      new SelfManaged();
	  }
}

```
执行结果：
```
Thread-1(5), Thread-0(5), Thread-1(4), Thread-0(4), Thread-1(3), Thread-0(3), Thread-1(2), Thread-0(2), Thread-1(1), Thread-0(1),
```
注意：start() 是在构造器中被调用的。但是应该意识到，在构造器中启动线程可能会变得有问题，因为另一个任务可能在构造器结束之前开始执行，这意味着该任务能够访问处于不稳定状态的对象。这也是我们优先选择 Executor 而不是显示的创建 Thread　的原因。

有时通过使用内部类将线程的代码隐藏在类中将会很有用：
```java
class InnerThread1 {
  private int countDown = 5;
  private Inner inner;
  private class Inner extends Thread {
    Inner(String name) {
      super(name);
      start();
    }
    public void run() {
      try {
        while(true) {
          print(this);
          if(--countDown == 0) return;
          sleep(10);
        }
      } catch(InterruptedException e) {
        print("interrupted");
      }
    }
    public String toString() {
      return getName() + ": " + countDown;
    }
  }
  public InnerThread1(String name) {
    inner = new Inner(name);
  }
}

// Using an anonymous inner class:
class InnerThread2 {
  private int countDown = 5;
  private Thread t;
  public InnerThread2(String name) {
    t = new Thread(name) {
      public void run() {
        try {
          while(true) {
            print(this);
            if(--countDown == 0) return;
            sleep(10);
          }
        } catch(InterruptedException e) {
          print("sleep() interrupted");
        }
      }
      public String toString() {
        return getName() + ": " + countDown;
      }
    };
    t.start();
  }
}

// Using a named Runnable implementation:
class InnerRunnable1 {
  private int countDown = 5;
  private Inner inner;
  private class Inner implements Runnable {
    Thread t;
    Inner(String name) {
      t = new Thread(this, name);
      t.start();
    }
    public void run() {
      try {
        while(true) {
          print(this);
          if(--countDown == 0) return;
          TimeUnit.MILLISECONDS.sleep(10);
        }
      } catch(InterruptedException e) {
        print("sleep() interrupted");
      }
    }
    public String toString() {
      return t.getName() + ": " + countDown;
    }
  }
  public InnerRunnable1(String name) {
    inner = new Inner(name);
  }
}

// Using an anonymous Runnable implementation:
class InnerRunnable2 {
  private int countDown = 5;
  private Thread t;
  public InnerRunnable2(String name) {
    t = new Thread(new Runnable() {
      public void run() {
        try {
          while(true) {
            print(this);
            if(--countDown == 0) return;
            TimeUnit.MILLISECONDS.sleep(10);
          }
        } catch(InterruptedException e) {
          print("sleep() interrupted");
        }
      }
      public String toString() {
        return Thread.currentThread().getName() +
          ": " + countDown;
      }
    }, name);
    t.start();
  }
}

// A separate method to run some code as a task:
class ThreadMethod {
  private int countDown = 5;
  private Thread t;
  private String name;
  public ThreadMethod(String name) { this.name = name; }
  public void runTask() {
    if(t == null) {
      t = new Thread(name) {
        public void run() {
          try {
            while(true) {
              print(this);
              if(--countDown == 0) return;
              sleep(10);
            }
          } catch(InterruptedException e) {
            print("sleep() interrupted");
          }
        }
        public String toString() {
          return getName() + ": " + countDown;
        }
      };
      t.start();
    }
  }
}

public class ThreadVariations {
  public static void main(String[] args) {
    new InnerThread1("InnerThread1");
    new InnerThread2("InnerThread2");
    new InnerRunnable1("InnerRunnable1");
    new InnerRunnable2("InnerRunnable2");
    new ThreadMethod("ThreadMethod").runTask();
  }
}
```
如果内部类具有你在其他的方法中需要访问的特殊能力，那这么做将会很有意义。但是，在大多数时候，创建线程的原因只是为了使用 Thread 的能力，因此不必创建匿名内部类。

#### 加入一个线程
一个线程可以在其他线程之上调用 join() 方法，其效果是等待一段时间直到第二个线程结束才继续执行。如果某个线程在另一个线程 t 上调用 join() 方法，此线程将会被挂起，直到目标线程 t 结束才恢复。也可以在调用 join() 时带上一个超时参数，这样如果目标线程在这段时期没有完成结束，join() 方法总能返回。对 join() 方法的调用可以被中断，做法是在调用线程上调用 interrupt() 方法。

下面的例子演示了所有的操作：

线程一：
```java
public class Sleeper extends Thread{
	  private int duration;
	  public Sleeper(String name, int sleepTime) {
	    super(name);
	    duration = sleepTime;
	    start();
	  }
	  public void run() {
	    try {
	      sleep(duration);
	    } catch(InterruptedException e) {
	    	Print.print(getName() + " was interrupted. " +
	        "isInterrupted(): " + isInterrupted());
	      return;
	    }
	    Print.print(getName() + " has awakened");
	  }
}

```
线程二：
```java
public class Joiner extends Thread {
	  private Sleeper sleeper;
	  public Joiner(String name, Sleeper sleeper) {
	    super(name);
	    this.sleeper = sleeper;
	    start();
	  }
	  public void run() {
	   try {
	      sleeper.join();
	    } catch(InterruptedException e) {
	      Print.print("Interrupted");
	    }
	   Print.print(getName() + " join completed");
	  }
	}

```
测试类：
```java
public class Joining {
	public static void main(String[] args) {
	    Sleeper
	      sleepy = new Sleeper("Sleepy", 1500),
	      grumpy = new Sleeper("Grumpy", 1500);
	    Joiner
	      dopey = new Joiner("Dopey", sleepy),
	      doc = new Joiner("Doc", grumpy);
	    grumpy.interrupt();
	  }
}

```
执行结果：
```
Grumpy was interrupted. isInterrupted(): false
Doc join completed
Sleepy has awakened
Dopey join completed
```
执行结果是这样的，先输出了前两句，当 Doc 被执行时，此时 Sleeper 里边的 hoin() 方法被挂起。休眠时间结束调用了 interrupt() 结束挂起之后线程又开始执行。

#### 捕获异常
由于线程的本质特征，使得你不能捕获从线程中逃逸的异常。一旦异常逃出任务的 run() 方法，它就会向外传播到控制台，除非你采取特殊的步骤捕获这种错误的异常。在 Java SE5 之后，可以用 Executor 来解决这个问题。

下面的任务总是会抛出一个异常，该异常会传播到其 run() 方法的外部：
```java
public class ExceptionThread implements Runnable{

	@Override
	public void run() {
		// TODO Auto-generated method stub
		throw new RuntimeException();
	}
	public static void main(String[] args) {
		 ExecutorService exec = Executors.newCachedThreadPool();
		 exec.execute(new ExceptionThread());
	}
}

```
执行结果：
```
Exception in thread "pool-1-thread-1" java.lang.RuntimeException
	at concurrency.ExceptionThread.run(ExceptionThread.java:11)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at java.lang.Thread.run(Thread.java:745)

```
我们把调用语句加入到 try-catch 语句块中：
```java
public static void main(String[] args) {
		 try {
			ExecutorService exec = Executors.newCachedThreadPool();
			 exec.execute(new ExceptionThread());
		} catch (Exception e) {
			// TODO Auto-generated catch block
			//e.printStackTrace();
		}
	}
```
执行结果：
```
Exception in thread "pool-1-thread-1" java.lang.RuntimeException
	at concurrency.ExceptionThread.run(ExceptionThread.java:11)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at java.lang.Thread.run(Thread.java:745)
```
产生于前面相同的结果：未捕获异常。

为了解决这个问题，我们需要修改 Executor 产生线程的方式。Thread.UncaughtExceptionHandler 是 Java SE5 中的新的接口，它允许你在每个 Thread 对象上都附着一个异常处理器。它的 uncaughtException() 会在线程因未捕获异常面临死亡时调用。

我们创建一个 ThreadFactory 它将在每一个新创建的 Thread 对象上附着一个 Thread.UncaughtExceptionHandler。

首先自定义一个异常捕获器：
```java
public class MyUncaughtExceptionHandler implements UncaughtExceptionHandler{

	@Override
	public void uncaughtException(Thread t, Throwable e) {
		// TODO Auto-generated method stub
		System.out.println("自定义的异常捕获"+e);
	}

}
```
实现一个线程工厂，将产生的线程加入异常捕获：
```java
public class HandlerThreadFactory implements ThreadFactory{

	@Override
	public Thread newThread(Runnable r) {
		System.out.println("创建一个新的线程");
		Thread thread = new Thread(r);
		thread.setUncaughtExceptionHandler(new MyUncaughtExceptionHandler());
		System.out.println("添加异常捕获结束");
		return thread;
	}

}
```
测试代码：
```java
public class CaptureUncaughtException {

	public static void main(String[] args) {
		//添加进到构造方法
		ExecutorService executorService = Executors.newCachedThreadPool(new HandlerThreadFactory());
		executorService.execute(new ExceptionThread());
	}

}
```
执行结果：
```
创建一个新的线程
添加异常捕获结束
创建一个新的线程
添加异常捕获结束
自定义的异常捕获java.lang.RuntimeException
```
现在看到了未捕获的异常是通过 uncaughtException 来捕获的。

如果你要在代码中使用相同的异常处理器，那么更简单的方法是在 Thread 类中设置一个静态域，并将这个处理器设置为默认的异常捕获处理器：
```java
public class SettingDefaultHandler {
  public static void main(String[] args) {
    Thread.setDefaultUncaughtExceptionHandler(
      new MyUncaughtExceptionHandler());
    ExecutorService exec = Executors.newCachedThreadPool();
    exec.execute(new ExceptionThread());
  }
}
```
注意：默认的异常处理器只有在线程未设置专有的异常处理器情况下才会被调用。
