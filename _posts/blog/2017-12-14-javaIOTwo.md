---
layout: post
title: java编程思想之 I/O 系统(典型使用方式)
categories: javaThinking
description: java编程思想之I/O 系统详解
keywords: java, javaI/O 系统,java 输入输出流，java I/O
---

可以通过不同的方式组合 I/O 流，但我们可能也就只用到其中的几种组合。下面的一些例子可以作为典型的 I/O 操作参考。

#### 缓冲输入文件
如果要打开一个文件读取字符，可以使用以 String 或 File 对象作为文件名的 FileInputReader。为了提高速度，我们希望对那个文件进行缓冲，那么我们将所产生的引用传递给一个 BufferedReader 构造器。由于 BufferedReader 也提供了 readLine() 方法，所以 BufferedReader 就是我们最终使用的对象。当 readLine() 返回 null 时，你就达到了文件的末尾。
```java
public class BufferedInputFile {

	public static String read(String filename) {
		StringBuilder builder = null;
		try {
			BufferedReader bReader = new BufferedReader(new FileReader(filename));
			String string;
			builder = new StringBuilder();
			while ((string = bReader.readLine())!= null) {
				builder.append(string+"\n");
			}
			bReader.close();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		return builder.toString();

	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		System.out.print(read("D:/eclipseWorkspace/ThreadTest/src/ioone/BufferedInputFile.java"));
	}

}

```
字符串 builder 用来累计文件的全部内容。最后，调用 close() 关闭文件。

#### 从内存输入
下面的示例中，从 BufferedInputFile.read() 读入的 String 结果被用来创建一个 StringReader。然后调用 read() 每次读取一个字符，并把它发送到控制台。
```java
public class MemoryInput {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		StringReader sReader = new StringReader(BufferedInputFile.read("D:/eclipseWorkspace/ThreadTest/src/ioone/MemoryInput.java"));
		int c;
		try {
			while ((c = sReader.read()) != -1) {
				System.out.print((char)c);
			}
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}

}

```
注意 read() 是以 int 形式返回下一个字节，因此必须使用类型转换为 char 才能正确打印。

#### 格式化的内存输入
要读取格式化的数据，可以使用 DataInputStream，它是一个面向字节的 I/O 类。因此我们使用 InputStream 类而不是 Reader 类。当然，我们可以用 InputStream 以字节的形式读取任何数据，不过，我们在这里使用的是字符串而已。
```java
public class FormattedMemoryInput {

	public static void main(String[] args) {
		DataInputStream inputStream = new DataInputStream(new ByteArrayInputStream(
				new BufferedInputFile().read("D:/eclipseWorkspace/ThreadTest/src/ioone/FormattedMemoryInput.java").getBytes()));
		while (true) {
			try {
				System.out.print((char) inputStream.readByte());
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
				return;
			}

		}

	}

}

```
必须为 ByteArrayInputStream 提供字节数组，为了产生该数组 String 包含了一个实现此工作的 getBytes() 方法。所产生的 ByteArrayInputStream 是一个适合传递给 DataInputStream 的 InputStream。如果我们从 DataInputStream 用 readByte() 一次一个字节的读取数据，那么任何字节都将产生合法的结果，因此返回值不能用来检测输入的值是否是合法的。我们看到代码中我在 catch 语句中加了一个 return。那是因为我们需要在读取不到字节的时候结束掉循环。否则将会死循环下去。我们可以使用 available() 方法查看还有多少可供存取的字符。下面的例子演示了怎样一次一个字节的读取文件：
```java
public class TestEOF {

	public static void main(String[] args) {
		try {
			DataInputStream inputStream = new DataInputStream(new BufferedInputStream(new FileInputStream("D:/eclipseWorkspace/ThreadTest/src/ioone/TestEOF.java")));
			try {
				//可以判断是否读取到最后一个字节
				while (inputStream.available() !=0) {
					System.out.print((char)inputStream.readByte());
				}
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		} catch (FileNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

	}

}

```
注意：available() 的工作方式会随着所读取的媒介的不同而有所不同；字面意思是 “在没有阻塞的情况下所能读取的字节数”。对于文件，这意味着整个文件；但是对于不同的流，可能就不是这样的，所以要谨慎使用。也可以像我上一个例子哪样通过捕获异常来检测输入的末尾。但是，这样是对异常流的错误使用。

#### 基本的文件输出
FileWriter 对象可以向文件写入数据。首先，创建一个与指定文件链接的 FileWriter。实际上，我们通常会使用 BufferedWriter 将其包装起来用以缓冲输出。
```java
public class BasicFileOutput {
	static String fiString = "BasicFileOutput.out";

	public static void main(String[] args) {
		BufferedReader iReader = new BufferedReader(new StringReader(BufferedInputFile.read("D:/eclipseWorkspace/ThreadTest/src/ioone/BasicFileOutput.java")));
		try {
			PrintWriter out = new PrintWriter(new BufferedWriter(new FileWriter(fiString)));
			int lineCount = 1;
			String string;
			while ((string = iReader.readLine()) != null) {
				//加行号一起写入
				out.println(lineCount++ + ":" + string);
			}
			out.close();
			System.out.println(BufferedInputFile.read(fiString));
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

	}

}

```
当文本被写入时，行号就会增加。一旦，读完输入数据流，readLine() 会返回 null。我们用 out 显式的调用 close()。如果我们不为所有的输出文件调用 close()，就会看到缓冲区内容不会被刷新清空。

```文本文件的快捷方式```

Java SE5 在 PrintWriter 中添加了一个辅助的构造器，使得你不必在每次希望创建文本文件向其中写入时，都去执行所有的装饰工作。
```java
public class FiltOutputShortcut {
	static String fString = "FiltOutputShortcut.out";
	public static void main(String[] args) {
		BufferedReader iReader = new BufferedReader(new StringReader(BufferedInputFile.read("D:/eclipseWorkspace/ThreadTest/src/ioone/FiltOutputShortcut.java")));
		try {
			//仍旧使用的是缓冲，只是不必自己去实现
			PrintWriter printWriter = new PrintWriter(fString);
			int lineCount =1;
			String string;
			try {
				while ((string = iReader.readLine()) != null) {
					printWriter.println(lineCount++ + ":"+string);
				}
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			printWriter.close();
			System.out.println(BufferedInputFile.read(fString));
		} catch (FileNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

	}

}

```
其他创建的写入任务都没有快捷方式。

#### 存储和恢复数据
PrintWriter 可以对数据进行格式化，以便我们来阅读。但是为了提供一个流来恢复数据，我们需要使用 DataOutputStream 写入数据，并用 DataInputStream 恢复数据。下面的示例使用一个文件，并且对于读和写都进行了缓冲处理。
```java
public class StoringAndRecoveringData {

	public static void main(String[] args) throws IOException {
		DataOutputStream outputStream = new DataOutputStream(new BufferedOutputStream(new FileOutputStream("D:/Data.txt")));
		outputStream.writeDouble(3.1415926);
		outputStream.writeUTF("That was pi");
		outputStream.writeDouble(1.415926);
		outputStream.writeUTF("Square root of 2");
		outputStream.close();

		DataInputStream inputStream = new DataInputStream(new BufferedInputStream(new FileInputStream("D:/Data.txt")));
		System.out.println(inputStream.readDouble());
		System.out.println(inputStream.readUTF());
		System.out.println(inputStream.readDouble());
		System.out.println(inputStream.readUTF());
		inputStream.close();


	}

}

```
执行结果：
```
3.1415926
That was pi
1.415926
Square root of 2
```
如果我们使用 DataOutputStream 写入数据，Java 保证我们可以使用 DataInputStream 读取数据，无论读写数据的平台多么不同。只要两个平台上都有 Java 就不用花费大量的时间去处理不同平台数据的问题。

当我们使用 DataOutputStream 写字符串并且让 DataInputStream 能够恢复的唯一可靠做法是使用了 UTF-8 编码，在这个示例中是用 writeUTF() 和 readUTF() 来实现的。writeDouble() 讲 double 类型的数字存储到流中，并用相应的 readDouble() 恢复他。但是为了保证所有的读写方法都能够正常的工作，我们必须知道流数据项所在的确切位置，因为极有可能将 double 数据作为一个简单的字节序列读入。因此，我们必须：要嘛为文件的数据采用固定的格式；要嘛将额外的信息保存到文件中，以便能够对其进行解析已确定数据的存储位置。注意：对象序列化和 XML 可能更容易的存储和读取复杂数据结构的方式。

#### 读写随机访问文件
使用 RandomAccessFile，类似于组合使用 DataOutputStream 和 DataInputStream。我们可以看到使用 seek() 可以在文件中到处移动，并修改文件中的某个值。在使用 RandomAccessFile 你必须知道文件的排版，这样才能正确的操作它。RandomAccessFile 拥有读取基本类型和 UTF-8 字符串的各种具体方法。
```java
public class UsingRandomAccessFile {
	static String file = "rtest.dat";
	public static void display() throws IOException{
		RandomAccessFile accessFile = new RandomAccessFile(file,"r");
		for (int i = 0; i < 7; i++) {
			System.out.println("Value" + i + ":"+accessFile.readDouble());
		}

		System.out.println(accessFile.readUTF());
		accessFile.close();

	}
	public static void main(String[] args)throws IOException {
		// TODO Auto-generated method stub
		RandomAccessFile rFile = new RandomAccessFile(file, "rw");
		for (int i = 0; i <7; i++) {
			rFile.writeDouble(i * 1.414);
		}
		rFile.writeUTF("The end of the file");
		rFile.close();
		display();
		rFile = new RandomAccessFile(file, "rw");
		rFile.seek(5*8);
		rFile.writeDouble(47.0001);
		rFile.close();
		display();
	}

}

```
执行结果：
```
Value0:0.0
Value1:1.414
Value2:2.828
Value3:4.242
Value4:5.656
Value5:7.069999999999999
Value6:8.484
The end of the file
Value0:0.0
Value1:1.414
Value2:2.828
Value3:4.242
Value4:5.656
Value5:47.0001
Value6:8.484
The end of the file
```
因为 double 总是 8 个字节，所以使用了 seek() 查找第五个双精度值，你只需要用 5*8 来产生查找位置。

RandomAccessFile 除了实现 DataInput 和 DataOutput 接口之外，有效的与 I/O 类的继承层次实现了分离。因为它不支持装饰，所以不能将其与 InputStream 和 OutputStream 子类的任何部分组合起来。我们必须假定 RandomAccessFile 已被正确的缓冲，因为我们不能为它添加这样的功能。

可以自行选择的是第二个构造参数：我们可以指定 “只读” ```r``` 方式或者 “读写” ```rw``` 方式打开一个 RandomAccessFile 文件。

#### 管道流
PipedInputStream、PipedOutputStream、PipedReader、PipedWrited 在本简单的提高。不代表他们没有用处，他们的价值会在我们后边的多线程中体现。因为管道流用于任务之间的通讯。

## 文件读写的实用工具
一个很常见的程序任务就是读取文件到内存，修改，然后在写出。Java I/O 类库的问题是：他需要编写很多的代码去执行这些常用的操作，没有未我们提供一些基本的帮助功能。更糟糕的是类型太多，要记住如何正确的打开文件是很困难的。因此我们添加一下帮助类是相当有意义的，这样就可以很容易的帮助我们完成基本任务。Java SE5 在 PrintWriter 中添加了很方便的构造器。但是，还有许多很常见的操作需要我们反复的执行，这就使得消除与这些任务相关联的代码就显得有意义了。

下面的 TextFile 类在本书前面的示例中就已经被用来简化对文件的读写操作。
```java
public class TextFile extends ArrayList<String>{
	public static String read(String fileName) {
	    StringBuilder sb = new StringBuilder();
	    try {
	      BufferedReader in= new BufferedReader(new FileReader(new File(fileName).getAbsoluteFile()));
	      try {
	        String s;
	        while((s = in.readLine()) != null) {
	          sb.append(s);
	          sb.append("\n");
	        }
	      } finally {
	        in.close();
	      }
	    } catch(IOException e) {
	      throw new RuntimeException(e);
	    }
	    return sb.toString();
	  }
	  // Write a single file in one method call:
	  public static void write(String fileName, String text) {
	    try {
	      PrintWriter out = new PrintWriter( new File(fileName).getAbsoluteFile());
	      try {
	        out.print(text);
	      } finally {
	        out.close();
	      }
	    } catch(IOException e) {
	      throw new RuntimeException(e);
	    }
	  }
	  // Read a file, split by any regular expression:
	  public TextFile(String fileName, String splitter) {
	    super(Arrays.asList(read(fileName).split(splitter)));
	    // Regular expression split() often leaves an empty
	    // String at the first position:
	    if(get(0).equals("")) remove(0);
	  }
	  // Normally read by lines:
	  public TextFile(String fileName) {
	    this(fileName, "\n");
	  }

	  public void write(String fileName) {
	    try {
	      PrintWriter out = new PrintWriter(new File(fileName).getAbsoluteFile());
	      try {
	        for(String item : this)
	          out.println(item);
	      } finally {
	        out.close();
	      }
	    } catch(IOException e) {
	      throw new RuntimeException(e);
	    }
	  }
	  // Simple test:
	  public static void main(String[] args) {
	    String file = read("D:/eclipseWorkspace/ThreadTest/src/ioone/TextFile.java");
	    write("test.txt", file);
	    TextFile text = new TextFile("test.txt");
	    text.write("test2.txt");
	    // Break into unique sorted list of words:
	    TreeSet<String> words = new TreeSet<String>(new TextFile("D:/eclipseWorkspace/ThreadTest/src/ioone/TextFile.java", "\\W+"));
	    // Display the capitalized words:
	    System.out.println(words.headSet("a"));
	  }
}

```
执行结果：
```
[0, ArrayList, Arrays, Break, BufferedReader, D, Display, File, FileReader, IOException, Normally, PrintWriter, Read, Regular, RuntimeException, Simple, String, StringBuilder, System, TextFile, ThreadTest, TreeSet, W, Write]

```
read() 将每行添加到 StringBuffer，并且为每行加上换行符，因为在读的过程中换行符会被去掉。接着返回一个包含整个文件的字符串。write() 打开文本并将其写入文件。在这两个方法完成时，都要记着用 close() 关闭文件。

注意在任何打开文件的代码在 finally 子句中，作为防卫措施都添加了对文件的 close() 调用，以保证文件会被正确的关闭。

#### 读取二进制文件
这个工具简化了读取二进制文件的过程。
```java
public class BinaryFile {
	public static byte[] read(File file) throws IOException {
		BufferedInputStream inputStream = new BufferedInputStream(new FileInputStream(file));
		try {
			byte[] data = new byte[inputStream.available()];
			inputStream.read(data);
			return data;
		} finally{
			inputStream.close();
		}

	}

	public static byte[] read(String files) throws IOException {
		return read(new File(files).getAbsoluteFile());
	}
}

```
其中一个方法接受 File 参数，第二个重载方法接受表示文件名的 String 参数。这两个方法都返回产生的 byte 数组。available() 方法被用来产生恰当的数组尺寸，并且 read() 方法的特定的重载版本填充了数组。

## 标准 I/O 流
标准 I/O 流术语参考的是 Unix 中 “程序所使用的单一信息流” 这个概念。程序的所有输入都可以来自于标准输入，它的所有输出都可以发送到标准输出，以及所有的错误信息都可以发送到标准错误。标准 I/O 的意义在于：我们可以很容易的把程序串联起来，一个程序的标准输出可以成为另一个程序的标准输入。

#### 从标准输入中读取
标准 I/O 模型，java 提供了 System.out、System.in、System.err。我们已经看到了 System.out、System.err 的使用。out 和 err 都被事先包装成了 PrintStream，但是 System.in 确实没有包装过的 InputStream。意味着我们读取 System.in 之前必须对其进行包装。

通常我们会用 readLine() 一次一行的读取输入，为此，我们将 System.in 换成 Reader。下面的例子直接显示你所输入的每一行。
```java
public class Echo {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		BufferedReader std = new BufferedReader(new InputStreamReader(System.in));
		String string;
		try {
			while ((string = std.readLine()) != null && string.length() != 0) {
				System.out.println(string);
			}
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}

}

```
执行结果：
```
aa
aa
```
注意：Syatem.in 和大多数流一样，通常应该对它进行缓冲。

#### 将 System.out 转换成 PrintWriter
System.out 是一个 PrintStream，而 PrintStream 是一个 OutputStream。PrintWrite 有一个可以接受 OutputStrean 作为参数的构造器。因此只要需要我们就可以将 Syatem.out 转换成 PrintWriter：
```java
public class ChangeSystemOut {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		PrintWriter out = new PrintWriter(System.out,true);
		out.println("hello,world");
	}

}

```
测试结果：
```
hello,world
```
重要的是使用两个参数的构造器，并将第二个参数设置为 true，以便开启自动清空功能；否则，你可能看不到输出。

#### 标准 I/O 重定向
Java 的 System 类提供了一些简单的静态方法调用，以允许我们对标准输入，输出和错误流进行重定向：
- setIn(InputStream)
- setOut(PrintStream)
- setErr(PrintStream)

如果我们突然开始在显示器上创建大量输出，滚动的速度太快又看不清楚。重定向输出就显得极为重要。下面简单演示了这些方法的使用：
```java
public class Redirecting {

	public static void main(String[] args) throws IOException{
		// TODO Auto-generated method stub
		PrintStream console = System.out;
		BufferedInputStream inputStream = new BufferedInputStream(new FileInputStream("D:/eclipseWorkspace/ThreadTest/src/ioone/Redirecting.java"));
		PrintStream out = new PrintStream(new BufferedOutputStream(new FileOutputStream("test.out")));
		System.setIn(inputStream);
		System.setOut(out);
		System.setErr(out);
		BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
		String string;
		while ((string = reader.readLine()) != null) {
			System.out.print(string);
		}
		out.close();
		System.setOut(console);
	}

}

```
这里将程序的输出转移到 test.out 这个文件上。注意：程序开头部分存储了对最初的 System.out 对象的引用，并且在结尾处将系统输出恢复到该对象上。I/O 重定向操作的是字节流而不是字符流；因为我们使用的是 InputStream 和 OutputStream。

## 进程控制
我们经常需要在 Java 内部执行其他操作系统的程序，并且要控制这些程序的输入和输出。Java 类库提供了这些操作的类。一项常见的任务是运行程序，并将产生的输出发送到控制台。下面展示一个简化这项任务的工具。使用工具时可能产生两种类型的错误：普通的异常错误我们需要抛出一个运行时异常，以及进程自身执行过程中产生的错误，我们希望用单独的异常来报告这些错误。
```java
public class OSExecuteException extends RuntimeException {
	  public OSExecuteException(String why) { super(why); }
}

```
要想运行一个程序，你需要向 OSExecute.command() 传递一个 command 字符串，它与你在控制台上运行该程序键入的命令是相同的。这个命令被传递给 java.lang.ProcessBuilder 构造器。它要求这个命令作为一个 String 对象序列而被传递，然后所产生的 ProcessBuilder 对象被启动。
```java
public class OSExecute {
	public static void command(String command) {
	    boolean err = false;
	    try {
	      Process process = new ProcessBuilder(command.split(" ")).start();
	      BufferedReader results = new BufferedReader(new InputStreamReader(process.getInputStream()));
	      String s;
	      while((s = results.readLine())!= null)
	        System.out.println(s);
	      BufferedReader errors = new BufferedReader( new InputStreamReader(process.getErrorStream()));
	      // Report errors and return nonzero value
	      // to calling process if there are problems:
	      while((s = errors.readLine())!= null) {
	        System.err.println(s);
	        err = true;
	      }
	    } catch(Exception e) {
	      // Compensate for Windows 2000, which throws an
	      // exception for the default command line:
	      if(!command.startsWith("CMD /C"))
	        command("CMD /C " + command);
	      else
	        throw new RuntimeException(e);
	    }
	    if(err)
	      throw new OSExecuteException("Errors executing " +
	        command);
	  }
}

```
为了捕获程序执行时产生的标准输出流，需要调用 getInputStream(),这是因为 InputStream 是我们可以读取信息的流。从程序中产生的结果每次输出一行，因此要使用 readLine() 来读取。改程序的错误被发送到标准错误流，并且通过调用 getErrorStream() 得以捕获。如果存在任何错误，它们都会被打印并且抛出 OSExecuteException，因此调用程序需要处理这个问题。

下面展示如何使用：
```java
public class OSExecuteDemo {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		OSExecute.command("javap OSExecuteDemo.java");
	}

}

```
反编译这个类。
