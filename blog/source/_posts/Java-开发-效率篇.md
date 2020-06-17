---
title: Java 开发-效率篇
date: 2020-06-17 13:27:42
tags: Java
---
## Java 开发-效率篇

- **注意SQLServer中的字符串型的字段类型**
  眼尖的同事应该都有发现我们在框架上的有关SQLServer的JDBC连接字符串中的“sendStringParametersAsUnicode=false”参数，遗憾的是，目前为止，仅有一名同学向我发出了疑问。为什么要使用它？答案是为了最大限度地避免出现隐式转换。JDBC协议定义了Java向数据库服务器发送字符串的两种方式：setString()和setNString()，这两个方法分别用于向SQLServer发送ASCII编码和Unicode编码的字符串。我们应该都知道对于Newegg来说，大部分有关字符串的字段都是CHAR或者VARCHAR，并非带“N”的类型，对于MyBatis和大多数数据层的框架来说默认都使用的是setString()方法，但是微软在其JDBC的实现中若不指定“sendStringParametersAsUnicode=false”参数，即便使用setString()方法，实际向服务器发送的仍然是Unicode编码的字符串，这刚好和我们的字段类型相悖，出现难以排查的隐式转换问题（不走索引）。

<escape><!-- more --></escape>

- **正确的日志记录方式**
  记录日志我们一般使用log.debug/info/warn方法，但这些方法并没有惰性求值特性。因此我们在使用日志功能时，当字符串中需要大剂量运算的情况下，要先判定当前的日志开关状态避免额外的开销。仔细观察如下的代码：

  ```java
  LOGGER.debug("ItemBaseInfo Response: " + JSONUtils.obj2String(itemBaseInfoList)); // 这个方法仅在debug级别下才打印日志
   
  if (LOGGER.isDebugEnabled()) {
      LOGGER.debug("ItemBaseInfo Response: " + JSONUtils.obj2String(itemBaseInfoList));
  }
   
  // 加上if判断有什么区别？难道不都是一样的结果吗？
  // 答
  ```

  

- **ObjectMapper开销**
  ObjectMapper属于重量级对象，不允许在方法级别构造它。



- **对于数据库中日期（DATE类型）字段作为WHERE条件的正确处理**
  我们大多数表的DATE类型的字段都会建立索引。为了使我们能使用到索引，我们不能在该字段上使用函数。

  ```java
  WHERE b.InDate > DATEADD(hh, -72, GETDATE())
  // 对比如下
  WHERE DATEDIFF(hh, GETDATE(), b.InDate) < 72
   
  // 前者能使用到索引
  ```

  

- **避免使用“双花括号”来初始化对象**
  “双花括号”实际属于一个“投机”的写法，Java社区从来没将它列为正式的写法或者是推荐的写法。究其原因在于，这种写法会生成一个额外的内部类，且可能会有意外的结果，除开使用Jmockit框架打桩，其它地方应该严格避免使用。参见https://hacpai.com/article/1498563483898。



- **优先使用Java运算，MyBatis运算靠后**
  MyBatis的Mapper.xml文件实际上是一种模板引擎，其中支持很多运算，包括if、when、静态方法调用、对象比较、表达式运算等等，这些运算都使用OGNL引擎来执行的，效率远不如Java代码，因此，在允许的情况下尽量先将参数在Java代码中计算、准备好在传送给MyBatis。



- **yml文件书写注意事项**
  关于yml内容书写问题引发了很多难以识别的错误。yml采用两个空格来缩进，而非制表符。为了避免出现错误，推荐打空格而非使用Tab键。关于yml的详细介绍请参见阮一峰大佬的文章，自行检索。



- **避免“脚本思想”影响Java代码**我们在查阅同事们写的代码的时候发现了这样的写法用于向前端返回一个简单的JSON对象：return new Object() { String name = "MKPL" }，这是不推荐的写法，我们应该以具体的POJO对象代替，或者使用HashMap来返回。



- **Java8 Stream之选择性使用**
  Stream为我们带来了巨大的效率提升（编写代码的效率），但是并不是所有的场景都适用于它，比如forEach并不能代替普通的循环，因为前者不能使用break关键字，我们推荐仅在运算方式较为复杂的时候使用Stream。



- **SQLServer字段转换**很多时候我们为了适配Java端这边的数据类型，不得不在SELECT语句中对数据表中的某些字段进行CAST或者ISNULL操作，其实这些操作在一些情况下可以避免。这取决于Mybatis对于这部分类型的转换兼容，我们推荐不在SQL语句中转换类型（提高SQL效率），而是在Java这边完成类型转换。这里举个例子，MyBatis可以兼容INT和CHAR类型的数据到Boolean的转换。