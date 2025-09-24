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

### 代码规范

1、抛出异常的地方都需要记录日志
2、日志的格式：时间+类名+方法名+异常信息：
```java
log.error("时间：{}，类名：{}，方法名：{}，异常信息：{}", LocalDateTime.now(), this.getClass().getName(), "方法名", e.getMessage());
```
3、如果需要有空返回，先返回空，而不是先走业务逻辑