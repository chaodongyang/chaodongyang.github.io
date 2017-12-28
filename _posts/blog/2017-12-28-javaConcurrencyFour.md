---
layout: post
title: java编程思想之并发(线程之间的协作)
categories: javaThinking
description: java编程思想之并发
keywords: java, java并发, java多线程，Java多线程并发
---

当你使用多线程来同时运行多个任务时，可以通过使用锁来同步两个任务的行为，从而使的一个任务不会干涉另一个任务的资源。也就是说，如果两个任务交替的步入某项共享资源，你可以使用互斥来保证任何时刻只有一个任务可以访问这项资源。

## 线程之间的协作
上面的问题已经解决了，下一步是如何使得任务彼此之间可以协作，使得多个任务可以一起工作去解决某个问题。现在的问题不是彼此之间的干涉，而是彼此之间的协作。解决这类问题的关键是某些部分必须在其他部分被解决之前解决。

当任务协作时，关键问题是这些任务之间的握手。为了实现握手，我们使用了相同的基础特性：互斥。在这种情况下，互斥能够确保只有一个任务可以响应某个信号，这样就能根除任何可能的竞争条件。在互斥上，我们为任务添加了一种途径，可以将自身挂起，直至某些外部条件发生变化，表示是时候让这个任务开始为止。

#### wait() 与 notifyAll()
wait() 可以使你等待某个条件发生变化，而改变这个条件通常是由另一个任务来改变。你肯定不想在你的任务测试这个条件的同时，不断的进行空循环，这被称为忙等待，是一种不良的 cpu 使用方式。因此 wait() 会在外部条件发生变化的时候将任务挂起，并且只有在 notif() 或 notifAll() 发生时，这个任务才会被唤醒并去检查所发生的变化。因此，wait() 提供了一种在任务之间对活动同步的方式。

调用 sleep() 时候锁并没有被释放，调用 yield() 也是一样。当一个任务在方法里遇到对 wait() 调用时，线程执行被挂起，对象的锁被释放。这就意味着另一个任务可以获得锁，因此在改对象中的其他 synchronized 方法可以在 wait() 期间被调用。因此，当你在调用 wait() 时，就是在声明：“我已经做完了所有的事情，但是我希望其他的 synchronized 操作在条件何时的情况下能够被执行”。

有两种形式的 wait():

- 第一种接受毫秒作为参数:指再次暂停的时间。
 - 在 wait() 期间对象锁是被释放的。
 - 可以通过 notif() 或 notifAll()，或者指令到期，从 wait() 中恢复执行。
- 第二种不接受参数的 wait().
 - 这种 wait() 将无线等待下去，直到线程接收到 notif() 或 notifAll()。

wait()、notif()以及 notifAll() 有一个比较特殊的方面，那就是这些方法是基类 Object 的一部分，而不是属于 Thread 类。仅仅作为线程的功能却成为了通用基类的一部分。原因是这些方法操作的锁，也是所有对象的一部分。所以你可以将 wait() 放进任何同步控制方法里，而不用考虑这个类是继承自 Thread 还是 Runnable。实际上，只能在同步方法或者同步代码块里调用 wait()、notif() 或者 notifAll()。如果在非同步代码块里操作这些方法，程序可以通过编译，但是在运行时会得到 IllegalMonitorStateException 异常。意思是，在调用 wait()、notif() 或者 notifAll() 之前必须拥有获取对象的锁。

比如，如果向对象 x 发送 notifAll()，那就必须在能够得到 x 的锁的同步控制块中这么做：
```java
synchronized(x){
  x.notifAll();
}
```
我们看一个示例：一个是将蜡涂到 Car 上，一个是抛光它。抛光任务在涂蜡任务完成之前，是不能执行其工作的，而涂蜡任务在涂另一层蜡之前必须等待抛光任务完成。
```java
public class Car {
  //涂蜡和抛光的状态
	private boolean waxOn = false;
	//打蜡
	public synchronized void waxed() {
		waxOn = true;
		notifyAll();
	}

	//抛光
	public synchronized void buffed() {
		waxOn = false;
		notifyAll();
	}

	//抛光结束被挂起即将开始打蜡任务
	public synchronized void waitForWaxing() throws InterruptedException{
		while (waxOn == false) {
			wait();
		}
	}

	//打蜡结束被挂起即将开始抛任务
	public synchronized void waitForBuffing() throws InterruptedException{
		while (waxOn == true) {
			wait();
		}
	}


}

```
开始打蜡的任务：
```java
public class WaxOn implements Runnable{
	private Car car;

	protected WaxOn(Car car) {
		super();
		this.car = car;
	}


	@Override
	public void run() {
		try {
			while (!Thread.interrupted()) {
				System.out.println("Wax one");
				TimeUnit.MICROSECONDS.sleep(200);
				//开始打蜡
				car.waxed();
				//当前任务被挂起
				car.waitForBuffing();
			}
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			System.out.println(" Exiting via interrupt");
		}
		System.out.println("Ending wax on task");
	}

}

```
开始抛光的任务：
```java
public class WaxOff implements Runnable{
	private Car car;

	protected WaxOff(Car car) {
		super();
		this.car = car;
	}

	@Override
	public void run() {
		// TODO Auto-generated method stub
		try {
			while (!Thread.interrupted()) {
				//如果还是在打蜡就挂起
				car.waitForWaxing();
				System.out.println("Wax off");
				TimeUnit.MICROSECONDS.sleep(200);
				//开始抛光
				car.buffed();
			}
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			System.out.println("Wxtiing via interrupt");
		}
		System.out.println("Ending wax off task");
	}


}

```
测试类：
```java
public class WaxOmatic {

	public static void main(String[] args) throws Exception{
		// TODO Auto-generated method stub
		Car car = new Car();
		ExecutorService service = Executors.newCachedThreadPool();
		service.execute(new WaxOff(car));
		service.execute(new WaxOn(car));
		//暂停2秒钟
		TimeUnit.SECONDS.sleep(1);
		//关闭所有的任务
		service.shutdownNow();
	}

}

```
执行结果：
```
/.....
Wax one
Wax off
Wax one
Wax off
Wax one
Wax off
 Exiting via interrupt
Wxtiing via interrupt
Ending wax on task
Ending wax off task
```
在 waitForWaxing() 中检查 WaxOn 标志，如果它是 false，那么这个调用任务将会被挂起。这个行为发生在 synchronized 方法中这一点很重要。因为在这个方法中任务已经获得了锁。当你调用 wait() 时，线程被挂起，而锁被释放。释放锁是本质所在，因为为了安全的改变对象的状态，其他某个任务就必须能够获得这个锁。

WaxOn.run() 表示给汽车打蜡的第一个步骤，它执行他的操作：调用 sleep() 模拟打蜡的时间，然后告知汽车打蜡结束，并且调用 waitForWaxing(),这个方法会调用 wait() 挂起当前打蜡的任务。直到 WaxOff 任务调用这两车的 buffed(),从而改变状态并且调用 notfiAll() 重新唤醒为止。翻过来也是一样的，在运行程序时，你可以看到控制权在两个任务之间来回的传递，这两个步骤过程在不断的重复。

**错失的信号**

当两个线程使用 notif()/wait() 或者 notifAll()/wait() 进行协作时，有可能会错过某个信号。假设线程 T1 是通知 T2 的线程，而这两个线程都使用下面的方式实现：
```java
T1:
synchronized(X){
  //设置 T2 的一个条件
  <setup condition for T2>
  x.notif();
}

T2:
while(someCondition){
  //Potit
  synchronized(x){
    x.wait();
  }
}
```

以上的例子假设 T2 对 someCondition 发现其为 true()。在执行 Potit 其中线程调度器可能切换到了 T1。而 T1 将会执行重新设置 condition，并且调用唤醒。当 T2 继续执行时，以至于不能意识到条件已经发生变化，因此会盲目的进入 wait()。此时唤醒在之前已经调用过了，而 T2 将无限的等待下去唤醒的信号。

解决该问题的方案是防止 someCondition 变量上产生竞争条件：
```java
synchronized(x){
  while(someCondition){
    x.wait();
  }
}

```
#### notif() 与 notifAll()
可能有多个任务在单个 Car 对象上被挂起处于 wait() 状态，因此调用 notifyAll() 比调用 notify() 更安全。使用 notify() 而不是 notifyAll() 是一种优化。使用 notify() 时，在众多等待同一个锁的任务中只有一个被唤醒，因此如果你希望使用 notify()，就必须保证被唤醒的是恰当的任务。另外使用 notify() ，所有任务都必须等待相同的条件，因为如果你有多个任务在等待不同的条件，那你就不会知道是否唤醒了恰当的任务。如果使用 notfiy(),当条件发生变化时，必须只有一个任务能从中收益。最后，这些限制对所有可能存在的子类都必须总起作用。如果这些规则任何一条不满足都必须使用 notifyAll()。

在 Java 的线程机制中，有一个描述是这样的：notifyAll() 将唤醒所有正在等待的任务。这是否意味着在程序中任何地方，任何处于 wait() 状态中的任务都将被任何对 notifyAll() 的调用唤醒呢？在下面的实例中说明了情况并非如此，当 notifyAll() 因某个特定锁被调用时，只有等待这个锁的任务才会被唤醒：
```java
public class Blocker {
	 synchronized void waitingCall() {
	    try {
	      while(!Thread.interrupted()) {
	        wait();
	        System.out.print(Thread.currentThread() + " ");
	      }
	    } catch(InterruptedException e) {
	      // OK to exit this way
	    }
	  }
	  synchronized void prod() {
		  notify();
	  }

	  synchronized void prodAll() {
		  notifyAll();
	  }
}

```
创建任务 Task:
```java
public class Task implements Runnable {
	  static Blocker blocker = new Blocker();
	  public void run() { blocker.waitingCall(); }
}

```
创建任务 Task2:
```java
public class Task2 implements Runnable {
	  // A separate Blocker object:
	  static Blocker blocker = new Blocker();
	  public void run() { blocker.waitingCall(); }
}

```
测试类：
```java
public class NotifyVsNotifyAll {
	public static void main(String[] args) throws Exception{
		ExecutorService service = Executors.newCachedThreadPool();
		for (int i = 0; i < 3; i++) {
			service.execute(new Task());
		}
			service.execute(new Task2());
			Timer timer = new Timer();
			timer.scheduleAtFixedRate(new TimerTask() {
				boolean prod = true;
				@Override
				public void run() {
					// TODO Auto-generated method stub
					if (prod) {
						System.out.println("notify");
						Task.blocker.prod();
						prod = false;
					}else {
						System.out.println("notifyAll");
						Task.blocker.prodAll();
						prod = true;
					}
				}
			}, 400, 400);
			TimeUnit.SECONDS.sleep(5);
			timer.cancel();
			System.out.println("Time cancle");
			TimeUnit.MILLISECONDS.sleep(500);
		    System.out.println("Task2.blocker.prodAll() ");
		    Task2.blocker.prodAll();
		    TimeUnit.MILLISECONDS.sleep(500);
		    System.out.println("\nShutting down");
		    service.shutdownNow(); // Interrupt all tasks

	}
}

```
测试结果：
```
notify
Thread[pool-1-thread-2,5,main] notifyAll
Thread[pool-1-thread-2,5,main] Thread[pool-1-thread-3,5,main] Thread[pool-1-thread-1,5,main] notify
Thread[pool-1-thread-2,5,main] notifyAll
Thread[pool-1-thread-2,5,main] Thread[pool-1-thread-1,5,main] Thread[pool-1-thread-3,5,main] notify
Thread[pool-1-thread-2,5,main] notifyAll
Thread[pool-1-thread-2,5,main] Thread[pool-1-thread-3,5,main] Thread[pool-1-thread-1,5,main] notify
Thread[pool-1-thread-2,5,main] notifyAll
Thread[pool-1-thread-2,5,main] Thread[pool-1-thread-1,5,main] Thread[pool-1-thread-3,5,main] notify
Thread[pool-1-thread-2,5,main] notifyAll
Thread[pool-1-thread-2,5,main] Thread[pool-1-thread-3,5,main] Thread[pool-1-thread-1,5,main] notify
Thread[pool-1-thread-2,5,main] notifyAll
Thread[pool-1-thread-2,5,main] Thread[pool-1-thread-1,5,main] Thread[pool-1-thread-3,5,main] Time cancle
Task2.blocker.prodAll()
Thread[pool-1-thread-4,5,main]
Shutting down
```
从上面输出的结果可以看出，我们启动了三个 Task 任务线程，一个 Task2 线程。使用 timer 做了一个定时器，每间隔 4 毫秒就轮换启动 Task.blocker 的 notify() 和 notifyAll()方法。我们看到 Task 和 Task2 都有 Blocker 对象，他们调用 Blocker 对象的时候都会被阻塞。我们看到当调用 Task.prod() 的时候只有一个在等待锁的任务被唤醒，其余两个继续挂起。当调用 Task.prodAll() 的时候等待的三个线程都会被唤醒。当调用 Task2。prodAll() 的时候 只有 Task2 的线程任务被唤醒。其余的三个 Task 任务继续挂起。

#### 生产者与消费者
请考虑这样一种情况，在饭店有一个厨师和一个服务员。这个服务员必须等待厨师做好膳食。当厨师准备好时会通知服务员，之后服务员上菜，然后返回继续等待。这是一个任务协作示例：厨师代表生产者，而服务员代表消费者。两个任务必须在膳食被生产和消费时进行握手，而系统必须是以有序的方式关闭。

膳食类：
```java
public class Meal {
	private final int orderNum;
	public Meal(int orderNum) { this.orderNum = orderNum; }
    public String toString() { return "Meal " + orderNum; }
}

```
服务生类：
```java
public class WaitPerson implements Runnable {
	  private Restaurant restaurant;
	  public WaitPerson(Restaurant r) {
		  restaurant = r;
	  }

	  public void run() {
	    try {
	      while(!Thread.interrupted()) {
	        synchronized(this) {
	          while(restaurant.meal == null)
	            wait(); // ... for the chef to produce a meal
	        }
	        Print.print("Waitperson got " + restaurant.meal);
	        synchronized(restaurant.chef) {
	          restaurant.meal = null;
	          restaurant.chef.notifyAll(); // Ready for another
	        }
	      }
	    } catch(InterruptedException e) {
	    	Print.print("WaitPerson interrupted");
	    }
	  }
}

```
厨师类：
```java
public class Chef implements Runnable {
	  private Restaurant restaurant;
	  private int count = 0;
	  public Chef(Restaurant r) {
		  restaurant = r;
	  }
	  public void run() {
	    try {
	      while(!Thread.interrupted()) {
	        synchronized(this) {
	          while(restaurant.meal != null)
	            wait(); // ... for the meal to be taken
	        }
	        if(++count == 10) {
	        	Print.print("Out of food, closing");
	          restaurant.exec.shutdownNow();
	        }
	        Print.printnb("Order up! ");
	        synchronized(restaurant.waitPerson) {
	          restaurant.meal = new Meal(count);
	          restaurant.waitPerson.notifyAll();
	        }
	        TimeUnit.MILLISECONDS.sleep(100);
	      }
	    } catch(InterruptedException e) {
	    	Print.print("Chef interrupted");
	    }
	  }
}

```
测试类：
```
public class Restaurant {
	  Meal meal;
	  ExecutorService exec = Executors.newCachedThreadPool();
	  WaitPerson waitPerson = new WaitPerson(this);
	  Chef chef = new Chef(this);
	  public Restaurant() {
	    exec.execute(chef);
	    exec.execute(waitPerson);
	  }
	  public static void main(String[] args) {
	    new Restaurant();
	  }
}

```
执行结果：
```
Order up! Waitperson got Meal 1
Order up! Waitperson got Meal 2
Order up! Waitperson got Meal 3
Order up! Waitperson got Meal 4
Order up! Waitperson got Meal 5
Order up! Waitperson got Meal 6
Order up! Waitperson got Meal 7
Order up! Waitperson got Meal 8
Order up! Waitperson got Meal 9
Out of food, closing
Order up! WaitPerson interrupted
Chef interrupted

```

**使用显示的 Lock 和 Condition 对象

在 java SE5 的类库中还有额外的显示工具。我们来重写我们的打蜡和抛光类。使用互斥并允许任务挂起的基本类是 Condition，你可以通过在 Condition 上调用 await() 来挂起一个任务。当外部条件发生变化时，意味着某个任务应该继续执行，你可以通过调用 signal() 来通知这个任务，从而唤醒一个任务，或者调用 signalAll() 来唤醒所有在这个 Condition 上被挂起的任务。(signalAll() 比 notifAll() 是更安全的方式)

下面是重写版本：
```java
class Car {
  private Lock lock = new ReentrantLock();
  private Condition condition = lock.newCondition();
  private boolean waxOn = false;
  public void waxed() {
    lock.lock();
    try {
      waxOn = true; // Ready to buff
      condition.signalAll();
    } finally {
      lock.unlock();
    }
  }
  public void buffed() {
    lock.lock();
    try {
      waxOn = false; // Ready for another coat of wax
      condition.signalAll();
    } finally {
      lock.unlock();
    }
  }
  public void waitForWaxing() throws InterruptedException {
    lock.lock();
    try {
      while(waxOn == false)
        condition.await();
    } finally {
      lock.unlock();
    }
  }
  public void waitForBuffing() throws InterruptedException{
    lock.lock();
    try {
      while(waxOn == true)
        condition.await();
    } finally {
      lock.unlock();
    }
  }
}

class WaxOn implements Runnable {
  private Car car;
  public WaxOn(Car c) { car = c; }
  public void run() {
    try {
      while(!Thread.interrupted()) {
        printnb("Wax On! ");
        TimeUnit.MILLISECONDS.sleep(200);
        car.waxed();
        car.waitForBuffing();
      }
    } catch(InterruptedException e) {
      print("Exiting via interrupt");
    }
    print("Ending Wax On task");
  }
}

class WaxOff implements Runnable {
  private Car car;
  public WaxOff(Car c) { car = c; }
  public void run() {
    try {
      while(!Thread.interrupted()) {
        car.waitForWaxing();
        printnb("Wax Off! ");
        TimeUnit.MILLISECONDS.sleep(200);
        car.buffed();
      }
    } catch(InterruptedException e) {
      print("Exiting via interrupt");
    }
    print("Ending Wax Off task");
  }
}

public class WaxOMatic2 {
  public static void main(String[] args) throws Exception {
    Car car = new Car();
    ExecutorService exec = Executors.newCachedThreadPool();
    exec.execute(new WaxOff(car));
    exec.execute(new WaxOn(car));
    TimeUnit.SECONDS.sleep(5);
    exec.shutdownNow();
  }
}
```
在 Car 的构造器中单个的 Lock 将产生一个 Condition 对象，这个对象被用来管理任务之间的通信。但是这个 Condition 不包含任何有关处理状态的信息，因此你需要额外的表示处理状态的信息，即 Boolean waxOn。

#### 生产者消费者与队列
wait() 和 notifAll() 方法以一种非常低级的方式解决了任务的互操作的问题，即每次交互时都握手。许多时候我们可以使用同步队列来解决协作的问题，同步队列在任何时刻只允许一个任务插入或移除元素。在 Java.util.concurrent.BlockingQueue 接口中提供了这个队列，这个接口有大量的标准实现。可以使用 LinkedBlockingQueue 他是一个无界队列，还可以使用 ArrayBlockingQueue，它具有固定的尺寸，可以在它被阻塞之前向其中放置有限数量的元素。

如果消费者任务试图从队列中获取对象，而该队列为空时，那么这些队列就可以挂起这些任务，并且当有更多的元素可用时恢复这些消费任务。阻塞队列可以解决非常大的问题，而其方式与 wait() 和 notifyAll() 相比，则简单切可靠。

下面是一个简单的测试，它将多个 LiftOff 对象执行串行化。消费者 LiftOffRunner 将每个 LiftOff 对象从 BlockIngQueue 中推出并直接运行。它通过显示的调用 run() 而是用自己的线程来运行，而不是为每个任务启动一个线程。

首先把之前写过的 LiftOff 类贴出来：
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
LiftOffRunner 类：
```java
public class LiftOffRunner implements Runnable{
	private BlockingQueue<LiftOff> rockets;

	protected LiftOffRunner(BlockingQueue<LiftOff> rockets) {
		super();
		this.rockets = rockets;
	}
    public void add(LiftOff lo) {
		try {
			rockets.put(lo);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			System.out.println("添加失败");
		}
	}

	@Override
	public void run() {
		// TODO Auto-generated method stub
		try {
			while (!Thread.interrupted()) {
				LiftOff rocket = rockets.take();
				rocket.run();
			}
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			System.out.println("运行中断");
		}
		System.out.println("退出运行");
	}

}

```
最后是测试类：
```java
public class TestBlockingQueues {
	static void getkey(){
		try {
			new BufferedReader(new InputStreamReader(System.in)).readLine();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}

	static void getkey(String message) {
	    Print.print(message);
	    getkey();
	  }

	static void test(String msg,BlockingQueue<LiftOff> queue){
		LiftOffRunner runner = new LiftOffRunner(queue);
		Thread thread = new Thread(runner);
		thread.start();
		//启动了，但是内容是空的，就一直挂起，等待有新的内容进去
		for (int i = 0; i < 5; i++) {
			runner.add(new LiftOff(5));
		}
		getkey("Press Enter "+ msg);
		thread.interrupt();

	}

	public static void main(String[] args) {
		test("LinkedBlockingQueue", new LinkedBlockingQueue<LiftOff>());
		test("ArrayBlockingQueue", new ArrayBlockingQueue<>(3));
		test("SynchronousQueue", new SynchronousQueue<>());
	}
}

```
**吐司 BlockingQueue**

下面是一个示例，每一台机器都有三个任务：一个只做吐司、一个给吐司抹黄油、另一个在涂抹黄油的吐司上抹果酱。我们来示例如果使用 BlockIngQueue 来运行这个示例：
```java
class Toast {
  public enum Status { DRY, BUTTERED, JAMMED }
  private Status status = Status.DRY;
  private final int id;
  public Toast(int idn) { id = idn; }
  public void butter() { status = Status.BUTTERED; }
  public void jam() { status = Status.JAMMED; }
  public Status getStatus() { return status; }
  public int getId() { return id; }
  public String toString() {
    return "Toast " + id + ": " + status;
  }
}

class ToastQueue extends LinkedBlockingQueue<Toast> {}

class Toaster implements Runnable {
  private ToastQueue toastQueue;
  private int count = 0;
  private Random rand = new Random(47);
  public Toaster(ToastQueue tq) { toastQueue = tq; }
  public void run() {
    try {
      while(!Thread.interrupted()) {
        TimeUnit.MILLISECONDS.sleep(
          100 + rand.nextInt(500));
        // Make toast
        Toast t = new Toast(count++);
        print(t);
        // Insert into queue
        toastQueue.put(t);
      }
    } catch(InterruptedException e) {
      print("Toaster interrupted");
    }
    print("Toaster off");
  }
}

// Apply butter to toast:
class Butterer implements Runnable {
  private ToastQueue dryQueue, butteredQueue;
  public Butterer(ToastQueue dry, ToastQueue buttered) {
    dryQueue = dry;
    butteredQueue = buttered;
  }
  public void run() {
    try {
      while(!Thread.interrupted()) {
        // Blocks until next piece of toast is available:
        Toast t = dryQueue.take();
        t.butter();
        print(t);
        butteredQueue.put(t);
      }
    } catch(InterruptedException e) {
      print("Butterer interrupted");
    }
    print("Butterer off");
  }
}

// Apply jam to buttered toast:
class Jammer implements Runnable {
  private ToastQueue butteredQueue, finishedQueue;
  public Jammer(ToastQueue buttered, ToastQueue finished) {
    butteredQueue = buttered;
    finishedQueue = finished;
  }
  public void run() {
    try {
      while(!Thread.interrupted()) {
        // Blocks until next piece of toast is available:
        Toast t = butteredQueue.take();
        t.jam();
        print(t);
        finishedQueue.put(t);
      }
    } catch(InterruptedException e) {
      print("Jammer interrupted");
    }
    print("Jammer off");
  }
}

// Consume the toast:
class Eater implements Runnable {
  private ToastQueue finishedQueue;
  private int counter = 0;
  public Eater(ToastQueue finished) {
    finishedQueue = finished;
  }
  public void run() {
    try {
      while(!Thread.interrupted()) {
        // Blocks until next piece of toast is available:
        Toast t = finishedQueue.take();
        // Verify that the toast is coming in order,
        // and that all pieces are getting jammed:
        if(t.getId() != counter++ ||
           t.getStatus() != Toast.Status.JAMMED) {
          print(">>>> Error: " + t);
          System.exit(1);
        } else
          print("Chomp! " + t);
      }
    } catch(InterruptedException e) {
      print("Eater interrupted");
    }
    print("Eater off");
  }
}

public class ToastOMatic {
  public static void main(String[] args) throws Exception {
    ToastQueue dryQueue = new ToastQueue(),
               butteredQueue = new ToastQueue(),
               finishedQueue = new ToastQueue();
    ExecutorService exec = Executors.newCachedThreadPool();
    exec.execute(new Toaster(dryQueue));
    exec.execute(new Butterer(dryQueue, butteredQueue));
    exec.execute(new Jammer(butteredQueue, finishedQueue));
    exec.execute(new Eater(finishedQueue));
    TimeUnit.SECONDS.sleep(5);
    exec.shutdownNow();
  }
}
```
这个示例中没有任何显示的同步，因为同步队列和系统的设计隐式的管理了每片 Toast 在任何时刻都只有一个任务在操作。因为队列的阻塞，使得处理过程将被自动挂起和恢复。
