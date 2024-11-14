---
title: Java学习笔记-配置文件
date: 2024-11-13 09:50:41
categories: [Java]
---

基于SpringBoot的Java学习笔记-配置文件

参考：[配置文件](https://www.didispace.com/spring-boot-2/2-1-config.html)

## 配置文件

Spring Boot的默认配置文件在：`src/main/resources/application.properties`

关于Spring Boot应用的配置内容都可以集中在该文件中了，根据我们引入的不同Starter模块，可以在这里定义诸如：容器端口名、数据库链接信息、日志级别等各种配置信息。比如，我们需要自定义web模块的服务端口号，可以在`application.properties`中添加`server.port=8888`来指定服务端口为8888，也可以通过`spring.application.name=hello`来指定应用名（该名字在Spring Cloud应用中会被注册为服务名）。

也可以使用yaml文件来配置。

如：

```yaml
server:
  port: 8888
  context-path: /hello
  shutdown:
    graceful: true
    port: 8889
    timeout: 30
    enabled: true
    path: /shutdown
```

对应的application.properties文件：
```properties
server.port=8888
server.context-path=/hello
server.shutdown.graceful=true
server.shutdown.port=8889
server.shutdown.timeout=30
server.shutdown.enabled=true
server.shutdown.path=/shutdown
```

### 自定义参数
在配置文件中可以定义一些自定义参数，如
```properties
book.name=Spring Boot 2.x
```
在Java代码中可以通过`@Value`注解来获取配置文件中的参数
```java
@Value("${book.name}")
private String bookName;
```

还可以使用随机数
```properties
my.secret=${random.value}
my.number=${random.int}
```
### 命令行参数
在启动应用时，可以通过命令行参数来覆盖配置文件中的参数
```shell
java -jar myapp.jar --server.port=8888
```

`--`后面的参数会覆盖配置文件中的参数。

### 多环境配置

在Spring Boot中多环境配置文件名需要满足`application-{profile}.properties`的格式，其中`{profile}`对应你的环境标识，比如：

- `application-dev.properties`：开发环境
- `application-test.properties`：测试环境
- `application-prod.properties`：生产环境
- `application.properties`：默认环境

在`application.properties`中通过`spring.profiles.active`来指定当前环境，如：
```properties
spring.profiles.active=dev
```

### 配置文件加载顺序

Spring Boot会按照如下顺序加载配置文件：

- 命令行中传入的参数。
- SPRING_APPLICATION_JSON中的属性。SPRING_APPLICATION_JSON是以JSON格式配置在系统环境变量中的内容。
- java:comp/env中的JNDI属性。
- Java的系统属性，可以通过System.getProperties()获得的内容。
- 操作系统的环境变量
- 通过random.*配置的随机属性
- 位于当前应用jar包之外，针对不同{profile}环境的配置文件内容，例如：application-{profile}.properties或是YAML定义的配置文件
- 位于当前应用jar包之内，针对不同{profile}环境的配置文件内容，例如：application-{profile}.properties或是YAML定义的配置文件
- 位于当前应用jar包之外的application.properties和YAML配置内容
- 位于当前应用jar包之内的application.properties和YAML配置内容
- 在@Configuration注解修改的类中，通过@PropertySource注解定义的属性
- 应用默认属性，使用SpringApplication.setDefaultProperties定义的内容

## 加密敏感信息

参考：[Spring Boot 2.x敏感信息加密](https://www.didispace.com/spring-boot-2/2-5-jasypt.html#%E4%B8%BA%E4%BB%80%E4%B9%88%E8%A6%81%E5%8A%A0%E5%AF%86)

