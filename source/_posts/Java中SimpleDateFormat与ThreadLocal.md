---
title: Java中SimpleDateFormat与ThreadLocal
tags:
  - SimpleDateFormat
  - ThreadLocal
id: 491
categories:
  - 学习
date: 2014-11-09 14:14:08
---

在Java中我们通常可能为了性能考虑，通过static设置一个全局的DateFormat，如下所示。

<!--more-->

```java
public class Utils {

	private static final DateFormat mDateTimeFormatter = new SimpleDateFormat("yy-MM-dd HH:mm:ss");

	public static Date parse(String dateTimeString) {
		try {
			return mDateTimeFormatter.parse(dateTimeString);
		} catch (ParseException e) {
			e.printStackTrace();
		}
		return null;
	}

	public static String format(Date dateTime) {
		return mDateTimeFormatter.format(dateTime);
	}

}
```

但我们忽略了一个很重要的地方，就是[SimpleDateFormat](https://docs.oracle.com/javase/7/docs/api/java/text/SimpleDateFormat.html "Date formats are not synchronized. It is recommended to create separate format instances for each thread. If multiple threads access a format concurrently, it must be synchronized externally.")并不是线程安全的（即non-thread-safe），Android版的[SimpleDateFormat](http://developer.android.com/reference/java/text/SimpleDateFormat.html "SimpleDateFormat is not thread-safe. Users should create a separate instance for each thread.")同样不是线程安全的，所以上面的代码是存在一定的问题，不过对于格式化日期时间使用很少或进程很少的情况下是很难发现这个问题。

要正确使用格式化日期时间的方式可以有以下几种
1\. 在每个方法中创建局部对象，每次调用都会创建，用完之后立刻垃圾回收掉，但这种方法有些资源浪费，没有合理复用资源，并且SimpleDateFormat也不是一个轻量级的对象，非常不推荐使用。

```java
	public static Date parse(String dateTimeString) {
		DateFormat mDateTimeFormatter = new SimpleDateFormat("yy-MM-dd HH:mm:ss");
		try {
			return mDateTimeFormatter.parse(dateTimeString);
		} catch (ParseException e) {
			e.printStackTrace();
		}
		return null;
	}

	public static String format(Date dateTime) {
		DateFormat mDateTimeFormatter = new SimpleDateFormat("yy-MM-dd HH:mm:ss");
		return mDateTimeFormatter.format(dateTime);
	}
```

2\. 在方法调用时使用synchronized同步块，如下图所示，这样但某一时刻同时又多个线程使用，则后面的线程会阻塞直到前面的使用完，这种方式可以节省资源，适用于线程不多，调用不频繁的情况。

```java
	public static Date parse(String dateTimeString) {
		synchronized (mDateTimeFormatter) {
			try {
				return mDateTimeFormatter.parse(dateTimeString);
			} catch (ParseException e) {
				e.printStackTrace();
			}
			return null;
		}
	}

	public static String format(Date dateTime) {
		synchronized (mDateTimeFormatter) {
			return mDateTimeFormatter.format(dateTime);
		}
	}
```

3\. 使用[ThreadLocal](https://docs.oracle.com/javase/7/docs/api/java/lang/ThreadLocal.html "Each thread holds an implicit reference to its copy of a thread-local variable as long as the thread is alive and the ThreadLocal instance is accessible; after a thread goes away, all of its copies of thread-local instances are subject to garbage collection (unless other references to these copies exist).")对没有线程创建一份非线程安全的对象实例，这种方式比较优雅

```java
	private static final ThreadLocal<SimpleDateFormat> mDateTimeFormatter = new ThreadLocal<SimpleDateFormat>() {
		protected SimpleDateFormat initialValue() {
			return new SimpleDateFormat("yy-MM-dd HH:mm:ss");
		};
	};

	public static Date parse(String dateTimeString) {
		try {
			return mDateTimeFormatter.get().parse(dateTimeString);
		} catch (ParseException e) {
			e.printStackTrace();
		}
		return null;
	}

	public static String format(Date dateTime) {
		return mDateTimeFormatter.get().format(dateTime);
	}
```

ThreadLocal的实现方式其实很直接，在每个Thread中都有一个map(ThreadLocalMap对象，对ThreadLocal是弱引用)

```java
// Thead.java
    ThreadLocal.ThreadLocalMap threadLocals = null;

// ThreadLocal.java
static class ThreadLocalMap {
static class Entry extends WeakReference<ThreadLocal<?>> {
......

    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }

```

但不恰当使用时也可能会出现内存泄漏，可以参考[A Painless Introduction to Java’s ThreadLocal Storage](http://www.appneta.com/blog/introduction-to-javas-threadlocal-storage/ "First of all, the value object put into the ThreadLocal would not purge itself (garbage collected) if there are no more Strong references to it. Instead, the Weak reference is done on the thread instance, which means Java garbage collection would clean up the ThreadLocal map if the thread itself is not strongly referenced elsewhere.")，[ThreadLocal and memory leaks](http://avasseur.blogspot.com/2003/11/threadlocal-and-memory-leaks.html) 。

4\. 使用线程安全的DateFormat，如Commons Lang的[FastDateFormat](https://commons.apache.org/proper/commons-lang/javadocs/api-2.6/org/apache/commons/lang/time/FastDateFormat.html "FastDateFormat is a fast and thread-safe version of SimpleDateFormat.")，joda-time的[DateTimeFormat](http://joda-time.sourceforge.net/apidocs/org/joda/time/format/DateTimeFormat.html "DateTimeFormat is thread-safe and immutable, and the formatters it returns are as well.")，或者JDK8中的[DateTimeFormatter](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html "A formatter created from a pattern can be used as many times as necessary, it is immutable and is thread-safe.")

参考资料
1\. [simpledateformat-thread-safety](http://stackoverflow.com/questions/6840803/simpledateformat-thread-safety)