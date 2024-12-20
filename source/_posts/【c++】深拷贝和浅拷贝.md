---
title: C++深拷贝和浅拷贝
date: 2021-09-18 07:33:02
categories: [C++]
---

## 两个的区别

- 在未定义显示拷贝构造函数的情况下，系统会调用默认的拷贝函数——即浅拷贝，它能够完成成员的一一复制。当数据成员中没有`指针`时，浅拷贝是可行的，但当数据成员中有指针时，如果采用简单的浅拷贝，`则两类中的两个指针将指向同一个地址`，当对象快结束时，会调用两次析构函数，从而导致指针悬挂现象，所以，此时，必须采用深拷贝。

```cpp
//系统默认的构造函数
class Widget{
public:
    Widget();//default构造函数
    Widget(const Widget&rhs);//copy构造函数  浅拷贝函数
    Widget& operator=(const Widget &rhs);//copy assignment操作符
};

Widget w1;//调用default函数
Widget w2(w1);//调用copy构造函数
//w2还不存在，所以调用的是copy
w1=w2;//调用copy assignment操作符
//赋值函数，是在对象已经存在的情况下才会进行调用
Widget w3=w2;//调用copy构造函数
    
```

- 深拷贝和浅拷贝的区别就在于深拷贝会在堆内存中另外申请空间来存储数据，从而也就解决了指针悬挂的问题。`当数据成员中有指针时，必须要用深拷贝。`

## 解释

- c++默认的拷贝构造函数是浅拷贝
  - 浅拷贝就是对象数据成员之间的简单复制，如你设计了一个类而没有通过它的复制构造函数，当用该类的一个对象去给另一个对象赋值时所执行的过程就是浅拷贝，如：

```cpp
class A
{
public:
    A(int _data):data(_data){}//传入一个值_data，然后把_data的值给data
    A();
private:
    int data;
}

int main()
{
    A a(5);
    A b=a;//仅仅是数据成员之间的赋值
    //b.data=5;
}
```

**深拷贝**

```cpp
class A
{
public:
    A(int _size):size(_size)
    {
		data = new int[size];//指针变量名 = new type[内存单元个数]
    }
    A();
    ~A()
    {
        delete []data;
    }//析构时释放资源
private:
    int *data;
    int size;
}

int main()
{
    A a(5),b=a;
}
```

- 注：这里的b=a会造成未定义行为，因为类A中的copy构造函数是编译器生成的，所以b = a执行的是一个浅拷贝的过程。

`这里b的指针和a的指针指向了堆上的同一块内存，a和b析构时，b先把data执行的分配的内存释放了一次，而后a析构时又将这块已经被释放的内存再释放一次，对同一块内存执行2次以上释放的结果是未定义的，所以这会导致内存泄漏或程序崩溃。`

**深拷贝就是指当拷贝的对象中有对其他资源（如堆、文件、系统等）的引用时，对象另外开辟一块新的资源，而不再对拷贝对象中有对其他资源的引用的指针或引用进行单纯的赋值。**

```cpp
class A
{
public:
    A(int _size):size(_size)
    {
		data = new int[size];//指针变量名 = new type[内存单元个数]
    }
    A();
    A(const A &_A):size(_A.size)
    {
		data = new int[size];//开辟了一个新的空间
    }//深拷贝
    ~A()
    {
        delete []data;
    }//析构时释放资源
private:
    int *data;
    int size;
}

int main()
{
    A a(5),b=a;
}
```



