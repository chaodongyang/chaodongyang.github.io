---
layout: post
title: java编程思想之容器深入研究(性能的测试)
categories: javaThinking
description: java编程思想之容器深入研究
keywords: java, java容器深入研究,容器深入研究，java容器
---

尽管只有四种容器，Map、List、Set、和 Queue。但是每一种接口都不止一个实现版本。我们应该如何选择使用哪一种实现呢？

## 选择接口的不同实现
每种不同的实现都有自己的特定和优缺点。从容器分类图可以看出，HashMap、Vector、和 Stack 是过去遗留下来的类，目的只是为了支持老的程序，最好不要在新的程序中使用它们。在 Java 类库中不同类型的 Queue 只在它们接受和产生数值的方式上有所差异。

容器之间的差别通常归结为由什么在背后支持它们。也就是说所使用的的容易是由什么样的数据结构实现的。例如，ArrayList 和 LinkedList 都实现了 List 接口，无论选择哪一个最基本的 List 操作都是支持的。然而 ArrayList 底层是又数组支持的；而 LinkedList 是由双向链表实现的，其中的每个对象包含数据的同时还指向链表中前一个和后一个元素的引用。如果经常在表中插入和删除数据，LinkedList 就比较合适；否则，应该使用速度更快的 ArrayList。

还有，Set 被实现为 TreeSet、HashSet、或 LinkedHashSet。每一种都有不同的行为；HashSet 是最常用的，查询速度最快；LinkedHashSet 保持元素的插入顺序；TreeSet 基于 TreeMap 生成一个总是处于排序状态的 Set。

有时某个容器的不同实现拥有一些共同的操作，但是这些操作的性能是不同的。你可以根据你使用某种特定操作的频率，以及你需要执行的速度来从中选择合适的接口。

#### 性能测试框架
为了防止代码重复以及提供测试的一致性，我们将测试的基本功能放在一个测试框架中。建立一个基本类，从中创建一个匿名内部类列表，每个匿名内部类都用于每种不同的测试，他们都当做测试过程中的一部分而被调用。这是模板方法设计模式的另一个示例：

TestParam 类：
```java
public class TestParam {
	//容器中的元素数量
	public final int size;
	//控制迭代的次数
	public final int loops;

	protected TestParam(int size, int loops) {
		super();
		this.size = size;
		this.loops = loops;
	}

	// 创建 TestParam 类型的数组以供测试
	public static TestParam[] array(int... values) {
	    int size = values.length/2;
	    TestParam[] result = new TestParam[size];
	    int n = 0;
	    for(int i = 0; i < size; i++)
	      result[i] = new TestParam(values[n++], values[n++]);
	    return result;
	 }

	// 创建 TestParam 类型的数组以供测试
	 public static TestParam[] array(String[] values) {
	    int[] vals = new int[values.length];
	    for(int i = 0; i < vals.length; i++)
	      vals[i] = Integer.decode(values[i]);
	    return array(vals);
	  }
}

```
抽象类：
```java
public abstract class Test<C> {
	String name;

	protected Test(String name) {
		super();
		this.name = name;
	}


	abstract int  test(C container,TestParam tp);
}

```
测试工具类：
```java
public class Tester<C> {
	  //格式化字符串的宽度
	  public static int fieldWidth = 8;
	  //创建的默认测试数组
	  public static TestParam[] defaultParams= TestParam.array(10, 5000, 100, 5000, 1000, 5000, 10000, 500);
	  // Override this to modify pre-test initialization:
	  protected C container;
	  protected C initialize(int size) {
		  return container;
	  }

	  private String headline = "";
	  private List<Test<C>> tests;
	  //格式化字符串
	  private static String stringField() {
	    return "%" + fieldWidth + "s";
	  }
	  private static String numberField() {
	    return "%" + fieldWidth + "d";
	  }

	  private static int sizeWidth = 5;
	  private static String sizeField = "%" + sizeWidth + "s";
	  private TestParam[] paramList = defaultParams;

	  public Tester(C container, List<Test<C>> tests) {
	    this.container = container;
	    this.tests = tests;
	    if(container != null)
	      headline = container.getClass().getSimpleName();
	  }


	  public Tester(C container, List<Test<C>> tests,TestParam[] paramList) {
	    this(container, tests);
	    this.paramList = paramList;
	  }


	  public void setHeadline(String newHeadline) {
	    headline = newHeadline;
	  }

	  // Generic methods for convenience :
	  public static <C> void run(C cntnr, List<Test<C>> tests){
	    new Tester<C>(cntnr, tests).timedTest();
	  }


	  public static <C> void run(C cntnr,List<Test<C>> tests, TestParam[] paramList) {
	    new Tester<C>(cntnr, tests, paramList).timedTest();
	  }

	  private void displayHeader() {
	    // Calculate width and pad with '-':
	    int width = fieldWidth * tests.size() + sizeWidth;
	    int dashLength = width - headline.length() - 1;
	    StringBuilder head = new StringBuilder(width);
	    for(int i = 0; i < dashLength/2; i++)
	      head.append('-');
	      head.append(' ');
	      head.append(headline);
	      head.append(' ');
	    for(int i = 0; i < dashLength/2; i++)
	      head.append('-');
	    System.out.println(head);
	    // Print column headers:
	    System.out.format(sizeField, "size");
	    for(Test test : tests)
	      System.out.format(stringField(), test.name);
	      System.out.println();
	  }
	  // Run the tests for this container:
	  public void timedTest() {
	    displayHeader();
	    for(TestParam param : paramList) {
	      System.out.format(sizeField, param.size);
	      for(Test<C> test : tests) {
	        C kontainer = initialize(param.size);
	        long start = System.nanoTime();
	        // Call the overriden method:
	        int reps = test.test(kontainer, param);
	         //产生执行操作的纳秒数
	        long duration = System.nanoTime() - start;
	        long timePerRep = duration / reps; // Nanoseconds
	        System.out.format(numberField(), timePerRep);
	      }
	      System.out.println();
	    }
	  }
}

```
#### 对 List 的选择
下面是对 List 操作中最本质的部分的性能测试。为了进行比较还 展示了 Queue 中的重要操作。该例子创建了两个分离的用于测试每一种容器类的测试列表。

创建一个自定义的 List 工具类：
```java
public class CountingIntegerList extends AbstractList<Integer> {
	  private int size;
	  public CountingIntegerList(int size) {
	    this.size = size < 0 ? 0 : size;
	  }
	  public Integer get(int index) {
	    return Integer.valueOf(index);
	  }
	  public int size() { return size; }

	  public static void main(String[] args) {
	    System.out.println(new CountingIntegerList(30));
	  }
}

```
执行结果：
```
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29]
```
创建一个数组的工具类：```CollectionData<T>``` 在我们前面的例子中已经创建了。它集成自 ```ArrayList<T>``` 返回一个 List 的容器。
```java
public class Generated {
	// Fill an existing array:
	  public static <T> T[] array(T[] a, Generator<T> gen) {
	    return new CollectionData<T>(gen, a.length).toArray(a);
	  }
	  // Create a new array:
	  @SuppressWarnings("unchecked")
	  public static <T> T[] array(Class<T> type,
	      Generator<T> gen, int size) {
	    T[] a = (T[])java.lang.reflect.Array.newInstance(type, size);
	    return new CollectionData<T>(gen, size).toArray(a);
	  }
}
```
创建的工具类：主要是通过内部类返回不同的泛型类型。其中 ```Generator<T>``` 也在我们前边的实例中创建过，它主要的作用就是 next() 方法返回当前泛型的类型。
```java
public class CountingGenerator {
	  public static class Boolean implements Generator<java.lang.Boolean> {
	    private boolean value = false;
	    public java.lang.Boolean next() {
	      value = !value; // Just flips back and forth
	      return value;
	    }
	  }

	  public static class Byte implements Generator<java.lang.Byte> {
	    private byte value = 0;
	    public java.lang.Byte next() { return value++; }
	  }

	  static char[] chars = ("abcdefghijklmnopqrstuvwxyz" +"ABCDEFGHIJKLMNOPQRSTUVWXYZ").toCharArray();

	  public static class Character implements Generator<java.lang.Character> {
	    int index = -1;
	    public java.lang.Character next() {
	      index = (index + 1) % chars.length;
	      return chars[index];
	    }
	  }

	  public static class String implements Generator<java.lang.String> {
	    private int length = 7;
	    Generator<java.lang.Character> cg = new Character();
	    public String() {

	    }
	    public String(int length) {
	    	this.length = length;
	    }

	    public java.lang.String next() {
	      char[] buf = new char[length];
	      for(int i = 0; i < length; i++)
	        buf[i] = cg.next();
	      return new java.lang.String(buf);
	    }
	  }


	  public static class Short implements Generator<java.lang.Short> {
	    private short value = 0;
	    public java.lang.Short next() {
	    	return value++;
	    }
	  }

	  public static class Integer implements Generator<java.lang.Integer> {
	    private int value = 0;
	    public java.lang.Integer next() {
	    	return value++;
	    }
	  }

	  public static class Long implements Generator<java.lang.Long> {
	    private long value = 0;
	    public java.lang.Long next() {
	    	return value++;
	    }
	  }

	  public static class Float implements Generator<java.lang.Float> {
	    private float value = 0;
	    public java.lang.Float next() {
	      float result = value;
	      value += 1.0;
	      return result;
	    }
	  }

	  public static class Double implements Generator<java.lang.Double> {
	    private double value = 0.0;
	    public java.lang.Double next() {
	      double result = value;
	      value += 1.0;
	      return result;
	    }
	  }
}

```
最后去测试不同的容器的性能：
```java
public class ListPerformance {
	  static Random rand = new Random();
	  static int reps = 1000;
	  static List<Test<List<Integer>>> tests = new ArrayList<Test<List<Integer>>>();
	  static List<Test<LinkedList<Integer>>> qTests = new ArrayList<Test<LinkedList<Integer>>>();
	  static {
	    tests.add(new Test<List<Integer>>("add") {
	      int test(List<Integer> list, TestParam tp) {
	        int loops = tp.loops;
	        int listSize = tp.size;
	        for(int i = 0; i < loops; i++) {
	          list.clear();
	          for(int j = 0; j < listSize; j++)
	            list.add(j);
	        }
	        return loops * listSize;
	      }
	    });

	    tests.add(new Test<List<Integer>>("get") {
	      int test(List<Integer> list, TestParam tp) {
	        int loops = tp.loops * reps;
	        int listSize = list.size();
	        for(int i = 0; i < loops; i++)
	          list.get(rand.nextInt(listSize));
	        return loops;
	      }
	    });

	    tests.add(new Test<List<Integer>>("set") {
	      int test(List<Integer> list, TestParam tp) {
	        int loops = tp.loops * reps;
	        int listSize = list.size();
	        for(int i = 0; i < loops; i++)
	          list.set(rand.nextInt(listSize), 47);
	        return loops;
	      }
	    });

	    tests.add(new Test<List<Integer>>("iteradd") {
	      int test(List<Integer> list, TestParam tp) {
	        final int LOOPS = 1000000;
	        int half = list.size() / 2;
	        ListIterator<Integer> it = list.listIterator(half);
	        for(int i = 0; i < LOOPS; i++)
	          it.add(47);
	        return LOOPS;
	      }
	    });

	    tests.add(new Test<List<Integer>>("insert") {
	      int test(List<Integer> list, TestParam tp) {
	        int loops = tp.loops;
	        for(int i = 0; i < loops; i++)
	          list.add(5, 47); // Minimize random-access cost
	        return loops;
	      }
	    });

	    tests.add(new Test<List<Integer>>("remove") {
	      int test(List<Integer> list, TestParam tp) {
	        int loops = tp.loops;
	        int size = tp.size;
	        for(int i = 0; i < loops; i++) {
	          list.clear();
	          list.addAll(new CountingIntegerList(size));
	          while(list.size() > 5)
	            list.remove(5); // Minimize random-access cost
	        }
	        return loops * size;
	      }
	    });

	    // Tests for queue behavior:
	    qTests.add(new Test<LinkedList<Integer>>("addFirst") {
	      int test(LinkedList<Integer> list, TestParam tp) {
	        int loops = tp.loops;
	        int size = tp.size;
	        for(int i = 0; i < loops; i++) {
	          list.clear();
	          for(int j = 0; j < size; j++)
	            list.addFirst(47);
	        }
	        return loops * size;
	      }
	    });

	    qTests.add(new Test<LinkedList<Integer>>("addLast") {
	      int test(LinkedList<Integer> list, TestParam tp) {
	        int loops = tp.loops;
	        int size = tp.size;
	        for(int i = 0; i < loops; i++) {
	          list.clear();
	          for(int j = 0; j < size; j++)
	            list.addLast(47);
	        }
	        return loops * size;
	      }
	    });

	    qTests.add(
	      new Test<LinkedList<Integer>>("rmFirst") {
	        int test(LinkedList<Integer> list, TestParam tp) {
	          int loops = tp.loops;
	          int size = tp.size;
	          for(int i = 0; i < loops; i++) {
	            list.clear();
	            list.addAll(new CountingIntegerList(size));
	            while(list.size() > 0)
	              list.removeFirst();
	          }
	          return loops * size;
	        }
	      });

	    qTests.add(new Test<LinkedList<Integer>>("rmLast") {
	      int test(LinkedList<Integer> list, TestParam tp) {
	        int loops = tp.loops;
	        int size = tp.size;
	        for(int i = 0; i < loops; i++) {
	          list.clear();
	          list.addAll(new CountingIntegerList(size));
	          while(list.size() > 0)
	            list.removeLast();
	        }
	        return loops * size;
	      }
	    });
	  }

	  static class ListTester extends Tester<List<Integer>> {
	    public ListTester(List<Integer> container,List<Test<List<Integer>>> tests) {
	      super(container, tests);
	    }

	    // Fill to the appropriate size before each test:
	    @Override protected List<Integer> initialize(int size){
	      container.clear();
	      container.addAll(new CountingIntegerList(size));
	      return container;
	    }

	    // Convenience method:
	    public static void run(List<Integer> list,List<Test<List<Integer>>> tests) {
	      new ListTester(list, tests).timedTest();
	    }

	  }

	  public static void main(String[] args) {
	    if(args.length > 0)
	      Tester.defaultParams = TestParam.array(args);
	    // Can only do these two tests on an array:
	    Tester<List<Integer>> arrayTest =  new Tester<List<Integer>>(null, tests.subList(1, 3)){
	        // This will be called before each test. It
	        // produces a non-resizeable array-backed list:
	        @Override protected
	        List<Integer> initialize(int size) {
	          Integer[] ia = Generated.array(Integer.class,new CountingGenerator.Integer(), size);
	          return Arrays.asList(ia);
	        }
	      };
	    arrayTest.setHeadline("Array as List");
	    arrayTest.timedTest();
	    Tester.defaultParams= TestParam.array(
	      10, 5000, 100, 5000, 1000, 1000, 10000, 200);
	    if(args.length > 0)
	      Tester.defaultParams = TestParam.array(args);
	    ListTester.run(new ArrayList<Integer>(), tests);
	    ListTester.run(new LinkedList<Integer>(), tests);
	    ListTester.run(new Vector<Integer>(), tests);
	    Tester.fieldWidth = 12;
	    Tester<LinkedList<Integer>> qTest =
	      new Tester<LinkedList<Integer>>(
	        new LinkedList<Integer>(), qTests);
	    qTest.setHeadline("Queue tests");
	    qTest.timedTest();
	  }
}

```
最后执行的结果：
```
--- Array as List ---
 size     get     set
   10      13      15
  100      13      14
 1000      13      13
10000      14      14
--------------------- ArrayList ---------------------
 size     add     get     set iteradd  insert  remove
   10      67      15      15      25     282     122
  100      18      13      14      17     219      38
 1000      21      15      14      49     226      76
10000       9      15      14     379    1019     434
--------------------- LinkedList ---------------------
 size     add     get     set iteradd  insert  remove
   10     106      35      37      59     345     136
  100      17      57      53      12     118      32
 1000      14     355     378      11      48      22
10000      17    4269    4679      69      82      15
----------------------- Vector -----------------------
 size     add     get     set iteradd  insert  remove
   10     100      17      19      27     327      75
  100      18      16      18      21     333      31
 1000      13      16      17      53     229      73
10000      11      16      22     412    1171     573
-------------------- Queue tests --------------------
 size    addFirst     addLast     rmFirst      rmLast
   10          85          61          62          80
  100          29          29          24          24
 1000          12          15          27          27
10000          11           8          11          11

```
从上面的执行结果我们可以总结出一些特点：get 和 set 操作对于背后有数组支撑的 List 和 ArrayList 无论列表的大小如何。访问速度都很快和一致；而对于 LinkedList 访问时间对于较大的列表将明显的增加。很显然对于要执行大量的随机访问操作链表不是一种好的选择。

iteradd 测试使用迭代器在列表中插入新的元素。对于，arrayList，当列表变大时，其开销将变的很大，但对于 LinkedList，相对来说比较低廉，并且不随尺寸变化而变化。这是因为 ArrayList 在插入时，必须创建空间并将它所有引用向前移动，这会随着 ArrayList 的尺寸增加而增加。LinkedList 只需要连接新的元素，而不需要修改表中剩余的元素。

insert 和 remove 测试都使用了索引位置 5 作为插入或移除点，而没有使用 List 两端的元素。LinkedList 会对 List 的端点进行特殊处理。这使得将 LinekdList 用作 Queue 时，速度可以得到提高。但是如果你在列表中增加或移除元素，其中会包含随机访问的代价。从输出可以看出，在 LinekdLIist 中插入和移除代价是低廉的，并且不会随着尺寸变化发生变化，但是对于 ArrayList ，插入操作代价非常的高昂，并且随着尺寸的增加而增加。

从 Queue 测试中，你可以看到 LinkedList 可以非常快速的从列表的端点插入和移除元素，这正是 Queue 所做的优化。

应该避免使用 Vector ，它只是为了支持遗留代码而已，只是为了向前兼容。

最佳的做法是将 ArrayList 做为默认首选，只有你需要使用额外的功能，或者当程序的性能因为经常从表中插入和删除而变差的时候，才回去选择 LinkedList。如果使用的是固定元素数量，那么既可以选择背后由数组支撑的 List ，也可以选择真正的数组。

#### 微基准测试的危险
我们上面做的测试就可以叫微基准测试，简言之就是小范围的测试。此时有可能会有很多问题，我们原则上不能做太多的假设，并且尽可能的将我们的测试窄化。还必须运行足够长的时间来保证测试数据有意义。根据计算机和所使用的 JVM 的不同，产生的记过也可能不同。所以我们不应该关心这些绝对的数字。另外，我们可以使用 java 中的剖析器。他可能做的更好。

有一个 Math.random() 相关的示例：它产生 0 到 1 的值吗？包括还是不包括 1。
```java
public class RandomBounds {
	     static void usage() {
		    System.out.println("Usage:");
		    System.out.println("\tRandomBounds lower");
		    System.out.println("\tRandomBounds upper");
		    System.exit(1);
		  }

		  public static void main(String[] args) {
		    if(args.length != 1) usage();

		    if(args[0].equals("lower")) {
		      while(Math.random() != 0.0); // Keep trying
		      System.out.println("Produced 0.0!");
		    } else if(args[0].equals("upper")) {
		      while(Math.random() != 1.0) ; // Keep trying
		      System.out.println("Produced 1.0!");
		    }
		    else
		      usage();
		  }
}

```
上面的程序看起来永远都不会产生 0.0 或 1.0.但这正是实验的误导之处。实际上 0.0 是包含在其中的。因此，我们有时候必须分析我们的实验，并理解他们的局限性。

#### 对 Set 的选择
可以根据需要选择 TreeSet、HashSet、或者LinkedHashSet。下面的测试表示了他们的性能表现：
```java
public class SetPerformance {
	static List<Test<Set<Integer>>> tests = new ArrayList<Test<Set<Integer>>>();
		  static {
		    tests.add(new Test<Set<Integer>>("add") {
		      int test(Set<Integer> set, TestParam tp) {
		        int loops = tp.loops;
		        int size = tp.size;
		        for(int i = 0; i < loops; i++) {
		          set.clear();
		          for(int j = 0; j < size; j++)
		            set.add(j);
		        }
		        return loops * size;
		      }
		    });

		    tests.add(new Test<Set<Integer>>("contains") {
		      int test(Set<Integer> set, TestParam tp) {
		        int loops = tp.loops;
		        int span = tp.size * 2;
		        for(int i = 0; i < loops; i++)
		          for(int j = 0; j < span; j++)
		            set.contains(j);
		        return loops * span;
		      }
		    });

		    tests.add(new Test<Set<Integer>>("iterate") {
		      int test(Set<Integer> set, TestParam tp) {
		        int loops = tp.loops * 10;
		        for(int i = 0; i < loops; i++) {
		          Iterator<Integer> it = set.iterator();
		          while(it.hasNext())
		            it.next();
		        }
		        return loops * set.size();
		      }
		    });
		  }

		  public static void main(String[] args) {
		    if(args.length > 0)
		    Tester.defaultParams = TestParam.array(args);
		    Tester.fieldWidth = 10;
		    Tester.run(new TreeSet<Integer>(), tests);
		    Tester.run(new HashSet<Integer>(), tests);
		    Tester.run(new LinkedHashSet<Integer>(), tests);
		  }
}

```
测试结果：
```
------------- TreeSet -------------
 size       add  contains   iterate
   10       284       117        34
  100        48        21         5
 1000        52        44         5
10000        69        67         5
------------- HashSet -------------
 size       add  contains   iterate
   10       179        57        57
  100        13         6         7
 1000        17         5         5
10000        17         4         5
---------- LinkedHashSet ----------
 size       add  contains   iterate
   10        96        29        17
  100        25         9         6
 1000        25         9         5
10000        25         9         5
```
HashSet 的性能基本上总是比 TreeSet 好，特别是添加和查询元素。TreeSet 存在的唯一原因是它可以维持元素的排序状态。对于插入操作，LinkedHashSet 比 HashSet 的代价更高。这是因为维护链表所带来的的额外的开销。

#### 对 Map 的选择
下面的程序展示对 Map 不同实现的性能测试：
```java
public class MapPerformance {

	static List<Test<Map<Integer,Integer>>> tests = new ArrayList<Test<Map<Integer,Integer>>>();
		  static {
		    tests.add(new Test<Map<Integer,Integer>>("put") {
		      int test(Map<Integer,Integer> map, TestParam tp) {
		        int loops = tp.loops;
		        int size = tp.size;
		        for(int i = 0; i < loops; i++) {
		          map.clear();
		          for(int j = 0; j < size; j++)
		            map.put(j, j);
		        }
		        return loops * size;
		      }
		    });
		    tests.add(new Test<Map<Integer,Integer>>("get") {
		      int test(Map<Integer,Integer> map, TestParam tp) {
		        int loops = tp.loops;
		        int span = tp.size * 2;
		        for(int i = 0; i < loops; i++)
		          for(int j = 0; j < span; j++)
		            map.get(j);
		        return loops * span;
		      }
		    });
		    tests.add(new Test<Map<Integer,Integer>>("iterate") {
		      int test(Map<Integer,Integer> map, TestParam tp) {
		        int loops = tp.loops * 10;
		        for(int i = 0; i < loops; i ++) {
		          Iterator it = map.entrySet().iterator();
		          while(it.hasNext())
		            it.next();
		        }
		        return loops * map.size();
		      }
		    });
		  }
		  public static void main(String[] args) {
		    if(args.length > 0)
		      Tester.defaultParams = TestParam.array(args);
		    Tester.run(new TreeMap<Integer,Integer>(), tests);
		    Tester.run(new HashMap<Integer,Integer>(), tests);
		    Tester.run(new LinkedHashMap<Integer,Integer>(),tests);
		    Tester.run(new IdentityHashMap<Integer,Integer>(), tests);
		    Tester.run(new WeakHashMap<Integer,Integer>(), tests);
		    Tester.run(new Hashtable<Integer,Integer>(), tests);
		  }

}

```
测试结果：
```
---------- TreeMap ----------
 size     put     get iterate
   10     251     111      31
  100      52      32       6
 1000      55      43       5
10000      75      61       8
---------- HashMap ----------
 size     put     get iterate
   10     188      63      36
  100      14       3       5
 1000      17       5       5
10000      17       4       5
------- LinkedHashMap -------
 size     put     get iterate
   10     105      31      13
  100      30       8       6
 1000      27       8       5
10000      27       9       7
------ IdentityHashMap ------
 size     put     get iterate
   10     102      44      26
  100      21      29      15
 1000      68      59      16
10000      65      63      17
-------- WeakHashMap --------
 size     put     get iterate
   10     131      42      21
  100      34       6       8
 1000      25       8      13
10000      25       7       8
--------- Hashtable ---------
 size     put     get iterate
   10      90      34      21
  100      39      21       8
 1000      25      24       8
10000      32      24       8

```
我们看到查找的速度总是比插入的速度要快。另外 HashTable 的性能比 HashMap 的性能要低。其实他们使用了相同的底层存储和查找机制。但是 HashMap 是用来取代 HashTable 的，而且我使用的是最新的 JDK8 所以测试的结果和书上的不太一样。可能在早期的版本中他们是一致的。TreeMap 比 HashMap 要慢，因为它也要维护元素的插入顺序。值得注意的是另外一个 IdentityHashMap 它与别的具有完全不同的性能，因为它是通过 ```==``` 来进行比较的。

#### HashMap 的性能因子
我们可以通过手工调整 HashMap 来提高其性能。调整 HashMap 提高性能我们还必须了解一下术语：
- 容量：表中的元素个数
- 初始容量：表在创建时所拥有的元素数。
- 尺寸：表中当前存储的个数。
- 负载因子：储存/容量。空表的负载因子是 0.而半满表的负载因子是 0.5。负载轻的表产生冲突的可能性小，因此对插入和查找都是最理想的，但是会减慢迭代器遍历的过程。HashMap 和 HashSet 都具有允许你指定负载因子的构造器，表示当负载情况达到负载因子的水平时，容器将自动增加容量，实现方式是大致增加一倍。并重新将现有的对象重新散列到表中。

HashMap 使用的负载因子是 0.75。只有当表达到四分之三时在散列。这个因子在时间和控件代价上达到了平衡。更高的负载因子可以降低表需要的空间，但是会增加查找代价。如果你知道将要在 HashMap 中存储多少元素，那么创建一个具有恰当大小的初始容量讲可以避免自动在散列的开销。
