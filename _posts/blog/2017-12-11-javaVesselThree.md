---
layout: post
title: java编程思想之容器深入研究(散列与散列码)
categories: javaThinking
description: java编程思想之容器深入研究
keywords: java, java容器散列与散列码,散列与散列码，java散列与散列码
---
在前面学习到，通过定义 hashCode() 使用散列码可以快速的查找元素。那么到底什么是散列，散列码又是什么？

## 散列与散列码
首先看第一个例子，当我们自己创建用作 HashMap 的键的类，有可能会忘记在其中放置必要的方法。例如，考虑一个天气预报系统，将 Groundhog 对象与 Prediction 对象联系起来。使用 Groundhog 做为键，将 Prediction 用作值：

Groundhog 类：
```java
public class Groundhog {
	  protected int number;
	  public Groundhog(int n) {
		  number = n;
	  }

	  public String toString() {
	    return "Groundhog #" + number;
	  }
}

```
Prediction 类：
```java
public class Prediction {
	  private static Random rand = new Random(47);

	  private boolean shadow = rand.nextDouble() > 0.5;

	  public String toString() {
	    if(shadow)
	      return "Six more weeks of Winter!";
	    else
	      return "Early Spring!";
	  }
}
```
测试类：
```java
public class SpringDetector {

	public static <T extends Groundhog> void detectSpring(Class<T> type) throws Exception {
	    Constructor<T> ghog = type.getConstructor(int.class);
	    Map<Groundhog,Prediction> map = new HashMap<Groundhog,Prediction>();
	    for(int i = 0; i < 3; i++)
	      map.put(ghog.newInstance(i), new Prediction());
	    System.out.println("map = " + map);
	    Groundhog gh = ghog.newInstance(1);
	    System.out.println("Looking up prediction for " + gh);
	    if(map.containsKey(gh))
	    	System.out.println(map.get(gh));
	    else
	    	System.out.println("Key not found: " + gh);
	  }

	  public static void main(String[] args) throws Exception {
	    detectSpring(Groundhog.class);
	  }
}

```
运行结果:
```
map = {Groundhog #1=Six more weeks of Winter!, Groundhog #0=Six more weeks of Winter!, Groundhog #2=Early Spring!}
Looking up prediction for Groundhog #1
Key not found: Groundhog #1
```
这个例子看起来很简单，可是结果却出乎我们的意料。我们看到 map 中有 ```Groundhog #1``` ，但是我们却无法找到这个键。问题出自于 Groundhog 自动继承了 Object 类，所以这里使用了 Object 的 hashCode() 方法生成散列码，默认是使用对象的地址计算散列码。因此这两个不同生成的 ghog.newInstance(1) 是不同的。

可能我们认为只需要覆盖 hashCode() 方法就可以了，但是这仍然不行。因为我们还需要同时覆盖 equals() 方法。HashMap 用 equals() 方法判断当前键是否与表中存在的键相同。

正确的 equals() 方法必须满足 5 个条件：
 - 自反性：对任意 x，x.equals(x) 一定返回 true。
 - 对称性：对任意 x 和 y，如果 x.equals(y) 返回 true，则 y.equals(x) 也返回 true。
 - 传递性：对任意 x、y、z。如果 x.equals(y) 返回 true，y.equals(z) 返回 true，则 x.equals(z) 也返回 true。
 - 一致性：对任意 x 和 y。如果对象中用于等价的信息没有变，那么无论调用多少次 equals() 方法，返回的结果保持一致。
 - 对任何不是 null 的 x。x.equals(null) 一定返回 false。

默认的 Object.equals() 只是比较对象地址，因此如果要使用自己的类作为 HashMap 的键，必须同时重载 hashCode() 和 equals()。

```java
public class Groundhog2 extends Groundhog{

	public Groundhog2(int n) {
		super(n);
		// TODO Auto-generated constructor stub
	}
	public int hashCode() {
		return number;
	}

    public boolean equals(Object o) {
	    return o instanceof Groundhog2 &&(number == ((Groundhog2)o).number);
	  }
}

```
测试代码：
```java
public class SpringDetector2 {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		try {
			SpringDetector.detectSpring(Groundhog2.class);
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}

}

```
执行结果：
```
map = {Groundhog #0=Six more weeks of Winter!, Groundhog #1=Six more weeks of Winter!, Groundhog #2=Early Spring!}
Looking up prediction for Groundhog #1
Six more weeks of Winter!
```
#### 理解 HashCode()
前面的例子说明如果不为你的键覆盖 hashCode() 和 equals()，那么使用散列的数据结构就无法正确处理你的键。要很好的解决这个问题，必须了解这些数据结构的内部构造。使用散列的目的在于：想要使用一个对象来查找另一个对象。

下面的例子展示了用 ArrayList 来实现一个 Map：

自定义的 MapEntry 类：
```java
public class MapEntry<K,V> implements Map.Entry<K,V> {

	private K key;
	private V value;

	protected MapEntry(K key, V value) {
		super();
		this.key = key;
		this.value = value;
	}

	@Override
	public K getKey() {
		// TODO Auto-generated method stub
		return key;
	}

	@Override
	public V getValue() {
		// TODO Auto-generated method stub
		return value;
	}

	@Override
	public V setValue(V value) {
		V result = this.value;
		this.value = value;
		return result;
	}

	@Override
	public int hashCode() {
		// TODO Auto-generated method stub
		return (key == null ? 0 : key.hashCode())^ (value == null ? 0: value.hashCode());
	}

	@Override
	public boolean equals(Object obj) {
		if (!(obj instanceof MapEntry)) {
			return false;
		}
		MapEntry mapEntry = (MapEntry)obj;

		return (key == null ? mapEntry.getKey() == null : key.equals(mapEntry.getKey()))
				&& (value == null ? mapEntry.getValue() == null : value.equals(mapEntry.getValue()));
	}

	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return key +"="+value;
	}

}

```
自定义的 Map 类：
```java
public class SlowMap<K,V> extends AbstractMap<K, V>{

	 private List<K> keys = new ArrayList<K>();
	 private List<V> values = new ArrayList<V>();

	 @Override
	public V put(K key, V value) {
		V oldValue = get(key);
		if (!keys.contains(key)) {
			keys.add(key);
			values.add(value);
		}else {
			values.set(keys.indexOf(key),value);
		}
		return oldValue;
	}

	 @Override
	public V get(Object key) {
		if (!keys.contains(key)) {
			return null;
		}
		return values.get(keys.indexOf(key));
	}



	@Override
	public Set<java.util.Map.Entry<K, V>> entrySet() {
		Set<Map.Entry<K, V>> set = new HashSet<>();
		Iterator<K> kIterator = keys.iterator();
		Iterator<V> vIterator = values.iterator();
		while (kIterator.hasNext()) {
			set.add(new MapEntry<K,V>(kIterator.next(),vIterator.next()));

		}
		return set;
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		SlowMap<String, String> map = new SlowMap<>();
		map.putAll(FlyweightMap.capitals(5));
		System.out.println(map);
		System.out.println(map.get("BENIN"));
		System.out.println(map.entrySet());
	}

}

```
执行结果：
```
{ANGOLA=Luanda, BURKINA FASO=Ouagadougou, BENIN=Porto-Novo, ALGERIA=Algiers, BOTSWANA=Gaberone}
Porto-Novo
[ANGOLA=Luanda, BURKINA FASO=Ouagadougou, BENIN=Porto-Novo, ALGERIA=Algiers, BOTSWANA=Gaberone]
```
前面的部分其实我们已经了解过 Map.entrySet() 方法必须产生一个 Map.Entry 对象集。但是，Map.Entry 是一个接口，用来描述依赖实现的结构，因此如果想创建自己的 Map 类型，就必须同时定义 Map.Entry 的实现。MapEntry 采用了非常简单的方式，只使用 key 的 hashCode() 方法。看起来是很简单的，但是不太恰当，因为他创建了键和值的副本。在 MapEntry 中的 equals() 方法必须同时检查键和值。

#### 为速度而散列
上面的例子表明创建一个新的 Map 并不难。但是它不会很快，问题在于对键的查询，键没有按照任何的特定顺序保存，所以只能使用简单的线性查询，而线性查询是最慢的查询方式。

散列的价值在于速度：散列使得查询得意快速进行。由于瓶颈位于键的查询速度，因此我们需要解决的就是键的排序状态，然后使用 Collections.binarySearch() 进行查询。散列更进一步，它的键保存在某处，以便能够很快的找到。存储一组元素最快的数据结构是数组，所以使用它来表示键的信息。但是因为数组不能调整容量，因此有一个问题：我们希望保存 Map 数量是不确定的，数组容量是限制的该怎么办呢？数组并不保存键本身，而是通过键生成的一个数字将其作为下标。这个数字就是散列码，由 hashCode() 生成。

为了解决数组容量被固定的问题，不同的键可以产生不同的下标。因此会有冲突，重复。可以重复数组大小就不重要了。

于是，查询键的过程首先是计算散列码，然后使用散列码查询数组。如果能够保证没有冲突那是最完美的散列函数。通常还是会冲突的，冲突由外部链接处理：数组并不直接保存值，而是保存值的 List。然后对 List 中的值使用 equals 进行线性查询。这部分查询是比较慢的，但是如果散列函数设计好的话，这部分是很少的。所以我们只是快速的跳到数组的某个位置，只对很少的元素进行比较。这边是 HashMap 如此之快的原因。

我们事先一个简单的散列 Map：
```java
public class SimpleHashMap<K,V> extends AbstractMap<K, V>{

	//数组的大小
	static final int SIZE = 997;
	//定义一个数组，数组保存的是散列码的外部链接 List
    LinkedList<MapEntry<K,V>>[] buckets = new LinkedList[SIZE];

    @Override
    public V put(K key, V value) {
    	 V oldValue = null;
    	 //生成一个散列码
    	 int index = Math.abs(key.hashCode()) % SIZE;
    	 //如果散列码不在数组中，就直接添加到数组中的末尾
    	  if(buckets[index] == null)
    	      buckets[index] = new LinkedList<MapEntry<K,V>>();
    	  	//如果存在就赋予给一个 List
    	    LinkedList<MapEntry<K,V>> bucket = buckets[index];

    	    MapEntry<K,V> pair = new MapEntry<K,V>(key, value);

    	    boolean found = false;
    	    //迭代这个散列码的集合
    	    ListIterator<MapEntry<K,V>> it = bucket.listIterator();
    	    while(it.hasNext()) {
    	      MapEntry<K,V> iPair = it.next();
    	      if(iPair.getKey().equals(key)) {
    	        oldValue = iPair.getValue();
    	        it.set(pair); // Replace old with new
    	        found = true;
    	        break;
    	      }
    	    }
    	    if(!found)
    	      buckets[index].add(pair);
    	return oldValue;
    }

    @Override
    public V get(Object key) {
    	 int index = Math.abs(key.hashCode()) % SIZE;
    	   if(buckets[index] == null)
    		   return null;
    	    for(MapEntry<K,V> iPair : buckets[index])
    	      if(iPair.getKey().equals(key))
    	        return iPair.getValue();
    	    return null;
    }

	@Override
	public Set<java.util.Map.Entry<K, V>> entrySet() {
		  Set<Map.Entry<K,V>> set= new HashSet<Map.Entry<K,V>>();
		    for(LinkedList<MapEntry<K,V>> bucket : buckets) {
		      if(bucket == null)
		    	  continue;
		      for(MapEntry<K,V> mpair : bucket)
		        set.add(mpair);
		    }
		    return set;
	}

	public static void main(String[] args) {
		 SimpleHashMap<String,String> m = new SimpleHashMap<String,String>();
			    m.putAll(FlyweightMap.capitals(5));
			    System.out.println(m);
			    System.out.println(m.get("BENIN"));
			    System.out.println(m.entrySet());
	}

}

```
执行结果：
```
{ANGOLA=Luanda, BURKINA FASO=Ouagadougou, BENIN=Porto-Novo, ALGERIA=Algiers, BOTSWANA=Gaberone}
Porto-Novo
[ANGOLA=Luanda, BURKINA FASO=Ouagadougou, BENIN=Porto-Novo, ALGERIA=Algiers, BOTSWANA=Gaberone]
```
```
完美的散列函数在 JavaSE5 中的 EnumMap 和 EnumSet 中得到了实现。因为 enum 定义了固定数量的实例。
```
由于散列表的槽位通常称为桶位，因此我们将表示实际散列表的数组命名为 bucket。每一个新的元素都是直接添加到 list 末尾的某个特定桶位中。

为了生成合适的数组大小，取模操作将按照数组的尺寸取。如果数组的某个位置是 null，这表示没有元素被散列到此，所以，为了保存刚散列到此的对象，需要创建一个新的 List。一般而言，查看当前 list 中是否有相同的元素，如果有，则将旧的值付给 oldValue，然后用心的值取代它。标记 found 用来跟踪是否找到相同的旧的键值对，如果没有，则将新的对添加到 list 的末尾。

#### 覆盖 hashCode()
了解了如何散列之后，编写自己的 hashCode() 就更有意义了。

首先你无法控制 bucket 数组的下标值的产生。这个值依赖于具体的 HashMap 对象的容量，而容量的改变与容器的充满程度和负载因子有关。

设计 hashCode() 时最重要的因素是：无论何时，对同一个对象调用 hashCode() 都应该产生同样的值。如果将一个对象 put() 进的时候产生的 hashCode() 值和 get() 的时候产生的不是一个值，那就无法重新获取该对象了。此外，也不应该使 hashCode() 依赖于具有唯一性的对象信息。因为这样做无法生成一个新的键。这正是上边例子 SpringDetector 的问题所在，因为默认的 hashCode() 使用的是对象地址。所以我们应该使用对象内部有意义的识别信息。

下面以 String 类为例。String 有个特点：如果程序中有多个 String 对象，都包含相同的字符串序列，那么这些对象都会影射到同一块内存区域。随意 new String("aa") 生成的虽然是两个相互独立的实例，但是他们的 hashCode() 值产生相同的结果。

```java
public class StringHashCode {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		String[] strings = "hello hello".split(" ");
		System.out.println(strings[0].hashCode());
		System.out.println(strings[1].hashCode());
	}

}
```
执行结果：
```
99162322
99162322

```
对于 String 而言，hashCode() 明显是基于内容的。因此，想要使 hashCode() 实用，它必须速度快，并且具有实际的意义。也就是说，她必须基于对象的内容生成散列码。散列码不必是独一无二的，我们应该关注速度而不是唯一性。但是，通过 hashCode() 和 equals() 我们必须能够完全确定对象的身份。

生成数组的下标前，hashCode() 还需要做进一步的处理，所以散列码的生成范围不重要，只要是 int 即可。

还有一个重要的因素：好的 hashCode() 应该产生分布均匀的散列码。如果散列码都集中在一块，那么 hashMap 和 hashSet 某一块区域的负载会很重。

写出一份像样的 hashCode() 的基本指导：
- 给 int 变量赋予某个非 0  值常量。
- 为对象内每个有意义的域 f (每个可以做 equals() 操作的域) 计算出一个 int 散列码 c。
- 合并计算得到的散列码：result = 37 * result +c；
- 返回 result。
- 检查 hashCode() 最后产生的结果，确保相同的对象具有相同的散列码。

下面是遵循指导的一个例子：
```java
public class CountedString {
	 private static List<String> created = new ArrayList<String>();
			  private String s;
			  private int id = 0;

			  public CountedString(String str) {
			    s = str;
			    created.add(s);
			    // id is the total number of instances
			    // of this string in use by CountedString:
			    for(String s2 : created)
			      if(s2.equals(s))
			        id++;
			  }

			  public String toString() {
			    return "String: " + s + " id: " + id +
			      " hashCode(): " + hashCode();
			  }

			  public int hashCode() {
			    // The very simple approach:
			    // return s.hashCode() * id;
			    // Using Joshua Bloch's recipe:
			    int result = 17;
			    result = 37 * result + s.hashCode();
			    result = 37 * result + id;
			    return result;
			  }

			  public boolean equals(Object o) {
			    return o instanceof CountedString &&
			      s.equals(((CountedString)o).s) &&
			      id == ((CountedString)o).id;
			  }

			  public static void main(String[] args) {
			    Map<CountedString,Integer> map = new HashMap<CountedString,Integer>();

			    CountedString[] cs = new CountedString[5];
			    for(int i = 0; i < cs.length; i++) {
			      cs[i] = new CountedString("hi");
			      map.put(cs[i], i); // Autobox int -> Integer
			    }

			    System.out.println(map);
			    for(CountedString cstring : cs) {
			    	System.out.println("Looking up " + cstring);
			    	System.out.println(map.get(cstring));
			    }
			  }
}

```
执行结果：
```
{String: hi id: 4 hashCode(): 146450=3, String: hi id: 5 hashCode(): 146451=4, String: hi id: 2 hashCode(): 146448=1, String: hi id: 3 hashCode(): 146449=2, String: hi id: 1 hashCode(): 146447=0}
Looking up String: hi id: 1 hashCode(): 146447
0
Looking up String: hi id: 2 hashCode(): 146448
1
Looking up String: hi id: 3 hashCode(): 146449
2
Looking up String: hi id: 4 hashCode(): 146450
3
Looking up String: hi id: 5 hashCode(): 146451
4

```
CountedString 由一个 String 和 id 组成，此 id 代表相同的 String 的编号。所有的 String 都被存储在 ArrayList 中，在构造器中通过迭代遍历来实现 id 的计算。

hashCode() he  equals() 都是基于 CountedString 的两个域来生成结果；如果只是基于一个，不同的对象可能产生相同的值。我们使用相同的 String 创建了多个 CountedString 对象。虽然 String 相同，但是 id 不同，所以生成的散列码是不同的。

第二个实例：
```java
public class Individual implements Comparable<Individual> {
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
  public int hashCode() {
    int result = 17;
    if(name != null)
      result = 37 * result + name.hashCode();
    result = 37 * result + (int)id;
    return result;
  }
  public int compareTo(Individual arg) {
    // Compare by class name first:
    String first = getClass().getSimpleName();
    String argFirst = arg.getClass().getSimpleName();
    int firstCompare = first.compareTo(argFirst);
    if(firstCompare != 0)
    return firstCompare;
    if(name != null && arg.name != null) {
      int secondCompare = name.compareTo(arg.name);
      if(secondCompare != 0)
        return secondCompare;
    }
    return (arg.id < id ? -1 : (arg.id == id ? 0 : 1));
  }
} ///:~

```
 compareTo() 方法是一个比较结构，因此会产生一个排序序列，排序的规则是首先按照实际类型排序，然后如果有名字的话按照名字排序，最后按照创建的顺序排序。

 为新类编写正确的 hashCode() 和 equals() 需要很多的技巧。Apache 的 jakarta commons 有许多工具可以帮你完成此事。项目地址：jakarta.apache.org/commons/lang 下面。
