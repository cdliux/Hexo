---
title: Java Jackson Json 处理_1
date: 2020-06-04 16:55:15
tags: Java
---

目前我们 Java 框架中使用的 Json 处理组件是 Jackson,所以这里学习下 Jackson 一些基础特性。

Jackson 的核心模块由三部分组成。

- jackson-core，核心包，提供基于"流模式"解析的相关 API，它包括 JsonPaser 和 JsonGenerator。 Jackson 内部实现正是通过高性能的流模式 API 的 JsonGenerator 和 JsonParser 来生成和解析 json。
- jackson-annotations，注解包，提供标准注解功能；
- jackson-databind ，数据绑定包， 提供基于"对象绑定" 解析的相关 API （ ObjectMapper ） 和"树模型" 解析的相关 API （JsonNode）；基于"对象绑定" 解析的 API 和"树模型"解析的 API 依赖基于"流模式"解析的 API。

<escape><!-- more --></escape>
#### 基础用法

- ##### ObjectMapper

  ```java
  Seller seller = new Seller("A001", "Lewis", "address1");
  String sellerStr = objectMapper.writeValueAsString(seller);
  System.out.println("objectMapper.writeValueAsString -> " + sellerStr);
  String sellerJson = "{\"sellerId\":\"A001\",\"sellerName\":\"Lewis\",\"sellerAddress\":{\"countryCode\":null,\"stateCode\":null,\"city\":null,\"zipCode\":null,\"address1\":\"address1\",\"address2\":null}}";
  Seller deserializedSeller = objectMapper.readValue(sellerJson,Seller.class);
  ```

  

  ObjectMapper 通过 writeValue 系列方法 将 java 对 象序列化 为 json，并 将 json 存储成不同的格式，String（writeValueAsString），Byte Array（writeValueAsString），Writer， File，OutStream 和 DataOutput。

  ObjectMapper 通过 readValue 系列方法从不同的数据源像 String ， Byte Array， Reader，File，URL， InputStream 将 json 反序列化为 java 对象。

  在使用 Object Mapper的时候需要注意，由于它是一个重量级对象，所以推荐重复使用此对象，官方描述如下：

  > Reuse heavy-weight objects: `ObjectMapper` (data-binding) and `JsonFactory` (streaming API)

  在我们 Java 开发-效率篇中同样也提到了这个问题：

  > **ObjectMapper开销**
  > ObjectMapper属于重量级对象，不允许在方法级别构造它。

  

  相关配置

  ```java
  //反序列化相关配置
  //忽略在 Json 中存在但是 Java 实体中不存在的属性
  objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
  
  //序列化相关配置
  //设置序列化的日期格式
  objectMapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false);
  //忽略为 Null 的属性
   objectMapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
  //忽略为默认值的属性
  objectMapper.setDefaultPropertyInclusion(JsonInclude.Include.NON_DEFAULT);
  ```

  注意：使用过程中不要更改ObjectMappers设置。映射器使用后，由于序列化程序和反序列化程序的缓存，并非所有设置都生效。配置一次实例，第一次使用后不要更改设置。这样做是为了线程安全和性能。

- #### Jackson 常用注解

  由于 Jaskson 的注解部分在开发中还没深入使用过，这里参考的 [Jackson Annotation Examples](https://www.baeldung.com/jackson-annotations),学习常用的注解.

  - #### 序列化注解

    - ##### @JsonAnyGetter

      将 Map 中的每个 key 都序列化为一个标准属性，接受一个 enabled 参数，为 false 时此注解不生效。

      ```java
      public class ExtendableBean {
          private String name;
          private Map<String, String> properties;
      
          @JsonAnyGetter(enabled = true)
          public Map<String, String> getProperties() {
              return properties;
          }
      }    
      ```

      ```java
      --ExtendableBean
      public class ExtendableBean {
          private String name;
          private Map<String, String> properties;
      
          @JsonAnyGetter(enabled = true)
          public Map<String, String> getProperties() {
              return properties;
          }
      }       
      --invoke
      ExtendableBean myBean = new ExtendableBean("myBean");
      myBean.add("prop1","value1");
      myBean.add("prop2","value2");
      String result = objectMapper.writeValueAsString(myBean);
      System.out.println("useJsonAnyGetter -> " + result);
      
      --output
      useJsonAnyGetter -> {"name":"myBean","prop2":"value2","prop1":"value1"}
      ```

    - ##### @JsonGetter

      用于标记一个方法为序列化时的 Getter 方法.根据测试结果，@JsonProperty 也能实现这样的效果，个人理解是为了让表意更加清晰。

      ```java
      --ExtendableBean
      @JsonGetter("sellerName")
          public String getName() {
              return name;
          }
      --invoke
      ExtendableBean myBean = new ExtendableBean("myBean");
      String result = objectMapper.writeValueAsString(myBean);
      System.out.println("useJsonAnyGetter -> " + result);
      
      --output
      useJsonAnyGetter -> {"sellerName":"myBean"}
      ```

    - ##### @JsonPropertyOrder

      用于指定序列化时属性的顺序

      ```java
      --ExtendableBean
      @JsonPropertyOrder({"name","properties","id"})
      public class ExtendableBean {
          private Integer id;
      --invoke
      System.out.println("JsonPropertyOrder 用于指定序列化时属性的顺序");
      ExtendableBean myBean = new ExtendableBean("myBean");
      myBean.setId(Integer.valueOf(1));
      myBean.add("prop1","value1");
      myBean.add("prop2","value2");
      String result = objectMapper.writeValueAsString(myBean);
      System.out.println("useJsonAnyGetter -> " + result);
      --output
      seJsonPropertyOrder -> {"sellerName":"myBean","id":1,"prop2":"value2","prop1":"value1"}
      ```

      同时，可以使用 @JsonPropertyOrder(alphabetic=true) 让结果按字母顺序自动排序。测试时发现，此注解对 Map 类型属性不生效。

    - ##### @JsonRawValue

      用于将属性值中的 Json 值也进行序列化，而不是原样输出.

      ```java
      --RawBean
      public class RawBean {
          private String name;
      
          @JsonRawValue
          private String json;
      
          public RawBean(String name,String json) {
              this.name = name;
              this.json = json;
          }
      }
      --invoke
      RawBean bean = new RawBean("My bean", "{\"attr\":false}");
      String result = objectMapper.writeValueAsString(bean);
      System.out.println("useJsonRawValue -> " + result);
      
      --output
      useJsonRawValue -> {"name":"My bean","json":{"attr":false}}
      notUseJsonRawValue -> {"name":"My bean","json":"{\"attr\":false}"}
      ```

    - ##### @JsonValue

      表明Jackson在序列化该类实例时，只序列化字段的值或方法返回值。 一个类中最多使用一个@JsonValue 注解。否则会抛出 IllegalArgumentException 异常。

      ```java
      --TypeEnumWithValue
      public enum TypeEnumWithValue {
          TYPE1(1, "Type A"),
          TYPE2(2, "Type 2");
      
          private Integer id;
          private String name;
          
          @JsonValue
          public String getName() {
              return name;
          }
      }
      --invoke
      String enumAsString = objectMapper.writeValueAsString(TypeEnumWithValue.TYPE1);
      System.out.println("useJsonValue -> " + enumAsString);
      --output
      useJsonValue -> "Type A
      ```

    - ##### @JsonRootName

      用于将序列化的结果再包装一层。使用时需要开启配置 **SerializationFeature.WRAP_ROOT_VALUE**

      ```java
      --UserWithRoot
      UserWithRoot {
          private Integer id;
      }
      --invoke
      UserWithRoot user = new UserWithRoot(Integer.valueOf(1),"Lewis");
      ObjectMapper mapper = new ObjectMapper();
      mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
      String resultWithRoot = mapper.writeValueAsString(user);
      System.out.println("resultWithRoot -> " + resultWithRoot);
      --output
      resultWithRoot -> {"user":{"id":1,"name":"Lewis"}}
      resultWithoutRoot -> {"id":1,"name":"Lewis"}
      ```

    - ##### @JsonSerialize

      作用在类，字段，getter方法，表示自定义序列化。字段和getter方法上面的优先级比在类上的高。

      ```java
      --CustomDateSerializer
      public class CustomDateSerializer extends StdSerializer<Date> {
          private static SimpleDateFormat formatter
                  = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
      
          public CustomDateSerializer() {
              this(null);
          }
      
          public CustomDateSerializer(Class<Date> t) {
              super(t);
          }
      
          @Override
          public void serialize(
                  Date value, JsonGenerator gen, SerializerProvider arg2)
                  throws IOException, JsonProcessingException {
              gen.writeString(formatter.format(value));
          }
      }
      
      --EventWithSerializer
      public class EventWithSerializer {
          private String name;
      
          @JsonSerialize(using = CustomDateSerializer.class)
          private Date eventDate;
      }
      
      --invoke
      SimpleDateFormat formatter = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");
      String toParse = "20-12-2014 02:30:00";
      Date date = formatter.parse(toParse);
      EventWithSerializer event = new EventWithSerializer("party", date);
      String result = objectMapper.writeValueAsString(event);
      System.out.println("userJsonSerialize -> " + result);
      
      --outout
      userJsonSerialize -> {"name":"party","eventDate":"2014-12-20 02:30:00"}
      ```

      

  - #### 反序列化注解

    - ##### @JsonCreator

      用于指定反序列化时使用的构造函数。当进行反序列化时，带有 @JsonCreator 注解的构造函数被会调用，如果没有@JsonCreator注解，则默认调用无参构造函数。值得注意的是，在添加了有参构造函数的情况下，如果不使用此构造函数进行反序列化，Json 反序列化时需要一个无参构造函数，在这种情况下，需要手动添加无参构造器，否则会报错。

      ```java
      --BeanWithCreator
      public class BeanWithCreator {
          private Integer id;
          private String name;
      
          @JsonCreator()
          public BeanWithCreator(@JsonProperty("id") Integer id, @JsonProperty("firstName") String name) {
              this.id = id;
              this.name = name;
          }
      }
      
      --invoke
      String json = "{\"id\":1,\"firstName\":\"Lewis\"}";
      BeanWithCreator myBean = objectMapper.readValue(json, BeanWithCreator.class);
      ```

    - ##### @JacksonInject

      用于指定反序列化时某个字段取注入的值，而不是从 Json 中读取。

      ```java
      System.out.println("\n@JacksonInject 用于指定反序列化时某个字段取注入的值，而不是从 Json 中读取。");
      InjectableValues inject = new InjectableValues.Std().addValue(Integer.class, Integer.valueOf(1));
      String json = "{\"name\":\"Lewis\"}";
      BeanWithInject bean = objectMapper.reader(inject)
              .forType(BeanWithInject.class)
              .readValue(json);
      ```

    - ##### @JsonAnySetter

      用于在反序列化时，处理一些未知的属性，如将 Json 中的属性加入到 Map 中

      ```java
      String json = "{\"sellerName\":\"Lewis\",\"attr2\":\"val2\",\"attr1\":\"val1\"}";
      ExtendableBean bean = new ObjectMapper()
                .readerFor(ExtendableBean.class)
                .readValue(json);
      ```

    - ##### @JsonSetter

      标记一个方法为序列化时的 Setter 方法。当我们需要读取一些 Json 数据但目标实体类与该数据不完全匹配时，这个注解非常有用。

      ```java
      --MyBean
      public class MyBean {
          public int id;
          private String name;
      
          @JsonSetter("sellerName")
          public void setName(String name) {
              this.name = name;
          }
      }
      --invoke
      String json = "{\"id\":1,\"sellerName\":\"My bean\"}";
      MyBean bean = objectMapper.readValue(json,MyBean.class);
      ```

    - ##### @JsonDeserialize

      使用自定义的反序列化器来进行反序列化，实现StdDeserializer或是其父类JsonSerializer

    - ##### @JsonAlias

      用于在反序列化期间为属性指定一个或多个别名

      ```java
      --AliasBean
      public class AliasBean {
          @JsonAlias({"fName", "f_name"})
          private String firstName;
          private String lastName;
      }
      --invoke
      String json = "{\"f_name\": \"Lewis\", \"lastName\": \"Liu\"}";
      AliasBean bean = objectMapper.readValue(json, AliasBean.class);
      ```

      

  - #### Jackson Property Inclusion Annotations

    - ##### @JsonIgnoreProperties

      作用在类上，用于指定在序列化/反序列化时需要忽略的属性。可以将它看做是@JsonIgnore的批量操作，但它的功能比 @JsonIgnore 要强，比如一个类是代理类，我们无法将将 @JsonIgnore 标记在属性或方法上，此时便可用 @JsonIgnoreProperties 标注在类上，它还有一个重要的功能是作用在反序列化时解析字段时过滤一些未知的属性，否则通常情况下解析到我们定义的类不认识的属性便会抛出异常。

      可以注明是想要忽略的属性列表如@JsonIgnoreProperties({"name"})，

      也可以注明过滤掉未知的属性如@JsonIgnoreProperties(ignoreUnknown=true)

      ```java
      @JsonIgnoreProperties("id")
      public class BeanWithIgnore {
          public Integer id;
          public String name;
      }
      ```

    - ##### @JsonIgnore

      作用在属性上，用于指定在序列化/反序列化时需要忽略的属性。

      ```java
      public class BeanWithIgnore {
      	@JsonIgnore
          public Integer id;
          public String name;
      }
      ```

    - ##### @JsonIgnoreType

      作用在类上，表示忽略该类的所有属性。

    - ##### @JsonInclude

      作用于类或属性，用于忽略值为 Null, Empty, 默认值的属性。

- #### 多态类型处理

  

- #### 通用注解

  - ##### @JsonProperty

    作用在字段或方法上，用来对属性的序列化/反序列化,表明属性在 Json 中对应的属性名称。

    ```java
    --myBean
    public class MyBean {
        private Integer id;
        @JsonProperty("sellerName")
        private String name;
    }
    
    --invoke
    MyBean bean = new MyBean(Integer.valueOf(1),"Lewis");
    String result = objectMapper.writeValueAsString(bean);
    System.out.println("useJsonProperty -> " + result);
    
    --output
    useJsonProperty -> {"id":1,"sellerName":"Lewis"}
    ```

  - ##### @JsonFormat

    用于属性或方法，指定序列化日期/时间值时的格式。

    ```java
    --EventWithFormat
    public class EventWithFormat {
        public String name;
        @JsonFormat(
                shape = JsonFormat.Shape.STRING,
                pattern = "yyyy-MM-dd hh:mm:ss")
        public Date eventDate;
    
        public EventWithFormat(String name,Date eventDate) {
            this.name = name;
            this.eventDate = eventDate;
        }
    }
    --invoke
    SimpleDateFormat df = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");
    df.setTimeZone(TimeZone.getTimeZone("UTC"));
    String toParse = "20-12-2014 02:30:00";
    Date date = df.parse(toParse);
    EventWithFormat event = new EventWithFormat("party", date);
    String result = new ObjectMapper().writeValueAsString(event);
    
    --output
    useJsonFormat -> {"name":"party","eventDate":"2014-12-20 02:30:00"}
    ```

    - ##### @JsonUnwrapped

      用于属性，将有层级结构的对象在序列化的展开为平级。

      ```java
      --UnwrappedUser
      public class UnwrappedUser {
          public Integer id;
      
          @JsonUnwrapped
          public Name name;
      
      
          public static class Name {
              public String firstName;
              public String lastName;
          }
      }
         
      --invoke
      UnwrappedUser.Name name = new UnwrappedUser.Name("Lewis","Liu");
      UnwrappedUser user = new UnwrappedUser(Integer.valueOf(1),name);
      String result = objectMapper.writeValueAsString(user);
      System.out.println("useJsonUnwrapped -> " + result);
      
      --output
      useJsonUnwrapped -> {"id":1,"firstName":"Lewis","lastName":"Liu"}
      ```

    - ##### @JsonView

      视图模板，作用于方法和属性上，用来指定哪些属性可以被包含在JSON视图中，在前面我们知道已经有@JsonIgnore和@JsonIgnoreProperties可以排除过滤掉不需要序列化的属性，可是如果一个POJO中有上百个属性，比如订单类、商品详情类这种属性超多，而我们可能只需要概要简单信息即序列化时只想输出其中几个或10几个属性，此时使用@JsonIgnore和@JsonIgnoreProperties就显得非常繁琐，而使用@JsonView便会非常方便，只许在你想要输出的属性(或对应的getter)上添加@JsonView即可

      ```java
      --Views
      public class Views {
          public static class Public {}
          public static class Internal extends Public {}
      }
      --Item
      public class Item {
          @JsonView(Views.Public.class)
          public int id;
       
          @JsonView(Views.Public.class)
          public String itemName;
       
          @JsonView(Views.Internal.class)
          public String ownerName;
      }
      --invoke
      Item item = new Item(2, "book", "Lewis");
      String result = new ObjectMapper()
               .writerWithView(Views.Public.class)
               .writeValueAsString(item);
      System.out.println("useJsonView -> " + result);
      --output
      useJsonView -> {"id":2,"itemName":"book"}
      ```

    - ##### @JsonFilter

      指定序列化期间要使用的筛选器，只保留通过筛选的属性。

      ```java
      --BeanWithFilter
      @JsonFilter("filter")
      public class BeanWithFilter {
          public int id;
          public String name;
      
          public BeanWithFilter(int id,String name) {
              this.id = id;
              this.name = name;
          }
      }
      --invoke
      FilterProvider filter = new SimpleFilterProvider()
      	.addFilter("filter",SimpleBeanPropertyFilter.filterOutAllExcept("id"));
      BeanWithFilter bean = new BeanWithFilter(1, "MyBean");
      String result = objectMapper
                .writer(filter)
      System.out.println("useJsonFilter -> " + result);
      
      --ouput
      useJsonFilter -> {"id":1}
      ```
 
