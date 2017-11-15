---
layout: post
title: java编程思想之初始化与清理(深入)
categories: javaThinking
description: java编程思想之初始化与清理
keywords: java, java初始化, java垃圾回收，java初始化与清理
---

上一节学习了 java 对象的初始化以及一些关键字的解释，了解了垃圾回收器是如何工作的。那么 Java 中的成员、数据是如何进行初始化的呢？又有哪些初始化的方式？

## 成员初始化

Java 给我们尽可能的保证所有的变量在使用前都被初始化。对于方法内的局部变量如果未被初始化，Java 就以编译时错误来提示我们初始化。

![](/images/blog/notInit.png)

如果是类的数据成员是基本类型，Java 就会为类的每个基本类型成员赋值一个初始值。

```java
public class JavaInitTest {

	static int i;
	static char x;
	static short s;
	static byte b;
	static long l;
	static double d;
	static float f;

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		System.out.println(i);
		System.out.println(x);
		System.out.println(s);
		System.out.println(b);
		System.out.println(l);
		System.out.println(d);
		System.out.println(f);
	}

}
```
```
0

0
0
0
0.0
0.0
```
char 的值为 0,所以显示为空白。如果在类里边定义一个对象引用时，如果不将其初始化，此引用会获得一个特殊的值 null。这会引起我们经常会看到的空指针异常。

#### 指定初始化

如果要为某个变量赋值，该怎么做？很直接的办法就是在定义类成员变量的地方为其赋值。

```java
static int i =0;
static char x = 'a';
static short s = 0;;
```
也可以用同样的方法初始化非基本类型的对象。

```java
public class JavaConstrutor {
	 Book book = new Book(true);
}

```
甚至可以调用某个方法来进行赋值

```java
public class JavaInitTest {

	static int i;

	static int f(){
		return 11;
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub

		i = f();

		System.out.println(i);

	}

}
```
## 构造器初始化

可以使用构造器进行初始化。在运行时刻，可以调用方法或者执行某个动作来确定初值，这为编程带来更大的灵活性。但要牢记：无法阻止自动初始化的进行，他将在构造器调用之前就发生。

```java
public class Book {

	boolean checkOut;

	public Book(boolean checkOut) {

		this.checkOut = checkOut;
	}

}

```
在这里 checkOut 会先有一个初始值 false。然后才会调用构造器为其赋值。对于所有基本类型和对象引用，包括在定义时已经指定初始值的变量，这种情况都成立的。因此，编译器不会强制你一定要在构造器的某个部分对其进行初始化。

#### 初始化的顺序

在类的内部变量的先后顺序决定了初始化的先后顺序。即使变量分散定义域方法之间，他们仍然会在任何方法包括构造器方法调用之前被初始化。

```java
public class JavaInitTest {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		House house = new House();
		house.f();

	}

}

class Windows{

	public Windows(int mark) {
		System.out.println("windows"+mark);
	}

}

class House{
	Windows windows = new Windows(1);

	public House() {
		System.out.println("调用 house构造器");
		windows3 = new Windows(33);
	}

	void f(){
		System.out.println("调用方法");
	}
	Windows windows2 = new Windows(2);

	Windows windows3 = new Windows(3);


}

```
```
windows1
windows2
windows3
调用 house构造器
windows33
调用方法
```
#### 静态数据的初始化

无论创建多少个对象，静态数据都只占用一份存储区。static 关键字不能应用于局部变量，因此它只能作用与域。初始化的赋值和非静态是一样的。稍微改一下上面的代码。

```java
public class JavaInitTest {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		House house = new House();
		house.f();

	}

}

class Windows{

	public Windows(int mark) {
		System.out.println("windows"+mark);
	}

	void f1(int mark){
		System.out.println("windows f1"+mark);
	}

}

class House{
	Windows windows = new Windows(1);

	public House() {
		System.out.println("调用 house构造器");
		windows = new Windows(33);
		windows.f1(10);
	}

	void f(){
		System.out.println("调用house的方法");
	}
	 Windows windows2 = new Windows(2);

	 static  Windows windows3 = new Windows(3);


}
```
```
windows3
windows1
windows2
调用 house构造器
windows33
windows f110
调用house的方法
```
初始化的顺序改变了。初始化的顺序是先初始化静态的对象，而后初始化非静态的对象。

总结一下对象初始化的过程

1. 即使没有显示的使用 static 关键字，构造器实际上也是静态的方法。因此当首次创建一个对象时，Java 解释器必须先查找类的路径，以定位 .class 文件。
2. 然后载入 .class 文件，有关静态初始化的所有动作都会被执行。因此静态初始化只有在对象的 .class 文件首次加载时进行一次。
3. 当使用 new 创建对象时，首先会在堆上为对象分配足够的存储空间。
4. 这块存储空间会被首先清零，这就自动的将所有的对象引用致为 null。所有的基本类型致为 0。
5. 执行所有出现于字段定义处的初始化动作。
6. 执行构造器。

#### 静态块

java 允许多个静态初始化动作组成一个特殊的 “静态子句” 。

```java
public class JavaInitTest {
	static House house = new House();
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		System.out.println("开始初始化");
		House.windows.f1(10);
	}

}

class Windows{

	public Windows(int mark) {
		System.out.println("windows"+mark);
	}

	void f1(int mark){
		System.out.println("windows f1"+mark);
	}

}

class House{
	static Windows windows;
	static Windows windows2;

	static{
		windows = new Windows(1);
		windows2 = new Windows(2);
	}
	public House() {
		System.out.println("调用 house构造器");
	}

}

```
```
windows1
windows2
调用 house构造器
开始初始化
windows f110

```
我们看到，我们即使执行了两次初始化动作，输出的结果也只有一次。表明静态初始化的动作只进行一次。另外，静态块里边的两个 Windows 对象都将会被初始化。

#### 非静态实例初始化

非静态实例的初始化，也可以用类似于静态块的处理方式进行初始化。

```java
public class JavaInitTest {
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		System.out.println("开始初始化");
		new House();
		new House(1);
	}

}

class Windows{

	public Windows(int mark) {
		System.out.println("windows"+mark);
	}

	void f1(int mark){
		System.out.println("windows f1"+mark);
	}

}

class House{
	 Windows windows;
	 Windows windows2;

	{
		windows = new Windows(1);
		windows2 = new Windows(2);
	}
	public House() {
		System.out.println("调用 house构造器");
	}
	public House(int x) {
		System.out.println("调用带参数的 house构造器");
	}

}
```
```
开始初始化
windows1
windows2
调用 house构造器
windows1
windows2
调用带参数的 house构造器
```
看起来和静态子句的初始化很相似，只不过少了 static 关键字。可以看到无论调用那个构造方法初始化操作都会发生。从结果可以看出，实例初始化字句是在构造器之前执行的。

## 数组初始化

数组是相同类型的、用一个标识符名称封装到一起的一个对象序列或基本类型数据序列。数组的定义有两种格式

```java
int[] al;
int al1[];
```
现在我们拥有的只是数组的一个引用，而没有给数组对象本身分配存储空间。为了给数组对象分配存储空间我们必须对数组进行初始化。

```java
int[] al = {1,2,3,4,5};
```
如果我们对一个数组进行赋值，其实就是对这个数组赋值引用。两个数组将会使用同一个引用。他们其中一个值进行了修改，另外一个也将会同步修改。

```java
public class ArrayTest {

	static int[] al = {1,2,3,4,5};

	static int[] a2;
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		a2 = al;
		for (int i = 0; i < a2.length; i++) {
			a2[i] = a2[i] + 1;
		}

		System.out.println(Arrays.toString(al));
	}

}
```
```
[2, 3, 4, 5, 6]
```
如果在编写程序时，并不能确定在数组里有多少个元素，可以直接用 new 在数组里创建元素。尽管只能创建基本类型的数组。

```java
static int[] a2;

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		a2 = new int[10];
		System.out.println(Arrays.toString(a2));
	}
```
```
[0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
```
如果你创建的是一个非基本类型的数组，那么你就创建了一个引用数组。即使你在 new 创建数组之后，它还是一个引用数组，直到创建新的对象，并且把对象赋值给数组，初始化进程才算结束。因为基本类型默认就会初始化，而对象不会默认初始化。如果忘记创建对象，并且试图用数组中的空引用做操作，就会产生运行时异常。

```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		Integer[] a2 = new Integer[5];
		System.out.println(Arrays.toString(a2));
	}
```
```
[null, null, null, null, null]
```
那么该如何正确的初始化对象数组呢？有两种方式：

```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		Integer[] a1 = new Integer[]{new Integer(0),new Integer(1),2};
		Integer[] a2 = {new Integer(0),new Integer(1),2};
		System.out.println(Arrays.toString(a1));
		System.out.println(Arrays.toString(a2));
	}
```
```
[0, 1, 2]
[0, 1, 2]
```
#### 可变参数列表

上面的方法提供了一个方便的语法来创建对象并赋值，以此获得与 C 语言可变参数列表一样的效果。由于所有的类都直接或间接的继承与 Object 类，所以我们可以这样写。

```java
public class ArrayTest {

	static void print(Object[] obj){
		for (Object object : obj) {
			System.out.print(object+"  ");
		}
		System.out.println("----------");;
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub

		print(new Object[]{new Integer(0),new Float(1.2f),new Double(2.23)});

		print(new Object[]{"ont","two","three"});
		A a = new A();
		print(new Object[]{1,2,3,a});
	}

}
class A{

}
```
```
0  1.2  2.23  ----------
ont  two  three  ----------
1  2  3  JavaInit.A@15db9742  ----------
```
然而我们可以不必这么写，java 为我们提供了可变参数列表的特性

```java
public class ArrayTest {

	static void print(Object... obj){
		for (Object object : obj) {
			System.out.print(object+"  ");
		}
		System.out.println("----------");;
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub

		print(new Object[]{new Integer(0),new Float(1.2f),new Double(2.23)});
		print(new Object[]{"ont","two","three"});
		A a = new A();
		print(new Object[]{1,2,3,a});
		print((Object[])new Integer[]{1,2,3,4,5});
		print();
	}

}
class A{

}

```
```
0  1.2  2.23  ----------
ont  two  three  ----------
1  2  3  JavaInit.A@15db9742  ----------
1  2  3  4  5  ----------
----------
```
请注意这段代码和上段代码的区别，这段代码在参数中使用的是类型三个点的形式来代表。当你在方法中填充数据时，编译器实际上会为你填充数组。你得到的实际上还是一个数组，所以我们可以使用 foreach 来迭代。倒数第二行，我们传入了一个 Integer[] 的数组。其实我们不进行类型转换依然可以通过的，只不过会有黄色的警告。因此，如果你有一组事物，可以当做参数列表来传递；如果你已经有一个数组，改方法可以把他当做一个可变参数列表来接受。最后一行表示我们把 0 个参数传递给可变参数列表也是可行的，当具有可选的尾随参数时，这一特性就很有用。

```java
public class ArrayTest {

	static void print(Integer x,Object... obj){
		for (Object object : obj) {
			System.out.print(object+"  ");
		}
		System.out.println(x+"----------");;
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub

		print(1,new Object[]{"ont","two","three"});
		print(2,(Object[])new Integer[]{1,2,3,4,5});
		print(3);
	}

}
```
```
ont  two  three  1----------
1  2  3  4  5  2----------
3----------
```
上面的例子我们可以注意到，即使我们不进行强制类型转换，可变参数列表依然可以运行，这就是可变参数列表与自动包装机制和谐共处的原因。然而这种机制将会使得方法的重载变动复杂。

```java
public class ArrayTest {

	static void print(char... c){
		for (Character object : c) {
			System.out.print(object+"  ");
		}
		System.out.println("----------");;
	}

	static void print(int... obj){
		for (int object : obj) {
			System.out.print(object+"  ");
		}
		System.out.println("----------");;
	}

	static void print(long... obj){
		for (long object : obj) {
			System.out.print(object+"  ");
		}
		System.out.println("----------");;
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		print(3);
		print(1,2);
		print(10l);
		print('a','b','c');
		print();

	}

}
```
```
3  ----------
1  2  ----------
10  ----------
a  b  c  ----------
----------
```
在每一种情况下，编译器都会使用自动包装机制来匹配重载的方法，然后调用最明确匹配的方法。但在最后一个不使用参数时，编译器就无法得知该调用那个方法了。如果我们此时给方法加一个非可变参数就可以解决这个问题了。但是这依然是个不可忽略的问题，所以我们应该只在重载方法的一个版本上使用可变参数列表，或者不使用它。

## 枚举

在 Java SE5 中有一个很小的特性，即 enum 关键字，我们现在可以很方便的使用枚举。

枚举的定义，枚举类型的实例都是常量，所以必须全部大写。

```java
enum Spicess{
	ONE,TWO,THREE,FOR
}
```
这里创建了一个枚举类型，它具有 4 个具名值。为了使用枚举，我们需要创建一个该类型的引用，并将其赋值给某个实例。

```java
public class EnumTest {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Spicess spicess = Spicess.ONE;
		System.out.println(spicess);
	}

}

enum Spicess{
	ONE,TWO,THREE,FOR
}
```
```
ONE
```
在创建 enum 时，编译器会自动添加一些属性。比如 toString() 方法，以便很好的显示某个枚举实例的名称，所以上面的打印输出了名称 ONE。还有 ordinal() 方法，用来表示某个枚举实例的声明顺序。以及 value() 方法，用来按照常量的声明顺序生成相应的数组。

```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		for (Spicess s : Spicess.values()) {
			System.out.println(s+"__"+s.ordinal());
		}

	}
```
```
ONE__0
TWO__1
THREE__2
FOR__3
```
重要的来了，enum 可以在 switch 语句中使用。

```java
public class EnumTest {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		switchTest(Spicess.TWO);
	}

	public static void switchTest(Spicess spicess) {
		switch (spicess) {
		case ONE:
			System.out.println("ONE");
			break;
        case TWO:
        	System.out.println("TWO");
			break;
        case THREE:
        	System.out.println("THREE");
	        break;
        case FOR:
        	System.out.println("FOR");
	        break;
		default:
			System.out.println("默认");
			break;
		}
	}

}

enum Spicess{
	ONE,TWO,THREE,FOR
}
```
## 总结

初始化在 Java 中占有至关重要的位置。C 语言中大量的变成错误都源于不正确的初始化。这种错误很难发现，并且不恰当的清理也会导致类似问题。Java 中的构造器能保证正确的初始化和清理，所以也很安全。

在 Java 中垃圾回收器会自动的释放内存，所以在很多场合下，类似的清理方法在 Java 中就不太需要了。Java 的垃圾回收器可以极大的简化编程的工作，而且在处理内存的时候也更安全。然而，垃圾回收器也确实增加了运行时的开销。而且 Java 解释器从来就很慢，所以这种开销造成多大影响很难得知。随着时间的推移，Java 在性能方面已经取到很大的进步，但速度问题仍然是它涉足某些特定编程领域的障碍。
