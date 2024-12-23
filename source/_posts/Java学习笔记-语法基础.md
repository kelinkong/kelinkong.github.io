---
title: Java学习笔记-语法基础篇
date: 2024-08-28 14:13:28
categories: [Java]
---

### 扫盲，Java和C++的一些区别

**Java项目需要编译吗？底层编译原理和C++有什么不同？**

1.	编译输出：

- Java：编译输出的是平台无关的字节码文件（.class 文件），这些字节码可以在任何安装了 JVM 的平台上运行。
- C++：编译输出的是特定平台的机器码文件（如 .exe 文件），这些文件只能在编译时指定的平台上运行。

2.	跨平台性：
- Java：通过 JVM 的跨平台特性，Java 程序可以“一次编写，到处运行”（Write Once, Run Anywhere）。
- C++：需要为每个平台分别编译代码才能生成可以执行的文件，不具备 Java 那样的跨平台能力。

3.	运行时性能：

- Java：由于字节码需要在运行时通过 JVM 翻译为机器码，可能会在启动时稍慢，但 JIT 编译可以在运行期间优化代码，提高性能。
- C++：由于是直接编译为机器码，C++ 程序的启动和运行速度通常会比 Java 程序快，尤其是在性能关键的应用中。

4.	编译速度：

- Java 编译通常更快，因为编译器只需将代码编译为字节码，而不需要生成机器码。
- C++ 编译通常更慢，尤其是在大型项目中，因为编译器需要生成并优化特定平台的机器码。

**Java编译是否需要再单独安装编译器？**

在编写 Java 程序时，不需要单独安装编译器，因为 Java 编译器是包含在 JDK（Java Development Kit）中

**JDK包含了什么？**

JDK 是 Java 开发的核心工具包，它包含了一整套开发 Java 应用程序所需的工具和库。JDK 的主要组成部分包括：

1.	Java 编译器 (javac)：
- 这是用于将 Java 源代码（.java 文件）编译成字节码（.class 文件）的工具。
- javac 是 JDK 中最重要的组件之一，负责将人类可读的 Java 代码转换为 JVM 可执行的字节码。

2.	Java 运行时环境 (JRE, Java Runtime Environment)：

- JRE 是运行 Java 程序所需的环境，包括 JVM、Java 类库和其他资源。
- JRE 包含了 Java 虚拟机（JVM）、核心类库和支持文件，但不包括编译器和调试工具。
- JDK 本身包含了一个完整的 JRE，所以在安装 JDK 时也会获得 JRE。

3.	Java 虚拟机 (JVM)：
- JVM 是一个平台独立的虚拟机，负责解释和执行编译后的字节码。
- JVM 使得 Java 程序可以在任何支持 JVM 的操作系统上运行。
4.	核心类库：
- 这是 Java 标准库（API），包含了大量的预定义类和接口，用于执行各种常见的编程任务（如数据结构、网络通信、文件操作、并发处理等）。
- 核心类库是 Java 程序构建的基础，几乎所有 Java 程序都会使用其中的类和方法。
5.	开发工具：
- java：用于启动 Java 应用程序的命令行工具，它调用 JVM 来执行字节码。
- javadoc：用于生成 Java 代码文档的工具，可以从源码中的注释生成 API 文档。
- jdb：Java 调试工具，允许开发者在运行时调试 Java 应用程序。
- jar：用于创建和管理 Java Archive（JAR）文件的工具，这些文件通常用于打包 Java 类库和应用程序。
- javap：Java 类文件反汇编工具，用于查看编译后的字节码。
6.	其他工具：
- javah：用于生成 C 头文件和源文件，支持 JNI（Java Native Interface）。
- jarsigner：用于对 JAR 文件进行签名和验证的工具。

![](../imgs/image-71.png)
![](../imgs/image-70.png)

Java 编译器不会将所有类都编译成一个机器代码程序。相反，它会独立编译每个类，而且不是编译成机器代码，而是编译成特殊的中间代码（字节码）。当程序启动时，该字节码被编译成机器代码。

有一个特殊的程序叫做 Java 虚拟机 (JVM)。当你需要运行一个字节码程序时，该虚拟机必须先启动。在程序执行之前，JVM 会将字节码编译成机器代码

### Java基础

在Java中，每个源文件都是一个类，类名必须与文件名相同。Java程序的入口是`main`方法，格式如下：

```java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello, Java!");
    }
    // Java入口程序规定的方法必须是静态方法，方法名必须为main，括号内的参数必须是String数组。
    // 其他方法
}
```
一个源码文件只能包含一个public类型的类。

#### 数据类型

Java的数据类型分为两大类：基本数据类型和引用数据类型。

1.	基本数据类型：byte、short、int、long、float、double、char、boolean。**基本类型的变量直接存储值。**
2.	引用数据类型：类、接口、数组、String等。**引用类型变量存储的是对象的内存地址（引用）**。

引用类型变量通常通过 `new` 关键字创建对象（但也有例外，比如字符串常量池），而基本类型变量则直接赋值。

#### 字符串

Java中的字符串是不可变的，即一旦创建，就不能再修改。字符串的比较要使用`equals()`方法，而不是`==`运算符。

```java
String s1 = "hello";
String s2 = "hello";
System.out.println(s1 == s2); // true
System.out.println(s1.equals(s2)); // true
```

**字符串拼接**：

- 使用`+`运算符拼接字符串。
- 使用`StringBuilder`类进行字符串拼接，它是可变的字符串，效率更高。

```java
StringBuilder sb = new StringBuilder(1024);
sb.append("Mr ")
  .append("Bob")
  .append("!")
  .insert(0, "Hello, ");
String s = sb.toString();
System.out.println(s); // Hello, Mr Bob!
```

多行字符串可以使用`"""..."""`格式：

```java
String s = """
    SELECT * FROM
    users
    WHERE id = 1
    """;
```

#### 输出和输入

```java
System.out.println("Hello, Java!"); // 输出并换行
System.out.print("Hello, "); // 输出不换行
System.out.printf("Hello, %s", "Java"); // 格式化输出
```

```java
import java.util.Scanner;  // 导入某个类

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in); // 创建Scanner对象
        System.out.print("Input your name: "); // 打印提示
        String name = scanner.nextLine(); // 读取一行输入并获取字符串
        System.out.print("Input your age: "); // 打印提示
        int age = scanner.nextInt(); // 读取一行输入并获取整数
        System.out.printf("Hi, %s, you are %d\n", name, age); // 格式化输出
    }
}
```

### 面向对象编程

在Java的类中，有构造函数，但是没有析构函数。Java的垃圾回收器会自动回收不再使用的对象。

**但是Java同样会有内存泄漏的问题，比如静态变量、集合类等。**

在Java的类中，也有一个this关键字，表示当前实例对象。

```java
public class Person {
    private String name; // 属性

    public Person(String name) { // 构造方法
        this.name = name;
    }

    public String getName() {
        return this.name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

// 使用extends关键字实现继承
class Student extends Person {
    // 不要重复name和age字段/方法,
    // 只需要定义新增score字段/方法:
    private int score;

    public int getScore() { … }
    public void setScore(int score) { … }
}
Person p = new Person("Bob");
```

- **在Java中一个子类只能继承一个父类，子类继承后无法访问父类的私有字段和方法，可以将父类的字段和方法设置为`protected`。**

- 子类不会继承父类的构造方法，但是可以通过`super()`调用父类的构造方法。

```java
public class Student extends Person {
    private int score;

    public Student(String name, int score) {
        super(name); // 调用父类的构造方法
        this.score = score;
    }
}
```

- 和C++一样，在Java中可以使用父类的引用指向子类的实例。
- Java中的方法可以被子类覆写，使用`@Override`注解可以让编译器检查是否正确覆写了父类的方法。
- 在Java中 ，同样存在抽象类，使用`abstract`关键字修饰，抽象类不能被实例化，只能被继承。`abstract class Person {}`

**实例化子类时，会调用父类的构造器，Java是单继承，且所有的类都继承于Object类，那么是这条线上的所有父类、祖父类的构造器都会被调用吗？**

>是的，在 Java 中，当实例化一个子类时，父类、祖父类乃至更高层次的所有父类的构造函数都会被依次调用，直到根类 Object 的构造函数。这个过程是通过构造函数链实现的。

**在Java中，继承是一棵树，构造函数的调用是从下往上的，即先调用子类的构造函数，再调用父类的构造函数，直到根类的构造函数。**

**在C++中，继承是一个图。**

#### 接口
[接口-参考文献](https://liaoxuefeng.com/books/java/oop/basic/interface/index.html)

在抽象类中，抽象方法本质上是定义接口规范：即规定高层类的接口，从而保证所有子类都有相同的接口实现，这样，多态就能发挥出威力。

如果一个抽象类没有字段，所有方法全部都是抽象方法：
```java
abstract class Person {
    public abstract void run();
    public abstract String getName();
}
```
就可以把该抽象类改写为接口：interface。

```java
interface Person {
    void run();
    String getName();
}
```

当一个具体的class去实现一个interface时，需要使用implements关键字。
    
```java
class Student implements Person, Hello {
    public void run() {
        System.out.println("Student.run");
    }
    public String getName() {
        return "Student";
    }
}
```
在Java中，一个类只能继承自另一个类，不能从多个类继承。但是，一个类可以实现多个interface。

#### 包
  在定义class的时候，我们需要在第一行声明这个class属于哪个包。
```java
package example; // 定义包名
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello, world!");
    }
}
```
所有Java文件对应的目录层次要和包的层次一致。

### 和C++作对比：

|关键字|Java|C++|
|:------: |:------:|------:|
|常量|final|const|
|自动类型推导|var|auto|
|空指针|null|nullptr|
|字符串|String|std::string|
|静态方法|static|static|
|继承|extends|: public|
|实现|implements|: public|
|抽象类|abstract class|class|
|接口|interface|class|
|包|package|namespace|
|异常处理|[try-catch-finally](https://liaoxuefeng.com/books/java/exception/java-exception/index.html)|try-catch|
|泛型|List\<String\>|vector\<string\> 模版|

#### Java和C++中的STL容器区别
|容器|Java|C++|
|:------: |:------:|------:|
|动态数组|ArrayList|vector|
|链表|LinkedList|list|
|栈|Stack|stack|
|队列|Queue|queue|
|双端队列|Deque|deque|
|集合|Set|set|
|映射|Map|map|
|哈希表|HashMap|unordered_map|
|哈希集合|HashSet|unordered_set|
|优先队列|PriorityQueue|priority_queue|

### 反射机制

Java的反射机制是指在运行状态中，对于任意一个类，都能知道这个类的所有属性和方法；对于任意一个对象，都能调用它的任意一个方法。

如果要实现一个通用的对象`拷贝`方法，就必须要用反射机制。

```java
import java.lang.reflect.Field;

class Person {
    public String name;
    public int age;
}

class Employee {
    public String name;
    public int age;
    public double salary;
}

public class Main {
    public static void main(String[] args) throws Exception {
        Person person = new Person();
        person.name = "John";
        person.age = 30;

        Employee employee = new Employee();
        copyProperties(person, employee);

        System.out.println("Employee Name: " + employee.name); // 输出: Employee Name: John
        System.out.println("Employee Age: " + employee.age);   // 输出: Employee Age: 30
    }

    public static void copyProperties(Object source, Object target) throws Exception {
        // 获取源对象的所有字段
        Field[] fields = source.getClass().getFields();

        for (Field field : fields) {
            // 获取字段的值
            Object value = field.get(source);

            // 获取目标对象中同名的字段
            Field targetField;
            try {
                targetField = target.getClass().getField(field.getName());
            } catch (NoSuchFieldException e) {
                continue; // 如果目标对象中没有这个字段，则跳过
            }

            // 检查字段类型是否匹配
            if (targetField.getType().equals(field.getType())) {
                // 将值设置到目标对象中
                targetField.set(target, value);
            }
        }
    }
}
```
### 注解

1. 注解的作用
- 标注和说明: 注解通常用来标注或说明代码中的某些元素。例如，@Override注解表明一个方法是重写父类或接口的方法，这是一种对代码的说明。
- 代码行为的调整: 某些注解会影响代码的行为。例如，@Deprecated注解标注某个方法已经过时，编译器会在使用该方法时发出警告。
- 与框架和工具的集成: 许多框架（如Spring、Hibernate）和工具（如JUnit、编译器）使用注解来配置和控制代码的行为。例如，@Autowired注解用于在Spring中自动注入依赖。

2. 注解的使用场景
- 编译时处理: 注解可以在编译时被处理，例如生成额外的代码、文档，或进行代码校验。例如，@SuppressWarnings注解可以告诉编译器忽略特定的警告。
- 运行时反射: 注解可以在运行时通过反射机制读取和使用。例如，JUnit在运行时通过反射读取@Test注解来识别哪些方法是测试方法。
- 框架配置: 许多Java框架通过注解来配置和管理对象的行为，这种配置方式比传统的XML配置更加直观和简洁。

**Java的注解和Python的装饰器有什么不同？**

1. 功能与用途：
- Java注解主要用于提供元数据，标记类、方法、字段等。主要用于配置和框架集成，如依赖注入、ORM映射等。
- Python装饰器用于动态改变函数或方法的行为，可以在运行时修改代码逻辑，常用于日志记录、权限控制等。
2.	实现方式：
- Java注解是静态的，不能直接改变代码逻辑，通过编译器或框架在编译时或运行时处理。
- Python装饰器是高阶函数，可以在运行时动态生成、修改或应用，具有更强的灵活性。
3.	使用场景：
- Java注解常见于企业级应用中的配置和元数据标注。
- Python装饰器用于简化代码、提高复用性，特别是在Web开发和数据处理领域。
  
两者虽然都是用于增强代码的功能性，但Java注解偏向于静态配置，而Python装饰器偏向于动态行为修改。