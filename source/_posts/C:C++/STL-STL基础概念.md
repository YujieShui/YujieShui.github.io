---
title: STL-STL基础概念
categories:
  - C/C++
tags:
  - stl
  - c++
abbrlink: 543335f8
date: 2019-06-07 23:15:18
---

STL(Standard Template Library,标准模板库)，是惠普实验室开发的一系列软件的统 称。现在主要出现在 c++ 中，但是在引入 c++ 之前该技术已经存在很长时间了。

STL 从广义上分为: **容器(container) 算法(algorithm) 迭代器(iterator)**,容器和算法之 间通过迭代器进行无缝连接。STL 几乎所有的代码都采用了模板类或者模板函数，这相比传 统的由函数和类组成的库来说提供了更好的代码重用机会。

<!-- more -->

STL(Standard Template Library)标准模板库,在我们 c++ 标准程序库中隶属于 STL 的 占到了 80%以上。

在 c++标准中，STL 被组织成以下 13 个头文件：

```
<algorithm>、<deque>、<functional>、<iterator>、<vector>、<list>、<map>、 <memory>、<numeric>、<queue>、<set>、<stack> 、<utility>
```

- 容器
  - 容器即数据结构，比如说 vector 是动态数组，stack 是栈。
  - 容器可以嵌套容器
  - 容器可以分为序列化容器和关联式容器。序列化容器即按照元素进入容器的先后顺序来排列；关联式容器即按照容器定义的其他规则来存放元素。
- 迭代器：迭代器是为了遍历容器中的元素，可以理解是指针
- 算法：STL 为我们提供的算法，算法即用有限的步骤解决问题

```c++
#include <iostream>
#include <vector>
#include <algorithm>

//----------------------------------
// stl 包括三部分：容器、迭代器、算法
//----------------------------------

using namespace std;

int main()
{
    vector<int> v; // 容器，stl 的标准容器之一，动态数组
    v.push_back(1); // Vector 容器提供的插入数据的方法
    v.push_back(3);
    v.push_back(6);
    v.push_back(4);
    v.push_back(3);

    vector<int>::iterator pBegin = v.begin(); // 迭代器，获取指向第一个元素的指针
    vector<int>::iterator pEnd = v.end(); // 迭代器，获取指向最后一个元素的指针

    int n = count(pBegin, pEnd, 3); // 算法，统计 3 出现的次数
    cout << "n = " << n << endl;

    // 使用迭代器遍历 vector
    while(pBegin != pEnd)
    {
        cout << *pBegin << endl;
        pBegin++;
    }

    return 0;
}
```

接下来一系列的文章将介绍 STL 中的常用容器和常用算法。