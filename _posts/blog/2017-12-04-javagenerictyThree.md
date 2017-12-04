---
layout: post
title: java编程思想之泛型三（通配符）
categories: javaThinking
description: java编程思想之泛型
keywords: java, java泛型, java泛型详解，泛型详解
---

本节将深入讨论一下通配符的问题：在泛型表达式中的问号。

## 通配符
我们看下边这个例子：我们将子类型的数组赋予基类型的数组引用。
```java
public class CovarianArrays {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Fruit[] fruits  = new Apple[10];
		fruits[0] = new Apple();
		fruits[1] = new Jonathan();
		fruits[0] = new Fulter();
		fruits[1] = new Orange();
	}

}
```
上面的代码有两处运行时错误之处：
```java
fruits[0] = new Fulter();
fruits[1] = new Orange();
```
错误：
```
Exception in thread "main" java.lang.Error: Unresolved compilation problem:
	Type mismatch: cannot convert from Fulter to Fruit

	at wildcard.CovarianArrays.main(CovarianArrays.java:12)

Exception in thread "main" java.lang.ArrayStoreException: wildcard.Orange
	at wildcard.CovarianArrays.main(CovarianArrays.java:13)
```
我们在第一行创建了一个 Apple[] 数组，并将其赋值给了 Fruit[] 数组引用。这是可以的，因为 Apple 是 Fruit 的子类。但是赋值之后实际的数据类型就变成了 Apple[]。我们就只能在其中放入 Apple 或者 Apple 的子类，这在编译期和运行期都可以工作。但是 fruits 同样具有 Fruit 引用的。所以我们在编译期可以将 Fruit 以及 Fruit 的子类放入进去。但是在运行期间数组发现它处理的是 Apple[]。因此会向你抛出异构类型异常。

原因在于，你在这里做的并不是向上转型，实际是将一个数组赋值给另外一个数组而已。所以很明显，数组对象可以保留有关它们包含的对象类型的限制。对数组的这种赋值并不是很复杂，因为在运行时我们可以发现插入的类型是否正确。但是对于泛型而言，我们可以把这种类型的检查移入到编译期。

![](/images/blog/nonException.png)

我们看到增加泛型之后，编译器不允许我们将一个 List<Apple> 赋值与一个 List<Fruit>。我们看到虽然 List<Frui> 持有了所有的 Fruit 类型，但是这并不意味着它可以是 List<Apple>。因为它仍旧是 List<Fruit>。这两个在类型上并不等价。真正的问题在于我们比较的是容器的类型，而不是容器持有的类型。但是有时候我们想在两个类型之间建立某种类型的向上转型的关系，我们可以使用通配符做到：

![](/images/blog/addException.png)

我们看到 List<? extends fruit>，可以将其读作 “具有任何从 Fruit 继承的类型的列表 ” 。但是这并不意味着这个容器可以持有任何类型的 Fruit。因为通配符引用的是明确的类型。因此这个赋值的容器必须持有明确的类型。但是我们看到我们并不能向容器中添加任何类型的对象，甚至是它自己。因为我们并不知道它具体持有的是那种类型。因为编译器并不知道这个 List 可以合法的指向你传入的类型。一旦执行这种类型的向上转型，你就丢失掉向其传入任何对象的能力。另外，你看到如果调用一个返回 Fruit 的方法，则是安全的。因为我们知道这个容器的任何对象至少具有 Fruit 类型。

#### 编译器有多聪明
我们来看一下修改后的代码：
```java
public class NonCovar {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		List<? extends Fruit> list = Arrays.asList(new Apple());
		Apple apple = (Apple)list.get(0);
		list.contains(new Apple());
		list.indexOf(new Apple());
		System.out.println(list.contains(new Apple())+"");
		System.out.println(list.indexOf(new Apple())+"");
	}

}
```
执行结果：
```
false
-1

```
我们看到上面的两个方法可以正确的执行。这意味着编译器实际上进行了代码的检查。我们查看 ArrayList 的文档可以知道 contains() 和 indexOf() 实际传入的是 Object。而添加 Add() 方法实际传入的是泛型 T。add() 方法的参数就变成了 <? extends Fruit>。这里编译器就无法了解你到底传入的是那种具体类型，因此就不接受任何类型的 Fruit。

#### 逆变
还可以使用另外一个方式，那就是超类型通配符。这里，可以声明通配符是由某个特定类的任何基类来限定的，方法是指定 <? super MyClass>。甚至可以使用类型参数 <? super T>。
```java
public class SuperType {
	public static void  writeTo(List<? super Apple> apples) {
		apples.add(new Apple());
		apples.add(new Jonathan());
	}
}
```
List 中的泛型指定的是参数 Apple 的某种基类，这样就知道向其添加元素的下界就是 Apple。既然知道了下界，那么添加 Apple 以及 Apple 的子类是安全的，相反添加的基类是不能的，因为无法确定是那种基类的类型。我们可以参考上面的例子，来思考一下一个泛型类型如何写入和一个泛型类型如何读取：来思考一下子类型和超类型边界。

超类型边界放松了在可以向方法传递参数上的限制：

```java
public class GenericWriting {

	static <T> void writeExact(List<T> list,T item){
		list.add(item);
	}

	static <T> T writeExtendExact(List<? extends T> list,int x){
		return list.get(x);
	}

	static <T> void writeWithExact(List<? super T> list,T item){
		list.add(item);
	}

	static List<Apple> list = new ArrayList<>();
	static List<Fruit> list2 = new ArrayList<>();

	static void f(){
		writeExact(list, new Apple());
		//writeExact(list2, new Apple());// Error
	}

	static void f2(){
		writeWithExact(list, new Apple());
		writeWithExact(list2,new Apple());
	}

	static void f3(){
		writeExtendExact(list,0);
		writeExtendExact(list2,0);
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		f();
		f2();
		f3();
	}

}

```
上面的例子其实展示了这三种的关系。第一种方法使用的是确切类型 T。所以可以正确运行，但是不允许在 List<Fruit> 中添加一个 Apple。第二个方法使用的超类型边界，因此 List 将持有从 T 导出的类型。在第三个方法中我们看到其实就是上面我们讨论过的。它不允许向其添加任何类型的参数，但是可以正确的读取。

#### 无界通配符
无界通配符 <?> 看起来意味着任何事物，因此使用无界通配符好像等价于使用原生类型。事实上，编译器初看起来是支持这种判断的。其实是不等价的。
```java
public class UnboundedWildcards1 {

	  static List list1;
	  static List<?> list2;
	  static List<? extends Object> list3;

	  static void assign1(List list) {
	    list1 = list;
	    list2 = list;
	    list3 = list; // Warning: unchecked conversion
	    // Found: List, Required: List<? extends Object>
	  }
	  static void assign2(List<?> list) {
	    list1 = list;
	    list2 = list;
	    list3 = list;
	  }
	  static void assign3(List<? extends Object> list) {
	    list1 = list;
	    list2 = list;
	    list3 = list;
	  }
	  public static void main(String[] args) {
	    assign1(new ArrayList());
	    assign2(new ArrayList());
	    assign3(new ArrayList()); // Warning:
	    // Unchecked conversion. Found: ArrayList
	    // Required: List<? extends Object>
	    assign1(new ArrayList<String>());
	    assign2(new ArrayList<String>());
	    assign3(new ArrayList<String>());
	    // Both forms are acceptable as List<?>:
	    List<?> wildList = new ArrayList();
	    wildList = new ArrayList<String>();
	    assign1(wildList);
	    assign2(wildList);
	    assign3(wildList);
	  }
}

```
看这个例子，事实上编译器也没有给出任何异常。事实上编译器并不关心你使用的是原生类型还是 <?>。<?> 号可以被看成是一种装饰，但是还是非常有用的。因为实际上他是在生命我是想用 Java 的泛型来编写代码，我在这里并不是想用原生类型。在当前这种情况下泛型参数可以持有任何类型。

看下面这个例子：当我们处理多个泛型参数时，其中一个是任何类型，同事其他参数是某种特定的类型：

```java
public class UnboundedWildcards2 {

	  static Map map1;
	  static Map<?,?> map2;
	  static Map<String,?> map3;
	  static void assign1(Map map) { map1 = map; }
	  static void assign2(Map<?,?> map) { map2 = map; }
	  static void assign3(Map<String,?> map) { map3 = map; }
	  public static void main(String[] args) {
	    assign1(new HashMap());
	    assign2(new HashMap());
	    assign3(new HashMap()); // Warning:
	    // Unchecked conversion. Found: HashMap
	    // Required: Map<String,?>
	    assign1(new HashMap<String,Integer>());
	    assign2(new HashMap<String,Integer>());
	    assign3(new HashMap<String,Integer>());
	  }
}
```
上面的例子会有一些警告，但是运行没有任何问题。我们看到当我们拥有的全是无界通配符的时候，编译器看起来好像无法与原生的 Map 进行区分。因此看起来好像是相同的事物。事实上，因为泛型参数将擦除到它的第一个边界，因此 List<?> 看起来等价于 List<Object>。而实际上 List 也是 List<Object>。而事实是：List 表示持有任何 Object 类型的原生 List。List<?> 表示具有某种特定类型的非原生 List，只是我们不知道它是什么类型。

编译器何时才会关注原生类型和涉及无界通配符的类型之间的差异呢？

```java
public class Holder<T> {
	  private T value;
	  public Holder() {}
	  public Holder(T val) { value = val; }

	  public void set(T val) { value = val; }

	  public T get() { return value; }

	  public boolean equals(Object obj) {
	    return value.equals(obj);
	  }

	  public static void main(String[] args) {
	    Holder<Apple> Apple = new Holder<Apple>(new Apple());
	    Apple d = Apple.get();
	    Apple.set(d);
	    // Holder<Fruit> Fruit = Apple; // Cannot upcast
	    Holder<? extends Fruit> fruit = Apple; // OK
	    Fruit p = fruit.get();
	    d = (Apple)fruit.get(); // Returns 'Object'
	    try {
	      Orange c = (Orange)fruit.get(); // No warning
	    } catch(Exception e) { System.out.println(e); }
	    // fruit.set(new Apple()); // Cannot call set()
	    // fruit.set(new Fruit()); // Cannot call set()
	    System.out.println(fruit.equals(d)); // OK
	  }
}
```
实例代码：
```java
public class Wildcards {

	static void rawArgs(Holder holder, Object arg) {
	     holder.set(arg); // Warning:
	    //   Unchecked call to set(T) as a
	    //   member of the raw type Holder
	    // holder.set(new Wildcards()); // Same warning

	    // Can't do this; don't have any 'T':
	    // T t = holder.get();

	    // OK, but type information has been lost:
	    Object obj = holder.get();
	  }

	  // Similar to rawArgs(), but errors instead of warnings:
	  static void unboundedArg(Holder<?> holder, Object arg) {
	    // holder.set(arg); // Error:
	    //   set(capture of ?) in Holder<capture of ?>
	    //   cannot be applied to (Object)
	    // holder.set(new Wildcards()); // Same error

	    // Can't do this; don't have any 'T':
	     Object t = holder.get();

	    // OK, but type information has been lost:
	    Object obj = holder.get();
	  }

	  static <T> T exact1(Holder<T> holder) {
	    T t = holder.get();
	    return t;
	  }

	  static <T> T exact2(Holder<T> holder, T arg) {
	    holder.set(arg);
	    T t = holder.get();
	    return t;
	  }

	  static <T> T wildSubtype(Holder<? extends T> holder, T arg) {
	    // holder.set(arg); // Error:
	    //   set(capture of ? extends T) in
	    //   Holder<capture of ? extends T>
	    //   cannot be applied to (T)
	    T t = holder.get();
	    return t;
	  }

	  static <T> void wildSupertype(Holder<? super T> holder, T arg) {
	    holder.set(arg);
	    // T t = holder.get();  // Error:
	    //   Incompatible types: found Object, required T

	    // OK, but type information has been lost:
	    Object obj = holder.get();
	  }


	  public static void main(String[] args) {
	    Holder raw = new Holder<Long>();
	    // Or:
	    raw = new Holder();
	    Holder<Long> qualified = new Holder<Long>();
	    Holder<?> unbounded = new Holder<Long>();
	    Holder<? extends Long> bounded = new Holder<Long>();
	    Long lng = 1L;

	    //原生类型的对象可以将任何类型的对象传递进去，而这个对象将会被向上转型为Object
	    rawArgs(raw, lng);
	    rawArgs(qualified, lng);
	    rawArgs(unbounded, lng);
	    rawArgs(bounded, lng);

	    //无界通配符虽然看起来和原生一样但是这里不能 set进一个对象，因为他持有的是具有同种类型的同构集合。这里还不能确定是那种类型
	    unboundedArg(raw, lng);
	    unboundedArg(qualified, lng);
	    unboundedArg(unbounded, lng);
	    unboundedArg(bounded, lng);

	    Object r1 = exact1(raw); // Warnings:
	    Long r2 = exact1(qualified);
	    Object r3 = exact1(unbounded); // Must return Object
	    Long r4 = exact1(bounded);

	    // Long r5 = exact2(raw, lng); // Warnings:
	    //   Unchecked conversion from Holder to Holder<Long>
	    //   Unchecked method invocation: exact2(Holder<T>,T)
	    //   is applied to (Holder,Long)
	    Long r6 = exact2(qualified, lng);
	     //Long r7 = exact2(unbounded, lng); // Error:
	    //   exact2(Holder<T>,T) cannot be applied to
	    //   (Holder<capture of ?>,Long)
	    // Long r8 = exact2(bounded, lng); // Error:
	    //   exact2(Holder<T>,T) cannot be applied
	    //   to (Holder<capture of ? extends Long>,Long)

	    // Long r9 = wildSubtype(raw, lng); // Warnings:
	    //   Unchecked conversion from Holder
	    //   to Holder<? extends Long>
	    //   Unchecked method invocation:
	    //   wildSubtype(Holder<? extends T>,T) is
	    //   applied to (Holder,Long)
	    Long r10 = wildSubtype(qualified, lng);
	    // OK, but can only return Object:
	    Object r11 = wildSubtype(unbounded, lng);
	    Long r12 = wildSubtype(bounded, lng);

	    // wildSupertype(raw, lng); // Warnings:
	    //   Unchecked conversion from Holder
	    //   to Holder<? super Long>
	    //   Unchecked method invocation:
	    //   wildSupertype(Holder<? super T>,T)
	    //   is applied to (Holder,Long)
	    wildSupertype(qualified, lng);
	    // wildSupertype(unbounded, lng); // Error:
	    //   wildSupertype(Holder<? super T>,T) cannot be
	    //   applied to (Holder<capture of ?>,Long)
	    // wildSupertype(bounded, lng); // Error:
	    //   wildSupertype(Holder<? super T>,T) cannot be
	    //  applied to (Holder<capture of ? extends Long>,Long)
	  }
}

```
经过上面的例子我们看到，Holder 和 Holder<?> 是不同的事物，Holder 将持有任何类型的组合，Holder<?> 将持有具有某种相同具体类型的组合。因此它也是不确定的，所以我们不能向其 set 任何类型。我们也不能将其作为一种类型传递给谁。但是我们可以使用 get。使用那种通配符取决于是否想要从泛型参数中返回类型确定的值，或者是否想要向泛型参数中传入类型确定的参数。

因此，使用确切类型来替代通配符的好处是，可以用泛型参数做更多的事情，但是使用通配符你必须接受范围更宽的参数化类型作为参数。

#### 捕获转换
有一种情况特别需要使用 <?> 而不是原生类型。如果向一个使用 <?> 的方法传递原生类型，那么对于编译器来说，可能会推断出实际的参数类型，使得这个方法可以回转并调用另一个使用确切类型的方法。这种技术被称为：捕获转换，因为未指定的通配符类型被捕获，并被转换为确切的类型。

```java
public class CaptureConversion {

	 static <T> void f1(Holder<T> holder) {
	    T t = holder.get();
	    System.out.println(t.getClass().getSimpleName());
	  }

	  static void f2(Holder<?> holder) {
	    f1(holder); // Call with captured type
	  }

	  @SuppressWarnings("unchecked")
	  public static void main(String[] args) {
	    Holder raw = new Holder<Integer>(1);
	    //f1(raw); // Produces warnings
	    f2(raw); // No warnings
	    Holder rawBasic = new Holder();
	    rawBasic.set(new Object()); // Warning
	    f2(rawBasic); // No warnings
	    // Upcast to Holder<?>, still figures it out:
	    Holder<?> wildcarded = new Holder<Double>(1.0);
	    f2(wildcarded);
	  }
}

```
执行结果：
```
Integer
Object
Double
```
f1() 方法中的类型参数都是确定的，没有通配符或边界。在 f2() 中，Holder 参数是一个无界通配符，因此它看起来是未知的。但是，在 f2() 中 f1() 被调用，而 f1() 需要一个已知的参数。这里所发生的是：参数类型在调用 f2() 的过程中被捕获，因此它可以在对 f1() 的调用中被使用。

你可能知道想这项技术是否可以用于写入，但是这要求在传递 Holder<?> 时同时传入一个具体的类型。捕获转换只有在这样的情况下才可以工作：即在方法内部，你需要使用确切的类型。捕获转换的使用非常有限。

```java
static <T> void f1(Holder<T> holder) {
		 T t = holder.get();
		 holder.set(t);
		 System.out.println(holder.get().toString());
	 }
```
执行结果：
```
1
```
通过转换我们可以set进去一个 T。实际上这个 T 是一个实际的类型。

## 问题
Java 泛型在使用中会出现各种各样的问题：

#### 任何基本类型都不能作为类型参数
第一个问题，不能将 Java 的基本类型作为类型参数。

解决办法是使用其基本类型的包装器类型以及 JavaSE5 的自动包装机制。
```java
public class ListOfInt {

	 public static void main(String[] args) {
		    List<Integer> li = new ArrayList<Integer>();
		    for(int i = 0; i < 5; i++)
		      li.add(i);
		    for(int i : li)
		      System.out.print(i + " ");
	 }

}
```
执行结果：
```
0 1 2 3 4
```
这种解决方案工作的很好，但是如果比较看重性能问题，就需要使用专门适配基本类型的容器版本。
```java
Org.apache.commons.collections.primitives
```
```java
class FArray {
	  public static <T> T[] fill(T[] a, Generator<T> gen) {
	    for(int i = 0; i < a.length; i++)
	      a[i] = gen.next();
	    return a;
	  }
}


public class PrimitiveGenericTest {

	public static void main(String[] args) {
	    String[] strings = FArray.fill(
	      new String[7], new RandomGenerator.String(10));
	    for(String s : strings)
	      System.out.println(s);
	    Integer[] integers = FArray.fill(
	      new Integer[7], new RandomGenerator.Integer());
	    for(int i: integers)
	      System.out.println(i);
	    // Autoboxing won't save you here. This won't compile:
	    // int[] b =
	    //   FArray.fill(new int[7], new RandIntGenerator());
	  }

}
```
有了自动包装机制，我们希望 gen.next() 的值从 Integer 转换为 int。但是，自动包装机制不能应用于数组，因此无法工作。

#### 实现参数化接口
一个类不能实现同一个泛型接口的两种变体，由于擦除的原因，这两个变体会指向同一个接口，也就是会产生冲突。
```java
public interface Payable<T> {

}
```
子类：
```java
public class Employee implements Payable<Employee>{

}
```
同时去实现上边的两个类：
```java
public class Hourly extends Employee implements Payable<Hourly>{

}

```
Hourly 将不能够编译，因为擦除机制会把 Payable<Employee> 和 Payable<Hourly> 简化为相同的类 Payable。这样就以为这你在重复的去实现同样的接口。从两种方法中都移除掉泛化参数就可以编译。

#### 转型和警告
使用泛型类型的参数进行的转型或 instanceof 不会有任何效果：
```java
public class FixedSizeStack<T> {
	 private int index = 0;
	  private Object[] storage;
	  public FixedSizeStack(int size) {
	    storage = new Object[size];
	  }
	  public void push(T item) { storage[index++] = item; }
	  //这里的转型是没有任何意义的，因为 T 会被擦除为 Object
	  @SuppressWarnings("unchecked")
	  public T pop() { return (T)storage[--index]; }
}

```
调用：
```java
public class GenericCast {

	public static final int SIZE = 10;
	  public static void main(String[] args) {
	    FixedSizeStack<String> strings =
	      new FixedSizeStack<String>(SIZE);
	    for(String s : "A B C D E F G H I J".split(" "))
	      strings.push(s);
	    for(int i = 0; i < SIZE; i++) {
	      String s = strings.pop();
	      System.out.print(s + " ");
	    }
	  }

}

```
执行结果：
```
J I H G F E D C B A
```
虽然运行成功了，但是 pop() 方法中的转型是没有意义的，因为编译器无法知道这个转型是否是安全的。擦除将会把 T 变为 Object。因此上边的做法实际上是将 Object 转型为 Object。

泛型没有消除对转型的需要：
```java
public class NeedCasting {
	//@SuppressWarnings("unchecked")
	  public void f(String[] args) throws Exception {
	    ObjectInputStream in = new ObjectInputStream(
	      new FileInputStream(args[0]));
	    //in.readObject() 无法知道我们正在读取什么，因此它返回的必须是转型的对象。
	    List<Apple> shapes = (List<Apple>)in.readObject();
	  }
}
```
其实上边的例子我们被强制要求转型，但是似乎又不该转型，因为不知道下一个读取的是什么内容。为了解决这个问题，JavaSE5 中引入了新的转型方式。即通过泛型类来解决。
```java
public class NeedCasting {
	 @SuppressWarnings("unchecked")
	  public void f(String[] args) throws Exception {
	    ObjectInputStream in = new ObjectInputStream(
	      new FileInputStream(args[0]));
	    List<Apple> shapes = List.class.cast(in.readObject());
	  }
}

```
按照上面的办法虽然不用强制类型转换了，但是依然无法转型到实际的类型。去掉注释之后，警告依然存在：
```java
 List<Apple> shapes = (List<Apple>)List.class.cast(in.readObject());
```
这样也不行，警告还是会存在。

#### 重载
我们看下面的程序：
```java
public class UseList<W,T> {
	 void f(List<T> v) {}
	  void f(List<W> v) {}
}

```
这个程序是不能编译的，由于擦除的原因，你将得到相同的两个方法。

解决办法，当参数由于擦除不能产生不同的参数列表时，必须提供不同的方法名：
```java
public class UseList<W,T> {
	 void f1(List<T> v) {}
	  void f2(List<W> v) {}
}

```
#### 基类劫持接口
看下面的接口：
```java
public interface Comparable<T> {

}
```
实现这个接口：
```java
public class ComparablePet implements Comparable<ComparablePet> {
	  public int compareTo(ComparablePet arg) { return 0; }
}

```
看下面的代码：
```java
public class Cat extends ComparablePet implements Comparable<Cat>{

}
```
不能够编译，还记得刚才的一个类不同重复实现相同的泛化类型的接口吗？还有一个原因是：一旦为 Comparable 确定了参数，那么就不能再更改为其他的类型参数。我们修改下面的代码：
```java
public class Cat extends ComparablePet implements Comparable<ComparablePet>{

}
```
这样就可以编译了，说明 ComparablePet 中的相同的接口是可能的，只要他们精确的相同。
