---
layout: post
title: java编程思想之泛型二 (神秘的擦除)
categories: javaThinking
description: java编程思想之泛型
keywords: java, java泛型, java泛型详解，泛型详解
---

回顾一下上一节的内容：泛型的一个重要好处是能够简单而且安全的创建复杂的模型。

## 构建复杂模型
我们可以很容易的构建一个 List 元组：
```java
public class TupeList<A,B> extends ArrayList<TwoTuple<A, B>>{

	public static void main(String[] args) {
		TupeList<String,Integer> mList = new TupeList<>();
		mList.add(TupleTest.f());
		mList.add(TupleTest.f());

		for (TwoTuple<String, Integer> twoTuple : mList) {
			System.out.println(twoTuple);
		}
	}

}
```
执行结果：
```
chao-47
chao-47
```
## 擦除的神秘之处
当我们深入的钻研泛型的时候，会看到很多东西初看起来是没有意义的。例如，我们可以声明一个 ArrayList.class，但是不能声明 ArrayList<Integer>.class。
```java
public class Erased {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Class class1 = new ArrayList<String>().getClass();
		Class class2 = new ArrayList<Integer>().getClass();

		System.out.println(class1 == class2);
	}

}

```
执行结果：
```
true
```
我们看到 ```ArrayList<String>``` 和 ```ArrayList<Integer>``` 很容易被认为是不同的类型。不同的类型在行为方面肯定是不同的。但是上面的程序却认为他们是相同的类型。

下面的示例是对上个谜题的补充：
```java
public class LostInfo {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		List<A> list =  new ArrayList<>();
		Map<A, B> map = new HashMap<>();
		System.out.println(Arrays.toString(list.getClass().getTypeParameters()));

		System.out.println(Arrays.toString(map.getClass().getTypeParameters()));
	}

}

```
执行结果：
```
[E]
[K, V]
```
根据 JDK 文档得知 getTypeParameters 得到的是泛型所有生命的类型参数。但是，正如所输出的哪样看到，你能发现的只是用作参数占位符的标识符，这是一下没有用的信息。因此，得到的结论是：在泛型代码内部，无法获得任何有关泛型参数类型的信息。

Java 泛型是使用擦除来实现的，这意味着当你使用泛型时，任何具体的类型信息都将被擦除。你唯一知道的就是你在使用一个对象。因此 ```List<String>``` 和 ```List<Integer>``` 在运行时事实上是相同的类型。这两种形式事实上都被擦除为他们的原生类型，即 List。

#### C++ 的方式
下面来看一段使用 C++ 的模板：
```C++
template<class T> class Manipulator {
  T obj;
public:
  Manipulator(T x) { obj = x; }
  void manipulate() { obj.f(); }
};

class HasF {
public:
  void f() { cout << "HasF::f()" << endl; }
};

int main() {
  HasF hf;
  Manipulator<HasF> manipulator(hf);
  manipulator.manipulate();
}
```

Manipulator 类存储了一个类型 T 的对象，但是我们看到 manipulate() 方法中 obj 调用了一个 f() 方法。它怎么知道 f() 方法是参数类型 T 的存在呢？当我们实例化模板的时候，C++ 的编译器会进行检查，因此在 Manipulator<HasF> 被实例化的一刻，它看到有一个 f() 的方法。如果没有就会得到一个编译错误。

我们来看一个 Java 版本的例子：

```java
class Manipulator<T> {
  private T obj;
  public Manipulator(T x) { obj = x; }
  // Error: cannot find symbol: method f():
  public void manipulate() { obj.f(); }
}

public class Manipulation {
  public static void main(String[] args) {
    HasF hf = new HasF();
    Manipulator<HasF> manipulator =
      new Manipulator<HasF>(hf);
    manipulator.manipulate();
  }
}
```
由于有了擦除，Java 编译器无法将 Manipulator 类中的 obj.f() 这一需求影射到 HasF 有用 f() 这一事实上。为了调用 f()。我们必须协助泛型类，给定他一个边界，以此告知编译器只能接受遵循这个边界的类型。这里我们使用 extends 关键字。
```java
class Manipulator2<T extends HasF> {
  private T obj;
  public Manipulator2(T x) { obj = x; }
  public void manipulate() { obj.f(); }
}
```
边界 ```<T extends HasF>``` 声明 T 必须具备有类型 HasF 或者 HasF 导出的类型。那么我们才可以安全的使用 obj.f()。我们说泛型类型参数将擦除到它的第一个边界，我们还提到类型参数的擦除。编译器实际上会把类型参数替换为它的擦除，就像上面的示例一样。T 擦除到了 HasF。

实际上我们也可以自己去擦除，就可以创建出没有泛型的类：
```java
class Manipulator3 {
  private HasF obj;
  public Manipulator3(HasF x) { obj = x; }
  public void manipulate() { obj.f(); }
}
```
那什么时候使用泛型合适呢？只有当你希望使用的类型参数比某个具体的类型更加泛化的时候，也就是说当你希望代码可以跨越多个类的时候，使用泛型才有所帮助。必须查看所有的源代码，并确定它是否足够复杂到必须使用泛型的程度。

#### 迁移兼容性
擦除是一个比较容易混淆的概念，所以我们必须认识到擦除并不是 Java 语言的特性。它是 Java 泛型实现中的一种折中，因为泛型并不是 Java 语言出现时就有的，为了向前兼容之前的类库，这种折中是必须的。但是擦除减少了泛型的泛化性。并没有当初设想的那么有用了。原因就出在擦除的问题上。

基于擦除的原因，泛型被当做第二类类型来使用。泛型类型只有在静态类型检查时才会出现，在此之后程序中所有的泛型类型都将会被擦除，替换为他们的非泛型上界。擦除的核心动机是，他使得泛化的客户端可以使用非泛化的类库，这被称为迁移兼容性。即便我们只编写泛化的代码，我们也必须去处理 JavaSE5 之前编写的非泛化类库。因此，Java 泛型不仅必须支持向后兼容性，现有的类库和文件还必须合法，并且继续保持之前的含义，而且还需要支持迁移兼容性。使得类库可以按照自己的步调变为泛型，又不破坏依赖他们的代码和程序。在这个目标决定之后，Java 设计者们认为擦除是唯一可行的方案。

#### 擦除的问题
擦除主要的正当理由是从非泛化代码到泛化代码的转变过程，以及在不破坏现有类的情况下，将泛型融入到 Java 语言中。擦除的代价是显著的。泛型不能显示的引用运行时类型的操作中。当在编写泛型代码时，必须时刻提醒自己，你只是看起来好像拥有有关参数的类型信息。

```java
class Foo<T>{
  T var;
}
```
当你在创建 Foo 的实例时：
```java
Foo<Cat> foo = new Foo<Cat>();
```
看起来 class Foo 中的代码应该知道工作与 Cat 之上，而泛型语法也在提示我们 T 应该被替换为 Cat。但事实并非如此，我们必须提醒自己：这只是一个 Object。

另外擦除和迁移兼容性意味着，使用泛型并不是强制的：
```java
public class GenericBase<T> {
	private T element;

	public T getElement() {
		return element;
	}

	public void setElement(T element) {
		this.element = element;
	}

}

```
子类：
```java
public class Derved1<T> extends GenericBase<T> {

}

public class Derved2 extends GenericBase {

}
```
不合法子类：
```java
public class Derved3 extends GenericBase<?> {

}
```
调用：
```java
public class DervenTest {

	@SuppressWarnings("unchecked")
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Derved2 derved2 = new Derved2();
		Object object = derved2.getElement();

		derved2.setElement(object);
	}

}
```
Derved2 继承了 GenericBase，但是没有任何的泛型参数，而编译器不会发出任何警告。只有在调用 set()  的时候才会出现警告。我们可以使用注解来关闭警告。Derved3 出现的错误意味着编译器希望得到一个原生基类。当你需要将类型参数不要仅仅当做 Object 来处理时，就需要付出额外的努力来管理边界。

#### 边界处的动作
正是有了擦除，我们发现泛型一个令人困惑的事实，即可以表示没有任何意义的事物。

```java
public class ArrayMaker<T> {

	  private Class<T> kind;

	  public ArrayMaker(Class<T> kind) {
		  this.kind = kind;
	 }

		//忽略警告信息的注解
	  @SuppressWarnings("unchecked")
	  T[] create(int size) {
	    return (T[])Array.newInstance(kind, size);
	  }

	  public static void main(String[] args) {
	    ArrayMaker<String> stringMaker = new ArrayMaker<String>(String.class);
	    String[] stringArray = stringMaker.create(9);
	    System.out.println(Arrays.toString(stringArray));
	  }
}
```
执行结果：
```
[null, null, null, null, null, null, null, null, null]
```
即使 kind 被存储为 ```Class<T>```，擦除也意味着它实际将被存储为 Class，没有任何参数。例如，创建数组时，因为不会产生任何结果，编译器会给我们一个警告。

如果我们要创建一个容器，而不是数组：

```java
public class ListMaker<T> {

	List<T> create() {
		return new ArrayList<T>();
	}
	  public static void main(String[] args) {
	    ListMaker<String> stringMaker= new ListMaker<String>();

	    List<String> stringList = stringMaker.create();
	  }

}

```
不会输出任何的结果，编译器也不会发出任何警告，尽管我们知道 ```new ArrayList<T>``` 的 T 将会被移除，在运行时，这个类的内部没有任何 T ，因此看起来毫无意义。但是如果我们将表达式修改为 new ArrayList()，编译器就会警告。

如果我们在返回 List 之前，将某些对象放入其中，就像下面这样。

```java
public class FilledListMaker<T> {

	 List<T> create(T t, int n) {
		    List<T> result = new ArrayList<T>();
		    for(int i = 0; i < n; i++)
		      result.add(t);
		    return result;
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		FilledListMaker<String> stringMaker = new FilledListMaker<String>();
	    List<String> list = stringMaker.create("Hello", 4);
	   System.out.println(list);
	}

}
```
执行结果：
```
[Hello, Hello, Hello, Hello]
```
我们看到即使是编译器无法知道 T 的具体类型，它仍旧可以在编译期确保你放入的类型是正确的。因此，即使擦除在类或方法内部移除了有关实际类型的信息编译器仍旧可以确保在方法或类使用类型的内部一致性。

因此方法中移除了类型信息，所以在运行时就要解决这个问题，那就是边界的问题：即对象进入和离开方法的地点。这正是编译器执行类型检查并插入转型代码的地点。

```java
public class SimpleHolder {

	  private Object obj;
	  public void set(Object obj) { this.obj = obj; }
	  public Object get() { return obj; }

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		 SimpleHolder holder = new SimpleHolder();
		 holder.set("Item");
		 String s = (String)holder.get();
	}

}

```
反编译这个类：
```
public void set(java.lang.Object obj);
    0  aload_0 [this]
    1  aload_1 [obj]
    2  putfield genericity.SimpleHolder.obj : java.lang.Object [18]
    5  return

public java.lang.Object get();
	  0  aload_0 [this]
	  1  getfield genericity.SimpleHolder.obj : java.lang.Object [18]
	  4  areturn
public static void main(java.lang.String[] args);
     0  new genericity.SimpleHolder [1]
     3  dup
     4  invokespecial genericity.SimpleHolder() [24]
     7  astore_1 [holder]
     8  aload_1 [holder]
     9  ldc <String "Item"> [25]
    11  invokevirtual genericity.SimpleHolder.set(java.lang.Object) : void [27]
    14  aload_1 [holder]
    15  invokevirtual genericity.SimpleHolder.get() : java.lang.Object [29]
    18  checkcast java.lang.String [31]
    21  astore_2 [s]
    22  return

}
```
set() 和 get() 将直接产生值，而转型是在我们调用 get() 的时候进行的。

现在使用泛型修改上面的代码：
```java
public class GenericHolder<T> {

	private T obj;
	public T getObj() {
		return obj;
	}

	public void setObj(T obj) {
		this.obj = obj;
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		GenericHolder<String> holder = new GenericHolder<String>();
	    holder.setObj("Item");
	    String s = holder.getObj();
	}

}

```
编译之后的代码：
```
public java.lang.Object getObj();
    0  aload_0 [this]
    1  getfield genericity.GenericHolder.obj : java.lang.Object [23]
    4  areturn

public void setObj(java.lang.Object obj);
	  0  aload_0 [this]
    1  aload_1 [obj]
    2  putfield genericity.GenericHolder.obj : java.lang.Object [23]
	  5  return

public static void main(java.lang.String[] args);
	      0  new genericity.GenericHolder [1]
	      3  dup
	      4  invokespecial genericity.GenericHolder() [30]
	      7  astore_1 [holder]
	      8  aload_1 [holder]
	      9  ldc <String "Item"> [31]
	     11  invokevirtual genericity.GenericHolder.setObj(java.lang.Object) : void [33]
	     14  aload_1 [holder]
	     15  invokevirtual genericity.GenericHolder.getObj() : java.lang.Object [35]
	     18  checkcast java.lang.String [37]
	     21  astore_2 [s]
	     22  return
```
我们发现我们上边的代码调用 get() 方法之后的转型消失了，我们看到返回的字节码是一样的。对 set() 的检查是不需要的，因为这是编译器执行的。但是对 get() 的转型仍然是需要的。此处它将由编译器自动的插入。因此我们自己写代码的噪声将会减少。所以对泛型发生的动作都产生在边界处，对传入的值进行编译器检查，并对传出的值自动插入转型。

我们没有写强制转型，因为自动插入了转型。
```
 18  checkcast java.lang.String [37]
```

## 擦除的补偿
正如我们看到的，擦除丢失了泛型代码中具体的类型信息。需要知道确切类型信息的操作都无法操作。

偶尔可以绕过这些问题进行编程，但是我们必须通过引入类型标签来解决这个问题。这就意味着你需要显示的传递你的类型的 Class 对象。以便你可以在表达式中使用它。

```java
public class ClassTypeCapture<T>{

   Class<T> kind;
	public ClassTypeCapture(Class<T> class1) {
		// TODO Auto-generated constructor stub
		this.kind = class1;
	}

	public  boolean f(Object arg) {
		return kind.isInstance(arg);
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		//传入具体的类型信息，称为 类型标签
		ClassTypeCapture<Building> capture = new ClassTypeCapture<Building>(Building.class);
		System.out.println(capture.f(new Building()));
		System.out.println(capture.f(new House()));

	}

}

```
执行结果：
```
true
true
```

#### 创建类型实例
如果我们使用 new T() 来创建一个对象是无法实现的，部分原因是因为擦除，还有一部分原因是编译器不能验证 T 是否具有默认的构造器。但是在 C++ 中确实可以的：
```C++
template<Class T> class Foo{
	T x;
	T* y;

	public Foo(){
		y = new T();
	}
}
```
Java 中的解决方案是传递一个工厂对象，并使用它来创建新的实例。最便利的工厂对象就是 Class 对象。因此，如果我们使用类型标签，就可以解决这个问题。
```java
public class ClassAsFactory<T> {

	T xT;
	public ClassAsFactory(Class<T> kind) {
		// TODO Auto-generated constructor stub
		try {
			xT = kind.newInstance();
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
调用并执行：
```java
public class InstantiateGenericType {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		 ClassAsFactory<BCoffer> fe = new ClassAsFactory<BCoffer>(BCoffer.class);


	     ClassAsFactory<Integer> fi = new ClassAsFactory<Integer>(Integer.class);

	}

}
```
执行结果：
```
java.lang.InstantiationException: java.lang.Integer
	at java.lang.Class.newInstance(Class.java:427)
	at genericity.ClassAsFactory.<init>(ClassAsFactory.java:9)
	at genericity.InstantiateGenericType.main(InstantiateGenericType.java:12)
Caused by: java.lang.NoSuchMethodException: java.lang.Integer.<init>()
	at java.lang.Class.getConstructor0(Class.java:3082)
	at java.lang.Class.newInstance(Class.java:412)
	... 2 more

```
因为 Integer 没有任何默认的构造器，所以运行出错了。但是这个错误不是在编译器捕获的。所以我们应该尽可能使用显示的工厂。并限制其类型。使得只能接受这个工厂的类：

接口：
```java
public interface Factory<T> {
  T create();
}
```
使用泛型：
```java
public class Foo2<T> {

	private T x;
	public <F extends TypeInfo.Factory<T>> Foo2(F fac) {
		x = fac.create();
	}


}
```
实现接口：
```java
public class IntegrFactory implements Factory<Integer>{

	@Override
	public Integer create() {
		// TODO Auto-generated method stub
		return new Integer(0);
	}

}
```
调用：
```java
public class FactoryConstraint {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		new Foo2<Integer>(new IntegrFactory());
	}

}

```
两种方式都传递了工厂对象，第一种是内建的工厂对象，第二种是显示的工厂对象。

#### 泛型数组
我们也不能像 new T[] 这样的去创建泛型的数组：一般的解决方案是在任何想要创建泛型数组的地方都使用 ArrayList。

```java
public class ListMaker<T> {
  List<T> create() { return new ArrayList<T>(); }
  public static void main(String[] args) {
    ListMaker<String> stringMaker= new ListMaker<String>();
    List<String> stringList = stringMaker.create();
  }
}
```
这时我们将获得数组的行为，并由泛型提供的编译期的类型安全。

我们看下面的例子，我们去创建泛型数组，可以先定义一个引用。

```java
public class Generic<T> {

}
```
创建数组：
```java
public class ArrayOfGeneric {
	static final int SIZE = 100;
	static Generic<Integer>[] gia;

	@SuppressWarnings("unchecked")
	public static void main(String[] args) {

	   gia = (Generic<Integer>[])new Object[SIZE];
		 /*gia = (Generic<Integer>[])new Generic[SIZE];
		 System.out.println(gia.getClass().getSimpleName());
	   gia[0] = new Generic<Integer>();*/
	}

}
```
执行结果：
```
Exception in thread "main" java.lang.ClassCastException: [Ljava.lang.Object; cannot be cast to [Lgenericity.Generic;
	at genericity.ArrayOfGeneric.main(ArrayOfGeneric.java:10)
```
理论上数组都具有相同的结构，我们创建一个 Object 的数组，并将其转型为所希望的数组类型。可以编译却不能运行。问题在于数组会跟踪他们的实际类型，而这个类型是在数组被创建事就确立的。即使上边已经被转型为 ```Generic<Integer>``` 数组，但是只存在编译器。在运行时它仍旧是 Object 数组。

成功创建一个泛型数组的唯一方式就是创建一个擦除类型的新数组。然后对其转型：

```java
public class GenericArray<T> {

	private T[] array;
	  @SuppressWarnings("unchecked")
	  public GenericArray(int sz) {
	    array = (T[])new Object[sz];
	  }
	  public void put(int index, T item) {
	    array[index] = item;
	  }
	  public T get(int index) { return array[index]; }
	  public T[] rep() { return array; }

	  public static void main(String[] args) {
		  GenericArray<Integer> gai =  new GenericArray<Integer>(10);
			    // This causes a ClassCastException:
			    //! Integer[] ia = gai.rep();
			    // This is OK:
			    Object[] oa = gai.rep();
	}

}
```
其实与前面相同，我们并不能直接去创建一个 T[] t = new T[siz]，这样的数组。因此我们创建了一个对象数组。既然指定了 T 的类型是 Integer。但是我们看到仍然不能以 Integr[] 的数组去捕获，因为擦除机制在运行时仍然实际运行是的 Object[]。如果我们在代码中去掉  @SuppressWarnings("unchecked") 编译器会产生警告。

警告的位置：
```java
private T[] array;

	 public GenericArray(int sz) {
		 //这一句代码的转型会产生警告
		 array = (T[])new Object[sz];
	 }
```
因为有了擦除机制，数组在运行时只能是 Object 类型。如果我们立即为其转型为 T[] 那么在编译器该数组的具体类型将会消失，而编译器可能会错过某些错误检查。正因如此，最好是在集合的内部使用 Object[]。然后你使用数组元素时，添加一个对 T 的转型。
```java
public class GenericArray2<T> {

	private Object[] array;
	  public GenericArray2(int sz) {
	    array = new Object[sz];
	  }
	  public void put(int index, T item) {
	    array[index] = item;
	  }
	  @SuppressWarnings("unchecked")
	  public T get(int index) { return (T)array[index]; }
	  @SuppressWarnings("unchecked")
	  public T[] rep() {
	    return (T[])array; // Warning: unchecked cast
	  }

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		GenericArray2<Integer> gai = new GenericArray2<Integer>(10);
			    for(int i = 0; i < 10; i ++)
			      gai.put(i, i);
			    for(int i = 0; i < 10; i ++)
			     System.out.print(gai.get(i) + " ");
			    System.out.println();

			    Integer[] ia = gai.rep();

	}

}
```
执行结果：
```
0 1 2 3 4 Exception in thread "main" 5 6 7 8 9
java.lang.ClassCastException: [Ljava.lang.Object; cannot be cast to [Ljava.lang.Integer;
	at genericity.GenericArray2.main(GenericArray2.java:28)
```
下面的例子是，我们用 Object[] 去建立了一个数组，在我们使用时在将其转换了 T[]。这样和上边其实没有区别，如果我们运行的时候依然是存在异常的。因为我们依然是视图将一个 Object[] 去转型为一个 Integer[]。

如果我们传递一个类型标记，使他可以在擦除之后恢复具体的类型就可以解决这个问题。
```java
public class GenericArrayWithTypeToken<T> {

	  private T[] array;
	  @SuppressWarnings("unchecked")
	  public GenericArrayWithTypeToken(Class<T> type, int sz) {
	    array = (T[])Array.newInstance(type, sz);
	  }

	  public void put(int index, T item) {
	    array[index] = item;
	  }

	  public T get(int index) { return array[index]; }

	  // Expose the underlying representation:
	  public T[] rep() { return array; }


	public static void main(String[] args) {
		// TODO Auto-generated method stub
		 GenericArrayWithTypeToken<Integer> gai = new GenericArrayWithTypeToken<Integer>(Integer.class, 10);
			    // This now works:
		 Integer[] ia = gai.rep();
	}

}

```
在 JavaSE5 中的标准类库的源代码中，其实存在大量的 Object[] 转型为参数化类型数组的代码。这会产生大量的警告。所以不要迷信源代码哦，源代码也是别人写的。哈哈！

## 边界
在前面我们讨论过边界，边界使得你可以在应用于泛型的参数类型上设置限制条件。尽管这可以使得你强制规定泛型可以应用的类型，但是其更重要的一个效果是按照自己的边界类型去调用方法。

因为擦除移除了类型信息，所以，用无界泛型参数只能去调用 Object 的方法。但是，如果能够将这个参数限制为某个类的子集，那么你就可以用这些类型的子集调用方法。为了执行这种限制，Java 泛型重用了 extends 关键字。这里我们写几个例子，注意 extends 关键字在泛型中和普通的继承中的区别：

接口 HasColor：
```java
public interface HasColor {
	java.awt.Color getColor();
}

```
泛型调用：
```java
public class Colored<T extends HasColor> {
	  T item;
	  Colored(T item) { this.item = item; }
	  T getItem() { return item; }
	  // The bound allows you to call a method:
	  java.awt.Color color() { return item.getColor(); }
}
```
普通类：Dimension
```java
public class Dimension {
	public int x, y, z;
}

```
泛型的第二次调用：
```java
public class ColoredDimension<T extends Dimension & HasColor> {
	  T item;
	  ColoredDimension(T item) { this.item = item; }
	  T getItem() { return item; }
	  java.awt.Color color() { return item.getColor(); }
	  int getX() { return item.x; }
	  int getY() { return item.y; }
	  int getZ() { return item.z; }
}
```
接口：Weight
```java
public interface Weight {
	int weight();
}

```
泛型的第三次使用：
```java
public class Solid<T extends  Dimension & Weight  & HasColor> {
	  T item;
	  Solid(T item) { this.item = item; }
	  T getItem() { return item; }
	  java.awt.Color color() { return item.getColor(); }
	  int getX() { return item.x; }
	  int getY() { return item.y; }
	  int getZ() { return item.z; }
	  int weight() { return item.weight(); }
}

```
创建综合的子类：
```java
public class Bounded extends Dimension implements HasColor,Weight{

	@Override
	public int weight() {
		// TODO Auto-generated method stub
		return 0;
	}

	@Override
	public Color getColor() {
		// TODO Auto-generated method stub
		return null;
	}

}
```
使用泛型类：
```java
public class BasicBounds {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Solid<Bounded> solid = new Solid<Bounded>(new Bounded());
			    solid.color();
			    solid.getY();
			    solid.weight();
	}

}

```
我们可以看到通过泛型可以消除在继承层次上的负责。并且泛型的边界被限制在基类和接口之下，包含基类和接口。

看下面这个例子：

```java
public class HoldItem<T> {
	  T item;
	  HoldItem(T item) { this.item = item; }
	  T getItem() { return item; }
}
```
继承自泛型类：
```java
public class Colored2<T extends HasColor> extends HoldItem<T> {

	  Colored2(T item) { super(item); }
	  java.awt.Color color() { return item.getColor(); }
}

```
第二级继承泛型类：
```java
public class ColoredDimension2<T extends Dimension & HasColor> extends Colored2<T> {
	  ColoredDimension2(T item) {  super(item); }
	  int getX() { return item.x; }
	  int getY() { return item.y; }
	  int getZ() { return item.z; }

	  java.awt.Color color() { return item.getColor(); }
}
```
继续继承：
```java
public class Solid2<T extends Dimension & HasColor & Weight> extends ColoredDimension2<T> {
	  Solid2(T item) {  super(item); }
	  int weight() { return item.weight(); }
}
```
调用：
```java
public class InheritBounds {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Solid2<Bounded> solid2 = new Solid2<Bounded>(new Bounded());
			    solid2.color();
			    solid2.getY();
			    solid2.weight();
	}

}

```
其实这两种方式是类似的，第二种 HoldItem 直接持有一个对象，因此这种行为被继承到 Colored2 中，他也要求其参数与 HasColor 一致。进一步向下扩展层次结构，每一次都添加了新的边界。
