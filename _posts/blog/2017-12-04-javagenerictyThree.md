---
layout: post
title: java编程思想之泛型三 (通配符)
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
