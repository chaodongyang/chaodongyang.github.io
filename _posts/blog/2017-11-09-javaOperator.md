---
layout: post
title: java编程思想之操作符详解
categories: javaThinking
description: Java 操作符使用详解
keywords: java, java操作符
---

在最底层，java 中的数据是通过操作符来操作的。Java 是建立在 C++ 基础之上的，所以 C 和 C++ 程序员会非常熟悉 java 的大多数操作符。当然 Java 也做了一些改进和简化。

## 使用 java 操作符

操作符接受一个或多个参数，并生成一个新值。参数的形式与普通的方法调用不同，但效果是相同的。加号 (+)、减号 (-)、乘号 ```(*)```、除号 (/) 以及赋值号 (=) 的用法和其他变成语言是类似的。

几乎所有的操作符都只能操作 「基本类型」。但是 = 、 == 、 != ,可以操作所有的对象。除此以外，String 类支持 + 和 += 。

## 优先级

当一个表达式中存在多个操作符时，操作符的优先级就决定了各部分的计算顺序。和我们数学中的优先级是一样的，先乘除后加减，有括号的先计算括号中的值。
``` java
public class JavaOperator {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		int x = 1, y = 2 , z =3;
		int a = x + y - 2/2 +z;
		int b = x + (y -2)/(2 +z);

		System.out.println("a = " + a + "b = " +b);

	}

}
```
输出结果：** a = 5 b = 1 **

请注意，输出语句中的 (+) 号操作符。在这种上下文环境中，(+) 号意味着 “字符串的连接”，并且如果必要他还执行字符串的转换。规则如下：当编译器观察到一个 String 后面紧跟着一个 (+) 号，而这个 (+) 号后面又紧跟着一个非 String 类型的元素时，就会尝试着将这个非 String 类型的参数转换为 String 类型。正如输出中将 a 和 b 从 int 成功的转为了 String。

## 赋值

赋值操作符使用 (=) 号。它的意思是取右边的值把它赋值给左边。右边可以是任何常量，变量，或者表达式。但左边必须是一个明确的变量。也就是说，必须有一个明确的物理空间可以存储等号右边的值。举例: ** a = 4 ** 。
但是不能把任何东西赋值给一个常数，也就是说常数 4 不能作为左边的值。

对于基本数据类型的赋值是很简单的。基本类型存储了实际的数值，而并非指向一个对象的引用。所以在为其赋值时是直接将一个地方的内容复制到了另外一个地方。所以两个值是互相不受影响的。但是在为对象赋值时，情况就不一样了。对一个对象操作时，我们操作的不是对象本身，而是对象的引用。所以如果将一个对象赋值给另外一个对象，实际上是将这个对象的引用赋值给另外一个对象。此时两个对象指向的是同一个引用。

```java
class NObj {
	int a;
}

public class JavaOperator {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		NObj n1 = new NObj();
		NObj n2 = new NObj();

		n1.a = 0;
		n2.a = 1;
		System.out.println("n1.a = " +n1.a + "n2.a =" + n2.a);

		//赋值
		n1 = n2;
		System.out.println("n1.a = " +n1.a + "n2.a =" + n2.a);

		n1.a = 20;
		System.out.println("n1.a = " +n1.a + "n2.a =" + n2.a);


	}

}

```
```
n1.a = 0   n2.a =1
n1.a = 1   n2.a =1
n1.a = 20  n2.a =1
```
原本 n1 包含的对象的引用 0 在对 n1 赋值的时候就被覆盖了，也就丢失了。而这个不在被引用的 0 对象引用就会被垃圾回收期自动的清理。这种特殊的现象被称为“别名现象”。

#### 方法调用中的别名问题

讲一个对象传递给方法时，也会产生别名问题。

```java
public class JavaOperator {

	static void f(NObj n){
		n.a = 100;
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		NObj n1 = new NObj();
		n1.a = 0;
		System.out.println("n1.a = " +n1.a);

		//赋值
		f(n1);
		System.out.println("n1.a = " +n1.a);		

	}

}
```
```
n1.a = 0
n1.a = 100
```
## 算数操作符

java 的基本算数操作符和大多数程序设计语言是相同的。但是 java 也使用一种 C 和 C++ 的简化符号同时进行运算和赋值操作。这种操作符后面紧跟着一个 = 号来表示，它对于 Java 中的所有操作符都适用。只要有其实际意义就可以。例如：要将 x 加 4，并将结果赋值给它，可以这么写 ``` x+=4 ```。

#### 一元加、减操作符

一元减号 (-) 和 一元加号 (+) 与二元减号和加号都是用相同的符号。根据表达式的书写格式，编译器会自动判断是那种操作符。

```
x = -a;

x = a * -b;
```
一元减号用于转变数据的符号，而一元加号只是为了与一元减号相对应。但是它唯一的动作仅仅是将比较小的类型的操作数提升为 int 。

## 自动递增和递减

Java 提供了大量的快捷运算符。这些快捷运算使得编码更方便，同时也使得编码更容易阅读，但有时也可能读起来更困难。
递增和递减运算符是两种很不错的运算符。其中递减运算符是 「--」，意味着减少一个单位。递增运算符是 「++」,意味着增加一个单位。这两个操作符还有两个不同的操作方式，通常为前缀式和后缀式。前缀式会先执行运算在生成值，后缀式会先生成值，在执行运算。

```java
public class JavaOperator {


	public static void main(String[] args) {
		// TODO Auto-generated method stub
		int a = 0;
		System.out.println("第一次a等于： " + a);

		System.out.println("++a: " + ++a);
		System.out.println("第二次a等于： " + a);

		System.out.println("a++: " + a++);
		System.out.println("第三次a等于： " + a);

		System.out.println("--a: " + --a);
		System.out.println("第四次a等于： " + a);

		System.out.println("a--: " + a--);
		System.out.println("第五次a等于： " + a);
	}
}
```
```
第一次a等于： 0
++a: 1
第二次a等于： 1
a++: 1
第三次a等于： 2
--a: 1
第四次a等于： 1
a--: 1
第五次a等于： 0
```
## 关系操作符

关系操作符生成的是一个 boolean 结果，计算的是操作数之间的关系。如果关系是真实的，表达式会生成 true ；如果关系是假的，表达式会生成 false 。关系操作符包括小于 (<) 、大于 (>)、小于或等于 (<=)、大于或等于 (>=)、等于 (==) 以及不等于 (!=)。等于和不等于适合于操作所有的基本数据类型，而其他的比较不适用 Boolean 类型，因为 true 和 false 比较大于小于没有实际意义。

#### 测试对象的等价性

关系操作符中 (==) 和 (!=) 适用于所有对象，下面看一个例子。

```java
public static void main(String[] args) {

		Integer n1 = new Integer(20);
		Integer n2 = new Integer(20);

		System.out.println(n1 == n2);
		System.out.println(n1 != n2);
	}
```
```
false
true
```

数值是相同的怎么会输出这样的结果呢，是因为 (==) 和 (!=) 对比的是对象的引用地址，而不是对象的实际值。这两个是 new 出来的不同对象。所以引用地址是不一样的。如果我们想比较两个对象的实际值该怎么办呢？必须使用特殊的方法 equals()。但这个方法不适用于基本类型，基本类型直接用上边的两个就可以。

```java
System.out.println(n1.equals(n2));
true
```
## 逻辑操作符

逻辑操作符 “与” (&&) 、“或” (||) 、“非” (!) 能根据参数的逻辑关系，生成一个 boolean 值。

```java
public static void main(String[] args) {

		Random random = new Random(20);
		int i = random.nextInt(100);
		int j = random.nextInt(100);

		System.out.println("i=:" + i);
		System.out.println("j=:" + j);
		System.out.println("i>j=:" + (i>j));
		System.out.println("i<j=:" + (i<j));
		System.out.println("i>=j=:" + (i>=j));
		System.out.println("i<=j=:" + (i<=j));

		System.out.println("i==j=:" + (i==j));
		System.out.println("i!=j=:" + (i!=j));

		System.out.println("i > 10 && j >10 =:" + ((i>10) && (j>10)));
		System.out.println("i < 10 || j <10 =:" + ((i>10) || (j>10)));

	}
```

```
i=:53
j=:36
i>j=:true
i<j=:false
i>=j=:true
i<=j=:false
i==j=:false
i!=j=:true
i > 10 && j >10 =:true
i < 10 || j <10 =:true

```
“与” 、"或" 、“非” 只可用于 boolean 值，不可直接用于基本类型。

#### 短路

当使用逻辑运算符时，我们会遇到一种 “短路” 现象。即一旦能够明确无误的计算整个表达式的值，就不在计算表达式余下的部分。因此整个表达式靠后的部分可能不会被计算到。如果所有的逻辑表达式都有一部分不必计算，那将获得潜在的性能提升。

```java
public static void main(String[] args) {

		boolean b = test1(0) && test2(5) && test3(2);
		System.out.println(" 结果没执行3" + b);

	}

	static boolean test1(int i){
		System.out.println("执行1");
		return i<1;
	}

	static boolean test2(int i){
		System.out.println("执行2");
		return i<2;
	}

	static boolean test3(int i){
		System.out.println("执行3");
		return i<3;
	}
```
```
执行1
执行2
 结果没执行3false
```
## 直接常量

如果在程序中使用直接常量，编译器可以准确的知道要生成什么样的类型，有时候确实模棱两可的。如果发生这种情况，必须对编译器加以指导，用于直接常量相关的字符来额外增加一些信息。

```
int i = 0x2f;
int b = 0177;
char s = 0xffff;
long n1 = 200L;
double d1 = 2D;
```
#### 指数计数法

java 采用了一种很不直观的计数法来表示指数，例如

```java
float f = 1.39e-43f;
double d = 470e-47;

System.out.println("指数结果" + f);
System.out.println("指数结果" + d);

指数结果1.39E-43
指数结果4.7E-45
```

注意：e 代表 10 的次方幂。看到 1.39e-43f 代表 1.39 乘以 10 的 -43 次幂。

## 三元操作符 if-else

三元操作符也称之为条件操作符，它显得比较特殊，因为它有三个操作数，但却是操作符的一种，因为它最终会产生一个值。采取的形式：

> boolean exp ? value0 : value1

如果 exp 结果为 true，就计算 value0 ，如果为 false ，就计算 value1。它的结果也会是操作符最终产生的值。和普通的 if else 语句很相似，但三元操作符更简洁。

```Java
public static void main(String[] args) {

		System.out.println(test1(5));
		System.out.println(test2(5));

	}

	static int test1(int x){
		return x < 10 ? x*10 : x/10;
	}

	static int test2(int x){
		if (x <10) {
			return x*10;
		}else {
			return x/10;
		}
	}
```
```
50
50
```
## 字符串操作符 + 和 +=

这两个操作符用于连接不同的字符串。字符串的连接有一个规则：如果表达式是以一个字符串开头，那么后续所有操作数都必须是字符串类型。编译器会把表达式后的序列自动转换成字符串类型。

```Java
public static void main(String[] args) {

		int x =0,y =1, z =2;
		String s = "abc";

		System.out.println(s + x + y +z);

		s += "edf";

		System.out.println(s + (x + y +z));

		System.out.println("" + x);


	}
```
```
abc012
abcedf3
0
```

## 类型转换操作符

类型转换。在适当的时候 java 会将一种数据类型自动的转换为另一种数据类型。假如我们为浮点类型赋值整数类型。编译器会将 int 自动转换为 float。类型转换允许我们进行显示的类型转换，或者进行强制的类型转换。

```Java
int i = 200;
long lng = (long)i;
lng = i;

long lng2 = (long)300;
lng2 = 200;
i = (int)lng2;
```
注意：这里可能会引入多余的转换，例如，编译器在必要的时候会自动进行 int 值到 long 值的提升。但是你仍然可以做这样的多余的事，以提醒自己需要留意。java 中类型转换是一种比较安全的操作。然而，如果要执行一种名为窄化转换的操作。更多信息数据的类型转换为无法容纳那么多信息的类型，就有可能面临信息丢失的可能。此时，编译器会强制要求我们进行类型转换。而对于扩展转换，而不必进行显示的类型转换，因为新类型肯定能容纳原来类型的信息，不会造成任何信息的丢失。java 允许我们把任何的基本数据类型转换为别的基本数据类型。boolean 类型不允许进行任何类型的转换。

#### 截尾和舍入

在执行窄化转换时，必须注意截尾和舍入的问题。例如，将一个浮点型转换为整类型，java 会如何处理呢？

```Java
public static void main(String[] args) {

	double above = 0.8, below = 0.2;

	float fabove = 0.8f, fbelow = 0.2f;

	System.out.println("int above" + (int)above);
	System.out.println("int below" + (int)below);
	System.out.println("int fabove" + (int)fabove);
	System.out.println("int fabove" + (int)fbelow);

	}
```
```
int above 0
int below 0
int fabove 0
int fabove 0
```
答案是转换为整数时总是对数字截尾。如果想取得四舍五入的结果呢？就需要使用 lang 包中的 Math 类的 round() 方法。

```java
System.out.println("int above" + Math.round(above));
System.out.println("int below" + Math.round(below));
System.out.println("int fabove" + Math.round(fabove));
System.out.println("int fabove" + Math.round(fbelow));
```
```
int above1
int below0
int fabove1
int fabove0
```

## 操作符小结

如果对基本类型执行算术运算或按位运算，大家会发现，只要是比 int 类型小的类型，比如，char、byte、short。那么在运算前，这些值都会自动转换为 int。这样最终生成的结果就是 int 类型。如果想把结果赋值给较小的类型，就必须进行类型转换。通常表达式中最大的数据类型决定了结果的数据类型。

java 中的数据类型在所有的机器中的大小都是相同的。所以，我们不必考虑移植的问题。

对于 boolean 值我们只能赋予它 true 或者 false。如果两个足够大的 int 类型进行乘法运算，结果会溢出 int 类型的范围。但是编译器也不会出现错误或者警告信息，运行时也不会产生异常。

另外 java 中还有专门用于二进制位操作和位移操作的运算符。
