---
title: 如何使用Jupyter notebook
toc: true
categories:
  - Python
tags:
  - python
abbrlink: 80ec4f16
date: 2019-03-22 07:51:30
---

![Jupyter Notebook](http://image.shuiyujie.com/2019-03-22-07-53-40.png)

Notebook 已迅速成为处理数据的必备工具。其已知用途包括[数据清理和探索](http://nbviewer.jupyter.org/github/jmsteinw/Notebooks/blob/master/IndeedJobs.ipynb)、可视化、[机器学习](http://nbviewer.jupyter.org/github/masinoa/machine_learning/blob/master/04_Neural_Networks.ipynb)和[大数据分析](http://nbviewer.jupyter.org/github/tdhopper/rta-pyspark-presentation/blob/master/slides.ipynb)。GitHub 上面也会自动提供 notebook。借助此出色的功能，你可以轻松共享工作。<http://nbviewer.jupyter.org/> 也会提供 GitHub 代码库中的 notebook 或存储在其他地方的 notebook。

<!-- more -->

Notebook 运行的核心是 notebook 服务器。你通过浏览器连接到该服务器，而 notebook 呈现为 Web 应用。你在 Web 应用中编写的代码通过该服务器发送给内核。内核运行代码并将代码发送回该服务器，之后，任何输出都会返回到浏览器中。保存 notebook 时，它作为 JSON 文件（文件扩展名为 `.ipynb`）写入到该服务器中。

# 使用 Jupyter notebook

## 安装 notebook

安装 Jupyter 的最简单方法是使用 Anaconda。该发行版自动附带了 Jupyter notebook。你能够在默认环境下使用 notebook。

要在 conda 环境中安装 Jupyter notebook，请使用 `conda install jupyter notebook`。

也可以通过 pip 使用 `pip install jupyter notebook` 来获得 Jupyter notebook。

查看[如何使用Anaconda](https://shuiyujie.com/post/9d29b615.html)了解如何使用Anaconda。

## 启动 notebook 服务器

要启动 notebook 服务器，请在终端或控制台中输入 `jupyter notebook`。服务器会在你运行此命令的目录中启动。这意味着任何 notebook 文件都会保存在该目录中。你通常希望在 notebook 所在的目录中启动服务器。不过，你可以在文件系统中导航到 notebook 所在的位置。

运行此命令时（请自己试一下！），服务器主页会在浏览器中打开。默认情况下，notebook 服务器的运行地址是 `http://localhost:8888`。如果启动其他服务器，新服务器会尝试使用端口 `8888`，但由于此端口已被占用，因此新服务器会在端口 `8889` 上运行。之后，可以通过 `http://localhost:8889` 连接到新服务器。每台额外的 notebook 服务器都会像这样增大端口号。

![Jupyter启动后的界面](http://image.shuiyujie.com/2019-03-21-07-37-58.png)

在右侧，你可以点击“New”（新建），创建新的 notebook、文本文件、文件夹或终端。“Notebooks”下的列表显示了你已安装的内核。由于我在 Python 3 环境中运行服务器，因此列出了 Python 3 内核。

顶部的选项卡是 *Files*（文件）、*Running*（运行）和 *Cluster*（聚类）。*Files*（文件）显示当前目录中的所有文件和文件夹。点击 *Running*（运行）选项卡会列出所有正在运行的 notebook。可以在该选项卡中管理这些 notebook。

## 关闭 notebook

通过在服务器主页上选中 notebook 旁边的复选框，然后点击“Shutdown”（关闭），你可以关闭各个 notebook。但是，在这样做之前，请确保你保存了工作！否则，在你上次保存后所做的任何更改都会丢失。下次运行 notebook 时，你还需要重新运行代码。

通过在终端中按两次 Ctrl + C，可以关闭整个服务器。再次提醒，这会立即关闭所有运行中的 notebook，因此，请确保你保存了工作！

# Notebook 界面

## Cell

新建的一个 notebook 之后，默认就会有一个 cell，也就是下图中蓝色的小框。

![Notebook 界面](http://image.shuiyujie.com/2019-03-21-07-49-10.png)

Cell 可以称为*单元格*。单元格是你编写和运行代码的地方。

## 工具栏

从左侧开始，工具栏上的其他控件是：

- 落伍的软盘符号，表示“保存”。请记得保存 notebook！
- `+` 按钮用于创建新的单元格
- 然后是用于剪切、复制和粘贴单元格的按钮。
- 运行、停止、重新启动内核
- 单元格类型：代码、Markdown、原始文本和标题
- 命令面板（见下文）
- 单元格工具栏，提供不同的单元格选项（例如将单元格用作幻灯片）

## 命令面板

小键盘符号代表命令面板。点击它会弹出一个带有搜索栏的面板，供你搜索不同的命令。这能切实帮助你加快工作速度，因为你无需使用鼠标翻查各个菜单。你只需打开命令面板，然后键入要执行的操作。

![命令面板](http://image.shuiyujie.com/2019-03-21-07-50-18.png)

# 快捷键

Notebook 提供了许多快捷键，我们可以使用`shift + command + P`呼出命令行面板，在输入`keyboard`就可以查看快捷键列表。

![快捷键列表](http://image.shuiyujie.com/2019-03-21-08-07-42.png)

键入`Esc`进入命令模式，即可使用这些快捷键。

# Magic 关键字

Magic 关键字是可以在单元格中运行的特殊命令，能让你控制 notebook 本身或执行系统调用（例如更改目录）。例如，可以使用 `%matplotlib` 将 matplotlib 设置为以交互方式在 notebook 中工作。

Magic 命令的前面带有一个或两个百分号（`%` 或 `%%`），分别对应行 Magic 命令和单元格 Magic 命令。行 Magic 命令仅应用于编写 Magic 命令时所在的行，而单元格 Magic 命令应用于整个单元格。

**注意：**这些 Magic 关键字是特定于普通 Python 内核的关键字。如果使用其他内核，这些关键字很有可能无效。

## 代码计时

有时候，你可能要花些精力优化代码，让代码运行得更快。在此优化过程中，必须对代码的运行速度进行计时。可以使用 Magic 命令 `timeit` 测算单元格中代买运行时间和函数的运行时间。

测试整个单元格代码运行时间的时候可以添加`%%timeit`

```python
%%timeit
sum = 0
for i in range(100):
    sum += i就会输出
```

```shell
3.97 µs ± 150 ns per loop (mean ± std. dev. of 7 runs, 100000 loops each)
```

测试整个某个函数代码运行时间的时候可以添加`%timeit`

```python
def sum():
    sum = 0
    for i in range(100):
        sum += i
```

```python
%timeit sum()
```

```
4.06 µs ± 180 ns per loop (mean ± std. dev. of 7 runs, 100000 loops each)
```

## 在 notebook 中进行调试

对于 Python 内核，可以使用 Magic 命令 `%pdb` 开启交互式调试器。出错时，你能检查当前命名空间中的变量。

要详细了解 `pdb`，请阅读[此文档](https://docs.python.org/3/library/pdb.html)。要退出调试器，在提示符中输入 `q` 即可。

## 补充读物

Magic 命令还有很多，我只是介绍了你将会用得最多的一些命令。要了解更多信息，请查看[此列表](http://ipython.readthedocs.io/en/stable/interactive/magics.html)，它列出了所有可用的 Magic 命令。

# 转换 notebook

Notebook 只是扩展名为 `.ipynb` 的大型 [JSON](http://www.json.org/) 文件。

```json
{
 "cells": [
  {
   "cell_type": "code",
   "execution_count": 8,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "3.97 µs ± 150 ns per loop (mean ± std. dev. of 7 runs, 100000 loops each)\n"
     ]
    }
   ],
   "source": [
    "%%timeit\n",
    "sum = 0\n",
    "for i in range(100):\n",
    "    sum += i"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 9,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "def sum():\n",
    "    sum = 0\n",
    "    for i in range(100):\n",
    "        sum += i"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 10,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "4.06 µs ± 180 ns per loop (mean ± std. dev. of 7 runs, 100000 loops each)\n"
     ]
    }
   ],
   "source": [
    "%timeit sum()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": []
  }
 ],
 "metadata": {
  "kernelspec": {
   "display_name": "Python 3",
   "language": "python",
   "name": "python3"
  },
  "language_info": {
   "codemirror_mode": {
    "name": "ipython",
    "version": 3
   },
   "file_extension": ".py",
   "mimetype": "text/x-python",
   "name": "python",
   "nbconvert_exporter": "python",
   "pygments_lexer": "ipython3",
   "version": "3.6.5"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 2
}

```

由于 notebook 是 JSON 文件，因此，可以轻松将其转换为其他格式。Jupyter 附带了一个名为 `nbconvert` 的实用程序，可将 notebook 转换为 HTML、Markdown、幻灯片等格式。

例如，要将 notebook 转换为 HTML 文件，请在终端中使用

```bash
jupyter nbconvert --to html notebook.ipynb
```

要将 notebook 与不使用 notebook 的其他人共享，转换为 HTML 很有用。而要在博客和其他接受 Markdown 格式化的文本编辑器中加入 notebook，Markdown 很合适。

像平常一样，要详细了解 `nbconvert`，请阅读相关[文档](https://nbconvert.readthedocs.io/en/latest/usage.html)。


