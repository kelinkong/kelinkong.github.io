---
title: 【c++】brtc项目开发记录
date: 2026-01-22 16:25:01
categories: [C++]
---

### 编译

同一个 C++ 进程里的所有 C++ SDK，ABI 必须一致，否则会出现莫名其妙的问题。

要使用
- 同一个编译器
- 同一个 libstdc++
- 同一个 _GLIBCXX_USE_CXX11_ABI

#### 什么是ABI？
ABI（Application Binary Interface）是应用程序二进制接口，它是一个接口，应用程序和动态链接库（DLL）之间进行通信的协议。 ABI 定义了函数调用约定、数据类型、内存布局等内容。

target_compile_definitions(<target> PUBLIC _GLIBCXX_USE_CXX11_ABI=0)

这里ABI=0表示使用旧版本ABI。

旧版本ABI的编译选项为：-D_GLIBCXX_USE_CXX11_ABI=0
新版本ABI的编译选项为：-D_GLIBCXX_USE_CXX11_ABI=1

#### 如何查看SDK的ABI版本？

```bash
readelf -Ws libbaidurtc.so | grep GLIBCXX_
```

### 初始化

C++的类的成员变量，都需要在构造函数中初始化吗？标准的写法是什么样子的？

C++ 成员变量：
- const / 引用 / 无默认构造 → 必须初始化
- 其他可以赋值，但 不推荐
- 初始化列表是唯一正确的工程实践

```cpp
class Foo {} {
public:
    Foo(int x, int y) : a_(x), b_(y), c_(0) {} // 初始化列表
}
```
