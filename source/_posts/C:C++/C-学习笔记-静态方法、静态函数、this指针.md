---
title: C++学习笔记-静态方法、静态函数、this指针
categories:
  - C/C++
tags:
  - c++
abbrlink: cd8fc6b5
date: 2019-05-18 09:44:28
---

静态变量、静态方法和 this 指针都是为了更好表现类的概念。

类中的静态变量能够实现同类对象中的信息共享，共享状态能够实现许多实用的功能，比如说计数。

静态变量不能用普通函数直接访问，就需要用到静态函数来管理静态变量。

this 指针用 `this->name = name` 的形式解决类中函数传入的参数与类中成员变量冲突的问题。

总之，通过代码来说明问题更容易理解。

<!-- more -->

静态变量、静态方法和 this 指针使用代码示例：

```c++
/* 面向对象，静态变量、静态方法, this 指针 */

#include <iostream>
#include <cstring>

class Student
{
private:
    // 学生编号
    int number;
    // 学生分数
    int score;

public:
    // 静态变量，学生总数
    static int studentCount;
    // 静态变量，所有学生总分
    static int sumScore;

public:

    Student(int number, int score)
    {
        this->number = number;
        this->score = score;

        studentCount++;
        sumScore += score;
    }

    // 静态方法
    static double getAvg()
    {
        return sumScore / studentCount;
    }

    Student(/* args */);
};

// 静态变量赋值，所有对象共享
int Student::sumScore = 0;
int Student::studentCount = 0;

int main()
{
    Student stu1(111, 50);
    Student stu2(112, 70);
    Student stu3(114, 90);

    // 静态方法调用
    double avg = Student::getAvg();

    std::cout << "average score = " << avg << std::endl;

    return 0;    
}
```

静态成员变量：

1. static 成员变量实现了同类对象间信息共享。
2. static 成员类外存储,求类大小,并不包含在内。
3. static 成员是命名空间属于类的全局变量,存储在 data 区
4. static 成员只能类外初始化。
5. 可以通过类名访问(无对象生成时亦可),也可以通过对象访问

静态成员函数：

1. 静态成员函数的意义,不在于信息共享,数据沟通,而在于管理静态数据成员, 完成对静态数据成员的封装。
2. 静态成员函数只能访问静态数据成员。原因:非静态成员函数,在调用时 this 指针被当作参数传进。而静态成员函数属于类,而不属于对象,没有 this 指针

this 指针:

1. 类成员变量的形参和类的属性名字相同，可以使用 this 指针，如 `this->name = name`