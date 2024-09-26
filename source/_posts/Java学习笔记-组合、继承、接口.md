---
title: Java学习笔记-组合、继承、接口
date: 2024-09-25 09:30:55
categories: [Java]
---

## 组合和聚合

- 组合（Composition）经常用来表示“拥有” 关系（has-a relationship）。例如，“汽车拥有引擎”

- 聚合（Aggregation）动态的组合。

**组合**：表示整体与部分的关系，整体和部分的生命周期一样，整体不存在了，部分也不存在了。
```java
class Engine {
    public void start() {
        System.out.println("Engine starting");
    }
}
class Car {
    private Engine engine;
    public Car() {
        engine = new Engine();
    }
    public void start() {
        engine.start();
    }
}
```
**聚合**：表示整体与部分的关系，整体和部分的生命周期不一样，整体不存在了，部分还存在。
```java
class Department {
    private String name;

    public Department(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}

class Company {
    private List<Department> departments; // Company 聚合多个 Department

    public Company() {
        this.departments = new ArrayList<>();
    }

    public void addDepartment(Department department) {
        departments.add(department);
    }

    public void showDepartments() {
        for (Department department : departments) {
            System.out.println(department.getName());
        }
    }
}

public class Main {
    public static void main(String[] args) {
        Department hr = new Department("HR");
        Department it = new Department("IT");

        Company company = new Company();
        company.addDepartment(hr); // Company 聚合 Department
        company.addDepartment(it);

        company.showDepartments();
    }
}
```

## 为什么推荐使用组合而不是继承
- 继承在某些特定情况下是有用的，比如实现“是一个”的关系（如 Dog 是一种 Animal），但由于它带来的耦合问题、封装性破坏以及灵活性不足，推荐尽可能优先使用组合。
- 组合更加灵活、可维护性更好，能够在不破坏现有代码结构的情况下进行修改和扩展。因此，在大多数情况下，推荐使用组合而不是继承。

> 组合和继承都可以实现在不修改源代码的情况下，扩张已有的类的功能。但是组合更加灵活，因为它可以在运行时动态地改变对象的行为。
>
> 组合是一种包含关系，一个类的对象包含另一个类的对象，通过调用另一个类的方法来实现功能。

## 继承和接口的区别

继承：
- 继承是面向对象编程中的一种机制，允许一个类（子类）从另一个类（父类）继承其属性和方法。
- 继承表示的是 "is-a" 关系，比如 Dog 继承自 Animal，意味着狗是一种动物。
- 在继承关系中，子类继承父类的非私有（private）的成员变量和方法，并且可以重写父类方法。
- Java 中类的继承是单继承的，一个类只能有一个父类。


接口：

- 接口是一种抽象类型，定义了类必须实现的行为，但不提供具体实现。
- 接口表示的是 "can-do" 或者 "contract-based" 关系，意味着类必须实现接口中声明的方法。
- 一个类可以实现多个接口，这使得接口在多态设计中非常有用。
- Java 支持多接口实现，一个类可以实现多个接口。

```java
interface Flyable {
    void fly();
}

interface Swimable {
    void swim();
}

class Duck implements Flyable, Swimable {
    @Override
    public void fly() {
        System.out.println("Duck is flying");
    }

    @Override
    public void swim() {
        System.out.println("Duck is swimming");
    }
}
```

## Java中的接口和C++中的抽象类的区别

### 语法和定义
Java 的接口：

- 接口使用 interface 关键字定义。
- 接口中的方法默认是抽象的（不提供实现），并且所有字段默认是 public static final（常量）。
- Java 8 及之后，接口可以有 default 和 static 方法，这些方法可以有实现。
- 接口中的所有方法默认是 public，即使没有显式声明。
```java
public interface Flyable {
    void fly(); // 抽象方法
    default void prepareForFlight() {
        System.out.println("Preparing for flight");
    }
}
```

C++ 的抽象类：
- 抽象类使用 class 关键字定义。
- 抽象类可以包含抽象方法（纯虚函数）和具体方法（有实现的方法）。
- 抽象类可以包含成员变量，并且可以是各种访问级别：private、protected 或 public。
- 纯虚函数在 C++ 中用 = 0 标记，表示该方法没有实现，必须由子类实现。

```cpp
class Flyable {
public:
    virtual void fly() = 0;  // 纯虚函数
    void prepareForFlight() {
        std::cout << "Preparing for flight" << std::endl;
    }
};
```
### 构造函数和成员变量
Java 的接口：

接口中不能有构造函数，因为接口不能直接实例化。
接口不能包含实例变量，只有 static 和 final 常量。

C++ 的抽象类：

抽象类可以有构造函数，尽管不能直接实例化抽象类，但它可以用来初始化子类。
抽象类可以包含实例变量，这些变量可以在子类中继承。

### 性能
Java 的接口：

接口的方法调用通过虚拟机的动态分派机制来实现，通常比直接调用类中的方法略慢。
随着 JVM 优化，接口的性能已经非常接近普通类的方法调用。

C++ 的抽象类：

C++ 使用虚函数表（vtable）来处理多态调用，性能相对较高，因为是在编译时确定虚函数表，运行时通过指针查找。
由于虚函数机制的直接性，C++ 抽象类的多态调用在许多情况下比 Java 的接口实现更快

## Java是如何实现多态的

### 编译时多态（静态多态）
方法重载（Method Overloading）：

方法重载是指同一个类中可以有多个方法名相同，但参数类型或参数数量不同的方法。根据传入参数的不同，编译器会在编译时决定调用哪个版本的重载方法。

这是静态多态的表现形式，因为在编译阶段就确定了调用的方法。

### 运行时多态（动态多态）
方法重写（Method Overriding）：

运行时多态的核心在于方法重写。当一个子类继承父类，并在子类中提供了对父类方法的不同实现时，Java 会在运行时根据对象的实际类型决定调用哪一个方法。

通过父类引用指向子类对象，调用重写的方法时，实际执行的是子类的实现。这是在运行时根据对象的实际类型动态决定的，因此称为动态多态。
实现方式：虚方法机制：

Java 中的所有非 final 和 private 方法都被认为是虚方法。虚方法是可以被子类重写的方法，Java 使用虚方法表（vtable）来实现多态。

当调用一个方法时，JVM 会通过对象的实际类型在运行时决定调用哪个方法。

```java
class Animal {
    public void sound() {
        System.out.println("Animal makes a sound");
    }
}

class Dog extends Animal {
    @Override
    public void sound() {
        System.out.println("Dog barks");
    }
}

class Cat extends Animal {
    @Override
    public void sound() {
        System.out.println("Cat meows");
    }
}

public class Main {
    public static void main(String[] args) {
        Animal myAnimal = new Animal();  // Animal 类型引用
        Animal myDog = new Dog();        // 父类引用指向子类对象
        Animal myCat = new Cat();        // 父类引用指向子类对象

        myAnimal.sound();  // 调用的是 Animal 的 sound() 方法
        myDog.sound();     // 调用的是 Dog 的 sound() 方法
        myCat.sound();     // 调用的是 Cat 的 sound() 方法
    }
}
```

### 接口多态
Java 通过接口也可以实现多态。当一个类实现了某个接口时，接口引用可以指向该类的实例，并且在运行时会根据对象的实际类型执行具体的方法实现。

### 当传入的参数为一个接口时，如何确定调用的是哪一个接口的实现？
如何确定调用的是哪一个接口的实现？

- 编译时：当你传入一个接口类型的参数时，编译器只会检查这个对象是否实现了该接口，但不会关心具体是哪一个实现类。
- 运行时：在程序运行时，JVM 会根据**传入对象**的实际类型决定调用具体实现类的方法。
