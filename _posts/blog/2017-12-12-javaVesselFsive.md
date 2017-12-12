---
layout: post
title: java编程思想之容器深入研究(实用方法)
categories: javaThinking
description: java编程思想之容器深入研究
keywords: java, java容器深入研究,容器深入研究，java容器
---
我们来看一下容器给我们提供了那些具体的方便的可执行的方法。
## 实用方法
Java 中有大量的用于容器的卓越的使用方法，它们被表示为 java.util.Collections 类内部的静态方法。我们已经在前面使用过一部分了。我们可查询 JDK 的文档来查找更多的方法。

[JDK API 在线地址](https://docs.oracle.com/javase/8/docs/api/)

看下面的示例：
```java
public class Utilities {
	static List<String> list = Arrays.asList( "one Two three Four five six one".split(" "));
		  public static void main(String[] args) {
			 //打印这个集合
		    print(list);
		    //Collections.disjoint()当两个集合没有任何相同元素时返回 true
		    //Collections.singletonList(T x)产生不可变的 Set<T> List<T> 或 Map<K,V>只包含给定内容的单一项
		    print("'list' disjoint (Four)?: " +Collections.disjoint(list,Collections.singletonList("Four")));
		    //返回Collection 中最大和最小的元素
		    print("max: " + Collections.max(list));
		    print("min: " + Collections.min(list));
		    //采用后边的 String.CASE_INSENSITIVE_ORDER 比较方法返回最大和最小的元素，后边是一个 Comparator
		    print("max w/ comparator: " + Collections.max(list,String.CASE_INSENSITIVE_ORDER));
		    print("min w/ comparator: " + Collections.min(list,String.CASE_INSENSITIVE_ORDER));

		    List<String> sublist =Arrays.asList("Four five six".split(" "));
		    //返回 sublist 在 list 中第一次出现的位置，找不到返回 -1
		    print("indexOfSubList: " + Collections.indexOfSubList(list, sublist));
		    //返回 sublist 在 list 中最后一次出现的位置，找不到返回 -1
		    print("lastIndexOfSubList: " + Collections.lastIndexOfSubList(list, sublist));
		    //使用 Yo 替换掉 List 中所有的 one
		    Collections.replaceAll(list, "one", "Yo");
		    print("replaceAll: " + list);
		    //翻转所有的元素次序
		    Collections.reverse(list);
		    print("reverse: " + list);
		    //所有元素向后移动3个位置，末尾的元素循环到前面来
		    Collections.rotate(list, 3);
		    print("rotate: " + list);

		    List<String> source =Arrays.asList("in the matrix".split(" "));
		    //将 sourse 的元素复制到 list
		    Collections.copy(list, source);
		    print("copy: " + list);
		    //交换 list 中 位置 0 和 位置 list.size() - 1 的元素
		    Collections.swap(list, 0, list.size() - 1);
		    print("swap: " + list);
		    //随机改变列表的顺序
		    Collections.shuffle(list, new Random(47));
		    print("shuffled: " + list);
		    //用 pop 替换掉 list 中的所有元素
		    Collections.fill(list, "pop");
		    print("fill: " + list);
		    //返回 list 中 等于 pop 元素的个数
		    print("frequency of 'pop': " + Collections.frequency(list, "pop"));
		    //返回大小为 3 的 List<snap>
		    List<String> dups = Collections.nCopies(3, "snap");
		    print("dups: " + dups);

		    print("'list' disjoint 'dups'?: " +Collections.disjoint(list, dups));
		    // 为参数生成一个旧样式的 Enumeration<dups>
		    Enumeration<String> e = Collections.enumeration(dups);
		    Vector<String> v = new Vector<String>();
		    while(e.hasMoreElements())
		      v.addElement(e.nextElement());
		    // Converting an old-style Vector
		    // to a List via an Enumeration:
		    ArrayList<String> arrayList =Collections.list(v.elements());
		    print("arrayList: " + arrayList);
		  }
}

```
执行结果：
```
[one, Two, three, Four, five, six, one]
'list' disjoint (Four)?: false
max: three
min: Four
max w/ comparator: Two
min w/ comparator: five
indexOfSubList: 3
lastIndexOfSubList: 3
replaceAll: [Yo, Two, three, Four, five, six, Yo]
reverse: [Yo, six, five, Four, three, Two, Yo]
rotate: [three, Two, Yo, Yo, six, five, Four]
copy: [in, the, matrix, Yo, six, five, Four]
swap: [Four, the, matrix, Yo, six, five, in]
shuffled: [six, matrix, the, Four, Yo, five, in]
fill: [pop, pop, pop, pop, pop, pop, pop]
frequency of 'pop': 7
dups: [snap, snap, snap]
'list' disjoint 'dups'?: true
arrayList: [snap, snap, snap]

```
注意：min() 和 max() 只能作用于 Collection 对象，而不能作用于 List。

#### List 的排序与查询
List 的排序和查询所使用的的方法和对象数组多使用的方法有相同的名字和语法。
```java
public class ListSortSearch {
	public static void main(String[] args) {
	    List<String> list = new ArrayList<String>(Utilities.list);
	    list.addAll(Utilities.list);
	    print(list);
	    //随机改变列表的顺序
	    Collections.shuffle(list, new Random(47));
	    print("Shuffled: " + list);
	    // 从第10个位置的元素开始返回一个 迭代器
	    ListIterator<String> it = list.listIterator(10);
	    while(it.hasNext()) {
	      it.next();
	      it.remove();
	    }

	    print("Trimmed: " + list);
	    //默认排序
	    Collections.sort(list);
	    print("Sorted: " + list);
	    String key = list.get(7);
	    //查询 key 在 list 的位置
	    int index = Collections.binarySearch(list, key);
	    print("Location of " + key + " is " + index +
	      ", list.get(" + index + ") = " + list.get(index));

	    //按照后面的参数
	    Collections.sort(list, String.CASE_INSENSITIVE_ORDER);

	    print("Case-insensitive sorted: " + list);
	    key = list.get(7);
	    index = Collections.binarySearch(list, key,
	      String.CASE_INSENSITIVE_ORDER);

	    print("Location of " + key + " is " + index +
	      ", list.get(" + index + ") = " + list.get(index));
	  }
}

```
执行结果：
```
[one, Two, three, Four, five, six, one, one, Two, three, Four, five, six, one]
Shuffled: [Four, five, one, one, Two, six, six, three, three, five, Four, Two, one, one]
Trimmed: [Four, five, one, one, Two, six, six, three, three, five]
Sorted: [Four, Two, five, five, one, one, six, six, three, three]
Location of six is 7, list.get(7) = six
Case-insensitive sorted: [five, five, Four, one, one, six, six, three, three, Two]
Location of three is 7, list.get(7) = three
```
与数组排序和查找一样，如果使用 Comparator 进行排序，那么 binarySearch() 必须使用相同的 Comparator。
#### 设定 Collection 或 Map 为不可修改
创建一个只读的 Collection 或 Map，有时可以带来某些方便。Collections 类可以帮助我们达成此目的，它有一个方法，参数为原本的容器，返回值是容器的只读版本。此方法有大量的变种，对应了 Collection、List、Set和Map。

示例说明如何生成各种只读的容器：
```java
public class ReadOnly {
	static Collection<String> data = new ArrayList<String>(FlyweightMap.names(6));
	public static void main(String[] args) {
		    Collection<String> c =Collections.unmodifiableCollection( new ArrayList<String>(data));
		    print(c); // Reading is OK
		    //! c.add("one"); // Can't change it

		    List<String> a = Collections.unmodifiableList(new ArrayList<String>(data));
		    ListIterator<String> lit = a.listIterator();
		    print(lit.next()); // Reading is OK
		    //! lit.add("one"); // Can't change it

		    Set<String> s = Collections.unmodifiableSet(new HashSet<String>(data));
		    print(s); // Reading is OK
		    //! s.add("one"); // Can't change it

		    // For a SortedSet:
		    Set<String> ss = Collections.unmodifiableSortedSet(new TreeSet<String>(data));

		    Map<String,String> m = Collections.unmodifiableMap(new HashMap<String,String>(FlyweightMap.capitals(6)));
		    print(m); // Reading is OK
		    //! m.put("Ralph", "Howdy!");

		    // For a SortedMap:
		    Map<String,String> sm =Collections.unmodifiableSortedMap( new TreeMap<String,String>(FlyweightMap.capitals(6)));
		  }
}

```
执行结果：
```
[ALGERIA, ANGOLA, BENIN, BOTSWANA, BURKINA FASO, BURUNDI]
ALGERIA
[BENIN, BOTSWANA, ANGOLA, BURKINA FASO, ALGERIA, BURUNDI]
{BENIN=Porto-Novo, BOTSWANA=Gaberone, ANGOLA=Luanda, BURKINA FASO=Ouagadougou, ALGERIA=Algiers, BURUNDI=Bujumbura}
```
无论哪一种情况，再将容器设为只读之前，必须填入有意义的数据。装载数据后，就应该使用不可修改的方法返回的引用替换掉原来的版本。方法的调用并不会产生编译时的检查，但是转换之后，任何会改变容器内容的操纵都会引起 UnsupportedOperationException 异常。

#### Collection 或 Map 的同步控制
关键字 Synchronized 是多线程议题中的重要部分，在这里只是提醒一下。Collections 类有办法能够自动同步整个容器。
```java
public class Synchronization {
	public static void main(String[] args) {
	    Collection<String> c =Collections.synchronizedCollection(new ArrayList<String>());
	    List<String> list = Collections.synchronizedList(new ArrayList<String>());
	    Set<String> s = Collections.synchronizedSet(new HashSet<String>());
	    Set<String> ss = Collections.synchronizedSortedSet(new TreeSet<String>());
	    Map<String,String> m = Collections.synchronizedMap(new HashMap<String,String>());
	    Map<String,String> sm =Collections.synchronizedSortedMap(new TreeMap<String,String>());
	  }
}
```
直接将新生成的容器传递给适当的同步方法；这样做就不会有任何机会暴露出不同的版本。

#### 快速报错
Java 容器有一种保护机制，能够防止多个进程同时修改同一个容器的内容。如果在你迭代遍历某个容器的过程中，另一个进程进入，并且插入，删除，或修改容器内的某个对象。那么就会出现问题。Java 容器类库采用快速报错机制。它会探查容器上除了你的进程的操作以外的所有变化，一旦它发现其他的进程修改了内容，就会立刻抛出 ConcurrentModificationException 异常。

示例：只需要创建一个迭代器，然后向迭代器所指的 Collection 添加点什么
```java
public class FailFast {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		 Collection<String> c = new ArrayList<String>();
		    Iterator<String> it = c.iterator();
		    c.add("An object");
		    try {
		      String s = it.next();
		    } catch(ConcurrentModificationException e) {
		      System.out.println(e);
		    }
	}

}
```
执行结果：
```
java.util.ConcurrentModificationException
```
程序发生了异常，因为在容器取得迭代器之后，又有东西呗放入到迭代器。当程序的不同部分修改同一容器时，就可能导致容器的状态不一致。这个例子应该在添加元素之后，重新再获得迭代器。

## 持有引用
java.lang.ref 类库包含一组类，这些类为垃圾回收提供了更大的灵活性。当存在可能耗尽内存的大对象的时候，这些类显得特别有用。有三个继承自抽象类 Reference 的类分别是：SoftReference(软引用)、WeakReference(弱引用)和PhantomReference(虚引用)。当垃圾回收器考察对象只能通过某个 Reference 对象才可获得时，上述这些不同的派生类为垃圾回收器提供了不同级别的间接性指示。

对象是可获得的指此对象在程序的某处找到。这意味着在栈中有一个普通的引用，正在指向次对象；也可能是引用指向某个对象，而那个对象包含有另一个引用指向正在使用的对象；也可能有更多的链接。如果一个对象可获得，垃圾回收期就不能释放他，因为它仍然为你的程序使用。如果一个对象是不可获得的，那么你的程序将无法使用它，所以将其回收是安全的。

如果想继续持有某个对象的引用，希望以后还可以使用它，但是也希望能够让垃圾回收器能够释放它，这时候就该使用 Reference 对象。

以 Reference 对象作为你和普通引用之间的媒介，另外，一定不能有普通的引用指向那个对象。

SoftReference(软引用)、WeakReference(弱引用)和PhantomReference(虚引用) 由强到弱排列，对应不同级别的可获得性。SoftReference 用以实现内存敏感的高速缓存。而后者 WeakReference 是为实现规范影射而设计的，它不妨碍垃圾回收器回收影射的键或者值。规范影射的对象可以在程序中的多处被使用，以节省内存空间。PhantomReference
用以调度回收前的清理工作，它比 Java 终止机制更灵活。使用 SoftReference(软引用)、WeakReference(弱引用) 可以将他们放入 ReferenceQueue。而 PhantomReference 只能依赖于 ReferenceQueue。

下面是一个简单的示例：
VeryBig 对象类：
```java
public class VeryBig {
	private static final int SIZE = 10000;
	private long[] la = new long[SIZE];
	private String ident;

	public VeryBig(String id) {
		ident = id;
	}

	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return ident;
	}

	protected void finalize() {
		System.out.println("finalize"+ident);
	}
}

```
测试类：
```java
public class References {

	private static ReferenceQueue<VeryBig> rQueue = new ReferenceQueue<>();

	private static void checkQueue() {
		Reference<? extends VeryBig> iReferences = rQueue.poll();
		if (iReferences != null) {
			System.out.println("IN queue :" + iReferences.get());
		}
	}


	public static void main(String[] args) {
		int size = 10;
		if (args.length >0) {
			size = new Integer(args[0]);
		}
		LinkedList<SoftReference<VeryBig>> sa = new LinkedList<>();

		for (int i = 0; i < size; i++) {
			sa.add(new SoftReference<VeryBig>(new VeryBig("Soft " + i),rQueue));
			System.out.println("Just created: " + sa.getLast());
		    checkQueue();
		}


	    LinkedList<WeakReference<VeryBig>> wa = new LinkedList<WeakReference<VeryBig>>();
	    for(int i = 0; i < size; i++) {
	    	 wa.add(new WeakReference<VeryBig>(new VeryBig("Weak " + i), rQueue));
	    	 System.out.println("Just created: " + wa.getLast());
	    	 checkQueue();
	    }


	    SoftReference<VeryBig> s = new SoftReference<VeryBig>(new VeryBig("Soft"));
	    WeakReference<VeryBig> w = new WeakReference<VeryBig>(new VeryBig("Weak"));
	    System.gc();
	    LinkedList<PhantomReference<VeryBig>> pa = new LinkedList<PhantomReference<VeryBig>>();
	    for(int i = 0; i < size; i++) {
	    	  pa.add(new PhantomReference<VeryBig>(new VeryBig("Phantom " + i), rQueue));
	    	  System.out.println("Just created: " + pa.getLast());
	    	  checkQueue();
	    }
	}

}

```
测试结果：
```
Just created: java.lang.ref.SoftReference@15db9742
Just created: java.lang.ref.SoftReference@6d06d69c
Just created: java.lang.ref.SoftReference@7852e922
Just created: java.lang.ref.SoftReference@4e25154f
Just created: java.lang.ref.SoftReference@70dea4e
Just created: java.lang.ref.WeakReference@5c647e05
Just created: java.lang.ref.WeakReference@33909752
Just created: java.lang.ref.WeakReference@55f96302
Just created: java.lang.ref.WeakReference@3d4eac69
Just created: java.lang.ref.WeakReference@42a57993
finalizeWeak 4
finalizeWeak 0
finalizeWeak
finalizeWeak 3
finalizeWeak 2
finalizeWeak 1
Just created: java.lang.ref.PhantomReference@75b84c92
IN queue :null
Just created: java.lang.ref.PhantomReference@6bc7c054
IN queue :null
Just created: java.lang.ref.PhantomReference@232204a1
IN queue :null
Just created: java.lang.ref.PhantomReference@4aa298b7
IN queue :null
Just created: java.lang.ref.PhantomReference@7d4991ad
IN queue :null
```
运行程序看到，我们可以通过 Reference 对象访问那些对象，使用 get() 可以获得实际的对象引用。但对象还是被垃圾回收器回收了。还可以看出，ReferenceQueue 总是生成一个包含 null 对象的 Reference。

#### WeakHashMap
容器类有一个特殊的 Map，即 WeakHashMap。它被用来保存 WeakReference。它使得规范影射更容易使用。影射可以作为初始化的一部分，不过通常是在需要的时候才生成值。在影射中每个值只保存一份实例。这是一种节约存储空间的技术，因为 WeakHashMap 允许垃圾回收器自动清理键和值，所以显得十分便利。对于向 WeakHashMap 添加值的操作，影射会自动使用 WeakReference 包装他们。允许清理元素的条件是，不再需要此键了。

Element 类：
```java
public class Element {
	  private String ident;
	  public Element(String id) { ident = id; }
	  public String toString() { return ident; }
	  public int hashCode() { return ident.hashCode(); }
	  public boolean equals(Object r) {
	    return r instanceof Element &&
	      ident.equals(((Element)r).ident);
	  }
	  protected void finalize() {
	    System.out.println("Finalizing " +
	      getClass().getSimpleName() + " " + ident);
	  }
}

public class Key extends Element {
	  public Key(String id) { super(id); }
}

public class Value extends Element {
	  public Value(String id) { super(id); }
}

```
测试类：
```java
public class CanonicalMapping {
	 public static void main(String[] args) {
		    int size = 1000;

		    WeakHashMap<Key,Value> map = new WeakHashMap<Key,Value>();

		    for(int i = 0; i < size; i++) {
		      Key k = new Key(Integer.toString(i));
		      Value v = new Value(Integer.toString(i));

		      if(i % 3 == 0)
		      map.put(k, v);
		    }
		    System.gc();
	}
}

```
测试结果：
```
Finalizing Value 469
Finalizing Key 44
Finalizing Key 77
Finalizing Key 99
Finalizing Key 125
Finalizing Key 117
Finalizing Key 217
Finalizing Key 237
Finalizing Key 269
Finalizing Key 490
Finalizing Key 761
```
