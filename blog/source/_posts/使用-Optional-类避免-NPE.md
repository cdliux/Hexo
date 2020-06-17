---
title: 使用 Optional 类避免 NPE
date: 2020-06-04 16:54:35
tags: Java
---

空指针异常在日常开发中大家应该都遇到过，一旦没处理好，程序直接将其抛出将非常影响体验。

在阿里巴巴的 Java 开发手册中针对 NPE 给出了如下的建议：

> 【推荐】防止 NPE，是程序员的基本修养，注意 NPE 产生的场景：  
> 1） 返回类型为基本数据类型，return 包装数据类型的对象时，自动拆箱有可能产生 NPE。  
> 反例：public int f() { return Integer 对象}， 如果为 null，自动解箱抛 NPE。  
> 2） 数据库的查询结果可能为 null。  
> 3） 集合里的元素即使 isNotEmpty，取出的数据元素也可能为 null。  
> 4） 远程调用返回对象时，一律要求进行空指针判断，防止 NPE。  
> 5） 对于 Session 中获取的数据，建议进行 NPE 检查，避免空指针。  
> 6） 级联调用 obj.getA().getB().getC()；一连串调用，易产生 NPE。  
> 正例：使用 JDK8 的 Optional 类来防止 NPE 问题。

目前我们使用的也是 JDK8，那本文就来学习下这个 Java8 中的 Optional 类，了解其基本的使用方式，并尝试改良项目中的代码。
<escape><!-- more --></escape>

#### 创建一个 Optional 实例

Optional 类提供了 3 个静态方法来创建一个 Optional 实例

![static method.png](https://upload-images.jianshu.io/upload_images/19433926-5b9834a00206305b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

- ##### empty()

  直接返回一个空的 Optional

  ```java
  ptional<String> emptyOpt = Optional.empty();
  ```

- ##### of()

  使用一个非空对象创建一个 Optional，如果传入的值是 Null, 会抛出 NPE

  ```java
  Optional<String> notnullOpt = Optional.of("String");
  ```

- ##### ofNullable

  接受一个可以为 Null 的值创建 Optional

  ```java
  Optional<String> nullableOpt = Optional.ofNullable("String");
  ```



#### isPresent() 与 get()

![get和isPresent.png](https://upload-images.jianshu.io/upload_images/19433926-937a20aba9c48e08.png?imageMogr2/auto-orient/strip|imageView2/2/w/1187/format/webp)

- ##### isPresent()

  ```java
  public boolean isPresent() {
  	return value != null;
  }
  ```

  通过 isPresent() 的实现可以它和 if(value != null) 并没有区别，如果只是单纯的想判断一个对象是否为 Null,不推荐使用这种方式，因为还需要新建一个 Optional 对象，这是无意义的。

- ##### get()

  get() 是从一个 Optional 对象中获取值最直接最简单的方式，但是需要注意的是，如果在使用get() 时没有使用 isPresent() 进行 Null 检查，假设 Optional 为空，会抛出 NoSuchElementException。

所以，在实际使用 Optional 类的时候，是需要避免使用这个两个方法的，当代码中出现了这种写法时，应当思考下是否使用传统的方式即可，不要为了使用特性而强行使用。



#### 常用方法介绍

​	Optional 类常用方法使用频率排行

> 1.public</U> Optional</U> map(Function<? super T, ? extends U> mapper)  
> 2.public T orElse(T other)  
> 3.public T orElseGet(Supplier<? extends T> other)  
> 4.public void ifPresent(Consumer<? super T> consumer)  
> 5.public Optional<T> filter(Predicate<? super T> predicate)  
> 6.public</U> Optional</U> flatMap(Function<? super T, Optional</U>> mapper)  
> 7.public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X  

- ##### orElse()

  当 Optional 有值时直接返回值，为 Null 时返回指定的值。

  ```java
  Seller seller = new Seller("A001", "Lewis");
  Optional<Seller> sellerNotNullOpt = Optional.of(seller);
  Optional<Seller> sellerNullOpt = Optional.empty();
  Seller notNullSeller = sellerNotNullOpt.orElse(new Seller("A002", "LewisDefault"));
  Seller nullSeller = sellerNullOpt.orElse(new Seller("A002", System.out.println("orElse() 有值 -> " + notNullSeller.getSellerId());
  System.out.println("orElse() 无值 -> " + nullSeller.getSellerId());
  
  --output
  orElse() 有值 -> A001
  orElse() 无值 -> A002
  ```

- ##### orElseGet()

  当 Optional 有值时直接返回值，为 Null 时调用函数产生结果并返回。

  ```java
  Seller notNullSeller = sellerNotNullOpt.orElseGet(() -> new Seller("A002", "LewisDefault"));
  Seller nullSeller = sellerNullOpt.orElseGet(() -> new Seller("A002", "LewisDefault"));
  System.out.println("orElseGet() 有值 -> " + notNullSeller.getSellerId());
  System.out.println("orElseGet() 无值 -> " + nullSeller.getSellerId());
  
  --output
  orElseGet() 有值 -> A001
  orElseGet() 无值 -> A002
  ```

  orElseGet() 与 orElse() 的区别

  orElse() 方法不管 Optional 是否为 Null 都会调用生成默认值的方法，而 orElseGet() 只会在 Optional 为空的时候调用。

- ##### orElseThrow()

  当 Optional 有值时直接返回值，无值抛出指定的异常。

  ```java
  Seller notNullSeller = sellerNotNullOpt.orElseThrow(() -> new NoSuchElementException("No value present."));
  System.out.println("orElseThrow() 有值 -> " + notNullSeller.getSellerId());
  try {
  	seller nullSeller = sellerNullOpt.orElseThrow(() -> new NoSuchElementException("No value present."));
  } catch (NoSuchElementException exception) {
  	System.out.println("orElseThrow() 无值 -> " + exception.getMessage());
  }
  --output
  orElseThrow() 有值 -> A001
  orElseThrow() 无值 -> No value present.
  ```

- ##### ifPresent()

  存在值则调用指定的函数。

  ```java
  sellerNotNullOpt.ifPresent(s -> System.out.println("存在值，进行方法调用 ->" + seller.getSellerId()));
  sellerNullOpt.ifPresent(s -> System.out.println("不存在值，不进行方法调用"));
  
  --output
  存在值，进行方法调用 ->A001
  ```

  

- ##### map() 

  判断值是否存在，存在则用该值包装一个新的 Optional 返回，值不存在则返回一个空 Optional。

  ```java
  Seller seller = new Seller("A001", "Lewis", "address1");
  Optional<Seller> sellerNotNullOpt = Optional.of(seller);
  Optional<Seller> sellerNullOpt = Optional.empty();
  String notNullAddress = sellerNotNullOpt
            .map(Seller::getSellerAddress)
            .map(Address::getAddress1)
            .orElse("No address1.");
  String nullAddress = sellerNullOpt
             .map(Seller::getSellerAddress)
             .map(Address::getAddress1)
             .orElse("No address1.");
  
  System.out.println("notNullAddress -> " + notNullAddress);
  System.out.println("nullAddress -> " + nullAddress);
  
  --output
  notNullAddress -> address1
  nullAddress -> No address1.
  
  ```

  使用 map() 可以有效的处理级联判断，比如，用传统的方式，上方获取 Address1 的代码会写成这样。

  ```java
  if (seller != null) {
  	Address address = seller.getSellerAddress();
  	if (address != null) {
  		String address1 = address.getAddress1();
  		} else {
  			return "No address1.";
  			}
  	} else {
  		return "No address1.";
      }
  
  ```

  

- ##### flatMap()





- ##### filter()



