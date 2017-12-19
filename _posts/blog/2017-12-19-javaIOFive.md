---
layout: post
title: java编程思想之 I/O 系统(序列化)
categories: javaThinking
description: java编程思想之I/O 系统详解
keywords: java, javaI/O 系统,java 输入输出流，java I/O
---

一般而言对象将会在程序结束后销毁。如果对象能够在程序不运行的情况下仍然能够存在并保存信息，那将是非常有用的。这样，在程序下次运行时，该对象将被重建并且拥有的信息和上次运行时所拥有的信息相同。当然我们也可以用文件或者数据库来保存信息，但是在面向对象的精神下能够把一个对象声明为持久性的，并为我们处理掉所有的细节，那将会显得十分方便。

## 对象序列化
Java 对象序列化将那些实现了 Serializable 接口的对象转换为一个字节序列，并能够在以后将这个字节序列完全恢复为原来的对象。这一过程也可以通过网络进行；这意味着序列化机制能够弥补不同操作系统之间的差异。

利用对象的序列化可以实现轻量级持久性。持久性意味着一个对象的生命周期并不取决于程序是否在执行；他可以存在于程序的调用之间。通过将一个序列化对象写入磁盘，然后在重新调用程序时恢复该对象，就能够实现持久化的效果。之所以称之为轻量级，是因为不能用某种关键字来简单的定义对象，并让系统自动维护其他细节问题。相反，对象必须在程序中显式的序列化和反序列化还原。

对象序列化的概念加入到语言中为了支持两种主要特性。一是 Java 的远程方法调用，他使得存活于其他计算机上的对象就像存活于本机一样。当远程对象发送消息时，需要通过对象序列化来传送参数和返回值。再者，对 Java beans 来说，对象的序列化也是必须的。使用一个 bean 时，一般情况下是在设计阶段对它的状态信息进行配置。这种状态信息必须保存下来，并且在程序启动时进行恢复；这种具体工作是由对象序列化完成的。

只要对象实现了 Serializable 接口。序列化就实现了，这是非常简单的。要序列化一个对象首先要创建某些 OutputStream 对象，然后将其封装成一个 ObjectOutputStream 对象内。这时，调用 writeObject() 即可将对象序列化，并将其发送给 OutputStream。要反向进行该过程，需要将一个 InputStream 封装在 ObjectInputStream 内，然后调用 readObject()。和之前一样我们会获得一个引用，它指向一个向上转型的 Object，所以必须向下转型才能直接设置它们。

对象序列化不仅保存了对象的全景图，而且能够追踪对象内所包含的所有引用，并保存那些对象；接着又能对对象内所含的每个引用进行追踪。这种情况被称为对象网，单个对象可与之建立连接，而且他还包含了对象的引用数组以及成员对象。

下面示例：通过对链接的对象生成一个 worm(蠕虫) 对序列化机制进行了测试。每个对象都是 worm 中的下一段链接，同时又与属于不同类的对象引用数组链接：
```java
public class Worm implements Serializable{

	private static Random random = new Random(47);
	private Data[] d = {new Data(random.nextInt(10)),new Data(random.nextInt(10)),new Data(random.nextInt(10))};

	private Worm next;
	private char c;
	protected Worm(int i, char c) {
		this.c = c;
		if (-- i >0) {
			next = new Worm(i, (char)(c+1));
		}
	}

	public Worm() {
		// TODO Auto-generated constructor stub
	}

	@Override
	public String toString() {
		StringBuilder result = new StringBuilder(":");
		result.append(c);
		result.append("(");
		for (Data data : d) {
			result.append(data);
		}
		result.append(")");
		if (next != null) {
			result.append(next);
		}
		return result.toString();
	}

	public static void main(String[] args) throws ClassNotFoundException,IOException{
		Worm w = new Worm(6,'a');
		System.err.println("w =" +w);
		ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("worm.out"));
		out.writeObject("worm storge\n");
		out.writeObject(w);
		out.close();

		ObjectInputStream in = new ObjectInputStream(new FileInputStream("worm.out"));
		String string = (String)in.readObject();
		Worm w2 = (Worm)in.readObject();
		System.out.println("w2 :" + w2);

		ByteArrayOutputStream bout = new ByteArrayOutputStream();
		ObjectOutputStream out2 = new ObjectOutputStream(bout);
		out2.writeObject("worm2 storge\n");
		out2.writeObject(w);
		out2.flush();

		ObjectInputStream in2 = new ObjectInputStream(new ByteArrayInputStream(bout.toByteArray()));

		string = (String) in2.readObject();
		Worm w3 = (Worm)in2.readObject();
		System.out.println(string + "w3 = "+ w3);

	}



}

```
测试结果：
```
w =:a(853):b(119):c(802):d(788):e(199):f(881)
w2 ::a(853):b(119):c(802):d(788):e(199):f(881)
worm2 storge
w3 = :a(853):b(119):c(802):d(788):e(199):f(881)
```
Worm 内的 data[] 数组是随机数初始化的，我们看到恢复的信息是一致的，这样就不用怀疑编辑器保留了原始信息。每个 Worm 段都有一个 char 加以标记。该 char 是在递归生成链接的 Worm 列表时自动产生的。要创建一个 Worm 必须告诉编译器你所希望的长度。以上的操作使得事情变得复杂，实际上对象序列化的过程是非常简单的。一旦从另外的某个流创建了一个 ObjectOutputStream，writeObject() 就会将对象序列化。

有两段很相似的代码。一个读写的是文件，而另一个读写的是字节数组。可利用序列化将对象读写到任何 DataInputStream 或者 DataOutputStream。甚至包括网络。

从输出的结果来看，被还原的对象确实包含了原对象中的所有连接。注意：在对一个 Serializable 对象进行还原的过程中，没有调用任何的构造器。整个对象都是从 InputStream 中取得数据恢复而来的。

#### 寻找类
将一个对象从它的序列化状态恢复过来，有哪些工作是必须的呢？假如我们将一个对象序列化，并通过网络将其作为文件传递给另一台计算机；那么另一台计算机可以只利用该文件的内容来还原这个对象吗？

做一个实验。创建一个文件：
```java
public class Alien implements Serializable{

}

```
创建一个用于序列化一个 Alien 对象的类：
```java
public class FreezeAlien {

	public static void main(String[] args) throws Exception{
		// TODO Auto-generated method stub
		ObjectOutput out = new ObjectOutputStream(new FileOutputStream("x.file"));
		Alien quelek = new Alien();
		out.writeObject(quelek);
	}

}
```
一旦程序被编译和运行，他会在目录下产生一个名为 x.file 的文件。

测试类：
```java
public class ThawAline {

	public static void main(String[] args) throws Exception{
		// TODO Auto-generated method stub
		ObjectInputStream in = new ObjectInputStream(new FileInputStream(new File("D:/eclipseWorkspace/ThreadTest/x.file")));
		Object object = in.readObject();
		System.out.println(object.getClass());
	}

}

```
测试结果：
```
class IOFour.Alien
```
注意：路径必须是 Java 虚拟机能够找到的路径。

#### 序列化的控制
默认的序列化机制并不难操作。如果有特殊的需求该怎么办呢？例如：也许要考虑特殊的安全问题，而且你不希望对象的某一部分被序列化；或者一个对象被还原以后，某子对象需要重新建立，从而不必将该子对象重新建立。

这些特殊的情况，可以通过实现 Externalizable 接口代替实现 Serializable 接口对序列化的过程进行控制。这个 Externalizable 接口继承了 Serializable 接口，同时添加了两个方法：writeExternal() 和 readExternal()。这两个方法会在序列化和反序列化还原的过程中被自动调用，以便执行一些特殊的操作。

下面的例子展示了 Externalizable 接口方法的简单实现：

Blip1 类：
```java
public class Blip1 implements Externalizable{

	public Blip1() {
		System.out.println("Blip1 构造函数");
	}

	@Override
	public void writeExternal(ObjectOutput out) throws IOException {
		System.out.println("Blip1 .writeExternal");

	}

	@Override
	public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
		System.out.println("Blip1 .readExternal");
	}

}

```

Blip2 类：
```java
public class Blip2 implements Externalizable{

	Blip2() {
		System.out.println("Blip2 构造函数");
	}

	@Override
	public void writeExternal(ObjectOutput out) throws IOException {
		System.out.println("Blip2 .writeExternal");

	}

	@Override
	public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
		System.out.println("Blip2 .readExternal");
	}

}

```
测试类：
```java
public class Blips {

	public static void main(String[] args) throws IOException,ClassNotFoundException{
		Blip1 blip1 = new Blip1();
		Blip2 blip2 = new Blip2();

		ObjectOutputStream oStream = new ObjectOutputStream(new FileOutputStream("Blips.oyt"));
		oStream.writeObject(blip1);
		oStream.writeObject(blip2);
		oStream.close();

		ObjectInputStream inputStream = new ObjectInputStream(new FileInputStream("D:/eclipseWorkspace/ThreadTest/Blips.oyt"));
		blip1 = (Blip1)inputStream.readObject();
		blip2 = (Blip2)inputStream.readObject();


	}

}

```
测试结果：
```java
Blip1 构造函数
Blip2 构造函数
Blip1 .writeExternal
Blip2 .writeExternal
Blip1 构造函数
Blip1 .readExternal
Exception in thread "main" java.io.InvalidClassException: IOFour.Blip2; no valid constructor
	at java.io.ObjectStreamClass$ExceptionInfo.newInvalidClassException(ObjectStreamClass.java:150)
	at java.io.ObjectStreamClass.checkDeserialize(ObjectStreamClass.java:790)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:1782)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1353)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:373)
	at IOFour.Blips.main(Blips.java:22)

```
Blip1 和 Blip2 的的区别是：Blip1 的构造器是公共的，Blip2 的构造器不是，这样就会在恢复时造成异常。上面已经报错了。恢复 Blip1 会调用默认的构造器。这与恢复一个 Serializable 对象不同。对于 Serializable 对象完全以它存储的二进制为基础来构造，而不调用构造器。而对于一个 Externalizable 对象，所有的普通的默认构造器都会被调用，然后调用 readExternal()。必须注意这一点：所有的默认构造器都会被调用，才能使 Externalizable 对象产生正确的行为。

下面的例子示范如何完整保存和恢复一个 Externalizable 对象：
```java
public class Blip3 implements Externalizable{

	private int i;
	private String s;

	public Blip3() {
		System.out.println("无参的构造函数");
	}



	protected Blip3(int i, String s) {
		super();
		System.out.println("有参数的构造函数");
		this.i = i;
		this.s = s;
	}

	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return s+i;
	}



	@Override
	public void writeExternal(ObjectOutput out) throws IOException {
		System.out.println("序列化对象");
		out.writeObject(s);
		out.writeInt(i);

	}

	@Override
	public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
		System.out.println("恢复对象");
		s = (String)in.readObject();
		i = in.readInt();
	}

	public static void main(String[] args) throws IOException,ClassNotFoundException{
		Blip3 blip3 = new Blip3(47,"a string");

		System.out.println(blip3);

		ObjectOutputStream oStream = new ObjectOutputStream(new FileOutputStream("Blip3.out"));
		oStream.writeObject(blip3);
		oStream.close();

		ObjectInputStream inputStream = new ObjectInputStream(new FileInputStream("D:/eclipseWorkspace/ThreadTest/Blip3.out"));
		blip3 = (Blip3)inputStream.readObject();
		System.out.println(blip3);

	}


}

```
测试结果：
```
有参数的构造函数
a string47
序列化对象
无参的构造函数
恢复对象
a string47
```
其中，字段 s 和 i 只在第二个构造器中初始化，而不是在默认的构造器中初始化。这意味着假如不在 readExternal() 中初始化 s 和 i。s 就会为 null，i 就会为 0。因此为了程序的正确运行，我们不仅需要在 writeExternal() 方法中将对象写入，还必须在 readExternal() 方法中恢复。

**transient (瞬时) 关键字**

当我们对序列化进行控制时，可能某个特定子对象不想让 Java 的序列化机制自动保存于恢复。如果子对象表示的是不希望被保存的敏感信息比如密码，通常就会面临这种情况。即使对象中的这些信息是私有的，一经序列化处理，人们就可以通过改文件或者拦截网络传输的方式来访问到它。

有一种办法是将类实现 Externalizable 接口，这样一来没有任何东西可以自动序列化，并且可以在 writeExternal() 内部只对所需要的部分进行序列化。

然而如果正在使用的是一个 Serializable 对象，那么所有的操作都会自动的序列化。为了能够予以控制，可以用 transient 关键字逐个字段的关闭序列化，他的意思是不用你保存或恢复数据。

示例代码：
```java
public class Logon implements Serializable{

	  private Date date = new Date();
	  private String username;
	  //不需要保存
	  private transient String password;
	  public Logon(String name, String pwd) {
	    username = name;
	    password = pwd;
	  }

	  public String toString() {
	    return "logon info: \n   username: " + username +
	      "\n   date: " + date + "\n   password: " + password;
	  }
	  public static void main(String[] args) throws Exception {
	    Logon a = new Logon("Hulk", "myLittlePony");
	    Print.print("logon a = " + a);
	    ObjectOutputStream o = new ObjectOutputStream(new FileOutputStream("Logon.out"));
	    o.writeObject(a);
	    o.close();
	    TimeUnit.SECONDS.sleep(1); // Delay
	    // Now get them back:
	    ObjectInputStream in = new ObjectInputStream(new FileInputStream("Logon.out"));
	    Print.print("Recovering object at " + new Date());
	    a = (Logon)in.readObject();
	    Print.print("logon a = " + a);
	  }

}

```
测试结果：
```
logon a = logon info:
   username: Hulk
   date: Tue Dec 19 14:42:57 CST 2017
   password: myLittlePony
Recovering object at Tue Dec 19 14:42:58 CST 2017
logon a = logon info:
   username: Hulk
   date: Tue Dec 19 14:42:57 CST 2017
   password: null
```

可以看到 password 是用  transient 修饰的。所以不会被自动保存到磁盘；另外，自动序列化机制也会尝试恢复它。当对象被恢复时，password 域就会变成 null。由于 Externalizable 对象在默认情况下不保存他们的任何字段，所以 transient 关键字只能和 Serializable 对象一起使用。

**Externalizable 的替代方法**

如果我们必须实现 Serializable 接口，那么还有另外一种方法就是添加名为 writeObject() 和 readObject() 的方法。注意是添加既不是覆盖也不是实现。这样一旦对象被序列化或者反序列化，就会自动的分别调用这两个方法。也就是说提供这两个方法就会使用它们，而不是默认的自动化机制。

接口中定义的所有东西都是 public 的，因此如果 writeObject() 和 readObject() 必须是私有的，那么这就不是接口的一部分了。因此我们必须完全尊徐其签名特征方法，所以效果和实现接口是一样的。在调用 ObjectOutputStream.writeObject() 时，会检查传递的 Serializable 对象，看看是否实现了自己的 writedObject()。如果是这样，就跳过正常的序列化过程并调用它的 writeObject()。readObject() 的情形与此相同。还有另外一个技巧，在你的 writedObject() 内部，可以调用 defaultWriteObject() 来选择执行默认的 writeObject()。

简单的示例演示如何对 Serializable 对象的存储与恢复进行控制：
```java
public class SerialCtl implements Serializable {
	  private String a;
	  private transient String b;
	  public SerialCtl(String aa, String bb) {
	    a = "Not Transient: " + aa;
	    b = "Transient: " + bb;
	  }
	  public String toString() { return a + "\n" + b; }
	  private void writeObject(ObjectOutputStream stream)throws IOException {
	    stream.defaultWriteObject();
	    stream.writeObject(b);
	  }

	  private void readObject(ObjectInputStream stream)throws IOException, ClassNotFoundException {
	    stream.defaultReadObject();
	    b = (String)stream.readObject();
	  }

	  public static void main(String[] args)throws IOException, ClassNotFoundException {
	    SerialCtl sc = new SerialCtl("Test1", "Test2");
	    System.out.println("Before:\n" + sc);
	    ByteArrayOutputStream buf= new ByteArrayOutputStream();
	    ObjectOutputStream o = new ObjectOutputStream(buf);
	    o.writeObject(sc);
	    // Now get it back:
	    ObjectInputStream in = new ObjectInputStream(
	      new ByteArrayInputStream(buf.toByteArray()));
	    SerialCtl sc2 = (SerialCtl)in.readObject();
	    System.out.println("After:\n" + sc2);
	  }
}

```
测试结果：
```
Before:
Not Transient: Test1
Transient: Test2
After:
Not Transient: Test1
Transient: Test2
```
有一个普通的 String 字段，而另外一个是 transient 字段，非 transient 字段由 defaultWriteObject 方法保存，而 transient 字段必须在程序中明确保存和恢复。如果我们打算使用默认的机制写入对象的非 transient 部分，那么必须调用 defaultWriteObject 作为 writeObject() 中的第一个操作，并且在恢复时让 defaultReadObject 作为 readObject() 的第一个操作。

**版本控制**

有时可能想要改变可序列化的版本。虽然 Java 支持这种做法，但是却可能只在特殊情况下才这么做，此外，需要对它有相当深的了解。这是因为 Java 的版本控制机制过于简单，因而不能在任何场合可靠的运转。需要深入了解看 JDK 的文档。

#### 使用持久性
使用序列化的一个诱人的想法是：存储程序的一些状态，以便我们可以很容易的将程序恢复到当前状态。先来看几个问题；如果我们将两个对象序列化，他们都具有指向第三个对象的引用。会发生什么呢？当我们从序列化状态恢复这两个对象时，第三个对象只会出现一次吗？如果将这两个序列化对象恢复成独立的文件，又会怎样呢？

示例代码：
```java
public class MyWorld {

	public static void main(String[] args) throws IOException,ClassNotFoundException{
		    House house = new House();
		    List<Animal> animals = new ArrayList<Animal>();
		    animals.add(new Animal("Bosco the dog", house));
		    animals.add(new Animal("Ralph the hamster", house));
		    animals.add(new Animal("Molly the cat", house));
		    Print.print("animals: " + animals);

		    ByteArrayOutputStream buf1 = new ByteArrayOutputStream();
		    ObjectOutputStream o1 = new ObjectOutputStream(buf1);
		    o1.writeObject(animals);
		    o1.writeObject(animals); // Write a 2nd set
		    // Write to a different stream:
		    ByteArrayOutputStream buf2 =new ByteArrayOutputStream();
		    ObjectOutputStream o2 = new ObjectOutputStream(buf2);
		    o2.writeObject(animals);
		    // Now get them back:
		    ObjectInputStream in1 = new ObjectInputStream(new ByteArrayInputStream(buf1.toByteArray()));
		    ObjectInputStream in2 = new ObjectInputStream(new ByteArrayInputStream(buf2.toByteArray()));
		    List
		      animals1 = (List)in1.readObject(),
		      animals2 = (List)in1.readObject(),
		      animals3 = (List)in2.readObject();
		    Print.print("animals1: " + animals1);
		    Print.print("animals2: " + animals2);
		    Print.print("animals3: " + animals3);

	}

}

```
测试结果：
```
animals: [Bosco the dog[IOFour.Animal@15db9742],IOFour.House@6d06d69c
, Ralph the hamster[IOFour.Animal@7852e922],IOFour.House@6d06d69c
, Molly the cat[IOFour.Animal@4e25154f],IOFour.House@6d06d69c
]
animals1: [Bosco the dog[IOFour.Animal@7ba4f24f],IOFour.House@3b9a45b3
, Ralph the hamster[IOFour.Animal@7699a589],IOFour.House@3b9a45b3
, Molly the cat[IOFour.Animal@58372a00],IOFour.House@3b9a45b3
]
animals2: [Bosco the dog[IOFour.Animal@7ba4f24f],IOFour.House@3b9a45b3
, Ralph the hamster[IOFour.Animal@7699a589],IOFour.House@3b9a45b3
, Molly the cat[IOFour.Animal@58372a00],IOFour.House@3b9a45b3
]
animals3: [Bosco the dog[IOFour.Animal@4dd8dc3],IOFour.House@6d03e736
, Ralph the hamster[IOFour.Animal@568db2f2],IOFour.House@6d03e736
, Molly the cat[IOFour.Animal@378bf509],IOFour.House@6d03e736
]

```
在上面的例子中，Animal 对象包含了 House 类型的字段。我们创建了一个 Animal 列表并将其两次序列化，分别送至不同的流。当期被反序列化还原打印时，我们可以看到执行后的结果。我们希望反序列化之后对象地址与原来的地址不同。但是看到结果 animals1 和 animals2 的结果是相同的，包括二者共享的指向 House 的引用。另一方面，当恢复 animals3 时，系统无法知道另外一个流内的对象别名，因此会产生不同的对象网。如果我们想保存系统的状态，最安全的办法是将其作为原子操作进行序列化。如果我们序列化了某些东西，再去做其他的一些工作，那么将无法安全的保存系统状态。

下面的示例是一个想想的计算机辅助系统，它引入了 static 字段的问题；查看 JDK 文档会发现 class 是 Serializable 的。因此只需要直接将 Class 对象序列化，就可以很容易的保存 static 字段。
```java
abstract class Shape implements Serializable {
	  public static final int RED = 1, BLUE = 2, GREEN = 3;
	  private int xPos, yPos, dimension;
	  private static Random rand = new Random(47);
	  private static int counter = 0;
	  public abstract void setColor(int newColor);
	  public abstract int getColor();

	  public Shape(int xVal, int yVal, int dim) {
	    xPos = xVal;
	    yPos = yVal;
	    dimension = dim;
	  }

	  public String toString() {
	    return getClass() +
	      "color[" + getColor() + "] xPos[" + xPos +
	      "] yPos[" + yPos + "] dim[" + dimension + "]\n";
	  }

	  public static Shape randomFactory() {
	    int xVal = rand.nextInt(100);
	    int yVal = rand.nextInt(100);
	    int dim = rand.nextInt(100);
	    switch(counter++ % 3) {
	      default:
	      case 0: return new Circle(xVal, yVal, dim);
	      case 1: return new Square(xVal, yVal, dim);
	      case 2: return new Line(xVal, yVal, dim);
	    }
	  }

}


class Circle extends Shape {
	  private static int color = RED;
	  public Circle(int xVal, int yVal, int dim) {
	    super(xVal, yVal, dim);
	  }
	  public void setColor(int newColor) {
		  color = newColor;
	  }
	  public int getColor() {
		  return color;
	  }
}


class Square extends Shape {
	  private static int color;
	  public Square(int xVal, int yVal, int dim) {
	    super(xVal, yVal, dim);
	    color = RED;
	  }
	  public void setColor(int newColor) {
		  color = newColor;
	  }
	  public int getColor() {
		  return color;
	  }
}


class Line extends Shape {
	  private static int color = RED;

	  public static void serializeStaticState(ObjectOutputStream os) throws IOException {
		  os.writeInt(color);
	  }

	  public static void deserializeStaticState(ObjectInputStream os) throws IOException {
		  color = os.readInt();
	  }

	  public Line(int xVal, int yVal, int dim) {
	    super(xVal, yVal, dim);
	  }

	  public void setColor(int newColor) {
		  color = newColor;
	  }

	  public int getColor() {
		  return color;
	  }

}
```
测试类：
```java
public class StoreCADState {
	public static void main(String[] args)  throws Exception{
		List<Class<? extends Shape>> shapeTypes = new ArrayList<Class<? extends Shape>>();
			    // Add references to the class objects:
			    shapeTypes.add(Circle.class);
			    shapeTypes.add(Square.class);
			    shapeTypes.add(Line.class);

			    List<Shape> shapes = new ArrayList<Shape>();
			    // Make some shapes:
			    for(int i = 0; i < 10; i++)
			      shapes.add(Shape.randomFactory());

			    // Set all the static colors to GREEN:
			    for(int i = 0; i < 10; i++)
			      ((Shape)shapes.get(i)).setColor(Shape.GREEN);

			    // Save the state vector:
			    ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("CADState.out"));
			    out.writeObject(shapeTypes);
			    Line.serializeStaticState(out);
			    out.writeObject(shapes);
			    // Display the shapes:
			    System.out.println(shapes);
	}
}

```
测试结果：
```
[class IOFour.Circlecolor[3] xPos[58] yPos[55] dim[93]
, class IOFour.Squarecolor[3] xPos[61] yPos[61] dim[29]
, class IOFour.Linecolor[3] xPos[68] yPos[0] dim[22]
, class IOFour.Circlecolor[3] xPos[7] yPos[88] dim[28]
, class IOFour.Squarecolor[3] xPos[51] yPos[89] dim[9]
, class IOFour.Linecolor[3] xPos[78] yPos[98] dim[61]
, class IOFour.Circlecolor[3] xPos[20] yPos[58] dim[16]
, class IOFour.Squarecolor[3] xPos[40] yPos[11] dim[22]
, class IOFour.Linecolor[3] xPos[4] yPos[83] dim[6]
, class IOFour.Circlecolor[3] xPos[75] yPos[10] dim[42]
]

```
Shape 类实现了 Serializable 接口，所以任何自 Shape 继承的类也都会自动是 Serializable 的。每个派生自 Shape 的类都含有一个 static 字段，用来确定颜色。在最终的测试代码中一个 ArrayList 用来保存 Class 对象，而另一个保存几何对象。

如何恢复对象:
```java
public class RecoverCADState {

	public static void main(String[] args) throws Exception{
		ObjectInputStream in = new ObjectInputStream(new FileInputStream("CADState.out"));
		// Read in the same order they were written:
	    List<Class<? extends Shape>> shapeTypes =(List<Class<? extends Shape>>)in.readObject();
		Line.deserializeStaticState(in);
	    List<Shape> shapes = (List<Shape>)in.readObject();
		System.out.println(shapes);

	}

}

```
测试结果：
```
[class IOFour.Circlecolor[1] xPos[58] yPos[55] dim[93]
, class IOFour.Squarecolor[0] xPos[61] yPos[61] dim[29]
, class IOFour.Linecolor[3] xPos[68] yPos[0] dim[22]
, class IOFour.Circlecolor[1] xPos[7] yPos[88] dim[28]
, class IOFour.Squarecolor[0] xPos[51] yPos[89] dim[9]
, class IOFour.Linecolor[3] xPos[78] yPos[98] dim[61]
, class IOFour.Circlecolor[1] xPos[20] yPos[58] dim[16]
, class IOFour.Squarecolor[0] xPos[40] yPos[11] dim[22]
, class IOFour.Linecolor[3] xPos[4] yPos[83] dim[6]
, class IOFour.Circlecolor[1] xPos[75] yPos[10] dim[42]
]

```

对比上面的结果我们可以看到，除了对 static 的值读取出了问题，其他的都是相同的。所有读回的颜色应该也是 3，但是事实却并非如此。这看上去 static 根本没有被序列化！确实如此，尽管 Class 类是 Serializable 的，但是却不能序列化静态类型的数据。加入想要序列化 static 必须手动实现。比如 Line 类中的哪样。serializeStaticState() 和 deserializeStaticState() 作为存储和读取过程的一部分被显式的调用。注意:必须维护写入和读取的顺序。
