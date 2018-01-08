---
layout: post
title: Java 8 新语法习惯 (提倡使用有帮助的编码)
categories: java8新语法习惯
description: Java8 新语法提倡使用有帮助的编码
keywords: java,java8新语法,JDK8新语法,java8提倡使用有帮助的编码
---

表达能力强是函数式编程的优势之一，但是这对于我们的代码意味着什么呢？在这部分内容中，我们将比较命令式和函数式代码的示例，判断这两种的表达能力和简洁性的能力。我们还能够了解到这些品质如何确保可读性，还需要考虑一个反面示例：对简洁性的过度追求导致代码无用。最后我们将会介绍 Java8 对于函数组合中的垂直对其点的约定。

尽管 Java8 函数式编程已经出现很长时间了，但是目前来说依然没有得到充分的推广。大部分的程序员还是更加理解命令式编程，面对同一个功能的代码对于函数式编程的理解比命令式编程要花费更多的时间。

## 尝试一个实验
函数式代码比命令式代码更富有简洁性-前提是需要精心编写代码。一个简单的示例可以证明这一点。

第一个命令式代码示例：
```java
public static void main(String[] args) {
		List<String> names = Arrays.asList("Jack", "Jill", "Nate", "Kara", "Kim", "Jullie", "Paul", "Peter");

		List<String> subList = new ArrayList<>();
		for(String name : names) {
		  if(name.length() == 4)
		    subList.add(name);

		}

		StringBuilder namesOfLength4 = new StringBuilder();
		for(int i = 0; i < subList.size() - 1; i++) {
		  namesOfLength4.append(subList.get(i));
		  namesOfLength4.append(", ");
		}

		if(subList.size() > 1)
		  namesOfLength4.append(subList.get(subList.size() - 1));

		System.out.println(namesOfLength4);
	}

```
打开计时器尝试着记录一下理解上面这段代码所需要的时间。

下面看函数式代码的示例：
```java
List<String> names = Arrays.asList("Jack", "Jill", "Nate", "Kara", "Kim", "Jullie", "Paul", "Peter");

System.out.println(
  names.stream()
    .filter(name -> name.length() == 4)
    .collect(Collectors.joining(", ")));
```
再次打开计时器尝试着记录理解这段代码所需要花费的时间。假设我们对两种编程方式都是了解的，我想很显然函数式编程的可读性和简洁性理解起来更高效更快。

## 函数式编码为什么至关重要
如果我们熟悉 Java8 ，那么对于上面的函数式编码理解起来会很顺利，即使是我们不熟悉 Java8 ，那么得益于描述性的方法名称，您也可能理解它。并且快速的理解此代码，因为它比命令式编码要简洁的多。

```
基本上讲，代码的含义是：给定一个集合，仅选择长度为 4 的名称，然后通过逗号将他们连接起来。
```

## 编写可读的代码
函数式代码富于表达且简洁，使得程序不但更短，而且更容易阅读。

看下面的命令式代码示例：
```java
int result = 0;
for(int e : numbers) {
  if(e > 3 && e % 2 == 0 && e < 8) {
    result += e * 2;
  }
}
System.out.println(result);
```
上面代码计算了在 numbers 列表中大于 3 且 小于 8 的偶数并且将该数字乘以 2，然后输出结果。

现在考虑使用 Java8 的函数式代码编写相同的效果：
```java
System.out.println(
 numbers.stream()
   .filter(e -> e  > 3)
   .filter(e -> e % 2 == 0)
   .filter(e -> e < 8)
   .mapToInt(e -> e * 2)
   .sum());
```
我们看到代码的编写行数是一样的，并没有减少代码。函数式代码并不总是比命令式代码短。更重要的是它富于表达。简洁但是难读的代码是无用的。

## 避免犯这个错误
函数式代码设计的目标是使得代码更简洁，但是这并不意味这你的代码就更具可读性。

看下面的示例，连接选定名称的函数式代码：
```java
System.out.println(names.stream().filter(name -> name.startsWith("J")).filter(name -> name.length() > 3)
  .map(name -> name.toUpperCase()).collect(Collectors.joining(", ")));
```
上面的代码尽管只有两行，但是此代码显得很冗长。阅读起来比较费劲。我们需要很努力的仔细的观察函数调用的节点，函数调用在哪里结束，下一个调用从何处开始。这段代码非常简单，但是编写的非常生硬。

**让你的代码简洁而不是生硬**

生硬的代码非常短，但是仍然很难读懂。简洁的代码也很短，但是阅读起来感觉更加容易理解。在编程过程中我们很容易忽略表达能力和可读性的价值。Java8 通过约定来鼓励这些品质，建议我们对齐函数组合的垂直方向上的点。经验丰富的程序员应该在代码评审期间执行这个约定。

使用对齐约定重写上面的代码：
```java
System.out.println(
 names.stream()
      .filter(name -> name.startsWith("J"))
      .filter(name -> name.length() > 3)
      .map(name -> name.toUpperCase())
      .collect(Collectors.joining(", ")));
```
同样的一段代码，由于我们遵循了对齐的约定。结果每行都具有凝聚力：范围狭小而专注，仅含有一个明确的任务。更加利于我们的理解和阅读。

## 一种有帮助的约定
遵循 Java8 的对齐约定肯定是非常有益的：

- 遵循此约定的代码更容易阅读、理解和解释。我们可以在详细检查每部分之前，快速掌握整个目标。
- 元素非常明确切容易找到，有助于更快的修改。如果我们想包含另一个条件，或者删除或者修改一个现有的条件，那么可以相对容易的找到并执行更改。
- 该代码更容易团队维护。

## 总结
保持每行代码都简短紧凑是一种不错的做法，但是走入极端可能导致代码变得生硬难以阅读。要提高表达能力，可以问自己代码是否容易理解。要提高可读性，可采用 Java8 垂直对齐点的约定。使用这些简单的技巧，就能创建出简洁、富于表达且可读的函数式代码。

文章学习地址：

感谢 Venkat Subramaniam 博士

Venkat Subramaniam 博士站点：http://agiledeveloper.com/
