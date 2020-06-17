---
title: Java BigDecimal
date: 2020-06-04 18:55:21
tags: Java
---

### Java 中的浮点数运算

``` java
System.out.println(0.2 + 0.1);
System.out.println(0.3 - 0.1);
System.out.println(0.2 * 0.1);
System.out.println(0.3 / 0.1);

--output
0.2 + 0.1 = 0.30000000000000004
0.2 - 0.1 = 0.19999999999999998
0.2 * 0.1 = 0.020000000000000004
0.3 / 0.1 = 2.9999999999999996
```

上方的代码段提示我们 Java 中浮点数的计算结果是不精确的，至于什么，请参考 [浮点数为什么不精确](https://mp.weixin.qq.com/s?__biz=MzAxOTc0NzExNg==&mid=2665513140&idx=1&sn=565517e977ac56904305a4a9f9d65012#rd)。但是商业计算中要求结果必须精确，为了解决这个问题，需要使用 BigDecimal,它可以表示一个任意大小且精度完全准确的浮点数。
<escape><!-- more --></escape>
#### BigDecimal  的常用构造函数
``` java
int intvalue = 3
double doubleValue = 3.3;
String stringValue = "3.3";
BigDecimal decimalInt = new BigDecimal(intvalue);
BigDecimal decimalDouble = new BigDecimal(doubleValue);
BigDecimal decimalString = new BigDecimal(stringValue);

System.out.println("new BigDecimal(intvalue) => " + decimalInt);
System.out.println("new BigDecimal(doubleValue) => " + decimalDouble);
System.out.println("new BigDecimal(stringValue) => " + decimalString);

--output
new BigDecimal(intvalue) => 3
new BigDecimal(doubleValue) => 3.29999999999999982236431605997495353221893310546875
new BigDecimal(stringValue) => 3.3
```
此处注意如果使用 BigDecimal(double val) 初始化 BigDecimal 时，同样会存在精度丢失的问题，因为 double 类型的数据由于上面的提到的计算机无法精确表示浮点数的原因，传入此构造函数的数据同样是不精确的。所以在实际的使用中需要将浮点数转为 String 再使用，因为使用 String 传入的结果是准确的，可预知的。

#### 正确处理 Double 类型的数据
``` java
double doubleValue = 3.3;

BigDecimal decimalDouble1 = new BigDecimal(doubleValue);
BigDecimal decimalDouble2 = new BigDecimal(String.valueOf(doubleValue));
BigDecimal decimalDouble3 = new BigDecimal(Double.toString(doubleValue));
BigDecimal decimalDouble4 = BigDecimal.valueOf(doubleValue);

System.out.println("doubleValue: " + doubleValue);
System.out.println("new BigDecimal(doubleValue) => " + decimalDouble1);
System.out.println("new BigDecimal(String.valueOf(doubleValue)) => " + decimalDouble2);
System.out.println("new BigDecimal(Double.toString(doubleValue)) => " + decimalDouble3);
System.out.println("BigDecimal.valueOf(doubleValue) => " + decimalDouble4);

--output
doubleValue: 3.3
new BigDecimal(doubleValue) => 3.29999999999999982236431605997495353221893310546875
new BigDecimal(String.valueOf(doubleValue)) => 3.3
new BigDecimal(Double.toString(doubleValue)) => 3.3
BigDecimal.valueOf(doubleValue) => 3.3
```
使用 BigDecimal(String val) 或者静态方法 BigDecimal.valueOf(doubleValue) 都能得到准确的结果，valueOf 实际上也是调用 Double.toString() 方法获取的值。

#### 四则运算
``` java
BigDecimal oprator1 = new BigDecimal("4.5");
BigDecimal oprator2 = new BigDecimal("1.3");

BigDecimal addResult = oprator1.add(oprator2);
BigDecimal subResult = oprator1.subtract(oprator2);
BigDecimal multipyResult = oprator1.multiply(oprator2);
try {
BigDecimal divideResult = oprator1.divide(oprator2);
System.out.println("4.5 / 1.3 = " + divideResult);
    } catch (Exception ex) {
System.out.println("4.5 / 1.3 = " + ex.getMessage());
 }

System.out.println("4.5 + 1.3 = " + addResult);
System.out.println("4.5 - 1.3 = " + subResult);
System.out.println("4.5 * 1.3 = " + multipyResult);

--output
4.5 / 1.3 = Non-terminating decimal expansion; no exact representable decimal result.
4.5 + 1.3 = 5.8
4.5 - 1.3 = 3.2
4.5 * 1.3 = 5.85
```
由于 BigDecimal 不是基础数据类型，所以不能使用四则运算符进行运算，需要使用 BigDecimal 类提供的对应方法。

|add(BigDecimal value)   | 加法    |
| ---- | ---- |
|subtract(BigDecimal value)    | 减法   |
|multipyResult(BigDecimal value)    | 乘法   |
|divide(BigDecimal value)    | 除法   |

需要注意的是，当使用 divide(BigDecimal value) 进行除法运算时，如果无法除尽，将抛出一个异常：java.lang.ArithmeticException: Non-terminating decimal expansion; no exact representable decimal result.
所以，在进行除法运算的时候需要指定结果的小数位数以及结果的截断方式。

#### 获取小数位数
``` java
BigDecimal num1 = new BigDecimal("123.12");
BigDecimal num2 = new BigDecimal("123.1200");
BigDecimal num3 = new BigDecimal("1234");

System.out.println("123.12 的小数位数: " + num1.scale());
System.out.println("123.1200 的小数位数: " + num2.scale());
System.out.println("1234 的小数位数: " + num3.scale());

--output
123.12 的小数位数: 2
123.1200 的小数位数: 4
1234 的小数位数: 0
```
使用 scale() 方法可以获取一个 BigDecimal 的小数位数，类似的，在 BigDecimal 各个方法签名中的 scale,都是指小数的位数。

#### 正确地进行除法运算
``` java
BigDecimal oprator1 = new BigDecimal("1");
BigDecimal oprator2 = new BigDecimal("0.33");
BigDecimal divideResult = oprator1.divide(oprator2,4, RoundingMode.DOWN);
BigDecimal divideResult2 = oprator1.divide(oprator2,4, RoundingMode.HALF_UP);
System.out.println("直接截断，1 / 3 = " + divideResult);
System.out.println("四舍五入，1 / 3 = " + divideResult2);
```
使用 divide(BigDecimal divisor, int scale, RoundingMode roundingMode) 方法传入结果保留的小数位数以及结果的截断方式，可以有效的避免除法运算出现异常。
RoundingMode 是一个枚举，包含以下值，常用的四舍五入对应 HALF_UP

#### 正确地比较 BigDecimal
``` java
BigDecimal num1 = new BigDecimal("123.12");
BigDecimal num2 = new BigDecimal("123.1200");
System.out.println("num1: " + num1);
System.out.println("num2: " + num2);
System.out.println("num1.equals(num2) => " + num1.equals(num2));
System.out.println("num1.equals(num2.stripTrailingZeros()) => " + num1.equals(num2.stripTrailingZeros()));
System.out.println("num1.compareTo(num2) => " + num1.compareTo(num2));

--output
num1: 123.12
num2: 123.1200
num1.equals(num2) => false
num1.equals(num2.stripTrailingZeros()) => true
num1.compareTo(num2) => 0
```
使用 equals() 方法可以比较两个 BigDecimal 是否相等，但是此方法不但要求两个数的值相等，还要求 scale() 也相等。所以直接比较 123.12 和 123.1200 的时候会发现结果不是 true。需要使用 stripTrailingZeros() 方法去掉尾部的 0 之后比较结果才相等。
而使用 compareTo() 方法则可以准确的对两个数的值进行比较，返回值是 负数，正数，0，分别表示小于，大于，等于。所以，推荐使用 compareTo() 比较两个 BigDecimal 是否相等。
