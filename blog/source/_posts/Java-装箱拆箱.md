---
title: Java 装箱拆箱
date: 2020-06-04 17:03:42
tags: Java
---

#### 什么是装箱和拆箱

在 Java 中，当基本数据类型和其对应的包装类型进行运算的时候，编译器会帮我们自动进行转换，这就是装箱和拆箱。简单来说，从基本类型转为包装类型即为装箱，从包装类型转为基本类型即为拆箱。

Java 中基本类型和其对应的包装类型。

| 基本类型 | 包装类型 |
| --- | --- |
| int | Integer |
| float | Float |
| double | Double |
| byte | Byte |
| char | Character |
| long | Long |
| short | Short |
| boolean | Boolean |

当表格中左边列出的基本类型与它们的包装类有如下几种情况时，编译器会**自动**帮我们进行装箱或拆箱.

*   进行 = 赋值操作（装箱或拆箱）

*   进行+，-，*，/混合运算 （拆箱）

*   进行>,<,==比较运算（拆箱）

*   调用equals进行比较（装箱）

*   ArrayList,HashMap等集合类 添加基础类型数据时（装箱）
<escape><!-- more --></escape>

#### 装箱和拆箱的实现

下方代码是以 Integer 类的一个简单装箱拆箱的例子。

```java
private void  testInteger(){
        Integer i1 = 10;
        int i2 = i1;
    }
```


根据上文装箱拆箱定义可知给 Integer 类型赋 int 类型值的时候会触发装箱操作，而给 int 类型变量赋值 Integer 类型数据的时候则会触发拆箱操作。下图是上方两行代码的字节码。
![boxAndUnbox.png](https://upload-images.jianshu.io/upload_images/19433926-70d01194d08d8103.png?imageMogr2/auto-orient/strip|imageView2/2/w/535/format/webp)

从字节码中可以看出，装箱时其实是调用了 Integer 类的静态方法 valueOf(), 拆箱时是调用了intValue()方法。类似的，Long，Double 等包装类型也是采用相同的方式进行处理。

综上所述，在进行装箱操作的时候，是调用了包装类型的 valueOf() 方法。在进行拆箱的时候，是调用了对应包装的类型的 xxxValue() 方法，xxx是对应的基础类型，如 intValue(), doubleValue()。

#### 实例

##### 1.Integer 类型

```java
Integer i1 = 127;
Integer i2 = 127;
Integer i3 = 128;
Integer i4 = 128;
​
System.out.println("i1 == i2 => " + (i1 == i2));
System.out.println("i3 == i4 => " + (i3 == i4));
​
--output
i1 == i2 => true
i3 == i4 => false
```

根据上文的描述，装箱时是调用 Integer.valueOf(int) 方法生成一个新对象，按理说两个比较的结果都应该是 false，但是为什么 i1 和 i2 的比较结果是相同的呢。既然是调用的 valueOf() 方法，那就需要看下这个方法的实现了。

```java
 public static Integer valueOf(int i) {
 if (i >= IntegerCache.low && i <= IntegerCache.high)
   return IntegerCache.cache[i + (-IntegerCache.low)];
 return new Integer(i);
 }
```

可以看到通过 valueOf(int) 方法初始化一个 Integer 对象的时候，有一个从 IntegerCache 直接取值的操作，IntegerCache 的实现如下：

```java
private static class IntegerCache {
 static final int low = -128;
 static final int high;
 static final Integer cache[];
​
 static {
 // high value may be configured by property
 int h = 127;
 String integerCacheHighPropValue =
 sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
 if (integerCacheHighPropValue != null) {
 try {
 int i = parseInt(integerCacheHighPropValue);
 i = Math.max(i, 127);
 // Maximum array size is Integer.MAX_VALUE
 h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
 } catch( NumberFormatException nfe) {
 // If the property cannot be parsed into an int, ignore it.
 }
 }
 high = h;
​
 cache = new Integer[(high - low) + 1];
 int j = low;
 for(int k = 0; k < cache.length; k++)
 cache[k] = new Integer(j++);
​
 // range [-128, 127] must be interned (JLS7 5.1.7)
 assert IntegerCache.high >= 127;
 }
​
 private IntegerCache() {}
 }
```

通过上面两端实现代码可以发现，当一个 int 型的数字在 [-128,127] 这个区间时，会直接从 IntegerCache 中取值并返回，超过这个区间时才会重新创建新对象。这也就是为什么两个 赋值为127 的两个 Integer 比较的结果是 true，而赋值为 128 的两个 Integer 比较的结果是 false。但这个结果并不绝对，因为 IntegerCache 的 high 是可以配置的，这也就意味着你可以手动改变这个缓存的范围，将 128 加入缓存中。

##### 2.Double 类型

```java
Double d1 = 127.0;
Double d2 = 127.0;
Double d3 = 128.0;
Double d4 = 128.0;
​
ystem.out.println("d1 == d2 => " + (d1 == d2));
System.out.println("d3 == d4 => " + (d3 == d4));
​
--output
d1 == d2 => false
d3 == d4 => false
```

可以看到，double 类型数据在进行装箱的时候似乎没有从缓存直接取值的操作，都是直接创建新对象，这是因为在一个有限的区间内，整型数的个数是有限的，但是浮点数无法用一个有限的几个表示，所以是直接创建新对象。

Double 和 Float 的 valueOf() 方法代码如下：

```java
//Double 
public static Double valueOf(double d) {
 return new Double(d);
 }
​
//Float
 public static Float valueOf(float f) {
 return new Float(f);
 }
```

##### 3.Boolean 类型

```java
Boolean b1 = false;
Boolean b2 = false;
Boolean b3 = true;
Boolean b4 = true;
​
System.out.println("b1 == b2 => " + (b1 == b2));
System.out.println("b3 == b4 => " + (b3 == b4));
​
--output
b1 == b2 => true
b3 == b4 => true
```

这是因为 Boolean.valueOf() 直接返回了两个静态成员变量 Boolean.TRUE 和 Boolean.FALSE，所以比较为 true.

```java
public static Boolean valueOf(boolean b) {
 return (b ? TRUE : FALSE);
 }
​
public static final Boolean TRUE = new Boolean(true);
​
​
public static final Boolean FALSE = new Boolean(false);
```

##### 4.综合测试

```java
Integer a = 1;
Integer b = 2;
Integer c = 3;
Integer d = 3;
Integer e = 321;
Integer f = 321;
Long g = 3L;
Long h = 2L;
​
System.out.println("c == d => " + (c == d));
System.out.println("e == f => " + (e == f));
System.out.println("c == (a + b) => " + (c == (a + b)));
System.out.println("c.equals(a + b) => " + (c.equals(a + b)));
System.out.println("g == (a + b) => " + (g == (a + b)));
System.out.println("g.equals(a + b) => " + g.equals(a + b));
System.out.println("g.equals(a + h) => " + g.equals(a + h));
​
--output
c == d => true
e == f => false
c == (a + b) => true
c.equals(a + b) => true
g == (a + b) => true
g.equals(a + b) => false
g.equals(a + h) => true
```

*   c == d 返回 true，因为是直接从 IntegerCache 取值返回。

*   e == f 返回 false，因为超出 IntegerCache 默认缓存返回，是比较两个新对象地址，所以是 false。

*   c == (a + b) 返回 ture，因为执行了算数运算，会先进行拆箱，然后直接比较值。

*   c.equals(a + b) 返回 true,，因为执行了算数运算，会先进行拆箱，因为是调用 equals，会再进行装箱， Integer.equals() 会调用 intValue() 比较值，所以结果是 true。

*   g == (a + b) 返回 true，拆箱后比较值是否相等。

*   g.equals(a + b) 返回 false，因为先拆箱后装箱为 Integer，而 g 是 Long 类型， 其 equals(obj) 接收到的参数不是 Long 类型时，会返回 false。

*   g.equals(a + h) 返回 true，因为此处触发类型晋升，在装箱时会调用 Long.valueOf()，所以比较结果相同。

