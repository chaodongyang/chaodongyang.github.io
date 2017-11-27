---
layout: post
title: java编程思想之字符串初步
categories: javaThinking
description: java编程思想之字符串初步
keywords: java, java字符串, java编程思想之字符串初步，java之字符串初步
---

字符串操作是计算机程序设计中最常见的行为，Java 程序中应用最广泛的类也是 String 类。

## 不可变 String
String 对象是不可变的。查看 JDK 可以得知。String 类中每一个看起来是修改 String值的方法。
其实都是创建一个新的 String 对象，以包含修改后的 String 内容。而最初的 String 是没有任
何改变的。

```java
public class Immutable {

	public static String upcase(String string) {
		return string.toUpperCase();
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		String a = "hoold";
		System.out.println(a);
		String qString = upcase(a);
		System.out.println(qString);
		System.out.println(a);
	}

}

```
执行结果：
```
hoold
HOOLD
hoold

```
当把 a 传递给 upcase() 方法的时候，实际传递的是一个引用的拷贝。而该引用所指的对象其实一直
还待在原地。参数 string 其实只有在方法运行的时候才存在。一旦结束就消失。当然 upcase() 的
返回值，其实只是最终结果的引用。这足以说明，upcase() 所返回的引用已经指向一个新的对象，而
原来的 a 还待在原地。

## 重载 “+” 与 StringBuilder
String 对象是不可变的，你可以给一个 String 对象增加任意多的别名。因为 String 对象具有只读性
，所以指向他的任何引用都不会改变他的值，因此也就不会对其他的引用有任何影响。

不可变性带来一定的效率问题。为 String 对象重载的 “+” 操作符就是例子。一个操作符被应用特定
的类时被赋予特殊的意义。Java 中仅有的两个重载的操作符就是 “+” 和 “+=” 号。Java 并不允许
我们重载任何的操作符。

```java
public class StringTest {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		String string = "chao";
		String string2 = "abc" + string + "def";
		System.out.println(string2);

	}

}

```
执行结果：
```
abcchaodef
```
这段代码是如何工作的呢？我们看下编译之后的文件：

![](/images/blog/stringClass.png)

我们看到了很熟悉的代码：
```java
invokevirtual java.lang.StringBuilder.append(java.lang.String) : java.lang.StringBuilder [25]
```
编译器自动引入了 StringBuilder 类。虽然我们没有在源代码中引入这个类。但编译器自己却使用了
他，因为他更高效。每个字符串都调用了 append() 方法。现在我们也许会认为我们可以随意的使用
String 对象，反正编译器会为我们自动优化性能。我们在看一下下面的例子：

```java
public class StringTest {

	public static String implicit(String[] fields) {
		String result = "";
		for (int i = 0; i < fields.length; i++) {
			result += fields[i];

		}
		return result;
	}

	public static String explicit(String[] fields) {

		StringBuilder builder = new StringBuilder();
		for (int i = 0; i < fields.length; i++) {
			builder.append(fields[i]);
		}
		return builder.toString();
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		String[] strings = {"a","b","c","d"};
		implicit(strings);
		explicit(strings);

	}

}
```
查看编译后的 implicit() 方法文件：

![] (/images/blog/stringimplicit.png)

查看编译后的 explicit() 方法文件：

![] (/images/blog/stringexplicit.png)

如果你懂汇编语言，你会看到第一个使用了 “+=” 操作的其实每次循环都为我们创建了一个全新的类 StringBuilder 。在循环内构造意味着每次循环都会创建一个。而 explicit() 方法我们直接自己创建了 StringBuilder 对象。只会创建一次。如果你已经知道字符串大小，那么显示的创建这个StringBuilder 类可以避免多次重新分配缓冲。因此，如果你在编写一个 toString() 方法时。如果字符串比较简单你可以完全依靠编译器，他会为你合理的构造字符串。如果你要在方法中使用循环，那么最好自己去创建一个 StringBuilder对象。

在第二个方法中我们其实是使用 append() 方法将字符串一点点的拼接起来的。如果你想走捷径 append(a+":" +c) 这样编译器就会陷入陷阱，从而为你创建一个新的对象。

StringBuilder 提供了丰富的方法:

| 方法        | 作用       |
| :----       |   -----:      |  
| append()        |  追加字符串内容        |
|  insert(int offset, int i)       | 把 i 插入第 offset 的位置           |
| replace(int start, int end, String str)                  |            替换 start - end 之间的字符为 str      |
|  substring(int start)         |     窃取从 start 开始的字符串并返回               |
|   reverse()           |    反转字符串并返回对象               |

StringBuilder 是 javaSE5 之后引入的，在这之前 Java 用的是 StringBuffer。后者是线程安全的，因此开销也会大一些。

## 无意识递归
Java 中的每个类从根本上继承自 Object ,容器类自然也不例外。因此容器类都有 toString() 方法。并且复写了该方法，使得生成的 string 结果能够表达自己。例如：ArrayList.toString() 。他会遍历 ArrayList 中包含的所有对象，调用每个元素的 toString() 方法。

如果你想用 toString() 打印内存地址：

```java
@Override
	public String toString() {
		// TODO Auto-generated method stub
		return "内存地址" + this;
	}
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		toString();

	}
```
执行结果：
```java
Exception in thread "main" java.lang.Error: Unresolved compilation problem:
	Cannot make a static reference to the non-static method toString() from the type StringTest

	at stringdemo.StringTest.main(StringTest.java:33)
```
这里发生了异常，为什么呢？因此 this 代表当前对象会调用 toString() 方法。于是发生了递归调用。如果真的想打印内存地址应该用 super.toString() 去使用 Object 基类的方法。

## String 上的操作

Java String 的 API 文档
https://docs.oracle.com/javase/8/docs/api/

## 格式化输出
JavaSE5 中推出了 C 语言中的 printf() 风格的格式化输出。使得开发者对输出格式与排列更加大的控制能力。

#### printf()
C 语言的 printf() 并不能像 Java 哪样拼接字符串，它使用了简单的格式化字符串，加上要插入的值。然后将其格式化输出。使用特殊的占位符来表示数据将来的位置。

```
printf( "Row 1: [%d %f]\n",x,y );
```
首先将 x 插入到 %d 的位置，然后将 y 的值插入到 %f 的位置。这些占位符称作格式修饰符。它不但说明了插入的位置，还说明了插入的类型，以及如何格式化。

#### System.out.format()
JavaSE5  后引入的 format()。模仿了 C语言的方法。

```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		int x = 5;
		double y = 3.1415926;
		System.out.format("Row 1:[" + x + "" + y +"]");
		System.out.format("Row 1: [%d %f]\n",x,y );


	}

```
输出结果：
```
Row 1:[53.1415926]
Row 1: [5 3.141593]
```
#### Formatter 类
在 Java 中所有的格式化功能都是由 Java.util.Formatter 类来处理。

```java
public class TurtleDemo {

	private String name;
	private Formatter formatter;

	protected TurtleDemo(String name, Formatter formatter) {
		this.name = name;
		this.formatter = formatter;
	}

	public void move(int x,int y) {
		formatter.format("%s 是好人 (%d,%d)\n", name,x,y);
	}
	public static void main(String[] args) {
		TurtleDemo demo = new TurtleDemo("我是", new Formatter(System.out));
		demo.move(10, 100);
	}

}
```
执行结果：
```
我是 是好人 (10,100)
```
#### 格式化说明符

直接上代码理解吧，比较复杂难记忆：
```java
public class Receipt {

	 private double total = 0;  
	    private Formatter f = new Formatter(System.out);    //指定输出的目的地  

	    public void printTitle()        //输出标题  
	    {  
	        //这个格式第一个%-15s 是说宽度为15的字符串，后面类同，- 表示对齐的方式在最左边表示左对齐  
	        f.format("%-15s %5s %10s\n", "Item", "Qty", "Price");  
	        f.format("%-15s %5s %10s\n", "----", "---", "-----");  
	    }  

	    public void print(String name, int qty, double price)  
	    {  
	        f.format("%-15.15s %5d %10.2f\n", name, qty, price);  
	        total += price;  
	    }  

	    public void printTotal()  
	    {  
	        f.format("%-15.15s %5s %10.2f\n", "Tax", "", total*0.06);  
	        f.format("%-15s %5s %10s\n", "", "", "------");  
	        f.format("%-15.15s %5s %10.2f\n", "Total", "", total*1.06);  
	    }  

	    public static void main(String [] args)  
	    {  
	        Receipt receipt = new Receipt();  
	        receipt.printTitle();  
	        receipt.print("Jack's Magic Beans", 4, 4.25);  
	        receipt.print("Princess Peas", 3, 5.1);  
	        receipt.print("Three Bears Porridge", 1, 14.29);  
	        receipt.printTotal();  
	        Formatter ff = new Formatter(System.out);  
	        ff.format("%5d", 998);  
	    }  
}

```
执行结果：
```
Item              Qty      Price
----              ---      -----
Jack's Magic Be     4       4.25
Princess Peas       3       5.10
Three Bears Por     1      14.29
Tax                         1.42
                          ------
Total                      25.06
  998
```
通过简洁的语法，Formatter 提供了对空格和对齐的强大控制力。

#### Formatter 转换
表格中包含了最常用的类型转换

| 类型转换字符  |     含义 |
| :----       |   -----:      |
| d        |  整数型 10 进制         |
| c        |Unicode 字符           |
| b        | boolean 值           |
| s        | String            |
| f        | 浮点数 10 进制           |
| e        | 浮点数科学计数           |
| x        | 整数 16 进制           |
| h        | 散列码 16 进制          |
| %        | 字符 %           |

示例代码：

```java
public class Coversion {

	static Formatter formatter = new Formatter(System.out);

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		char u = 'a';
		formatter.format("s: %s\n", u);
		formatter.format("c: %c\n", u);
		formatter.format("b: %b\n", u);
		formatter.format("h: %h\n", u);

		int v = 100;
		formatter.format("d: %d\n", v);
		formatter.format("c: %c\n", v);
		formatter.format("b: %b\n", v);
		formatter.format("s: %s\n", v);
		formatter.format("x: %x\n", v);
		formatter.format("h: %h\n", v);

	}

}

```
执行结果：
```
s: a
c: a
b: true
h: 61
d: 100
c: d
b: true
s: 100
x: 64
h: 64
```
注意：转换时注意数据类型，如果无效的转换会报错。另外 b 对于任意类型都是对的。但是如果不是
Boolean 类型的转换，只要结果不为空，结果永远都是 true。

#### String.format()
String.format() 是一个静态的方法，它接收和 Formatter.format() 一样的参数，但返回是一个
 String 的对象。这样更加的方便。

 在处理二进制文件时，我们常常希望以十六进制的格式看看内容。

 ```java
 public class StringFormat {

	public static String format(byte[] data) {
		StringBuilder builder = new StringBuilder();
		int n =0;
		for (byte b : data) {
			if (n % 16 ==0) {
				builder.append(String.format("%05x",n));
			}
			builder.append(String.format("%02x",b));
			n++;
			if (n % 16 ==0) {
				builder.append("\n");
			}
		}
		builder.append("\n");
		return builder.toString();

	}

	public static void main(String[] args) throws Exception{
		// TODO Auto-generated method stub
		byte[] bs = {1,2,3,4,5};
		if (args.length ==0) {
			System.out.println(format(bs));
		}
	}

}

 ```
 执行结果：
 ```
 000000102030405
 ```
