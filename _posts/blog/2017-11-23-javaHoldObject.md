---
layout: post
title: java编程思想之持有对象一
categories: javaThinking
description: java编程思想之持有对象
keywords: java, java持有对象详解, java持有对象
---

如果一个程序只包含固定数量的且生命周期都是已知的对象，那么这是一个简单的程序。

通常程序总是在运行的时候才知道某些条件的去创建新的对象。在此之前并不知道对象的数量和确切的类型。例如前面讲过的数组，如果你想保存一组基本类型的数据也可以用数组。但是数组是固定的大小。我们需要更加复杂的方式来存储对象。Java 中的类库给我们提供了一套完整的容器类来解决这个问题，其中基本的类型有 List、Set、Quene、Map。这些对象类型被称之为集合。我们可以使用容器来称呼他们。容器都有自己的特殊的属性。例如，Set 只保存一个对象。Map 是将某些对象和其他一些对象关联起来的关联数组，Java 容器可以调整自己的尺寸。在变成时你可以将任意数量的对象放置到容器中。

## 泛型和类型安全的容器

在使用 JavaSE5 之前的容器允许我们向容器插入一些不正确的类型。例如我们用 ArayList 容器考虑传入一个 Apple 对象。但是我们又可以放入一些其他的对象。正常情况下编译器会报出警告的信息，因为这个示例没有添加泛型。在这里我们可以使用 JavaSE5 的一个注解来暂时抑制这些警告信息。``` @SuppressWarnings``` 这个注解表示只有有关 “不受检查的异常” 的警告信息应该被抑制。

Apple 类：
```java
public class Apple {

	private static long counter;
	private final  long ID = counter++;

	public long id() {
		return ID;
	}

}
```
Orange 类：
```java
public class Orange {

}

```
加入容器并调用：
```java
public class ApplesAndOrange {

	@SuppressWarnings("unchecked")
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		ArrayList list = new ArrayList<>();

		for (int i = 0; i < 3; i++) {
			list.add(new Apple());
			list.add(new Orange());
		}

		for (int i = 0; i < list.size(); i++) {
			((Apple) list.get(i)).id();
		}
	}

}
```
执行结果：
```
Exception in thread "main" java.lang.ClassCastException: collection.Orange cannot be cast to collection.Apple
	at collection.ApplesAndOrange.main(ApplesAndOrange.java:18)

```
我们在添加进容器的没有问题，因为他们都继承了 Object 类。容器默认就放入 Object 类型。但是当我们用 get() 方法取出你认为是 Apple 对象的时候，你得到的只是 Object 的引用。我们需要强制的类型转换才能使用 id() 这个方法。但是在运行的时候会报错。因为你将一个 Orange 对象转换为一个 Apple 对象。这两个对象是完全不相关的。

但是我们使用泛型，使用预定义的泛型通常会很简单的解决这个问题。想要声明保存 Appple 对象的 ArrayList 容器。你可以声明这样的样式 ``` ArrayList<Apple>``` 。尖括号内保存的是类型参数，这样就指定了这个容器可以保存什么样的类型。就可以在编译器防止将错误的类型放入容器中。

```java
public class ApplesAndOrange {


	public static void main(String[] args) {
		// TODO Auto-generated method stub
		ArrayList<Apple> list = new ArrayList<Apple>();

		for (int i = 0; i < 3; i++) {
			list.add(new Apple());
			//list.add(new Orange());
		}

		for (int i = 0; i < list.size(); i++) {
			System.out.println((list.get(i)).id());
		}
	}

}

```
此时就没有警告了，也不需要类型转换了。因为此时编译器将阻止你将 Orange 对象放入容器。而且在 get() 取出的时候也会替你执行转换。

当我们指定了容器的参数类型的时候，也不是一定要放入明确的类型才可以，我们也可以将参数类型的子类通过向上转型放入容器中

继承 Apple
```java
public class Orange extends Apple{

}
```
重新执行上面的代码：
```java
public class ApplesAndOrange {


	public static void main(String[] args) {
		// TODO Auto-generated method stub
		ArrayList<Apple> list = new ArrayList<Apple>();

		for (int i = 0; i < 3; i++) {
			list.add(new Apple());
			list.add(new Orange());
		}

		for (int i = 0; i < list.size(); i++) {
			System.out.println((list.get(i)).id());
		}
	}

}
```
执行结果：
```
012345
```
此时我们看到 Orange 又可以放进去了。

## 基本概念
Java 容器类的用途是用来保存对象，并且分为两个不同的类型。

- Collection 一个独立元素的序列，这些元素都服从一条或者多条规则。List 必须按照插入的顺序保存值，而 Set 不能有重复的元素。Queue 按照排队规则来确定对象产生的顺序。
- Map 一组成对的键值对对象，允许你使用键来查找值。ArrarList 允许你使用数组来查找值，它将数字和对象关联在一起。映射表允许我们使用一个对象来查找另一个对象，被称之为关联数组。或者被称之为字典。因为你可以通过键对象来查找值对象。就像在字典中使用单词来定义一样。

大部分的编程情况下我们都是与这些接口打交道。唯一需要制定所使用的精确类型的时候就是在创建的时候。

```
List<Apple> apples = new ArrayList<Apple>();
```
ArrayList 已经被向上转型为 List。使用接口的目的是如果你决定去修改你的实现，你所需的只是在创建的时候修改他。因此你应该创建一个具体类的对象，将其转型为对象的接口。接口的实现类都具有不同的功能，如果你需要使用这些功能，就不能向上转型为通用的接口类型。

下面的例子展示了 Collection 存放一组 Integer 对象的例子。

```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		Collection<Integer> cIntegers = new ArrayList<Integer>();
		for (int i = 0; i < 10; i++) {
			cIntegers.add(i);
		}

		for (Integer integer : cIntegers) {
			System.out.print(integer);
		}
	}
```
执行结果：
```
0123456789
```
这里我们只是用了 Collection 的 add 方法。因此任何继承自 Collection 类的对象都可以使用 add 的方法将对象存放进来。但是 Set 也是继承自 Collection。而且 Set 不允许有重复的元素。所以我们要确保，Collection 应该包含所指定的元素。

## 添加一组元素
介绍一下 Arrays 和 Collections 类。Arrays.asList() 方法接受一个数组或是用逗号隔开的元素列表并将其转化为一个 List 对象。Collections.addAll() 方法接受一个他需要的 Collection 对象，以及一个数组或用逗号隔开的元素列表。并且将后边的元素添加到第一个 Collection 参数中。其实是对第一个参数 Collection 的初始化。

第一种初始化方式：
```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		Collection<Integer> cIntegers = new ArrayList<Integer>(Arrays.asList(1,2,3,4,5));
		for (Integer integer : cIntegers) {
			System.out.print(integer);
		}
	}
```
第二种初始化方式：
```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		Collection<Integer> cIntegers = new ArrayList<Integer>();
		Collections.addAll(cIntegers, 1,2,3,4,5);
		for (Integer integer : cIntegers) {
			System.out.print(integer);
		}
	}
```
也可以添加一个数组：
```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		Collection<Integer> cIntegers = new ArrayList<Integer>();
		Integer[] integers = {1,2,3,4,5};
		Collections.addAll(cIntegers,integers);
		for (Integer integer : cIntegers) {
			System.out.print(integer);
		}
	}
```
容器自己添加一个数组：
```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		Collection<Integer> cIntegers = new ArrayList<Integer>();
		Integer[] integers = {1,2,3,4,5};
		cIntegers.addAll(Arrays.asList(integers));
		//Collections.addAll(cIntegers,integers);
		for (Integer integer : cIntegers) {
			System.out.print(integer);
		}
	}
```
我们通过 Arraye.asList() 来创建一个 List：
```java
List<Integer> list = Arrays.asList(1,2,3,4,5);
		list.add(6);
		list.set(1,99);
		for (Integer integer : list) {
			System.out.println(integer);
		}
```
执行结果：
```
Exception in thread "main" java.lang.UnsupportedOperationException
	at java.util.AbstractList.add(AbstractList.java:148)
	at java.util.AbstractList.add(AbstractList.java:108)
	at collection.ApplesAndOrange.main(ApplesAndOrange.java:24)
```
得到的确实报错：不支持这样的操作，不支持的是哪样的操作呢？原来我们通过 Arrays.asList() 得到的 List 其底层操作其实是一个数组。我们不能够调整他的尺寸。所以我们使用 add() 方法是不对的。但是我们可以通过 set() 方法替换值。

## 容器的打印
容器可以直接被打印：
```java
public class PrintContaines {

	static Collection fill(Collection<String> collection){
		collection.add("aaa");
		collection.add("bbb");
		collection.add("ccc");
		collection.add("bbb");
		return collection;
	}

	static Map fill(Map<String,String> map){
		map.put("aaa", "哈哈");
		map.put("bbb", "等等");
		map.put("ccc", "洋洋");
		map.put("bbb", "小小");
		return map;

	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		System.out.println(fill(new ArrayList<String>()));
		System.out.println(fill(new LinkedList<String>()));
		System.out.println(fill(new HashSet<String>()));
		System.out.println(fill(new TreeSet<String>()));
		System.out.println(fill(new LinkedHashSet<String>()));
		System.out.println(fill(new HashMap<String,String>()));
		System.out.println(fill(new TreeMap<String,String>()));
		System.out.println(fill(new LinkedHashMap<String,String>()));

	}

}

```
执行结果：
```
[aaa, bbb, ccc, bbb]
[aaa, bbb, ccc, bbb]
[aaa, ccc, bbb]
[aaa, bbb, ccc]
[aaa, bbb, ccc]
{aaa=哈哈, ccc=洋洋, bbb=小小}
{aaa=哈哈, bbb=小小, ccc=洋洋}
{aaa=哈哈, bbb=小小, ccc=洋洋}
```
通过查看结果我们来看一下他们的区别和共性：
- Collection
 - ArrarList 和 LinkedList 都是 List 类型，他们都是按照被插入的顺序保存元素。两者不同之处在于执行某些操作的性能。而且 LinkedList 包含的操作更多。
 - HashSet、TreeSet、LinkedHashSet 都是 Set 类型。每个相同的元素只保存一次。HashSet 保存的方式最复杂，但是这种技术的获取速度是最快的。TreeSet 将按照比较结果的升序保存对象。LinkedHashSet 将按照保存的顺序保存对象。
- Map
 - HashMap 使用了一种非常快的算法来控制顺序，并不是按照存储顺序排列的。也提供了最快的查找速度。
 - TreeMap 按照比较结果的升序来保存键。
 - LinkedHashMap 按照插入顺序保存键，同时提供了 HashMap 的查询速度。

## List
List 承诺将元素维护在特定的序列中。List 接口在 Collection 的基础上增加了大量的方法。可以使得我们在 List 之间插入和移除元素。

有两种类型的 List:

- ArrayList 随机访问性能高，但是插入和移除数据速度慢
- LinkedList 随机访问慢，他通过较低的代码在元素之间插入和删除操作。

```java
public class ListFeature {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Random random = new Random(47);
		List<Apple> list = new ArrayList<>(Arrays.asList(new Apple()));
		System.out.println("1"+list);

		Orange orange = new Orange();
		list.add(orange);
		System.out.println("2"+list);
		//查找集合中是否包含这个元素
		System.out.println("3"+list.contains(orange));
		//删除 orange元素
		list.remove(orange);
		Apple apple = new Apple();
		list.add(apple);
		list.add(orange);
		//查看apple元素的索引位置
		System.out.println("4"+list.indexOf(apple));
		//获取List的索引1和2的数据组成新的集合
		List<Apple> sub = list.subList(1,3);
		System.out.println("5"+sub);
		//判断旧集合是否包含新的集合
		System.out.println("6"+list.containsAll(sub));
		//排序
		Collections.sort(sub);
		System.out.println("排序"+sub);
		//排序之后仍然是true
		System.out.println("7"+list.containsAll(sub));
		Banner banner = new Banner();
		sub.add(banner);
		//打乱顺序
		Collections.shuffle(sub,random);
		System.out.println("8"+sub);
		//拷贝上一个集合
		List<Apple> list2 = new ArrayList<Apple>(list);
		sub = Arrays.asList(list.get(1),list.get(2));
		System.out.println("9"+sub);
		System.out.println("10"+list2);
		list2 = new ArrayList<>(list);
		System.out.println("11"+list2);
		list2.remove(2);
		System.out.println("12"+list2);
		list2.removeAll(sub);
		System.out.println("13"+list2);
		list2.set(0, new Banner());
		System.out.println("14"+list2);
		list2.addAll(2,sub);
		System.out.println("15"+list2);
		System.out.println("16"+list.isEmpty());
		list.clear();
		System.out.println("17"+list);


	}

}

```
执行结果：
```
1[collection.Apple@6d06d69c]
2[collection.Apple@6d06d69c, collection.Orange@7852e922]
3true
41
5[collection.Apple@4e25154f, collection.Orange@7852e922]
6true
排序[collection.Apple@4e25154f, collection.Orange@7852e922]
7true
8[collection.Orange@7852e922, collection.Apple@4e25154f, collection.Banner@70dea4e]
9[collection.Orange@7852e922, collection.Apple@4e25154f]
10[collection.Apple@6d06d69c, collection.Orange@7852e922, collection.Apple@4e25154f, collection.Banner@70dea4e]
11[collection.Apple@6d06d69c, collection.Orange@7852e922, collection.Apple@4e25154f, collection.Banner@70dea4e]
12[collection.Apple@6d06d69c, collection.Orange@7852e922, collection.Banner@70dea4e]
13[collection.Apple@6d06d69c, collection.Banner@70dea4e]
14[collection.Banner@5c647e05, collection.Banner@70dea4e]
15[collection.Banner@5c647e05, collection.Banner@70dea4e, collection.Orange@7852e922, collection.Apple@4e25154f]
16false
17[]

```
## 迭代器
迭代器是一种设计模式。迭代器是一个对象类，他的工作是遍历并选择序列中的对象，而程序员不必知道该序列底层的结构。迭代器通常被称为轻量级对象，创建它的代价很小。Java 中的迭代器只能单向移动。

- 使用方法 iterator() 要求容器返回一个 Iterator。Iterator 将准备好返回序列的第一个元素。
- 使用 next() 获得序列中下一个元素
- 使用 hasNext() 检查序列中是否还有元素
- 使用 remove() 将迭代器新近返回的元素删除

```java
public class SimpleIteration {

	static List<Integer> list = new ArrayList<>(Arrays.asList(1,2,3,4,5));
	static Iterator<Integer> iterator = list.iterator();
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		/*while (iterator.hasNext()) {
			System.out.print(iterator.next());
		}*/

		System.out.println("-----------");
		for (int i = 0; i < 5; i++) {
			if (iterator.hasNext()) {
				iterator.next();
				iterator.remove();
			}

		}

		System.out.print(list);
	}

}

```
#### ListIterator
ListIterator 是一个更加强大的 Iterator 的子类。他只能用于 List 类的访问。但是他可以双向移动。他还可以产生前一个和后一个的索引。并且可以使用 set() 方法替换他访问过的最后一个元素。

```java
public class SimpleIteration {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		List<Integer> list = new ArrayList<>(Arrays.asList(1,2,3,4,5));
		ListIterator<Integer> iterator = list.listIterator(3);
	/*	while (iterator.hasNext()) {
			System.out.println(iterator.next()+"----"+iterator.nextIndex()+"-------"+iterator.previousIndex());
			iterator.set(10);
		}
		System.out.println(list);*/

		while (iterator.hasPrevious()) {
			System.out.println(iterator.previous().intValue());
		}
	}

}
```
## LinkedList
LinkedList 也是实现了 List 接口，但是他执行插入和移除时更高效，随机访问稍微慢一点。它添加了很多独特的方法.由于方法很多我们可以写一下代码去观察一下这些方法都是什么用处。

```java
public class SimpleIteration {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		LinkedList<Integer> list = new LinkedList<>(Arrays.asList(1,2,3,4,5,6,7,8,9,10));
		System.out.println(list);
		System.out.println(list.getFirst());
		System.out.println(list.element());
		System.out.println(list.peek());
		System.out.println(list.remove());
		System.out.println(list.removeFirst());
		System.out.println(list.poll());
		list.addFirst(11);
		System.out.println(list);
		list.offer(12);
		System.out.println(list);
		list.add(13);
		System.out.println(list);
		list.addLast(14);
		System.out.println(list);
		list.removeLast();
		System.out.println(list);

	}

}

```
执行结果：
```
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
1
1
1
1
2
3
[11, 4, 5, 6, 7, 8, 9, 10]
[11, 4, 5, 6, 7, 8, 9, 10, 12]
[11, 4, 5, 6, 7, 8, 9, 10, 12, 13]
[11, 4, 5, 6, 7, 8, 9, 10, 12, 13, 14]
[11, 4, 5, 6, 7, 8, 9, 10, 12, 13]

```
## Stack
栈通常是指先进后出的容器。有时候栈也被称之为叠加栈，因为最后压入栈的元素最先弹出栈。

LinkedList 具有实现栈的功能的所有方法，因此可以直接将 LinkedList 作为栈使用。使用到了泛型个概念。类名之后的 T 告诉编译器这将是一个参数化类型，而其中的类型参数，在类中被使用时将被替换为实际的类型参数。
```java
public class Stack<T> {
	LinkedList<T> linkedList = new LinkedList<T>();
	public void push(T v) {
		linkedList.addFirst(v);
	}

	//返回容器的第一个元素，容器空报null
	public T peek() {
		return linkedList.peek();
	}

	//删除并返回容器的头部，如果容器是空报null
	public T pop() {
		return linkedList.poll();

	}

	public boolean empty() {
		return linkedList.isEmpty();

	}

	public String  toString() {

		return linkedList.toString();
	}
}

```
使用我们自己创建的栈：
```java
public class StackTest {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Stack<String> stack = new Stack<>();
		for (String string : new String[]{"a","b","c","d","e","f","g"}) {
			stack.push(string);
		}

		while (!stack.empty()) {
			System.out.println(stack.pop());
		}
	}

}

```
执行的结果：
```
gfedcba
```
## Set
Set 不保存重复的元素。如果你试图将相同对象的多个实例添加到 Set 中，那么他会阻止这种重复现象。Set 最长用来测试归属性，可以很容易的询问某个对象是否在某个 Set 中。所以，查找成了 Set 中最重要的操作。通常我们会选择一个 HashSet 来实现。

Set 与 Collection 具有完全一样的接口，没有任何额外的功能。只是行为不同而已。

```java
public static void main(String[] args) {
		Random random = new Random(47);
		java.util.Set<Integer> integers = new HashSet<>();
		for (int i = 0; i < 1000; i++) {
			integers.add(random.nextInt(10));
		}
		System.out.println(integers);
	}
```
执行结果：
```
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```
我们循环 1000 次，但是你看到只有 10 个实例存在于容器中。这是因为 Set 将不会存储重复的数据。HashSet 是用散列的方式存储的数据，由于速度的原因我们看到了顺序排列，其实很多时候这些数据不是按照顺序排列的。TreeSet 不是散列排序。TreeSet 使用的是红黑树的数据结构。LinkedHashSet 也是用了散列但是同时使用了链表来维护插入速度。

我们使用 TreeSet 对数据排序：
```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		SortedSet<Integer> set = new TreeSet<>();
		Random random = new Random(47);
		for (int i = 0; i < 1000; i++) {
			set.add(random.nextInt(10));
		}
		System.out.println(set);
	}
```
执行结果：
```
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```
还有一种最常见的操作，就是使用 contains() 测试 Set 的归属性。
```java
public class TreeSetTest {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		SortedSet<Integer> set = new TreeSet<>();
		java.util.Set<Integer> integers = new HashSet<>();
		Random random = new Random(47);
		for (int i = 0; i < 1000; i++) {
			integers.add(random.nextInt(20));
		}
		System.out.println(integers);
		for (int i = 0; i < 1000; i++) {
			set.add(random.nextInt(10));
		}
		System.out.println(set);

		System.out.println(integers.contains(10));
		System.out.println(integers.containsAll(set));

	}

}

```
执行结果：
```
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19]
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
true
true
```
