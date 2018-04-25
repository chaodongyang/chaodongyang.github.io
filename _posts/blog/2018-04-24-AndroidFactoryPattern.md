---
layout: post
title: Android架构设计之简单工厂与工厂方法模式
categories: Android架构设计专题
description: Android架构设计之简单工厂与工厂方法模式
keywords: Android, Android架构设计,简单工厂与工厂方法模式
---

Android 架构设计专题第二弹简单工厂和工厂模式。工厂模式在我们的 Android 源码中应用的非常的广泛，比如我们经常使用的 BitmapFactory 就是使用了工厂模式。我们只需要传入我们对应的参数并不需要具体的知道应该如何创建相应的 Bitmap 对象。

## BitmapFactory 源码解析
BitmapFactory 源码非常的复杂，我们这里只针对工厂模式的具体点进行一些剖析。
- 第一步
```java
BitmapFactory.decodeResource()
```
- 第二步
```java
 public static Bitmap decodeResource(Resources res, int id) {
        return decodeResource(res, id, null);
    }
```
 我们进入方法体的内部看到当我们传入资源 res 以及 id 方法内部调用了 decodeResource() 方法并且返回一个 Bitmap 对象。那么这个 Bitmap 是如何被创建的呢 ？我们不需要关心具体的过程。

- 第三步
```java
public static Bitmap decodeResource(Resources res, int id, Options opts) {
        Bitmap bm = null;
        InputStream is = null;

        try {
            final TypedValue value = new TypedValue();
            is = res.openRawResource(id, value);
            bm = decodeResourceStream(res, value, is, null, opts);
        } catch (Exception e) {
            /*  do nothing.
                If the exception happened on open, bm will be null.
                If it happened on close, bm is still valid.
            */
        } finally {
            try {
                if (is != null) is.close();
            } catch (IOException e) {
                // Ignore
            }
        }
        if (bm == null && opts != null && opts.inBitmap != null) {
            throw new IllegalArgumentException("Problem decoding into existing bitmap");
        }

        return bm;
    }
```
我们看到在这个方法体内部又通过使用 decodeResourceStream() 方法为我们创建一个 Bitmap 对象并且返回。

- 第四步
```java
public static Bitmap decodeResourceStream(Resources res, TypedValue value,
            InputStream is, Rect pad, Options opts) {

        if (opts == null) {
            opts = new Options();
        }

        if (opts.inDensity == 0 && value != null) {
            final int density = value.density;
            if (density == TypedValue.DENSITY_DEFAULT) {
                opts.inDensity = DisplayMetrics.DENSITY_DEFAULT;
            } else if (density != TypedValue.DENSITY_NONE) {
                opts.inDensity = density;
            }
        }

        if (opts.inTargetDensity == 0 && res != null) {
            opts.inTargetDensity = res.getDisplayMetrics().densityDpi;
        }

        return decodeStream(is, pad, opts);
    }
```
我们再次进入 decodeStream() 方法内看 Bitmap 对象是如何创立的

- 第五步
```java
public static Bitmap decodeStream(InputStream is, Rect outPadding, Options opts) {
        // we don't throw in this case, thus allowing the caller to only check
        // the cache, and not force the image to be decoded.
        if (is == null) {
            return null;
        }

        Bitmap bm = null;

        Trace.traceBegin(Trace.TRACE_TAG_GRAPHICS, "decodeBitmap");
        try {
            /**
            *通过对我们传入参数的判断来创建并且返回具体的Bitmap 对象
            */
            if (is instanceof AssetManager.AssetInputStream) {
                final long asset = ((AssetManager.AssetInputStream) is).getNativeAsset();
                bm = nativeDecodeAsset(asset, outPadding, opts);
            } else {
                bm = decodeStreamInternal(is, outPadding, opts);
            }

            if (bm == null && opts != null && opts.inBitmap != null) {
                throw new IllegalArgumentException("Problem decoding into existing bitmap");
            }

            setDensityFromOptions(bm, opts);
        } finally {
            Trace.traceEnd(Trace.TRACE_TAG_GRAPHICS);
        }

        return bm;
    }
```
关键的地点我已注释起来。通过我们传入的参数的不同，返回给我们不同的 Bitmap 对象。不需要我们知道具体的 Bitmap 应该如何实现。

## 简单工厂
简单工厂是属于创建型模式，又叫做静态工厂方法（Static Factory Method）模式。是工厂模式中最简单实用的模型。

#### 普通的对象创建
我们创建一个接口叫 Api：
```java
public interface Api {

}
```
创建一个它的实现类：
```java
public class Impl implements Api {

}

```
我们如何在客户端使用呢 ？
```java
public class Client {

	public static void main(String[] args) {
			Api api = new Impl();
	}

}

```
上面的办法是我们最常使用的，使用 new 关键字去创建一个对象。但是这样做的前提是我们必须知道有多少个对象以及如何去创建。对于接口最好是隔离起来，但是这个地方我们却给客户端展示了我们的接口是谁实现的。实际上最好是我们不需要知道接口是如何实现的。我们最好要做到最少知识原则，不需要自己去实现接口。

#### 简单工厂创建对象
我们再次创建两个具体的实现类：

```java
class ImplB implements Api {

}
```
```java
class ImplC implements Api {

}
```
我们创建一个工厂类：

```java
public class Factory {
	public static Api create(int type) {
		switch (type) {
		case 0:
			return new Impl();
		case 1:
			return new ImplB();
		case 2:
			return new ImplC();
		default:
			return new ImplC();
		}
	}

}
```
通过客户端调用：

```java
public class Client {

	public static void main(String[] args) {
			/**
			 * 简单工厂方法，封装了具体实现。需要什么就创建什么就可以。
			 * 应用场景：创建对象
			 * 提供创建对象的功能，不需要具体的实现。降低客户端与模块之间的耦合度。具体的实现放在子类里边实现
			 * Android 中 BitmapFactory.decodeResource() 的创建过程就是工厂模式
			 */

			Api api2 = Factory.create(0);
			api2 = Factory.create(1);
			api2 = Factory.create(2);
	}

}
```
看到这个代码是不是很熟悉，和刚刚我们看到的 BitmapFactory 其实原理是一样的。通过上面的方法我们就做到了把具体的实现封装起来，根据传递进来的参数调用者需要什么就给什么。

## 工厂方法
工厂方法模式，又称工厂模式、多态工厂模式和虚拟构造器模式，通过定义工厂父类负责定义创建对象的公共接口，而子类则负责生成具体的对象。
#### 简单工厂的缺点
- 工厂类集中了所有对象创建的逻辑，一旦这个工厂类不能使用将会影响到整个系统的运行
- 一旦添加新的对象就必须修改工厂类
- 简单工厂使用的是静态方法无法被继承和重写

#### 工厂方法原理
将类的实例化（具体产品的创建）延迟到工厂类的子类（具体工厂）中完成，即由子类来决定应该实例化（创建）哪一个类。

#### 使用步骤
我们来模仿一个导出文件的示例：有一个数据库可以让我们导出不同类型的文件。
- 第一步，创建导出类

```java
public interface ExportFileApi {
	public void export(String data);
}
```
- 第二步，创建接口的实现类

```java
/**
 * 导出Excel文件
 * @author 晁东洋
 *
 */
public class ExportExcelFile implements ExportFileApi{

	public void export(String data) {
		// TODO Auto-generated method stub
		System.out.println("导出ExcelFile文件");

	}

}

/**
 * 导出数据库文件
 * @author 11842
 *
 */
public class ExportDBFile implements ExportFileApi {

	public void export(String data) {
		// TODO Auto-generated method stub
		System.out.println("导出数据库文件");
	}

}

```
- 第三步，创建抽象工厂类

```java
public abstract class ExportOperate {
	/**
	 * 实例化ExportFileApi
	 * @return
	 */
	public abstract ExportFileApi newFileApi();

	/**
	 * 导出数据
	 * @param data
	 */
	public void export(String data){
		ExportFileApi file = newFileApi();
		file.export(data);
	}
}

```

- 第四步，创建工厂类的子类，在子类中具体实现对象的创建

```java
public class ExportDBOperator extends ExportOperate {

	public ExportFileApi newFileApi() {
		// TODO Auto-generated method stub
		return new ExportDBFile();
	}

}
```

```java
public class ExportExcelOperator extends ExportOperate {

	public ExportFileApi newFileApi() {
		// TODO Auto-generated method stub
		return new ExportExcelFile();
	}

}

```
- 第五步，具体的调用

```java
ExportDBOperator op = new ExportDBOperator();
			op.export(data);

ExportExcelOperator op2 = new ExportExcelOperator();
		  op2.export(data);
```

工厂方法的使用到这里就结束了，其中最核心的知识点就是把类的实例化延迟到工厂类的子类，由子类决定实例化那个对象。

## 优点

- 更符合开闭原则，新增一种对象时，只需要新增具体的工厂类和具体的对象即可
- 每个工厂类负责创建对应的产品
- 不使用静态工厂方法可以形成继承的等级结构

## 缺点

- 新增产品时除了增加具体的类还需要增加具体的工厂类，增加了系统的复杂性。
- 代码中使用了抽象，增加了抽象性和理解难度
- 一个具体工厂只能创建一个具体产品

## 总结
再具体的实现中我们还需要根据具体的业务场景来实现我们的代码，设计模式不是一成不变的。本系列专题将会持续更新关于 Android 架构的知识内容。设计模式是最后期学习 Android 架构能够更好的理解和掌握所必须具备的技能。
