---
title: Java-项目开发记录
date: 2025-09-11 11:12:45
categories: [Java]
---

### mybatis-plus相关

**软删除：**

使用标志位进行软删除，通常在数据库表中添加一个`deleted`字段，标记记录是否被删除。通过在查询时过滤掉`deleted`字段为`true`的记录，实现软删除功能。

在MyBatis-Plus中，可以通过`@TableLogic`注解来实现软删除功能。只需在实体类的`deleted`字段上添加该注解，MyBatis-Plus会自动处理软删除逻辑。

```java
public class User {

    private Long id;

    private String name;

    // 逻辑删除标志: 0代表未删除，1代表已删除
    @TableLogic(value = "0", delval = "1")
    private Integer deleted;
}
```
使用：
```java
List<User> users = userMapper.selectList(null);
```
会自动拼接为：

```sql
SELECT id, name, type, deleted
FROM user
WHERE deleted = 0
```

**在数据库中查询时，使用mybaits plus，Ipage有可能返回null吗？还是说只会返回零条记录？**

在使用MyBatis Plus进行分页查询时，如果查询结果为空，则返回的IPage对象中的total属性为0，而不是null。

如果查找不到记录，使用gettRecords()方法获取list得到的结果为[];

### 请求和响应

在请求和响应中，有一些注解可以帮助我们更好地处理数据：
```java
@RequestBody // 用于将请求体中的JSON数据转换为Java对象
@ResponseBody // 用于将Java对象转换为JSON格式的响应体
@PathVariable // 用于从URL路径中提取变量
@RequestParam // 用于从请求参数中提取数据
```
在请求和响应中vo对象中，有一些注解可以帮助我们更好地处理数据：
```java
@JsonFormat // 用于格式化日期和时间
@Max // 用于指定字段的最大值
@Min // 用于指定字段的最小值
@NotBlank // 用于验证字符串字段不能为空
@NotNull // 用于验证字段不能为空
@Size // 用于验证字符串或集合的大小
```

**在写请求的vo时，应该去校验前端返回的字段是否正确。**
比如：
```java
public class UserRequestVO {
    @NotBlank(message = "用户名不能为空")
    private String username;
}
```

使用validator去校验，并把错误信息返回给前端：
```java
Set<ConstraintViolation<UserRequestVO>> violations = validator.validate(userRequestVO);
if (!violations.isEmpty()) {
    StringBuilder errorMessage = new StringBuilder();
    for (ConstraintViolation<UserRequestVO> violation : violations) {
        errorMessage.append(violation.getMessage()).append("; ");
    }
    throw new IllegalArgumentException(errorMessage.toString());
}
```

#### 在action中，使用@Valid注解，自动校验请求参数
```java
@PostMapping("/users")
public ResponseEntity<Void> createUser(@Valid @RequestBody UserRequestVO userRequestVO) {}
```

### 对枚举值进行校验
示例枚举类：
```java
public enum AudioType {
    NORMAL(0, "普通音频"),
    TRIAL(1, "试听音频"),
    VIP(2, "会员音频");

    private final int code;
    private final String desc;
}
```
创建一个枚举值校验器：
```java
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = EnumValidator.class)
public @interface EnumValid {
    String message() default "枚举值不合法";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};

    Class<? extends Enum<?>> enumClass();
}

public class EnumValidator implements ConstraintValidator<EnumValid, Object> {
    private Class<? extends Enum<?>> enumClass;

    @Override
    public void initialize(EnumValid annotation) {
        this.enumClass = annotation.enumClass();
    }

    @Override
    public boolean isValid(Object value, ConstraintValidatorContext context) {
        if (value == null) return true; // 允许空
        for (Enum<?> e : enumClass.getEnumConstants()) {
            if (e.name().equals(value.toString())) return true;
        }
        return false;
    }
}
```

### IDEA配置

#### 自动导包
File -> Settings -> Editor -> General -> Auto Import -> Add unambiguous imports on the fly
#### 自动格式化代码
File -> Settings -> Editor -> Code Style -> Java -> Formatter Control -> Enable formatter markers in comments
#### 自动删去无用的包
File -> Settings -> Editor -> General -> Auto Import -> Optimize imports on the fly
#### 自动添加类描述
File -> Settings -> Editor -> File and Code Templates -> Class -> 在模板开头添加注释

### 异常处理

不能抛出非受检异常，应该定义自定义异常，常见的：BizException、RuntimeException等
```java
public void someMethod() throws BizException {
    // 业务逻辑
}
```

#### 全局异常处理


### 代码规范

1、抛出异常的地方都需要记录日志
2、日志的格式：时间+类名+方法名+异常信息：
```java
log.error("时间：{}，类名：{}，方法名：{}，异常信息：{}", LocalDateTime.now(), this.getClass().getName(), "方法名", e.getMessage());
```
3、如果需要有空返回，先返回空，而不是先走业务逻辑

### Spring Boot相关

#### @Value注解

为什么@Value注解可以注入配置文件中的值？

> **Spring 在创建 Bean 时，会使用专门的处理器（BeanPostProcessor）扫描 `@Value` 注解，把 `${xxx}` 转成真正的配置值，然后通过反射赋值给字段。**

这背后分三步：

##### **1. Spring 会加载配置文件进 Environment**

`application.yml` / `application.properties`
➡ 会被 Spring 解析成一堆 key-value
➡ 存进 **Environment**（它是所有配置的统一容器）

例如：

```yaml
app.name: demo
```

Spring 会存成：

```
Environment["app.name"] = "demo"
```

##### **2. Spring 创建 Bean 时，会扫描字段上的 @Value**

Spring 在创建 Bean 时，会用到一个特殊的处理器：

```
AutowiredAnnotationBeanPostProcessor
```

它会检查字段上是否有：

* `@Autowired`
* `@Value`
* `@Resource`
* ……等

看到 `@Value("${app.name}")`
➡ 它会把 `${app.name}` 交给 **PlaceholderResolver** 去解析。

##### **3. Spring 解析占位符并通过反射把值注入字段**

`${app.name}`
➡ 解析得到 `"demo"`
➡ 反射赋值给字段：

```java
this.name = "demo";
```

Bean 就成功拿到配置文件中的值了。

#### 配置的优先级

1. 命令行参数
2. Java 系统属性（-Dkey=value）
3. OS 环境变量
4. jar 外部的 application.yml / properties
5. jar 内部（/resources）的 application.yml / properties
6. @PropertySource 指定的配置文件
7. 默认配置（Spring Boot 自动配置设置的默认值）


### 启动参数

Java和 Spring Boot 常用启动参数：

```bash
java -Xms1g -Xmx2g \
  -Denv=prod \
  -Duser.timezone=Asia/Shanghai \
  -jar app.jar \
  --spring.profiles.active=prod \
  --server.port=9000 \
  --logging.level.root=INFO
```

## Serializable
在 Java 中，实现 `Serializable` 接口的类可以将其实例转换为字节流，从而实现对象的持久化存储或通过网络传输。

### serialVersionUID 的作用
`serialVersionUID` 是 Java 序列化机制中的一个重要概念。它是一个唯一的标识符，用于验证序列化和反序列化过程中类的版本一致性。

- 当一个对象序列化后写入文件或缓存，再次反序列化时，JVM 会检查类的 serialVersionUID 是否一致。
- 如果类结构改变（新增/删除字段等），但 serialVersionUID 没变，可能导致 序列化不兼容问题。
- 如果没有显式定义，JVM 会根据类的结构生成一个默认的 serialVersionUID，但类结构一改，生成的值会变，导致反序列化失败。

## tips
1. 构造器注入优于@Autowired注入，推荐使用构造器注入。
2. 少用 @Component 扫一切
   1. 推荐使用 @Service、@Repository、@Controller 等更具体的注解，明确类的角色和职责。
