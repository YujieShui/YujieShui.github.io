---
title: STL-string 容器
toc: true
categories:
  - C/C++
tags:
  - stl
  - c++
abbrlink: 2d250e1c
date: 2019-06-07 23:42:37
---

string 是对 char * 的封装，具有如下特性：

- char* 是一个指针，string 是一个类。string 实现了对 char* 的封装，是 char*  的容器
- string 封装了许多字符串操作的方法，比如 find、copy、delete、replace、insert 等
- 不用考虑内存释放和越界，内存由 string 类负责维护

<!-- more -->

# string 与 char* 互相转换

```c++
// string 转 char*
string str = "www.shuiyujie.com";
const char* cstr = str.c_str();

// char* 装 string
char * s = "www.shuiyujie.com";
string sstr(s);
```



# string 的构造函数

```c++
string();//创建一个空的字符串,例如: string str;
string(const string& str);//使用一个string对象初始化另一个string对象
string(const char* s);//使用字符串 s 初始化 
string(int n, charc);//使用 n 个字符 c 初始化

//例子:
//默认构造函数
string s1;
//拷贝构造函数
string s2(s1);
string s2 = s1;
//带参数构造函数
char* str = "www.shuiyujie.com";
string s3(str);
string s4(10, 'a');
```



# string 基本赋值操作

```c++
string& operator=(const char* s);//char*类型字符串赋值给当前的字符串
string& operator=(const string &s);//把字符串 s 赋给当前的字符串
string& operator=(char c);//字符赋值给当前的字符串
string& assign(const char *s);//把字符串 s 赋给当前的字符串
string& assign(const char *s, int n);//把字符串 s 的前 n 个字符赋给当前的字符串
string& assign(const string &s);//把字符串 s 赋给当前字符串
string& assign(int n, char c);//用 n 个字符 c 赋给当前字符串
string& assign(const string &s, int start, int n);//将 s 从 start 开始 n 个字符赋值给字符串
```



# string 取字符串操作

```c++
char& operator[](int n);//通过[]方式取字符
char& at(int n);//通过 at 方法获取字符

//例子:
string s = "www.shuiyujie.com";
char c = s[];
c = s.at(1);
```

Q: string 中存取字符[]和 at 的异同?

A: 

1. 相同,[]和 at 都可以返回第 n 个字符
2. 不同，at 访问越界会抛出异常，[]越界会直接程序会挂掉。



# string 拼接操作

```C++
string& operator+=(const string& str);//重载+=操作符
string& operator+=(const char* str);//重载+=操作符
string& operator+=(const char c);//重载+=操作符
string& append(const char *s);//把字符串 s 连接到当前字符串结尾
string& append(const char *s, int n);//把字符串 s 的前 n 个字符连接到当前字符串结尾
string& append(const string &s);//同 operator+=()
string& append(const string &s, int pos, int n);//把字符串 s 中从 pos 开始的 n 个字符连接到当前字符串结尾
string& append(int n, char c);//在当前字符串结尾添加 n 个字符 c
```



# string 查找和替换

```c++
int find(const string& str, int pos = 0) const; //查找 str 第一次出现位置,从 pos 开始查找
int find(const char* s, int pos = 0) const; //查找 s 第一次出现位置,从 pos 开始查找
int find(const char* s, int pos, int n) const; //从 pos 位置查找 s 的前 n 个字符第一次位置
int find(const char c, int pos = 0) const; //查找字符 c 第一次出现位置
int rfind(const string& str, int pos = npos) const;//查找 str 最后一次位置,从 pos 开始查找
int rfind(const char* s, int pos = npos) const;//查找 s 最后一次出现位置,从 pos 开始查找
int rfind(const char* s, int pos, int n) const;//从 pos 查找 s 的前 n 个字符最后一次位置
int rfind(const char c, int pos = 0) const; //查找字符 c 最后一次出现位置
string& replace(int pos, int n, const string& str); //替换从 pos 开始 n 个字符为字符串 str
string& replace(int pos, int n, const char* s); //替换从 pos 开始的 n 个字符为字符串 s
```



# string 比较操作

```c++
/*
compare 函数在>时返回 1， <时返回 -1，==时返回 0。
比较区分大小写，比较时参考字典顺序，排越前面的越小。
大写的 A 比小写的 a 小。
*/

int compare(const string &s) const;//与字符串 s 比较
int compare(const char *s) const;//与字符串 s 比较
```



# string 子串

```c++
string substr(int pos = 0, int n = npos) const;//返回由 pos 开始的 n 个字符组成的字符串
```



# string 插入和删除操作

```c++
string& insert(int pos, const char* s); //插入字符串 
string& insert(int pos, const string& str); //插入字符串
string& insert(int pos, int n, char c);//在指定位置插入 n 个字符 c
string& erase(int pos, int n = npos);//删除从 Pos 开始的 n 个字符
```



