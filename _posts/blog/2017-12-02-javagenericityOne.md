---
layout: post
title: java编程思想之泛型一 (初解)
categories: javaThinking
description: java编程思想之泛型
keywords: java, java泛型, java泛型详解，泛型详解
---

一般的类和方法，只能使用具体的类型：要嘛是基本类型，要嘛是自定义类型。如果要编写可以应用与多种类型的代码，这种刻板的限制对代码的束缚就会很大。

面向对象的语言设计中，多态算是一种泛化机制。例如，你可以将方法的参数类型设为基类，那么该方法就可以接受从这个基类导出的任何类作为参数。有时候，拘泥于单继承的体系，也会使得程序受限太多。如果方法的参数是一个接口，这种限制就会减少很多。因为任何实现了该接口的类都可以应用与该方法，这也包括暂时不存在的类。可是有的时候即便是使用了接口，对程序的约束性还是太强。因为一旦指明了接口，他就要求你的代码必须实现特定的接口。而我们希望达到的目的是编写更加通用的代码，要使得代码应用于更加非具体的类型。

javaSE5 的重大变化之一：泛型的概念被提出。泛型实现了参数化类型的概念，使得代码可以应用于多种类型。泛型的意思是:可以适应许多许多的类型。泛型最初的目的是希望类或方法能够具备更广泛的表达能力。正是通过解耦类或者方法所使用的的类型之间的约束达到。在你创建一个参数化类型的一个实例时，编译器会为你负责转型的操作，并且保证类型的正确性。这次我们将介绍到 Java 泛型的优缺点和局限性，希望可以更加有效的使用这个功能。

## 简单泛型
有许多原因促成了泛型的出现，最重要的一个原因就是容器的设计。容器就是存放需要使用对象的地方。数组也是如此，不过与简单的数组相比，容器类更加的灵活，具备更多的不同的功能。实际上，所有的程序在运行时都要求你持有一大堆的对象，容器类算是最具重要性的类库。

我们先来看一个持有单个对象的类。

```java
public class Automobile {

}

public class Holder1 {

	private Automobile automobile;

	public Holder1(Automobile automobile) {
		// TODO Auto-generated constructor stub
		this.automobile = automobile;
	}

	Automobile get(){
		return automobile;
	}
}

```
这个类的可复用性就不好了，他无法持有其他类型的任何对象。在 JavaSE5 之前我们可以让这个类直接持有 Object 类的对象：

```java
public class Holder2 {

	private Object aObject;

	public Holder2(Object aObject) {
		// TODO Auto-generated constructor stub
		this.aObject = aObject;
	}

	public Object getaObject() {
		return aObject;
	}

	public void setaObject(Object aObject) {
		this.aObject = aObject;
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Holder2 holder2 = new Holder2(new Automobile());
		Automobile automobile = (Automobile) holder2.getaObject();
		holder2.setaObject("abc");
		String string = (String)holder2.getaObject();
		holder2.setaObject(string);
	}

}

```
在这种情况下我们可以存储任何类型的对象。

但是，通常而言，我们只会使用容器来存储一种类型的对象。泛型的主要目的之一就是来指定容器持有什么类型的对象。而且用编译器来保证类型的正确性。要达到这个目的，需要使用类型参数，用尖括号括起来，放在类名的后面。然后在使用这个类的时候，在用实际的类型替换这个类型参数。

```java
public class Holder3<T> {

	private T aT;
	public Holder3(T aT) {
		// TODO Auto-generated constructor stub
		this.aT = aT;
	}


	public T getaT() {
		return aT;
	}

	public void setaT(T aT) {
		this.aT = aT;
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
			Holder3<Automobile> holder3 = new Holder3<Automobile>(new Automobile());


	}

}

```
现在我们创建 Holder3 对象时，必须指明持有什么类型的参数，将其置于尖括号内。

#### 一个元组类库
仅一次方法调用就能返回多个对象，你应该经常需要这个功能。可是 return 语句只允许返回单个对象。可是有了泛型我们就能一次性的解决这个问题。这个概念称为元组，它是将一组对象直接打包存储于其中的一个单一对象。这个容器对象允许读取其中的元素，但是不允许向其存放新的对象。通常，元组具有可变的长度，同时，元组中的对象可以是任意不同的类型。不过我们希望能够为每一个对象指明其类型。并且从容器中读取出来的时候也能够保证其类型的正确性。

下面是一个二维的元组，他能够持有两个不同的对象。

```java
public class TwoTuple<A,B> {

	public final A a;
	public final B b;
	public TwoTuple(A a,B b) {
		// TODO Auto-generated constructor stub
		this.a = a;
		this.b = b;
	}

	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return a+"-"+b;
	}

}

```
构造器捕获要存储的对象。我们使用 public final 可以保证客户端程序要可以使用这两个对象，但是不能对其赋值。如果你要改变这两个对象的引用。那么你其可以创建一个新的 TwoTuple 对象。

我们可以利用继承机制实现长度更长的元组。

```java
public class ThreeTuple<A,B,C> extends TwoTuple<A, B>{

	public final C c;

	public ThreeTuple(A a, B b,C c) {
		super(a, b);
		// TODO Auto-generated constructor stub
		this.c = c;
	}

	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return a +"" + b+ "" +c;
	}


}

```
为了使用元组，你只需要定义一个长度适合的元组，将其作为方法的返回值，然后在 return 语句中创建改元组即可。

```java
public class TupleTest {
	static TwoTuple<String, Integer> f(){
		return new TwoTuple<String, Integer>("chao",47);
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		TwoTuple<String, Integer> tuple = f();
		System.out.println(tuple);
	}

}
```
执行结果：
```
chao-47
```
由于有了泛型你可以很容易的创建元组，令其返回一组任意类型的对象。而你所做的只是需要编写表达式而已。

#### 一个堆栈类
在之前的章节中我们使用了 LinkedList 来实现一个堆栈，现在我们不用 LinkedList，来实现自己的内部链式存储机制。

```java
public class LinkedStack<T> {

	private static class Node<U>{
		U item;
		Node<U> next;

		public Node() {
			// TODO Auto-generated constructor stub
			item = null;
			next = null;
		}

		protected Node(U item, Node<U> next) {
			super();
			this.item = item;
			this.next = next;
		}

		boolean end(){
			return item == null && next == null;
		}
	}

	private Node<T> top = new Node<T>();

	public void push(T item) {
		top = new Node<T>(item, top);
	}

	public T pop() {
		T result = top.item;
		if (!top.end()) {
			top = top.next;
		}
		return result;
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		LinkedStack<String> lStack = new LinkedStack<>();
		for (String string : "wo men shi zhong guo ren".split("")) {
			lStack.push(string);
		}
		String string;
		while ((string = lStack.pop())!= null) {
			System.out.print(string);
		}
	}

}

```
执行结果：
```
ner oug gnohz ihs nem ow
```
这个例子使用了末端哨兵来判断堆栈何时为空。这个末端哨兵在构造 LinkedStack 的时候创建。然后每次调用一次 push() 方法，就会创建一个 ```Node<T>``` 的对象，并将其连接到前一个对象。当你调用 pop() 方法时，总会返回 top.item。然后对其当前 top 所指的 ```Node<T>``` 。并将 top 转移到下一个 ```Node<T>```。除非你碰到末端哨兵。这个时候就不在移动了。如果到了末端客户端程序还会调用 pop() 方法。它只能得到 null，说明堆栈已经是空。

#### RandomList
假设我们需要一个持有特定类型对象的列表，每次调用其上的 select() 方法，他可以随机的选取一个元素。如果我们需要构建的是应用于各种类型的工具，就需要泛型。

```java
public class RandomList<T> {

	private ArrayList<T> stor = new ArrayList<>();
	private Random random = new Random(47);

	public void add(T item) {
		stor.add(item);
	}

	public T select() {
		return stor.get(random.nextInt(stor.size()));
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		RandomList<String> sList = new RandomList<>();
		for (String string : "wo men shi zhong guo ren".split("")) {
			sList.add(string);
		}

		for (int i = 0; i < 10; i++) {
			System.out.print(sList.select());
		}
	}

}

```
执行结果：
```
 zononehus
```
## 泛型接口
泛型也可以应用于接口。例如，我们利用此来做一个生成器，用于生成不同的对象。这是工厂方法设计模式的一种应用。一般而言一个生成器只定义一个方法，该方法用于生成新的对象。在这里我们使用的是 next()。

接口：
```java
public interface Generator <T>{
	T next();
}

```
我们还需要一些层次的类：

基类：
```java
public class Coffer {

	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return getClass().getSimpleName();
	}

}
```
子类：
```java
public class ACoffer extends Coffer{

}

public class BCoffer extends Coffer{

}

```
最后一步我们实现这个接口：他能够随机生成不同类型的 Cofer 对象
```java
public class CofferGenerator implements Generator<Coffer> ,Iterable<Coffer>{

	private Class[] type = {ACoffer.class,BCoffer.class};
	private Random random = new Random(47);
	private int size = 0;

	protected CofferGenerator(int size) {
		super();
		this.size = size;
	}

	protected CofferGenerator() {
		super();
		// TODO Auto-generated constructor stub
	}

	public Coffer next() {
		try {
			return (Coffer) type[random.nextInt(type.length)].newInstance();
		} catch  (Exception e) {
			// TODO Auto-generated catch block
			throw new RuntimeException();
		}
	}

	class CofferIterator implements Iterator<Coffer>{
		int count = size;


		@Override
		public boolean hasNext() {
			// TODO Auto-generated method stub
			return count > 0;
		}


		@Override
		public Coffer next() {
			// TODO Auto-generated method stub
			count --;
			return CofferGenerator.this.next();
		}

	}

	@Override
	public Iterator<Coffer> iterator() {
		// TODO Auto-generated method stub
		return  new CofferIterator();
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		CofferGenerator generator = new CofferGenerator();
		for (int i = 0; i <2; i++) {
			System.out.println(generator.next());
		}

    for (Coffer coffer : new CofferGenerator(5)) {
			System.out.println(coffer);
		}

	}


}

```
参数化的 Generator 接口确保 next() 的返回值是参数的类型。CofferIterator 同时还实现了 Iterable 接口，所以他可以在循环语句中使用。不过他还需要一个末端哨兵来判断何时停止。这正是第二个构造器的功能。

下面我们来看另外一种实现的方式：生成一个 Fibonacci 数列

```java
public class Fibonacci implements Generator<Integer>{

	private int count =0;

	public int fib(int n) {
		if (n<2) {
			return 1;
		}else {
			return fib(n-2) + fib(n-1);
		}
	}

	@Override
	public Integer next() {
		// TODO Auto-generated method stub
		return fib(count++);
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Fibonacci gFibonacci = new Fibonacci();
		for (int i = 0; i < 18; i++) {
			System.out.print(gFibonacci.next()+" ");
		}
	}

}

```
我们看到我们的类里边都是 int 类型，但是其参数确实 Integer 类型。这里引出了泛型的局限性。基本类型无法作为其参数类型。不过我们有自动打包和自动拆包的功能。可以很方便的进行包装类和基本类之间的转换。

下面我们更进一步，编写一个实现了 Iterable 的 Fibonacci 生成器。我们需要创建一个适配器来实现所有的接口。

```java
public class IterableFiboncci extends Fibonacci implements Iterable<Integer>{


	private int n;


	protected IterableFiboncci(int n) {
		super();
		this.n = n;
	}


	@Override
	public Iterator<Integer> iterator() {
		// TODO Auto-generated method stub
		return new Iterator<Integer>() {

			@Override
			public boolean hasNext() {
				// TODO Auto-generated method stub
				return n>0;
			}

			@Override
			public Integer next() {
				// TODO Auto-generated method stub
				n--;
				return IterableFiboncci.this.next();
			}

		};
	}


	public static void main(String[] args) {
		// TODO Auto-generated method stub
		for (int i : new IterableFiboncci(18)) {
			System.out.print(i+" ");
		}
	}


}

```
执行结果：
```
1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987 1597 2584
```

## 泛型方法
我们同样可以在类中包含泛型方法，而这个方法所在的类可以是泛型类，也可以不是泛型类。也就是说是否拥有泛型方法，与其是否所在泛型类没有关系。泛型方法使得方法能够独立于类而产生变化。基本原则：无论何时，只要你能做到就应该尽量使用泛型方法。对于一个 static 的方法而言，无法访问泛型类的类型参数，所以，如果是静态的方法想要实现参数化能力，就必须提供成为泛型方法。

要定义泛型方法，只需要将泛型参数列表置于返回值之前：

```java
public class GenericMethods {

	public <T> void f(T x) {
		System.out.println(x.getClass().getName());
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		GenericMethods gMethods = new GenericMethods();
		gMethods.f("");
		gMethods.f(1);
		gMethods.f(1.0);
		gMethods.f('f');
	}

}

```
执行结果：
```
java.lang.String
java.lang.Integer
java.lang.Double
java.lang.Character
```

#### 杠杆利用类型参数推断
使用泛型有时候我们需要向程序中加入更多的代码。

```java
Map<Person,List<? extends Pet>> pet = new HashMap<Person,List<? extends Pet>>();
```
我们在重复自己做过的事情，编译器本应该从泛型参数列表中的一个参数推断出另外的参数。可惜还做不到。但是在泛型方法中类型推断却可以做到。我们看下面的类：

```java
public class New {

	public static <K,V> Map<K, V> map() {
		return new HashMap<K, V>();
	}

	public static <T> List<T> list() {
		return new ArrayList<>();
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Map<String, Integer> map = New.map();
		List<String> list = New.list();
	}

}
```
类型推断只对赋值操作有效，其他时候并不起作用。如果你将一个泛型方法调用的结果作为参数，传递给另一个方法，这时编译器并不会执行类型推断。这种情况，编译器认为：调用泛型方法之后，其返回值赋予了一个 Object 类型的变量。不过我们可以进行显示的类型说明来解决这个问题。显示的类型说明必须在 . 操作符和方法名之间插入尖括号，然后把类型置于尖括号内。

```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		Map<String, Integer> map = New.map();
		List<String> list = New.list();

		//显示的类型说明
		f(New.<String, Integer> map());
	}
```
这种语法抵消了 New 给我们带来的好处，不过，只有在编写非赋值语句时，我们才需要额外的说明。

#### 可变参数与泛型方法
泛型方法与可变参数列表能够很好的共同相处：

```java
public class GenericVarargs {

	public static <T> List<T> makeList(T...  arg) {
		List<T> result = new ArrayList<>();
		for (T t : arg) {
			result.add(t);
		}

		return result;
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		List<String> strings = makeList("A","B","C");
		System.out.println(strings);
	}

}

```
执行结果：
```
[A, B, C]
```
#### 用于 Generator 的泛型方法
我们使用泛型化来做一个示例：

```java
public class Generatons {

	public static <T> Collection<T> fill( Collection<T> coll,Generator<T> generator,int n) {
		for (int i = 0; i <n; i++) {
			coll.add(generator.next());
		}

		return coll;
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Collection<Coffer> coffers = fill(new ArrayList<Coffer>(), new CofferGenerator(), 4);
		for (Coffer coffer : coffers) {
			System.out.print(coffer+" ");
		}
	}

}

```
执行结果：
```
BCoffer ACoffer BCoffer ACoffer
```

#### 简化元组的使用
有了类型参数推断，再加上 static 方法，我们可以重新编写之前看到的元组工具，使其成为更通用的类库。

```java
public class Tuple {

	public static <A,B> TwoTuple<A, B> tuple(A a,B b) {
		return new TwoTuple<A, B>(a, b);
	}

	public static <A,B,C> ThreeTuple<A, B,C> tuple(A a,B b,C c) {
		return new ThreeTuple<A, B,C>(a, b,c);
	}

}

```
我们来看一下修改后的 TupleTest 类：

```java
public class TupleTest {
	static TwoTuple<String, Integer> f(){
		return Tuple.tuple("chao",47);
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		TwoTuple<String, Integer> tuple = f();
		System.out.println(tuple);
	}

}
```
执行结果：
```
chao-47
```
一样的执行结果，当然你也可以添加更多的方法返回类型。都是通用的。

#### 一个 Set 实用工具
作为泛型的另外一个示例，我们看看如何使用 Set 来表达数学中的关系。

```java
public class Sets {

	//将两个参数合并在一起
	public static <T> Set<T> union(Set<T> a,Set<T> b) {
		Set<T> result = new HashSet<>();
		result.addAll(b);
		return result;
	}

	//返回只包含两个参数共有的部分
	public static <T> Set<T> intersection(Set<T> a,Set<T> b) {
		Set<T> result = new HashSet<>();
		result.retainAll(b);
		return result;
	}

	//从前一个set中移除后一个的内容
	public static <T> Set<T> difference(Set<T> a,Set<T> b) {
		Set<T> result = new HashSet<>();
		result.removeAll(b);
		return result;
	}

	//除了交集之外所有的元素
	public static <T> Set<T> complement(Set<T> a,Set<T> b) {

		return difference(union(a, b) ,intersection(a, b));
	}

}

```
在前边三个方法中，都将第一个 Set 赋值了一份，将 Set 中的引用都存入一个新的 HashSet 中，因此，我们并未直接修改 Set。返回的是一个全新的 Set。我们来演示一下上面的功能。

首先创建一个枚举：

```java
public enum Watercolors {
  ZINC, LEMON_YELLOW, MEDIUM_YELLOW, DEEP_YELLOW, ORANGE,
  BRILLIANT_RED, CRIMSON, MAGENTA, ROSE_MADDER, VIOLET,
  CERULEAN_BLUE_HUE, PHTHALO_BLUE, ULTRAMARINE,
  COBALT_BLUE_HUE, PERMANENT_GREEN, VIRIDIAN_HUE,
  SAP_GREEN, YELLOW_OCHRE, BURNT_SIENNA, RAW_UMBER,
  BURNT_UMBER, PAYNES_GRAY, IVORY_BLACK
}
```

为了方便起见我们引入一个工具类，WatercolorSets。这个示例使用了 EnumSet，用来从 Enum 直接创建 Set。

```java
public class WatercolorSets {

	      static Set<Watercolors> set1 =
		      EnumSet.range(Watercolors.BRILLIANT_RED, Watercolors.VIRIDIAN_HUE);
		    static Set<Watercolors> set2 =
		      EnumSet.range(Watercolors.CERULEAN_BLUE_HUE, Watercolors.BURNT_UMBER);

		    public static void main(String[] args) {
		    	 System.out.println("set1: " + set1);
		    	 System.out.println("set2: " + set2);
		    	 System.out.println("union(set1, set2): " + Sets.union(set1, set2));
				 Set<Watercolors> subset = Sets.intersection(set1, set2);

				 System.out.println("intersection(set1, set2): " + subset);
				 System.out.println("difference(set1, subset): " +
				    		Sets.difference(set1, subset));
				 System.out.println("difference(set2, subset): " +
				    		Sets.difference(set2, subset));
				 System.out.println("complement(set1, set2): " +
				    		Sets.complement(set1, set2));
			}

}
```
执行结果：
```
set1: [BRILLIANT_RED, CRIMSON, MAGENTA, ROSE_MADDER, VIOLET, CERULEAN_BLUE_HUE, PHTHALO_BLUE, ULTRAMARINE, COBALT_BLUE_HUE, PERMANENT_GREEN, VIRIDIAN_HUE]
set2: [CERULEAN_BLUE_HUE, PHTHALO_BLUE, ULTRAMARINE, COBALT_BLUE_HUE, PERMANENT_GREEN, VIRIDIAN_HUE, SAP_GREEN, YELLOW_OCHRE, BURNT_SIENNA, RAW_UMBER, BURNT_UMBER]
union(set1, set2): [PHTHALO_BLUE, PERMANENT_GREEN, VIRIDIAN_HUE, COBALT_BLUE_HUE, BURNT_UMBER, BURNT_SIENNA, YELLOW_OCHRE, CERULEAN_BLUE_HUE, ULTRAMARINE, RAW_UMBER, SAP_GREEN]
intersection(set1, set2): []
difference(set1, subset): []
difference(set2, subset): []
complement(set1, set2): []
```
## 匿名内部类
泛型还可以用于内部类以及匿名内部类。

内部类1：

```java
public class Customer {

	private static long counter = 1;
	private final long id = counter++;

	private Customer() {
		// TODO Auto-generated constructor stub
	}

	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return "Customer" + id;
	}

	public static Generator<Customer> generator() {

		return new Generator<Customer>() {

			@Override
			public Customer next() {
				// TODO Auto-generated method stub
				return new Customer();
			}
		};
	}
}

```
内部类2：

```java
public class Teller {

	private static long counter = 1;
	private final long id = counter++;

	private Teller() {
		// TODO Auto-generated constructor stub
	}

	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return "Teller"+id;
	}

	public static Generator<Teller> generator = new Generator<Teller>() {

		@Override
		public Teller next() {
			// TODO Auto-generated method stub
			return new Teller();
		}
	};
}

```
调用并执行：
```java
public class BankTeller {

	public static void save(Teller teller,Customer customer) {
		System.out.println(teller + "  "+ customer);
	}
	public static void main(String[] args) {
		// TODO Auto-generated method stub
			Random  random = new Random(47);
			Queue<Customer> queue = new LinkedList<>();
			Generatons.fill(queue, Customer.generator(),15);
			List<Teller> list = new ArrayList<>();
			Generatons.fill(list, Teller.generator,4);
			for (Customer customer : queue) {
				save(list.get(random.nextInt(list.size())), customer);
			}
	}

}

```
执行结果：
```
Teller3  Customer1
Teller2  Customer2
Teller3  Customer3
Teller1  Customer4
Teller1  Customer5
Teller3  Customer6
Teller1  Customer7
Teller2  Customer8
Teller3  Customer9
Teller3  Customer10
Teller2  Customer11
Teller4  Customer12
Teller2  Customer13
Teller1  Customer14
Teller1  Customer15
```
Customer 和 Teller 类都只有 private 的构造器，这可以强制你必须使用 Generator 对象。Customer 有一个 generator() 方法，每次执行它都会生成一个新的 ```Generator<Customer>``` 对象。我们其实不需要多个 Generator 对象，Teller 就只创建了一个 public 的 generator 对象。
