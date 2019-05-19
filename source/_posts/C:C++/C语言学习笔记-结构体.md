---
title: C语言学习笔记-结构体
toc: true
categories:
  - C/C++
tags:
  - c
  - c++
abbrlink: e261fd69
date: 2019-05-11 09:40:29
---

- 结构体的类型和定义

- 结构体的赋值

- 结构体的数组

- 结构体嵌套一级指针
- 结构体嵌套二级指针
- 结构体作为函数参数
- 结构体深拷贝、浅拷贝

<!-- more -->

# 结构体的类型和定义

结构体是一种数据类型，用 struct 关键字来修饰，定义一个结构体可以这样：

```c
struct Teacher
{
    char name[50];
    int age;
}
```

如果用 typedef 修饰，就可以直接使用 Teacher 

```c
typedef struct Teacher
{
    char name[50];
    int age;
}

Teacher teacher = NULL;
```

为结构体申明变量有多种方式：

```c
// 初始化结构体变量1: 定义类型的同时定义变量
struct Teacher
{
    char name[50];
    int age;
}t1, t2;

// 初始化结构体变量2
struct Student
{
    char name[50];
    int age;
}s1 = {"Mike", 15}; 

// 初始化结构体变量3
struct
{
    char name[50];
    int age;
}dog = {"Luck", 3}; 

// 初始化结构体变量4
struct Teacher t3 = {"Mary", 21};

// 初始化结构体变量5
struct Teacher t4;
struct Teacher *pTeacher = NULL;
pTeacher = &t4;

strcpy(pTeacher->name, "John");
pTeacher->age = 30;
```



# 结构体的赋值

结构体赋值之前要先给结构体申请内存。

```c
/* 结构体赋值 */

#include <stdio.h>
#include <string.h>

struct Teacher
{
    char name[50];
    int age;
};

typedef struct Teacher teacher_t;

void show_teacher(teacher_t t);
void copyTeacher(teacher_t *to, teacher_t *from);

int main()
{
    teacher_t t1, t2, t3;

    // 结构体是一种数据类型，分配空间之后才能赋值
    memset(&t1, 0, sizeof(t1));
    strcpy(t1.name, "teacher");
    t1.age = 30;

    show_teacher(t1);

    // 直接赋值
    t2 = t1;
    show_teacher(t2);

    // 作为参数传递到用于拷贝的函数
    copyTeacher(&t3, &t1);
    show_teacher(t3);

    return 0;
}

void copyTeacher(teacher_t *to, teacher_t *from)
{
    *to = *from;
}

void show_teacher(teacher_t t)
{
    printf("teacher name = %s\n", t.name);
    printf("teacher age = %d\n", t.age);
}
```



# 结构体的数组

指针数组可以分成静态结构体数组和动态结构体数组两种。

```c
// 静态结构体数组
Teacher t1[3] = {"a", 18, "a", 28, "a", 38};
for(int i = 0; i < 3; i++)
{
	printf("%s, %d\n", t1[i].name, t1[i].age);
}

// 动态结构体数组
Teacher *p = NULL;
p = (Teacher *)malloc(3 * sizeof(Teacher));

char buf[50];
for(int i = 0; i < 3; i++)
{
	sprintf(buf, "name%d", i);
	strcpy(p[i].name, buf);
	p[i].age = 20 + i;
}
```

[结构体数组](https://github.com/YujieShui/up-up-c/blob/master/struct/struct03.c)

# 结构体嵌套指针

我们在定义结构体的时候，其中有一个属性使用了一级指针：

```c
struct Teacher
{
    char *name;
    int age;
};
```

如上所示`char *name`是一个指针类型的变量，指针类型的变量当我们为其赋值时要分配内存空间，释放结构体的时候也要先释放其中指针内存的变量。

[示例代码，结构体嵌套一级指针](https://github.com/YujieShui/up-up-c/blob/master/struct/struct04.c)

[示例代码，结构体嵌套二级指针](https://github.com/YujieShui/up-up-c/blob/master/struct/struct06.c)

#  结构体深拷贝、浅拷贝

当结构体中嵌套了指针，采用浅拷贝的方式如下

```c
Student stu;
stu.name = (char *)malloc(30);
strcpy(stu.name, "aaaaaa");

// 浅拷贝，stu 和 sut2 的name 指针指向同一块内存
Student stu2;
stu2 = stu;
printf("name = %s\n", stu2.name);
```

此时 stu.name 和 stu2.name 指向同一块内存空间，当释放其中一个指针的时候，另一个 name 的值也就消失了。

深拷贝的方式指的就是 stu2.name 指向一块新的内存空间，这块内存空间拷贝 stu.name 指向的空间的内容，就像下面这样。

```c
// 深拷贝，stu3 重新申请一块内存
Student stu3;
stu3.name = (char *)malloc(30);
strcpy(stu3.name, stu.name);
printf("name = %s\n", stu3.name);
```

[示例代码，结构体的深拷贝和浅拷贝](https://github.com/YujieShui/up-up-c/blob/master/struct/struct07.c)