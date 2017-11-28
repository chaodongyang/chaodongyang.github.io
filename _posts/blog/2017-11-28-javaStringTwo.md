---
layout: post
title: java编程思想之字符串深入
categories: javaThinking
description: java编程思想之字符串深入
keywords: java, java字符串, java编程思想之字符串深入，java之字符串深入
---

上一节学习到使用 Formatter 可以进行各种各样的格式化操作。格式化中的表达式各种各样，我们通常最常见的是正则表达式。

## 正则表达式
Java 中的字符串与正则表达式相比只是提供了相当简单的功能。正则表达式是一种强大而灵活的文本处理工具。使用正则表达式，我们能够以编程的方式构造复杂的文本模式，并对输入的字符串进行搜索。一旦找到了匹配的部分，就可以随心所欲的进行处理。正则表达式提供了一种完全通用的方式，能够解决各种字符串处理相关的问题：匹配、选择、编辑以及验证。

#### 基础
一般来说正则表达式就是以某种方式来描述字符串。例如，要找一个数字，它可能有一个负号在最前边，那你就写一个负号加一个问号：

```
-?
```
要描述一个整数，可以说它有一位或多位阿拉伯数字。在正则表达式中用 「\d」表示一位数字。如果在其他语言中使用过正则表达式，会发现 Java 中对 「\」的处理是不同的。在其他语言中「\\」表示我想要在正则表达式中插入一个普通的反斜杠。而在 Java 中「\\」表示我要插入一个正则表达式的反斜杠。例如要表示一个数字，那么正则表达式应该是「\\d」。如果想要插入一个普通的反斜杠，则应该是「\\\\」。不过换行和制表符只需要单反斜杠：「\n」「\t」。

要表示一个或者多个之前的表达式，应该用「+」。所以，所以如果要表示，可能有一个负号，后面跟着一位或多位数字。可以这样：

```
-?\\d+
```
应用正则表达式最简单的途径，就是利用 String 类的内建功能。例如可以检查一个 String 是否匹配：

```java
public class IntegerMatch {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		System.out.println("-12345".matches("-?\\d+"));
		System.out.println("12345".matches("-?\\d+"));
		System.out.println("+12345".matches("-?\\d+"));
		System.out.println("+12345".matches("(-|\\+)?\\d+"));

	}

}

```
执行结果：
```
true
true
false
true
```
前两个满足对应的表达式，匹配成功。第三个有一个「+」。他也是一个合法的整数，但与对应的表达式却不匹配。第四个表达式含义。可能是一个加号或减号开头或没有。在最后一个表达式中括号表示要分组的效果，而竖线「|」则表示或操作。

```
(-|\\+)?
```
因为字符「+」在正则表达式中有特殊的意义，所以必须用「\\」转义，使其成为一个普通的字符。

String 类自带了一个非常有用的正则表达式工具 —— split() 方法。其功能是将字符串从正则表达式匹配的地方切开。

```java
public class Splitting {

	public static String thinkinjava = "wo,shi yi ming zhong guo gong chan dang yuan ,wo men "
			+"bi xu shi xian zhong hua min zu de fuxing ... "+
			"ha ha ha...fen dou ba cheng xu yuan";
	public static void  split(String string) {
		System.out.println(Arrays.toString(thinkinjava.split(string)));
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		split(" ");
		split("\\W+");
		split("n\\W+");
	}

}

```
执行结果：
```
[wo,shi, yi, ming, zhong, guo, gong, chan, dang, yuan, ,wo, men, bi, xu, shi, xian, zhong, hua, min, zu, de, fuxing, ..., ha, ha, ha...fen, dou, ba, cheng, xu, yuan]

[wo, shi, yi, ming, zhong, guo, gong, chan, dang, yuan, wo, men, bi, xu, shi, xian, zhong, hua, min, zu, de, fuxing, ha, ha, ha, fen, dou, ba, cheng, xu, yuan]

[wo,shi yi ming zhong guo gong cha, dang yua, wo me, bi xu shi xia, zhong hua mi, zu de fuxing ... ha ha ha...fe, dou ba cheng xu yuan]
```
第一个 split() 只是按照空格来划分字符串。第二个和第三个都用到了「\W」。它的意思是非单词字符 (如果 W 小写，则表示一个单词字符)。通多第二字例子可以看出它将标点字符删除了。第三个 split() 后面表示后面字母 n 跟着一个或多个非单词字符。在原始字符串中与正则表达式匹配的部分都不存在了。

String.split() 还有一个重载的版本，它允许你限制字符串分隔的次数。String 还自带了一个正则表达式工具就是替换。你可以替换正则表达式中匹配的字符串。

```java
public class Replac {
	static String  string  = Splitting.thinkinjava;

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		System.out.println(string.replaceFirst("f\\w+", "替换"));
		System.out.println(string.replaceFirst("aaa|bbb|ccc", "匹配"));
	}

}
```
执行结果：
```
wo,shi yi ming zhong guo gong chan dang yuan ,wo men bi xu shi xian zhong hua min zu de 替换 ... ha ha ha...fen dou ba cheng xu yuan

wo,shi yi ming zhong guo gong chan dang yuan ,wo men bi xu shi xian zhong hua min zu de fuxing ... ha ha ha...fen dou ba cheng xu yuan
```
第一个表达式要匹配，以字母 f 开头，后面跟一个或多个字母。并且只替换掉第一个匹配的地方。第二个表达式要匹配的是第一个参数中的三个单词中任意一个。并且替换所有匹配的部分。

#### 创建正则表达式
我们先看一些正则表达式的子集。完整的正则表达式列表在 java.util.regex 包中的 Pattern 类中。

|   表达式   |    含义 |
| :----      | :---    |
|   B   |    指定字符 B |
|   \xhh   |    十六进制值的 oxhh 的字符 |
|   \uhhhh   |    十六进制表示为 oxhhhh 的 Unicode 字符 |
|   \t   |    制表符 |
|   \n   |    换行符 |
|   \r   |    回车 |
|   \f   |    换页 |
|   \e   |    转义 Escape |

character classes 字符类，一下是创建字符类的一些典型方式。

|   表达式   |    含义 |
| :----      | :---    |
|   .   |    任意字符 |
|   [abc]   |   包含 a、b 或者 c 的任意字符|
|   [^abc]   |   除了 a、b 和 c 之外的任意字符 |
|   [a-zA-Z]   |   从 a - z 的任意字符包括大小写 |
|   [abc[hij]]   |   任意 a、b、c、h、i、j 字符 |
|   [a-z&&[hij]]   |   任意 h、i、j (交) |
|   \s   |    空白符包括：空格、tab、换行、换页、回车 |
|   \S   |   非空白符  [^\s]|
|   \d     |     数字 [0-9]             |
|   \D     |     非数字 [^0-9]             |
|   \w     |     词字符 [a-zA-Z0-9]            |
|   \W     |     非词字符 [^\w]             |

逻辑操作符：

|   表达式   |    含义 |
| :----      | :---    |
|   XY   |    Y跟在X后边 |
|   X竖线Y    |   X或Y |
|   (X)   |   捕获组。可以在表达式中用 \i 引用第 i 个捕获组 |

边界匹配符：

|   表达式   |    含义 |
| :----      | :---    |
|  ^    |    一行的起始 |
|   $    |   一行的结束 |
|   \b   |  词的边界 |
|   \B    |   非词的边界 |
|   \G   |  前一个匹配的结束 |

演示：下面每一个正则表达式都能匹配字符序列 “Rudolph” ：

```java
public class Rudolph {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		for (String patterrn : new String[]{"Rudolph","[rR]udolph","[rR][aeiou][a-z]ol.*","R.*"}) {
			System.out.println("Rudolph".matches(patterrn));
		}
	}

}
```
执行结果：
```
true
true
true
true

```
我们的任务是以最简单以及最必要的正则表达式。编写表达式之前往往会复用之前的表达式。

## 量词
量词描述了一个模式吸收输入文本的方式：

- 贪婪型：量词总是贪婪的，除非有其他的选项被设置。贪婪表达式会为所有可能的模式发现尽可能多的匹配。假定我们的模式只能匹配第一个可能的字符串。如果他是贪婪的，那么它会继续向下匹配。

- 勉强型：用问号来指定，这个两次匹配满足模式所需的最少字符串。因此也称作懒惰的、最少匹配的、非贪婪的。

- 占有型：目前这种类型的量词只有在 Java 中才可用，并且也更高级。当正则表达式被用于字符串时，会产生相当多的状态，以便在匹配失败事可以回溯。而 “占有的” 量词并不保存这些状态，因此他们可以防止回溯。它常常用于正则表达式失控，可以使得表达式执行起来更有效。

|   贪婪型     |  勉强型     |  占有型      |   如何匹配     |
| :---         | :---       | :---        |:---          |
|   X?           |  X??     |   X?+   |一个或零个X       |
|    X*          |  X*?     |   X*+   | 零个或多个X        |
|    X+         |   X+?     |    X++  | 一个或多个X         |
|    X{n}       |   X{n}?   |  X{n}+  |  恰好N次X            |
|    X{n,}      |  X{n,}?   | X{n,}+  |  至少n次X            |
|    X{n,m}     | X{n,m}?   | X{n,m}+ |  X至少n次，且不超过m次 |

应该非常的清楚表达式 X 应该用括号括起来，以便他能够按照我们的期望去执行。

```
abc+
```
看起来是应该匹配一个或者多个 abc 序列。然而实际上匹配的是 ab，后面跟随一个或者多个的 c 。实际上我们应该这样写：

```
(abc)+
```

在使用正则表达式时很容易混淆，因为这是一种 Java 上的新语言。

接口 CharSequence 从 CharBuffer、String、StringBuffer、StringBuilder 类中抽象除了字符序列的一般化定义,这些类都实现了该接口。多数正则表达式都接受 CharSequence 类型的参数。

```java
interface CharSequence{
	charAt(int i);
	length();
	subSequence(int start,int end);
	toString();
}
```

#### Pattern 和 Matcher
一般来说，比起功能有限的 String 类，我们更愿意构造功能强大的正则表达式对象。使用 Java 提供的 static Pattern.compile() 方法来编译你的正则表达式即可。他会根据你的 String 类型的正则表达式生成一个 Pattern 对象。接下来把你想要检索的字符串传入 Pattern 对象的 matcher() 方法。matcher() 方法会生成一个 Matcher 对象，他有很多的功能可用。

看下面一个示例：

```java
public class TestRegulay {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		if (args.length <2) {
			System.exit(0);
		}

		System.out.println("Input:\" " + args[0] + "\"");
		for (String string : args) {
			System.out.println("arg是多少:"+string);
			Pattern pattern = Pattern.compile(string);
			Matcher matcher = pattern.matcher(args[0]);
			while (matcher.find()) {
				System.out.println("Matcher \" "+matcher.group() + "\" at positions" + matcher.start() + "-"+(matcher.end()-1));
			}
		}
	}

}

```
Matcher 有很多方法可以供我们使用

###### find()
Matcher.find() 方法可以用来在 CharSequence 中查找多个匹配。

```java
public class Finding {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Matcher matcher = Pattern.compile("\\w+").matcher("Wo shi xiang jiao ");
		while (matcher.find()) {
			System.out.print(matcher.group()+" ");

		}

		System.out.println();

		int i =0;
		while (matcher.find(i)) {
			System.out.print(matcher.group()+" ");
			i++;

		}
	}

}

```
执行结果：
```
Wo shi xiang jiao
Wo o shi shi hi i xiang xiang iang ang ng g jiao jiao iao ao o
```
模式「\\w+」将字符串划分为单词。find() 就像迭代器一样从前向后遍历输入的单词。第二个 find() 能接收一个整数作为参数，该整数表示字符串中字符的位置，并以此作为搜索的起点。从结果看，后一个版本能够不断的重新设定搜索的起始位置。

###### 组 Groups
组是用括号划分的正则表达式，可以根据组的编号来引用某个组。租号为 0 来表示整个表达式，租号为 1 表示被第一个括号括起来的租，以此类推。

```
A(B(C))D
```
组 0 是 ABCD，组 1 是 BC,组 2 是 C。

介绍一下组的方法，示例代码：

```java
public class Groups {

	static public final String POEM =
			"白日依山尽\n"+
	        "黄河入海流\n"+
		    "欲穷千里目\n"+
	        "更上一层楼";

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Matcher matcher = Pattern.compile("(?m)(\\S+)\\s+((\\S+)\\s+(\\S+))$").matcher(POEM);

		while (matcher.find()) {
			//获取分组的数目
			Sys
			for (int i = 0; i < matcher.groupCount(); i++) {
				//返回前一次匹配成功的组号的内容，如果是空则返回null
				System.out.println(matcher.group(i));
			}
		}
	}

}
```
执行结果：
```
白日依山尽
黄河入海流
欲穷千里目
白日依山尽
黄河入海流
欲穷千里目
黄河入海流
```
此段内容由明白的可以来探讨一下。

###### start() 和 end()
修改上面的代码：
```java
public class Groups {

	static public final String POEM =
			"World  shi chao\n"+
	        "Ni shi shui \n"+
		    "Ni guan wo \n"+
	        "Ha ha ha ";

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Matcher matcher = Pattern.compile("(?m)(\\S+)\\s+((\\S+)\\s+(\\S+))$").matcher(POEM.toString());
		System.out.println(matcher.groupCount()+"");
		while (matcher.find()) {
			//获取分组的数目
			for (int i = 0; i < matcher.groupCount(); i++) {
				//返回前一次匹配成功的组号的内容，如果是空则返回null
				System.out.println(matcher.group(i) + "start:" + matcher.start()+"-"+"end:"+matcher.end());
			}
		}
	}

}

```
执行结果：
```
4
World  shi chaostart:0-end:15
Worldstart:0-end:15
shi chaostart:0-end:15
shistart:0-end:15
```
start() 返回先前匹配的起始位置的索引，而 end() 返回所匹配的最后字符的索引加 1.

```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		Matcher matcher = Pattern.compile("(?m)(\\S+)\\s+((\\S+)\\s+(\\S+))$").matcher(POEM.toString());
		System.out.println(matcher.groupCount()+"");
		while (matcher.matches()) {
			//获取分组的数目
			for (int i = 0; i < matcher.groupCount(); i++) {
				//返回前一次匹配成功的组号的内容，如果是空则返回null
				System.out.println(matcher.group(i) + "start:" + matcher.start()+"-"+"end:"+matcher.end());
			}
		}

		while (matcher.find()) {
			//获取分组的数目
			for (int i = 0; i < matcher.groupCount(); i++) {
				//返回前一次匹配成功的组号的内容，如果是空则返回null
				System.out.println(matcher.group(i) + "start:" + matcher.start()+"-"+"end:"+matcher.end());
			}
		}

		while (matcher.lookingAt()) {
			//获取分组的数目
			for (int i = 0; i < matcher.groupCount(); i++) {
				//返回前一次匹配成功的组号的内容，如果是空则返回null
				System.out.println(matcher.group(i) + "start:" + matcher.start()+"-"+"end:"+matcher.end());
			}
		}
	}
```
注意：find() 可以在输入的任意位置定位正则表达式，而 lookingAt() 只要输入的第一部分匹配成功就成功。 matchers() 只有整个输入都匹配才会成功。

###### Pattern 标记
Pattern 类的 compile() 方法还有另外一个版本，它接受一个标记参数，以调整匹配的行为：

```java
public static Pattern compile(String regex, int flags) {
			 return new Pattern(regex, flags);
	 }
```
其中的 flag 来自以下 Pattern 类的常量：

###### split()
split() 方法将输入字符串断开成字符串对象的数组，断开边界由正则表达式确定：

```java
public class SplitDemo {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		String input = "wo!shi chao! chao!!hahaahchao!!h";
		System.out.println(Arrays.toString(Pattern.compile("!").split(input)));

		System.out.println(Arrays.toString(Pattern.compile("!").split(input,3)));
	}

}

```
执行结果：
```
[wo, shi chao,  chao, , hahaahchao, , h]
[wo, shi chao,  chao!!hahaahchao!!h]
```
后面一个方法多了一个参数，表示要断开为多少个字符串对象。

#### 替换操作
正则表达式特别方便与替换文本，并提供了很多的方法：

- replaceFirst(String s) 以参数字符串替换掉第一个匹配的地方

- replaceAll(String s) 以参数字符串替换掉所有匹配的地方

- appendReplacment(StringBufer sbuf,String s) 允许你调用其他方法来生成或处理 s。使你更够以编程的方式将目标分成组。从而具备更强大的功能。

- appendTail(StringBuffer sbuf) 在执行appendReplacment() 方法之后可以将剩余的字符串复制到 sbuf 中。

```java
public class TheReplacements {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		String input = "wo!shi chao! chao";
		Matcher matcher =Pattern.compile("[a-c]").matcher(input);
		String string = matcher.replaceAll("Q");
		System.out.println(string);

		Pattern pattern = Pattern.compile("[ab]");
		StringBuffer buffer = new StringBuffer();
		Matcher matcher2 = pattern.matcher(input);

		while (matcher2.find()) {
			matcher2.appendReplacement(buffer, "我");
			matcher2.appendTail(buffer);
			System.out.println(buffer+"   ");
		}


	}

}
```
执行结果：
```
wo!shi QhQo! QhQo
wo!shi ch我o! chao   
wo!shi ch我o! chaoo! ch我o   
```
#### reset()
通过 reset() 方法可以将现有的 Matcher 对象应用于一个新的字符序列，不带参数的方法可以重新设置到当前字符序列的起始位置。

```java
public class RestIng {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Matcher matcher = Pattern.compile("[abc]").matcher("abcdefg");
		while (matcher.find()) {
			System.out.print(matcher.group());

		}

		matcher.reset("efghijklabcdg");
		while (matcher.find()) {
			System.out.print(matcher.group());
		}
	}

}
```
执行结果：
```
abcabc
```

## 扫描输入

```java
public class SimpleRead {

	private static BufferedReader bReader = new BufferedReader(new StringReader("李文革  哈哈ad\n22 1.7"));
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		System.out.println("你的名字？");
		Scanner scanner = new Scanner(bReader);
		System.out.println(scanner.nextLine());
		System.out.println("介绍一下你名字后边是啥？");
		int age = scanner.nextInt();
		double height = scanner.nextDouble();
		System.out.println(age);
		System.out.println(height);


	}

}

```
执行结果：
```
你的名字？
李文革  哈哈ad
介绍一下你名字后边是啥？
22
1.7

```
Scanner 的构造器可以接受任何类型的输入对象。所有的 next() 方法只有在找到一个完整的分词之后才会返回。Scanner 还有相应的 HasNext　方法，用于判断下一个分词是否是所需的类型。

#### Scanner 定界符
默认情况下 Scanner 用空白来划分定界符，但是也可以用正则表达式来指定自己需要的定界符。

```java
public class ScannerDelimiter {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Scanner scanner = new Scanner("12, 42, 78, 99, 42");
		scanner.useDelimiter("\\s*,\\s*");
		while (scanner.hasNextInt()) {
			System.out.println(scanner.nextInt());
		}
	}

}

```
执行结果：
```
12
42
78
99
42
```
这个例子使用「,」号作为定界符。

#### 正则表达式用于扫描
除了可以扫描基本类型之外，你还可以使用自定义的正则表达式进行扫描，这在扫描复杂数据的时候非常有用。

```java
public class ThreatAnalyzer {

	static String threatData =
			"58.27.82.161@02/10/2005\n"+
					"58.27.82.161@02/10/2005\n"+
					"58.27.82.161@02/10/2005\n"+
					"58.27.82.161@02/10/2005\n"+
					"Next log ";

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Scanner scanner = new Scanner(threatData);
		String pattern = "(\\d+[.]\\d+[.]\\d+[.]\\d+)@(\\d{2}/\\d{2}/\\d{4})";

		while (scanner.hasNext(pattern)) {
			scanner.next(pattern);
			MatchResult result = scanner.match();
			String ip = result.group(1);
			String data = result.group(2);

			System.out.format("Thread on %s form %s\n",data,ip);
		}
	}

}

```
执行结果：
```
Thread on 02/10/2005 form 58.27.82.161
Thread on 02/10/2005 form 58.27.82.161
Thread on 02/10/2005 form 58.27.82.161
Thread on 02/10/2005 form 58.27.82.161
```
当 next() 方法配合正则表达式的时候，将匹配到下一个该模式的输入部分，调用 match() 方法就可以获取匹配的结果。在配合正则表达式使用扫描的时候，有一点注意：他仅仅针对输入的下一个分词进行匹配。你的正则表达式含有定界符那么可能永远匹配不成功。

## StringTokenizer
在 Java 引入正则表达式和 Scanner 类之前，分割字符的唯一方法是使用 StringTokenizer 来分词。

```java
public class ReplacingStringTokenizer {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		String input = "But I'm not dead yet!";
		StringTokenizer tokenizer = new StringTokenizer(input);
		while (tokenizer.hasMoreElements()) {
			System.out.print(tokenizer.nextToken()+ " ");

		}
		System.out.println();
		System.out.print(Arrays.toString(input.split(" ")));
		System.out.println();
		Scanner scanner = new Scanner(input);
		while (scanner.hasNext()) {
			System.out.print(scanner.next()+ " ");
		}
	}

}

```
执行结果：
```
But I'm not dead yet!
[But, I'm, not, dead, yet!]
But I'm not dead yet!
```
## 总结
Java 中字符串的操作我们可以看到很丰富。特别是正则表达式的操作不好记忆。还需要多多熟练才好。另外字符串注意性能上的问题，恰当的使用 StringBuild 类。
