---
title: C++学习笔记-多态
toc: true
categories:
  - C/C++
tags:
  - c++
abbrlink: e7609a12
date: 2019-05-18 23:03:11
---

多态指不同的对象发送同一个消息，不同对象对应同一消息产生不同行为。多态存在于具有继承关系的类中，当父类函数被申明为虚函数，子类重写了父类虚函数时，就具备了多态发生的条件。

<!-- more -->

# 多态

```c++
/* C++学习笔记：面向对象，多态*/
#include <iostream>

class Animal
{
public:
    void bark()
    {
        std::cout << "动物叫..." << std::endl;
    }
};

class Dog: public Animal
{
public:
    void bark()
    {
        std::cout << "wang wang wang ..." << std::endl;
    }
};

class Cat: public Animal
{
public:
    void bark()
    {
        std::cout << "miao miao miao ..." << std::endl;
    }
};

void animalBark(Animal *animal)
{
    animal->bark();
}

int main()
{
    Animal *animal = new Animal();
    Dog *dog = new Dog();
    Cat *cat = new Cat();

    animalBark(animal);
    animalBark(dog);
    animalBark(cat);

    return 0;
}
```

```
动物叫...
动物叫...
动物叫...
```

父类函数添加 `virtual` ，将其声明为虚函数。

```c++
class Animal
{
public:
    virtual void bark()
    {
        std::cout << "动物叫..." << std::endl;
    }
};
```

```
动物叫...
wang wang wang ...
miao miao miao ...
```

基类中的虚函数是一个使用关键字 **virtual** 声明的函数。派生类中已经对函数进行定义的情况下，定义一个基类的虚函数，就是要告诉编译器我们不想对这个函数进行静态链接。

我们所希望的是根据调用函数的对象的类型对程序中在任何给定指针中被调用的函数的选择。这种操作被称为动态链接，或者后期绑定。

多态的三个条件

1. 类间存在继承
2. 要有虚函数重写
3. 父类指针或引用指向子类对象

# 重载、重写、重定义

重载：同一个类中函数名相同，参数列表相同。以下代码表示 bark() 函数的重载。

```c++
class Animal
{
private:
    std::string name;

public:
    virtual void bark()
    {
        std::cout << "动物叫..." << std::endl;
    }
    
    virtual void bark(std::string name)
    {
        std::cout << name << "正在叫..." << std::endl;
    }
};
```

重写：指子类覆盖父类的虚函数，要求函数名和参数均相同，如下所示：

```c++
class Animal
{
public:
    virtual void bark()
    {
        std::cout << "动物叫..." << std::endl;
    }
};

class Dog: public Animal
{
public:
    void bark()
    {
        std::cout << "wang wang wang ..." << std::endl;
    }
};
```