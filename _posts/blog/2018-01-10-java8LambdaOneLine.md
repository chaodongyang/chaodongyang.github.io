---
layout: post
title: Java8 新语法习惯 (完美的 lambda 表达式只有一行)
categories: java8新语法习惯
description: Java8 新语法完美的 lambda 表达式只有一行
keywords: java,java8新语法,JDK8新语法,java8新语法完美的 lambda 表达式只有一行
---

现在我们已经了解到函数组合的一个好处是它会获得富于表达的代码。编写简短的 lambda 表达式是实现这一功能的关键能力。本文会加深您目前对创建单行 lambda 表达式的各个方面的了解。通过学习函数组合的结构和好处，您很快就会掌握完美的 lambda 表达式，一个仅仅只有一行的 lambda 表达式。

## 编写 lambda 表达式的两种方式
lambda 表达式是匿名函数，它们天生就是很简洁。普通的函数或方法通常有 4 个元素：
- 一个名称
- 返回类型
- 参数列表
- 主体

lambda 表达式只有 4 个元素中的最后两个：
```java
(parameter list) -> body
```

```->``` 将参数列表和函数主体分离，旨在对给定的参数进行处理。函数的主体可能是一个表达式或一条语句。下面是示例：
```java
(Integer e)  -> e*2
```
在此代码中，主体只有一行：一个返回两倍给定参数的表达式。信噪比很高，没有分号，也不需要 return 关键字。这就是一个理想的 lambda 表达式。

**多行 lambda 表达式**

在 Java 中一个 lambda 表达式的主体也可以是一个复杂的表达式或声明语句；也就是说，一个 lambda 表达式包含多行。在这种情况下，分号必不可少。如果 lambda 表达式返回一个结果，也需要 return 关键字。

下面是一个示例：
```java
(Integer e) -> {
  double sqrt = Math.sqrt(e);
  double log = Math.log(e);

  return sqrt + log;
}
```
本例中的 lambda 表达式返回了 sqrt 和给定参数的 log 的和。因为主体包含多行，所以```{}```、分号```(;)```和 ```return```关键字都是必须的。

## 函数组合的强大功能
函数式编码利用了函数组合的强大表达能力。比较下面两段代码：

命令式编码：
```java
int result = 0;
for(int e : values) {
  if(e > 3 && e % 2 == 0) {
    result = e * 2;
    break;
  }
}
```

函数式编程：
```java
int result = values.stream()
  .filter(e -> e > 3)
  .filter(e -> e % 2 == 0)
  .map(e -> e * 2)
  .findFirst()
  .orElse(0);
```
两段代码获取相同的结果。在命令式代码中，我们需要读入 for 循环，按照分支和中断来跟随流程。第二段代码使用了函数组合，更容易阅读一些。因为它是从上往下执行的，所以我们只需要传递该代码一次。本质上，第二段代码读起来像是一个问题陈述。这样的代码不仅优雅，而且工作量也不会增加。得益于 Stream 的惰性计算能力，这里没有浪费计算资源。

注意：函数组合的强大表达能力很大程度上依赖于每个 lambda 表达式的简洁性。如果您的 lambda 表达式包含多行代码，您可能没有理解函数式编程的关键点。

## 充满危险的长 lambda 表达式
我们看一个包含多行代码的杂乱的 lambda 表达式：
```java
System.out.println(
values.stream()
  .mapToInt(e -> {     
    int sum = 0;
    for(int i = 1; i <= e; i++) {
      if(e % i == 0) {
        sum += i;
      }
    }

    return sum;
  })
  .sum());
```

尽管这段代码是函数式风格编写的，但是它丢失了函数式编程的优点。

1. 难以读懂

 好代码应该很容易读懂，这段代码很难找到开始结尾。
2. 用途不明

 好代码应该读起来像是一个故事，而不是一个字谜。这样冗长的、无特色的代码隐藏了它的具体用途，会耗费读者的时间和精力。
3. 代码质量差

 无论你的代码有何用处，您可能都希望在某个时候重用它。这段代码的逻辑已经嵌入 lambda 表达式中，后者又以参数形式传递给另一个函数 mapToInt。如果我们需要的程序的某个地方使用，我们就需要重写。这回引起代码库的不一致性。
4. 难以测试

 代码始终依靠键入的内容进行操作，而且不一定是我们打算执行的操作，所以这代表着必须测试任何非平凡的代码。如果 lambda 表达式的代码无法用作一个单元，则无法对他进行单元测试。
5. 代码覆盖范围小

 嵌入 lambda 表达式的代码无法轻松的作为单元提取出来，而且许多处报告红色。让我们很难假设这些代码在正常工作。

## 使用 lambda 作为粘合代码
解决问题的方法是让您的 lambda 表达式高度的简洁。首先应该避免在 lambda 表达式中使用括号。
```java
System.out.println(
values.stream()
  .mapToInt(e -> sumOfFactors(e))
  .sum());
```
不需要解字谜，因为此代码直接表明了它的用途。计算因数之和的代码已模块化为一个名为 sumOfFactors 的单独方法，该方法可以重用。因为它是一个单独方法，所以对它的逻辑执行单元测试也很容易。因为此代码如此容易测试，所以您可以确保良好的代码覆盖范围。

简言之，曾经杂乱的 lambda 表达式现在成为了 粘合代码— 它没有承担大量责任，只是将命名函数粘合到 mapToInt 函数。

**使用方法引用进行调优**

可以将 lambda 表达式替换为方法引用，以让上述代码更富于表达能力：
```java
System.out.println(
values.stream()
  .mapToInt(Sample::sumOfFactors)
  .sum());
```

重写后的 sumOfFactors 方法：
```java
public static int sumOfFactors(int number) {
  return IntStream.rangeClosed(1, number)
    .filter(i -> number % i == 0)
    .sum();
}
```

现在它是一个简短的方法。该方法中的 lambda 表达式也很简洁：只有一行，没有过多的繁杂过程或噪音。

## 总结
简短的 lambda 表达式是提高代码可读性，这是函数式编程的重要好处之一。包含多行的 lambda 表达式具有相反的效果，会让代码变得杂乱且难以阅读。多行 lambda 表达式还难以测试和重用，这可能导致重复工作和代码质量差。通过将多行 lambda 表达式转移到一个命名函数中，然后从 lambda 表达式内调用该函数，这样很容易避免这些问题。简言之，避免多行 lambda 表达式。

文章学习地址：

感谢 Venkat Subramaniam 博士

Venkat Subramaniam 博士站点：http://agiledeveloper.com/
