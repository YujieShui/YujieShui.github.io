---
title: Numpy 小结
categories:
  - Python
tags:
  - python
  - numpy
abbrlink: a404f4e2
date: 2017-12-10 18:41:29
---

NumPy 是 Python 语言的一个扩充程序库。支持高级大量的维度数组与矩阵运算，此外也针对数组运算提供大量的数学函数库，也是学习 python 必学的一个库。

![numpy](<http://image.shuiyujie.com/2019-05-09-23-07-28.png>)

<!-- more -->

# 1. 读取文件

**numpy.genfromtxt() 用于读取 txt 文件**，其中传入的参数依次为：
1. 需要读取的 txt 文件位置，此处文件与程序位于同一目录下
2. 分割的标记
3. 转换类型，如果文件中既有文本类型也有数字类型，就先转成文本类型

**help(numpy.genfromtxt)用于查看帮助文档：**
如果不想看 API 可以启动一个程序用 help 查看指令的详细用法

```python
import numpy

world_alcohol = numpy.genfromtxt("world_alcohol.txt", delimiter=",",dtype=str)
print(type(world_alcohol))
print(world_alcohol)
print(help(numpy.genfromtxt))
```

# 2. 构造 ndarray

## numpy.array()构造 ndarray

**numpy.array()**中传入数组参数，可以是一维的也可以是二维三维的。numpy 会将其转变成 ndarray 的结构。

```python
vector = numpy.array([1,2,3,4])
matrix = numpy.array([[1,2,3],[4,5,6]])
```

**传入的参数必须是同一结构,不是同一结构将发生转换。**

```python
vector = numpy.array([1,2,3,4])

array([1, 2, 3, 4])
```
*均为 int 类型*

```python
vector = numpy.array([1,2,3,4.0])

array([ 1.,  2.,  3.,  4.])
```
*转为浮点数类型*

```python
vector = numpy.array([1,2,'3',4])

array(['1', '2', '3', '4'],dtype='<U21')
```
*转为字符类型*

## 利用 .shape 查看结构

能够了解 array 的结构，debug 时通过查看结构能够更好地了解程序运行的过程。

```python
print(vector.shape)
print(matrix.shape)
(4,)
(2, 3)
```

## 利用 dtype 查看类型

```python
vector = numpy.array([1,2,3,4])
vector.dtype

dtype('int64')
```

## ndim 查看维度

一维

```python
vector = numpy.array([1,2,3,4])
vector.ndim

1
```

二维

```python
matrix = numpy.array([[1,2,3],
                      [4,5,6],
                     [7,8,9]])
matrix.ndim

2
```

## size 查看元素数量

```python
matrix.size
9
```

# 3. 获取与计算

## numpy 能使用切片获取数据

```python
matrix = numpy.array([[1,2,3],
                      [4,5,6],
                     [7,8,9]])
```

## 根据条件获取

numpy 能够依次比较 vector 和元素之间是否相同

```python
vector = numpy.array([5, 10, 15, 20])
vector == 10

array([False,  True, False, False], dtype=bool)
```

根据返回值获取元素

```python
vector = numpy.array([5, 10, 15, 20])
equal_to_ten = (vector == 10)
print(equal_to_ten)
print(vector[equal_to_ten])

[False  True False False]
[10]
```

进行运算之后获取

```python
vector = numpy.array([5, 10, 15, 20])
equal_to_ten_and_five = (vector == 10) & (vector == 5)
```

```python
vector = numpy.array([5, 10, 15, 20])
equal_to_ten_or_five = (vector == 10) | (vector == 5)
```

## 类型转换

将整体类型进行转换

```python
vector = numpy.array([5, 10, 15, 20])
print(vector.dtype)
vector = vector.astype(str)
print(vector.dtype)

int64
<U21
```

## 求和

sum() 能够对 ndarray 进行各种求和操作，比如分别按行按列进行求和

```python
matrix = numpy.array([[1,2,3],
                      [4,5,6],
                     [7,8,9]])
print(matrix.sum())
print(matrix.sum(1))
print(matrix.sum(0))

45
[ 6 15 24]
[12 15 18]
```

*sum(1) 是 sum(axis=1)) 的缩写，1表示按照 x轴方向求和，0表示按照y轴方向求和*

# 4. 常用函数

## reshape

生成从 0-14 的 15 个数字，使用 reshape(3,5) 将其构造成一个三行五列的 array。

```python
import numpy as np
arr = np.arange(15).reshape(3, 5)
arr

array([[ 0,  1,  2,  3,  4],
       [ 5,  6,  7,  8,  9],
       [10, 11, 12, 13, 14]])
```

## zeros

生成指定结构的默认为 0. 的 array

```python
np.zeros ((3,4))

array([[ 0.,  0.,  0.,  0.],
       [ 0.,  0.,  0.,  0.],
       [ 0.,  0.,  0.,  0.]])
```

## ones

生成一个三维的 array,通过 dtype 指定类型

```python
np.ones( (2,3,4), dtype=np.int32 )

array([[[1, 1, 1, 1],
        [1, 1, 1, 1],
        [1, 1, 1, 1]],

       [[1, 1, 1, 1],
        [1, 1, 1, 1],
        [1, 1, 1, 1]]])
```

## range

指定范围和数值间的间隔生成 array，*注意范围包左不包右*

```python
np.arange(0,10,2)

array([0, 2, 4, 6, 8])
```

## random 随机数

生成指定结构的随机数，可以用于生成随机权重

```python
np.random.random((2,3))

array([[ 0.86166627,  0.37756207,  0.94265883],
       [ 0.9768257 ,  0.96915312,  0.33495431]])
```

# 5. ndarray 运算

元素之间依次相减相减

```python
a = np.array([10,20,30,40])
b = np.array(4)

a - b
array([ 6, 16, 26, 36])
```

乘方
```python
a**2
array([ 100,  400,  900, 1600])
```

开根号

```python
np.sqrt(B)

array([[ 1.41421356,  0.        ],
       [ 1.73205081,  2.        ]])
```

e 求方

```python
np.exp(B)

array([[  7.3890561 ,   1.        ],
       [ 20.08553692,  54.59815003]])
```

向下取整

```python
a = np.floor(10*np.random.random((2,2)))
a

array([[ 0.,  0.],
       [ 3.,  6.]])
```

行列变换

```python
a.T

array([[ 0.,  3.],
       [ 0.,  6.]])
```

变换结构

```python
a.resize(1,4)
a

array([[ 0.,  0.,  3.,  6.]])
```

## 6. 矩阵运算

矩阵之间的运算

```python
A = np.array( [[1,1],
               [0,1]] )
B = np.array( [[2,0],
               [3,4]] )
```

对应位置一次相乘

```python
A*B

array([[2, 0],
       [0, 4]])
```

矩阵乘法

```python
print (A.dot(B))
print(np.dot(A,B))

[[5 4]
 [3 4]]
```

横向相加

```python
a = np.floor(10*np.random.random((2,2)))
b = np.floor(10*np.random.random((2,2)))

print(a)
print(b)
print(np.hstack((a,b)))

[[ 2.  3.]
 [ 9.  3.]]
[[ 8.  1.]
 [ 0.  0.]]
[[ 2.  3.  8.  1.]
 [ 9.  3.  0.  0.]]
```

纵向相加

```python
print(np.vstack((a,b)))

[[ 2.  3.]
 [ 9.  3.]
 [ 8.  1.]
 [ 0.  0.]]
```

矩阵分割

```python
#横向分割
print( np.hsplit(a,3))
#纵向风格
print(np.vsplit(a,3))
```

# 7. 复制的区别

## 地址复制

通过 b = a 复制 a 的值，b 与 a 指向同一地址，改变 b 同时也改变 a。

```python
a = np.arange(12)
b = a
print(a is b)

print(a.shape)
print(b.shape)
b.shape = (3,4)
print(a.shape)
print(b.shape)

True
(12,)
(12,)
(3, 4)
(3, 4)
```

## 复制值

通过 a.view() 仅复制值，当对 c 值进行改变会改变 a 的对应的值，而改变 c 的 shape 不改变 a 的 shape

```python
a = np.arange(12)
c = a.view()
print(c is a)

c.shape = 2,6
c[0,0] = 9999

print(a)
print(c)

False
[9999    1    2    3    4    5    6    7    8    9   10   11]
[[9999    1    2    3    4    5]
 [   6    7    8    9   10   11]]
```

## 完整拷贝

a.copy() 进行的完整的拷贝，产生一份完全相同的独立的复制

```python
a = np.arange(12)
c = a.copy()
print(c is a)

c.shape = 2,6
c[0,0] = 9999

print(a)
print(c)

False
[ 0  1  2  3  4  5  6  7  8  9 10 11]
[[9999    1    2    3    4    5]
 [   6    7    8    9   10   11]]
```


