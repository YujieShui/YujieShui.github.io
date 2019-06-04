---
title: OpenCV-Mat表达式
categories:
  - OpenCV
tags:
  - opencv
abbrlink: 1208a55a
date: 2019-06-02 17:49:20
---

利用 C++中的运算符重载，OpenCV 2 中引入了 Mat 运算表达式。这一新特 点使得使用 C++进行编程时，就如同写 Matlab 脚本，代码变得简洁易懂，也便于维护。

下面给出 Mat 表达式所支持的运算。下面的列表中使用 A 和 B 表示 Mat 类 型的对象，使用 s 表示 Scalar 对象，alpha 表示 double 值。

- 加法，减法，取负：`A+B`，`A-B`，`A+s`，`A-s`，`s+A`，`s-A`，`-A` 
- 缩放取值范围：`A*alpha`
- 矩阵对应元素的乘法和除法： `A.mul(B)`，`A/B`，`alpha/A` 
- 矩阵乘法：`A*B` （注意此处是矩阵乘法，而不是矩阵对应元素相乘）
- 矩阵转置：`A.t()`
- 矩阵求逆和求伪逆：`A.inv()`
- 矩阵比较运算：`A cmpop B`，`A cmpop alpha`，`alpha cmpop A`。此处 cmpop 可以是>，>=，==，!=，<=，<。如果条件成立，则结果矩阵（8U 类型矩阵）的对应元素被置为 255；否则置 0。 
- 矩阵位逻辑运算：`A logicop B`，`A logicop s`，`s logicop A`，`~A`，此处 logicop 可以是&，|和^。
- 矩阵对应元素的最大值和最小值：`min(A, B)`，`min(A, alpha)`，`max(A, B)`， `max(A, alpha)`。
- 矩阵中元素的绝对值：`abs(A) `
- 叉积和点积：`A.cross(B)`，`A.dot(B)`