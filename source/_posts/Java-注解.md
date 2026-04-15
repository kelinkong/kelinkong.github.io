---
title: Java-注解
date: 2026-04-14 15:47:45
categories: [Java]
---
## Java的注解究竟是什么？

注解本质上是一个接口，继承了 java.lang.annotation.Annotation 接口，注解的成员变量都是抽象方法，注解的成员变量只能是基本数据类型、String、Class、枚举、注解类型或者以上类型的数组。
```java
@interface Entity {}
```
等价于：
```java
public interface Entity extends Annotation {}
```

注解只是配置，它本身并没有任何功能，注解的功能是由使用注解的框架或者工具来实现的。注解可以用来生成代码、编译时检查、运行时反射等。

### 实例
实现一个枚举值校验的注解：
```java
@Documented // 表示该注解会被 javadoc 记录
@Target(ElementType.FIELD) // 表示该注解只能用在字段上
@Retention(RetentionPolicy.RUNTIME) // 表示该注解在运行时仍然可用
@Constraint(validatedBy = EnumValueValidator.class) // 表示该注解由 EnumValueValidator 来校验
public @interface EnumValue {
    String message() default "Invalid enum value"; // 校验失败时的默认错误消息
    Class<?>[] groups() default {}; // 分组校验时使用
    Class<? extends Payload>[] payload() default {}; // 负载信息，可以用来携带一些额外的信息
    String field() default ""; // 枚举类的字段名，默认为空
    Class<? extends Enum<?>> enumClass(); // 枚举类的 Class 对象，必须要指定
}
```
实现一个校验器：
```java
public class EnumValueValidator implements ConstraintValidator<EnumValue, String> {
    private Class<? extends Enum<?>> enumClass; // 枚举类的 Class 对象, 由注解传入
    
    @Override
    public void initialize(EnumValue constraintAnnotation) {
        this.enumClass = constraintAnnotation.enumClass();
    }

    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        if (value == null) {
            return true; // 可以根据需要决定是否允许 null 值
        }
        for (Enum<?> enumConstant : enumClass.getEnumConstants()) {
            try {
                String code = (String) enumConstant.getClass().getMethod("getCode").invoke(enumConstant);
                if (code.equals(value)) {
                    return true; // 找到匹配的枚举常量，校验通过 
                }
            }
        }
        return false; // 如果没有匹配的枚举常量，则校验失败
    }
```
使用：
```java
public class User {
    @EnumValue(enumClass = UserType.class, message = "Invalid user type")
    private String userType;
    // 其他字段和方法
}
```

```text
@EnumValue(enumClass=Gender.class)
                │
                ▼
      Bean Validation 框架
                │
                ▼
initialize(EnumValue annotation)
                │
                ▼
annotation.enumClass()
                │
                ▼
EnumValueValidator 使用该枚举校验
```

### 以事务注解为例：
```java
@Transactional
```

Spring启动时：
```text
扫描注解
↓
发现@Transactional
↓
创建代理
↓
开启事务
```

事务只保证要么成功，要么失败，不保证并发安全，事务的隔离级别可以设置为 READ_COMMITTED、REPEATABLE_READ、SERIALIZABLE 等。

### 注解的原理

```java
@EnumValue(enumClass = StatusEnum.class)
private Integer status;
```

编译后：
.class文件会包含一个注解属性，里面有一个名为 enumClass 的成员变量，值为 StatusEnum.class。

```text
字段 status：
    注解 = EnumValue
    参数 enumClass = StatusEnum.class
```

## 保留级别

注解的保留级别决定了注解在编译后是否存在，以及是否可以通过反射访问：
- SOURCE：注解只存在于源代码中，编译后会被丢弃，无法通过反射访问。
- CLASS：注解存在于编译后的字节码中，但在运行时无法通过反射访问。
- RUNTIME：注解存在于编译后的字节码中，并且在运行时可以通过反射访问。

## 注解和装饰器模式的区别
注解和装饰器模式都是在不改变原有代码的情况下，添加新的功能，但是它们的实现方式不同。注解是通过反射来实现的，而装饰器模式是通过组合来实现的。注解是一种静态的配置，而装饰器模式是一种动态的设计模式。注解通常用于框架或者工具中，而装饰器模式通常用于业务逻辑中。

## 为什么 Annotation 在 JVM 里其实是一个“动态代理对象”？

最原始的情况：
```java
UserService s = new UserServiceImpl();
```

用spring之后：
```java 
@Autowired
UserService s;
```
Spring会扫描到 UserService 接口上的 @Autowired 注解，然后通过反射创建一个 UserService 的代理对象，这个代理对象实现了 UserService 接口，并且在调用方法时会执行增强逻辑（如事务、日志等）。所以在运行时，s 实际上是一个动态代理对象，而不是 UserServiceImpl 的实例。
```java
UserService s = (UserService) Proxy.newProxyInstance(
    UserService.class.getClassLoader(),
    new Class<?>[]{UserService.class},
    new InvocationHandler() {
        private UserServiceImpl target = new UserServiceImpl();
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            // 增强逻辑（如事务、日志等）
            return method.invoke(target, args); // 调用目标对象的方法
        }
    }
);
```

本质上，注解只是一个标记，真正的功能是由使用注解的框架或者工具来实现的，这些框架或者工具会在运行时通过反射来读取注解的信息，并根据这些信息来创建动态代理对象，从而实现增强功能。

## 为什么类内@Transactional注解的方法调用会失效？


常规的调用：

```java
public Object invoke(Method method, Object[] args) {

    TransactionAttribute attr = getAnnotation(method);

    if (attr == null) {
        return method.invoke(target, args);
    }
    return invokeWithinTransaction(method, args, attr);
}
```

```java
public Object invokeWithinTransaction(Object proxy, Method method, Object[] args) throws Throwable {

    // ❗ 1. 开事务
    beginTransaction();

    try {
        // ❗ 2. 调用真实方法
        Object result = method.invoke(target, args);

        // ❗ 3. 提交事务
        commit();

        return result;

    } catch (Exception e) {
        // ❗ 4. 回滚事务
        rollback();
        throw e;
    }
}
```
类内调用：
```java
@Service
public class OrderService {

    public void A() {
        B(); // 直接调用B方法
    }

    @Transactional
    public void B() {}
}
```
实际代理调用流程：
```text
invoke(A)
  ↓
A 没 @Transactional → 直接执行
  ↓
target.A()
  ↓
this.B() // 这里B是真实对象的调用，没有经过代理
```
