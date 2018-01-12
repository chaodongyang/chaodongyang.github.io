---
layout: post
title: Java8 新语法习惯 (级联 lambda 表达式)
categories: java8新语法习惯
description: Java8 新语法级联 lambda 表达式
keywords: java,java8新语法,JDK8新语法,java8新语法级联 lambda 表达式
---

在函数式编程中，函数既可以接收也可以返回其他函数。函数不在像传统的面向对象编程一样，只是一个对象的工厂或生成器，它也能够创建和返回另一个函数。返回函数的函数可以变成级联 lambda 表达式，代码非常短。尽管这样的语法初次看起来非常的陌生，但是它有自己的用途。本文将帮助您认识级联 lambda 表达式，理解它们的性质和在代码中的用途。

## 神秘的语法
看下面的一端代码：
```java
x -> y -> x > y
```
对于不熟悉使用 lambda 表达式编程的开发人员，此语法可能看起来像货物正在从快速行驶的卡车上一件件掉下来。这非常的令人感到好奇。幸运的是我们不会经常看到他们，但理解如何创建级联 lambda 表达式和如何在代码中理解它们会是很有好处的。

## 高阶函数
在谈论级联 lambda 表达式之前，有必要首先理解如何创建它们。对此，我们需要回顾一下高阶函数和它们在函数分解中的作用，函数分解是一种将复杂流程分解为更小、更简单的部分的方式。

首先区分一下高阶函数和常规函数的规则：

常规函数

- 可以接收对象
- 可以创建对象
- 可以返回对象

高阶函数

- 可以接收函数
- 可以创建函数
- 可以返回函数

开发人员可以将匿名函数或 lambda 表达式传递给高阶函数，以让代码简短且富于表达。让我们看看这个高阶函数的两个示例。

**一个接收函数的函数**

在 Java 中，我们使用函数接口引用 lambda 表达式和方法引用。下面这个函数接收一个对象和一个函数：
```java
public static int totalSelectedValues(List<Integer> values,
  Predicate<Integer> selector) {

  return values.stream()
    .filter(selector)
    .reduce(0, Integer::sum);  
}
```
totalSelectedValues 的第一个参数是集合对象，而第二个参数是 Predicate 函数接口。 因为参数类型是函数接口 (Predicate)，所以我们现在可以将一个 lambda 表达式作为第二个参数传递给 totalSelectedValues。例如，如果我们想仅对一个 numbers 列表中的偶数值求和，可以调用 totalSelectedValues，如下所示：
```java
totalSelectedValues(numbers, e -> e % 2 == 0);
```
假设我们现在在 Util 类中有一个名为 isEven 的 static 方法。在此情况下，我们可以使用 isEven 作为 totalSelectedValues 的参数，而不传递 lambda 表达式：
```java
totalSelectedValues(numbers, Util::isEven);
```
作为规则，只要一个函数接口显示为一个函数的参数的类型，您看到的就是一个高阶函数

**一个返回函数的函数**

函数可以接收函数、lambda 表达式或方法引用作为参数。同样的，函数也可以返回 lambda 表达式或方法引用。在此情况下，返回类型将是函数接口。

让我们首先看一个创建并返回 Predicate 来验证给定值是否为奇数的函数：
```java
public static Predicate<Integer> createIsOdd() {
  Predicate<Integer> check = (Integer number) -> number % 2 != 0;
  return check;
}
```
为了返回一个函数，我们必须提供一个函数接口作为返回类型。在本例中，我们的函数接口是 Predicate。尽管上述代码在语法上是正确的，但它可以更加简短。 我们使用类型引用并删除临时变量来改进该代码：
```java
public static Predicate<Integer> createIsOdd() {
  return number -> number % 2 != 0;
}
```
这是使用的 createIsOdd 方法的一个示例：
```java
Predicate<Integer> isOdd = createIsOdd();

isOdd.test(4);
```
## 创建可重用的函数
现在已经大体了解高阶函数和如何在代码中找到他们，我们可以考虑使用他们来让代码更加简短。

设想我们有两个列表 numbers1 和 numbers2。假设我们想从第一个列表中仅提取大于 50 的数，然后从第二个列表中提取大于 50 的值并乘以 2。
```java
List<Integer> result1 = numbers1.stream()
  .filter(e -> e > 50)
  .collect(toList());

List<Integer> result2 = numbers2.stream()
  .filter(e -> e > 50)
  .map(e -> e * 2)
  .collect(toList());
```
此代码很好，但您注意到它很冗长了吗？我们对检查数字是否大于 50 的 lambda 表达式使用了两次。 我们可以通过创建并重用一个 Predicate，从而删除重复代码，让代码更富于表达：
```java
Predicate<Integer> isGreaterThan50 = number -> number > 50;

List<Integer> result1 = numbers1.stream()
  .filter(isGreaterThan50)
  .collect(toList());

List<Integer> result2 = numbers2.stream()
  .filter(isGreaterThan50)
  .map(e -> e * 2)
  .collect(toList());
```
通过将 lambda 表达式存储在一个引用中，我们可以重用它，这是我们避免重复 lambda 表达式的方式。如果我们想跨方法重用 lambda 表达式，也可以将该引用放入一个单独的方法中，而不是放在一个局部变量引用中。

现在假设我们想从列表 numbers1 中提取大于 25、50 和 75 的值。我们可以首先编写 3 个不同的 lambda 表达式：
```java
List<Integer> valuesOver25 = numbers1.stream()
  .filter(e -> e > 25)
  .collect(toList());

List<Integer> valuesOver50 = numbers1.stream()
  .filter(e -> e > 50)
  .collect(toList());

List<Integer> valuesOver75 = numbers1.stream()
  .filter(e -> e > 75)
  .collect(toList());
```
尽管上面每个 lambda 表达式将输入与一个不同的值比较，但它们做的事情完全相同。如何以较少的重复来重写此代码？

## 创建和重用 lambda 表达式
尽管上一个示例中的两个 lambda 表达式相同，但上面 3 个表达式稍微不同。创建一个返回 Predicate 的 Function 可以解决此问题。

首先，函数接口 ```Function<T, U>``` 将一个 T 类型的输入转换为 U 类型的输出。例如，下面的示例将一个给定值转换为它的平方根：
```java
Function<Integer, Double> sqrt = value -> Math.sqrt(value);
```
在这里，返回类型 U 可以很简单，比如 Double、String 或 Person。或者它也可以更复杂，比如 Consumer 或 Predicate 等另一个函数接口

在本例中，我们希望一个 Function 创建一个 Predicate。所以代码如下：
```java
Function<Integer, Predicate<Integer>> isGreaterThan = (Integer pivot) -> {
  Predicate<Integer> isGreaterThanPivot = (Integer candidate) -> {
    return candidate > pivot;
  };

  return isGreaterThanPivot;
};
```
引用 isGreaterThan 引用了一个表示 ```Function<T, U>``` 或更准确地讲表示 ```Function<Integer, Predicate<Integer>>``` 的 lambda 表达式。输入是一个 Integer，输出是一个 ```Predicate<Integer>```。在 lambda 表达式的主体中（外部 ```{}``` 内），我们创建了另一个引用 isGreaterThanPivot，它包含对另一个 lambda 表达式的引用。这一次，该引用是一个 Predicate 而不是 Function。最后，我们返回该引用。

isGreaterThan 是一个 lambda 表达式的引用，该表达式在调用时返回另一个 lambda 表达式 — 换言之，这里隐藏着一种 lambda 表达式级联关系。

现在，我们可以使用新创建的外部 lamba 表达式来解决代码中的重复问题：
```java
List<Integer> valuesOver25 = numbers1.stream()
  .filter(isGreaterThan.apply(25))
  .collect(toList());

List<Integer> valuesOver50 = numbers1.stream()
  .filter(isGreaterThan.apply(50))
  .collect(toList());

List<Integer> valuesOver75 = numbers1.stream()
  .filter(isGreaterThan.apply(75))
  .collect(toList());
```
在 isGreaterThan 上调用 apply 会返回一个 Predicate，后者然后作为参数传递给 filter 方法。

## 保持简单的秘诀
我们已从代码中成功删除了重复的 lambda 表达式，但 isGreaterThan 的定义看起来仍然很杂乱。幸运的是，我们可以组合一些 Java 8 约定来减少杂乱，让代码更简短。

可以使用类型引用来从外部和内部 lambda 表达式的参数中删除类型细节：
```java
Function<Integer, Predicate<Integer>> isGreaterThan = (pivot) -> {
  Predicate<Integer> isGreaterThanPivot = (candidate) -> {
    return candidate > pivot;
  };

  return isGreaterThanPivot;
};
```
接下来，我们删除多余的 ()，以及外部 lambda 表达式中不必要的临时引用：
```java
Function<Integer, Predicate<Integer>> isGreaterThan = pivot -> {
  return candidate -> {
    return candidate > pivot;
  };
};
```
可以看到内部 lambda 表达式的主体只有一行，显然 ```{}``` 和 return 是多余的。让我们删除它们：
```java
Function<Integer, Predicate<Integer>> isGreaterThan = pivot -> {
  return candidate -> candidate > pivot;
};
```
现在可以看到，外部 lambda 表达式的主体也只有一行，所以 ```{}``` 和 return 在这里也是多余的。在这里，我们应用最后一次重构：
```java
Function<Integer, Predicate<Integer>> isGreaterThan =
  pivot -> candidate -> candidate > pivot;
```
现在可以看到 — 这是我们的级联 lambda 表达式。

## 理解级联 lambda 表达式
我们通过一个适合每个阶段的重构过程，得到了最终的代码 - 级联 lambda 表达式。在本例中，外部 lambda 表达式接收 pivot 作为参数，内部 lambda 表达式接收 candidate 作为参数。内部 lambda 表达式的主体同时使用它收到的参数 (candidate) 和来自外部范围的参数。也就是说，内部 lambda 表达式的主体同时依靠它的参数和它的词法范围或定义范围。

大体上讲，当您看到两个向右箭头时，可以将第一个箭头右侧的所有内容视为一个黑盒：一个由外部 lambda 表达式返回的 lambda 表达式。

## 总结
级联 lambda 表达式不是很常见，但您应该知道如何在代码中识别和理解它们。当一个 lambda 表达式返回另一个 lambda 表达式，而不是接受一个操作或返回一个值时，您将看到两个箭头。


感谢 Venkat Subramaniam 博士

Venkat Subramaniam 博士站点：http://agiledeveloper.com/
