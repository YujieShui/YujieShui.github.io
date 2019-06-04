---
title: C/C++ 中的命令行参数
categories:
  - C/C++
tags:
  - c++
  - c
abbrlink: b482364a
date: 2019-05-23 21:27:25
---

C/C++ 中的 main 函数经常带有 argc, argv ,比如 `int main(int argc, char** argv)` 或者 `int main(int argc, char* argv[])`，其中 argc 表示我们从命令行键入的参数，argv[] 即为参数列表。

Java 中的 `public static void main(String argc[])` 和 Python 中的 `sys.argv` 中也都带有命令行参数。

通过命令行参数我们可以就能由 main 函数入口传递参数到程序内部。

<!-- more -->

```c++
#include <stdio.h> 

int main(int argc, char ** argv) 
{
	int i;

	for (i=0; i < argc; i++)
		printf("Argument %d is %s.\n", i, argv[i]);

	return 0; 
}
```

加入我们编译之后运行 `./hello a b c d` 将会输出

```
Argument 0 is ./hello.
Argument 1 is a.
Argument 2 is b.
Argument 3 is c.
Argument 4 is d.
```

由此得出两个结论

1. 参数列表中包含 `./hello`
2. 参数个数要算上 `./hello`

