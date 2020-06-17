---
title: Java Object 类方法概览
date: 2020-06-04 16:51:55
tags: Java
---

##### getClass

```java
public final native Class<?> getClass();
```

getClass() 是一个 native 方法，返回的是此 Object 对象的类对象/运行时类对象 Class<?>。效果与Object.class 相同。

首先解释下"类对象"的概念：在Java中，类是是对具有一组相同特征或行为的实例的抽象并进行描述，对象则是此类所描述的特征或行为的具体实例。作为概念层次的类，其本身也具有某些共同的特性，如都具有类名称、由类加载器去加载，都具有包，具有父类，属性和方法等。于是，Java中有专门定义了一个类，java.lang.Class，去描述其他类所具有的这些特性，因此，从此角度去看，类本身也都是属于 Class 类的对象。为与经常意义上的对象相区分，在此称之为"类对象"。

```java
private void getClassMethod() {
        System.out.print("------------------------------");
        System.out.println("\ngetClass() 方法");
        MyBean bean = new MyBean();
        System.out.println(bean.getClass());
        System.out.println(MyBean.class);
    }
--output
class pojo.jsonProcess.MyBean
class pojo.jsonProcess.MyBean
```


<escape><!-- more --></escape>
##### hashCode

```java
public native int hashCode();
```

回该对象的哈希码值。支持此方法是为了提高哈希表性能，比如 HashMap。

以集合类中，以 HashSet为例，当新加一个对象时，需要判断现有集合中是否已经存在与此对象相等的对象，如果没有 hashCode()方法，需要将 Set 进行一次遍历，并逐一用 equals() 方法判断两个对象是否相等，此种算法时间复杂度为 o(n)。通过借助于 hasCode() 方法，先计算出即将新加入对象的哈希码，然后根据哈希算法计算出此对象的位置，直接判断此位置上是否已有对象即可。（注：HashSet 是基于 HashMap 实现，除了 clone() 、writeObject()、readObject()是 HashSet 自己不得不实现之外，其他方法都是直接调用 HashMap 中的方法）

`hashCode` 的常规协定是：

- 在 Java 应用程序执行期间，在对同一对象多次调用 hashCode 方法时，必须一致地返回相同的整数，前提是将对象进行 equals 比较时所用的信息没有被修改。从某一应用程序的一次执行到同一应用程序的另一次执行，该整数无需保持一致。
- 如果根据 equals(Object) 方法，两个对象是相等的，那么对这两个对象中的每个对象调用 hashCode 方法都必须生成相同的整数结果。
- 如果根据 [equals(java.lang.Object)](http://itmyhome.com/java-api/java/lang/Object.html#equals(java.lang.Object)) 方法，两个对象不相等，那么对这两个对象中的任一对象上调用 hashCode 方法不要求一定生成不同的整数结果。但是，程序员应该意识到，为不相等的对象生成不同整数结果可以提高哈希表的性能。

实际上，由 Object 类定义的 hashCode 方法确实会针对不同的对象返回不同的整数。（这一般是通过将该对象的内部地址转换成一个整数来实现的，但是 Java 编程语言不需要这种实现技巧。）

```java
System.out.println("\nlewis ->"+"lewis".hashCode());
System.out.println("\nlewis2 ->"+"lewis".hashCode());

--output
lewis ->102866888
lewis2 ->102866888
```



##### equals

```java
public boolean equals(Object obj) {
        return (this == obj);
    }
```

可以到在 Object 类中的实现是直接对比了两个对象的引用。但是当子类重写了 Object 方法时，就需要具体的看各自实现了，如 String 类和 Integer 类。更多信息参见 [Java 中 == 和 equals() 的区别](https://juejin.im/post/5ec25ffaf265da49981aceb1)



##### clone

创建并返回此对象的一个副本。“副本”的准确含义可能依赖于对象的类。这样做的目的是，对于任何对象 x

- 表达式：x.clone() != x 为 true
- 表达式：x.clone().getClass() == x.getClass() 也为  true，但这并非必须要满足的要求。
- 一般情况下：x.clone().equals(x) 为 true，但这并非必须要满足的要求。

```java
protected native Object clone() throws CloneNotSupportedException;
```

clone() 是一个 native 方法，使用 protected 修饰，所以子类使用需要实现 Cloneable 接口并重写 clone() 方法。此方法属于浅拷贝，当对象被复制时只复制它本身和其中包含的值类型的成员变量，而引用类型的成员对象并没有复制。

```java
public class JavaObject implements Cloneable

@Override
protected Object clone() throws CloneNotSupportedException {
	return  super.clone();
}

private void cloneMethod() throws CloneNotSupportedException {
        System.out.print("------------------------------");
        System.out.println("\nclone() 方法");
        JavaObject o1 = new JavaObject();
        JavaObject o2 = (JavaObject) o1.clone();
        System.out.println(o1 == o2);
        System.out.println("\no1 ->"+o1.hashCode());
        System.out.println("\no2 ->"+o2.hashCode());

    }
--output
false
o1 ->1627674070
o2 ->1360875712
```



##### toString

toString() 方法返回一个对象的字符串表示，实现如下：

```java
public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
```

Integer.toHexString() 方法以十六进制（基数 16）无符号整数形式返回一个整数参数的字符串表示形式。也就是将十进制转换为十六进制。

toString()是由对象的类型和其哈希码唯一确定，同一类型但不相等的两个对象分别调用toString()方法返回的结果可能相同。



##### wait 系列方法 与 notify/notifyAll

这几个方法与多线程协作相关：

wait()：调用此方法所在的当前线程等待，直到在其他线程上调用此方法的主调（某一对象）的notify()/notifyAll()方法。

wait(long timeout)/wait(long timeout, int nanos)：调用此方法所在的当前线程等待，直到在其他线程上调用此方法的主调（某一对象）的notisfy()/notisfyAll()方法，或超过指定的超时时间量。

notify()/notifyAll()：唤醒在此对象监视器上等待的单个线程（随机）/所有线程

```java
public final native void notify();
 
public final native void notifyAll(); 

public final native void wait(long timeout) throws InterruptedException;
 
 public final void wait() throws InterruptedException {
        wait(0);
    }
    
 public final void wait(long timeout, int nanos) throws InterruptedException {
        if (timeout < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                                "nanosecond timeout value out of range");
        }

        if (nanos > 0) {
            timeout++;
        }

        wait(timeout);
    }

```

延申：Java中的notify和notifyAll有什么区别？

> 原文：[Java中的notify和notifyAll有什么区别](https://www.zhihu.com/question/37601861)

先说两个概念：锁池和等待池

- 锁池:假设线程A已经拥有了某个对象(注意:不是类)的锁，而其它的线程想要调用这个对象的某个synchronized方法(或者synchronized块)，由于这些线程在进入对象的synchronized方法之前必须先获得该对象的锁的拥有权，但是该对象的锁目前正被线程A拥有，所以这些线程就进入了该对象的锁池中。
- 等待池:假设一个线程A调用了某个对象的wait()方法，线程A就会释放该对象的锁后，进入到了该对象的等待池中

> Reference：[java中的锁池和等待池 ](https://link.zhihu.com/?target=http%3A//blog.csdn.net/emailed/article/details/4689220)

然后再来说notify和notifyAll的区别

- 如果线程调用了对象的 wait()方法，那么线程便会处于该对象的**等待池**中，等待池中的线程**不会去竞争该对象的锁**。
- 当有线程调用了对象的 **notifyAll**()方法（唤醒所有 wait 线程）或 **notify**()方法（只随机唤醒一个 wait 线程），被唤醒的的线程便会进入该对象的锁池中，锁池中的线程会去竞争该对象锁。也就是说，调用了notify后只要一个线程会由等待池进入锁池，而notifyAll会将该对象等待池内的所有线程移动到锁池中，等待锁竞争
- 优先级高的线程竞争到对象锁的概率大，假若某线程没有竞争到该对象锁，它**还会留在锁池中**，唯有线程再次调用 wait()方法，它才会重新回到等待池中。而竞争到对象锁的线程则继续往下执行，直到执行完了 synchronized 代码块，它会释放掉该对象锁，这时锁池中的线程会继续竞争该对象锁。

> Reference：[线程间协作：wait、notify、notifyAll ](https://link.zhihu.com/?target=http%3A//wiki.jikexueyuan.com/project/java-concurrency/collaboration-between-threads.html)

综上，所谓唤醒线程，另一种解释可以说是将线程由等待池移动到锁池，notifyAll调用后，会将全部线程由等待池移到锁池，然后参与锁的竞争，竞争成功则继续执行，如果不成功则留在锁池等待锁被释放后再次参与竞争。而notify只会唤醒一个线程。

##### finalize

```java
protected void finalize() throws Throwable { }

```

finalize 是Object的 protected 方法，子类可以覆盖该方法以实现资源清理工作，GC在回收对象之前调用该方法。具体信息参见 [finalize 方法与对象复活](https://juejin.im/post/5ed711a4518825432e25cae5)