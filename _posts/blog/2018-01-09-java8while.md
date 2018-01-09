---
layout: post
title: Java 8 新语法习惯 (传递表达式的替代方案)
categories: java8新语法习惯
description: Java8 新语法传递表达式的替代方案
keywords: java,java8新语法,JDK8新语法,java8新语法传递表达式的替代方案
---

Lambda 表达式广泛用在函数式编程中，但它们很难阅读和理解。在许多情况下，lambda 表达式存在只是为了传递一个或多个形参，最好将它替换为方法引用。在本文中，将学习如何识别代码中的传递 lambda 表达式，以及如何将他们替换为相应的方法引用。方法引用的使用需要学习，但是长期收益将会大于你的付出。

## 传递 lambda 表达式是什么？
在函数式编程中常常传递 lambda 表达式作为匿名函数，使用 lambda 作为高阶函数的实参。

示例我们将 lambda 表达式传递给 filter 方法：
```java
public class LambdaDemo {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
		numbers.stream()
		.filter(e -> e % 2 == 0)
		.forEach(e -> System.out.println(e));
	}

}

```
在这段代码中，我们将一个 lambda 表达式传递给 forEach 方法。尽管这两个 lambda 表达式明显具有不同的作用，但它们之间还有另一个重要的细微区别：第一个 lambda 表达式执行了实际的工作，而第二个没有。传递给 forEach 方法的 lambda 表达式就是我们所称的```传递 lambda 表达式```。表达式 ````e -> System.out.println(e)``` 将它的形参作为实际参数传递给了 println 方法。这个表达式没有任何的错误，但是它的语法相对于这个任务而言过于复杂。为了理解 ```(parameters) -> body``` 的用途，我们需要进入 body (也就是表达式右侧) 来查看这个形参发生了什么变化。如果 lambda 表达式没有对该形参执行任何的操作，则努力就是白费的。

在这种情况下将 lambda 表达式替换为方法引用会比较有益。不同于方法的调用，方法引用指的是我们传递形参的方法。使用方法引用也会带来各种各样的形参传递方式。

重写前面的代码，通过方法引用传递形参：
```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
		numbers.stream()
		.filter(e -> e % 2 == 0)
		.forEach(System.out::println);
	}
```
执行结果和上面的例子一样。使用方法引用会减少理解代码的工作，随着编写和阅读更多的代码好处会倍增。

## 传递形参作为实参
下面我们将讨论传递 lambda 表达式的变形，将介绍如何将每种表达式替换为方法引用。

**实例方法的实参**

lambda 表达式将其形参作为实参传递给实例方法，这非常的常见。在上面的例子中，形参 ```e``` 作为实参传递给 println 方法。在变形版本中我们将这个 lambda 表达式替换为方法引用 ```System.out::println```。

下面的图展示了 lambda 表达式将形参作为实参传递给实例方法：

![](/images/blog/20180109101023.png)

如果不熟悉方法引用，查看这样的 lambda 表达式能够帮助理解它的结构和形参传递到何处。要将 lambda 表达式更改为方法引用，只需删除通用部分(形参和实参)，并在方法调用上将点替换为冒号。

**this 上的一个方法实参**

前面的例子传递表达式的一种特殊情况是，在当前方法的 context 实例上调用实例方法。假设我们有一个名为 Example 的类，其中包含一个实例方法 increment:
```java
public class Example {
  public int increment(int number) {
    return number + 1;
  }

  //...
}
```
现在假设我们有另一个实例方法，我们在其中创建了一个 lambda 表达式并将它传递给 stream 的 map 方法，如下：
```java
.map(e -> increment(e))
```
这可能不是那么的显而易见，但是这个代码结构非常类似于上一个示例。我们都将形参作为实参传递给实例方法。稍微重写此代码，以让相似性更加明显：
```java8
.map(e -> this.increment(e))
```
通过引入 this 做为对 increment 的调用，让表达式的结构变动更加清晰。我们再次以方法引用的方式重写这个代码：
```java
.map(this::increment)
```
与将 ```e -> System.out.println(e)``` 替换为 ```System.out::println``` 非常相似的是，可以将 lambda 表达式 ```e -> increment(e)（或更准确地讲 e -> this.increment(e)）``` 替换为 ```this::increment```。在两种情况下，代码都更加清晰。

**静态方法的实参**

上面的例子我们替换的 lambda 表达式将一个形参作为实参传递给实例方法。也可以替换将形参传递给静态方法的 lambda 表达式。

将形参传递给静态方法：
```java
.map(e -> Integer.valueOf(e))
```
这个例子，我们将 lambda 表达式的形参作为实参传递给了 Integer 类的 valueOf 方法。区别在于，在这个例子中，被调用的方法是静态方法而不是实例方法。与前面的办法一样，我们将这个例子替换为方法引用，只不过注意细节这个方法引用并不是放在实例上，而是将它放在一个类上。

对静态方法的方法引用：
```java
.map(Integer::valueOf)
```
总结一下：如果 lambda 表达式的目的仅是将一个形参传递给实例方法，那么可以将它替换为实例上的方法引用。如果传递表达式要传递给静态方法，可以将它替换为类上的方法引用。

## 将形参传递给目标
可能在两种不同的场景中使用 ClassName::methodName 格式。上面看到的是第一种格式，其中的形参作为实参传递给静态方法。现在我们考虑一种变形：形参是方法调用的目标。

使用形参作为目标：
```java
.map(e -> e.doubleValue())
```
下面是这种 lambda 表达式的结构：

![](/images/blog/20180109112228.png)

**模糊性和方法引用**

查看方法引用，不容易确定形参传递给了静态方法还是用作了目标。要了解区别，我们需要知道方法是静态方法还是实例方法。从代码可读性角度讲，这不那么重要，但知道该区别对成功编译至关重要。

如果一个类的一个静态方法和一个兼容的实例方法同名，而且我们使用了方法引用，则编译器将认为该调用模糊不清。所以，举例而言，我们不能将 lambda 表达式 (Integer e) -> e.toString() 替换为方法引用 Integer::toString，因为 Integer 类同时包含静态方法 public static String toString(int i) 和实例方法 public String toString()。

您或您的 IDE 可能建议使用 Object::toString 解决这个特定的问题，因为 Object 中没有 statictoString 方法。尽管该解决方案可以编译，但这种小聪明通常没什么帮助。您必须能够确认方法引用正在调用想要的方法。在存在疑问时，最好使用 lambda 表达式，以避免任何混淆或可能的错误。

## 传递构造函数调用
除了静态和实例方法，也可以使用方法引用来表示对构造函数的调用。考虑从 Supplier 中发出的构造函数调用，Supplier 做为实参提供给 toCollection 方法。

一个构造函数调用示例：
```java
.collect(toCollection(() -> new LinkedList<Double>()));
```
代码的目的是获取一个数据 Stream ，将它精减或收集到一个 LinkedList 中。toCollection 方法接受一个 Supplier 作为其实参。Supplier 不接受任何形参，因此 () 为空。它返回一个 Collection 实例，该实例在本例中是 LinkedList。

从形参到构造函数实参：

![](/images/blog/20180109124605.png)

收到的形参 (可能是空的) 被作为实参传递给构造函数。在下面的示例中，我们可以将 lambda 表达式替换为对 new 的方法引用。

将构造函数替换为方法引用：
```java
.collect(toCollection(LinkedList::new));
```
这个例子包含方法引用的代码比包含 lambda 表达式原始的代码简洁的多，因此更加容易的理解。

## 传递多个实参
我们已经看到了传递一个多 0 个形参的例子。但是拉姆表达式不仅限于 0 或 1 个形参，他们也适用于多个实参。

将 lambda 表达式执行 reduce():
```java
.reduce(0, (total, e) -> Integer.sum(total, e)));
```

在 Stream<Integer> 上调用 reduce 方法，并使用 Integer 的 sum 方法对流中的值求和。这个例子中的 lambda 表达式接受两个形参，它们作为实参（按完全相同的顺序）传递给 sum 方法。图 4 显示了这个 lambda 表达式的结构。

传递两个形参做为实参：

![](/images/blog/20180109125453.png)

也可以将这个拉姆表达式替换为方法引用：
```java
.reduce(0, Integer::sum));
```
**作为目标和实参传递**

无需将所有形参作为实参传递给 static 方法，lambda 表达式可以使用一个形参作为实例方法调用的目标。如果第一个形参用作目标，则可以将 lambda 表达式替换为方法引用。

对使用形参作为目标的 lambda 表达式执行 reduce():
```java
.reduce("", (result, letter) -> result.concat(letter)));
```
在这个例子中，在 Stream<String> 上调用 reduce 方法。该 lambda 表达式使用 String 的 concat 实例方法串联字符串。这个 lambda 表达式中的传递结构不同于您在上一个 reduce 示例中看到的结构：

![](/images/blog/20180109130124.png)

lambda 表达式的第一个形参用作实例方法调用的目标。第二个形参用作该方法的实参。根据此顺序，可以将该 lambda 表达式替换为方法引用.
```java
.reduce("", String::concat));
```
请注意，尽管该 lambda 表达式调用了一个实例方法，但您再次使用了类名称。换句话说，无论您调用静态方法还是将第一个形参作为目标来调用实例方法，方法引用看起来都是一样的。只要不存在模糊性，就没有问题。

## 最好使用方法引用
要掌握传递 lambda 表达式的变形和结构，以及取代它们的方法引用，需要花费一定的时间和精力。在理解之后，就开始感受到使用方法引用取代传递表达式变得更加自然。

比 lambda 表达式更好的是，方法引用使得您的代码变得非常简洁和富于表达，这可以大大减少阅读代码的工作。我们看下面的示例。

使用 lambda 表达式的示例：
```java
List<String> nonNullNamesInUpperCase =
    names.stream()
      .filter(name -> Objects.nonNull(name))
      .map(name -> name.toUpperCase())
      .collect(collectingAndThen(toList(), list -> Collections.unmodifiableList(list)));
```

给定一个 List<String> names，上面的代码删除列表中的所有 null 值，将每个名称转换为大写，并将结果收集到一个无法修改的列表中。

现在让我们使用方法引用重写上述代码。在本例中，每个 lambda 表达式都是一个传递表达式，无论是传递给静态方法还是实例方法。因此，我们将每个 lambda 表达式替换为一个方法引用：
```java
List<String> nonNullNamesInUpperCase =
    names.stream()
      .filter(Objects::nonNull)
      .map(String::toUpperCase)
      .collect(collectingAndThen(toList(), Collections::unmodifiableList));
```
比较这两个清单，很容易看到使用方法引用的代码更加流畅且更容易阅读。它的意思很简单：给定名称，过滤非 null 值，映射到大写形式，然后收集到一个不可修改的列表中。

## 总结
只要看到一个 lambda 表达式的唯一目的是将形参传递给一个或多个其他函数，就需要考虑将该 lambda 表达式替换为方法引用是否更好。决定因素在于，lambda 表达式内没有完成任何实际工作。在这种情况下，lambda 表达式就是一个传递表达式，而且它的语法对于当前这个任务而言可能过于复杂了。

对于方法引用一旦熟悉了，您就会发现与使用 lambda 表达式的代码相比，使用方法引用会让同样的代码更流畅且富于表达。

文章学习地址：

感谢 Venkat Subramaniam 博士

Venkat Subramaniam 博士站点：http://agiledeveloper.com/
