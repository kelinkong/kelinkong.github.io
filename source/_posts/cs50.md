---
title: CS50课程学习笔记
date: 2023-10-08 03:26:51
categories: [Operating-System]
---

## Background

### Computational Thinking

-  input --> black box --> output
-  binary/bit: A bit is a zero or one
- text: using ASCII
- Emojis: Unicode
- RGB: three numbers
- Images, Video and Sound are simply collections of RGB values

### Algorithms

- Problem-solving is central to computer science and computer programming

### Pseudocode

- function：pick up
- conditions：if elseif
- Boolean expression
- loops：for/while

## C

### compiler

**command line interface**：CLI， send commands to the computer

```c
#include<stdio.h>

int main(){
    printf("hello world\n");
}

make hello
./hello
```

*这里并没有写makefile文件，为什么可以直接编译呢？*
*make程序默认会根据源代码文件的后缀名，自动生成并使用一个默认的Makefile规则进行编译。对于c语言的源代码，会有一个默认规则，类似如下：*
```makefile
%.o : %.c # % 是通配符，匹配任意字符串，匹配所有.c结尾的文件，生成以.o结尾的目标文件
    $(CC) $(CFLAGS) -c $< -o $@ # $(CC) 变量，使用CC编译器。 $(CFLAGS)编译参数， -c表示进行编译而不链接， $< 取出第一个前置文件的名字，即.c文件，-o指定输出目标文件，&@ 取出目标文件的名字，既.o文件
```

这里提到一个进行编译而不链接。回顾一下c语言编译的过程
1. 预处理，生成扩展后的.i文件。删除所有注释、#define 宏展开、文件包含 #include<文件名>
2. 编译，汇编代码文件.s。会进行语法检查
3. 组装。将汇编文件转化为机器码。生成.o文件
4. 链接。链接是将库文件包含在我们的程序中的过程。生成可执行文件.out

**静态成员变量是在哪一个阶段被初始化的呢？**

*我们都知道静态成员变量是在运行`main`函数前初始化，那么究竟是在编译的哪一个阶段呢？*
1. 定义在类内部的静态成员变量,其初始化是在编译期初期完成的。
2. 定义在类外部的静态成员变量,其初始化是在链接阶段完成的。

区别在于:

对于类内部定义的静态成员变量,编译器可以在编译当前类的定义时直接初始化它。

而对于类外部定义的静态成员变量,需要到链接阶段不同的目标文件合并时,由链接器完成初始化操作。

**预编译阶段，编译器会把#include包含的头文件内容展开Inline，但是对于lib库的引用则不会展开，而是保留该引用。真正使用库里的目标文件是在链接阶段。由链接器解析引用，并从外部库获取目标代码进行连接。**

## Algorithm
- linear and binary search
- data structures
- sorting
- recursion

## Memory

**内存地址为什么用16进制存放？**
1. 紧凑表示：使用尽可能少的数字来表示较大的内存空间
2. 与CPU寻址匹配： CPU使用总线地址线表示内存地址，地址线数量是2的指数倍（如8条、16条），这与16进制表示是匹配的
3. 转换为二进制方便，16进制中的每一个恰好对应2进制中的4位，可以非常方便的转换为二进制数。

### pointer

指针的本质就是一个地址变量。指向的是操作系统给模拟出来的虚拟内存空间的地址。

```c
int a = 50;
int *p = &a; // p是一个指针，(*p)表示获取指针所指向的值，&符合代表取地址
```
在c语言中
`string`类型本质就是一个指针，以`'\0'`位标识符结尾。

```c
char *s = "HI!";
printf("%c\n",s[0]); // H
printf("%c\n",s[1]); // I
printf("%c\n",s[2]); // !
printf("%c\n",s[3]); // 
printf("%c\n",s[4]); // %
```
为什么没有提示数组越界呢？

*在c语言中数组边界检查并不是强制的。所以这访问越界并没有报错，但是可能会导致其他问题。当访问s[3]之后的值时，读取的是垃圾内存的值。*

### Valgrind

valgrind是一个检查是否有内存泄漏的工具，比如使用malloc但是没有使用free。

```c
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    int *x = malloc(3 * sizeof(int));
    x[0] = 72;
    x[1] = 73;
    x[2] = 33;
}
```

`make test`

`valgrind --leak-check=full ./test`

```
==61782== HEAP SUMMARY:
==61782==     in use at exit: 12 bytes in 1 blocks
==61782==   total heap usage: 1 allocs, 0 frees, 12 bytes allocated
==61782== 
==61782== 12 bytes in 1 blocks are definitely lost in loss record 1 of 1
==61782==    at 0x483B7F3: malloc (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
==61782==    by 0x10915E: main (in /home/kelin/cs50/test)
```

**valgind可以检查哪些类型的错误呢？**

1. 内存管理错误：访问越界、未释放、访问未初始化的内存等
2. 线程错误：程序未正确join线程、线程同步错误（多个线程争用资源未加锁）
3. I/O操作错误：文件描述符泄漏、socket使用错误
4. 未定义行为：访问未初始化的变量

### 在c语言中，打开文件是如何实现的？

1. open系统调用
2. 分配文件描述符：文件描述符是内核用来标识这个文件打开状态的整数
3. 更新文件表：将文件描述符放入进程的文件描述符表，该表维护了进程打开文件的状态
4. 返回文件描述符：open系统调用返回文件描述符给应用程序
5. read/write调用
6. 关闭

## network

**`curl`**

curl用来发送各种HTTP请求，获取或传输数据。

|参数|作用|例子|
|---|---|---|
|-X|指定http请求的方法|-X GET|
|-d|指定发送的数据体|-d 'data=test'|
|-I|只显示响应头信息，不显示响应内容||
|-w|将响应头信息保存到文件||
|-O|将服务器访问保存为文件|curl -O example.com/file1.zip|