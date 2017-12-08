---
layout: post
title: java编程思想之容器深入研究一
categories: javaThinking
description: java编程思想之容器深入研究
keywords: java, java容器深入研究, 容器，java容器详解
---

在之前的持有类型中已经介绍了 Java 容器的概念，对于如何的使用容器已经是足够了。今天开始再次深入的探索这个重要的类库。学习本章之前需要对前面的泛型先进一步的了解。

## 完整的容器分类法
先展示一张集合类库的完整图谱：

![](/images/blog/collections.png)

java SE5 新添加了：

- Queue 接口及其实现 PriorityQueue 和各种风格的 BlockingQueue。
- ConcurrentMap 接口及其实现 ConcurrentHashMap。他们是用于多线程机制的。
- CopyOnWriteArrayList 和 CopyOnWriteArraySet。也是用于多线程机制。
- EnumSet 和 EnumMap。为使用 enum 而设计的 Set 和 Map 的特殊实现。
- collections 类中的多个方法。

虚线框表示 abstract 类，你可以看到大量的类名都是以 abstract 开头。这些类只是实现了特定接口的工具。比如，我们如果要创建自己的 Set，那么并不用从 Set 接口开始实现全部的方法，只需要从 abstractSet 开始继承，然后执行一些创建类必须的工作即可。

## 填充容器
相应的 collections 类也有一些使用的静态方法，其中就有 fill()。与 Arrays 版本一样，此 fill() 方法也只是复制同一个对象引用来填充整个容器的，并且只对 List 有用，但是所产生的列表可以传递给构造器或 addAll() 方法：

StringAddress 类：
```java
public class StringAddress {
	  private String s;

	  public StringAddress(String s) {
		  this.s = s;
	  }

	  public String toString() {
	    return " " + s;
	  }
}

```
测试类：
```java
public class FillingLists {

	public static void main(String[] args) {
		// Collections.nCopies()创建传递给构造器的 List
		List<StringAddress> list= new ArrayList<StringAddress>(Collections.nCopies(4, new StringAddress("Hello")));
		System.out.println(list);
		//容器的内容替换为 World
	    Collections.fill(list, new StringAddress("World!"));
	    System.out.println(list);
	}

}
```
执行结果：
```java
[ Hello,  Hello,  Hello,  Hello]
[ World!,  World!,  World!,  World!]
```

#### 一种 Generator 解决方案
所有的 Collection 子类都有一个接收 Collection 对象的构造器，用所接收的对象的元素来填充新的元素。为了更容易的创建测试数据，我们在这里构建一个接收 Generator 和 quantity 数值为构造器参数的类：

接口：
```java
public interface Generator <T>{
	T next();
}
```
创建的构造器类：
```java
public class CollectionData<T> extends ArrayList<T>{

	public CollectionData(Generator<T> generator,int quantity) {
		for (int i = 0; i < quantity; i++) {
			add(generator.next());
		}
	}

	public static <T> CollectionData<T> list(Generator<T> generator,int quantity) {
		return new CollectionData<>(generator, quantity);

	}
}
```
这个类使用 Generator 在容器中放置所需数量的对象，然后所产生的容器可以传递给任何的 Collection 类型的构造器，这个构造器会把其中的数据复制到自身中。泛型方法可以减少在使用类时所必需的的类型检查数量。

下面展示一个 LinkedHashSet
```java
public class Government implements Generator<String>{

	private int index;

	String[] foundation = ("strange women lying in ponds " +
			    "distributing swords is no basis for a system of " +
			    "government").split(" ");

	@Override
	public String next() {
		// TODO Auto-generated method stub
		 return foundation[index++];
	}

}
```
初始化：
```java
public class CollectionDataTest {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		 Set<String> set = new LinkedHashSet<String>(new CollectionData<String>(new Government(),5));
			    // Using the convenience method:
			    set.addAll(CollectionData.list(new Government(),5));
			    System.out.println(set);
	}

}
```
执行结果：
```
[strange, women, lying, in, ponds]
```
元素的顺序和插入的顺序是相同的，LinkedHashSet 维护了保持插入顺序的链接列表。

#### Map 生成器
我们可以为 Map 使用相同的方法，但是需要一个键值对的泛型类，因为为了组装 Map，每次用 Generator 的 next() 方法都必须产生一个对象对：
```java
public class Pair<K,V> {
	public final K key;
	public final V value;
	protected Pair(K key, V value) {
		this.key = key;
		this.value = value;
	}

}
```
Map 适配器现在可以使用各种不同的 Generator、Iterator 和常量值的组合来填充 Map 初始化对象：
```java
public class MapData<K, V> extends LinkedHashMap<K, V>{

    public MapData(Generator<Pair<K, V>> generator,int quantity) {
		// TODO Auto-generated constructor stub
    	  for (int i = 0; i < quantity; i++) {
			Pair<K, V> pair = generator.next();
			put(pair.key, pair.value);
		}
	}

    public MapData(Generator<K> generatorK,Generator<V> generatorV,int quantity) {
		// TODO Auto-generated constructor stub
    	  for (int i = 0; i < quantity; i++) {
			put(generatorK.next(),generatorV.next());
		}
	}

    public MapData(Generator<K> generatorK,V value,int quantity) {
		// TODO Auto-generated constructor stub
    	  for (int i = 0; i < quantity; i++) {
			put(generatorK.next(),value);
		}
	}

    public MapData(Iterable<K> generatorK,Generator<V> generatorV) {
		// TODO Auto-generated constructor stub
    	for (K k : generatorK) {
    		put(k,generatorV.next());
		}

	}

    public MapData(Iterable<K> generatorK,V value) {
		// TODO Auto-generated constructor stub
    	for (K k : generatorK) {
    		put(k,value);
		}

	}

    public static <K,V> MapData<K, V> map(Generator<Pair<K, V>> generator,int quantity) {
		return new MapData<>(generator, quantity);

	}

    public static <K,V> MapData<K, V> map(Generator<K> generatorK,Generator<V> generatorV,int quantity) {
		return new MapData<>(generatorK, generatorV,quantity);

	}

    public static <K,V> MapData<K, V> map(Generator<K> generatorK,V value,int quantity) {
		return new MapData<>(generatorK, value,quantity);

	}

    public static <K,V> MapData<K, V> map(Iterable<K> generatorK,Generator<V> generatorV) {
		return new MapData<>(generatorK, generatorV);

	}

    public static <K,V> MapData<K, V> map(Iterable<K> generatorK,V value) {
		return new MapData<>(generatorK, value);

	}

}

```
上面的例子给了我们多种创建 Map 的方式，泛型方法可以减少在创建 MapData 类时所必需的的类型检数量。

下面展示的是一个使用 MapData 的示例：

Letters 类同时实现了 ```Generator<Pair<Integer,String>>``` 以及 ```Iterable<Integer>``` 。可以让我们测试上面任何一个方法。
```java
public class Letters implements Generator<Pair<Integer, String>>,Iterable<Integer> {

	private int size = 9;
    private int number = 1;
    private char letter = 'A';

	@Override
	public Iterator<Integer> iterator() {
		// TODO Auto-generated method stub
		return new Iterator<Integer>() {

			@Override
			public boolean hasNext() {
				// TODO Auto-generated method stub
				return number < size;
			}

			@Override
			public Integer next() {
				// TODO Auto-generated method stub
				return number++;
			}

			public void remove() {
		        throw new UnsupportedOperationException();
		      }
		};
	}

	@Override
	public Pair<Integer, String> next() {
		// TODO Auto-generated method stub
		return new Pair<Integer,String>(number++, "" + letter++);
	}

}

```
创建测试类：
```java
public class MapDataTest {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		System.out.println(MapData.map(new Letters(), 3));
	    // Two separate generators:
		System.out.println(MapData.map(new Letters(),new Letters(), 2));
	    // A key Generator and a single value:
		System.out.println(MapData.map(new Letters(), "Value", 3));
	    // An Iterable and a value Generator:
		System.out.println(MapData.map(new Letters(),3));
	    // An Iterable and a single value:
		System.out.println(MapData.map(new Letters(), "Pop"));
	}

}

```
执行结果：
```
{1=A, 2=B, 3=C}
{containers.Pair@15db9742=containers.Pair@7852e922, containers.Pair@6d06d69c=containers.Pair@4e25154f}
{containers.Pair@70dea4e=Value, containers.Pair@5c647e05=Value, containers.Pair@33909752=Value}
{1=A, 2=B, 3=C}
{1=Pop, 2=Pop, 3=Pop, 4=Pop, 5=Pop, 6=Pop, 7=Pop, 8=Pop}
```
#### 使用 Abstract 类
每个 Java 容器类都有自己的 abstract 类，他们提供了该容器的部分实现，你必须做的是去实现那些产生容器需要实现的方法。

本例中还会演示到一种设计模式：享元。你需要过多的对象，或者产生对象太占用空间时使用享元。享元模式使得对象的一部分可以被具体化，因此，与对象中的所有事物都包含在对象内不同，我们可以更加高效的外部表中查找对象的一部分或者整体。

为了创建只读的 Map，可以继承 AbstractMap 并实现 entrySet()。为了创建只读的 Set，可以继承 AbstractSet 并实现 iterator() 和 size()。

本例中使用的数据集是由国家以及首都组成的 Map，capitals() 方法产生国家与首都的 Map。name() 方法产生国名的 List。
```java
public class Countries {
	public static final String[][] DATA = {
		    // Africa
		    {"ALGERIA","Algiers"}, {"ANGOLA","Luanda"},
		    {"BENIN","Porto-Novo"}, {"BOTSWANA","Gaberone"},
		    {"BURKINA FASO","Ouagadougou"},
		    {"BURUNDI","Bujumbura"},
		    {"CAMEROON","Yaounde"}, {"CAPE VERDE","Praia"},
		    {"CENTRAL AFRICAN REPUBLIC","Bangui"},
		    {"CHAD","N'djamena"},  {"COMOROS","Moroni"},
		    {"CONGO","Brazzaville"}, {"DJIBOUTI","Dijibouti"},
		    {"EGYPT","Cairo"}, {"EQUATORIAL GUINEA","Malabo"},
		    {"ERITREA","Asmara"}, {"ETHIOPIA","Addis Ababa"},
		    {"GABON","Libreville"}, {"THE GAMBIA","Banjul"},
		    {"GHANA","Accra"}, {"GUINEA","Conakry"},
		    {"BISSAU","Bissau"},
		    {"COTE D'IVOIR (IVORY COAST)","Yamoussoukro"},
		    {"KENYA","Nairobi"}, {"LESOTHO","Maseru"},
		    {"LIBERIA","Monrovia"}, {"LIBYA","Tripoli"},
		    {"MADAGASCAR","Antananarivo"}, {"MALAWI","Lilongwe"},
		    {"MALI","Bamako"}, {"MAURITANIA","Nouakchott"},
		    {"MAURITIUS","Port Louis"}, {"MOROCCO","Rabat"},
		    {"MOZAMBIQUE","Maputo"}, {"NAMIBIA","Windhoek"},
		    {"NIGER","Niamey"}, {"NIGERIA","Abuja"},
		    {"RWANDA","Kigali"},
		    {"SAO TOME E PRINCIPE","Sao Tome"},
		    {"SENEGAL","Dakar"}, {"SEYCHELLES","Victoria"},
		    {"SIERRA LEONE","Freetown"}, {"SOMALIA","Mogadishu"},
		    {"SOUTH AFRICA","Pretoria/Cape Town"},
		    {"SUDAN","Khartoum"},
		    {"SWAZILAND","Mbabane"}, {"TANZANIA","Dodoma"},
		    {"TOGO","Lome"}, {"TUNISIA","Tunis"},
		    {"UGANDA","Kampala"},
		    {"DEMOCRATIC REPUBLIC OF THE CONGO (ZAIRE)",
		     "Kinshasa"},
		    {"ZAMBIA","Lusaka"}, {"ZIMBABWE","Harare"},
		    // Asia
		    {"AFGHANISTAN","Kabul"}, {"BAHRAIN","Manama"},
		    {"BANGLADESH","Dhaka"}, {"BHUTAN","Thimphu"},
		    {"BRUNEI","Bandar Seri Begawan"},
		    {"CAMBODIA","Phnom Penh"},
		    {"CHINA","Beijing"}, {"CYPRUS","Nicosia"},
		    {"INDIA","New Delhi"}, {"INDONESIA","Jakarta"},
		    {"IRAN","Tehran"}, {"IRAQ","Baghdad"},
		    {"ISRAEL","Jerusalem"}, {"JAPAN","Tokyo"},
		    {"JORDAN","Amman"}, {"KUWAIT","Kuwait City"},
		    {"LAOS","Vientiane"}, {"LEBANON","Beirut"},
		    {"MALAYSIA","Kuala Lumpur"}, {"THE MALDIVES","Male"},
		    {"MONGOLIA","Ulan Bator"},
		    {"MYANMAR (BURMA)","Rangoon"},
		    {"NEPAL","Katmandu"}, {"NORTH KOREA","P'yongyang"},
		    {"OMAN","Muscat"}, {"PAKISTAN","Islamabad"},
		    {"PHILIPPINES","Manila"}, {"QATAR","Doha"},
		    {"SAUDI ARABIA","Riyadh"}, {"SINGAPORE","Singapore"},
		    {"SOUTH KOREA","Seoul"}, {"SRI LANKA","Colombo"},
		    {"SYRIA","Damascus"},
		    {"TAIWAN (REPUBLIC OF CHINA)","Taipei"},
		    {"THAILAND","Bangkok"}, {"TURKEY","Ankara"},
		    {"UNITED ARAB EMIRATES","Abu Dhabi"},
		    {"VIETNAM","Hanoi"}, {"YEMEN","Sana'a"},
		    // Australia and Oceania
		    {"AUSTRALIA","Canberra"}, {"FIJI","Suva"},
		    {"KIRIBATI","Bairiki"},
		    {"MARSHALL ISLANDS","Dalap-Uliga-Darrit"},
		    {"MICRONESIA","Palikir"}, {"NAURU","Yaren"},
		    {"NEW ZEALAND","Wellington"}, {"PALAU","Koror"},
		    {"PAPUA NEW GUINEA","Port Moresby"},
		    {"SOLOMON ISLANDS","Honaira"}, {"TONGA","Nuku'alofa"},
		    {"TUVALU","Fongafale"}, {"VANUATU","< Port-Vila"},
		    {"WESTERN SAMOA","Apia"},
		    // Eastern Europe and former USSR
		    {"ARMENIA","Yerevan"}, {"AZERBAIJAN","Baku"},
		    {"BELARUS (BYELORUSSIA)","Minsk"},
		    {"BULGARIA","Sofia"}, {"GEORGIA","Tbilisi"},
		    {"KAZAKSTAN","Almaty"}, {"KYRGYZSTAN","Alma-Ata"},
		    {"MOLDOVA","Chisinau"}, {"RUSSIA","Moscow"},
		    {"TAJIKISTAN","Dushanbe"}, {"TURKMENISTAN","Ashkabad"},
		    {"UKRAINE","Kyiv"}, {"UZBEKISTAN","Tashkent"},
		    // Europe
		    {"ALBANIA","Tirana"}, {"ANDORRA","Andorra la Vella"},
		    {"AUSTRIA","Vienna"}, {"BELGIUM","Brussels"},
		    {"BOSNIA","-"}, {"HERZEGOVINA","Sarajevo"},
		    {"CROATIA","Zagreb"}, {"CZECH REPUBLIC","Prague"},
		    {"DENMARK","Copenhagen"}, {"ESTONIA","Tallinn"},
		    {"FINLAND","Helsinki"}, {"FRANCE","Paris"},
		    {"GERMANY","Berlin"}, {"GREECE","Athens"},
		    {"HUNGARY","Budapest"}, {"ICELAND","Reykjavik"},
		    {"IRELAND","Dublin"}, {"ITALY","Rome"},
		    {"LATVIA","Riga"}, {"LIECHTENSTEIN","Vaduz"},
		    {"LITHUANIA","Vilnius"}, {"LUXEMBOURG","Luxembourg"},
		    {"MACEDONIA","Skopje"}, {"MALTA","Valletta"},
		    {"MONACO","Monaco"}, {"MONTENEGRO","Podgorica"},
		    {"THE NETHERLANDS","Amsterdam"}, {"NORWAY","Oslo"},
		    {"POLAND","Warsaw"}, {"PORTUGAL","Lisbon"},
		    {"ROMANIA","Bucharest"}, {"SAN MARINO","San Marino"},
		    {"SERBIA","Belgrade"}, {"SLOVAKIA","Bratislava"},
		    {"SLOVENIA","Ljuijana"}, {"SPAIN","Madrid"},
		    {"SWEDEN","Stockholm"}, {"SWITZERLAND","Berne"},
		    {"UNITED KINGDOM","London"}, {"VATICAN CITY","---"},
		    // North and Central America
		    {"ANTIGUA AND BARBUDA","Saint John's"},
		    {"BAHAMAS","Nassau"},
		    {"BARBADOS","Bridgetown"}, {"BELIZE","Belmopan"},
		    {"CANADA","Ottawa"}, {"COSTA RICA","San Jose"},
		    {"CUBA","Havana"}, {"DOMINICA","Roseau"},
		    {"DOMINICAN REPUBLIC","Santo Domingo"},
		    {"EL SALVADOR","San Salvador"},
		    {"GRENADA","Saint George's"},
		    {"GUATEMALA","Guatemala City"},
		    {"HAITI","Port-au-Prince"},
		    {"HONDURAS","Tegucigalpa"}, {"JAMAICA","Kingston"},
		    {"MEXICO","Mexico City"}, {"NICARAGUA","Managua"},
		    {"PANAMA","Panama City"}, {"ST. KITTS","-"},
		    {"NEVIS","Basseterre"}, {"ST. LUCIA","Castries"},
		    {"ST. VINCENT AND THE GRENADINES","Kingstown"},
		    {"UNITED STATES OF AMERICA","Washington, D.C."},
		    // South America
		    {"ARGENTINA","Buenos Aires"},
		    {"BOLIVIA","Sucre (legal)/La Paz(administrative)"},
		    {"BRAZIL","Brasilia"}, {"CHILE","Santiago"},
		    {"COLOMBIA","Bogota"}, {"ECUADOR","Quito"},
		    {"GUYANA","Georgetown"}, {"PARAGUAY","Asuncion"},
		    {"PERU","Lima"}, {"SURINAME","Paramaribo"},
		    {"TRINIDAD AND TOBAGO","Port of Spain"},
		    {"URUGUAY","Montevideo"}, {"VENEZUELA","Caracas"},
		  };
}

```

测试类：
```java
public class FlyweightMap extends AbstractMap<String, String> {

	 private static class Entry implements Map.Entry<String,String> {
	      int index;
	      Entry(int index) {
	    	  this.index = index;
	      }
	      public boolean equals(Object o) {
	        return Countries.DATA[index][0].equals(o);
	      }

	      public String getKey() {
	    	  return Countries.DATA[index][0];
	      }
	      public String getValue() {
	    	  return Countries.DATA[index][1];
	      }
	      public String setValue(String value) {
	        throw new UnsupportedOperationException();
	      }

	      public int hashCode() {
	        return Countries.DATA[index][0].hashCode();
	      }
	   }

	 static class EntrySet extends AbstractSet<Map.Entry<String,String>> {
	      private int size;
	      EntrySet(int size) {
	        if(size < 0)
	          this.size = 0;
	        // Can't be any bigger than the array:
	        else if(size > Countries.DATA.length)
	          this.size = Countries.DATA.length;
	        else
	          this.size = size;
	      }
	      public int size() {
	    	  return size;
	      }

	      private class Iter implements Iterator<Map.Entry<String,String>> {
	        // Only one Entry object per Iterator:
	        private Entry entry = new Entry(-1);
	        public boolean hasNext() {
	          return entry.index < size - 1;
	        }
	        public Map.Entry<String,String> next() {
	          entry.index++;
	          return entry;
	        }
	        public void remove() {
	          throw new UnsupportedOperationException();
	        }
	      }


	      public Iterator<Map.Entry<String,String>> iterator() {
	        return new Iter();
	      }
	    }

	private static Set<Map.Entry<String,String>> entries = new EntrySet(Countries.DATA.length);

	@Override
	public Set<java.util.Map.Entry<String, String>> entrySet() {
		// TODO Auto-generated method stub
		return entries;
	}

	static Map<String,String> select(final int size) {
	    return new FlyweightMap() {
	      public Set<Map.Entry<String,String>> entrySet() {
	        return new EntrySet(size);
	      }
	    };
	  }
	  static Map<String,String> map = new FlyweightMap();
	  public static Map<String,String> capitals() {
	    return map; // The entire map
	  }
	  public static Map<String,String> capitals(int size) {
	    return select(size); // A partial map
	  }
	  static List<String> names = new ArrayList<String>(map.keySet());
	  // All the names:
	  public static List<String> names() {
		  return names;
	  }
	  // A partial list:
	  public static List<String> names(int size) {
	    return new ArrayList<String>(select(size).keySet());
	  }


	  public static void main(String[] args) {
		  System.out.println(capitals(10));
		  System.out.println(names(10));
		  System.out.println(new HashMap<String,String>(capitals(3)));
		  System.out.println(new LinkedHashMap<String,String>(capitals(3)));
		  System.out.println(new TreeMap<String,String>(capitals(3)));
		  System.out.println(new Hashtable<String,String>(capitals(3)));
		  System.out.println(new HashSet<String>(names(6)));
		  System.out.println(new LinkedHashSet<String>(names(6)));
		  System.out.println(new TreeSet<String>(names(6)));
		  System.out.println(new ArrayList<String>(names(6)));
		  System.out.println(new LinkedList<String>(names(6)));
		  System.out.println(capitals().get("BRAZIL"));
	  }

}

```
执行结果：
```
{ALGERIA=Algiers, ANGOLA=Luanda, BENIN=Porto-Novo, BOTSWANA=Gaberone, BURKINA FASO=Ouagadougou, BURUNDI=Bujumbura, CAMEROON=Yaounde, CAPE VERDE=Praia, CENTRAL AFRICAN REPUBLIC=Bangui, CHAD=N'djamena}
[ALGERIA, ANGOLA, BENIN, BOTSWANA, BURKINA FASO, BURUNDI, CAMEROON, CAPE VERDE, CENTRAL AFRICAN REPUBLIC, CHAD]
{BENIN=Porto-Novo, ANGOLA=Luanda, ALGERIA=Algiers}
{ALGERIA=Algiers, ANGOLA=Luanda, BENIN=Porto-Novo}
{ALGERIA=Algiers, ANGOLA=Luanda, BENIN=Porto-Novo}
{ALGERIA=Algiers, ANGOLA=Luanda, BENIN=Porto-Novo}
[BENIN, BOTSWANA, ANGOLA, BURKINA FASO, ALGERIA, BURUNDI]
[ALGERIA, ANGOLA, BENIN, BOTSWANA, BURKINA FASO, BURUNDI]
[ALGERIA, ANGOLA, BENIN, BOTSWANA, BURKINA FASO, BURUNDI]
[ALGERIA, ANGOLA, BENIN, BOTSWANA, BURKINA FASO, BURUNDI]
[ALGERIA, ANGOLA, BENIN, BOTSWANA, BURKINA FASO, BURUNDI]
Brasilia

```
这么绕还是捋一捋步骤吧：
 1. ```class FlyweightMap extends AbstractMap<String, String>``` 首先是万年不变的第一步，继承 ```AbstractMap```。
 2. 必须实现 ```entrySet()``` 方法。
 3. ```entrySet()``` 方法需要定制的一个 ```Set``` 和 定制的 ```Map.Entry``` 类。
 4. ```class Entry implements Map.Entry<String,String>```  对象存储了它的索引，而不是实际的键值对。当你调用 ```getKey()``` 和 ```getValue()``` 会返回给你实际的元素。
 5. ```class EntrySet```来确保她的 size 不会大于数据源。
 6. 每个迭代器都只包含一个 ```Map.Entry```。```Entry``` 对象被用作数据的窗口，它只包含静态字符串数组中的索引。每次调用 ```next()``` ,```Entry``` 中的 ```index``` 都会递增，指向下一个元素。
 7. ```select()``` 方法将产生一个包含指定尺寸的 ```EntrySet``` 的 ```FlyweightMap```。
