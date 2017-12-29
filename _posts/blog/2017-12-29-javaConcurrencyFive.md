---
layout: post
title: java编程思想之并发(死锁)
categories: javaThinking
description: java编程思想之并发
keywords: java, java并发, java多线程，Java多线程并发
---

一个对象可以有 synchronized 方法或其他形式的加锁机制来防止别的任务在互斥还没有释放的时候就访问这个对象。

## 死锁
任务有可能变成阻塞状态，所以就可能发生这样的情况：某个任务在等待另一个任务，而后者又在等待别的任务，这样一直下去，直到这个链条上的任务又在等待第一个任务释放锁。这就形成了一个相互等待的循环，没有那个线程能够继续。这被称之为死锁。

我们真正需要解决的问题是程序看起来可能工作良好，但是具有潜在的死锁风险。这时，死锁可能发生，而事先却没有任何征兆，所以缺陷会潜伏在你的程序里，直到被人以外的发现了。因此，在编写并发编程的时候，进行仔细的程序设计以防止死锁是非常关键的。

下面引入一个问题，一共有 5 个哲学家。这些哲学家将花部分时间思考，花部分时间就餐。作为哲学家他们很穷，所以他们只能买 5 根筷子。他们围坐在桌子的周围，每人之间放一根筷子。当一个哲学家要就餐的时候，这个哲学家必须同时得到左边和右边的筷子。如果一个哲学家左边或者右边已经得到筷子，那么这个哲学家就必须等待，直至可得到必须的筷子。
```java
public class Chopstick {
	private boolean taken = false;

	public synchronized void take() throws InterruptedException{
		while (taken) {
			wait();
		}
		taken = true;
	}

	public synchronized void drop() {
		taken = false;
		notifyAll();
	}
}

```
任何两个哲学家都不能使用同一根筷子。也就是不能同时 taken() 同一个筷子。另外如果一个 Chopstick 被一个哲学家获得，那么另一个哲学家可以 wait()，直到当前的这根筷子的持有者调用 drop() 结束使用。
```java
public class Philosopher implements Runnable{

	private Chopstick left;
	private Chopstick right;
	private final int id;
	private final int ponderFactor;
	private Random random = new Random(47);
	public void pause() throws InterruptedException{
		if (ponderFactor ==0) {
			return;
		}

		TimeUnit.MICROSECONDS.sleep(random.nextInt(ponderFactor * 250));
	}
	protected Philosopher(Chopstick left, Chopstick right, int id, int ponderFactor) {
		super();
		this.left = left;
		this.right = right;
		this.id = id;
		this.ponderFactor = ponderFactor;
	}

	@Override
	public void run() {
		// TODO Auto-generated method stub
		try {
			while (!Thread.interrupted()) {
				System.out.println(this+"开始思考");
				pause();
				System.out.println(this+"开始拿左边的筷子");
				left.take();
				System.out.println(this+"开始拿右边的筷子");
				right.take();

				System.out.println(this+"开始就餐");
				pause();
				left.drop();
				right.drop();
			}
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			System.out.println("当前线程被中断了");
		}
	}

	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return "哲学家的编号："+id;
	}
}

````
在哲学家的任务中，每个哲学家都是不断的思考和吃饭。如果 ponderFactor 不为 0，则 pause() 就会休眠一会。通过这样的方法你会看到哲学家会思考一段时间。然后尝试着去获取左边和右边的筷子，随后再在吃饭上花掉一段随机的时间，之后重复此过程。

现在我们来建立这个程序的死锁版本：
```java
public class DeadLockingDiningPhilosophers {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		int ponder = 5;
		int size = 5;
		ExecutorService service = Executors.newCachedThreadPool();
		Chopstick[] sChopsticks = new Chopstick[size];
		for (int i = 0; i < size; i++) {
			sChopsticks[i] = new Chopstick();
		}
		for (int i = 0; i < size; i++) {
			//每一个哲学家都会持有他左边和右边的筷子对象
			service.execute(new Philosopher(sChopsticks[i],sChopsticks[(i+1)%size],i,ponder));
		}
		System.out.println("执行结束");
		service.shutdownNow();
	}

}
```
执行的结果：
```
执行结束
哲学家的编号：2开始思考
哲学家的编号：4开始思考
哲学家的编号：1开始思考
哲学家的编号：0开始思考
哲学家的编号：3开始思考
当前线程被中断了
当前线程被中断了
当前线程被中断了
当前线程被中断了
当前线程被中断了
```
这个程序表示每一个哲学家都有可能要表示进餐，从而等待其临近的 Philosopher 放下他们的 Chopstick。这将会使得程序死锁。

要修正死锁必须明白，当以下四个条件同时满足时，就会发生死锁：

1. 互斥条件。任务使用的资源中至少有一个是不能共享的。
2. 至少有一个任务必须持有一个资源，并且正在等待获取一个当前被别的任务持有的资源。也就是说必须是拿着一根筷子等待另一个筷子。
3. 资源不能被任务抢占，任务必须把资源释放当做普通事件。你不能抢别人手里的筷子。
4. 必须有循环等待，这时一个任务等待其他任务所持有的资源，后者又在等待另一个任务所持有的资源，这样循环下去直到有一个任务等待第一个任务所持有的资源，使得大家都被锁住。

因为要发生死锁所有这些条件必须满足；所以要防止死锁的话只需要破坏其中一个就可以。在程序中防止死锁的最容易的办法就是破坏第四个循环条件。
```java
public class FixedDiningPhilosophers {
	 public static void main(String[] args) throws Exception {
		    int ponder = 5;
		    if(args.length > 0)
		      ponder = Integer.parseInt(args[0]);
		    int size = 5;
		    if(args.length > 1)
		      size = Integer.parseInt(args[1]);
		    ExecutorService exec = Executors.newCachedThreadPool();
		    Chopstick[] sticks = new Chopstick[size];
		    for(int i = 0; i < size; i++)
		      sticks[i] = new Chopstick();
		    for(int i = 0; i < size; i++)
		      if(i < (size-1))
		        exec.execute(new Philosopher(sticks[i], sticks[i+1], i, ponder));
		      else
		        exec.execute(new Philosopher(sticks[0], sticks[i], i, ponder));
		    if(args.length == 3 && args[2].equals("timeout"))
		      TimeUnit.SECONDS.sleep(5);
		    else {
		      System.out.println("Press 'Enter' to quit");
		      System.in.read();
		    }
		    exec.shutdownNow();
		  }
}

```
执行结果：
```
哲学家的编号：2开始拿左边的筷子
哲学家的编号：4开始思考
哲学家的编号：1开始思考
哲学家的编号：0开始拿左边的筷子
哲学家的编号：0开始拿右边的筷子
哲学家的编号：0开始就餐
哲学家的编号：3开始就餐
哲学家的编号：2开始拿右边的筷子
/.....
```
通过确保最后一个哲学家先拿起和放下左边的筷子，我们可以移除死锁，从而使得程序运行。Java 并没有对死锁提供类库上的支持；能否通过仔细的程序设计避免死锁需要我们自己努力。
