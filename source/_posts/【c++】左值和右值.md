---
title: 左值和右值
date: 2022-08-15 07:33:02
categories: cpp
tags: [cpp]
---

### 左值和右值

**左值：有地址的值**

**右值：只能放在等式右边的值**

- 常量
- 将亡值
- 算术表达式
```cpp
int f(){
	return 10;
}
int i = 10;//i左值，10右值
int j = f();//将亡值，用完即销
```
**函数返回值为左值引用，就可以放在等式左边**
```cpp
int& GetValue() {
	static int value = 10;
	return value;
}
int main() {
	
	int i = GetValue();
	cout << i << endl;//10
	GetValue() = 5;//左值引用
	cout << GetValue() << endl;//5
}
```
**非const引用，只能引用左值**
```cpp
void f(int& value);
f(10);//报错,正确 f(const int& value);
```
### 左值引用和右值引用
`const`加左值引用，可以兼容右值，相当于创建了一个临时变量，然后进行左值引用。`int tmp = 10;f(tmp);`
左值引用仅仅接受左值，右值引用仅仅接受右值。
```cpp
void MyPrint(int& value) {
	cout << "左值引用" << endl;
}
void MyPrint(int&& value) {
	cout << "右值引用" << endl;
}
void MyPrint(const int& value) {
	cout << "兼容左值和右值引用" << endl;
}
int main() {
	
	int a = 10;
	MyPrint(a);//左值引用
	MyPrint(10);//右值引用
}
```
### 移动语义
移动语义用来传递所有权，仅仅移动对象，而不进行复制
```cpp
#include<iostream>
using namespace std;

//传递所有权
//仅仅移动对象，而不复制，提高效率

class String {
public:
	String(const char* data) {
		printf("Creat!\n");
		m_size = strlen(data);
		m_data = new char[m_size];
		memcpy(m_data, data,m_size);
	}
	//深拷贝
	String(const String& other) {
		printf("copy\n");
		m_size = other.m_size;
		m_data = new char[m_size];
		memcpy(m_data, other.m_data, m_size);
	}
	//移动语义
	String(String&& other) {
		printf("move\n");
		m_size = other.m_size;
		m_data = other.m_data;
		other.m_size = 0;
		other.m_data = nullptr;
	}
	~String() {
		printf("String的析构函数\n");
		delete m_data;
	}
private:
	char* m_data;
	int m_size;
};

class ENT {
public:
	ENT(String&& data):m_data(move(data)){
		//这里要将data转换成一个右值才会调用移动构造函数
		printf("R Printf\n");
	}
	ENT(const String& data):m_data(data){
		printf("ENT Printf\n");
	}
	String m_data;
};

int main() {
	String s("abd");
	ENT t1(s);//调用了Creat和copy
	ENT t2("abd");//调用了Creat和move
	String str = "hello";
	String str1(move(str));
	cout << str.m_size << endl;//0
}
```

