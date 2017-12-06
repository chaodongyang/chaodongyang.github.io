---
layout: post
title: java编程思想之泛型五(潜在类型机制)
categories: javaThinking
description: java编程思想之泛型
keywords: java, java泛型, java泛型详解，泛型详解
---

泛型对于我们来说的作用是尽可能编写广泛应用的代码。为了实现这一点，我们需要各种途径来放松对我们的代码将要作用的类型所作的限制，同时不丢失静态类型检查的好处。然后我们就可以编写出无需修改就可以应用于更多情况的代码，即更加泛化的代码。

## 潜在类型机制

潜在类型机制是一种代码组织和复用机制。有了它编写出的代码相对于没有他编写的代码，更够更容易的复用。代码组织和复用是所有计算机编程的基本手段：编写一次，多次使用，并在一个位置保存代码。

两种支持潜在类型机制的语言实例是 Python 和 C++。前者是动态类型语言，后者是静态类型语言。因此潜在类型机制不要求静态或者动态类型检查。因为泛型是后期才添加到 Java 中的，因此没有机会去实现任何类型的潜在类型机制，因此，Java 没有对这种特性的支持。所以，初看起来 Java 的泛型机制比支持潜在类型机制的语言更 “缺乏泛化化”。我们在使用 Java 实现潜在类型机制的时候就会被强制要求使用一个类或接口，并在边界表达式中指定它。

先看一段 C++ 的代码：
```C++
class Dog {
public:
  void speak() {}
  void sit() {}
  void reproduce() {}
};

class Robot {
public:
  void speak() {}
  void sit() {}
  void oilChange() {
};

template<class T> void perform(T anything) {
  anything.speak();
  anything.sit();
}

int main() {
  Dog d;
  Robot r;
  perform(d);
  perform(r);
}
```
Dog 和 Robot 没有任何共同的东西，只是碰巧有两个签名一样的方法。从类型观点看它们是完全不同的两个类型。但是，perform() 不关心其参数的具体类型，并且潜在类型机制允许它接受这两种类型的对象。

我们再来看一下 Java 代码的实例：
```java
public interface Performs {
	  void speak();
	  void sit();
}
```
实现这个接口的 PergormingDog 类：
```java
public class PergormingDog extends Dog implements Performs{

	@Override
	public void speak() {
		// TODO Auto-generated method stub

	}

	@Override
	public void sit() {
		// TODO Auto-generated method stub

	}

	public void  reproduce() {

	}

}

public class Dog {

}
```
实现接口的 Robot 类：
```java
public class Robot implements Performs{

	@Override
	public void speak() {
		// TODO Auto-generated method stub

	}

	@Override
	public void sit() {
		// TODO Auto-generated method stub

	}
	public void  oilChange() {

	}

}

```
泛型类：
```java
public class Communicate {
	public static <T extends Performs> void name(T performs) {
		performs.speak();
		performs.sit();
	}
}
```
调用并执行：
```java
public class DogsAndRobots {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		PergormingDog dog = new PergormingDog();
		Robot robot = new Robot();
		Communicate.name(dog);
		Communicate.name(robot);
	}

}

```
注意：这个实例泛型也不是必须的，name() 方法可以不需要泛型，他可以简单的指定一个 Performs 对象来工作：因为这些类已经被强制要求实现了 Performs 接口。
```java
public class Communicate {
	public static  void name(Performs performs) {
		performs.speak();
		performs.sit();
	}
}
```
## 对缺乏潜在类型机制的补偿
尽管 Java 不支持潜在类型机制，但是这不意味着 Java 中的有界泛型代码不能在不同的类型层次结构执行。也就是说我们可以创建真正的泛型代码：

#### 反射
我们可以使用反射的机制，我们来看下面修改过的代码：

定义一个 Mime 类：
```java
public class Mime {
	public void getName() {
		System.out.println("获得名字");
	}

	public void spit() {
		System.out.println("共同的方法 spit");
	}


}
```
定义一个 SmartDog 类：
```java
public class SmartDog {
	public void getName() {
		System.out.println("获得名字");
	}

	public void spit() {
		System.out.println("共同的方法 spit");
	}

	public void speak() {
		System.out.println("说狗话");
	}
}

```
使用反射机制定义一个 CommunicatReflect 类：
```java
public class CommunicatReflect {
		public static void perform(Object speak) {
			Class<?> spkr = speak.getClass();

			Method method;
			try {
				method = spkr.getMethod("spit");
				try {
					method.invoke(speak);
				} catch (IllegalAccessException | IllegalArgumentException | InvocationTargetException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			} catch (NoSuchMethodException | SecurityException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}


				Method method2;
				try {
					method2 = spkr.getMethod("speak");
					try {
						method2.invoke(speak);
					} catch (IllegalAccessException | IllegalArgumentException | InvocationTargetException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				} catch (NoSuchMethodException | SecurityException e) {
					// TODO Auto-generated catch block
					System.out.println("Mime 不能说狗话");
				}




		}
}

```
调用并执行：
```java
public class LatentReflect {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		CommunicatReflect.perform(new SmartDog());
		CommunicatReflect.perform(new Mime());
	}

}

```
执行结果：
```
共同的方法 spit
说狗话
共同的方法 spit
Mime 不能说狗话
```
我们看到这些类是完全彼此分离的，没有任何公共的基类或接口。通过反射能够动态的确定所需要的方法可用并且调用。

#### 将一个方法应用于序列
反射提供了实现的可能性，但是它将所有的类型检查都转移到了运行时期，因此在许多情况下我们不希望这么做。如果能够实现编译期类型检查会更符合我们的要求。我们使用 JavaSE5 的可变参数来试着解决这个问题。我们创建一个方法可以将任何序列中的对象执行他们相应的方法。

首先我们创建一个 Apply 类：
```java
public class Apply {
	public static <T,S extends Iterable<? extends T>> void apply(S seq,Method fMethod,Object... objects ) {
		//S 是一个实现了 Iterable 的对象所以可以for 循环
		for (T t : seq) {
			try {
				fMethod.invoke(t, objects);
			} catch (Exception e) {
				// TODO Auto-generated catch block
				throw new RuntimeException(e);
			}
		}
	}

}
```
创建 Shape 类：
```java
public class Shape {
	public void rotate() {
		System.out.println("shape:rotate()");
	}

	public void resize(int newSize) {
		System.out.println("resize"+newSize);
	}
}
--------------
public class Square extends Shape{

}
```
自定义一个泛型集合类：
```java
public class FilledList<T> extends ArrayList<T>{
	public FilledList(Class<? extends T> tClass,int size) {
		// TODO Auto-generated constructor stub
		for (int i = 0; i < size; i++) {
			try {
				add(tClass.newInstance());
			} catch (Exception e) {
				// TODO Auto-generated catch block
				throw new RuntimeException(e);
			}
		}
	}
}
```
我们来创建一个测试的类，来测试我们的代码：
```java
public class ApplyTest {

	public static void main(String[] args) throws Exception {
		// TODO Auto-generated method stub
		List<Shape> shapes = new ArrayList<>();
		for (int i = 0; i < 2; i++) {
			shapes.add(new Shape());
		}
		//持有一个对象的集合，对象的方法，以及参数
		Apply.apply(shapes, Shape.class.getMethod("rotate"));
		Apply.apply(shapes, Shape.class.getMethod("resize",int.class),5);

		List<Square> squares = new ArrayList<>();
		for (int i = 0; i < 2; i++) {
			squares.add(new Square());
		}
		Apply.apply(squares, Square.class.getMethod("rotate"));
		Apply.apply(squares, Square.class.getMethod("resize",int.class),5);

		Apply.apply(new FilledList<Shape>(Shape.class,2),Shape.class.getMethod("rotate"));
		Apply.apply(new FilledList<Shape>(Shape.class,2),Square.class.getMethod("rotate"));

	}

}

```
执行结果：
```java
shape:rotate()
shape:rotate()
resize5
resize5
shape:rotate()
shape:rotate()
resize5
resize5
shape:rotate()
shape:rotate()
shape:rotate()
shape:rotate()
```
#### 当你并未碰巧拥有正确的接口时
上面的实例是恰当的，因为 Iterable 接口刚好是已经内建的，而我们又刚好需要。但是对于更一般的情况，如果不存在刚好适合我们需要的接口就很麻烦。所以其实我们还是限制在继承层次之内的。因此这样的代码也不是特别的泛化。有了潜在类型机制情况就会不同了。

#### 用适配器仿真潜在类型机制
Java 泛型并不具备潜在类型机制，而我们需要像潜在类型机制哪样编写跨界的代码。存在某种方式可以绕过这种限制吗？潜在类型机制是如何实现的？实际上，潜在类型机制创建了一个包含所有方法的隐式接口。因此它遵循这样的规则：如果我们手工编写必须的接口，那么久应该解决这个问题。

从我们拥有的接口中编写代码来产生我们需要的接口，这是适配器模式的一个典型事例。我们可以使用适配器来适配已有的接口，以产生想要的接口。

创建一个接口：
```java
public interface Addable<T> {
	void add(T t);
}
```
使用接口中的方法来添加泛型对象：
```java
public class Fill2 {
	      public static <T> void fill(Addable<T> addable,Class<? extends T> classToken, int size) {
			    for(int i = 0; i < size; i++)
			      try {
			    	  //使用接口中的方法来添加一个泛型的对象
			        addable.add(classToken.newInstance());
			      } catch(Exception e) {
			        throw new RuntimeException(e);
			      }
		  }

}
```
我们去实现我们定义的接口：
```java
public class AddableCollectionAdapter<T> implements Addable<T> {

	  private Collection<T> c;

	  public AddableCollectionAdapter(Collection<T> c) {
	    this.c = c;
	  }

	@Override
	public void add(T t) {
		// TODO Auto-generated method stub
		c.add(t);
	}

}
```
适配器模式的重点：我们用适配器来返回我们想要的接口：
```java
public class Adapter {
	 public static <T> Addable<T> collectionAdapter(Collection<T> c) {
	    return new AddableCollectionAdapter<T>(c);
	  }
}
```
测试我们的类：
```java
public class Fill2Test {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		 List<Coffee> carrier = new ArrayList<Coffee>();
		    Fill2.fill(new AddableCollectionAdapter<Coffee>(carrier),Coffee.class, 3);
		    // Helper method captures the type:
		    Fill2.fill(Adapter.collectionAdapter(carrier),Latte.class, 2);
		    for(Coffee c: carrier)
		      System.out.println(c);

	}

}

```
运行结果：
```
Coffee 0
Coffee 1
Coffee 2
Latte 3
Latte 4

```
使用适配器模式是对潜在类型机制的一种补偿，因此允许编写真正的泛化的代码。但是，这是一种额外的步骤，潜在类型机制通过移除这个额外的步骤，使得泛化的代码更容易应用。

## 将函数对象用作策略
此段内容过于复杂，话不多说，还是直接看代码吧。多看几遍就懂了。

先定义几个接口：
```java
public interface Combiner<T>  {
	T combine(T x, T y);
}
-------------
public interface UnaryFunction<R,T>  {
	 R function(T x);
}
-------------
public interface Collector<T> extends UnaryFunction<T,T> {
	 T result();
}
-------------
public interface UnaryPredicate<T> {
	boolean test(T x);
}

```
最后直接上一个大类：
```java
public class Functional {
	 public static <T> T reduce(Iterable<T> seq, Combiner<T> combiner) {
	    Iterator<T> it = seq.iterator();
	    if(it.hasNext()) {
	      T result = it.next();
	      while(it.hasNext())
	        result = combiner.combine(result, it.next());
	      return result;
	    }
	    // If seq is the empty list:
	    return null; // Or throw exception
	  }

	  public static <T> Collector<T> forEach(Iterable<T> seq, Collector<T> func) {
	    for(T t : seq)
	      func.function(t);
	    return func;
	  }

	  // Creates a list of results by calling a
	  // function object for each object in the list:
	  public static <R,T> List<R> transform(Iterable<T> seq, UnaryFunction<R,T> func) {
	    List<R> result = new ArrayList<R>();
	    for(T t : seq)
	      result.add(func.function(t));
	    return result;
	  }

	  // Applies a unary predicate to each item in a sequence,
	  // and returns a list of items that produced "true":
	  public static <T> List<T> filter(Iterable<T> seq, UnaryPredicate<T> pred) {
	    List<T> result = new ArrayList<T>();
	    for(T t : seq)
	      if(pred.test(t))
	        result.add(t);
	    return result;
	  }
	  // To use the above generic methods, we need to create
	  // function objects to adapt to our particular needs:
	  static class IntegerAdder implements Combiner<Integer> {
	    public Integer combine(Integer x, Integer y) {
	      return x + y;
	    }
	  }

	  static class IntegerSubtracter implements Combiner<Integer> {
	    public Integer combine(Integer x, Integer y) {
	      return x - y;
	    }
	  }

	  static class BigDecimalAdder implements Combiner<BigDecimal> {
	    public BigDecimal combine(BigDecimal x, BigDecimal y) {
	      return x.add(y);
	    }
	  }

	  static class BigIntegerAdder implements Combiner<BigInteger> {
	    public BigInteger combine(BigInteger x, BigInteger y) {
	      return x.add(y);
	    }
	  }


	  static class AtomicLongAdder implements Combiner<AtomicLong> {
	    public AtomicLong combine(AtomicLong x, AtomicLong y) {
	      // Not clear whether this is meaningful:
	      return new AtomicLong(x.addAndGet(y.get()));
	    }
	  }

	  // We can even make a UnaryFunction with an "ulp"
	  // (Units in the last place):
	  static class BigDecimalUlp implements UnaryFunction<BigDecimal,BigDecimal> {
	    public BigDecimal function(BigDecimal x) {
	      return x.ulp();
	    }
	  }

	  static class GreaterThan<T extends Comparable<T>> implements UnaryPredicate<T> {
	    private T bound;
	    public GreaterThan(T bound) { this.bound = bound; }
	    public boolean test(T x) {
	      return x.compareTo(bound) > 0;
	    }
	  }

	  static class MultiplyingIntegerCollector implements Collector<Integer> {
	    private Integer val = 1;
	    public Integer function(Integer x) {
	      val *= x;
	      return val;
	    }
	    public Integer result() { return val; }
	  }

	  public static void main(String[] args) {
	    // Generics, varargs & boxing working together:
	    List<Integer> li = Arrays.asList(1, 2, 3, 4, 5, 6, 7);
	    Integer result = reduce(li, new IntegerAdder());
	   System.out.println(result);

	    result = reduce(li, new IntegerSubtracter());
	    System.out.println(result);

	    System.out.println(filter(li, new GreaterThan<Integer>(4)));

	    System.out.println(forEach(li,
	      new MultiplyingIntegerCollector()).result());

	    System.out.println(forEach(filter(li, new GreaterThan<Integer>(4)),
	      new MultiplyingIntegerCollector()).result());

	    MathContext mc = new MathContext(7);
	    List<BigDecimal> lbd = Arrays.asList(
	      new BigDecimal(1.1, mc), new BigDecimal(2.2, mc),
	      new BigDecimal(3.3, mc), new BigDecimal(4.4, mc));
	    BigDecimal rbd = reduce(lbd, new BigDecimalAdder());
	    System.out.println(rbd);

	    System.out.println(filter(lbd,new GreaterThan<BigDecimal>(new BigDecimal(3))));

	    // Use the prime-generation facility of BigInteger:
	    List<BigInteger> lbi = new ArrayList<BigInteger>();
	    BigInteger bi = BigInteger.valueOf(11);
	    for(int i = 0; i < 11; i++) {
	      lbi.add(bi);
	      bi = bi.nextProbablePrime();
	    }
	    System.out.println(lbi);

	    BigInteger rbi = reduce(lbi, new BigIntegerAdder());
	    System.out.println(rbi);
	    // The sum of this list of primes is also prime:
	    System.out.println(rbi.isProbablePrime(5));

	    List<AtomicLong> lal = Arrays.asList(
	      new AtomicLong(11), new AtomicLong(47),
	      new AtomicLong(74), new AtomicLong(133));
	    AtomicLong ral = reduce(lal, new AtomicLongAdder());
	    System.out.println(ral);

	    System.out.println(transform(lbd,new BigDecimalUlp()));
	  }
}

```
执行结果：
```
28
-26
[5, 6, 7]
5040
210
11.000000
[3.300000, 4.400000]
[11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47]
311
true
265
[0.000001, 0.000001, 0.000001, 0.000001]

```
不用怕，如果不理解，可以看后续我设计模式的系列专题。设计模式系列专题将会在这个专题结束之后奉献上。敬请期待！

## 总结
使用泛型机制最吸引人的地方，就是在使用容器类的地方。在 javaSE5 之前，当你将一个对象放入到容器中时，这个对象就会被向上转型为 Object。因此你将丢失类型信息。当你想要将这个对象从容器中取回，用它去执行某些操作时，必须将其向下转型为正确的类型。如果没有 JavaSE5 版本的泛型容器，你放进容器和从容器取回的都是 Object。因此我们很可能将一个 Dog 放入到 Cat 的 List。

但是泛型出现之前的 Java 并不会让你吴用放入到容器中的对象。因为如果你将一个 Dog 当做 Cat 来处理，那你从容器中取回这个引用，并试图将他转型为 Cat 的时候，你会得到一个 RuntimeException 异常。你仍旧发现了问题，但是这是在运行时期发现的。但是猫和狗这个论据并不足以支撑泛型设计的理由。因为几乎没有人会这么做。

我相信泛型设计的目的在于可表达性，而不仅仅的为了创建类型安全的容器。类型安全的容器只不过是能够创建更通用代码这一能力的附加产品。泛型是一种方法，通过它可以编写出更泛化的代码，这些代码对于他们能够作用的类具有更少的限制，因此单个的代码段可以应用到更多的类型上。
