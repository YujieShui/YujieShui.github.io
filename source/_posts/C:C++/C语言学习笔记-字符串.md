---
title: C语言学习笔记-字符串
toc: true
categories:
  - C/C++
tags:
  - c
abbrlink: a99472bd
date: 2019-05-11 15:40:14
---

- 字符串初始化
- 访问字符串
- 字符串的拷贝

<!-- more -->

# 字符串初始化

C 语言没有字符串类型，利用字符类型来模拟，以 \0 结尾。

```c
#include <stdlib.h>
#include <string.h>
#include <stdio.h>
int main()
{
    // 如果用字符数组来初始化字符串，末尾不是\0结尾后面的会乱码
    char buf[] = {'a', 'b', 'c'};
    printf("buf = %s\n", buf);
    // 指定数组长度未初始化的地方会补零，读到0就作为字符串结束标记，则不会乱码
    char buf1[100] = {'a', 'b', 'c'};
    printf("buf1 = %s\n", buf1);
    // '字符 0' ≠ 0 = '\0'
    char buf2[100] = {'a', 'b', 'c', '0', '1', '2'};
    printf("buf2 = %s\n", buf2);
    char buf3[100] = {'a', 'b', 'c', 0, '1', '2'};
    printf("buf3 = %s\n", buf3);
    char buf4[100] = {'a', 'b', 'c', '\0', '1', '2'};
    printf("buf4 = %s\n", buf4);
    // 常用初始化字符串的方法
    char str[] = "safafasga";
    // strlen() 和 sizeof() 的区别:
    // 字符串末尾以\0结尾，strlen() 计算字符串长短; sizeof()计算大小还会包含\0
    printf("strlen() = %d, sizeof() = %d\n", strlen(str), sizeof(str));
    char str2[100] = "safafasga";
    // 制定数组大小的情况下，未制定的位置将会自动补零
    printf("strlen() = %d, sizeof() = %d\n", strlen(str2), sizeof(str2));
    return 0;
}
```

# 访问字符串

```c
#include <stdio.h>
#include <string.h>
/**
* 遍历字符串的方法
**/
int main()
{
    char *buf = "fdsafasdgag";
    for(int i = 0; i < strlen(buf); ++i)
    {
        printf("%c", buf[i]);
    }
    printf("\n");
    char *p = buf;
    for(int i = 0; i < strlen(buf); i++)
    {
        printf("%c", p[i]);
    }
    printf("\n");
    for(int i = 0; i < strlen(buf); i++)
    {
        printf("%c", *(p + i));
    }
    printf("\n");
    for(int i = 0; i < strlen(buf); i++)
    {
        printf("%c", *(buf + i));
    }
    printf("\n");
    // p 和 buf 等价么
    // buf 是常量，常量是不可变的
    // p 是变量，可变
    // p++;
    // buf++; // err
    return 0;
}
```

# 字符串的拷贝

```c
#include <stdio.h>
#include <string.h>
/**
* 1. 进行非空判断，避免异常
* 2. 不要直接使用形参
**/
int my_strcpy(char *dst, char *src)
{
    // 非空判断
    if(dst == NULL || src == NULL)
        return -1;
    // 不要直接使用形参
    // 会改变数组的首地址位置
    char *to = dst;
    char *from = src;
    while(*to++ == *from++);
    
    // 上面把形参结果来不起作用，只有这样才行，之后再看一下
    // while(*dst++ == *src++);
    return 0;
}
/**
* 1. 实现字符串拷贝函数
* 2. 写出健壮的代码的注意事项
**/
int main()
{
    char *src = "dfagfadga";
    char dst[100] = {0};
    int res = my_strcpy(dst, src);
    if(res != 0)
    {
        fprintf(stderr,"copy str error.");
        return -1;
    }
    printf("dst: %s\n", dst);
    return 0;
}
```

