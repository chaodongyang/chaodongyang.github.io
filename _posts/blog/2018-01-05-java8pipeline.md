---
layout: post
title: Java 8 新语法习惯 (函数组合与集合管道模式)
categories: java8新语法习惯
description: Java8 新语法函数组合与集合管道模式
keywords: java,java8新语法,JDK8新语法,java8函数组合与集合管道模式
---

本章节将介绍函数组合和集合管道，您可以结合这两种模式来迭代代码中的集合。了解这些模式的结构有助于您搭建自己的 java 程序，从而充分利用高阶函数和拉姆表达式。

## 语句与表达式
我们在代码中查找 for 循环，回惊奇的发现您的代码中对 for 循环的使用非常频繁。我们将这种情形称为 for 重复：只要我们需要重复似乎就会用到 for。

在 Java 中 for 和 while 都是语句。语句执行一个操作但是不会生成结果。就本质而言，任何执行有用的操作语句都会导致数据变化。这是语句表达其效果的唯一方式。而表达式相反：它们可以得出结果而不会导致变化。在代码中使用语句就像是团队成员合作处理一部分工作，但是无法在团队之间交接工作结果。分享结果的唯一方法是将它放在桌子上或架子上，让另一位团队成员可以获得它。表达式的工作更像是一条链条：当某个人完成一项任务时，他将结果转交给链条的下一个人。

表达式帮助我们实现集合管道模式，也可以称之为运算序列，会将从一次运算收集的输出提供给下一次运算。集合管道模式尽管在面向对象编程中得到了使用，但是它在函数编程中更常见。

函数组合和集合管道模式是两种可协同工作的模式。下面我们先用 for 语句解决一个问题。然后将介绍如何使用这两种模式更高效地解决同一个问题。

## 使用语句进行迭代和排序
首先创建一个 Car 类：
```java
public class Car {
	private String make;
	private String model;
	private int year;
	protected Car(String make, String model, int year) {
		super();
		this.make = make;
		this.model = model;
		this.year = year;
	}
	public String getMake() {
		return make;
	}
	public String getModel() {
		return model;
	}
	public int getYear() {
		return year;
	}

}

```
下面我们添加一个 Car 的实例集合：
```java
public class Iterating {
	public static List<Car> createCars() {
		return Arrays.asList(new Car("Jeep", "Wrangler",2011),new Car("长城","越野",2017)
				,new Car("比亚迪E6","电动",2016),new Car("特斯拉", "modles", 2015)
				,new Car("Jeep", "Comanche", 1990));
	}
}
```
我们使用命令编程的方式来迭代这个列表，获取 2000 年后制造的汽车名称，然后按照年份进行升序排序：
```java
public class GetCar {
	public static List<String> getModelsAfter2000For(List<Car> cars) {
		//找出2000年后生产的车
		List<Car> cars2 = new ArrayList<>();
		for (Car car : cars) {
			if (car.getYear() >2000) {
				cars2.add(car);
			}
		}
		//排序
		Collections.sort(cars2, new Comparator<Car>() {
			@Override
			public int compare(Car o1, Car o2) {
				// TODO Auto-generated method stub
				return new Integer(o1.getYear()).compareTo(o2.getYear());
			}
		});

		List<String> models = new ArrayList<>();
		for (Car car : cars2) {
			models.add(car.getModel());
		}

		return models;
	}
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		System.out.println(getModelsAfter2000For(Iterating.createCars()));
	}

}

```
测试结果：
```
[Wrangler, modles, 电动, 越野]
```
我们在上面的例子中使用了三个循环，第一个循环我们接收一个汽车的列表作为参数提取出 2000 后制造的汽车，然后将它放入下一个列表中。接下来这个列表按照年份进行排序。最后，我们在循环处理列表已得到列表中的型号名称。

这个演示程序向我们展示了语句的效果。尽管函数和方法可用作表达式，但是 Collectionssort 方法并没有返回结果。因为它被用作语句，所以也改变了参数列表的内容。两个 for 循环在迭代的过程中都改变了相应的列表。因此需要很多的垃圾变量来接收和处理。

![](/images/blog/20180105150647.png)

## 使用集合管道进行迭代和排序
函数式编程中，通常会通过一系列更小的模块化函数或运算来对复杂运算进行排序。这个系列的运算被称之为函数组合。当一个数据集合流经一个函数组合时，它就变成一个集合管道。函数组合和集合管道是函数式编程中最常用的两种设计模式。无需很多的 for 循环，依据问题我们能使用多种工具。无需像命令式编程哪样对所有的运算都使用语句，函数式编程鼓励使用表达式。表达式没有改变对象的副作用。而是返回给我们一个结果。我们可以将它传递给另一个函数，我们通过这种方式创建集合管道。
```java
public static List<String> getModelsAfter2000UsingFor(List<Car> cars) {
		return
			     cars.stream()
			         .filter(car -> car.getYear() > 2000)
			         .sorted(Comparator.comparing(Car::getYear))
			         .map(Car::getModel)
			         .collect(Collectors.toList());
	}
```
执行结果：
```
[Wrangler, modles, 电动, 越野]

```
两个方法得到的结果是一样的。但是要注意代码中的不同之处：

1. 函数式代码比命令式代码更简洁
2. 函数式代码不会表现出明显的易变性，而且使用更少的垃圾变量
3. 第二个方法中使用的函数都是有返回值的表达式。
4. 最后一个方法使用了集合管道模式，而且非常富余表达

短短的几行代码，我们的意图就表现的很明显：给定一个汽车的集合，过滤和提取仅在 2000 年后制造的汽车；然后按照年份排序，将这些对象映射转换为他们的型号，最后将结果收集到一个列表中。这个例子中的代码丰富且富余表达，部分原因是使用了方法引用。将一个拉姆表达式传递给 filter 很有用。因为它可以获取给定对象的 year 属性将其与 year 2000 进行比较。传递给 Map 方法的表达式 car -> car.getModel()，该表达式仅返回给定对象的某个属性，不执行任何实际计算或运算。最好把它替换为一个方法引用。我们将方法引用 Car::getModel 传递给 map 方法，而不传递拉姆达表达式。类似地，我们将方法引用 Car::getYear 传递给 comparing 方法，而不传递拉姆达表达式 car -> car.getYear()。方法引用简短、简洁且富于表达。最好尽可能地使用它们。

![](/images/blog/20180105154316.png)

## 总结
在命令式编程中，对于大部分的数据处理，通常都会使用 for 和 while 循环。在 Java 8 的版本中函数式编程是一个非常流行的替代方法。函数组合是一项简单技术，有助于对模块化函数进行排序，从而创建更复杂的运算。按照这个顺序处理数据时您有一个集合管道。结合使用函数组合和集合管道模式，可以创建复杂的程序，让数据从上游到下游，经历一些列的转换。
