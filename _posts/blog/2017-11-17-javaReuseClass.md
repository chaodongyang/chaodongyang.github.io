---
layout: post
title: java编程思想之复用类
categories: javaThinking
description: java编程思想之复用类
keywords: java, java复用类, 复用类
---

复用代码是 Java 的重要功能之一。但是仅仅能够复用代码并对其加以改变是不够的，它必须还要做更多的事情。Java 中所有的问题都围绕类展开。我们可以创建新类来复用代码，而不必从头开始编写。

  复用类可以有两种方法：

  1. 在新的类中产生现有类的对象。新的类是由现有类的对象组合的，所以这种方法称为组合。
  2. 按照现有类的类型来创建新类。无需改变现有类的形式，只需在其中添加新的代码。这种方式称之为继承。继承是面向对象程序设计的基础之一。


## 组合语法

只需要将对象引用置于新类中即可。

```java
package reuseClass;

public class Sourse {


	private String s;

	public Sourse() {
		s = "构造器赋值";
	}

	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return s;
	}


}

```

调用
```java
package reuseClass;


public class GroupTest {



	static class Sprint{
		private String s1,s2,s3;

		private  Sourse sourse =  new Sourse();

		private int i;

		@Override
		public String toString() {
			// TODO Auto-generated method stub
			return "s1 = " + s1 + "  " +
			"s2 = " + s2 + "  " +
			"s3 = " + s3 + "  " +
			"i = " + i + "  " +
			"sourse = " + sourse + "  " ;
		}
	}


	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Sprint sprint = new Sprint();

		System.out.println(sprint);
	}

}
```
输出结果

```
s1 = null  s2 = null  s3 = null  i = 0  sourse = 构造器赋值  
```

在上面两个类的定义中都有一个共同的方法：toString() 。每一个非基本类型的对象都有一个这样的方法，而且当编译器需要一个 String 而你却只有一个对象时，该方法就会被调用。所以我们在打印 sprint 对象的时候我们输出了一段字符串。

而且我们看到编译器并不会简单的为每一个引用创建默认对象，正如我们之前所看到的它只会为基本类型创建默认的值。

## 继承

继承是 OOP 语言不可缺少的一部分。当创建一个类时总是需要继承。除非明确指出要从其他类继承，否则就是隐式的继承自 Object 根类。如何使用继承？只许在子类和父类之间加上一个 extends 关键字。这么做子类就继承了父类的所有成员和方法。当然注意访问权限的控制。

```java
public class Sourse {


	private String s;
	protected int x;

	public Sourse() {
		s = "构造器赋值";
	}

	protected int getX() {
		return x;
	}

	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return s+"复制了"+x;
	}

  public static void main(String[] args) {

	}

}
```
子类继承父类

```java
package reuseClass;

public class Son extends Sourse{

	private void name() {

	}

	@Override
	protected int getX() {
		// TODO Auto-generated method stub

		return super.getX();
	}
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Sourse sourse = new Sourse();
		sourse.x = 10;
		sourse.getX();
		System.out.println(sourse);
	}

}
```
我们看到子类和父类中都有 main() 方法。这种在每个类中都创建有 main() 方法可以使得单元测试做起来更加的简单。即便是在完成单元测试之后也无需删除。

另外在上次学习访问权限控制的时候我们知道，继承也不完全是所有的成员和方法都能继承过来的，在同一个包中只能继承 public 和 protected 以及无修饰符的成员。在不同的包中只能继承 public 权限修饰的。请注意这一点。在上面的代码中子类可以自动获取和使用父类的 x 变量以及 getX() 方法。getX() 方法是从父类继承过来的，此方法中并不能直接调用父类的 getX() 方法。因为这样会产生递归死循环。为此，我们使用 super 关键字表示超类的意思。为此 super.getX() 将会调用超类中的方法。

在继承的过程中，并不是一定要使用超类的方法。也可以在新类中添加新的方法。

#### 初始化基类

单纯来看子类是与父类是具有相同接口的新类，或许还有一些额外的方法和成员。但继承并不是简单的复制父类的接口。当我们创建了一个子类的对象时，该对象其实包含了父类的子对象。这个子对象与你使用父类创建的对象是一样的。区别在于，后者来自于外部创建，而父类的子对象被包装在子类的内部。

对父类子对象的初始化也是重要的，而且仅有一种方法：就是在构造器中调用父类的构造器来进行初始化，而子类构造器具有默认创建父类构造器的所有能力。

```java
public class A {

	public A() {
		System.out.println("超级类A");
	}


}

```
创建子类
```java
public class B extends A{

	public B() {
		System.out.println("B继承超级类A");
	}

}
```
再次创建子类并调用
```java
public class C extends B{


	public C() {
		System.out.println("最终类C");
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		C c = new C();
	}

}
```
输出结果
```
超级类A
B继承超级类A
最终类C
```
可以看到执行的顺序，我们的编译器先创建了父类初始化，而后才是子类。构建过程是从父类向外扩散的。

**带参数的构造器**

上面的例子是默认的构造器，那么如果没有默认的构造器，想调用一个带参数的构造器就必须使用 super 关键字来显示的调用父类的构造器。

父类 A

```java
public class A {

	public A(int x) {
		System.out.println("超级类A"+"__"+x);
	}

}
```
子类 B
```java
public class B extends A{

	public B(int x) {
		super(x);
		System.out.println("B继承超级类A");
	}

}
```
子类 C 并且调用
```java
public class C extends B{


	public C() {
		super(11);
		System.out.println("最终类C");
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		C c = new C();
	}

}

```
输出结果
```
超级类A__11
B继承超级类A
最终类C
```
## 代理

java 并没有直接提供对它的支持。如果我们使用继承来的子类创建对象，并使用这个对象来调用父类的方法。这样的话父类的所有方法就会暴露出来。而且实际上我们是间接的在使用子类的对象来调用父类的方法。我们可以采取一种迂回的办法来操作父类的对象。那就是代理。代理该如何做呢？

```java
public class SpaceShip {
	void up(int x){
		System.out.println("向上"+x+"米");
	}

	void down(int x){
		System.out.println("向下"+x+"米");
	}
	void left(int x){
		System.out.println("向左"+x+"米");
	}
	void right(int x){
		System.out.println("向右"+x+"米");
	}
	void forward(int x){
		System.out.println("前进"+x+"米");
	}

}
```
代理类的创建
```java
public class SpaceShipDelegation {
	private String name;

	private SpaceShip ship =  new SpaceShip();

	protected SpaceShipDelegation(String name) {
		super();
		this.name = name;
	}

	void up(int x){
		ship.up(x);
	}

	void down(int x){
		ship.down(x);
	}
	void left(int x){
		ship.left(x);
	}
	void right(int x){
		ship.right(x);
	}
	void forward(int x){
		ship.forward(x);
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
	 SpaceShipDelegation delegation = new SpaceShipDelegation("代理");
	 delegation.forward(100);
	}

}
```
输出结果
```
前进100米
```
## 正确的清理垃圾

前边学习到了 Java 是有垃圾回收期自动清理垃圾的，并不需要我们手动的清理。但是垃圾回收期的清理动作是不可控的。如果我们想自己动手去清理一下垃圾就必须显式的去编写一些特定的回收动作。

```java
public class Shape {
	public Shape(int i) {
		System.out.println("开始绘画");
	}


	void disPose(){
		System.out.println("清理图画");
	}

}

```
画圆形
```java
public class Circle extends Shape{

	public Circle(int i) {
		super(i);
		// TODO Auto-generated constructor stub
		System.out.println("画圆形");
	}

	@Override
	void disPose() {
		// TODO Auto-generated method stub
		System.out.println("清除圆形");
		super.disPose();
	}

}

```
画方形
```java
public class Triangle extends Shape{

	public Triangle(int i) {
		super(i);
		System.out.println("画方形");
	}

	@Override
	void disPose() {
		System.out.println("清除方形");
		super.disPose();
	}

}
```
开始绘画
```java
public class CADSystem extends Shape{

	public CADSystem(int i) {
		super(i);

		circle = new Circle(i);
		triangle = new Triangle(i);
		// TODO Auto-generated constructor stub
	}

	private Circle circle;
	private Triangle triangle;

	@Override
	void disPose() {
		// TODO Auto-generated method stub
		circle.disPose();
		triangle.disPose();
		super.disPose();
	}
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		CADSystem cadSystem = new CADSystem(20);
		//重点来了
		try {

		} finally {
			// TODO: 无论什么情况下都必须执行、这样就使得我们自己控制了清除的顺序
			cadSystem.disPose();
		}

	}

}
```
执行结果
```
开始绘画
开始绘画
画圆形
开始绘画
画方形
清除圆形
清理图画
清除方形
清理图画
清理图画
```
我们写的每个类都继承自 Shape 类，而且都复写了 dispose() 方法。并运用 super 关键字调用该方法的父类版本。最后我们在 main() 方法中看到了 try _ finally 这两个关键字。finally 表示 try 代码块中的代码无论怎样都会执行 finally 中的方法。我们在此调用了清理方法。在清理过程中我们必须注意父类清理方法和子类清理方法的调用顺序。先清理子类然后在清理父类。如果我们使用垃圾回收器他可能使用它任何想要的顺序进行清理。另外如果要进行清理最好编写自己的清理方法，但不要使用 finalize()。

## 子类重载父类的方法

如果我们在子类中重载了父类的方法，那么这也是允许的。并且我们重载之后的方法不会对父类的方法产生任何影响。对上面的例子进行稍微的修改。

```java
public class Shape {
	public Shape(int i) {
		System.out.println("开始绘画");
	}


	void disPose(){
		System.out.println("清理图画");
	}

	void chocies(int x){
		System.out.println("重新执行"+x);
	}

}

```
子类继承之后重载
```java
public class Circle extends Shape{

	public Circle(int i) {
		super(i);
		// TODO Auto-generated constructor stub
		System.out.println("画圆形");
	}

	@Override
	void disPose() {
		// TODO Auto-generated method stub
		System.out.println("清除圆形");
		super.disPose();
	}

	void chocies(String  x) {
		// TODO Auto-generated method stub
		System.out.println("执行了重载"+x);
	}

}
```
调用方法

```java
public class CADSystem extends Shape{

	public CADSystem(int i) {
		super(i);

		circle = new Circle(i);
	}

	private Circle circle;

	@Override
	void disPose() {
		// TODO Auto-generated method stub
		circle.chocies("重载参数");
		circle.chocies(10);
		super.disPose();
	}
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		CADSystem cadSystem = new CADSystem(20);
		//重点来了
		try {

		} finally {
			// TODO: 无论什么情况下都必须执行、这样就使得我们自己控制了清除的顺序
			cadSystem.disPose();
		}

	}

}

```
执行结果
```
开始绘画
开始绘画
画圆形
执行了重载重载参数
重新执行10
清理图画
```
结果看到方法重载并不影响父类的方法。但是如果我们是复写呢？就必须一样了。复写方法使用 @Override 注解。如果不一样编译器就会报错。

![](/images/blog/fuxie.png)

## 组合和继承的选择

组合和继承都允许在新类中放置父类的对象，组合是显示的这样做，而继承是隐式的做。

组合技术通常是想在新类中使用现有类的功能而非接口。而继承是使用某个现有类来开发一个它的特殊的版本。这意味着你在使用一个通用类，并为了某种特殊需要将其特殊化。

## protected 关键字

在访问权限控制的学习的时候，就已经介绍过了这个关键字。我们在实际开发中可能希望某些方法对其他类进行隐藏起来，但仍然允许子类的成员对其进行访问。protected 关键字就起到这样的作用。任何继承与此类的子类和任何位于同一个包中的类都具有访问权限。

## 向上转型

为新的类提供方法并不是继承技术最重要的方面，其最重要的方面是用来表示新类和父类之间的关系。这种关系可以描述为：新类是父类的一种类型。

```java
public class Animals {

	static String name;

	protected Animals(String name) {
		super();
		this.name = name;
		// TODO Auto-generated constructor stub
	}

	static void eat(Animals animals,String s){
		System.out.println(animals.name+s);
	}

}
```
继承动物类并调用
```java
public class Dog extends Animals{

	protected Dog(String name) {
		super(name);
		// TODO Auto-generated constructor stub
	}

    public static void main(String[] args) {
	  Dog dog = new Dog("狗");
	  dog.eat(dog,"吃饭");
    }
}
```
执行结果
```
狗吃饭
```
我们看到 eat() 方法中需要传递一个 Animals 的对象引用。但是我们却传递了一个 Dog 的对象引用。编译器也没有报错，运行通过。这就说明 DOG 对象也是 animals 对象的一种。这种引用的动作，我们称呼为向上转型。

由于向上转型是由一个较为普通的类型向一个较为通用的类型转换，所以总是安全的。因为子类必须具备父类所包含的方法。在向上转型的过程中类接口中方法的丢失。

## 到底需要继承还是组合

在平时的程序开发中，为了使用一个类中的成员的方法，我们既可以使用组合技术创建一个对象的引用，也可以使用继承技术。但继承并不是常见的，我们更多的是显式的去创建一个对象。尽管在 OOP 编程中我们多次强调继承。我们应当慎用继承。那么在什么情况下适用继承呢？一个清晰额判断就是我们是否需要向上转型。

## final 关键字

final 意味着不可改变的。不可改变可能出于多种理由。final 主要用于以下三种情况中：数据、方法、和类。

#### final 数据

final 修饰数据来告诉编译器这一块数据是恒定不变的。数据的恒定不变主要用于以下情况。
 1. 一个永不改变的编译时常量
 2. 一个在运行时被初始化的值，而你不希望改变他。

对于编译器常量，在 Java 中必须是基本类型，并且以关键字 final 表示。 在对这个常量定义时必须对其赋值。

一个既有 final 又有 static 关键字修饰的常量只占据一段不能改变的存储空间。
有一个特殊的情况，当对象引用使用 final 修饰时，不能改变的是它的引用指向，而不是他的值。

![](/images/blog/final.png)

#### 空白 final

final 修饰的参数可以空白，但是必须在 构造器中进行初始化。这就意味着在使用之前必须初始化。
```java
public class Dog extends Animals{

    final  int x;
	final  Value VALUE;
	final static String NET = "ABC";


	protected Dog() {
		super();
		x = 3;
		VALUE = new Value();
		// TODO Auto-generated constructor stub
	}

	protected Dog(String name) {
		super(name);
		x = 3;
		VALUE = new Value();
		// TODO Auto-generated constructor stub
	}

    public static void main(String[] args) {


    }
}
```

**另外我在这里有一个问题？当你同时使用static 和 final 的时候就无法使用空白 fianl 这是什么原因？回忆一下之前的初始化与加载。下面的 「初始化及类的加载」会阐明这个问题**

#### final 参数

java 允许在参数列表中以声明的方式将参数指明为 final。这意味着你无法在方法中对参数进行更改。

![](/images/blog/finalmethod.png)

#### final 方法

使用 final 的原因是我们把方法锁定，以防止任何继承类修改这个方法。还有一个效率的原因，早期的 Java 虚拟机在发现 final 方法时，会根据自己的判断跳过插入程序代码机制，直接以方法中的实际代码为副本进行调用。这将消除方法调用的开销。在新的 Java SE5 之后就不在需要这样优化了。因为已经去掉了这个机制，效率问题应该让编译器和 jvm 来处理。

注意：private 修饰的方法都隐式的指定了 final的修饰。

锁定方法
```java
public class Abc {
	final void s(){

	}

}

```
尝试覆盖
```java
public class FinalMethod extends Abc{
    void s(){

	}
}
```

#### final 类

当你将某各类的整体定义为 final 时，就表明你不打算继承这个类，而且也不允许别人这么做。处于某种考虑你对这个类的设计用不需要做变动。

注意：final 类的变量可以根据自己的意愿选择是不是 final。不论类是否被定义为 final。然而 final 类不允许被继承，所以 final 类中的所有方法都隐式的指定为 final ，因此无法覆盖他们。

## 初始化及类的加载

程序是作为启动过程的一部分是立刻被加载的。然后才是初始化，紧接着程序开始运行。初始化的过程是仔细和严谨的，以确保 static 的东西其初始化顺序不会造成麻烦。

Java 就不会造成这个问题，因为他采用的是一种不同的加载方式。因为 Java 所有的事物都是对象。每个类的编译文件都存在于他自己的独立文件中。改文件只在需要使用代码事才会被加载。一般来说，类的代码只要在初次加载时才加载。这通常是指加载发生在创建类的第一个对象时，但是当访问 static 成员或者 static 方法时也会发生加载。初次使用之处，也是 static 初始化之处。所有的 static对象和 static 代码块都会在加载时依照程序中的顺序而依次初始化。当然定义为 static 的东西只会被初始化一次。所有上面的代码中为何静态的 final 不能再构造函数中初始化。因为可能静态的在没够初始化构造函数的时候就已经被先调用了。

#### 继承与初始化

如果一个类有父类，那么在加载的过程中父类将会先于初始化。以此类推，根基类中的被定义为 static 会被首先初始化，然后是在一个子类，以此类推。这种方式很重要因为子类成员的初始化可能依赖于父类某个成员的初始化。至此所有的类加载完毕之后，对象就可以被初始化了。首先对象中的所有基本类型都会被设定为默认值，对象会被设为 null。然后根类的构造器会被调用。在构造器初始胡完成之后，实例变量按照其次序进行初始化。
