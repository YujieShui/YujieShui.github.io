---
title: 快，学会shell
toc: true
categories:
  - Linux
tags:
  - linux
  - shell
abbrlink: 92782b0b
date: 2019-07-21 22:22:05
---

![学习shell编程](http://image.shuiyujie.com/hacker-2300772_1920.jpg)

*题图：https://pixabay.com/illustrations/hacker-cyber-crime-internet-2300772/*

本文分成入门篇和基础篇。基础篇包括变量、字符串处理、数学运算三部分。基础篇包括流控制、函数和函数库三部分。主要是基于例子进行讲解，其中有 4 个复杂一点的脚本，看懂了也就入门了。

<!-- more -->

我们先来聊一聊 shell 和 shell script 的概念。计算机的运行离不开硬件，我们通过操作系统（OS，Operating System）操作硬件，而我们所说的 linux 严格来说是操作系统（OS）的核心部分——内核（Kernel）。我们无法直接操作 kernel，需要借助于 kernel 外的一层壳 shell 才能与 kernel 进行交互。如果把操作系统(OS)看做是一家公司，shell 就是前台，kernel 就是董事会。当我们访问公司的时候，先和前台(shell)打个招呼，前台通知董事会(kernel)，董事会来控制公司(OS)。

俗话说“铁打的营盘流水的兵”，就是公司人来人往，都不会影响公司的运转。对于操作系统也一样，我们可以替换操作系统的前台(shell)，甚至董事会(kernel)。如果你想知道你的系统中用到的是什么 shell 可以访问 /etc/shells 文件。,我的电脑上就有下面几种 shell

```shell
# /etc/shells: valid login shells
/bin/sh
/bin/dash
/bin/bash
/bin/rbash
/bin/zsh
/usr/bin/zsh
```

# 入门篇

```shell
#!/bin/bash
for ((i=0; i<10; i++));
do
    echo ${i}
done
```

直接来看一个例子吧。创建一个名为 shell001.sh 的文件，写上上面几行代码： 

- 第 1 行，指定 shell 的解释器。shell 脚本就和 python 或者 jsp 需要用解释器来解析，第 1 行就是用于指定解释器，也就是之前提到的 /etc/shells 下面列出来的。 
- 第 2 行，循环语句，共循环 10 次，会在后面流控制章节讲解，加上 do 和 done 是 for 循环中的语法规则，for、do、done 是 shell script 中的关键字 
- 第 4 行，打印 i 这个变量 

那么怎么运行这个脚本呢？既然是运行我们既要给它赋予可执行权限`chmod +x shell001.sh`，接着用 `./shell001.sh` 执行这个脚本。脚本运行起来将会在终端输出 0 到 9 这几个数字。 

## 变量

前面没有解释第 4 行`echo ${i}`。echo 是一个简单的 linux 命令，它会将输入从到标准输出(stdout)上，然后在终端中显示出来，这里显示的就是`${i}`这个变量的值。

在 shell 中定义变量的规则如下：

- 变量和等号之间不能有空格
- 变量名称由字母、数字和下划线组成
- 变量名称的第一个字符必须是字母或者下划线
- 变量名称中不允许空格和标点

比如说一个变量为`name="shuiyj"`，那么使用变量就要加上`$`符号，打印这个变量就使用`echo ${name}`。此外变量除了显示地赋值，还可以使用语句给变量赋值:

```shell
# 获取该文件夹下后缀为 jpg 结尾的列表
for image in `ls *.jpg`
```



### 变量匹配

我们会定义和使用一个变量了，接下来我会介绍几个使用的处理变量的方法。

现在一张图片的名字叫做 cat.jpg，我想要获取文件的名称，即 cat。当然这有很多的中方法，这里介绍一种实用的方法——变量匹配。

| 语法                         | 说明                                                 |
| ---------------------------- | ---------------------------------------------------- |
| ${变量名#匹配规则}           | 从变量**开头**进行规则匹配，将符合**最短**的数据删除 |
| ${变量名##匹配规则}          | 从变量**开头**进行规则匹配，将符合**最长**的数据删除 |
| ${变量名%匹配规则}           | 从变量**结尾**进行规则匹配，将符合**最短**的数据删除 |
| ${变量名%%匹配规则}          | 从变量**结尾**进行规则匹配，将符合**最长**的数据删除 |
| ${变量名/旧字符串/新字符串}  | 变量中符合规则的**第一个**旧字符串将会被旧字符串代替 |
| ${变量名//旧字符串/新字符串} | 变量中符合规则的**所有**旧字符串将会被旧字符串代替   |

回到最开始的需求，就可以使用`%`来实现

```shell
cat="cat.jpg"
echo ${cat} # cat.jpg
echo ${cat%.*} # cat
```

首先定义一个变量 cat 并为其赋值，接着用`$`获取 cat 变量的值并打印出来，最后使用变量替换截取字符串。

变量匹配在 shell 中会被高频使用，要记住这些规则。

### 特殊变量的含义

shell 中有一些特殊的变量，它们有很多实用的功能，比如说校验输入的参数，允许追加更多参数，判断上一条命令是否执行成功等。

| 变量 | 含义                                                         |
| ---- | ------------------------------------------------------------ |
| $0   | 当前脚本的文件名                                             |
| $n   | 传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是`$1`，第二个参数是`$2` |
| $#   | 传递给脚本或函数的参数个数。                                 |
| $*   | 传递给脚本或函数的所有参数。                                 |
| $@   | 传递给脚本或函数的所有参数。被双引号`(" ")`包含时，与 `$* `稍有不同，下面将会讲到。 |
| $?   | 上个命令的退出状态，或函数的返回值。                         |
| $$   | 当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID。 |

比如说我们要对 shell 中的输入参数进行校验，可以添加这样一段

```shell
#!/bin/bash
if [ $# != 2 ];then
    echo "Usage: $0 <change ID> <target ID>"
    exit -1
fi
...
```

其中`$#`表示输入的参数数量，我们通过条件判断在程序的输入参数不为 2 的时候将会进行提示，并退出程序。其中`$0`一般是可执行文件的名称。

## 字符串

变量的话题就先讲到这里，接下来讲 shell 中处理字符串的一些注意事项和技巧。无论学习哪一门编程语言，字符串的处理都是一个绕不开的话题，并且在 shell 编程中用的最多的就是字符串。

### 单引号和双引号的区别

字符串可以用单引号，也可以用双引号，还可以不用引号，我们要注意它们之间的区别。

```shell
# 单引号
str='this is a string'

# 双引号
your_name='qinjx'
str="Hello, I know your are \"$your_name\"! \n"
```

单引号

- 单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的
- 单引号字串中不能出现单引号（对单引号使用转义符后也不行）

双引号

- 双引号里可以有变量
- 双引号里可以出现转义字符

想要了解更多它们之间的区别可以看这篇文章：[shell十三问之4：""(双引号)与''(单引号)差在哪？](https://github.com/wzb56/13_questions_of_shell/blob/master/4.%22%22%E4%B8%8E''%E7%9A%84%E5%B7%AE%E5%9C%A8%E5%93%AA.md)。

### 拼接字符串

shell 中的字符串拼接只要直接连在一起就可以。

```shell
first_name="yujie"
last_name="shui"
greeting="hello,${first_name} ${last_name}"

echo ${greeting}
```

### 计算字符串的长度

shell 中有两种方法可以计算字符串长度。

```shell
str="abcde"
echo ${#str} # 5
expr length "${str}" # 5	
```

### 获取子串在字符串中的索引

```shell
expr index ${string} ${substring}
```

### 抽取子串

| 语法                                    | 说明                               |
| --------------------------------------- | ---------------------------------- |
| ${string:position}                      | 从string中的position开始           |
| ${string:position}                      | 从position开始，并指定长度为length |
| ${string:-position}                     | 从右边开始匹配                     |
| ${string:(postion)}                     | 从左边开始匹配                     |
| `expr substr $string $position $length` | 从position开始，匹配长度为length   |

## 命令替换

命令替换指的是 shell执行命令并将命令替换部分替换为执行该命令后的结果（先执行该命令，然后用结果代换到命令行中），共有有两种实现命令替换的方式：

```shell
# 方法一
`command`
# 方法二
$(command)
```

还记得之前讲变量的时候提到变量既可以直接获取，也可以从语句获取么？

```shell
# 获取该文件夹下后缀为 jpg 结尾的列表
for image in `ls *.jpg`
```

可以看到 Image 获取到的是`ls *.jpg`的返回值，也就是一个文件的列表。这里就用到命令替换的符号``即两个反引号。

再比如获取系统的所有用户并输出

```shell 
index=1 
for user in `cat /etc/passwd | cut -d ":" -f 1` 
do 
	echo "This is ${index} user: ${user}" 
	index = $(($index + 1)) 
done 
```

`cat /etc/passwd | cut -d ":" -f 1 `将会截取用户名，由于使用了命令替换，其执行结果会返回给 user 变量，此时的 user 就是一个包含用户名称的列表。

最后再举一个使用`$()`的例子，比如获取系统时间计算今年或明年

```shell 
echo "This is $(date +%Y) year" 
echo "This is $(($(date +%Y) + 1)) year" 
```

## 数学运算

shell 中就两种变量，字符串和数字，数字又要按照整型和浮点型分开进行处理，处理它们的函数是不同的。整型运算需要使用`expr $num1 operator $num2`或者`$(($num1 operator $num2))`，浮点型运算则需要使用`bc`。

任然是通过案例的方式进行说明，假设现在有这样一个需求：提示用户输入一个正整数 num，然后计算 1+2+3+…+num 的值，并且必须对 num 是否为正整数做判断，不符合应该允许再次输入。

```shell
#!/bin/bash
#
while true
do
    read -p "please input a positive number: " num
    expr $num + 1 &> /dev/null
    if [ $? -eq 0 ];then
        if [ `expr $num \> 0` -eq 1 ];then
            for((i=1;i<=$num;i++))
            do
                sum=`expr $sum + $i`
            done    
            echo "1+2+3+....+$num = $sum"
            exit
        fi
    fi
    echo "error,input enlegal"
done
```

这个脚本优点复杂但是不用着急，我们先关注于数学运算`expr $num + 1` 这一部分，其中关于`if`判断的部分会在下一节讲解。

`expr $num + 1` 意思就是做一次整数运算，将 num 和 1 相加。做这个操作的目的是判断 num 是不是一个整数，因为 expr 只能应用在整数运算上，所以执行`expr $num + 1`之后，如果 num 是整数**退出状态**就是正常的` $? = 0`，否则 `$? ≠ 0` ，并且我们并不需要返回结果，可以将结果重定向到`/dev/null`中，即`expr $num + 1 &> /dev/null`。

*注：在特殊变量的含义这一节可以了解`$?`的含义。退出状态指的是命令执行完毕之后像操作系统返回的值，成功则为 0。*

expr 支持整数运算，不支持浮点数运算，要做浮点数运算那就要用到 bc。bc 是 bash 内建的运算器，支持浮点数运算，使用方法如下所示：

```shell
echo "23.3+30" | bc
53.3
echo "scale=4;23.3/3.2" | bc
7.2812
```



# 基础篇

在前面的入门篇，我们了解了变量、字符串和数学运算，接下来我将会介绍 shell 中流程控制的语法规则，以及 shell 中如何使用函数以及函数库。当我们掌握以上这些内容，shell 就可以算是入门了，那么就一起开始吧。

## 流控制

流控制就是用判断语句，循环语句来控制程序执行的逻辑，就从我们在上一节数学运算中的那个脚本讲起吧，它既包含了`if`又包含了`while`循环，是一个很好的例子。

### IF 控制语句

```shell
#!/bin/bash
#
while true
do
    read -p "please input a positive number: " num
    expr $num + 1 &> /dev/null
    if [ $? -eq 0 ];then
        if [ `expr $num \> 0` -eq 1 ];then
            for((i=1;i<=$num;i++))
            do
                sum=`expr $sum + $i`
            done    
            echo "1+2+3+....+$num = $sum"
            exit
        fi
    fi
    echo "error,input enlegal"
done
```

上一节中的脚本中`expr $num + 1 &> /dev/null`是关于数学运算的部分，紧跟着的`if`就是一个控制语句，我们抛开无关部分，开看一下关于`if`的骨架

```shell
expr $num + 1 &> /dev/null

if [ $? -eq 0 ];then
		...
fi
```

这里首选要进一步解释**退出状态**的含义。之前已经说了，退出状态指的是命令（包括脚本和函数）在执行完毕之后，向操作系统返回的值。这个值是一个 0~255 的整数，用来表示命令执行成功还是失败，其中 0 代表命令执行成功。参数`$?`则用于保存这个返回值。

由此可以看出`if`在这里做的就是判断`expr $num + 1 &> /dev/null`是否执行成功。

此外，我们也可以使用`if...elif..else`的形式，如下：

```shell
if condition1
then
	command1
elif condition2
	command2
else
	commandN
fi
```

此外，在实际开发过程中还经常会对文件状态进行判断，比如说判断这是不是一个文件夹、是不是一个文本文件等；或者会对字符串进行判断，比如说字符串是否为空，字符串长度是否符合要求；还会对数值进行比较操作，就像例子中提到到值是不是为 0 等。

#### 判断表达式

```shell
if test     #表达式为真
if test !   #表达式为假
test 表达式1 –a 表达式2     #两个表达式都为真
test 表达式1 –o 表达式2     #两个表达式有一个为真
test 表达式1 ! 表达式2       #条件求反
```

#### 文件表达式

```shell
test File1 –ef File2    #两个文件是否为同一个文件，可用于硬连接。主要判断两个文件是否指向同一个inode。
test File1 –nt File2    #判断文件1是否比文件2新
test File1 –ot File2    #判断文件1比是否文件2旧
test –b file   #文件是否块设备文件
test –c File   #文件并且是字符设备文件
test –d File   #文件并且是目录
test –e File   #文件是否存在 （常用）
test –f File   #文件是否为正规文件 （常用）
test –g File   #文件是否是设置了组id
test –G File   #文件属于的有效组ID
test –h File   #文件是否是一个符号链接（同-L）
test –k File   #文件是否设置了Sticky bit位
test –b File   #文件存在并且是块设备文件
test –L File   #文件是否是一个符号链接（同-h）
test –o File   #文件的属于有效用户ID
test –p File   #文件是一个命名管道
test –r File   #文件是否可读
test –s File   #文件是否是非空白文件
test –t FD     #文件描述符是在一个终端打开的
test –u File   #文件存在并且设置了它的set-user-id位
test –w File   #文件是否存在并可写
test –x File   #文件属否存在并可执行
```

#### 字符串表达式

```shell
test string #string不为空
test –n 字符串    #字符串的长度非零
test –z 字符串    #字符串的长度是否为零
test 字符串1＝字符串2       #字符串是否相等，若相等返回true
test 字符串1＝=字符串2      #字符串是否相等，若相等返回true
test 字符串1!＝字符串2      #字符串是否不等，若不等反悔false
test 字符串1>字符串2				# 在排序时，string1 在 string2 之后
test 字符串1<字符串2				# 在排序时，string1 在 string2 之前
```

#### 整数表达式

```shell
test 整数1 -eq 整数2    #整数相等
test 整数1 -ge 整数2    #整数1大于等于整数2
test 整数1 -gt 整数2    #整数1大于整数2
test 整数1 -le 整数2    #整数1小于等于整数2
test 整数1 -lt 整数2    #整数1小于整数2
test 整数1 -ne 整数2    #整数1不等于整数2
```

以上表达式摘自[test - shell环境中测试条件表达式工具](https://www.jishuchi.com/read/linux-command-collection/1576?wd=%E7%BC%96%E7%A8%8B)，test 是测试条件表达式的工具，test 后面部分的内容可以用于`if`条件判断中。

### WHILE 和 UNTIL 循环

再接之前的脚本来讲解 while 的用法

```shell
#!/bin/bash
#
while true
do
    read -p "please input a positive number: " num
    expr $num + 1 &> /dev/null
    if [ $? -eq 0 ];then
        if [ `expr $num \> 0` -eq 1 ];then
            for((i=1;i<=$num;i++))
            do
                sum=`expr $sum + $i`
            done    
            echo "1+2+3+....+$num = $sum"
            exit
        fi
    fi
    echo "error,input enlegal"
done
```

前面也说过，这个脚本的目的是接收一个 num，如果输入的 num 不是一个整数就一直让用户输入，直到输入的 num 是一个整数为止。这里就用到了 while 循环，并且将循环条件设置为 true，也就是一个永久的循环。循环不能终止，按照逻辑我们要在用户输入整数并完成计算之后推出程序,所以这里通过`exit`来退出程序，这里也可以使用`break`来跳出循环。我们还可以配合使用`continue`表示继续执行，这里没有举例说明。

while命令退出状态不为0时终止循环，而until命令则刚好相反。除此之外，until命令与while命令很相似。until循环会在接收到为0的退出状态时终止。在while-count脚本中，循环会一直重复到count变量小于等于5。使用until改写脚本也可以达到相同的效果。

```shell
#!/bin/bash
# until-count: display a series of numbers

count=1

until [ $count -gt 5 ]; do
　　　　　echo $count
　　　　　count=$((count + 1))
done
echo "Finished."
```

将测试表达式改写为`count –gt 5 until`就可以在合适的时刻终止循环。选择使用while还是until，通常取决于哪种循环能够允许程序员写出最明了的测试表达式。

### case 分支

前面讲了使用`if`做条件判断，在其他语言比如 C++ 或者 Java 等中都存在`switch..case..`这样的语句，shell 也提供了`case`这个多选项符合命令，它的命令格式是这样的：

```shell
case word in
　　　　[pattern [| pattern]...) commands ;;]...
esac
```

在这里我想举一个做算数运算的例子，这和例子与下一节讲解的函数相关，其中用到了`case`，但是即使不了解函数怎么使用，也不会对理解`case`的运用造成影响。

```shell
#!/bin/bash
#
function calcu
{
    case $2 in
        +)
            echo "`expr $1 + $3`"
            ;;
        -)
            echo "`expr $1 - $3`"
            ;;
        \*)
            echo "`expr $1 \* $3`"
            ;;
        /)
            echo "`expr $1 / $3`"
            ;;
    esac
}
calcu $1 $2 $3
```

这个脚本希望做的是一次算数运算，根据操作符是`+ - * /`来进行运算。

### for 循环

现在到了流控制的最后一节了，for 循环其实在文章一开始我们就见过了，在文章最开始我举了两个例子：

```shell
# 获取该文件夹下后缀为 jpg 结尾的列表
for image in `ls *.jpg`
do
	....
done

# 输出 0-9 共 10 个数字
for ((i=0; i<10; i++));
do
    echo ${i}
done
```

第一种是传统的形式，和 python 中的 for 循环很像，我们可以像这样`for i in A B C D;do echo $i; done `使用 for 循环，可以将循环的内容就当成 python 中的一个列表，也可以像`for i in {A..E};do echo $i; done`这样创建字符列表。

第二种方式就是 C 语言的形式了`for ((i=0; i<10; i++))`，比较常规也没什么值得讲的。

## 函数

在上一节中，我们介绍了 shell 中的流控制的语法：if, while, until, case和 for。再之前我们讲了变量、字符串和数学运算。到这一节就可以讲一下函数这个话题了。

了解了上面这些内容理论上已经能够写出任何的程序的，不过写程序的过程中会有许多类似的代码，如果全部重新写一遍程序就会显得冗长，所以一般的做法就是将可以复用的代码抽取出来形成函数。shell 自然也支持函数的使用，接下里就看看在 shell 中怎么定义和使用函数。

首先看 shell 中函数是怎么定义的。shell 中的函数有两种定义格式，使用任意一种都可以。

```shell
name()
{
	command1
  command2
  ...
  commandn
}
```

```
function name
{
	command1
  command2
  ...
  commandn
}
```

我比较习惯用第二种形式，你可以选择任何一种方式，不过我接下来的例子是按照第二种定义方式。

我们知道了函数定义的架构，但是如果你用过其他编程语言会发现它没有参数列表，也没有返回值，这在一开始也让我觉得很困惑，但是我们在变量那一节学过特殊变量的含义，其中有一个变量是`$0`表示函数的名称，shell 中的变量是通过命令行键入，再用`$1 $2 $3`读取的。这就和 C++ 或者 Java 中的主函数读取参数一个道理，`char** argv`和`String[] args`就是由命令行键入的参数列表。

下面就用一个具体函数的例子进行说明，这个例子在 流控制——case 这一节也讲过，但是没有讲完：

```shell
#!/bin/bash
#
function calcu
{
    case $2 in
        +)
            echo "`expr $1 + $3`"
            ;;
        -)
            echo "`expr $1 - $3`"
            ;;
        \*)
            echo "`expr $1 \* $3`"
            ;;
        /)
            echo "`expr $1 / $3`"
            ;;
    esac
}

calcu $1 $2 $3
echo ""calcu $1 $2 $3""
```

首先这个脚本的文件名为 calcu，其中定义了一个函数 calcu，采用的是第二种函数定义方式，在函数体中我们利用`case`做了一个多条件判断。

在脚本的结尾我们调用了 calcu 这个函数，并将输入了三个参数，紧接着为了给大家看到三个参数分别是什么我选择将其打印出来。调用函数的过程是这样的：

```shell
ubuntu@VM-0-2-ubuntu:/tmp$ ./calcu.sh 5 + 3
8
calcu 5 + 3
```

可以看到在这里`$1 = 5, $2 = +, $3 = 3`，这就是 shell 中传递参数的方式。

再看 shell 中返回值这个问题，shell 有两种返回值的方式，一种是使用`return`，一种是使用`echo`。

- 使用`return`
  - 使用 return 返回值，只能返回 1-255 的整数
  - 函数使用 return 返回值。通常只能用来供其他地方调用获取状态，因此通常仅返回0或1；0表示成功，1表示失败
- 使用`echo`
  - 使用 echo 可以返回任何字符串结果
  - 通常用于返回数据，比如一个字符串值或者列表值

`echo`的内容作为返回值可以看一下上面这个例子，这里再举一个`return`来返回值的例子。

```shell
#!/bin/bash
#
this_pid=$$
function is_nginx_running
{
    ps -ef | grep nginx | grep -v grep | grep -v $this_pid &> /dev/null
    if [ $? -eq 0 ];then
        return
    else
        return 1
    fi
}
is_nginx_running && echo "Nginx is running" || echo "Nginx is stoped"
```

和前面说过的退出状态一样，return 也是 0 表示函数执行成功，其他表示执行失败。这里举的例子是通过查看 Nginx 的进程来来确认 Nginx 是否运行。

首先是用`$$`来接收这个 shell 脚本的 Pid，因为脚本名字中带有 Nginx 就需要将其利用 Pid 过滤掉。还记得之前讲的的退出状态么，如果 ps 命令找到了 nginx 进程退出状态就会是 0，表示成功。最后就通过`return`来返回 nginx 是否正常运行。

之后可以用后台挂起的形式运行这个脚本，将其作为 nginx 的守护进程：`nohup sh nginx.sh & `，使用`tail -f nohup.out `来查看监听结果。

## 局部变量和全局变量

讲到函数就还有一个作用域的问题需要讨论，shell 中的局部变量、全局变量和一般编程语言没什么区别。

```shell
#!/bin/bash
#
var1="Hello world"
function test
{
    local var2=87
}
test
echo $var1
echo $var2
```

这里给到一个脚本自行体会一下即可。

## 函数库

基础篇的最后一部分就用来介绍一下 shell 中的函数库。函数库是每一门编程语言中非常重要的一部分，比如说 C++ 中的标准库，Java 中的 JDK，正式这些优秀的库存在才让编程变得更加高效。

shell 也是可以封装自己的库的，比如我现在下一个你要加加减乘除封装成一个函数库，作为之前 calcu 脚本的升级版，该函数库实现以下几个函数：

1. 加法函数 add 
2. 减法函数 reduce 
3. 乘法函数 multiple 
4. 除法函数 divide 
5. 打印系统运行情况的函数 sys_load，该函数显示内存运行情况， 

```shell
function add
{
    echo "`expr $1 + $2`"
}
function reduce
{
    echo "`expr $1 - $2`"
}
function multiple
{
    echo "`expr $1 \* $2`"
}
function divide
{
    echo "`expr $1 / $2`"
}

function sys_load
{
    echo "Memory Info"
    echo
    free -m
    echo
    
    echo "Disk Usage"
    echo
    df -h
    echo
}
```

我们将上面文件封装成一个函数库 lib/base_function。

- 经常使用的重复代码封装成函数文件 
- 一般不直接执行，而是由其他脚本调用 

接着用一个 shell 脚本 calculate.sh 调用函数库中的函数

```shell
#!/bin/bash
#
. /root/lesson/3.5/lib/base_function
add 12 23
reduce 90 30
multiple 12 12
divide 12 2
```

编写函数库文件的建议 

- 库文件名的后缀是任意的，但一般使用 .lib 
- 库文件通常没有可执行选项 
- 库文件无序和脚本在同级目录，只需在脚本中引用时指定 
- 第一行一般使用 #/bin/echo, 输出警告信息，避免用户执行 

# 参考资料

[《Linux 命令行大全》](https://book.douban.com/subject/22226727/)

[Shell脚本编程30分钟入门](https://github.com/qinjx/30min_guides/blob/master/shell.md)

[Awesome Shell](https://github.com/alebcay/awesome-shell)

