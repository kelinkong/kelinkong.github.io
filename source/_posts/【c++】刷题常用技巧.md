---
title: C++刷题常用技巧（自用）
date: 2022-08-30 07:33:02
categories: algorithm
tags: [cpp]
---

### 自带的宏常量

```cpp
int N = INT_MIN;
int N = INT_MAX;
const int INF = 0x3f3f3f3f; //通常用来代替最大值，防止运算过程中溢出
```
### 字符串判断函数

```cpp
isdigit(c)  //判断c字符是不是数字
isalpha(c)  //判断c字符是不是字母
isalnum(c)  //判断c字符是不是数字或者字母
tolower(c)  //转为小写
toupper(c)  //转为大写
```
### 字符串和数值之间的转换

```cpp
int num = 100;
string str = to_string(num); //整形转字符串
int number = stoi(str);  //字符串转为整形 stol()是字符串转为长整形
```
### 迭代器的二分

```cpp
vector<int> nums{1,2,34,44,99};
int k = lower_bound(nums.begin(), nums.end(), 56) - nums.begin(); //第一个大于等于目标值的迭代器位置
int k = upper_bound(nums.begin(), nums.end(), 56) - nums.begin(); // 找到第一个大于目标值的迭代器位置
```
### 字符串转为小写

```cpp
transform(str.begin(), str.end(), str.begin(), ::tolower());
```
### 小根堆和大根堆

```cpp
priority_queue<int> pq; //默认是大根堆
priority_queue<int, vector<int>, greater<int>> pq; //小根堆
//greater比较器
```
### 快速初始化数组

```cpp
// 注意：这个函数是按字节初始化的
memset(nums, 0, sizeof nums);
memset(nums, -1, sizeof nums);
memset(nums, 0x3f, sizeof nums);

string s(10, 'a');
cout<<s<<endl;
```
### c++11的特性

```cpp
auto p = new ListNode(); // auto 关键字
Node* pre = nullptr   // nullptr代替NULL
unordered_map<int,int> mp; //哈希表 内部是无序的
unordered_set<int> st; //无序集合
```

### bitset

```cpp
uint32_t reverseBits(uint32_t n) {
    string s = bitset<32>(n).to_string();
    reverse(s.begin(), s.end());
    return bitset<32>(s).to_ulong();
}
```

### 字符串分割
将字符串按照空格分隔
```cpp
string s = "hello world my name is yao jun";
stringstream ss(s);
string str;
int cnt = 0;
while(ss >> str){
    cnt++;
    cout<<str<<endl;
}
cout<<cnt<<endl;
```
### 字符串按格式拆分

```cpp
string a = "12:59:36";
char str2[100];
memcpy(str2, a.c_str(), strlen(a.c_str()));
int u, v, w;
sscanf(str2, "%d:%d:%d", &u, &v, &w);
cout<<u<<" "<<v<<" "<<w<<endl;
//12 59 36
```

### 四舍五入保留小数

```cpp
char str[10];
double num = 22.23434;
sprintf(str, "%.2f", num);
//将num存储到str中
string s = str;
cout<<s<<endl;
```
### 结构体排序

```cpp
struct node{
    int a, b;
    // 从小到大排序
    bool operator < (const node& node_)const{
        if(a != node_.a) return a < node_.a;
        return b < node_.b;
    }
};
int main(){
    vector<node> tt;
    tt.push_back({1,5});
    tt.push_back({2,3});
    sort(tt.begin(), tt.end());
    for(auto &node: tt){
        cout<<node.a<<" "<<node.b<<endl;
    }
    return 0;
}
```
### 优先队列自定义排序

```cpp
struct node{
    int a, b;
    // 在优先队列中，跟排序的规则是反的，这里是指a大的排在前面，a相同时，b大的排在前面
    bool operator < (const node& node_)const{
        if(a != node_.a) return a < node_.a;
        return b < node_.b;
    }
};
int main(){
    priority_queue<node> pq;
    pq.push({1,5});
    pq.push({2,3});
    pq.push({2,5});
    while(!pq.empty()) {
        cout<<pq.top().a<<" "<<pq.top().b<<endl;
        pq.pop();
    }
    return 0;
}
```

### 二维数组按列排序
**按照第一列排序，如果第一列相等按照第二列排序**
```cpp
vector<vector<int>> v;
sort(v.begin(),v.end(),[](vector<int>&v1,vector<int>&v2){if(v1[0] != v2[0])return v1[0]<v2[0];else return v1[1]<v2[1];});
```

### reverse()函数
要包含头文件`#include<algorithm>`，该函数没有返回值。
如果是二维数组，按照行翻转，比如第一行翻转之后到最后一行。
```cpp
reverse[first,last);
```
### `.`,`->`的区别
实例和指针的区别。
`A.B`则A为对象或者结构体； 点号（`.`）：左边必须为实体。
`A->B`则A为指针，`->`是成员提取，`A->B`是提取`A`中的成员`B`，`A`只能是指向类、结构、联合的指针； 箭头（`->`）：左边必须为指针。

### 排序，同时保留原坐标
`sort(ids.begin(),ids.end(),[&](int i,int j){return nums2[i] < nums2[j];});`
此时ids[0]就是原数组中最小数的坐标
或者是：
```cpp
vector<pair<int,int>> v;
//第一个位置放数值，第二个位置放坐标
sort(v.begin(),v.end());
```
### function 函数的用法
std::function是一个通用的多态函数包装器，可以调用普通函数、Lambda函数、仿函数、bind对象、类的成员函数和指向数据成员的指针，function定义在名为`function.h`头文件中。是一个模板，在创建function实例时，必须指明类型，如：
```cpp
#include <functional>
function<int(int,int)> func//表示接受两个int，返回一个int的可调用函数
function<int(int,int)> func = [&](int a,int b){};//可以等于某个匿名函数
```
使用function函数可以不需要将主函数中的参数传入，直接以引用的方式捕获就可以。
