---
layout: post
title: java编程思想之并发(性能优化)
categories: javaThinking
description: java编程思想之并发
keywords: java, java并发, java多线程，Java多线程并发
---

## 性能调优
在 Java SE5 类库中的 java.util.concurrent 类库中存在着数量庞大的用于性能提高的类。当细细读取这个类库时很难辨认哪些类适用于那些常规应用，而那些类适用于提高性能。

#### 比较各类互斥技术
Java 包括老式的 synchronized 关键字和 Java SE5 中新的 Lock 和 Atomic 类，那么比较这些不同的方式，更多的理解他们的各自价值和使用范围，就会显得很有意义。

我们针对每种方式都做一个简单的测试：
```java
public abstract class Incrementable {
	protected long counter =0;
	public abstract void increment();
}

```
synchronized 方式：
```java
public class SynchronizingTest extends Incrementable{

	@Override
	public synchronized void increment() {
		// TODO Auto-generated method stub
		++counter;
	}

}
```
Lock 方式：
```java
public class LockingTest extends Incrementable{

	private Lock lock = new ReentrantLock();
	@Override
	public void increment() {
		// TODO Auto-generated method stub
		lock.lock();
		try {
			++counter;
		} finally {
			lock.unlock();
		}
	}

}

```
测试两个类：
```java
public class SimpleMicroBenchmark {

	static long test(Incrementable incrementable){
		long start = System.nanoTime();
		for (int i = 0; i < 10000000l; i++) {
			incrementable.increment();
		}
		return System.nanoTime()-start;
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
	    long synchTime = test(new SynchronizingTest());
	    long lockTime = test(new LockingTest());

	    System.out.printf("Synchronizing: %1$10d\n",synchTime);
	    System.out.printf("lockTime: %1$10d\n",lockTime);

	}

}

```
测试结果：
```
Synchronizing:  231087528
lockTime:  199864257
```
使用 Lock 要比使用 synchronized 要高效的多，而且 synchronized 的开销看起来变化范围太大，而 Lock 相对比较一致。(多运行几次上面的代码)

这里是否意味着你永远都不应该使用 synchronized 关键字呢？这里有两个因素要考虑：首先要看互斥方法体的大小，在实际开发中互斥部分可能会非常大，因此在方法体中所花费的时间的百分比可能会明显大于进入和退出互斥的开销，这样就是淹没了提高互斥速度所带来的好处。当然对于这一点我们需要在性能调优时尝试各种不同的方法观察他们的影响。其次，很明显 synchronized 关键字所产的代码要比 Lock 要少的多，可读性也提高了许多。代码被阅读的次数远高于被编写的次数。在编程时与其他人的交流相对于与计算机的交流要重要的多，因此代码的可读性至关重要。

#### 免锁容器
容器是所有编程的基础工具，这其中也包括并发编程。早期的容器类 Vector 和 Hashtable 具有许多的 synchronized 方法，当他们用于非多线程程序时，便会导致不可接受的开销。在 java 1.2 中，新的容器类库是不同步的，并且 Collections 类提供了各种 static 的同步修饰方法，从而来同步不同类型的容器。尽管这是一种改进，因为它可以使你选择在你的容器中是否要使用同步，但是这种开销仍旧是基于 synchronized 加锁的机制。Java SE5 特别添加了新的容器，通过使用更灵巧的技巧来消除加锁，从而提高线程安全的性能。

这些免锁容器背后的通用策略是：对容器的修改可以和读取操作同时发生，只要读取者只能看到完成修改后的结果即可。修改是在容器数据结构的某个部分的一个单独的副本上执行的，并且这个副本在修改过程中是不可见的。只有当修改完成时，被修改的结构才会自动地与主数据结构交换，之后读取者就可以看到修改之后的结果了。

在 CopyOnWriteArrayList 中，写入将导致创建整个底层数组的副本，而源数组将保留在原地，使得复制的数组再被修改时，读取操作可以安全的执行。当修改完成时，一个原子性的操作将把新的数组换入，使得新的读取操作可以看到这个新的修改。CopyOnWriteArrayList 的好处之一是当多个迭代器同时遍历和修改这个列表时，不会抛出 ConcurrentModificationException。CopyOnWriteArraySet 将使用 CopyOnWriteArrayList 来实现其免锁行为。ConcurrentHashMap 和 ConcurrentLinkedQueue 使用了类似的技术，允许并发的读取和写入，但是容器中只有部分内容而不是整个容器可以被复制和修改。然而，任何修改在完成之前，读取者仍旧不能看到他们。

**乐观锁**

只要你主要是从免锁容器中读取，那么就会比其对应的 synchronized 快许多，因为获取和释放锁的开销被省掉了。如果要向免锁容器执行少量的写入，那么情况也是如此。下面我们将用一个测试的例子来展示在各种不同的情况下，这些容器在性能方面差异的大致概念。

从一个泛型的框架入手，专门用于在任何类型的容器上执行测试，包括各种 Map 在内，其中泛型参数 c 表示容器的类型：
```Java
public abstract class Tester<C> {
  static int testReps = 10;
  static int testCycles = 1000;
  static int containerSize = 1000;
  abstract C containerInitializer();
  abstract void startReadersAndWriters();
  C testContainer;
  String testId;
  int nReaders;
  int nWriters;
  volatile long readResult = 0;
  volatile long readTime = 0;
  volatile long writeTime = 0;
  CountDownLatch endLatch;
  static ExecutorService exec =
    Executors.newCachedThreadPool();
  Integer[] writeData;
  Tester(String testId, int nReaders, int nWriters) {
    this.testId = testId + " " +
      nReaders + "r " + nWriters + "w";
    this.nReaders = nReaders;
    this.nWriters = nWriters;
    writeData = Generated.array(Integer.class,
      new RandomGenerator.Integer(), containerSize);
    for(int i = 0; i < testReps; i++) {
      runTest();
      readTime = 0;
      writeTime = 0;
    }
  }
  void runTest() {
    endLatch = new CountDownLatch(nReaders + nWriters);
    testContainer = containerInitializer();
    startReadersAndWriters();
    try {
      endLatch.await();
    } catch(InterruptedException ex) {
      System.out.println("endLatch interrupted");
    }
    System.out.printf("%-27s %14d %14d\n",
      testId, readTime, writeTime);
    if(readTime != 0 && writeTime != 0)
      System.out.printf("%-27s %14d\n",
        "readTime + writeTime =", readTime + writeTime);
  }
  abstract class TestTask implements Runnable {
    abstract void test();
    abstract void putResults();
    long duration;
    public void run() {
      long startTime = System.nanoTime();
      test();
      duration = System.nanoTime() - startTime;
      synchronized(Tester.this) {
        putResults();
      }
      endLatch.countDown();
    }
  }
  public static void initMain(String[] args) {
    if(args.length > 0)
      testReps = new Integer(args[0]);
    if(args.length > 1)
      testCycles = new Integer(args[1]);
    if(args.length > 2)
      containerSize = new Integer(args[2]);
    System.out.printf("%-27s %14s %14s\n",
      "Type", "Read time", "Write time");
  }
}
```

abstract 方法 containerInitializer() 返回将被测试的初始化后的容器，它被存储在 testContainer 域内。另一个 abstract 方法 startReadersAndWriters() 启动读取和写入任务，他们将读取和修改待测容器。不同的测试在运行时将具有数量变化的读取者和写入者，这样就可以观察到锁竞争和写入的效果。我们向构造器提供了各种有关的测试的信息，然后回调用 runTest() 方法 repetions 次。runTest() 将创建一个 CountDownLatch、初始化容器，然后调用 startReadersAndWriters()，并等待他们全部完成。

每个 reader 和 writer 类都基于 TestTask，它可以度量其抽象方法 test() 的执行时间，然后在一个 synchronized 块中调用 putResults() 去存储度量结果。

为了使用这个框架，我们必须让想要测试的特定类型容器继承 Tester，并提供适合的 Reader 和 Writer 类：
```java

abstract class ListTest extends Tester<List<Integer>> {
  ListTest(String testId, int nReaders, int nWriters) {
    super(testId, nReaders, nWriters);
  }
  class Reader extends TestTask {
    long result = 0;
    void test() {
      for(long i = 0; i < testCycles; i++)
        for(int index = 0; index < containerSize; index++)
          result += testContainer.get(index);
    }
    void putResults() {
      readResult += result;
      readTime += duration;
    }
  }
  class Writer extends TestTask {
    void test() {
      for(long i = 0; i < testCycles; i++)
        for(int index = 0; index < containerSize; index++)
          testContainer.set(index, writeData[index]);
    }
    void putResults() {
      writeTime += duration;
    }
  }
  void startReadersAndWriters() {
    for(int i = 0; i < nReaders; i++)
      exec.execute(new Reader());
    for(int i = 0; i < nWriters; i++)
      exec.execute(new Writer());
  }
}

class SynchronizedArrayListTest extends ListTest {
  List<Integer> containerInitializer() {
    return Collections.synchronizedList(
      new ArrayList<Integer>(
        new CountingIntegerList(containerSize)));
  }
  SynchronizedArrayListTest(int nReaders, int nWriters) {
    super("Synched ArrayList", nReaders, nWriters);
  }
}

class CopyOnWriteArrayListTest extends ListTest {
  List<Integer> containerInitializer() {
    return new CopyOnWriteArrayList<Integer>(
      new CountingIntegerList(containerSize));
  }
  CopyOnWriteArrayListTest(int nReaders, int nWriters) {
    super("CopyOnWriteArrayList", nReaders, nWriters);
  }
}

public class ListComparisons {
  public static void main(String[] args) {
    Tester.initMain(args);
    new SynchronizedArrayListTest(10, 0);
    new SynchronizedArrayListTest(9, 1);
    new SynchronizedArrayListTest(5, 5);
    new CopyOnWriteArrayListTest(10, 0);
    new CopyOnWriteArrayListTest(9, 1);
    new CopyOnWriteArrayListTest(5, 5);
    Tester.exec.shutdown();
  }
}
```
执行结果：
```
Type                             Read time     Write time
Synched ArrayList 10r 0w      232158294700              0
Synched ArrayList 9r 1w       198947618203    24918613399
readTime + writeTime =        223866231602
Synched ArrayList 5r 5w       117367305062   132176613508
readTime + writeTime =        249543918570
CopyOnWriteArrayList 10r 0w      758386889              0
CopyOnWriteArrayList 9r 1w       741305671      136145237
readTime + writeTime =           877450908
CopyOnWriteArrayList 5r 5w       212763075    67967464300
readTime + writeTime =         68180227375
```
从输出结果来看，synchronized ArrayList 无论读取者和写入者的数量是多少，都具有大致相同的性能。读取者与其他读取者竞争锁的方式与写入者相同。但是，CopyOnWriteArrayList 在没有写入者时速度回更快。并且在有多个写入者时速度仍然很快。看起来你应该尽量使用 CopyOnWriteArrayList ，对列表写入的影响并没有超过短期同步整个列表的影响。当然，不同的环境下测试是不同的，你必须在你的具体应用中尝试两种不同的方式，以了解那个到底更好。

**比较各种 Map 实现**

我们来比较一下 synchronizedHashMap 和 ConcurrentHashMap 在性能方面比较：
```java
abstract class MapTest
extends Tester<Map<Integer,Integer>> {
  MapTest(String testId, int nReaders, int nWriters) {
    super(testId, nReaders, nWriters);
  }
  class Reader extends TestTask {
    long result = 0;
    void test() {
      for(long i = 0; i < testCycles; i++)
        for(int index = 0; index < containerSize; index++)
          result += testContainer.get(index);
    }
    void putResults() {
      readResult += result;
      readTime += duration;
    }
  }
  class Writer extends TestTask {
    void test() {
      for(long i = 0; i < testCycles; i++)
        for(int index = 0; index < containerSize; index++)
          testContainer.put(index, writeData[index]);
    }
    void putResults() {
      writeTime += duration;
    }
  }
  void startReadersAndWriters() {
    for(int i = 0; i < nReaders; i++)
      exec.execute(new Reader());
    for(int i = 0; i < nWriters; i++)
      exec.execute(new Writer());
  }
}

class SynchronizedHashMapTest extends MapTest {
  Map<Integer,Integer> containerInitializer() {
    return Collections.synchronizedMap(
      new HashMap<Integer,Integer>(
        MapData.map(
          new CountingGenerator.Integer(),
          new CountingGenerator.Integer(),
          containerSize)));
  }
  SynchronizedHashMapTest(int nReaders, int nWriters) {
    super("Synched HashMap", nReaders, nWriters);
  }
}

class ConcurrentHashMapTest extends MapTest {
  Map<Integer,Integer> containerInitializer() {
    return new ConcurrentHashMap<Integer,Integer>(
      MapData.map(
        new CountingGenerator.Integer(),
        new CountingGenerator.Integer(), containerSize));
  }
  ConcurrentHashMapTest(int nReaders, int nWriters) {
    super("ConcurrentHashMap", nReaders, nWriters);
  }
}

public class MapComparisons {
  public static void main(String[] args) {
    Tester.initMain(args);
    new SynchronizedHashMapTest(10, 0);
    new SynchronizedHashMapTest(9, 1);
    new SynchronizedHashMapTest(5, 5);
    new ConcurrentHashMapTest(10, 0);
    new ConcurrentHashMapTest(9, 1);
    new ConcurrentHashMapTest(5, 5);
    Tester.exec.shutdown();
  }
}
```
测试结果：
```
Type                             Read time     Write time
Synched HashMap 10r 0w        306052025049              0
Synched HashMap 9r 1w         428319156207    47697347568
readTime + writeTime =        476016503775
Synched HashMap 5r 5w         243956877760   244012003202
readTime + writeTime =        487968880962
ConcurrentHashMap 10r 0w       23352654318              0
ConcurrentHashMap 9r 1w        18833089400     1541853224
readTime + writeTime =         20374942624
ConcurrentHashMap 5r 5w        12037625732    11850489099
readTime + writeTime =         23888114831
```
向 ConcurrentHashMap 添加写入的影响甚至还不如 CopyOnWriteArrayList 明显，这是因为 ConcurrentHashMap 使用了一种不同的技术，它可以明显地最小化写入所造成的影响。

#### 乐观加锁
尽管 Atomic 对象将执行像 decrementAndGet() 这样的原子操作，但是某些 Atomic 类还允许你执行所谓的乐观加锁。这意味着当你执行某项计算时，实际上没有使用互斥，但是在这个对象计算完成并且准备更新这个对象时，你需要使用一个 compareAndSet() 的方法。你将旧值和新值一起提交给这个方法，如果不一样，那么这个操作失败。这意味着某个地方的任务在这个操作期间修改了这个对象。但是我们是乐观的，因为我们保持数据在未锁定的状态，并希望没有任何其他的任务插入修改它。通过使用 Atomic 来替代 synchronized 或 lock，可以获得性能上的好处。

如果 compareAndSet() 操作失败会怎么样呢？这是一个棘手的问题，也是应用这项技术受限的条件，即只能针对处理相同条件下的问题。如果失败就必须决定做些什么。因为不能执行某些恢复操作，那么你就不能使用这项技术。

下面看一个示例,一旦你运行该程序发现它变慢，并开始应用性能调优技术：
```java
public class FastSimulation {
  static final int N_ELEMENTS = 100000;
  static final int N_GENES = 30;
  static final int N_EVOLVERS = 50;
  static final AtomicInteger[][] GRID =
    new AtomicInteger[N_ELEMENTS][N_GENES];
  static Random rand = new Random(47);
  static class Evolver implements Runnable {
    public void run() {
      while(!Thread.interrupted()) {
        // Randomly select an element to work on:
        int element = rand.nextInt(N_ELEMENTS);
        for(int i = 0; i < N_GENES; i++) {
          int previous = element - 1;
          if(previous < 0) previous = N_ELEMENTS - 1;
          int next = element + 1;
          if(next >= N_ELEMENTS) next = 0;
          int oldvalue = GRID[element][i].get();
          // Perform some kind of modeling calculation:
          int newvalue = oldvalue +
            GRID[previous][i].get() + GRID[next][i].get();
          newvalue /= 3; // Average the three values
          if(!GRID[element][i]
            .compareAndSet(oldvalue, newvalue)) {
            // Policy here to deal with failure. Here, we
            // just report it and ignore it; our model
            // will eventually deal with it.
            print("Old value changed from " + oldvalue);
          }
        }
      }
    }
  }
  public static void main(String[] args) throws Exception {
    ExecutorService exec = Executors.newCachedThreadPool();
    for(int i = 0; i < N_ELEMENTS; i++)
      for(int j = 0; j < N_GENES; j++)
        GRID[i][j] = new AtomicInteger(rand.nextInt(1000));
    for(int i = 0; i < N_EVOLVERS; i++)
      exec.execute(new Evolver());
    TimeUnit.SECONDS.sleep(5);
    exec.shutdownNow();
  }
}
```
所有的元素都被置于数组内，这被认为有助于提高性能。每个 Evolver 对象会用它前一个元素和后一个元素来平均它的值，如果在更新时失败，那么将直接打印这个值并继续执行。注意，在上面的程序中并没有出现任何互斥。

#### ReadWriteLock
ReadWriteLock 对于那种向数据结构中不频繁的写入，但是有多个任务要经常读取这个数据结构的情况进行了优化。ReadWriteLock 使得你可以同时有多个读取者，只要他们都不试图写入即可。如果写锁已经被其他任务持有，那么任何读取者都不能访问，直至这个写锁被释放。

ReadWriteLock 这样就能提高程序的性能吗？这是不确定的。这取决于数据被读的频率和修改的频率比较的结果，读取和写入操作的时间，有多少线程竞争者是否是多处理器上运行等因素。只有实验才能证明你的程序是否被优化了。

下面是 ReadWriteLock 的最基本用法示例：
```java
public class ReaderWriterList<T> {
  private ArrayList<T> lockedList;
  // Make the ordering fair:
  private ReentrantReadWriteLock lock =
    new ReentrantReadWriteLock(true);
  public ReaderWriterList(int size, T initialValue) {
    lockedList = new ArrayList<T>(
      Collections.nCopies(size, initialValue));
  }
  public T set(int index, T element) {
    Lock wlock = lock.writeLock();
    wlock.lock();
    try {
      return lockedList.set(index, element);
    } finally {
      wlock.unlock();
    }
  }
  public T get(int index) {
    Lock rlock = lock.readLock();
    rlock.lock();
    try {
      // Show that multiple readers
      // may acquire the read lock:
      if(lock.getReadLockCount() > 1)
        print(lock.getReadLockCount());
      return lockedList.get(index);
    } finally {
      rlock.unlock();
    }
  }
  public static void main(String[] args) throws Exception {
    new ReaderWriterListTest(30, 1);
  }
}

class ReaderWriterListTest {
  ExecutorService exec = Executors.newCachedThreadPool();
  private final static int SIZE = 100;
  private static Random rand = new Random(47);
  private ReaderWriterList<Integer> list =
    new ReaderWriterList<Integer>(SIZE, 0);
  private class Writer implements Runnable {
    public void run() {
      try {
        for(int i = 0; i < 20; i++) { // 2 second test
          list.set(i, rand.nextInt());
          TimeUnit.MILLISECONDS.sleep(100);
        }
      } catch(InterruptedException e) {
        // Acceptable way to exit
      }
      print("Writer finished, shutting down");
      exec.shutdownNow();
    }
  }
  private class Reader implements Runnable {
    public void run() {
      try {
        while(!Thread.interrupted()) {
          for(int i = 0; i < SIZE; i++) {
            list.get(i);
            TimeUnit.MILLISECONDS.sleep(1);
          }
        }
      } catch(InterruptedException e) {
        // Acceptable way to exit
      }
    }
  }
  public ReaderWriterListTest(int readers, int writers) {
    for(int i = 0; i < readers; i++)
      exec.execute(new Reader());
    for(int i = 0; i < writers; i++)
      exec.execute(new Writer());
  }
}
```
ReaderWriterList 可以持有固定数量的任何类型的对象。你必须向构造器提供所希望的列表尺寸和组装这个列表时所用的初始对象。set() 方法要获取一个写锁，以调用底层的 arrayList.set()，而 get() 方法要获取一个读锁，以调用底层的 arrayList.get()。另外，get() 方法将检查是否已经有多个读取者获取了锁，如果是，咋将显示这种读取者的数量，以证明可以有多个读取者获取锁。

如果查看 JDK 文档 ReentrantReadWriteLock 就会发现有大量的其他方法可用，涉及公平性和政策性等问题。这是一个相当复杂的工具，只有当你在考虑提高性能的方法时，才应该使用它。你的程序第一个方案应该考虑更直观的同步。

## 活动对象
多线程变成非常复杂并且难以正确使用。每个细节都很重要我们需要处理每个细节，没有任何编译器检查形式的安全防护。是多线程模型出了问题吗？毕竟它来自于过程性编程世界，并且几乎没有任何改变。

有一种可替代的方式被称为活动者或行动者。之所以称这些对象是活动的，是因为每个对象都维护着他自己的工作器线程和消息队列，并且所有这种对象的请求都将进入队列排队，任何时刻都只能运行其中一个。因此，有了活动对象，我们就可以串行化消息而不是方法，这意味着不在需要防备一个任务在其循环的中间被中断这种问题。当你向一个活动对象发送消息时，这条消息将被转化为一个任务，该任务会被插入到这个对象队列中，等待在以后的某个时刻运行。Java se5 的 Future 在实现这种模式时将排上用场。

下面是一个示例，有两个方法，可以将方法调用排进队列：
```java
public class ActiveObjectDemo {
	private ExecutorService ex = Executors.newSingleThreadExecutor();
	private Random rand = new Random(47);

	private void pause(int factor) {
		try {
			TimeUnit.MILLISECONDS.sleep(100+ rand.nextInt(factor));
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			System.out.println("sleep interrupted");
		}
	}

	public Future<Integer> calculateInt(final int x,final int y) {
		return ex.submit(new Callable<Integer>() {

			@Override
			public Integer call() throws Exception {
				System.out.println("staring"+x+"+"+y);
				pause(500);
				return x+y;
			}
		});
	}

	 public Future<Float> calculateFloat(final float x, final float y) {
	    return ex.submit(new Callable<Float>() {
	      public Float call() {
	    	  System.out.println("starting " + x + " + " + y);
	        pause(2000);
	        return x + y;
	      }
	    });
	  }

	 public void shutdown() {
		ex.shutdown();
	}

	 public static void main(String[] args) {
		ActiveObjectDemo dl = new ActiveObjectDemo();
		List<Future<?>> results = new CopyOnWriteArrayList<>();
		 for(float f = 0.0f; f < 1.0f; f += 0.2f)
		      results.add(dl.calculateFloat(f, f));
		 for(int i = 0; i < 5; i++)
		      results.add(dl.calculateInt(i, i));
		 System.out.println("All asynch calls made");

		 while (results.size() >0) {
			for (Future<?> future : results) {
				 if(future.isDone()) {
			          try {
			        	  System.out.println(future.get());
			          } catch(Exception e) {
			            throw new RuntimeException(e);
			          }
			          results.remove(future);
			        }
			}
		}
		 dl.shutdown();

		}
	}

```
执行结果：
```
All asynch calls made
starting 0.0 + 0.0
0.0
starting 0.2 + 0.2
0.4
starting 0.4 + 0.4
0.8
starting 0.6 + 0.6
1.2
starting 0.8 + 0.8
1.6
staring0+0
0
staring1+1
staring2+2
2
4
staring3+3
6
staring4+4
8
```
newSingleThreadExecutor() 的调用产生的单线程执行器维护者它自己的无界阻塞队列，并且只有一个线程从该队列中取走任务并执行他们直至完成。我们需要在 calculateInt() 和 calculateFloat() 中做的就是 submit() 提交一个新的 Callable 对象，以相应对这些方法的调用，这样就可以把方法转变为消息，而 submit() 的方法体包含在匿名内部类中。注意，每个活动对象的方法返回值都是一个具有泛型参数的 Future,而这个泛型参数就是该方法的实际的返回类型。通过这种方式方法调用可以立即返回，调用者可以通过 Future 来发现任务合适完成，并收集实际的返回值。

main() 方法中创建了一个 List<Future<?>> 来捕获由活动对象的 calculateInt() 和 calculateFloat() 消息返回的 Future 对象。对于每个 Future，都使用 isDone() 来从这个列表中抽取的。这种方式使得当 Future 完成并且其结果被处理之后，就会从 List 中移除。注意，使用 CopyOnWriteArrayList 可以移除为了防止 ConcurrentModificationException 异常而复制 List 的这种需求。

为了能够防止在不经意间防止线程的耦合，任何传递给活动对象的方法调用的参数必须都是只读的其他活动对象，或者是不连接对象，即没有连接其他任何任务的对象。

有了活动对象：

1. 每个对象都可以拥有自己的工作器线程
2. 每个对象都将维护对它自己的域的全部控制权
3. 所有的活动对象之间的通讯都将以这些对象之间的消息形式发生
4. 活动对象之前的所有消息都要排队

由于从一个活动对象到另一个活动对象的消息只能被排队时的延迟所阻塞，并且因为这个延迟总是非常短暂的而且独立于任何其他对象，所以发送消息实际上是不可阻塞的。由于一个活动对象系统只是经由消息来通信，所以两个对象在竞争调用另一个对象上的方法时，是不会被阻塞的，而这意味着不会发生死锁。这是一种巨大的进步。因为在活动对象的工作区线程在任何时刻只执行一个消息，所以不存在任何资源竞争，而你也不用操心如何同步方法。同步仍旧会发生，但是它通过将方法调用排队，使得任何时刻都只能发生一个调用，从而将同步控制在消息级别上发生。

## 总结
这个系列介绍了 Java 并发编程设计的基础知识，我们需要理解 Java 并发编程的内容：

1. 可以运行多个独立的任务
2. 必须考虑当这些任务关闭时，可能出现的所有问题。
3. 任务可能会在共享资源上彼此干涉。互斥(锁) 是用来防止这种冲突的基本工具。
4. 如果任务设计的不够仔细，就可能会死锁。

明白设么时候使用并发、什么时候应该避免使用并发是非常关键的。使用它的主要原因是：

1. 要处理很多任务，他们交织在一起，应用并发能够有效的使用计算机
2. 要能够更好的组织代码
3. 要更便于用户使用

多线程的主要缺陷：

1. 等待共享资源的时候性能降低
2. 需要处理线程的额外 CPU 花费
3. 糟糕的程序设计导致不必要的复杂性
4. 有可能产生一些病态行为，如，饿死、竞争、死锁和活锁(多个任务各自运行的线程使得整体无法完成)
5. 不同平台导致的不一致性。

因为线程可能共享资源，比如一个对象的内存，而且你必须确定多个线程不会同时读取和改变这个资源，这就是线程锁产生的最大问题。这需要明智的使用可用的加锁机制，他们仅仅是工具，同时也可能引入潜在的死锁条件，所以要对他们理解透彻。
