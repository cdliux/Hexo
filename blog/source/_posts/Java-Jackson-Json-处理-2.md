---
title: Java Jackson Json 处理_2
date: 2020-06-04 18:10:07
tags: Java
---

#### 高级用法

- ##### 泛型反序列化

  在 Jackson 中，如果处理泛型类型的反序列化 ,可以调用 constructXXXType 系列方法构造需要的类型来进行，也可以构造 TypeReference 来反序列化。在我们老版本的 OatuhClient 中可以看到这样的使用方式。

  ```java
  CollectionType javaType = objectMapper.getTypeFactory().constructCollectionType(List.class, MyBean.class);
  List<MyBean> myList = objectMapper.readValue(json, new TypeReference<List<MyBean>>() {});
  List<MyBean> myList2 = objectMapper.readValue(json, javaType);
  ```

  Map 的处理方法类似：

  ```java
  MapType javaType = objectMapper
             .getTypeFactory()
             .constructMapType(HashMap.class,String.class ,MyBean.class);
  HashMap<String,MyBean> beanMap = objectMapper.readValue(json,javaType);
  HashMap<String,MyBean> beanMap2 = objectMapper.readValue(json, new TypeReference<HashMap<String, MyBean>>() {});
  ```

  ​	
<escape><!-- more --></escape>
- ##### 属性可视化

  属性可视化是指在使用 Jackson 时，不是所有属性都可能被序列化/反序列化，和属性及其 Getter/Setter 方法的可见性有关，默认的属性可视化的规则如下：

  - 若该属性修饰符是 public，该属性可序列化和反序列化。
  - 若属性的修饰符不是 public，但是它的 getter 方法和 setter 方法是 public，该属性可序列化和反序列化。因为 getter 方法用于序列化， 而 setter 方法用于反序列化。
  - 若属性只有 public 的 setter 方法，而无 public 的 getter 方 法，该属性只能用于反序列化。

  若想更改默认的属性可视化的规则，需要调用 ObjectMapper 的方法 setVisibility。

  ```java
  objectMapper.setVisibility(PropertyAccessor.FIELD, JsonAutoDetect.Visibility.ANY);
  ```

  PropertyAccessor 用来指定操作的对象，Visibility 用来限定可见性。

  
