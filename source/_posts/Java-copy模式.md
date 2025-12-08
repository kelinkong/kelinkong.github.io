---
title: Java-copy模式
date: 2025-12-08 16:07:01
categories: [Java]
---

## Java中的赋值

在 Java 中，对象变量（非基本数据类型，如 int、boolean 等）存储的实际上是 引用（或者说内存地址），它指向堆（Heap）上的实际对象。

当使用等号进行赋值时，默认执行的是 引用赋值（Reference Assignment）：

- ObjectA = ObjectB

这个操作不会创建新的对象。

它使得变量 ObjectA 存储的引用与变量 ObjectB 存储的引用 相同，它们现在都指向 堆上同一个对象实例。

**这意味着通过 ObjectA 或 ObjectB 对对象进行的任何修改，都会影响到同一个底层数据。**

```java
MyObject a = new MyObject(); // 创建对象1
MyObject b = a;              // 引用赋值，b 和 a 都指向 对象1
b.setValue(10);              // 通过 b 修改 对象1
// 此时 a.getValue() 也是 10
```

### C++ (默认复制构造函数或重载运算符)
在 C++ 中，一个普通的 类对象变量 通常直接包含其数据成员。

当使用等号进行赋值时，默认执行的是 成员逐个复制（Memberwise Copy）：

- ObjectA = ObjectB

如果 ObjectA 已经存在，这个操作会调用 赋值运算符（Assignment Operator, 即 operator=）。

如果没有显式定义，默认的赋值运算符 会将 ObjectB 的所有非静态数据成员 逐个复制 给 ObjectA。

这会创建一个 新的独立的对象状态（虽然数据一样），ObjectA 和 ObjectB 仍是两个独立的对象。

```cpp
MyClass A;      // 对象 A
MyClass B;      // 对象 B
B.setValue(10);
A = B;          // 默认赋值：将 B 的数据成员复制给 A
A.setValue(20); // 修改 A 的值，不会影响 B
// 此时 B 的值仍是 10
```

#### 注意 (C++ 浅拷贝陷阱)：
默认的逐个复制对于对象内包含 动态分配的内存（例如，一个类中有一个 int* 指针）时，会导致 浅拷贝 问题。[C++浅拷贝](https://kelinkong.github.io/2021/09/18/%E3%80%90c++%E3%80%91%E6%B7%B1%E6%8B%B7%E8%B4%9D%E5%92%8C%E6%B5%85%E6%8B%B7%E8%B4%9D/)

赋值后，`ObjectA` 和 `ObjectB` 的指针成员将指向 同一块 动态内存。

当其中一个对象（如 `ObjectB`）被销毁时，它会释放这块内存，导致另一个对象（`ObjectA`）的指针变成悬空指针，再次访问或销毁时会导致程序崩溃。

为了解决这个问题，在 C++ 中包含动态内存的类通常需要显式定义 深拷贝 的 复制构造函数 和 赋值运算符，以确保新对象拥有自己独立的数据副本。

## Java是怎么避免浅拷贝陷阱的？

**简而言之：** Java 避免 C++ 默认赋值导致的浅拷贝陷阱，是因为它们对“赋值”的定义根本不同。

### 1\. Java 如何避免浅拷贝陷阱？

Java 通过其 **引用的本质** 避免了 C++ 默认赋值运算符所导致的浅拷贝陷阱。

#### 1.1 赋值运算符（`=`）的角色不同

| 语言 | 操作符 (`=`) 的默认行为 | 结果 |
| :--- | :--- | :--- |
| **Java** | **引用赋值 (Reference Assignment)** | `A = B` 意味着 `A` 和 `B` 都指向堆内存中的 **同一对象**。这永远不会创建新的数据副本，因此不会发生 C++ 意义上的浅拷贝（即复制对象数据时只复制指针地址）。 |
| **C++** | **数据赋值 (Data Copying)** | 默认调用赋值运算符，将 `B` 对象的数据成员 **逐个复制** 给 `A` 对象。当数据成员是指针时，复制的只是指针的值（地址），导致两个对象共享同一块堆内存。 |

#### 1.2 Java 的内存管理机制

在 Java 中，当你说：

```java
MyObject obj1 = new MyObject();
MyObject obj2 = obj1; // 引用赋值
```

这是 **引用赋值**。

  * `obj1` 和 `obj2` 都是 **遥控器**。
  * 它们都指向 **堆上同一个电视机（对象）**。

因此，当你通过 `obj2` 改变电视机的频道时，`obj1` 看到的也是新频道。既然没有发生数据的“复制”行为，自然就不会触发 C++ 那种“复制指针”的陷阱。

#### 1.3 `clone()` 方法与浅拷贝风险

需要注意的是，虽然 Java 的 **默认赋值** 是安全的，但当你尝试在 Java 中进行 **显式拷贝** 时，风险仍然存在：

Java 的 `Object` 类中自带的 `clone()` 方法，其默认实现是 **浅拷贝（Shallow Copy）**。

  * **浅拷贝**：它创建了一个新对象，并将原对象的所有字段值复制给新对象。
      * 如果字段是 **基本类型** 或 **不可变对象**（如 `String`），则安全。
      * 如果字段是 **可变对象引用**（如 `ArrayList` 或自定义类），则只复制了该引用地址。新对象和原对象会 **共享** 内部的这个可变对象，当你修改其中一个时，另一个也会被影响。

所以，Java 避免了 C++ **赋值** 带来的默认陷阱，但如果你不正确地使用 `clone()` 或自定义拷贝方法，**浅拷贝的风险依然存在**。


### 2\. 如何在 Java 中实现 C++ 风格的独立拷贝（深拷贝）？

实现 C++ 风格的独立拷贝（即 **深拷贝 Deep Copy**），目的是创建一个全新的、与原对象在数据上完全独立的副本，包括所有嵌套的子对象。

实现深拷贝主要有两种推荐的方法：**拷贝构造函数** 和 **重写 `clone()`**。

#### 方式一：使用拷贝构造函数（推荐）

这是最常用、最清晰、也是最推荐的方法。它在创建新对象时，递归地为所有可变（Mutable）引用字段创建新的实例。

```java
public class Employee {
    private String name;
    private Department department; // 假设 Department 是一个可变对象

    // 原始构造函数
    public Employee(String name, Department department) {
        this.name = name;
        this.department = department;
    }

    // ⭐ 拷贝构造函数 (执行深拷贝)
    public Employee(Employee original) {
        this.name = original.name; // String 是不可变的，浅拷贝即可

        // 关键：对可变引用类型执行深拷贝（调用其拷贝构造函数）
        this.department = new Department(original.department); 
    }
    
    // ... 其他方法
}
```

**优点：** 1.  **明确性高：** 意图清晰，易于维护。
2\.  **更灵活：** 不依赖于 `Cloneable` 接口。


#### 方式二：正确重写 `clone()` 方法

使用 `clone()` 需要遵循特定的步骤来实现深拷贝：

1.  实现 `Cloneable` 接口（标记接口）。
2.  重写 `Object` 的 `protected clone()` 方法，并将其访问权限改为 `public`。
3.  在方法中，先调用 `super.clone()` 获得浅拷贝。
4.  **手动** 对所有可变引用字段进行深拷贝。

<!-- end list -->

```java
public class Department implements Cloneable {
    private String name;
    // ...

    @Override
    public Object clone() throws CloneNotSupportedException {
        // 1. 获得对象的浅拷贝
        Department newDept = (Department) super.clone(); 
        
        // 2. 如果 Department 内部还有可变对象，需要在这里递归进行深拷贝。
        // 例如：newDept.staff = new ArrayList<>(this.staff);
        
        return newDept;
    }
}
```

**优点：** 1.  实现了 Java 标准库的约定。

**缺点：** 1.  **侵入性强：** 必须实现 `Cloneable` 接口。
2\.  **不安全：** `clone()` 返回 `Object` 类型，需要进行类型转换。
3\.  **复杂：** 必须处理 `CloneNotSupportedException` 异常。

#### 方式三：通过序列化（不推荐用于日常操作）

通过将对象写入输出流（序列化）再从输入流读出（反序列化），可以最简单地保证深拷贝，因为序列化过程会为整个对象图创建全新的副本。

**缺点：** 1.  **性能差：** 涉及 I/O 操作，比前两种方法慢得多。
2\.  **限制：** 对象及其所有内部对象必须实现 `Serializable` 接口。

## 可变对象和不可变对象

我的类A中有一个Integer对象，我使用B=A，那么当我修改B中的Integer对象时，A中的也会跟着改变？

这是一个非常关键的问题，它涉及到 **Java 的引用赋值** 和 **Integer 的不变性**。

答案是：**不会**。当你修改 `B` 中的 `Integer` 字段时，**`A` 中的 `Integer` 字段不会跟着改变**。

### 为什么不会改变？

虽然 `B = A` 是引用赋值（意味着 `A` 和 `B` 指向堆上同一个 `A` 类实例），并且它们共享了内部的 `Integer` 字段，但这种共享是安全的，原因在于 `Integer` 类的核心特性：**不可变性 (Immutability)**。

让我们通过一个假设的例子来详细分析这个过程：

#### 1\. 初始状态 (引用共享)

假设你有一个 `MyClass`，其中包含一个 `Integer` 字段：

```java
class MyClass {
    Integer value; // 字段
    public MyClass(Integer v) {
        this.value = v;
    }
}

MyClass A = new MyClass(100);
MyClass B = A; // A 和 B 引用赋值，指向同一个 MyClass 实例
```

  * 此时，`A.value` 和 `B.value` 都指向 **同一个 `Integer(100)` 对象**。
      * `A` 指向 Object X。
      * `B` 也指向 Object X。
      * Object X 内部的 `value` 字段指向 Integer(100)。

#### 2\. 执行 "修改" 操作 (断开共享)

当你执行一个看似修改 `B` 中 `Integer` 字段的操作时，实际上并不是在修改那个 `Integer` 对象，而是在 **改变 `B` 内部的引用指向**：

```java
// 你尝试修改 B.value 的值
B.value = 200; // 假设 B 类中有一个公共的 setter 或直接访问字段
```

这个操作的底层机制是：

1.  `Integer` 对象是 **不可变** 的。你无法修改那个值为 `100` 的对象本身。
2.  Java 会创建一个 **新的 `Integer(200)` 对象**。
3.  `B.value` 这个字段的 **引用** 被更新，使其指向这个 **新的 `Integer(200)` 对象**。
4.  因为 `A` 和 `B` 仍然指向同一个 `MyClass` 实例 (Object X)，所以 `A.value` 和 `B.value` 都改变了。

#### 3\. 陷阱的真正发生条件：可变对象（Mutable Object）

如果你的类 `A` 中包含的不是 `Integer`（不可变对象），而是像 **`ArrayList`** 或你自定义的 **可变对象**，那么陷阱就会发生：

```java
class MyMutableClass {
    ArrayList<String> data; // 可变对象
    // ... 构造函数等
}

MyMutableClass A_mut = new MyMutableClass(new ArrayList<>(Arrays.asList("X")));
MyMutableClass B_mut = A_mut; // 引用赋值，共享同一个 MyMutableClass 实例

// 尝试修改 B 中的字段内容
B_mut.data.add("Y"); // ⚠️ 陷阱发生！
```

在这个例子中：

1.  `B_mut.data.add("Y")` 是在调用 `ArrayList` 对象的 **内部修改方法**。
2.  这个方法修改了 **堆上共享的那个 `ArrayList` 实例**。
3.  因为 `A_mut` 和 `B_mut` 都指向同一个 `ArrayList`，所以 `A_mut.data` 也会看到新的元素 `"Y"`。

### 总结 🚀

| 字段类型 | 赋值 (`B = A`) 后修改 `B` 中的字段 | 结果 |
| :--- | :--- | :--- |
| **`Integer` (不可变)** | `B.value = 200;` | **A 不变**。因为该操作创建了新的 `Integer` 对象并更新了 `B` 的引用。 |
| **`ArrayList` (可变)** | `B.data.add("Y");` | **A 跟着改变**。因为该操作修改了共享的 `ArrayList` 对象的内部状态。 |

**对于 不可变对象 (Immutable Object)，当对它进行“修改”操作时，本质上就是创建了一个新的对象。**

### 常见的可变对象和不可变对象

常见的 **可变对象**（Mutable Objects）和 **不可变对象**（Immutable Objects）是 Java 语言设计的基础。

#### 🔒 常见的不可变对象 (Immutable Objects)

不可变对象是线程安全的，它们的值在创建后不会改变。

| 类别 | 示例类 | 备注 |
| :--- | :--- | :--- |
| **基本类型包装类** | `java.lang.Integer` | 所有基本数据类型的包装类，包括 `Byte`, `Short`, `Long`, `Float`, `Double`, `Character`, `Boolean`，都是不可变的。 |
| **字符串** | `java.lang.String` | 最常用的不可变对象。所有修改字符串的操作（如 `concat()`）都会返回一个新的 `String` 实例。 |
| **日期时间 API** | `java.time.LocalDate` | Java 8 引入的新的日期时间 API 中的核心类，如 `LocalDateTime`, `ZonedDateTime` 等，都是不可变的。 |
| **通用类** | `java.lang.Class` | 表示类的对象，是不可变的。 |
| **集合 (部分)** | `java.util.Collections.unmodifiableList(...)` | 通过工具类创建的 **不可修改的集合视图**，虽然不是真正的不可变，但不能通过视图修改内容。 |

#### 🔓 常见的可变对象 (Mutable Objects)

可变对象可以随时修改其内部状态，因此在多线程环境下需要特别注意同步问题。

| 类别 | 示例类 | 备注 |
| :--- | :--- | :--- |
| **日期时间 API (旧)** | `java.util.Date` | 老的日期时间类，它的时间可以被 `setTime()` 方法修改。 |
| | `java.util.Calendar` | 同样是可变的，可以通过 `set()` 方法修改内部日期字段。 |
| **字符串构建** | `java.lang.StringBuilder` | 用于在单线程环境中高效地修改字符串。 |
| | `java.lang.StringBuffer` | 线程安全的版本，用于在多线程环境中高效地修改字符串。 |
| **集合框架** | `java.util.ArrayList` | 可以通过 `add()`, `remove()`, `set()` 等方法修改列表内容。 |
| | `java.util.HashMap` | 可以通过 `put()`, `remove()` 等方法修改映射内容。 |
| **IO 框架** | `java.io.File` | 表示文件路径的对象，虽然文件路径本身通常不变，但它不符合不可变对象的严格定义。 |
| **数组** | 所有数组类型 (e.g., `String[]`, `int[]`) | 数组的内容可以被直接修改。 |