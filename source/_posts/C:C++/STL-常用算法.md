---
title: STL-常用算法
toc: true
categories:
  - C/C++
tags:
  - stl
  - c++
abbrlink: b974f564
date: 2019-06-08 14:50:13
---

常用遍历、查找、拷贝和替换、算数生成、集合算法。

<!-- more -->

# 遍历算法

```c++
/*
遍历算法,遍历容器元素
@param beg 开始迭代器
@param end 结束迭代器
@param _callback 函数回调或者函数对象
@return 函数对象
*/
for_each(iterator beg, iterator end, _callback);

/*
transform 算法,将指定容器区间元素搬运到另一容器中
注意: transform 不会给目标容器分配内存，所以需要我们提前分配好内存
@param beg1 源容器开始迭代器
@param end1 源容器结束迭代器
@param beg2 目标容器开始迭代器
@param _cakkback 回调函数或者函数对象
@return 返回目标容器迭代器
*/
transform(iterator beg1, iterator end1, iterator beg2, _callbakc)
```



# 查找算法

```c++
/*
find 算法, 查找元素
@param beg 容器开始迭代器
@param end 容器结束迭代器
@param value 查找的元素
@return 返回查找元素的位置
*/
find(iterator beg, iterator end, value)
  
/*
adjacent_find 算法,查找相邻重复元素
@param beg 容器开始迭代器
@param end 容器结束迭代器
@param _callback 回调函数或者谓词(返回 bool 类型的函数对象)
@return 返回相邻元素的第一个位置的迭代器
*/
adjacent_find(iterator beg, iterator end, _callback);

/*
binary_search 算法二分查找法
注意: 在无序序列中不可用
@param beg 容器开始迭代器
@param end 容器结束迭代器
@param value 查找的元素
@return bool 查找返回 true 否则 false
*/
bool binary_search(iterator beg, iterator end, value);

/*
find_if 算法 条件查找
@param beg 容器开始迭代器
@param end 容器结束迭代器
@param callback 回调函数或者谓词(返回 bool 类型的函数对象)
@return bool 查找返回 true 否则 false
*/
find_if(iterator beg, iterator end, _callback);

/*
count 算法 统计元素出现次数
@param beg 容器开始迭代器
@param end 容器结束迭代器
@param value 回调函数或者谓词(返回 bool 类型的函数对象)
@return int 返回元素个数
*/
count(iterator beg, iterator end, value);

/*
count 算法 统计元素出现次数
@param beg 容器开始迭代器
@param end 容器结束迭代器
@param callback 回调函数或者谓词(返回 bool 类型的函数对象)
@return int 返回元素个数
*/
count_if(iterator beg, iterator end, _callback);
```



# 排序算法

```c++
/*
merge 算法 容器元素合并，并存储到另一容器中
@param beg1 容器 1 开始迭代器
@param end1 容器 1 结束迭代器
@param beg2 容器 2 开始迭代器
@param end2 容器 2 结束迭代器
@param dest 目标容器开始迭代器
*/
merge(iterator beg1, iterator end1, iterator beg2, iterator end2, iterator dest)
  
/*
sort 算法 容器元素排序
注意:两个容器必须是有序的
@param beg 容器 1 开始迭代器
@param end 容器 1 结束迭代器
@param _callback 回调函数或者谓词(返回 bool 类型的函数对象)
*/
sort(iterator beg, iterator end, _callback)

/*
sort 算法 对指定范围内的元素随机调整次序
@param beg 容器开始迭代器
@param end 容器结束迭代器
*/
random_shuffle(iterator beg, iterator end)
  
/*
reverse 算法 反转指定范围的元素
@param beg 容器开始迭代器
@param end 容器结束迭代器
*/
reverse(iterator beg, iterator end)
```



# 拷贝和替换算法

```c++
/*
copy 算法 将容器内指定范围的元素拷贝到另一容器中
@param beg 容器开始迭代器
@param end 容器结束迭代器
@param dest 目标容器结束迭代器
*/
copy(iterator beg, iterator end, iterator dest)
  
/*
replace 算法 将容器内指定范围的旧元素修改为新元素
@param beg 容器开始迭代器
@param end 容器结束迭代器
@param oldvalue 旧元素
@param oldvalue 新元素
*/
replace(iterator beg, iterator end, oldvalue, newvalue)
  
/*
replace_if 算法 将容器内指定范围满足条件的元素替换为新元素
@param beg 容器开始迭代器
@param end 容器结束迭代器
@param callback 函数回调或者谓词(返回 Bool 类型的函数对象)
@param oldvalue 新元素
*/
replace_if(iterator beg, iterator end, _callback, newvalue)

/*
swap 算法 互换两个容器的元素
@param c1 容器 1
@param c2 容器 2
*/
swap(container c1, container c2)
```



# 算数生成算法

```c++
/*
accumulate 算法 计算容器元素累计总和
@param beg 容器开始迭代器
@param end 容器结束迭代器
@param value 累加值
*/
accumulate(iterator beg, iterator end, value)

/*
fill 算法 向容器中添加元素
@param beg 容器开始迭代器
@param end 容器结束迭代器
@param value t 填充元素
*/
fill(iterator beg, iterator end, value)
```



# 集合算法

```c++
/*
set_intersection 算法 求两个 set 集合的交集
注意:两个集合必须是有序序列
@param beg1 容器 1 开始迭代器
@param end1 容器 1 结束迭代器
@param beg2 容器 2 开始迭代器
@param end2 容器 2 结束迭代器
@param dest 目标容器开始迭代器
@return 目标容器的最后一个元素的迭代器地址
*/
set_intersection(iterator beg1, iterator end1, iterator beg2, iterator end2, iterator dest)
  
/*
set_union 算法 求两个 set 集合的并集
注意:两个集合必须是有序序列
@param beg1 容器 1 开始迭代器
@param end1 容器 1 结束迭代器
@param beg2 容器 2 开始迭代器
@param end2 容器 2 结束迭代器
@param dest 目标容器开始迭代器
@return 目标容器的最后一个元素的迭代器地址
*/
set_union(iterator beg1, iterator end1, iterator beg2, iterator end2, iterator dest)

/*
set_difference 算法 求两个 set集合的差集
注意:两个集合必须是有序序列
@param beg1 容器 1 开始迭代器
@param end1 容器 1 结束迭代器
@param beg2 容器 2 开始迭代器
@param end2 容器 2 结束迭代器
@param dest 目标容器开始迭代器
@return 目标容器的最后一个元素的迭代器地址
*/
set_difference(iterator beg1, iterator end1, iterator beg2, iterator end2, iterator dest)
```

