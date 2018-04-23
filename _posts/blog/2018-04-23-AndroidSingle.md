---
layout: post
title: Android架构设计之单例模式
categories: Android架构设计专题
description: Android架构设计之单例模式
keywords: Android, Android架构设计,单例模式
---

Android架构设计是进阶高级工程师升职加薪必备的技能。结构在所有的 IT 开发中都是非常重要的。一个好的软件架构会使得开发和后期的迭代节省大量的成本。基于此两项目的我们也应该好好的掌握这项技能。另外 Android 的源码中也存在大量的架构设计，如果不能对此有所了解是很难看懂源码的。源码对于我们技能的提升也是非常重要的。掌握架构设计之前必须先掌握设计模式。

## 单例模式
设计模式是一种开发经验、开发结构，是前人在不断的实战中总结的一种更好实践。结构设计离不开设计模式，一个项目中到处都可见到设计模式的存在。我们可以看到 Android 的源代码中大量存在着设计模式。其中我们应用最广泛和最好理解的是单例模式。

## 恶汉模式
恶汉模式代码最简洁，但是应用不是很广。首先看一下恶汉模式的代码：

```java
/**
 * 饿汉模式，属于类的成员变量，在类加载的时候就加载，线程安全。缺点：加载字节码就已经加载了无法做到随用随加载
 * @author 晁东洋
 *
 */
public class SingleEasy {

	private static SingleEasy singleEasy = new SingleEasy();
  private SingleEasy() {

	}

  public static SingleEasy getSingleEasy() {
		return singleEasy;
	}

}
```
我们看第一行代码，singleEasy 对象是静态的在 SingleEasy.class 类加载的时候就已经被加载，但是实际上单例模式的初衷是谁调用谁加载，违背了我们的初衷。表面上可以理解为加载太着急了所以叫恶汉。但是由于是在加载的过程中就已经加载所以也是线程安全的。

## 懒汉模式
懒汉模式是我们经常会看到的一种写法：
```java
/**
 * 懒汉模式，线程安全
 * @author 11842
 *
 */
public class SingleNotEmpty {
	private static SingleNotEmpty singleNotEmpty;
	private SingleNotEmpty() {

	}
	//同步的线程安全，每次调用getSingleNotEmpty都要同步保证了线程安全，但是不友好造成性能降低
	private static synchronized SingleNotEmpty getSingleNotEmpty() {
		if (singleNotEmpty == null) {
			singleNotEmpty = new SingleNotEmpty();
		}
		return singleNotEmpty;

	}

}

```
我们不在将类的对象的创建变为成员变量，而是放在一个方法中去创建。这就避免了在类加载的时候就加载的弊端，做到了谁调用谁加载。并且我们在 getSingleNotEmpty() 方法上加了同步锁以保证线程的安全，避免创造出多个对象。但是由于我们使用的 synchronized 同步锁这样一来我们的线程去访问这个方法时都必须等待上一个线程访问完毕才能获取锁。这就导致了性能的降低。

## DCL 模式
DCL 模式也叫双检查模式，是对懒汉模式的一种改进

```java
/**
 * 双检查模式，先判断是否为空，在用同步锁就行创建对象
 * @author 11842
 *
 */
public class DCLSingle {
	private static DCLSingle dclSingle;
	private DCLSingle(){

	}
	public static DCLSingle getDclSngle() {
		if (dclSingle == null) {
			synchronized (DCLSingle.class) {
				if (dclSingle == null) {
					dclSingle = new DCLSingle();
				}
			}
		}
		return dclSingle;
	}

}

```
我们看到上面的代码区别之处在于加锁的地方，懒汉模式是在方法体上加锁。DCL 模式是在方法内加锁。我们首先判断对象是否为空，如果不是空就直接返回，避免了懒汉模式的等待释放锁。如果是空才进入锁中去创建对象。避免了性能上的不必要消耗。

## 指令乱序以及 Volatile 关键字
DCL 方式可以避免 ```99%```的情况下线程是安全的。在面对复杂的序列化和反射时可能会创建出多个实例。在 Java 1.5 之后虚拟机支持了指令乱序的特性。指令乱序是为了提高程序性能而导致的执行时的指令顺序和代码中的顺序不一致。Volatile 关键字用来修饰一个变量，提示编译器这个变量可能随时会修改。被这个关键字修饰过的变量会重新访问内存，这样就保证这个变量的真实性。

![](/images/blog/neicun.png)

看下面这张程序的简易执行图：

![](/images/blog/luanxu.png)

由于指令乱序的原因 dclSingle 变量和 new DCLSingle() 的执行顺序有可能混乱所以有可能造成已经 new DCLSingle() 之后 dclSingle 变量仍然为空。

## 内部类单例
内部类实现单例是一种很优雅的方式，使得我们可以从 jvm 虚拟机的层面实现多线程同步的问题。因为 jvm 虚拟机对于一个类的初始化同一时间只允许一个线程去访问。

```Java
public class InnnerClassSingle {

	private InnnerClassSingle()
	{

	}
	private static class SingleHodler{
		private static final InnnerClassSingle instance=new InnnerClassSingle();
	}

	public static InnnerClassSingle getInstance()
	{
		return SingleHodler.instance;
	}

}

```
## 枚举单例
枚举单例模式是在 Java 1.5 之后利用枚举这一特性来实现的。可以面对复杂的序列化和反射的工具保证对象只有一个。

```Java
/**
 * 枚举单例模式
 * @author 11842
 *
 */
public enum EnumSingle {
	    DATASOURCE;
	    private DBConnection connection = null;
	    private void EnumSingle() {
	        connection = new DBConnection();
	    }
	    public DBConnection getConnection() {
	        return connection;
	    }
	    public class DBConnection {}
}

```
调用单例对象：
```Java
public class MainDemo {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		EnumSingle.DATASOURCE.getConnection();
	}

}
```

## 枚举是如何保证单例的实现的
首先看一个简单的枚举：
```Java
public enum EnumSingle {
    DATASOURCE;
}  
```
查看反编译之后的代码：
```Java
public final class EnumSingle extends Enum<EnumSingle> {
      public static final EnumSingle DATASOURCE;
      public static EnumSingle[] values();
      public static EnumSingle valueOf(String s);
      static {};
}
```
我们看到 EnumSingle 被修饰为 static。根据前边的类加载原理分析，Jvm 虚拟机类加载的过程中 EnumSingle 是被同步一起加载的，会被正确的加锁。所以这是线程安全的。

#### 序列化的问题
Java 中规定，每一个枚举类型及其变量在 jvm 虚拟机中都是唯一的。在序列化的时候仅仅是将枚举类型的 name 结果输出，然后在反序列化的时候通过 valuOf() 方法查找。因此保证了序列化之前和序列化之后的对象也是相同的。

## 总结
介绍了五中最常见的单例模式，特别是我们应该最好去使用枚举的模式，枚举模式更简单，简洁与安全。
