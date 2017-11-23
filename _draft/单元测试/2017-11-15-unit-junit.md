# 单元测试基础
<pre>
 大纲：
   JUnit基础用法
   Mock Spy 和 Stubs 对象
   Mockito库的使用
</pre>
## JUnit
### @Test

声明测试方法，要求必须public 返回值void 且没有参数

```java
@Test
public void a() {
} 
```


### assertEquals
通过assertEquals方法来验证结果 

```java
import static org.junit.Assert.*;
public void a() {
  assertEquals(4, 2+2);
  
  assertNull(null)

  assertTrue(true)
}
```

### @Before @BeforeClass @After @AftoerClass

```java
@Before //每个测试方法前都会触发
public void setUp() {}

@After //每个测试方法执行后触发
public void tearDown() 

@BeforeClass //当前类中的测试方法执行前触发
public static void beforeClass() [}  

@AfterClass //当前类中的测试方法执行后触发
public static void afterClass() {}

```

### 验证异常

```java
//验证抛出空指针异常
@Test(expected = NullPointException.class)
public void a() {}
```  

## Mock Spy Stubs对象  

对象 |  方法默认实现  | 是否可以重写覆盖方法  | 是否可验证方法交互| 
---- | ---------------| --------------------- | ----------------- |
Mock | 空实现  | 是 | 是 |
Spy  | 真实实现 | 是 | 是 |
Stubs | 空实现  | 是 | 否

## Mockito库  
提供各种Mock对象的库  
[github](https://github.com/mockito/mockito)

### mock对象  

```java
import static org.mockito.Mockito.*;
@Test
public void a() {
 B b = mock(B.class);
}

```

### spy对象 

```java
import static org.mockito.Mockito.*;
@Test
public void a() {
 B b = spy(B.class);
}
```

### stubs对象 

```java
@Test 
public void a() {
  B b = mock(B.class, withSetting().stubOnly());
}
``` 

### 使用注解的方式模拟  

``` 
public class ATest() {
@Mock
B b;
@Spy
C c;

@Before
public void setUp() {
   MockitoAnnotations.initMocks(this);	
}
}
```

### 模拟方法  

```java
import static org.mockito.Mockito.*;
@Test
public void a() {
  List list = mock(List.class);
  //当调用list的size方法时返回10
  doReturn(10).when(list).size();
  assertEquals(10, list.size());
  //验证调用了size方法
  verify(list).size();
  
  //验证没有调用size方法
  verify(list, never).size();

  //验证按顺序触发
  list.add(1);
  list.size();
  InOrder order = inOrder(list);
  order.verify(list).add(1);
  order.verify(list).size(); 	
}

public void b() {
  List list = mock(List.class);
  list.add(new Object());
  //验证调用了add方法且参数是任何Object类型的
  verify(list).add(any(Object.class)); 
}
```
完整的使用说明 请参照 [Mockito 文档](https://static.javadoc.io/org.mockito/mockito-core/2.9.0/org/mockito/Mockito.htmli)
