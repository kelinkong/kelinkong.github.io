---
title: Spring Cloud: 服务注册与发现
date: 2024-11-14 10:51:57
categories: [Java]
---

## 微服务之间的通信

### RestTemplate

RestTemplate是Spring提供的用于访问Rest服务的客户端模板工具集。

添加配置类`RestTemplateConfig`：

```java
@Configuration
public class RestTemplateConfig {

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```
为什么要用配置类？而不是每次使用`RestTemplate`时都创建一个新的实例？
> 因为`RestTemplate`的实例化是一个比较耗时的操作，如果每次使用都创建一个新的实例，会影响性能。
> 
> RestTemplate 是设计为线程安全的，能够在多个线程之间共享。通过定义一个单例的 RestTemplate，可以确保多个线程安全地访问同一个实例，而不需要担心并发问题。
>
> 使用配置类可以集中管理 RestTemplate 的配置，比如超时设置、拦截器、消息转换器等。这使得应用程序的配置更加一致和易于维护。


1. 创建一个`RestTemplate`实例，并注入到Spring容器中。
2. 使用`RestTemplate`发送HTTP请求，并接收响应。

```java
@Component
public class RestTemplateClient {
    @Autowired
    private RestTemplate restTemplate;
    public String getUserName() {
        return restTemplate.getForObject("http://localhost:8080/user/name", String.class);
    }
}
```

`RestTemplate` 是 Spring 框架中的一个类，用于在客户端上发送 HTTP 请求并处理响应。它支持多种 HTTP 请求方法，主要包括以下几种：

#### 1. `GET` 请求

用于从服务器获取资源。`RestTemplate` 提供了 `getForObject` 和 `getForEntity` 方法：

- **`getForObject`**：返回指定类型的对象（自动转换）。
  
  ```java
  String result = restTemplate.getForObject("http://example.com/resource", String.class);
  ```

- **`getForEntity`**：返回 `ResponseEntity` 对象，其中包含 HTTP 状态码、头部信息和实体内容。
  
  ```java
  ResponseEntity<String> response = restTemplate.getForEntity("http://example.com/resource", String.class);
  ```

#### 2. `POST` 请求

用于向服务器提交数据，`RestTemplate` 提供了 `postForObject` 和 `postForEntity` 方法：

- **`postForObject`**：提交数据后返回响应的实体内容（转换为指定类型）。
  
  ```java
  String response = restTemplate.postForObject("http://example.com/resource", requestBody, String.class);
  ```

- **`postForEntity`**：提交数据后返回 `ResponseEntity`，包含状态码、头部信息和实体内容。
  
  ```java
  ResponseEntity<String> response = restTemplate.postForEntity("http://example.com/resource", requestBody, String.class);
  ```

#### 3. `PUT` 请求

用于更新服务器上的资源，`RestTemplate` 提供了 `put` 方法：

```java
restTemplate.put("http://example.com/resource/{id}", updatedResource, id);
```

`PUT` 请求通常不返回响应内容，因此 `put` 方法的返回类型为 `void`。

#### 4. `DELETE` 请求

用于删除服务器上的资源，`RestTemplate` 提供了 `delete` 方法：

```java
restTemplate.delete("http://example.com/resource/{id}", id);
```

同样，`DELETE` 请求通常不返回响应内容，因此 `delete` 方法的返回类型为 `void`。



## 提取通用模块

将所有使用的通用模块提取到一个单独的模块中，方便其他模块引用。

1. 创建一个新的模块，命名为`common`。
2. 将通用模块的代码复制到`common`模块中。
3. 在其他模块中添加`common`模块的依赖。

```xml
<dependency>
    <groupId>com.example</groupId>
    <artifactId>common</artifactId>
    <version>1.0.0</version>
</dependency>
```
## consul介绍
Consul 是 HashiCorp 公司提供的一款开源的服务发现和配置管理工具。它提供了一种简单的方式来注册和发现服务，并提供了服务健康检查功能。

### 下载
[consul下载链接](https://developer.hashicorp.com/consul/install?product_intent=consul)
```shell
brew tap hashicorp/tap
brew install hashicorp/tap/consul
```

### 安装和使用
启动consul：
```shell
consul agent -dev
```
访问：
[http://localhost:8500](http://localhost:8500)

配置文件地址：[Spring- cloud-consul](https://cloud.spring.io/spring-cloud-consul/reference/html/)

### 服务注册与发现

1. 添加依赖
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-consul-discovery</artifactId>
</dependency>
```
2. 配置文件
```yaml
spring:
  application:
    name: my-service
  cloud:
    consul:
      host: localhost         # Consul 的地址
      port: 8500              # Consul 的端口号
      discovery:
        service-name: my-service   # 服务名，可随项目需求更改
        health-check-interval: 10s # 健康检查时间间隔
```
3. 在主启动类上添加 @EnableDiscoveryClient 注解，Spring Boot 应用启动时会自动将服务注册到 Consul。
```java
@SpringBootApplication
@EnableDiscoveryClient
public class MyServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyServiceApplication.class, args);
    }
}
```
4. 通过 @LoadBalanced 注解和 RestTemplate 或 Feign 客户端来进行服务调用。
```java
@Bean
@LoadBalanced
public RestTemplate restTemplate() {
    return new RestTemplate();
}
```

## 服务消费者
服务消费者需要依赖服务提供者的服务名，而不是具体的IP和端口。
```java
private final String PAYMENT_URL = "http://cloud-provider-payment";

    @Resource // 注入RestTemplate
    private RestTemplate restTemplate;

    @PostMapping("/pay/add")
    public ResultData addOrder(PayDTO payDTO){
        return restTemplate.postForObject(PAYMENT_URL + "/pay/add", payDTO, ResultData.class);
    }
```

## consul 配置
使用 Consul 作为配置管理工具，可以将应用的配置信息集中存储在 Consul 中，应用从 Consul 读取配置，而不是依赖本地文件。Spring Cloud 提供了 `spring-cloud-starter-consul-config` 模块，方便与 Consul 集成，以下是如何配置和使用 Consul 来进行配置管理的步骤：

### 1. 引入依赖

在 `pom.xml` 中添加 Consul 配置依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-consul-config</artifactId>
</dependency>
```

### 2. 配置 Consul 连接

在 `application.yml` 或 `application.properties` 中配置 Consul 的连接地址和一些关键设置：

这里也可以在`bootstrap.yml`中配置：`bootstrap.yaml`的优先级高于`application.yaml`。

```yaml
spring:
  cloud:
    consul:
      host: localhost         # Consul 的主机地址
      port: 8500              # Consul 的端口
      config:
        enabled: true         # 启用 Consul 配置管理
        default-context: application  # 默认上下文（通用配置）
        profile-separator: '-'        # 配置文件和环境的分隔符
        format: yaml                  # 配置格式 (yaml 或 properties)
        fail-fast: true               # 启动时加载配置失败是否立即失败
```

### 3. 在 Consul 中存储配置

Consul 的配置管理使用 Key-Value 存储系统。将配置文件内容以键值对形式存储在 Consul 中，格式如下：

#### Key 结构

Consul 中的键通常按以下路径结构组织：

- `config/{application-name}/{profile}/data`

例如，应用名为 `my-app`，环境为 `dev` 的配置路径为：

```
config/my-app/dev/data
```

#### 存储配置

可以通过 Consul 的 Web UI 或命令行将配置写入 Consul。在 Web UI 中可以添加键值对，也可以使用 `curl` 命令来添加配置。例如：

```bash
curl -X PUT --data-binary @application.yml http://localhost:8500/v1/kv/config/my-app/data
```

将配置数据 `application.yml` 上传到 Consul 的 `config/my-app/data` 路径。

### 4. Consul 配置的层次结构

Consul 的配置会覆盖，层次结构为：

- `config/application/`：通用配置（所有应用和环境共享）。
- `config/{application-name}/`：特定应用的配置。
- `config/{application-name}/{profile}/`：特定应用的特定环境配置（如 `dev`、`prod`）。

### 5. 读取配置

在 Spring Boot 应用中，不需要额外的代码来读取 Consul 配置。Spring Cloud Consul Config 会自动将 Consul 中的配置加载到 Spring 的 `Environment` 中，和 `application.yml` 或 `application.properties` 中的配置一样使用。

```yaml
# 从 Consul 加载的配置会自动生效
my:
  config:
    value: "Hello from Consul"
```

在应用中，可以像平常一样使用 `@Value` 注解或 `@ConfigurationProperties` 注解获取配置值：在data中写：

```yaml
atguigu:
    info: "Hello from Consul"
``` 
获取：

```java
@RestController
public class MyController {

    @Value("${atguigu.info}")
    private String configValue;

    @GetMapping("/config")
    public String getConfigValue() {
        return configValue;
    }
}
```

### 6. 配置动态刷新

Spring Cloud Consul 支持配置的动态刷新。如果 Consul 中的配置发生变化，可以自动更新应用中的配置值。启用动态刷新需要在 `@RefreshScope` 注解的支持下：

1. 在需要动态刷新的 Bean 类上添加 `@RefreshScope` 注解。
2. 默认配置会每 60 秒刷新一次，可以在 `application.yml` 中通过 `spring.cloud.consul.config.watch-delay` 配置刷新频率（单位为毫秒）。

```yaml
spring:
  cloud:
    consul:
      config:
        watch-delay: 1000    # 每秒检查一次配置更新
```

在需要动态刷新的配置类上使用 `@RefreshScope`：

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RefreshScope
@RestController
public class MyController {

    @Value("${my.config.value}")
    private String configValue;

    @GetMapping("/config")
    public String getConfigValue() {
        return configValue;
    }
}
```

但是这个配置不是持久化的，当服务重启后，配置不会保留。