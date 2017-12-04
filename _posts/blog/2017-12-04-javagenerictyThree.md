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
