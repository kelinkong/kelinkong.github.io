---
title: C++代码调试
date: 2023-10-10 08:15:48
categories: [C++]
---

## GCC 工作流程

|说明|文件后缀|参数|
|---|---|---|
|预处理|.c|-|
|编译|.i|-E|
|汇编|.s|-S|
|链接|.o|-c|

```bash
# 预处理。 -o 参数是为了指定编译后的文件名
gcc -E test.c -o test.i
# 编译
gcc -S test.i -o test.s 
# 汇编
gcc -c test.s -o test.o
#链接
gcc test.o -o test
```

|gcc编译选项|说明|
|---|---|
|-g|生成调试信息，即可调试|
|-D|指定一个宏|
|-l|指定链接库|
|-std|指定C++版本|
|-I|指定头文件路径|
|-Wall|打开所有警告信息|

**`-D`参数应用场景：**
在发布程序的时候，一般会将所有log输出去掉，如果不去掉会影响程序的执行效率。这时候可以在编译的时候加上`-D`参数，定义一个宏，然后在程序中使用这个宏，如果是发布程序，就不定义这个宏，这样就可以在编译的时候去掉所有的log输出。

### 多文件编译
```bash
gcc -o test string.c main.c
```

### gcc和g++
**在编译阶段：**
- 后缀为.c的文件，gcc会将其当做C语言源文件，g++会将其当做C++语言源文件。
- 后缀为.cpp的文件，两者都会将其当做C++语言源文件。
- g++会调用gcc，对于C++代码，两者是等价的，也就是说gcc和g++都可以编译C/C++代码

**在链接阶段：**
- gcc和g++都可以自动连接到标准C库
- g++会自动连接到标准C++库，gcc如果要链接到标准C++库需要加参数`-lstdc++`

## gdb调试

**gdb调试的代码必须添加-g参数，-g的作用是在可执行文件中加入源代码的信息，比如可执行文件中第几条机器指令对应源代码的第几行。**

*使用CMake编译的，添加`-DCMAKE_BUILD_TYPE=Debug`参数*。

### 启动和退出gdb

**gdb进程启动之后，需要被调试的应用程序是没有执行的。**

```bash
gdb <program> # 启动

set args <arg1> <arg2> # 设置参数,启动应用程序之前可能需要传参

show args # 查看参数
```

**在gdb中启动程序：**
```bash
(gdb) run
(gdb) start # 会阻塞到main函数第一行
```

### 常用参数
|参数|说明|示例|
|---|---|---|
|run|启动程序|
|continue = c|继续执行程序|
|quit == q|退出gdb|
|list == l|列出源代码|l 行号, l 函数名， l 文件名：行号|
|break == b|设置断点|b 行号, b 函数名， b 文件名：行号， b 行数 if 变量名=某个值|
|info == i|查看信息|i breakpoints, i locals, i args|
|delete = del = d|删除断点|d 断点号|
|disable = dis|禁用断点|disable 断点号|
|enable = ena|启用断点|enable 断点号|
|print = p|打印变量|p 变量名|
|ptype|打印变量类型|ptype 变量名|
|display|跟踪查看变量|display 变量名|
|step = s|单步执行，进入函数|
|finish|执行到当前函数返回为止|
|next = n|单步执行，不进入函数|
|until|跳出循环|
|set|修改变量值|set 变量名=值|

## CMake

参考[Cmake快速入门](https://www.hahack.com/codes/cmake/)

cmake的基本语法：

```CMake
cmake <option> <path>
cmake -DCMAKE_BUILD_TYPE=Debug ..
```

### 指令

|含义|指令|
|---|--|
|编译|cmake `<path>`, 当前目录为`.`,上一级目录为`..`|
|清理cmake缓存|`rm -rf CMakeFiles/ CMakeCache.txt cmake_install.cmake Makefile`|

### 注意事项

- 每一个层级都需要写CMakeLists.txt文件
- 需要在父层级添加子层级的目录
- 需要在父层级添加子层级的库/子目录包含可执行文件

### 示例

**文件结构：**
```
├── cmake_notes
│   ├── build
│   └── math
│       ├── CMakeLists.txt
|       ├── myfunction.h
│       └── myfunction.cpp
├── CMakeLists.txt
└── main.txt
```

第一层级的CMakeLists.txt：

```CMake
cmake_minimum_required(VERSION 3.1)

# 项目名称
project(Demo)

# 是否用自己的 MathFunctions库
option(USE_MYMATH "Use provided math implementation" OFF)

# 添加自己的 MathFunctions库
add_definitions(-DUSE_MYMATH)

# 是否加入MathFunctions库
if (USE_MYMATH)
    include_directories("${PROJECT_SOURCE_DIR}/math")
    add_subdirectory(math)
endif (USE_MYMATH)

# add_executable(Demo main.cpp print.cpp)

# 查找当前目录下的所有源文件，保存到DIR_SRCS变量中
aux_source_directory(. DIR_SRCS)

# 生成指定目标
add_executable(Demo ${DIR_SRCS})

# 添加子目录
# add_subdirectory(math)

# 链接到库
if (USE_MYMATH)
    target_link_libraries(Demo MathFunctions)
endif (USE_MYMATH)

target_link_libraries(Demo -lpthread)
```

**在源代码中使用：**
```cpp
#ifdef USE_MYMATH
    cout<<"use my math"<<endl;
#else
    cout<<"use system math"<<endl;
#endif // USE_MYMATH
```

**子目录的CMakeLists文档，需要add_executable吗?**

子目录包含可执行文件：

```cmake
# 在子目录的 CMakeLists.txt 中
add_executable(my_executable main.cpp other_source.cpp)
```

子目录只包含库代码：

```cmake
# 在子目录的 CMakeLists.txt 中
add_library(my_library my_source.cpp)
```

这将创建一个库目标而不是可执行文件，以便在主目录的CMakeLists.txt文件中或其他子目录中使用。
