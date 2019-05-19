---
title: C语言学习笔记-const关键字
categories:
  - C/C++
tags:
  - c
abbrlink: 5c7372a9
date: 2019-05-11 15:32:21
---

c语言中 const 关键字使用示例

1. const 修饰的变量定义时要初始化，不初始化后面就没有办法赋值了
2. const 运用在指针上
3. c 语言中的 const 是个冒牌货，使用指针任然能够修改

<!-- more -->

```c
#include <stdio.h>
 
int main()
{
    char buf[] = "dfasfadsgafdg";
    const char *p = buf;
    // p[1] = "2"; err
    p = "0x11";
    // const 此时修饰的是 *p，表示 p 指向的那块内存的内容 *p 是不可变的
 
    // char const *p1 = buf;
    // p1[1] = '2;
    // 交换顺序这里 const char 与 char const 相同
 
    char * const p1 = buf;
    p1[1] = '2';
    // p1 = "0x11"; err
    // const 此时修饰的是 p，表示指针变量 p 本身是不可修改的，而指针变量指向的内容 *p 是可以修改的
 
    const char * const p3 = buf;
    // p3[1] = '2; err
    // p3 = "abc"; err
    // 两个 const 分别修饰了 p 和 *p，此时无论是指针还是指针指向的内存都不可变，完成处于只读状态
 
    const int a = 123;
    // a = 321; err
    int *q = NULL;
    q = &a;
    *q = 321;
    printf("a: %d\n", a);
    // c 语言中的 const 是个冒牌货，使用指针任然能够修改
 
    return 0;
}
```

