---
layout: post
title: java编程思想之访问权限控制
categories: javaThinking
description: java编程思想之访问权限控制
keywords: java, java访问权限, java权限控制，java访问权限控制
---

我们所开发的程序的某些部分往往需要我们不断重复的修改才能使其变得更加完善。如果你将一段代码经过一段时间之后在看，你会发现你会有更好的方式去实现它。这也正是重构的原动力之一。因此程序也能够得到更好的维护性。

## 简介

修改和完善代码也存在着一定的风险。通常情况下使用这些代码和方法的老的用户和客户端程序员需要你的代码尽可能的保持不变。因为你想修改代码，但是他们却不想修改代码。由此产生了一个问题，如何把变动的事物和保持不变的事物区分开来。对于开发类库而言尤为重要。类库的使用者必须依赖他所使用的这个类库，并且即使类库更新了新的版本，他们也并不需要更改代码。从开发者的角度来说，开发者必须有权限更改类库，并确保客户的代码不会受到影响。

这一目标可以通过约定来达到。比如，类库开发者必须在更改类库的同时不能删除现有的任何方法，因为那样会破坏客户端程序员的代码。但是开发者并不知道哪些方法和参数已经被开发者所使用了。这对于开发者来说是不是无法对任何事物进行改动呢。为了解决这个问题 Java 提供了访问权限控制修饰符，以供类库开发人员向客户端程序员指明哪些可用，哪些不可用。
访问权限控制的等级从最大到最小以此是：public、protected、包访问权限控制、private。作为一名类库的设计者，你会尽可能将一切方法都定义为 private。而仅仅公开你愿意向客户端提供的方法。

## 包：库单元

包是 Java 代码编辑的一个组成部分，它和它的名字一样其实就是一个包，把一部分Java源文件组织在一起，以便和其他的代码相互分离。

![](/images/blog/bao.png)

如果你想导入包中的类使用，你可以使用 import 关键字来实现。

![](/images/blog/importJava.png)

但是这个包中的其他类仍然是不可使用的，要想导入所有的类需要使用新的语法。

![](/images/blog/importAll.png)

我们之所以要使用包进行访问权限的控制，那是因为我们的项目中可能有相同的方法名，甚至有可能有相同的类名。那么我们如何区分这些相同的方法呢。这个时候为每个类提供唯一的标识符和命名空间正是解决此类问题的很好的办法。

当编写一个 Java 文件的时候，此文件就是编译的最小单元。后缀名为 .java。而在一个编译单元内有且仅有一个 public 类，该类的名称必须与文件名称相同。编译单元内可以有额外的类，但是外界是无法看见的，这是因为他不是 public 类，他们主要用于对 public 类提供支持。

#### 代码组织

java 中有一个文档文件叫 JAR。它会为我们生成 jar 文件以便其他人使用。类库实际上是一组类文件。每个文件都有一个 public 类。如何把这些类组织在不同的命名空间下。需要带代码的第一行使用 package 关键字。

![](/images/blog/package.png)

牢记：package 和 import 关键字允许你做的，是将单一的全局名字控件分隔开来，以便无论多少人使用都不会出现冲突。

#### 创建独一无二的包名

按照惯例，package 的名称的第一部分是类的创建者的域名。如果你遵循惯例，域名是第一无二的，因此你的包名也是独一无二的。当然如果你没有自己的域名你就需要构造一个不大可能和别人冲突的包名。如果你打算发布你的代码，申请一个域名还是很有必要的。

另外如果你将两个相同类名的类，以 .* 的形式导入，在你不使用这个类的时候编译器不会产生任何错误。但是当你需要使用的时候编译器就不能确定你到底需要的是哪个包下的类。这个时候需要你包名全路径后缀类名的方式。

```java
java.util.Vector v = new java.utile.Vector();
```

#### 定制工具库

```java
package constructor;

public class Print {

	public static void printlin(Object... arg) {
		// TODO Auto-generated method stub
		for (Object object : arg) {
			System.out.print(object);
		}

	}

}
```
静态导入

```java
import static constructor.Print.*;


public class JavaInitTest {
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		printlin("1");
	}

}
```
输出结果
```
1
```
一个简便的输出语句就完成了。

#### 条件编译

java 中没有条件编译的功能，该功能可以在你不改变任何代码的情况下，就能切换开关产生不同的行为。C 语言中主要用于解决跨平台的问题，但是 Java 本身就已经是跨平台的语言。这可能是 Java 不使用这个功能的原因吧。但是条件编译还是具有一些价值的。调试就是一个很常见的例子。调试功能在开发过程中是开启的，而在发布的产品中是禁止的。可以通过修改被导入的包的方法来实现这一目的，修改的办法是将你程序中的代码从调试版修改为发布版。

#### 使用包注意

无论何时创建包，都已经在给定包的名称的时候隐含的指定了目录的结构。这个包必须位于其名称所指定的目录之中。

## Java 访问全新修饰词

public、protected、private这几个访问权限修饰词在使用的过程中是置于类的每个成员之前的。无论它是变量、常量、方法、类。如果不提供任何的修饰符，则意味着是包访问权限。因此所有的元素都是具有不同的访问权限的。

```java
package constructor;

public class Print {
	public int x;
	private final static int NOT = 0;

	public static void printlin(Object... arg) {
		// TODO Auto-generated method stub
		for (Object object : arg) {
			System.out.print(object);
		}

	}

}

```

#### 包访问权限

假设在没有任何修饰符的情况下。包访问权限允许将包内所有相关的类组合起来，以使他们可以轻松的相互作用。当把类组织起来放在同一个包的时候，也就给他们包访问权限的成员赋予了相互访问的权限。这就意味着当前包中的所有其他类对其他类的成员都具有访问权限。但是，往往我们并不会这么做，我们也会使用其他的权限修饰符。在使用权限修饰符的情况下。取得对其他某成员的访问权限的唯一途径是：

1. 使该成员成为 public。无论是谁都可以访问。
2. 通过不加访问权限修饰符将其他类放入同一个包中。
3. 通过继承。继承的类既可以访问 public 也可以访问 protected成员。
4. 提供访问器方法，也称作 get、set 方法。以用来读取和设置值。

#### public 访问权限

public 关键字意味着最高权限，类和类中的成员对所有访问者可见。

```java
package JavaControlFlow;

public class Book {

	boolean checkOut ;

	public Book(boolean checkOut) {

		this.checkOut = checkOut;
	}

	void checkIn(){
		checkOut = false;
	}
}
```
调用者

![](/images/blog/public.png)

这里看到类创建了对象没有问题，但是类中的方法却没法访问，这是因为 Book 类的方法访问权限是非公开的。它只提供给类本身访问。那么如何也可以让其他类访问呢？那就需要把这两个类放进同一个包中。也就是上述提到的具有相同的包访问权限就可以了。

#### private 访问权限

关键字 private 的意思是，除了自身类之外，其他任何类都无法访问这个成员。即使是在同一个包中也是如此。为此，private 就允许我们可以任意修改该成员，而不必考虑这样做是否会影响其他包内的类。

举个例子，来说明一下 private 的重要性：

```java
public class Book {

	static boolean checkOut ;

	private Book() {

	}

	public static Book getBook(){
		checkOut = false;
		return new Book();
	}
}
```
创建 Book 对象
```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		Book book = Book.getBook();
		System.gc();
	}
```
此时我们就可以控制如何创建对象，并阻止别人通过构造器来创建对象。

#### protected 访问权限

关键字 protected 处理的是继承的概念，通过继承可以利用一下现有的类，我们称之为基类，里边所有的方法和成员都赋予新的类同样的方法和成员。

我们看下面这个例子。

```java
package JavaControlFlow;

public class Book {

	boolean checkOut ;

	public Book() {

		this.checkOut = checkOut;
	}

	void checkIn(){
		checkOut = false;
	}

	public void checkOut(){
		checkOut = true;
	}

}

```
新的类继承与 Book
```java
package constructor;

import JavaControlFlow.Book;

public class NewBook extends Book{

	public void f(){

	}
}

```
通过继承类调用 基类 Book 中的方法

![](/images/blog/extenBook.png)

我们可以看到 checkIn() 方法是无法调用的，这是因为它是非 public 权限的方法，新的继承类和基类又不在同一个包内。那么我们要向让继承类访问这个方法该如何做呢，那么肯定会有人想到可以放在同一个包中。那么问题来了，放在同一个包中自然包中的其他类也具有了访问权限。我们想仅仅让新的继承类访问如何做呢？此时就用到了 protected 关键字。

![](/images/blog/extendAll.png)

修改访问权限

```java
package constructor;

public class Book {

	boolean checkOut ;

	public Book() {

		this.checkOut = checkOut;
	}

	protected void checkIn(){
		checkOut = false;
	}

	public void checkOut(){
		checkOut = true;
	}

}
```
调用 checkIn() 方法

```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		NewBook book = new NewBook();
		book.checkOut();
		book.checkIn();

	}

```

## 接口和实现

注意这里的接口并非 Java 程序中的接口内容。而是对访问权限的一种理解概念。访问权限的控制通常会使我们的类成员和方法得到有效的隐藏。把数据和方法包装进类中，以及具体实现的隐藏通常称之为封装。

访问权限控制出于两个很重要的原因。第一个，访问权限将权限的边界划在了数据类型的内部。这样就划归了客户端程序员可以访问的和不可以访问的权限。第二个，接口和具体实现的分离。客户端程序员可以向接口发送消息之外什么也不可以做，不可以修改任何内容。

## 类的访问权限

在 java 中访问修饰符也可以修饰类名。用来定义这些类对库中的那些类可以使用。为了控制某各类的访问权限，修饰词必须出现在 class 之前。

```java
public class JavaConstrutor {}
```
这里有一些额外的限制

1. 每个类都只能有一个 public 类。
2. public 类的名称必须和文件名一样。
3. 完全不带 public 的类也是有可能的。这种情况下可以随意对文件命名。
4. 类的访问权限只有 public 和 包访问权限。如果想不可以让别人对类又任何访问权限，可以将该类的所有构造器都指定为 private。

## 总结

创建一个类库，也与该类库的用户建立了某种关系，这些用户就是客户端程序员，他们是另外一些程序员，他们将你的类库聚合为一个应用，或是运用你的类库来创建一个更大的类库。如果不指定规则，客户端程序员就可以对类的所有成员进行随心的操作，这种情况下所有的事物都是公开的。

注意：访问权限控制专注于类库创建者和类库使用者之间的关系，这种关系也是一种通信方式。然而，假如你自己编写了所有的代码，或者你在一个项目中和成员一起工作，所有的东西都放在同一个包中。这就是另外一种情况了，因此严格的遵守访问权限并不是必须的。
