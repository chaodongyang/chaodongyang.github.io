---
layout: post
title: java编程思想之类型信息(反射)
categories: javaThinking
description: java编程思想之类型信息
keywords: java, java反射, java编程思想之(反射)，java之类型信息(反射)
---

对 RTTI 来说编译器在编译时打开和检查 .class 文件。而对于反射来说，.class 文件在编译时是
不可获取的，所以是在运行时打开和检查 .class 文件。
## 反射概念

如果不知道某个对象的确切类型，RTTI 可以告诉你。但是有一个限制：这个类型在编译时期必须是
已经知道的，这样才能使用 RTTI 识别它，并利用这些信息做一些有用的事情。但是如果你获取一个
指向并不在你的程序空间中的对象的引用；在编译时期你的程序根本就无法获知这个对象所属的类。
那么怎样才能使用这个类呢？

在传统的编程环境中很少出现这样的情况。但当我们身处于更大规模的编程世界中，在许多重要的情况
下就会发生上面的事情。反射提供了一种机制，用来检查可用的方法，并返回方法名。我们在运行时
获取类的另一个原因是，希望提供跨网络的远程平台上创建和运行对象的能力。这被称为远程方法调用
，它允许一个 Java 程序将对象分布到多台机器上。

Class 类与 java.lang.reflect 类库一起对反射的概念提供支持，该类库包含 Field、Method、
以及 Constructor 类。这些类型的对象是由 JVM 在运行时创建的，用来表示未知类里对应的成员。
这样你就可以使用 Constructor 创建新的对象，用 get() 和 set() 方法读取和修改 Field 对象
关联的字段，用 invoke() 方法调用与 Method 对象关联的方法。另外还可以调用 getFields()
、getMethods() 和 getConstructors() 等很多便利的方法，以返回表示字段、方法和构造器的对象
的数组。这样匿名对象的类信息就能在运行时被完全确定起来。而在编译时不需要知道任何事情。

反射机制其实并没有神奇之处。当通过反射与一个未知的类型的对象打交道时，JVM 只是简单的检查
这个对象，看它属于那个特定的类。在用它做其他事情之前必须先加载那个类的 Class 对象。因此，
那个类的 .class 文件对于 JVM 来说必须是可获取的：要嘛在本地机器上，要嘛可以通过网络获取。
所以 RTTI 和反射之间真正的区别只在于，对 RTTI 来说编译器在编译时打开和检查 .class 文件。
而对于反射来说，.class 文件在编译时是不可获取的，所以是在运行时打开和检查 .class 文件。

#### 类方法提取器
通常我们并不需要直接使用反射工具，但是反射在我们需要创建更加动态的代码时会很有用。反射在
Java 中其实是用来支持其他的特性的，例如对象的序列化和 Javabean。但是，能够动态的提取某各类
的信息还是很有用的。使用方法提取器。比如我们浏览一个类的源代码或者开发文档，我们只能看到
这个类中定义的方法或被覆盖的方法。但是可能还有数十个很有用的类都是继承在基类中看不到。反射
提供一种方法可以使我们能够编写可以自动展示完整接口的简单工具。

示例代码：
```java
public class ShowMethods {

	private static String usage =
			    "usage:\n" +
			    "ShowMethods qualified.class.name\n" +
			    "To show all methods in class or:\n" +
			    "ShowMethods qualified.class.name word\n" +
			    "To search for methods involving 'word'";
	private static Pattern pattern = Pattern.compile("\\w+\\.*");

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		if (args.length <1) {
			System.out.println(usage);
			System.exit(0);
		}

		int lines =0;
		try {
			Class<?> class1 = Class.forName(args[0]);
      //获取所有的方法
			Method[] methods = class1.getMethods();
      //获取所有的构造方法
			Constructor[] constructors = class1.getConstructors();

			if (args.length ==1) {
				for (Constructor constructor : constructors) {
					System.out.println(pattern.matcher(constructor.toString()).replaceAll(""));

				}

				for (Method method : methods) {
					System.out.println(pattern.matcher(method.toString()).replaceAll(""));

				}

				lines = methods.length + constructors.length;
			}else {
				for (Constructor constructor : constructors) {
					if (constructor.toString().indexOf(args[1]) != -1) {
						System.out.println(pattern.matcher(constructor.toString()).replaceAll(""));

					}

				}

				for (Method method : methods) {
					if (method.toString().indexOf(args[1]) != -1) {
						System.out.println(pattern.matcher(method.toString()).replaceAll(""));
						lines++;
					}

				}
			}
		} catch (ClassNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}

}

```

Class 的 getMethods() 和 getConstructors() 方法分别返回 Method 对象的数组和 Constructor 对象的数组。这两个对象都提供了更深层次的方法用来解析对象所代表的方法。但是我们用一种简单的方式，只是用 toString() 生成一个包含完整方法特征签名的字符串。Class.forName() 生成的结果在编译时是不可知的，因此所有的方法特征签名信息都是在执行时被提取的。在编程时，特别是如果我们不记得一个类是否有某个方法，或者不知道某各类究竟能做什么。而又不想通过 JDK 去查找文档，这时用这个工具是非常有用的。

## 动态代理
代理是基本的设计模式之一，它为你提供了额外的或不同的操作，而插入的是用来代替实际对象的对象。这些操作通常都涉及到实际对象的通信，因此代理充当着中间人的操作。

接口：
```java
public interface Interface {

	void doSomething();
	void somethingElse(String arg);
}
```
实现类：
```java
public class RealObject implements Interface{

	@Override
	public void doSomething() {
		// TODO Auto-generated method stub
		System.out.println("doSomething");

	}

	@Override
	public void somethingElse(String arg) {
		// TODO Auto-generated method stub
		System.out.println("somethingElse"+arg);
	}

}

```
代理类：
```java
public class SimpleProxy implements Interface{

	private Interface proxied;


	protected SimpleProxy(Interface proxied) {

		this.proxied = proxied;
	}

	@Override
	public void doSomething() {
		// TODO Auto-generated method stub
		System.out.println("SimpleProxy doSomething");
		proxied.doSomething();
	}

	@Override
	public void somethingElse(String arg) {
		// TODO Auto-generated method stub
		System.out.println("SimpleProxy somethingElse"+arg);
		proxied.somethingElse(arg);
	}

}

```
调用：
```java
public class SimpleProxyDemo {

	public static void  consumer(Interface interface1) {
		interface1.doSomething();
		interface1.somethingElse("bobobo");
	}
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		consumer(new RealObject());
		consumer(new SimpleProxy(new RealObject()));
	}

}

```
执行结果：
```
doSomething
somethingElsebobobo
SimpleProxy doSomething
doSomething
SimpleProxy somethingElsebobobo
somethingElsebobobo
```
因为 consumer() 方法接受的是 Interface。所以他无法知道正在获得的到底是 RealObject 还是 SimpleProxy，因为这二者都实现了接口。但是 SimpleProxy 被插入到客户端和 RealObject 之间，因此他会执行操作，然后调用 RealObject 上相同的方法。

在任何时刻只要你想将额外的操作从实际对象中分离到不同的地方。代理就显得很有用。Java 动态代理的思想更向前迈进了一步，因为他可以动态的去创建代理并动态的处理对所代理方法的调用。在动态代理上所做的所有的方法的调用都会被重定向到单一的调用处理器上。它解释调用的类型并不确定调用的对策。

动态代理示例代码：
```java
public class DynamicProxyHandler implements InvocationHandler{

	private Object object;


	protected DynamicProxyHandler(Object object) {

		this.object = object;
	}


	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		// TODO Auto-generated method stub
		System.out.println("*** proxy:" + proxy.getClass() + ", method:" + method + "args:" +args);
		if (args != null) {
			for (Object object : args) {
				System.out.println("   " +object);
			}
		}
		return method.invoke(object, args);
	}



}

```
调用代码：
```
public class SimpleDynamicProxy {

	public static void consumer(Interface interface1) {
		interface1.doSomething();
		interface1.somethingElse("axaxax");
	}


	public static void main(String[] args) {
		// TODO Auto-generated method stub
		RealObject realObject = new RealObject();
		//consumer(realObject);

		Interface proxy = (Interface) Proxy.newProxyInstance(Interface.class.getClassLoader(), new Class[]{Interface.class}, new DynamicProxyHandler(realObject));
		consumer(proxy);
	}

}

```
执行结果：
```
*** proxy:class com.sun.proxy.$Proxy0, method:public abstract void reflect.Interface.doSomething()args:null
doSomething
*** proxy:class com.sun.proxy.$Proxy0, method:public abstract void reflect.Interface.somethingElse(java.lang.String)args:[Ljava.lang.Object;@6bc7c054
   axaxax
somethingElseaxaxax
```
通过调用静态方法 Proxy.newProxyInstance() 可以创建动态代理，这个方法需要得到一个类加载器 (通常可以从已经被加载的对象中获取类加载器)，一个你希望代理实现的接口列表，以及 InvocationHandler 接口的实现类。动态代理可以将所有调用重定向到调用处理器，因此会通常向调用处理器传递一个实际的对象的引用，从而使得调用处理器在执行中介任务时，可以将请求转发。

invoke() 方法中传递进来了代理对象，这是默认传递的，以防你需要区分请求的来源，在许多情况下我们并不关心这一点。然而在 invoke() 内部，在代理上调用方法时需要格外小心，因为对接口的调用将被重定向对代理的调用。

通常我们都是执行被代理的操作，然后使用 Method.invoke() 将请求转发给被代理对象，并传递必要的参数。我们可以传递其他的参数来过滤某些方法调用：

```java
public class DynamicProxyHandler implements InvocationHandler{

	private Object object;

	protected DynamicProxyHandler(Object object) {

		this.object = object;
	}

	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		// TODO Auto-generated method stub
	    if (method.getName().equals("doSomething")) {
	    	System.out.println("调用了 doSomething");
	    }
		return method.invoke(object, args);
	}
}
```
调用：
```java
public class SimpleDynamicProxy {

	public static void consumer(Interface interface1) {
		interface1.doSomething();
		interface1.somethingElse("axaxax");
	}


	public static void main(String[] args) {
		// TODO Auto-generated method stub
		RealObject realObject = new RealObject();
		//consumer(realObject);

		Interface proxy = (Interface) Proxy.newProxyInstance(Interface.class.getClassLoader(), new Class[]{Interface.class}, new DynamicProxyHandler(realObject));
		proxy.doSomething();
	}

}

```
执行结果：
```
调用了 doSomething
doSomething
```
动态代理并非日常使用的工具，但他可以很好的解决某些类型的问题。

## 接口与类型信息
interface 关键字的一种重要目标就是允许程序员个例构件，进而降低耦合性。如果你编写接口，那么就可以实现这一目标，但是通过类型信息，这种耦合度还是会传播出去。接口并非是对解耦的一种无懈可击的保障。

```java
public interface A {

	void f();
}
```
然后实现这个接口：
```java
public class B implements A{

	@Override
	public void f() {
		// TODO Auto-generated method stub

	}

	public void g() {

	}
}

```
调用这个示例：
```java
public class InterfaceViolation {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		A a = new B();
		a.f();
		System.out.println(a.getClass().getName());
		if (a instanceof B) {
			B b = (B)a;
			b.g();
		}
	}

}
```
执行结果：
```
reflect.B
```
通过 RTTI 我们发现 a 是被当做 B 实现的。通过将其转型为 B，我们可以调用不在 A 中的方法。这完全是是合法可以被接受的。但是你也许并不想让客户端程序员这么做。因为这使得给了他们一个机会，使得他们的代码与你的代码耦合。

最简单的方式是使用包访问权限控制：

```java
public class HidddenImp {

	public static void  Methodname(Object cObject,String nethodName) throws Exception{
		//getDeclaredMethods(),该方法是获取本类中的所有方法，包括私有的(private、protected、默认以及public)的方法。
		Method gMethod = cObject.getClass().getDeclaredMethod(nethodName);
		//Java反射机制提供的setAccessible()方法可以取消Java的权限控制检查,使private方法可以被调用
		gMethod.setAccessible(true);

		gMethod.invoke(cObject);
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		A a = HiddenB.meakA();
		a.f();
		System.out.println(a.getClass().getName());

		/*if (a instanceof B) {
			B b = (B)a;
			//只能调用f不能调用g
			b.f();
		}*/

		try {
			Methodname(a,"f");
			Methodname(a,"g");
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}

}
```
执行结果：
```
B.F
reflect.B
B.F
B.G
```
虽然我们无法通过类型转换来调用方法，但是我们看到我们可以通过反射仍然可以调用所有的方法。即便你是私有的内部类或者匿名内部类都无法阻止反射来访问这些方法。但是，final 域实际上在遭遇修改时是安全的。运行时系统会在不抛出已让的情况下接受任何修改尝试，但是实际上不会发生任何改变。

## 总结一些反射的基本用法
如何获取一个对象的类类型，以及如何利用类类型创建对象实例：


#### 荣国反射得到类对象

```java
public class FootClass {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Foot foot = new Foot();
		//类字面常量
		Class class1 = Foot.class;
		//通过对象获取
		Class<Foot> class2 = (Class<Foot>) foot.getClass();
		//动态获取
		try {
			Class class3 = Class.forName("reflect.Foot");
		} catch (ClassNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		//通过类来得到实际的对象
		try {
			foot = (Foot) class1.newInstance();
		} catch (InstantiationException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IllegalAccessException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}

}

```
#### 通过反射得到方法信息

```java
public class FootClass {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Foot foot = new Foot();
		//类字面常量
		Class class1 = Foot.class;

		Method[] methods = class1.getDeclaredMethods();
		for (Method method : methods) {
			//获取参数的返回值类型
			Class returnType = method.getReturnType();
			System.out.println(returnType.getName());

			System.out.println("方法名称"+method.getName());  
            //获取参数类型  
            Class[] paramTypes = method.getParameterTypes();  
            for (Class class2 : paramTypes) {
				System.out.println(class2.getName());
			}


		}
	}

}
```
执行结果：
```
void
方法名称show
void
方法名称show
java.lang.String
```
#### 通过反射得到成员变量信息

```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		Foot foot = new Foot();
		//类字面常量
		Class class1 = Foot.class;
		//成员变量类型集合
		Field[] fields = class1.getDeclaredFields();
		for (Field field : fields) {
			//成员变量的类类型
			Class fClass = field.getType();
			//成员变量的名称
			String name = fClass.getName();

			//System.out.println(fClass);
			System.out.println(name);
		}
	}
```
执行结果：
```
int
int
java.lang.String

```
#### 反射获取类的构造方法的信息

```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		Foot foot = new Foot();
		//类字面常量
		Class class1 = Foot.class;
		//构造函数列表
		Constructor[] constructors = class1.getConstructors();
		for (Constructor constructor : constructors) {
			System.out.println(constructor.getName());

			//获取参数列表

			Class[] patter = constructor.getParameterTypes();
			for (Class class2 : patter) {
				System.out.println(class2.getName());
			}

		}
	}
```
执行结果：
```
reflect.Foot
reflect.Foot
java.lang.String
```
#### 方法的反射操作 invoke() 的使用

```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		Foot foot = new Foot();
		//类字面常量
		Class class1 = Foot.class;
		//获取带参数的方法
		try {
			Method method = class1.getDeclaredMethod("show", String.class);
			method.invoke(foot, "带参数的方法被调用");
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

	}
```
执行结果：
```
显示：带参数的方法被调用
```
