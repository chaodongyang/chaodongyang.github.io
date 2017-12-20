---
layout: post
title: java编程思想之枚举类型详解(深入)
categories: javaThinking
description: java编程思想之枚举类型详解
keywords: java, java枚举类型详解,java枚举详解，java枚举
---
Java 的 enum 有一个非常有趣的特性，允许程序员为 enum 实例编写方法，从而为每个 enum 实例赋予各自不同的行为。

## 常量相关的方法
要实现常量相关的方法，你需要在 enum 定义一个或者多个 abstract 方法，然后为每个 enum 实例实现该抽象方法。
```java
public enum ConstantSpecificMethod {
	 DATE_TIME {
	    String getInfo() {
	      return
	        DateFormat.getDateInstance().format(new Date());
	    }
	  },
	  CLASSPATH {
	    String getInfo() {
	      return System.getenv("CLASSPATH");
	    }
	  },
	  VERSION {
	    String getInfo() {
	      return System.getProperty("java.version");
	    }
	  };
	  abstract String getInfo();
	  public static void main(String[] args) {
	    for(ConstantSpecificMethod csm : values())
	      System.out.println(csm.getInfo());
	  }
}

```
通过相应的 enum 实例，我们可以调用其上的方法。这通常也被称为表驱动的代码。
在面向对象的设计中不同的行为与不同的类相关联。通过常量相关的方法，每个 enum 实例可以具备自己独特的行为，这似乎说明每个 enum 实例就像一个独特的类。然而，enum 实例与类的相似之处仅限于此。我们并不能真的将 enum 实例作为一个类型来使用。

我们看一个有关洗车的例子。每个顾客在洗车时，都有一个选择菜单，每个选择对应不同的动作。可以将一个常量相关的方法关联到一个选择上，在使用 EnumSet 来保存客户的选择：
```java
public class CarWash {
	public enum Cycle {
	    UNDERBODY {
	      void action() { Print.print("Spraying the underbody"); }
	    },
	    WHEELWASH {
	      void action() { Print.print("Washing the wheels"); }
	    },
	    PREWASH {
	      void action() { Print.print("Loosening the dirt"); }
	    },
	    BASIC {
	      void action() { Print.print("The basic wash"); }
	    },
	    HOTWAX {
	      void action() { Print.print("Applying hot wax"); }
	    },
	    RINSE {
	      void action() { Print.print("Rinsing"); }
	    },
	    BLOWDRY {
	      void action() { Print.print("Blowing dry"); }
	    };
	    abstract void action();
	}

	EnumSet<Cycle> cycles = EnumSet.of(Cycle.BASIC, Cycle.RINSE);
	public void add(Cycle cycle) { cycles.add(cycle); }
	public void washCar() {
		 for(Cycle c : cycles)
		   c.action();
	}
	public String toString() {
		return cycles.toString();
	}

	public static void main(String[] args) {
	    CarWash wash = new CarWash();
	    Print.print(wash);
	    wash.washCar();
	    // Order of addition is unimportant:
	    wash.add(Cycle.BLOWDRY);
	    wash.add(Cycle.BLOWDRY); // Duplicates ignored
	    wash.add(Cycle.RINSE);
	    wash.add(Cycle.HOTWAX);
	    Print.print(wash);
	    wash.washCar();
	  }
}
```
测试结果：
```
[BASIC, RINSE]
The basic wash
Rinsing
[BASIC, HOTWAX, RINSE, BLOWDRY]
The basic wash
Applying hot wax
Rinsing
Blowing dry

```
与匿名内部类相比较，定义常量相关方法的语法更高效、简洁。EnumSet 添加的元素不能重复，添加的顺序并不重要，因为其输出的次序是由 enum 实例定义的次序决定。

除了实现抽象方法之外，那么能不能覆盖普通的方法呢？答案是肯定的：
```java
public enum OverrideConstantSpecific {
  NUT, BOLT,
  WASHER {
    void f() { print("Overridden method"); }
  };
  void f() { print("default behavior"); }
  public static void main(String[] args) {
    for(OverrideConstantSpecific ocs : values()) {
      printnb(ocs + ": ");
      ocs.f();
    }
  }
}
```

#### 使用 enum 的职责链
在职责链设计模式中，我们可以以多种不同的方式来解决问题，然后将他们连接在一起。当一个请求来临时，它遍历这个链，直到链中的某个解决方案能够处理该请求。

通过常量相关的方法，我们可以很容易的实现一个简单的职责链。
```java
class Mail {
  // The NO's lower the probability of random selection:
  enum GeneralDelivery {YES,NO1,NO2,NO3,NO4,NO5}
  enum Scannability {UNSCANNABLE,YES1,YES2,YES3,YES4}
  enum Readability {ILLEGIBLE,YES1,YES2,YES3,YES4}
  enum Address {INCORRECT,OK1,OK2,OK3,OK4,OK5,OK6}
  enum ReturnAddress {MISSING,OK1,OK2,OK3,OK4,OK5}
  GeneralDelivery generalDelivery;
  Scannability scannability;
  Readability readability;
  Address address;
  ReturnAddress returnAddress;
  static long counter = 0;
  long id = counter++;
  public String toString() { return "Mail " + id; }
  public String details() {
    return toString() +
      ", General Delivery: " + generalDelivery +
      ", Address Scanability: " + scannability +
      ", Address Readability: " + readability +
      ", Address Address: " + address +
      ", Return address: " + returnAddress;
  }
  // Generate test Mail:
  public static Mail randomMail() {
    Mail m = new Mail();
    m.generalDelivery= Enums.random(GeneralDelivery.class);
    m.scannability = Enums.random(Scannability.class);
    m.readability = Enums.random(Readability.class);
    m.address = Enums.random(Address.class);
    m.returnAddress = Enums.random(ReturnAddress.class);
    return m;
  }
  public static Iterable<Mail> generator(final int count) {
    return new Iterable<Mail>() {
      int n = count;
      public Iterator<Mail> iterator() {
        return new Iterator<Mail>() {
          public boolean hasNext() { return n-- > 0; }
          public Mail next() { return randomMail(); }
          public void remove() { // Not implemented
            throw new UnsupportedOperationException();
          }
        };
      }
    };
  }
}

public class PostOffice {
  enum MailHandler {
    GENERAL_DELIVERY {
      boolean handle(Mail m) {
        switch(m.generalDelivery) {
          case YES:
            print("Using general delivery for " + m);
            return true;
          default: return false;
        }
      }
    },
    MACHINE_SCAN {
      boolean handle(Mail m) {
        switch(m.scannability) {
          case UNSCANNABLE: return false;
          default:
            switch(m.address) {
              case INCORRECT: return false;
              default:
                print("Delivering "+ m + " automatically");
                return true;
            }
        }
      }
    },
    VISUAL_INSPECTION {
      boolean handle(Mail m) {
        switch(m.readability) {
          case ILLEGIBLE: return false;
          default:
            switch(m.address) {
              case INCORRECT: return false;
              default:
                print("Delivering " + m + " normally");
                return true;
            }
        }
      }
    },
    RETURN_TO_SENDER {
      boolean handle(Mail m) {
        switch(m.returnAddress) {
          case MISSING: return false;
          default:
            print("Returning " + m + " to sender");
            return true;
        }
      }
    };
    abstract boolean handle(Mail m);
  }
  static void handle(Mail m) {
    for(MailHandler handler : MailHandler.values())
      if(handler.handle(m))
        return;
    print(m + " is a dead letter");
  }
  public static void main(String[] args) {
    for(Mail mail : Mail.generator(10)) {
      print(mail.details());
      handle(mail);
      print("*****");
    }
  }
}
```
职责链设计模式的相关内容会在下一个专题设计模式中详细介绍，在这里不做过多解释。重点在于理解枚举的知识点。

#### 使用 enum 的状态机
枚举类型非常适合用来创建状态机。一个状态机可以具有有限个特定的状态，它通常根据输入，从一个状态转移到下一个状态，不过也可能存在瞬时状态，而一旦任务执行结束，状态机就会立刻离开瞬时状态。每个状态都具有某些可接受的输入，不同的输入会使状态机从当前状态转移到不同的新状态。
```java
public enum Input {
	  NICKEL(5), DIME(10), QUARTER(25), DOLLAR(100),
	  TOOTHPASTE(200), CHIPS(75), SODA(100), SOAP(50),
	  ABORT_TRANSACTION {
	    public int amount() { // Disallow
	      throw new RuntimeException("ABORT.amount()");
	    }
	  },
	  STOP { // This must be the last instance.
	    public int amount() { // Disallow
	      throw new RuntimeException("SHUT_DOWN.amount()");
	    }
	  };
	  int value; // In cents
	  Input(int value) {
		  this.value = value;
	  }
	  Input() {}

	  int amount() {
		  return value;
	  };
	  // In cents
	  static Random rand = new Random(47);
	  public static Input randomSelection() {
	    // Don't include STOP:
	    return values()[rand.nextInt(values().length - 1)];
	  }
}

```
下面的例子演示了 enum 是如何使代码变得更加清晰且易于管理的：
```java
enum Category {
	  MONEY(NICKEL, DIME, QUARTER, DOLLAR),
	  ITEM_SELECTION(TOOTHPASTE, CHIPS, SODA, SOAP),
	  QUIT_TRANSACTION(ABORT_TRANSACTION),
	  SHUT_DOWN(STOP);

	  private Input[] values;

	  Category(Input... types) {
		  values = types;
	  }

	  private static EnumMap<Input,Category> categories = new EnumMap<Input,Category>(Input.class);

	  static {
	    for(Category c : Category.class.getEnumConstants())
	      for(Input type : c.values)
	        categories.put(type, c);
	  }

	  public static Category categorize(Input input) {
	    return categories.get(input);
	  }
}
public class VendingMachine {
	  private static State state = State.RESTING;
	  private static int amount = 0;
	  private static Input selection = null;
	  enum StateDuration { TRANSIENT } // Tagging enum
	  enum State {
	    RESTING {
	      void next(Input input) {
	        switch(Category.categorize(input)) {
	          case MONEY:
	            amount += input.amount();
	            state = ADDING_MONEY;
	            break;
	          case SHUT_DOWN:
	            state = TERMINAL;
	          default:
	        }
	      }
	    },
	    ADDING_MONEY {
	      void next(Input input) {
	        switch(Category.categorize(input)) {
	          case MONEY:
	            amount += input.amount();
	            break;
	          case ITEM_SELECTION:
	            selection = input;
	            if(amount < selection.amount())
	            	Print.print("Insufficient money for " + selection);
	            else state = DISPENSING;
	            break;
	          case QUIT_TRANSACTION:
	            state = GIVING_CHANGE;
	            break;
	          case SHUT_DOWN:
	            state = TERMINAL;
	          default:
	        }
	      }
	    },
	    DISPENSING(StateDuration.TRANSIENT) {
	      void next() {
	    	  Print.print("here is your " + selection);
	        amount -= selection.amount();
	        state = GIVING_CHANGE;
	      }
	    },
	    GIVING_CHANGE(StateDuration.TRANSIENT) {
	      void next() {
	        if(amount > 0) {
	        	Print.print("Your change: " + amount);
	          amount = 0;
	        }
	        state = RESTING;
	      }
	    },
	    TERMINAL { void output()
	    { Print.print("Halted"); }
	    };

	    private boolean isTransient = false;
	    State() {}
	    State(StateDuration trans) { isTransient = true; }

	    void next(Input input) {
	      throw new RuntimeException("Only call " +
	        "next(Input input) for non-transient states");
	    }
	    void next() {
	      throw new RuntimeException("Only call next() for " +
	        "StateDuration.TRANSIENT states");
	    }
	    void output() { Print.print(amount); }
	  }
	  static void run(Generator<Input> gen) {
	    while(state != State.TERMINAL) {
	      state.next(gen.next());
	      while(state.isTransient)
	        state.next();
	      state.output();
	    }
	  }
	  public static void main(String[] args) {
	    Generator<Input> gen = new RandomInputGenerator();
	    if(args.length == 1)
	      gen = new FileInputGenerator(args[0]);
	    run(gen);
	  }
	}

	// For a basic sanity check:
	class RandomInputGenerator implements Generator<Input> {
	  public Input next() { return Input.randomSelection(); }
	}

	// Create Inputs from a file of ';'-separated strings:
	class FileInputGenerator implements Generator<Input> {
	  private Iterator<String> input;
	  public FileInputGenerator(String fileName) {
	    input = new TextFile(fileName, ";").iterator();
	  }
	  public Input next() {
	    if(!input.hasNext())
	      return null;
	    return Enum.valueOf(Input.class, input.next().trim());
	  }
}

```
设计模式的内容这里不做过多的解释。

## 多路分发
当要处理多种交互类型时，程序可能会显得相当杂乱。如果一个系统要分析和执行数学表达式。我们可能会声明 Number.plus(Number)、Number.multiple(Number) 等等。然而，当你声明 a.plus(b) 时，你并不知道 a 或 b 的确切类型，那你如何能让他们正确的交互呢？

Java 只支持单路分发。也就是说如果要执行的操作包含不止一个类型未知的对象，那么 Java 的动态绑定机制只能处理其中的一个类型。这样就无法解决我们上面的问题。所以，你必须自己去判定其他的类型，从而实现自己的动态绑定行为。

解决上面问题的办法就是多路分发。多态只能发生在发生调用时，所以如果要使用多路分发，那么久必须有两个方法调用：分别决定自己的未知类型。要利用多路分发，我们必须为每一个类型提供一个实际的方法调用，如果要处理的是两个不同的类型体系，就需要为每个类型体系执行一个方法调用。一般而言，需要设定好某种配置，以便一个方法调用能够引出更多的方法调用，从而能够在这个过程中处理多种类型。为了表达这个效果，我们需要和多个方法一同工作：因为每个分发都需要一个方法调用。

下面的示例石头剪刀布游戏，对应的方法是 compete() 和 eval()，两者都是同一个类型的成员，他们可以产生三种 Outcom 实例中的一个作为结果：
```java
public enum Outcome {
	WIN, LOSE, DRAW
}

```
测试类的代码：
```java
interface Item {
   Outcome compete(Item it);
   Outcome eval(Paper p);
   Outcome eval(Scissors s);
   Outcome eval(Rock r);
}

class Paper implements Item {
    public Outcome compete(Item it) { return it.eval(this); }
	public Outcome eval(Paper p) { return DRAW; }
	public Outcome eval(Scissors s) { return WIN; }
	public Outcome eval(Rock r) { return LOSE; }
	public String toString() { return "Paper"; }
}

class Scissors implements Item {
	public Outcome compete(Item it) { return it.eval(this); }
	public Outcome eval(Paper p) { return LOSE; }
	public Outcome eval(Scissors s) { return DRAW; }
	public Outcome eval(Rock r) { return WIN; }
	public String toString() { return "Scissors"; }
}

class Rock implements Item {
	public Outcome compete(Item it) { return it.eval(this); }
	public Outcome eval(Paper p) { return WIN; }
	public Outcome eval(Scissors s) { return LOSE; }
	public Outcome eval(Rock r) { return DRAW; }
	public String toString() { return "Rock"; }
}
public class RoShamBo1 {
	  static final int SIZE = 3;
	  private static Random rand = new Random(47);

	  public static Item newItem() {
	    switch(rand.nextInt(3)) {
	      default:
	      case 0: return new Scissors();
	      case 1: return new Paper();
	      case 2: return new Rock();
	    }
	  }

	  public static void match(Item a, Item b) {
	    System.out.println(a + " vs. " + b + ": " +  a.compete(b));
	  }

	  public static void main(String[] args) {
	    for(int i = 0; i < SIZE; i++)
	      match(newItem(), newItem());
	  }
}

```
测试结果：
```
Rock vs. Rock: DRAW
Paper vs. Rock: WIN
Paper vs. Rock: WIN
```
Item 是这几种类型的接口，将会被用作多路分发。RoShamB01.match() 有两个 Item 参数，通过调用 Item.compete() 方法开始两路分发。要配置好多路分发需要很多的工序，不过要记住，他的好处在于方法调用时的优雅语法，这避免了在一个方法中判断多个对象类型的丑陋代码。不过在使用多路分发前请先明确，这种优雅的代码确实对你有意义。

#### 使用 enum 分发
上面的实例不能说是直接基于 enum 的，因为 enum 实例不是类型，不能将 enum 实例作为类型参数，所以无法重载 eval() 方法。不过还有很多方式可以实现多路分发。

一种方式是使用构造器来初始化每个 enum 实例，并以 “一组” 结果作为参数。
```java
public enum RoShamBo2 implements Competitor<RoShamBo2> {
	  PAPER(DRAW, LOSE, WIN),
	  SCISSORS(WIN, DRAW, LOSE),
	  ROCK(LOSE, WIN, DRAW);

	  private Outcome vPAPER, vSCISSORS, vROCK;

	  RoShamBo2(Outcome paper,Outcome scissors,Outcome rock) {
	    this.vPAPER = paper;
	    this.vSCISSORS = scissors;
	    this.vROCK = rock;
	  }

	  public Outcome compete(RoShamBo2 it) {
	    switch(it) {
	      default:
	      case PAPER: return vPAPER;
	      case SCISSORS: return vSCISSORS;
	      case ROCK: return vROCK;
	    }
	  }
	  public static void main(String[] args) {
	    RoShamBo.play(RoShamBo2.class, 3);
	  }
}

```
执行结果：
```
ROCK vs. ROCK: DRAW
SCISSORS vs. ROCK: LOSE
SCISSORS vs. ROCK: LOSE
```
在 compete() 方法中，一旦两种类型都被确定了，那么唯一的操作结果就是返回 OutCome。这个例子更简单更直接。注意：我们仍然是使用两路分发来判断两个对象的类型。在这个例子中分发只做了一次调用。第二个是使用的是 switch，因为 enum 限制了 switch 语句的选择分支。

Competitor 接口定义了一个类型，该类型的对象可以与另一个 Competitor 相竞争：
```java
public interface Competitor<T extends Competitor<T>>  {
	Outcome compete(T competitor);
}
```
随后我们定义了两个静态的方法。第一个 match() 方法他会为一个 Competitor 对象调用 compete() 方法。并与另外一个 Competitor 对象比较。match() 方法需要的参数是 ```Competitor<T>``` 类型。但是在 play() 方法中，类类型参数必须同时是 ```Enum<T>``` 类型和 ```Competitor<T>``` 类型。
```java
public class RoShamBo {
	public static <T extends Competitor<T>>
	  void match(T a, T b) {
	    System.out.println(
	      a + " vs. " + b + ": " +  a.compete(b));
	  }
	  public static <T extends Enum<T> & Competitor<T>>
	  void play(Class<T> rsbClass, int size) {
	    for(int i = 0; i < size; i++)
	    	RoShamBo.match( Enums.random(rsbClass),Enums.random(rsbClass));
	  }
}
```
#### 使用常量相关的方法
常量相关的方法允许我们为每个 enum 实例提供方法的不同实现，这使得常量相关的方法似乎是实现多路分发的解决方案。不过，通过这种方式，enum 实例虽然可以执行具有不同的行为，但他仍然不是类型，不能将其作为方法的参数传递。最好的办法是将 enum 用再 switch 语句中。
```java
public enum RoShamBo3 implements Competitor<RoShamBo3> {
  PAPER {
    public Outcome compete(RoShamBo3 it) {
      switch(it) {
        default: // To placate the compiler
        case PAPER: return DRAW;
        case SCISSORS: return LOSE;
        case ROCK: return WIN;
      }
    }
  },
  SCISSORS {
    public Outcome compete(RoShamBo3 it) {
      switch(it) {
        default:
        case PAPER: return WIN;
        case SCISSORS: return DRAW;
        case ROCK: return LOSE;
      }
    }
  },
  ROCK {
    public Outcome compete(RoShamBo3 it) {
      switch(it) {
        default:
        case PAPER: return LOSE;
        case SCISSORS: return WIN;
        case ROCK: return DRAW;
      }
    }
  };
  public abstract Outcome compete(RoShamBo3 it);
  public static void main(String[] args) {
    RoShamBo.play(RoShamBo3.class, 20);
  }
}
```
#### 使用 EnumMap 分发
使用 EnumMap 可以实现 “真正的” 两路分发。EnumMap 是为 enum 专门设计的一种性能非常好的 map。由于我们的目的是摸索出两种未知的类型，所以可以用 EnumMap 来实现：
```java
public enum RoShamBo5 implements Competitor<RoShamBo5> {
	  PAPER, SCISSORS, ROCK;

	  static EnumMap<RoShamBo5,EnumMap<RoShamBo5,Outcome>>
	    table = new EnumMap<RoShamBo5,
	      EnumMap<RoShamBo5,Outcome>>(RoShamBo5.class);

	  static {
	    for(RoShamBo5 it : RoShamBo5.values())
	      table.put(it,
	        new EnumMap<RoShamBo5,Outcome>(RoShamBo5.class));
	    initRow(PAPER, DRAW, LOSE, WIN);
	    initRow(SCISSORS, WIN, DRAW, LOSE);
	    initRow(ROCK, LOSE, WIN, DRAW);
	  }

	  static void initRow(RoShamBo5 it,
	    Outcome vPAPER, Outcome vSCISSORS, Outcome vROCK) {
	    EnumMap<RoShamBo5,Outcome> row =
	      RoShamBo5.table.get(it);
	    row.put(RoShamBo5.PAPER, vPAPER);
	    row.put(RoShamBo5.SCISSORS, vSCISSORS);
	    row.put(RoShamBo5.ROCK, vROCK);
	  }
	  public Outcome compete(RoShamBo5 it) {
	    return table.get(this).get(it);
	  }
	  public static void main(String[] args) {
	    RoShamBo.play(RoShamBo5.class, 3);
	  }
}
```
测试结果：
```
ROCK vs. ROCK: DRAW
SCISSORS vs. ROCK: LOSE
SCISSORS vs. ROCK: LOSE
```

## 总结
虽然 Java 的枚举比 C/C++ 中的枚举更成熟，但它仍然是一个小功能，Java 已经没有它存在很多年了。有时，恰恰因为它，你才能够干净优雅的解决问题。优雅与清晰很重要，正是他们区分了成功的解决方案与失败的解决方案。而失败的解决方案是因为其他人无法理解。
