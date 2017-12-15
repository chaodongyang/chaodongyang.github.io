---
layout: post
title: java编程思想之 I/O 系统(新 I/O)
categories: javaThinking
description: java编程思想之I/O 系统详解
keywords: java, javaI/O 系统,java 输入输出流，java I/O
---

JDK 1.4 的 ```java.nio.*``` 包中引入了新的 Java I/O 类库，其目的是为了提高速度。实际上旧的 I/O 包已经使用了 nio 重新实现过，以便充分利用这种速度的提高。因此，即使我们不显式的使用 nio 编写代码也能从中收益。

## 新 I/O
速度的提高来自于使用的结构更接近于操作系统执行 I/O 的方式：通道和缓冲器。我们可以把它想成一个煤矿，通道是包含煤层的矿藏，而缓冲器则是运送煤层的卡车。卡车载满煤矿而归，我们在从卡车上获得煤矿。也就是说我们并没有直接和通道交互；我们只是和缓冲器交互，并把缓冲器派送到通道。通道要嘛从缓冲器获得数据，要嘛向缓冲器发送数据。

唯一直接与通道交道的缓冲器是 ByteBuffer，可以存储未加工字节的缓冲器。查询 JDK 文档的 java.nio.ByteBuffer 时，会发现它是相当基础的类：通过告知分配多少存储空间来创建一个 ByteBuffer 对象，并且还有一个方法选集，用来以原始的字节形式或基本的数据类型输出和读取数据。但是，没办法输出和读取对象，即使是字符串对象也不行。这种处理正好，因为这是大多数操作系统中更有效的映射方式。

旧 I/O 类库中有三个类被修改了，用以产生 FileChannel。这三个被修改的类是 FileInputStream、FileOutputStream 以及用来即读又写的 RandomAccessFile。注意，这些是字节操作流，与底层的 nio 性质是一致的。Reader 和 Writer 这种字符模式类不能用以产生通道；但是 java.nio.channels.Channels 类提供了实用方法，用以在通道中产生 Reader 和 Writer。

下面示例演示上面的三种流，用以产生可写的、可读可写的以及可读的通道。
```java
public class GetChannel {
	private static final int BSIZE = 1024;
	public static void main(String[] args) throws IOException{
		// TODO Auto-generated method stub
		FileChannel fChannel = new FileOutputStream("data.txt").getChannel();
		fChannel.write(ByteBuffer.wrap("Some text".getBytes()));
		fChannel.close();

		fChannel = new RandomAccessFile("data.txt","rw").getChannel();
		fChannel.position(fChannel.size());
		fChannel.write(ByteBuffer.wrap("Some more".getBytes()));
		fChannel.close();

		//最后读文件
		fChannel = new FileInputStream("data.txt").getChannel();
		ByteBuffer buffer = ByteBuffer.allocate(BSIZE);
		fChannel.read(buffer);
		buffer.flip();
		while (buffer.hasRemaining()) {
			System.out.print((char)buffer.get());
		}
	}

}

```
测试结果：
```
Some textSome more
```
这里所展示的流类，getChannel() 将会产生一个 FileChannel。通常是一种很基础的东西：可以向他传送用于读写的 ByteBuffer，并且可以锁定文件的某些区域用于独占式的访问。

将字节存放在 ByteBuffer 的方法之一是：使用 put() 方法直接对其进行填充，填入一个或多个字节，或基本数据类型的值。也可以使用 warp() 方法将已存在的字节数组包装到 ByteBuffer 中。一旦如此，就不在赋值底层的数组，而是把它作为所产生的的 ByteBuffer 的存储器，称之为数组支持的 ByteBuffer。

data.txt 文件用 RandomAccessFile 被再次打开。我们可以在文件内随处移动 FileChannel;这里我们把它移动到最后，以便附加其他的操作。

对于只读访问我们必须显式的调用静态的 allocate() 方法来分配 ByteBuffer。nio 的目标就是快速移动大量数据，因此 ByteBuffer 的大小就显得很重要。甚至达到跟高的速度也是有可能的，方法就是使用 allocateDirect() 而不是 allocate()，以产生一个与操作系统有更高耦合性的直接缓冲器。但是，这种分配的开支很大，具体的实现随着操作系统的不同而不同。因此必须每次都要重新运行程序来查看直接缓冲是否可以使我们获得速度上的优势。

一旦调用 read() FileChannel 向 ByteBuffer 存储字节，就必须调用缓冲器上的 flip()，让它做好让别人读取字节的准备。如果我们打算使用缓冲器执行进一步的 read() 操作，我们也必须得调用 clear() 为每个 read() 做好准备。看下面的示例:
```java
public class ChannelCopy {
	private static final int BISE = 1024;

	public static void main(String[] args) throws IOException{
		// TODO Auto-generated method stub
		FileChannel out = new FileOutputStream("data.text").getChannel(),
				    in = new FileInputStream("data.text").getChannel();
		ByteBuffer buffer = ByteBuffer.allocate(BISE);
		while (in.read(buffer) != -1) {
			buffer.flip();
			out.write(buffer);
			buffer.clear();


		}
	}

}

```
 打开一个 FileChannel 用于都，再打开另外一个用于写。ByteBuffer 被分配了存储空间，当 FileChannel.read() 返回 -1 时，表示我们已经达到输入的末尾。每次 read() 之后就会把数据输入到缓冲器中，flip() 则是准备缓冲器以便它的信息可以由 write() 提取。write() 操作之后信息仍在缓冲器中，接着 clear() 操作则对所有的内部指针重新安排，以便缓冲器在另一个 read() 操作期间能够做好接收数据的准备。

 上面的例子也不是处理此类操作的理想操作。特殊方法 transferTo() 和 transferForm() 则允许我们将一个通道和另一个通道直接相连：
 ```java
 public class TransferTo {

	public static void main(String[] args) throws IOException{
		// TODO Auto-generated method stub
		FileChannel out = new FileOutputStream("data.text").getChannel(),
				    in = new FileInputStream("data.text").getChannel();
		in.transferTo(0, in.size(),out);

		//或者也可以这么做
		out.transferFrom(in, 0, in.size());
	}

}

 ```
#### 转换数据
 上面的例子可以看出，为了读取文件中信息，我们每次只读取一个字节的数据，然后将每个 byte 类型强制转换为 char。如果我们查看一个 java.nio.CharBuffer 这个类，将会发现它有一个 toString() 方法是这样定义的：返回一个包含缓冲器中所有字符的字符串。既然 ByteBuffer 可以看做是具有 asCharBuffer() 方法的 CharBuffer，那么为什么不用他呢？
 ```java
 public class BufferToText {
	 private static final int BSIZE = 1024;
	  public static void main(String[] args) throws Exception {
	    FileChannel fc = new FileOutputStream("data2.txt").getChannel();
	    fc.write(ByteBuffer.wrap("Some text".getBytes()));
	    fc.close();
	    fc = new FileInputStream("data2.txt").getChannel();
	    ByteBuffer buff = ByteBuffer.allocate(BSIZE);
	    fc.read(buff);
	    buff.flip();
	    // 然而并不能正确的工作
	    System.out.println(buff.asCharBuffer());
	    // 使用系统的默认字符集解码
	    buff.rewind();//返回数据的最开始部分
	    String encoding = System.getProperty("file.encoding");//获取默认的字符集
	    System.out.println("Decoded using " + encoding + ": " + Charset.forName(encoding).decode(buff));

	    // 或者直接写入的时候带编码
	    fc = new FileOutputStream("data2.txt").getChannel();
	    fc.write(ByteBuffer.wrap("Some text".getBytes("UTF-16BE")));
	    fc.close();
	    // 然后再次阅读
	    fc = new FileInputStream("data2.txt").getChannel();
	    buff.clear();
	    fc.read(buff);
	    buff.flip();
	    //这次就可以正确的阅读了
	    System.out.println(buff.asCharBuffer());
	    // 直接使用 CharBuffer 写也可以读
	    fc = new FileOutputStream("data2.txt").getChannel();
	    buff = ByteBuffer.allocate(24); // More than needed
	    buff.asCharBuffer().put("Some text");
	    fc.write(buff);
	    fc.close();
	    // Read and display:
	    fc = new FileInputStream("data2.txt").getChannel();
	    buff.clear();
	    fc.read(buff);
	    buff.flip();
	    System.out.println(buff.asCharBuffer());
	  }
}

 ```
 测试结果：
 ```
 卯浥?數
Decoded using GBK: Some text
Some text
Some text
 ```
缓冲器容纳的是普通字节，为了把他们转换成字符，我们要嘛在输入的时候对其编码，要嘛将其从缓冲器输出的时候对他们进行编码。可以使用 java.nio.Charset.Charset 类实现这些功能，该类提供了把数据编码成多种不同类型的字符集的工具：
```java
public class AvailableCharSets {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		SortedMap<String,Charset> cMap = Charset.availableCharsets();
		Iterator<String> iterator = cMap.keySet().iterator();

		while (iterator.hasNext()) {
			String string = iterator.next();
			System.out.print(string);
			Iterator aIterator = cMap.get(string).aliases().iterator();

			if (aIterator.hasNext()) {
				System.out.println(" ");
			}

			while (aIterator.hasNext()) {
				System.out.print(aIterator.next());
				if (aIterator.hasNext()) {
					System.out.println(" ");
				}

			}
		}
	}

}

```
测试结果：
```
Big5
csBig5Big5-HKSCS
big5-hkscs
big5hk
Big5_HKSCS
big5hkscsCESU-8
CESU8
csCESU-8EUC-JP
csEUCPkdFmtjapanese
x-euc-jp
eucjis
///....
```
我们回顾一下上面的例子：BufferToText.java

如果我们相对缓冲器调用 rewind() 方法(回到数据的最开始部分)，接着使用平台的默认字符集进行 decode()，那么作为结果的 CharBuffer 可以很好的输出打印到控制台。可以使用 System.getProperty("file.eccoding") 发现默认的字符集，他会产生代表字符集名称的字符串。把该字符串传递给 Charset.forName() 用以产生 Charset 对象，可以用它对字符串进行编码。

另一种是在读取文件时，使用能够产生可打印的输出的字符集进行 encode()，上面代码看到 UTF-16BE 可以把文本写到文件中，当读取时，我们需要把它转换成 CharBuffer，就会产生所期望的结果。

最后，我们通过 CharBuffer 向 ByteBuffer 写入，会发生什么呢？我们为 ByteBuffer 分配了 24个字节。一个字符需要 2 个字节，那么我们可以容纳 12 个字符。但是我们只写了 9 个字符，剩余的内容为 0 的字节扔是存在的。会由他的 toString() 所产生的 CharBuffer 表示。可以在输出中看到。

#### 获取基本类型
尽管 ByteBuffer 只能保存字节类型的数据，但是它具有从它所容纳的字节中产生出各种不同基本类型值的方法。

示例展示使用这些方法来插入和抽取各种数值：
```java
public class GetData {
	private static final int BSIZE = 1024;
	  public static void main(String[] args) {
	    ByteBuffer bb = ByteBuffer.allocate(BSIZE);
	    // Allocation automatically zeroes the ByteBuffer:
	    int i = 0;
	    while(i++ < bb.limit())
	      if(bb.get() != 0)
	    	  Print.print("nonzero");
	    Print.print("i = " + i);
	    bb.rewind();
	    // Store and read a char array:
	    bb.asCharBuffer().put("Howdy!");
	    char c;
	    while((c = bb.getChar()) != 0)
	    	 Print.printnb(c + " ");
	    Print.print();
	    bb.rewind();
	    // Store and read a short:
	    bb.asShortBuffer().put((short)471142);
	    Print.print(bb.getShort());
	    bb.rewind();
	    // Store and read an int:
	    bb.asIntBuffer().put(99471142);
	    Print.print(bb.getInt());
	    bb.rewind();
	    // Store and read a long:
	    bb.asLongBuffer().put(99471142);
	    Print.print(bb.getLong());
	    bb.rewind();
	    // Store and read a float:
	    bb.asFloatBuffer().put(99471142);
	    Print.print(bb.getFloat());
	    bb.rewind();
	    // Store and read a double:
	    bb.asDoubleBuffer().put(99471142);
	    Print.print(bb.getDouble());
	    bb.rewind();
	  }
}

```
测试结果：
```
i = 1025
H o w d y !
12390
99471142
99471142
9.9471144E7
9.9471142E7
```
在分配一个 ByteBuffer 之后，可以通过检测它的值来查看缓冲器内容是否自动清零了。这里共检测出 1024 个值，并且所有的值都是 0。

向 ByteBuffer 插入基本类型数据最简单的方法是：利用 asCharBuffer()、asShortBuffer() 等获得该缓冲器上的视图，然后使用视图的 put() 方法。此方法适用于所有基本类型的数据。注意：使用 asShortBuffer 的 put() 方法时，需要进行类型转换，类型转换会截取和改变结果。而其他的使用 put() 方法时不需要进行转换。

#### 视图缓冲器
视图缓冲器可以让我们通过某种特定的基本数据类型的视窗查看器底层的 ByteBuffer。ByteBuffer 依然是实际存储数据的地方，支持着前面的视图。因此，对视图的任何修改都会影射成为对 ByteBuffer 的修改。我们看到上面的例子，我们可以很方便的插入数据。视图还允许我们从 ByteBuffer 一次一个或者成批的读取基本类型值。

下面的例子通过 IntBuffer 操纵 ByteBuffer 中的 int 类型数据。
```java
public class InBufferDemo {
	private static final int BISE = 1024;
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		ByteBuffer buffer = ByteBuffer.allocate(BISE);
		IntBuffer intBuffer = buffer.asIntBuffer();
		intBuffer.put(new int[]{11,42,47,99,143,811,1016});
		System.out.println(intBuffer.get(3));
		intBuffer.put(3,1811);
		intBuffer.flip();
		while (intBuffer.hasRemaining()) {
			int i = intBuffer.get();
			System.out.print(i);

		}
	}

}

```
测试结果：
```
99
11424718111438111016
```
先用重载后的方法 put() 存储一个整数数组。接着 get() 和 put() 方法调用直接访问底层 ByteBuffer 中某个位置的整数。

一旦底层的 ByteBuffer 通过视图缓冲器填满了整数或其他的基本类型时，就可以直接被写到通道中了。正想从通道中读取一样简单，然后使用视图缓冲器可以把任何数据转换为特定的基本类型。下面的例子，在同一个 ByteBuffer 上建立不同的视图缓冲器，将同一字节序列翻译成了 short、int、float、long 和 doube 类型的数据。
```java
public class ViewBuffers {
	public static void main(String[] args) {
	    ByteBuffer bb = ByteBuffer.wrap(new byte[]{ 0, 0, 0, 0, 0, 0, 0, 'a' });
	    bb.rewind();
	    Print.printnb("Byte Buffer ");
	    while(bb.hasRemaining())
	    	 Print.printnb(bb.position()+ " -> " + bb.get() + ", ");
	    Print.print();

	    CharBuffer cb =((ByteBuffer)bb.rewind()).asCharBuffer();
	    Print.printnb("Char Buffer ");
	    while(cb.hasRemaining())
	    	 Print.printnb(cb.position() + " -> " + cb.get() + ", ");
	    Print.print();

	    FloatBuffer fb =
	      ((ByteBuffer)bb.rewind()).asFloatBuffer();
	    Print.printnb("Float Buffer ");
	    while(fb.hasRemaining())
	    	 Print.printnb(fb.position()+ " -> " + fb.get() + ", ");
	    Print.print();
	    IntBuffer ib =
	      ((ByteBuffer)bb.rewind()).asIntBuffer();
	    Print.printnb("Int Buffer ");
	    while(ib.hasRemaining())
	    	 Print.printnb(ib.position()+ " -> " + ib.get() + ", ");
	    Print.print();
	    LongBuffer lb =
	      ((ByteBuffer)bb.rewind()).asLongBuffer();
	    Print.printnb("Long Buffer ");
	    while(lb.hasRemaining())
	    	 Print.printnb(lb.position()+ " -> " + lb.get() + ", ");
	    Print.print();
	    ShortBuffer sb =
	      ((ByteBuffer)bb.rewind()).asShortBuffer();
	    Print.printnb("Short Buffer ");
	    while(sb.hasRemaining())
	    	 Print.printnb(sb.position()+ " -> " + sb.get() + ", ");
	    Print.print();
	    DoubleBuffer db =
	      ((ByteBuffer)bb.rewind()).asDoubleBuffer();
	    Print.printnb("Double Buffer ");
	    while(db.hasRemaining())
	    	 Print.printnb(db.position()+ " -> " + db.get() + ", ");
	  }
}

```
测试结果：
```
Byte Buffer 0 -> 0, 1 -> 0, 2 -> 0, 3 -> 0, 4 -> 0, 5 -> 0, 6 -> 0, 7 -> 97,
Char Buffer 0 ->
Float Buffer 0 -> 0.0, 1 -> 1.36E-43,
Int Buffer 0 -> 0, 1 -> 97,
Long Buffer 0 -> 97,
Short Buffer 0 -> 0, 1 -> 0, 2 -> 0, 3 -> 97,
Double Buffer 0 -> 4.8E-322,
```
ByteBuffer 通过一个被包装的 8 字节数组产生，然后通过各种不同版本的基本类型的视图缓冲器显示出来。从不同的类型的缓冲器读取时，数据显示的方式也不同。

这与上面程序的输出相对应

<table width="864" border="0" cellpadding="0" cellspacing="0" style='width:432.00pt;border-collapse:collapse;table-layout:fixed;'>
  <col width="96" span="9" style='width:48.00pt;'/>
  <tr height="28" style='height:14.00pt;'>
   <td height="28" width="96" style='height:14.00pt;width:48.00pt;' x:str>bytes</td>
   <td width="96" align="right" style='width:48.00pt;' x:num>0</td>
   <td width="96" align="right" style='width:48.00pt;' x:num>0</td>
   <td width="96" align="right" style='width:48.00pt;' x:num>0</td>
   <td width="96" align="right" style='width:48.00pt;' x:num>0</td>
   <td width="96" align="right" style='width:48.00pt;' x:num>0</td>
   <td width="96" align="right" style='width:48.00pt;' x:num>0</td>
   <td width="96" align="right" style='width:48.00pt;' x:num>0</td>
   <td width="96" align="right" style='width:48.00pt;' x:num>97</td>
  </tr>
  <tr height="28" style='height:14.00pt;'>
   <td height="28" style='height:14.00pt;' x:str>chars</td>
   <td class="xl65" colspan="2" style='border-right:none;border-bottom:none;'></td>
   <td class="xl66" colspan="2" style='border-right:none;border-bottom:none;'></td>
   <td class="xl66" colspan="2" style='border-right:none;border-bottom:none;'></td>
   <td class="xl66" colspan="2" style='border-right:none;border-bottom:none;' x:str>a</td>
  </tr>
  <tr height="28" style='height:14.00pt;'>
   <td height="28" style='height:14.00pt;' x:str>shorts</td>
   <td class="xl66" colspan="2" style='border-right:none;border-bottom:none;' x:num>0</td>
   <td class="xl66" colspan="2" style='border-right:none;border-bottom:none;' x:num>0</td>
   <td class="xl66" colspan="2" style='border-right:none;border-bottom:none;' x:num>0</td>
   <td class="xl66" colspan="2" style='border-right:none;border-bottom:none;' x:num>97</td>
  </tr>
  <tr height="28" style='height:14.00pt;'>
   <td height="28" style='height:14.00pt;' x:str>ints</td>
   <td class="xl66" colspan="4" style='border-right:none;border-bottom:none;' x:num>0</td>
   <td class="xl66" colspan="4" style='border-right:none;border-bottom:none;' x:num>97</td>
  </tr>
  <tr height="28" style='height:14.00pt;'>
   <td height="28" style='height:14.00pt;' x:str>floats</td>
   <td class="xl66" colspan="4" style='border-right:none;border-bottom:none;' x:num>0</td>
   <td class="xl67" colspan="4" style='border-right:none;border-bottom:none;' x:num="1.3600000000000001e-043">1.36E-43</td>
  </tr>
  <tr height="28" style='height:14.00pt;'>
   <td height="28" style='height:14.00pt;' x:str>long</td>
   <td class="xl66" colspan="8" style='border-right:none;border-bottom:none;' x:num>97</td>
  </tr>
  <tr height="28" style='height:14.00pt;'>
   <td height="28" style='height:14.00pt;' x:str>doubles</td>
   <td class="xl66" colspan="8" style='border-right:none;border-bottom:none;' x:str>4.8E-322</td>
  </tr>
</table>
** 字节存放次序 **

不同的机器可能采用不同的字节排序方式来存放数据。“big endian” 高位优先将最重要的字节存放在地址最低的存储单元。而 “little endian” 低位优先则是将最重要的字节存放在地址最高的存放单元。当存储了大于一个字节时，像 int、float 等，就要考虑字节的顺序问题了。ByteBuffer 是以高位优先的形式存储数据的，数据在网上传送时也常常采用高位优先的形式。我们可以使用带参数的 ByteOrder.BIG_ENDIAN 或 ByteOrder.LITTLE_ENDIAN 的 order() 方法改变 ByteBuffer 的字节排序方式。

看下面的两个字节的排序方式：
<table width="1536" border="0" cellpadding="0" cellspacing="0" style='width:768.00pt;border-collapse:collapse;table-layout:fixed;'>
   <col width="96" span="16" style='width:48.00pt;'/>
   <tr height="28" style='height:14.00pt;'>
    <td height="28" width="96" align="right" style='height:14.00pt;width:48.00pt;' x:num>0</td>
    <td width="96" align="right" style='width:48.00pt;' x:num>0</td>
    <td width="96" align="right" style='width:48.00pt;' x:num>0</td>
    <td width="96" align="right" style='width:48.00pt;' x:num>0</td>
    <td width="96" align="right" style='width:48.00pt;' x:num>0</td>
    <td width="96" align="right" style='width:48.00pt;' x:num>0</td>
    <td width="96" align="right" style='width:48.00pt;' x:num>0</td>
    <td width="96" align="right" style='width:48.00pt;' x:num>0</td>
    <td width="96" align="right" style='width:48.00pt;' x:num>0</td>
    <td width="96" align="right" style='width:48.00pt;' x:num>1</td>
    <td width="96" align="right" style='width:48.00pt;' x:num>1</td>
    <td width="96" align="right" style='width:48.00pt;' x:num>0</td>
    <td width="96" align="right" style='width:48.00pt;' x:num>0</td>
    <td width="96" align="right" style='width:48.00pt;' x:num>0</td>
    <td width="96" align="right" style='width:48.00pt;' x:num>0</td>
    <td width="96" align="right" style='width:48.00pt;' x:num>1</td>
   </tr>
   <tr height="28" style='height:14.00pt;'>
    <td class="xl65" height="28" colspan="8" style='height:14.00pt;border-right:none;border-bottom:none;' x:str>b1</td>
    <td class="xl65" colspan="8" style='border-right:none;border-bottom:none;' x:str>b2</td>
   </tr>

</table>

如果我们以 short(ByteBuffer.asShortBuffer()) 读取数据，得到的数字是 97(二进制形式 00000000 01100001)；但是如果改为低位优先的形式，仍然以 short 形式读出来，得到的数字是 24832 (二进制形式 01100001 00000000)。

示例：改变字节存放模式来改变字符中的字节顺序：
```java
  public class Endians {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		ByteBuffer byteBuffer = ByteBuffer.wrap(new byte[12]);
		byteBuffer.asCharBuffer().put("abcdef");
		Print.print(Arrays.toString(byteBuffer.array()));

		byteBuffer.rewind();
		byteBuffer.order(ByteOrder.BIG_ENDIAN);
		byteBuffer.asCharBuffer().put("abcdef");
		Print.print(Arrays.toString(byteBuffer.array()));

		byteBuffer.rewind();
		byteBuffer.order(ByteOrder.LITTLE_ENDIAN);
		byteBuffer.asCharBuffer().put("abcdef");
		Print.print(Arrays.toString(byteBuffer.array()));
	}

  }

```
ByteBuffer 有足够的空间来存储作为外部缓冲器的 charArray 中的所有字节，因此可以调用 array() 方法显示视图底层的字节。array() 方法是可选的，并且只能对由数组支持的缓冲器调用此方法；否则将会抛出 UnsupportedOperationException。在底层的字节显示时我们发现随着高低位优先次序的相反，字节的顺序也交换了。

#### 用缓冲器操纵数据
我们想把一个字节数组写入到文件中，那么就应该使用 ByteBuffer.wrap() 方法把字节数组包装起来，然后用 getChannel() 方法在 FileOutputStream 上打开一个通道，接着将来自于 ByteBuffer 的数据写到 FileChannel 中。注意：ByteBuffer 是将数据移进移出通道的唯一方式，并且我们只能创建一个独立的基本类型缓冲器，或者使用 “as” 方法从 ByteBuffer 中获得。

#### 缓冲器的细节
Buffer 由数据和可以高效的访问以及操作这些数据的四个索引组成，这四个索引是：mark (标记)、position (位置)、limit (界限) 和 capacity (容量)。还有一些具体的方法请查看 JDK 的文档。

在缓冲器中插入和提取数据的方法会更新索引，用于反应所发生的变化。下面的示例用一个简单的算法：交换相邻的字符。对 CharBuffer 中的字符进行编码和译码。
```java
  public class UsingBuffers {
	private static void symmetricScramble(CharBuffer buffer){
	    while(buffer.hasRemaining()) {
	    	//将mark 设置为 position
	      buffer.mark();
	      char c1 = buffer.get();
	      char c2 = buffer.get();
	      buffer.reset();
	      buffer.put(c2).put(c1);
	    }
	  }
	  public static void main(String[] args) {
	    char[] data = "UsingBuffers".toCharArray();
	    ByteBuffer bb = ByteBuffer.allocate(data.length * 2);
	    CharBuffer cb = bb.asCharBuffer();
	    cb.put(data);
	    Print.print(cb.rewind());
	    symmetricScramble(cb);
	    Print.print(cb.rewind());
	    symmetricScramble(cb);
	    Print.print(cb.rewind());
	  }
}

```
测试结果：
```
UsingBuffers
sUniBgfuefsr
UsingBuffers
```
尽管可以通过对某个 char 数组调用 warp() 方法直接产生一个 CharBuffer，但是我们在例子中是分配一个底层的 ByteBuffer，产生的 CharBuffer 只是 ByteBuffer 的一个视图而已。这里要强调的是，我们总是以操纵 ByteBuffer 为目标，因为他可以和通道交互。

#### 内存映射文件
内存映射文件允许我们创建和修改那些因为太大而不能放入内存的文件。有了内存映射文件，我们就可以假定整个文件都放在内存中，而且可以完全把他当做非常大的数组来访问。这种方法极大的简化了修改文件的代码。
```java
 public class LargeMappedFiles {
	static int length = 0x8FFFFFF; // 128 MB
	  public static void main(String[] args) throws Exception {
		  //FileChannel.MapMode.READ_WRITE 映射模式
		  //后面两个参数代表从哪里开始到哪里结束。表示我们可以映射一个大文件的较小部分
	    MappedByteBuffer out =new RandomAccessFile("test.dat", "rw").getChannel().map(FileChannel.MapMode.READ_WRITE, 0, length);
	    for(int i = 0; i < length; i++)
	      out.put((byte)'x');
	    Print.print("Finished writing");
	    for(int i = length/2; i < length/2 + 6; i++)
	    	Print.printnb((char)out.get(i));
	  }
}

```
测试结果：
```
 Finished writing
 xxxxxx
```
MappedByteBuffer 继承了 ByteBuffer。因此它具有 ByteBuffer 的所有的方法。前面的程序创建的文件为 128M，这比操作系统一次允许载入的内存大。但是似乎我们可以一次访问到所有文件，因为只有一部分文件放入了内存，文件的其他部分被交换了出去。注意：底层操作系统的文件映射工具是用来最大化的提高性能。

** 性能 **

尽管旧的 I/O 流在用 nio 实现后性能有所提高，但是 “映射文件访问” 往往可以更加显著的提高速度。下面的程序进行了简单的性能比较
```java
 public class MappedIO {
	  private static int numOfInts = 4000000;
	  private static int numOfUbuffInts = 200000;
	  private abstract static class Tester {
	    private String name;
	    public Tester(String name) { this.name = name; }
	    public void runTest() {
	      System.out.print(name + ": ");
	      try {
	        long start = System.nanoTime();
	        test();
	        double duration = System.nanoTime() - start;
	        System.out.format("%.2f\n", duration/1.0e9);
	      } catch(IOException e) {
	        throw new RuntimeException(e);
	      }
	    }
	    public abstract void test() throws IOException;
	  }
	  private static Tester[] tests = {
	    new Tester("Stream Write") {
	      public void test() throws IOException {
	        DataOutputStream dos = new DataOutputStream(
	          new BufferedOutputStream(
	            new FileOutputStream(new File("temp.tmp"))));
	        for(int i = 0; i < numOfInts; i++)
	          dos.writeInt(i);
	        dos.close();
	      }
	    },
	    new Tester("Mapped Write") {
	      public void test() throws IOException {
	        FileChannel fc =
	          new RandomAccessFile("temp.tmp", "rw")
	          .getChannel();
	        IntBuffer ib = fc.map(
	          FileChannel.MapMode.READ_WRITE, 0, fc.size())
	          .asIntBuffer();
	        for(int i = 0; i < numOfInts; i++)
	          ib.put(i);
	        fc.close();
	      }
	    },
	    new Tester("Stream Read") {
	      public void test() throws IOException {
	        DataInputStream dis = new DataInputStream(
	          new BufferedInputStream(
	            new FileInputStream("temp.tmp")));
	        for(int i = 0; i < numOfInts; i++)
	          dis.readInt();
	        dis.close();
	      }
	    },
	    new Tester("Mapped Read") {
	      public void test() throws IOException {
	        FileChannel fc = new FileInputStream(
	          new File("temp.tmp")).getChannel();
	        IntBuffer ib = fc.map(
	          FileChannel.MapMode.READ_ONLY, 0, fc.size())
	          .asIntBuffer();
	        while(ib.hasRemaining())
	          ib.get();
	        fc.close();
	      }
	    },
	    new Tester("Stream Read/Write") {
	      public void test() throws IOException {
	        RandomAccessFile raf = new RandomAccessFile(
	          new File("temp.tmp"), "rw");
	        raf.writeInt(1);
	        for(int i = 0; i < numOfUbuffInts; i++) {
	          raf.seek(raf.length() - 4);
	          raf.writeInt(raf.readInt());
	        }
	        raf.close();
	      }
	    },
	    new Tester("Mapped Read/Write") {
	      public void test() throws IOException {
	        FileChannel fc = new RandomAccessFile(
	          new File("temp.tmp"), "rw").getChannel();
	        IntBuffer ib = fc.map(
	          FileChannel.MapMode.READ_WRITE, 0, fc.size())
	          .asIntBuffer();
	        ib.put(0);
	        for(int i = 1; i < numOfUbuffInts; i++)
	          ib.put(ib.get(i - 1));
	        fc.close();
	      }
	    }
	  };
	  public static void main(String[] args) {
	    for(Tester test : tests)
	      test.runTest();
	  }
}

```
测试结果：
```
Stream Write: 0.28
Mapped Write: 0.02
Stream Read: 0.43
Mapped Read: 0.01
Stream Read/Write: 3.24
Mapped Read/Write: 0.01
```
test() 包含初始化各种 I/O 对象的时间，因此，即使建立映射文件的花费很大，但是整体收益比起 I/O 流来说还是很显著的。

#### 文件加锁
JDK 1.4 引入了文件加锁的机制，它允许我们同步访问某个文件作为共享资源的文件。不过，竞争同一文件的两个线程可能在不同的 Java 虚拟机上；或者一个是 Java 线程，另一个是操作系统中的某个本地线程。文件加锁对操作系统进程是可见的，因为 Java 的文件加锁直接映射到操作系统的加锁工具。

下面是一个文件加锁的简单示例：
```java
 public class FileLocking {

	public static void main(String[] args) throws Exception{
		// TODO Auto-generated method stub
		FileOutputStream fOutputStream = new FileOutputStream("file.txt");
		FileLock fLock = fOutputStream.getChannel().tryLock();
		if (fLock != null) {
			System.out.println("加锁文件");
			TimeUnit.MILLISECONDS.sleep(100);
			fLock.release();
			System.out.println("解锁");
		}
		fOutputStream.close();
	}

}

```
测试结果：
```
加锁文件
解锁
```
通过对 FileChannel 调用 tryLock() 或者 lock()，就可以获得整个文件的 FileLock。tryLock() 是非阻塞式的，它设法获取锁，但是如果不能获取将直接从方法调用处返回。lock() 则是阻塞式的，他要阻塞进程直至锁可以获得，或调用 lock() 的线程中断，或调用 lock() 的通道关闭。使用 FileLock.release() 可以释放锁。

也可以使用如下方法对文件的一部分上锁：
```java
 tryLock(long position,long size,boolean shard)
```
或者：
```java
lock(long position,long size,boolean shard)
```
其中加锁的区域是由 position 和 size 决定的。第三个参数指定是否共享锁。

无参数的加锁方法会根据文件的大小变化而变化，但是具有固定尺寸的锁不会随着文件的大小变化。无参数的加锁方式是对整个文件进行加锁。对独占锁或者共享锁需要操作系统底层的支持。如果操作系统不支持共享锁，那么会为每一个请求创建一个锁，那么就会成为独占锁。锁的类型可以通过 FileLock.isShared() 进行查询。

** 映射文件的部分加锁 **

文件映射通常应用于极大的文件。我们可能需要对这种巨大的文件进行部分加锁，一遍其他进程可以修改文件中未被加锁的部分。例如：数据库就是这样，因此多个用户可以同时访问他。

下面例子两个线程，分别加锁文件的不同部分。
```java
 public class LockingMappedFiles {
	static final int LENGTH = 0x8FFFFFF; // 128 MB
	  static FileChannel fc;
	  public static void main(String[] args) throws Exception {
	    fc =new RandomAccessFile("test.dat", "rw").getChannel();
	    MappedByteBuffer out =fc.map(FileChannel.MapMode.READ_WRITE, 0, LENGTH);
	    for(int i = 0; i < LENGTH; i++)
	      out.put((byte)'x');
	    new LockAndModify(out, 0, 0 + LENGTH/3);
	    new LockAndModify(out, LENGTH/2, LENGTH/2 + LENGTH/4);
	  }
	  private static class LockAndModify extends Thread {
	    private ByteBuffer buff;
	    private int start, end;
	    LockAndModify(ByteBuffer mbb, int start, int end) {
	      this.start = start;
	      this.end = end;
	      mbb.limit(end);
	      mbb.position(start);
	      buff = mbb.slice();
	      start();
	    }
	    public void run() {
	      try {
	        // Exclusive lock with no overlap:
	        FileLock fl = fc.lock(start, end, false);
	        System.out.println("Locked: "+ start +" to "+ end);
	        // Perform modification:
	        while(buff.position() < buff.limit() - 1)
	          buff.put((byte)(buff.get() + 1));
	        fl.release();
	        System.out.println("Released: "+start+" to "+ end);
	      } catch(IOException e) {
	        throw new RuntimeException(e);
	      }
	    }
	  }
}

```
测试结果：
```
Locked: 75497471 to 113246206
Locked: 0 to 50331647
Released: 75497471 to 113246206
Released: 0 to 50331647
```
线程类创建了缓冲区和用于修改的 slice()，然后在 run() 中获得文件通道上的锁。我们不能获得缓冲器上的锁，只能是通道上的。lock() 类似于获得一个对象的线程锁，即对该部分文件具有独占访问权。
