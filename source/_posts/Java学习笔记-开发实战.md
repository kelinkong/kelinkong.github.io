---
title: Java学习笔记-开发实战一
date: 2024-09-05 11:47:07
categories: [Java]
---

## 项目背景

开发一个通用项目管理项目，前端使用react，后端使用Spring Boot

## 开发中遇到的知识点

### Spring Boot的开发框架

```

project-root/
│
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/project/
│   │   │       ├── controller/ # 存放控制器类,处理HTTP请求
│   │   │       ├── service/ # 存放业务逻辑类
│   │   │       ├── repository/ #  存放数据访问层的接口和实现
│   │   │       ├── model/ # 存放实体类和数据传输对象(DTO)
│   │   │       ├── └── dto/ 
│   │   │       └── ProjectApplication.java # Spring Boot的主应用类,包含main()方法
│   │   │
│   │   └── resources/
│   │       ├── static/ # 存放静态资源如CSS、JavaScript、图片等。
│   │       ├── templates/ # 存放模板文件(如Thymeleaf模板)。
│   │       ├── db/ # 存放sql文件，scheme.sql  data.sql
│   │       └── application.properties # 主配置文件,用于设置应用程序属性。
│   │
│   └── test/  # 存放测试代码。
│       └── java/
│           └── com/example/project/
│
├── target/ # Maven构建生成的目录,包含编译后的类文件和可执行JAR
├── pom.xml # Maven项目配置文件,定义项目依赖和构建过程。
└── README.md

```

### Lombok

[Spring Boot 整合 Lombok，用注解简化 Java 代码，比如说 getter和setter | 二哥的Java进阶之路 (javabetter.cn)](https://javabetter.cn/springboot/lombok.html)

Lombok可以更方便的生成set、get方法，在Maven管理的Java项目中，需要添加：

```
<dependency>
	<groupId>org.projectlombok</groupId>
	<artifactId>lombok</artifactId>
	<version>1.18.6</version>   # 在jdk21以上的版本，需要设置版本为1.18.30以上
	<scope>provided</scope>
</dependency>
```

其中` scope=provided`，就说明 Lombok 只在编译阶段生效。也就是说，Lombok 会在编译期静悄悄地将带 Lombok 注解的源码文件正确编译为完整的 class 文件。

SpringBoot 2.1.x 版本后不需要再显式地添加 Lombok 依赖了。之后，还需要为 Intellij IDEA 安装 Lombok 插件，否则 Javabean 的 getter / setter 就无法自动编译，也就不能被调用。不过，新版的 Intellij IDEA 也已经内置好了，不需要再安装。

#### 常用的Lombok注解

```java
class CmowerLombok {
	@Getter @Setter private int age;
	@Getter private String name;
	@Setter private BigDecimal money;
}

@ToString
class CmowerLombok {
	private int age;
	private String name;
	private BigDecimal money;
}

@Data. // @Data 注解可以生成 getter / setter、equals、hashCode，以及 toString，是个总和的选项。
class CmowerLombok {
	private int age;
	private String name;
	private BigDecimal money;
}
```

### JPA

[Spring Boot 整合 JPA | 二哥的Java进阶之路 (javabetter.cn)](https://javabetter.cn/springboot/jpa.html)

JPA（Java Persistence API）是一种Java对象持久化技术，它提供了一种将Java对象映射到关系数据库表的机制。当我们定义一个实体类时，JPA会根据类上的注解（如@Entity、@Table、@Column等）来生成对应的数据库表结构。

可以使用JPA创建：

- Entity（User）
- Repository（UserRepository）

JPA可以通过实体类来生成数据库表（spring.jpa.hibernate.ddl-auto），常见的配置有：（在生成环境中，不建议使用create、create、drop、update）

- create: 每次应用程序启动时，都会删除现有的数据库表，并根据实体类重新创建。
- create-drop: 与create类似，但是在应用程序关闭时会删除所有表。
- update: 每次应用程序启动时，会根据实体类的定义来更新数据库表结构。
- validate: 仅验证数据库表结构与实体类是否匹配，不会做任何修改。
- none：不做任何操作。

在`application.properties`中设置：`spring.jpa.hibernate.ddl-auto=update`

**更新数据库表时，不会自动更新实体类**

JPA生成Repository内置了一些方法，但是也可以通过注解的方式来实现

```java
import java.util.List;

public interface TeamRepository extends JpaRepository<Team, Integer>, JpaSpecificationExecutor<Team> {

    // Find all teams
    @Query("SELECT t FROM Team t")
    List<Team> findAllTeams();

    // Find a team by its ID
    @Query("SELECT t FROM Team t WHERE t.id = :id")
    Team findTeamById(@Param("id") Integer id);

    // Custom SQL query to find teams by name
    @Query(value = "SELECT * FROM teams WHERE name = :name", nativeQuery = true)
    List<Team> findTeamsByName(@Param("name") String name);
}
```

@Query注解的nativeQuery属性作用：

- nativeQuery=false：使用HQL，JPA会根据实体类和关联关系自动生成SQL。
- nativeQuery=true：使用原生SQL，JPA直接执行你写的SQL语句

### RequestMapping

在Spring Boot中，`@RequestMapping` 是一个非常基础且强大的注解，用于将HTTP请求映射到特定的处理方法上。它可以配置请求路径、HTTP方法（GET、POST、PUT、DELETE等）、参数等信息。

为了简化开发，Spring Boot提供了几个更具体的注解，它们都是`@RequestMapping`的缩写形式：

* **@GetMapping:** 通常用于获取数据，比如查询列表、获取详情等。
* **@PostMapping:** 通常用于创建新的资源，比如添加用户、提交表单等。
* **@PutMapping:** 通常用于更新整个资源，比如修改用户信息。
* **@DeleteMapping:** 通常用于删除资源，比如删除用户。
* **@PatchMapping:** 通常用于部分更新资源，比如修改用户的部分信息。

```java
@RestController
@RequestMapping("/api/teams")
public class TeamController {

private final TeamService teamService;

public TeamController(TeamService teamService) {
    this.teamService = teamService;
}

@DeleteMapping("/{teamId}/members/{userId}")
public ResponseEntity<Void> deleteMember(@PathVariable Integer teamId, @PathVariable Integer userId) {
    teamService.deleteMember(teamId, userId);
    return ResponseEntity.noContent().build();
}
}
```

`@RequestMapping`注解可以接受多个参数，如`value`、`method`、`params`、`headers`等，用于指定请求路径、HTTP方法、请求参数、请求头等信息。例如：

```java
@RequestMapping(value = "/api/teams", method = RequestMethod.GET)
public List<Team> getTeams() {
    return teamService.getTeams();
}
```

### 用到的注解

#### 注解是什么？

注解（Annotation）是一种提供元数据（metadata）的机制。它可以用于标记类、方法、字段等程序元素，从而为编译器或运行时环境提供额外的信息。注解本身不会影响程序的执行逻辑，但可以被编译器、IDE、框架等工具读取并进行相应的处理。

#### 注解的实现原理

注解本质上是一个接口。当我们定义一个注解时，实际上是在定义一个接口，并且这个接口继承自`java.lang.annotation.Annotation`接口。

* **注解处理器：**
  * 注解处理器是实现注解功能的关键。它们在编译时或运行时读取注解信息，并根据注解的定义执行相应的操作。
  * 常用的注解处理器有：
    * **编译时注解处理器：** 在编译时处理注解，如APT（Annotation Processing Tool）。
    * **运行时注解处理器：** 在运行时处理注解，如反射机制。
* **反射机制：**
  * 通过反射机制，可以在运行时获取类、方法、字段的注解信息，并动态地调用这些元素。

#### 注解的作用

注解在Java开发中发挥着重要的作用，主要有以下几个方面：

* **提供元数据：** 为编译器、IDE、框架等工具提供额外的信息，如：
  * **生成代码：** 比如Lombok注解可以自动生成getter、setter、构造方法等。
  * **配置信息：** 比如Spring框架中的`@Autowired`注解用于自动装配Bean。
  * **验证：** 比如Hibernate Validator中的`@NotNull`注解用于验证字段不能为空。
* **减少重复代码：** 通过注解可以减少重复的代码编写，提高开发效率。
* **提高代码可读性：** 注解可以明确地表达代码的意图，提高代码的可维护性。
* **实现AOP：** 注解可以作为AOP切入点，实现横切关注点。

#### 注解的分类

* **内置注解：** Java内置了一些注解，如`@Override`、`@Deprecated`等。
* **元注解：** 用于定义注解的注解，如`@Target`、`@Retention`、`@Documented`等。
* **自定义注解：** 开发者可以自定义注解，以满足特定的需求

#### 用到的注解

- 声明 Bean 的注解
  @Component：通用注解，可以标注任何类为 Spring 组件。
  @Repository：专用于数据访问层（DAO）的组件。
  @Service：专用于业务逻辑层（Service）的组件。
  @Controller：专用于表现层（Controller）的组件。
- 注入 Bean 的注解@Autowired：根据类型自动装配 Bean，可以用于字段、方法参数和构造函数。
  @Qualifier：配合 @Autowired 使用，根据 Bean 的名称进行注入。
  @Resource：来自 JSR-250，也可以用于注入 Bean，功能类似于 @Autowired。
- 配置类相关的注解
  @Configuration：声明一个配置类，等同于 XML 配置文件。
  @ComponentScan：指定 Spring 在创建容器时要扫描的包。
  @Bean：把方法的返回值作为 Bean 对象存入 Spring 容器。
- 切面（AOP）相关的注解
  @Aspect：声明一个切面。
  @Before、@After、@Around：定义切面中的增强方法。
  @Pointcut：声明切点，用于定义拦截规则。
- 其他常用注解
  @Value：注入配置文件中的值。
  @RequestMapping：用于处理 Web 请求。
  @ResponseBody：将方法的返回值直接写入 HTTP 响应体。
  @RequestBody：将 HTTP 请求中的 JSON 数据绑定到 Java 对象。
  @PathVariable：用于获取路径变量的值。
  @Transactional：声明事务。

## 工具/技巧使用

Intellj idea快速创建某个接口的实现类：`alt+enter`

### 接口类和实现类的调用

Java中的接口类类似C++中的抽象类。

我的tokenServiceImpl是tokenService接口的实现，我在别的class中调用这个服务时，我应该使用tokenService还是tokenServiceImpl？

> 应该使用tokenService。
>
> * **依赖注入（Dependency Injection）：** 当 Spring 容器管理你的 Bean 时，它会将 `TokenServiceImpl` 的实例注入到需要使用 token 服务的类中，但是注入的变量名通常是接口名 `tokenService`，而不是实现类名 `tokenServiceImpl`。
> * **面向接口编程：** 面向接口编程是一种良好的编程习惯。通过使用接口 `tokenService`，你的代码与具体的实现类解耦，提高了代码的可维护性、可测试性和扩展性。
> * **多态：** 由于 Java 的多态特性，当你使用 `tokenService` 时，实际上调用的是 `TokenServiceImpl` 中实现的方法。这样，如果你以后想更换 `TokenServiceImpl` 的实现，只需要修改 Spring 配置，而不需要修改调用代码。

**Service的注解应该放到实现类上，而不是接口类。**

### 单元测试

为什么要使用JUnit4、mock和verify？

* **JUnit4** 是Java中最流行的单元测试框架，提供了丰富的断言和测试注解。
* **mock** （模拟对象）可以让我们在测试中替换掉真实对象，从而隔离被测代码，方便测试。
* **verify** 用于验证模拟对象的方法调用情况，确保代码的正确性。

示例，测试一个简单的`UserService`

```java
public interface UserService {
    User getUserById(int id);
}

public class UserServiceImpl implements UserService {
    private UserDao userDao; // 依赖注入,在单元测试中，使用mock模拟

    public UserServiceImpl(UserDao userDao) {
        this.userDao = userDao;
    }

    @Override
    public User getUserById(int id) {
        return userDao.getUserById(id);
    }
}
```
```java
import org.junit.Test;
import org.mockito.Mockito;
import static org.mockito.Mockito.*;

public class UserServiceImplTest {

    @Test
    public void testGetUserById() {
        // 创建UserDao的模拟对象
        UserDao userDao = mock(UserDao.class);
        // 创建期望的User对象
        User expectedUser = new User(1, "张三"); 
        // 设置模拟对象的行为，当userDao调用getUserById(1)时，返回期望的User对象
        when(userDao.getUserById(1)).thenReturn(expectedUser);

        // 创建UserService实例，这是我们要测试的对象
        UserService userService = new UserServiceImpl(userDao);

        // 调用UserService的方法，这是我们要测试的方法
        User actualUser = userService.getUserById(1);

        // 验证UserDao的getUserById方法被调用了一次
        verify(userDao, times(1)).getUserById(1);

        // 断言实际的User对象和期望的User对象相等
        assertEquals(expectedUser, actualUser);
    }
}
```

## 一些疑问

### 在@RequestMapping注解中添加了url参数后，为什么前端就可以通过url传递参数？

@RequestMapping 注解在 Spring MVC 中起着至关重要的作用，它建立了 HTTP 请求与控制器方法之间的映射关系。当前端发送一个 HTTP 请求时，Spring MVC 框架会根据请求的 URL，去寻找匹配的 @RequestMapping 注解，然后调用对应的控制器方法来处理这个请求。

整个过程可以简化为以下几步：

1. **前端发送请求:** 用户在浏览器中输入 URL，浏览器将这个请求发送到服务器。
2. **Spring MVC 拦截请求:** Spring MVC 作为 Web 框架，会拦截所有的 HTTP 请求。
3. **匹配 @RequestMapping:** Spring MVC 会根据请求的 URL，去容器中查找所有被`@RequestMapping` 注解标注的方法，并尝试匹配。
4. **执行控制器方法:** 如果找到了匹配的方法，Spring MVC 就会调用这个方法，并把请求参数传递给方法。
5. **返回响应:** 控制器方法执行完成后，会返回一个 ModelAndView 对象，Spring MVC 会将这个对象转换为 HTTP 响应，返回给前端。

### Spring MVC 与 Socket 的关系

- Spring MVC 的工作原理： Spring MVC 作为一款基于 Servlet 的 Web 框架，其核心是处理 HTTP 请求。当一个 HTTP 请求到达服务器时，Servlet 容器会将请求封装成 HttpServletRequest 和 HttpServletResponse 对象，然后交给 DispatcherServlet 处理。
  - Servlet 和 Socket： Servlet 本质上是运行在 Servlet 容器中的一个 Java 类，它提供了处理 HTTP 请求和响应的接口。Servlet 容器（如 Tomcat、Jetty）则是基于 Socket 实现的，负责监听网络端口，接收客户端的 HTTP 请求，并将其传递给 Servlet。
- Spring MVC 与 Socket 的关系：
  - 间接依赖： Spring MVC 依赖于 Servlet 容器，而 Servlet 容器直接基于 Socket 工作。因此，Spring MVC 可以说是间接地利用了 Socket 的功能。
  - 抽象层级： Spring MVC 提供了一层更高级的抽象，将开发者从底层的 Socket编程细节中解放出来。开发者只需要关注业务逻辑，而不需要关心如何处理网络连接、协议解析等。
- Socket 的作用：
  - 建立连接： Socket 用于在客户端和服务器之间建立网络连接。
  - 数据传输： 通过 Socket 进行数据传输，实现客户端和服务器之间的通信。
  - 协议解析： Socket 负责解析 HTTP 协议，将请求和响应数据进行编码和解码。

### 当我在使用Spring boot开发时，我还需要手动建立数据库连接池吗？

**一般情况下，在 Spring Boot 中，我们不需要手动创建数据库连接池。** Spring Boot 默认集成了 HikariCP 这个高性能的数据库连接池，并提供了自动配置。只需在配置文件（如 application.properties 或 application.yml）中配置数据库连接信息，Spring Boot 就会自动创建并管理连接池。

```java
spring.datasource.url=jdbc:mysql://localhost:3306/mydatabase
spring.datasource.username=root
spring.datasource.password=yourpassword
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver // 这个配置项在 Spring Boot 中用于指定连接数据库的 JDBC 驱动类的全限定名。
```

### Spring Boot 如何处理多个用户请求?

通常情况下，不需要手动创建线程池。 Spring Boot 已经为您内置了许多自动配置，其中就包括线程池的创建和管理。

Spring Boot 默认的线程池

- Tomcat线程池: 对于传统的 Servlet 容器 Tomcat，Spring Boot 会默认使用 Tomcat 的线程池来处理 HTTP 请求。
- Undertow线程池: 如果您使用的是 Undertow 作为嵌入式 Servlet 容器，那么 Spring Boot 会使用 Undertow 的线程池。

这些线程池通常已经经过优化，可以满足大多数应用的并发处理需求。
