---
layout: post
title: java编程思想之多态
categories: javaThinking
description: java编程思想之多态
keywords: java, java多态, 多态，多态详解
---

在面向对象的程序设计语言中，多态是继数据抽象和继承之后的第三种基本特征。

多态通过分离做什么和怎么做，从另一个角度将接口和实现分离开来。多态不但能够改善代码的组织结构和可读性，还能够创建可扩展的程序。即无论在项目最初创建时还是在需要添加新功能时都可以生长的程序。

封装通过合并特征和行为来创建新的数据类型。隐藏是通过将细节私有化把接口和实现分离开来。而多态的作用则是消除类与类之间的耦合关系。继承允许我们将对象视为它自己本身的类型或基本类型来处理，而同一份代码也就可以运行在不同的类型之上。多态允许我们表现之出一种类型与其它相似类型之间的区别，只要他们是基于同一个父类导出的。

## 再论向上转型

向上转型中，对象既可以作为它的本身的类型使用，也可以作为它的基类类型来使用。这种把对象的引用当做它基类类型的引用的做法称作向上转型。因为继承树的画法是基类放置在上面。

首先看下面这个例子：

枚举类：
```java
public enum Note {
	MID,SH,AD;
}
```
基类：
```java
public class Instrument {

	public void  play(Note note) {
		System.out.println("唱歌");
	}

}
```
继承类：
```java
public class Wind extends Instrument{

	@Override
	public void play(Note note) {
		// TODO Auto-generated method stub
		System.out.println("Window.paly"+note);
	}
}
```
调用：
```java
public class UpOverloadTest {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Wind wind = new Wind();
		tune(wind);
	}

	public static void  tune(Instrument instrument) {
		instrument.play(Note.MID);
	}

}
```
执行结果：
```
Window.palyMID
```
tune() 方法接受 Instrument 引用，同时也接受任何继承自它的引用。我们看到当一个对象 wind 传递给它时并没有进行任何的转换。从 wind 转型为 Instrument 可能会缩小接口。但不会比 Instrument 的全部接口更窄。

如果我们不去接受一个基类类型引用而是直接接受一个对象的类型也是行得通的。那么就会产生一个问题，我们需要为我们每一个对象的类去写一个接受这个对象的 tune() 方法。这样做的工作量是非常大的。还有就是我们如果忘记写了某个方法编译器也不会报错。但是我们只写一个简单的 tune() 方法让他去接收一个基类类型的引用，而不是那些特殊的导出类。这正是多态所允许的。

## 转型原因

我们看到上述的程序最后输出的结果是：Window.palyMID。这似乎看起来没有任何问题。因为我们传递的正是 Window 对象。我们来看一下这个方法

```java
public static void  tune(Instrument instrument) {
  instrument.play(Note.MID);
}
```
它接受一个 Instrument 的引用，编译器是如何知道这个 Instrument 引用指向的是我们传递的 Wind 对象呢？而不是别的导出类。实际上编译器无法知道。我们要深入理解这个问题。

#### 方法调用绑定

将一个方法调用和一个方法主体关联起来被称作绑定。若在程序执行前进行绑定，叫做前期绑定。既然是前期绑定，当编译器只有一个 Instrument 引用时，它无法知道应该调用那个方法。解决的办法是后期绑定。运行时根据对象的类型进行绑定。后期绑定也叫做动态绑定和运行时绑定。如果想实现后期绑定就必须实现某种机制，以便在运行时能判断对象的类型，从而调用恰当的方法。

java 中除了 static 方法和 final 方法之外，其他所有的方法都是后期绑定。这就意味着我们不必去判断是否应该进行后期绑定。为什么要将方法声明为 final 呢？这样做可以防止别人覆盖这个方法。更重要的是可以关闭后期绑定。或者说告诉编译器不需要对其进行动态绑定。其实这样做不会对程序的性能有什么影响。完全是根据你程序的设计决定。

#### 产生正确的行为

Java 中所有的方法都是通过动态绑定实现多态这个事实的。我们基于动态绑定的特性可以写出基于基类打交道的代码了，并且这些代码对于所有的子类都是可以正确运行的。

![](/images/blog/up.png)

Shape 基类为自他哪里继承而来的所有导出类建立一个公用的接口，也就是说所有形状都可以描述和擦除。

#### 可扩展性

我们继续看之前的乐器例子。由于有了多态的机制，我们可以为系统任意添加自己需要的新的类型，而不需要修改 tune() 方法。在一个良好的 OOP 程序中大多数或者所有的方法都会遵守 tune() 的模型。只与基类接口通讯。这样的程序是可扩展的，而且那些操作基类接口的方法并不需要任何的改变。

![](/images/blog/upxulie.png)

基类：
```java
public class Instrument {

	public void  play(Note note) {
		System.out.println("唱歌");
	}

	String what(){
		return "Instrument";
	}

	void adjust(){

	}
}
```
继承类一：
```java
public class Wind extends Instrument{

	@Override
	public void play(Note note) {
		// TODO Auto-generated method stub
		System.out.println("Window.paly"+note);
	}

	@Override
	String what() {
		// TODO Auto-generated method stub
		return "Wind";
	}

	@Override
	void adjust() {
		// TODO Auto-generated method stub

	}
}
```
继承类二：
```java
public class Percussion extends Instrument{

	@Override
	public void play(Note note) {
		// TODO Auto-generated method stub
		System.out.println("Percussion.paly"+note);
	}

	@Override
	String what() {
		// TODO Auto-generated method stub
		return "Percussion";
	}

	@Override
	void adjust() {
		// TODO Auto-generated method stub

	}
}
```
继承类三：
```java
public class Stringed extends Instrument{

	@Override
	public void play(Note note) {
		// TODO Auto-generated method stub
		System.out.println("Stringed.paly"+note);
	}

	@Override
	String what() {
		// TODO Auto-generated method stub
		return "Stringed";
	}

	@Override
	void adjust() {
		// TODO Auto-generated method stub

	}
}

```
二级继承类：
```java
public class Woodwind extends Wind{
    @Override
    public void play(Note note) {
    	// TODO Auto-generated method stub
    	System.out.println("Woodwind.paly"+note);
    }

    @Override
    String what() {
    	// TODO Auto-generated method stub
    	return "Woodwind";
    }
}

public class Brass extends Wind{

	@Override
	public void play(Note note) {
		// TODO Auto-generated method stub
		System.out.println("Brass.paly"+note);
	}

	@Override
	String what() {
		// TODO Auto-generated method stub
		return "Brass";
	}

	@Override
	void adjust() {
		// TODO Auto-generated method stub

	}
}

```
调用并执行：
```java
public class UpOverloadTest {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Instrument[] instruments = {new Wind(),new Percussion(),new Stringed(),new Woodwind(),new Brass()};

		tune(instruments);
	}

	public static void  tune(Instrument instrument) {
		instrument.play(Note.MID);
	}

	private static void  tune(Instrument[] instruments) {
		for (Instrument instrument : instruments) {
			tune(instrument);
		}
	}


}

```
执行结果：
```
Window.palyMID
Percussion.palyMID
Stringed.palyMID
Woodwind.palyMID
Brass.palyMID
```
我们在基类和新添加的几个子类中添加了新的方法。在 main() 方法中我们使用了向上转型。可以看到 tune() 方法完全可以忽略他周围代码的变化，依旧可以正常的运行。这正是我们所期望的多态所具有的的特性。多态是程序员将改变的事物和未改变的事物分离开来的重要技术。

#### 缺陷：不能覆盖私有方法

在之前的权限控制中我们就学到了，由于 private 方法被自动认为是 final 方法，而且对所有的子类屏蔽。因此在子类中是无法对父类的私有方法进行复用的。如果你的方法名和父类中的私有方法是一样的，那么等于你是添加了一个新的方法而不是重载。但是这样会容易引起歧义，所以我们建议最好新添加的方法名不要和父类重复。

#### 缺陷：域与静态方法

多态只可以发生在普通的方法中，不能发生在静态的方法中，也不能发生于基本数据类型的变量。

```java
public class Super {

	public int field = 0;

	public void getS(){
		System.out.println("超级类的非静态方法");
	}

	public static void getSu(){
		System.out.println("超级类的静态方法");
	}

}

```
子类：
```java
public class Sub extends Super{

   public int field = 1;


   @Override
	public void getS() {
		// TODO Auto-generated method stub
	   System.out.println("子类的非静态方法");
	}

    public static void getSu() {
	// TODO Auto-generated method stub
       System.out.println("子类的静态方法");
    }
}
```
调用并执行
```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		Super super1 = new Sub();
		super1.getSu();

		System.out.println("变量值："+super1.field);

	}
```
执行结果
```
超级类的静态方法
变量值：0
```
我们看到输出的结果是父类中的值，这是因为域也就是变量都是由编译器解析的，因此不是多态。当创建变量 field 的时候其实是分配了不同的存储空间。这样其实包含了两个 field。
如果某个方法是静态的，它的行为也不是多态。静态方法是与类相关的，不是和某个对象相关联。所以静态方法也不具备多态性。

## 构造器与多态

构造器比较特殊，它不同于其他种类的方法。构造器也不具备多态性，因为构造器是隐式的指定 static 的属性。

#### 构造器的调用顺序

基类的构造器总是在子类的构造过程中被调用，而且按照继承层次逐渐向上链接，以使得每个类的构造器都能被调用。为什么会这样呢？因为构造器的目的是为了检查对象是否存在。子类并不能访问基类的全部成员。只有基类的构造器才具有恰当的知识和权限来对自己的元素进行初始化。因此必须使得所有的构造器都得到调用，否则就不可能正确的调用完整的数据对象。在子类的构造过程中如果没有明确的指出调用基类的那个构造器，就默认的调用无参的构造器。如果不存在默认构造器编译器就会报错。

他们的构建顺序是这样的：

- 调用基类构造器。这个步骤会不断的递归下去，直到最外层的子类。
- 按照声明顺序调用成员的初始化方法。
- 调用子类的构造器主题。

#### 继承与清理

通过组合和继承方法来创建新类的时候，永远不必担心对象的清理问题，子对象通常都会留给垃圾回收器进行清理。如果确实遇到清理问题，那么必须用心为新类创建清理的方法。清理动作的顺序刚好和构造器的调用顺序是相反的。在垃圾回收与清理章节我们已经举过例子。先清理子对象然后在清理父类对象。这是因为某个子对象要依赖于其他的对象，销毁的顺序应该和初始化顺序相反。这是因为子类的清理可能会调用基类中的某些方法，所以这时我们希望基类的构建是仍然起作用的，而不应该过早的去销毁它。

有一个特殊的情况，如果你的成员对象存在于其他一个或者多个对象共享时，问题就会变得复杂起来。你就不能简单的去调用销毁方法了。在这种情况下我们就必须引用计数来跟踪仍旧访问着共享对象的对象数量。

```java
public class Shared {
	private int refcound = 0;
	private static long counter = 0;
	private final long id = counter++;
	public Shared() {
		// TODO Auto-generated constructor stub
		System.out.println("Create" + this);
	}
	public void addRef() {
		refcound ++;
	}

	protected void dispose() {
		if (--refcound ==0) {
			System.out.println("Dispose");
		}
	}
	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return "Shared "+id;
	}

}

```
调用这个对象
```java
public class Composing {
	private Shared shared;
	private static long counter =0;
	private final long id = counter++;

	public Composing(Shared shared) {
		// TODO Auto-generated constructor stub
		System.out.println("Create" + this);
		this.shared = shared;
		shared.addRef();
	}

	protected void dispose() {
		System.out.println("dispose" + this);
		shared.dispose();
	}

	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return "Composing "+id;
	}

}

```
执行
```java
public class ReferCountTest {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Shared shared = new Shared();

		Composing[] composings = new Composing[5];
		for (int i = 0; i < composings.length; i++) {
			composings[i] = new Composing(shared);
		}

		for (Composing composing : composings) {
			composing.dispose();
		}
	}

}

```
执行的结果
```
CreateShared 0
CreateComposing 0
CreateComposing 1
CreateComposing 2
CreateComposing 3
CreateComposing 4
disposeComposing 0
disposeComposing 1
disposeComposing 2
disposeComposing 3
disposeComposing 4
Dispose
```
#### 构造器内部多态方法的行为

如果我们在构造器的内部调用正在构造的对象的某个动态绑定的方法，将会发生什么情况呢?

父类：
```java

public class Glyph {

	void draw(){

	}

	public Glyph() {
		// TODO Auto-generated constructor stub
		System.out.println("调用基类方法之前");
		draw();
		System.out.println("调用基类方法之后");
	}

}
```
子类：
```java
public class RoundGlyph extends Glyph{
	private int radius = 2;

	public RoundGlyph(int r) {
		// TODO Auto-generated constructor stub
		radius = r;

		System.out.println("调用子类的构造方法"+radius);
	}

	void draw() {
		System.out.println("子类的draw方法被调用"+radius);
	};

}
```
调用：
```java
public class PloyGlayTest {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		new RoundGlyph(5);
	}

}

```
执行结果：
```
调用基类方法之前
子类的draw方法被调用0
调用基类方法之后
调用子类的构造方法5
```
我们看到当子类的构造方法创建时，先去初始化父类的构造方法，这时父类的构造方法去调用绘画 draw() 方法。根据多态的特性实际调用的是子类的绘画方法。但是我们看到 radius
的结果是 0。默认值明明是 2，为什么会是 0 呢？

再次来说明一下初始化的顺序问题。

- 在其他任何事物发生之前，将分配给对象的存储空间初始化为二进制的 0。
- 如前所述调用基类的构造器。此时调用被覆盖的 draw() 方法，由于是在构造器之前调用。所以此时 radius 的值被初始化为 0。
- 按照声明的顺序调用成员的初始化。
- 调用子类的构造器主体。

基于以上的内容，我们在编写构造器的时候应当遵循以下原则:用尽可能简单的方法使对象进入正常状态，如果可以的话避免调用其他方法。在构造器内唯一能够安全调用的那些方法是基类中的 final 方法。这些方法不能被覆盖，因此也就不会出现上述惊讶的错误。

## 协变返回类型

javaSE5 中加了可斜边返回参数，表示子类覆盖父类的方法时，这个方法的返回值既可以是基类类型的任何对象。

基类：
```java
public class Gran {
	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return "Gran";
	}

}
```
子类：
```java
public class Wheat extends Gran{

	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return "Wheat";
	}
}
```
基类：
```java
public class Mill {

	Gran pGran(){
		return new Gran();
	}
}
```
注意：可斜边返回参数
```java
public class WheatMill extends Mill{

	@Override
	Wheat pGran() {
		// TODO Auto-generated method stub
		return new Wheat();
	}
}

```
注意这里我们覆盖了父类中的 pGran() 方法，父类中的返回值是 Gran。而我们子类中的返回值是 Wheat 是 Gran 的子类。这在 javaSE5 之前是不允许的。

## 用继承进行设计

似乎所有的东西实现多态都可以用继承来实现，如果我们有时候创建新的类时首先考虑继承技术反倒会加重我们的设计负担。更好的选择应该是首选 “组合” 技术。组合不会强制我们进入继承的层级结构。而且更加的灵活。因为它可以动态的选择类型；相反，继承在编译期就必须知道确切的类型。

我们将上边的代码增加一个子类：
```java
public class Happ extends Gran{
	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return "Happ";
	}
}
```
创建一个新的类：
```java
public class Test {
	private static Gran gran = new Wheat();

	static void change(){
		gran = new Happ();
	}

	public static void main(String[] args) {
		System.out.println(gran);
		change();
		System.out.println(gran);

	}
}
```
执行结果：
```java
Wheat
Happ
```
引用可以在运行时与另外一个不同的对象重新进行绑定。这样一来我们在运行期间就获得很大的灵活性。这也称之为状态模式。与此相反，我们不能再运行期间决定继承不同的对象。

一条通用的准则是：用继承表达行为间的差异，并用字段表示状态上的变化。

#### 向下转型与运行时类型识别

由于向上转型会丢失具体的类型信息，所以通过向下转型应该可以获得具体的类型信息。然而我们知道向上转型是类型安全的，因为基类不会大于子类的接口。但是向下转型我们就不清楚了。要解决这个问题必须有某种方法来确保向下转型的正确性。在 Java 中所有的转型都会得到检查。所以，即使我们进行一次普通的加括号类型转换，在进入运行时仍然是会对其进行检查的，以便保证确实是我们需要的类型。这种在运行期间进行检查的行为称为：「运行时类型识别 (RTTI)」。

基类：
```java
public class Userful {

	void f(){

	}

	void g(){

	}
}
```
子类：
```java
public class MoreUserful extends Userful{

	@Override
	void f() {
		// TODO Auto-generated method stub
		super.f();
	}

	@Override
	void g() {
		// TODO Auto-generated method stub
		super.g();
	}

	void h(){

	}

}

```
调用并执行：
```java
public class Test {

	public static void main(String[] args) {
		Userful[] xUserfuls = {new Userful(),new MoreUserful()};
		xUserfuls[0].f();
		xUserfuls[1].g();
		//xUserfuls[1].h(); The method h() is undefined for the type Userful
		((MoreUserful) xUserfuls[1]).h();
	}
}

```
我们在视图用 xUserfuls[1].h() 时就会看到一条编译时的错误。如果我们尝试去调用扩展的接口，就可以尝试向下转型。如果所转类型是正确的，那么转型成功；否则就会出异常。

![](/images/blog/classcast.png)

## 总结

多态意味着 “不同的形式” 。在面向对象程序设计中，我们持有从基类继承而来的相同接口，以便使用该接口的不同形式：不同版本的动态绑定方法。
