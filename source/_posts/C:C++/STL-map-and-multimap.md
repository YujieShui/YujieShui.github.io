---
title: STL-map and multimap
toc: true
categories:
  - C/C++
tags:
  - stl
  - c++
abbrlink: 8d66854b
date: 2019-06-08 12:15:12
---

map 以键值对的形式存储数据，所有元素根据键值自动排序。pair 的第 一元素被称为键值，第二元素被称为实值。map 也是以红黑树为底层实现机制。

map 和 multimap 区别在于，map 不允许相同 key 值存在，multimap 则允许相同 key 值存在。

<!-- more -->

# 对组

类模板：template <class T1, class T2> struct pair.

```c++
//第一种方法创建一个对组
pair<string, int> pair1(string("name"), 20);
cout << pair1.first << endl; //访问 pair 第一个值
cout << pair1.second << endl;//访问 pair 第二个值

// 第二种
pair<string, int> pair2 = make_pair("name", 30);
cout << pair2.first << endl;
cout << pair2.second << endl;
//pair=赋值
pair<string, int> pair3 = pair2;
cout << pair3.first << endl;
cout << pair3.second << endl;
```

# map 构造函数

```c++
map<T1, T2> mapTT;//map 默认构造函数:
map(const map &mp);//拷贝构造函数
```

# map 赋值操作

```c++
map& operator=(const map &mp);//重载等号操作符
swap(mp);//交换两个集合容器
```

# map 大小操作

```c++
size();//返回容器中元素的数目
empty();//判断容器是否为空
```

# map 插入数据元素操作

```c++
map.insert(...);//往容器插入元素，返回pair<iterator,bool>
map<int, string> mapStu; 
// 第一种通过 pair 的方式插入对象
mapStu.insert(pair<int, string>(3, "小张"));
// 第二种通过 pair 的方式插入对象
mapStu.inset(make_pair(-1, "校长"));
// 第三种通过 value_type 的方式插入对象
mapStu.insert(map<int, string>::value_type(1,"小李"));
// 第四种通过数组的方式插入值 
mapStu[3] = "小刘";
mapStu[5] = "小王";
```

# map 删除操作

```c++
clear();//删除所有元素
erase(pos);//删除 pos 迭代器所指的元素，返回下一个元素的迭代器。
erase(beg,end);//删除区间[beg,end)的所有元素 ，返回下一个元素的迭代器。
erase(keyElem);//删除容器中 key 为 keyElem 的对组。
```

# map 查找操作

```c++
find(key);//查找键 key 是否存在,若存在， 返回该键的元素的迭代器；/若不存在，返回 map.end();
count(keyElem);//返回容器中 key 为 keyElem 的对组个数。对 map 来说，要么是 0，要么是 1。对 multimap 来说，值可能大于 1。
lower_bound(keyElem);//返回第一个 key<=keyElem 元素的迭代器。
upper_bound(keyElem);//返回第一个 key>keyElem 元素的迭代器。
equal_range(keyElem);//返回容器中 key 与 keyElem 相等的上下限的两个迭代器。
```

