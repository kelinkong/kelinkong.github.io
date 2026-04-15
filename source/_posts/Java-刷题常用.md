---
title: Java-刷题常用
date: 2026-04-15 09:05:07
categories: [Java]
---

## 获取size

普通类型获取size：

```java
int[] arr = new int[10]; // jvm原生对象，数组的长度是固定的，是属性
int size = arr.length; // 获取数组的长度
// 判空
arr == null || arr.length == 0;
```

对于集合类型获取size：

```java
List<String> list = new ArrayList<>();
int size = list.size(); // 获取集合的大小
// 判空
list == null || list.isEmpty();
```

字符串获取size，字符串并不是一个集合，是一个不可变的对象：

```javaString str = "Hello, World!";
int size = str.length(); // 获取字符串的长度
// 判空
str == null || str.isEmpty();
```

## 排序

Java的对象排序方法：
```java
Arrays.sort(array); // 对数组进行排序
Collections.sort(list); // 对列表进行排序

// Java 8 及以上版本，可以使用 lambda 表达式进行排序
list.sort((o1, o2) -> o1.getField() - o2.getField()); // 根据某个字段进行排序
list.sort(Comparator.comparingInt(Object::getField)); // 使用Comparator进行排序
```
如何去理解`list.sort((a, b) -> ??? ); `

表达式返回的值：

|返回值|含义|
|---|---|
|负数|a应该排在b前面|
|0|a和b的顺序不变|
|正数|a应该排在b后面|

举例：
```java
list.sort((a, b) -> (b + a).compareTo(a + b));
```
`compareTo`方法返回一个整数，表示两个字符串的字典顺序比较结果。它的返回值有以下含义：
- 返回负数：如果调用`compareTo`方法的字符串（即`b + a`）在字典顺序上小于参数字符串（即`a + b`)，则返回一个负数。
- 返回0：如果调用`compareTo`方法的字符串（即`b + a`）在字典顺序上等于参数字符串（即`a + b`)，则返回0。
- 返回正数：如果调用`compareTo`方法的字符串（即`b + a`）在字典顺序上大于参数字符串（即`a + b`)，则返回一个正数。

### 字符串内部排序
```java 
char[] arr = s.toCharArray(); // 将字符串转换为字符数组
Arrays.sort(arr);
```
### 最大数问题
[最大数问题](https://leetcode.cn/problems/largest-number/description/)

```java
public String largestNumber(int[] nums) {
    StringBuilder res = new StringBuilder();
    List<String> list = new ArrayList<String>();
    for (int num : nums) {
        list.add(String.valueOf(num));
    }
    list.sort((a, b) -> (b + a).compareTo(a + b));
    for (String str : list) {
        res.append(str);
    }
    String ans = res.toString().replaceFirst("^0+", ""); // 去除前导0
    return ans.isEmpty() ? "0" : ans;
}
```
## 类型转化

数字转字符串：

```java
int num = 123;
String str1 = String.valueOf(num); // 使用 String.valueOf() 方法
String str2 = Integer.toString(num); // 使用 Integer.toString() 方法
String str3 = num + ""; // 使用字符串连接的方式
```

字符串转数字：

```java
String str = "123";
int num1 = Integer.parseInt(str); // 使用 Integer.parseInt
```

String 和 char[] 之间的转化：

```java
String str = "Hello";
char[] charArray = str.toCharArray(); // String 转 char[]
String str2 = new String(charArray); // char[] 转 String
```


## 去除前导0的办法
```java
String ans = res.toString().replaceFirst("^0+", "");

while (ans.length() > 1 && ans.charAt(0) == '0') {
    ans = ans.substring(1);
}

// Stringbuilder的本质是：char[] value
while (res.length() > 1 && res.charAt(0) == '0') {
    res.deleteCharAt(0); // 这里每次删除的都是第一个字符，不存在索引失效的问题
}
```
