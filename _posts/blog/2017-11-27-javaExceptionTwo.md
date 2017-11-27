---
layout: post
title: java编程思想之通过异常处理错误深入
categories: javaThinking
description: java编程思想之通过异常处理错误
keywords: java异常, java编程思想之通过异常处理错误, java通过异常处理错误
---

我们知道了异常如何的进行拦截和获取以及处理。但是有可能还会有不能够处理的。

## 使用 finally 进行清理
对于一些代码我们可能希望无论 try 块中的异常是否抛出，他们都能够得到执行。通常应用了内存
回收之外的情况。为了达到这个效果可以在异常处理程序后面加上 finally 子句。

```java
public static void main(String[] args) {
		try {
			// TODO Auto-generated method stub
			f();
		} catch (Exception e) {
			// TODO: handle exception
			e.printStackTrace();
		}finally {
			//需要必须执行的代码
			System.out.println("处理了异常我也要执行");
		}

	}
```
执行结果：
```
处理了异常我也要执行
java.lang.RuntimeException: 方法f()抛出
	at exceptionDemo.NeverCaught.f(NeverCaught.java:6)
	at exceptionDemo.NeverCaught.main(NeverCaught.java:13)
```
可以看出无论你的异常是拦截了还是未被拦截，finally 都会被执行。

#### finally 用来做什么
finally 非常重要。他能够使程序员保证：无论 try 块里发生了什么，内存总能得到释放。Java 有
垃圾回收机制，所以内存释放不再是问题。当要把除内存之外的资源恢复到初始状态，那就要用到我们
的 finally 子句。这种需要清理的资源包括：已经打开的文件或网络，在屏幕上绘制的图形，甚至可以
是外部世界的某个开关。

```java
public class Switch {

	private boolean state = false;

	public boolean read() {
		return state;
	}

	public void on() {
		state = true;
		System.out.println(this);
	}

	public void off() {
		state = false;
		System.out.println(this);
	}

	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return state ? "on" : "off";
	}
}

```
异常类：
```java
public class OnOffException extends Exception{

}

public class OffOnException extends Exception{

}
```
调用开关抛出异常：
```java
public class OnOffSwitch {

	private static Switch sw = new Switch();

	public static void  f() throws OneException,OffOnException{

	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
			try {
				sw.on();
				f();
				sw.off();
			} catch (OneException | OffOnException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
				sw.off();
			}finally {
				sw.off();
			}
	}

}

```
执行结果：
```
on
off
off

```
我们要确保的是 main() 结束的时候开关必须是关闭的，所以在每个 try 块和异常处理程序的末尾
都加入对 sw.off() 调用。但是有可能我们抛出的异常没被捕获就可能不会被调用。我们只有把方法
放在 finally 中才能保证代码必须得到执行。即使当前异常处理机制没有捕获异常，异常处理机制也
会在跳到更高一层的异常处理程序之前，执行 finally 子句。当涉及到 break 和 continue 语句的
时候，finally 子句也会得到执行。

#### return 中使用 finally
因为 finally 子句总是会被执行，所以在一个方法中，可以从多个点返回，并且可以保证清理工作
仍会执行。

```java
public class MultipleReturns {

	public static void f(int i) {
		try {
			if (i == 1) {
				return;
			}
			if (i == 2) {
				return;
			}
			if (i == 3) {
				return;
			}
			if (i == 4) {
				return;
			}
			return;
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}finally {
			System.out.println("清理垃圾");
		}
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
			for (int i = 0; i <5; i++) {
				f(i);
			}
	}

}

```
执行结果：
```
清理垃圾
清理垃圾
清理垃圾
清理垃圾
清理垃圾

```
在 finally 内部，从何处返回无关紧要。

#### 异常丢失
Java 异常实现也有瑕疵。异常作为程序出错的标志，绝不应该被忽略，但还是有可能被忽略。

修改之前的一个例子：

```java
public class OnOffSwitch {

	private static Switch sw = new Switch();

	public static void  f() throws OneException{
		throw new OneException();
	}

	public static void  g() throws TwoException{
		throw new TwoException();
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
			OnOffSwitch switch1 = new OnOffSwitch();

			try {
				try {
					switch1.f();
				} finally {
					switch1.g();
				}
			} catch (Exception e) {
				e.printStackTrace();
			}
	}

}

```
执行结果：
```
exceptionDemo.TwoException
	at exceptionDemo.OnOffSwitch.g(OnOffSwitch.java:12)
	at exceptionDemo.OnOffSwitch.main(OnOffSwitch.java:23)
```
从输出的结果看 OneException 被 finally 子句里的 switch1.g() 给取代了。这是很严重的缺陷。
因为异常会被一种很微妙的方式完全丢失。

一种更加简单的丢失异常的方法是从 finally 子句中返回：

```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
			OnOffSwitch switch1 = new OnOffSwitch();

			try {
				try {
					switch1.f();
				} finally {
					return;
				}
			} catch (Exception e) {
				e.printStackTrace();
			}
	}
```
执行结果是没有抛出任何异常，也不会产生任何的输出。

## 异常的限制
当覆盖方法的时候，只能抛出基类方法的异常说明里列出的异常。意味着当基类使用的代码应用到其他
的派生类对象的时候，一样能够工作，异常也不例外。

异常类：
```java
public class StormException extends Exception{

}

public class RainedOut extends StormException{

}

public class BaseException extends Exception{

}

public class Strike extends BaseException{

}

public class Foul extends BaseException{

}

public class PopFoul extends Foul{

}
```
抽象类：
```java
public abstract class Inning {

	public Inning() throws BaseException{
		// TODO Auto-generated constructor stub
	}

	public void event() throws BaseException{

	}

	public abstract void atBat() throws Foul, Strike;

	public void  walk() {

	}

}
```
接口：
```java
public interface Storm {
	public void  event() throws RainedOut;
	public void  rainHard() throws RainedOut;
}
```
实现类：
```java
public class StormyInning extends Inning implements Storm{

	public StormyInning() throws RainedOut,BaseException {
		super();
		// TODO Auto-generated constructor stub
	}

	public StormyInning(String s) throws Foul,BaseException {

		// TODO Auto-generated constructor stub
	}

	@Override
	public void rainHard() throws RainedOut {
		// TODO Auto-generated method stub

	}

	@Override
	public void atBat() throws PopFoul {
		// TODO Auto-generated method stub

	}

	@Override
	public void event() {
		// TODO Auto-generated method stub

	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		try {
			StormyInning inning = new StormyInning();
			inning.atBat();
		} catch (PopFoul e) {
			// TODO Auto-generated catch block
			System.out.println("PopFoul");
		} catch (RainedOut e) {
			// TODO Auto-generated catch block
			System.out.println("RainedOut");
		} catch (BaseException e) {
			// TODO Auto-generated catch block
			System.out.println("BaseException");
		}

		try {
			Inning in = new StormyInning();
			in.atBat();
		} catch (RainedOut e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (Foul e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (Strike e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (BaseException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}

}
```
在抽象类中，可以看到构造器和 event() 方法都声明了抛出异常，但实际上并没有抛出。这种方式会
强制用户去捕获可能在覆盖后的 event() 版本中增加异常，所以这很合理。对于抽象方法 atBat()
同样的成立。

接口 Storm 包含了一个在 Inning 中定义的方法 event() 和一个不再接口中定义的方法。这两个方
法都会抛出新的异常 RainedOut。如果新的类在继承了 Inning 同时又实现了接口 Storm 接口，那么
接口中的 event() 方法就不能返回和 Inning 中不同的异常类型。否则就不能判断是否捕获了正常的
异常。如果接口中的方法和基类不一样那就没有必要。

异常限制对构造器不起作用。我们的实现类构造器可以抛出任何的异常，而不必理会其基类抛出的异常。
但是，基类构造器可以被调用，所以派生类构造器的异常说明必须包含基类构造器的异常说明。派生类
构造器不能捕获基类构造器抛出的异常。

如果基类中的方法没有抛出异常，那么派生类实现这个方法也不能抛出异常。还有一种情况，即使是基类
的方法抛出了异常，派生类的实现方法也可以不抛出异常。如果派生类的实现方法抛出的是基类的子异常
。那么这两个异常都会被捕获。

最后在 main() 方法中。如果处理的是派生类的对象调用的 atBat() 方法。那么它只会不会当前类的
异常。如果是基类的对象调用的。那么它会捕获派生类和基类的异常。

尽管在继承过程中，编译器会对异常进行强制的说明，但异常说明本身不属于方法的类型的一部分。方法
类型是由方法名与参数类型构成的。因此，不能基于异常说明来重载方法。此外，一个出现在基类方法的
异常说明，不一定会出现在派生类的异常说明。

## 构造器
异常发生了所有的东西都能够被正确的清理吗？涉及构造器的时候会有特殊的情况，构造器会把对象设置
为安全的状态，但还会有其他的操作，比如打开一个文件，这样的动作只有对象使用完毕，用户调用了
清理动作才会被清理。如果在构造器内抛出异常，这些行为也许就不会执行。编写构造器时一定要小心。

如果我们使用 finally。也可能会有问题，因为 finally 总是会被执行。如果构造器在其执行过程
中半途而废，也许该对象的某些部分还没有被创建，却已经被清理。

```java
public class InputFile {

	private BufferedReader in;

	public InputFile(String name) throws Exception {
		// TODO Auto-generated constructor stub
		try {
			in = new BufferedReader(new FileReader(name));

		} catch (FileNotFoundException e) {
			// TODO Auto-generated catch block
			throw e;
		} catch (Exception e) {
			// TODO: handle exception
			try {
				in.close();
			} catch (IOException e1) {
				// TODO Auto-generated catch block
				e1.printStackTrace();
			}
			throw e;
		}finally {

		}

	}


	public String getLine() {

		String string;
		try {
			string = in.readLine();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			throw new RuntimeException();
		}
		return string;
	}

	public void  dispose() {

		try {
			in.close();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			throw new RuntimeException();
		}
	}


}

```
InputFile 构造器接受字符串作为参数，该字符串表示所要打开的文件名。如果 FileReader 构造
失败，将会抛出 FileNotFoundException 异常。对于这个异常不需要关闭文件，因为文件还没有被
打开。而其他捕获异常的 catch 子句必须关闭，因为文件已经打开。这时就需要把方法分别放在各自
的 try 块中。close() 方法也可能抛出异常，所以必须在加一个捕获子句。在处理完成之后异常被
重新抛出。因为如果不抛出，可能会误导调用方认为这个对象已经创建完成了。

由于 finally 会在每次构造之后都调用一遍，因此它不应该调用 close()。getLine() 方法会返回
表示文件下一行的内容字符串。他调用了 readLine() 方法，并且在方法内对异常进行了处理，因此
不抛出任何的异常。

用户在不需要 InputFile 对象时，必须调用 dispose() 方法，这将释放 BufferedReader 以及
和 FileReader 对象所占用的资源。或许你考虑将上述功能放在 finalize() 里边。但是我们不知道
 finalize() 会不会被调用。这是 Java 的缺陷，除了清理内存之外，所有的清理都不会自动的发生。

 对于在构造阶段可能会抛出异常，并且必须要求清理的类，安全的方式是使用嵌套的 try 块。

 ```java
 public class Cleanup {
	static InputFile inputFile;
	public static void main(String[] args) {

		try {
			inputFile = new InputFile("aa.java");
			try {
				String string;
				int i = 1;
				while ((string = inputFile.getLine()) != null) {

				}
			} catch (Exception e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		} catch (Exception e) {
			// TODO Auto-generated catch block
			inputFile.dispose();
		}

	}



}

 ```
 我们看到 finally 子句在构建失败的时候是不会被执行的，构建成功的时候总是会被执行的。这种
 方式在构造器不抛出任何异常时也应该运用。

## 异常匹配
 抛出异常的时候，异常处理程序会按照代码的书写顺序找出最近的处理程序。找到匹配的处理程序之后
 他会认为异常将会得到处理，然后就不会继续寻找。查找匹配的时候并不要求异常同处理程序所声明
 的异常完全匹配。派生类的对象也可以匹配其基类的处理程序。

 ```java
 public class Human {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		try {
			throw new Sneeze();
		} catch (Sneeze e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}catch (Annoyance e) {
			// TODO: handle exception
			e.printStackTrace();
		}

		try {
			throw new Sneeze();
		} catch (Annoyance e) {
			// TODO: handle exception
			e.printStackTrace();
		}

	}

}
 ```
 Sneeze 异常会被第一个 catch 子句捕获，如果将第一个去掉，只留下 Annoyance 程序仍然可以
 运行。因此这次捕获的是基类。换句话说，catch 子句会捕获 Annoyance 以及所有从它派生出来的
 类。如果把捕获基类的 catch 子句放在最前边，一次想把派生类的异常给屏蔽掉。这样编译器会发现
 Sneeze 子句永远得不到执行，因此会向你报错。

##  被检查异常
 异常处理系统就像一个活门，使得你能够放弃程序的正常执行序列。开发异常处理系统的原因是，如果
 为每个方法所发生的的错误都进行处理的话，任务会显得过于繁重。异常处理的一个重要目标是把错误
 代码通错误发生地点相分离。这使得你在一段代码中专注于要完成的事情，置于如何处理错误代码则放
 在另外一段代码中。

 “被检查异常” ：讨论这个问题是因为他强制你在可能还没有准备好处理错误的时候被迫加上 catch
 子句。这回导致吞噬错误的问题。

 ```
 try{
   //可能会有问题，而且不知道如何处理。不知道先 try 起来吧。
 }catch(){

 }
 ```
我们发现如果我们一旦这么做了虽然程序可能会正常的运行了。但是如果导致了运行的过程中出现了
问题，我们又发现不了这个就很难的处理。找到这类的异常是非常困难的。

#### 把异常传递给控制台
我们可以把他们从 main() 方法传递给控制台。

```java
public class Cleanup {
	static InputFile inputFile;
	public static void main(String[] args) throws Exception{

			inputFile = new InputFile("aa.java");
			inputFile.dispose();

	}

}
```
执行结果：
```
Exception in thread "main" java.io.FileNotFoundException: aa.java (系统找不到指定的文件。)
	at java.io.FileInputStream.open0(Native Method)
	at java.io.FileInputStream.open(FileInputStream.java:195)
	at java.io.FileInputStream.<init>(FileInputStream.java:138)
	at java.io.FileInputStream.<init>(FileInputStream.java:93)
	at java.io.FileReader.<init>(FileReader.java:58)
	at exceptionDemo.InputFile.<init>(InputFile.java:15)
	at exceptionDemo.Cleanup.main(Cleanup.java:9)

```

#### 转换为不检查异常
如果我们不知道该怎样处理这个异常，但是又不想把他吞噬。异常链给我们提供一种解决办法。可以把
这个被检查异常包装进 RuntimeException 里面。

```java
try{
  //
}catch(DontWhatCheckException e){
  throw new RuntimeException(e);
}
```

#### 异常链的处理
我们可以不写 try catch 子句，直接忽略异常。让他们沿着调用栈往上冒泡。同时，可以用异常链
的 cause 捕获并处理特定的异常。

```java
public class WarpCheckedException {

	void throwRuntimeException(int type){
		try {
			switch (type) {
			case 0:
				throw new FileNotFoundException();
			case 1:
				throw new IOException();
			case 2:
				throw new RuntimeException("什么异常？");
			default:
				return;
			}
		} catch (Exception e) {
			// TODO Auto-generated catch block
			throw new RuntimeException(e);
		}
	}
}

```
其他异常：
```java
public class SomeOtherException extends Exception{

}
```
调用并执行：
```java
public class TurnOffChecking {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		WarpCheckedException exception = new WarpCheckedException();
		exception.throwRuntimeException(3);

		for (int i = 0; i < 4; i++) {
			try {
				if (i <3) {
					exception.throwRuntimeException(i);
				} else {
					throw new SomeOtherException();
				}
			} catch (SomeOtherException e) {
				// TODO Auto-generated catch block
				System.out.println("SomeOtherException");
			}catch (RuntimeException e) {
				// TODO: handle exception
				try {
					throw e.getCause();
				} catch (FileNotFoundException e1) {
					// TODO Auto-generated catch block
					System.out.println("FileNotFoundException");
				}catch (IOException e2) {
					// TODO: handle exception
					System.out.println("IOException");
				}catch (Throwable e2) {
					// TODO: handle exception
					System.out.println("Throwable");
				}
			}
		}
	}

}

```
执行结果：
```
FileNotFoundException
IOException
Throwable
SomeOtherException
```

## 异常使用指南
应该在下列情况下使用异常：
- 再恰当的级别处理问题。
- 解决问题并且重新调用产生异常的方法。
- 进行少许的修补，然后绕过异常发生的地方继续执行。
- 用别的数据进行计算，以代替方法预计会返回的值。
- 把当前运行环境下能做的事尽量做完，然后把相同的异常重新抛出到更高层。
- 把当前环境下能做的事尽量做完，然后把不同的异常重新抛出到更高层。
- 终止程序。
- 进行简化。
- 让类库和程序更安全。

## 总结
异常是 Java 设计程序不可分割的一部分，如果不了解他们只能完成很有限的工作。
