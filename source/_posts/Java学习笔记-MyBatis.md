---
title: Java学习笔记-MyBatis
date: 2024-10-22 14:46:45
categories: [Java]
---
## 一个简单的示例
MyBatis是一个流行的持久层框架，它支持自定义SQL、存储过程和高级映射，消除了几乎所有的JDBC代码和参数的手动设置以及结果集的检索。

1. 首先添加依赖(pom.xml)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.2.0</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
    </dependencies>
</project>
```

2. 配置数据库连接(application.yml)
```
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/demo?useSSL=false
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver

mybatis:
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: com.example.entity
```
3. 创建实体类
```java
package com.example.entity;

@Data
public class User {
    private Long id;
    private String name;
    private Integer age;
}
```
 4. 创建Mapper接口
```java
package com.example.mapper;

import com.example.entity.User;
import org.apache.ibatis.annotations.Mapper;

@Mapper
public interface UserMapper {
    User findById(Long id);
    void insert(User user);
}
```

 5. 创建XML映射文件(`resources/mapper/UserMapper.xml`)
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mapper.UserMapper">
    <select id="findById" resultType="User">
        SELECT * FROM user WHERE id = #{id}
    </select>
    
    <insert id="insert" parameterType="User">
        INSERT INTO user (name, age) VALUES (#{name}, #{age})
    </insert>
</mapper>
```
 6. 创建Service层
```java
package com.example.service;

import com.example.entity.User;
import com.example.mapper.UserMapper;
import org.springframework.stereotype.Service;

@Service
public class UserService {
    private final UserMapper userMapper;
    
    public UserService(UserMapper userMapper) {
        this.userMapper = userMapper;
    }
    
    public User getUser(Long id) {
        return userMapper.findById(id);
    }
    
    public void createUser(User user) {
        userMapper.insert(user);
    }
}
```
 7. 创建Controller
```java
package com.example.controller;

import com.example.entity.User;
import com.example.service.UserService;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/users")
public class UserController {
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.getUser(id);
    }
    
    @PostMapping
    public void createUser(@RequestBody User user) {
        userService.createUser(user);
    }
}

```

这个示例展示了MyBatis在Spring Boot中的基本使用，包括：

1. Maven依赖配置
2. 数据库连接配置
3. 实体类定义
4. Mapper接口定义
5. XML映射文件
6. Service层实现
7. Controller层实现

使用这个示例，可以：
- 通过 GET /users/{id} 查询用户
- 通过 POST /users 创建新用户

在使用前需要确保：
1. 创建对应的数据库和表
2. 修改数据库连接配置
3. 启动类添加 `@MapperScan("com.example.mapper")` 注解
