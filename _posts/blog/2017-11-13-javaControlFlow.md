---
layout: post
title: java编程思想之控制执行流程
categories: javaThinking
description: java编程思想之控制执行流程
keywords: java, java控制执行流程, java编程思想，java循环语句
---

程序必须在执行过程中控制它的世界，并做出选择。在 java 中你要使用执行控制语句来做出选择。

Java 使用了 C 的所有流程控制语句。在 java 中涉及到的关键字包括 if-else、while、do-while、for、return、break 以及选择语句 switch。然而，java 并不支持 goto 语句。所有条件语句都利用条件表达式的真或假来决定执行路程。该表达式返回 true 或 false。所有的关系操作符都可以拿来构造条件语句。

## if-else语句

if-else 语句是控制程序执行流程的最基本形式。其中的 else 是可选的。判断语句必须使用表达式来产生一个 boolean 值。判断语句之间可以是简单语句或者复合语句，使用 「;」号隔开。下面我们来看一个简单的例子。

```Java
public class ifElseTest {

	static int result = 0;

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		test(10, 5);
		System.out.println(result);
		test(3, 5);
		System.out.println(result);
		test(5, 5);
		System.out.println(result);
	}

	static void test(int testval , int target){

		if(testval > target){
			result = +1;
		}else if (testval == target) {
			result = 0;
		}else {
			result = -1;
		}
	}

}
```
```
1
-1
0
```
注意：尽管 java 语言的格式可以自由，但习惯上还是需要将流程控制语句的主体部分进行缩进排列，使读者能够方便的确定起始和终止。

## 迭代

while、do-while、和 for 用来控制循环，优势将他们划分为迭代语句。语句会重复执行，直到起控制作用的 boolean 表达式得到假的结果为止。

#### while

while 循环语句的格式如下：

```
while(Bolean-expression)
statement
```
在刚开始进行计算的时候，会进行一次 Boolean 表达式的值。而在语句的下一次迭代执行开始前会再进行一次计算。下面的例子得到一个随机数，直到得到符合条件的值。

```Java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		while (condition()) {
			System.out.println("得到了大于0.99的值。");
		}
	}

	static boolean condition(){
		boolean result = Math.random() < 0.99;
		System.out.println(result+",");
		return result;
	}

```

condition() 方法用到了 Math 库里边的 Random() 方法，该方法产生一个 0 到 1 之间的随机数。resut 的值是通过比较操作符来实现的。while 的条件表达意思是说：只要下边的方法 condition() 返回 true，就重复执行循环体中的语句。

#### do-while

while 和 do-while 问唯一区别就是 do-while 中的语句会至少执行一次，即便表达式第一次就计算为 false。而在 while 循环结构中，如果条件第一次为 false,那么其中的语句根本不会执行。

```Java
int x = 1;

do {
  x++;
  System.out.println(x);
} while (x <10);

```

#### for

for 循环语句是最经常使用的迭代语句，在第一次迭代之前要进行初始化。随后，它会进行条件测试，而且在每一次迭代结束时，进行某种形式的步进。for 循环的格式如下：

```
for(initialization; Boolean-espression; step)
statement
```
初始化 initialization、布尔表达式 Boolean-espression、step都可以为空。每次迭代前测试 boolean 表达式。若得到的结果是 false 就会执行 for 语句中的代码。

```Java
for (int i = 0; i < 128; i++) {
   if (Character.isLowerCase(i)) {
   System.out.println("value:" + i+"character:" + (char)i);
   }
 }   
```
注意：变量 i 是在用到它的地方定义的，也就是在 for 循环的控制表达式中，而不是在 main 方法的地方定义。i 的作用域就是 for 循环控制的范围。

#### 逗号操作符

java 里边唯一用到逗号操作符的地方就是 for 循环的控制表达式。在表达式的初始化和步进部分，可以使用一系列逗号分隔的方式。而那些语句均会独立的执行。

```Java
for (int i = 0,j = i+10; i < 128; i++,j = i*2) {
			if (Character.isLowerCase(i)) {
			System.out.println("value:" + i+"character:" + (char)i);
			}
}
```

#### foreach

java SE5 引入了一种新的更加简洁的 for 语法用于数组和容器，即 foreach 语法，表示不必创建 int 变量去对由访问项构成的序列进行计数，foreach 将会自动产生每一项。

```Java
public static void main(String[] args) {
		// TODO Auto-generated method stub

		Random random = new Random(20);
		float[] fs = new float[10];


		for (int i = 0; i < 10; i++) {
			fs[i] = random.nextFloat();
		}

		for (float f : fs) {
			System.out.println(f);
		}
	}
```
这条语句定义了一个 float 类型的变量 f，继而将每一个 fs 赋值给 f。任何返回数组的方法都可以使用 foreach。

```Java
for (char f : "abcdefg".toCharArray()) {
			System.out.println(f);
		}
```
foreach 语法不仅在录入代码时可以节省时间，更重要的是，它阅读起来也更要容易很多，它说明你正在努力做什么，而不是你正在如何做。

## 无条件分支

在 java 中有多个关键词表示无条件分支，他们只是表示这个分支无需任何测试即可发生。这些关键词包括 return、break、continue 和一种与其他语言中的 goto 类似的跳转到标号语句的方式。

#### return

retun 关键词有两个用途，第一个是指定一个方法返回什么值，另外一方面它会导致当前的方法退出，并返回那个值。

```Java
public static void main(String[] args) {
		// TODO Auto-generated method stub

		System.out.println(test(10, 5));

		System.out.println(test(3, 5));

		System.out.println(test(5, 5));
	}

	static int test(int testval , int target){

		if(testval > target){
			return +1;
		}else if (testval == target) {
			return 0;
		}else {
			return -1;
		}
	}
```
如果满足条件之后 return 关键词会跳出当前分支并且返回值。如果在返回 void 的方法中没有 return 语句，那么在方法结尾处会有一个隐士的 return，因此在方法中并非必须有一个 return 语句。但是如果一个方法声明它将返回一个非 void 的其他东西，那么必须确保每一条代码路径都必须有一个 retun。

#### break 和 continue

```Java
Integer[] integers = {0,1,2,3,4,5,6,7,8,9};
 for (int x : integers ) {
   if (x == 5) {
     break;
   }

   if (x%2 != 0) {
     continue;
   }
   System.out.println(x);
 }   
```
在任何迭代语句的主体部分，都可用 break 和 continue 控制执行流程。其中，break 用于强制结束退出循环，不执行循环中剩余的语句。而 continue 则停止当前迭代，人后退回循环开始部分，开始下一次迭代。

只有在不知道中断条件何时满足时，才需要这样使用 break。

#### 臭名昭著的 goto

goto 起源于汇编语言的程序控制：若条件 A 成立，则跳转到这里；否则跳转到那里。注意：Java 编译器生成它自己的汇编代码，但这个代码运行在 java 虚拟机上，而不是直接运行在 CPU 硬件上。goto 语句是在源码级别上的跳转，这使其招致了不好的评价。自从 Edsger Dijkstra 发表了著名的论文 《Goto considered harmful》众人就开始吐槽 goto 的不是，甚至建议直接将它从关键字中删除。

尽管 goto 仍是 java 的保留字，但在语言中并未使用它。Java 没有 goto。然而，Java 也能完成类似的操作，这与 break 和 continue 这两个关键词有关。它们其实不是语句的跳转，而是中断迭代语句的一种方法。之所以将他们纳入到 goto 来讨论，是因为他们使用了同一种机制：标签。

标签是后面跟有冒号的标识符。

> label:

在 Java 中标签起作用的地方是迭代语句之前。在标签和迭代语句之前不能插入任何语句。这样做的目的是，我们希望在一个迭代中嵌套另一个迭代。比如双重循环语句。这是因为 break 和 continue 通常只能中断当前循环。但若是随标签一起使用，它就会中断循环到标签所在的地方。

```
lable1:
outer-iteration{
	inner-iteration{
		break; //(1)
		continue; //(2)
		break lable1; //(3)
		continue lable1; //(4)
	}
}
```
我们来解释一下上边的例子：在 1 中 break 将结束内部循环，回到外部循环继续执行。在 2 中 continue 将使得执行从内部循环的开始初执行。在 3 中break lable1 将会使内部循环和外部循环全部结束。在 4 中 continue lable1 将会跳出内部循环和外部循环，但是流程将会继续从外部循环从新开始执行。

下面是标签用于 for 循环的例子：

```Java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		int i = 0;
		//外部的标签
		outer: for(;true;){
			inner: for(;i<10;i++){
				System.out.println(i);

				if (i ==2) {
					continue;
				}

				if (i ==3) {
					i++;
					break;
				}

				if (i == 6) {
					i++;
					continue outer;
				}

				if (i == 7) {
					break outer;
				}
			}
		}
	}
```
执行的结果：

```
0
1
2
3
4
5
6
7
```
注意：break 语句会中断 for 循环，而且在抵达 for 循环的末尾之前，递增表达式不会执行。由于 break 跳过了递增表达式，所以 i ==3 的情况下，i++ 不执行而是直接返回循环体从内循环开始出继续循环。在 i=6 的情况下直接返回到外部循环开始执行。由于内部循环先加了一次，所以 i=7 直接结束了。

在 Java 中需要是要标签的理由就是因为有循环嵌套的存在，而且想从多层嵌套中 break 和 continue。在 Dijkstra 的论文中，他最反对的就是标签，而非 goto。他发现在一个程序里随着标签的增多，产生的错误也越来越多，并且标签和 goto 使得程序难以分析。但是，Java 的标签不会造成这个问题，因为他们的应用场合已经受到限制，没有特别的方式用于改变程序的控制。

## switch

switch 有时也被划归为一种选择语句。根据整数表达式的值，switch 语句可以从一系列代码中选出一段去执行。它的格式如下:

```
switch(selector){
	case value1 : statement; break;
	case value2 : statement; break;
	case value3 : statement; break;
	//...
	default: statement;
}
```
其中 selector 是一个能够产生整数值的表达式，switch 能将整个表达式的结果和 value 相比较。若发现相符，就执行对应的语句，如发现没有相符，就执行 default 语句。每个 case 都以 break 结尾，这样可以使得执行流程跳转到 switch 主体的末尾。但 break 是可选的，若胜率 break 将会继续执行后面的语句，直到遇到 break 为止。

```Java
public static void main(String[] args) {
		// TODO Auto-generated method stub

		Random random = new Random(40);

		for (int i = 0; i < 20; i++) {
			int c = random.nextInt(20);

			switch (c) {
			case 1:
			case 2:
			case 3:
			case 4:
			case 5:
				System.out.println("结束循环" +c);
				break;

			default:
				System.out.println("没有符合的情况");
				break;
			}
		}
	}
```
```
结束循环2
没有符合的情况
没有符合的情况
结束循环3
没有符合的情况
没有符合的情况
...
```
请注意 case 语句可以堆叠在一起，为一段代码形成多重匹配，即只要符合多种条件中的一种，就执行那段代码。这时也要注意将 break 语句置于特定 case 的末尾，否则控制流程会简单的下移，处理后面的 case。
