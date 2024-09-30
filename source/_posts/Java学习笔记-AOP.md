---
title: Java学习笔记-AOP
date: 2024-09-30 10:19:44
categories: [Java]
---
## 前言
公司培训的前辈提到：AOP可以理解为添加了一个代理类，这个代理类可以在方法执行前后添加一些操作，比如日志记录、事务管理等。这样就可以将核心业务逻辑和横切关注点分离开来，提高代码的可维护性和可扩展性。

这里他举了一个例子，例如每个方法都需要统计运行时间，如果不使用AOP，那么每个方法都需要添加统计时间的代码，这样会导致代码冗余，可维护性差。使用AOP，只需要在一个地方添加统计时间的代码，就可以实现所有方法的统计时间。

这个思想和设计模式中的装饰器模式有点类似，都是在不改变原有代码的情况下，添加新的功能。

假设用装饰器模式来实现，应该怎么实现呢？

### 装饰器模式

**装饰器模式是面向对象的思想**，它允许向一个现有的对象添加新的功能，同时又不改变其结构。装饰器模式是继承关系的一个替代方案。

1. 定义 Service 接口：
```java
public interface Service {
    void performTask();
}
```
2. 原始实现 ServiceImpl：
```java
public class ServiceImpl implements Service {
    @Override
    public void performTask() {
        // 原始任务逻辑
        System.out.println("Performing the task...");
    }
}
```
3. 定义装饰器类 ServiceDecorator：

装饰器类实现了 Service 接口，并组合了一个 Service 实例。我们通过组合模式，将对原始对象的调用进行包装。

```java
public class ServiceDecorator implements Service {
    private final Service decoratedService;

    public ServiceDecorator(Service decoratedService) {
        this.decoratedService = decoratedService;
    }

    @Override
    public void performTask() {
        // 记录开始时间
        long startTime = System.currentTimeMillis();

        // 调用原始方法
        decoratedService.performTask();

        // 记录结束时间
        long endTime = System.currentTimeMillis();
        System.out.println("Task executed in " + (endTime - startTime) + " ms");
    }
}
```
4. 使用装饰器类：

现在，可以使用装饰器模式为 ServiceImpl 添加统计运行时间的功能，而无需修改原始 ServiceImpl 类。

```java
public class Main {
    public static void main(String[] args) {
        // 原始服务类
        Service service = new ServiceImpl();

        // 使用装饰器包装原始服务类
        Service decoratedService = new ServiceDecorator(service);

        // 调用方法，自动统计运行时间
        decoratedService.performTask();
    }
}
```
如果使用AOP又该怎么实现呢？

## AOP是什么？
AOP（Aspect Oriented Programming）即面向切面编程，AOP 是 OOP（面向对象编程）的一种延续，二者互补，并不对立。

AOP 的目的是将横切关注点（如日志记录、事务管理、权限控制、接口限流、接口幂等等）从核心业务逻辑中分离出来，通过动态代理、字节码操作等技术，实现代码的复用和解耦，提高代码的可维护性和可扩展性。OOP 的目的是将业务逻辑按照对象的属性和行为进行封装，通过类、对象、继承、多态等概念，实现代码的模块化和层次化（也能实现代码的复用），提高代码的可读性和可维护性。

以上是我搜索到的内容，但是“面向切面编程”确实抽象，难以理解。

如果核心是不侵入原始代码去增加一些功能。

那么这里很容易联想到C++中的函数指针、回调函数以及后面出现的Lambda表达式，这些都是在函数调用时，传递的是函数的地址，而不是函数的返回值。这样就可以在函数调用前后，执行一些操作。

### 举例说明
这里都以Lambda表达式为例。

```java
public class Main {
    public static void main(String[] args) {
        // 使用lambda传递行为
        performTask(() -> System.out.println("Task executed!"));
    }

    public static void performTask(Task task) {
        task.execute();
    }
}
```

```c++
#include <iostream>
#include <functional>

void performTask(std::function<void()> func) {
    func();
}

int main() {
    performTask([]() {
        std::cout << "Task executed!" << std::endl;
    });
    return 0;
}
```

两者在形式上没有什么很大的区别。

但在实现上还是有一些区别：

|特性	|Java AOC	|C++ 函数作为参数传入|
|---|---|---|
|传递方式	|通过实现接口或继承类创建匿名类实例	|通过函数指针、函数对象、lambda 传递函数|
|类型要求	|必须实现接口或继承类	|直接传递函数，使用函数指针或 lambda|
|语法简洁性	|Java 8 之前语法较冗长，Java 8 后可用 lambda 简化	|Lambda 语法较简洁，函数指针较底层|
|捕获上下文	|只能捕获 final 或 effectively final 变量	|可以捕获任意上下文变量，提供闭包特性|
|灵活性	|必须依赖接口或类	|不需要接口，函数指针和 lambda 都可以使用|
|性能	|由于会生成匿名内部类的实例，可能开销较大	|函数指针直接传递，效率更高|

**尽管思想上相似，但两者在实现上有所不同：**

- Java 更加面向对象。Java 中，匿名内部类和 Lambda 本质上是依赖接口或者类的实现，体现了 Java 面向对象的特性。即便在 Java 8 之后使用 Lambda，依然是函数式接口的简化语法。这意味着 Java 的行为参数化是通过类或接口来进行的。

- C++ 更加底层。C++ 允许更直接地操作函数本身，既可以通过函数指针传递行为，也可以通过 Lambda 捕获上下文变量，并将其作为参数传递。这种设计体现了 C++ 语言的灵活性，不需要依赖面向对象的设计。C++ 中的函数传递更接近于函数式编程的思想

## 使用AOP来实现开头提到的统计时间功能
```java
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Aspect // 声明这个类是一个切面类
@Component // 让 Spring 容器能够管理这个切面。
public class ExecutionTimeAspect {

    // 定义切点：匹配所有在 com.example.service 包下的类和它们的所有方法
    @Around("execution(* com.example.service.*.*(..))")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        // 记录开始时间
        long startTime = System.currentTimeMillis();

        // 执行目标方法
        Object proceed = joinPoint.proceed();

        // 记录结束时间
        long endTime = System.currentTimeMillis();

        // 打印方法执行时间
        System.out.println(joinPoint.getSignature() + " executed in " + (endTime - startTime) + "ms");

        return proceed;
    }
}
```
匹配所有在 `com.example.service` 包下的类和它们的所有方法，所以当执行 `com.example.service` 包下的任意方法时，都会执行 `logExecutionTime` 方法。

## AOP的关键术语
- **切面（Aspect）**：切面是一个类，它包含了一些横切关注点（例如日志记录、事务管理）。在 Spring AOP 中，切面是通过 `@Aspect` 注解声明的 Java 类。
- **连接点（Join Point）**：连接点是在应用执行过程中能够插入切面的点。这些点可以是方法的调用、方法的执行、异常的处理等。在 Spring AOP 中，连接点总是表示方法的执行。
- **通知（Advice）**：通知是切面在特定连接点（Join Point）执行的动作。在 Spring AOP 中，有以下几种类型的通知：
  - **前置通知（Before Advice）**：在连接点之前执行的通知。
  - **后置通知（After Advice）**：在连接点之后执行的通知，无论连接点是否正常执行。
  - **返回通知（After Returning Advice）**：在连接点正常执行后执行的通知。
  - **异常通知（After Throwing Advice）**：在连接点抛出异常后执行的通知。
  - **环绕通知（Around Advice）**：在连接点之前和之后执行的通知。
- **切点（Pointcut）**：切点是一个表达式，它定义了哪些连接点应该被通知。在 Spring AOP 中，切点使用 `@Pointcut` 注解定义。
- **目标对象（Target Object）**：被一个或多个切面通知的对象。
- **代理对象（Proxy Object）**：在 Spring AOP 中，代理对象是 Spring 框架创建的对象，它包含了目标对象的增强方法。
- **织入（Weaving）**：织入是将切面应用到目标对象并创建代理对象的过程。织入可以发生在编译时、类加载时、运行时。
- **引入（Introduction）**：引入允许向现有的类添加新方法和属性。Spring AOP 不支持引入。

### 举例说明

我们有一个 `UserService` 类，它有一个 `login` 方法，登录时我们希望记录这个方法的执行时间。

1. 目标对象 (Target Object):
这是我们想要增强的对象，也就是 `UserService` 类中的 `login` 方法：
```java
public class UserService {

    public void login(String username, String password) {
        // 模拟登录逻辑
        System.out.println("User " + username + " is logging in.");
    }
}
```
2. 切面 (Aspect):我们创建一个切面，用于记录方法执行时间的逻辑。
```java
@Aspect
public class LoggingAspect {

    // 3. 切点 (Pointcut) - 指定在哪些方法上应用增强逻辑
    @Pointcut("execution(* UserService.*(..))")
    public void allUserServiceMethods() {}

    // 4. 通知 (Advice) - 具体的增强逻辑，这里使用环绕通知来记录执行时间
    @Around("allUserServiceMethods()")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        Object proceed = joinPoint.proceed(); // 执行目标方法
        long executionTime = System.currentTimeMillis() - start;
        System.out.println(joinPoint.getSignature() + " executed in " + executionTime + "ms");
        return proceed;
    }
}

```
3. 当我们调用 login 方法时，增强逻辑（记录执行时间）会自动插入：
```java
UserService userService = new UserService();
userService.login("Alice", "password123");
```
输出：
```
User Alice is logging in.
void UserService.login(String, String) executed in 50ms
```
