---
layout: post
title: java编程思想之接口详解
categories: javaThinking
description: java编程思想之接口详解
keywords: java, java接口, java接口详解，接口详解
---

接口和内部类为我们提供了一种将接口和实现分离的方法。

我们在学习接口之前，首先要学习抽象类，他是普通的类与接口之间的一种间接实现。尽管我们在做一些编程未实现的方法时，我们第一想法可能是接口，但是抽象类仍然是实现这些目的的必须的技术。我们不可能的单纯去使用接口。

## 抽象类和抽象方法

在上一节中我们创建了一个基类 Instrument。这个基类中的方法往往都是哑方法，不能直接调用的。我们通常是希望通过继承它的子类去实现具体的方法。而基类的目的只是为所有的子类建立一个通用的接口。建立这个通用接口的理由是，不同的子类可以用不同的方法去表达这个接口。另外一种做法就是将 Instrument 类作为抽象类。

如果我们只有一个 Instrument 这样的抽象类，那么该类的对象几乎是没有任何意义的。我们创建抽象类的目的是希望通过这个通用的接口去操纵一系列的类。因此抽象类只是表示一个接口并不能表示具体的对象。没有具体的实现内容。因此，创建一个抽象类的对象是没有实际意义的。那么如何创建抽象类呢？Java 提供了一个抽象方法的机制，这个方法是不完整的，仅有声明而没有方法体。使用 abstract 关键字作为修饰。

```
abstract void f();
```
包含抽象方法的类叫做抽象类。如果一个类包含一个或者多个抽象方法，该类必须被限定为抽象的。我们也可以创建一个没有任何抽象方法的抽象类。

看下面的示例图：

![](/images/blog/abstractIns.png)

下面是修改过的例子：

```java
abstract class Instrument {

	abstract void play(Note note);

	String what(){
		return "Instrument";
	}

	abstract void adjust();
}
```
继承它的子类 Wind：
```java
public class Wind extends Instrument{

	@Override
	public void play(Note note) {
		// TODO Auto-generated method stub
		System.out.println("Window.paly"+note);
	}

	@Override
	String what() {
		// TODO Auto-generated method stub
		return "Wind";
	}

	@Override
	void adjust() {
		// TODO Auto-generated method stub

	}
}
```
继承它的子类 Percussion：
```java
public class Percussion extends Instrument{

	@Override
	public void play(Note note) {
		// TODO Auto-generated method stub
		System.out.println("Percussion.paly"+note);
	}

	@Override
	String what() {
		// TODO Auto-generated method stub
		return "Percussion";
	}

	@Override
	void adjust() {
		// TODO Auto-generated method stub

	}
}
```
继承它的子类 Stringed：
```java
public class Stringed extends Instrument{

	@Override
	public void play(Note note) {
		// TODO Auto-generated method stub
		System.out.println("Stringed.paly"+note);
	}

	@Override
	String what() {
		// TODO Auto-generated method stub
		return "Stringed";
	}

	@Override
	void adjust() {
		// TODO Auto-generated method stub

	}
}
```
二级子类 Brass：
```java
public class Brass extends Wind{

	@Override
	public void play(Note note) {
		// TODO Auto-generated method stub
		System.out.println("Brass.paly"+note);
	}

	@Override
	String what() {
		// TODO Auto-generated method stub
		return "Brass";
	}

	@Override
	void adjust() {
		// TODO Auto-generated method stub

	}
}
```
二级子类 Woodwind：
```java
public class Woodwind extends Wind{
    @Override
    public void play(Note note) {
    	// TODO Auto-generated method stub
    	System.out.println("Woodwind.paly"+note);
    }

    @Override
    String what() {
    	// TODO Auto-generated method stub
    	return "Woodwind";
    }
}
```
调用并执行：
```java
public class UpOverloadTest {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Instrument[] instruments = {new Wind(),new Percussion(),new Stringed(),new Woodwind(),new Brass()};

		tune(instruments);

	}

	public static void  tune(Instrument instrument) {
		instrument.play(Note.MID);
	}

	private static void  tune(Instrument[] instruments) {
		for (Instrument instrument : instruments) {
			tune(instrument);
		}
	}

}
```
执行结果：
```
Window.palyMID
Percussion.palyMID
Stringed.palyMID
Woodwind.palyMID
Brass.palyMID

```
我们可以看出来，除了基类别的和上一节的例子没有任何区别。创建抽象类和抽象方法非常有用，因为他可以使得类的抽象性明确起来，并且告诉用户和编译器打算怎么来使用它。他可以很容易的将公共方法沿着继承层次结构向上移动。

## 接口

interface 关键字使得抽象的概念更进一步。abstract 关键字允许我们在类中创建一个或者多个没有任何定义的方法，但是并没有提供相应的具体实现，这些实现是又此类的继承者实现的。interface 关键字产生一个完全抽象的类，它不提供任何具体实现，这是不允许的。它允许创建者确定方法名、参数列表和返回类型，但是没有任何方法体。同样不提供具体的实现。一个接口表示：所有实现该接口的类看起来都像这样。

但是，interface 关键字不仅仅是一个极度抽象的类，它允许我们创建一个能够被向上转型为多种基类的类型。

要想创建接口，需要用 interface 关键字来替代 class 关键字。可以在关键字前面加权限修饰符。接口也可以包含成员变量，但是这些变量都是隐式的 static 和 final。要想某各类遵循特定的接口，需要使用 implements 关键字。

看下面的示例图：

![](/images/blog/interfaceIm.png)

从最下面两个类可以看出，一旦实现了某个接口就变成了一个普通的类。我们可以任意的去进行扩展。接口中的方法不管你是否定义为 public 权限他们都将会被默认为 public 的。如果不这样做的话，它就只有包访问权限，这样在方法被继承的过程中，其访问权限就被降低了。Java 编译器是不允许的。

我们继续修改上面的例子：

接口:
```java
interface Instrument {
	int VALUE = 5;

	void play(Note note);

	void adjust();

}

```
实现接口的 Wind：
```java
public class Wind implements Instrument{

	public void play(Note note) {
		// TODO Auto-generated method stub
		System.out.println("Window.paly"+note);
	}

	public void adjust() {
		// TODO Auto-generated method stub

	}

	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return "Wind";
	}
}

```
实现接口的 Percussion：
```java
public class Percussion implements Instrument{

	@Override
	public void play(Note note) {
		// TODO Auto-generated method stub
		System.out.println("Percussion.paly"+note);
	}

	@Override
	public void adjust() {
		// TODO Auto-generated method stub

	}

	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return "Percussion";
	}
}
```
实现接口的 Stringed：
```java
public class Stringed implements Instrument{

	@Override
	public void play(Note note) {
		// TODO Auto-generated method stub
		System.out.println("Stringed.paly"+note);
	}

	@Override
	public void adjust() {
		// TODO Auto-generated method stub

	}

	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return "Stringed";
	}
}
```
继承实现 Brass 类：
```java
public class Brass extends Wind{
	@Override
	public void play(Note note) {
		// TODO Auto-generated method stub
		System.out.println("Brass.paly"+note);
	}

	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return "Brass";
	}
}
```
继承实现 Woodwind 类：
```java
public class Woodwind extends Wind{

	@Override
	public void play(Note note) {
		// TODO Auto-generated method stub
		System.out.println("Woodwind.paly"+note);
	}
	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return "Woodwind";
	}
}
```
调用并执行：
```java
public class UpOverloadTest {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Instrument[] instruments = {new Wind(),new Percussion(),new Stringed(),new Woodwind(),new Brass()};

		tune(instruments);

	}

	public static void  tune(Instrument instrument) {
		instrument.play(Note.MID);
	}

	private static void  tune(Instrument[] instruments) {
		for (Instrument instrument : instruments) {
			tune(instrument);
		}
	}

}
```
执行结果：
```
Window.palyMID
Percussion.palyMID
Stringed.palyMID
Woodwind.palyMID
Brass.palyMID
```
我们可以看到无论是向上转型为 Instrument 普通类，还是 Instrument 抽象类，或者是转型为 Instrument 的接口，都不会有问题。它的行为都是相同的。

## Java 中的多重继承
接口不仅仅是一个更纯粹的抽象类。在 C++ 中组合多个类的接口的行为被称作是多重继承。在 Java 中只能是单继承但是我们可以多重实现。也就是可以多重实现接口。并且可以向上转型为每个接口。

创建接口：

```java
public interface A {
	void  fight();
}

public interface B {
	void left();
}

public interface C {
	void up();
}
```
创建基类：
```java
public class D {
	void up(){
		System.out.println("这是D的up");
	}

}
```
实现类：
```java
public class Hero extends D implements A,B,C{


	@Override
	public void left() {
		// TODO Auto-generated method stub
		System.out.println("这是left");
	}

	@Override
	public void fight() {
		// TODO Auto-generated method stub
		System.out.println("这是right");
	}

	@Override
	public void up() {
		// TODO Auto-generated method stub
		System.out.println("这是up");
	}


}
```
调用并执行：
```java
public class Adventure {
	private static void r(A a) {
		a.fight();
	}

    private static void l(B b) {
		b.left();
	}

    private static void u(C c) {
    	c.up();
    }
    private static void up(D d) {
    	d.up();
    }

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Hero hero = new Hero();

		r(hero);
		l(hero);
		u(hero);
		up(hero);
	}

}
```
执行结果：
```
这是right
这是left
这是up
这是up
```
在最后的执行类中，我们看到创建了四个方法接受各种接口和具体类型作为参数。当 Hero 对象被创建时，它可以被传递给四个方法中的任何一个，这意味着它依次被转型为每一个接口。这个例子就是展示了使用接口的核心原因：为了能够向上转型为多个基类型。使用接口还有第二个原因，防止客户端程序员创建该类的对象，并确保这仅仅是创建一个接口。如果知道某事物应该成为一个基类，我们应该第一选择使用接口去实现。

## 通过继承来扩展接口
接口可以继承接口来创建一个新的接口，而且接口还可以多重继承。在 Java 中我们说过只能单继承，但是在接口中是个例外。

```java
public interface A {
	void  fight();

}

public interface B extends A{
	void left();

}

public interface C extends A,B{
	void up();
}

```
实现类：
```java
public class Hero extends D implements C{

	@Override
	public void up() {
		// TODO Auto-generated method stub
		super.up();
	}

	@Override
	public void fight() {
		// TODO Auto-generated method stub

	}

	@Override
	public void left() {
		// TODO Auto-generated method stub

	}
}

```
实现一个接口 C 就同时实现了接口 A B C 中的所有方法。这是因为接口 C 通过继承组合了三个接口中的方法。

#### 组合接口中的名字冲突
在实现多重继承时，可能会碰到一些问题。在前面的例子中我们就看到了接口中和继承类中有两个相同的 up() 方法。此时我们已经体会到了有些混乱了。但是如果我们不但具有相同的方法名还有一些不同的签名和返回方法。此时设计到了方法的覆盖、重载、实现全都堆集在一起，难以直观的去判断这些都是那些。所以，在打算组合不同的接口中使用相同的方法名通常会造成代码可读性的混乱。

## 适配接口
接口最吸引人的地方就是允许同一个接口具有多个不同的具体实现。因此接口的一个常用做法就是前面提到的策略模式，此时你编写一个执行某些操作的方法，而该方法接收一个你指定的接口。例如，Java 中 Scanner 类的构造器就是一个 Readable 接口。通过这样 Scanner 可以接受更多的类型。

```java
public class RandomWoeds implements Readable{
	private static final Random RANDOM = new Random(47);
	private static final char[] CAP = "ABCDEFG".toCharArray();
	private static final char[] LOW = "abcdefg".toCharArray();

	private int count;


	protected RandomWoeds(int count) {

		this.count = count;
	}


	@Override
	public int read(CharBuffer cb) throws IOException {
		// TODO Auto-generated method stub
				if (count-- ==0)
					return -1;
					cb.append(CAP[RANDOM.nextInt(CAP.length)]);
					cb.append(LOW[RANDOM.nextInt(LOW.length)]);
					cb.append("");
		        return 10;
	}


	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Scanner scanner = new Scanner(new RandomWoeds(10));
		while (scanner.hasNext()) {
			System.out.println(scanner.next());
		}
	}


}

```
执行结果：
```
GcGbFcAfFaCfEbDdBfAc
```
我们再来看一次适配器模式，被适配的类可以通过继承和实现接口来实现。

```java
public class AdapterRandomDoubles extends RandomDoubles implements Readable{

	private int count;


	protected AdapterRandomDoubles(int count) {
		super();
		this.count = count;
	}

	@Override
	public int read(CharBuffer cb) throws IOException {
		// TODO Auto-generated method stub
		if (count-- ==0) {
			return -1;
		}
		String result = Double.toString(next())+" ";
		cb.append(result);

		return result.length();
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Scanner scanner = new Scanner(new AdapterRandomDoubles(7));
		while (scanner.hasNextDouble()) {
			System.out.println(scanner.nextDouble());

		}
	}


}

```
执行结果：
```
0.7273845313896193
0.022312278302438093
0.4493142872518773
0.841638561955569
0.40597021504841146
0.2491698451515626
0.5258141677786065

```
在这种方式中，我们可以在任何现有类之上添加新的接口，所以这意味着让方法接受接口类型，是一种让任何类都可以对该方法进行适配的方式。这就是使用接口的强大之处。

## 接口中的域
因为我们放入接口中的任何的变量都会自动加上 static 和 final。所以接口就成为了一种很便捷的创建常量组的工具。在 JavaSE5 之前常常使用这个机制来实现 C 或者 C++ 中的枚举特性。

```java
public interface Months {
	int JAS =1, SD =2, SF=3;

}
```
我们在一些老的代码中可能会看到这样的代码，现在 Java 已经支持了枚举就不用这么写了。

#### 初始化接口中的域
我们知道接口中的域全部都是 static 和 final 的。既然是 static 的他们就可以在类第一次被加载的时候初始化，这发生在任何域被首次访问的时候。

```Java
public interface Months {
	Random eRandom = new Random(40);
	int JAS = eRandom.nextInt(10);
	long SD = eRandom.nextLong();


}

```
调用并执行：
```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		System.out.println(Months.JAS);
		System.out.println(Months.SD);
	}
```
执行结果：
```
2
-7512810983669743157
```
当然这些常量并不是接口的一部分，他们的值被存储在该接口的静态存储区域内。

## 嵌套接口
接口可以嵌套在类或者其他的接口中。

类中嵌套接口并嵌套接口实现：

```java
public class SuperA {

	interface B{
		void f();
	}


	interface C{
		void f();
	}

	private interface D{
		void f();
	}

	public class BImp implements B{

		@Override
		public void f() {
			// TODO Auto-generated method stub

		}

	}

	private class BImp2 implements B{

		@Override
		public void f() {
			// TODO Auto-generated method stub

		}

	}

	public class CImp implements C{

		@Override
		public void f() {
			// TODO Auto-generated method stub

		}

	}

	private class CImp2 implements C{

		@Override
		public void f() {
			// TODO Auto-generated method stub

		}

	}

	public class DImp implements D{

		@Override
		public void f() {
			// TODO Auto-generated method stub

		}

	}

	private class DImp2 implements D{

		@Override
		public void f() {
			// TODO Auto-generated method stub

		}

	}

	public D getD() {
		return new DImp2();

	}

	private D derf;

	public void  receiveD(D d) {
		derf = d;
		d.f();

	}

}

```
接口中嵌套接口:
```java
public interface SuperD {

	interface G{
		void f();
	}

	public interface H{
		void f();
	}

	void g();
}

```
实现类
```java
public class Test {

	public class BImp implements SuperA.B{

		@Override
		public void f() {
			// TODO Auto-generated method stub

		}

	}

	public class CImp implements SuperA.C{

		@Override
		public void f() {
			// TODO Auto-generated method stub

		}

	}

	class EImp implements SuperD{

		@Override
		public void g() {
			// TODO Auto-generated method stub

		}

	}

	class EImp1 implements SuperD.G{

		@Override
		public void f() {
			// TODO Auto-generated method stub

		}


	}


	public static void main(String[] args) {
		// TODO Auto-generated method stub
		SuperA a = new SuperA();

		a.receiveD(a.getD());
	}

}
```
作为一种新的添加的方式我们可以看到，接口也可以被实现为 private。而且在内部也可以被实现。而且接口之间也可以实现互相的嵌套。但是接口的元素都必须是 public。还有当我们实现某个接口时并不需要同时去实现接口内部的接口。被声明为 private 的接口不能再类的外部被实现。

## 接口与工厂
工厂方法设计模式遵循某个接口的对象的生成不是直接调用构造器，而是在工厂对象上调用创建方法，该工厂对象将生成接口的某个实现的具体对象。这样做我们的代码将完全与接口的实现代码分离。

服务的接口：

```java
public interface Service {

	void method1();
	void method2();

}
```
工厂的接口：

```java
public interface ServiceFactory {
	Service getService();
}
```
实现服务接口1：

```java
public class ServiceImp1 implements Service{


	protected ServiceImp1() {

		// TODO Auto-generated constructor stub
	}

	@Override
	public void method1() {
		// TODO Auto-generated method stub

	}

	@Override
	public void method2() {
		// TODO Auto-generated method stub

	}

}
```
实现服务接口2：

```java
public class ServiceImpl2 implements Service{



	protected ServiceImpl2() {

		// TODO Auto-generated constructor stub
	}

	@Override
	public void method1() {
		// TODO Auto-generated method stub

	}

	@Override
	public void method2() {
		// TODO Auto-generated method stub

	}

}

```
实现工厂接口1:

```java
public class ServiceFactoryImp1 implements ServiceFactory{


	@Override
	public Service getService() {
		// TODO Auto-generated method stub
		return new ServiceImp1();
	}

}

```
实现工厂接口2:

```java
public class ServiceFactoryImp2 implements ServiceFactory{

	@Override
	public Service getService() {
		// TODO Auto-generated method stub
		return new ServiceImpl2();
	}

}
```
创建具体的实现：

```java
public class Factories {

	public static void serviceConsumer(ServiceFactory serviceFactory) {
		Service service = serviceFactory.getService();
		service.method1();
		service.method2();
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		serviceConsumer(new ServiceFactoryImp1());
		serviceConsumer(new ServiceFactoryImp2());
	}

}

```
如果不是工厂方法，我们就必须在某处指定将要创建的 Service 的确切类型，以便调用合适的构造器。为什么要添加这样额外的连接性呢？假设 Factories 是一段复杂的代码。我们就可以在不同的地方去复用这段代码。

## 总结
对于创建类，几乎任何时候都可以替代为创建一个接口和一个工厂。许多人都会这样想。只要有可能就去创建接口和工厂。这实际上是不恰当的。任何抽象都应该应真正的需求而产生。你应该重构接口而不该额外的添加间接性，并由此带来更复杂的逻辑。这种额外的复杂性非常的显著。恰当的原则是：应该优先选择类而不是接口。从类开始如果接口的必要性变得非常明确，那么就进行重构。不应该滥用接口。
