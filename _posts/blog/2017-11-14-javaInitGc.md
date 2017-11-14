---
layout: post
title: java编程思想之初始化与清理
categories: javaThinking
description: java编程思想之初始化与清理
keywords: java, java初始化, java垃圾回收，java初始化与清理
---

安全的编程方式是编程代价高昂的主要原因之一。初始化和清理正式涉及安全的两个问题。忘记初始化变量或者初始化构建都会造成程序错误，或者忘记清理一个元素，这个元素所占用的资源就会一直得不到释放，结果是资源用尽。C++ 中采用了构造器的概念，Java 中也采用了构造器，并额外提供了垃圾回收。对于不在使用的资源，垃圾回收能将其自动释放。

## 用构造器确保初始化

我们可以为编写的每一个类提供 init() 初始化的方法。该方法用于提醒用户使用时应该首先调用 init() 方法。这也意味着用户必须记得调用此方法。在 Java 中通过提供构造器，类的设计者可以保证每个对象都会得到初始化。创建对象时，如果其类具有构造器，java 就会在用户操作之前自动调用构造器，从而保证了初始化的进行。构造器的命名：第一，所取的名字必须不能喝类的其他成员变量重名。第二，调用构造器是编译器的责任，所以必须让编译器知道应该调用那个方法。在 Java 中采用构造器和类名相同的命名方式。

```java
public class ConstrutorTest {

	public ConstrutorTest() {
		System.out.println("test");
	}

}

```
```java
public class JavaConstrutor {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		for (int i = 0; i < 5; i++) {
			new ConstrutorTest();
		}
	}

}
```
```
test
test
test
test
test

```
现在，在创建对象时调用 new ConstrutorTest() 方法，将会为对象分配存储空间，并调用相应的构造器。这就确保了你在操作对象之前它就已经被初始化。不接受任何参数的构造器叫无参构造器。但是和其他方法一样，构造器也能带有形式参数，以便指定如何创建对象。

```java
public class ConstrutorTest {

	public ConstrutorTest(int i) {
		System.out.println("test"+i);
	}

}

```

```java
public class JavaConstrutor {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		for (int i = 0; i < 5; i++) {
			new ConstrutorTest(i);
		}
	}

}
```
```
test0
test1
test2
test3
test4
```

有了构造器的形式参数，就可以在初始化对象时提供实际的参数。构造器有助于减少错误，并使得代码更容易阅读。从概念上讲，「初始化」与 「创建」是彼此独立的，然而在上面的代码中，却找不到初始化方法的明确调用。在 Java 中初始化和创建是捆绑在一起的，两者不能分离。构造器是一种特殊类型的方法，因为他没有返回值。对于返回值为 void 的方法虽然返回值为空，但仍可选择让他返回别的东西。构造器则不会返回任何东西。但 new 表达式返回了对对象的引用。

## 方法重载

在日常生活中相同的词可以表达不同的含义。特别是含义相差很小时，这种方式十分有用。那么在程序语言中我们如何用同一个方法名表示不同的含义呢？这就要用到方法的重载。我们在 Java 语言中，构造器是强制重载方法名的一个原因。既然构造器的名字已经由类名所决定，就只能有一个构造器名称。那么想要用多种方式创建一个对象该怎么办呢？这就需要两个构造器：一个无参构造器，一个有参构造器。这种方法名相同而参数不同的方法，就叫做方法的重载。

```java
public class ConstrutorTest {

	public ConstrutorTest() {
		System.out.println("test");
	}

	public ConstrutorTest(int i) {
		System.out.println("test"+i);
	}

	void info(){
		System.out.println("无参方法");
	}

	void info(int x){
		System.out.println("有参方法"+x);
	}

}
```
```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		for (int i = 0; i < 2; i++) {
			ConstrutorTest construtorTest = new ConstrutorTest(i);
			construtorTest.info();
			construtorTest.info(i);
		}

		new ConstrutorTest();
	}
```
```
test0
无参方法
有参方法0
test1
无参方法
有参方法1
test
```
#### 区分重载方法

方法相同的名称，Java 如何才能知道你指的是哪一个？其实很简单，每个重载的方法都有一个独一无二的参数类型列表。甚至参数顺序的不同也足以区分两个方法。不过，一般情况下别这么做，因为这会使代码难以维护。

#### 涉及基本类型的重载

基本类型能从一个较小的类型自动提升至一个较大的类型，此过程一旦涉及到重载，可能会造成一些混淆。下面的例子中你会发现常数 5 会被当做 int 类型来处理，所以有某个重载方法接受 int 类型的值，它就会被调用。如果传入的数据类型小于方法中声明的数据类型，实际数据类型就会被提升。char 类型略有不同，如果无法找到恰好接受 char 类型的参数的方法，就会把 char 直接提升为 int 类型。如果传入的实际参数大于重载方法声明的参数类型，就得通过类型转换进行窄化转换。

```java
public class PrimitLoading {

	void f1(char x){
		System.out.print("f1 char");
	}
	void f1(byte x){
		System.out.print("f1 byte");
	}
	void f1(short x){
		System.out.print("f1 short");
	}
	void f1(int x){
		System.out.print("f1 int");
	}
	void f1(long x){
		System.out.print("f1 long");
	}
	void f1(float x){
		System.out.print("f1 float");
	}


	void f2(int x){
		System.out.println("---"+"f2 int");
	}
	void f2(long x){
		System.out.println("---"+"f2 long");
	}
	void f2(float x){
		System.out.println("---"+"f2 float");
	}


	void test1(){
		int x =5;
		System.out.print("int_5:");
		f1(x);
		f2(x);
	}

	void test2(){
		byte x = 0;
		System.out.println("byte_x:");
		f1(x);
		f2(x);
	}

	void test3(){
		char x = 'x';
		System.out.println("char_x:");
		f1(x);
		f2(x);
	}

	void test4(){
		double x = 0;
		System.out.println("double_x:");
		f1((float)x);
		f2((float)x);
	}
}

```
```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		PrimitLoading pLoading = new PrimitLoading();
		pLoading.test1();
		pLoading.test2();
		pLoading.test3();
		pLoading.test4();
	}
```
```
int_5:f1 int---f2 int
byte_x:
f1 byte---f2 int
char_x:
f1 char---f2 int
double_x:
f1 float---f2 float
```

#### 以返回值区分重载方法

是否可以通过方法的返回值来区分重载的方法呢？看下面的两个例子：

> void f() {}

> int f() {return 1;}

只要编译器可以通过语境明确的判断出语义，比如 int x = f() 中，那么的确可以区分此方法。但现实是我们调用方法的时候很可能并不关心返回值，我们想要的只是调用方法的效果，此时 Java 就不能判断该调用哪一个。因此，根据方法的返回值是不能区分重载方法的。

## 默认构造器

如前边的代码，默认构造器就是没有参数的构造器——他的作用就是创建一个默认的对象。如果你的类中没有构造器，则编译器会自动帮你创建一个默认的构造器。但是，如果你已经创建了一个构造器，无论这个构造器有没有参数，编译器都不会帮你自动去创建默认的构造器。这就好比，如果你没有提供任何构造器，编译器会认为你需要一个构造器，但假设你写了一个构造器，编译器会认为你自己写了一个，你自己知道需要做什么。

## this 关键字

this 关键字在 Java 中表示对当前对象的引用。this 关键字只能在方法内部使用，表示调用方法的那个对象的引用。但是要注意，如果在方法内部调用同一个类的另一个方法，就不必使用 this，直接调用即可。只有当明确需要指出对当前对象的引用时，才需要 this 关键字。例如，当需要返回对当前对象的引用时，就常常在 return 语句里写。另外，this 关键字对于将当前对象传递给其他方法也很有用。

```java
public class Leaf {
	int i = 0;

	Leaf increment(){
		i++;
		//返回当前的对象
		return this;
	}

	void print(){
		System.out.println(i);
	}

}

```
```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		Leaf leaf = new Leaf();
		leaf.increment().increment().increment().print();;
	}
```
this 关键字传递给方法
```java
public class Apple {
	Apple getPeel(){
		return Peel.peel(this);
	}

}
```
```java
public class Peel {
	static Apple peel(Apple apple){
		return apple;
	}

}
```
#### 在构造器中调用构造器

有时我们为一个类写了多个构造器，可能我们想在一个构造器中调用另一个构造器。以避免重复代码，我们可以 this 关键字来解决。在构造器中如果为 this 添加了参数列表，那么就表示对符合参数列表的构造器的明确调用。这样，调用其他构造器就有明确的途径了。

```java
static int s = 0;
public ConstrutorTest() {
	this(s);
	System.out.println("test");
}

public ConstrutorTest(int i) {
	this.s = i;
	System.out.println("test"+i);
}

void info(){
	this(s);
}
```
注意：尽管可以用 this 调用一个构造器，但却不能调用两个。此外，必须将构造器的调用置于最起始处，否则编译器会报错。最后一个 info() 方法表明，编译器禁止在其他任何地方调用构造器。

#### static 的含义

static 方法就是没有 this 的方法。在 static 方法的内部不能调用非 static 方法。反过来倒是可以的。因为 static 方法代表已经初始化会在内存中分配空间，非 static 方法表示未被初始化需要调用的对象才能初始化。所以一个在内存空间的怎么能调用一个未初始化的方法呢？反过来倒是可以的，而且可以在没有创建任何对象的前提下调用 static 方法。仅仅通过类本身来调用。这实际上正式 static 方法的主要用途。由于不存在 this。所以不是通过向对象发送消息的方式来完成的。如果在代码中出现大量的 static 就需要重新考虑自己的设计了。

## 清理：终结处理和垃圾回收

我们都了解初始化的重要性，但常常会忘记同样重要的清理工作。但在使用程序库时，把一个对象用完之后弃之不顾的做法并非是安全的。当然，Java 有垃圾回收器负责回收无用对象占据的内存资源。但也有特殊情况：假定你的对象并非是通过 new 关键字来创建的获得了一个特殊的内存区域，由于垃圾回收器只负责那些由 new 分配的内存，所以它不知道该如何释放这段内存。为了应对这种情况，Java 允许在类中定义一个名为 finalize() 的方法。它的工作原理是这样的：一旦垃圾回收器准备好释放对象所占用的内存空间，将首先调用 finalize() 的方法，并在下一次垃圾回收动作发生时，才会真正回收对象所占用的内存。所以使用 finalize() 就能在垃圾回收时刻做一些重要的清理工作。

在 C++ 中对象一定会被销毁，在 Java 中对象却并非总被垃圾回收。

1. 对象可能不被垃圾回收

2. 垃圾回收并不等于析构。

也许你会发现，只要程序没有濒临存储空间用完的那一刻，对象占用的控件就总也得不到释放。如果程序执行结束，并且垃圾回收器一直也没回收你所创建的任何对象的存储空间，则随着程序的退出，那些资源也会全部交还给操作系统。这个策略是恰当的，因为垃圾回收本身也有开销，要是不使用它，那就不会支付这部分开销。

#### finalize 的用途何在

通过以上的解释我们发现 finalize 方法也并不能真正的及时清理垃圾。使用垃圾回收器的唯一原因是为了回收程序不在使用的内存。所以对于垃圾回收的任何行为来说，他们也必须同内存及其回收相关。无论对象是如何创建的，垃圾回收器都会负责释放对象所占据的内存。这就将 finaize 的使用的情况限制到一种特殊的情况，即通过创建对象以外的特殊形式为对象分配了空间并不能由垃圾回收机制直接回收。这时才需要我们使用 finalize 方法进行回收。Java 中一切皆为对象。这种情况主要发生在使用 「本地方法」时候，本地方法是一种在我们的 Java 语言中调用非 Java 语言的一种方式。本地方法目前只支持 C 和 C++。也许你会调用 C 的 malloc() 函数来分配存储空间，除非调用 free() 函数，否则存储空间将得不到释放，从而造成内存的泄漏。所以此时我们就需要在 finalize() 中用本地方法调用它释放内存。

#### 你必须实施清理

既然 finalize 不是进行普通清理工作的合适场所。那么普通的清理该如何执行呢？如果要清理一个对象用户必须调用执行清理动作的方法。这似乎很简单，但却与 C++ 中的析构函数的概念相冲突。在 C++ 中所有对象都会被销毁，如果对象是局部对象那么对象的作用域就在右边花括号的结尾处。如果对象是 new 创建的，当 C++ 调用 delete 时就会执行相应的析构函数。如果程序员忘记调用 delete 这样就会出现内存泄漏，对象的其他部分也会得不到清理。相反，Java 必须使用 new 来创建对象。在 Java 中也没有释放对象的 delete 方法。因为垃圾回收器会帮你释放存储空间。但是垃圾回收器并不能完全的代替析构函数。如果希望进行除释放存储空间之外的清理工作，还是得明确的调用适当的 Java 方法。这就等同于使用析构函数。最后，无论是垃圾回收还是终结，都不保证一定会发生。如果 Java 虚拟机并未面临内存耗尽的情形，它是不会浪费时间去执行垃圾回收以恢复内存的。

#### 终结条件

通常我们并不能指望 finalize，必须创建其他的方法进行清理。不过 finalize 还有一个别的用法，它并不依赖于每次都要对 finalize 进行调用，这就是对象终结条件的验证。

当我们对某一个对象不感兴趣的时候也就是它可以被清理了。这个对象应该处于某种状态，使他战友的内存可以被安全的释放。以下简单的例子示范：

```java
public class Book {

	boolean checkOut = false;

	public Book(boolean checkOut) {

		this.checkOut = checkOut;
	}

	void checkIn(){
		checkOut = false;
	}

	@Override
	protected void finalize() throws Throwable {
		// TODO Auto-generated method stub
		if (checkOut) {
			System.out.println("错误：checkOut");
			//super.finalize();
		}

	}
```
```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		Book book = new Book(true);

		book.checkIn();

		new Book(true);

		System.gc();
	}
```
```
错误：checkOut
```

本例的终结条件是：所有的 Book 对象在被垃圾回收之前都要被 checkIn()。但是 main 方法中 new Book(true) 之后没有操作 checkIn() 方法。所以我们在进行终结的时候发现错误，打出了结果。如果没有 finalize() 来验证终结条件将很难发现这个错误。

#### 垃圾回收器如何工作

在堆上分配对象的代价是十分高昂的，因此我们或许认为 Java 中所有的对象都分配到堆上的方式也是十分高昂的。然而垃圾回收器对于创建对象的速度却有明显的效果。存储空间的释放竟然会影响存储空间的分配？但这确实是某些 Java 虚拟机的工作方式。

在某些 Java 虚拟机中，堆的实现更像一个传送带，没分配一个对象，它就向前移动一格。这就意味着对象存储空间的分配速度非常快。Java 的堆指针只是简单的移动到尚未分配的区域。其效率可以和 C++ 在堆栈上分配空间的速度媲美。

其实 Java 中的堆未必完全像传送带哪样工作。如果哪样的话势必会造成频繁的页面调动，因此会造成比实际需要更多的内存。页面调度会显著的影响性能，最终在创建足够多的对象后内存也将会耗尽。秘密就在于垃圾回收器的介入。当它工作时，将一面回收空间，一面使得堆中的对象紧凑的排列。这样通过垃圾回收器的重新排列，实现了一种高速、有无限存储空间的模型。

引用计数式垃圾回收方式：引用计数是一种简单但速度很慢的回收技术。每个对象都含有一个引用计数器，当他有引用连接对象时，引用计数加 1.当引用为空时，引用计数减 1。这个方法有个缺陷，当对象存在循环引用的时候，对象该被回收却计数器不为 0 的现象。对于回收器而言查找这些交互引用的工作量极大。这种回收方法似乎未被任何 Java 虚拟机采用。

在一些更快的模型中垃圾回收期并非使用的是计数计数。他们依据的思想是：对任何活的对象一定能最终追述到其存活的堆栈或静态存储区之中的引用。这个引用链条可能会传过数个引用层次。由此，如果从堆栈或者静态存储区开始，遍历所有的引用，就能找到所有活的对象。在这种情况下 Java 虚拟机将采用一种自适应的垃圾回收技术。置于如何找到活的对象不同的虚拟机有不同的实现。

```
停止复制
```

**这意味着先暂停程序的运行，然后将所有的活的对象从当前堆栈复制到另一个堆栈，没有被复制的全部都是垃圾。当对象呗复制到新堆栈时，它们是一个挨着一个的，所以新堆栈保持紧凑排列，然后就可以简单的直接分配新的控件。对于这种复制式回收器而言效率会降低。有两个原因。首先，得有两个堆栈，然后还要在两个分离的堆栈之间来回的折腾，从而得需要维护比原来多一倍的控件。第二个问题在于复制。程序进入稳定状态后，可能只会产生少量的垃圾甚至不产生垃圾，复制回收器仍然会把所有的内存复制一遍到另一处，这很浪费。**

```
标记-清扫
```

**标记清扫所依据的思路同样是从堆栈和静态存储区出发，遍历所有的引用，进而找出所有的存活的对象。每当它找到一个存活的对象，就会给对象设置一个标记，这个过程不会回收任何对象。只有全部标记工作完成之后清理动作才开始。在清理过程中，没有标记的对象将会被释放，不会发生任何复制的动作。所以剩下的堆空间是不连续的，垃圾回收器如果需要得到连续的空间，就得重新整理剩下的对象。**

java 虚拟机中有很多技术用以提升速度。尤其是与加载器操作有关的，被称为「即时编译技术」。这种技术可以把程序全部或部分编译成机器码，程序运行速度因此得到提升。当需要装载某各类时，编译器会先找到它的 .class 文件，然后将该类的代码装入内存。有两种方案：一种是编译所有的代码。这种做法时间周期长，增加代码的长度。另一种是惰性评估，只在必要的时候才编译代码。新版的 JDK 中 Java HotSpot 技术就采用类似方法。代码每次执行过程中都进行一次优化，执行次数越多，它的速度越快。
