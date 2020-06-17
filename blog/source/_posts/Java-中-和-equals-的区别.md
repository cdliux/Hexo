---
title: Java 中 == 和 equals() 的区别
date: 2020-06-04 16:57:44
tags: Java
---
### == 和 equals() 的区别

由于之后学习装箱和拆箱的时候中会使用到 == 和 equals() 两种方式来进行两个对象的对比，所以这里先看下在 Java 中 ==  和 equals()的区别。
<escape><!-- more --></escape>
##### ==

下方代码段分别对两个 int, float，String 类型的数据进行了比较。

```java
int i1 = 333;
int i2 = 333;
System.out.println("i1 == i2 -> " + (i1 == i2));

float f1 = 0.3f;
float f2 = 0.3f;
System.out.println("f1 == f2 -> " + (f1 == f2));

String str1 = new String("hello");
String str2 = new String("hello");

System.out.println("str1 == str2 -> " + (str1 == str2));

--output
i1 == i2 -> true
f1 == f2 -> true
str1 == str2 -> false
```

结果可以看到 int 和 float 类型的数使用 == 进行比较时结果为 true,而 String 类型比较的结果是 false.这是因为int 和 float 都是基本数据类型，变量中存储的就是其本身的值，所以在使用 == 进行比较的时候直接对比的就是值，所以结果是相等的。而 String 是非基本类型（或者描述为引用类型），变量存储的是一个引用，而 str1 和 str2 是通过 new String() 方式生成的两个不同的对象，其在内存中的地址不同，所以使用 == 比较的结果是 false。

而如下代码段则展示了当两个变量引用同一个对象时，其比较的结果。

```java
String str1 = new String("hello");
String str2 = new String("hello"); 
String str3 = new String("no hello");
str1 = str3;
str2 = str3;
System.out.println("str1 == str2 ->" + (str1 == str2));

--output
str1 == str2 -> true
```

当为 str1 和 str2 都赋值为 str3 时，str1 和 str2 存储的都是 str3 值的地址，所以使用 == 对比结果是相同的。

综上所述，== 是直接比较的值，但是因为基本类型变量直接存储值，引用类型存储地址的原因。所以结果表现为基本类型使用 == 比较时直接比较值，而引用类型比较了地址，当两个引用类型的变量真实的值是相同的时候，比较结果也是 false。

##### equals()

equals() 是 Object 类提供的方法，所以所有继承自 Object 类的类都会拥有此方法，其在 Object 类中的实现如下：

```java
public boolean equals(Object obj) {
        return (this == obj);
    }
```

可以到在 Object 类中的实现是直接对比了两个对象的引用。但是当子类重写了 Object 方法时，就需要具体的看各自实现了，如 String 类和 Integer 类。

```java
String str1 = new String("hello");
String str2 = new String("hello");
System.out.println("str1.equals(str2) -> " + (str1.equals(str2)));

Integer i4 = 564;
Integer i5 = 564;
System.out.println("i4 == i5 ->" + (i4.equals(i5)));

--output
str1.equals(str2) -> true
i4.equals(i5) -> true
```

String 类的 equals() 实现如下：

```java
public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```

可以看到当两个 String 对象的引用相同或值相同时都会返回 true,这样就是上个例子中两个 String 比较返回 true 的原因。类似的，Integer 类的实现如下：

```java
    public boolean equals(Object obj) {
        if (obj instanceof Integer) {
            return value == ((Integer)obj).intValue();
        }
        return false;
    }
```

也就是当两个 Integer 类型的对象进行比较的时候，会有拆箱操作，取出参数的 int 值进行对比。但是，这个地方需要注意，在 equals() 方法中有类型判断，当传入的参数不是 Integer 类型时，会直接返回 false,这也可以理解为 equals() 方法不会进行类型转换。

综上所述，equals() 方法默认比较的是两个对象的引用，但当子类有重写行为时，需要看具体实现，Integer，Long，Double 等包装类型都重写了此方法，进行值的比较。

根据以上结论，我们在开发中需要注意: 包装类型间的相等判断应该用equals，而不是'=='.