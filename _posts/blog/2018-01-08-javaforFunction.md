---
layout: post
title: Java 8 新语法习惯 (for 循环的函数替代方案)
categories: java8新语法习惯
description: Java8 新语法 for 循环的函数替代方案
keywords: java,java8新语法,JDK8新语法,java8 for 循环的函数替代方案
---

我们最常用的迭代一个数据集的方式就是 for 循环，开发人员对它可谓是非常的熟悉。从 Java 8 开始，我们有多个强大的新方法可以帮助我们简化复杂的迭代。在本文中，您将了解如何使用 InStream 方法、range、iterate 和 limit 来迭代范围和跳过范围中的值。还将了解新的 takeWhile 和 dropWhile 方法。

## for 循环的麻烦
在 Java 语言的第一个版本中就开始引入了传统的 for 循环，它有一个更加简单的变体 for-each 是在 Java5 中引入的。我们平时最喜欢使用的还是 for-each 执行普通的迭代，但是对于迭代一个范围或者是跳过范围中的值等操作，他们仍会使用 for 循环。

我们看这样一个示例：
```java
public class ForDemo {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		System.out.println("Get set...");
		for (int i = 0; i < 4; i++) {
			System.out.println(i+"...");
		}
	}

}

```
测试结果：
```
Get set...
0...
1...
2...
3...
```
上面的方法中没有太多的代码非常的简单，但是我们认为这样的迭代还是比较繁琐。Java8 提供了一种更简单、更优雅的替代方法：IntStranm 的 range 方法。我们重写上面的方法：
```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		System.out.println("Get set...");

		IntStream.range(0,4).forEach(i -> System.out.print(i +"..."));
	}
```
测试结果：
```
Get set...
0...1...2...3...
```
上面重写的例子我们看到并没有显著的减少代码量，但是降低了它的复杂性。这样做有两个重要的原因：

1. 不同于 for，range 不会强迫我们初始化某个可变变量。
2. 迭代会自动执行，所以我们不需要像循环索引一样定义增量。

在语义上，最初的 for 循环中的变量 i 是一个可变变量。理解 range 和类似方法的价值对理解这个设计的结果很有帮助。

## 可变变量与参数
for 循环中的变量 i 是一个可变变量，它会在每次对循环执行迭代时发生改变。range 示例中的变量 i 是拉姆表达式的参数，所以它在每次迭代中都是一个全新的变量。这是一个细微的区别，但是决定了这两种方法的不同。请看下面的示例：

在内部类中使用索引变量：
```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
	 ExecutorService service = Executors.newFixedThreadPool(10);
	 for (int i = 0; i < 5; i++) {
		int temp = i;
		service.submit(new Runnable() {

			@Override
			public void run() {
				// 访问 i 不允许这么做
				//System.out.println("Running task " + i);
				System.out.println("Running task " + temp);
			}
		});
	 }
	 service.shutdown();
	}

```
测试结果：
```
Running task 0
Running task 4
Running task 1
Running task 2
Running task 3

```
我们想要在 run 方法中访问变量 i 但是编译器不允许我们这么做。作为这个限制的解决办法，我们可以创建一个局部的临时变量，比如 temp，它是索引变量的一个副本。每次新的迭代变量都会创建变量 temp。在 Java8 之前，我们需要将该变量标记为 final。从 Java8 开始，可以将它视为实际的最终结果，因为我们不会在更改它。无论如何，由于事实上索引变量是一个在迭代中改变的变量，for 循环中就会出现这个额外变量。

下面我们使用 range 函数解决同一个问题。在内部类中使用拉姆表达式参数：
```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
	 ExecutorService service = Executors.newFixedThreadPool(10);
	 IntStream.range(0, 5).forEach(i -> service.submit(new Runnable() {

		@Override
		public void run() {
			// TODO Auto-generated method stub
			System.out.println("Running task " + i);
		}
	}));
	 service.shutdown();
	}
```
测试的结果：
```
Running task 0
Running task 2
Running task 1
Running task 3
Running task 4
```
在作为一个参数被拉姆表达式接受后，索引变量 i 的语义与循环索引变量有所不同。与上面例子中 的 temp 非常相似，这个 i 参数在每次迭代中都表现为一个全新的变量。它就是实际的最终值，因为我们不会在任何地方修改它的值。因此，我们可以直接在内部类的上下文中使用它。

因为 Runnable 是一个函数接口，所以我们可以轻松地将匿名的内部类替换为拉姆表达式。
```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
	 ExecutorService service = Executors.newFixedThreadPool(10);
	 IntStream.range(0, 5).forEach(i -> service.submit(() -> System.out.println("Running task " + i)));
	 service.shutdown();
	}
```
测试结果：
```
Running task 1
Running task 0
Running task 2
Running task 3
Running task 4
```
对于简单的迭代，使用 range 替代 for 具有一定优势，但 for 的特殊价值体现在于它能处理更复杂的迭代场景。让我们看看 range 和其他 java8 方法孰优孰劣。

## 封闭范围
创建 for 循环时我们可以将索引变量封闭在一个范围之内，比如：
```java
for(int i = 0; i <= 5; i++) {

}
```
索引变量 i 接受值 0-5.无需使用 for，我们可以使用 rangeClosed 方法。在下面的示例中我们告诉 IntStranm 将最后一个值限制在该范围：
```java
IntStream.rangeClosed(0, 5)
```
迭代此范围时，我们将获得包含边界值 5 在内的值。

## 跳过值
对于基本的循环，range 和 rangeClosed 方法是 for 的更简单、更优雅的替代方法，但是如果想跳过一些值该怎么办？在这种情况下，for 循环前期工作的需求使得该运算变得非常容易。

for 循环在迭代期间快速跳过两个值：
```java
int total = 0;
for(int i = 1; i <= 100; i = i + 3) {
  total += i;
}
```
我们现在考虑使用 IntStream 的 range 方法，再结合使用 filter 或 map。但是，所涉及的工作将比使用 for 循环要多。一种更可行的解决方案是结合使用 iterate 和 limit：
```java
IntStream.iterate(1, e -> e + 3)
  .limit(34)
  .sum()
```
iterate 方法很容易使用；它只需要获取一个初始值即可以迭代。作为第二个参数传入的拉姆表达式决定了迭代中的下一个值。类似于我们将一个表达式传递给 for 循环来递增索引变量的值。但是，上面的例子没有参数来告诉 iterate 方法合适停止迭代。如果我们没有限制该值，迭代会一直进行下去。

如何解决这个问题呢？我们对 1 到 100 之间的值感兴趣，而且想从 1 开始跳过两个值。稍加运算，可确定给定范围中有 34 个符合要求的值。所以我们将该数字传递给 limit 方法。

上面的代码是有效的，但是过程太复杂：提前执行数学运算是很麻烦的，而且限制了我们的代码。如果我们决定跳过 3 个值而不是 2 个值，该如何呢？我们又需要更改我们的代码，而且从新数学运算。我们需要一个更好的方法来解决这个问题。

## takeWhile 方法
在新版的 Java 9 中即将引入 takeWhile 是一个全新的方法，它使得执行有限的迭代变得更容易。学过 SQL 的同学很容易想到 while 条件语句。使用 takeWhile 可以直接表明只要满足想要的条件，迭代就应该继续执行。

下面是使用 takeWhile 实现迭代的代码：
```java
IntStream.iterate(1, e -> e + 3)
     .takeWhile(i -> i <= 100) //available in Java 9
     .sum()
```
无需将迭代限制到预先我们计算过的次数，我们使用提供给 takeWhile 的条件，动态确定如何终止迭代。这种方法很简单，而且更不容易出错。与 takeWhile 方法相反的是 dropWhile，它跳过满足给定条件前的值，这两个方法都是 JDK 中非常需要的补充方法。takeWhile 类似于 break，而 dropWhile 则类似于 continue。从 Java 9 开始，他们将可用于任何类型的 Stream。测试上面的代码需要将 jdk 的版本更新到 9。

## 逆向迭代
逆向迭代和正向迭代都是非常简单的，无论我们采用 for 循环还是 IntStream。

逆向的 for 循环示例：
```java
for(int i = 5; i > 0; i--) {}
```
由于 range 或 rangeClosed 中的第一个参数不能大于第二个参数，所以我们无法使用这两个方法进行逆向的迭代。但是可以使用 iterate 方法：
```java
IntStream.iterate(5, e -> e - 1)
	     .limit(5);
```
将一个拉姆表达式传递给 iterate 方法，该方法对给定值进行递减，以便沿着相反方向直行迭代。我们使用 limit 函数指定我们希望在迭代周期直行的次数。

## 总结
传统的 for 循环非常的强大，但是它过于复杂。Java 8 和 Java 9 中的新方法可以帮助我们简化迭代，甚至是复杂的迭代。方法 range、iterate 和 limit 的可变部分较少，有助于提高我们的代码效率。并且还满足了 Java 的一个要求，那就是局部连拉必须声明为 final 然后才能从内部类中访问它。将一个可变索引变量更换为实际的 final 参数只有很小的语义差别，但是它减少了大量的垃圾变量。最终得到更简洁、更优雅的代码。

文章学习地址：

感谢 Venkat Subramaniam 博士

Venkat Subramaniam 博士站点：http://agiledeveloper.com/
