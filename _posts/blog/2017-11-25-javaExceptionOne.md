---
layout: post
title: java编程思想之通过异常处理错误初级
categories: javaThinking
description: java编程思想之通过异常处理错误
keywords: java异常, java编程思想之通过异常处理错误, java通过异常处理错误
---

发现错误的理想时机是在编译阶段，也就是在你程序运行之前。然而编译期并不能找出所有的错误，余下的问题必须在运行期间解决。这就需要错误元通过某种方式，把适当的信息传递给某个接受者——改接受者知道如何正确的处理这个问题。

Java 中的异常处理的目的在于通过使用少于目前数量的代码来简化大型，可靠的程序的生成。并且通过这种方式使你的程序更佳的自信。你的应用中理想的状态是没有未处理的错误。

## 概念
如果在每次调用方法的时候都需要对方法进行彻底的错误检查，代码可能会变得难以阅读。对于构建大型、健壮、可维护的程序而言，这种错误处理模式已经成为了主要的障碍。既然早期的处理模式非常难以接受，那么现在的处理模式的解决办法是，用强制规定的形式来消除错误处理过程中随心所欲的因素。

我们使用异常来描述错误，使用异常带来一个好处，他能够降低处理错误代码的复杂度。不必须去检查特定的错误。使用异常我们不必在方法的调用处进行检查，因为异常机制将保证能够捕获到这个错误。并且我们只需要在一个地方处理错误，即异常处理程序中。这会节省我们的代码，而且把正常的代码和出了问题的代码相分离。总之，与以前的错误处理代码相比，异常机制使得代码的阅读、编写、和调试工作更加井井有条。

## 基本异常
异常情形是指阻止当前方法或作用域继续执行的问题。普通问题是指在当前环境下能够得到足够的信息，总能处理这个错误。对于异常情形就不能继续下去，因为在当前环境下无法获得必要的信息来解决这个问题。所能做的就是在当前环境下跳出，并把问题交给上一节环境。这就是抛出异常时发生的所有的事情。

举一个抛出异常的简单例子。对于对象引用 t。传给你的时候可能还未被初始化。所以在使用这个对象引用调用方法之前，会先对引用进行检查。可以创建一个代表错误信息的对象，并且将它从当前环境中抛出，这样就把错误信息传播到更大的环境中去。这被称为抛出一个异常。

```java
if(t == null)
   throw new NullPointerException();
```

异常可以使得我们把每件事都当做一个事务来考虑。事务是计算机中的合同法，如果出了问题我们就要放弃整个计算。我们也可以将异常看做是内嵌的一种恢复系统，因为我们的程序可以拥有各种不同的恢复点。

#### 异常参数
与 Java 使用其他对象一样，我们也是用 new 在堆上创建异常对象，这也会有存储空间的分配和构造器的调用。所有标准异常类都有两个构造方法：一个是默认构造器；一个是带有参数的构造器。带参数的构造器接受一个字符串参数，以便能够把相关信息放入到对象的构造器。

```java
throw new NullPointerException("t == null");
```
关键字 throw 将会把用 new 创建的异常对象的引用传递过来。然后用抛出的方式从当前的作用域退出，并且返回一个异常的对象。此外，能够抛出任意类型的是 Throwable 对象，它是异常类的根类。通常我们要针对不同类型的错误抛出相应的错误。错误信息可以保存在异常对象的内部或者用异常类的名称来暗示。

## 捕获异常
要明白异常是如何被捕获的，就必须理解监控区域的概念。它是一段可能产生异常的代码，并且后面跟着处理异常的代码。

#### try 块
如果在方法内部抛出异常，这个方法将会在抛出异常后结束。如果不希望方法结束，可以在方法内设置一个特殊的块来捕获异常。因为在这个块里 “尝试” 各种方法调用，所以称为 try 块。

```java
tyr{
  //包含的语句
}
```
#### 异常处理程序
当然，抛出的异常必须在某处得到处理。这个地点就是 “异常处理程序” ，而且针对每个要捕获的异常，得准备相应的处理程序。异常处理程序紧跟在 try 块之后，以关键字 catch 表示：看下面的实例

```java
try{
  //程序代码
} catch(Type1 t){
  //处理错误的代码
} catch(Type12 t){
    //处理错误的代码
}
```
每个 catch 子句看起来好像是接受一个且仅接受一个特殊类型的参数的方法。异常处理程序必须紧跟在 try 块之后。当异常被抛出时候，异常处理机制将会搜寻参数与异常类型相匹配的 catch 子句，然后进去 catch 子句执行，此时认为异常得到了处理。只有匹配的 catch 子句才会被执行。

在 try 的内部许多不同的方法可能会产生相同类型的异常，而我们只需要提供一种针对此类型的异常处理程序即可。

#### 终止与恢复
异常处理程序理论上有两种模型：

- Java 支持终止模型。这种模型下将错误设置为非常关键，一旦异常被抛出，程序就不能继续执行。
- 另外一种恢复模型。意思是异常处理程序的工作是修正错误。然后重新尝试调用问题的方法。并认为第二次可以成功。通常希望调用异常处理程序之后继续执行程序。

长久以来，尽管我们使用的操作系统支持恢复模型的异常处理。但最终还是转向终止模型的代码，别并忽略恢复行为。

## 创建自定义异常
不必拘泥于 Java 中已有的异常类型。 Java 提供的异常体系不可能遇见所有的异常信息。所以我们可以自定义异常类来表示程序中可能遇到的特定问题。

要定义异常类，必须从已有的异常类继承，最好选择意思相近的异常类继承。

```java
public class SimpleException extends Exception{

	public static class ExceptionTest{
		public void  f() throws SimpleException{
			System.out.println("这是一个自定义的异常信息");
			throw new SimpleException();
		}

	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		ExceptionTest test = new ExceptionTest();
		try {
			test.f();
		} catch (SimpleException e) {
			// TODO Auto-generated catch block
			System.out.println("异常被处理");
		}

	}

}

```
执行结果：
```
这是一个自定义的异常信息
异常被处理

```
通常我们会使用 e.printStackTrace() 这会把异常信息发送给标准错误流程。这样会更容易被用户注意到。

上面的自定义异常类其实编译器默认为我们实现了一个构造器。我们也可以为异常类定义一个带有字符串参数的构造器：

```java
public class SimpleException extends Exception{



	protected SimpleException() {
		super();
		// TODO Auto-generated constructor stub
	}

	protected SimpleException(String message) {
		super(message);
		// TODO Auto-generated constructor stub
	}

	public static class ExceptionTest{
		public void  f() throws SimpleException{
			System.out.println("这是一个自定义的异常信息方法f()");
			throw new SimpleException();
		}

		public void  g() throws SimpleException{
			System.out.println("这是一个自定义的异常信息方法g()");
			throw new SimpleException("带参数的构造器");
		}
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		ExceptionTest test = new ExceptionTest();
		try {
			test.f();
		} catch (SimpleException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

		try {
			test.g();
		} catch (SimpleException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}

}

```
执行结果：
```Console
这是一个自定义的异常信息方法f()
exceptionDemo.SimpleException
这是一个自定义的异常信息方法g()
	at exceptionDemo.SimpleException$ExceptionTest.f(SimpleException.java:20)
	at exceptionDemo.SimpleException.main(SimpleException.java:33)
exceptionDemo.SimpleException: 带参数的构造器
	at exceptionDemo.SimpleException$ExceptionTest.g(SimpleException.java:25)
	at exceptionDemo.SimpleException.main(SimpleException.java:40)

```
#### 异常与记录日志
我们可以使用 java.util.logging 工具将输出记录到日志中

```java
public class SimpleException extends Exception{

	//记录异常的日志
	Logger logger = Logger.getLogger("SimpleException");

	protected SimpleException() {
		StringWriter tWriter = new StringWriter();
		printStackTrace(new PrintWriter(tWriter));
		//记录日志的方法
		logger.severe(tWriter.toString());

		// TODO Auto-generated constructor stub
	}

	protected SimpleException(String message) {
		super(message);
		// TODO Auto-generated constructor stub
	}

	public static class ExceptionTest{
		public void  f() throws SimpleException{
			System.out.println("这是一个自定义的异常信息方法f()");
			throw new SimpleException();
		}

		public void  g() throws SimpleException{
			System.out.println("这是一个自定义的异常信息方法g()");
			throw new SimpleException("带参数的构造器");
		}
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		ExceptionTest test = new ExceptionTest();
		try {
			test.f();
		} catch (SimpleException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

		try {
			test.g();
		} catch (SimpleException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}

}

```
我们通常需要捕获的是其他人编写的异常信息。一次我们可以在异常处理程序中记录日志。

```java
public class SimpleException extends Exception{

	//记录异常的日志
	static Logger logger = Logger.getLogger("SimpleException");

	protected SimpleException() {

		// TODO Auto-generated constructor stub
	}

	protected SimpleException(String message) {
		super(message);
		// TODO Auto-generated constructor stub
	}

	public static class ExceptionTest{
		public void  f() throws SimpleException{
			System.out.println("这是一个自定义的异常信息方法f()");
			throw new SimpleException();
		}

		public void  g() throws SimpleException{
			System.out.println("这是一个自定义的异常信息方法g()");
			throw new SimpleException("带参数的构造器");
		}


	}

	static void logException(Exception e){
		StringWriter tWriter = new StringWriter();
		e.printStackTrace(new PrintWriter(tWriter));
		//记录日志的方法
		logger.severe(tWriter.toString());
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		ExceptionTest test = new ExceptionTest();

		try {
			test.g();
		} catch (SimpleException e) {
			// TODO Auto-generated catch block
			logException(e);
		}
	}

}

```
执行结果：
```
这是一个自定义的异常信息方法g()
十一月 25, 2017 5:16:40 下午 exceptionDemo.SimpleException logException
严重: exceptionDemo.SimpleException: 带参数的构造器
	at exceptionDemo.SimpleException$ExceptionTest.g(SimpleException.java:30)
	at exceptionDemo.SimpleException.main(SimpleException.java:48)

```
我们还可以更进一步的自定义异常类：

```java
public class SimpleException extends Exception{

	//记录异常的日志
	static Logger logger = Logger.getLogger("SimpleException");

	private int x;


	protected SimpleException() {

		// TODO Auto-generated constructor stub
	}

	protected SimpleException(String message) {
		super(message);
		// TODO Auto-generated constructor stub
	}

	protected SimpleException(String message,int x) {
		super(message);
		this.x = x;
	}

	@Override
	public String getMessage() {
		// TODO Auto-generated method stub
		return "这是一个错误："+x + "  "+super.getMessage();
	}

	public static class ExceptionTest{
		public void  f() throws SimpleException{
			System.out.println("这是一个自定义的异常信息方法f()");
			throw new SimpleException();
		}

		public void  g() throws SimpleException{
			System.out.println("这是一个自定义的异常信息方法g()");
			throw new SimpleException("带参数的构造器");
		}

		public void  h() throws SimpleException{
			System.out.println("这是一个自定义的异常信息方法h()");
			throw new SimpleException("带参数的构造器",47);
		}


	}

	static void logException(Exception e){
		StringWriter tWriter = new StringWriter();
		e.printStackTrace(new PrintWriter(tWriter));
		//记录日志的方法
		logger.severe(tWriter.toString());
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		ExceptionTest test = new ExceptionTest();
		try {
			test.f();
		} catch (SimpleException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

		try {
			test.g();
		} catch (SimpleException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

		try {
			test.h();
		} catch (SimpleException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}

}

```
执行结果：
```
这是一个自定义的异常信息方法f()
exceptionDemo.SimpleException: 这是一个错误：0  null
这是一个自定义的异常信息方法g()
	at exceptionDemo.SimpleException$ExceptionTest.f(SimpleException.java:39)
	at exceptionDemo.SimpleException.main(SimpleException.java:66)
exceptionDemo.SimpleException: 这是一个错误：0  带参数的构造器
	at exceptionDemo.SimpleException$ExceptionTest.g(SimpleException.java:44)
	at exceptionDemo.SimpleException.main(SimpleException.java:73)
这是一个自定义的异常信息方法h()
exceptionDemo.SimpleException: 这是一个错误：47  带参数的构造器
	at exceptionDemo.SimpleException$ExceptionTest.h(SimpleException.java:49)
	at exceptionDemo.SimpleException.main(SimpleException.java:80)

```
## 异常说明
Java 鼓励我们把可能会抛出异常的信息告知使用此方法的用户。他使得调用者可以确切的知道写什么样的代码可以捕获所有潜在的异常。Java 提供了一种语法，使得你可以以礼貌的方式告知用户可能抛出的异常类型，然后客户端程序员就可以进行相应的异常处理，这就是异常说明。它属于方法声明的一部分。

使用关键字 throws 后面紧接着异常类型：

```java
void f() throws ExceptionClass{}
```
代码必须与异常说明一致，如果你抛出了异常却没有处理，编译器会提醒你要处理这个异常。这种在编译时被强制检查的异常被称为被检查的异常。

## 捕获所有异常
可以只写一个异常处理程序来捕获所有类型的异常。通过捕获异常类型的基类 Exception 就可以做到这一点。

```java
catch(Exception e){

}
```
这将会捕获所有的异常，所以最好把他放在捕获的末尾。这样可以防止他抢在其他程序之前捕获异常。

Exception 是所有变成异常类的基类，所获得的信息并不多。我们可以从它的从基类 Throwable 继承的方法来调用信息：

```java
String getMessage()
String getLocalizedMessage()
```
用来获取详细信息或是用本地语言描述的详细信息

```java
String toString()
```
包含 Throwable 的详细描述信息

```java
void printStackTrace()
void printStackTrace(PrintStream)
void printStackTrace(java.io.PrintWriter)
```
打印 Throwable 和 它的调用栈轨迹。第一个版本输出标准的错误，后面两个允许选择要输出的流。

```java
Throwable fillInStackTrace()
```
用于在 Throwable 内部记录栈针的状态。

示例：
```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		try {
			throw new Exception("My Exception");
		} catch (Exception e) {
			// TODO Auto-generated catch block

			System.out.println("getMessage" + e.getMessage());
			System.out.println("getLocalizedMessage" + e.getLocalizedMessage());
			System.out.println("toString" + e.toString());
			System.out.println("printStackTrace");
			e.printStackTrace();


		}
	}
```
执行结果：
```
getMessageMy Exception
getLocalizedMessageMy Exception
toStringjava.lang.Exception: My Exception
printStackTrace
java.lang.Exception: My Exception
	at exceptionDemo.ExceptionMethods.main(ExceptionMethods.java:8)
```
每个方法都提供了比前一个更多的信息，其实后一个是前一个超集。

#### 栈轨迹
printStackTrace() 方法所提供的的元素可以通过 getStackTrace() 方法来直接访问，这个方
法将返回一个由栈轨迹元素所组成的一个数组。其中的每一个元素都表示栈中的一帧。

```java
public class ExceptionMethods {

	static void f(){
		try {
			throw new Exception("My Exception");
		} catch (Exception e) {
			// TODO Auto-generated catch block
		   for (StackTraceElement string : e.getStackTrace()) {
			System.out.println(string.getMethodName());
		}
	 }
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		f();
	}

}

```
执行结果：
```
f
main
```
#### 重新抛出异常
有时我们希望刚捕获的异常重新抛出，尤其是在使用 Exception 捕获所有异常的时候。

```java
catch(Exception e){
  throw e;
}
```
重新抛出异常会把异常传给上一级的异常处理程序。同一个 try 块的后续 catch 子句将会被忽略。
此外，异常对象的所有信息都将会保持，所以更高一级环境也可以处理异常对象的所有信息。如果我
们只把异常对象抛出，那么 printStackTrace() 方法显示的是原来异常抛出点的信息。要想更新
这个信息，可以调用 fillInStackTrace() 方法。将会返回一个 Throwable 对象，他是通过把
当前的调用栈信息填入到那个异常对象而建立的。

```java
public class Rethrowing {

	public static void f() throws Exception {
		System.out.println("调用f()");
		throw new Exception("抛出f()的异常");
	}

	public static void g() throws Exception {
		try {
			f();
		} catch (Exception e) {
			// TODO Auto-generated catch block
			System.out.println("调用方法g()抛回上一级异常");
			e.printStackTrace();
			throw e;
		}
	}


	public static void h() throws Exception {
		try {
			f();
		} catch (Exception e) {
			// TODO Auto-generated catch block
			System.out.println("调用方法h()抛回上一级异常");
			e.printStackTrace();
			throw (Exception)e.fillInStackTrace();
		}
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		try {
			g();
		} catch (Exception e) {
			// TOO Auto-generated catch block
			System.out.println("调用方法Main()处理g()");
			e.printStackTrace();

		}

		/*try {
			h();
		} catch (Exception e) {
			// TOO Auto-generated catch block
			System.out.println("调用方法Main()处理h()");
			e.printStackTrace();

		}*/
	}

}

```
第一种执行结果：
```
调用f()
调用方法g()抛回上一级异常
java.lang.Exception: 抛出f()的异常
调用方法Main()处理g()
	at exceptionDemo.Rethrowing.f(Rethrowing.java:7)
	at exceptionDemo.Rethrowing.g(Rethrowing.java:12)
	at exceptionDemo.Rethrowing.main(Rethrowing.java:36)
java.lang.Exception: 抛出f()的异常
	at exceptionDemo.Rethrowing.f(Rethrowing.java:7)
	at exceptionDemo.Rethrowing.g(Rethrowing.java:12)
	at exceptionDemo.Rethrowing.main(Rethrowing.java:36)
```
第二种执行结果：
```
调用f()
调用方法h()抛回上一级异常
java.lang.Exception: 抛出f()的异常
调用方法Main()处理h()
	at exceptionDemo.Rethrowing.f(Rethrowing.java:7)
	at exceptionDemo.Rethrowing.h(Rethrowing.java:24)
	at exceptionDemo.Rethrowing.main(Rethrowing.java:45)
java.lang.Exception: 抛出f()的异常
	at exceptionDemo.Rethrowing.h(Rethrowing.java:29)
	at exceptionDemo.Rethrowing.main(Rethrowing.java:45)

```
调用 fillInStackTrace() 的哪一行相当于异常的新发生地。

我们在捕获异常的地点可能抛出一个不同的异常。这样的话就相当于使用了 fillInStackTrace()
,有关原来异常发生点的信息会丢失。剩下的是与新的发生点有关的信息。

```java
public class Rethrowing {

	public static void f() throws OneException {
		System.out.println("调用f()");
		throw new OneException("抛出f()的异常");
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		try {
			f();
		} catch (Exception e) {
			// TOO Auto-generated catch block
			System.out.println("调用方法Main()处理f()");
			e.printStackTrace();
			try {
				throw new TwoException("抛出另外的异常类型");
			} catch (TwoException e1) {
				// TODO Auto-generated catch block
				e1.printStackTrace();
			}
		}


	}

}

```
执行结果：
```
调用f()
调用方法Main()处理f()
exceptionDemo.OneException: 抛出f()的异常
	at exceptionDemo.Rethrowing.f(Rethrowing.java:7)
	at exceptionDemo.Rethrowing.main(Rethrowing.java:36)
exceptionDemo.TwoException: 抛出另外的异常类型
	at exceptionDemo.Rethrowing.main(Rethrowing.java:42)
```

#### 异常链

如果在捕获一个异常后抛出了另外一个异常，并且希望把原始异常的信息保存下来。这被称为异常链
。JavaSE5 之后所有的 Throwable 子类都在构造器中接受一个 cause 对象作为参数。这个参数用
来表示原始异常，这样把原始异常传递给新的异常也能通过这个异常链追踪到原始异常。

```java
public class OneException extends Exception{


	protected OneException() {
		super();
		// TODO Auto-generated constructor stub
	}

	protected OneException(String message) {
		super(message);
		// TODO Auto-generated constructor stub
	}


}

```
第二个类型
```java
public class TwoException extends Exception{


	protected TwoException() {
		super();
		// TODO Auto-generated constructor stub
	}

	protected TwoException(Throwable cause) {
		super(cause);
		// TODO Auto-generated constructor stub
	}

	protected TwoException(String message) {
		super(message);
		// TODO Auto-generated constructor stub
	}


}

```
调用并抛出异常：
```java
public class Rethrowing {

	public static void f() throws OneException {
		System.out.println("调用f()");
		throw new OneException("抛出f()的异常");
	}

	public static void g() throws TwoException {
		try {
			f();
		} catch (OneException e) {
			// TODO Auto-generated catch block
			throw new TwoException(e);
		}
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		try {
			g();
		} catch (Exception e) {
			// TOO Auto-generated catch block
			e.printStackTrace();
		}


	}

}

```
执行结果：
```
调用f()
exceptionDemo.TwoException: exceptionDemo.OneException: 抛出f()的异常
	at exceptionDemo.Rethrowing.g(Rethrowing.java:15)
	at exceptionDemo.Rethrowing.main(Rethrowing.java:34)
Caused by: exceptionDemo.OneException: 抛出f()的异常
	at exceptionDemo.Rethrowing.f(Rethrowing.java:7)
	at exceptionDemo.Rethrowing.g(Rethrowing.java:12)
	... 1 more

```
上边可以看到，第二次的打印结果和上边一样。最后一个其实是被控制台隐藏了。如果是自定义的类
。我们也可以通过一个 initCause() 方法链接起来。

```java
public static void g() throws TwoException {
		try {
			f();
		} catch (OneException e) {
			// TODO Auto-generated catch block
			TwoException twoException = new TwoException();
			twoException.initCause(e);
			throw twoException;
		}
	}
```
## Java 标准异常
Throwable 这个 Java 类被用来表示可以作为异常被抛出的类。 Throwable 对象可以分为两种：

- Error 用来表示编译时和系统错误，一般不用关系。
- Exception 是可以被抛出的基本类型，在 Java 类库，用户方法和运行时故障都可能抛出此异常。
这正是我们所关心的。

#### 特例：RuntimeException
属于运行时异常的种类很多，他们会被 Java 虚拟机自动的抛出。这些异常都是从 RuntimeException
继承而来的。我们不需要再异常说明中抛出 RuntimeException 异常。此类异常是不受检查的。这属于
我们的编码错误。如果不捕获此类异常，它也会穿越所有执行的方法来到 main() 方法，而不会被捕获。

```java
public class NeverCaught {

	static void f(){
		throw new RuntimeException("方法f()抛出");
	}


	public static void main(String[] args) {
		// TODO Auto-generated method stub
			f();
	}

}

```
执行结果：
```
Exception in thread "main" java.lang.RuntimeException: 方法f()抛出
	at exceptionDemo.NeverCaught.f(NeverCaught.java:6)
	at exceptionDemo.NeverCaught.main(NeverCaught.java:12)
```
RuntimeException 异常没有被捕获，在程序退出之前将会调用 printStackTrace()。

注意：只能在代码中忽略 RuntimeException 异常，其他异常的处理都由编译器强制实施。这是因为
RuntimeException 异常代表的是变成错误。

- 无法预料到的错误，比如 null 的引用。
- 作为程序员应该在代码中检查错误。
