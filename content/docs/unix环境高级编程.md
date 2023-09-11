# Unix环境高级编程

> 基于Unix环境高级编程第三版的学习笔记。在学习之前一定要自己安装一个Unix操作系统。
>
> 书本中的源码：http://www.apuebook.com/code3e.html

## 第一章

### 第一个程序

> 列出一个目录中所有文件的名称。(一级，不嵌套)

```c
#include "apue.h"  //该头文件包含了所有的标准I/O函数的原型
#include <dirent.h>

int main(int argc,char *argv[]){

	DIR *dp;
	struct dirent *dirp;

	if(argc!=2)
	  err_quit("usage:ls directory_name");
	if ((dp = opendir(argv[1]))==NULL)
	  err_sys("can't open %s",argv[1]);
	while ((dirp = readdir(dp))!=NULL)
	  printf("%s\n",dirp->d_name);

	closedir(dp);
	exit(0);
}

```

> 书中给出的编译指令是:
>
> ```bash
> cc  myls.c #历史上，cc是C编译器。在配置了GNU  C编译系统的系统中，C 编译器是gcc。其中,cc 通常链接至 gcc. cc的参数与gcc一致
> ```

### 输入和输出

> 1. 文件描述符
>
> 文件描述符(file descriptor)通常是一个小的非负整数，内核用以标识一个特定进程正在访问的文件。当内核打开一个现有文件或创建一个新文件时，它都返回一个文件描述符。在读、写文件时，可以使用这个文件描述符。
>
> 2. 标准输入、标准输出和标准错误
>
> 按惯例，每当运行一个新程序时，所有的 shell 都为其打开 3个文件描述符，即标准输入(standard input)、标准输出(standard output)以及标准错误(standard error)。
>
> 如果不做特殊处理例如就像简单的命令 1s，则这3 个描述符都链接向终端。大多数 shell 都提供一种方法，使其中任何一个或所有这3个描述符都能重新定向到某个文件，例如:
>
> ```bash
> ls > file.list
> ```
> 执行 kls 命令，其标准输出重新定向到名为 file.list 的文件.
