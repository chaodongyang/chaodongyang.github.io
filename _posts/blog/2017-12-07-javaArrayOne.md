---
layout: post
title: java编程思想之数组详解
categories: javaThinking
description: java编程思想之数组
keywords: java, java数组, 数组，java数组详解
---

对数组的基本看法，你可以创建并组装他们，通过使用整型的索引值来访问他们的元素，并且他们的尺寸不能改变。

## 数组为什么特殊
Java 总有大量的方式可以持有对象，那么数组何以与众不同呢？数组与其他容器的区别主要有三方面：效率、类型、和保存基本类型的能力。

在 Java 中数组是一种效率最高的存储和随机访问对象引用序列的方式。数组就是一个简单的线性序列，这使得元素访问速度非常快。但是这种速度付出的代价就是数组的大小在其生命周期内是不可改变的。ArrayList 容器可以通过创建一个新实例，然后把所有旧实例的引用移动到新的实例中，从而实现更多空间的自动分配，但是这种弹性是需要开销的。因此 ArrayList 的效率比数组要低很多。

数组和容器都可以保证你不能滥用他们。无论是数组还是容器如果你越界都会产生异常。在泛型之前，其他的容器类在处理对象时，都将他们视作没有任何的具体类型。也就是说他们将这些对象当做 Java 中的根类 Object 处理。数组优于泛型之前的容器是因为数组而已持有某种具体的类型。这意味着可以通过编译期检查问题，以防止插入错误的类型和抽取不当的类型。

数组可以持有基本的类型，而泛型之前的容器则不能。但是有了泛型，容器也可以指定和检查他们所持有的类型，并且有额自动包装机制容器看起来也能持有基本类型。

将数组和容器比较：
```java
public class Beryll {
	private static long counter;
	private final long id = counter++;
	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return "编号："+id;
	}
}
```
测试类：
```java
public class ContainerComparson {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Beryll[] sBerylls = new Beryll[10];
		for (int i = 0; i <5; i++) {
			sBerylls[i] = new Beryll();
		}

		System.out.println(Arrays.toString(sBerylls));
		System.out.println(sBerylls[4]);

		List<Beryll> berylls = new ArrayList<>();
		for (int i = 0; i < 5; i++) {
			berylls.add(new Beryll());
		}
		System.out.println(berylls);
		System.out.println(berylls.get(4));

		int[] integers = { 0, 1, 2, 3, 4, 5 };
		System.out.println(Arrays.toString(integers));
		System.out.println(integers[4]);

		List<Integer> intList = new ArrayList<Integer>(Arrays.asList(0, 1, 2, 3, 4, 5));
		intList.add(97);
		System.out.println(intList);
		System.out.println(intList.get(4));
	}

}

```
执行结果：
```
[编号：0, 编号：1, 编号：2, 编号：3, 编号：4, null, null, null, null, null]
编号：4
[编号：5, 编号：6, 编号：7, 编号：8, 编号：9]
编号：9
[0, 1, 2, 3, 4, 5]
4
[0, 1, 2, 3, 4, 5, 97]
4
```
这两种方式都可以进行类型检查，明显的差异就是访问元素的方式不一样。数组用的是 ```[]``` 这种方式。而容器是使用 ```get``` 和 ```add``` 。但是容器比数组具有更多的功能。我们看到利用自动包装机制，容器也可以用于基本类型。数组仅存的优点就剩下效率了。

## 数组是第一级对象
无论哪种数组对象，数组标识符其实是一种引用，指向堆中的一个真实对象，数组对象就是用来保存指向其他对象的引用。

下面总结了数组的初始化方式以及如何对指向数组的引用赋值，使得指向另一个数组对象。对象数组和基本类型数组的区别是，对象数组保存的是引用，基本类型的数组保存的是基本类型的值。
```java
public class ArrayOptions {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		    BerylliumSphere[] a; // Local uninitialized variable
		    BerylliumSphere[] b = new BerylliumSphere[5];

		    System.out.println("b: " + Arrays.toString(b));
		    BerylliumSphere[] c = new BerylliumSphere[4];
		    for(int i = 0; i < c.length; i++)
		      if(c[i] == null) // Can test for null reference
		        c[i] = new BerylliumSphere();

		    // Aggregate initialization:
		    BerylliumSphere[] d = { new BerylliumSphere(), new BerylliumSphere(), new BerylliumSphere()};
		    // Dynamic aggregate initialization:
		    a = new BerylliumSphere[]{new BerylliumSphere(), new BerylliumSphere(),};
		    // (Trailing comma is optional in both cases)
		    System.out.println("a.length = " + a.length);
		    System.out.println("b.length = " + b.length);
		    System.out.println("c.length = " + c.length);
		    System.out.println("d.length = " + d.length);
		    a = d;
		    System.out.println("a.length = " + a.length);

		    // Arrays of primitives:
		    int[] e; // Null reference
		    int[] f = new int[5];

		    // automatically initialized to zero:
		    System.out.println("f: " + Arrays.toString(f));
		    int[] g = new int[4];
		    for(int i = 0; i < g.length; i++)
		      g[i] = i*i;
		    int[] h = { 11, 47, 93 };

		    //!print("e.length = " + e.length);
		    System.out.println("f.length = " + f.length);
		    System.out.println("g.length = " + g.length);
		    System.out.println("h.length = " + h.length);
		    e = h;
		    System.out.println("e.length = " + e.length);
		    e = new int[]{ 1, 2 };
		    System.out.println("e.length = " + e.length);
	}

}
```
执行结果：
```
b: [null, null, null, null, null]
a.length = 2
b.length = 5
c.length = 4
d.length = 3
a.length = 3
f: [0, 0, 0, 0, 0]
f.length = 5
g.length = 4
h.length = 3
e.length = 3
e.length = 2

```
数组 a 是一个未被初始化的局部变量，在你正确初始化之前编译器不允许用次引用做任何事情。length 关键字表示数组的大小，而不是实际保存的元素，所以你无法知道数组中确切地有多少元素。新生成一个数组对象时，其中所有的引用都会被自动初始化为 null。同样，基本类型的数组如果是数值型的，就被自动初始化为 0；如果是字符型，就被初始化为 (char)0；如果是 boolean，就被自动初始化为 false。

动态的去创建数组：
```java
BerylliumSphere[] d = { new BerylliumSphere(), new BerylliumSphere(), new BerylliumSphere()};
```
表达式：说明指向某个数组的引用被赋值给另外一个数组。
```java
a = d;
```
## 返回一个数组
加入你要写一个方法返回一个数组，而不是一个值。对于 C/C++ 而言是有点困难的，因为他们不能返回一个数组，只能返回指向数组的指针。这回造成生命周期控制起来变动困难，容易造成内存泄漏。在 Java 中可以直接返回一个数组，当你使用完之后，垃圾回收器会清理掉他。

下面演示如何返回一个 String 类型的数组：
```java
public class IceCream {

	private static Random random = new Random(47);

	static final String[] FLAVORS = {
		    "Chocolate", "Strawberry", "Vanilla Fudge Swirl",
		    "Mint Chip", "Mocha Almond Fudge", "Rum Raisin",
		    "Praline Cream", "Mud Pie"
		  };

	public static String[] flavor(int n) {
		if (n> FLAVORS.length) {
			throw new IllegalArgumentException("超过了数组最大值");
		}

		String[] strings = new String[n];
		boolean[] bs = new boolean[FLAVORS.length];
		for (int i = 0; i <n; i++) {
			int t;
			do {
				t = random.nextInt(FLAVORS.length);
			} while (bs[t]);
			strings[i] = FLAVORS[t];
			bs[t] = true;
		}
		return strings;
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		for (int i = 0; i <2; i++) {
			System.out.println(Arrays.toString(flavor(3)));
		}
	}

}
```
执行结果：
```
[Rum Raisin, Mint Chip, Mocha Almond Fudge]
[Chocolate, Strawberry, Mocha Almond Fudge]
```

## 多维数组
创建多维数组很方便。对于基本类型的多维数组，可以通过使用花括号将每个向量分隔开：
```java
public class Multidimens {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		int[][] a = {{1,2,3},{4,5,6}};
		//Arrays.deepToString(a)可以把多个数组转换为多个String
		System.out.println(Arrays.deepToString(a));
	}

}

```
每个花括号括起来的集合都会把你带到下一个集合。

还可以使用 new 来分配数组，下面看一个三维数组的例子：
```java
public class ThreeD {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		int[][][] a = new int[2][2][3];
		System.out.println(Arrays.deepToString(a));
	}

}
```
执行结果：
```
[[[0, 0, 0], [0, 0, 0]], [[0, 0, 0], [0, 0, 0]]]

```
数组中构成矩阵的每个向量都可以具有任意长度。这被称为粗糙数组：
```java
public class RaggedArray {
	public static void main(String[] args) {
		Random rand = new Random(47);
	    // 3-D array with varied-length vectors:
	    int[][][] a = new int[3][][];

	    for(int i = 0; i < a.length; i++) {
	      a[i] = new int[rand.nextInt(5)][];
	      for(int j = 0; j < a[i].length; j++)
	        a[i][j] = new int[rand.nextInt(5)];
	    }
	    System.out.println(Arrays.deepToString(a));
	}

}
```
执行结果：
```
[[[], [0, 0, 0], [0]], [[0, 0, 0, 0]], [[], [0, 0], [0, 0]]]
```
第一个 new 创建的数组是 3 个，第二个 new 的长度是随机定义的；直到碰到第三个长度才得以确定。

可以用类似的方式处理非基本类型的对象数组。
```java
public class MultidimensionalObjectArrays {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		 BerylliumSphere[][] spheres = {
			      { new BerylliumSphere(), new BerylliumSphere() },
			      { new BerylliumSphere(), new BerylliumSphere(),new BerylliumSphere(), new BerylliumSphere() },
			      { new BerylliumSphere(), new BerylliumSphere(),
			        new BerylliumSphere(), new BerylliumSphere(),
			        new BerylliumSphere(), new BerylliumSphere(),
			        new BerylliumSphere(), new BerylliumSphere() },
			    };
			    System.out.println(Arrays.deepToString(spheres));
	}

}

```
执行结果：
```
[[编号：0, 编号：1], [编号：2, 编号：3, 编号：4, 编号：5], [编号：6, 编号：7, 编号：8, 编号：9, 编号：10, 编号：11, 编号：12, 编号：13]]
```
## 数组与泛型
通常，数组与泛型不能很好的结合，因为你不能实例化具有参数类型的数组：
```java
Peel<Banner>[] peels = new Peel<Banner>[10];
```
擦除机制会移除参数类型信息，而数组必须知道他们所持有的确切类型，以强制保证类型安全。但是我们可以参数化数组本身的类型：
```java
public class ParameterizedArrayType {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		 Integer[] ints = { 1, 2, 3, 4, 5 };
		 Double[] doubles = { 1.1, 2.2, 3.3, 4.4, 5.5 };
		 Integer[] ints2 = new ClassParameter<Integer>().f(ints);
		 Double[] doubles2 = new ClassParameter<Double>().f(doubles);
		 ints2 = MethodParameter.f(ints);
		 doubles2 = MethodParameter.f(doubles);
	}

}

class MethodParameter {
	  public static <T> T[] f(T[] arg) {
		  return arg;
	  }
}

class ClassParameter<T> {
	  public T[] f(T[] arg) {
		  return arg;
	  }
}
```
注意：使用参数化方法比参数化类方便之处在于：你不必为需要的每种不同类型都使用一个参数去实例化它，并且你可以将其定义为静态的。

编译器不允许创建泛型数组，但是允许你创建对这种数组的引用：
```java
public class ArrayOfGenerics {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		    List<String>[] ls;
		    List[] la = new List[10];
		    ls = (List<String>[])la; // "Unchecked" warning
		    ls[0] = new ArrayList<String>();
		    //一旦确定了类型List<String>就不能在赋予其他的类型
		    //! ls[1] = new ArrayList<Integer>();

		    // The problem: List<String> is a subtype of Object
		    Object[] objects = ls; // So assignment is OK
		    // Compiles and runs without complaint:
		    objects[1] = new ArrayList<Integer>();

		    // However, if your needs are straightforward it is
		    // possible to create an array of generics, albeit
		    // with an "unchecked" warning:
		    List<BerylliumSphere>[] spheres = (List<BerylliumSphere>[])new List[10];
		    for(int i = 0; i < spheres.length; i++)
		      spheres[i] = new ArrayList<BerylliumSphere>();
	}

}

```
一旦拥有了对 List<String>[] 的引用，就会拥有编译期的检查。

一般而言，泛型在类或方法边界处很有效，而在类或者方法内部，擦除通常会使得泛型变得不适用。例如，我们不能创建泛型数组：
```java
public class ArrayOfGenericType<T> {
	  T[] array; // OK
	  @SuppressWarnings("unchecked")
	  public ArrayOfGenericType(int size) {
		  //不能创建泛型数组因为T会被擦除，因此不能确定类型
	    //! array = new T[size]; // Illegal
		  //但是却可以创建一个引用进行转型
	    array = (T[])new Object[size]; // "unchecked" Warning
	  }
	  // Illegal: 同样在方法内部也不适用
	  public <U> U[] makeArray() {
		  return new U[10];
	  }
}
```
## 创建测试数据
我们通过实验数组和程序，能够很方便的填充测试数据的数组。

#### Arrays.fill()
这个方法的作用：只能用同一个值填充各个位置，而针对对象而言，就是复制同一个引用进行填充。
```java
public class FillingArrays {
	public static void main(String[] args) {
		    int size = 6;
		    boolean[] a1 = new boolean[size];
		    byte[] a2 = new byte[size];
		    char[] a3 = new char[size];
		    short[] a4 = new short[size];
		    int[] a5 = new int[size];
		    long[] a6 = new long[size];
		    float[] a7 = new float[size];
		    double[] a8 = new double[size];
		    String[] a9 = new String[size];
		    Arrays.fill(a1, true);
		    System.out.println("a1 = " + Arrays.toString(a1));
		    Arrays.fill(a2, (byte)11);
		    System.out.println("a2 = " + Arrays.toString(a2));
		    Arrays.fill(a3, 'x');
		    System.out.println("a3 = " + Arrays.toString(a3));
		    Arrays.fill(a4, (short)17);
		    System.out.println("a4 = " + Arrays.toString(a4));
		    Arrays.fill(a5, 19);
		    System.out.println("a5 = " + Arrays.toString(a5));
		    Arrays.fill(a6, 23);
		    System.out.println("a6 = " + Arrays.toString(a6));
		    Arrays.fill(a7, 29);
		    System.out.println("a7 = " + Arrays.toString(a7));
		    Arrays.fill(a8, 47);
		    System.out.println("a8 = " + Arrays.toString(a8));
		    Arrays.fill(a9, "Hello");
		    System.out.println("a9 = " + Arrays.toString(a9));
		    // Manipulating ranges:
		    Arrays.fill(a9, 3, 5, "World");
		    System.out.println("a9 = " + Arrays.toString(a9));
	}
}

```
执行结果：
```
a1 = [true, true, true, true, true, true]
a2 = [11, 11, 11, 11, 11, 11]
a3 = [x, x, x, x, x, x]
a4 = [17, 17, 17, 17, 17, 17]
a5 = [19, 19, 19, 19, 19, 19]
a6 = [23, 23, 23, 23, 23, 23]
a7 = [29.0, 29.0, 29.0, 29.0, 29.0, 29.0]
a8 = [47.0, 47.0, 47.0, 47.0, 47.0, 47.0]
a9 = [Hello, Hello, Hello, Hello, Hello, Hello]
a9 = [Hello, Hello, Hello, World, World, Hello]
```
使用 Arrays.fill() 可以填充整个数组，也可以像最后两天哪样，填充数组的某个区域。但是只能填充单一的数值。

##  Arrays 实用功能
在Java.util 类库中可以找到 Arrays 类，他有一套属于自己的静态方法，其中有 6 个基本方法：
 - equals() 用于比较两个数组是否相等。(deepEquals() 用于多维数组)
 - fill() 用于填充数组
 - sort() 用于对数组排序
 - binarySearch() 用于在已经排序的数组中查找元素
 - toString() 产生数组的 String 表示
 - hashCode() 产生数组的散列码

这些方法都对各种基本类型和 Object 类重载过。此外，Arrays.asList() 接受任意的序列或者数组，将其转变为 List 容器。

#### 复制数组
这个方法不属于 Arrays 但是非常的好用。Java 标准类库提供有静态的方法 System.arraycopy()，用它复制数组比用 for 循环复制要快的多。System.arraycopy() 针对所有类型做了重载。
```java
public class CopyingArrays {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		int[] i = new int[7];
	    int[] j = new int[10];
	    Arrays.fill(i, 47);
	    Arrays.fill(j, 99);
	    System.out.println("i = " + Arrays.toString(i));
	    System.out.println("j = " + Arrays.toString(j));
	    System.arraycopy(i, 0, j, 0, i.length);
	    System.out.println("j = " + Arrays.toString(j));
	    int[] k = new int[5];
	    Arrays.fill(k, 103);
	    System.arraycopy(i, 0, k, 0, k.length);
	    System.out.println("k = " + Arrays.toString(k));
	    Arrays.fill(k, 103);
	    System.arraycopy(k, 0, i, 0, k.length);
	    System.out.println("i = " + Arrays.toString(i));
	    // Objects:
	    Integer[] u = new Integer[10];
	    Integer[] v = new Integer[5];
	    Arrays.fill(u, new Integer(47));
	    Arrays.fill(v, new Integer(99));
	    System.out.println("u = " + Arrays.toString(u));
	    System.out.println("v = " + Arrays.toString(v));
	    System.arraycopy(v, 0, u, u.length/2, v.length);
	    System.out.println("u = " + Arrays.toString(u));
	}

}

```
这个例子说明了基本类型数组和对象数组都可以复制。但是复制对象数组只是复制了引用，而不是对象本身的拷贝。这被称为浅复制。

看一下这个方法的解释含义：
```java

   /* @param      src      源数组
		* @param      srcPos   开始复制源数组的索引位置
		* @param      dest     目标数组
		* @param      destPos  目标数组开始复制的索引位置
		* @param      length   复制的长度
		* @exception  IndexOutOfBoundsException  if copying would cause
		*               access of data outside array bounds.
		* @exception  ArrayStoreException  if an element in the <code>src</code>
		*               array could not be stored into the <code>dest</code> array
		*               because of a type mismatch.
		* @exception  NullPointerException if either <code>src</code> or
		*               <code>dest</code> is <code>null</code>.
		*/
	 public static native void arraycopy(Object src,  int  srcPos,
								 Object dest, int destPos,
								 int length);

```

#### 数组的比较
Arrays 类提供了重载后的 equals() 方法，用来比较整个数组。同样对于基本类型和 Object 类型都做了重载。数组相等的条件是，数组长度相等，对应位置的元素也相等。
```java
public class ComparingArrays {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		int[] a1 = new int[10];
	    int[] a2 = new int[10];
	    Arrays.fill(a1, 47);
	    Arrays.fill(a2, 47);
	    System.out.println(Arrays.equals(a1, a2));
	    a2[3] = 11;
	    System.out.println(Arrays.equals(a1, a2));
	    String[] s1 = new String[4];
	    Arrays.fill(s1, "Hi");
	    String[] s2 = { new String("Hi"), new String("Hi"),new String("Hi"), new String("Hi") };
	    System.out.println(Arrays.equals(s1, s2));
	}

}
```
执行结果：
```
true
false
true
```
#### 数组元素的比较
排序必须根据对象的实际类型执行比较操作。一种自然的解决方案是为每种不同的类型各编写一种不同的排序方法，但是这样的代码很难以被复用。

程序设计的目标是 “将保持不变的事物与会发生改变的事物相分离”，这里不变的是通用的算法，变化的是各种对象相互比较的方式。使用策略模式可以将策略对象传递给总是相同的代码，这些代码将使用其策略完成算法。

Java 有两种方式来提供比较功能。第一种实现 Comparable 接口，使得你的类具有天生的比较能力。此接口只有一个方法 compareTo() 方法。接收一个 Object 为参数，如果当天对象小于参数则返回负值，如果相等返回零，如果大于则返回正值。
```java
public class CompType implements Comparable<CompType>{
	int i;
	int j;
	private static int count =1;



	protected CompType(int i, int j) {
		super();
		this.i = i;
		this.j = j;
	}

	@Override
	public String toString() {
		// TODO Auto-generated method stub
		String result = "i=:"+i+"j=:"+j;
		if (count++ % 3 ==0) {
			result += "\n";
		}
		return result;
	}

	@Override
	public int compareTo(CompType o) {
		// TODO 我们进行倒序排序
		return (i < o.i ? 1 :(i == o.i ? 0 :-1));
	}


	public static void main(String[] args) {
		CompType[] compTypes = new CompType[]{new CompType(58, 55),new CompType(93, 61),new CompType(61, 29)};
		System.out.println(Arrays.toString(compTypes));
		Arrays.sort(compTypes);
		System.out.println(Arrays.toString(compTypes));
	}

}

```
执行结果：
```
[i=:58j=:55, i=:93j=:61, i=:61j=:29]
[i=:93j=:61, i=:61j=:29, i=:58j=:55]
```
在定义比较方法的时候我们自己来负责一个对象与另一个对象比较的含义。我们在上边的比较重只比较了一个值。如果我们没有实现 Comparable 接口，在调用 sort() 的时候会抛出异常 ClassCastException 这个运行时异常。因为 sort() 方法需要将参数类型转变为 Comparable。

Collections 类包含一个 reverseOrder() 方法，该方法产生一个 Comparator，他可以翻转排序的顺序。我们看这个示例：
```java
public static void main(String[] args) {
		CompType[] compTypes = new CompType[]{new CompType(58, 55),new CompType(93, 61),new CompType(61, 29)};
		System.out.println(Arrays.toString(compTypes));
		Arrays.sort(compTypes,Collections.reverseOrder());
		System.out.println(Arrays.toString(compTypes));
	}
```
执行结果：
```
[i=:58j=:55, i=:93j=:61, i=:61j=:29]
[i=:58j=:55, i=:61j=:29, i=:93j=:61]
```
也可以编写自己的 Comparator 接口。在这个示例中我们将基于第二个参数比较：
```java
public class ComTypeComparator implements Comparator<CompType>{

	@Override
	public int compare(CompType o1, CompType o2) {
		// TODO Auto-generated method stub
		return (o1.j < o2.j ? -1 :(o1.j == o2.j ? 0 :1));
	}

}
```
测试代码：
```java
public static void main(String[] args) {
		CompType[] compTypes = new CompType[]{new CompType(58, 55),new CompType(93, 61),new CompType(61, 29)};
		System.out.println(Arrays.toString(compTypes));
		Arrays.sort(compTypes,new ComTypeComparator());
		System.out.println(Arrays.toString(compTypes));
	}
```
执行结果：按照 ```J``` 进行排序
```
[i=:58j=:55, i=:93j=:61, i=:61j=:29]
[i=:61j=:29, i=:58j=:55, i=:93j=:61]
```
#### 数组排序
使用内置的排序方法，就可以对任意的基本类型数组排序；也可以对任意的对象数组排序，只要改对象实现了 Comparable 接口或者相关联的 Comparator。

对 String 对象排序：
```java
public class StringSorting {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		String[] strings = {"wo","shi","zhong","guo","ren","HAHA"};
		System.out.println("排序前："+Arrays.toString(strings));
		Arrays.sort(strings);
		System.out.println("排序后："+Arrays.toString(strings));
		Arrays.sort(strings,String.CASE_INSENSITIVE_ORDER);
		System.out.println("排序后1："+Arrays.toString(strings));

	}

}

```
执行结果：
```
排序前：[wo, shi, zhong, guo, ren, HAHA]
排序后：[HAHA, guo, ren, shi, wo, zhong]
排序后1：[guo, HAHA, ren, shi, wo, zhong]
```
默认的 String 排序按照词典编排顺序排序，所以大写字母开头的词在前面输出，然后才是小写字母开头的词。如果要忽略大小写只按照单词的顺序排列，就像上面那种加入常量。java 标准类库中的排序算法针对正排序的特殊类进行了优化；针对基本类型进行的 “快速排序”，针对对象设计的 “稳定归排序”。所以无需担心排序的性能问题。

#### 在已排序的数组中查找
如果数组已经排序好了，就可以使用 Arrays.binarySearch() 执行快速查找。如果对未排序的数组使用 binarySearch()，那么将产生不可预料的后果。
```java
public class ArraySearching {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		int[] a = {23,1,45,2,5,77,45,30,40,10};
		//先排序
		Arrays.sort(a);
		System.out.println(Arrays.toString(a));
		int location = Arrays.binarySearch(a, 10);
		System.out.println("找到的位置："+location);
		int locationS = Arrays.binarySearch(a, 3);
		System.out.println("应该插入的位置："+locationS);

	}

}

```
执行结果：
```
[1, 2, 5, 10, 23, 30, 40, 45, 45, 77]
找到的位置：3
应该插入的位置：-3

```
如果找到目标奖返回一个大于或等于 0 的值。否则，它将返回负值，表示要保持数组的排序状态次目标元素应该插入的位置。插入点的计算方式：
```
-(插入点) -1；
```
“插入点” 是指第一个大于此元素在数组中的位置，如果数组中所有的元素都小于这个值，插入点就是 ```a.size()```。如果数组中的元素有重复的值，则无法保证搜索到的是哪一个副本。

如果使用 Comparator 排序了某个对象数组，在使用 binarySearch() 时必须提供同样的接口 Comparator。例如：
```java
public class StringSorting {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		String[] strings = {"wo","shi","zhong","guo","ren","HAHA"};
		System.out.println("排序前："+Arrays.toString(strings));
		Arrays.sort(strings);
		System.out.println("排序后："+Arrays.toString(strings));
		Arrays.sort(strings,String.CASE_INSENSITIVE_ORDER);
		System.out.println("排序后1："+Arrays.toString(strings));
		int loaction = Arrays.binarySearch(strings,"zhong",String.CASE_INSENSITIVE_ORDER);
		System.out.println("查找到的位置："+loaction);

	}

}

```
执行结果：
```
排序前：[wo, shi, zhong, guo, ren, HAHA]
排序后：[HAHA, guo, ren, shi, wo, zhong]
排序后1：[guo, HAHA, ren, shi, wo, zhong]
查找到的位置：5
```
## 总结
你看到 Java 对尺寸固定的低级数组提供了适度的支持。这种数组强调的是性能而不是灵活性。在 Java 的初始版本中，尺寸固定的低级数组绝对是必要的，因为在早期的版本中对容器的支持是非常少的。其后的 Java 版本对容器的支持得到了明显的改进，并且现在容器除了性能之外的各个方面都比数组支持的更好。有了额外的自动包装机制和泛型，在容器中持有基本类型变得易如反掌，这也进一步促使我们用容器替换数组。因为泛型可以产生类型安全的容器。

所有的结论都表示：当你使用最新的 Java 版本时，应该优先使用容器而不是数组。只有在你证明性能成为问题时，你才应该将程序重构使用数组。
