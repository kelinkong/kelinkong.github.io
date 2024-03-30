---
title: C++运算符重载的一些规则
date: 2022-08-16 07:33:02
categories: CPP
tags: [CPP]
---


1. C++规定重载后的运算符的操作对象必须至少有一个使用户定义的类型
2. 使用运算符不能违反运算符原来的语法规则
3. 不能修改运算符的优先级
4. 不能进行重载的运算符：成员运算符`.`、作用域运算符`::`, 条件运算符，`sizeof`运算符等

**<<运算符的重载**

使用全局函数(友元函数)进行重载

`s.operator<<(cout);`可以简写为：`s<<out;`

`MyClass& operator+(MyClass& s1,MyClass& s2)`：调用时：

`s1+s2`等价于`s1.operator+(s2)`

```cpp
class MyClass{
public:
	MyClass(int data):data(data){}
	friend ostream& operator<<(ostream& os){
		os<<data<<endl;
	}
private:
	int data;
}
ostream& operator<<(MyClass& s,ostream& os){
		os<<data<<endl;
		return os;//为了重复调用所以必须要返回os
	}
int main(){
	MyClass s(10);
	s.operator<<(cout);//s<<out;
	cout<<s<<endl;
}
```
**`=,[],->,()`这几个符号只能在成员函数中进行重载，即有一个默认参数一定是类。**
为什么？
[为什么不能在类外重载](https://blog.csdn.net/u014610226/article/details/47679323)

