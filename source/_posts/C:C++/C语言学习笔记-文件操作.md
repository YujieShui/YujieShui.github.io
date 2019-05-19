---
title: C语言学习笔记-文件操作
toc: true
categories:
  - C/C++
tags:
  - c
abbrlink: fe52dfa7
date: 2019-05-11 09:11:29
---

C语言中我们使用一个指针变量指向一个文件，这个文件就是文件指针。这个文件指针就是 FILE 结构体，它被包含在头文件 "stdio.h" 中。拿到文件指针再结合文件操作的 API，我们就可以对文件进行读写操作。

<!-- more -->

# 文件操作

## 打开文件 fopen()

```
#include <stdio.h>
FILE *fopen(const char *path, const char *mode);
```

文件的打开操作表示返回一个指向制定文件的 **FILE 结构体**。我们需要指定文件位置和操作方式，特别需要注意的是区分不同的文件操作方式将会产生的结果。

```
r      Open text file for reading.  The stream  is  positioned  at  the beginning of the file.

r+     Open  for  reading and writing.  The stream is positioned at the beginning of the file.

w      Truncate file to zero length or create text  file  for  writing. The stream is positioned at the beginning of the file.

w+     Open  for  reading  and writing.  The file is created if it does not exist, otherwise it is truncated.  The stream is  positioned at the beginning of the file.

a      Open  for  appending (writing at end of file).  The file is created if it does not exist.  The stream is positioned at the  end of the file.

a+     Open  for  reading  and appending (writing at end of file).  The file is created if it does not exist.  The initial file position for  reading  is  at  the  beginning  of the file, but output is always appended to the end of the file.
```

## 关闭文件 fclose()

```
#include <stdio.h>
int fclose(FILE *stream);
```

文件操作完成后,必须要用fclose()函数进行关闭,这是因为对打开的文件进行写入时,若文件缓冲区的空间未被写入的内容填满,这些内容不会写到打开的文件中去而丢失。只有对打开的文件进行关闭操作时,停留在文件缓冲区的内容才能写到该文件中去,从而使文件完整。再者一旦关闭了文件,该文件对应的FILE结构将被释放,从而使关闭的文件得到保护,因为这时对该文件的存取操作将不会进行。文件的关闭也意味着释放了该文件的缓冲区。

它表示该函数将关闭FILE指针对应的文件,并返回一个整数值。若成功地关闭了文件,则返回一个0值,否则返回一个非0值。

## fgetc() 和 fputc() 按字符读写文件

首先用 fopen() 读入一个文件`FILE *fp = **fopen**("../data/file01.txt", "r+");` 指定文件位置以及读写格式，r+表示打开文件进行读写操作。

写操作使用 fputc() 逐个字符写入：

```c
char text[] = "This is a text for test.";
int length = strlen(text);

for(int i = 0; i < length; i++)
{
	fputc(text[i], fp);
}
```

读操作使用 fgetc() 逐个字符读取，当读到结束标志位 EOF 时停止

```
// 方法一
while( (ch = fgetc(fp)) != EOF )
{
	printf("%c", ch);
}

// 方法二
while(!feof(fp))
{
	ch = fgetc(fp);
	printf("%c", ch);
}
```

[逐个字符读写文件，示例代码](https://github.com/YujieShui/up-up-c/blob/master/file_operation/file01.c)

## fgets() 和 fputs() 按行读写文件

```c
/* 按行读写 */
#include <stdio.h>
#include <stdlib.h>

int main()
{

    FILE *fp = fopen("../data/file02.txt", "r+");

    if(fp == NULL)
    {
        perror("fopen(): file is NULL.");
    }

    char tmp[100] = "aaaaaaaaa";
    fputs(tmp, fp);

    char buf[100];
    while(!feof(fp))
    {
        char *p = fgets(buf, sizeof(buf), fp);
        if(p != NULL)
        {
            printf("buf = %s", buf);
            printf("p = %s", p);
        }
    }

    return 0;
}
```

## fwrite() 和 fread() 按块读写文件

```c
#define  _CRT_SECURE_NO_WARNINGS 
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

#define FILE_NAME  "../data/file03.txt"


struct teacher
{
	int age;
	int id;
	char *name;  
	int name_len;
};

int main(void)
{
	FILE *fp = NULL;
#if 0
	int write_ret = 0;

	fp = fopen(FILE_NAME, "wb+");
	if (fp == NULL) {
		fprintf(stderr, "fopen error\n");
		return -1;
	}


	struct  teacher t1;
	char *name = "zhang3";

	t1.age = 10;
	t1.id = 20;
	t1.name = malloc(64);
	memset(t1.name, 0, 64);
	strcpy(t1.name, name);
	t1.name_len = strlen(name);

	write_ret = fwrite(&t1, sizeof(struct teacher), 1, fp);
	if (write_ret < 0) {
		fprintf(stderr, "write error\n");
		return -1;
	}
	write_ret = fwrite(t1.name, t1.name_len, 1, fp);
	if (write_ret < 0) {
		fprintf(stderr, "write error\n");
		return -1;
	}

	if (fp != NULL) {
		fclose(fp);
	}

#endif


// #if 0
	struct teacher t2 = { 0 };
	int read_ret = 0;

	fp = fopen(FILE_NAME, "rb+");
	if (fp == NULL) {
		fprintf(stderr, "fopen r+error\n");
		return -1;
	}

	read_ret = fread(&t2, sizeof(struct teacher), 1, fp);
	if (read_ret < 0) {
		fprintf(stderr, "fread error\n");
		fclose(fp);
		return -1;
	}

	t2.name = malloc(t2.name_len + 1);
	memset(t2.name, 0, t2.name_len + 1);

	read_ret = fread(t2.name, t2.name_len, 1, fp);
	if (read_ret < 0) {
		fprintf(stderr, "fread error\n");
		fclose(fp);
		return -1;
	}

	printf("id : %d, age : %d, name:%s, name_len :%d\n", t2.id, t2.age, t2.name, t2.name_len);


	if (fp != NULL)
	{
		fclose(fp);
	}

// #endif

	return 0;
}
```

