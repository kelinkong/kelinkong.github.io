---
title: Java学习笔记-单元测试
date: 2024-09-26 10:18:29
categories: [Java]
---

## 前言
使用Mockito和JUnit5进行单元测试。

需要注入依赖：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>3.12.4</version>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <scope>test</scope>
</dependency>
```
### 基础知识
- 使用@Test注解标记测试方法。
- 测试代码和被测试代码放到同一个包中。
### 示例背景
- 假设有一个服务类`MockService`，其中有一个`serviceMethod`方法。
```java
public class MockService {
    public String serviceMethod(String name) ;
}
```
- 假设有一个临时类`MockTmp`，其中有一个`tmpMethod`方法。`MockTmp`的构造器需要传入一个`MockService`对象。
```java
public class MockTmp {

    private final MockService mockService;

    public MockTmp(MockService mockService);

    public String tmpMethod(String name);
}
```
- 假设有一个被测试类`MockUser`，其中包含了`MockService`类、一个私有方法、一个静态方法、一个公有方法。
```java
public class MockUser {

    private final MockService mockService;

    public MockUser(MockService mockService);

    private String privateMethod(String name) ;

    public static String staticMethod(String name);

    public String publicMethod(String name);
}
```

测试将会覆盖：私有方法测试、静态方法测试、公有方法测试、构造器测试。
## 基本用法
### 创建Mock对象
1. 使用`@Mock`注解或`Mockito.mock`方法创建Mock对象。
2. 使用`@Spy`注解或`Mockito.spy`方法创建Mock对象。
3. 使用`MockBean`注解创建Mock对象。
4. 使用`@InjectMocks`注解或`MockitoAnnotations.initMocks`方法注入Mock对象。

```java
    @Mock
    private MockService mockService;

    @Spy
    @InjectMocks
    private MockUser mockUser;

    @BeforeEach
    public void setUp(){
        MockitoAnnotations.openMocks(this);
    }
```

**注意**：`@Mock`和`@Spy`注解的区别：
- `@Mock`注解创建的对象是一个真正的Mock对象，它会覆盖被Mock的方法。
- `@Spy`注解创建的对象是一个真实的对象，它会保留被Mock的方法的原始实现。

### Mock方法
1. 使用`Mockito.when`方法模拟方法调用。
2. 使用`Mockito.doReturn`方法模拟方法调用。
3. 使用`Mockito.doThrow`方法模拟方法调用。
4. 使用`Mockito.doNothing`方法模拟方法调用。
5. 使用`Mockito.doAnswer`方法模拟方法调用。
...
    
```java
Mockito.when(mockService.serviceMethod("Tom")).thenReturn("Hello, Tom!");

Mockito.doReturn("Hello, Tom!").when(mockService).serviceMethod("Tom");

Mockito.doThrow(new RuntimeException()).when(mockService).serviceMethod("Tom");

Mockito.doNothing().when(mockService).serviceMethod("Tom");
...
```
### verify方法和assert方法
1. 使用`Mockito.verify`方法验证方法调用。
2. 使用`Assert.assertEquals`方法断言方法调用。
3. 使用`Assert.assertTrue`方法断言方法调用。
...
```java
Mockito.verify(mockService, Mockito.times(1)).serviceMethod("Tom");

Assert.assertEquals("Hello, Tom!", mockService.serviceMethod("Tom"));

Assert.assertTrue("Hello, Tom!".equals(mockService.serviceMethod("Tom")));
...
```

## 示例代码
### 测试私有方法
**被测方法**：
```java
private String privateMethod(String name) {
    System.out.println("Private Method Called");
    name = name + " private";
    return name;
}
```
**测试方法**：
```java
@Test
void privateMethod() throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
    String name = "test";
    // 通过反射调用私有方法
    Method method = mockUser.getClass().getDeclaredMethod("privateMethod", String.class);
    method.setAccessible(true);
    String result = (String) method.invoke(mockUser, name);
    assertEquals("test private", result);
}
```
mockito无法直接模拟私有方法，需要通过反射调用。

### 测试静态方法
**被测方法**：
```java
public static String staticMethod(String name) {
    System.out.println("Static Method Called");
    name = name + " static";
    return name;
}
```
**测试方法**：
```java
@Test
void staticMethod() {
    String name = "test";
    // 通过类名调用静态方法
    String result = MockUser.staticMethod(name);
    assertEquals("test static", result);
}
```

### 测试公有方法
被调用的所有方法都mock掉，只测试公有方法。这样可以保证测试的是公有方法的逻辑，而不是依赖的其他方法。
**被测方法**：
```java
public String publicMethod(String name) {
    try{
        // 调用其他服务  name + " service"
        String name1 = mockService.serviceMethod(name);
        // 调用类内私有方法  name + " private"
        String name2 = privateMethod(name);
        // 调用静态方法 name + " static"
        String name3 = staticMethod(name);
        // 不带参的构造函数 name + " service"
        MockService mockService2 = new MockService();
        String name4 = mockService2.serviceMethod(name);
        // 带参的构造函数 name + " tmp"
        MockTmp mocktmp = new MockTmp(mockService2);
        String name5 = mocktmp.tmpMethod(name);

        return name1 + " " + name2 + " "  + name3 + " "  + name4 + " "  + name5;
    } catch (RuntimeException e){
        throw new RuntimeException("Exception");
    }
}
```
**测试方法**：
```java
@Test
void publicMethod() {
    String name = "test";
    // 想要最后的结果都是 "test"
    Mockito.when(mockService.serviceMethod(anyString())).thenReturn("test");

    /* 静态方法不能直接在stubbing中调用 */
    //Mockito.when(MockUser.staticMethod(anyString())).thenReturn("test");

    // 同一个类内的私有方法无法mock
    //Mockito.when(mockUser.privateMethod(anyString())).thenReturn("test");

    // mock新对象和静态方法
    try(MockedStatic<MockUser> mockUserStatic = Mockito.mockStatic(MockUser.class);
        MockedConstruction<MockService> service = Mockito.mockConstruction(MockService.class, (mock, context) -> {
            Mockito.when(mock.serviceMethod(anyString())).thenReturn("test");
        });
        MockedConstruction<MockTmp> tmp = Mockito.mockConstruction(MockTmp.class, (mock, context) -> {
            Mockito.when(mock.tmpMethod(anyString())).thenReturn("test");
        })
        ){

        mockUserStatic.when(() -> MockUser.staticMethod(anyString())).thenReturn("test");

        String result = mockUser.publicMethod(name);
        assertEquals("test test private test test test", result);
    }
}
```
最后输出的结果是`test test private test test test`。

可以看出，只有`privateMethod`方法没有被`mock`掉，其他方法都被`mock`掉了。

目前还没有找到方法去`mock`类内私有方法（基于不能修改源代码、也不能使用powerMockito）。（如果有大佬看到，求教，邮箱1763605980@qq.com）

#### 异常处理
```java
@Test
public void publicMethodException() {
    String name = "test";
    Mockito.doThrow(new RuntimeException()).when(mockService).serviceMethod(anyString());
    assertThrows(RuntimeException.class, () -> mockUser.publicMethod(name));
}
```
## 完整代码
`MockService`类：
```java
@Service
public class MockService {
    public String serviceMethod(String name) {
        System.out.println("serviceMethod");
        name = name + " service";
        return name;
    }
}
```
`MockTmp`类：
```java
package com.example.demo.mockito;

public class MockTmp {
    private final MockService mockService;

    public MockTmp(MockService mockService){
        this.mockService = mockService;
    };

    public String tmpMethod(String name) {
        return name + " tmp";
    }
}
```
`MockUser`类：
```java
package com.example.demo.mockito;

public class MockUser {
    private final MockService mockService;

    public MockUser(MockService mockService){
        this.mockService = mockService;
    };

    private String privateMethod(String name) {
        System.out.println("Private Method Called");
        name = name + " private";
        return name;
    }

    public static String staticMethod(String name) {
        System.out.println("Static Method Called");
        name = name + " static";
        return name;
    }

    public String publicMethod(String name) {
        try{
            // 调用其他服务  name + " service"
            String name1 = mockService.serviceMethod(name);
            // 调用类内私有方法  name + " private"
            String name2 = privateMethod(name);
            // 调用静态方法 name + " static"
            String name3 = staticMethod(name);
            // 不带参的构造函数 name + " service"
            MockService mockService2 = new MockService();
            String name4 = mockService2.serviceMethod(name);
            // 带参的构造函数 name + " tmp"
            MockTmp mocktmp = new MockTmp(mockService2);
            String name5 = mocktmp.tmpMethod(name);

            return name1 + " " + name2 + " "  + name3 + " "  + name4 + " "  + name5;
        } catch (RuntimeException e){
            throw new RuntimeException("Exception");
        }
    }
}
```
`MockUserTest`类：
```java
package com.example.demo.mockito;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.*;

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.ArgumentMatchers.anyString;

class MockUserTest {

    @Mock
    private MockService mockService;

    @Spy
    @InjectMocks
    private MockUser mockUser;

    @BeforeEach
    public void setUp(){
        MockitoAnnotations.openMocks(this);
    }

    @Test
    void privateMethod() throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        String name = "test";
        Method method = mockUser.getClass().getDeclaredMethod("privateMethod", String.class);
        method.setAccessible(true);
        String result = (String) method.invoke(mockUser, name);
        assertEquals("test private", result);
    }

    @Test
    void staticMethod() {
        String name = "test";
        String result = MockUser.staticMethod(name);
        assertEquals("test static", result);
    }

    @Test
    void publicMethod() {
        String name = "test";
        // 想要最后的结果都是 "test"
        Mockito.when(mockService.serviceMethod(anyString())).thenReturn("test");

        /* 静态方法不能直接在stubbing中调用 */
        //Mockito.when(MockUser.staticMethod(anyString())).thenReturn("test");

        // 同一个类内的私有方法无法mock
        //Mockito.when(mockUser.privateMethod(anyString())).thenReturn("test");

        // mock新对象和静态方法
        try(MockedStatic<MockUser> mockUserStatic = Mockito.mockStatic(MockUser.class);
            MockedConstruction<MockService> service = Mockito.mockConstruction(MockService.class, (mock, context) -> {
                Mockito.when(mock.serviceMethod(anyString())).thenReturn("test");
            });
            MockedConstruction<MockTmp> tmp = Mockito.mockConstruction(MockTmp.class, (mock, context) -> {
                Mockito.when(mock.tmpMethod(anyString())).thenReturn("test");
            })
            ){

            mockUserStatic.when(() -> MockUser.staticMethod(anyString())).thenReturn("test");

            String result = mockUser.publicMethod(name);
            assertEquals("test test private test test test", result);
        }
    }

    @Test
    public void publicMethodException() {
        String name = "test";
        Mockito.doThrow(new RuntimeException()).when(mockService).serviceMethod(anyString());
        assertThrows(RuntimeException.class, () -> mockUser.publicMethod(name));
    }
}
```

## 期望抛出异常
```java
@Test(expected = RuntimeException.class)
public void publicMethodException() {
    String name = "test";
    Mockito.doThrow(new RuntimeException()).when(mockService).serviceMethod(anyString());
    assertThrows(RuntimeException.class, () -> mockUser.publicMethod(name));
}
```

doThrow() 与 thenThrow() 区别（用于 stubbing 有返回值方法）
- doThrow()：用于 void 方法的 stubbing。
- thenThrow()：用于有返回值的方法的 stubbing。

```java
// void 方法
doThrow(new RuntimeException()).when(mockObject).voidMethod();
// 有返回值的方法
when(mockObject.method()).thenThrow(new RuntimeException());
```


如果 **源代码中有 `try-catch` 块**，而在测试方法上使用了：

```java
@Test(expected = Exceptions.class)
```

那你**根本无法覆盖 `catch` 中的内容**，原因是：

**JUnit 的 `@Test(expected = ...)` 会中断执行流程**

当测试中抛出指定异常时，JUnit 会立即判定测试通过，不再执行后续逻辑：

* 如果源代码中 **没有 catch**，这个方式可以验证是否真的抛出了异常。
* 但如果源代码中 **已经 catch 掉异常**，异常就不会冒泡到测试层，`@Test(expected = ...)` 就**无效**。
* 即使异常冒泡成功了，**`catch` 块内容也不会执行到**，所以覆盖率里 catch 是“未执行”。

---

### ✅ 要覆盖 catch 中的代码，正确做法是：

**不使用 `@Test(expected = ...)`，而是让测试代码走完整流程，触发并验证 catch 块被执行**。

```java
@Test
public void testRemoveUser_withException() {
    // mock 抛出异常
    when(userService.deleteUserById(1L)).thenThrow(new Exceptions("boom"));

    // 调用实际方法，触发 try-catch
    controller.removeUser(1L);

    // 可以用日志检查、状态标志、verify 等方式验证 catch 是否执行
}
```