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
