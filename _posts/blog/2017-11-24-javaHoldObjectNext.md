---
layout: post
title: java编程思想之持有对象二
categories: javaThinking
description: java编程思想之持有对象
keywords: java, java持有对象详解, java持有对象
---

Java 持有对象的存储方式多种多样，昨天主要是学习了单个序列存储。主要是 Collection 的接口。今天我们主要来看看键值对的存储方式。

## Map
将对象影射到其他对象额能力是解决编程问题的重要能力。下面的例子我们将使用随机数来作为键的值，随机数出现的次数用来作为值存储。来统计一下随机数的概率：

```java
public class MapTest {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Random random = new Random(47);
		Map<Integer, Integer> map = new HashMap<>();
		for (int i = 0; i < 1000; i++) {
			int r = random.nextInt(10);
			Integer m = map.get(r);
			map.put(r, m == null ? 1: m+1);
		}

		System.out.println(map);
	}

}

```
执行结果：
```
{0=100, 1=99, 2=101, 3=98, 4=96, 5=103, 6=95, 7=103, 8=97, 9=108}
```
我在再来看一下 Map 的其他的一些方法：

```java
public class MapTest {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Random random = new Random(47);
		Map<String, Apple> map = new HashMap<>();
		map.put("这是一个水果", new Apple());
		map.put("这是一个橘子", new Orange());
		map.put("这是一个香蕉", new Banner());

		System.out.println(map);

		Apple apple = map.get("这是一个橘子");
		System.out.println(apple);

		System.out.println(map.containsKey("这是一个橘子"));
		System.out.println(map.containsValue(apple));

	}

}

```
执行结果：
```
{这是一个水果=collection.Apple@6d06d69c, 这是一个香蕉=collection.Banner@7852e922, 这是一个橘子=collection.Orange@4e25154f}
collection.Orange@4e25154f
true
true
```
Map 本身是一个数组，他和其他的数组并没有多大的区别。Map 的值也可以是一个容器甚至是他自己。这样我们就可以将容器组合起来从而快速的生成复杂的数据结构。

```java
public class MapTest {

	public static void main(String[] args) {
		// TODO Auto-generated method stub

		Map<String, List<Integer>> map = new HashMap<String, List<Integer>>();

		map.put("小明的成绩", Arrays.asList(90,110,130,85,120,130,99));
		map.put("海洋的成绩", Arrays.asList(100,120,140,95,110,100,99));
		map.put("我的成绩", Arrays.asList(150,150,130,95,100,100,99));

		System.out.println(map.keySet());
		System.out.println(map.values());

		for (String string : map.keySet()) {
			System.out.println(string + "成绩单：");
			for (Integer integer : map.get(string)) {
				System.out.print(integer);
			}
			System.out.println("\n");
		}

	}

}

```
执行结果：
```
[海洋的成绩, 我的成绩, 小明的成绩]
[[100, 120, 140, 95, 110, 100, 99], [150, 150, 130, 95, 100, 100, 99], [90, 110, 130, 85, 120, 130, 99]]
海洋的成绩成绩单：
1001201409511010099

我的成绩成绩单：
1501501309510010099

小明的成绩成绩单：
901101308512013099
```
## Queue

队列是一个典型的先进先出的容器。即从容器的一端放入事物，从另一端取出，并且事物放入容器的顺序和取出的顺序是相同的。队列常常作为一种可靠的将对象从程序的某个区域传输到另外一个区域的途径。特别是并发编程。他可以安全的将对象从一个任务传输到下一个任务。

LinkedList 提供了方法用来支持队列的行为，并且它实现了 Queue 接口，通过将它向上转型为 Queue 来实现下面这个例子：
```java
public class QueueDemo {

	private static void  printQueue(Queue queue) {
		//返回队列头部是否为空
		while (queue.peek() !=null) {
			//将队列头部移除并返回
			System.out.print(queue.remove()+"-");
		}
		System.out.println("");
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Queue<Integer> queue = new LinkedList<>();
		Random random = new Random(47);
		for (int i = 0; i < 10; i++) {
			//将元素插入到队尾
			queue.offer(random.nextInt(20));
		}
		printQueue(queue);
		Queue<Character> queue2 = new LinkedList<>();
		for (char character : "abcdefghijklmn".toCharArray()) {
			//将元素插入到队尾
			queue2.offer(character);
		}
		printQueue(queue2);
	}

}

```
执行结果：
```
18-15-13-1-1-9-8-0-2-7-
a-b-c-d-e-f-g-h-i-j-k-l-m-n-

```
通过上面的方法的注释，我们可以看出，我们首先将一个元素插入到队尾，然后去检查队列中有第一个元素有没有，有第一个就将第一个移除并返回。这样一来。每次插入的元素其实都是上一次元素移除之后的位置。此外，Queue 接口窄化了对 LinkedList 的方法的访问权限。因此你能够访问 LinkedList 的方法会变少。

#### priority queue 优先队列
先进先出描述了最典型的的队列规则。队列规则是指在给定一组队列元素的情况下，确定下一个弹出元素的规则。先进先出声明的是下一个元素应该是等待时间最长的元素。优先队列声明下一个弹出的元素是最需要的元素。例如机场当飞机临时起飞时，这架飞机的乘客将会办理登记手续时排到队头。如果构建一个消息系统，某些消息更重要，因而你应该更快的处理。

当你调用 priorityQueue 的 offer() 方法插入一个对象时，这个对象在队列中排序。默认的排序将使用自然排序，但是你也可以通过提供自己的 Comparator 来修改这个排序。优先队列可以保证你获取元素将是队列中优先级最高的。

```java
public class PriorityQueueDemo {


	public static void main(String[] args) {
		// 优先队列
		PriorityQueue<Integer> qIntegers = new PriorityQueue<>();
		Random random = new Random(47);
		for (int i = 0; i < 10; i++) {
			qIntegers.offer(random.nextInt(20));
			QueueDemo.printQueue(qIntegers);
		}
		System.out.println("");
		System.out.println("优先小的先出");
		List<Integer> integers = Arrays.asList(25,22,20,18,14,9,3,1,1,2,3,9,14,21,23,25);
		qIntegers = new PriorityQueue<>(integers);
		QueueDemo.printQueue(qIntegers);
		System.out.println("");
		System.out.println("----------------");
    //Collections.reverseOrder()产生一个反序的 Comparator
		qIntegers = new PriorityQueue<Integer>(integers.size(),Collections.reverseOrder());
		qIntegers.addAll(integers);
		System.out.println(qIntegers);
		//打乱顺序之后，优先小的出
		QueueDemo.printQueue(qIntegers);

	}

}

```
执行结果：
```
18-15-13-1-1-9-8-0-2-7-
优先小的先出
1-1-2-3-3-9-9-14-14-18-20-21-22-23-25-25-
----------------
[25, 25, 23, 22, 14, 14, 21, 18, 1, 2, 3, 9, 9, 3, 20, 1]
25-25-23-22-21-20-18-14-14-9-9-3-3-2-1-1-
```
我们看到重复是允许的，最小的值拥有最高的优先级。最后一个 qIntegers 容器我们通过给定一个 Comparator 来自定义一个优先级。Collections.reverseOrder() 将会产生一个反序的 Comparator。

## Collection 和 Iterator

Collection 是描述有序序列容器的共性的跟接口，即要表示其他若干接口的共性而出现的接口。AbstractCollection 类提供了 Collection 的默认实现。我们可以去创建它的子类来实现容器。可以减少很多没有必要的代码重复。

使用这些接口来创建可以使我们创建出更通用的代码。通过针对接口而非具体实现来编码。我们编写一个方法去接受 Collection ，那么该方法就可以接受任何实现了 Collection 的子类。在 Java 中我们也可以用迭代器来表示容器的共性。但是这两个方法都是一起使用的。因为实现 Collection 就意味着要提供 iterator() 方法。

```java
public class InterfaceVsIterator {

	public static void display(Iterator<Integer> iterator) {
		while (iterator.hasNext()) {
			System.out.print(iterator.next()+"*");
		}
		System.out.println("");
	}

	private static void display(Collection<Integer> integers) {
		for (Integer integer : integers) {
			System.out.print(integer+"*");
		}
		System.out.println("");
	}

	public static void main(String[] args) {
		List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8,9);
		Set<Integer> set = new HashSet<>(list);
		Map<String, Integer> map = new LinkedHashMap<>();
		for (int i = 0; i < 10; i++) {
			map.put("map"+i, i*10);
		}

		display(list);
		display(set);
		display(list.iterator());
		display(set.iterator());
		display(map.values());
		display(map.values().iterator());

	}

}

```
执行结果：
```
1*2*3*4*5*6*7*8*9*
1*2*3*4*5*6*7*8*9*
1*2*3*4*5*6*7*8*9*
1*2*3*4*5*6*7*8*9*
0*10*20*30*40*50*60*70*80*90*
0*10*20*30*40*50*60*70*80*90*
```
当我们要实现一个不是 Collection 的外部类的时候，我们可以让他去继承 Collection 类。但是这样做是非常麻烦的。因为这其中有很多方法我们不必使用。但是现在我们可以去继承
AbstractCollection 就会很容易的实现。

```java
public class CollectionSequence extends AbstractCollection<Integer>{

	private Integer[] integers = {1,2,3,4,5,6,7,8};

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		CollectionSequence sequence = new CollectionSequence();
		InterfaceVsIterator.display(sequence);
		InterfaceVsIterator.display(sequence.iterator());

	}

	@Override
	public Iterator<Integer> iterator() {
		// TODO Auto-generated method stub
		return new Iterator<Integer>() {
			private int index =0;
			@Override
			public boolean hasNext() {
				// TODO Auto-generated method stub
				return index < integers.length;
			}

			@Override
			public Integer next() {
				// TODO Auto-generated method stub
				return integers[index++];
			}

		};
	}

	@Override
	public int size() {
		// TODO Auto-generated method stub
		return integers.length;
	}

}

```
执行结果：
```
1*2*3*4*5*6*7*8*
1*2*3*4*5*6*7*8*
```
我们知道 Java 中是单继承，如果我们已经继承了其他的类该怎么办呢？此时我们可以手动提供迭代器来实现。

```java
public class PetSequence {
	protected Integer[] integers ={1,2,3,4,5,6,7,8,9};
}
```
继承 PetSequence 并且手动实现迭代器：

```java
public class NotCollectionSequence extends PetSequence{

	public Iterator<Integer> iterator() {

		return new Iterator<Integer>() {
			private int index =0;
			@Override
			public boolean hasNext() {
				// TODO Auto-generated method stub
				return index<integers.length;
			}

			@Override
			public Integer next() {
				// TODO Auto-generated method stub
				return integers[index++];
			}

		};

	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		NotCollectionSequence sequence = new NotCollectionSequence();
		InterfaceVsIterator.display(sequence.iterator());
	}

}

```
执行结果：
```
1*2*3*4*5*6*7*8*9*
```
最后一种方法生成 Iterator 是将队列与消费队列的方法连接在一起的耦合度最小的方式，与实现 Collection 相比约束性更小。

## Foreach 与迭代器
我们的 Foreach 语法主要用再数组中，但是他也可以应用于任何的 Collection 对象。

```java
public class ForeachCollections {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Collection<String> cStrings = new LinkedList<>();
		Collections.addAll(cStrings,"chao dong yang ".split(" "));
		for (String string : cStrings) {
			System.out.print(string+" ");
		}

	}

}
```
执行结果：
```
chao dong yang
```
上边的例子之所以能够工作，是因为在 JavaSE5 引入了新的被称为 Iterator 的接口，改接口包含了一个能够产生 Iterator 的 iterator() 方法，并且 Iterator 接口被用来在使用 Foreach 在序列中移动。因此如果我们手动去实现 Iterator 接口，那么我们的类也可以使用 Foreach。

```java
public interface Collection<E> extends Iterable<E> {}
```
Iterable 接口：
```java
public interface Iterable<T> {
    /**
     * Returns an iterator over elements of type {@code T}.
     *
     * @return an Iterator.
     */
    Iterator<T> iterator();
}
```
自定义一个实现 Iterable 接口的类：
```java
public class IterableClass implements Iterable<String>{
	private String[] words = "chao dong yang shi hao ren ".split(" ");

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		for (String string : new IterableClass()) {
			System.out.print(string+" ");
		}
	}

	@Override
	public Iterator<String> iterator() {
		// TODO Auto-generated method stub
		return new Iterator<String>() {
			private int x =0;
			@Override
			public boolean hasNext() {
				// TODO Auto-generated method stub
				return x<words.length;
			}

			@Override
			public String next() {
				// TODO Auto-generated method stub
				return words[x++];
			}

		};
	}

}

```
执行结果：
```
chao dong yang shi hao ren
```
大量的类都是 Iterable 类型的，主要包括所有的 Collection 类。Foreach 可以用于数组和所有的 Iterable 类。但这不意味着数组也是 Iterable 类。

```java
public class NotIterable {

	static <T> void test(Iterable<T> iterable){
		for (T t : iterable) {
			System.out.print(t);
		}
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		test(Arrays.asList(1,2,3));
		String[] strings = {"A","B","C"};
		//这种方法是错误的
		//test(strings);
		test(Arrays.asList(strings));
	}

}

```
尝试把数组传递给一个接受 Iterable 参数的方法是失败的。你必须手动执行这种转换。

## 适配器方法
我们使用适配器的方案来新增一个我们的方法。我们希望在迭代器的基础上新增一家反向迭代器的能力。我们此时需要添加一个能够产生 Iterable 对象的方法。改对象用于 Foreach。

```java
public class ReversibleArrayList<T> extends ArrayList<T> {

	protected ReversibleArrayList(Collection<? extends T> c) {
		super(c);
		// TODO Auto-generated constructor stub
	}

	public Iterable<T> reversed() {
		return new Iterable<T>() {

			@Override
			public Iterator<T> iterator() {
				// TODO Auto-generated method stub
				return new Iterator<T>() {
					int current = size() -1;
					@Override
					public boolean hasNext() {
						// TODO Auto-generated method stub
						return current > -1;
					}

					@Override
					public T next() {
						// TODO Auto-generated method stub
						return get(current--);
					}

				};			}
		};
	}


}

```
调用并执行：
```java
public class AdapterMethod {


	public static void main(String[] args) {
		// TODO Auto-generated method stub
		ReversibleArrayList<String> rList = new ReversibleArrayList<>(Arrays.asList("chao dong yang ".split(" ")));
		for (String string : rList) {
			System.out.print(string);
		}
		System.out.println("  ");
		for (String string : rList.reversed()) {
			System.out.print(string);
		}
	}

}
```
执行结果：
```
chaodongyang  
yangdongchao
```
下面我们来修改一下前面的一个例子：

```java
public class IterableClass implements Iterable<String>{
	private String[] words = "chao dong yang shi hao ren ".split(" ");

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		for (String string : new IterableClass()) {
			System.out.print(string+" ");
		}

		System.out.println(" ");
		for (String string : new IterableClass().reversed()) {
			System.out.print(string+" ");
		}

		System.out.println(" ");
		for (String string : new IterableClass().random()) {
			System.out.print(string+" ");
		}

	}

	//翻转顺序
	public Iterable<String> reversed() {
		return new Iterable<String>() {

			@Override
			public Iterator<String> iterator() {
				// TODO Auto-generated method stub
				return new Iterator<String>() {
					int current = words.length -1;
					@Override
					public boolean hasNext() {
						// TODO Auto-generated method stub
						return current > -1;
					}

					@Override
					public String next() {
						// TODO Auto-generated method stub
						return words[current--];
					}

				};
			}
		};
	}

	//打乱顺序
	public Iterable<String> random() {
		return new Iterable<String>() {

			@Override
			public Iterator<String> iterator() {
				// TODO Auto-generated method stub
				//用这个方法改变的顺序是 strings的引用。words数组顺序不变
				//List<String> strings = new ArrayList<>(Arrays.asList(words));
				//Arrays.asList(words)产生的list将会被直接打乱。words数组会被从底层打乱
				List<String> strings = Arrays.asList(words);
				Collections.shuffle(strings,new Random(47));
				return strings.iterator();
			}
		};

	}

	@Override
	public Iterator<String> iterator() {
		// TODO Auto-generated method stub
		return new Iterator<String>() {
			private int x =0;
			@Override
			public boolean hasNext() {
				// TODO Auto-generated method stub
				return x<words.length;
			}

			@Override
			public String next() {
				// TODO Auto-generated method stub
				return words[x++];
			}

		};
	}



}

```
执行结果：
```
chao dong yang shi hao ren  
ren hao shi yang dong chao  
dong hao shi ren chao yang
```
## 总结
- 数组将数字和对象关联起来。他保存类型明确的对象，查阅对象时不需要对结果进行类型转换。可以是多维的。但是数组一旦形成，容量不允许改变。

- Collection 保存单一的元素，而 Map 保存相关的键值对。有了 Java 的泛型机制就可以指定容器中存在的对象，并且获取元素的时候不需要类型转换。各种 Collection 和 Map 可以添加更多的元素，自动调整尺寸。容器不能持有基本类型，但是自动包装机制可以进行转换。

- List 和数组一样建立了数字和对象之间的关联。所以他们都是排好序的容器。List 可以自动填充尺寸。

- 如果进行大量的随机访问，就是用 ArrayList。如果要经常插入和删除元素就是用 LinkedList.

- 各种 Queue 以及栈的行为，由 LinkedList 提供支持。

- Map 是一种将对象与对象关联起来的设计。HasMap 用来快速的访问；TreeMap 保持键始终处于排序状态，所以没有 HasMap 快。LinkedHashMap 保持元素插入的顺序，但也可以通过散列提供快速访问能力。

- Set 不接受重复的元素。 HashSet 提供最快的查询速度，而 TreeSet 保持元素处于排序状态。LinkedHasSet 以插入顺序保存元素。

- Vector、Hashtable 和 Stack 过时不应该在使用。

Java 容器简图

![](/images/blog/collection.png)
