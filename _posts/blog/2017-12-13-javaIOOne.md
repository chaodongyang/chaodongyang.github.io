---
layout: post
title: java编程思想之 I/O 系统(File简介)
categories: javaThinking
description: java编程思想之I/O 系统详解
keywords: java, javaI/O 系统,java 输入输出流，java I/O
---
创建一个好的输入/输出系统是比较困难的。因为我们不仅要解决不同平台之间的通讯，还要解决不同的通信方式 (顺序、随机存取、缓冲、二进制、按行、按字等等)。Java 类库的设计者通过设计大量的类来帮助我们解决这个问题。从 Java 1.0 版本开始，Java 的 I/O 类库就发生了很大的变化，在原来面向字节的类库中添加了面向字符和基于 Unicode 的类。在 JDK 4.0 中，添加了 nio 类，添加进来是为了改进性能和功能。因此，我们在充分理解 Java I/O 系统以便正确运行之前，我们需要学习相当数量的类。另外，我们还需要理解 I/O 类库的演化史，如果缺乏历史的了解很快我们就会忘记该什么时候使用什么样的类。因为实在太多了会让人感到迷惑。

## File 类
学习使用流来读写数据之前我们先了解一个实用的类库，可以帮助我们处理文件目录的问题。

File(文件) 并非单单指代的是文件。它既能代表一个特定文件的名称，又能代表一个目录下一组文件的名称。如果它指的是一个文件集，我们可以调用 list() 方法，这个方法返回一个字符的数组。为什么不返回的是一个容器类呢？因为目录元素的个数是固定的。如果我们想取得不同的目录列表，只需要在创建出一个不同的 File 对象即可。

#### 目录列表器
假设我们想要查看一个目录下的文件列表，可以有两种方法来使用 File 对象。如果我们调用了不带参数的 list() 方法，便可以获取此 File 对象包含的所有列表。然而，我们想要获取一个受限制的列表。比如，获取扩展名为 .java 的文件，那么就需要使用目录过滤器，这个类会告诉我们如何显示符合条件的 File 对象。

下面是一个示例：
```java
public class DirList {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		File file = new File(".");
		String[] list = null;

		list = file.list(new DirFilter(args[0]));
		//对数组进行排序
		Arrays.sort(list,String.CASE_INSENSITIVE_ORDER);
		for (String string : list) {
			System.out.print(string+"  ");
		}

	}

	static class DirFilter implements FilenameFilter{

		private Pattern pattern;

		public DirFilter(String regex) {
			pattern = Pattern.compile(regex);
		}

		@Override
		public boolean accept(File dir, String name) {
			// TODO Auto-generated method stub
			return pattern.matcher(name).matches();
		}

	}

}

```
这里 DirFilter 实现了 FilenameFilter 接口：
```java
@FunctionalInterface
public interface FilenameFilter {
    /**
     * Tests if a specified file should be included in a file list.
     *
     * @param   dir    the directory in which the file was found.
     * @param   name   the name of the file.
     * @return  <code>true</code> if and only if the name should be
     * included in the file list; <code>false</code> otherwise.
     */
    boolean accept(File dir, String name);
}

```
DirFilter 的目的是在于把 accept() 方法提供给 list() 使用，使 list() 可以回调 accept()，进而确定哪些文件包含在列表中。这种结构常常称之为回调。

accept() 方法必须接收一个代表某个特定文件所在目录的 File 对象，以及包含了那个文件名称的一个 String。list() 方法会在此目录对象下的每个文件名调用 accept()，来判断文件是否包含在内；判断结果由 accept() 返回的 booelan 值来表示。我们使用一个正则表达式的 matcher 对象，来查看此正则表达式 regex 是否匹配这个文件的名称。

#### 匿名内部类
我们使用一个匿名内部类来修改上面的例子：
```java
public class DirList2 {

	public static FilenameFilter filter(final String regex) {
		return new FilenameFilter() {
			private Pattern pattern = Pattern.compile(regex);

			@Override
			public boolean accept(File dir, String name) {
				// TODO Auto-generated method stub
				return pattern.matcher(name).matches();
			}
		};
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		File file = new File(".");
		String[] strings;
		strings = file.list(filter(args[0]));
		Arrays.sort(strings,String.CASE_INSENSITIVE_ORDER);
		for (String string : strings) {
			System.out.println(string);
		}
	}

}

```
注意：传向 filter() 的参数必须是 final。这是匿名内部类必须的，这样才能够使用来自类范围之外的对象。

为了使得程序变得更小一点，我们再次修改这个方法。把匿名内部类写进方法。
```java
public class DirList3 {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		File file = new File(".");
		String[] list = file.list(new FilenameFilter() {
			private Pattern pattern = Pattern.compile(args[0]);
			@Override
			public boolean accept(File dir, String name) {
				// TODO Auto-generated method stub
				return pattern.matcher(name).matches();
			}
		});

		Arrays.sort(list,String.CASE_INSENSITIVE_ORDER);
		for (String string : list) {
			System.out.println(string);
		}
	}

}

```
这个例子展示了匿名内部类如何通过创建特定的、一次性的类来解决问题。此方法的一个优点就是将解决特定问题的代码隔离、聚拢于一点。而另一方面这种方法却不易阅读，因此要谨慎使用。

#### 目录实用工具
程序中最常见的任务是在文件集上执行操作，这些文件要嘛在本地目录，要嘛遍布于整个目录书中。如果提供一种工具为我们产生这个文件集非常的有用。下面的实用工具类可以通过 local() 方法产生由本地目录中的文件构成的 File 对象数组，或者通过使用 walk() 方法产生给定目录下由整个目录树中所有文件构成的 List<File>。这些文件是根据你提供的正则表达式来确定的。
```java
public final class Directory {
	public static File[]
			  local(File dir, final String regex) {
			    return dir.listFiles(new FilenameFilter() {
			      private Pattern pattern = Pattern.compile(regex);
			      public boolean accept(File dir, String name) {
			        return pattern.matcher(
			          new File(name).getName()).matches();
			      }
			    });
			  }
			  public static File[]
			  local(String path, final String regex) { // Overloaded
			    return local(new File(path), regex);
			  }
			  // A two-tuple for returning a pair of objects:
			  public static class TreeInfo implements Iterable<File> {
			    public List<File> files = new ArrayList<File>();
			    public List<File> dirs = new ArrayList<File>();
			    // The default iterable element is the file list:
			    public Iterator<File> iterator() {
			      return files.iterator();
			    }
			    void addAll(TreeInfo other) {
			      files.addAll(other.files);
			      dirs.addAll(other.dirs);
			    }
			    public String toString() {
			      return "dirs: " + PPrint.pformat(dirs) +
			        "\n\nfiles: " + PPrint.pformat(files);
			    }
			  }
			  public static TreeInfo
			  walk(String start, String regex) { // Begin recursion
			    return recurseDirs(new File(start), regex);
			  }
			  public static TreeInfo
			  walk(File start, String regex) { // Overloaded
			    return recurseDirs(start, regex);
			  }
			  public static TreeInfo walk(File start) { // Everything
			    return recurseDirs(start, ".*");
			  }
			  public static TreeInfo walk(String start) {
			    return recurseDirs(new File(start), ".*");
			  }
			  static TreeInfo recurseDirs(File startDir, String regex){
			    TreeInfo result = new TreeInfo();
			    for(File item : startDir.listFiles()) {
			      if(item.isDirectory()) {
			        result.dirs.add(item);
			        result.addAll(recurseDirs(item, regex));
			      } else // Regular file
			        if(item.getName().matches(regex))
			          result.files.add(item);
			    }
			    return result;
			  }
			  // Simple validation test:
			  public static void main(String[] args) {
			    if(args.length == 0)
			      System.out.println(walk("."));
			    else
			      for(String arg : args)
			       System.out.println(walk(arg));
			  }
}

```
这些工具类都是在 Java 编程思想工具类中提供了。下面是上面用到的一个格式化的打印工具：
```java
public class PPrint {
	public static String pformat(Collection<?> c) {
	    if(c.size() == 0) return "[]";
	    StringBuilder result = new StringBuilder("[");
	    for(Object elem : c) {
	      if(c.size() != 1)
	        result.append("\n  ");
	      result.append(elem);
	    }
	    if(c.size() != 1)
	      result.append("\n");
	    result.append("]");
	    return result.toString();
	  }
	  public static void pprint(Collection<?> c) {
	    System.out.println(pformat(c));
	  }
	  public static void pprint(Object[] c) {
	    System.out.println(pformat(Arrays.asList(c)));
	  }
}
```
这些工具都存放在 net.mindview.util 包中。你可以到编程思想这本书作者的官网下载。或者我在这本书笔记完结之后也会提供给大家。下面的例子展示我们应该如何使用它：
```java
public class DirectoryDemo {
	public static void main(String[] args) {
	    // All directories:
	    PPrint.pprint(Directory.walk(".").dirs);
	    // All files beginning with 'T'
	    for(File file : Directory.local(".", "T.*"))
	      System.out.println(file);
	    System.out.println("----------------------");
	    // All Java files beginning with 'T':
	    for(File file : Directory.walk(".", "T.*\\.java"))
	    	System.out.println(file);
	    System.out.println("======================");
	    // Class files containing "Z" or "z":
	    for(File file : Directory.walk(".",".*[Zz].*\\.class"))
	    	System.out.println(file);
	  }
}

```
内容太长，只展示部分的输出结果：
```
.\bin\array\ParameterizedArrayType.class
.\bin\containerstwo\Synchronization.class
.\bin\exceptionDemo\Sneeze.class
.\bin\match\ReplacingStringTokenizer.class
.\bin\match\ThreatAnalyzer.class
.\bin\type\ClassInitialization.class
.\bin\wildcard\FixedSizeStack.class
```
#### 目录的检查及创建
File 类不仅仅代表存在的文件或目录。也可以用 File 对象创建新的目录或尚不存在的整个目录路径。还可以用它查看文件的特性，检查某个 File 对象代表的是一个文件还是一个目录，还可以删除文件。

下面的示例展示了 File 类的基本用法：
```java
public class MakeDirectories {

	public static void usage() {
		 System.err.println(
			      "Usage:MakeDirectories path1 ...\n" +
			      "Creates each path\n" +
			      "Usage:MakeDirectories -d path1 ...\n" +
			      "Deletes each path\n" +
			      "Usage:MakeDirectories -r path1 path2\n" +
			      "Renames from path1 to path2");
			    System.exit(1);
	}

	public static void fileData(File f) {
		System.out.println( "Absolute path: " + f.getAbsolutePath() +
			      "\n Can read: " + f.canRead() +
			      "\n Can write: " + f.canWrite() +
			      "\n getName: " + f.getName() +
			      "\n getParent: " + f.getParent() +
			      "\n getPath: " + f.getPath() +
			      "\n length: " + f.length() +
			      "\n lastModified: " + f.lastModified());
			    if(f.isFile())
			      System.out.println("It's a file");
			    else if(f.isDirectory())
			      System.out.println("It's a directory");
	}

	public static void main(String[] args) {
		 if(args.length < 1) usage();
		    if(args[0].equals("-r")) {
		      if(args.length != 3) usage();
		      File
		        old = new File(args[1]),
		        rname = new File(args[2]);
		      //重命名文件或移动到参数所指的路径
		      old.renameTo(rname);
		      fileData(old);
		      fileData(rname);
		      return; // Exit main
		    }
		    int count = 0;
		    boolean del = false;
		    if(args[0].equals("-d")) {
		      count++;
		      del = true;
		    }
		    count--;
		    while(++count < args.length) {
		      File f = new File(args[count]);
					//此抽象路径名定义的文件或目录是否存在
		      if(f.exists()) {
		        System.out.println(f + " exists");
		        if(del) {
		          System.out.println("deleting..." + f);
							//删除文件或目录
		          f.delete();
		        }
		      }
		      else { // Doesn't exist
		        if(!del) {
		        	//可以创建多级目录，父目录不一定存在
		          f.mkdirs();
		          System.out.println("created " + f);
		        }
		      }
		      fileData(f);
		    }
	}
}

```
## 输入和输出
流是一个抽象的概念，它代表任何有能力产生数据源的对象或者有能力接收数据的接收对象。“流” 实际上屏蔽了实际的 I/O 设备中处理数据的细节。

Java 类库中的 I/O 类分成输入和输出两部分。通过继承，任何自 Inputstream 或 Reader 派生而来的类都具有 read() 的基本方法。用于读取单个字节或者字节数组。同样，任何继承自 OutputStream 或 Writer 派生而来的类都含有名为 write() 的基本方法，用于写单个字节或字节数组。我们很少使用单一的类创建流对象，而是通过叠合多个对象来提供所期望的功能。Java I/O 流类库让人迷惑的原因就在于：创建单一的结果流，却需要创建多个对象。

有必要按照这些类的功能对他们进行分类。类库的设计者首先限定于输入有关的所有的类都应该从 Inputstream 继承，而与输出有关的所有类都应该从 OutputStream 继承。

#### Inputstream 类型
Inputstream 的作用是用来表示那些从不同数据源产生输入的类。这些数据源包括：
- 字节数组
- String 对象
- 文件
- 管道
- 一个由其他种类的流组成的序列。以便我们可以将他们收集合并到一个流内。
- 其他数据源，比如网络上获取的。

#### OutputStream 类型
该类别的类决定了输出所要去往的目标：字节数组、文件、或管道。

## 总结
以上的内容只是对 I/O 流的简单说明。I/O 流中的类错综复杂，非常多。要想恰当的去理解每一个类的作用和方法。还需要我们去查看 JDK 文档。
