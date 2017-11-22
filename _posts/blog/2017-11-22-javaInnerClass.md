---
layout: post
title: java编程思想之内部类详解
categories: javaThinking
description: java编程思想之内部类详解
keywords: java, java内部类, java内部类详解
---

将一个类的定义放在另一个类的定义内部，这就是内部类。

内部类是一种非常有用的特性，它允许你将一些逻辑相关的类组织在一起，并控制位于内部类的可视性。内部类与组合是完全不同的概念。内部类可以与外部类进行通讯。

## 创建内部类
创建内部类很简单，把类的定义置于外部类的里面：

```java
public class Parcell {

	class Contents{
		private int i =0;
		public int value(){
			return i;
		}

	}

	class Destination{
		private String lable;

		protected Destination(String lable) {
			super();
			this.lable = lable;
		}

		private String readLable(){
			return lable;
		}
	}

	public void  ship(String des) {
		Contents contents = new Contents();
		Destination destination = new Destination(des);
		System.out.println(destination.readLable());

	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Parcell parcell = new Parcell();
		parcell.ship("InnerTest");
	}

}
```
我们看到在 ship() 方法里边使用内部类的时候和使用普通的类并没有什么差别。更典型的情况，外部类有一个方法，该方法返回一个内部类的引用。

```java
public class Parcell {

	class Contents{
		private int i =0;
		public int value(){
			return i;
		}

	}

	class Destination{
		private String lable;

		protected Destination(String lable) {
			super();
			this.lable = lable;
		}

		private String readLable(){
			return lable;
		}
	}

	public Destination getDes(String string) {
		return new Destination(string);
	}

	public Contents getCon() {
		return new Contents();
	}

	public void  ship(String des) {
		Contents contents = getCon();
		Destination destination = getDes(des);
		System.out.println(destination.readLable());

	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Parcell parcell = new Parcell();
		parcell.ship("InnerTest");

		Parcell parcell2 = new Parcell();
		Parcell.Contents contents = parcell2.getCon();
		Parcell.Destination destination = parcell2.getDes("Borneo");
		System.out.println(destination.readLable());

	}

}
```
## 链接到外部类
当生成一个内部类对象时，此对象与制造他的外部类之间就有了一种联系，内部类就可以访问外部类的所有成员，而不需要任何特殊的条件。

```java
public interface Selector {
	boolean end();
	Object current();
	void next();

}

```
示例代码：
```java
public class OutherClass {

	private Object[] items;
	private int next =0;

	public  OutherClass(int size) {
		items = new Object[size];
	}

	public void add(Object object) {
		if (next < items.length) {
			items[next++] = object;
		}
	}

	private class SequenceSelector implements Selector{

		private int i =0;

		@Override
		public boolean end() {
			// TODO Auto-generated method stub
			return i == items.length;
		}

		@Override
		public Object current() {
			// TODO Auto-generated method stub
			return items[i];
		}

		@Override
		public void next() {
			// TODO Auto-generated method stub
			if (i<items.length) {
				i++;
			}
		}

	}

	public Selector selector() {
		return new SequenceSelector();
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		OutherClass oClass = new OutherClass(10);
		for (int i = 0; i < 10; i++) {
			oClass.add(Integer.toString(i));
		}

		Selector selector = oClass.selector();
		while (!selector.end()) {
			System.out.print(selector.current()+"");
			selector.next();
		}
	}

}
```
内部类可以访问外部类的方法和字段，就像自己拥有的。内部类自动拥有外部类的访问权限这是如何做到的呢？当创建一个内部类对象时，此内部类对象会秘密的捕获一个指向外部类对象的引用。然后，当你访问外部类成员时，就是用那个引用来选择外部类的成员的。编译器会帮你处理所有的细节。

## 使用 .this 和 .new

如果想要生成对外部类对象的引用，可以使用外部类名字加上 .this这样的语法。这样产生的引用自动具有正确的地址。这一点在编译器就被检查。因此没有任何运行时开销。

```java
public class InnerClassTest {

	void f(){
		System.out.println("InnerClassTest f()");

	}

	public class Inner{
		public InnerClassTest outer() {
			//使用 .this创建对象
			return InnerClassTest.this;
		}
	}

	public Inner getInner() {
		return new Inner();
	}
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		InnerClassTest test = new InnerClassTest();
		InnerClassTest.Inner inner = test.getInner();


	}

}
```
当我们用外部对象去创建内部对象的引用。

```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		InnerClassTest test = new InnerClassTest();
		//使用 .new 创建对象
		InnerClassTest.Inner inner2 = test.new Inner();
	}
```

另外一种情况，当我们的内部类是静态的，我们就不必使用外部类对象去创建内部类对象了，而是可以直接像我们的普通类一样去 new。

```java
public  class InnerClassTest {

	void f(){
		System.out.println("InnerClassTest f()");

	}

	public static class Inner{

	}

	public Inner getInner() {
		return new Inner();
	}
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		InnerClassTest test = new InnerClassTest();
		//使用 .new 创建对象
		InnerClassTest.Inner inner2 = new Inner();

	}

}

```
**记住：以上的两种创建内部类对象方式都是我们要使用外部类对象的引用方式。如果我们不需要那么当然可以直接 new。我们这样做的原因在于为了解决内部类的作用域的局限性。**

## 内部类与向上转型
将内部类向上转型为其基类，尤其是转型为一个接口的时候。所得到的只是指向基类或者接口的引用，所以能够很方便的隐藏实现细节。我们再来复习一下链接到外部类中的那段代码示例。SequenceSelector 是一个实现了 Selector 接口的内部类。我们将其私有化。此时我们在同一个包中的另外一个类中去实例化他。

![](/images/blog/innernot.png)

我们看到，我们可以通过的公开的方法去得到他的基类或者接口的对象引用。但是我们不能同过 new 去创建引用。这样我们就完全隐藏了实现细节，客户端程序员仅仅能够得到一个接口引用，这种方式可以完全阻止任何依赖于类型的编码。

## 在方法和作用域内的内部类
我们可以在一个方法里或者在任意的作用域内定义内部类。

在当前类的方法作用域内创建一个完整的类被称作局部内部类。

```java
public class OutherClass {

	private Object[] items;
	private int next =0;

	public  OutherClass(int size) {
		items = new Object[size];
	}

	public void add(Object object) {
		if (next < items.length) {
			items[next++] = object;
		}
	}


	public Selector selector() {

		 class SequenceSelector implements Selector{

			private int i =0;

			@Override
			public boolean end() {
				// TODO Auto-generated method stub
				return i == items.length;
			}

			@Override
			public Object current() {
				// TODO Auto-generated method stub
				return items[i];
			}

			@Override
			public void next() {
				// TODO Auto-generated method stub
				if (i<items.length) {
					i++;
				}
			}

		}

		return new SequenceSelector();
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		OutherClass oClass = new OutherClass(10);
		for (int i = 0; i < 10; i++) {
			oClass.add(Integer.toString(i));
		}

		//Selector selector = oClass.selector();
		Selector selector =oClass.selector();
		while (!selector.end()) {
			System.out.print(selector.current()+"");
			selector.next();
		}
	}

}
```
方法内的内部类的作用域仅限于方法内部，不能再方法外部直接访问。

我们在来看一个创建在另外作用域内的内部类：

```java
public class InnerScope {

	public void name(boolean b) {
		if (b) {
			class Contents{
				private int i =0;
				public int value(){
					return i;
				}

			}
			System.out.println("任意作用域");
		}
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		InnerScope scope = new InnerScope();
		scope.name(true);
	}

}

```
这个内部类被嵌入逻辑语句的作用域，这并不意味着该类的创建是有条件的。仅仅意味着他的作用范围在哪。除此之外他和一个普通的类没什么区别。

## 匿名内部类
先看一个例子：

```java
public class BaseClass {

	public int  name() {
		return 1;
	}

}
```
创建匿名内部类：
```java
public class AnonyInnerClass {

	public BaseClass name() {
		return new BaseClass(){
			private int  i=0;
			@Override
			public int name() {
				// TODO Auto-generated method stub
				return i;
			}
		};
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		AnonyInnerClass innerClass = new AnonyInnerClass();
		BaseClass bClass = innerClass.name();
	}

}
```
name()方法将返回值的生成和返回值的创建结合在了一起！另外这个类是匿名的，没有名字。这种语法表示：创建一个继承自 BaseClass 的匿名内部类的对象。通过 new 表达式创建的对象通过自动向上转型为对 BaseClass 的引用。

上述代码其实展示的是使用的默认的不带参数的构造器创建了一个匿名内部类，如果我们使用带参数的构造器该如何创建呢？只需要传递合适的参数给基类的构造器即可。

基类：

```java
public class BaseClass {


	protected BaseClass(int x) {
		super();
		// TODO Auto-generated constructor stub
	}

	public int  name() {
		return 1;
	}

}

```
创建内部类：

```java
public class AnonyInnerClass {

	public BaseClass name(int x) {
		return new BaseClass(x){
			private int  i=0;
			@Override
			public int name() {
				// TODO Auto-generated method stub
				return i;
			}
		};
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		AnonyInnerClass innerClass = new AnonyInnerClass();
		BaseClass bClass = innerClass.name(10);
	}

}
```
在匿名内部类的结尾有一个分号，这并不是来标记此内部类的结束。实际上是标记的表达式的结束。

匿名内部类中的字段初始化操作:

```java
public class AnonyInnerClass {

	public BaseClass name(String  x) {
		return new BaseClass(){
			public String s =x;
			@Override
			public int name() {
				// TODO Auto-generated method stub
				return 0;
			}


		};
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		AnonyInnerClass innerClass = new AnonyInnerClass();
		BaseClass bClass = innerClass.name("aa");
	}

}
```
在匿名内部类中没有构造器，因为他是匿名的没有名字，但是通过实例的初始化，就能够达到为匿名内部类创建一个构造器的效果：

```java
public class AnonyInnerClass {

	public static BaseClass name(int  x) {
		return new BaseClass(x){
			public int  s =x;
			{System.out.println("内部类初始化被执行");}
			@Override
			public int name() {
				// TODO Auto-generated method stub
				System.out.println("内部类中非name方法被执行");
				return 0;
			}


		};
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		AnonyInnerClass innerClass = new AnonyInnerClass();
		BaseClass bClass =name(10);
		bClass.name();
	}

}

```
执行结果：
```
基类构造器被初始化
内部类初始化被执行
内部类中非name方法被执行

```
**我们看到两个大括号内的内容被执行了。貌似没什么意义。如果你们知道请在文章留言。**
匿名内部类与正规的继承相比有些受限，因为匿名内部类对于继承和实现接口只能选择其一

## 工厂方法
我们使用匿名内部类来修改一下之前的一个工厂方法：

```java
public interface Service {

	void method1();
	void method2();

}

```
工厂接口：
```java
public interface ServiceFactory {
	Service getService();
}

```
创建服务的实现类1：
```java
public class ServiceImp1 implements Service{

	private ServiceImp1() {}

	@Override
	public void method1() {}

	@Override
	public void method2() {	}

	public static ServiceFactory factory = new ServiceFactory() {

		@Override
		public Service getService() {
			// TODO Auto-generated method stub
			return new ServiceImp1();
		}
	};

}

```
创建服务的实现类2：

```java
public class ServiceImpl2 implements Service{

	private ServiceImpl2() {}

	@Override
	public void method1() {}

	@Override
	public void method2() {}

public static ServiceFactory factory = new ServiceFactory() {

		@Override
		public Service getService() {
			// TODO Auto-generated method stub
			return new ServiceImpl2();
		}
	};

}

```
调用并执行：
```java
public class Factories {

	public static void serviceConsumer(ServiceFactory serviceFactory) {
		Service service = serviceFactory.getService();
		service.method1();
		service.method2();
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		serviceConsumer(ServiceImp1.factory);
		serviceConsumer(ServiceImpl2.factory);
	}

}
```
## 嵌套类
如果不需要内部类对象与外部类对象有联系，那么可以将内部类声明为 static。这通常被称为嵌套类。static 应用于内部类的含义：普通的内部类对象隐式的保存一个引用，指向创建它的外围类对象。如果加上 static 就不是这样了。
 - 要创建嵌套类的对象，并不需要外部类的对象
 - 不能从嵌套类的对象中访问非静态的外部类对象。

#### 静态内部类
修改一下上面的例子:

```java
public  class InnerClassTest {
	private static int x =0;
	void f(){
		System.out.println("InnerClassTest f()");

	}

	public static class Inner{
		public  void name() {
			x = 10;
		}
	}

	public static  Inner getInner() {
		return new Inner();
	}
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		InnerClassTest test = new InnerClassTest();
		Inner inner1 = new Inner();
		Inner inner2 = getInner();

	}

}
```
一个普通的内部类可以通过 .this 来链接外部类的引用。但是在静态内部类中却没有这样的引用。

#### 接口内部类
在正常情况下不能再接口中放置任何代码，但是嵌套类可以作为接口的一部分。因为接口中的成员被默认赋予 public 和 static 的。所以把嵌套类放在接口的命名空间内并不违反规则。

```java
public interface InterfaceInnerClass {

	void f();

	class Test implements InterfaceInnerClass{

		@Override
		public void f() {
			// TODO Auto-generated method stub
			System.out.println("接口内部类");
		}

		public static void main(String[] args) {
			new Test().f();
		}

	}

}
```
调用接口的内部类
```java
public class TestInterfae implements InterfaceInnerClass{

	@Override
	public void f() {
		// TODO Auto-generated method stub

	}

	public static void main(String[] args) {
		InterfaceInnerClass.Test.main(args);
	}

}

```
如果想要创建某些公共代码，使得他可以被接口的不同实现所公用。那么使用接口内部类是方便的。

#### 从多重嵌套类中访问外部类的成员
一个内部类不管嵌套多少层次，他都能透明的访问所有他所嵌入的外围类的所有成员。
```java
public class MNA {
	public void f() {

	}

	class A{
		private void g() {

		}

		class B{
			void h(){
				f();
				g();
			}
		}
	}

}
```
调用代码：
```java
public class MultiNesting {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		MNA mna = new MNA();
		MNA.A name = mna.new A();
		MNA.A.B name2 = name.new B();
		name2.h();
	}
}
```
上边的例子展示了如何从不同的类里边创建多层嵌套内部类对象的基本语法。

## 为什么需要内部类
一般来说内部类继承自某各类或实现某个接口，内部类提供了某种进入外部类的窗口。

每个内部类都能独立的继承自一个接口的实现，所以无论外围类是否已经继承了某个接口的实现，对于内部类都没有影响。如果没有内部类提供的可以继承多个具体的或者抽象的类的能力，一些设计与编程问题就难以解决。从这个角度看，内部类使得多重继承的方案变得完整。也就是说内部类允许继承多个非接口类型。

比如我们继承的是抽象的类或具体的类，而不是接口，那么就只能使用内部类才能实现多重继承。如果不需要解决多重继承问题，那么自然可以使用别的代码方式，而不需要使用内部类。但是使用内部类还可以获得其他的一些特性：

- 内部类可以有多个实例，每个实例都有自己的状态信息。
- 在单个外围类中，可以让多个内部类以不同的方式实现同一个接口，或继承同一个类。
- 创建内部类对象的时刻并不依赖于外围类对象的创建。
- 内部类并没有令人疑惑的 is-a 关系。它是一个独立的实体。

## 内部类的继承
因为内部类的构造器必须连接指向其外围类对象的引用，所以在继承内部类的时候，事情会变得复杂。问题在于，那个指向外部类的秘密引用必须被初始化。而在继承中不存在任何可连接的默认对象。如何解决呢？看下面的例子

内部类：
```java
public class WithInner {

	public class Inner{

	}


}

```
继承内部类：
```java
public class InheritInner extends WithInner.Inner{


	protected InheritInner(WithInner inner) {
		inner.super();
		// TODO Auto-generated constructor stub
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		WithInner inner = new WithInner();
		InheritInner inheritInner = new InheritInner(inner);
	}

}

```
## 内部类可以被覆盖吗？
如果我们继承一个外部类，并且试图去覆盖它的外部类的时候，会发生什么？

```java
public class Egg {
	private Yolk yolk;
	class Yolk{
		public Yolk() {
			// TODO Auto-generated constructor stub
			System.out.println("Egg的Yolk");
		}
	}

	public Egg() {
		// TODO Auto-generated constructor stub
		System.out.println("new Egg()");
		yolk = new Yolk();
	}

}
```
继承外部类：
```java
public class BigEgg extends Egg{

	class Yolk{
		public Yolk() {
			// TODO Auto-generated constructor stub
			System.out.println("BigEgg的Yolk");
		}
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		new BigEgg();
	}

}

```
我们看到内部类并没有被覆盖。这两个内部类是完全不同的两个实体。

当我们明确的继承某个内部类也是可以的。

```java
public class Egg {
	private Yolk yolk = new Yolk();;
	class Yolk{
		public Yolk() {
			// TODO Auto-generated constructor stub
			System.out.println("Egg的Yolk");

		}

		public void f() {
			// TODO Auto-generated method stub
			System.out.println("Egg的f方法");
		}
	}

	public Egg() {
		// TODO Auto-generated constructor stub
		System.out.println("new Egg()");		
	}

	public void  insert(Yolk yolk) {
		this.yolk = yolk;
	}
	public void g(){
		yolk.f();
	}

}
```
如何覆盖呢？
```java
public class BigEgg extends Egg{

	public BigEgg() {
		// TODO Auto-generated constructor stub
		insert(new Yolk());
	}

	class Yolk extends Egg.Yolk{


		public Yolk() {
			// TODO Auto-generated constructor stub
			System.out.println("BigEgg的Yolk");

		}

		@Override
		public void f() {
			// TODO Auto-generated method stub
			System.out.println("BigEgg的f方法");
		}
	}


	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Egg egg = new BigEgg();
		egg.g();

	}

}

```
执行结果：
```
Egg的Yolk
new Egg()
Egg的Yolk
BigEgg的Yolk
BigEgg的f方法
```
## 内部类标识符
由于每个类都会产生一个 .class 文件，其中包含了该类型的对象的全部信息。内部类也会产生一个 .Class 文件。这类文件有明确的命名规则：外围类的名字加上「$」符号在加上内部类的名字。如果内部类是嵌套在别的内部类中，那么继续在后面加符号加内部类名字。如果内部类是匿名的，编译器会简单的产生一个数字作为标识符。

![](/images/blog/innerClassName.png)
