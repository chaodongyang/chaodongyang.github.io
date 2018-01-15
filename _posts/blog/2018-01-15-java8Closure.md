---
layout: post
title: Java8 新语法习惯 (使用闭包捕获状态)
categories: java8新语法习惯
description: Java8 新语法使用闭包捕获状态
keywords: java,java8新语法,JDK8新语法,java8新语法使用闭包捕获状态
---

在 Java 编程中，我们以不严格的术语 lambda 表达式来表示 lambda 表达式和闭包。但是在某些情况下，理解它们的区别很重要。lambda 表达式是无状态的，而闭包是带有状态的。将 lambda 表达式替换为闭包，是一种管理函数式程序中的状态的好方法。

## 无状态的生活
我们在这个系列中介绍了 lambda 表达式，您应该已经对他们非常的了解了。它们是小巧的匿名函数，接受可选的参数，执行某种计算或操作，而且可能返回一个结果。lambda 表达式也是无状态的，这可能会在您的代码中产生重大影响。

我们首先来看一个使用 lambda 表达式的简单示例。假设我们想将一个数字集合中的偶数乘以二。一种是使用 Stream 和 lambda 表达式创建一个函数管道，入下所示：
```java
numbers.stream()
  .filter(e -> e % 2 == 0)
  .map(e -> e * 2)
  .collect(toList());
```
我们传入 filter 中的 lambda 表达式取代了 Predicate 函数接口。它接收一个数字，如果该数字是偶数，则返回 true，否则返回 false。另一方面，我们传递给 map 的 lambda 表达式取代了 Function 函数接口：它接受任何数字并返回该值的两倍。这个lambda 表达式都依赖传入的参数和字面常量。二者都是独立的，这意味着他们没有任何外部依赖项。因为它们依赖于传入的参数，而且可能还依赖于一些常量，所以 lambda 表达式是无状态的。

## 我们为什么需要状态
现在让我们更仔细地看看传递给 map 方法的 lambda 表达式。如果我们希望计算给定值的三倍或四倍，该怎么办？我们可以将常量 2 转换为一个变量（比如 factor），但 lambda 表达式仍需要一种方式来获取该变量。

我们可以推断，lambda 表达式可以采用与接收参数 e 的相同方式来接收 factor，如下所示：
```java
.map((e, factor) -> e * factor)
```
还不错，但不幸的是它不起作用。方法 map 要求接受函数接口 ```Function<T, R>``` 的一个实现作为参数。如果我们传入该接口外的任何内容（比如一个 ```BiFunction<T, U, R>```），map 不会接受。需要采用另一种方式将 factor 提供给我们的 lambda 表达式。

## 词法范围
函数要求变量在限定范围内。因为它们实际上是匿名函数，所以 lambda 表达式也要求引用的变量在限定范围内。一些变量以参数形式被函数或 lambda 表达式接收。一些变量是局部定义的。一些变量来自函数外部，位于所谓的词法范围中。

词法范围示例：
```java
public static void print() {
  String location = "World";

  Runnable runnable = new Runnable() {
    public void run() {
      System.out.println("Hello " + location);
    }
  };

  runnable.run();
}
```
在 print 方法中，location 是一个局部变量。但是，Runnable 的 run 方法还引用了一个不是 run 方法的局部变量或参数的 location。对 Hello 旁边的 location 的引用被绑定到 print 方法的 location 变量。

词法范围是函数的定义范围。反过来，它也可能是该定义范围的定义范围，等等。

在前面的代码中，方法 run 没有定义 location 或接收它作为参数。run 的定义范围是 Runnable 的匿名内部对象。因为没有将 location 定义为该实例中的字段，所以会继续搜索匿名内部对象的定义范围 — 在本例中为方法 print 的局部范围。

如果 location 不在该范围中，编译器会继续在 print 的定义范围内搜索，直到找到该变量或搜索失败。

**lambda表达式中的词法范围**

我们使用 lambda 表达式重写前面的代码：
```java
public static void print() {
  String location = "World";

  Runnable runnable = () -> System.out.println("Hello " + location);

  runnable.run();
}
```
得益于 lambda 表达式，代码变得更简洁，但 location 的范围和绑定没有更改。lambda 表达式中的变量 location 被绑定到 lambda 表达式的词法范围中的变量 location。严格来讲，此代码中的 lambda 表达式是一个闭包。

## 闭包如何携带状态
Lambda 表达式不依赖于任何外部实体；它们是依赖于自身参数和常量的内容。另一方面，闭包既依赖于参数和常量，也依赖于它们的词法范围中的变量。从逻辑上讲，闭包被绑定到它们的词法范围中的变量。但是，尽管逻辑上讲是这样，但实际上并不总是这么做。有时甚至无法执行这样的绑定。两个场景可以证明这一点。

下面这段代码将一个 lambda 表达式或闭包传递给一个 call 方法:
```java
class Sample {
  public static void call(Runnable runnable) {
    System.out.println("calling runnable");

    //level 2 of stack
    runnable.run();
  }

  public static void main(String[] args) {
    int value = 4;  //level 1 of stack
    call(
      () -> System.out.println(value) //level 3 of stack
    );
  }
}
```
此代码中的闭包使用了来自它的词法范围的变量 value。如果 main 是在堆栈级别 1 上执行的，那么 call 方法的主体会在堆栈级别 2 上执行。因为 Runnable 的 run 方法是从 call 内调用的，所以该闭包的主体会在级别 3 上运行。如果 call 方法要将该闭包传递给另一个方法（进而推迟调用的位置），则执行的堆栈级别可能高于 3。

您现在可能想知道在一个堆栈级别中的执行究竟如何能获取之前的另一个堆栈级别中的变量 — 尤其是未在调用中传递上下文时。简单来讲就是无法获取该变量。

看另外一个示例：
```java
class Sample {
  public static Runnable create() {                   
    int value = 4;
    Runnable runnable = () -> System.out.println(value);

    System.out.println("exiting create");
    return runnable;
  }

  public static void main(String[] args) {
    Runnable runnable = create();

    System.out.println("In main");
    runnable.run();
  }
}
```
测试结果：
```
exiting create
In main
4
```
在这个示例中，create 方法有一个局部变量 value，该变量的寿命很短：只要我们退出 create，它就会消失。create 内创建的闭包在其词法范围中引用了这个变量。在完成 create 方法后，该方法将闭包返回给 main 中的调用方。在此过程中，它从自己的堆栈中删除变量 value，而且 lambda 表达式将会执行。

我们知道，在 main 中调用 run 时，create 中的 value 就会终止。尽管我们可以假设 lambda 表达式中的 value 直接被绑定到它的词法范围中的变量，但该假设并不成立。

**闭包午休时间**

假设我的办公室离家约 16 公，而且我早上 8 点出门上班。中午，我有短暂的时间用午餐，但出于健康考虑，我喜欢吃家里烹饪的饭菜。由于休息时间很短，只有在离家时带上午餐，我才能吃上家里的饭菜。

这就是闭包要完成的任务：它们携带自己的午餐（或状态）。

让我们再看看 create 中的 lambda 表达式：
```java
Runnable runnable = () -> System.out.println(value);
```
我们编写的 lambda 表达式没有接受任何参数，但需要它的 value。编译类 Sample 并运行 javap -c -p Sample.class 来检查字节码。您会注意到，编译器为该闭包创建了一个方法，该方法接受一个 int 参数：
```java
private static void lambda$create$0(int);
    Code:
       0: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
       3: iload_0
       4: invokevirtual #9                  // Method java/io/PrintStream.println:(I)V
       7: return
}
```
现在看看为 create 方法生成的字节码：
```java
0: iconst_4
1: istore_0
2: iload_0
3: invokedynamic #2,  0              // InvokeDynamic #0:run:(I)Ljava/lang/Runnable;

```
值 4 存储在一个变量中，然后，该变量被加载并传递到为闭包创建的函数。在本例中，闭包保留着 value 的一个副本。这就是闭包携带状态的方式。

## 使用闭包
现在，我们回头看看本文开头的示例。除了计算集合中的偶数值的两倍，如果我们想要计算它们的三倍或四倍，该怎么办？为此，我们可以将原始 lambda 表达式转换为一个闭包。

这是我们之前看到的无状态代码：
```java
numbers.stream()
  .filter(e -> e % 2 == 0)
  .map(e -> e * 2)
  .collect(toList());
```
使用闭包而不是 lambda 表达式，代码就会变成：
```java
int factor = 3;

numbers.stream()
  .filter(e -> e % 2 == 0)
  .map(e -> e * factor)
  .collect(toList());
```
map 方法现在接受一个闭包，而不是一个 lambda 表达式。我们知道，这个闭包接受一个参数 e，但它也捕获并携带 factor 变量的状态。

此变量位于该闭包的词法范围中。它可以是定义 lambda 表达式的函数中的局部变量；可以作为该外部函数的一个参数传入；可以位于闭包的定义范围（或该定义范围的定义范围等）中的任何地方。无论如何，该闭包将状态从定义该闭包的代码传递到了需要该变量的执行点。

## 总结
闭包不同于 lambda 表达式，因为它们依赖于自己的词法范围来获取一些变量。因此，闭包可以捕获并携带状态。lambda 表达式是无状态的，闭包是有状态的。可以在您的程序中使用闭包，将状态从定义上下文携带到执行点。
