---
layout: post
title: java编程思想之容器深入研究(基本容器类介绍)
categories: javaThinking
description: java编程思想之容器深入研究
keywords: java, java容器深入研究, 容器，java容器详解
---

本章节逐一介绍容器中各个类的用法。

## Collection 的功能方法
先展示一个例子：下面的例子展示了 Collection 的大部分的常用的方法。任何实现了 Collection 子类都具备相同的方法。所以，我们使用 ArrayList 来展示一下这些方法的用法。以即产生的结果。
```java
public class CollectionMethods {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		 Collection<String> c = new ArrayList<String>();
		    c.addAll(FlyweightMap.names(6));
		    c.add("ten");
		    c.add("eleven");
		    System.out.println(c);
		    // Make an array from the List:
		    Object[] array = c.toArray();
		    // Make a String array from the List:
		    String[] str = c.toArray(new String[0]);
		    // Find max and min elements; this means
		    // different things depending on the way
		    // the Comparable interface is implemented:
		    System.out.println("Collections.max(c) = " + Collections.max(c));
		    System.out.println("Collections.min(c) = " + Collections.min(c));
		    // Add a Collection to another Collection
		    Collection<String> c2 = new ArrayList<String>();
		    c2.addAll(FlyweightMap.names(6));
		    c.addAll(c2);
		    System.out.println(c);
		    c.remove(Countries.DATA[0][0]);
		    System.out.println(c);
		    c.remove(Countries.DATA[1][0]);
		    System.out.println(c);
		    // Remove all components that are
		    // in the argument collection:
		    c.removeAll(c2);
		    System.out.println(c);
		    c.addAll(c2);
		    System.out.println(c);
		    // Is an element in this Collection?
		    String val = Countries.DATA[3][0];
		    System.out.println("c.contains(" + val  + ") = " + c.contains(val));
		    // Is a Collection in this Collection?
		    System.out.println("c.containsAll(c2) = " + c.containsAll(c2));
		    Collection<String> c3 = ((List<String>)c).subList(3, 5);
		    // Keep all the elements that are in both
		    // c2 and c3 (an intersection of sets):
		    c2.retainAll(c3);
		    System.out.println(c2);
		    // Throw away all the elements
		    // in c2 that also appear in c3:
		    c2.removeAll(c3);
		    System.out.println("c2.isEmpty() = " +  c2.isEmpty());
		    c = new ArrayList<String>();
		    c.addAll(FlyweightMap.names(6));
		    System.out.println(c);
		    c.clear(); // Remove all elements
		    System.out.println("after c.clear():" + c);
	}

}

```
执行结果：
```
[ALGERIA, ANGOLA, BENIN, BOTSWANA, BURKINA FASO, BURUNDI, ten, eleven]
Collections.max(c) = ten
Collections.min(c) = ALGERIA
[ALGERIA, ANGOLA, BENIN, BOTSWANA, BURKINA FASO, BURUNDI, ten, eleven, ALGERIA, ANGOLA, BENIN, BOTSWANA, BURKINA FASO, BURUNDI]
[ANGOLA, BENIN, BOTSWANA, BURKINA FASO, BURUNDI, ten, eleven, ALGERIA, ANGOLA, BENIN, BOTSWANA, BURKINA FASO, BURUNDI]
[BENIN, BOTSWANA, BURKINA FASO, BURUNDI, ten, eleven, ALGERIA, ANGOLA, BENIN, BOTSWANA, BURKINA FASO, BURUNDI]
[ten, eleven]
[ten, eleven, ALGERIA, ANGOLA, BENIN, BOTSWANA, BURKINA FASO, BURUNDI]
c.contains(BOTSWANA) = true
c.containsAll(c2) = true
[ANGOLA, BENIN]
c2.isEmpty() = true
[ALGERIA, ANGOLA, BENIN, BOTSWANA, BURKINA FASO, BURUNDI]
after c.clear():[]

```
创建 ArrayList 来保存不同的数据集，然后向上转型为 Collection，所以代码很明显只用到了 Collection 接口。
## 可选操作
执行各种不同的操作的方法在 Collection 接口中都是可选操作。这意味着实现类并不需要为这些方法提供定义。为什么要将方法定义为可选的呢？那是因为这样做可以防止设计中出现接口过多的情况。如果容器类库的设计总是为了去描述各种不同的设计变体而最终患上了最终令人困惑的接口过剩。甚至是这样也仍然不能满足接口的各种特例对象。因此总是有人去发明新的接口。
 - 在 Java 类库中基本上能满足我们所需要的所有方法。因为我们 ```99%``` 的时间都是在使用容器类。例如，ArrayList、LinkedList、HashSet、和 HashMap。以及其他的具体实现基本都支持我们所有的操作。可选操作为我们留下了一个后门，如果我们想创建新的 Collection。并不需要为其中的接口全部提供有意义的定义这也是可以的。
 - 如果一个想要的操作未获得支持，那么实现接口的时候可能会导致 UnsupportedOperationException 异常。值得注意的是：未获得支持的操作只有在运行时才会被检测到。因此他们都是动态类型检查。

## 未获支持的操作
最常见的未获支持的操作，都是源自于背后由固定尺寸的数据结构支持的容器所导致的。比如，当我们用 Arrays.asList() 将一个数组转换为 List 的时候，就会得到这样的容器。我们还可以使用 Collection 类的方法创建出可以抛出 UnsupportedOperationException 异常的容器。
```java
public class Unsupported {

	static void test(String msg, List<String> list) {
	    System.out.println("--- " + msg + " ---");
	    Collection<String> c = list;
	    Collection<String> subList = list.subList(1,8);
	    // Copy of the sublist:
	    Collection<String> c2 = new ArrayList<String>(subList);
	    try { c.retainAll(c2); } catch(Exception e) {
	      System.out.println("retainAll(): " + e);
	    }
	    try { c.removeAll(c2); } catch(Exception e) {
	      System.out.println("removeAll(): " + e);
	    }
	    try { c.clear(); } catch(Exception e) {
	      System.out.println("clear(): " + e);
	    }
	    try { c.add("X"); } catch(Exception e) {
	      System.out.println("add(): " + e);
	    }
	    try { c.addAll(c2); } catch(Exception e) {
	      System.out.println("addAll(): " + e);
	    }
	    try { c.remove("C"); } catch(Exception e) {
	      System.out.println("remove(): " + e);
	    }
	    // The List.set() method modifies the value but
	    // doesn't change the size of the data structure:
	    try {
	      list.set(0, "X");
	    } catch(Exception e) {
	      System.out.println("List.set(): " + e);
	    }
	  }

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		 List<String> list = Arrays.asList("A B C D E F G H I J K L".split(" "));
			    test("Modifiable Copy", new ArrayList<String>(list));
			    test("Arrays.asList()", list);
			    test("unmodifiableList()",
			    Collections.unmodifiableList(new ArrayList<String>(list)));
	}

}

```
执行结果：
```
--- Modifiable Copy ---
--- Arrays.asList() ---
retainAll(): java.lang.UnsupportedOperationException
removeAll(): java.lang.UnsupportedOperationException
clear(): java.lang.UnsupportedOperationException
add(): java.lang.UnsupportedOperationException
addAll(): java.lang.UnsupportedOperationException
remove(): java.lang.UnsupportedOperationException
--- unmodifiableList() ---
retainAll(): java.lang.UnsupportedOperationException
removeAll(): java.lang.UnsupportedOperationException
clear(): java.lang.UnsupportedOperationException
add(): java.lang.UnsupportedOperationException
addAll(): java.lang.UnsupportedOperationException
remove(): java.lang.UnsupportedOperationException
List.set(): java.lang.UnsupportedOperationException

```
因为 Arrays.asList() 会产生一个 List。它基于一个固定大小的数组，仅支持那些不会改变数组大小的操作。任何会对底层数据结构引起修改的方法都会产生一个 UnsupportedOperationException 异常，以此来表示未获得支持的操作。Collections.unmodifiableList() 产生不可修改的列表。unmodifiableList() 的结果在任何情况下都应该不是可修改的。
## List 的功能方法
基本的 List 很容易使用：大多数我们只是调用 add() 添加对象，使用 get() 以此取出一个元素，以及调用 iterator() 获取用于该序列的 Iterator。

下面展示一个使用 LinkedList 的示例：
```java
public class Lists {
	private static boolean b;
	  private static String s;
	  private static int i;
	  private static Iterator<String> it;
	  private static ListIterator<String> lit;

	  //List 所支持的所有操作
	  public static void basicTest(List<String> a) {
	    a.add(1, "x"); // Add at location 1
	    a.add("x"); // Add at end
	    // Add a collection:
	    a.addAll(FlyweightMap.names(25));
	    // Add a collection starting at location 3:
	    a.addAll(3, FlyweightMap.names(25));
	    b = a.contains("1"); // Is it in there?
	    // Is the entire collection in there?
	    b = a.containsAll(FlyweightMap.names(25));
	    // Lists allow random access, which is cheap
	    // for ArrayList, expensive for LinkedList:
	    s = a.get(1); // Get (typed) object at location 1
	    i = a.indexOf("1"); // Tell index of object
	    b = a.isEmpty(); // Any elements inside?
	    it = a.iterator(); // Ordinary Iterator
	    lit = a.listIterator(); // ListIterator
	    lit = a.listIterator(3); // Start at loc 3
	    i = a.lastIndexOf("1"); // Last match
	    a.remove(1); // Remove location 1
	    a.remove("3"); // Remove this object
	    a.set(1, "y"); // Set location 1 to "y"
	    // Keep everything that's in the argument
	    // (the intersection of the two sets):
	    a.retainAll(FlyweightMap.names(25));
	    // Remove everything that's in the argument:
	    a.removeAll(FlyweightMap.names(25));
	    i = a.size(); // How big is it?
	    a.clear(); // Remove all elements
	  }
	  //使用Iterator 遍历元素
	  public static void iterMotion(List<String> a) {
	    ListIterator<String> it = a.listIterator();
	    b = it.hasNext();
	    b = it.hasPrevious();
	    s = it.next();
	    i = it.nextIndex();
	    s = it.previous();
	    i = it.previousIndex();
	  }
	  //使用 Iterator 修改元素
	  public static void iterManipulation(List<String> a) {
	    ListIterator<String> it = a.listIterator();
	    it.add("47");
	    // Must move to an element after add():
	    it.next();
	    // Remove the element after the newly produced one:
	    it.remove();
	    // Must move to an element after remove():
	    it.next();
	    // Change the element after the deleted one:
	    it.set("47");
	  }
	  //查看最终List 的操作结果
	  public static void testVisual(List<String> a) {
		  System.out.println(a);
	    List<String> b = FlyweightMap.names(25);
	    System.out.println("b = " + b);
	    a.addAll(b);
	    a.addAll(b);
	    System.out.println(a);
	    // Insert, remove, and replace elements
	    // using a ListIterator:
	    ListIterator<String> x = a.listIterator(a.size()/2);
	    x.add("one");
	    System.out.println(a);
	    System.out.println(x.next());
	    x.remove();
	    System.out.println(x.next());
	    x.set("47");
	    System.out.println(a);
	    // Traverse the list backwards:
	    x = a.listIterator(a.size());
	    while(x.hasPrevious())
	    	System.out.println(x.previous() + " ");
	    System.out.println();
	    System.out.println("testVisual finished");
	  }
	  // There are some things that only LinkedLists can do:
	  public static void testLinkedList() {
	    LinkedList<String> ll = new LinkedList<String>();
	    ll.addAll(FlyweightMap.names(25));
	    System.out.println(ll);
	    // Treat it like a stack, pushing:
	    ll.addFirst("one");
	    ll.addFirst("two");
	    System.out.println(ll);
	    // Like "peeking" at the top of a stack:
	    System.out.println(ll.getFirst());
	    // Like popping a stack:
	    System.out.println(ll.removeFirst());
	    System.out.println(ll.removeFirst());
	    // Treat it like a queue, pulling elements
	    // off the tail end:
	    System.out.println(ll.removeLast());
	    System.out.println(ll);
	  }
	  public static void main(String[] args) {
	    // Make and fill a new list each time:
	    basicTest(new LinkedList<String>(FlyweightMap.names(5)));
	    basicTest(new ArrayList<String>(FlyweightMap.names(5)));
	    iterMotion(new LinkedList<String>(FlyweightMap.names(5)));
	    iterMotion(new ArrayList<String>(FlyweightMap.names(5)));
	    iterManipulation(new LinkedList<String>(FlyweightMap.names(5)));
	    iterManipulation(new ArrayList<String>(FlyweightMap.names(5)));
	    testVisual(new LinkedList<String>(FlyweightMap.names(5)));
	    testLinkedList();
	  }
}

```
如果要知道运行结果就把代码放进编辑器运行一下看看。这里的方法和操作非常的多。在这里只是为了演示正确的语法。使用方法我们还是应该提前去查询一下 JDK 的帮助文档最好。

## Set 和 存储顺序
诸如 Integer 和 String 这样的 Java 预定义的类，可以很方便的放入 Set 容器中使用。当你创建自己的类型时，要注意到 Set 需要一种方式来维护存储顺序，而存储顺序如何维护，则在 Set 的不同实现之间会有变化。因此，不同的 Set 实现具有不同的行为，而且对于可以在特定的 Set 中放置的元素类型也有不同的要求。

*****************
- Set()：存入 Set 的每个元素都必须是唯一的，因为 Set 不保存重复的元素。加入元素必须定义 equals() 来确保对象的唯一性。Set 与 Collection 接口有完全一样的接口。Set 接口不保证维护元素的顺序。
- HashSet： 查找速度快，存入的元素必须实现 hashCode()
- ThreeSet：保持次序的 Set 底层为树结构。使用它可以从 Set 中提取有序列的数据。元素必须实现 Comparable 接口。
- LinkedHashSet： 具有 HashSet 的查询速度，内部使用链表维护元素的顺序。元素也必须实现 hashCode() 方法。
*****************
对于良好的代码风格，我们应该在覆盖 equals() 方法时，总是同时去覆盖 hashCode()。

下面例子演示了为了成功的使用特定的 Set 实现类型而必须定义的方法：

SetType 类：保持单一性
```java
public class SetType {
	  int i;
	  public SetType(int n) {
		  i = n;
	  }

	  public boolean equals(Object o) {
	    return o instanceof SetType && (i == ((SetType)o).i);
	  }

	  public String toString() {
		  return Integer.toString(i);
	  }
}

```
HashType 类：快速查找
```java
public class HashType extends SetType {
	  public HashType(int n) {
		  super(n);
	  }
	  public int hashCode() {
		  return i;
	  }
}

```
TreeType类：执行排序
```java
public class TreeType extends SetType implements Comparable<TreeType> {
	  public TreeType(int n) {
		  super(n);
      }
	  public int compareTo(TreeType arg) {
	    return (arg.i < i ? -1 : (arg.i == i ? 0 : 1));
	  }
}

```
调用测试：
```java
public class TypesForSets {

	static <T> Set<T> fill(Set<T> set, Class<T> type) {
	    try {
	      for(int i = 0; i < 10; i++)
	          set.add(type.getConstructor(int.class).newInstance(i));
	    } catch(Exception e) {
	      throw new RuntimeException(e);
	    }
	    return set;
	  }

	  static <T> void test(Set<T> set, Class<T> type) {
	    fill(set, type);
	    fill(set, type); // Try to add duplicates
	    fill(set, type);
	    System.out.println(set);
	  }

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		    test(new HashSet<HashType>(), HashType.class);
		    test(new LinkedHashSet<HashType>(), HashType.class);
		    test(new TreeSet<TreeType>(), TreeType.class);
		    // Things that don't work:
		    test(new HashSet<SetType>(), SetType.class);
		    test(new HashSet<TreeType>(), TreeType.class);
		    test(new LinkedHashSet<SetType>(), SetType.class);
		    test(new LinkedHashSet<TreeType>(), TreeType.class);
		    try {
		      test(new TreeSet<SetType>(), SetType.class);
		    } catch(Exception e) {
		      System.out.println(e.getMessage());
		    }
		    try {
		      test(new TreeSet<HashType>(), HashType.class);
		    } catch(Exception e) {
		      System.out.println(e.getMessage());
		    }
	}

}

```
执行结果：
```
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
[9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
[7, 3, 1, 9, 6, 5, 9, 1, 3, 8, 9, 0, 5, 2, 6, 1, 5, 4, 2, 0, 7, 4, 3, 8, 8, 7, 6, 4, 0, 2]
[8, 0, 3, 9, 0, 2, 8, 5, 2, 3, 4, 1, 2, 9, 4, 1, 9, 6, 7, 7, 4, 1, 3, 8, 6, 5, 7, 5, 6, 0]
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
java.lang.ClassCastException: containers.SetType cannot be cast to java.lang.Comparable
java.lang.ClassCastException: containers.HashType cannot be cast to java.lang.Comparable
```
从输出的结果来看，HashSet 以某种神秘的顺序元素。LinkedHashSet 按照元素插入的顺序保存元素，而 TreeSet 按照排序的顺序维护元素。

如果我们视图将没有恰当的支持必要操作的类型用于需要这些方法的 Set。那么就会有不正确的行为。对于没有重新定义 hashCode 的类 SetType 和 TreeType，他们将会产生重复的值，这样就违反了 Set 的基本契约。这也不会有运行错误，因为，默认的 hashCode 是合法的。如果我们尝试在 TreeSet中使用没有实现 Comparable 的类型，那么你将会得到一个异常抛出。

#### SortedSet
SortedSet 中的元素可以保证处于排序状态。SortedSet 的接口提供了下列附加方法以供我们使用：

- Comparator comparator() 返回当前 Set 使用的 Comparator；返回 null，表示以自然顺序排序。
- Object first() 返回容器中的第一个元素
- Object last() 返回容器中的最后一个元素
- SortedSet subSet(formElement,toElement) 生成次 Set 的子集，范围从 formElement(包含) 到 toElement(不包含)
- SortedSet headSet(toElement) 生成此 Set 的子集，由小于 toElement 的元素组成
- SortedSet tailSet(formElement) 生成此 Set 的子集，优大于或等于 formElement 的元素组成。

简单的示例：
```java
public class SortedSetDemo {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		 SortedSet<String> sortedSet = new TreeSet<String>();
		    Collections.addAll(sortedSet,
		      "one two three four five six seven eight"
		        .split(" "));
		    System.out.println(sortedSet);
		    String low = sortedSet.first();
		    String high = sortedSet.last();
		    System.out.println(low);
		    System.out.println(high);
		    Iterator<String> it = sortedSet.iterator();
		    for(int i = 0; i <= 6; i++) {
		      if(i == 3) low = it.next();
		      if(i == 6) high = it.next();
		      else it.next();
		    }
		    System.out.println(low);
		    System.out.println(high);
		    System.out.println(sortedSet.subSet(low, high));
		    System.out.println(sortedSet.headSet(high));
		    System.out.println(sortedSet.tailSet(low));
	}

}

```
执行结果：
```
[eight, five, four, one, seven, six, three, two]
eight
two
one
two
[one, seven, six, three]
[eight, five, four, one, seven, six, three]
[one, seven, six, three, two]

```
SortedSet 的意思是按照对象的比较函数顺序对元素排序，而不是指元素的插入顺序。如果要按照插入顺序排序请使用 LinkedHashSet 来保存。

## 队列
Queue 在 Java SE5 中仅有两个实现 LinkedList 和 PriorityQueue 。他们的差异在于排序的行为而不是性能。你可以将元素从队列的一端插入，并与另一端将他们取出。下面是涉及 Queue 实现的操作示例：
```java
public class QueueBehavior {

	 private static int count = 10;
	  static <T> void test(Queue<T> queue, Generator<T> gen) {
	    for(int i = 0; i < count; i++)
	      queue.offer(gen.next());
	    while(queue.peek() != null)
	      System.out.print(queue.remove() + " ");
	    System.out.println();
	  }
	  static class Gen implements Generator<String> {
	    String[] s = ("one two three four five six seven " +
	      "eight nine ten").split(" ");
	    int i;
	    public String next() { return s[i++]; }
	  }

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		    test(new LinkedList<String>(), new Gen());
		    //优先队列
		    test(new PriorityQueue<String>(), new Gen());
		    test(new ArrayBlockingQueue<String>(count), new Gen());
		    test(new ConcurrentLinkedQueue<String>(), new Gen());
		    test(new LinkedBlockingQueue<String>(), new Gen());
		    //优先队列
		    test(new PriorityBlockingQueue<String>(), new Gen());
	}

}

```
执行结果：
```
one two three four five six seven eight nine ten
eight five four nine one seven six ten three two
one two three four five six seven eight nine ten
one two three four five six seven eight nine ten
one two three four five six seven eight nine ten
eight five four nine one seven six ten three two
```
可以看到，除了优先队列，Queue 将精确的按照元素的插入顺序来产生他们。

#### 优先队列
先了解一下 to -do 列表：该列表包含一个字符串和一个主要的以及次要的优先级值。该列表的排列顺序也是通过实现 Comparable 而进行控制的。
```java
public class ToDoItem implements Comparable<ToDoItem>{
	private char primary;
	private int secondary;
	private String item;


	protected ToDoItem(char primary, int secondary, String item) {
		super();
		this.primary = primary;
		this.secondary = secondary;
		this.item = item;
	}


	@Override
	public int compareTo(ToDoItem o) {
		// TODO Auto-generated method stub
		if (primary >o.primary) {
			return +1;
		}

		if (primary ==o.primary) {
			if (secondary > o.secondary) {
				return +1;
			}else if (secondary == o.secondary) {
				return 0;
			}
		}
		return -1;

	}

	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return 	Character.toString(primary)+ secondary + ":" + item;
	}

}

```
实现 Queue 并且使用上述的排列顺序：
```java
public class ToDoList extends PriorityQueue<ToDoItem>{

	 public void add( char pri, int sec,String td) {
		    super.add(new ToDoItem(pri, sec, td));
	 }


	 public static void main(String[] args) {
		 ToDoList toDoList = new ToDoList();
		    toDoList.add('C', 4,"Empty trash");
		    toDoList.add('A', 2,"Feed dog");
		    toDoList.add( 'B', 7,"Feed bird");
		    toDoList.add( 'C', 3,"Mow lawn");
		    toDoList.add( 'A', 1,"Water lawn");
		    toDoList.add('B', 1,"Feed cat");
		    while(!toDoList.isEmpty())
		      System.out.println(toDoList.remove());
	}

}

```
执行结果：
```
A1:Water lawn
A2:Feed dog
B1:Feed cat
B7:Feed bird
C3:Mow lawn
C4:Empty trash

```
首先优先排列第一个元素，然后在排列第二个元素。

#### 双向队列
双向队列就像一个队列，不同的是我们可以在任何一端加入或移除元素。在 LinkedList 中包含支持双向队列的方法。但是 Java 类库中却没有这样的方法。我们无法实现这样的接口，但是可以通过组合来实现这样的功能。
```java
public class Deque<T> {
	  private LinkedList<T> deque = new LinkedList<T>();
	  public void addFirst(T e) { deque.addFirst(e); }
	  public void addLast(T e) { deque.addLast(e); }
	  public T getFirst() { return deque.getFirst(); }
	  public T getLast() { return deque.getLast(); }
	  public T removeFirst() { return deque.removeFirst(); }
	  public T removeLast() { return deque.removeLast(); }
	  public int size() { return deque.size(); }
	  public String toString() { return deque.toString(); }
}

```
实现双向插入：
```java
public class DequeTest {
	static void fillTest(Deque<Integer> deque) {
	    for(int i = 20; i < 27; i++)
	      deque.addFirst(i);
	    for(int i = 50; i < 55; i++)
	      deque.addLast(i);
	  }

	 public static void main(String[] args) {
	    Deque<Integer> di = new Deque<Integer>();
	    fillTest(di);
	    System.out.print(di);
	    while(di.size() != 0)
	    	System.out.print(di.removeFirst() + " ");
	    System.out.println();
	    fillTest(di);
	    while(di.size() != 0)
	    	System.out.print(di.removeLast() + " ");
	  }
}

```
执行结果：
```
[26, 25, 24, 23, 22, 21, 20, 50, 51, 52, 53, 54]
26 25 24 23 22 21 20 50 51 52 53 54
54 53 52 51 50 20 21 22 23 24 25 26
```
## 理解 Map
映射表的基本思想是维护键值对的关联，因此你可以通过键来查找值。Java 的标准类库中实现 Map 的接口的类有很多。但是行为特征却各不相同，这表现在效率，键值对的保存以及呈现次序，对象的保存周期，映射表如何在多线程程序中工作和判定“键”等价策略等方面。

下面展示一个简单的示例：
```java
public class AssociativeArray<K,V> {
	  private Object[][] pairs;
	  private int index;

	  public AssociativeArray(int length) {
	    pairs = new Object[length][2];
	  }

	  public void put(K key, V value) {
	    if(index >= pairs.length)
	      throw new ArrayIndexOutOfBoundsException();
	    pairs[index++] = new Object[]{ key, value };
	  }

	  @SuppressWarnings("unchecked")
	  public V get(K key) {
	    for(int i = 0; i < index; i++)
	      if(key.equals(pairs[i][0]))
	        return (V)pairs[i][1];
	    return null; // Did not find key
	  }

	  public String toString() {
	    StringBuilder result = new StringBuilder();
	    for(int i = 0; i < index; i++) {
	      result.append(pairs[i][0].toString());
	      result.append(" : ");
	      result.append(pairs[i][1].toString());
	      if(i < index - 1)
	        result.append("\n");
	    }
	    return result.toString();
	  }

	  public static void main(String[] args) {
	    AssociativeArray<String,String> map = new AssociativeArray<String,String>(6);
	    map.put("sky", "blue");
	    map.put("grass", "green");
	    map.put("ocean", "dancing");
	    map.put("tree", "tall");
	    map.put("earth", "brown");
	    map.put("sun", "warm");
	    try {
	      map.put("extra", "object"); // Past the end
	    } catch(ArrayIndexOutOfBoundsException e) {
	    	System.out.print("Too many objects!");
	    }
	    System.out.print(map);
	    System.out.print(map.get("ocean"));
	  }

}

```
执行结果：
```
Too many objects!sky : blue
grass : green
ocean : dancing
tree : tall
earth : brown
sun : warmdancing
```
为了使用 get() 方法，你需要传递你想要查找的 key，然后将它与之相关联的值作为结果返回，或者找不到的情况下返回 null。get() 方法使用的是通过 equals() 方法从数组的头部开始比较到末尾。这样的效率其实是很差的，但是这里关键是简单性而不是效率。

#### 性能
性能是映射表中的一个重要问题，当使用 get() 方法进行搜索时，执行速度相当的慢，而这正是 Map 的实现类 HashMap 提高速度的地方。HashMap 用了特殊的值称作散列码来取代对键的缓慢搜索。散列码是相对唯一的，用以代表对象的 int 值，他是通过将对象的某些信息进行转换而成的。hashCode() 是根类 Object 的方法，因此，所有的 Java 对象都会产生散列码。HashMap 就是使用对象的 HashCode() 进行快速查找的。

下面是 HashMap 的基本实现：

***************
- HashMap：Map 基于散列码的实现。插入和查询键值对的开销是固定的。可以通过构造器设置容量和负载因子，已调整容器的性能。在默认情况下应该首选 HashMap
- LinkedHashMap：类似于 HashMap，但是迭代遍历她时，取得键值对的顺序是插入顺序，或者最近最少使用的次序。比 HashMap 慢一点。但是迭代访问的时候更快，因为他是用链表维护内部次序的。
- TreeMap：根据红黑树实现。查看键值对时会被排序。TreeMap 的特点在于，所得到的结果是经过排序的。TreeMap 是唯一带有 subMap() 方法的 Map。可以返回一个子树。
- WeakHashMap：弱键影射，允许释放影射所指的对象；这是为解决某类特殊问题而设计的。可以被垃圾回收器回收。
- ConcurrentHashMap：一种线程安全的 Map，它不涉及同步加锁。
- IdentityHashMap：使用 == 代替 equals() 对键进行比较的散列映射。
***************

对 Map 中使用键的要求和 Set 是一样的，任何键都必须有一个 equals() 方法；如果键被用于散列 Map，那么他还必须具有恰当的 hashCode() 方法；如果键被用于 TreeMap,那么它必须实现 Comparable 接口。

下面的一个示例展示对 Map 接口的可用操作：

自定义的 Map：
```java
public class CountingMapData extends AbstractMap<Integer,String> {
	  private int size;
	  private static String[] chars =
	    "A B C D E F G H I J K L M N O P Q R S T U V W X Y Z"
	    .split(" ");
	  public CountingMapData(int size) {
	    if(size < 0) this.size = 0;
	    this.size = size;
	  }
	  private static class Entry
	  implements Map.Entry<Integer,String> {
	    int index;
	    Entry(int index) { this.index = index; }
	    public boolean equals(Object o) {
	      return Integer.valueOf(index).equals(o);
	    }
	    public Integer getKey() { return index; }
	    public String getValue() {
	      return
	        chars[index % chars.length] +
	        Integer.toString(index / chars.length);
	    }
	    public String setValue(String value) {
	      throw new UnsupportedOperationException();
	    }
	    public int hashCode() {
	      return Integer.valueOf(index).hashCode();
	    }
	  }
	  public Set<Map.Entry<Integer,String>> entrySet() {
	    // LinkedHashSet retains initialization order:
	    Set<Map.Entry<Integer,String>> entries =
	      new LinkedHashSet<Map.Entry<Integer,String>>();
	    for(int i = 0; i < size; i++)
	      entries.add(new Entry(i));
	    return entries;
	  }
	  public static void main(String[] args) {
	    System.out.println(new CountingMapData(60));
	  }
}

```
展示 Map 的基本操作：
```java
public class Maps {
	public static void printKeys(Map<Integer,String> map) {
		System.out.println("Size = " + map.size() + ", ");
		System.out.println("Keys: ");
		System.out.println(map.keySet()); // Produce a Set of the keys
	  }
	  public static void test(Map<Integer,String> map) {
		  System.out.println(map.getClass().getSimpleName());
	    map.putAll(new CountingMapData(25));
	    // Map has 'Set' behavior for keys:
	    map.putAll(new CountingMapData(25));
	    printKeys(map);
	    // Producing a Collection of the values:
	    System.out.println("Values: ");
	    System.out.println(map.values());
	    System.out.println(map);
	    System.out.println("map.containsKey(11): " + map.containsKey(11));
	    System.out.println("map.get(11): " + map.get(11));
	    System.out.println("map.containsValue(\"F0\"): "
	      + map.containsValue("F0"));
	    Integer key = map.keySet().iterator().next();
	    System.out.println("First key in map: " + key);
	    map.remove(key);
	    printKeys(map);
	    map.clear();
	    System.out.println("map.isEmpty(): " + map.isEmpty());
	    map.putAll(new CountingMapData(25));
	    // Operations on the Set change the Map:
	    map.keySet().removeAll(map.keySet());
	    System.out.println("map.isEmpty(): " + map.isEmpty());
	  }
	  public static void main(String[] args) {
	    test(new HashMap<Integer,String>());
	    test(new TreeMap<Integer,String>());
	    test(new LinkedHashMap<Integer,String>());
	    test(new IdentityHashMap<Integer,String>());
	    test(new ConcurrentHashMap<Integer,String>());
	    test(new WeakHashMap<Integer,String>());
	  }
}

```
keySet() 方法返回由 Map 的键组成的 Set。你可以直接打印 value() 方法的结果，该方法产生一个包含所有 Map 的值的 Collection。键值对，键是唯一的，而值是可以重复的。

#### SortedMap
使用 SortedMap 可以确保键处于排序的状态。

下面的例子演示了 SortedMap 的新增的功能：
```java
public class SortedMapDemo {
	public static void main(String[] args) {
	    TreeMap<Integer,String> sortedMap = new TreeMap<Integer,String>(new CountingMapData(10));
	    System.out.println(sortedMap);
	    //取得第一个键
	    Integer low = sortedMap.firstKey();
	    //取得最后一个键
	    Integer high = sortedMap.lastKey();
	    System.out.println(low);
	    System.out.println(high);
	    //返回一个 Iterator 用于迭代
	    Iterator<Integer> it = sortedMap.keySet().iterator();
	    for(int i = 0; i <= 6; i++) {
	      if(i == 3) low = it.next();
	      if(i == 6) high = it.next();
	      else it.next();
	    }
	    System.out.println(low);
	    System.out.println(high);
	    //生成此Map 的子集，范围从 low(包含) 到 high
	    System.out.println(sortedMap.subMap(low, high));
	    //生成子集，由键小于 hig 的所有键值组成
	    System.out.println(sortedMap.headMap(high));
	    //由键大于或等于 low 的键值组成
	    System.out.println(sortedMap.tailMap(low));
	  }
}

```
执行结果：
```
{0=A0, 1=B0, 2=C0, 3=D0, 4=E0, 5=F0, 6=G0, 7=H0, 8=I0, 9=J0}
0
9
3
7
{3=D0, 4=E0, 5=F0, 6=G0}
{0=A0, 1=B0, 2=C0, 3=D0, 4=E0, 5=F0, 6=G0}
{3=D0, 4=E0, 5=F0, 6=G0, 7=H0, 8=I0, 9=J0}
```
键值对是按照键的次序排序的。

#### LinkedHashMap
为了提高速度，LinkedHashMap 散裂化所有的元素，但是在遍历时，却又以元素的插入顺序来返回键值对。此外，可以在构造器中设定 LinkedHashMap，使之采用基于访问的最近最少使用算法，于是没有被访问过的元素会出现在队列的最前面。对于需要定期清理元素节省空间的程序来说非常容易实现。

来演示一下 LinkedHashMap 的两种特点：
```java
public class LinkedHashMapDemo {
	public static void main(String[] args) {
	    LinkedHashMap<Integer,String> linkedMap = new LinkedHashMap<Integer,String>(new CountingMapData(9));
	    System.out.println(linkedMap);
	    // Least-recently-used order:
	    linkedMap = new LinkedHashMap<Integer,String>(16, 0.75f, true);
	    linkedMap.putAll(new CountingMapData(9));
	    System.out.println(linkedMap);
	    for(int i = 0; i < 6; i++) // Cause accesses:
	    	//先访问前面的 6 个元素，0,-6
	      linkedMap.get(i);
	    // 没有被访问的 6-8 被移动到队列最前面
	    System.out.println(linkedMap);
	    linkedMap.get(0);
	    //再次访问 0 时，就变成了最新被访问的，移动到队列后边
	    System.out.println(linkedMap);
	  }
}

```
执行结果：
```
{0=A0, 1=B0, 2=C0, 3=D0, 4=E0, 5=F0, 6=G0, 7=H0, 8=I0}
{0=A0, 1=B0, 2=C0, 3=D0, 4=E0, 5=F0, 6=G0, 7=H0, 8=I0}
{6=G0, 7=H0, 8=I0, 0=A0, 1=B0, 2=C0, 3=D0, 4=E0, 5=F0}
{6=G0, 7=H0, 8=I0, 1=B0, 2=C0, 3=D0, 4=E0, 5=F0, 0=A0}
```
