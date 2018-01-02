---
layout: post
title: java编程思想之并发(SE5 新特性)
categories: javaThinking
description: java编程思想之并发
keywords: java, java并发, java多线程，Java多线程并发
---

## 新类库中的构建
Java SE5 的类库中引入了大量的新设计来解决并发问题的新类。学习他们将有助于编写更加简单而健壮的并发程序。

#### CountDownLatch
他被用来同步一个或多个任务，强制他们等待由其他的任务执行的一组操作完成。

你可以向 CountDownLatch 对象设置一个初始计数值，任何在这个对象上调用 wait() 的方法都将阻塞，直至这个计数值到达 0。其他任务在结束其工作时，可以在该对象上调用 CountDown() 来减小这个计数值。CountDownLatch 被设计为只触发一次，计数值不能被重置。如果需要重置计数值的版本，则可以使用 CyclicBarrier 版本。调用 countDown() 的任务在产生这个调用时并没有被阻塞，只有对 await() 的调用会被阻塞，直至技术值到达 0。

CountDownLatch 的典型用法是将一个任务分为 n 个互相独立的可解决任务，并创建值为 0 的 CountDownLatch。当每个任务完成时，都会在这个锁存器上调用 countDown()。等待问题被解决的任务在这个锁存器上调用 await()，将他们自己拦住，直至锁存器计数结束。
```java
import java.util.concurrent.*;
import java.util.*;
import static net.mindview.util.Print.*;

// Performs some portion of a task:
class TaskPortion implements Runnable {
  private static int counter = 0;
  private final int id = counter++;
  private static Random rand = new Random(47);
  private final CountDownLatch latch;
  TaskPortion(CountDownLatch latch) {
    this.latch = latch;
  }
  public void run() {
    try {
      doWork();
      latch.countDown();
    } catch(InterruptedException ex) {
      // Acceptable way to exit
    }
  }
  public void doWork() throws InterruptedException {
    TimeUnit.MILLISECONDS.sleep(rand.nextInt(2000));
    print(this + "completed");
  }
  public String toString() {
    return String.format("%1$-3d ", id);
  }
}

// Waits on the CountDownLatch:
class WaitingTask implements Runnable {
  private static int counter = 0;
  private final int id = counter++;
  private final CountDownLatch latch;
  WaitingTask(CountDownLatch latch) {
    this.latch = latch;
  }
  public void run() {
    try {
      latch.await();
      print("Latch barrier passed for " + this);
    } catch(InterruptedException ex) {
      print(this + " interrupted");
    }
  }
  public String toString() {
    return String.format("WaitingTask %1$-3d ", id);
  }
}

public class CountDownLatchDemo {
  static final int SIZE = 100;
  public static void main(String[] args) throws Exception {
    ExecutorService exec = Executors.newCachedThreadPool();
    // All must share a single CountDownLatch object:
    CountDownLatch latch = new CountDownLatch(SIZE);
    for(int i = 0; i < 10; i++)
      exec.execute(new WaitingTask(latch));
    for(int i = 0; i < SIZE; i++)
      exec.execute(new TaskPortion(latch));
    print("Launched all tasks");
    exec.shutdown(); // Quit when all tasks complete
  }
} /* (Execute to see output) *///:~

```
所有的任务都使用在 main() 中定义的同一个单一的 CountDownLatch。

**类库的线程安全**

类中包含了一个静态的 Random 对象，这意味着多个任务可能会同时调用 Random.nextInt()。在这种情况下，可以通过向 TaskPortion 提供自己的 Random 对象来解决。也就是说通过移除 static 限定符的方式解决。对于 Java 标准类库来说那些是线程安全的，那些是线程不安全的，JDK 文档并没有指出。要理解这一点必须逐个的去查看源码，恰好 Random.nextInt() 是线程安全的。

#### CyclicBarrier
你希望创建一组任务，他们并行的执行工作，然后在进行下一个步骤之前等待，直至所有任务都完成。他使得所有的任务都在栅栏处等待，因此可以一致的向前移动。

下面是模仿赛马游戏的一个仿真版本：
```java
class Horse implements Runnable {
  private static int counter = 0;
  private final int id = counter++;
  private int strides = 0;
  private static Random rand = new Random(47);
  private static CyclicBarrier barrier;
  public Horse(CyclicBarrier b) { barrier = b; }
  public synchronized int getStrides() { return strides; }
  public void run() {
    try {
      while(!Thread.interrupted()) {
        synchronized(this) {
          strides += rand.nextInt(3); // Produces 0, 1 or 2
        }
        barrier.await();
      }
    } catch(InterruptedException e) {
      // A legitimate way to exit
    } catch(BrokenBarrierException e) {
      // This one we want to know about
      throw new RuntimeException(e);
    }
  }
  public String toString() { return "Horse " + id + " "; }
  public String tracks() {
    StringBuilder s = new StringBuilder();
    for(int i = 0; i < getStrides(); i++)
      s.append("*");
    s.append(id);
    return s.toString();
  }
}

public class HorseRace {
  static final int FINISH_LINE = 75;
  private List<Horse> horses = new ArrayList<Horse>();
  private ExecutorService exec =
    Executors.newCachedThreadPool();
  private CyclicBarrier barrier;
  public HorseRace(int nHorses, final int pause) {
    barrier = new CyclicBarrier(nHorses, new Runnable() {
      public void run() {
        StringBuilder s = new StringBuilder();
        for(int i = 0; i < FINISH_LINE; i++)
          s.append("="); // The fence on the racetrack
        print(s);
        for(Horse horse : horses)
          print(horse.tracks());
        for(Horse horse : horses)
          if(horse.getStrides() >= FINISH_LINE) {
            print(horse + "won!");
            exec.shutdownNow();
            return;
          }
        try {
          TimeUnit.MILLISECONDS.sleep(pause);
        } catch(InterruptedException e) {
          print("barrier-action sleep interrupted");
        }
      }
    });
    for(int i = 0; i < nHorses; i++) {
      Horse horse = new Horse(barrier);
      horses.add(horse);
      exec.execute(horse);
    }
  }
  public static void main(String[] args) {
    int nHorses = 7;
    int pause = 200;
    if(args.length > 0) { // Optional argument
      int n = new Integer(args[0]);
      nHorses = n > 0 ? n : nHorses;
    }
    if(args.length > 1) { // Optional argument
      int p = new Integer(args[1]);
      pause = p > -1 ? p : pause;
    }
    new HorseRace(nHorses, pause);
  }
}
```
可以向 CyclicBarrier 提供一个栅栏动作，它是一个 Runnable，当计数值到达 0 时自动执行。这里栅栏动作是作为匿名内部类来创建的，它被提交给 CyclicBarrier 的构造器。CyclicBarrier 使得每匹马都执行了向前移动所必须执行的工作，然后等待栅栏出所有的马都准备完毕。当所有的马向前移动时，CyclicBarrier 将自动调用 Runnable 栅栏动作任务，按顺序显示马和终点线的位置。一旦所有的任务越过了栅栏，它就会自动地为下一回合比赛做好准备。

#### DelayQueue
这是一个无界的 BlockingQueue，用于放置实现 DelayQueue 接口的对象，其中的对象只能在其到期时才能从队列中取走。这种队列是有序的，即队列对象的延迟到期的时间最长。如果没有任何延迟到期时间，那么就不会有任何头元素，并且 poll() 将返回 null。

下面示例：其中的 Delayed 对象自身就是任务，而 DelayedTaskConsumer 将最紧急的任务(到期时间最长的任务)从队列中取出，然后运行它。
```java
class DelayedTask implements Runnable, Delayed {
  private static int counter = 0;
  private final int id = counter++;
  private final int delta;
  private final long trigger;
  protected static List<DelayedTask> sequence =
    new ArrayList<DelayedTask>();
  public DelayedTask(int delayInMilliseconds) {
    delta = delayInMilliseconds;
    trigger = System.nanoTime() +
      NANOSECONDS.convert(delta, MILLISECONDS);
    sequence.add(this);
  }
  public long getDelay(TimeUnit unit) {
    return unit.convert(
      trigger - System.nanoTime(), NANOSECONDS);
  }
  public int compareTo(Delayed arg) {
    DelayedTask that = (DelayedTask)arg;
    if(trigger < that.trigger) return -1;
    if(trigger > that.trigger) return 1;
    return 0;
  }
  public void run() { printnb(this + " "); }
  public String toString() {
    return String.format("[%1$-4d]", delta) +
      " Task " + id;
  }
  public String summary() {
    return "(" + id + ":" + delta + ")";
  }
  public static class EndSentinel extends DelayedTask {
    private ExecutorService exec;
    public EndSentinel(int delay, ExecutorService e) {
      super(delay);
      exec = e;
    }
    public void run() {
      for(DelayedTask pt : sequence) {
        printnb(pt.summary() + " ");
      }
      print();
      print(this + " Calling shutdownNow()");
      exec.shutdownNow();
    }
  }
}

class DelayedTaskConsumer implements Runnable {
  private DelayQueue<DelayedTask> q;
  public DelayedTaskConsumer(DelayQueue<DelayedTask> q) {
    this.q = q;
  }
  public void run() {
    try {
      while(!Thread.interrupted())
        q.take().run(); // Run task with the current thread
    } catch(InterruptedException e) {
      // Acceptable way to exit
    }
    print("Finished DelayedTaskConsumer");
  }
}

public class DelayQueueDemo {
  public static void main(String[] args) {
    Random rand = new Random(47);
    ExecutorService exec = Executors.newCachedThreadPool();
    DelayQueue<DelayedTask> queue =
      new DelayQueue<DelayedTask>();
    // Fill with tasks that have random delays:
    for(int i = 0; i < 20; i++)
      queue.put(new DelayedTask(rand.nextInt(5000)));
    // Set the stopping point
    queue.add(new DelayedTask.EndSentinel(5000, exec));
    exec.execute(new DelayedTaskConsumer(queue));
  }
}
```
DelayedTask 包含一个称为 sequence 的 List<DelayedTask>，它保存了任务被创建的顺序，因此我们看到排序是按照实际发生的顺序执行的。Delsyed 接口有一个方法名叫 getDealay()，他可以用来告知延迟到期多长时间，或者延迟在多长时间以前已经到期。这个方法将强制我们使用 TimeUnit 类，因为这就是参数类型。

#### PriorityBlockingQueue

这是一个很基础的优先级队列，它具有可阻塞的读取操作。下面是一个示例，其中在优先级队列中的对象是按照优先级顺序从队列中出现的任务。PrioritizedTask 被赋予一个优先级数字，以此来提供顺序：
```java
class PrioritizedTask implements
Runnable, Comparable<PrioritizedTask>  {
  private Random rand = new Random(47);
  private static int counter = 0;
  private final int id = counter++;
  private final int priority;
  protected static List<PrioritizedTask> sequence =
    new ArrayList<PrioritizedTask>();
  public PrioritizedTask(int priority) {
    this.priority = priority;
    sequence.add(this);
  }
  public int compareTo(PrioritizedTask arg) {
    return priority < arg.priority ? 1 :
      (priority > arg.priority ? -1 : 0);
  }
  public void run() {
    try {
      TimeUnit.MILLISECONDS.sleep(rand.nextInt(250));
    } catch(InterruptedException e) {
      // Acceptable way to exit
    }
    print(this);
  }
  public String toString() {
    return String.format("[%1$-3d]", priority) +
      " Task " + id;
  }
  public String summary() {
    return "(" + id + ":" + priority + ")";
  }
  public static class EndSentinel extends PrioritizedTask {
    private ExecutorService exec;
    public EndSentinel(ExecutorService e) {
      super(-1); // Lowest priority in this program
      exec = e;
    }
    public void run() {
      int count = 0;
      for(PrioritizedTask pt : sequence) {
        printnb(pt.summary());
        if(++count % 5 == 0)
          print();
      }
      print();
      print(this + " Calling shutdownNow()");
      exec.shutdownNow();
    }
  }
}

class PrioritizedTaskProducer implements Runnable {
  private Random rand = new Random(47);
  private Queue<Runnable> queue;
  private ExecutorService exec;
  public PrioritizedTaskProducer(
    Queue<Runnable> q, ExecutorService e) {
    queue = q;
    exec = e; // Used for EndSentinel
  }
  public void run() {
    // Unbounded queue; never blocks.
    // Fill it up fast with random priorities:
    for(int i = 0; i < 20; i++) {
      queue.add(new PrioritizedTask(rand.nextInt(10)));
      Thread.yield();
    }
    // Trickle in highest-priority jobs:
    try {
      for(int i = 0; i < 10; i++) {
        TimeUnit.MILLISECONDS.sleep(250);
        queue.add(new PrioritizedTask(10));
      }
      // Add jobs, lowest priority first:
      for(int i = 0; i < 10; i++)
        queue.add(new PrioritizedTask(i));
      // A sentinel to stop all the tasks:
      queue.add(new PrioritizedTask.EndSentinel(exec));
    } catch(InterruptedException e) {
      // Acceptable way to exit
    }
    print("Finished PrioritizedTaskProducer");
  }
}

class PrioritizedTaskConsumer implements Runnable {
  private PriorityBlockingQueue<Runnable> q;
  public PrioritizedTaskConsumer(
    PriorityBlockingQueue<Runnable> q) {
    this.q = q;
  }
  public void run() {
    try {
      while(!Thread.interrupted())
        // Use current thread to run the task:
        q.take().run();
    } catch(InterruptedException e) {
      // Acceptable way to exit
    }
    print("Finished PrioritizedTaskConsumer");
  }
}

public class PriorityBlockingQueueDemo {
  public static void main(String[] args) throws Exception {
    Random rand = new Random(47);
    ExecutorService exec = Executors.newCachedThreadPool();
    PriorityBlockingQueue<Runnable> queue =
      new PriorityBlockingQueue<Runnable>();
    exec.execute(new PrioritizedTaskProducer(queue, exec));
    exec.execute(new PrioritizedTaskConsumer(queue));
  }
}
```
PrioritizedTask 对象的创建序列被记录在 sequence List 中，用于和实际的执行顺序比较。PrioritizedTaskProducer 和 PrioritizedTaskConsumer 通过 PriorityBlockingQueue 彼此连接。因为这种队列的阻塞特性提供了所有必须的同步，所以你应该注意到，这里不需要任何显示的同步，不必考虑当你从这种队列读取时，其中是否还有元素，因为这种队列在没有元素时将直接阻塞读取者。

#### 使用 ScheduledThreadPoolExecutor 的温室控制器
假定温室控制系统的示例，它可以控制各种设施的开关，或者是对他们进行调节。这可以被看做是一种并发问题，每个期望的事件都是一个预定事件运行的任务。通过使用 schedule() 运行一次任务或者使用 scheduleAtFixedRate() 每隔规则的时间重复执行任务，你可以将 Runnable 对象设置为在将来的某个时刻执行。
```java
public class GreenhouseScheduler {
  private volatile boolean light = false;
  private volatile boolean water = false;
  private String thermostat = "Day";
  public synchronized String getThermostat() {
    return thermostat;
  }
  public synchronized void setThermostat(String value) {
    thermostat = value;
  }
  ScheduledThreadPoolExecutor scheduler =
    new ScheduledThreadPoolExecutor(10);
  public void schedule(Runnable event, long delay) {
    scheduler.schedule(event,delay,TimeUnit.MILLISECONDS);
  }
  public void
  repeat(Runnable event, long initialDelay, long period) {
    scheduler.scheduleAtFixedRate(
      event, initialDelay, period, TimeUnit.MILLISECONDS);
  }
  class LightOn implements Runnable {
    public void run() {
      // Put hardware control code here to
      // physically turn on the light.
      System.out.println("Turning on lights");
      light = true;
    }
  }
  class LightOff implements Runnable {
    public void run() {
      // Put hardware control code here to
      // physically turn off the light.
      System.out.println("Turning off lights");
      light = false;
    }
  }
  class WaterOn implements Runnable {
    public void run() {
      // Put hardware control code here.
      System.out.println("Turning greenhouse water on");
      water = true;
    }
  }
  class WaterOff implements Runnable {
    public void run() {
      // Put hardware control code here.
      System.out.println("Turning greenhouse water off");
      water = false;
    }
  }
  class ThermostatNight implements Runnable {
    public void run() {
      // Put hardware control code here.
      System.out.println("Thermostat to night setting");
      setThermostat("Night");
    }
  }
  class ThermostatDay implements Runnable {
    public void run() {
      // Put hardware control code here.
      System.out.println("Thermostat to day setting");
      setThermostat("Day");
    }
  }
  class Bell implements Runnable {
    public void run() { System.out.println("Bing!"); }
  }
  class Terminate implements Runnable {
    public void run() {
      System.out.println("Terminating");
      scheduler.shutdownNow();
      // Must start a separate task to do this job,
      // since the scheduler has been shut down:
      new Thread() {
        public void run() {
          for(DataPoint d : data)
            System.out.println(d);
        }
      }.start();
    }
  }
  // New feature: data collection
  static class DataPoint {
    final Calendar time;
    final float temperature;
    final float humidity;
    public DataPoint(Calendar d, float temp, float hum) {
      time = d;
      temperature = temp;
      humidity = hum;
    }
    public String toString() {
      return time.getTime() +
        String.format(
          " temperature: %1$.1f humidity: %2$.2f",
          temperature, humidity);
    }
  }
  private Calendar lastTime = Calendar.getInstance();
  { // Adjust date to the half hour
    lastTime.set(Calendar.MINUTE, 30);
    lastTime.set(Calendar.SECOND, 00);
  }
  private float lastTemp = 65.0f;
  private int tempDirection = +1;
  private float lastHumidity = 50.0f;
  private int humidityDirection = +1;
  private Random rand = new Random(47);
  List<DataPoint> data = Collections.synchronizedList(
    new ArrayList<DataPoint>());
  class CollectData implements Runnable {
    public void run() {
      System.out.println("Collecting data");
      synchronized(GreenhouseScheduler.this) {
        // Pretend the interval is longer than it is:
        lastTime.set(Calendar.MINUTE,
          lastTime.get(Calendar.MINUTE) + 30);
        // One in 5 chances of reversing the direction:
        if(rand.nextInt(5) == 4)
          tempDirection = -tempDirection;
        // Store previous value:
        lastTemp = lastTemp +
          tempDirection * (1.0f + rand.nextFloat());
        if(rand.nextInt(5) == 4)
          humidityDirection = -humidityDirection;
        lastHumidity = lastHumidity +
          humidityDirection * rand.nextFloat();
        // Calendar must be cloned, otherwise all
        // DataPoints hold references to the same lastTime.
        // For a basic object like Calendar, clone() is OK.
        data.add(new DataPoint((Calendar)lastTime.clone(),
          lastTemp, lastHumidity));
      }
    }
  }
  public static void main(String[] args) {
    GreenhouseScheduler gh = new GreenhouseScheduler();
    gh.schedule(gh.new Terminate(), 5000);
    // Former "Restart" class not necessary:
    gh.repeat(gh.new Bell(), 0, 1000);
    gh.repeat(gh.new ThermostatNight(), 0, 2000);
    gh.repeat(gh.new LightOn(), 0, 200);
    gh.repeat(gh.new LightOff(), 0, 400);
    gh.repeat(gh.new WaterOn(), 0, 600);
    gh.repeat(gh.new WaterOff(), 0, 800);
    gh.repeat(gh.new ThermostatDay(), 0, 1400);
    gh.repeat(gh.new CollectData(), 500, 500);
  }
}
```
DataPoint 可以持有并显示单个的数据段，而 CollectData 是被调度的任务，它在每次运行时，都可以产生仿真数据，并将其添加到 greenhouse 的 List<DataPoint> 中。注意：volatile 和 synchornized 在适当的场合都得到了应用，以防止任务之间的互相干涉。在持有 DataPoint 的 List 中的所有方法都是 synchronized 的，这是因为 List 被创建时，使用了synchronizedList()。

#### Semaphore
正常的锁在任何时刻都只允许一个任务访问资源，而技术信号量允许 n 个任务同时访问这个资源。可以看做信号量是向外分发使用资源的许可证。

作为一个示例，请考虑对象池的概念，他管理着数量有限的对象，当要使用对象时可以迁出他们，而在用户使用完毕时，可以将他们签回。这种功能可以封装到一个泛型类中：
```java
public class Pool<T> {
  private int size;
  private List<T> items = new ArrayList<T>();
  private volatile boolean[] checkedOut;
  private Semaphore available;
  public Pool(Class<T> classObject, int size) {
    this.size = size;
    checkedOut = new boolean[size];
    available = new Semaphore(size, true);
    // Load pool with objects that can be checked out:
    for(int i = 0; i < size; ++i)
      try {
        // Assumes a default constructor:
        items.add(classObject.newInstance());
      } catch(Exception e) {
        throw new RuntimeException(e);
      }
  }
  public T checkOut() throws InterruptedException {
    available.acquire();
    return getItem();
  }
  public void checkIn(T x) {
    if(releaseItem(x))
      available.release();
  }
  private synchronized T getItem() {
    for(int i = 0; i < size; ++i)
      if(!checkedOut[i]) {
        checkedOut[i] = true;
        return items.get(i);
      }
    return null; // Semaphore prevents reaching here
  }
  private synchronized boolean releaseItem(T item) {
    int index = items.indexOf(item);
    if(index == -1) return false; // Not in the list
    if(checkedOut[index]) {
      checkedOut[index] = false;
      return true;
    }
    return false; // Wasn't checked out
  }
}
```
在这个简化的形式中，构造器 newInstance() 来把对象加载到池中。如果你需要一个新对象，那么可以调用 checkOut()，并且在使用完之后，将其递交给 checkIn()。

boolean 类型的数组 checkedOut 可以跟踪被签出的对象，并且可以通过 getItem() 和 releaseItem() 方法来管理。而这些都将由 Semaphore 类型的 available 来加以确保，因此在 checkOut() 中，如果没有任何信号量许可证可用，available 将阻塞调用过程。在 checkIn() 中，如果被签入的对象有效，则会向信号量返回一个许可证。

我们看下面一个示例：创建一个 Fat:
```java
public class Fat {
  private volatile double d; // Prevent optimization
  private static int counter = 0;
  private final int id = counter++;
  public Fat() {
    // Expensive, interruptible operation:
    for(int i = 1; i < 10000; i++) {
      d += (Math.PI + Math.E) / (double)i;
    }
  }
  public void operation() { System.out.println(this); }
  public String toString() { return "Fat id: " + id; }
}
```
我们在池中管理这些对象，以限制这个构造器所造成的影响。我们创建一个任务将签出 Fat 对象，持有一段时间以后在将他们签入：
```java
class CheckoutTask<T> implements Runnable {
  private static int counter = 0;
  private final int id = counter++;
  private Pool<T> pool;
  public CheckoutTask(Pool<T> pool) {
    this.pool = pool;
  }
  public void run() {
    try {
      T item = pool.checkOut();
      print(this + "checked out " + item);
      TimeUnit.SECONDS.sleep(1);
      print(this +"checking in " + item);
      pool.checkIn(item);
    } catch(InterruptedException e) {
      // Acceptable way to terminate
    }
  }
  public String toString() {
    return "CheckoutTask " + id + " ";
  }
}

public class SemaphoreDemo {
  final static int SIZE = 25;
  public static void main(String[] args) throws Exception {
    final Pool<Fat> pool =
      new Pool<Fat>(Fat.class, SIZE);
    ExecutorService exec = Executors.newCachedThreadPool();
    for(int i = 0; i < SIZE; i++)
      exec.execute(new CheckoutTask<Fat>(pool));
    print("All CheckoutTasks created");
    List<Fat> list = new ArrayList<Fat>();
    for(int i = 0; i < SIZE; i++) {
      Fat f = pool.checkOut();
      printnb(i + ": main() thread checked out ");
      f.operation();
      list.add(f);
    }
    Future<?> blocked = exec.submit(new Runnable() {
      public void run() {
        try {
          // Semaphore prevents additional checkout,
          // so call is blocked:
          pool.checkOut();
        } catch(InterruptedException e) {
          print("checkOut() Interrupted");
        }
      }
    });
    TimeUnit.SECONDS.sleep(2);
    blocked.cancel(true); // Break out of blocked call
    print("Checking in objects in " + list);
    for(Fat f : list)
      pool.checkIn(f);
    for(Fat f : list)
      pool.checkIn(f); // Second checkIn ignored
    exec.shutdown();
  }
}
```
在 main() 中创建了一个持有 Fat 对象的 Pool,而一组 CheckoutTask 则开始操作这个 Pool，然后，main() 线程签出池中的 Fat 对象，但是并不签入他们。一旦池中所有的对象都被签出，Semaphore 将不再允许执行任何签出的操作。blocked 的 run() 方法因此会被阻塞， 2 秒之后 cancel() 方法被调用，依次来挣脱 Future 的束缚。

## 总结
SE5 中的新类库包含了许多的新特性，为我们解决问题提供了许多的方便。现在 JDK 已经更新到了 8。要学习更多的新的内容还需不断的学习新版本的 JDK。
