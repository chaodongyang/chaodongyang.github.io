---
layout: post
title: java编程思想之泛型四 (类型安全)
categories: javaThinking
description: java编程思想之泛型
keywords: java, java泛型, java泛型详解，泛型详解
---

泛型中的类型和安全问题是一个重要的内容。Java 泛型中的类型显示其实是一个非常复杂的问题。我们来探讨一下 Java 中的类型限定以及安全和异常问题是非常有必要的！

## 自限定的类型
在 Java 的泛型中经常出现下面的语法例子：
```java
class Self<T extends Self<T>>{}
```
这就像两面镜子互相反射一样。Self 类型接受泛型参数 T。而 T 的边界是由拥有 T 作为其参数的 Self 本身。看到这里很难去解析它，它强调的是 extends 关键字用于边界与用于创建子类明显是不同的。

#### 古怪的循环泛型
为了理解自限定类型的含义，我们县从一个简单的版本入手：它没有自限定的边界，不能直接继承一个泛型参数。
```java
public class GenericType<T> {

}

```
我们继承上面的类：
```java
public class Curious extends GenericType<Curious> {

}
```
这种行为我们称为古怪的循环泛型「CRG」。“古怪的循环” 是指类相当古怪的出现在自己的基类中这一事实。我们可以这样认为：我在创建一个类，它继承自一个泛型类，这个泛型类接受我的类的名字作为其参数。那么此时这个泛型的基类能够做什么呢？我们知道 Java 中的泛型关乎参数和返回类型，因此它能够产生使用导出类作为其参数和返回类型的基类。它还能够将导出类用作其作用域。下面看一个简单的泛型类：
```java
public class Basic<T> {
	T element;

	public T getElement() {
		return element;
	}

	public void setElement(T element) {
		this.element = element;
	}

	void f(){
		System.out.println(element.getClass().getSimpleName());
	}
}
```
这个普通的泛型类，他的方法将接收和产生具有其参数类型的对象，还有一个方法在存储域进行操作。这些操作都是操作的 T 这个参数类型。

我们利用古怪循环来使用上边的类：
```java
public class Sybtype extends Basic<Sybtype>{

}
```
调用这个类将看到什么结果：
```java
public class CRGWithBase {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Sybtype sybtype = new Sybtype(),sybtype2 = new Sybtype();
		sybtype.setElement(sybtype2);
		Sybtype sybtype3 = sybtype.getElement();
		sybtype2.f();
	}

}
```
注意：新类 Sybtype 接受的参数和返回的值都是 Sybtype 类型而不仅仅是基类 Basic 类型。这就是 CRG 的本质：基类用导出类作为参数。这意味着泛型基类变成一种其所有导出类的公共的功能模板，但是这些功能的所有参数和返回值将使用导出类自己。也就是说在所产生的类中将使用确切类型而不是基类型。

#### 自限定
Basic 可以使用任何类型做为其泛型参数。
```java
public class Other {

}

```
继承它：
```java
public class Sybtype extends Basic<Other>{

}
```
调用这个代码：
```java
public class CRGWithBase {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Sybtype sybtype = new Sybtype(),sybtype2 = new Sybtype();
		sybtype.setElement(new Other());
		Other  other  = sybtype.getElement();
		sybtype.f();
	}

}
```
执行结果：
```java
Other
```
我们看到依然可以执行，那么自限定可以做什么呢？自限定可以强制泛型参数当做其自己的边界参数来使用。
```java
public class SelfBound<T extends SelfBound<T>> {
	T elemnt;

	public T getElemnt() {
		return elemnt;
	}

	public SelfBound<T> setElemnt(T elemnt) {
		this.elemnt = elemnt;
		return this;
	}

}

```
我们继承我们的自限定类型：
```java
public class A extends SelfBound<A>{

}
```
下一个：
```java
public class B extends SelfBound<A>{

}
```
上边的内容显示继承与它本身或者它的继承类都是可以的。

下面的例子：
```java
public class D {

}
```
继承与它：
```java
public class C extends SelfBound<D>{

}
```
我们的编译器会提示 D 是无法编译的。因为这是一个不相干的类。

我们继续看下面的例子：
```java
public class F extends SelfBound{

}

```
F 是可以运行的，但是会有警告提示。这说明自限定类型的用法并不是强制可执行的。如果这实在是非常重要，可以要求一个外部工具来确保不会使用原生类型来替代参数化类型。因此很明显，自限定类型只能强制作用于继承关系。如果使用自限定，就应该了解这个类所用的类型参数将于使用这个参数的类既有相同的基类关系。

还可以将自限定用于泛型方法：
```java
public class SelfBoundMethods {

	static <T extends SelfBound<T>> T f(T arg){
		return arg.setElemnt(arg).getElemnt();
	}
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		A a = f(new A());
	}

}

```
这可以防止这个方法被应用于自限定参数之外的事物。

#### 参数协变
自限定类型的价值在于它们可以产生协变参数类型，方法参数类型会随子类型而变化。尽管自限定类型还可以产生和子类型相同的类型，但这不是很重要。协变返回类型是 Java SE5 中引入的。

在非泛型代码中，参数类型不能随子类型发生变化：
```java
public class Ordinary {

	void  set(Base base){
		System.out.println("base");
	}

}
```
继承与它：
```java
public class Derived extends Ordinary {
	void set(Derived derived){
		System.out.println("derived");
	}
}
```
调用并执行：
```java
public class OrdinaryArug {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Base base = new Base();
		Derived derived = new Derived();
		derived.set(base);
		derived.set(derived);
	}

}
```
我们看到 set() 方法放入的两个类型都可以，因此 Derived 并没有覆盖 Ordinary 中的方法，而是重载了这个方法。但是如果我们使用自限定类型时，在导出类中就只有一个方法，并且这个方法接受导出类型而不是基类作为参数。
```java
public interface SelfBoundSetter<T extends SelfBoundSetter<T>> {
	void  set(T arg);
}

```
继承与它：
```java
public interface Setter extends SelfBoundSetter<Setter> {

}
```
调用：
```java
public class SelfBoundingAnd {

	void f(Setter setter,Setter setter2,SelfBoundSetter selfBoundSetter){
		setter.set(setter2);
		//Error:只能放入限定的类型
		setter.set(selfBoundSetter);

	}
}

```
总结：如果使用自限定类型，只能获得某个方法的一个版本，它将接受确切的类型参数。如果不使用自定类型，将重载参数类型。

## 动态类型安全
我们可以向 JavaSE5	之前的代码传递泛型容器，所以旧的代码有可能会破坏容器。JavaSE5 有一组边里的静态方法来帮助我们动态的检查类型安全问题。他们是：checkedCollection()、checkedList()、checkedMap()、checkedSet()、checkedSortedMap()、checkedSortedSet()。这些方法每一个都会将你希望动态检查的容器作为第一个参数，希望强制要求的类型作为第二个参数。
```java
ublic class CheckList {

	static void oldAdd(List oList){
		oList.add(new A());
	}

	public static void main(String[] args) {
		//有警告但是可以正常的运行
		List<D> list = new ArrayList<>();
		oldAdd(list);

		//添加动态验证，不能运行
		List<D> list2 = Collections.checkedList(new ArrayList<D>(), D.class);
		oldAdd(list2);
	}

}
```
执行结果：
```java
Exception in thread "main" java.lang.ClassCastException: Attempt to insert class genericty.A element into collection with element type class genericty.D
	at java.util.Collections$CheckedCollection.typeCheck(Collections.java:3037)
	at java.util.Collections$CheckedCollection.add(Collections.java:3080)
	at genericty.CheckList.oldAdd(CheckList.java:12)
	at genericty.CheckList.main(CheckList.java:22)
```
如果不添加动态的类型检查程序可以正常的运行，如果添加了动态检查，就会在添加的时候识别出您所添加的类型是不正确的。

## 异常
由于擦除的原因，将泛型应用于异常是非常受限制的。catch 语句不能捕获泛型类型的异常，因为在编译期和运行期都必须知道异常的确切类型。泛型类也不能直接或间接继承自 Throwable。但是类型参数可能会在一个方法的 throws 子句中用到。看下面的例子：

接口：
```java
public interface Processor<T,E extends Exception> {
	void process(List<T> resultCollector) throws E;
}

```
抛出异常的泛化类：
```java
public class ProcessRunner<T,E extends Exception> extends ArrayList<Processor<T,E>> {
	 List<T> processAll() throws E {
		    List<T> resultCollector = new ArrayList<T>();
		    for(Processor<T,E> processor : this)
		      processor.process(resultCollector);
		    return resultCollector;
	 }
}

```
两个自定义的异常类：
```java
public class Failure1 extends Exception{

}
--------------
public class Failure2 extends Exception{

}

```
Processor1：
```java
public class Processor1 implements Processor<String,Failure1>{

	 static int count = 3;
	@Override
	public void process(List<String> resultCollector) throws Failure1 {
		// TODO Auto-generated method stub
		 if(count-- > 1)
		      resultCollector.add("Hep!");
		    else
		      resultCollector.add("Ho!");
		    if(count < 0)
		       throw new Failure1();
	}

}

```
Processor2：
```java
public class Processor2 implements Processor<Integer,Failure2>{
	 static int count = 2;
	@Override
	public void process(List<Integer> resultCollector) throws Failure2 {
		// TODO Auto-generated method stub
		 if(count-- == 0)
		      resultCollector.add(47);
		    else {
		      resultCollector.add(11);
		    }
		    if(count < 0)
		       throw new Failure2();
	}

}
```
调用并执行：
```java
public class ThrowGenericException {

	 public static void main(String[] args) {
		    ProcessRunner<String,Failure1> runner = new ProcessRunner<String,Failure1>();
		    for(int i = 0; i < 3; i++)
		      runner.add(new Processor1());
		    try {
		      System.out.println(runner.processAll());
		    } catch(Failure1 e) {
		      System.out.println(e);
		    }

		    ProcessRunner<Integer,Failure2> runner2 =  new ProcessRunner<Integer,Failure2>();
		    for(int i = 0; i < 3; i++)
		      runner2.add(new Processor2());
		    try {
		      System.out.println(runner2.processAll());
		    } catch(Failure2 e) {
		      System.out.println(e);
		    }
		  }
}
```
集合泛型添加了我们的实现类，去在内部循环自己，并且抛出异常 E。我们在调用 processAll() 方法的时候捕获这个异常。

## 混型
混型最基本的概念就是混合多个类的能力，产生一个可以表示混型中所有类型的类。它将使得组装多各类变得简单。

混型的价值之一是他们可以将特性和行为一致的应用于多个类上。如果想在混型类中修改某些东西，这些修改将会应用于混型所应用的所有类型之上。正是这点，混型有一点面向方面编程的味道（AOP）。而方面经常用来解决混型问题。

#### 与接口混合
由于 Java 泛型的擦除机制，我们不能像 C++ 哪样简单的创建混型。一种更常见的推荐解决方案是使用接口来产生混型效果：

首先建立三个接口：
```java
public interface TimeStamped {
	 long getStamp();
}
-----------
public interface SerialNumbered {
	long getSerialNumber();
}
-----------
public interface BasicT {
	  public void set(String val);
	  public String get();
}
```
实现这三个接口：

TimeStampedImp:
```java
public class TimeStampedImp implements TimeStamped{

	 private final long timeStamp;
	  public TimeStampedImp() {
	    timeStamp = new Date().getTime();
	  }
	  public long getStamp() { return timeStamp; }

}
```
SerialNumberedImp:
```java
public class SerialNumberedImp implements SerialNumbered {

	  private static long counter = 1;
	  private final long serialNumber = counter++;
	  public long getSerialNumber() { return serialNumber; }

}
```
BasicImp：
```java
public class BasicImp implements BasicT{

	  private String value;
	  public void set(String val) { value = val; }
	  public String get() { return value; }

}
```
组合这三个接口：
```java
public class Mixin extends BasicImp implements TimeStamped,SerialNumbered{

	  private TimeStamped timeStamp = new TimeStampedImp();
	  private SerialNumbered serialNumber = new SerialNumberedImp();
	@Override
	public long getSerialNumber() {
		// TODO Auto-generated method stub
		return serialNumber.getSerialNumber();
	}

	@Override
	public long getStamp() {
		// TODO Auto-generated method stub
		return timeStamp.getStamp();
	}

}

```
调用并执行：
```java
public class Mixins {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		    Mixin mixin1 = new Mixin(), mixin2 = new Mixin();
		    mixin1.set("test string 1");
		    mixin2.set("test string 2");
		    System.out.println(mixin1.get() + " " +
		      mixin1.getStamp() +  " " + mixin1.getSerialNumber());
		    System.out.println(mixin2.get() + " " +
		      mixin2.getStamp() +  " " + mixin2.getSerialNumber());
	}

}

```
执行结果：
```
test string 1 1512460466569 1
test string 2 1512460466570 2
```
Mixin 类其实是使用了代理模式，因此每个混入类型都要求在 Mixin 中有一个相应的域，而你必须在 Mixin 中编写所有的必须的方法，将方法调用转发给恰当的对象。

#### 使用装饰器模式
装饰器经常应用于各种可能的组合，装饰器模式使用分层对象来动态透明的向单个对象添加责任。可以将功能分层。装饰器是通过使用组合和形式化结构来实现的，而混型是基于继承。因此可以将基于参数化类型的混型当做是一种泛型装饰器机制，这种机制不需要装饰器设计模式的继承结构。

基类：Basic
```java
class Basic {
  private String value;
  public void set(String val) { value = val; }
  public String get() { return value; }
}
```
子类：Decorator
```java
class Decorator extends Basic {
  protected Basic basic;
  public Decorator(Basic basic) { this.basic = basic; }
  public void set(String val) { basic.set(val); }
  public String get() { return basic.get(); }
}
```
子类：TimeStamped
```java
class TimeStamped extends Decorator {
  private final long timeStamp;
  public TimeStamped(Basic basic) {
    super(basic);
    timeStamp = new Date().getTime();
  }
  public long getStamp() { return timeStamp; }
}
```
子类：SerialNumbered
```java
class SerialNumbered extends Decorator {
  private static long counter = 1;
  private final long serialNumber = counter++;
  public SerialNumbered(Basic basic) { super(basic); }
  public long getSerialNumber() { return serialNumber; }
}
```
调用并执行：
```java
public class Decoration {
  public static void main(String[] args) {
    TimeStamped t = new TimeStamped(new Basic());
    TimeStamped t2 = new TimeStamped(
      new SerialNumbered(new Basic()));
    //! t2.getSerialNumber(); // Not available
    SerialNumbered s = new SerialNumbered(new Basic());
    SerialNumbered s2 = new SerialNumbered(
      new TimeStamped(new Basic()));
    //! s2.getStamp(); // Not available
  }
}
```
使用装饰器所产生的类型是最后被装饰的类型。也就是说尽管可以有多个层次，但是最后一层才是实际的类型，因此只有最后一层的方法是可视的，而混型的类型是所有被混合到一起的类。

#### 与动态代理混合
可以使用动态代理来创建一种比装饰器更贴近混型模型的机制。(在类型信息章节中我们之前讲过如何创建动态代码)。通过使用动态代理，所产生的的类的动态类型将会是已经混入的组合类型。

动态代理限制每个被混入的类都必须是某个接口的实现：

基类：
```java
public class TwoTuple<A,B> {
  public final A first;
  public final B second;
  public TwoTuple(A a, B b) { first = a; second = b; }
  public String toString() {
    return "(" + first + ", " + second + ")";
  }
}
```
动态代理类：
```java
public class MixinProxy implements InvocationHandler{

	 Map<String,Object> delegatesByMethod;

	 public MixinProxy(TwoTuple<Object,Class<?>>... pairs) {
		 delegatesByMethod = new HashMap<String,Object>();
		 for (TwoTuple<Object, Class<?>> pair : pairs) {
			 //使用 Class<?>获得方法集合
			 for(Method method : pair.second.getMethods()) {
				 //获得方法名字
				 String methodName = method.getName();
				 if (!delegatesByMethod.containsKey(methodName))
					 //如果容器中不包含这个名字那么将方法名和对象传递进去
			          delegatesByMethod.put(methodName, pair.first);
			 }
		}
	}



	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		    String methodName = method.getName();
		    Object delegate = delegatesByMethod.get(methodName);
		    //将对象和参数传递给代理方法进行转接
		    return method.invoke(delegate, args);
	}

	 @SuppressWarnings("unchecked")
	  public static Object newInstance(TwoTuple... pairs) {
		 //用数组保存 Class
	    Class[] interfaces = new Class[pairs.length];
	    for(int i = 0; i < pairs.length; i++) {
	      interfaces[i] = (Class)pairs[i].second;
	    }

	    ClassLoader cl = pairs[0].first.getClass().getClassLoader();
	    //返回代理的对象
	    return Proxy.newProxyInstance(cl, interfaces, new MixinProxy(pairs));
	  }

}

```
调用动态代理：
```java
public class DynamicProxyMixin {

	public static void main(String[] args) {
		 Object mixin = MixinProxy.newInstance(
			      new TwoTuple(new BasicImp(), BasicT.class),
			      new TwoTuple(new TimeStampedImp(), TimeStamped.class),
			      new TwoTuple(new SerialNumberedImp(),SerialNumbered.class));
		        BasicT b = (BasicT)mixin;
			    TimeStamped t = (TimeStamped)mixin;
			    SerialNumbered s = (SerialNumbered)mixin;
			    b.set("Hello");
			    System.out.println(b.get());
			    System.out.println(t.getStamp());
			    System.out.println(s.getSerialNumber());
	}
}

```
执行结果：
```
Hello
1512467067275
1
```
为了让 Java 支持混型，我们正在做着大量的工作朝着这个目标努力。
