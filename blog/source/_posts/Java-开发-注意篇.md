---
title: Java 开发-注意篇
date: 2020-06-17 13:36:32
tags:
---
- **replace和replaceAll**

  很多同学到现在可能都还没有弄清楚在Java中这两者之间的区别，可能是受其他语言的影响，也有可能是看字面意思只认“replaceAll”。事实上，这两个都是用于将一个字符串里的所有指定的字符或者字符串替换为目标字符或字符串，并非是明面上的“替换”和“替换所有”。唯一的区别在于，前者只支持普通字符或字符串的替换，而后者支持正则表达式匹配替换（接受的第一个参数是正则表达式，这要特别注意，因为很多字符会被当成是正则的特殊字符）。从效率上来说前者优于后者，所以，视情况选择吧。


<escape><!-- more --></escape>
- **MyBatis Boolean转换之谜**

  MyBtais可以直接将数据库中的CHAR或者是INT字段映射为Java中的Boolean类型，但是转换会有一些区别：
  -- 对于CHAR型的数据：当且仅当为‘1’时，所映射的Boolean才会是true，其他的有效值全部映射为false。为NULL时映射为NULL。
  -- 对于INT型的数据：当且仅当为0时，所映射的Boolean才会是false，其他的有效值（包括负数）映射为true。为NULL时映射为NULL。



- **SQLServer JDBC对于Unicode字符串的处理方式**

  SQLServer对JDBC标准有自己的实现。我们在实际开发中最容易被坑的地方在于微软对PrepareStatement.setString()方法的实现。实时上JDBC在PrepareStatement中规定了两个方法用于向数据库发送字符串形式的参数：setString()和setNString()，前者在于向数据库服务器发送非unicode编码的数据，后者用于发送unicode编码的数据。微软的实现默认两个方法都向SQLServer发送unicode编码的字符串。这便是我们在使用JDBC向SQLServer的非Unicode（char/varchar）字段写入数据时变慢的真正原因（存在隐式转换。对于Unicode字段不受影响，可能是微软觉得默认情况下使用unicode字段的场景较多）。但对于Newegg来说，大量字段使用了非unicode编码，这意味着在默认情况下使用Java JDBC写入Newegg数据库会出现走不了索引的严重问题。因此java-framework在每个连接字符串后都附加了“sendStringParametersAsUnicode=false”的参数，意在告诉SQLServer JDBC将所有的字符串看作是非Unicode编码的字符串。但是又出现了另外一个问题：乱码，将含中文的非Unicode的字符串写入到Unicode编码的字段会出现乱码。这是个取舍，今后，遇到Unicode编码的字段需要额外注意，在MyBatis中使用我们提供的UnicodeStringTypeHandler和UnicodeSqlXmlTypeHandler来处理。



- **Mockito和SpringBoot的集成**

  Mockito使用动态代理实现mock对象。@InjectMock可以帮助我们将@Mock的对象注入到目标对象里，但是在SpringBoot环境下，很多Controller是被CGLIB代理的Controller，@InjectMock对此会失效。为了确保万无一失，我们在书写Junit代码的时候务必使用Spring提供的ReflectionUtils进行手动注入：

  ```java
  @ExtendWith(SpringExtension.class)
  @ContextConfiguration(classes = Application.class,
      loader = MkplTestingApplicationContextLoader.class)
  @AutoConfigureMockMvc
  @WebAppConfiguration
  public class ManufacturerOwnershipControllerTest {
      @Autowired
      private MockMvc mockMvc;
   
      @Autowired
      @InjectMocks // 这个注解不是万能的，这里期望将mockService注入到Controller，但是如果controller是被CGLIB代理过的，注入将失效
      private ManufacturerOwnershipController controller;
   
      @Mock
      private ManufacturerOwnershipService mockService;
   
      @SuppressWarnings("serial")
      @BeforeEach
      public void initMocks(){
          MockitoAnnotations.initMocks(this);
          ReflectionTestUtils.setField(controller, "manufacturerOwnershipService", mockService); // 这是重点
      }
  }
  ```


- MyBatis XML映射文件中使用内部类

  在MyBatis的映射文件中使用内部类应该使用“$”作为分隔符，而不再是FQCN规定的“.”。