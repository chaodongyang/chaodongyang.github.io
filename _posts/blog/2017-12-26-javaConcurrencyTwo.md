---
layout: post
title: java编程思想之并发(共享资源)
categories: javaThinking
description: java编程思想之并发
keywords: java, java并发, java多线程，Java多线程并发
---

有了并发我们可以同时做很多事情，但是，两个或者多个线程互相干扰的问题也存在。如果不防范这种冲突，就可能出现两个线程同时访问一个银行账户，向同一个打印机打印，改变同一个值等问题。

## 共享资源
单个线程每次只能做一件事情。因为只有一个实体所以永远不用担心两个人在同一个地方停车的问题。但是多线程会在同时访问一个资源。

### 不正确的访问资源
我们先做一个实验，多个任务。一个任务产生一个偶数，其他的任务检验偶数的有效性。
```java
public abstract class IntGenerator {
	//为了表示可见性，使用 volatile 修饰
	private volatile boolean canceled = false;
	public abstract int next();
	public void cancel(){
		canceled = true;
	}
	//查看该对象是否已经被撤销
	public boolean isCanceled() {
		return canceled;
	}
}

```
任何 IntGenerator 都可以用下面的 EvenChecker 类来测试：
```java
public class EvenChecker implements Runnable{
	private IntGenerator generator;
	private final int id;

	protected EvenChecker(IntGenerator generator, int id) {
		super();
		this.generator = generator;
		this.id = id;
	}

	@Override
	public void run() {
		while (!generator.isCanceled()) {
			int val = generator.next();
			if (val %2 !=0) {
				System.out.println("不是偶数");
				generator.cancel();
			}

		}

	}

	public static void test(IntGenerator gp,int count) {
		ExecutorService service = Executors.newCachedThreadPool();
		for (int i = 0; i < count; i++) {
			service.execute(new EvenChecker(gp, i));
		}
		service.shutdown();
	}

	public static void test(IntGenerator gp) {
		test(gp,10);
	}

}
```
上面的示例中 ```generator.cancel()``` 撤销的不是任务本身，而是 IntGenerator 对象是否可以被撤销的条件。必须仔细考虑并发系统失败的所有可能途径，例如，一个任务不能依赖于另一个任务。因为任务关闭的顺序无法得到保证。这里，通过使任务依赖于非任务对象，我们可以消除潜在的竞争条件。

EvenChecker 总是读取和测试 IntGenerator 的返回值。如果 isCanceled() 返回值为 true，则 run() 返回，这将告知 test() 中的 Executor 该任务完成了。任何 EvenChecker 任务都可以在与其关联的 IntGenerator 上调用 cancel()，这将导致所有其他使用该 IntGenerator 的 EvenChecker 得到关闭。

第一个 IntGenerator 有一个可以产生一些列偶数值的 next() 方法：
```java
public class EvenGenerator extends IntGenerator{
	private int currentEvenValue = 0;

	@Override
	public int next() {
		++currentEvenValue;
		++currentEvenValue;

		return currentEvenValue;
	}

	public static void main(String[] args) {
		EvenChecker.test(new EvenGenerator());
	}
}

```
执行结果：
```
1537不是偶数
1541不是偶数
1539不是偶数

```
一个任务可能在另一个任务执行第一个递增操作之后，但是没有执行第二个递增操作之前，调用 next() 方法。这将使这个值处于不恰当状态。为了证明这是可能发生的，text() 方法创建了一组 EvenChecker 对象，以用来连续的读取并输出同一个 EvenGenerator,并检测每个数值是否都是偶数。如果不是就报错终止。

这个程序最终会失败终止，因为每个 EvenChecker 任务在 EvenGenerator 处于不恰当的状态时，仍能够访问其中的信息。但是根据不同的操作系统和实现细节这个问题在循环多次之后也可能不会被探测到。有一点很重要，那就是递增程序自身也需要多个步骤，并且在递增过程中任务可能被挂起。也就是说递增在 Java 中不是原子性操作。因此，如果不保护任务，即使单一的递增也不是安全的。

#### 解决共享资源竞争
前面的示例展示使用线程的一个基本问题：你永远不知道一个线程何时在运行。对于并发操作，你需要某种方式来防止两个任务访问相同的资源，至少在关键阶段不能出现这种情况。防止这种冲突的方法是当资源被一个任务使用时，在其上加锁。第一个访问某项资源的任务必须锁定这个资源，使其他任务在其被解锁前无法访问他，而在其解锁之时，另一个任务就可以锁定并使用它，以此类推。

基本上所有的并发模式在解决线程冲突问题的时候，都是采用序列化访问共享资源的方案。这意味着在给定时刻只允许一个任务访问共享资源。通常这种是通过在代码前面加上一句锁语句来实现的，这就使得在一段时间内只有一个任务可以运行这段代码。因为锁语句产生一种相互排斥的效果，这种机制称为互斥量。

另外当一个锁被解锁的时候，我们并不能确定下一个使用锁的任务，因为线程调度机制并不是确定性的。可以通过 yield() 和 setPriorit() 来给线程调度器提供建议。

Java 以提供关键字 synchronized 的形式，为防止资源冲突提供了内在支持。当任务要执行被 synchronized 关键字保护的代码片段的时候，它将检查锁是否可用，然后获取锁，执行代码，释放锁。共享资源一般是以对象像是存在于内存片段，可以是文件、输入输出端口。要控制对共享资源的访问，得先把它包装进一个对象。然后把所有要访问这个资源的方法标记为 synchronized。

下面是声明 synchronized 方法的方式：
```java
synchronized void f(){};
synchronized void g(){};
```
所有对象都自动含有单一的锁(监视器)。当在对象上调用其任意 synchronized 方法的时候，此对象被加锁，这时这个对象上的其他 synchronized 方法只有等到前一个方法调用完毕并释放了锁之后才能被调用。对于某个特定对象来说，其所有 synchronized 方法共享同一个锁，这可以被用来防止多个任务同时访问被编码为对象内存。

注意：使用并发时将对象设置为 private 是非常重要的，否则，synchronized 关键字就不能防止其他的任务直接访问域，这样就会产生冲突。

针对每个类也有一个锁，所以 synchronized static 方法可以在类的范围内防止对 static 数据的并发访问。

该什么时候同步呢？
```
如果你正在写一个变量，它可能接下来被另一个线程读取，或者正在读取一个上一次被另一个线程写过的变量，那么你必须使用同步，并且，读写线程都必须用相同的监视器锁同步。
```

**同步控制 EvenGenerator**

通过在 EvenGenerator 中加入 synchronized 关键字，可以防止不希望的线程访问：
```java
public class EvenGenerator extends IntGenerator{
	private int currentEvenValue = 0;

	@Override
	public synchronized int next() {
		++currentEvenValue;
		Thread.yield();
		++currentEvenValue;

		return currentEvenValue;
	}

	public static void main(String[] args) {
		EvenChecker.test(new EvenGenerator());
	}
}
```
对 Thread.yield() 的调用被插入到两个线程之间，以提高奇数的可能性。因为互斥可以防止多个任务同时进入临界区，所以上面不会产生任何的失败。第一个进入 next() 的任务获得锁，任何其他试图获取锁的任务都将被阻塞，直到第一个任务释放锁。

**使用显示的 Lock 对象**

Java SE5 的类库中还包含定义在 java.util.concurrent.locks 中的显示的互斥机制。Lock 对象必须被显示地创建、锁定和释放。因此，它与内建的锁形式相比，代码缺乏有雅性。但是对于解决某些类型的问题时更加的灵活。

下面用显示的 Lock 重写上面的代码：
```java
public class EvenGenerator extends IntGenerator{
	private int currentEvenValue = 0;
	//创建锁
	private Lock lock = new ReentrantLock();
	@Override
	public int next() {
		//锁定
		lock.lock();
		try {
			++currentEvenValue;
			Thread.yield();
			++currentEvenValue;

			return currentEvenValue;
		}finally {
			//计算完毕后释放锁
			lock.unlock();
		}
	}

	public static void main(String[] args) {
		EvenChecker.test(new EvenGenerator());
	}
}

```
当你在使用 lock 对象时，示例的惯用法很重要：对 unlock() 方法的调用必须放在 try-finlly 语句中。注意，return 语句必须在 try 子句中出现，以确保 unlock() 不会过早的发生，从而将数据暴露在第二个任务。尽管 try-finlly 子句比 synchronized 关键字要多，但显示的 lock 的优点也是显而易见的。如果在使用 synchronized 关键字时某些事物失败了，那么就会抛出一个异常。但是我们并没有机会去处理，以维护系统良好的状态。显示的 lock 对象，你就可以使用 finlly 子句维护系统的正确状态。大体上我们使用 synchronized 的情况更多，只有遇到解决特殊问题时才是用显示的 lock 对象。

示例：使用 synchronized 关键字不能尝试着获取锁且获取锁会失败，或者尝试着获取一段时间然后放弃它。
```java
public class AttemptLocking {
	private ReentrantLock lock = new ReentrantLock();
	  public void untimed() {
		  //尝试获取锁
	    boolean captured = lock.tryLock();
	    try {
	      System.out.println("tryLock(): " + captured);
	    } finally {
	      if(captured)
	        lock.unlock();
	    }
	  }
	  public void timed() {
	    boolean captured = false;
	    try {
	    	//尝试2秒后失败
	      captured = lock.tryLock(2, TimeUnit.SECONDS);
	    } catch(InterruptedException e) {
	      throw new RuntimeException(e);
	    }
	    try {
	      System.out.println("tryLock(2, TimeUnit.SECONDS): " +
	        captured);
	    } finally {
	      if(captured)
	        lock.unlock();
	    }
	  }
	  public static void main(String[] args) {
	    final AttemptLocking al = new AttemptLocking();
	    al.untimed(); // True -- lock is available
	    al.timed();   // True -- lock is available
	    // Now create a separate task to grab the lock:
	    new Thread() {
	      { setDaemon(true); }
	      public void run() {
	        al.lock.lock();
	        System.out.println("acquired");
	      }
	    }.start();
	    Thread.yield(); // Give the 2nd task a chance
	    al.untimed(); // False -- lock grabbed by task
	    al.timed();   // False -- lock grabbed by task
	  }
}

```
执行结果：
```
tryLock(): true
tryLock(2, TimeUnit.SECONDS): true
tryLock(): true
tryLock(2, TimeUnit.SECONDS): true
acquired
```
ReentrantLock 允许我们尝试着获取锁但是最终未获取锁，这样如果其他人已经获取了锁，那么你就可以决定离开做一些其他的事情，而不是一直等待这个锁被释放。显示的 Lock 对象在加锁和释放锁方面，相对于内建的 synchronized 锁来说，还赋予了你更细粒度的控制力。

#### 原子性和易变性
在 Java 线程中，常常我们会认为原子操作不需要进行同步控制。原子操作是不能被线程调度机制中断的。这样的想法是错误的，依赖于原子性是危险的。原子性在 Java 的类库中已经实现了一些更加巧妙的构建。原子性可以应用于除了 long 和 double 之外的所有基本类型之上的 “简单操作”。但是 jvm 会把 64 位的 long 和 double 操作当做两个分离的 32 位的操作来执行，这就产生了一个读取和写入操作之间产生上下文切换，从而导致了不同的任务产生不正确结果的可能性。但是如果我们使用 volatile 关键字就会获得原子性 (在 Java SE5 之前一直未能正确工作)。

因此，原子操作可由线程机制来保证其不可中断，但是即便这样，这也是一种简化的机制。有时看起来很安全的原子性操作实际上也可能不安全。

在多核处理器上，可视性问题远比原子性问题多得多。一个任务做出的修改可能对其他任务是不可见的。因为每个任务都会暂时把信息存储在缓存中。同步机制强制在处理器中一个任务做出的修改必须是可见的。volatile 关键字确保了这种可视性。一个任务修改了对这个修饰对象的操作，那么其他的任务读写操作都能看到这个修改。即使是用了缓存也能被看到，因为 volatile 会被立即写入主存。而读写操作就发生在主存中。同步也会导致向主存中刷新，所以如果一个对象是 synchronized 保护的那么久不必使用 volatile 修饰。使用 volatile 而不是 synchronized 的唯一安全的情况是类中只有一个可变的域。我们的第一选择应该是 synchronized 关键字，这是最安全的方式。

#### 什么是原子性操作？
对域中的值做赋值和返回操作通常都是原子性的。但是递增和递减并不是：
```java
public class Atomicity {
	int i;
	void f(){
		i++;
	}
	void g(){
		i +=3;
	}
}
```
我们看编译后的文件：
```java
void f();
		0  aload_0 [this]
		1  dup
		2  getfield concurrency.Atomicity.i : int [17]
		5  iconst_1
		6  iadd
		7  putfield concurrency.Atomicity.i : int [17]
 // Method descriptor #8 ()V
 // Stack: 3, Locals: 1
 void g();
		0  aload_0 [this]
		1  dup
		2  getfield concurrency.Atomicity.i : int [17]
		5  iconst_3
		6  iadd
		7  putfield concurrency.Atomicity.i : int [17]
}
```
每个指令都产生了一个 get 和 put ，他么之间还有一些其他的指令。因此在获取和修改之间，另一个任务可能会修改这个域。所以，这些操作不是原子性的：

我们再看下面这个例子是否符合上面的描述：
```java
public class AtomicityTest implements Runnable {
	  private int i = 0;
	  public int getValue() {
		  return i;
	  }

	  private synchronized void evenIncrement() {
		  i++;
		  i++;
	  }

	  public void run() {
	    while(true)
	      evenIncrement();
	  }

	  public static void main(String[] args) {
	    ExecutorService exec = Executors.newCachedThreadPool();
	    AtomicityTest at = new AtomicityTest();
	    exec.execute(at);
	    while(true) {
	      int val = at.getValue();
	      if(val % 2 != 0) {
	        System.out.println(val);
	        System.exit(0);
	      }
	    }
	  }
}

```
测试结果：
```
1

```
改程序找到奇数并终止。尽管 return i 是原子性操作，但是缺少同步使得其数值可以在不稳定的中间状态时被读取。还有由于 i 不是 volatile 的也存在可视性的问题。getValue() 和 evenIncrement() 必须都是 synchronized 的。对于基本类型的读取和赋值操作被认为是安全的原子性操作。但是当对象处于不稳定状态时，仍旧很有可能使用原子性操作得到访问。最明智的做法是遵循同步的规则。

#### 原子类
Java SE5 中引入了诸如 AtomicInteger、AtomicLong、AtomicReference 等等特殊的原子性变量类，他们提供下面形式的原子性条件更新操作：
```java
boolean compareAndSet(expectedValue,updateValue);
```
这些类被调整为可以使用在现代处理器上，并且是机器级别的原子性，因此在使用他们时不需要担心。常规来说很少使用他们，但是对于性能调优来说，他们就大有用武之地了。

示例，重写上面的实例：
```java
public class AtomicIntegerTest implements Runnable{
	private AtomicInteger ger = new AtomicInteger(0);
	public int getValue() {
		return ger.get();
	}
	private void eventIncrement() {
		ger.addAndGet(2);
	}

	@Override
	public void run() {
		while (true) {
			eventIncrement();
		}

	}

	public static void main(String[] args) {
		ExecutorService exec = Executors.newCachedThreadPool();
		AtomicIntegerTest aIntegerTest = new AtomicIntegerTest();
		exec.execute(aIntegerTest);
		while (true) {
			int val = aIntegerTest.getValue();
			if (val % 2 !=0) {
				System.out.println(val);
				System.exit(0);
			}
		}
	}

}

```
Atomic 类被设计为构建 Java.util.concurrent 中的类，因此只有在特殊情况下才在代码中使用他们。上面的例子没有使用任何加锁机制也能得到很好的同步。但是通常依赖于锁对我们来说更安全一点。

#### 临界区
有时我们需要防止多个线程同时访问方法内部的部分代码而不是防止访问整个方法。通过这种方式分离出来的代码被称为临界区，也是使用 synchronized 关键字修饰。语法是：synchronized 被用来指定某个对象，此对象的锁被用来对括号内的代码进行同步控制：
```java
synchronized (syncObject){
	//被同步控制的代码块
}
```
这被称之为同步代码块；在进入此段代码之前，必须得到 syncObject 对象的锁。如果其他线程已经得到锁，那么就得等到锁被释放之后，才能进入临界区。通过使用同步控制块，而不是整个方法进行同步控制，可以使多个任务访问对象的时间性得到显著提高。

下面的例子比较了两种同步控制方法：
```java
public class Pair {
	  private int x, y;
	  public Pair(int x, int y) {
	    this.x = x;
	    this.y = y;
	  }
	  public Pair() { this(0, 0); }
	  public int getX() { return x; }
	  public int getY() { return y; }
	  //递增操作是非线程安全的

	  public void incrementX() {
		  x++;
	  }
	  public void incrementY() {
		  y++;
	  }

	  public String toString() {
	    return "x: " + x + ", y: " + y;
	  }

	  public class PairValuesNotEqualException extends RuntimeException {
	    public PairValuesNotEqualException() {
	      super("Pair values not equal: " + Pair.this);
	    }
	  }
	  // Arbitrary invariant -- both variables must be equal:
	  public void checkState() {
	    if(x != y)
	      throw new PairValuesNotEqualException();
	  }
}

```
模板类：
```java
public abstract class PairManager {
      //线程安全的
	  AtomicInteger checkCounter = new AtomicInteger(0);
	  protected Pair p = new Pair();
	  //集合也是线程安全的
	  private List<Pair> storage =Collections.synchronizedList(new ArrayList<Pair>());

	  //方法是线程安全的
	  public synchronized Pair getPair() {
	    // Make a copy to keep the original safe:
	    return new Pair(p.getX(), p.getY());
	  }

	  // 每次添加一次间隔 50毫秒
	  protected void store(Pair p) {
	    storage.add(p);
	    try {
	      TimeUnit.MILLISECONDS.sleep(50);
	    } catch(InterruptedException ignore) {

	    }
	  }

	  public abstract void increment();
}

```
实现模板：
```java
public class PairManager1 extends PairManager{

	//在方法体上修饰表明方法是同步控制的
	@Override
	public synchronized void increment() {
		// 递增和递减是非线程安全的
		 p.incrementX();
		 p.incrementY();
		 store(getPair());
	}

}

public class PairManager2 extends PairManager{

	@Override
	public void increment() {
		Pair temp;
		//同步代码块，计算完毕之后赋值
	    synchronized(this) {
	      p.incrementX();
	      p.incrementY();
	      temp = getPair();
	    }
	    store(temp);

	}

}

```
创建两个线程：
```java
public class PairManipulator implements Runnable {

	  private PairManager pm;
	  public PairManipulator(PairManager pm) {
	    this.pm = pm;
	  }

	  public void run() {
	    while(true)
	      pm.increment();
	  }

	  public String toString() {
	    return "Pair: " + pm.getPair() +
	      " checkCounter = " + pm.checkCounter.get();
	  }

}

public class PairChecker implements Runnable{

	  private PairManager pm;
	  public PairChecker(PairManager pm) {
	    this.pm = pm;
	  }
	  public void run() {
	    while(true) {
	      pm.checkCounter.incrementAndGet();
	      pm.getPair().checkState();
	    }
	  }

}
```

测试类：
```java
public class CriticalSection {
	 static void testApproaches(PairManager pman1, PairManager pman2) {
	    ExecutorService exec = Executors.newCachedThreadPool();

	    PairManipulator
	      pm1 = new PairManipulator(pman1),
	      pm2 = new PairManipulator(pman2);
	    PairChecker
	      pcheck1 = new PairChecker(pman1),
	      pcheck2 = new PairChecker(pman2);

	    exec.execute(pm1);
	    exec.execute(pm2);
	    exec.execute(pcheck1);
	    exec.execute(pcheck2);
	    try {
	      TimeUnit.MILLISECONDS.sleep(500);
	    } catch(InterruptedException e) {
	      System.out.println("Sleep interrupted");
	    }
	    System.out.println("pm1: " + pm1 + "\npm2: " + pm2);
	    System.exit(0);
	  }
	  public static void main(String[] args) {
	    PairManager
	      pman1 = new PairManager1(),
	      pman2 = new PairManager2();
	    testApproaches(pman1, pman2);
	  }
}

```
最后的测试结果：
```
pm1: Pair: x: 11, y: 11 checkCounter = 2183
pm2: Pair: x: 12, y: 12 checkCounter = 24600386
```
尽管每次运行的结果可能会不同，但一般情况下 PairChecker 的检查频率 PairManager1 比 PairManager2 少。后者采用同步代码块进行控制，所以对象不加锁的时间更长。使得其他线程能够更多的访问。

#### 在其他对象上同步
synchronized 块必须给定一个在其上同步的对象，并且合理的方式是，使用其方法正在被调用的当前对象：synchronized(this)，在这种方式中如果获得了 synchronized 块上的锁，那么该对象其他的 synchronized 方法和临界区就不能被调用了。

有时必须在另外一个对象上同步，但是如果你这样做，就必须确保所有相关的任务都是在同一个对象上同步的。

下面的例子演示了两个任务可以同时进入同一个对象，只要这个对象上的方法是在不同的锁上同步的即可：
```java
class DualSynch {
  private Object syncObject = new Object();
  public synchronized void f() {
    for(int i = 0; i < 5; i++) {
      print("f()");
      Thread.yield();
    }
  }
  public void g() {
    synchronized(syncObject) {
      for(int i = 0; i < 5; i++) {
        print("g()");
        Thread.yield();
      }
    }
  }
}

public class SyncObject {
  public static void main(String[] args) {
    final DualSynch ds = new DualSynch();
    new Thread() {
      public void run() {
        ds.f();
      }
    }.start();
    ds.g();
  }
}
```
执行结果：
```
g()
f()
g()
f()...
```
其中 f() 是在 this 上同步的，而 g() 是在一个 syncObject 上同步的 synchronized 块。因此，这两个同步是相互独立的。通过在 main() 中的方法调用可以看到，这两个方法并没有阻塞。

#### 线程本地存储
防止任务在共享资源上产生冲突的第二种方式是根除对变量内存的共享。线程本地存储是一种自动化机制，可以使用相同变量的每个不同的线程创建不同的存储。因此，如果你有5个线程，那么线程会在本地生成 5 个不同的存储块。它们使得你可以将状态和线程关联起来。

创建和管理线程本地存储可以由 java.lang.ThreadLocal 类来实现：
```java
public class Accessor implements Runnable{

	private final int id;
	protected Accessor(int id) {
		super();
		this.id = id;
	}
	@Override
	public void run() {
		while (!Thread.currentThread().isInterrupted()) {
			ThreadLocalVariableHolder.increment();
			System.out.println(this);
			Thread.yield();
		}

	}
	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return "#"+id+":"+ThreadLocalVariableHolder.get();
	}

}
```
线程本地存储：
```java
public class ThreadLocalVariableHolder {
	private static ThreadLocal<Integer> value = new ThreadLocal<Integer>(){
		private Random dRandom = new Random(47);
		protected synchronized Integer initialValue(){
			return dRandom.nextInt(10000);
		}
	};

	public static void increment() {
		value.set(value.get()+1);
	}

	public static int get() {
		return value.get();
	}

	public static void main(String[] args) throws Exception{
		ExecutorService executorService = Executors.newCachedThreadPool();
		for (int i = 0; i < 5; i++) {
			executorService.execute(new Accessor(i));
		}
		TimeUnit.SECONDS.sleep(3);
		executorService.shutdown();
	}
}

```
测试结果：
```
#0:712564
#0:712565
#0:712566
#0:712567
#0:712568/...
```
ThreadLocal 对象通常当做静态存储域。创建 ThreadLocal 方法时只能通过 get() 和 set() 方法来访问内容，其中，get() 方法返回与对象相关联的副本，而 set() 将会将参数插入到为其线程存储的对象中，并返回存储中原有对象。运行这个程序的时候会发现每个单独的线程都分配了自己的存储，因为他们每个都要跟踪自己的计数值。
