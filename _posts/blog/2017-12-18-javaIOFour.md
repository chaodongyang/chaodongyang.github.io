---
layout: post
title: java编程思想之 I/O 系统(ZIP 与 XML)
categories: javaThinking
description: java编程思想之I/O 系统详解
keywords: java, javaI/O 系统,java 输入输出流，java I/O
---

本节将介绍 I/O 流中压缩文件、XML文件、序列化等问题。

## 压缩
Java I/O 类库支持读写压缩格式的数据流。你可以用他们对其他的 I/O 类进行封装，以提供压缩功能。这些类也是通过 InputStream 和 OutputStream 继承层次结构的一部分。这是因为压缩类库是按字节而不是字符方式处理的。压缩类有哪些请查看 JDK 文档。

尽管存在许多种压缩算法，但是 ZIP 和 GZIP 可能是最常用的。因为我们可以很容易的使用多种可读写这种格式的工具来操纵我们的压缩数据。

#### 利用 GZIP 进行简单压缩
如果我们想对单个数据流进行压缩 GZIP 接口是最合适的，因为它比较简单。下面是对单个文件进行压缩的示例：
```java
public class GZIPcompress {

	public static void main(String[] args) throws IOException{
		// TODO Auto-generated method stub
		BufferedReader in = new BufferedReader(new FileReader("file.txt"));
		BufferedOutputStream outputStream = new BufferedOutputStream(new GZIPOutputStream(new FileOutputStream("test.gz")));
		System.out.println("写入压缩文件");
		int c;
		while ((c = in.read())!= -1) {
			outputStream.write(c);
		}
		in.close();
		outputStream.close();

		System.out.println("读文件");

		BufferedReader ins = new BufferedReader(new InputStreamReader(new GZIPInputStream(new FileInputStream("test.gz"))));
		String string;

		while ((string = ins.readLine()) != null) {

			System.err.println(string);

		}
	}

}

```
测试结果：

![](/images/blog/20171218111320.png)

压缩类使用比较简单，直接将输出流封装成 GZIPOutputStream 或 ZipOutputStream。并将输入流封装成 GZIPInputStream 或 ZipInputStream。其他全部操作就是通常的 I/O 读写。

#### 用 ZIP 进行多文件保存
支持 ZIP 格式的 Java 库更加全面。利用该库可以方便的保存多个文件，他甚至有一个独立的类，使得读取 ZIP 文件更加方便。这个类库使用的是标准 ZIP 格式，所以能与当前那些可以通过因特网下载的压缩工具很好的协作。
```java
public class ZipCompress {

	public static void main(String[] args) throws IOException{
		FileOutputStream f = new FileOutputStream("test.zip");
		//Adler32 比 CRC32更快，但是 CRC32更精准
	    CheckedOutputStream csum =new CheckedOutputStream(f, new Adler32());
	    ZipOutputStream zos = new ZipOutputStream(csum);
	    BufferedOutputStream out = new BufferedOutputStream(zos);
	    zos.setComment("A test of Java Zipping");
	    // No corresponding getComment(), though.
	    for(String arg : args) {
	      Print.print("Writing file " + arg);
	      BufferedReader in = new BufferedReader(new FileReader(arg));
	      zos.putNextEntry(new ZipEntry(arg));
	      int c;
	      while((c = in.read()) != -1)
	        out.write(c);
	      in.close();
	      out.flush();
	    }
	    out.close();
	    // Checksum valid only after the file has been closed!
	    Print.print("Checksum: " + csum.getChecksum().getValue());
	    // Now extract the files:
	    Print.print("Reading file");
	    FileInputStream fi = new FileInputStream("test.zip");
	    CheckedInputStream csumi =
	      new CheckedInputStream(fi, new Adler32());
	    ZipInputStream in2 = new ZipInputStream(csumi);
	    BufferedInputStream bis = new BufferedInputStream(in2);
	    ZipEntry ze;
	    while((ze = in2.getNextEntry()) != null) {
	    	Print.print("Reading file " + ze);
	      int x;
	      while((x = bis.read()) != -1)
	        System.out.write(x);
	    }
	    if(args.length == 1)
	    	Print.print("Checksum: " + csumi.getChecksum().getValue());
	    bis.close();
	    // Alternative way to open and read Zip files:
	    ZipFile zf = new ZipFile("test.zip");
	    Enumeration e = zf.entries();
	    while(e.hasMoreElements()) {
	      ZipEntry ze2 = (ZipEntry)e.nextElement();
	      Print.print("File: " + ze2);
	      // ... and extract the data as before
	    }
	    /* if(args.length == 1) */
	  }

}

```
测试结果：
```
Checksum: 1923287158
Reading file
```
对于每一个要加入压缩文档的文件，都必须调用 putNextEntry()，并将其传递一个 ZipEntry 对象。ZipEntry 对象包含了一个很广泛的接口，允许你获取和设置 ZIP 文件内该特定项上所有可利用的数据：名字、压缩和未压缩的文件大小、日期、CRC 校验和、额外字段数据、注释、压缩方法以及它是否是一个目录入口等等。Java 的 ZIP 类库不支持设置密码的方法。

为了能够解压文件，ZipInputStream 提供了一个 getNextEntry() 方法返回一个 ZipEntry。解压缩文件有一个更简单的方法利用 ZipFile 对象读取文件。改对象有一个 entries() 方法用来向 ZipEntry 返回一个 Enumeration (枚举)。

为了读取校验和，必须拥有对与之相关联的 Checksum 对象的访问权限。在这里保留了指向 CheckedOutputStream 和 CheckedInputStream 对象的引用。但是也可以只保留一个指向 Checksum 对象的引用。

#### Java 档案文件
Zip 格式也被广泛应用于 JAR 文件格式中。这种文件格式就像 Zip 一样，可以将一组文件压缩到一组文件中。同 Java 中其他文件一样， JAR 文件也是跨平台的。声音和图像文件可以像类文件一样被包含在其中。

JAR 文件非常有用，尤其是涉及互联网应用的时候。如果不采用 jar 文件，web 浏览器在下载构成一个应用的所有文件时会重复多次请求 web 服务器；而所有文件都是未经压缩的。如果将这些文件都合并到 jar 文件中，只需向远程服务器发送一次请求即可。同时采用了压缩技术，可以使得传输时间更短。

一个 JAR 文件由一组压缩文件组成，同时还有描述了这一组文件的文件清单。在 JDK 文档中，可以找打与 jar 文件清单相关联的更多资料。

## XML
将数据转换为 XML 格式，可以被其他各种各样的平台所使用。因为 XML 十分的流行，用它来编程是十分广阔的。包括随 jdk 发布的 Javax.xml.* 类库。这里我们使用 Elliotte Rusty Harold 的开源 XOM 类库，因为这个看起来更简单，同时也是最直观的用 Java 产生和修改 XML 的方式。另外 XOM 还验证 XML 的正确性。

作为一个示例，假设有一个 Person 对象，它包含姓和名，你想将他们序列化到 XML 中。下面的类中有一个 getXML() 方法，它使用 XOM 来产生被转换为 XML 的 Element 对象的 Person 数据；还有一个构造器接受 Element 并从中抽取恰当的 Person 数据。
```java
public class Person {
	private String first,laSt;

	protected Person(String first, String laSt) {
		super();
		this.first = first;
		this.laSt = laSt;
	}

	public Element getXML() {
		Element person = new Element("person");
		Element firstName = new Element("first");
		firstName.appendChild(first);
		Element lastName = new Element("last");
		lastName.appendChild(laSt);
		person.appendChild(firstName);
		person.appendChild(lastName);
		return person;
	}

	public Person(Element person) {
		first = person.getFirstChildElement("first").getValue();
		laSt = person.getFirstChildElement("last").getValue();

	}

	@Override
	public String toString() {
		// TODO Auto-generated method stub
		return first + " " +laSt;
	}

	public static void format(OutputStream os,Document doc) throws Exception{
		Serializer serializer = new Serializer(os,"ISO-8859-1");
    // 设置行缩进  
		serializer.setIndent(4);
     // 设置行最大长度  
		serializer.setMaxLength(60);
		serializer.write(doc);
		serializer.flush();
	}

	public static void main(String[] args) throws Exception{
		List<Person> persons = Arrays.asList(new Person("chao","haha"),new Person("guo","xiaoyang"),new Person("li","dazui"));
		System.out.println(persons);
		Element root = new Element("people");
		for (Person person : persons) {
			root.appendChild(person.getXML());

		}
		Document document = new Document(root);
		format(System.out,document);
    //输出流将格式化的内容输出到文件
		format(new BufferedOutputStream(new FileOutputStream("People.xml")),document);
	}
}

```
测试结果：
```
[chao haha, guo xiaoyang, li dazui]
<?xml version="1.0" encoding="ISO-8859-1"?>
<people>
    <person>
        <first>chao</first>
        <last>haha</last>
    </person>
    <person>
        <first>guo</first>
        <last>xiaoyang</last>
    </person>
    <person>
        <first>li</first>
        <last>dazui</last>
    </person>
</people>

```
XOM 包含一个 Serializer 类，你可以在 format() 方法中看到它被用来将 XML 转换为更具可读性的格式。如果只调用 toXML()，那么所有的内容就会混在一起，Serializer 是一种便利的工具。

toXML() 格式：

![](/images/blog/20171218152736.png)

从 XML 中反序列化 Person 对象：
```java
public class People extends ArrayList<Person>{

	public People(String filename) throws Exception{
		//这里需要强行指定为 file 对象
		Document document = new Builder().build(new File(filename));
		Elements elements = document.getRootElement().getChildElements();
		for (int i = 0; i < elements.size(); i++) {
			add(new Person(elements.get(i)));
		}
	}

	public static void main(String[] args) throws Exception{
		// TODO Auto-generated method stub
		People people = new People("D:/eclipseWorkspace/ThreadTest/People.xml");
		System.out.println(people);

	}

}

```
测试结果：
```
[chao haha, guo xiaoyang, li dazui]
```
如果我们这样写：
```java
Document document = new Builder().build(fileName);
```
测试结果：
```java
Exception in thread "main" java.net.MalformedURLException: unknown protocol: d
	at java.net.URL.<init>(URL.java:600)
	at java.net.URL.<init>(URL.java:490)
	at java.net.URL.<init>(URL.java:439)
	at com.sun.org.apache.xerces.internal.impl.XMLEntityManager.setupCurrentEntity(XMLEntityManager.java:620)
	at com.sun.org.apache.xerces.internal.impl.XMLEntityManager.startEntity(XMLEntityManager.java:1305)
	at com.sun.org.apache.xerces.internal.impl.XMLEntityManager.startDocumentEntity(XMLEntityManager.java:1256)
	at com.sun.org.apache.xerces.internal.impl.XMLDocumentScannerImpl.setInputSource(XMLDocumentScannerImpl.java:257)
	at com.sun.org.apache.xerces.internal.parsers.DTDConfiguration.parse(DTDConfiguration.java:508)
	at com.sun.org.apache.xerces.internal.parsers.DTDConfiguration.parse(DTDConfiguration.java:590)
	at com.sun.org.apache.xerces.internal.parsers.XMLParser.parse(XMLParser.java:141)
	at com.sun.org.apache.xerces.internal.parsers.AbstractSAXParser.parse(AbstractSAXParser.java:1213)
	at nu.xom.Builder.build(Unknown Source)
	at nu.xom.Builder.build(Unknown Source)
	at IOFour.People.<init>(People.java:14)
	at IOFour.People.main(People.java:23)

```
因为如果我们传入的 URL 不能被识别为一个正确的文件路径，那么它将会向网络去访问这个路径。因而会产生异常。

People 构造器使用 XOM 的 Builder.build() 方法打开并读取一个文件，而 getChildElements() 方法产生一个 Elements 列表。在这个列表中每个 Element 都是一个 Person 对象。注意我们在 add() 添加的时候需要我们事先对文件的结构了解。如果文件的结构和我们预定的不一样，那么 XOM 将会抛出异常。

[参考XOM官方文档](https://xom.nu/#autolink "参考XOM官方文档")

## Preferences
Preferences 是一个键值集合，存储在一个节点层次结构中。节点层次结构可以创建更为复杂的结构，但通常是创建以你的类名命名的单一节点，然后将信息存储于其中。Preferences API 用于存储和读取用户的偏好以及程序配置项的设置。
```java
public class PreferencesDemo {
	public static void main(String[] args) throws Exception{
		Preferences preferences = Preferences.userNodeForPackage(PreferencesDemo.class);
		preferences.put("Location","oz");
		preferences.put("Foot","Ruby");
		preferences.putInt("Companions", 4);
		preferences.putBoolean("Are ss", true);

		int count = preferences.getInt("UsageCount",0);
		count++;
		preferences.putInt("UsageCount", count);
		for (String string : preferences.keys()) {
			System.out.println(string+" "+preferences.get(string, null));
		}
		System.out.println("Companions"+" "+preferences.getInt("Companions",0));
	}
}

```
测试结果：
```
Location oz
Foot Ruby
Companions 4
Are ss true
UsageCount 1
Companions 4
```
这里用的是 userNodeForPackage()，但是我们可以选择用 systemNodeForPackage()；虽然可以任意选择，但是最好将 “user” 用于个别用户的偏好，将 “system” 用于通用的安装配置。因为 main() 是静态的。因此，PreferencesDemo.class 可以用来标识节点；但是在非静态方法内部，我们通常使用 getClass()。尽管我们不一定非要把当前的类作为节点标识符。

一旦我们创建了节点，就可以用它来加载和读取数据。注意 get() 方法的第二个参数值，如果某个关键字下没有任何条目，那么这个参数就是产生的默认值。通常情况下我们希望提供一个合理的默认值。典型的语法如下：
```java
int count = preferences.getInt("UsageCount",0);
		count++;
```
这样在我们运行程序时，count 的值是 0。但在随后的引用中将会是非零值。

在我们运行 PreferencesDemo 时，会发现每次运行程序时，count 的值都会增 1。但是数据存储在哪里呢？在程序第一次运行之后我们发现本地并没有什么文件。Preference API 利用合适的系统资源来完成这个任务，并且这些资源会随操作系统不同而不同。例如在 windows 里边就是使用注册表。最重要的是，它已经神奇的为我们存储了信息，所以不必担心不同的操作系统是怎么运作的。
