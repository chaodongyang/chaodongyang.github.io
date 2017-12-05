---
layout: post
title: java编程思想之类型信息一
categories: javaThinking
description: java编程思想之类型信息
keywords: java, java类型信息, java编程思想之类型信息，java之类型信息详解
---

运行时类型信息使得你可以在程序运行时发现和使用类型信息。

它使得你从只能在编译器执行面向类型的操作中解放出来，可以使用某些非常强大的程序。对 RTTI 的需要，揭示了面向对象设计中许多有趣的问题。我们在运行时识别对象和类的信息主要有两种方式：一种是传统的 RTTI，它假定我们在编译时已经知道所有的类型；另一种是反射机制，他允许我们在运行时发现和使用类的信息。

## 为什么需要 RTTI
看一个很熟悉的例子，它使用了多态的类层次结构。

![](/images/blog/up.png)

这是一个典型的类层次结构，基类位于顶部，派生类向下扩展。面向对象编程的目的是：让代码只操纵对基类的引用。这样如果代码新添加一个类来扩展程序，就不会影响到原来的代码。在这个 Shape 的接口中定义了 draw() 方法，目的是让所有的派生类都会覆盖，并且由于它是动态绑定的，所以通过泛化的所有的 Shape 引用来调用，也能产生正确的行为。这就是多态。

看下面的层次结构：
```java
abstract class Shape {
	void draw(){
		System.out.println(this + ".draw()");
	}

	abstract public String toString();
}
```
派生类：
```java
public class Circle extends Shape{

	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return "Circle";
	}

}
------------------
public class Square extends Shape{

	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return "Square";
	}

}
------------------
public class Triangle extends Shape{

	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return "Triangle";
	}

}

```
调用并执行：
```java
public class Shapes {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		List<Shape> sList = Arrays.asList(new Circle(),new Square(),new Triangle());

		for (Shape shape : sList) {
			shape.draw();
		}
	}

}
```
执行结果：
```
Circle.draw()
Square.draw()
Triangle.draw()
```
当把 Shape 对象放入 ```List<Shape>``` 的数组时会向上转型。但是在向上转型 Shape 的时候也会丢失具体的 Shape 对象的类型。对于数组而言，它们只是 Shape 类的对象。当从数组中取出元素时，实际上它将所有的元素都当做 Object 来持有，自动将结果转型回 Object。这是 RTTI 最基本的使用形式，因为在 Java 中，所有的类型转换都是在运行时进行正确性的检查。这也是 RTTI 名字的含义：在运行时，识别一个对象的类型。

这个例子中，RTTI 类型转换并不彻底：Object 被转型为 Shape。而不是转型为 Circle 或者其他的具体类型。这是因为我们目前只知道数组中存放的是 Shape 类型。在编译时将容器与 Java 的泛型系统来强制确保这一点，而在运行时用类型转换来确保这一点。接下来就是多态的机制，Shape 对象实际执行什么样的代码，是由引用所指向的具体类型来完成的。通常，也正是这样的要求；你希望大部分代码尽可能的少了解对象的具体类型，而是只与对象家族的一个通用表示打交道。这样代码更容易写，更容易读，更加维护；设计也更容易实现、理解、改变。所以多态是面向对象编程的基本目标。

## Class 对象
要理解 RTTI 在 Java 中的工作原理，首先必须知道类型信息在运行时是如何表示的。这项工作是由称作 Class 对象的特殊对象来完成的，它包含了与类的有关信息。事实上 Class 对象就是用来创建类的所有的常规的对象的。

类是程序的一部分，每个类都有一个 Class 对象。每当编写一个新类，就会产生一个 Class 对象。为了生成这个类的对象，运行这个程序的 Java 虚拟机将使用被称为 “类加载器” 的子系统。所有的类都在其第一次使用时，动态加载到 JVM 中的。当程序创建第一个对类的静态成员引用时，就会加载这个类。这个证明构造器也是静态方法，即使在构造器之前没有 static 关键字。因此，使用 new 操作符创建类的新对象也会被当做对类的静态成员的引用。因此，Java 程序在他开始运行之前并未被完全加载。如果尚未加载，默认的类加载器就会根据类名查找 .class 文件。在这个类的字节码被加载时，他们会接收验证，以确保其没有被破坏，并不包含不良的代码。一旦某个类的 Class 对象被载入内存，他就被用来创建这个类的所有对象。

Candy类：

```java
public class Candy {

	static {
		System.out.println("Candy");
	}

}
```
Gum类：
```java
public class Gum {

	static {
		System.out.println("Gum");
	}

}
```
Cookie类：
```java
public class Cookie {

	static {
		System.out.println("Cookie");
	}

}
```
加载类示例：
```java
public class SweetShop {


	public static void main(String[] args) {
		// TODO Auto-generated method stub
		new Candy();
		try {
			Class.forName("type.Gum");
		} catch (ClassNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		new Cookie();
	}

}

```
执行结果：
```
Candy
Gum
Cookie

```
三个类中每个类都有一个子句，该子句在类第一次被加载时执行。

看这一行：

```java
Class.forName("type.Gum")
```
这个方法是 Class 类的一个静态成员。Class 对象和其他的对象一样，我们可以获取并操作它的引用。forName() 是取得 Class 对象引用的一种方法。他是用包含目标类的文件名的 String 做输入参数，返回的是 Class 对象的引用。注意：forName() 的调用类还没有被加载就加载它，在加载的过程中类的静态子句会被执行。

无论何时只要你想在运行时获取类型信息，就必须首先获得对象的 Class 引用。Class.forName() 就是实现此功能的便捷途径，因为你不需要为了获取Class 引用而持有该类型的对象。但是，如果你已经拥有了一个类型对象，那就可以通过调用 getClass() 方法来获取 Class 引用了，这个方法属于根类 Object 的一部分，它将返回表示该对象的实际类型的 Class 引用。

接口：
```java
public interface HasBatteries {

}

public interface Waterproof {

}

public interface Shoots {

}
```
基类：
```java
public class Toy {

	public Toy() {
		// TODO Auto-generated constructor stub
	}


	public Toy(int i) {
		// TODO Auto-generated constructor stub
	}
}

```
实现类：
```java
public class FancyToy extends Toy implements HasBatteries,Waterproof,Shoots{

	public FancyToy() {
		// TODO Auto-generated constructor stub
		super(1);
	}

}
```
执行：
```java
public class ToyTest {

	static void printInfo(Class class1){
		//产生全限定的类名包含包名
		System.out.println(class1.getName());
		//是否是接口
		System.out.println(class1.isInterface());
		//不包含包名的类名
		System.out.println(class1.getSimpleName());
		//包含包名的类名
		System.out.println(class1.getCanonicalName());

	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Class class1 = null;
		try {
			class1 = Class.forName("type.FancyToy");
		} catch (ClassNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
			System.exit(1);
		}
		printInfo(class1);

		// class1.getInterfaces() 返回包含接口的 Class 对象
		for (Class fClass : class1.getInterfaces()) {
			printInfo(fClass);
		}


		//获取其基类的 Class 对象
		Class up = class1.getSuperclass();
		Object object = null;

		try {
			object = up.newInstance();
		} catch (InstantiationException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IllegalAccessException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

		printInfo(object.getClass());



	}

}

```
执行结果：
```
type.FancyToy
false
FancyToy
type.FancyToy
type.HasBatteries
true
HasBatteries
type.HasBatteries
type.Waterproof
true
Waterproof
type.Waterproof
type.Shoots
true
Shoots
type.Shoots
type.Toy
false
Toy
type.Toy

```
Class 的 newInstance() 方法实现虚拟构造器的一种途径，虚拟构造器允许你声明我不知道你的类型，但无论如何我要正确的创建你。up 之后仅仅是一个 Class 的引用，new 之后会得到 Object 引用，但这个引用指向的是 Toy 对象。

#### 类字面常量
java 还提供了另外一种方法来生成新的 Class 对象的引用，即使用**类字面常量**。就像这样：

```
FancyToy.class
```
这样做不仅简单而且安全，因为他会在编译时受到检查。并且根除了 forName() 方法的调用，所以更高效。类字面常量不仅仅可以应用于普通的类，也可以应用于接口，数组以及基本数据类型。另外对于基本类型的包装器类，还有一个标准的字段 TYPE。和 .Class 效果相同

|    .class      |       .TYPE        |
|   :---         |    :---            |
| boolean.class  |  Boolean.TYPE      |
| byte.class     |  Byte.TYPE         |

我们建议使用 .class 形式，以保持与普通类的一致性。

注意：使用 .class 来创建 Class 对象的引用时，不会自动的初始化该 Class 对象。为了实用类做的准备包含三个步骤：

- 加载，由类加载器执行。将检查字节码，并从这些字节码中创建一个 Class 对象。

- 链接， 在连接阶段将验证类中的字节码，为静态域分配存储空间，并且如果必要的话将解析这个类创建的对其他类的所有引用。

- 初始化，如果该类具有超类，则对其初始化，执行静态初始化器和静态初始化块。

初始化被延迟到了静态方法或者非常数静态域进行首次引用时执行:

```java
class Initable {

	static final int S = 47;
	static final int B = ClassInitialization.random.nextInt(1000);

	static{
		System.out.println("初始化 Initable");
	}
}
------------

public class Initable2 {

	static int stN = 147;

	static{
		System.out.println("初始化 Initable2");
	}
}

--------------
public class Initable3 {
	static int snb = 74;
	static {
		System.out.println("初始化 Initable3");
	}
}

```
调用并执行：
```java
public class ClassInitialization {

	public static Random random = new Random(47);

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Class initc = Initable.class;
		System.out.println("还没有被初始化");
		System.out.println(initc);


		System.out.println(Initable2.stN);

		Class class1 = null;
		try {
			class1 = Class.forName("type.Initable3");
		} catch (ClassNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		System.out.println(class1);		
	}
}
```
执行结果：
```
还没有被初始化
class type.Initable
初始化 Initable2
147
初始化 Initable3
class type.Initable3
```
注意：
```
System.out.println(Initable.S);
System.out.println(Initable.B);
```
如果一个 static final 值是编译期常量，就像 static final int S = 47; 这样。那么这个值不需要进行初始化就可以获取。但是，如果只是将一个域设置为 static final 的，还不足以确保这种行为。

#### 泛化的 Class 引用
Class 引用总是指向某个 Class 对象，它可以制造类的实例，并包含作用于这些实例的所有方法代码。它还包含该类的静态成员，因此，Class 引用表示的就是它所指向的对象的确切类型，而改对象便是 Class 类的一个对象。

我们使用泛型可以对 Class 对象的类型进行限定。

```java
public class Gener {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Class inti = int.class;
		Class<Integer> iClass = int.class;
		iClass = Integer.class;
		inti = double.class;
	}

}
```
通过使用泛型语法可以使得编译器强制的执行额外的类型检查。而且你看到尽管有类型限制，普通的类引用还可以被重新赋值为任何的 Class 对象。

我们如果需要放开这些限制该如何做呢？

```
Clas<number> class1 = int.class;
```
上面的代码是行不通的，虽然 Integer 继承自 numer。但是 Integer class 对象不是 number Class 对象的子类。我们使用通配符来解决这个问题。

```java
public class Wiled {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Class<?> iClass = int.class;
		iClass = double.class;
	}

}
```
```Class<?>``` 优于普通的 Class。即便他们是等价的。```Class<?>``` 表示你并非是碰巧和疏忽，而是使用一个非具体的类引用。为了创建一个 Class 引用，他被限定为某种类型，或该类型的任何子类型，你需要将通配符与 extendx 关键字相结合，创建一个范围。

```java
public class Wiled {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Class<? extends Number> iClass = int.class;
		iClass = double.class;
		iClass = Number.class;
	}

}

```
向 Class 引用添加泛型语法的原因仅仅是为了提供编译期类型检查。

```java
public class Cound {

	private static long counter;
	private final long id = counter++;

	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return Long.toString(id);
	}

}
```
执行：
```java
public class Filed<T> {

	private Class<T> type;

	public Filed(Class<T> type) {
		// TODO Auto-generated constructor stub
		this.type = type;
	}

	public List<T> create(int element) {
		List<T> result = new ArrayList<>();

		for (int i = 0; i <element; i++) {
			try {
				result.add(type.newInstance());
			} catch (Exception e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}

		return result;

	}
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Filed<Cound> filed = new Filed<>(Cound.class);
		System.out.println(filed.create(10));
	}

}
```
执行结果：
```
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```
当我们把泛型用于 Class 对象的时候，type.newInstance() 将返回该对象的确切类型。

#### 新的转型语法
javaSE5 中有一种新的类型转换语法，即 cast() 方法。

```java
public class ClassCast {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Building building = new House();

		Class<House> hClass = House.class;
		House house = hClass.cast(building);

		house = (House)building;
	}

}
```
你编写泛型代码时，如果你存储了 Class 引用，并希望以后通过这个引用来执行转换。

## 类型转换前先做检查
至今为止我们已经的 RTTI 形式包括：

- 传统的类型转换，比如 Shape。

- 代表对象的类型的 Class 对象。通过查询 Class 对象可以获取运行时所需的信息。

- 关键字 instanceof。它返回一个 boolean 值，告诉我们对象是不是某个特定类型的实例。

下面我们来看一个综合的实例代码：

Individual :
```java
public class Individual  {
  private static long counter = 0;
  private final long id = counter++;
  private String name;
  public Individual(String name) { this.name = name; }
  // 'name' is optional:
  public Individual() {}
  public String toString() {
    return getClass().getSimpleName() +
      (name == null ? "" : " " + name);
  }
  public long id() { return id; }
  public boolean equals(Object o) {
    return o instanceof Individual &&
      id == ((Individual)o).id;
  }

}
```
下面是继承他的类：
```java
public class Pet extends Individual{

	public Pet() {
		// TODO Auto-generated constructor stub
		super();
	}

	public Pet(String name) {
		// TODO Auto-generated constructor stub
		super(name);
	}

}
---------
public class Dog extends Pet{
	public Dog() {
		// TODO Auto-generated constructor stub
		super();
	}


	public Dog(String name) {
		// TODO Auto-generated constructor stub
		super(name);
	}
}
---------
public class Mutt extends Dog{

	public Mutt() {
		// TODO Auto-generated constructor stub
		super();
	}

	public Mutt(String name) {
		// TODO Auto-generated constructor stub
		super(name);
	}
}
--------------
public class Cat extends Pet{
	public Cat() {
		// TODO Auto-generated constructor stub
		super();
	}

	public Cat(String name) {
		// TODO Auto-generated constructor stub
		super(name);
	}

}
--------------
public class Manx extends Cat{

	public Manx() {
		// TODO Auto-generated constructor stub
		super();
	}

	public Manx(String name) {
		// TODO Auto-generated constructor stub
		super(name);
	}


}
----------------

```
抽象类：
```java
public abstract class PetCreator {
	private Random random = new Random(47);

	public abstract List<Class<? extends Pet>> types();

	//随机产生一个 pet 对象
	public Pet randomPet(){
		int n = random.nextInt(types().size());
		try {
			return types().get(n).newInstance();
		} catch (InstantiationException | IllegalAccessException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
			return null;
		}

	}

	//产生一个 pet数组
	public Pet[] createArray(int size){
		Pet[] result = new Pet[size];
		for (int i = 0; i < result.length; i++) {
			result[i] = randomPet();
		}
		return result;
	}

	//添加到一个集合中去
	public ArrayList<Pet> arrayList(int size){
		ArrayList<Pet> result = new ArrayList<>();
		Collections.addAll(result, createArray(size));
		return result;
	}
}

```
实现类：
```java
public class ForNameCreator extends PetCreator{

	private static List<Class<? extends Pet>> types = new ArrayList<>();

	private static String[] typesname = {"TypeInfo.Dog","TypeInfo.Mutt","TypeInfo.Cat","TypeInfo.Manx"};

	@SuppressWarnings("unchecked")
	public static void loader() {
		for (String name : typesname) {
			try {
				types.add((Class<? extends Pet>) Class.forName(name));
			} catch (ClassNotFoundException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}

	static{
		loader();
	}

	@Override
	public List<Class<? extends Pet>> types() {
		// TODO Auto-generated method stub
		return types;
	}

}

```
调用并执行：
```java
public class PetCount {

	static Map<String, Integer> map = new HashMap<>();
	static int Peti =0;
	static int Dogi =0;
	static int Mutti =0;
	static int Cati =0;
	static int Manxi =0;

	public static void countPets(PetCreator creator) {
		for (Pet pet : creator.createArray(20)) {
			System.out.println(pet.getClass().getSimpleName());
			if (pet instanceof Pet) {
				map.put("Pet",Peti++);
			}

			if (pet instanceof Dog) {
				map.put("Dog",Dogi++);
			}
			if (pet instanceof Mutt) {
				map.put("Mutt",Mutti++);
			}
			if (pet instanceof Cat) {
				map.put("Cat",Cati++);
			}
			if (pet instanceof Manx) {
				map.put("Manx",Manxi++);
			}
		}

		System.out.print(map);
	}
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		countPets(new ForNameCreator());
	}

}

```
执行结果：
```
Cat Mutt Cat Dog Dog Cat Dog
Mutt Cat Cat Mutt Manx Mutt
Dog Dog Cat Manx Dog Manx Cat
{Cat=9, Manx=2, Mutt=3, Dog=9, Pet=19}
```
我们看到我们使用了 instanceof 去比较数组中的每个 Pet 进行测试和计数。对 instanceof 有比较严格的限制：只可将其与命名类型进行比较，而不能与 Class 对象进行比较。

#### 使用类字面常量
我们用类字面常量来重新修改一下上面的例子：

```java
public class ListerPetCreator extends PetCreator{

	public static final List<Class<? extends Pet>> alltype = Collections.unmodifiableList(Arrays.asList(Pet.class,Dog.class,Mutt.class,Cat.class,Manx.class));


	@Override
	public List<Class<? extends Pet>> types() {
		// TODO Auto-generated method stub
		return alltype;
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		System.out.println(alltype);
	}


}
```
这一次我们生成的 types() 方法不需要放在 try 块中，因为它会在编译时得到检查，因此它不会抛出任何异常。这与 Class.forName() 不一样。

为了默认使用第二种方式我们可以建立一个 ListerPetCreator 的外观：

```java
public class Pets {

	public static final PetCreator CREATOR = new ListerPetCreator();

	public static Pet randomPet() {
		return CREATOR.randomPet();
	}

	public static Pet[] creatArray(int size) {
		return CREATOR.createArray(size);
	}

	public static ArrayList<Pet> arrayList(int size) {
		return CREATOR.arrayList(size);
	}

}

```
调用并执行：
```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		PetCount.countPets(Pets.CREATOR);
	}
```
执行结果：
```
{Cat=10, Manx=2, Mutt=1, Dog=5, Pet=19}
```
#### 动态的 instanceof
Class.isInstance 方法提供一种动态的测试对象的途径。

```java
class A{  

}  

class B extends A {  

}  

class C extends B {  

}  


public class tt {  

    /**
     * @param args
     */  

    public static void main(String[] args) {  
        // TODO Auto-generated method stub  
        C c = new C();  
        B b = new B();  
        A a = new A();  

        B bc = new C();  
        A ac = new C();  

        System.out.println(c instanceof C);  
        System.out.println(c instanceof B);  
        System.out.println(c instanceof A);  

        System.out.println();  

        System.out.println(c.getClass().isInstance(c));  
        System.out.println(c.getClass().isInstance(b));  
        System.out.println(c.getClass().isInstance(a));  

        System.out.println();  

        System.out.println(c.getClass().isInstance(bc));  
        System.out.println(c.getClass().isInstance(ac));  
		}

	}
```
执行结果：
```
true
true
true

true
false
false

true
true
```
instanceof运算符 只被用于对象引用变量，检查左边的被测试对象 是不是 右边类或接口的 实例化。如果被测对象是null值，则测试结果总是false。
形象地：自身实例或子类实例 instanceof 自身类   返回true

Class类的isInstance(Object obj)方法，obj是被测试的对象，如果obj是调用这个方法的class或接口 的实例，则返回true。这个方法是instanceof运算符的动态等价。
形象地：自身类.class.isInstance(自身实例或子类实例)  返回true

#### 注册工厂
这里我们使用工厂设计模式，将对象的创建工作交给类自己去做。工厂方法可以被多态的调用，从而为你创建恰当的类型。

```java
public interface Factory<T> {
  T create();
}
```
泛型参数 T 可以使得 create() 可以在每种 Factory 实现中返回不同的类型。这也是可协变返回类型的运用。

具体的类：
```java
public class Fulter extends Part{

	public static class Factory implements TypeInfo.Factory<Fulter>{

		@Override
		public Fulter create() {
			// TODO Auto-generated method stub
			return new Fulter();
		}

	}


}

```
第二个具体的类：
```java

public class Air extends Part {

	public static class Factory implements TypeInfo.Factory<Air>{

		@Override
		public Air create() {
			// TODO Auto-generated method stub
			return new Air();
		}

	}

}
```
创建工厂:
```java
public class Part {

	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return getClass().getSimpleName();
	}

	static List<Factory<? extends Part>> pFactories = new ArrayList<>();
	static{
		pFactories.add(new Fulter.Factory());
		pFactories.add(new Air.Factory());
	}

	private static Random random = new Random(47);

	public static Part creatRandom() {
		int n = random.nextInt(pFactories.size());
		return pFactories.get(n).create();
	}
}
```
调用代码：
```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		for (int i = 0; i < 2; i++) {
			System.out.println(Part.creatRandom());
		}
	}
```
执行结果：
```
Air
Fulter
```
## instanceof 与 Class 的等价性
查询类型信息时，以 instanceof的形式产生的结果与直接比较 Class 对象有一个很重要的差别：

```java
public class FamilyTest {
	static void test(Object xObject){
		System.out.println(xObject.getClass());
		System.out.println(xObject instanceof Base);
		System.out.println(xObject instanceof Derived);

		System.out.println(Base.class.isInstance(xObject));
		System.out.println(Derived.class.isInstance(xObject));

		System.out.println((xObject.getClass()).equals(Base.class));
		System.out.println((xObject.getClass()).equals(Derived.class));


		System.out.println((xObject.getClass()) == (Base.class));
		System.out.println((xObject.getClass())== (Derived.class));

		System.out.println("-------------------------");
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		test(new Base());
	}

}

```
执行结果：
```
class TypeInfo.Base
true
false
true
false
true
false
true
false
```
test() 方法使用了两种形式的 instanceof 来进行参数类型的检查，结果是一样的。而 equals() 和 == 也一样。但是两组的测试结果却是不一样的。instanceof 对比的是类型的概念，它指你是这个类吗，或是你是这个派生类吗？而如果用 == 比较的实际的 Class 对象，就没有考虑继承。它只是个确切的类型或者不是。
