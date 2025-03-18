
[[toc]]

2023-07-30 12:59


本书希望让你达到以下学习目的
- 掌握标准Linux C语言函数库 和由各种Linux或UNIX标准指定的其他工具的使用方法
- 掌握如何使用大多数标准Linux开发工具
- 学会通过DBM 和MySQL数据库系统 存储Linux中的数据
- 理解如何为 X视窗系统建立图形用户界面。我们将同时使用GTK和Qt函数库
- 拥有开发自己的实际应用程序的信心和能力

。。源代码提供了下载，下载在 G:\w\ff_dl\780470147627_download 中。


# 目录
- 第一章 入门
    - UNIX，Linux，GNU简介
- 2 shell程序设计
  - shell语法
- 3 文件操作
  - Linux文件结构
  - 系统调用和设备驱动程序
  - 库函数
  - 底层文件访问
  - 标准IO库
  - 格式化输入输出
  - 错误处理
  - 高级主题：fcnt1和mmap
- 4 linxu环境
  - 程序参数
  - 环境变量
  - 时间和日期
  - 临时文件
  - 日志
  - 资源和限制
- 5 终端
  - 对终端进行读写
  - 终端驱动程序和通用终端接口
  - termios结构
- 6 使用curses函数库管理基于文本的屏幕
  - 屏幕
  - 键盘
  - 窗口
  - 子窗口
  - 彩色显式
- 7 数据管理
  - 内存管理
  - 文件锁定
  - 数据库
- 8 MySQL
  - 使用C语言访问MySQL数据库
- 9 开发工具
  - make命令和makefile文件
- 10 调试
  - 常用调试技巧
  - 使用gdb进行调试
- 11 进程与信号
  - 进程的结构
  - 启动新进程
- 12 POSIX线程
  - 同时执行
  - 同步
  - 多线程
- 13 进程间通信：管道
  - 进程管道
  - 将输出送往popen
  - pipe调用
  - 父进程和子进程
- 14 信号量，共享内存和消息队列
- 15 套接字
- 16 使用GTK+进行GNOME编程
  - X视窗系统简介
- 17 用Qt进行KDE编程
- 18 Linux标准
  - C编程语言
  - 接口和LSB
  - 文件系统层次结构标准



# 第一章 入门

UNIX是由Open Group（开放组织）管理的一个商标，它指的是一种遵循特定规范的计算机操作系统

许多类UNIX系统都是具有商业性质的，如IBM的AIX、HP的HP-UX和Sun的Solaris。还有一些可以免费获得，如FreeBSD和Linux。
如今只有少数系统完全遵守开放组织的规范，从而允许它们挂上“UNIX”的商标。

## 一些典型的UNIX程序和系统所具有的特点。
- 简单性， KISS，keep it small and simple。==越大、越复杂的系统注定包含越大、越复杂的错误==，而调试是我们所有人都想避免的
苦差事。
- 集中性， ==功能臃肿的程序难于使用和维护，只有单一目标的程序更容易随着更好的算法或界面被开发出来而得到改进==。在UNIX中，当用户出现新的需求时，我们通常是把小工具组合起来以完成更复杂的任务，而不是试图将一个用户期望的所有功能放在一个大程序里。
- 可重用组件， 将应用程序的核心实现为库。具有简单而灵活的编程接口、文档齐备的库可以帮助其他人开发出同类程序
- 过滤器， 许多UNIX应用程序可用作过滤器。也就是说，它们对输入进行转换并产生输出。正如你将在后面看到的，UNIX提供了一些机制，让我们可以把一些UNIX程序通过一种新颖的方式组合起来，以开发出相当复杂的应用程序。
- 开放的文件格式， 比较成功并流行的UNIX程序都使用==纯ASCII码的文本文件或XML文件作为配置文件和数据文件==。
- 灵活性， 你不能期待用户都能非常正确地使用你的程序。所以，你在编程时应尽量考虑到灵活性，尽量避免随意限制字段长度或记录数目。

------------------------

Linux是一个可以自由发布的类UNIX内核实现，它是一个操作系统的底层核心。因为Linux以UNIX系统为其灵感来源，所以Linux程序和UNIX程序是非常相似的。事实上，几乎所有为UNIX编写的程序都可以在Linux上编译运行。


----------------------

自由软件基金会（Free Software Foundation）由Richard Stallman创立，他是UNIX及其他系统上最著名的文本编辑软件之一 GNU Emacs的开发者。
Stallman是自由软件这一概念的倡导者，并发起了GNU项目，这个项目的宗旨是：试图创建一个与UNIX系统兼容，但并不受UNIX名字和源代码私有权限制的操作系统和开发环境。


当登录进Linux系统时，你与一个shell程序（通常是bash）进行交互；它在一组指定的目录路径下按照你给出的程序名搜索与之同名的文件。
搜索的目录路径存储在shell变量PATH里搜索路径（你也可以添加这个路径）由系统管理员配置，
它通常包含如下一些存储系统程序的标准路径：
- /bin：二进制文件目录，用于存放启动系统时用到的程序
- /usr/bin：用户二进制文件目录，用于存放用户使用的标准程序
- /usr/local/bin：本地二进制文件目录，用于存放软件安装的程序

系统管理员（例如root用户）登录后使用的PATH变量可能还包含存放系统管理程序的目录，如/sbin和/usr/sbin。

可选的操作系统组件和第三方应用程序可能被安装在==/opt目录==下，安装程序可以通过用户安装脚本将路径添加到PATH环境变量中。

Linux像UNIX一样，使用冒号（:）分隔PATH变量里的条目，而不是像MS-DOS和Windows使用分号（;）。

下面是一个PATH变量的例子
`/usr/local/bin:/bin:/usr/bin:.:/home/neil/bin:/usr/X11R6/bin`

上面的PATH变量包含的条目有：标准程序存放位置、当前目录（.）、一个用户的家目录和X视窗系统的目录。

在典型的Linux系统上有许多编辑器可用，较流行的编辑器是vi。
本书的两位作者都喜欢Emacs，所以我们建议你花一点时间来学习这个功能强大的编辑器。

#### gcc -o
`gcc -o hello hello.c`



### 应用程序
系统为正常使用提供的程序，包括用于程序开发的工具，都可在目录/usr/bin中找到；
系统管理员为某个特定的主机或本地网络添加的程序通常可在目录/usr/local/bin或/opt中找到。

保持这种方式，这样，在升级操作系统的时候，只需要保留 /opt和 /usr/local 中的内容。

我们建议对于系统级的应用程序，你可以将它放在/usr/local目录中来运行和访问所需的文件。
对于开发用和个人的应用程序，最好在你的家目录中使用一个文件夹来存放它。

其他一些功能特性和编程系统可能有其自己的目录结构和程序目录。其中最主要的一个就是X视窗系统，它通常安装在/usr/X11或/usr/bin/X11目录中

GNU编译系统的驱动程序gcc（你已在本章前面的编程示例中用过）一般位于/usr/bin或/usr/local/bin目录中，但它会从其他位置运行各种编译器支持的应用程序。


### 头文件

对C语言来说，这些头文件几乎总是位于`/usr/include目录及其子目录`中。
那些依赖于特定Linux版本的头文件通常可在目录`/usr/include/sys和/usr/include/linux`中找到。

#### gcc -I
在调用C语言编译器时，你可以使用-I标志来包含保存在子目录或非标准位置中的头文件
`gcc -I/usr/openwin/include notTest.c`


#### grep EXIT_ *.h
用grep命令来搜索包含某些特定定义和函数原型的头文件是很方便的。
假设想知道用于从程序中返回退出状态的#define定义的名字，
你只需切换到`/usr/include`目录下，然后用grep命令搜索可能的名字部分
`grep EXIT_ *.h`

。。下面是 ubuntu20 的
```text
aa@yiqi:/usr/include$ grep EXIT_ *.h
argp.h:#define ARGP_HELP_EXIT_ERR	0x100 /* Call exit(1) instead of returning.  */
argp.h:#define ARGP_HELP_EXIT_OK	0x200 /* Call exit(0) instead of returning.  */
argp.h:  (ARGP_HELP_SEE | ARGP_HELP_EXIT_ERR)
argp.h:  (ARGP_HELP_SHORT_USAGE | ARGP_HELP_SEE | ARGP_HELP_EXIT_ERR)
argp.h:  (ARGP_HELP_SHORT_USAGE | ARGP_HELP_LONG | ARGP_HELP_EXIT_OK \
stdlib.h:#define	EXIT_FAILURE	1	/* Failing exit status.  */
stdlib.h:#define	EXIT_SUCCESS	0	/* Successful exit status.  */
```

上面的grep命令在当前目录下的所有以.h结尾的文件中搜索字符串EXIT_
。。最后2个stdlib是我们想要的。


### 库文件
库是一组预先编译好的函数的集合
它们通常由一组相互关联的函数组成以执行某项常见的任务，比如屏幕处理函数库（curses和ncurses库）和数据库访问例程（dbm库）

标准系统库文件一般存储在`/lib和/usr/lib`目录中

C语言编译器（或更确切地说是链接程序）需要知道要搜索哪些库文件，因为在默认情况下，它只搜索标准C语言库

库文件的名字总是以lib开头，随后的部分指明这是什么库（例如，c代表C语言库，m代表数学库）。文件名的最后部分以.开始，然后给出库文件的类型：
- .a  传统的静态函数库
- .so  共享函数库

。。ubuntu20 里没有这些后缀。。

你可以通过给出`完整的库文件路径名`或`用-l标志`来告诉编译器要搜索的库文件
`gcc -o OuhaNotTest fred.c /usr/lib/libm.a`


#### gcc -L -l
下面的和上面的一样。
-lm 是简写方式，代表的是 标准库目录 中名为 lib.a 的函数库。
`gcc -o OuhaNotTest fred.c -lm`

使用-L（大写字母）标志为编译器增加库的搜索路径
`gcc -o not -L/usr/openwin/lib xilfred.c -lX11`

用/usr/openwin/lib目录中的libXll库版本来编译和链接程序xllfred。

。。就是前面加一个 lib，然后作为文件名。


### 静态库

函数库最简单的形式是一组处于“准备好使用”状态的目标文件。

当程序需要使用函数库中的某个函数时，它包含一个声明该函数的头文件。
编译器和链接器负责将程序代码和函数库结合在一起以组成一个单独的可执行文件。
你必须使用`-l选项`指明除标准C语言运行库外还需使用的库。

静态库，也称作归档文件（archive），按惯例它们的文件名都以.a结尾。比如，标准C语言函数库/usr/lib/libc.a和Xll函数库/usr/lib/libXll.a

。。但是 ubuntu20的/usr/lib 里没有 libc.a


#### gcc -c && ar

你可以很容易地创建和维护自己的静态库，只要使用`ar`（代表archive，即建立归档文件）程序和使用`gcc -c`命令对函数分别进行编译。
你应该尽可能把函数分别保存到不同的源文件中。
如果函数需要访问公共数据，你可以把它们放在同一个源文件中，并使用在该文件中声明的静态变量



### Code Example

1. 创建被调用的源码，即库的源码
```C
// bill.c
// freg.c 也类似。
#include <stdio.h>

void bill(char* c)
{
    printf("bill %s\n", c);
}
```

2. 编译第一步的2个文件
```text
gcc -c bill.c freg.c
// ls *.o   可以看到 bill.o 和 freg.o
```

3. 为库源码创建头文件
```C
// lib.h
void bill(char*);
void freg(int);
```

4. 创建调用程序。

```C
// use_bill_freg_lib.c
#include <stdlib.h>
#include "lib.h"

int main()
{
    bill("hi, bill, from use");
    freg(123123);
    exit(0);
}
```

5. 编译运行
```text
aa@yiqi:~/linux_program/ch01$ gcc -c use_bill_freg_lib.c 
aa@yiqi:~/linux_program/ch01$ gcc use_bill_freg_lib.o bill.o freg.o
aa@yiqi:~/linux_program/ch01$ ls
a.out   bill.o  freg.o  hello.c  use_bill_freg_lib.c
bill.c  freg.c  hello   lib.h    use_bill_freg_lib.o
aa@yiqi:~/linux_program/ch01$ ./a.out
bill hi, bill, from use
freg. 123123
```

```text
aa@yiqi:~/linux_program/ch01$ gcc -o use use_bill_freg_lib.o bill.o freg.o
aa@yiqi:~/linux_program/ch01$ ./use
bill hi, bill, from use
freg. 123123
```

6. 创建库文件
```text
aa@yiqi:~/linux_program/ch01$ ar crv libmy.a bill.o freg.o
a - bill.o
a - freg.o
```

7. 为函数库生成内容表(可选)
Linux中，当你使用的是GNU的软件开发工具时，这一步骤并不是必需的

`ranlib libmy.a`

8. 使用库文件
`gcc -o use use_bill_freg_lib.o libmy.a`
`gcc -o use use_bill_freg_lib.o -L. -lmy`

```text
aa@yiqi:~/linux_program/ch01$ gcc -o use use_bill_freg_lib.o libmy.a 
aa@yiqi:~/linux_program/ch01$ ./use
bill hi, bill, from use
freg. 123123
aa@yiqi:~/linux_program/ch01$ gcc -o use use_bill_freg_lib.o -L. -lmy
aa@yiqi:~/linux_program/ch01$ ./use
bill hi, bill, from use
freg. 123123
```

-L.选项告诉编译器在当前目录（.）中查找函数
库。-lfoo选项告诉编译器使用名为libfoo.a的函数
库（或者名为libfoo.so的共享库，如果它存在的
话）。
。。我用的是 libmy.a

要查看哪些函数被包含在目标文件、函数库
或可执行文件里，你可以使用nm命令。如果你查
看program和libfoo.a，你就会看到函数库libfoo.a
中包含fred和bill两个函数，而program里只包含函
数bill
。。书上的 program.c 就是 我的 use_bill_freg_use.c，不过 书上只调用了 bill，我调用了bill和 freg。


#### nm

```text
aa@yiqi:~/linux_program/ch01$ nm libmy.a 
bill.o:
0000000000000000 T bill
                 U printf
freg.o:
0000000000000000 T freg
                 U printf
```

。。nm use (use_bill_freg_use.c 的可执行文件) 有用，不过好长，感觉是类似 java的字节码。

当程序被创建时，它只包含函数库中它实际需要的函数

-------------------------

windows与Unix的对应关系
|项目|Unix|Windows|
|--|--|--|
|目标模块|func.o|func.obj|
|静态函数库|lib.a|lib.lib|
|程序|program|program.exe|



### 共享库

静态库的一个缺点是，当你同时运行许多应用程序并且它们都使用来自同一个函数库的函数时，内存中就会有==同一函数的多份副本==，而且在程序文件自身中也有多份同样的副本

对共享库及其在不同系统上实现方式的详细讨论超出了本书的范围，所以我们将仅讨论Linux下的实现

共享库的保存位置与静态库是一样的，但共享库有不同的文件名后缀

当一个程序使用共享库时，它的链接方式是这样的：
程序本身不再包含函数代码，而是引用运行时可访问的共享代码。
当编译好的程序被装载到内存中执行时，函数引用被解析并产生对共享库的调用，==如果有必要，共享库才被加载到内存中==。

通过这种方法，系统可以==只保留共享库的一份副本==供许多应用程序同时使用，并且在磁盘上也仅保存一份。
另一个好处是共享库的==更新可以独立于依赖它的应用程序==。例如，文件/lib/libm.so就是对实际库文件修订版本（/lib/libm.so.N，其中N代表主版本号，在写作本书时它是6）的符号链接

。。`sudo find /usr -name lib*.so` 可以找到很多 so文件。


#### ldd

```text
aa@yiqi:~/linux_program/ch01$ ldd use
	linux-vdso.so.1 (0x00007fffb06d5000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f2221400000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f222163e000)
```



### man
`man man`


### info gcc





# 第二章 shell程序设计


## 一点哲学
我们来关注一点UNIX（当然也是
Linux）的哲学。UNIX架构非常依赖于代码的高度
可重用性。如果你编写了一个小巧而简单的工具，
其他人就可以将它作为一根链条上的某个环节来构
成一条命令。Linux让用户满意的原因之一就是它
提供了各种各样的优秀工具。下面是一个简单的例
子：
`ls -al | more`
。。能用，但是 带不带 | more ，好像都一样。。
。。 more是一个命令：A file perusal filter for CRT viewing.
。。是因为文件太少了，文件多了以后， ls -al 一次性打印全部的文件信息， 加上 | more 后，只显示一个屏幕的信息，按回车 继续显示下一行。

这个命令使用了ls和more工具并通过管道实现了文件列表的分屏显示

打印bash使用手册的参考副本，可以使用如下命令
`man bash | col -b | lpr`

。。man bash，第一次只显示一个屏幕，回车显示下一行，
加上 | col -b 后 一次性打印完，。。不过 加不加 -b 都一样。。
| lpr 不能用， 报错： 错误 - 无默认目标。


因为Linux具备自动文件类型处理功能，
所以使用这些工具的用户一般不必了解它们是用哪
种语言编写的。如果想要这些工具运行得更快，常
见的做法是首先在shell中实现工具的原型，一旦确
定值得这么做，然后再用C或C++、Perl、Python
或者其他执行得更快速的语言来重新实现它们。相
反，如果在shell中这些工具工作得已足够好，就不
必再重新实现它们。

。。但是，shell的开发速度 还能比 python 快？ 感觉开发 python已经非常快了。


Linux系统中已
经装有许多的shell脚本例子，包括软件包安装程
序、.xinitrc和startx文件以及/etc/rc.d目录中用
于启动时配置系统的脚本程序。

。。没有 /etc/rc.d 目录。。 root用户也没有这个 目录。


## what is shell

shell是一个作为用户与Linux系统
间接口的程序，它允许用户向操作系统输入需要执
行的命令。这点与Windows的命令提示符类似，但
正如先前所提到的，Linux shell的功能更强大。例
如，
我们可以使用<和>对输入输出进行重定向，
使用|在同时执行的程序之间实现数据的管道传递，
使用$(...)获取子进程的输出。
在Linux中安装多个
shell是完全可行的，用户可以挑选一种自己喜欢的
shell来使用。

Linux系统中，总是作为/bin/sh安装的标准shell是
GNU工具集中的bash（GNU Bourne-Again Shell）

在本章中，我们将使用bash的第3版

我们假设bash被安装
为/bin/sh并且它是你的登录所使用的默认shell。
在大多数Linux发行版中，默认的shell程序/bin/sh
实际上是对程序/bin/bash的一个连接

`/bin/bash --version`
。。我的是5.1.16.
。。/bin/sh 确实应该是一个链接， 因为`/bin/sh --version 不能用`。illegal option。。

可以切换用户的shell。

常用的shell
|名称|相关历史|
|--|--|
|sh(Bourne)|源于UNIX早期版本的最初的shell|
|csh,tcsh,zsh|C shell及其变体|
|ksh,pdksh|Korn shell和 public domain korn shell,许多商业版UNIX的默认shell|
|bash|GNU|

除了C shell和少数变体以外，所有这些shell都
很相似，并且都与X/Open 4.2和POSIX 1003.2规
范中对于shell的规定非常一致。POSIX 1003.2对
于shell的规定很少，但在X/Open中的扩展规定则
提供了一个更加友好、功能更加强大的shell。
X/Open通常是一个提出更多要求的规范，但遵循
它的系统也更加友好。


## 2.4 管道和重定向

### 重定向输出

`ls -l > lsoutput.txt`
把ls命令的输出保存到文件lsoutput.txt中。

将在第3章学习更多关于
标准文件描述符的内容，现在你只需知道
==文件描述符 0 代表一个程序的 标准输入，
文件描述符 1 代表 标准输出，
而文件描述符 2 代表标准 错误输出==

上面的例子通过 `>操作符` 把标准输出重定向到一个文件
在默认情况下，如果该文件已经存在，它的内容将被覆盖
你可以使用命令`set -o noclobber（或set -C）`命令设置
noclobber选项，从而阻止重定向操作对一个已有
文件的覆盖。你可以使用`set +o noclobber`命令取消该选项

用 `>>操作符` 将输出内容附加到一个文件中

如果想对标准错误输出进行重定向，你需要把
想要重定向的文件描述符编号加在>操作符的前
面。因为`标准错误输出的文件描述符编号是2`，所
以使用 `2>操作符` 。当需要丢弃错误信息并阻止它显
示在屏幕上时，这个方法很有用。

下面的命令将把标准输出和标准错误输出分别重定向到不同的文件中
`kill -HUP 1234 >kill.out 2>kill.err`
。。可用，这2个文件都会建立，只不过 kill.out 是空文件。 kill.err里是 该进程不存在。

如果你想把两组输出都重定向到一个文件中，
你可以用 `>&操作符` 来结合两个输出
`kill -l 1234 >kill.all 2>&1`
。。ok

请==注意操作符出现的顺序==。这
条命令的含义是“将标准输出重定向到文件
killouterr.txt，然后将标准错误输出重定向到与标
准输出相同的地方。”如果顺序有误，重定向将不
会按照你预期的那样执行。

。。& 感觉更像 C++的引用。 2放到1里。

因为可以通过==返回码==（我们将在本章的后面对
其进行详细介绍）来了解kill命令的执行结果，所以
通常并不需要保存标准输出或标准错误输出的内
容。你可以用Linux的通用“回收站”/dev/null来
有效地丢弃所有的输出信息
`kill -l 1234 >/dev/null 2>&1`


### 重定向输入

`more < kill.out`

很明显，在Linux下这样做意义不大，因为
Linux的more命令可以接受文件名作为参数，这与
Windows命令行中对应的命令不同。

### 管道

你可以用 `管道操作符|` 来连接进程。Linux与
MS-DOS不同，在Linux下通过管道连接的进程可
以同时运行，并且随着数据流在它们之间的传递可
以自动地进行协调

你可以使用sort命令对ps命令的输出进行排序。
如果不使用管道，你就必须分几个步骤来完成
这个任务，如下所示
`ps > ps.out`
`sort ps.out > ps.sorted`

一个更精巧的解决方案是用管道来连接进程，
如下所示
`ps | sort > ps.sorted`

如果想在屏幕上分页显示输出结果，你可以再
连接第三个进程more，将它们都放在同一个命令
行上，如下所示：
`ps | sort | more`

#### grep -v

允许连接的进程数目是没有限制的。
假设你想看看系统中运行的所有进程的名字，但不包括shell
本身，可以使用下面的命令
`ps -xo comm | sort | uniq | grep -v sh | more`

这个命令首先按字母顺序排序ps命令的输出，
再用uniq命令去除名字相同的进程，
然后用`grep -v sh`命令删除名为sh的进程，
最终将结果分页显示在屏幕上


注意：如果你有一系列的命令需要执行，相应的输出文件是在这
一组命令被创建的同时立刻被创建或写入的，所以
==决不要在命令流中重复使用相同的文件名==


### 2.5 shell编程

假设你想要从大量C语言源文件中查找包含字符
串POSIX的文件。与其使用grep命令在每个文件中
搜索字符串，然后再分别列出包含该字符串的文
件，不如用下面的交互式脚本来执行整个操作
```shell
$ for file in *
> do
> if grep -l POSIX $file
> then
> more $file
> fi
> done
```

。。
```text
aa@yiqi:~$ for f in *
> do 
> if grep -l POSIX $f
> then
> more $f
> fi
> done
grep: 公共的: 是一个目录
grep: 模板: 是一个目录
grep: 视频: 是一个目录
grep: 图片: 是一个目录
grep: 文档: 是一个目录
grep: 下载: 是一个目录
grep: 音乐: 是一个目录
grep: 桌面: 是一个目录
grep: git_repo: 是一个目录
grep: snap: 是一个目录
```
。。

在这个例子中，grep命令输出它找到的包含
POSIX字符串的文件，然后more命令将文件的内
容显示在屏幕上。最后，返回shell提示符

shell还提供了通配符扩展（通常称为 globbing）

`ls my_{finger,toe}s`
这个命令将列出文件my_fingers和my_toes

```text
aa@yiqi:~/ch01$ ls li{b,bmy}.*
lib.h  libmy.a
aa@yiqi:~/ch01$ ls li{b,my}.*
ls: 无法访问 'limy.*': 没有那个文件或目录
 lib.h
```

有经验的Linux用户可能会用一种更有效的方式来执行这个简单的操作
```shell
more `grep -l POSIX *`
```
。。反引号，不过不能用， 返回 more: bad usage。。。
。。单引号的话，会当做 文件，所以more 无法打开这个文件。

或使用功能相同的另一种命令形式
`more ${grep -l POSIX *}`

。。也不行。。
ch01$ more ${grep -I POSIX *}
bash: ${grep -I POSIX *}: 错误的替换

下面的命令将输出包含POSIX字符串的文件名
`grep -l POSIX * | more`

。。可以
```
aa@yiqi:~/ch01$ grep -l include * | more
bill.c
freg.c
hello.c
use_bill_freg_lib.c
```

### 创建脚本

新建文件：
```
#!/bin/sh

for file in *
do
	if grep -q include $file
	then
		echo $file
	fi
done

exit 0
```

注意第一行#!/bin/sh，它是一种特殊形式的注释，#!字符告诉
系统同一行上紧跟在它后面的那个参数是用来执行本文件的程序。

路径按惯例最好不要超过32个字符，因为一些老版本的UNIX在使用#!时只能使用
这个限制之内的字符数，虽然Linux通常不存在这样的限制

因为脚本程序本质上被看作是shell的标准输
入，所以它可以包含任何能够通过你的PATH环境变
量引用到的Linux命令

exit命令的作用是确保脚本程序能够返回一个有意义的退出码

我们很少需要检查它的退出码，但如果你打算从另一个脚本程序里
调用这个脚本程序并查看它是否执行成功，那么返回一个适当的退出码就很重要了

在shell程序设计里，0表示成功

#### file
Linux，UNIX 很少利用文件扩展名来决定文件的类型。
检查文件的类型 最好的方法是 file 命令
`file hi_shell`
`file /bin/bash`


### 运行脚本
2种方法
。。hi_shell 是shell的文件名
1. `/bin/sh hi_shell`

2. `hi_shell`
可能需要 `chmod +x hi_shell`。  
。。要的。
很可能发生错误：找不到命令"hi_shell"。因为shell环境变量PATH并没有被设置为在当前目录下查找要执行的命令
一种办法是在命令行上直接输入命令`PATH=$PATH:.`或 编辑你的.bash_profile文件，将刚才这条命令添加到文件的末尾，然后退出登录后再重新登录进来
也可以`./hi_shell`
。。直接 hi_shell 确实不行， 终端执行 PATH=$PATH:. 后，hi_shell 可以

用./来指定路径还有另一个好处，它能够保证你
不会意外执行系统中与你的脚本文件同名的另一个命令

你不应该用这种方法来修改超级用
户（一般其用户名为root）的PATH变
量。这是一个安全方面的漏洞，因为以
root用户身份登录的系统管理员可能会
因此误调用了某个标准命令的伪装版本

如果这个命令只供你本人使用，你可以在自己的家目录中
创建一个bin目录，并且将该目录添加到你自己的
PATH变量中。如果你想让其他人也能够执行这个脚
本程序，你可以将/usr/local/bin或其他系统目录
作为添加新程序的适当位置
为了防止其他用户修改脚本程序，哪怕只是意
外地修改，你也应该去掉脚本程序的写权限
```text
cp first /usr/local/bin
chown root /usr/local/bin/first
chgrp root /usr/local/bin/first
chmod 755 /usr/local/bin/first
```

## shell语法
将学习
- 变量
- 条件
- 程序控制
- 命令列表
- 函数
- shell内置命令
- 获得命令的执行结果
- here文档

### 变量
在shell里，使用变量之前通常并不需要事先为它们做出声明。

在默认情况下，==所有变量都被看作字符串并以字符串来存储，即使它们被赋值为数值时也是如此==

区分大小写

在变量名前加==一个$符号==来访问它的内容

```text
a=1   // 不能 a = 1, a会被当做命令的，
echo $a
a="asd wf ds"     // 没有"" 的话， wf 被当做命令。
echo $a     // 返回 asd wf ds 。。 没有 ""
a=7+1
echo $a   // 7+1
```

注意，如果字符串里包含空格，就
必须用引号把它们括起来。此外，等号
两边不能有空格。

#### read
使用read命令将用户的输入赋值给一个变量
。。`read a` 回车， 终端等待你的输入， 输入完回车。  `echo $a` ，就可以看到你的输入， 输入可以直接空格，不需要"" 。   所见即所得。
。。不过它会 trim 两侧的空格。


#### 使用引号

一般情况下，脚本文件中的参数以空白字符分
隔（例如，一个空格、一个制表符或者一个换行
符）。如果你想在一个参数中包含一个或多个空白
字符，你就必须给参数加上引号。

像$foo这样的变量在引号中的行为取决于你所使用的引号类型。
如果你把一个$变量表达式放在双引号中，程序执行到这一行时就会把变量替换为它的值；
如果你把它放在单引号中，就不会发生替换现象。
你还可以通过在$字符前面加上一个\字符以取消它的特殊含义

。。挺乱的。不过应该不会 "' 混用
```text
aa@yiqi:~/ch01$ echo $a
" adff "
aa@yiqi:~/ch01$ echo "asd + $a"
asd + " adff "
aa@yiqi:~/ch01$ echo ""asd + $a""
asd + " adff "
aa@yiqi:~/ch01$ echo '""asd + $a""'
""asd + $a""
aa@yiqi:~/ch01$ echo '"asd + $a"'
"asd + $a"
aa@yiqi:~/ch01$ echo "'"asd + $a"'"
'asd + " adff "'
aa@yiqi:~/ch01$ echo ""asd + $a""
asd + " adff "
aa@yiqi:~/ch01$ echo ""asd + \$a""
asd + $a
```

字符串通常都被放在双引号中，以防止变量被
空白字符分开，同时又允许$扩展。


#### 环境变量

当一个shell脚本程序开始执行时，一些变量会
根据环境设置中的值进行初始化。这些变量通常用
大写字母做名字，以便把它们和用户在脚本程序里
定义的变量区分开来，后者按惯例都用小写字母做
名字

具体创建的变量取决于你的个人配置。在系统的使用手册中列出了许多这样的环境变量
一些主要的变量
|环境变量|说明|
|--|--|
|$HOME|home目录|
|$PATH|PATH|
|$PS1|命令提示符，通常是\$字符，但bash中有更复杂的值，例如[\u@\h \W]$ 是一个流行的默认值，给出用户名，机器名，当前目录名及$提示|
|$PS2|二级提示符，用来提示后续输入，通常是 > 字符|
|$IFS|输入域分隔符，当shell读取输入时，它给出 用来分隔单词的一组字符，通常是空格，制表符，换行符|
|$0|shell脚本的名字|
|$#|传递给脚本的参数个数|
|$$|shell脚本的进程号。脚本程序通常会用它来生成一个唯一的临时文件，例如/tmp/tmpfile_$$|

如果想通过执行`env <command>`
命令来查看程序在不同环境下是如何工
作的，请查阅env命令的使用手册。你
也将在本章的后面看到如何使用export
命令在子shell中设置环境变量


#### 参数变量
如果脚本程序在调用时带有参数，一些额外的变量就会被创建。
即使没有传递任何参数，环境变量$#也依然存在，只不过它的值是0罢了

|参数变量|说明|
|--|--|
|$1,$2...|脚本程序的参数|
|$*|一个变量中列出所有参数，各个参数之间使用 IFS中的第一个字符 分隔|
|$@|$*的一个变体，不使用IFS，所以即使IFS为空，参数也不会挤一起|

$@ 与 $* 的区别
```text
aa@yiqi:~/ch01$ set foo bar bam
aa@yiqi:~/ch01$ echo "$@"
foo bar bam
aa@yiqi:~/ch01$ echo "$*"
foo bar bam
aa@yiqi:~/ch01$ IFS=''
aa@yiqi:~/ch01$ echo "$*"
foobarbam
aa@yiqi:~/ch01$ echo "$@"
foo bar bam
aa@yiqi:~/ch01$ unset IFS
aa@yiqi:~/ch01$ echo "$*"
foo bar bam
```

一般来说，如果你想访问脚本程序的参数，使用 $@ 是明智的选择。

可以使用read命令来读取它们。


#### 例子
```
#!/bin/sh

hi="hello"
echo $hi
echo "the program $0 is now running"
echo "the second parameter was $2"
echo "the first parameter was $1"
echo "the parameter list was $*"
echo "user's home dir is $HOME"

echo "please enter a new greeting"
read hi

echo $hi
echo "script is end"

exit 0
```

### 条件

#### test或[命令

大多数脚本程序都会广泛使用
shell的`布尔判断命令[或test`。在一些系统上，这两
个命令的作用是一样的，只是为了增强可读性，当
使用`[`命令时，我们还使用符号`]`来结尾。把`[`符号当
作一条命令多少有点奇怪

如果程序不能正常工
作，很可能是因为它与shell中的test命
令发生了==冲突==。要想查看系统中是否有
一个指定名称的外部命令，你可以尝试
使用`which test`这样的命令来检查执行
的是哪一个test命令，或者使用`./test`这
种执行方式以确保你执行的是当前目录
下的脚本程序

检查一个文件是否存在。用于实现这一操作的命令是
`test -f <filename>`

所以，可以写成：
```shell
if test -f hi_world.c
then
...
fi
```
或
```shell
if [ -f hi_world.c ]
then
...
fi
```

test命令的退出码（表明条件是否被满足）决定
是否需要执行后面的条件代码

必须在 [符号和被检查的条件之间留出空格

如果你喜欢把then和if放在同一行
上，就必须要用一个分号把test语句和
then分隔开

```shell
if [ -f hi_world.c ]; then
...
fi
```

test命令可以使用的条件类型可以归为3类：
- 字符串比较
- 算术比较
- 与文件有关的条件测试

|字符串比较|结果|
|--|--|
|str1 = str2|如果字符相同则结果为真|
|str1 != str2|字符串不同则结果为真|
|-n str|不为空则 真|
|-z str|为null 则 真|


|算术比较|结果|
|--|--|
|expression1 -eq expression2|表达式 是否相等|
|exp1 -ne exp2|表达式是否 不等|
|exp1 -gt exp2|exp1 > exp2|
|exp1 -ge exp2| >= |
|exp1 -lt exp2| < |
|exp1 -le exp2| <= |
|! exp| 取反|


|文件条件测试|结果|
|--|--|
|-d file|文件是否是 目录|
|-e file|文件是否存在，注意，历史上-e不可移植，所以通常使用-f|
|-f file|文件是否是 一个普通文件|
|-g file|文件的set-group-id位 是否被设置|
|-r file|是否可读|
|-s file|大小是否 不为0|
|-u file|set-user-id位 是否被设置|
|-w file|文件是否可写|
|-x file|是否可执行|


什么是set-groupid和set-user-id（也叫做set-gid和setuid）位。
set-uid位授予了程序其拥有者的访问权限而不是其使用者的访问权限，
而set-gid位授予了程序其所在组的访问权限。这两个特殊位是通过chmod命令的选项s和g设置的。
setgid和set-uid标志对shell脚本程序不起作用，它们只对可执行的二进制文件有用。


```text
#!/bin/sh

if [ -f /bin/bash ]
then
    echo "file /bin/bash exists"
fi

if [ -d /bin/bash ]
then
    echo "/bin/bash is a directory"
else
    echo "/bin/bash is not a directory"
fi
```
。。没有 exit(0)， 没有问题。

`help test`


### 控制结构

#### if

```text
if condition
then
  statements
else
  statements
fi
```

```shell
#!/bin/sh

echo "is is morning? yes or no"
read t2

if [ $t2 = "yes" ]; then
    echo "Good morning"
else
    echo "Good afternoon"
fi

exit 0
```
。。"yes" 和 ] 要留空格


#### elif

```shell
#!/bin/sh

echo "is is morning? yes or no"
read t2

if [ $t2 = "yes" ]; then
    echo "Good morning"
elif [ $t2 = "no" ]; then
    echo "Good afternoon"
else
    echo "??? yes or no"
    exit 1
fi

exit 0
```


运行脚本，不输入任何字符，直接回车。会显示一些错误信息。
是因为输入空，导致变成了 `if [ = "yes" ]` 。这不是一个合法的条件。
需要加上引号：
`if [ "$t2" = "yes" ]; then`
这样，输入空，就变成了 `if [ "" = "yes" ]` 。合法的条件。


如果想要去除 echo 输出的每行后面的换行：
- printf 代替 echo ，可移植性最好
- echo -n， bash使用，有些shell使用 echo -e。


#### for

```text
for variable in values
do
  statements
done
```

```shell
#!/bin/sh

for foo in bar fud 43
do
    echo -n $foo
done
exit 0
```

`for foo in $(ls t*.sh)`


#### while

```text
while condition do
  statement
done
```

```shell
#!/bin/sh

echo "enter pwd"
read t2

while [ "$t2" != "pwd" ]; do
    echo "again."
    read t2
done
exit 0
```

#### until

```text
until condition
do
  statements
done
```

```shell
#!/bin/sh

until who | grep "$1" > /dev/null
do
    sleep 60
done

echo -e '\a'
echo "**** $1 has just logged in ****"

exit 0
```

。。echo *， 是打印目录下所有文件。。 所以上面的 不加 "" 。就。。 不过只打印2次，前面一次，后面一次。
。。这个一般用 ./test_until.sh ${MyName's part} 来触发。 就是 用自己的名字来触发。



#### case
case variable in 
  pattern [ | pattern ] ...) statements;;
  pattern [ | pattern ] ...) statements;;
esac


case结构允许你通过一种比较复杂的方式将变量的内容和模式进行匹
配，然后再根据匹配的模式去执行不同的代码。这
要比使用多条if、elif和else语句来执行多个条件检查要简单得多

请注意，每个模式行都以双分号（;;）结尾。
因为你可以在前后模式之间放置多条语句，所以
需要使用一个双分号来标记前一个语句的结束和
后一个模式的开始。

你在case结构的模式中使用如*这样
的通配符时要小心。因为case将使用==第
一个匹配的==模式，即使后续的模式有==更
加精确==的匹配也是如此
。。从上往下，依次比较。符合就执行。

```shell
#!/bin/sh

echo "is it morning? yes or no"
read t2

case "$t2" in 
    yes)    echo "Good morning";;
    no )    echo "Good afternoon";;
    y  )    echo "Good morning";;
    n  )    echo "Good afternoon";;
    *  )    echo "Sorry, answter not recognized";;
esac

exit 0
```

```shell
#!/bin/sh

echo "is it morning?"
read t2

case "$t2" in
    yes|y|YES)  echo "Good Morning";;
    n*|N*)  echo "Good afternoon";;
    *)  echo "not recognized";;
esac

exit 0
```

```shell
#!/bin/sh

echo "is it morning?"
read t2

case "$t2" in
    yes | y | YES )
        echo "Good morning"
        echo "Up bright"
        ;;
    [nN]*)
        echo "Good afternoon"
        ;;
    *)
        echo "no recognized"
        echo "plz check"
        exit 1
        ;;
esac

exit 0
```

如果最后一个case模式是默认模式，那么省略最
后一个双分号（;;）是没有问题的，因为后面没
有其他的case模式需要考虑了。

可以使用如下的模式
`[yY] | [yY][eE][sS] )`


#### 命令列表

##### AND列表
`statement1 && statement2 && statement3 ...`

`if [ -f file1 ] && echo "hello" && [ -f file2 ] && echo "there"`

。。echo很6

```shell
#!/bin/sh

touch file1
rm -f file2

if [ -f file1 ] && echo "1111" && [ -f file2 ] && echo "2222"
then
    echo "in if"
else
    echo "in else"
fi

exit 0
```

==touch和rm命令确保当前目录中的有关文件处于已知状态==
。。这个有点意思。如果不写是怎么样的？

##### OR列表
`statement1 || statement2 || ...`


#### 语句块{}

如果你想在某些只允许使用单个语句的地方
（比如在AND或OR列表中）使用多条语句，你可
以把它们括在`花括号{}`中来构造一个语句块


### 函数
```text
function_name(){
  statement
}
```

```shell
#!/bin/sh

foo() {
    echo "function foo is executing"
}

echo "start"
foo
echo "end"

exit 0
```

当一个函数被调用时，脚本程序的位置参数
（$*、$@、$#、$1、$2等）会被替换为函数的参
数。这也是你读取传递给函数的参数的办法。当函
数==执行完毕==后，这些参数会==恢复==为它们先前的值

一些老版本的shell在函数执行之后
可能不会恢复位置参数的值。

可以通过return命令让函数==返回数字值==

让函数==返回字符串值==的常用方法是让函数将字符串保
存在一个变量中，该变量然后可以在函数结束之后被使用。
此外，你还可以echo一个字符串并捕获其结果

```text
foo () { echo JAY; }
...
result="${foo}"
```

可以使用local关键字在shell函数中声明局部变量，局部变量将仅在函数的作用范围内有效

如果一个局部变量和一个全局变量的名字相同，前者就会覆盖后者

```shell
#!/bin/sh

txt="global v"

foo(){
    local txt="local v"
    echo "function foo is executing"
    echo $txt
}

echo "start"
echo $txt

foo

echo "en"
echo $txt

exit 0
```

如果在函数里没有使用return命令指定一个返
回值，函数返回的就是执行的最后一条命令的退出码


```shell
#!/bin/sh

yesOrNo() {
    echo "is your name $* ?"
    while true
    do
        echo -n "Enter yes or no: "
        read x
        case "$x" in 
            y | yes )   return 0;;
            n | no )    return 1;;
            * )         echo "yes or no!!!"
        esac
    done
}

echo "Original parameter are $*"

if yesOrNo "$1"
then
    echo "hi, $1, nice name"
else
    echo "never mind"
fi
exit 0
```

```text
aa@yiqi:~/ch02$ ./test_fun_return.sh asd
Original parameter are asd
is your name asd ?
Enter yes or no: yes
hi, asd, nice name
```


### 命令
可以在shell脚本程序内部执行两类命令。
一类是可以在命令提示符中执行的“普通”命令，也
称为外部命令（external command），
一类是我们前面提到的“内置”命令，也称为内部命令
（internal command）。内置命令是在shell内部
实现的，它们不能作为外部程序被调用

然而，大多数的内部命令同时也提供了独立运行的程序版本
——这一需求是POSIX规范的一部分。
通常情况下，命令是内部的还是外部的并不重要，
只是内部命令的执行效率更高。

#### break命令

在控制条件未满足之前，跳出for、while或until循环

```shell

for f in *
do
  if [ -d "$f" ]; then
    break;
  fi
done
```

在默认情况下，break只跳出一层循环。
为break命令提供一个额外的数值参数来表明需要跳出的循环层数，不建议这么做，降低了可读性


#### :命令

冒号（:）命令是一个空命令。它偶尔会被用于
简化条件逻辑，相当于true的一个别名。由于它是
内置命令，所以它运行的比true快，但它的输出可
读性较差。

`while :`实现了一个无限循环，代替了更常见的`while true`。

:结构也会被用在变量的条件设置中，例如
`: ${var:=value}`


在一些shell脚本，主要是一些旧的
shell脚本中，你可能会看到冒号被用在
一行的开头来表示一个注释。但现代的
脚本总是用#来开始一个注释行，因为
这样做执行效率更高。


#### continue命令

非常类似C语言中的同名语句，这个命令使
for、while或until循环跳到下一次循环继续执行，
循环变量取循环列表中的下一个值。

continue可以带一个可选的参数以表示希望继
续执行的循环嵌套层数，也就是说你可以部分地跳
出嵌套循环。这个参数很少使用，因为它会导致脚
本程序极难理解


#### .命令

点（.）命令用于在当前shell中执行命令

通常，当一个脚本执行一条外部命令或脚本程
序时，它会创建一个==新的==环境（一个子shell），命
令将在这个新环境中执行，在命令执行完毕后，这
个==环境被丢弃==，留下==退出码返回给父shell==。但外部
的source命令和点命令（这两个命令差不多是同义
词）在执行脚本程序中列出的命令时，使用的是调
用该脚本程序的同一个shell。

因为在默认情况下，shell脚本程序会在一个新
创建的环境中执行，所以脚本程序对环境变量所作
的==任何修改都会丢失==。而点命令允许执行的脚本程
序改变当前环境。

在shell脚本程序中，点命令的作用有点类似于C
或C++语言里的#include指令。尽管它并没有从字
面意义上包含脚本，但它的确是在当前上下文中执
行命令，所以你可以使用它将变量和函数定义结合
进脚本程序。

例子
假设你有 2个包含环境设置的 文件， 它们针对不同的开发环境。
对于老的环境，使用class_set，内容如下：
```shell
#!/bin/sh

version=classic
PATH=/usr/local/old_bin:/usr/bin:bin:.
PS1="classic> "
```
对于新命令，使用 latest_set：
```shell
#!/bin/sh

version=latest
PATH=/usr/local/new_bin:/usr/bin:/bin:.
PS1="latest version> "
```

然后你就可以通过 点命令 和上面的脚本，来设置 环境。


#### echo命令

虽然，X/Open建议在现代shell中使用printf命
令，但我们还是依照常规使用echo命令来输出结尾
带有换行符的字符串

一个常见的问题是如何去掉换行符。遗憾的
是，不同版本的UNIX对这个问题有着不同的解决
方法。
`echo -n "output"`

`echo -e "output\c"`

。。需要 \c 。

第二种方法echo -e确保启用了反斜线转义字符
（如\c代表去掉换行符，\t代表制表符，\n代表回
车）的解释

如果你需要一种删除结尾换行符的
可移植方法，则可以使用外部命令tr来
删除它，但它执行的速度比较慢


#### eval

例子
```text
foo=10
x=foo
y='$'$x
echo $y
```
上面输出 $foo

```text
foo=10
x=foo
eval y='$'$x
echo $y
```
上面输出 10

eval命令有点像一个额外的$，它给出一个变量的值的值。

eval命令十分有用，它允许代码被随时生成和运
行。虽然它的确增加了脚本调试的复杂度，但它可
以让你完成使用其他方法难以或者根本无法完成的
事情。


#### exec
有2种用法
典型的用法是，将当前shell替换为一个不同的程序：
`exec wall "thanks for all the fish"`
脚本中的这个 命令会用 wall命令 替换当前的 shell。脚本程序中 exec 后面的代码都不会执行，因为 执行这个脚本的 shell 已经不存在了。

第二种用法是 修改当前 文件描述符
`exec 3< afile`
这使得文件描述符3被打开以便从文件afile中读取数据。这种用法非常少见。

#### exit n

exit命令使脚本程序以退出码n 结束运行。

如果你允许自己的脚本
程序在退出时不指定一个退出状态，那么该脚本中
最后一条被执行命令的状态将被用作为返回值

退出码0 表示成功，1-125 是可以自定义的错误码。其余数字有保留的含义
126 - 文件不可执行
127 - 命令未找到
128及以上 - 出现一个信号


#### export
export命令将作为它参数的变量导出到子shell
中，并使之在子shell中有效

在默认情况下，在一
个shell中被创建的变量在这个shell调用的下级
（子）shell中是不可用的。
export命令把自己的参
数创建为一个环境变量，而这个环境变量可以被当
前程序调用的其他脚本和程序看见

set -a或set -allexport命令将导出它之后声明的所有变量。


#### expr
expr命令将它的参数当作一个表达式来求值
最常见用法就是进行如下形式的简单数学运算
```text
x=`expr $x + 1`
```

反引号（`）字符使x取值为命令expr $x + 1的执行结果

你也可以用语法$( )替换反引号`
`x=$(expr $x + 1)`

|表达式求值|说明|
|--|--|
|expr1 | expr2|如果expr1非0，则等于expr1，否则等于expr2|
|expr1 & expr2|只要有一个表达式为0，则等于0，否则等于expr1|
|expr1 = expr2|等于|
|expr1 > expr2 (还有>=,<,<=,!=)|大于。等|
|expr1 + expr2 (还有-,*,/,%)|加法，减法等|

在较新的脚本程序中，expr命令通常被替换为更有效的$((...))语法


#### printf
只有最新版本的shell才提供printf命令。
X/Open规范建议我们应该用它来代替echo命令，
以产生格式化的输出，但看来几乎没什么人接受这
一建议。
语法：
`printf "format string" param1 param2 ...`

格式字符串与C/C++中使用的非常相似，但有
一些自己的限制。主要是不支持浮点数，因为shell
中所有的算术运算都是按照整数来进行计算的

|字符转换限定符|说明|
|--|--|
|d|十进制|
|c|字符|
|s|字符串|
|%|%字符|


#### return
return命令的作用是使函数返回。

return命令有一个数值参
数，这个参数在调用该函数的脚本程序里被看做是
该函数的返回值

#### set
set命令的作用是为shell设置参数变量。许多命
令的输出结果是以空格分隔的值，如果需要使用输
出结果中的某个域，这个命令就非常有用

。。不理解。

你还可以通过set命令和它的参数来控制shell的
执行方式。其中最常用的命令格式是`set -x`，它让
一个脚本程序跟踪显示它当前执行的命令




#### shift
shift命令把所有参数变量左移一个位置，使$2变成$1，$3变成$2，以此类推

原来$1的值将被丢弃，而$0仍将保持不变

如果调用shift命令时指
定了一个数值参数，则表示所有的参数将左移指定
的次数。$*、$@和$#等其他变量也将根据参数变
量的新安排做相应的变动。


#### trap

trap命令用于指定在接收到信号后将要釆取的
行动，我们将在本书后面的内容中详细介绍信号。

trap命令的一种常见用途是在脚本程序被中断时完
成清理工作。
历史上，shell总是用数字来代表信
号，但新的脚本程序应该使用信号的名字，它们定
义在头文件signal.h中，在使用信号名时需要省略
SIG前缀。你可以在命令提示符下输入命令`trap -l`
来查看信号编号及其关联的名称

trap有2个参数，第一个是接收到指定信号时要采取的行动，第二个是要处理的型号名。
`trap command signal`

请记住，脚本程序通常是以从上到下的顺序解
释执行的，所以你必须在你想保护的那部分代码之
前指定trap命令

|比较重要的信号|说明|
|--|--|
|HUP(1)|挂起，通常因终端掉线 或用户退出而引发|
|INT(2)|中断，通常因 Ctrl+C|
|QUIT(3)|退出，因Ctrl+\|
|ABRT(6)|中止，某些严重的执行错误|
|ALRM(14)|报警，通常用来处理超时|
|TERM(15)|终止，通常系统关机时发送|

```shell
#!/bin/sh

trap 'rm -f /tmp/my_tmp_$$' INT
echo creating file /tmp/my_tmp_$$
date > /tmp/my_tmp_$$

echo "press ctrl+c to interrupt...."
while [ -f /tmp/my_tmp_$$ ]; do
    echo File exists
    sleep 1
done

echo file no longer exists

trap INT
echo creating file /tmp/my_tmp_$$
date > /tmp/my_tmp_$$

echo "press ctrl+c to interrupt ..."
while [ -f /tmp/my_tmp_$$ ]; do
    echo file exists
    sleep 1
done

echo we never get there

exit 0
```

```
aa@yiqi:~/ch02$ ./test_trap.sh 
creating file /tmp/my_tmp_20561
press ctrl+c to interrupt....
File exists
File exists
File exists
^Cfile no longer exists
creating file /tmp/my_tmp_20561
press ctrl+c to interrupt ...
file exists
file exists
file exists
^C
aa@yiqi:~/ch02$ 
```

在这个脚本程序中，我们先用trap命令安排它
在出现一个INT（中断）信号时执行rm -
f/tmp/my_tmp_file_$$命令删除临时文件


脚本程序再次调用trap命令，这次是
指定当一个INT信号出现时不执行任何命令
这次当用户按下Ctrl+C组合键时，没有语句被
指定执行，所以采取默认处理方式，即立即终止脚
本程序。因为脚本程序被立即终止


#### unset
unset命令的作用是从环境中删除变量或函数。
这个命令不能删除shell本身定义的只读变量（如
IFS）。这个命令并不常用


#### find, grep, 正则

用于搜索文件的命令
它极其有用，但Linux初学者常常觉
得它不易使用，这不仅仅是因为它有选项、测试和
动作类型的参数，还因为其中一个参数的处理结果
可能会影响到后续参数的处理。

```text
aa@yiqi:/$ find /usr/bin -name test -print
/usr/bin/test
```
这个命令的含义是：从根目录开始查找名为
test的文件，并且输出该文件的完整路径。这非常
简单
。。根目录，太多了。所以改了。。

如果你指定-mount选项，你就可以告诉
find命令不要搜索挂载的其他文件系统的目录。
`find / -mount -name test -print`

语法
`find [path] [option] [tests] [actions]`

path部分很容易理解：你既可以使用绝对路
径，如/bin，也可以使用相对路径，如.。如果需
要，你也可以指定多个路径，如find /var /home。

|option|含义|
|--|--|
|-depth|查看目录本身之前，先搜索目录里的内容|
|-follow|跟随符号链接|
|-maxdepths N|最多搜索N层目录|
|-mount / -xdev|不搜索其他文件系统中的目录|


每种测试返回的结果有两种可能：true或false

|常用测试|含义|
|--|--|
|-atime N|文件N天前被最后访问过|
|-mtime N|文件N天前被最后修改过|
|-name pattern|文件名匹配。为确保pattern被传递给find命令而不是由 shell处理，pattern必须总是用 引号包围|
|-newer otherfile|文件比otherfile文件要新|
|-type c|文件类型是c，c是一个特殊类型，常见的是 d(目录) 和 f(普通文件)|
|-user username|文件拥有者是username|

可以用操作符 来组合测试。 大多数操作符有2种格式，短格式，长格式
|短格式操作符|长格式|含义|
|--|--|--|
|!|-not|测试取反|
|-a|-and|2个测试都必须为真|
|-o|-or|2个测试必须有一个为真|

你可以通过使用圆括号来强制测试和操作符的
优先级。由于圆括号对shell来说有其特殊的含义，
所以你还必须使用反斜线来引用圆括号


|常见动作|含义|
|--|--|
|-exec command|执行一条命令，这个动作必须使用 \; 字符来结束|
|-ok command|和-exec类似，但是执行每条命令前，需要用户确认。必须使用 \; 字符结束|
|-print|打印文件名|
|-ls|对当前文件使用 ls-dils 命令|


-exec和-ok命令将命令行上后续的参数作为它
们参数的一部分，直到被\;序列终止。
实际上，-exec和-ok命令执行的是一个嵌入式命令，所以嵌
入式命令必须以一个转义的分号结束


##### grep
general regular expression parser。

find搜索文件
grep在文件中搜索字符串。

事实上，一种非
常常见的用法是在使用find命令时，将grep作为传
递给-exec的一条命令

`grep [options] pattern [files]`

如果没有文件名，那么搜索 标准输入

|option|含义|
|--|--|
|-C|输出匹配行的数目，而不是输出匹配的行|
|-E|启用扩展表达式|
|-h|取消每个输出行的普通前缀，即匹配查询模式的文件名|
|-i|忽略大小写|
|-l|只列出包含匹配行的文件名，而不输出真正的匹配行|
|-v|对匹配模式取反|

`grep in words.txt`
`grep -c in words.txt words2.txt`
`grep -c -v in words.txt words2.txt`

|字符|含义|
|--|--|
|^|指向一行的开头|
|$|指向一行的结尾|
|*|任意单个字符|
|[]|方括号内包含一个字符范围，其中任意一个字符都可以被匹配，例如，字符范围a-e，或在字符范围前面加 ^符号 表示反向匹配|


|匹配模式|含义|
|--|--|
|[:alnum:]|字母与数字|
|[:alpha:]|字母|
|[:ascii:]|ascii字符|
|[:blank:]|空格 或制表符|
|[:cntrl:]|ascii控制字符|
|[:digit:]|数字|
|[:graph:]|非控制，非空格字符|
|[:lower:]|小写字母|
|[:print:]|可打印字符|
|[:punct:]|标点符号字符|
|[:sapce:]|空白字符，包括垂直制表符|
|[:upper:]|大写字母|
|[:xdigit:]|16进制数字|
。。用的时候是 [[:lower:]]。 单层[] 会报错。

如果指定了用于扩展匹配的-E选项，那
些用于控制匹配完成的其他字符可能会遵循正则表
达式的规则（见下表）。对于grep命令来说，
我们还需要在这些字符之前加上\字符。

|选项|含义|
|--|--|
|?|匹配是可选的，最多匹配一次|
|*|必须匹配0次或多次|
|+|必须匹配1次或多次|
|{n}|必须匹配n次|
|{n,}|必须匹配n次或以上|
|{n,m}|n<=匹配次数<=m|


查找以字母e结尾的行
`grep e$ words2.txt`

以字母a结尾的单词
`grep a[[:blank:]] words2.txt`
。。a后面是空格，就说明是a结尾。。不太对， 换行呢，句号呢？。。比较简陋啊。

Th开头的3个字母组成的单词
`grep Th.[[:space:]] words2.txt`

10个字符长的全部由小写字母组成的单词
通过a-z的字符 和一个重复10次的匹配 来完成任务。
`grep -E [a-z]\{10\} words2.txt`



### 命令的执行
你想要执行一条命令，并把该命令的输出放到一个变量中

你可以用在本章前面set命令示例中介绍的
`$(command)`语法来实现，也可以用一种比较老的
语法形式`command`，这种用法目前依然很常见
。。。command外面的翻译号不是我加的。  $()外面的是我加的。 下同

新脚本程序都应该使用`$(...)`形式
避免在使用反引号执行命令
时，处理其内部的$、`、\等字符所需要应用的相当复杂的规则。

如果在反引号`...`结构中需要用到反引
号，它就必须通过\字符进行转义

`$(command)`的结果就是其中命令的输出
这不是该命令的退出状态，而是它的字符串形
式的输出结果

`echo curreny dir is $PWD`
`echo current user is $(who)`
。。试的时候 ${PWD} 也可以打印 目录出来。
。。`$()` 是括号 圆括号 。。

```shell
whoisthere=$(who)
echo $whoisthere
```
。。直接 $whoisthere 的话，会把 用户的信息 当做 命令解释， 大概率会报错：找不到 命令 xxx


这种把命令的执行结果放到变量中的能力是非
常强大的，它使得在脚本程序中使用现有命令并捕
获其输出变得很容易。
如果需要把一条命令在标准
输出上的输出转换为一组参数，并且将它们用做为
另一个程序的参数，你会发现命令xargs可以帮你
完成这一工作

当你打算调用的命令在输出你想要的内
容之前先输出了一些空白字符，或者它输出的内容
比你想要的要多的时候也会出现问题。此时，你可
以用前面介绍的set命令来解决


#### 算术扩展
expr可以处理一些简单的算术命令，但速度很慢
更好更新的方法是使用 `$((...))`

```shell
#!/bin/sh

x=0
while [ "$x" -ne 10 ]; do
  echo $x
  x=$(($x+1))
done

exit 0
```

注意，这与x=$(...)命令不同，两对
圆括号用于算术替换，而我们之前见到
的一对圆括号用于命令的执行和获取输
出。


#### 参数扩展

最简单的参数赋值和扩展
```shell
foo=fred
echo $foo
```

但当你想在变量名后附加额外的字符时就会遇
到问题。假设你想编写一个简短的脚本程序，来处
理名为1_tmp和2_tmp的两个文件。你可能会这样
写
```shell
#!/bin/sh

for i in 1 2
do
  my_prcess $i_tmp
done
```
会报错： too few arguments

问题在于shell试图替换变量$i_tmp的值，而这
个变量其实并不存在。shell并不会认为这是一个错
误，仅仅会将它替换为一个空值，因此根本不会有
参数被传递给my_process

为了保护变量
名中类似于$i部分的扩展，你需要把i放在花括号中
```
  my_process ${i}_tmp
```


|参数扩展|说明|
|--|--|
|${param:-default}|如果param为空，则设置为default的值|
|${#param}|param的长度|
|${param%word}|从param尾部开始删除与word匹配的最小部分，然后返回剩余部分|
|${param%%word}|从param尾部开始删除与word匹配的最大部分，返回剩余|
|${param#word}|从头开始删除word匹配的最小部分，返回剩余|
|${param##word}|从头开始删除 最大匹配，返回剩余|

当处理字符串时，这些替换通常是很有用的。
特别是上表中对字符串进行部分删除的最后4个参
数扩展方法，在处理文件名和路径时非常有用

```shell
#!/bin/sh

unset foo
echo ${foo:-bar}

foo=fud
echo ${foo:-bar2}

foo=/usr/bin/X11/startx
echo ${foo#*/}
echo ${foo##*/}

bar=/usr/local/etc/local/networks
echo ${bar%local*}
echo ${bar%%local*}

exit 0
```

输出如下：
```text
bar
fud
usr/bin/X11/startx
startx
/usr/local/etc/
/usr/
```

第一条语句${foo:-bar}给出的值是bar，这是因
为在这条语句执行时foo没有值。这条语句执行
后，变量foo未发生变化，它还停留在未设置状
态。

如果这条语句是${foo:=bar}，那么
变量$foo就会被赋值。这个字符串操作
符的作用是，检查变量foo是否存在且
不为空。如果它不为空，就返回它的
值，否则就把变量foo赋值为bar并返回
这个值。

${foo:?bar}语句将在变量foo不存在
或它设置为空的情况下，输出foo:bar
并异常终止脚本程序。

${foo:+bar}语句将在变量foo存在且不
为空的情况下返回bar。选择可太多
了！

```text
{foo#*/}语句仅仅匹配并删除最左边的/（记
住，*匹配零个或多个字符）。{foo##*/}语句匹配
并删除尽可能多的字符，所以它删除最右边的/及
其前面的所有字符。

{bar%local*}语句匹配从右边起直到第一次出现
local（及跟在它后面的所有字符），而
{bar%%local*}语句则从右边起尽可能多地匹配字
符，直到遇到最靠左边的local。
```

假设你想使用cjpeg程序将一个GIF文件转换为
一个JPEG文件
`cjpeg image.git > image.jpg`

有时，你可能希望对大量文件执行这类操作，那么如何实现自动重定向呢？
```shell
#!/bin/sh

for image in *.gif
do
  cjpeg $iamge > ${image%%gif}jpg
done
```
为当前目录中的每个GIF文件创建一个对应的JPEG文件。



### here文档
在shell脚本程序中向一条命令传递输入的一种
特殊方法是使用here文档。它允许一条命令在获得
输入数据时就好像是在读取一个文件或键盘一样，
而实际上是从脚本程序中得到输入数据。

here文档以两个连续的小于号<<开始，紧跟着
一个特殊的字符序列，该序列将在文档的结尾处再
次出现。
<<是shell的标签重定向符，在这里，它
强制命令的输入是一个here文档。
这个特殊字符序
列的作用就像一个标记，它告诉shell here文档结
束的位置

最简单的例子就是给cat命令提供输入数据
```shell
#!/bin/sh

cat <<!Funky!
hello
there is a here
document
!Funky!
```

。。这个是 !Funky! 整个单词作为标记。

它可以用来调用交互式的程序，
比如一个编辑器，并向它提供一些事先定义好的输
入。但它更常见的用途是在脚本程序中输出大量的
文本，就像你在刚才的示例中看到的那样，从而可
以避免用echo语句来输出每一行


如果想按预定的方式处理一个文件中的几行，
你可以使用ed行编辑器，并在脚本程序中通过here
文档向它提供命令。
1. 一个名为 a_text_file 的文件，内容如下
```text
That is line 1
That is line 2
That is line 3
That is line 4
```

2. 通过 here 和 ed 来编辑这个文件
```shell
#!/bin/sh

ed a_text_file <<!zxc!
3
d
.,\$s/is/was/
w
q
!zxc!

exit 0
```

3. 运行脚本后，文件内容：
```text
That is line 1
That is line 2
That was line 4
```

这个脚本程序只是调用ed编辑器并向它传递命
令，先让它移动到第三行，然后删除该行，再把当
前行（因为第三行刚刚被删除了，所以当前行现在
就是原来的最后一行，即第四行）中的is替换为
was。完成这些操作的ed命令来自脚本程序中的
here文档——在标记！FunkyStuff！之间的那些内
容。

我们在here文档中用\字符来
防止$字符被shell扩展。\字符的作用是
对$进行转义，让shell知道不要尝试把
$3/is/was/扩展为它的值，而它也确实
没有值。shell把\$传递为$，再由ed编
辑器对它进行解释。


### 调试脚本程序

出现错误时，shell一般都会打印出包含错误的
行的行号。如果这个错误并不是非常明显，你可以
添加一些额外的echo语句来显示变量的内容，也可
以通过在shell中交互式地输入代码片段来对它们进
行测试。

跟踪脚本程序中复杂错误的主要方法是设置各种shell
选项。为此，你可以在调用shell时加上命令行选
项，或是使用set命令

|命令行选项|set选项|说明|
|--|--|--|
|sh -n script|set -o noexec / set -n| 只检查语法错误，不执行命令|
|sh -v script|set -o verbose / set -v| 执行命令之前 回显它们|
|sh -x script|set -o xtrace / set -x| 处理完命令后 回显它们|
|sh -u script|set -o nounset / set -u| 如果使用了 未定义的变量，就给出出错消息|

你可以用-o选项启用set命令的选项标志，用+o
选项取消设置，对简写版本也是一样的处理方法

如果想获得更好的调试效果，你
可以将xtrace标志（用来启用或关闭执行命令的跟
踪）放到脚本程序中问题代码的前后

启用 xtrace
`set -o xtrace`

关闭 xtrace
`set +x xtrace`

默认情况下，变量扩展的层次由每行代码前的
+号个数指出。你可以通过对shell配置文件中的
shell变量PS4进行设置，将+号修改为更有意义的
字符。

在shell中，你还可以通过捕获EXIT信号，从而
在脚本程序退出时查看到程序的状态。具体做法是
在脚本程序的开始处添加类似下面这样的一条语
句：
`trap 'echo Exiting: critical variable = $critical_variable' EXIT`



## 图形化工具：dialog

脚本程序只需要运行在Linux控
制台上，则可以使用dialog工具命令，它以一种非
常整洁的方式润色你的脚本程序。这个命令使用文
本模式的图形和色彩，但它的确提供了友好的面向
图形的解决方案。

一些linux发行版没有 dialog， 有 gdialog，和dialog工具非
常相似，但它依赖GNOME用户接口来显示其对话框，获得一个真正的图形化界面。
一般来说，你可以将任何使用dialog工具的
程序中对dialog工具的调用替换为对
gdialog工具的调用，从而获得程序的一个图形化版本

。。ubuntu也没有 dialog， 有 gdialog。

例子：
`dialog --msgbox "hi world" 9 18`
。。gdialog。弹出一个 消息框。
。。安装了 dialog。。 类似 DOS 的界面。太落后了。。

。。fedora 31 毛都没有。

。。p64

|类型|用于创建类型的选项|含义|
|--|--|--|
|复选框|--checklist|选项列表，每个选项都可以被单独选择|
|信息框|--infobox|显示消息后，对话框立刻返回，但不清除屏幕|
|输入框|--inputbox|允许用户输入文本|
|菜单框|--menu|允许用户选择列表中的一项|
|消息框|--msgbox|显示一条消息，和一个ok按钮|
|单选框|--radiolist|选择列表中的一项|
|文本框|--textbox|带滚动条的文本框中显示文件的内容|
|是/否框|--yesno|允许用户提问，用户可以选择yes 或 no|


如果想获得任何类型的允许文本输入或进行选
择的对话框的输出,你必须捕获标准错误流,通常
是把它指向某个临时文件以便后续处理。要想获得
Yes/No对话框的输出结果,只需查看它的退出码,
与所有设计良好的程序一样,返回0表示成功


控制对话框的大小和形状
|对话框类型|参数|
|--|--|
|--checklist|text high width list-height [tag text status]...|
|--infobox|text high width|
|--inputbox|text high width [initial string]|
|--menu|text high width menu-height [tag item]|
|--msgbox|text high width|
|--radiolist|text high width list-height [tag text status]...|
|--textbox|filename high width|
|--yesno|text high width|


所有的对话框类型都有几个相同的
参数选项。在此我们不一一列出,只介绍两个选
项:--title和-clear。前者用于指定对话框的标题,
后者用来完成清屏操作。

`dialog --title "Check me" --checklist "Pick number" 15 25 3 1 "one" "off" 2 "two" "on" 3 "three" "off"`


。。跳 2页


## 综合应用

将编写一个CD数据库应用程序

。。跳



# 3 文件操作 p79

本章将涵盖如下各种与文件相关的主题:
- 文件和设备
- 系统调用
- 库函数
- 底层文件访问
- 管理文件
- 标准I/O库
- 格式化输入和输出
- 文件和目录的维护
- 扫描目录
- 错误及其处理
- /proc文件系统
- 高级主题:fcnt1和mmap


## 3.1 Linux文件解构

在Linux中,一切(或几乎一切)都是文件。
通常程序完全可以像使用文件那样使用磁盘文件、串行口、打印机和其他设备

大多数情况下,你只需要使用5个基本的函数——open、close、read、write和ioctl。


### 目录

文件,除了本身包含的内容以外,它还会有一个名字和一些属性,即“管理信息”,包括文件的创建/修改日期和它的访问权限。
这些属性被保存在文件的==inode==(节点)中,它是文件系统中的一个特殊的数据块,它同时还包含文件的长度和文件
在磁盘上的存放位置。系统使用的是==文件的inode编号==,目录结构为文件命名仅仅是为了便于人们使用。

目录是用于保存其他文件的节点号和名字的文件。

目录文件中的每个数据项都是指向某个文件节点的链接,删除文件名就等于删除与之对应的链接
(文件的节点号可以通过ln -i命令查看)。
你可以通过使用ln命令在不同的目录中创建指向同一个文件的链接

。。ln -i 怎么是创建 hard link。。 没有节点号啊。

删除一个文件时,==实质上==是删除了该文件对应的目录项,同时指向该文件的链接数减1。

如果指向某个文件的链接数(即ls –l命令的输出中跟在访问权限后面的那个数字)变为零,
就表示该节点以及其指向的数据不再被使用,磁盘上的相应位置就会被标记为可用空间。

许多UNIX和Linux的shell都允许用户通
过波浪线符号(~)直接进入自己的家目录。要想
进入他人的家目录,就键入~user(~加用户名)
即可。
。。fedora不行，就我一个帐号，然后 ~qwer 不行。 cd ~ qwer, ~ qwer 都不行。


根目录中通常包含用于存放系统程序(二进制可执行文件)的/bin子目
录、用于存放系统配置文件的/etc子目录和用于存放系统函数库的/lib子目录
代表物理设备并为这些设备提供接口的文件按照惯例会被放在/dev子目录中。


### 文件与设备

作为超级用户,你可以使用如下命令将IDE CD-ROM驱动器挂载为一个文件

`mount -t iso9660 /dev/hdc /mnt/cdrom`
`cd /mnt/cdrom`

这个命令将CD-ROM设备(在本例中,是在系
统启动时被装载为/dev/hdc的第二个主IDE设备,
其他类型的设备对应不同的/dev条目)中的当前内
容挂载为/mnt/cdrom目录下的文件结构。然后,
你就可以像往常一样浏览CD-ROM的目录,只不过
该目录中的内容是只读的


UNIX和Linux中比较重要的设备文件有3
个:/dev/console、/dev/tty和/dev/null。


#### /dev/console
这个设备代表的是系统控制台。错误信息和诊
断信息通常会被发送到这个设备。

#### /dev/tty

如果一个进程有控制终端的话,那么特殊文
件/dev/tty就是这个控制终端(键盘和显示屏,或
键盘和窗口)的别名(逻辑设备)。例如,由系统
自动运行的进程和脚本就没有控制终端,所以它们
不能打开/dev/tty。

虽然/dev/console设备只有一个,但通
过/dev/tty却能够访问许多不同的物理设备。

#### /dev/null
/dev/null文件是空(null)设备。所有写向这
个设备的输出都将被丢弃,而读这个设备会立刻返
回一个文件尾标志,所以在cp命令里可以把它用做
==复制空文件的源文件==。人们常把不需要的输出重定
向到/dev/null。

创建空文件的另一个方法是使用`touch <filename>`命令,该命令的作用是改变文件的
修改时间。如果指定的文件不存在,就创建它,
但该命令并不会把有内容的文件变成空文件。


/dev目录中甚至还有/dev/zero设备,它可以作为创建空文件的null字节源。

设备被分为==字符设备和块设备==。两者区别在于
访问设备时是否需要一次读写一整块。一般情况
下,块设备是那些支持某些类型文件系统的设备,
例如硬盘


在本章中,我们将集中讨论磁盘文件和目录。
我们将在第5章中讨论另一种设备——用户终端


## 3.2 系统调用和设备驱动程序

只需用很少量的函数就可以对文件和设备进
行访问和控制。这些函数被称为==系统调用==,由
UNIX(和Linux)直接提供,它们也是通向操作系
统本身的接口。

操作系统的核心部分,即内核,是一组设备驱
动程序。它们是一组对系统硬件进行控制的底层接口。

为了向用户提供一个一致的接口,设备驱动程
序封装了所有与硬件相关的特性。硬件的特有功能
通常可通过==ioctl==(用于I/O控制)系统调用来提供。

/dev目录中的设备文件的用法都是相同的,它们都可以被打开、读、写和关闭。

下面是用于访问设备驱动程序的底层函数(系统调用)。
- open: 打开文件或设备。
- read: 从打开的文件或设备里读数据。
- write: 向文件或设备写数据。
- close: 关闭文件或设备。
- ioctl: 把控制信息传递给设备驱动程序。

ioctl并不需要具备可移植性。此外,每个驱动程序都定义了它自己的一组ioctl命令。

这些系统调用和其他系统调用的文档一般放在手册页的第二节。

## 库函数

针对输入输出操作==直接使用底层系统调用==的一个问题是它们的==效率非常低==。为什么呢?
- 使用系统调用会影响系统的性
能。与函数调用相比,系统调用的开销要
大些,因为在执行系统调用时,Linux必
须==从运行用户代码切换到执行内核代码,
然后再返回用户代码==。减少这种开销的一
个好方法是,在程序中尽量减少系统调用
的次数,并且让每次系统调用完成尽可能
多的工作。例如,每次读写大量的数据而
不是每次仅读写一个字符

- 硬件会限制对底层系统调用一次
所能读写的数据块大小。例如,磁带机通
常一次能写的数据块长度是10k。所以,
如果你试图写的数据量不是10k的整数
倍,磁带机还是会以10k为单位卷绕磁
带,从而在磁带上留下了空隙。


为了给设备和磁盘文件提供更高层的接口,
Linux发行版(和UNIX)提供了一系列的标准函数库。

比如提供输出缓冲功能的标准I/O库。你可以==高效地写任意长度的数据块==,库
函数则==在数据满足数据块长度要求时安排执行底层系统调用==。这就极大降低了系统调用的开销。

库函数的文档一般被放在手册页的第三节

![c03cebf35206bea6644dad68b1b2b13e.png](../_resources/c03cebf35206bea6644dad68b1b2b13e.png)


## 底层文件访问

每个运行中的程序被称为进程(process),它
有一些与之关联的文件描述符。这是一些小值整
数,你可以通过它们访问打开的文件或设备。有多
少文件描述符可用取决于系统的配置情况。当一个
程序开始运行时,它一般会有3个已经打开的文件
描述符：
- 0: 标准输入
- 1: 标准输出
- 2: 标准错误

可以通过系统调用open把其他文件描述符与
文件和设备相关联

### write

系统调用write的作用是把缓冲区buf的前
nbytes个字节写入与文件描述符fildes关联的文件
中。它返回实际写入的字节数。如果文件描述符有
错或者底层的设备驱动程序对数据块长度比较敏
感,该返回值可能会小于nbytes。如果这个函数返回0,就表示未写入任何数据;如果它返回的
是-1,就表示在write调用中出现了错误,错误代码
保存在全局变量errno里。

write system call 的原型
```C
#include <unistd.h>

size_t write(int fildes, const void *buf, size_t nbytes);
```

。。
unix std
unistd.h 是 C 和 C++ 程序设计语言中提供对 POSIX 操作系统 API 的访问功能的头文件的名称。
该头文件由 POSIX.1 标准（可移植系统接口）提出，故所有遵循该标准的操作系统和编译器均应提供该头文件（如 Unix 的所有官方版本，包括 Mac OS X、Linux 等）
。。


```C
#include <unistd.h>
#include <stdlib.h>

int main()
{
  if ((write(1, "here is some data\n", 18)) != 18)
    write(2, "error occur", 46);    // 原文的string很长，估计就是46个字符。

  exit(0);
}
```

这个程序只是在标准输出上显示一条消息。当
程序退出运行时,所有已经打开的文件描述符都会
自动关闭,所以你不需要明确地关闭它们。但处理
被缓冲的输出时,情况就不一样了。

需要再次提醒的是,write可能会报告写入的字
节比你要求的少。这并不一定是个错误。在程序
中,你需要检查errno以发现错误,然后再次调用
write写入剩余的数据。


### read system call

从与文件描述符fildes
相关联的文件里读入nbytes个字节的数据,并把它
们放到数据区buf中。它返回实际读入的字节数,
这可能会小于请求的字节数。如果read调用返回
0,就表示未读入任何数据,已到达了文件尾。同
样,如果返回的是-1,就表示read调用出现了错
误。

```C
#include <unistd.h>

size_t read(int fildes, void *buf, size_t nbytes);
```

把标准输入的前128个字节复制到标准输出。如果输入少于128个
字节,就把它们全体复制过去
```C
#include <unistd.h>
#include <stdlib.h>

int main()
{
  char buffer[128];
  int nread;

  nread = read(0, buffer, 128);
  if (nread == -1)
    write(2, "error occur", 26);
  if ((write(1, buffer, nread)) != nread)
    write(2, "errir", 27);
  exit(0);
}
```

。。nread = read(0, buffer, 12);  // 可以的，多余的变成了终端的命令。。然后命令找不到。

```shell
echo hello there | ./read

./read < draft.txt
```

第一次运行程序时，你使用echo通过管道为程
序提供输入。在第二次运行时，你通过文件重定向
输入



### open system call
创建一个新的文件描述符，你需要使用系统调用open。

```C
#include <fcntl.h>
#include <sys/types.h>
#include <sys/stat.h>

int open(const char *path, int oflags);
int open(const char *path, int oflags, mode_t mode);
```

严格来说，在遵循POSIX规范的系统上，使用
open系统调用并不需要包括头文件sys/types.h
和sys/stat.h，但在某些UNIX系统上，它们可能
是必不可少的。


open建立了一条到文件或设备的访
问路径。如果调用成功，它将返回一个可以被
read、write和其他系统调用使用的==文件描述符==

这个文件描述符是唯一的，它==不会==与任何其他运行中
的进程共享。如果两个程序同时==打开同一个文件==，
它们会分别得到==两个不同的文件描述符==。如果它们
都对文件进行写操作，那么它们会==各写各的==，它们
分别接着上次离开的位置继续往下写。它们的数据
不会交织在一起，而是彼此==互相覆盖==

两个程序对文件的读写位置（偏移值）不同。你可以通过使用
文件锁功能来防止出现冲突，我们将在第7章里介
绍该功能

准备打开的文件或设备的名字作为参数path传
递给函数，oflags参数用于指定打开文件所采取的
动作

oflags参数是通过必需文件访问模式与其他可选
模式相结合的方式来指定的。open调用必须指定表
3-1中所示的文件访问模式之一

|模式|说明|
|--|--|
|O_RDONLY|只读|
|O_WRONLY|只写|
|O_RDWR|读写|

open调用还可以在oflags参数中包括下列可选
模式的组合（用“按位或”操作）：

- O_APPEND：把写入数据追加在文件的末尾。
- O_TRUNC：把文件长度设置为零，丢弃已有的内容。
- O_CREAT：如果需要，就按参数mode中给出的访问模式创建文件。
- O_EXCL：与O_CREAT一起使
用，确保调用者创建出文件。Open调用
是一个原子操作，也就是说，它只执行一
个函数调用。使用这个可选模式可以防止
两个程序同时创建同一个文件。如果文件
已经存在，open调用将失败。

其他可以使用的oflag值请参考open调用的手册
页，它们出现在该手册页的第二节（使用man 2
open命令查看）。

open调用在成功时返回一个新的文件描述符
（它总是一个非负整数），在失败时返回-1并设置
全局变量errno来指明失败的原因

POSIX规范还标准化了一个creat调用，但它并
不常用。这个调用不仅会像我们预期的那样创建文
件，还会打开文件。它的作用相当于以oflags标志
O_CREAT|O_WRONLY|O_TRUNC来调用open。

任何一个运行中的==程序能够同时打开的文件数
是有限制的==。这个限制通常是由==limits.h头文件中的
常量OPEN_MAX==定义的，它的值随系统的不同而
不同，但POSIX要求它至少为16。

在Linux系统中，这个限制可以在系统==运行时调整==。通常一开始被设置为256。


### 访问权限的初始值

当你使用带有O_CREAT标志的open调用来创建
文件时，你必须使用有3个参数格式的open调用。
第三个参数mode是几个标志按位或后得到的，这
些标志在头文件sys/stat.h 中定义

- S_IRUSR：读权限，文件属主。
- S_IWUSR：写权限，文件属主。
- S_IXUSR：执行权限，文件属主。
- S_IRGRP：读权限，文件所属组。
- S_IWGRP：写权限，文件所属组。
- S_IXGRP：执行权限，文件所属组。
- S_IROTH：读权限，其他用户。
- S_IWOTH：写权限，其他用户。
- S_IXOTH：执行权限，其他用户。

`open("myfile", O_CREAT, S_IRUSR | S_IXOTH)`
它的作用是创建一个名为myfile的文件，文件属
主拥有读权限，其他用户拥有执行权限，且只设置
了这些权限

。。 R W X , USR GRP OTH

有几个因素会对文件的访问权限产生影响。首
先，指定的访问权限只有在创建文件时才会使用。
其次，用户掩码（由shell的umask命令设定）会影
响到被创建文件的访问权限。open调用里给出的
==mode值将与当时的用户掩码的反值做AND操作==。
举例来说，如果用户掩码被设置为001，并且指定
了S_IXOTH模式标志，那么其他用户对创建的文件
不会拥有执行权限，**因为用户掩码中指定了不允许
向其他用户提供执行权限**。因此，open和creat调
用中的标志实际上是发出设置文件访问权限的请
求，所请求的权限是否会被设置取决于当时umask
的值。

#### umask
当文件被创建时，为文件的访问权限设定一个掩码
由3个八进制数字组成的值
umask命令可以修改这个变量的值

。。终端执行 umask，返回 0002， 第一个是 特殊权限，可以暂时忽略

第一位
- 0: 允许属主任何权限
- 4: 禁止属主的读
- 2: 禁止属主写
- 1: 禁止数值执行

第二位
- 0: 允许组任何权限
- 4: 禁止组读
- 2: 禁止组写
- 1: 禁止组执行

第三位
- 0: 允许其他用户任何权限
- 4: 禁止其他用户读
- 2: 禁止其他用户写
- 1: 禁止其他用户执行

。。。？ 777不是全部权限吗。。这个的 umask 和 chmod 的 正好相反，所以上面说 掩码的反值 。。


#### close system call
使用close调用终止文件描述符fildes与其
对应文件之间的关联。文件描述符被释放并能够重
新使用。close调用成功时返回0，出错时返回-1

```C
#include <unistd.h>

int close(int fildes);
```

检查close调用的返回结果非常重要。
有的文件系统，特别是网络文件系统，可能不会
在关闭文件之前报告文件写操作中出现的错误，
这是因为在执行写操作时，数据可能未被确认写
入。


#### ioctl system call

具体细节可以参考特定设备的手册页。
POSIX规范只为流（stream）定义了ioctl调用

```C
#include <unistd.h>

int ioctl(int fildes, int cmd, ...);
```

例如，在Linux系统上对ioctl的如下调用将打开键盘上的LED灯
`ioctl(tty_fd, KDSETLED, LED_NUM|LED_CAP|LED_SCR);`


### 程序:文件复制 666

```C
#include <unistd.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>

int main()
{
    char c;
    int in, out;

    in = open("file.in", O_RDONLY);
    out = open("file.out", O_WRONLY | O_CREAT, S_IRUSR | S_IWUSR);
    while (read(in, &c, 1) == 1)
        write(out, &c, 1);
    exit(0);
}
```

`#include <unistd.h>` 行必须首先出
现，因为它定义的与POSIX规范有关的标志可能
会影响到其他的头文件


```shell
$ TIMEFORMAT="" time ./a.out 
0.00user 0.00system 0:00.00elapsed 16%CPU (0avgtext+0avgdata 1240maxresident)k0inputs+8outputs (0major+62minor)pagefaults 0swaps
```

在这台相当老的系统上，1MB的输入文件file.in被成功复制到file.out，
后者只允许属主拥有读写权限。但这次复制花费了
大约两分半钟，并且几乎消耗了所有的CPU时间。
之所以这么慢，是因为它必须完成超过两百万次的
系统调用。

```text
。。我没有用1mb的文件，就几个char。
。。试了下，因为下面的 block。。需要个对比。。

ch03$ TIMEFORMAT="" time ./a.out 
0.31user 3.71system 0:04.25elapsed 94%CPU (0avgtext+0avgdata 1240maxresident)k
0inputs+2848outputs (0major+63minor)pagefaults 0swaps

。。上面是 1.4mb的 文本文件。 大约117万个字符。。。但是看不懂啊。尝试了下直接time：

/ch03$ time ./a.out 

real	0m4.550s
user	0m0.299s
sys	0m3.711s
。。 是 4.5秒， 核心态用了 3.7， 用户态是0.3。。 其余的时间不知道干什么了。 上面的也差不多， 0.31user, 3.71system.  94%CPU . elaspsed 应该对应 real，

ch03$ time ./a.out 

real	0m3.277s
user	0m0.364s
sys	0m2.704s
。。不删除file.out 的情况， 比 删除file.out 的时候 sys少了1秒。 
。。==这一秒是不是文件扩容的时间？==
```


Linux在系统调用和文件系统性能方
面有了很大改善。一个类似的测试在Linux 2.6内核
下只需不到14秒就完成了

通过复制大一些的数据块来改善效率较
低的问题，请看下面这个改进后的程序
copy_block.c，它每次复制长度为1K的数据块

```C
#include <unistd.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>

int main()
{
    char block[1024];
    int in, out;
    int nread;

    in = open("file.in", O_RDONLY);
    out = open("file.out", O_WRONLY | O_CREAT, S_IRUSR | S_IWUSR);
    while ((nread = read(in, block, sizeof(block))) > 0) {
        write(out, block, nread);
    }
    
    exit(0);
}
```

。。
ch03$ TIMEFORMAT="" time ./a.out 
0.00user 0.01system 0:00.04elapsed 25%CPU (0avgtext+0avgdata 1240maxresident)k
0inputs+2848outputs (0major+63minor)pagefaults 0swaps

快了很多很多。 从 0.3 -> 0, 从3.71 -> 0.01, 从94% -> 25%
。。

这些时间与系统本身的性能有很大的关系，但它们确实显
示了==系统调用需要巨大的开支==，因此值得对其使用进行优化。


### 其他与文件管理有关的系统调用

#### lseek system call

lseek系统调用对文件描述符fildes的读写指针进
行设置。也就是说，你可以用它来设置文件的下一
个读写位置
读写指针既可被设置为文件中的某个
绝对位置，也可以把它设置为相对于当前位置或文
件尾的某个相对位置。

```C
#include <unistd.h>
#include <sys/types.h>

off_t lseek(int fildes, off_t offset, int whence);
```

offset 用于指定位置，
whence 定义了 offset 的用法，取值范围：
- SEEK_SET：offset是一个绝对位置。
- SEEK_CUR：offset是相对于当前位置的一个相对位置。
- SEEK_END：offset是相对于文件尾的一个相对位置。

lseek返回从==文件头到==文件指针被设置处的字节
偏移值，失败时返回-1。

参数offset的类型off_t是
一个与具体实现有关的整数类型，它定义在头文件
sys/types.h中。

#### fstat, stat, lstat

fstat系统调用返回与打开的文件描述符相关的
文件的状态信息，该信息将会写到一个buf结构
中，buf的地址以参数形式传递给fstat。

```C
#include <unistd.h>
#include <sys/stat.h>
#include <sys/types.h>

int fstat(int fildes, struct stat *buf);
int stat(const char *path, struct stat *buf);
int lstat(const char *path, struct stat *buf);
```

包含头文件sys/types.h是可选的，但
由于一些系统调用的定义针对那些某天可能会做
出调整的标准类型使用了别名，所以要在程序中
使用系统调用时，我们还是推荐将这个头文件包
含进去

stat和lstat返回的是通过文件名查到的
状态信息。它们产生相同的结果，但当文件是一个
符号链接时，lstat返回的是该符号链接本身的信
息，而stat返回的是该链接指向的文件的信息


stat结构的成员在不同的类UNIX系统上会有所
变化，但一般会包括表3-4中所示的内容
|stat成员|说明|
|--|--|
|st_mode|文件权限和文件类型信息|
|st_ino|与该文件关联的 inode|
|st_dev|保存文件的设备|
|st_uid|文件属主的uid|
|st_gid|文件属主的gid|
|st_atime|上一次被访问的时间|
|st_ctime|文件的权限，属主，组，内容 上次被改变的时间|
|st_mtime|文件内容 上一次被改变的时间|
|st_nlink|该文件上 硬链接的个数|

stat结构中返回的 st_mode 还有一些相关的宏，它们定义在 sys/stat.h 中。

文件类型标志如下所示
- S_IFBLK：文件是一个特殊的块设备。
- S_IFDIR：文件是一个目录。
- S_IFCHR：文件是一个特殊的字符设备。
- S_IFIFO：文件是一个FIFO（命名管道）。
- S_IFREG：文件是一个普通文件。
- S_FLNK：文件是一个符号链接。以下是其他模式标志。
- S_ISUID：文件设置了SUID位。
- S_ISGID：文件设置了SGID位。下面列出了用于解释st_mode标志的掩码。
- S_IFMT：文件类型。
- S_IRWXU：属主的读/写/执行权限。
- S_IRWXG：属组的读/写/执行权限。
- S_IRWXO：其他用户的读/写/执行权限。


一些用来帮助确定文件类型的宏定义。
它们只是对经过掩码处理的模式标志和相应的设备
类型标志进行比较。
- S_ISBLK：测试是否是特殊的块设备文件。
- S_ISCHR：测试是否是特殊的字符设备文件。
- S_ISDIR：测试是否是目录。
- S_ISFIFO：测试是否是FIFO。
- S_ISREG：测试是否是普通文件。
- S_ISLNK：测试是否是符号链接。

如果想测试一个文件代表的不是一个目
录，设置了属主的执行权限，并且不再有其他权
限，你可以使用如下的代码进行测试
```C
struct stat statbuf;
mode_t modes;

stat("fileName", &statbuf);
modes = statbuf.st_node;

if (!S_ISDIR(modes) && (modes & S_IRWXU) == S_IXUSR)
  // ...
```


#### dup, dup2
dup系统调用提供了一种复制文件描述符的方
法，使我们能够通过两个或者更多个不同的描述符
来访问同一个文件

dup系统调用复制文件描述符fildes，返回一个新的描述符。
dup2系统调用则是通过明确指定目标描述符来把一个文件描述符复制为另外一个。

```C
#include <unistd.h>

int dup(int fildes);
int dup2(int fildes, int fildes2);
```

。。dup 新建一个副本
。。dup2 将已存在的 修改为 其他的副本


## 3.5 标准IO库 stdio

标准I/O库（stdio）及其头文件stdio.h为底层
I/O系统调用提供了一个通用的接口。这个库现在
已经==成为ANSI标准C的一部分==，而你前面见到的系
统调用却还不是。标准I/O库提供了许多复杂的函
数用于格式化输出和扫描输入。它还负责满足设备
的==缓冲需求==。

在标准I/O库中，与底层文件描述
符对应的是流（stream），它被实现为指向结构
FILE的指针。

不要把这里的文件流与C++语言中的
输入输出流（iostream）以及AT&T UNIX
System V Release 3中引入的进程间通信中的
STREAMS模型相混淆，STEAMS模型不在本书的
讨论范围之内。要想进一步了解STREAMS，请
查阅X/Open规范（http://www.opengroup.org）和随System V
版本一起提供的AT&T STREAMS Programming
Guide（《AT&T STREAMS程序设计指南》）

在启动程序时，有==3个文件流是自动打开==的。它
们是==stdin、stdout和stderr==。它们都是在stdio.h头
文件里定义的，分别==代表着标准输入、标准输出和标准错误输出，与底层文件描述符0、1和2相对应==。

在本节里，我们将介绍标准I/O库中的下列库函数：
- fopen、fclose
- fread、fwrite
- fflush-
- fseek-
- fgetc、getc、getchar
- fputc、putc、putchar
- fgets、gets
- printf、fprintf和sprintf
- scanf、fscanf和sscanf


### fopen
类似于底层的open系统调用
主要用于文件和终端的输入输出。如果你需要对设
备进行明确的控制，那最好使用底层系统调用，因
为这可以避免用库函数带来的一些潜在问题，如输
入/输出缓冲。

```C
#include <stdio.h>

FILE *fopen(const char *filename, const char *mode);
```

fopen打开由filename参数指定的文件，并把它
与一个文件流关联起来。mode参数指定文件的打
开方式，它取下列字符串中的值
- “r”或“rb”：以只读方式打开。
- “w”或“wb”：以写方式打开，并把文件长度截短为零。
- “a”或“ab”：以写方式打开，新内容追加在文件尾。
- “r+”或“rb+”或“r+b”：以更新方式打开（读和写）。
- “w+”或“wb+”或“w+b”：以更新方式打开，并把文件长度截短为零。
- “a+”或“ab+”或“a+b”：以更新方式打开，新内容追加在文件尾

字母b表示文件是一个二进制文件而不是文本文件。

UNIX和Linux把所有文件都看作为二进制文件
mode参数，它必须是一个字符串，而不是一个字符

fopen在成功时返回一个非空的FILE*指针，失
败时返回NULL值，NULL值在头文件stdio.h里定
义。

可用的文件流数量和文件描述符一样，都是有
限制的。实际的限制是由头文件stdio.h中定义的
FOPEN_MAX来定义的，它的值至少为8，在Linux
系统中，通常是16。


### fread
从一个文件流里读取数据
数据从==文件流stream读到由ptr指向的数据缓冲区==里

。。文件流 -> 缓冲区

fread和fwrite都是对数据记录进行操作，size
参数指定每个数据记录的长度，计数器nitems给出
要传输的记录个数。它的返回值是成功读到数据缓
冲区里的记录个数（而不是字节数）。当到达文件
尾时，它的返回值可能会小于nitems，甚至可以是
零。

```C
# include <stdio.h>

size_t fread(void *ptr, size_t size, size_t nitems, FILE *stream);
```

对所有向缓冲区里写数据的标准I/O函数来说，
为数据分配空间和检查错误是程序员的责任

### fwrite

fwrite库函数与fread有相似的接口。它从指定
的数据缓冲区里取出数据记录，并把它们写到输出
流中。它的返回值是成功写入的记录个数

。。缓冲区 -> 流
。。write。

```C
#include <stdio.h>

size_t fwrite(const void *ptr, size_t size, size_t nitems, FILE *stream);
```
我们不推荐把fread和fwrite用于结
构化数据。部分原因在于用fwrite写的文件在不
同的计算机体系结构之间可能不具备可移植性。
。。序列化？

### fclose
关闭指定的文件流stream，使==所有尚未写出的数据都写出==

如果程序需要确保数据已经全部写出，就应该调用fclose函数

虽然当程序正常结束时，会自动对所有还打开
的文件流调用fclose函数，但这样做你就没有机会
检查由fclose报告的错误了

```C
#include <stdio.h>

int fclose(FILE *stream);
```

### fflush
把文件流里的所有未写出数据立刻写出

有时在调试程序时，你还可以用它来确认程序是正在写数据
而不是被挂起了。注意，调用fclose函数隐含执行
了一次flush操作，所以你不必在调用fclose之前调用fflush。

```C
#include <stdio.h>

int fflush(FILE *stream);
```

### fseek
fseek函数是与lseek系统调用对应的文件流函数。
它在文件流里为下一次读写操作指定位置。

offset和whence参数的含义和取值与前面的lseek
系统调用完全一样。但lseek返回的是一个off_t数
值，而fseek返回的是一个整数：0表示成功，-1表
示失败并设置errno指出错误

```C
#include <stdio.h>

int fseek(FILE *stream, long int offset, int whence);
```


### fgetc, getc, getchar

fgetc函数从文件流里取出下一个字节并把它作
为一个字符返回。当它到达文件尾或出现错误时，
它返回EOF。你必须通过ferror或feof来区分这两
种情况。

```C
#include <stdio.h>

int fgetc(FILE *stream);
int getc(FILE *stream);
int getchar();
```

getc函数的作用和fgetc一样，但它有可能被实
现为一个宏，如果是这样，stream参数就可能被计
算不止一次，所以它不能有副作用（例如，它不能
影响变量）。此外，你也不能保证能够使用getc的
地址作为一个函数指针

getchar函数的作用相当于getc（stdin），它
从标准输入里读取下一个字符


### fputc, putc, putchar
fputc函数把一个字符写到一个输出文件流中。
它返回写入的值，如果失败，则返回EOF

```C
#include <stdio.h>

int fputs(int c, FILE *stream);
int putc(int c, FILE *stream);
int putchar(int c);
```
putc函数的作
用也相当于fputc，但它可能被实现为一个宏。

putchar函数相当于putc（c, stdout）

putchar和getchar
都是把字符当作int类型而不是char类型来使用的。
这就允许文件尾（EOF）标识取值-1，这是一个超
出字符数字编码范围的值。


### fgets, gets

fgets函数从输入文件流stream里读取一个字符串。

```C
#include <stdio.h>

char *fgets(char *s, int n, FILE *stream);
char *gets(char *s);
```

fgets把读到的字符写到s指向的字符串里，直到
出现下面某种情况：==遇到换行符，已经传输了n-1
个字符，或者到达文件尾==。它会把遇到的换行符也
传递到接收字符串里，再加上一个表示结尾的空字
节\0。一次调用最多只能传输n-1个字符，因为它
必须把空字节加上以结束字符串。

当成功完成时，fgets返回一个指向字符串s的指
针。如果文件流已经到达文件尾，fgets会设置这个
文件流的EOF标识并返回一个空指针。如果出现读
错误，fgets返回一个空指针并设置errno以指出错
误的类型。

gets函数类似于fgets，只不过它从标准输入读
取数据并==丢弃遇到的换行符==。它在接收字符串的尾
部加上一个null字节。

gets对传输字符的个数并没有限制，
所以它可能会溢出自己的传输缓冲区。因此，你
应该==避免使用它==并用fgets来代替。许多安全问题
都可以追溯到在程序中使用了可能造成各种缓冲
区溢出的函数，gets就是一个这样的函数，所以
==千万要小心==！


## 格式化输入和输出

### printf, fprintf, sprintf

```C
#include <stdio.h>

int printf(const char *format, ...);
int sprintf(char *s, const char *format, ...);
int fprintf(FILE *stream, const char *format, ...);
```

- printf 函数把自己的输出送到标准输出。
- fprintf 函数把自己的输出送到一个指定的文件流。
- sprintf 函数把自己的输出和一个结尾空字符写到作为参数
传递过来的字符串s里。这个字符串必须足够容纳所有的输出数据。

转换控制符总是以%字符开头。下面是一个简单的例子
`printf("Some number: %d, %d, and %d\n", 1, 2, 3);`

要想输出%字符，你需要使用%%

一些常用的转换控制符:
- %d，%i：以十进制格式输出一个整数。
- %o，%x：以八进制或十六进制格式输出一个整数。
- %c：输出一个字符。
- %s：输出一个字符串。
- %f：输出一个（单精度）浮点数。
- %e：以科学计数法格式输出一个双精度浮点数。
- %g：以通用格式输出一个双精度浮点数。

整数参数的类型可以用一个可选的长度限定符来指定。
它可以是h，例如%hd表示这是一个短整数（short int），
或者l，例如%ld表示这是一个长整数（long int）


有的编译器能够对printf语句
进行检查，但并非万无一失。如果你使用的是GNU
编译器gcc，你可以在编译命令中添加-Wformat选
项以实现这一功能。

可以利用字段限定符对数据的输出格式做进一步的控制
|格式|参数|输出|
|--|--|--|
|%10s|"hello"|______hello|
|%-10s|"hello"|hello______|
|%10d|1234|______1234|
|%-10d|1234|1234______|
|%010d|1234|0000001234|
|%10.4f|12.34|______12.3400|
|%*s|10,"hello"|_____hello|

。。 _ 代表空格。

上表中的所有示例都输出到一个==10个字符宽==的
区域里。注意：==负值==的字段宽度表示数据在该字段
里以==左对齐==的格式输出。==可变字段宽度用一个星号（*）来表示==。
在这种情况下，下一个参数用来==表示字段宽==。
%字符后面以0开头表示数据前面要==用数字0填充==。
根据POSIX规范的要求，printf不对数据字段进行截断，
而是扩充数据字段以适应数据的
宽度。因此，如果你想打印一个比字段宽度长的字
符串，数据字段会加宽

printf函数返回一个整数以表明它输出的字符个
数。但在sprintf的返回值里没有算上结尾的那个
null空字符。如果发生错误，这些函数会返回一个
负值并设置errno。


### scanf, fscanf, sscanf
从一个文件流里读取数
据，并把数据值放到以指针参数形式传递过来的地
址处的变量中

它们也使用一个格式字符串来控制
输入数据的转换，它所使用的许多转换控制符都与
printf系列函数的一样。

```C
#include <stdio.h>

int scanf(const char *format, ...);
int fscanf(FILE *stream, const char *format, ...);
int sscanf(const char *s, const char *format, ...);
```

scanf函数读入的值将保存到对应的变量里去，
这些变量的类型必须正确，并且它们必须精确匹配
格式字符串。否则，内存数据就可能会遭到破坏，
从而使程序崩溃

```C
int num;
scanf("hello %d", &num);
```
这个scanf调用只有在标准输入中接下来的五个
字符匹配“Hello”的情况下才会成功。然后，如果
后面的字符构成了一个可识别的十进制数字，该数
字就将被读入并赋值给变量num。格式字符串中的
==空格用于忽略输入数据中位于转换控制符之间的各
种空白字符==（空格、制表符、换页符和换行符）。
这意味着在下面两种输入情况下，这个scanf调用
都会执行成功，并把1234放到变量num里

```
hello1234
hello     1234
```

输入的空白字符在进行数据转换时一般也会被
忽略。这意味着，格式字符串%d将持续读取输
入，忽略空格和换行符，直到找到一组数字为止。
如果预期的字符没有在输入流里出现，转换将失
败，scanf也将返回

如果不注意，这会产生问题。如果用户在输
入中应该出现一个整数的地方放的是一个非数字
字符，就可能在程序里导致一个无限循环。

一些其他的转换控制符。
- %d：读取一个十进制整数。
- %o、%x：读取一个八进制或十六进制整数。
- %f、%e、%g：读取一个浮点数。
- %c：读取一个字符（不会忽略空格）。
- %s：读取一个字符串。
- %[]：读取一个字符集合（见下面的说明）。
- %%：读取一个%字符。

。。 %[] 。。
。。下的长度限定符，  %lu, %ld

==长度限定符（h对应于短，l对应于长）==指明接收参数的长度是否比默
认情况更短或更长。这意味着，%hd表示要读入一
个短整数，%ld表示要读入一个长整数，而%lg表示要读入一个双精度浮点数

以星号（*）开头的控制符表示对应位置上的输
入数据将被忽略。这意味着，这个数据不会被保
存，因此不需要使用一个变量来接收它。

使用%c控制符从输入中读取一个字符。它
不会跳过起始的空白字符。

使用%s控制符来扫描字符串，但使用时必
须小心。它会跳过起始的空白字符，并且会在字符
串里出现的第一个空白字符处停下来

接收字符串必须有足够的空间来容纳输入流中可能的最长字符串。
较好的选择是使用一个字段限定符，或者结合使用
fgets和sscanf从输入中读入一行数据，再对它进行
扫描。这样可以避免可能被恶意用户利用的缓冲区溢出

我们使用%[]控制符读取由一个字符集合中的字
符构成的字符串。格式字符串`%[A-Z]`将读取一个由
大写字母构成的字符串。如果字符集中的第一个字
符是^，就表示将读取一个由不属于该字符集合中
的字符构成的字符串。因此，读取一个其中带空格
的字符串，并且在遇到第一个逗号时停止，你可以
用`%[^,]`。

一般来说，对scanf系列函数的评价并不高，这主要有下面3方面原因。
- 从历史来看，它们的具体实现都有漏洞。
- 它们的使用不够灵活。
- 使用它们编写的代码不容易看出究竟正在解析什么。

此外，你应尝试使用其他函数，如fread或fgets
来读取输入行，再用字符串函数把输入分解成你需
要的数据项。


### 其他流函数

stdio函数库里还有一些其他的函数使用流参数或标准流stdin、stdout和stderr，如下所示。
- fgetpos：获得文件流的当前（读写）位置。
- fsetpos：设置文件流的当前（读写）位置。
- ftell：返回文件流当前（读写）位置的偏移值。
- rewind：重置文件流里的读写位置。
- freopen：重新使用一个文件流。
- setvbuf：设置文件流的缓冲机制。
- remove：相当于unlink函数，但如果它的path参数是一个目录的话，其作用就相当于rmdir函数。

所有这些库函数在手册页的第三节中都有说明。

### 程序：文件复制3

```C
#include <stdio.h>
#include <stdlib.h>

int main()
{
    int c;
    FILE *in, *out;

    in = fopen("file.in", "r");
    out  = fopen("file.out", "w");

    while ((c = fgetc(in)) != EOF)
        fputc(c, out);
    
    exit(0);
}
```

。。
ch03$ time ./a.out

real	0m0.057s
user	0m0.001s
sys	0m0.026s
。。


这一次程序运行了0.11秒，虽然不如底层数据
块复制版本快，但比那个一次复制一个字符的版本要快得多

因为stdio库在FILE结构里使用了一
个内部缓冲区，只有在缓冲区满时才进行底层系统调用



### 文件流错误

为了表明错误，许多stdio库函数会返回一个超
出范围的值，比如空指针或EOF常数。此时，错误
由外部变量errno指出

```C
#include <errno.h>

extern int errno;
```

许多函数都可能改变errno的值。它的
值只有在函数调用失败时才有意义。你必须在函
数表明失败之后立刻对其进行检查。你应该总是
在==使用它之前将它先复制到另一个变量中==，因为
像fprintf这样的输出函数本身就可能改变errno的值。

你也可以通过检查文件流的状态来确定是否发
生了错误，或者是否到达了文件尾。

```C
#include <stdio.h>

int ferror(FILE *stream);
int feof(FILE *stream);
void clearerr(FILE *stream);
```

ferror函数测试一个文件流的错误标识，如果该
标识被设置就返回一个非零值，否则返回零。
feof函数测试一个文件流的文件尾标识，如果
该标识被设置就返回非零值，否则返回零

clearerr函数的作用是清除由stream指向的文件
流的文件尾标识和错误标识。它没有返回值，也未
定义任何错误。你可以通过使用它从文件流的错误
状态中恢复。例如，在“磁盘已满”错误解决之
后，继续开始写入文件流。


### 3.6.5 文件流和文件描述符

每个文件流都和一个底层文件描述符相关联。
你可以把底层的输入输出操作与高层的文件流操作
混合使用,但一般来说,这并不是一个明智的做
法,因为数据缓冲的后果难以预料。

```C
#include <stdio.h>

int fileno(FILE *stream);
FILE *fdopen(int fildes, const char *mode);
```

你可以通过调用fileno函数来确定文件流使用的
是哪个底层文件描述符。它返回指定文件流使用的
文件描述符,如失败就返回-1

通过调用fdopen函数在一个已打开的文
件描述符上创建一个新的文件流。

fdopen函数的操作方式与fopen函数是一样
的,只是前者的参数不是一个文件名,而是一个底
层的文件描述符。
如果你已经通过open系统调用创
建了一个文件(可能是出于为了更好地控制其访问
权限的目的),但又想通过文件流来对它进行写操
作,这个函数就很有用了。

fdopen函数的mode参数与fopen函数的完全一样,但它必须符合该文件
在最初打开时所设定的访问模式。

fdopen返回一个新的文件流,失败时返回NULL。


## 文件和目录的维护

### chmod system call
通过chmod系统调用来改变文件或目录的访问权限。

```C
#include <sys/stat.h>

int chmod(const char *path, mode_t mode);
```
参数mode的定义与open系统调用
中的一样,也是对所要求的访问权限进行按位OR操作

除非程序被赋予适当的特权,否则只有文件
的属主或超级用户可以修改它的权限。


### chown system call

改变一个文件的属主

```C
#include <sys/types.h>
#include <unistd.h>

int chown(const char *path, uid_t owner, gid_t group);
```


### unlink, link, symlink
使用unlink系统调用来删除一个文件

unlink系统调用删除一个文件的目录项并减少它
的链接数。它在成功时返回0,失败时返回-1
ni必须拥有该文件所属目录的写和执行权限

```C
#include <unistd.h>

int unlink(const char *path);
int link(const char *path1, const char *path2);
int syslink(const char *path1, const char *path2);
```

如果一个文件的链接数减少到零,并且没有进程打开它,这个文件就会被删除。

事实上,目录项总是被立刻删除,但文件所占用的空间要等到最后
一个进程(如果有的话)关闭它之后才会被系统回收

先用open创建一个文件,然后对其调用
unlink是某些程序员用来创建临时文件的技巧。
这些文件只有在被打开的时候才能被程序使用,
当程序退出并且文件关闭的时候它们就会被自动
删除掉。


ink系统调用将创建一个指向已有文件path1的
新==链接==。新目录项由path2给出。
你可以通过
symlink系统调用以类似的方式创建==符号链接==。注
意,一个文件的符号链接并==不会增加==该文件的链接
数,所以它不会像普通(硬)链接那样防止文件被删除


### mkdir, rmdir

使用mkdir和rmdir系统调用来建立和删除目录

```C
#include <sys/types.h>
#include <sys/stat.h>

int mkdir(const char *path, mode_t mode);
```
目录的权限由参数mode设定,其含义将按
open系统调用的O_CREAT选项中的有关定义设置。还要服从umask的设置情况

```C
#include <unistd.h>

int rmdir(const char *path);
```

rmdir系统调用用于删除目录,但只有在目录为空时才行。


### chdir, getcwd

切换目录
```C
#include <unistd.h>

int chdir(const char *path);
```

```C
#include <unistd.h>

char *getcwd(char *buf, size_t size);
```
getcwd函数把当前目录的名字写到给定的缓冲区buf里。如果目录名的长度超出了参数size给出的
缓冲区长度(一个ERANGE错误),它就返回NULL。如果成功,它返回指针buf。



## 扫描目录

与目录操作有关的函数在dirent.h头文件中声
明。它们使用一个名为DIR的结构作为目录操作的基础

被称为目录流的指向这个结构的指针
(DIR*)被用来完成各种目录操作,其使用方法与
用来操作普通文件的文件流(FILE *)非常相似。

### opendir
打开一个目录并建立一个目录流
如果成功,它返回一个指向DIR结构的指针
失败时返回一个空指针。

目录流使用一个底层文件描述符来访问目录本身,所以
如果打开的文件过多,opendir可能会失败。

```C
#include <sys/types.h>
#include <dirent.h>

DIR *opendir(const char *name);
```

### readdir (可以读文件信息)

。。看后续的代码，readdir 后做了一次 lstat，然后判断是否为 目录。 所以 它能读到 文件的信息。
。。不过 目录也是文件。

返回一个指针,该指针指向的结构
里保存着目录流dirp中下一个目录项的有关资料。

后续的readdir调用将返回后续的目录项。

如果发生错误或者到达目录尾,readdir将返回NULL。

```C
#include <sys/types.h>
#include <dirent.h>

struct dirent *readdir(DIR *dirp);
```

dirent结构中包含的目录项内容包括以下部分。
- ino_t d_ino:文件的inode节点号。
- char d_name[]:文件的名字。


### telldir

返回值记录着一个目录流里的当前
位置。你可以在随后的seekdir调用中利用这个值来
重置目录扫描到当前位置。

```C
#include <sys/types.h>
#include <dirent.h>

long int telldir(DIR *dirp);
```

### seekdir

设置目录流dirp的目录项
指针。loc的值用来设置指针位置,它应该通过前一
个telldir调用获得

```C
#include <sys/types.h>
#include <dirent.h>

void seekdir(DIR *dirp, long int loc);
```


### closedir
关闭一个目录流并释放与之关联的
资源。它在执行成功时返同0,发生错误时返
回-1。

```C
#include <sys/types.h>
#include <dirent.h>

int closedir(DIR *dirp);
```


### 程序：目录列表

```C
#include <unistd.h>
#include <stdio.h>
#include <dirent.h>
#include <string.h>
#include <sys/stat.h>
#include <stdlib.h>
// #include <errno.h>

void printdir(char *dir, int depth)
{
    DIR *dp;
    struct dirent *entry;
    struct stat statbuf;

    if ((dp = opendir(dir)) == NULL)        // md, dp == opendir .....
    {
        // printf("..%d, %d \n", dp, NULL);
        // printf("%d", dp == NULL);

        perror("error desc");       // success...
        fprintf(stderr, "cannot open dir: %s\n", dir);
        return;
    }

    chdir(dir);
    while ((entry = readdir(dp)) != NULL)
    {
        lstat(entry->d_name, &statbuf);
        if (S_ISDIR(statbuf.st_mode))
        {
            if (strcmp(".", entry->d_name) == 0 || strcmp("..", entry->d_name) == 0)
                continue;
            printf("%*s%s/\n", depth, " ", entry->d_name);

            printdir(entry->d_name, depth + 4);
        }
        else 
            printf("%*s%s\n", depth, " ", entry->d_name);
    }
    chdir("..");
    closedir(dp);
}

int main()
{
    printf("dic scan\n");
    // printdir("/sdwf/qwer", 0);      // perror : no such file or dir
    // printdir("/boot", 0);
    printdir("/home", 0);       // ok, but tooooo much files....
    printf("done\n");

    exit(0);
}
```

。。最开始的时候写错了， dp == opendir， 导致一直是 cannot open。。。
。。home 下文件太多了， 

```C
int main(int argc, char *argv[])
{
  char *topdir = ".";
  if (argc >= 2)
    topdir = argv[1];
  
  printf("scan %s\n", topdir);
  printdir(topdir, 0);
  printf("done\n");

  exit(0);
}
```

`./printdir2 /usr/local | more`
输出结果将分页显示,用户可以通过翻页查看
其输出。


## 3.9 错误处理

程序必须在函数报告出错之后立刻检查errno变量,因为它可能被下一
个函数调用所覆盖,即使下一个函数自身并没有出错,也可能会覆盖这个变量。
。。就像上面的代码，perror 返回 success，覆盖掉了 errno。

。。高并发怎么处理？ 这个 errno 是 每个线程一个，还是 全局只有一个？

错误代码的取值和含义都列在头文件`errno.h`里,如下所示。
- EPERM:操作不允许。
- ENOENT:文件或目录不存在。
- EINTR:系统调用被中断。
- EIO:I/O错误。
- EBUSY:设备或资源忙。
- EEXIST:文件存在。
- EINVAL:无效参数。
- EMFILE:打开的文件过多。
- ENODEV:设备不存在。
- EISDIR:是一个目录。
- ENOTDIR:不是一个目录。

两个非常有用的函数可以用来报告出现的错误,它们是strerror和perror。


### strerror
把错误代码映射为一个字符串,该字符串对发生的错误类型进行说明

```C
#include <string.h>

char *strerror(int errnum);
```

### perror
把errno变量中报告的当前错误映射到一个字符串,并把它输出到 标准错误输出流
该字符串的前面先加上字符串s(如果不为空)中给出的信息,再加上一个冒号和一个空格。

```C
#include <stdio.h>

void perror(const char *s);
```


## /proc 文件系统

Linux提供了一个特殊的文件系统procfs,它通常以/proc目录的形式呈现。该目录中包含了许多
特殊文件用来==对驱动程序和内核信息进行更高层的访问==。只要应用程序有正确的访问权限,它们就可
以通过读写这些文件来获得信息或==设置参数==。

在多数情况下,只需直接读取这些文件就可以
获得状态信息。例如,/proc/cpuinfo给出的是
cpu的详细信息

`cat /proc/cpuinfo`

/proc/meminfo和/proc/version分别
给出的是内存使用情况和内核版本信息

```text
[qwer@192 proc]$ ls *info
buddyinfo  cpuinfo  meminfo  pagetypeinfo  slabinfo  vmallocinfo  zoneinfo
[qwer@192 proc]$ ls *version
version
[qwer@192 proc]$ ls *stat*
diskstats  latency_stats  mdstat  schedstat  stat  vmstat
```

每次==读取==这些文件的内容时,它们所提供的信
息都会==及时更新==。所以再读一次meminfo文件会给
出最新的信息

可以通过特定内核函数获得更多的信息,它们位于/proc目录的子目录中。
例如,你可以通过`/proc/net/sockstat`文件获得网络套接字的使用统计

```text
[qwer@192 net]$ cat sockstat
sockets: used 1174
TCP: inuse 6 orphan 0 tw 8 alloc 13 mem 4
UDP: inuse 4 mem 16
UDPLITE: inuse 0
RAW: inuse 0
FRAG: inuse 0 memory 0
```

/proc目录中的有些条目不仅可以被读取,而且可以被修改

系统中所有运行的程序同时能打开的文件总数是Linux内核的一个参数。它的当
前值可通过读取`/proc/sys/fs/file-max`文件得到

。。
[qwer@192 fs]$ cat file-max 
9223372036854775807
。。。

如果要将系统范围的文件句柄限制增加为
80000,你只需将新的上限值写入file-max文件即可
`echo 80000 >/proc/sys/fs/file-max`


现在,你只需要知道每个进程都有一个唯一的
标识符:一个在1~32000的数字。ps命令会给出
当前正在运行进程的列表。
`ps -a`

。。
[qwer@192 proc]$ ps -a
    PID TTY          TIME CMD
   1630 tty2     00:00:00 gnome-session-b
   3559 pts/0    00:00:13 python
   9010 pts/2    00:00:00 ps
。。。

`ls -l /proc/3559`

。。很多信息。


你还可以看到启动它的命令行
以及它的shell环境。cmdline和environ文件以一系
列null终止的字符串来提供这些信息

。。
[qwer@192 proc]$ od -c /proc/3559/cmdline
0000000   p   y   t   h   o   n  \0   d   o   w   n   _   t   x   t   .
0000020   p   y  \0
0000023
。。


fd子目录提供该进程正在使用的打开的文件描
述符的信息。这个信息在确定一个程序同时打开了
多少文件时十分有用。

。。
[qwer@192 proc]$ ls /proc/3559/fd
0  1  2  3  4  5  6  7  8
。。



## 高级主题: fcntl, mmap

### fcntl 。。L
对底层文件描述符提供了更多的操纵方法。

```C
#include <fcntl.h>

int fcntl(int fildes, int cmd);
int fcntl(int fildec, int cmd, long arg);
```

利用fcntl系统调用,你可以对打开的文件描述
符执行各种操作,包括对它们进行
复制、获取和设置文件描述符标志、获取和设置文件状态标志,以
及管理建议性文件锁等。

对不同操作的选择是通过选取命令参数cmd不
同的值来实现的,其取值定义在头文件fcntl.h中

- fcntl(fildes,F_DUPFD,newfd)
  返回一个新的文件描述符,其数值等于或大于整数newfd。新文件描述符是描述符fildes的一个副本，效果可能和系统调用dup(fildes)完全一样。
- fcntl (fildes, F_GETFD)
  返回在fcntl.h头文件里定义的文件描述符标志,其中包括FD_CLOEXEC,它的作用是决定是否在成功调用了某个exec系列的系统调用之后关闭该文件描述符
- fcntl(fildes, F_SETFD, flags)
  调用用于设置文件描述符标志,通常仅用来设置FD_CLOEXEC
- fcntl(fildes, F_GETFL)和fcnt;(fildes, F_SETFL, flags)
  分别用来获取和设置文件状态标志和访问模式。可以利用在fcnt1.h头文件中定义的掩码O_ACCMODE来提取出文件的访问模式

详细信息请参考fcntl手册页的第二节,或者阅读本书的第7章,我们将在那里讨论文件锁。
。。man 2 fcntl

### mmap (内存映射，用于共享内存)
允许程序共享内存

mmap(内存映射)函数的作用是建立一段可以被两个或更多个程序读写的内存。一个程序对它所做出的修改可以被其他程序看见。

。。进程间通信？

这一功能还可以用在文件的处理上。你可以使
某个磁盘文件的全部内容看起来就像是内存中的一
个数组。如果文件由记录组成,而这些记录又能够
用C语言中的结构来描述的话,你就可以通过访问
结构数组来更新文件的内容了。

mmap函数创建一个指向一段内存区域的指
针,该内存区域与可以通过一个打开的文件描述符
访问的文件的内容相关联

```C
#include <sys/mman.h>

void *mmap(void *addr, size_t len, int prot, int flags, int fildes, off_t off);
```

通过传递off参数来改变经共享内存段访问的文件中数据的起始偏移值。打开的文件描述符
由fildes参数给出。可以访问的数据量(即内存段的长度)由len参数设置。

通过addr参数来请求使用某个特定的内
存地址。如果它的取值是==零==,结果指针就将自动分
配。这是推荐的做法,否则会降低程序的可移植
性,因为不同系统上的可用地址范围是不一样的

prot参数用于设置内存段的访问权限。它是下列常数值的按位OR结果
- PROT_READ:允许读该内存段。
- PROT_WRITE:允许写该内存段。
- PROT_EXEC:允许执行该内存段。
- PROT_NONE:该内存段不能被访问。

flags参数控制程序对该内存段的改变所造成的
影响,可以使用的选项
- MAP_PRIVATE, 内存段是私有的，对它的修改只对本进程有效
- MAP_SHARED， 把对该内存段的修改保存到磁盘文件中
- MAP_FIXED， 该内存段必须位于addr指定的地址处


#### msync
msync函数的作用是:把在该内存段的某个部
分或整段中的修改写回到被映射的文件中(或者从被映射文件里读出)。

```C
#include <sys/mman.h>

int msync(void *addr, size_t len, int flags);
```

内存段需要修改的部分由作为参数传递过来的
起始地址addr和长度len确定。flags参数控制着执
行修改的具体方式,可以使用的选项
- MS_ASYNC， 异步写
- MS_SYNC， 同步写
- MS_INVALIDATE， 从文件中读回数据


#### munmap
释放内存段

```C
#include <sys/mman.h>

int munmap(void *addr, size_t len);
```

### 代码: mmap

下面的程序mmap.c演示了如何利用mmap和数组方式的存取操作来修改一个结构化数据文件


```C

#include <unistd.h>
#include <stdio.h>
#include <sys/mman.h>
#include <fcntl.h>
#include <stdlib.h>

typedef struct {
    int integer;
    char string[24];
} RECORD;

#define NRECORDS (10)

/*
这里出了大问题：在执行 43改为143 的时候，发生了 Segmentation fault (core dumped)。
然后发现可以通过 gdb 查看 core文件。
但是我没有在 目录下发现 core文件。

`/proc/sys/kernel` 目录下有一个 core_pattern 
[qwer@192 kernel]$ cat core_pattern
|/usr/lib/systemd/systemd-coredump %P %u %g %s %t %c %h

这个位置是 core dump 的地址。 。。 这个不是，这个是 软件。。艹。

最开始的时候是认为没有生成core，所以查看了 ulimit -a， 无限的。
并且生成 core文件 需要 gcc 的时候 -g 。
。。但是第一次的时候没有 -g 也报出了这个错误。
主要是 后来才发现 core dumped， 就是说明 core 已经 dump 了。 所以第一次的时候应该也有 dump 文件。


https://www.epubit.com/articleDetails?id=Nb5fe85e1-3a32-4809-9ae4-84bb29c979a4
fedoraproject.coredumpctl

fedora 24以后， 默认 不生成 core文件。
使用 coredumpctl 这个工具，更加强。

`coredumpctl gdb`  调试最近一次 coredump，并进入 gdb 调试模式。

其他程序的调试，可以通过 `coredumpctl gdb pid` 来进行调试。

要生成core dump 的文件的话：
1. sysctl kernel.core_pattern="|/usr/libexec/abrt-hook-ccpp %s %c %p %u %g %t %P %I” 
2. vi /etc/abrt/plugins/CCpp.conf 修改MakeCompatCore = yes 
3. systemctl stop abrt-journal-core 
4. systemctl start abrt-ccpp
。。上面的4步， 我没有执行。


coredumpctl gdb
后进入了 gdb的界面，可以看到是 fseek 的问题。
并且看到了 目录： /var/lib/systemd/coredump 。里面有7,8个dump 了。。都是今天的。

gdb 的命令： ru， bt 。。 不知道干什么的。 2个都执行了下， 都是显示 fseek 的问题。
。。但是 fseek 为什么报错？？？？

报的错是 SIGSEGV ， 内存问题。

printf 了 fp， 是 0。。。
艹， records.dat,  record.dat 。。

并且试了下， gcc 不带 -g , 也可以 coredumpctl 的。

把 /var/lib/systemd/coredump 中的文件全部 rm 后，  
coredumpctl gcc 可以启动，会报错，xxxxxxxx 文件不可读， no such file.

*/


int main()
{
    RECORD record, *mapped;
    int i, f;
    FILE *fp;


    // 初始化 record.dat 。。。不过里面怎么有乱码。。应该是空位
    // fp = fopen("records.dat", "w+");
    // for (i = 0; i < NRECORDS; ++i)
    // {
    //     record.integer = i;
    //     sprintf(record.string, "record-%d", i);
    //     fwrite(&record, sizeof(record), 1, fp);
    // }
    // fclose(fp);




    // // 将第43条记录中的整数值 由43改为143。并写入到第43条记录中。
    // // 这里缩水了， 把4改成143。  ok的。
    // fp = fopen("records.dat", "r+");
    // printf("fpppppp : %d \n", fp);
    // fseek(fp, 4 * sizeof(record), SEEK_SET);
    // fread(&record, sizeof(record), 1, fp);          // 这个可以 序列化？
    // record.integer = 143;
    // sprintf(record.string, "RECORD-%d", record.integer);
    // fseek(fp, 4 * sizeof(record), SEEK_SET);
    // fwrite(&record, sizeof(record), 1, fp);
    // fclose(fp);




    // 把records 映射到 内存中，然后访问 并修改。
    f = open("records.dat", O_RDWR);
    mapped = (RECORD *) mmap(0, NRECORDS * sizeof(record), PROT_READ | PROT_WRITE, MAP_SHARED, f, 0);
    mapped[3].integer = 243;
    sprintf(mapped[3].string, "RECORD-%d", mapped[3].integer);
    
    msync((void *) mapped, NRECORDS * sizeof(record), MS_ASYNC);
    munmap((void *) mapped, NRECORDS * sizeof(record));
    close(f);

    exit(0);
}
```


在第13章中,你将学习另外一种共享内存机制:System V共享内存。






# 第4章 Linux环境

当为Linux(或UNIX和类UNIX系统)编写程序
时,你必须考虑到程序将在一个多任务环境中运
行。这意味着在同一时间会有多个程序运行,它们
共享内存、磁盘空间和CPU周期等机器资源。甚至
同一程序也会有多个实例同时运行。最重要的是,
这些程序能够互不干扰,能够了解它们的环境,并
且能正确运行,不产生冲突

- 向程序传递参数
- 环境变量
- 查看时间
- 临时文件
- 获得有关用户和主机的信息
- 生成和配置日志信息
- 了解系统各项资源的限制


## 程序参数

`int main(int argc, char *argv[])`

argc是程序参数的个数,argv是一个代表参数自身的字符串数组。

如果给shell 如下命令
`myprog left right 'and center'`

程序 myprog 将从main函数开始,main带的参数是
```text
argc: 4
argv: ["myprog", "left", "right", "and center"]
```

参数个数包括==程序名自身==,argv数组也包含程序名并将它作为第一个元素`argv[0]`

如果你用ISO/ANSIC语言编写过程序,就会对
上面的这些很熟悉。main的参数对应shell脚本里的
位置参数$0、$1等。ISO/ANSIC只规定main必须
返回int,而X/Open规范则早已给出了如上所示的
明确声明


命令行参数在向程序传递信息方面是很有用
的。例如,我们可以在一个数据库应用程序中使用
命令行参数来传递想用的数据库的名字,这样就可
以在多个数据库上使用同一个程序

许多工具程序
也使用命令行参数来改变程序的行为或设置选项。
通常,你可以使用一个以短横线(-)开头的命令
行参数来设置这些所谓的标志(flag)或开关
(switch)。例如,sort程序可以用一个开关来进
行逆向排序(与正常排序相反)

`sort -r file`

我们建议在应用程序中,所有的命令行开关都
应以一个短横线开头,其后包含单个字母或数字。
如果需要,不带后续参数的选项可以在一个短横线
后归并到一起。

建议最好能为单字
符开关增加一个更长的、更有意义的开关名,这样
你就可以使用-h或--help选项来获得帮助了。

有些程序还有一个奇怪的地方,就是用选项
+x(举例来说)执行与-x相反的功能。例如,在第
2章中,我们使用命令set-oxtrace来设置shell执行
跟踪,使用命令set+o xtrace来关闭它。

撇开风格各异的语法格式不谈,单是记住所有
这些程序选项的顺序和含义就已经非常困难了。
通常,你只有求助于-h(帮助)选项或man手册页
(如果程序员提供了的话)。你将在本章稍后看
到,==getopt==提供了对这些问题的一个优雅的解决方
案。


```C
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
    int arg;
    for (arg = 0; arg < argc; ++arg)
    {
        if (argv[arg][0] == '-')
            printf("option: %s\n", argv[arg] + 1);          // 。。char *, 所以 +1 ，就是从 第二个char开始输出。 跳过了 - 。
        else
            printf("argument %d: %s\n", arg, argv[arg]);
    }
    exit(0);
}
```

。。
[qwer@192 ch04]$ ./a.out -i -lr 'hi here' -f fer.c
argument 0: ./a.out
option: i
option: lr
argument 3: hi here
option: f
argument 5: fer.c
。。。

如果打算支持-l选项和-r选项,那么
我们就忽略了一个事实:-lr选项应该和-l -r一样处理。


X/Open规范(可以在http://opengroup.org/
上找到)定义了命令行选项的标准用法(工具语法
指南),同时定义了在C语言程序中提供命令行开
关的标准编程接口:getopt函数。

### getopt

```C
#include <unistd.h>

int getopt(int argc, char *const argv[], const char *optstring);
extern char *optarg;
extern int optind, opterr, optopt;
```

getopt函数将传递给程序的main函数的argc和
argv作为参数,同时接受一个选项指定符字符串
optstring,该字符串告诉getopt哪些选项可用,
以及它们是否有关联值。

`getopt(argc, argv, "if:lr")`

它允许几个简单的选项:-i、-l、-r和-f,其中-
f选项后要紧跟一个文件名参数。使用相同的参数,
但以不同的顺序来调用命令将改变程序的行为

。。？是 我的代码 要主动实现这个 还是什么？ 如果只是调用，那么怎么注册呢？
。。看下面的代码， 是 你告诉getopt 参数个数，参数值， 预定义的参数列表， getopt 帮你分解出来。

getopt有如下行为。
- 如果选项有一个关联值,则外部变量optarg指向这个值。
- 如果选项处理完毕,getopt返回-1,特殊参数--将使getopt停止扫描选项。
- 如果遇到一个无法识别的选项,getopt返回一个问号(?),并把它保存到外部变量optopt中。
- 如果一个选项要求有一个关联值(例如例子中的-f),但用户并未提供这个值,getopt通常将返回一个问号(?)。如果我们将选项字符串的第一个字符设置为冒号(:),那么getopt将在用户未提供值的情况下返回冒号(:)而不是问号(?)。

外部变量optind被设置为下一个待处理参数的
索引。getopt利用它来记录自己的进度。程序很少
需要对这个变量进行设置。

有些版本的getopt会在第一个非选项参数处停
下来,返回-1并设置optind的值。而其他一些版
本,如Linux提供的版本,能够处理出现在程序参
数中任意位置的选项。


#### 代码: getopt


```C
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
    int opt;

    while ((opt = getopt(argc, argv, ":if:lr")) != -1)
    {
        switch (opt)
        {
            case 'i':
            case 'l':
            case 'r':
                printf("option: %c\n", opt);
                break;
            case 'f':
                printf("filename: %s\n", optarg);
                break;
            case ':':
                printf("option need a value\n");
                break;
            case '?':
                printf("unknow: %c\n", optopt);
                break;
        }
    }
    for (; optind < argc; ++optind)
        printf("argument: %s\n", argv[optind]);
    exit(0);
}
```



。。
[qwer@192 ch04]$ ./a.out  -i -lr 'hi ehere' -f asd.c -q 
option: i
option: l
option: r
filename: asd.c
unknow: q
argument: hi ehere
。。


程序循环调用getopt对选项参数进行处
理,直到处理完毕,此时getopt返回-1。每个选项(包括未知选项和缺少关联值的选项)都有相应的处理动作。

所有选项都处理完毕后,程序像以前一样把
其余参数都打印出来,但这次是从optind位置开始


### getopt_long

GNU C函数库包含getopt的另一个版本,称作getopt_long,
它接受以双划线(--)开始的长参数。

新的长选项和原来的单字符选项可以
混合使用。只要它们能够被区分开,长选项也可以
缩写。有关联值的长选项可以按照格式--option=value作为单个参数给出

getopt_long函数比getopt多两个参数。第一个附加参数是一个结构数组,它描述了每个长选项并
告诉getopt_long如何处理它们。第二个附加参数是一个变量指针,它可以作为optind的长选项版本
使用。对于每个识别的长选项,它在长选项数组中的索引就写入该变量。在本例中,你不需要这一信
息,因此第二个附加参数是NULL。

长选项数组由一些类型为struct option的结构
组成,每个结构描述了一个长选项的行为。该数组
必须以一个包含全0的结构结尾。
长选项结构在头文件getopt.h中定义,并且该
头文件必须与常量_GNU_SOURCE一同包含进来,
该常量启用getopt_long功能

```C
struct option {
  const char *name;   // 长选项的名字，缩写也可以，只要不与其他选项混淆
  int has_arg;    // 是否带参数，0不带，1必须有一个参数，2有一个可选参数
  int *flag;    // 设置为NULL表示 当找到该选项时，getopt_long返回成员 val 中的值，否则 getopt_long 返回0，并将 val 的值写入flag 指向的变量
  int val;    // getopt_long 为该选项返回的值
}
```

```C
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

#define _GNU_SOURCE
#include <getopt.h>

int main(int argc, char *argv[])
{
    int opt;
    struct option longopts[] = {
        {"initialize", 0, NULL, 'i'},
        {"file", 1, NULL, 'f'},
        {"list", 0, NULL, 'l'},
        {"restart", 0, NULL, 'r'},
        {0,0,0,0}
    };

    while ((opt = getopt_long(argc, argv, ":if:lr", longopts, NULL)) != -1)
    {
        switch(opt)
        {
            case 'i':
            case 'l':
            case 'r':
                printf("option: %c\n", opt);
                break;
            case 'f':
                printf("filename: %s\n", optarg);
                break;
            case ':':
                printf("option need a value \n");
                break;
            case '?':
                printf("unknow option: %c\n", optopt);
                break;
        }
    }
    for (; optind < argc; ++optind)
        printf("argument: %s\n", argv[optind]);
    exit(0);
}
```


。。。
[qwer@192 ch04]$ ./a.out --initialize --list 'hi asd' --file file.c -q
option: i
option: l
filename: file.c
unknow option: q
argument: hi asd
。。



## 环境变量

使用shell的set命令来列出所有的环境变量。

。。fedora。 直接 set 是返回 很长很长的一段 shell 脚本。。
。。set --help， 也没有看到 返回全部 环境变量的 设置。

### putenv, getenv

```C
#include <stdlib.h>

char *getenv(const char *name);
int putenv(const char *string);
```

getenv函数以给定的名字搜索环境中的一个字符
串,并返回与该名字相关的值。如果请求的变量不
存在,它就返回null。
如果变量存在但无关联值,
它将运行成功并返回一个空字符串,即该字符串的
第一个字节是null

由于getenv返回的字符串是存储在getenv提供的静态空间中,所以如果想进一步
使用它,你就必须将它==复制到另一个字符串==中,以免它被后续的getenv调用所覆盖。

putenv函数以一个格式为“名字=值”的字符
串作为参数,并将该字符串加到当前环境中
如果由于可用内存不足而不能扩展环境,它会失败并返
回-1。此时,错误变量errno将被设置为ENOMEM。




```C
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char *argv[])
{
    char *var, *value;

    if (argc == 1 || argc > 3)
    {
        fprintf(stderr, "usage: environ var [value]\n");
        exit(1);
    }

    var = argv[1];
    value = getenv(var);
    if (value)
        printf("variable %s has value %s\n", var, value);
    else
        printf("variable %s has no value\n", var);
    
    if (argc == 3)
    {
        char *string;
        value = argv[2];
        string = malloc(strlen(var) + strlen(value) + 2);
        if (!string)
        {
            fprintf(stderr, "out of memory\n");
            exit(1);
        }
        strcpy(string, var);
        strcat(string, "=");
        strcat(string, value);
        printf("calling putenv with: %s\n", string);
        if (putenv(string) != 0)
        {
            fprintf(stderr, "putenv failed\n");
            free(string);
            exit(1);
        }
        value = getenv(var);
        if (value)
            printf("new value of %s is %s\n", var, value);
        else
            printf("new value of %s is null??\n", var);
    }
    exit(0);
}
```

。。
[qwer@192 ch04]$ ./a.out 
usage: environ var [value]
[qwer@192 ch04]$ ./a.out HOME
variable HOME has value /home/qwer

[qwer@192 ch04]$ ./a.out ddd
variable ddd has no value

[qwer@192 ch04]$ ./a.out ddd asd
variable ddd has no value
calling putenv with: ddd=asd
new value of ddd is asd

[qwer@192 ch04]$ ./a.out ddd
variable ddd has no value
。。。


环境仅对程序本身有效。你在程序里做
的改变不会反映到外部环境中,这是因为变量的值
不会从子进程(你的程序)传播到父进程(shell)


### 环境变量的用途 666

用户可以通过以下方式设置环境变量的值:在
默认环境中设置、通过登录shell读取的。profile文
件来设置、使用shell专用的启动文件(rc)或在
shell命令行上对变量进行设定

```shell
[qwer@192 ch04]$ asd=2314 ./a.out asd
variable asd has value 2314
```

shell将行首的变量赋值作为对环境变量的临时改变


### environ变量

环境由一组格式为“名字=值”的字符串组成。程序可以通过
environ变量直接访问这个字符串数组。environ变量的声明如下所示:

```C
#include <stdlib.h>

extern char **environ;
```


```C
#include <stdlib.h>
#include <stdio.h>

extern char **environ;      // <--------

int main()
{
    char **env = environ;

    while (*env)
    {
        printf("%s\n", *env);
        env++;
    }
    exit(0);
}
```


## 时间和日期 time_t

时间通过一个预定义的类型time_t来处理。这
是一个大到能够容纳以秒计算的日期和时间的整数
类型。在Linux系统中,它是一个长整型,与处理
时间值的函数一起定义在头文件time.h中

```C
#include <time.h>

time_t time(time_t *tloc);
```
返回的是从纪元开始至今的秒数
如果tloc不是一个空指针,time函数还会把返回值写入tloc指针指向的位置。


```C
#include <time.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

int main()
{
    int i;
    time_t the_time;

    for (i = 1; i <= 10; ++i)
    {
        the_time = time((time_t *)0);
        printf("the time is %ld\n", the_time);
        sleep(2);
    }
    exit(0);
}
```
在20秒时间内每两秒钟打印一次底层的时间值。

个程序用一个空指针参数调用time函数，返
回以秒数计算的时间和日期。程序休眠两秒后再重
复调用time函数，总共调用10次。


### difftime()

ISO/ANSIC标准委员会经过审议，并没有
规定用time_t类型来测量任意时间之间的秒数，他
们发明了一个函数difftime，该函数用来计算两个
time_t值之间的秒数并以double类型返回它。

```C
#include <time.h>

double difftime(time_t time1, time_t time2);
```

difftime函数计算两个时间值之间的差，并将
time1-time2的值作为浮点数返回

### gmtime()

把时间值转换为可读的时间和日期

gmtime函数把底层时间值分解为一个结构，该结构包含一些常用的成员
```C
#include <time.h>

struct tm *gmtime(const time_t timeval);
```

struct tm 至少包含下列成员
|tm成员|说明|
|--|--|
|int tm_sec|秒 0-61|
|int tm_min|分 0-59|
|int tm_hour|小时 0-23|
|int tm_mday|月份中的日期 1-31|
|int tm_mon|月份 0-11|
|int tm_year|从1900开始的年份|
|int tm_wday|星期几 0-6 (0是周日)|
|int tm_yday|年中的日期 0-365|
|int tm_isdst|是否夏令时|

tm_sec的范围允许临时闰秒或双闰秒

```C
#include <time.h>
#include <stdio.h>
#include <stdlib.h>

int main()
{
    struct tm *tm_ptr;
    time_t the_time;

    (void) time(&the_time);
    tm_ptr = gmtime(&the_time);

    printf("Raw time is %ld\n", the_time);
    printf("gmtime give:\n");
    printf("data: %02d-%02d-%02d\n", tm_ptr->tm_year + 1900, tm_ptr->tm_mon+1, tm_ptr->tm_mday);
    printf("time: %02d:%02d:%02d\n", tm_ptr->tm_hour, tm_ptr->tm_min, tm_ptr->tm_sec);

    exit(0);
}
```

gmtime按GMT返回时间（现在GMT被称为世界标准时间，或UTC）

### localtime()

要看当地时间，你需要使用localtime函数
```C
#include <time.h>

struct tm *localtime(const time_t *timeval);
```

如果把上面程序中的gmtime换成
localtime，再编译运行一次，你就会看到正确的时
间和日期了。

### mktime()
tm结构再转换为 原始的 time_t时间值，你可以使用mktime函数

```C
#include <time.h>

time_t mktime(struct tm *timeptr);
```
如果tm结构不能被表示为time_t值，mktime将返回-1。

### asctime(), ctime()

为了得到更“友好”的时间和日期表示，像
date命令输出的那样，你可以使用asctime函数和
ctime函数

```C
#include <time.h>

char *asctime(const struct tm *timeptr);
char *ctime(const time_t *timeval);
```

asctime返回值类似： `Sun Jun 9 12:32:45 2022\n\0`
它总是这种 26个字符的固定格式。

ctime 等价于 `asctime(localtime(timeval))`

```C
#include <time.h>
#include <stdio.h>
#include <stdlib.h>

int main()
{
    time_t timeval;

    (void) time(&timeval);
    printf("date is %s\n", ctime(&timeval));

    exit(0);
}
```
。。
date is Mon Aug 21 09:44:10 2023
。。


### strftime()

```C
#include <time.h>

size_t strftime(char *s, size_t maxsize, const char *format, struct tm *timeptr);
```

strftime函数格式化timeptr指针指向的tm结构
所表示的时间和日期，并将结果放在字符串s中。
字符串被指定（至少）maxsize个字符长。format
字符串用于控制写入字符串s的字符。与printf一
样，它包含将被传给字符串的普通字符和用于格式
化时间和日期元素的转换控制符：
- %a: 星期几的缩写
- %A: 星期几的全称
- %b: 月份缩写
- %B: 月份全称
- %c: 日期和时间
- %d: 月份中的日期，01-31
- %H: 小时，00-23
- %I: 12小时制的时间，01-12
- %j: 年份中的日期，001-366
- %m: 年份中的月份，01-12
- %M: 分钟，00-59
- %p: a.m 或 p.m
- %S: 秒，00-61
- %u: 周几，1-7 (周一为1)
- %U: 一年中的第几周，01-53 (周日是一周的第一天)
- %V: 一年中的第几周，01-53 (周一是一周的第一天)
- %w: 周几，0-6 (周日为0)
- %x: 本地格式的日期
- %X: 本地格式的时间
- %y: 年份-1900
- %Y: 年份
- %Z: 时区名
- %%: 字符%

。。12小时制 从 1点开始。。12点59结束。。


### strptime() #define _XOPEN_SOURCE
以一个代表日期和时间的字符串为参数，并创
建表示同一日期和时间的tm结构

```C
#include <time.h>

char *strptime(const char *buf, const char *format, struct tm *timeptr);
```
根据format字符串来填充tm结构的成员

trptime返回一个指针，指向转换过程处理的
最后一个字符后面的那个字符。如果碰到不能转换
的字符，转换过程就在该处停下来

```C

// #define __USE_XOPEN     // time.h 中 strptime 有 #ifdef
#define _XOPEN_SOURCE

/*
最终这个可运行，ubuntu 22

编译失败，找不到strptime，visual studio code 点开 time.h。看到
#ifdef __USE_XOPEN
extern char *strptime (const char *__restrict __s,
		       const char *__restrict __fmt, struct tm *__tp)
     __THROW;
#endif

所以有了第一行的 #define __USE_XOPEN， 不过无效。 还是找不到。

baidu后，说编译时 增加 -D _XOPEN_SOURCE
也给了图，图中是 #define _XOPEN_SOURCE。


vs code在 time.h 中点击 __USE_XOPEN 后会跳到 features.h ，可以看到如下的宏

#ifdef	_XOPEN_SOURCE
# define __USE_XOPEN	1
# if (_XOPEN_SOURCE - 0) >= 500


。。不过还是不清楚，为什么 我直接 #define __USE_XOPEN 不行。
看到了， features.h 中
#undef	__USE_XOPEN

。。太贱啦！！！

*/
#include <time.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>     // compiler need for strcpy

int main()
{
    struct tm *tm_ptr, timestruct;
    time_t the_time;
    char buf[256];
    char *result;

    (void) time(&the_time);
    tm_ptr = localtime(&the_time);
    strftime(buf, 256, "%A %d %B, %I:%S %p", tm_ptr);

    printf("strftime gives: %s\n", buf);

    strcpy(buf, "Thu 1 July 2022, 12:34 will do fine");

    printf("calling strptime with %s\n", buf);
    tm_ptr = &timestruct;

    result = strptime(buf, "%a %d %b %Y, %R", tm_ptr);
    printf("strptime consume to : %s\n", result);

    printf("strptime gives:\n");
    printf("date: %02d/%02d/%02d\n", tm_ptr->tm_year % 100, tm_ptr->tm_mon + 1, tm_ptr->tm_mday);
    printf("time: %02d:%02d\n", tm_ptr->tm_hour, tm_ptr->tm_min);

    exit(0);
}
```

。。
$ ./a.out 
strftime gives: Monday 21 August, 02:32 PM
calling strptime with Thu 1 July 2022, 12:34 will do fine
strptime consume to :  will do fine
strptime gives:
date: 22/07/01
time: 12:34
。。

转换控制符 `%R` 是strptime中对 `%H:%M` 的缩写形式

。。艹。。。
编译strftime.c时，你可能会看到编译器有一个
警告信息。这是因为GNU库在默认情况下并未声明
strptime函数。要解决这个问题，你需要明确请求
使用X/Open的标准功能，这需要在包含time.h头
文件之前加上如下一行
`#define _XOPEN_SOURCE`

。。下次把话说前面啊啊啊。



## 4.4 临时文件

很多情况下，程序会利用一些文件形式的临时
存储手段。这些临时文件可能保存着一个计算的中
间结果，也可能是关键操作前的文件备份。
例如，一个数据库应用程序在删除记录时就可能使用临时
文件。该文件收集需要保留的数据库条目，然后在
处理结束后，这个临时文件就变成新的数据库，原
来文件则被删除。

临时文件的这种用法很常见，但也有一个隐藏
的缺点。你必须确保应用程序为临时文件选取的文
件名是唯一的

用tmpnam函数可以生成一个唯一的文件名
```C
#include <stdio.h>

char *tmpnam(char *s);
```
tmpnam函数返回一个不与任何已存在文件同
名的有效文件名。如果字符串s不为空，文件名也
会写入它

对tmpnam的后续调用会覆盖存放返回
值的静态存储区，所以如果tmpnam要被多次调
用，就有必要给它传递一个字符串参数了。这个字
符串的长度至少要有L_tmpnam（通常为20）个字
符。tmpnam可以被一个程序最多调用TMP_MAX
次（至少为几千次），每次它都会返回一个不同的
文件名。

。。返回值 拿到不就可以了吗？ 为什么 必须传一个 参数？
。。难道 并发调用的话， 返回值 可能相同？


用tmpfile函数在给它命名的同时打开它。这点非
常重要，因为另一个程序可能会创建出一个与
tmpnam返回的文件名同名的文件。tmpfile函数则
完全避免了这个问题的发生
```C
#include <stdio.h>

FILE *tmpfile(void);
```

。。这个的意思 应 该 就是 本线程 tmpnam 获得了一个 不存在的文件名， 然后 另一个线程 也用 tmpnam 获得了一个 不存在的文件名， 由于 本线程 还没有创建文件，所以 另外一个线程的 tmpnam 可能和 本线程的 相同。


tmpfile函数返回一个文件流指针，它指向一个
唯一的临时文件。该文件以读写方式打开（通过
w+方式的fopen），当对它的所有引用全部关闭
时，该文件会被==自动删除==。

如果出错，tmpfile返回空指针并设置errno的值。

```C
#include <stdio.h>
#include <stdlib.h>

int main()
{
    char tmpnam2[L_tmpnam];
    char *filename;
    FILE *tmpfp;

    filename = tmpnam(tmpnam2);

    printf("tmp file name is: %s\n", filename);
    tmpfp = tmpfile();  // parameter is void
    if (tmpfp)
        printf("open a tmp file ok\n");
    else
        perror("tmpfile");
    exit(0);
}
```

。。
tmpnam_tmpfile.c:(.text+0x1e): 警告： the use of `tmpnam' is dangerous, better use `mkstemp'
。。

。。
tmp file name is: /tmp/fileZcfJgg
open a tmp file ok
。。

当编译一个使用tmpnam函数的程序时，GNU C编译器会对它的使用给出警告信息。


UNIX有另一种生成临时文件名的方式，就是使
用mktemp和mkstemp函数。Linux也支持这两个
函数
```C
#include <stdlib.h>

char *mktemp(char *template);
int mkstemp(char *template);
```

mktemp函数以给定的模板为基础创建一个唯
一的==文件名==。template参数必须是一个以6个X字符
结尾的字符串。mktemp函数用有效文件名字符的
一个唯一组合来替换这些X字符。它返回一个指向
生成的字符串的指针，如果不能生成一个唯一的名
字，它就返回一个空指针。

mkstemp函数类似于tmpfile，它也是同时创建
==并打开==一个临时文件。文件名的生成方法和
mktemp一样，但是它的返回值是一个打开的、底
层的文件描述符。

你应该在程序中使用“创建并打开”函数
tmpfile和mkstemp，而不要使用tmpnam和
mktemp。




---

## 用户信息

除了著名的init程序以外，所有的Linux程序都
是由其他程序或用户启动的

用户通常是在一个响应他们命令的shell中启动程序

当一个用户要登录进Linux系统时，他有一个用
户名和密码。一旦用户名和密码通过验证，用户就
可以进入一个shell。

用户还有一
个唯一的用户标识符UID。Linux运行的每个程序实
际上都是以某个用户的名义在运行，因此都有一个
关联的UID。

你可以对程序进行设置，让它们的运行看上去
好像是由另一个用户启动的。当一个程序的SUID
位被置位时，它的运行就好像是由该可执行文件的
属主启动的。当su命令被执行时，程序的运行就好
像它是由超级用户启动的


UID有它自己的类型：uid_t，它定义在头文
件sys/types.h中。它通常是一个小整数

一般情况下，用户的UID值都大于100。

```C
#include <sys/types.h>
#include <unistd.h>

uid_t getuid(void);
char *getlogin(void);
```

getuid函数返回程序关联的UID，它通常是启动程序的用户的UID。

getlogin函数返回与当前用户关联的登录名。

系统文件/etc/passwd包含一个用户账号数据
库。它由行组成，每行对应一个用户，包括用户
名、加密口令、用户标识符（UID）、组标识符
（GID）、全名、家目录和默认shell。

因为为了提高系统的安全性，现代的类
UNIX系统都不再使用简单的密码文件了。许多系
统，包括Linux，都有一个使用shadow密码文件的
选项，原来的密码文件中不再包含任何有用的加密
口令信息（这些信息通常存放在/etc/shadow文件
中，这是一个普通用户不能读取的文件）

一组函数来提供一个标准而又有效的获取用户信息的编程接口
```C
#include <sys/types.h>
#include <pwd.h>

struct passwd *getpwuid(uid_t uid);
struct passwd *getpwnam(const char* name);
```

|passwd结构成员|说明|
|--|--|
|char *pw_name|用户登录名|
|uid_t pw_uid|UID|
|gid_t pw_gid|GID|
|char *pw_dir|home目录|
|char *pw_gecos|用户全名|
|char *pw_shell|默认shell|

用户全名字段使用一个不同的名字。在某些系统（如Linux）上，它是
pw_gecos，而在其他系统上，它是pw_comment

出错时，它们都返回一个空指针并设置errno。



```C
#include <sys/types.h>
#include <pwd.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

int main()
{
    uid_t uid;
    gid_t gid;

    struct passwd *pw;
    uid = getuid();
    gid = getgid();

    printf("user is %s\n", getlogin());

    pw = getpwuid(uid);

    printf("name=%s, uid=%d, gid=%d, home=%s, shell=%s\n", pw->pw_name, pw->pw_uid, pw->pw_gid, pw->pw_dir, pw->pw_shell);;

    exit(0);
}
```


如果要扫描密码文件中的所有信息，你可以使
用getpwent函数。它的作用是依次取出文件数据项
```C
#include <pwd.h>
#include <sys/types.h>

void endpwent(void);
struct passwd *getpwent(void);
void setpwent(void);
```

getpwent函数依次返回每个用户的信息数据
项。当到达文件尾时，它返回一个空指针。如果已
经扫描了足够多的数据项，

你可以使用endpwent函数来终止处理过程。

setpwent函数重置读指针到密码文件的开始位置，这样下一个getpwent调用
将重新开始一个新的扫描


（有效的和实际的）用户和组标识符还可以被
其他一些不太常用的函数获得
```C
#include <sys/types.h>
#include <unistd.h>

uid_t geteuid(void);
gid_t getgid(void);
gid_t getegid(void);
int setuid(uid_t uid);
int setgid(gid_t gid);

```
只有超级用户才能调用setuid和setgid函数


---

## 主机信息

程序也可以获得运行它的计算机的有关细节。uname命令就提供这类信息
`man 2 uname`

主机信息在许多情况下都是很有用的。
你可能希望根据程序运行的机器在网络上的名字来定制程
序的行为，比如说，这台机器是学生用的还是管理员用的。
从许可证的角度考虑，你可能希望限制程
序只能在一台机器上运行。所有这些都意味着你需
要一个方法来确定程序运行在哪台机器上。

如果系统安装了网络组件，你可以通过
gethostname函数很容易地获取它的网络名：

```C
#include <unistd.h>

// 成功时，gethostbyname返回0，否则返回-1
int gethostname(char *name, size_t namelen);
```

把机器的网络名写入name字
符串。该字符串至少有namelen个字符长。成功
时，gethostbyname返回0，否则返回-1


通过uname系统调用获得关于主机的更多详细信息
```C
#include <sys/utsname.h>

// uname在成功时返回一个非负整数，否则返回-1并设置errno来指出错误。
int uname(struct utsname *name);
```

utsname 结构至少包含以下成员：
- char sysname[] : 操作系统名
- char nodename[] : 主机名
- char release[] : 系统发行级别
- char version[] : 系统版本号
- char machine[] : 硬件类型


```C
#include <sys/utsname.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

int main()
{
    char computer[256];
    struct utsname uts;

    if (gethostname(computer, 255) != 0 || uname(&uts) < 0)
    {
        fprintf(stderr, "cannot get host info");
        exit(1);
    }

    printf("computer hostname is %s\n", computer);
    printf("system is %s on %s hardware\n", uts.sysname, uts.machine);
    printf("nodename is %s\n", uts.nodename);
    printf("version is %s, %s\n", uts.release, uts.version);
    exit(0);
}
```

。。
computer hostname is yiqi
system is Linux on x86_64 hardware
nodename is yiqi
version is 5.19.0-42-generic, #43~22.04.1-Ubuntu SMP PREEMPT_DYNAMIC Fri Apr 21 16:51:08 UTC 2
。。


每台主机的唯一标识符可以通过gethostid函数获得：
```C
#include <unistd.h>

long gethostid(void);
```

gethostid函数返回与主机对应的一个唯一值

Linux，返回一个基于该机器因特网地址的值，但这对许可证管理来说还不够安全


## 4.7 日志 syslog()

通常这些日志信息被记录在系统文件中，而这
些系统文件又被保存在专用于此目的的目录中。它
可能是/usr/adm或/var/log目录。

对一个典型的Linux安装来说，
文件/var/log/messages包含所有系统信息，
/var/log/mail包含来自邮件系统的其他日志信息，
/var/log/debug可能包含调试信息。

根据你所使用Linux版本的不同，可以通过查
看/etc/syslog.conf文件或者/etc/syslogng/
syslog-ng.conf文件来检查系统配置。

。。没看到 /var/log/message， 不过 /var/log/syslog 挺6的。
。。ubuntu 22 没有 /etc/syslog.conf, /etc/syslogng

UNIX规范通过syslog函数为所有程序产生日志信息提供了一个接口：
```C
#include <syslog.h>

void syslog(int priority, const char *message, arguments...);
```

priority参数，该参数是一个严重级别与一个设施值的按位或。严重
级别控制日志信息的处理方式，设施值记录日志信息的来源。

头文件syslog.h中的设施值包括
LOG_USER（默认值）——它指出消息来自一个用
户应用程序，以及LOG_LOCAL0、LOG_LOCAL1
直到LOG_LOCAL7
按照严重等级递减：
- LOG_EMERG : 紧急情况
- LOG_ALERT : 高优先级故障，例如数据库崩溃
- LOG_CRIT : 严重错误，例如硬件故障
- LOG_ERR : 错误
- LOG_WARNING : 警告
- LOG_NOTICE : 需要注意的特殊情况
- LOG_INFO : 一般信息
- LOG_DEBUG : 调试信息

根据系统配置，
LOG_EMERG信息可能会广播给所有用户，
LOG_ALERT信息可能会EMAIL给管理员，
LOG_DEBUG信息可能会被忽略，
而其他信息则写入日志文件

当编写的程序需要使用日志记录功能时，你只需要在希望创建日志信息时调用
syslog函数即可

syslog的其他参数要根据
message字符串中printf风格的转换控制符而定。
此外，转换控制符 `%m` 可用于插入与错误变量errno
当前值对应的出错消息字符串。这对于记录错误消
息很有用

```C
#include <syslog.h>
#include <stdio.h>
#include <stdlib.h>

int main()
{
    FILE *f;
    f = fopen("not_exists", "r");
    if (!f)
        syslog(LOG_ERR | LOG_USER, "oops - %m\n");
    exit(0);
}
```
编译并运行这个程序syslog.c，你没有看到输
出，但是/var/log/messages文件尾现在有如下一行

。。我的/var/log 没有message， 但是有syslog，这个syslog文件中确实有：
Aug 22 09:49:33 yiqi a.out: oops - No such file or directory
。。
。。文件名syslog，方法也是syslog。

日志信息并未指明是哪个程序调用了日
志设施，它仅仅记录syslog被调用以记录一条信息
的事实。％m转换控制符被替换为一个错误描述，
在本例中就是“文件没有找到”。这比仅仅报告一
个原始的错误码更有用。

。。但是 我的记录了。 a.out 。。。

头文件syslog.h中还定义了一些能够改变日志记录行为的其他函数
```C
#include <syslog.h>

void closelog();
void openlog(const char *ident, int logopt, int facility);
int setlogmask(int maskpri);
```

通过调用openlog函数来改变日志信息的
表示方式。它可以设置一个字符串ident，该字符串
会添加在日志信息的前面。你可以通过它来指明是
哪个程序创建了这条信息

facility参数记录一个将
被用于后续syslog调用的默认设施值，其默认值是
LOG_USER。logopt参数对后续syslog调用的行为
进行配置，它是0个或多个表4-7中参数的按位或

|logopt参数|说明|
|--|--|
|LOG_PID|日志信息中包含进程标识符|
|LOG_CONS|如果信息不能被记录到日志文件中，就发送到控制台|
|LOG_ODELAY|在第一次调用syslog时，才打开日志设施|
|LOG_NDELAY|立刻打开日志设施，而不是等到第一次记录日志时|

openlog函数会分配并打开一个文件描述符，并
通过它来写日志。你可以调用closelog函数来关闭
它。注意，在调用syslog之前无需调用openlog，
因为syslog会根据需要自行打开日志设施

使用setlogmask函数来设置一个日志掩
码，并通过它来控制日志信息的优先级。优先级未
在日志掩码中置位的后续syslog调用都将被丢弃

用LOG_MASK（priority）为日志信息创
建一个掩码，它的作用是创建一个只包含一个优先
级的掩码。你还可以用LOG_UPTO（priority）来
创建一个由指定优先级之上的所有优先级（包括指
定优先级）构成的掩码
```C
#include <syslog.h>
# stdio.h   unistd.h  stdlib.h

int main()
{
    int logmask;
    
    openlog("logmask", LOG_PID|LOG_CONS, LOG_USER);
    syslog(LOG_INFO, "info message, pid = %d", getpod());
    syslog(LOG_DEBUG, "debug , should appear");
    logmask = setlogmask(LOG_UPTO(LOG_NOTICE));
    syslog(LOG_DEBUG, "debug, should not appear");
    exit(0);
}
```

在`/var/log/messages`文件尾，你
会看到如下信息
`info message xxxxxx`

接收调试日志信息的文件（根据日志配置而
定，通常是`/var/log/debug`，有时也可能
是`/var/log/messages`）会包含如下信息
`debug, should appear`

没有启用调试信息日志记录功能,看不到调试信息

查看系统中针对syslog或syslog-ng的文档

getpid的定义
```C
#include <sys/types.h>
#include <unistd.h>

pid_t getpid(void);
pid_t getppid(void);
```
调用进程和调用进程的父进程的进程标识符（PID）


## 资源和限制

运行的程序会受到资源限制的影响。它们可能是
- 硬件方面的物理性限制（例如内存）、
- 系统策略的限制（例如，允许使用的CPU时间）
- 具体实现的限制（如整数的长度或文件名中所允许的最大字符数）

==头文件limits.h==中定义了许多代表操作系统方面限制的显式常量

|限制常量|含义|
|--|--|
|NAME_MAX|文件名的最大字符数|
|CHAR_BIT|char的位数|
|CHAR_MAX|char的最大值|
|INT_MAX|int的最大值|

。。INT_MAX。 ^_^

==头文件sys/resource.h==提供了资源操作方面的定
义，其中包括对程序长度、==执行优先级和资源==等方
面限制进行查询和设置的函数

```C
#include <sys/resource.h>

int getpriority(int which, id_t who);
int setpriority(int which, id_t who, int priority);
int getrlimit(int resource, struct rlimit *r_limit);
int getrlimit(int resource, const struct rlimit *r_limit);
int getrusage(int who, struct rusage *r_usage);
```
id_t是一个整数类型，它用于用户和组标识符。

在头文件sys/resource.h中定义的rusage结构用来
确定当前程序已耗费了多少CPU时间，它至少包含表两个成员:
- struct timeval ru_utime: 使用的用户时间
- struct timeval ru_stime: 使用的系统时间

timeval结构定义在头文件sys/time.h中，它包含成员tv_sec和tv_usec，分别代表秒和微秒

一个程序耗费的CPU时间可分为==用户时间（程
序执行自身的指令所耗费的时间）和系统时间（操
作系统为程序执行所耗费的时间，即执行输入输出
操作的系统调用或其他系统函数所花费的时间）==


getrusage函数将CPU时间信息写入参数r_usage指向的rusage结构中。参数who是下列常量之一
- RUSAGE_SELF: 仅返回当前程序的使用信息
- RUSAGE_CHILDREN: 还包括子进程的使用信息

将在第11章讨论子进程和任务优先级

应用程序可以用getpriority和setpriority函数确定和更改它们（和其他程序）的优先级
被优先级函数检查或更改的进程可以用进程标识符、组标识符或用户来确定
which参数指定了对待who参数的方式
|which参数|说明|
|--|--|
|PRIO_PROCESS|who参数是进程标识符|
|PRIO_PGRP|who参数是进程组|
|PRIO_USER|who参数是用户标识符|

确定当前进程的优先级：
`priority = getpriority(PRIO_PROCESS, getpid());`

默认的优先级是0。正数优先级用于后台任务，
它们只在没有其他更高优先级的任务准备运行时才
执行。负数优先级使一个程序运行更频繁，获得更
多的CPU可用时间。优先级的有效范围是-20～
+20。这很容易让人困惑，因为==数值越高，执行优
先级反而越低==。

getpriority在成功时返回一个有效的优先级，
失败时返回-1并设置errno变量。因为-1本身是一
个有效的优先级，所以在调用getpriority之前应将
errno设置为0，并在函数返回时检查它是否仍为
0。
setpriority在成功返回0，否则返回-1。


系统资源方面的限制可以通过getrlimit和
setrlimit来读取和设置。这两个函数都利用一个通
用结构rlimit来描述资源限制。该结构定义在头文件
sys/resource.h中，
- rlim_t rlim_cur: 当前的软限制
- rlim_t rlim_max: 硬限制

类型rlim_t是一个整数类型，它用来描述资源级别。
一般来说，软限制是一个建议性的最好不要超越的限制，如果超越可能会导致库函数返回错误。
硬限制如果被超越，则可能会导致系统通过发送信号的方式来终止程序的运行。


有许多系统资源可以进行限制，它们由rlimit函
数中的resource参数指定，并在头文件
sys/resource.h中定义，
- RLIMIT_CORE: 内存转储(core dump) 文件的大小限制(单位：字节)
- RLIMIT_CPU: CPU时间限制(秒)
- RLIMIT_DATA: 数据段限制(字节)
- RLIMIT_FSIZE: 文件大小限制(字节)
- RLIMIT_NOFILE: 可以打开的文件数限制
- RLIMIT_STACK: 栈大小限制(字节)
- RLIMIT_AS: 地址空间(栈和数据)限制(字节)



```C
#include <sys/types.h>
#include <sys/resource.h>
#include <sys/time.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <math.h>

// work 函数将一个字符串写入一个临时文件 10000次，然后做一些算术运算以产生CPU负载
void work()
{
    FILE *f;
    int i;
    double x = 4.5;

    f = tmpfile();
    for (i = 0; i < 10000; ++i)
    {
        fprintf(f, "do sth output\n");
        if (ferror(f)) {
            fprintf(stderr, "error writing to tmp file\n");
            exit(1);
        }
    }
    for (i = 0; i < 1000000; ++i)
    {
        x = log((double) x*x + 3.21);
    }
}

int main()
{
    struct rusage r_usage;
    struct rlimit r_limit;
    int priority;

    // 调用work，通过 getrusage 获得消耗的CPU 时间
    work();
    getrusage(RUSAGE_SELF, &r_usage);

    printf("cpu usage: user: %ld.%06ld, system: %ld.%06ld\n", 
        r_usage.ru_utime.tv_sec, r_usage.ru_utime.tv_usec,
        r_usage.ru_stime.tv_sec, r_usage.ru_stime.tv_usec
    );


    // 设置 优先级和 文件大小限制
    priority = getpriority(PRIO_PROCESS, getpid());
    printf("now priority: %d\n", priority);

    getrlimit(RLIMIT_FSIZE, &r_limit);
    printf("now file size limit: soft: %ld, hard: %ld\n", r_limit.rlim_cur, r_limit.rlim_max);


    // setrlimit 设置文件大小限制，并再次调用work，这次work 会失败，因为它试图创建一个太大的文件
    r_limit.rlim_cur = 2048;
    r_limit.rlim_max = 4096;
    printf("set 2k to file size limit\n");
    setrlimit(RLIMIT_FSIZE, &r_limit);

    work();
    exit(0);
}
```

。。
$ gcc resource_limit.c -lm
$ ./a.out 
cpu usage: user: 0.021005, system: 0.000000
now priority: 0
now file size limit: soft: -1, hard: -1
set 2k to file size limit
文件大小超出限制 (核心已转储)
。。

。。
编译器找不到log的具体实现.虽然我们包括了正确的头文件,但是我们在编译的时候还是要连接确定的库.在Linux下,为了使用数学函数,我们必须和数学库连接,为此我们要加入 -lm 选项。
。。


可以用nice命令启动程序来改变程序的优先
级。这里，你看到程序的优先级变成了+10。因
此，程序的执行时间变长了

。。
$ nice ./a.out 
cpu usage: user: 0.018271, system: 0.003654
now priority: 10
now file size limit: soft: -1, hard: -1
set 2k to file size limit
文件大小超出限制 (核心已转储)

$ ./a.out 
cpu usage: user: 0.022341, system: 0.000000
now priority: 0
now file size limit: soft: -1, hard: -1
set 2k to file size limit
文件大小超出限制 (核心已转储)
。。
没有看到明显的 nice 后，速度下降。 试了很多次， 直接运行是 user 0.02，system 0.    nice 后，user 0.18， system 0.003
。。是的， 一个system 永远0， 一个 system永远有值。



# 5 终端 (跳)

在本章为第7章做大量的底层准备工作。第6章是基于curses
的，但它并不是古老的咒语，而是一个函数库，
它提供了控制终端屏幕显示的高级代码

学习以下内容：
- 对终端进行读写
- 终端驱动程序和通用终端接口
- termios
- 终端的输出和terminfo
- 检测键盘击键动作


## 对终端进行读写

当一个程序在命令提示符
中被调用时，shell负责将标准输入和标准输出流连
接到你的程序。通过在程序中使用getchar和printf
函数，你可以很容易地对这些默认流进行读写，实
现程序和用户之间的交互

# 6 使用curses函数库管理基于文本的屏幕 (跳)

本章将介绍以下几方面的内容：
- curses函数库的使用
- curses函数库的概念
- 基本的输入输出控制
- 多窗口的使用
- keypad模式的使用
- 彩色显示

在本章最后，我们将用C语言重新实现CD唱片
管理程序，将其作为对目前为止所学知识的一个总结


# 7 数据管理

首先介绍资源分配的管理方式，
然后介绍如何对可能被多个用户同时访问的文件进行处理，
最后介绍一个Linux系统提供的工具，我们可以利用它来克服以普通文件作为数据存贮介质时受到的限制

将这些问题归纳为数据管理的3个方面。
- ==动态内存管理==：可以做什么以及Linux不允许做什么。
- ==文件锁定==：协调锁、共享文件的锁定区域和避免死锁。
- ==dbm数据库==：一个大多数Linux系统都提供的、基本的、不基于SQL的数据库函数库。


## 内存管理
在过去，人们还认为256MB的内存已经足够了。但现
在，即使对桌面系统，2GB的内存也已经是其最低
要求了，服务器系统通常需要的内存量就更多了

。。2g 也没人要了。。

UNIX风格的操
作系统就以一种非常干净的方式来管理内存，因为
Linux系统实现了X/Open规范，所以它也继承了这
一特点。除了一些特殊的嵌入式应用程序以外，
==Linux程序决不允许直接访问物理内存==。也许应用
程序看起来好像可以这样做，但应用程序看到的只
是一个精心控制的假象而已。

Linux为应用程序提供了一个简洁的视图，它能
反映一个巨大的可直接寻址的内存空间。此外，
Linux还提供了内存保护机制，它避免了不同的应
用程序之间的互相干扰。如果机器被正确配置并且
有足够的交换空间，Linux还允许应用程序访问==比
实际物理内存更大的内存空间==。


### 简单的内存分配 malloc

```C
#include <stdlib.h>

void *malloc(size_t size);
```

遵循X/Open规范的Linux与一些UNIX
系统不同，它不要求包含malloc.h头文件。此
外，用来指定待分配内存字节数量的参数size不
是一个简单的整型，虽然它通常是一个无符号整型。


```C
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>

#define ONE_MB (1024 * 1024)

int main()
{
    char *some_memory;
    int mb = ONE_MB;
    int exit_code = EXIT_FAILURE;

// #ifndef __SIZE_TYPE__
// #define __SIZE_TYPE__ long unsigned int
// #endif
// #if !(defined (__GNUG__) && defined (size_t))
// typedef __SIZE_TYPE__ size_t;

    some_memory = (char *) malloc((size_t) mb);
    if (some_memory != NULL)
    {
        sprintf(some_memory, "hi world\n");
        printf("mem: %s", some_memory);
        exit_code = EXIT_SUCCESS;
    }
    exit(exit_code);
}
```

由于malloc函数返回的是一个void*指
针，因此需要通过类型转换，将其转换至你需要的
char*类型指针。malloc函数可以保证其返回的内
存是==地址对齐==的，所以它可以被==转换为任何类型==的指针


### 7.1.2 分配大量内存

这个程序将请求系统分配比机器本身所拥有的物
理内存更多的内存


```C
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>

#define ONE_MB (1024 * 1024)
#define ALL_MEM (1024 * 5)          // 按照实际调整，本次虚拟机分配了5g内存

int main()
{
    char *some_mem;
    size_t allocate_size = ONE_MB;
    int megs_obtained = 0;

    while (megs_obtained < (ALL_MEM * 2))
    {
        some_mem = (char *) malloc((size_t) allocate_size);
        if (some_mem != NULL)
        {
            ++megs_obtained;
            sprintf(some_mem, "hi world");
            printf("%s - now allocated % d MB\n", some_mem, megs_obtained);
        }
        else
        {
            exit(EXIT_FAILURE);
        }
    }
    exit(EXIT_SUCCESS);
}
```

。。
hi world - now allocated  10237 MB
hi world - now allocated  10238 MB
hi world - now allocated  10239 MB
hi world - now allocated  10240 MB
。。。10g



可用内存
每次只分配1K字节的内存并在获得的每个内存块上写入数据

```C

#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>

#define ONE_K (1024)

int main()
{
    char *some_mem;
    int allo_size = ONE_K;
    int megs = 0;
    int ks = 0;

    // while (true)     // need <stdbool.h> 。。。。。unbelievable.
    while (1)
    {
        for (ks = 0; ks < 1024; ++ks)
        {
            some_mem = (char *) malloc(allo_size);
            if (some_mem == NULL)
                exit(EXIT_FAILURE);
            
            sprintf(some_mem, "hi world");
        }
        ++megs;
        printf("now allocated %d MB\n", megs);
    }
    exit(EXIT_SUCCESS);
}
```


。。卡住了。 2733mb 后 虚拟机卡住了。。 但是上面 申请了10g啊。
now allocated 4043 MB
now allocated 4044 MB
已杀死
。。最终4044 后 被kill 了。。


这个程序还是分配了大大超出机器物理内存容量的内存
。。可能是我是虚拟机的缘故吧。

当物理内存耗尽时，它便会开始使用所谓的交换空间（swap
space）。在Linux系统中，交换空间是一个在安装系统时分配的独立的磁盘区域

Linux实现了一个“按需换页的虚拟内存系统”。用户程序看到的所有内存
全是虚拟的，也就是说，它并不真正存在于程序使用的物理地址上

Linux将所有的内存都以页为单位进行划分，通常每一页的大小为4096字节。每
当程序试图访问内存时，就会发生==虚拟内存到物理
内存的转换==，转换的具体实现和耗费的时间取决于
你所使用的特定硬件情况。当所访问的内存在物理
上并不存在时，就会产生一个==页面错误==并将控制权交给内核。

Linux内核会对访问的内存地址进行检查，如果
这个地址对于程序来说是==合法==可用的，内核就会确
定需要向程序提供哪一个物理内存页面。然后，如
果该页面之前从未被写入过，内核就直接分配它，
如果它已经被保存在硬盘的交换空间上，内核就读
取包含数据的内存页面到物理内存（可能需要把一
个已有页面从内存中移出到硬盘）。接着，在完成
虚拟内存地址到物理地址的映射之后，内核允许用
户程序继续运行。Linux应用程序并不需要操心这
一过程，因为所有的具体实现都已隐藏在内核中
了。

最终，当应用程序耗尽所有的物理内存和交换
空间，或者当最大栈长度被超过时，内核将拒绝此
后的内存请求，并可能提前终止程序的运行

在使用动态分配内存的C语言程序中，一个最
常见的问题是试图在一个已分配的内存块之后写数
据。在发生这种情况时，程序可能并不会立即终
止，但你可能已覆盖了malloc库例程内部使用的一
些数据。
通常这可能会导致后续的malloc调用失败，但
这并不是因为没有足够的内存可以分配，而是因为
内存的结构已经被破坏。追踪这类问题非常困难，
在程序里越早检测到这类错误，就越有机会找到其
原因。在本书的第10章介绍调试和优化的时候，我
们将讨论一些有助于追踪这类内存问题的工具。



### 7.1.3 滥用内存


```C
#include <stdlib.h>

#define ONE_K (1024)

int main()
{
    char *some_mem;
    char *scan_ptr;

    some_mem = (char *) malloc(ONE_K);
    if (some_mem == NULL)
        exit(EXIT_FAILURE);
    
    scan_ptr = some_mem;
    while (1)
    {
        *scan_ptr = '\0';
        ++scan_ptr;
    }
    exit(EXIT_SUCCESS);
}
```

。。
[qwer@192 ch07]$ gcc bad_use_malloc_4.c 
[qwer@192 ch07]$ ./a.out 
Segmentation fault (core dumped)
。。

。。 增加了一个 计数器
this is 134495
this is 134496
Segmentation fault (core dumped)
。。
。。申请1k， 1024byte， 怎么 ++ 了 13万次。 早就越界了啊。  我这里应该是使用了 130kb
。。不过也是，C C++ 本来就不检查内存是否越界。
。。不过不知道，kill掉进程的 条件是什么。


Linux内存管理系统能保护系统的其他部分免受
这种内存滥用的影响。为了确保一个行为恶劣的程
序(如本例)无法破化任何其他程序,Linux会终
止其运行。

。。应该是 影响到其他进程了，就被kill 了。


### 空指针


```C
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>

int main()
{
    char *mem = (char *) 0;

    printf("a read : %s\n", mem);

    sprintf(mem, "write\n");
    exit(EXIT_SUCCESS);
}
```


。。
[qwer@192 ch07]$ gcc visit_nullpointer_5.c 
[qwer@192 ch07]$ ./a.out 
a read : (null)
Segmentation fault (core dumped)
。。



```C
char z = *(const char*) 0;      // 会 segment fault
```

你尝试直接从零地址处读取数据,而且
这次在你和内核之间并没有GNU的libc库存在,于
是,程序被终止了


### 释放内存 free()

通过==free==调用,来把不用的内存释放给malloc内存管理器

这样做可以将分散的内存块重新合并到一起,并由malloc函
数库而不是应用程序来管理它。

如果一个运行中的程序(进程)自己使用并释放内存,则这些自由内
存==实际上仍然处于被分配给该进程的状态==

Linux将程序员使用的内存块作为一个物理页
面集来管理,通常内存中的每个页面为4K字节。但
如果一个内存页面未被使用,Linux内存管理器就
可以将其从物理内存置换到交换空间中(术语叫换
页),从而减轻它对资源使用的影响。如果程序试
图访问位于已置换到交换空间中的内存页中的数
据,那么Linux会短暂地暂停程序,将内存页从交
换空间再次置换到物理内存,然后允许程序继续运
行,就像数据一直存在于内存中一样

```C
#include <stdlib.h>

void free(void *ptr_to memory);
```

调用free时使用的指针参数必须是指向由
malloc、calloc或realloc调用所分配的内存


一旦调用free释放了一块
内存,它就不再属于这个进程。它将由
malloc函数库负责管理。在对一块内存
调用free之后,就绝不能再对其进行读
写操作了。

。。和上面的说法反了啊， 上面是 free 后，依然属于进程。  这里说不属于。
。。baidu 有的说可以， 有的说不可以。。 文心一言：不能被其他进程获取。
。。bing搜： free() will let memory back to os or process
www.cplusplus-soup.com/2010/01/freedelete-not-returning-memory-to-os.html
When you call free () or delete (), it will NOT really release any memory back to OS. Instead, that memory is kept with the same process until it is terminated. 
。。


### 其他内存分配函数

```C
#include <stdlib.h>

void *calloc(size_t number_of_elements, size_t element_size);
void *realloc(void *existing_memory, size_t new_size);
```

realloc函数可能不得不移动数据,因此
特别重要的一点是,你要确保一旦内存被重新分配
之后,你必须使用新的指针而不是使用realloc调用
前的那个指针去访问内存。

如果realloc无法调整内存块大小的话,它会返回一个null指针
所以要避免如下：
```C
my_ptr = malloc(1000);

my_ptr = realloc(my_ptr, 100000000000000);
```

如果realloc调用失败,它将返回一个空指针,
my_ptr就将指向null,而先前用malloc分配的内存
将无法再通过my_ptr进行访问。
因此,在释放老内
存块之前,==最好的方法是先用malloc请求一块新内
存,再通过memcpy调用把数据从老内存块复制到
新的内存块==。


## 文件锁定

程序经常需要共享数据,而这
通常是通过文件来实现的。因此,对于这些程序来
说,建立某种控制文件的方式就非常重要了

Linux提供了多种特性来实现文件锁定。
- 其中最简单的方法就是以原子操作的方式创建锁文件,所谓“原子操作”就是在创建锁文件时,系统将不允许任何其他的事情发生。
- 第二种方法更高级一些,它允许程序锁定文件的一部分,从而可以独享对这一部分内容的访问。有两种不同的方式可以实现第二种形式的文件锁定。我们将只对其中的一种做详细介绍,因为两种方式非常相似——第二种方式只不过是程序接口稍微不同而已。


### 创建锁文件
许多应用程序只需要能够针对某个资源创建一个锁文件即可。
然后,其他程序就可以通过检查这个文件来判断它们自己是否被允许访问这个资源

这些锁文件通常都被放置在一个特定位置,并
带有一个与被控制资源相关的文件名。例如,当一
个调制解调器正在被使用时,Linux通常会
在/var/spool目录下创建一个锁文件。

锁文件只是建议锁,而不是强制锁,在后者
中,系统将强制锁的行为。

为了创建一个用作锁指示器的文件,你可以使
用在==fcntl.h==头文件(你在前面的章节中见过这个文
件)中定义的==open系统调用==,并带上==O_CREAT和
O_EXCL==标志。这样能够以==一个原子操作==同时完成
两项工作:确定文件不存在,然后创建它。


```C
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <fcntl.h>
#include <errno.h>

int main()
{
    int file_desc;
    int save_errno;

    file_desc = open("/tmp/lock.text.lk", O_RDWR | O_CREAT | O_EXCL, 0444);

    if (file_desc == -1)
    {
        save_errno = errno;
        printf("failed : %d\n", save_errno);
    } 
    else 
    {
        printf("success\n");
    }
    exit(EXIT_SUCCESS);
}
```



。。
[qwer@192 ch07]$ gcc file_lock_by_open_6.c 
[qwer@192 ch07]$ ./a.out 
success
[qwer@192 ch07]$ ./a.out 
failed : 17
。。


在Linux系统中,错误号17代表的是EEXIST,这个错误用来表示一个文件已存在。

错误号定义在头文件errno.h或(更常见的)它所包含的头文件中。在本例中,这个错误号实际定义在头文件/usr/include/asm-generic/errno-base.h中
。。fedora 也是这个文件。



利用这个锁机制来协调工作
在进入临界区之前使用open系统调用创建锁文件,然后在退出临界区时用unlink系统调用删除该锁文件

```C
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <fcntl.h>
#include <errno.h>

const char *lock_file = "/tmp/lock.test2.lk";

int main()
{
    int f_desc;
    int tries = 10;

    while (tries--)
    {
        f_desc = open(lock_file, O_RDWR | O_CREAT | O_EXCL, 0444);
        if (f_desc == -1)
        {
            printf("%d - lock already present\n", getpid());
            sleep(3);
        }
        else
        {
            // 临界区开始
            printf("%d - have exclusive access\n", getpid());
            sleep(1);
            (void) close(f_desc);           // !!!
            (void) unlink(lock_file);       // !!!
            // 临界区结束
            sleep(2);
        }
    }
    exit(EXIT_SUCCESS);
}
```


。。
[qwer@192 ch07]$ gcc manager_by_file_lock_7.c 
[qwer@192 ch07]$ ./a.out & ./a.out                      # 命令
[1] 63565
63565 - have exclusive access
63566 - lock already present
63565 - have exclusive access
63566 - lock already present
63566 - have exclusive access
63565 - lock already present
。。

`./lock2 & ./lock2`在后台运行lock2的一份副本,在前台运行另一份副本


### 区域锁定
用创建锁文件的方法来控制对诸如串行口或不
经常访问的文件之类的资源的独占式访问,是一个
不错的选择,

但它并不适用于访问大型的共享文件。

文件段锁定或文件区域锁定

Linux提供了至少两种方式来实现这一功能
- 使用fcnt1系统调用
- 使用lockf调用。

主要介绍fcnt1接口,因为它是最常使用的接口

lockf和fcnt1非常相似,在Linux中,
它一般作为fcnt1的备选接口。但是,fcnt1和lockf
的锁定机制不能同时工作:它们使用不同的底层实现

第3章中已经有fcntl的定义：
```C
#include <fcntl.h>
int fcntl(int fildes, int command, ...);
```

fcnt1对一个打开的文件描述符进行操作,并能
根据command参数的设置完成不同的任务。它为
我们提供了3个用于文件锁定的命令选项
- F_GETLK
- F_SETLK
- F_SETLKW

当使用这些命令选项时,fcnt1的第三个参数必
须是一个指向flock结构的指针,所以实际的函数原
型应为
```C
int fcntl(int fildes, int command, struct flock *flock_structure);
```

flock(文件锁)结构依赖具体的实现,但它至少包含下述成员:
- short l_type
- short l_whence
- off_t l_start
- off_t l_len
- pid_t l_pid

l_type成员的取值定义在头文件fcnt1.h中
- F_RDLCK : 共享锁/读锁
- F_UNLCK : 解锁
- F_WRLCK : 独占锁/写锁


l_whence、l_start和l_len成员定义了文件中的一个区域,即一个连续的字节集合。
l_whence的取值必须是SEEK_SET、SEEK_CUR、SEEK_END(在头文件unistd.h中定义)中的一个。它们分别对应于文件头、当前位置和文件尾。l_whence定义了l_start的相对偏移值,
其中,l_start是该区域的第一个字节。l_whence通常被设为SEEK_SET,这时l_start就从文件的开始计算。
l_len参数定义了该区域的字节数。
l_pid参数用来记录持有锁的进程,参见下面对F_GETLK的介绍

。。whence 就是 start 的基准 是从头开始？从当前开始？从尾巴开始？
。。start 就是 第一个byte 对于 基准的 偏移量
。。len 就是 长度。


#### F_GETLK command

获取fildes(第一个参数)打开的文件的锁信息。它==不会尝试去锁
定文件==。调用进程把自己想创建的锁类型信息传递给fcnt1,使用F_GETLK命令的fcnt1就会返回将会
阻止获取锁的任何信息。

struct flock 中的值：
- l_type : F_RDLCK 共享； F_WRLCK 独占
- l_whence : SEEK_SET, SEEK_CUR, SEEK_END 中的一个
- l_start : 第一个字节的相对位置
- l_len : 文件区域的字节数
- l_pid : 持有锁的进程的标识符

进程可能使用F_GETLK调用来查看文件中某个
区域的当前锁状态。它应该设置flock结构来表明它
需要的锁类型,并定义它感兴趣的文件区域。
fcnt1调用如果成功就返回非-1的值。
如果文件已被锁定从而阻止锁请求成功执行,fcnt1会用相关
信息覆盖flock结构。如果锁请求可以成功执行,
flock结构将保持不变。如果F_GETLK调用无法获得
信息,它将返回-1表明失败。

如果F_GETLK调用成功(例如,它返回一个
非-1的值),调用程序就必须检查flock结构的内容
来判断其是否被修改过。因为==l_pid的值被设置成持
有锁的进程(如果有的话)的标识符==,所以通过检
查这个字段就可以很方便地判断出flock结构是否被
修改过。


#### F_SETLK command

对文件的某个区域 加锁或解锁。
flock解构
- l_type : F_RDLCK 加读锁 ; F_WRLCK 加写锁 ; F_UNLCK 解锁
- l_pid : 不使用

与F_GETLK一样,要加锁的区域由flock结构中的l_start、l_whence和l_len的值定义。如果加锁成
功,fcnt1将返回一个非-1的值;如果失败,则返回-1。这个函数总是==立刻返回==。


#### F_SETLKW command
与上面介绍的F_SETLK命令作用相同
但在无法获取锁时,这个调用将等待直到可以为止

程序对某个文件拥有的所有锁都将在相应的==文件描述符被关闭时自动清除==。在==程序结束==时也会自动清除各种锁


### 锁定状态下的读写操作 (只能用read,write,不能fread,fwrite)

当对文件区域加锁之后,你必须使用底层的read和write调用来访问文件中的数据,
而不要使用更高级的fread和fwrite调用,
这是因为fread和fwrite会对读写的数据进行==缓存==,所以执行一次
fread调用来读取文件中的头100个字节可能(事实
上,是几乎肯定如此)会读取超过100个字节的数
据,并将多余的数据在函数库中进行缓存。如果程序再次使用fread来读取下100个字节的数据,它实
际上将==读取已缓冲在函数库中的数据==,而不会引发
一个底层的read调用来从文件中取出更多的数据。

。。？读cache有什么问题？ 不是加锁了吗？  难道它 加锁，cache读，解锁， 其他进程写入， 加锁，cache读。  第二次读的还是从第一次的cache中的？
。。书上举例了， 是的。。 不是。 是：  加锁，cache读100byte，cache了200byte， 其他进程写入后100个byte， 然后 本线程 加锁，读后100个byte， 会是从 第一次cache中获得的。


。。有代码。不过有点长。。就是测试锁的。 一些关键：

```C
int file_desc = open("/tmp/test_lock", O_RDWR | O_CREATE, 0666);

struct flock region;
region.l_type = F_RDLCK;
region.l_whence = SEEK_SET;
region.l_start = 10;
region.l_len = 20;

result = fcntl(file_desc, F_SETLK, &region);
if (result == -1)
    fprintf(stderr, "failed to lock region\n");

```

```C
region.l_type = F_WRLCK;
region.l_whence = SEEK_SET;
region.l_start = 10;
region.l_len = 100;
region.l_pid = -1;

res = fcntl(file_desc, F_GETLK, &region);
if (res == -1) {
    // failed
}
if (region.l_pid != -1) {
    // lock failed, file has been locked.
} else {
    // lock success
}
```


### 7.2.4 文件锁的竞争

。代码太长了。


### 其他锁命令 lockf()

另外一种锁定文件的方法:lockf函数。它也通过文件描述符进行操作

```C
#include <unistd.h>

int lockf(int fildes, int function, off_t size_to_lock);
```

function参数的取值如下所示。
- F_ULOCK:解锁。
- F_LOCK:设置独占锁。
- F_TLOCK:测试并设置独占锁。
- F_TEST:测试其他进程设置的锁。

size_to_lock参数是操作的字节数,它从文件的
当前偏移值开始计算。

与文件锁定的fcnt1方法一样,lockf设置的所有
锁都是建议锁,它们并不会真正地阻止你读写文件
中的数据。对锁的检测是程序的责任。混合使用
fcnt1锁和lockf锁的效果未被定义



### 死锁

在本例中,死锁是非常容易避免
的:两个程序只需要使用==相同的顺序来锁定==它们需
要的字节或==锁定一个更大的区域==即可。

如果你有兴趣了解更多,请参
阅Principles of Concurrent and Distributed
Programming(《并发和分布式程序设计原
理》,M. Ben-Ari,Prentice Hall,1990)

。。
Principles of Concurrent and Distributed Programming
ISBN-13: 9780321312839
ISBN-10: 032131283X
Author: Mordechai Ben-Ari
Edition: 2
Binding: Paperback
Publisher: Addison-Wesley
Published: 2005
。。 应该没有中文版。




## 7.3 数据库
与使用文件来存储数据相比,使用数据库有如下两方面的优势。
- 你可以存储长度可变的数据记录,这对平面的、非结构化的文件来说实现起来有点困难。
- 数据库使用索引来有效地存储和检索数据。这样做的一个显著优点是这个索引不必非得是一个简单的记录号——这在平面文件中很容易实现,它可以是一个任意的字符串。


### dbm数据库

所有版本的Linux以及大多数的UNIX版本都随系统带有一个基本的、但却非常高效的数据存储例
程集,它被称为dbm数据库。dbm数据库适合于存储相对比较静态的索引化数据。一些数据库纯粹主
义者可能会认为dbm根本算不上是一个数据库,充其量就是一个索引化的文件存储系统。但X/Open
规范把dbm看作是一个数据库,因此在本书里我们也会这么称呼它。

大多数主流的Linux发行版都会默认安装gdbm


如果对SQL数据库很熟悉,你会发现dbm数据
库没有与之关联的表格或列结构。这些结构对于
dbm数据库来说并不是必需的,因为dbm不仅对待
存储的每个数据项没有固定长度的要求,而且对数
据的内部结构也无要求。dbm数据库工作在非结构
化的二进制数据块基础上。
。。NoSQL ?...

。。跳了。 网上 dbm的信息不多。 而且 fedora， gdbm, dbm, ndbm, 都没有这种命令。


## 7.4 CD唱片应用程序 (使用dbm数据库，所以跳)



# ch 8 MySQL

RDBMS或关系型数据库管理系统(Relational Database Management System)

两个最著名的开源RDBMS应用软件是PostgreSQL和MySQL。
- PostgreSQL能在任何情况下免费使用。
- MySQL尽管在某些环境下需要收取许可证费用,但在许多场合下它还是免费的。



检查MySQL服务器是否正在运行

```shell
[qwer@192 include]$ ps -el | grep mysqld
4 S    27    4289       1  0  80   0 - 588988 -     ?        00:00:10 mysqld
[qwer@192 include]$ ps -el | grep mariad
[qwer@192 include]$ 
```

终端命令
```
mysql -u root -p1234
mysql -uroot -p1234
// 都可以，但是不能 -p 1234，会把 1234 识别成 数据库的。

登录后
\s  查看服务器信息
\q  退出控制台

```


使用命令`mysql -?`可以获得更多有关服务器的信息。
。。mariadb 不行， maria-?  mariadb-? 都不行。

。。应该是分开的 需要空格， 就是 终端 进入 help 页面。


在该命令输出参数列表之后,你通常将看到类
似Default options are read from the following
files in the given order:这样的一句话。它告诉
你在哪里可以找到配置文件

。。有的， 最开始的地方。。。
[qwer@192 etc]$ mysql -?
mysql  Ver 15.1 Distrib 10.3.27-MariaDB, for Linux (x86_64) using readline 5.1
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Usage: mysql [OPTIONS] [database]

Default options are read from the following files in the given order:
/etc/my.cnf ~/.my.cnf 
The following groups are read: mysql client client-server client-mariadb
The following options may be given as the first argument:
。。。

配置文件通常是/etc/my.cnf,也有一些发行版(如
Ubuntu)使用的是/etc/mysql/my.cnf。



#### 获得所有配置 (mysqladmin variables)

```shell
$ mysqladmin -u root version
$ mysqladmin variables

# 但是我的都得加 用户和密码，即：
mysqladmin -uroot -p1234 version
mysqladmin -uroot -p1234 variables      # 这个打印的东西太多了，显示不全，要重定向到文件。试了下，474kb。。
```


mysqladmin variables 上面的命令将输出一长串变量设置。
其中两个特别有用的变量是:datadir和have_innodb
- 前者告诉你MySQL在哪里存储它的数据
- 后者的值通常是YES,表明MySQL服务器支持InnoDB存储引擎。

。。datadir 有， 是 var/lib/mysql，   innodb 是默认的存储引擎了， 所以 必然有。
。。 default_storage_engine 就是 InnoDB

在 /etc/my.cnf 文件的 mysqld 节中添加
```text
[mysqld]
default-storage-engine=INNODB
datadir=/var/lib/mysql
```


如果你使用的是InnoDB存储引擎,准
备将数据文件放在/vo102目录中,将日志文件放
在/vol03目录中,设置数据文件的初始大小为
10M,并允许它自扩充,你可以使用如下的配置行
```text
innodb_data_home_dir=/vo102/mysql/data
innodb_data_file_path=ibdata1:10M:autoextend
innodb_log_group_home_dir=/vo103/mysql/logs
```


`mysqladmin -u root password newpassword`
修改密码为 newpassword， 但是有个问题，就是 shell 的历史记录中 明文保存了 密码。。

一个更好的方法是再次使用MySQL控制台
`set password=PASSWORD('secretpassword');`

如果你又想要删除这个密码,你只需用一个空字符串代替“secretpassword”即可

```text
use mysql
select user, host, password from uesr;
```

确保安装安全的下一步将是去除那些由
MySQL默认安装的不需要的用户。下面的命令将
会从权限表中删除所有非root用户

```mysql
> delete from user where user!='root';
> delete from user where host!='localhost';
```


可以使用参数--password,
如--password=secretpassword,或使用-psecretpassword,但显然这是==不太安全==的,因为
密码可能被ps命令或通过命令历史记录看到。


创建用户，并限制权限。
`> grant all on *.* to newUser@localhost identified by 'init_password';`
`> grant all on *.* to newUser@'192.168.0.0/255.255.255.0' identified by 'init_password';`
`> grant all on *.* to newUser@'%.wiley.com' identified by 'init_password';`



## MySQL管理

除mysqlshow命令以外,所有的MySQL命令都接受表8-1所示的3个标准参数
|选项|参数|说明|
|--|--|--|
|-u|用户名|默认情况下，使用当前linux用户名作为mysql用户名|
|-p|密码|有-p，但没有密码，那么mysql会让你输入密码；如果没有-p，那么mysql认为不需要密码|
|-h|主机名|对于本机可以省略|

我们再次建议你不要把密码放在命令行上,因为它可以被ps命令看到。

。。mysqlshow -uroot -p1234。 会返回 数据库列表


### myisamchk 命令
myisamchk工具是设计用来检查和修复使用默认MYISAM表格式的任何数据表

### mysql 命令
通过在命令行的最后添加数据库名称作为参数,你就无需在MySQL的控制台中使用`use <database>`命令

用户名rick、提示输入密码(注意-p参数后面有一个空格)、默认使用数据库foo来启动控制台的命令如下所示
`mysql -u rick -p foo`


使用 `mysql -help | less` 命令来逐页查看mysql控制台的其他命令行选项列表。


登录mysql后，可以使用下面的命令：
|命令|简写|说明|
|--|--|--|
|help, ?|\h, \?|显示命令列表|
|edit|\e|编译命令|
|exit, quit|\q|退出mysql客户端|
|go|\g|执行命令|
|source filename|`\.`|从指定文件执行sql|
|status|\s|显示服务器状态|
|system command|`\!`|执行一个系统命令|
|tee filename|\T|把所有输出的副本添加到指定文件|
|use databasename|\u|使用指定的数据库|


可以通过非交互模式来运行mysql,
只需捆绑命令到一个输入文件中并从命令行读取它
即可。在这种情况下,你必须在命令行上指定密码
`$ mysql -u rick --password=secretpwd foo < sqlcommands.sql`


许多其他数据库服务器,如Oracle和Sybase,使用术语schema(方案),
而MySQL最经常使用的术语是database(MySQL查询浏览器使用的是术语schema)。
每个数据库(在MySQL的术语中)都是一个基本独立的表格集。
这使得你可以针对不同的目的建立不同的数据库,并为每个数据库指定不同的用户

特定数据库mysql是由MySQL安装自动创建的,它用于保存如用户和权限这样的数据



### mysqladmin 命令

除了常见的参数以外，还支持：
|命令|说明|
|--|--|
|create database_name|新建一个数据库|
|drop database_name|删除一个数据库|
|password new_pwd|修改密码|
|ping|检查服务器是否运行|
|reload|重载 控制权限的 grant 表|
|status|服务器状态|
|shutdown|停止服务器|
|variables|显示 控制mysql操作的变量 和当前值|
|version|服务器的版本和持续运行的时间|

不带参数调用mysqladmin命令,我们就可
以从命令提示符下看到完整的选项列表。你也许想
使用| less来分页显示。


### mysqlbug

。。mariadb 没有

### mysqldump

。。这些到时候自己百度吧。
### mysqlimport

### mysqlshow


## 创建用户并赋予权限

### grant

`grant <privilege> on <object> to <user> [identified by user-pwd] [with grant option];`

特权值
|特权|说明|
|--|--|
|alter|改变 表和索引|
|create|创建 数据库和表|
|delete|从数据库中删除数据|
|drop|删除 数据库 和表|
|index|管理索引|
|insert|在数据库中插入数据|
|lock tables|锁定表|
|select|提取数据|
|update|修改数据|
|all|以上所有|

一些命令还有其他选项。例如,create view授
予用户创建视图的权限。要想了解最权威的权限列
表,请查阅MySQL版本的文档

授予特权的对象被标识为:
`databasename.tablename`

`*代表的是通配符,因此*.*代表每个数据库中的每个对象,而foo.*代表数据库foo中的每个表`

如果指定的用户已经存在,他的特权会被编辑以反映你所做的修改。
如果该用户不存在,他就会以指定的特权被创建

在SQL语法中,特殊字符%代表通配符,它与
shell环境中*号的作用完全一样。你当然可以为每
个期望的特权使用单独的命令,但是如果你想授予
用户rick从wiley.com域中任何主机访问的权限,可
以把rick描述为:
rick@'%.wiley.com'
任何时候使用%通配符都必须把它放在引号
中,以与其他文本分开。

你还可以使用IP/网络掩码标识
(N.N.N.N/M.M.M.M)来为访问控制设置一个网
络地址。

你需要格外小心在用户名、主机名或数据库名
中包含下划线的情况,因为SQL中的下划线是一种
匹配任意单个字符的模式,这与%匹配一个字符串
非常类似。因此只要有可能,请==尽量不要在用户名
和数据库名中包含下划线==。

### revoke
剥夺用户权限

`revoke <a_privilege> on <an_object> from <a_user>`

`mysql> REVOKE INSERT ON foo.* FROM rick@'%';`

revoke命令不能删除用户

如果想要完全删除一个用户,不要只是修改他们的权限,而应
用revoke来删除他们的权限。然后,你就可以切换
到内部的mysql数据库,通过从user表中删除相应
的行来完全删除一个用户


## 创建数据库
```SQL
create database rick;
use rick;
```
```shell
mysql -u rick -p rick
```


## 数据类型

### BOOL
TRUE, FALSE, NULL

### 字符类型(6种)
前3个是标准的,后3个是MySQL特有的

|定义|说明|
|--|--|
|char|单字符|
|char(N)|正好 N个字符的字符串，空格字符填充，最多255个字符|
|varchar(N)|N个字符的可变长字符，限制255个字符|
|tinyText|类似于varchar(N)|
|mediumText|最长65535个字符的字符串|
|longText|最长2^31 - 1 个字符的字符串|

。。应该过时了。 现在 varchar 65535 了。 mediumtext最长16777215。

### 数值类型(9种)
分为整型，浮点

整型:
- tinyint : 8位
- smallint : 16
- mediumint : 24
- int : 32，是标准类型，一般情况下，最好的选择
- bigint : 64位有符号

浮点:
- float(P) : 精度至少P位数字的浮点数
- double(D, N) : 有符号双精度浮点数，D位数字，N位小数
- numeric(P, S) : 总长为P位的真实数据，小数点后S位，和double不同，这是一个精准的数字，适合用来存储 货币值，但处理效率低一些
- decimal(P, S) : 和numeric相同。

一般情况下，使用 int, double, numeric。


### 时间类型(5种)

- date : 1000-1-1 到 9999-12-31 之间的日期
- time : 存储从 -838:59:59 到 858:59:59 之间的时间
- timestamp : 从 1970-1-1 到 2037 年之间的时间戳
- datetime : 1000-1-1 到 9999-12-31 最后一秒 之间的日期
- year : 存储年份， 2位数的年份 自动转为 四位数的 年份


## 创建表 DDL
data definition language

## 图形化工具


## 使用C访问MySQL数据

### 连接例程
C连接MySQL包含2个步骤
- 初始化一个连接句柄解构
- 实际进行连接

使用mysql_init来初始化连接句柄
```C
#include <mysql.h>

MYSQL *mysql_init(MYSQL *);
```

通常你传递NULL给这个例程,它会返回一个指
向新分配的连接句柄结构的指针。如果你传递一个
已有的结构,它将被重新初始化。这个例程在出错
时返回NULL。

目前为止,你只是分配和初始化了一个结构。
你仍然需要使用mysql_real_connect来向一个连接
提供参数:
```C
MYSQL *mysql_real_connect(MYSQL *connection,
    const char *server_host,
    const char *sql_user_name,
    const char *sql_password,
    const char *db_name,
    unsigned int port_number,
    const char *unix_socket_name,
    unsigned int flags);
```

。。C++ 连接mysql 简单多了。

指针connection必须指向已经被mysql_init初始化过的结构。
server_host既可以是主机名,也可以是IP地址。如果只是连接到本地机器,你可以通过指定localhost来优化连接类型。

如果登录名为NULL,则假设登录名为当前Linux用户的登录ID。
如果密码为NULL,你将只能访问服务器上无需密码就可访问的数据。
密码会在通过网络传输前进行加密。

port_number和unix_socket_name应该分别为==0和NULL==,
除非你改变了MySQL安装的默认设置。它们将默认使用合适的值。

flags参数用来对一些定义的位模式进行OR操作,使得改变使用协议的某些特性。

如果无法连接,它将返回NULL。mysql_error函数可以提供有帮助的信息。

使用完连接之后,通常在程序退出时,你要像下面这样调用函数mysql_close:
```C
void mysql_close(MYSQL *conn);
```

如果连接是由mysql_init建立的,MySQL结构会被释放。指针将会失效并无法
再次使用。保留一个不需要的连接是对资源的浪费,但是重新打开连接也会带来额外的开销,所以
你必须自己权衡何时使用这些选项

==mysql_options==例程(仅能在mysql_init和mysql_real_connect之间调用)可以设置一些选项:
```C
int mysql_options(MYSQL *conn, enum option_to_set, const char *argument);
```

mysql_options一次只能设置一个选项,所
以每设置一个选项就得调用它一次。
3个最常用的选项
- MySQL_OPT_CONNECT_TIMEOUT : const unsigned int *; 超时前的等待秒数
- MySQL_OPT_COMPRESS : None, 使用NULL; 网络连接中 使用压缩机制
- MySQL_INIT_COMMAND : const char *; 每次连接后发送的命令

一次成功的调用将返回0。因为它仅仅是用来设
置标志,所以失败总是意味着使用了一个无效的选项。



# ==from there==


```C
#include <stdlib.h>
#include <stdio.h>

#include "mysql.h"

int main(int argc, char *argv[])
{
    MYSQL *pconn;

    pconn = mysql_init(NULL);
    if (!pconn)
    {
        fprintf(stderr, "mysql init failed\n");
        return EXIT_FAILURE;
    }

    pconn = mysql_real_connect(pconn, "localhost", "root", "1234", "down_txt", 0, NULL, 0);
    
    if (pconn)
    {
        printf("connect success\n");
    }
    else
    {
        printf("connect fail\n");
    }

    mysql_close(pconn);

    return EXIT_SUCCESS;
}
```

。。没有驱动。。
https://mariadb.com/docs/skysql-previous-release/connect/programming-languages/c/install


。。下面直接下载驱动，然后自己解压，找个地方放吧。链接的时候就指定下路径。
https://mariadb.com/downloads/?tab=connectors&group=connectors_dataaccess&product=c%20connector


。。
$ gcc 01_connect_mysql.c -I/home/aa/mariadb-c/include/mariadb -I/home/aa/mariadb-c/include/mariadb/mysql -L/home/aa/mariadb-c/lib/mariadb/ -lmariadb
。。 不行， 缺少很多库。不知道怎么弄。。
。。上面的命令，可以 进入 压缩包解压后的 bin 目录中，执行 `./mariadb_config` 会显示命令。

。。
/usr/bin/ld: warning: **libssl.so.1.1**, needed by /home/aa/mariadb-c/lib/mariadb//libmariadb.so, not found (try using -rpath or -rpath-link)
/usr/bin/ld: warning: libcrypto.so.1.1, needed by /home/aa/mariadb-c/lib/mariadb//libmariadb.so, not found (try using -rpath or -rpath-link)
/usr/bin/ld: /home/aa/mariadb-c/lib/mariadb//libmariadb.so: undefined reference to `SSL_shutdown@OPENSSL_1_1_0'
。还有很多，二十个吧。
。。。看了下， /lib/x84_64-linux-gnu/ 下只有 libssl.so.3 。。没有.1.1.。哎。。要装 openssl 1.1 。。 
。。 `openssl version` 显式ubuntu 22 安装的是 3.0.2


。换了一个connector的版本。 可以 编译了， 下面的应该是 从默认的 lib中找不到。

$ ./a.out 
./a.out: error while loading shared libraries: libmariadb.so.3: cannot open shared object file: No such file or directory
。。


$ ./a.out 
connect fail
。。可以了。 直接 cp libmariadb.so.3 到 /lib/ 下，就可以了。  /lib/mariadb/ 下不行。



### 8.3.2 错误处理

MySQL使用一系列由连接句柄结构报告的返回码。两个必备的例程是

`unsigned int mysql_errno(MYSQL *conn);`
获得错误码，通常是非0值。 没有设定错误码，就是0
因为每次调用库都会更新错误码，所以你只能得到最后一个执行命令的错误码
返回的错误码，在头文件 errmsg.h 或 mysqld_error.h 中定义。前者报告客户端错误，后者是服务端错误。

`char *mysql_error(MYSQL *conn);`
提供有意义的文本，而不是 错误码。

```C
#include <stdlib.h>
#include <stdio.h>

#include "mysql.h"

int main(int argc, char *argv[])
{
    MYSQL conn;

    mysql_init(&conn);

    if (mysql_real_connect(&conn, "localhost", "rick", "i do not know", "foo", 0, NULL, 0))
    {
        printf("connect success\n");
        mysql_close(&conn);
    }
    else
    {
        fprintf(stderr, "conn fail\n");
        if (mysql_errno(&conn))
        {
            fprintf(stderr, "conn error %d: %s\n",
                mysql_errno(&conn), mysql_error(&conn));
        }
    }
    return EXIT_SUCCESS;
}
```
通过避免使用返回值覆盖连接指针的方法，你
可以很容易地解决mysql_real_connect失败所带来
的问题。不仅如此，这也是另一种使用连接结构的
好例子



### 执行sql语句

`int mysql_query(MYSQL *conn, const char *query);`

这里的sql不需要;
如果成功，它返回0。对于包含二进制数据的查询，你可以使用第二个例程mysql_real_query，

。。这个好像没有地方返回数据啊。


#### 不返回数据的sql
先看一些不返回数据的sql： update， delete， insert

下面的方法用于 检查 受影响的行数
`my_ulonglong mysql_affected_rows(MYSQL *conn);`

printf 时，推荐使用 %lu 将 my_ulonglong 转为 无符号长整型。

这个函数返回受之前执行的UPDATE、INSERT或DELETE查询影响的行数

登录mysql服务器创建表
```SQL
create table children (
    childno int(11) auto_increment not null primary key,
    fname varchar(30),
    age int
);
```


```C
#include <stdlib.h>
#include <stdio.h>

#include "mysql.h"

int main(int agrc, char *argv[])
{
    MYSQL conn;
    int res;

    mysql_init(&conn);
    if (mysql_real_connect(&conn, "localhost", "rick", "secretpwd", "foo", 0, NULL, 0))
    {
        printf("conn success\n");
        res = mysql_query(&conn, "insert into children(fname, age) values ('Ann', 3)");
        if (!res)
        {
            printf("inserted %lu rows\n", (unsigned long) mysql_affected_rows(&conn));
        }
        else
        {
            fprintf(stderr, "insert error %d: %s\n", mysql_errno(&conn), mysql_error(&conn));
        }
        mysql_close(&conn);
    }
    else
    {
        fprintf(stderr, "conn fail\n");
        if (mysql_errno(&conn))
        {
            fprintf(stderr, "conn error %d : %s\n", mysql_errno(&conn), mysql_error(&conn));
        }
    }
    return EXIT_SUCCESS
}
```

替换上面的 mysql_query。 就可以。
`res = mysql_query(&conn, "update children set age=4 where fname='Ann'");`

如果有下面的数据
|childNo|fname|age|
|--|--|--|
|1|Ann|4|
|2|Ann|4|
|3|Ann|3|
|4|Ann|3|

那么上面的 mysql_affected_rows 会是 2。 而不是 4。


你可以使用mysql_real_connect的CLIENT_FOUND_ROWS标志来获得更传统的报告。
`if (mysql_real_connect(&conn, "localhost", "rick", "secretpwd", "foo", 0, NULL, CLIENT_FOUND_ROWS))`

这样的话， 上面的 upfate 语句 的 mysql_affected_rows 就是4。

mysql_affected_rows, 使用where从数据库删除时，返回删除的行数。
但是不带where时，返回0。
这是因为MySQL优化了删除所有
行的操作，它并不是执行许多个单行删除操作。这
一行为不会受CLIENT_FOUND_ROWS选项标志的影响。


发现插入的内容

主键是 auto_increment时， 怎么知道刚插入的row 的主键值呢？ 用select 的话，效率低，并且 如果重名，不能保证唯一。 或者 并发的时候。

mysql 提供了 last_insert_id()
无论何时MySQL向AUTO_INCREMENT列中插
入数据，MySQL都会基于每个用户对最后分配的
值进行跟踪。用户程序可以通过SELECT专用函数
LAST_INSERT_ID()来发现该值，这个函数的作用
有点像是表中的虚拟列

。。是用户的， 但是 多个 app instance， 用户是同一个吧。

```SQL
insert into children(fname, age) values ('Tom', 13);

select last_insert_id();
```



```C
    MYSQL conn;
    MYSQL_RES *req_ptr;
    MYSQL_ROW sqlrow;
    int res;

    mysql_init(&conn);

        res = mysql_query(&conn, "select last_insert_id()");
        if (res)
        {
            printf("select error: %s\n", mysql_error(&conn));
        }
        else
        {
            res_ptr = mysql_use_result(&conn);
            if (res_ptr)
            {
                while ((sqlrow = mysql_fetch_row(res_ptr)))
                {
                    printf("we inserted childno: %s\n", sqlrow[0]);
                }
                mysql_free_result(res_ptr);
            }
        }
```


#### 返回数据的语句

MySQL也支持使用SQL语句SHOW、DESCRIBE和EXPL AIN来返回结果，但我们不会在这里涉及它们。

在C应用程序中提取数据一般需要下面4个步骤：
- 执行查询；
- 提取数据；
- 处理数据；
- 必要的清理工作。

- mysql_query来发送SQL语句。
- mysql_store_result或mysql_use_result来提取数据
- mysql_fetch_row调用来处理数据。
- mysql_free_result释放查询占用的内存资源

mysql_use_result和mysql_store_result的区别
主要在于，你是想一次返回一行数据，还是一次返
回所有的结果。当你预计结果集比较小时，后者会
更加合适


一次提取所有数据
`MYSQL_RES *mysql_store_result(MYSQL *conn);`

返回一个指向结果集结构的指针，如果失败则返回NULL。

mysql_store_result调用成功之后，你需要调
用mysql_num_rows来得到返回记录的数目，我们
希望这是个正数，但是如果没有返回行，这个值将是0。

`my_ulonglong mysql_num_rows(MYSQL_RES *result);`

函数接受由mysql_store_result返回的结果
结构，并返回结果集中的行数。如果
mysql_store_result调用成功，mysql_num_rows
将始终都是成功的。

如果你碰巧使用的是一个==特别庞大的数据集==，
那么最好==提取小一些、更容易管理的信息块==，因为
这将==更快地将控制权返回给应用程序==，并且==不会占
用大量的网络资源==。我们将在介绍
mysql_use_result的时候，详细探讨这一想法

可以使用mysql_fetch_row来处理它，也可以使用mysql_data_seek、
mysql_row_seek和mysql_row_tell在数据集中来回移动

- `MYSQL_ROW mysql_fetch_row(MYSQL_RES *result);`
从使用mysql_store_result得到的结果结构
中提取一行，并把它放到一个行结构中。当数据用完或发生错误时返回NULL

- `void mysql_data_seek(MYSQL_RES *result, my_ulonglong offset);`
在结果集中进行跳转，设置将会被下一个mysql_fetch_row操作返回的行。参数
offset的值是一个行号，它必须在0到结果集总行数减1的范围内

- `MYSQL_ROW_OFFSET mysql_row_tell(MYSQL_RES *result);`
返回一个偏移值，它用来表示结果集中的当前位
置。它不是行号，你不能把它用于mysql_data_seek

- `MYSQL_ROW_OFFSET mysql_row_seek(MYSQL_RES *result, MYSQL_ROW_OFFSET offset);`
使用 mysql-row-tell 的返回值。移动当前位置，并返回之前的位置。

不要混淆了由row_tell和row_seek使用的偏移量和data_seek使用的行号

- `void mysql_free_result(MYSQL_RES *result);`
显式调用mysql_free_result来让MySQL库完成善后处理


```C
#include <stdlib.h>
#include <stdio.h>

#include "mysql.h"

MYSQL conn;
MYSQL_RES *req_ptr;
MYSQL_ROW sqlrow;

int main(int agrc, char *argv[])
{
    int res;

    mysql_init(&conn);
    if (mysql_real_connect(&conn, "localhost", "rick", "secretpwd", "foo", 0, NULL, 0))
    {
        printf("conn success\n");
        
        res = mysql_query(&conn, "select childno, fname, age from children where age > 5");

        if (!res)
        {
            // printf("inserted %lu rows\n", (unsigned long) mysql_affected_rows(&conn));
            res_ptr = mysql_store_result(&conn);
            if (res_ptr)
            {
                printf("retrieve %lu rows\n", (unsigned long) mysql_num_rows(res_ptr));
                while ((sqlrow = mysql_fetch_row(res_ptr)))
                {
                    printf("fetching data...\n");
                }
                if (mysql_errno(&conn))
                {
                    fprintf(stderr, "retrieve error %s\n", mysql_error(&conn));
                }
                mysql_free_result(res_ptr);
            }
        }
        else
        {
            fprintf(stderr, "select error %d: %s\n", mysql_errno(&conn), mysql_error(&conn));
        }
        mysql_close(&conn);
    }
    else
    {
        fprintf(stderr, "conn fail\n");
        if (mysql_errno(&conn))
        {
            fprintf(stderr, "conn error %d : %s\n", mysql_errno(&conn), mysql_error(&conn));
        }
    }
    return EXIT_SUCCESS
}
```



一次提取一行

`MYSQL_RES *mysql_use_result(MYSQL *conn);`

与mysql_store_result函数一样，
mysql_use_result在遇到错误时也返回NULL。如
果成功，它返回指向结果集对象的指针。但是，不
同之处在于它未将提取的数据放到它初始化的结果集中。

必须反复调用 mysql_fetch_row直到提取了所有的数据，否则后续的提取数据操作可能会返回遭到破坏的信息

调用mysql_use_result和调用mysql_store_result的效果有何不同呢？前者具备
资源管理方面的实质性好处，但是它不能与
mysql_data_seek、或mysql_row_seek或
mysql_row_tell一起使用，并且由于直到所有数据
都被提取后才能实际生效，mysql_num_rows的使用也受到限制。

还增加了时延，因为每个行请求和结果的返
回都必须通过网络。另外还存在一种可能性是，网
络连接可能在操作中途失败，留给你不完整的数据

。。是指 一次一条的会增加时间， 一次一条的 不能和 xxx,xxx,xxx 一起使用。

但是，无论怎样，这些都不会抹去我们之前提
到的它带来的好处：==更好地平衡了网络负载，以及
减少了可能非常大的数据集带来的存储开销==

```C
    res_ptr = mysql_use_result(&conn);
    if (res_ptr)
    {
        whlie ((sqlrow = mysql_fetch_row(res_ptr)))
        {
            printf("fetched data...\n");
        }
        // ...
    }
```

在提取最后一个结果之前，你仍然
无法得到行数。但是，通过早期和经常性的错误检
查，可以使得程序调整为使用mysql_use_result变
得更加容易。以这种方式编写代码可以减少许多程
序后期修改带来的烦恼。



#### 处理返回的数据

如同大多数SQL数据库一样，MySQL返回两种
类型的数据。
- 从表中提取的信息，也就是列数据。
- 关于数据的数据，即所谓的元数据（metadata），例如列名和类型。

mysql_field_count函数提供了一些关于查询结
果的基本信息。它接受连接对象，并返回结果集中
的字段（列）数目

`unsigned int mysql_field_count(MYSQL *conn);`

在更通用的方式下，你可以用
mysql_field_count做其他事情，比如判断为何
mysql_store_result的调用会失败。例如，如果
mysql_store_result返回NULL，但是
mysql_field_count返回一个正数，你可以推测这
是一个提取错误。但是，如果mysql_field_count
返回0，则表示没有列可以提取，这可以解释为何
存储结果会失败。我们有理由认为，你应该了解一
个特定查询应返回的列数。因此，对于通用查询处
理模块或任何随意构造查询的情况，这个函数是非
常有用的。

在为旧版本的MySQL所写的代码中，你可能
会看到使用mysql_num_fields的情况。它可以接
受一个连接结构或一个结果结构指针作为参数，
并返回列数。


```C
// 04_select.c 中

void display_row()
{
    unsigned int field_count;

    field_count = 0;
    while (field_count < mysql_field_count(&conn))
    {
        printf("%s ", sqlrow[field_count]);
        ++field_count;
    }
    printf("\n");
}
```

`gcc -I/usr/include/mysql 04_select.c -L/usr/lib/mysql -lmysqlclient -o select`



要同时得到MySQL返回的数据和元数据。你可以使用mysql_fetch_field来

`MYSQL_FIELD *mysql_fetch_field(MYSQL_RES *result);`

需要重复调用此函数，直到返回表示数据结
束的NULL值为止。然后，你可以使用指向字段结
构数据的指针来得到关于列的信息

struct MYSQL_FIELD 定义在 mysql.h 中。

|成员|说明|
|--|--|
|char *name|列名|
|char *table|列所属的表名，当一个查询涉及多个表时，特别有用。对于结果集中的可计算值如MAX，它对应的表名为 空字符串|
|char *def|如果调用 mysql_list_fields，它将包含该列的默认值|
|enum enum_field_types type|列类型|
|unsigned int length|列宽，在定义表时指定|
|unsigned int max_length|使用mysql_store_result，将包含 字节为单位的 提取的最长列值的长度； 使用mysql_use_result，它不会被设置|
|unsigned int flags|关于列定义的标志，与得到的数据无关。常见标志：NOT_NULL_FLAG, PRI_KEY_FLAG, UNSIGNED_FLAG, AUTO_INCREMENT_FLAG, BINARY_FLAG|
|unsigned int decimals|小数点后的数字个数，仅对数字字段有效|


列类型相当广泛。完整列表见头文件 mysql_com.h 和文档
常见的有：
- FIELD_TYPE_DECIMAL
- FIELD_TYPE_LONG
- FIELD_TYPE_STRING
- FIELD_TYPE_VAR_STRING

一个特别有用的预定义宏为IS_NUM，当字段类型为数字时，它返回true.如下使用：
`if (IS_NUM(mysql_field_ptr->type)) printf("numeric type field\n");`


`MYSQL_FIELD_OFFSET mysql_field_seek(MYSQL_RES *result, MYSQL_FIELD_OFFSET offset);`

用此函数来覆盖当前的字段编号，该编
号会随每次mysql_fetch_field调用而自动增加。如
果给参数offset传递值0，你将跳回第一列

```C
#include <stdlib.h>
#include <stdio.h>

#include "mysql.h"


MYSQL conn;
MYSQL_RES *req_ptr;
MYSQL_ROW sqlrow;

void display_header();
void display_row();

int main(int agrc, char *argv[])
{
    int res;
    int first_row = 1;

    mysql_init(&conn);
    if (mysql_real_connect(&conn, "localhost", "rick", "secretpwd", "foo", 0, NULL, 0))
    {
        printf("conn success\n");
        
        res = mysql_query(&conn, "select childno, fname, age from children where age > 5");

        if (!res)
        {
            // one select one row
            res_ptr = mysql_use_result(&conn);
            if (res_ptr)
            {
                whlie ((sqlrow = mysql_fetch_row(res_ptr)))
                {
                    if (first_row)
                    {
                        first_row = 0;
                        display_header();
                    }
                    display_row();
                }

                if (mysql_errno(&conn))
                {
                    fprintf(stderr, "retrieve error %s\n", mysql_error(&conn));
                }
                mysql_free_result(res_ptr);
            }
        }
        else
        {
            fprintf(stderr, "select error %d: %s\n", mysql_errno(&conn), mysql_error(&conn));
        }
        mysql_close(&conn);
    }
    else
    {
        fprintf(stderr, "conn fail\n");
        if (mysql_errno(&conn))
        {
            fprintf(stderr, "conn error %d : %s\n", mysql_errno(&conn), mysql_error(&conn));
        }
    }
    return EXIT_SUCCESS
}

void display_row()
{
    unsigned int field_count;

    field_count = 0;
    while (field_count < mysql_field_count(&conn))
    {
        if (sqlrow[field_count])
            printf("%s ", sqlrow[field_count]);
        else
            printf("NULL");
        ++field_count;
    }
    printf("\n");
}

void display_header()
{
    MYSQL_FIELD *field_ptr;
    printf("column detail:\n");

    while ((field_ptr = mysql_fetch_field(res_ptr)) != NULL)
    {
        printf("\t Name: %s\n", field_ptr->name);
        printf("\t Type: ");
        if (IS_NUM(field_ptr->type))
        {
            printf("Numeric field\n");
        }
        else
        {
            switch (field_ptr->type)
            {
            case FIELD_TYPE_VAR_STRING:
                printf("varchar\n");
                break;
            case FIELD_TYPE_LONG:
                printf("long\n");
                break;
            default:
                printf("type is %d, check in mysql_com.h\n", field_ptr->type);
                break;
            }
        }
        printf("\t max witdh %ld\n", field_ptr->length);
        if (field_ptr->flags & AUTO_INCREMENT_FLAG)
        {
            printf("\t auto inc\n");
        }
        printf("\n");
    }
}
```


### 更多的函数

|函数|说明|
|--|--|
|char *mysql_get_client_info(void);|客户端使用的库的版本信息|
|char *mysql_get_host_info(MYSQL *conn);|服务器连接信息|
|char *mysql_get_server_info(MYSQL *conn);|当前连接的服务器的信息|
|char *mysql_info(MYSQL* conn)|返回最近执行的查询的信息|
|int mysql_select_db(MYSQL* conn, const char *dnname)|如果用户有权限，则把默认数据库改为参数指定的数据库，成功返回0|
|int mysql_shutdown(MYSQL* conn, enum mysql_enum_shutdown_level)|如果用户有权限，关闭连接的数据库服务器。只能用SHUTDOWN_DEFAULT，成功返回0|



## CD数据库应用程序





# ch 9 开发工具

工具包括：
- make命令和makefile文件
- 使用RCS和CVS系统对源代码进行控制
- 编写手册页
- 使用patch和tar命令来发布软件
- 开发环境

## 9.1 多个源文件带来的问题

大型程序来说，一次修改，重新编译全部源文件，这是不可接受的。

如果在程序中创建了多个头文件，并在不同的
源文件中包含它们，这种处理方式就会带来一个潜
在的、更严重的问题，修改后，会忘记重新编译哪些包含了这个头文件的源文件。


make工具可以解决上述这些问题，它会在必要时重新编译所有==受改动影响==的源文件

make命令不仅仅用于编译程序，无论何时，
当需要通过多个输入文件来生成输出文件时，你
都可以利用它来完成任务。它的其他用法还包括
文档处理（例如针对troff或TeX文档）。


## make命令 和makefile文件

make很强大，但是它不知道应该如何建立应用程序。
需要为它提供一个文件，告诉它应用程序 应该如何构造，这个文件被称为makefile。

makefile文件一般都会和项目的其他源文件放在同一目录下

如果管理的是一个大项目，你可以用多个不同的makefile文件来分别管理项目的不同部分


### makefile 语法

makefile文件由一组依赖关系和规则构成。
- 每个依赖关系由一个目标（即将要创建的文件）和一组该目标所依赖的源文件组成。
- 而规则描述了如何通过这些依赖文件创建目标。一般来说，目标是一个单独的可执行文件

。。目标和它的依赖， 如何通过依赖构建目标。
。。如果目标是 exe程序的话， 依赖就是 源码 + 源码中的include + 递归的include

make命令会读取makefile文件的内容，它先确
定目标文件或要创建的文件，然后比较该目标所依
赖的源文件的日期和时间以决定该采用哪条规则来
构造目标。通常在创建最终的目标文件之前，它需
要先创建一些中间目标。make命令会根据
makefile文件来确定目标文件的创建顺序以及正确
的规则调用顺序。


### make 的选项和参数

最常用的3个选项
- -k：它的作用是让make命令在发现错误时仍然继续执行，而不是在检测到第一个错误时就停下来。你可以利用这个选项在一次操作中发现所有未编译成功的源文件。
- -n：它的作用是让make命令输出将要执行的操作步骤，而不真正执行这些操作。
- -f <filename>：它的作用是告诉make命令将哪个文件作为makefile文件。如果未使用这个选项，标准版本的
make命令将首先在当前目录下查找名为
makefile的文件，如果该文件不存在，它
就会查找名为Makefile的文件。但如果你
是在Linux系统中，你使用的可能是GNU
Make，这个版本的make命令将在搜索
makefile文件和Makefile文件之前，首先
查找名为GNUmakefile的文件。按惯例，
许多Linux程序员使用文件名Makefile，
因为如果一个目录下都是以小写字母为名
称的文件，则Makefile文件将在目录的文
件列表中第一个出现。我们建议不要使用
文件名GNUmakefile，因为它是特定于
make命令的GNU实现的。

为了指示make命令创建一个特定的目标（通常
是一个可执行文件），你可以把该目标的名字作为
make命令的一个参数。如果不这么做，make命令
将试图创建列在makefile文件中的第一个目标。许
多程序员都会在自己的makefile文件中将第一个目
标定义为all，然后再列出其他从属目标。这个约定
可以明确地告诉make命令，在未指定特定目标
时，默认情况下应该创建哪个目标。我们建议读者
都坚持使用这一约定。

#### 依赖关系

```C
// main.c
#incldue "a.h"

// 2.c
#include "a.h"
#include "b.h"

// 3.c
#include "b.h"
#include "c.h"
```

依赖关系：
。。上面说过，是 目标 和 依赖

```text
myapp: main.o 2.o 3.o
main.o: main.c a.h
2.o: 2.c a.h b.h
3.o: 3.c b.h c.h
```

你可以很容易地看出，如果文件
b.h发生改变，你就需重新编译2.o和3.o，而由于
2.o和3.o发生了改变，你还需要重新创建目标
myapp。

如果想一次创建多个文件，你可以利用伪目标
all。假设应用程序由二进制文件myapp和使用手册
myapp.1组成。你可以用下面这行语句进行定义
```text
all: myapp myapp.1
```

如果未指定一个all目标，则make命令将只创建它在文件makefile中找到的第一个目标


#### 规则

定义了目标的创建方式

当make命令确定需要重建2.o时，它具体应该使用哪条命
令呢？看上去只需使用命令gcc -c 2.c就够了

规则所在的行必须以制表符tab开头，用空格是不行的

。。下面的都是4个空格。。不是tab，需要自己修正。
```text
myapp: main.o 2.o 3.o
    gcc -o myapp main.o 2.o 3.o

main.o: main.c a.h
    gcc -c main.c

2.o: 2.c a.h b.h
    gcc -c 2.c

3.o: 3.c b.h c.h
    gcc -c 3.c
```

你在调用make命令时加上-f选项，这是因为
makefile文件并未使用常见的默认文件名makefile
或Makefile


。。make没有预装。
。。
$ make -f 01_makefile.mk 
make: *** 没有规则可制作目标“main.c”，由“main.o” 需求。 停止。
。。

头文件都是空文件
```text
touch a.h
touch b.h
touch c.h
```

```C
// main.c
#include <stdlib.h>
#include "a.h"

extern void fun2();
extern void fun3();

int main()
{
    fun2();
    fun3();
    exit(EXIT_SUCCESS);
}
```

```C
// 2.c
#include <stdio.h>
#include "a.h"
#include "b.h"

void fun2()
{
    printf("fun2");
}
```

```C
// 3.c
#include <stdio.h>
#include "b.h"
#include "c.h"

void fun3()
{
    printf("fun3");
}
```
。。
$ make -f 01_makefile.mk 
gcc -c main.c
gcc -c 2.c
gcc -c 3.c
gcc -o myapp main.o 2.o 3.o
。。

touch b.h ，可以看到重新执行了部分命令
rm 2.o  也可以看到执行了部分命令。

。。根据touch，看来是 根据 最后修改时间判断 是否需要 重新make 的


### makefile 中的注释

`#` 开头 到 行末


### makefile 的宏

`MACRONAME=value` 在 makefile 文件中定义宏

引用宏的方法是使用 `$(MACRONAME)` 或 `${MACRONAME}`
make的 某些版本还接受 `$MACRONAME` 的用法

如果想把一个宏的值设置为空，你可以令等号（=）后面留空。

makefile文件中的宏常被用于设置编译器的选
项。在软件的开发过程中，通常开发人员不会对编
译结果进行优化，而是将调试信息包含进去。但对
于软件的发行版，往往又需反过来做，即编译结果
是一个不包含调试信息的容量较小的二进制可执行
文件，使其执行速度尽可能快

Makefile1文件的另一问题是，它假设编译器的
名字是gcc，而在其他UNIX系统中，编译器的名字
可能是cc或c89。如果想将makefile文件移植到另
一版本的UNIX系统中，或在现有系统中使用另一
个编译器，为了使其工作，你将不得不修改
makefile文件中许多行的内容。宏是用来收集所有
这些与系统相关内容的好方法，通过使用宏定义，
你可以方便地修改这些内容。


宏通常都是在makefile文件中定义的，但你也
可以在调用make命令时在命令行上给出宏定义，
例如命令make CC=c89。命令行上的宏定义将覆
盖在makefile文件中的宏定义

应避免在宏定义中使用空格或应像下面这样给宏定义加上引号：make "CC=c89"。

```text
all: myapp

# compiler
CC = gcc

INCLUDE = .

# option for development
CFLAGS = -g -Wall -ansi

# option for release
# CFLAGS = -O -Wall -ansi

myapp: main.o 2.o 3.o
	$(CC) -o myapp main.o 2.o 3.o

main.o: main.c a.h
	$(CC) -I$(INCLUDE) $(CFLAGS) -c main.c

2.o: 2.c a.h b.h
	$(CC) -I$(INCLUDE) $(CFLAGS) -c 2.c

3.o: 3.c b.h c.h
	$(CC) -I$(INCLUDE) $(CFLAGS) -c 3.c
```


make命令内置了一些特殊的宏定义
- $? : 当前目标所依赖的文件列表中 比当前目标文件 还要新的文件
- $@ : 当前目标的名字
- $< : 当前依赖文件的名字
- $* : 不包括后缀的当前依赖文件的名字


在makefile文件中，你可能还会看到下面两个有用的特殊字符，它们出现在命令之前。
- -：告诉make命令忽略所有错
误。例如，如果想创建一个目录，但又想
忽略任何错误（比如目录已存在），你就
可以在mkdir命令的前面加上一个减号。
你将在本章后面的例子中看到符号-的应
用。
- @：告诉make在执行某条命令前
不要将该命令显示在标准输出上。如果想
用echo命令给出一些说明信息，这个字符
将非常有用。


### 多个目标

通常制作不止一个目标文件或者将多组命令集
中到一个位置来执行是很有用的。你可以通过扩展
makefile文件来达到这一目的。在下面的例子中，
你在makefile文件中增加一个clean选项来删除不需
要的目标文件，增加一个install选项来将编译成功
的应用程序安装到另一个目录下。

```text

all: myapp

# compiler
CC = gcc

# where to install
INSTDIR = /usr/local/bin

INCLUDE = .

# option for development
CFLAGS = -g -Wall -ansi

# option for release
# CFLAGS = -O -Wall -ansi

myapp: main.o 2.o 3.o
	$(CC) -o myapp main.o 2.o 3.o

main.o: main.c a.h
	$(CC) -I$(INCLUDE) $(CFLAGS) -c main.c

2.o: 2.c a.h b.h
	$(CC) -I$(INCLUDE) $(CFLAGS) -c 2.c

3.o: 3.c b.h c.h
	$(CC) -I$(INCLUDE) $(CFLAGS) -c 3.c

clean:
	-rm main.o 2.o 3.o

install: myapp
	@if [ -d ${INSTDIR} ]; \
		then \
		cp myapp ${INSTDIR};\
		chmod a+x $(INSTDIR)/myapp;\
		chmod og-w ${INSTDIR}/myapp;\
		echo "installed in ${INSTDIR}";\
	else \
		echo "Sorry, ${INSTDIR} not exist";\
	fi
```

。。本地 不执行 clean 及后续， 也不输出任何错误信息。
。。当然也没有 删除。
。。/usr/local/bin 里也没有东西。
。。。。。下面的命令

。。
$ make -f 03_makefile.mk install
cp: 无法创建普通文件 '/usr/local/bin/myapp': 权限不够
chmod: 无法访问 '/usr/local/bin/myapp': 没有那个文件或目录
chmod: 无法访问 '/usr/local/bin/myapp': 没有那个文件或目录
installed in /usr/local/bin

$ sudo su
[sudo] aa 的密码： 

root # make -f 03_makefile.mk install
installed in /usr/local/bin

root # ls
01_makefile.mk  2.c  3.c  a.h  c.h     main.o
03_makefile.mk  2.o  3.o  b.h  main.c  myapp

root # make -f 03_makefile.mk clean
rm main.o 2.o 3.o

root # ls
01_makefile.mk  03_makefile.mk  2.c  3.c  a.h  b.h  c.h  main.c  myapp
。。


### 内置规则 (make -p)

```C
#include <stdio.h>
#include <stdlib.h>

int main()
{
    printf("hi\n");
    exit(EXIT_SUCCESS);
}
```

。。
$ make 04_hi.c
make: 对“04_hi.c”无需做任何事。
$ make 04_hi
cc     04_hi.c   -o 04_hi
。。

它选择的是cc而不是gcc（在Linux系统
中，这没有问题，因为cc通常是gcc的一个连接文
件）。有时，这些内置规则又被称为推导规则，由
于它们都会使用宏定义，因此可以通过给宏赋予新
值来改变其默认行为。

。。
$ rm 04_hi
$ make CC=gcc CFLAGS="-Wall -g" 04_hi
gcc -Wall -g    04_hi.c   -o 04_hi
。。


你可以通过-p选项让make命令打印出其所有内
置规则。由于内置规则实在太多


在内置规则的帮助下，makefile：
```text
main.o: main.c a.h
2.o: 2.c a.h b.h
3.o: 3.c b.h c.h
```


### 后缀 和模式规则

你看到的内置规则在使用时都利用了文件的后
缀名（这类似Windows和MS-DOS的文件扩展
名），所以当给出带有某个特定后缀名的文件时，
make命令知道应该用哪个规则来创建带有另一个
不同后缀名的文件


要想增加一条新的后缀规则，首先需要在
makefile文件中增加一行语句，告诉make命令这
个新的后缀名。然后即可用这个新的后缀名来定义
规则。make使用特殊语法：
`.<old_suffix>.<new_suffix>:`
来定义一条通用规则，利用该规则可以从带有
旧后缀名的文件创建带有新后缀名的文件，并保留
原文件的前半部分。

用一个新的通用规则将.cpp文件编译为.o文件
```text
.SUFFIXES:	.cpp
.cpp.o:
	$(CC) -xc++ $(CFLAGS) -I$(INCLUDE) -c $<
```

-xc++标志的作用是告诉gcc编译器这是一个C++源文件

最新的make版本还包含一个新的语法以实现同
样的效果，而且功能更强大。例如，模式规则可以
用%通配符语法来匹配文件名，而不是仅依赖于文
件的后缀名。
```text
%.cpp: %o
	$(CC) -xc++ $(CFLAGS) -I$(INCLUDE) -c $<
```


### 用make 管理函数库

对于大型项目，一种比较方便的做法是用函数
库来管理多个编译产品。函数库实际上就是文件，
它们通常以.a（a是英文archive的首字母）为后缀
名，在该文件中包含了一组目标文件。make命令
用一个特殊的语法来处理函数库，这使得函数库的
管理工作变得非常容易。


用于管理函数库的语法是lib (file.o)，它的含义
是目标文件file.o是存储在函数库lib.a中的。make
命令用一个内置规则来管理函数库，该规则的常见形式如下所示

```text
.c.a:
	$(CC) -c $(CFLAGS) $<
	$(AR) $(ARFLAGS) $@ $*.o
```
宏`$(AR)`和 `$(ARFLAGS)` 的默认取值通常分别是
命令ar和选项rv。这个相当简洁的语法告诉
make，要想从.c文件得到.a库文件，它必须应用上
面两条规则:
- 第一条规则告诉它必须编译源文件以生成目标文件。
- 第二条规则告诉它用ar命令将新的目标文件添加到函数库中。

如果有一个名为fud的函数库，其中包含
目标文件bas.o，则第一条规则中的$<将被替换为
bas.c，而第二条规则中的`$@`和`$*`将被分别替换为
库文件fud.a和名字bas

。。修改的那部分。
```text
# local libraries
MYLIB = mylib.a

myapp: main.o $(MYLIB)
	$(CC) -o myapp main.o $(MYLIB)

$(MYLIB): $(MYLIB)(2.o) $(MYLIB)(3.o)
main.o: main.c a.h
2.o: 2.c a.h b.h
3.o: 3.c b.h c.h

clean:
	-rm main.o 2.o 3.o $(MYLIB)
```


### 9.2.9 高级主题: makefile文件和子目录

对于大型的项目，有时我们希望能把构成一个
函数库的几个文件从主文件中分离出来，并将它们
保存到一个子目录中。使用make命令完成这一工
作的方法有两个

第一个方法是，你可以在子目录中编写出第二
个makefile文件，它的作用是编译该子目录下的源
文件，并将它们保存到一个函数库中，然后将该库
文件复制到上一级的主目录中。在主目录中的
makefile文件包含一条用于制作函数库的规则，该
规则会调用第二个makefile文件
```text
mylib.c:
    (cd mylibdirectory:$(MAKE))
```
这就是说，你必须总是执行命令make
mylib.a。当make命令调用这条规则来创建函数库
时，它将切换到子目录mylibdirectory中，然后调
用一个新的make命令来管理函数库

第二个方法是，在原来的makefile文件中添加
一些宏。新添加的宏通过在我们已见过的宏的尾部
追加一个字母得到，字母D代表目录，字母F代表文
件名。然后你就可以用下面的规则来替换内置
的.c.o后缀规则
```text
.c.o:
    $(CC) $(CFLAGS) -c $(@D)/S(<F) -o $(@D)/$(@F)
```
这条规则的作用是：编译子目录中的源文件并
将目标文件放在该子目录中。然后，你用如下的依
赖关系和规则来更新当前目录下的函数库
```text
mylib.a:    mydir/2.o mydir/3.o
    ar -rv mylib.a $?
```

### GNU make 和 gcc

GNU的make命令和GNU的gcc编译器有下面两个有趣的选项
第一个选项是make命令的`-jN`（字母j是英文单词jobs的首字母）选
项，它允许make命令`同时执行N条命令`。如果`项目的不同部分可以彼此独立地进行编译`，
make命令就可以同时调用几条规则。根据系统的配置情况，这种做法可以
`极大地缩短`重新编译所需要花费的时间。如果有许多源文件，这个选项就值得一
试。一般来说，你可以先从较小的数字（比如-j3）开始尝试。但如果需要与其他
用户共享你的计算机，就要小心使用这个选项，因为其他用户可能并不喜欢你每次
编译时都启动大量的进程

另一个有用的选项是gcc的`-MM`选项。它的作用是`产生一个适用于make
命令的依赖关系清单`。如果某个项目包含非常多的源文件，每个源文件又包含不同
的头文件组合，则理清它们之间的依赖关系将非常困难（但又非常重要）。如果让
每个源文件都依赖于所有的头文件，有时候你就会编译一些没有必要编译的文件。
但从另一方面来看，如果忽略一些依赖关系，问题会变得更严重，因为你没有重新
编译一些需要编译的文件。


可以尝试使用makedepend工具，它的功能与-MM
选项很类似，但其做法是将依赖关系直接附加到指
定的makefile文件的末尾

只要是可以通过一系列命令从某些类型的
输入文件得到输出文件的任务，你都可通过
makefile文件来自动地完成。一个典型的“非编译
器”用途是，通过调用awk或sed命令对一些文件
进行处理，或甚至通过makefile文件来生成使用手册

另一种用于控制程序创建或完成其他自动化任
务的工具是ANT。它是一个基于Java的工具，它使
用基于XML的配置文件。这个工具并不常被用于自
动化处理Linux系统上C语言文件的创建
。。ANT。。


## 9.3 源代码控制

。。没有git。

## 编写手册页

- Header（标题）
- Name（名称）
- Synopsis（语法格式）
- Description（说明）
- Options（选项）
- Files（相关文件）
- See also（其他参考）
- Bugs（已知漏洞）

UNIX的手册页是通过工具nroff排版的，在大
多数Linux系统中，用于完成相同功能的工具为
groff，它是由GNU项目开发的。

。。代码

你可以将新的手册页放
置到一个本地手册页目录中，或者将其直接放到系
统目录/usr/man/man1中


## 发行软件

### patch

### tar
tar命令的如下6个选项的组合。
- c：创建新档案文件。
- f：指定目标为一个文件而不是一个设备。
- t：列出档案文件的内容，但并不真正释放它们。
- v（verbose）：显示tar命令执行的详细过程。
- x：从档案文件中释放文件。
- z：在GNU版本的tar命令中用gzip命令压缩档案文件。


## RPM软件包





# ch10 调试

## 错误类型
- 功能定义错误 : 
  在开始程序设计（或规划）之前，你==必须确认自己知道并理解这个程序究竟是用来干什么==的。认真==分析用户需求==并加强==和用户之间的沟通==，有助于查找和纠正许多（即使不是全部）程序功能定义方面的错误
- 设计规划错误 : 
  一定要==多花点时间思考：如何构造程序，需要什么样的数据结构，它又应该如何在程序中使用==。尽量把这些细节问题提前确定下来，这样将节省今后很多改写代码的时间
- 代码编写错误 : 
  在程序中遇到错误时，要重新阅读源代码或与其他人进行探讨，这个办法虽然因为简单而容易被人忽视，但它却非常有效。


## 常用调试技巧

一般做法是==先运行程序并观察其输出==结果，如
果不能正常工作，我们就需要决定应该釆取哪些措
施。可以==修改程序然后重新尝试==（代码检查-试运
行-出错法），也可以在程序中==增加一些语句来获
得更多关于程序内部运行情况的信息==（取样法），
还可以==直接检查程序的执行情况==（受控执行法）。

程序调试可以分为如下5个阶段。
- 测试：找出程序中存在的缺陷或错误。
- 固化：让程序的错误可重现。
- 定位：确定相关的代码行。
- 纠正：修改代码纠正错误。
- 验证：确定修改解决了问题。


如果想捕捉到数组访问方面的错误，最好增加
数组元素的大小，因为这样同时也增加了错误的大小。



### 代码检查
`gcc -Wall -pedantic -ansi`

我们建议大家养成使用这些选项的习惯，特别是-Wall选项。在追踪程
序的错误时，它可以产生非常有用的信息

稍后介绍lint和splint等工


### 取样法
在程序中添加一些代码以收集更多与程序运行时的行为相关的信息的方法

常见做法是，在程序中添加printf函数调用以打印出变量在程序运行的不同阶段的值

两种取样法的技巧
第一种技巧是用C语言的`预处理器`有选择地包括取样代码，
这样只需重新编译程序就可以包含或去除调试代
码。实现方法非常简单，只需使用如下的语句结构：
```C
#ifdef DEBUG
    printf("variable x's value: %d\n", x);
#endif
```
在编译程序时可以加上编译器标志`-DDEBUG`。如果加上这个标志，就定义了DEBUG符号

还可以用数值调试宏来完成更复杂的调试应用
```C
#define BASIC_DEBUG 1
#define EXTRA_DEBUG 2
#define SUPER_DEBUG 4

#if (DEBUG & EXTRA_DEBUG)
    printf("--");
#endif
```
在这种情况下，我们必须总是定义DEBUG宏，
但我们可以设置它为代表一组调试信息或代表一个
调试级别。比如编译器标志-DDEBUG=5将启用
BASIC_DEBUG和SUPER_DEBUG，但不包括
EXTRA_DEBUG。标志-DDEBUG=0将禁用所有的调试信息。

另外，也可以在程序中添加如下语句。
这样，当不需要调试时，就不必在命令行上定义DEBUG宏
```C
#ifndef DEBUG
#define DEBUG 0
#endif
```

C语言预处理器定义的一些宏可以帮助我们进行
调试。这些宏在扩展后会提供 ==当前编译操作== 的相关信息
|宏|说明|
|--|--|
|`__LINE__`|当前行号的十进制值|
|`__FILE__`|当前文件名的字符串|
|`__DATE__`|mmm dd yyyy 格式的 当前日期|
|`__TIME__`|hh:mm:ss 格式的当前时间|

。。看下面， ==编译时的时间==。。

```C
#include <stdio.h>
#include <stdlib.h>

int main()
{
#ifdef DEBUG
    printf("Compiled: " __DATE__ " @ " __TIME__ "\n");
#endif

    printf("hi\n");
    exit(0);
}
```

。。
$ gcc -DDEBUG 02_macro_DEBUG_printf.c 

$ date
2023年 09月 01日 星期五 10:14:52 CST

$ ./a.out 
Compiled: Sep  1 2023 @ 10:14:48
hi

$ ./a.out 
Compiled: Sep  1 2023 @ 10:14:48
hi
。。



---------------------
第二种就是无需重新编译的调试技巧

printf 的一个技巧，可以 无需 #ifdef DEBUG 语句

方法是在程序中增加一个作为==调试标志的全局变量==，
这使得用户可以在命令行上通过 `-d` 选项切换
是否启用调试模式，即使程序已经发布了，仍然可
以这样做，该方法同时还会在程序中增加一个用于
记录调试信息的函数。现在我们可以把如下的内容
加入到程序代码中
```C
if (debug) {
    sprintf(msg);
    write_debug(msg);
}
```

你应该将调试信息输出到标准错误输出stderr，
或者，如果因为程序的原因不能这样做，你还可以
使用syslog函数提供的日志功能。

好处体现在：当程序发布之后，如果用户遇到了问题，
他们自己就可以在运行程序时打开调试功能，自己
完成诊断错误的工作

一个明显的不足，就是程序的长度会有所增加。
但在大多数情况下，它只是一个表面问题，算不上是实际意义上的问题。
程序的长度可能会增加20％或30％，但往往并不会对程序的性能造成真正的影响。


### 程序的受控执行

较复杂的调试器可以在源代码级别
查看程序的比较详细的状态信息。GNU的调试器
==gdb==（可以在Linux和许多类UNIX系统中使用）就
可以做到这一点。目前有一些针对gdb的“前
端”程序，它们提供非常==友好的用户界面==，
xxgdb、KDbg和ddd都是这样的程序

一些IDE，比如我们在第9章介绍的，也提供了调试功能或一
个用于gdb的前端。==Emacs==编辑器甚至还提供了一
个功能（==gdb-mode==），允许用户在程序上运行
gdb，设置断点并查看现在执行到源代码中的哪一行。

为了能够调试程序，我们需要在编译它时加上
一个或多个特殊的==编译器选项==。这些选项的作用是
让编译器在程序中添加==额外的调试信息==。这些信息
包括各种==符号和源代码行号==，调试器将利用这些信
息向用户显示程序已经执行到源代码的位置。

`-g` 标志是对程序进行调试性编译时常用的一个
选项。我们必须在编译每个需要调试的源文件时都
加上这个选项，对链接器也要这样做（编译器会把
这个标志自动传递给链接器），它将使用特殊版本
的C语言标准库以提供库函数中的调试支持。对那
些在编译时没有加上调试功能的函数库，虽然调试
工作也能够进行，但灵活性就要差些。

命令`strip <file>`将可执行文件中的调试信息删除而不需要重新编译程序。


## 10.3 ==use gdb to debug==

### 启动gdb (gdb a.out)

`gdb a.out`

退出：`quit`

所有版本都支持“空命令”，即直接按下
回车键再次执行最近执行过的那条命令。在用step
或next命令单步执行程序时，这个“空命令”非常有用。


### 运行一个程序 (run)

```text
(gdb) run
Starting program: ch10_debug/a.out 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
Compiled: Sep  1 2023 @ 15:09:31
hi
```

程序运行失败时，gdb会报告出失败的原因及位置


### 栈跟踪 (backtrace/where)

出错的时候，执行 `backtrace` 。可以获得 调用栈信息

backtrace命令可以简写为 `bt`，
为了与其他调试器兼容，gdb还有一个命令 `where` 用来完成相同的功能


### 检查变量 (print)

`print varName`

gdb将命令的结果保存在伪变量 `$<number>`
中。最后一次操作的结果总是为 `$`，倒数第二次操
作的结果为 `$$`。这使得我们可以把某次操作的结果
用在另一个命令中

```text
print a[$ - 1].key
```


### 列出程序源码 (list)

如果没有显示完，可以再次 list，显示后续。

可以给list命令提供一个行号或函数名作为参数，它将显示指定位置前后的代码




### 设置断点 (break, cont)

`help breakpoint` 可以看到很多命令

在21行设置一个断点。
`break 21`

然后执行
`run`

`cont` 继续执行

`print arr[0]` 打印1个数据

`print arr[0]@5` 打印5个数组元素

`display arr[0]@5` display命令告诉gdb，每次程序停在断点位置时 自动显示数组的内容。

`commands` 指定在程序到达断点位置时需要执行的调试器命令。因为我们已设置了display命令，所以只需设置断点命令为继续执行即可
```text
commands
cont
end
```
由于 已经display 设置 在断点时 自动显示数组内容， 所以这里 设置为 继续 即可。

gdb报告这个程序在退出时带有一个不常见的退
出码，这是因为程序本身未调用exit函数，并且也
没有从main函数返回一个值。本例中的退出码没有
实际意义，只有exit函数才会提供有意义的退出码。


### 用调试器打补丁

我们可以通过调试器设置断点
和查看变量的取值。通过将断点的设置与相应的操
作结合起来，就可以尝试修改程序（也被称为打补
丁）而不需要改变程序的源代码并重新编译

重新开始执行这个程序。首先，必须删除刚才设置的断点和display命令的内容。

`info display`
`info break` 查看 已设置的 断点和 display 的内容。

`disable break 1`
`disable display 1` 禁用

`break 30`
```text
commands 2
set variable n = n + 1
cont
end
```
在 30行上设置断点； 修改 变量n。


### 深入学习gdb

在支持硬件断点功能的系统上，可以用gdb实时监控变量取值的变化情况

gdb还可以监控表达式，即当某个表达式取了一个特定值时，gdb可以暂停程序的运行

断点可以和计数、条件结合在一起设置，只有在经过了指定的次数或满足某个条件时才触发断点

gdb还可以将其自身附在已经运行的程序上。这
对调试客户／服务器系统很有帮助，因为你可以在
异常服务器正在==运行时对其进行调试==

可以用gdb来调试已经崩溃的程序。程序
运行失败时，Linux和UNIX系统通常会产生一个核
心转储（core dump），并将它保存在core文件
中。这个文件其实是程序的内存映像文件，它包含
程序在运行失败的那个时刻的全局变量的取值。你
可以用gdb找出程序发生崩溃的位置


## 10.4 其它调试工具
静态分析工具
- ctags
- cxref
- cflow

动态分析
- prof
- gprof

ctags、cxref和cflow这3个工具构成了X/Open规范的一部分内容


--------------------

ctags
ctags为程序中的所有函数创建索引。每个函数
对应一个列表，在列表中列出该函数在程序中的调
用位置，就像书籍的索引
```text
ctags [-a] [-f filename] sourcefile sourcefile ...
ctags -x sourcefile sourcefile ...
```

默认情况下，ctags在当前目录下创建文件
tags。在该文件中包含每个输入源文件中声明的每
个函数，文件中每行的格式如下所示
`announce   app.c   /^static void announce(void) /`


ctags的-x选项（如果你使用
的版本有该选项）在标准输出上列出类似上面格式的内容

。。
$ ctags -x 03_bug.c 
f                  5 03_bug.c         int f(
main              10 03_bug.c         int main(
。。
。。和书上一样。

- -f 选项将输出重定向到另一个不同的文件中
- -a 选项将输出结果附加到一个已有文件的结尾


---------------------

cxref
cxref程序分析C语言源代码并生成一个交叉引
用表。它可以显示每个符号（变量、#define定义
和函数）都在程序的哪个位置使用过。它生成的是
一个经过排序的列表，每个符号的定义位置用一个
星号（*）做标记

。。cxref  ubuntu22 没有预装。。试了下，命令不行。

这个命令的正确语法格式随版本的不同而不
同。请参考系统文档或手册页来了解cxref命令的更
多信息


----------------

cflow

cflow程序打印出一个函数调用树（function
call tree），它显示了函数之间调用的关系。它可
以让我们看清楚一个程序的框架结构，理解它的操
作流程，了解对某个函数的改动将会产生怎样的影
响。有些版本的cflow除了可以处理源代码外，还
可以处理目标文件。详细的用法请参考它的手册
页。


。。ubuntu 22 没有预装
。。
$ cflow 03_bug.c 
main() <int main () at 03_bug.c:10>:
    printf()
    f() <int f (int a) at 03_bug.c:5>
。。


cflow有一个-i选项，它将产生一个反向的函数调用树



### ==prof/gprof== 产生执行存档

想查找程序的性能问题时，一种常用的技巧是
使用执行存档（execution profiling）。它通常需
要特殊的编译器选项和辅助程序的支持。程序的执
行存档可以显示执行它所==花费的时间具体都用在什
么操作==上了

编译程序时，给编译器加上`-p`标志（针对prof
程序）或`-pg`标志（针对gprof程序）就可以创建出
profile程序。而prof程序（及其GNU等效程序
gprof）可以根据profile程序运行时所产生的执行
跟踪文件打印出一个报告

`cc -pg -o xxxx xxxx.c`

程序用特殊版本的C语言函数库链接起来并且将
包括监控代码。不同的系统具体实现方法有所不
同，但一般都要靠程序的频繁中断来记录执行地
点。监控数据将被写入当前目录下的文件
mon.out（gprof程序用的是gmon.out）

。。
$ cc -pg 02_macro_DEBUG_printf.c 
$ ./a.out 
hi
$ ls
02_macro_DEBUG_printf.c  03_bug.c  a.out  gmon.out  my
$ gprof
Flat profile:

Each sample counts as 0.01 seconds.
 no time accumulated

  %   cumulative   self              self     total           
 time   seconds   seconds    calls  Ts/call  Ts/call  name    
。。 代码太短了，没什么输出。 但是感觉也不应该 表格里为空吧。



## 断言
X/Open提供了assert宏，它的作用是测
试某个假设是否成立，如果不成立就停止程序的运行。

```C
#include <assert.h>

void assert(int expression)
```

assert宏对表达式进行求值，如果结果==非零==，它
就往标准错误写一些诊断信息，然后调用abort函
数结束程序的运行

头文件assert.h定义的宏受NDEBUG的影响。
如果程序在处理这个头文件时已经定义了
NDEBUG，就不定义assert宏。这意味着，你可以
在编译期间使用-DNDEBUG关闭断言功能或把下
面这条语句
`#define NDEBUG` 加到每个源文件中，但这条语句必须放在
`#include <assert.h>`语句之前。

注意不要让assert表达式带上副作用


## 10.6 内存调试

### ElectricFence
。。不是 Elastic

```C
#include <stdio.h>
#include <stdlib.h>

int main()
{
    char *ptr = (char *) malloc(1024);
    ptr[0] = 0;

    ptr[1024] = 0;      // yue jie
    exit(0);
}
```

`cc -o efence efence.c -lefence`
与ElectricFence函数库libefence.a链接起来
运行的时候会收到报错信息

。。没有 efence，不试了。
。。看网上说， electricfence 是比较轻量级的。 valgrind 和 AddressSanitizer 是比较强大的。

### valgrind

`valgrind --leak-check=yes -v ./a.out`

。。没有。不过可以 直接 snap 下载安装。

`valgrind --help`


# **==ch11 进程和信号==**

我们将介绍以下几方面的内容:
- 进程的结构、类型和调度
- 用不同的方法启动新进程
- 父进程、子进程和僵尸进程
- 什么是信号以及如何使用它们


## 什么是进程

UNIX标准(特别是IEEE Std 1003.1,2004年
版)把==进程定义为:“一个其中运行着一个或多个
线程的地址空间和这些线程所需要的系统资源。”==

像Linux这样的多任务操作系统可以同时运行多个程序。每个运行着的程序实例就构成一个进程

正在运行的程序或进程由程序代码、数据、变量(占用着系统内存)、
打开的文件(文件描述符)和环境组成。一般来
说,Linux系统会在进程之间共享程序代码和系统
函数库,所以在任何时刻内存中都只有代码的一份副本。


## 进程的结构

。。一个例子，2个用户 同时执行 各自的 grep 命令

每个进程都会被分配一个唯一的数字编号,我
们称之为进程标识符或PID。它通常是一个取值范
围从2到32768的正整数

当数字已经回绕一圈时,新的PID重新从2开
始。数字1一般是为特殊进程init保留的,init进程
负责管理其他进程

将要被grep命令执行的程序代码被保存在一个
磁盘文件中。正常情况下,Linux进程不能对用来
存放程序代码的内存区域进行写操作,即程序代码
是以只读方式加载到内存中的

系统函数库也可以被共享。例如,不管有多少
个正在运行的程序要调用printf函数,内存中只要
有它的一份副本即可。

把常用例程提取出来放入(比如说)C语言
的标准函数库中将节省大量的磁盘空间。


传递给grep程序的搜索字符串以变量s的形式出现在==每个进程的数
据区==中。它们之间是分离的,通常不能被其他进程读取。

==进程有自己的栈空间==,用于==保存函数中的局部变量和控制函数的调用与返回==。进程还
有==自己的环境空间==,包含专门为这个进程建立的环境变量

。。线程也有自己的栈吧。

在许多Linux系统(也包括一些UNIX系统)
上,在目录/proc中有一组特殊的文件,这些文件
的特殊之处在于它们允许你“窥视”正在运行的进
程的内部情况,就好像这些进程是目录中的文件一
样。


### 进程表

Linux进程表就像一个数据结构,它把当前加载在内存中的所有进程的有关信息保存在一个表中,
其中包括==进程的PID、进程的状态、命令字符串和其他一些ps命令输出的各类信息==。

操作系统通过进程的PID对它们进行管理,这些PID是进程表的索引。

进程表的长度是有限制的,所以系统能够支持的同时运行的进程数也是有限制的。
早期的UNIX系统只能同时运行256个进程。最新的实现版本已
大幅度放宽这一限制,可以同时运行的进程数可能
只与用于建立进程表项的内存容量有关,而没有具体的数字限制了。


### 查看进程 (ps)

`ps -ef`

TTY一列显示了进程是从哪一个终端启动的,
TIME一列是进程目前为止所占用的CPU时间,
CMD一列显示启动进程所使用的命令。

默认情况下,ps程序只显示与终端、主控台、
串行口或伪终端保持连接的进程的信息。
其他进程
在运行时不需要通过终端与用户进行通信,它们通
常都是一些系统进程,Linux用它们来管理共享的
资源。
我们可以用ps命令的-a选项查看所有的进
程,用-f选项显示进程完整的信息。


### 系统进程

`ps ax`

|stat代码|说明|
|--|--|
|S|睡眠，通常是在等待某个事件的发生，如一个信号或有输入可用|
|R|运行，严格说来，是"应运行"，即在运行队列中，处于正在执行或即将运行状态|
|D|不可中断的睡眠(等待),通常是在等待输入或输出完成|
|T|停止。通常是被shell作业监控 所停止，或进程正处于 调试器的控制之下|
|Z|死进程或 僵尸进程|
|N|低优先级任何，nice|
|W|分页。(不适用于2.6版本开始的linux内核)|
|s|进程是会话期首进程|
|+|进程属于前台进程组|
|l|进程是多线程的|
|<|高优先级任务|


。。
[qwer@192 2613]$ ps ax
    PID TTY      STAT   TIME COMMAND
      1 ?        Ss     0:01 /usr/lib/systemd/systemd --switched-root --system --deserialize 30
      2 ?        S      0:00 [kthreadd]
      3 ?        I<     0:00 [rcu_gp]
      4 ?        I<     0:00 [rcu_par_gp]
      6 ?        I<     0:00 [kworker/0:0H-kblockd]
      7 ?        I      0:00 [kworker/0:1-rcu_par_gp]
      9 ?        I<     0:00 [mm_percpu_wq]
     10 ?        S      0:00 [ksoftirqd/0]
。。


一个非常重要的进程：
`1 ? Ss 0:03 init[5]`

。。这个是书上的。

一般而言,==每个进程都是由另一个我们称之为
父进程的进程启动的==,被父进程启动的进程叫做子
进程。Linux系统启动时,它将运行一个名为init的进程,该进程是系统运行的第一个进程,它的进程
号为1。你可以把init进程看作为操作系统的进程管
理器,它是其他所有进程的祖先进程。我们将要看
到的其他系统进程要么是由init进程启动的,要么
是由被init进程启动的其他进程启动的。


用户登录的处理过程就是一个这样的例子。init
进程为每个用户用来登录的串行终端或拨号调制解
调器启动一次getty程序。对应的ps命令输出如下
所示:
`9523   tty2    Ss+     0:00    /sbin/mingetty  tty2`

。。fedora没有 getty， mingetty需要安装。

getty进程等待来自终端的操作,向用户显示熟
悉的登录提示符,然后把控制移交给登录程序,登
录程序设置用户环境,最后启动一个shell。用户退
出系统时,init进程将再次启动另一个getty进程

启动新进程并等待它们结束的能力是整个系统
的基础。我们将在本章的后面看到如何从自己的程
序中用系统调用fork、exec和wait来完成同样的任
务。


### 进程调度

Linux内核用进程调度器来决定下一个时间片应
该分配给哪个进程。它的判断依据是进程的优先级

。。我在想，这个优先级是不是可以直接插队。因为上面提到了队列(stat代码的R代表了在任务队列中)
。。为了防止饥饿，肯定不能 多个任务队列。
。。所以 优先级低的直接 队尾， 优先级高的 队中。
。。我想起来一个 3/8 ，但我忘记是什么方面的了， 感觉是 线程 之类的。 就是3/8 的地方 分为 段， 这样的话，2个queue。
。。优先级高的 3/8处， 低的最后。 但是 多个优先级，不可能3/8 一个。
。。查了下。。mysql 的 LRU 队列的实现，是 3/8 。。

多个程序可能会竞争使用同一个资源。在这种情况下,执行
短期的突发性工作并暂停运行来等待输入的程序,
要比持续占用处理器来进行计算或不断轮询系统来
查看是否有新的输入到达的程序要更好。

我们称表现良好的程序为nice程序,而且在某种意义上,这
个nice是可以被计算出来的。

操作系统根据进程的nice值来决定它的优先级,一个进程的nice值默认
为0并将根据这个程序的表现而不断变化。
长期不间断运行的程序的优先级一般会比较低。而(例如)暂停来等待输入的程序会得到奖励。

nice命令设置进程的nice值,使用renice命令调整它的值。nice命令是
将进程的nice值增加10,从而降低该进程的优先级。
。。增加nice值， 降低优先级

`ps -l`
`ps -f`
可以查看 nice 值 (NI列)

。。ps -f 没有 NI， -l 有。

用下面的命令启动oclock， 它将被分配 一个 +10 的nice值
`nice oclock &`

`renice 10 {pid}`


ps命令输出中的 ==PPID栏给出的是父进程的进程ID==,它是启动这个进程的进程的PID。如果原来的
父进程已经不存在,该栏显示的就是init进程的进程ID(PID为1)

Linux调度器根据进程的优先级来决定运行哪个
进程。每个系统的具体实现各有不同,但高优先级的进程总是运行得更频繁。某些情况下,只要还有
高优先级的进程可以运行,低优先级的进程就根本不能运行。



## 11.3 启动新进程 (system,)

```C
#include <stdlib.h>

int system(const char *string);
```

system函数的作用是,运行以字符串参数的形
式传递给它的命令并等待该命令的完成。命令的执
行情况就如同在shell中执行如下的命令
`sh -c string`

如果无法启动shell来运行这个命令,system函
数将返回错误代码`127`;如果是其他错误,则返
回`-1`。否则,system函数将返回该命令的退出码。


```C
#include <stdlib.h>
#include <stdio.h>

int main()
{
    printf("run ps with system\n");
    
    system("ps ax");

    printf("done\n");
    exit(0);
}
```

使用system函数远非启动其他进
程的理想手段,因为它必须用一个shell来启动需
要的程序。由于在启动程序之前需要先启动一个
shell,而且对shell的安装情况及使用的环境的依赖也很大,所以使用system函数的效率不高。



### 替换进程映像 (execl/v,p,e)

exec系列函数由一组相关的函数组成,它们在
进程的启动方式和程序参数的表达方式上各有不
同。exec函数可以==把当前进程替换为一个新进程==,
新进程由path或file参数指定。你可以使用exec函
数将程序的执行从一个程序切换到另一个程序。

exec函数比system函数更有效,因为在新的程序启动后,原来的程序就不再运行了

```C
#include <unistd.h>

char **environ;

int execl(const char *path, const *arg0, ..., (char *) 0);
int execlp(const char *file, const char *arg0, ..., (char *)0);
int execle(const char *path, const char *arg0, ..., (char *)0, char *const envp[]);
int execv(const char *path, char *const argv[]);
int execvp(const char *path, char *const argv[]);
int execve(const char *path, char *const argv[], char *const envp[]);
```

分为两大类。execl、execlp和
execle的参数个数是可变的,参数以一个空指针结
束。execv和execvp的第二个参数是一个字符串数
组。不管是哪种情况,新程序在启动时会把在argv数组中给定的参数传递给main函数。
这些函数通常都是用execve实现的,虽然并不是必须要这样做。

以字母p结尾的函数通过搜索`PATH环境变量`来
查找新程序的可执行文件的路径。如果可执行文件
不在PATH定义的路径中,我们就需要把包括目录在
内的使用绝对路径的文件名作为参数传递给函数。

全局变量environ可用来把一个值传递到新的程
序环境中。此外,函数execle和execve可以通过参
数envp传递字符串数组作为新程序的环境变量。

如果想通过exec函数来启动ps程序,我们可以
从6个exec函数中选择一个,如下面的代码片段所示:
```C
#include <unistd.h>

char *const ps_argv[] = {"ps", "ax", 0};
char *const ps_envp[] = {"PATH=/bin:/usr/bin", "TERM=console", 0};

execl("/bin/ps", "ps", "ax", 0);
execlp("ps", "ps", "ax", 0);
execle("/bin/ps", "ps", "ax", 0, ps_envp);

execv("/bin/ps", ps_argv);
execvp("ps", ps_argc);
execve("/bin/ps", ps_argv, ps_envp);
```

```C
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

int main()
{
    printf("run ps with execlp\n");
    
    execlp("ps", "ps", "ax", 0);

    printf("done\n");       // won't print, for process has been replaced
    exit(0);
}
```

ps命令结束时,我们看到一个新的shell提示
符,因为我们并没有再返回到pexec程序中,所以
第二条消息是不会打印出来的。新进程的PID、PPID和nice值与原先的完全一样。事实上,这里发
生的一切其实就是,运行中的程序开始执行exec调
用中指定的新的可执行文件中的代码。

对于由exec函数启动的进程来说,它的参数表
和环境加在一起的总长度是有限制的。上限由
ARG_MAX给出,在Linux系统上它是128K字节

一般情况下,exec函数是不会返回的,除非发
生了错误。出现错误时,exec函数将返回-1,并且
会设置错误变量errno

由exec启动的新进程继承了原进程的许多特
性。特别地,==在原进程中已打开的文件描述符在新
进程中仍将保持打开,除非==它们的“执行时关闭标
志”(close on exec flag)被置位(详细说明请
参考第3章中对fcnt1系统调用的介绍)。任何在原
进程中已打开的==目录流都将在新进程中被关闭==。


### 复制进程印象 (fork)

让进程同时执行多个函数,我们可以使用
==线程==(将在第12章介绍)或从==原程序中创建一个完全分离的进程==,
后者就像init的做法一样,而不像exec调用那样用新程序替换当前执行的线程。

调用fork创建一个新进程。这个系统调用复制当前进程,在进程表中创建一个新的
表项,新表项中的许多属性与当前进程是相同的。
新进程几乎与原进程一模一样,执行的代码也完全相同,但新进程有自己的数据空间、环境和文件描
述符。
fork和exec函数结合在一起使用就是创建新进程所需要的一切了。

在父进程中的fork调用==返回的是新的子进程的PID==。新进程将继续执行,就像原
进程一样,不同之处在于,==子进程中的fork调用返
回的是0==。父子进程可以==通过这一点来判断究竟谁
是父进程,谁是子进程==

如果fork失败,它将返回-1。失败通常是因为
父进程所拥有的子进程数目超过了规定的限制
(CHILD_MAX),此时errno将被设为EAGAIN。
如果是因为进程表里没有足够的空间用于创建新的
表单或虚拟内存不足,errno变量将被设为
ENOMEM。

```C
#include <sys/types.h>
#include <unistd.h>

pid_t fork(void);
```

```C

#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

int main()
{
    pid_t pid;
    char *msg;
    int n;

    printf("fork starting\n");

    pid = fork();

    switch(pid)
    {
    case -1:
        perror("fork failed");
        exit(1);
    case 0:
        msg = "this is child";
        n = 5;
        break;
    default:
        msg = "this is parent";
        n = 3;
        break;
    }
    for (; n > 0; --n)
    {
        puts(msg);
        sleep(1);
    }
    exit(0);
}
```

以两个进程的形式在运行。子进程被
创建并且输出消息5次。原进程(即父进程)只输
出消息3次。父进程在子进程打印完它的全部消息
之前就结束了,因此我们将看到在输出内容中混杂
着一个shell提示符

程序在调用fork时被分为两个独立的进程。程
序通过fork调用返回的非零值确定父进程,并根据
该值来设置消息的输出次数,两次消息的输出之间间隔一秒。


### 等待一个进程 (wait)

```C
#include <sys/types.h>
#include <sys/wait.h>

pid_t wait(int *stat_loc);
```

wait系统调用将暂停父进程直到它的子进程结
束为止。这个调用返回子进程的PID,它通常是已
经结束运行的子进程的PID。状态信息允许父进程
了解子进程的退出状态,即子进程的main函数返回
的值或子进程中exit函数的退出码。如果stat_loc不
是空指针,状态信息将被写入它所指向的位置。


可以用sys/wait.h文件中定义的宏来解释状态信息
|宏|说明|
|--|--|
|WIFEXITED()|如果子进程正常结束，返回非0值|
|WEXITSTATUS()|如果wifexited非0，返回子进程的退出码|
|WIFSIGNALED()|如果子进程因为一个未捕获的信号而终止，返回非0|
|WTERMSIG()|如果wifsignaled非0，返回信号代码|
|WIFSTOPPED()|如果子进程意外终止，返回非0|
|WSTOPSIG|如果上面非0，返回一个信号代码|


```C
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

int main()
{
    pid_t pid;
    char *msg;
    int n;
    int exit_code;

    printf("fork starting\n");

    pid = fork();

    switch(pid)
    {
    case -1:
        perror("fork failed");
        exit(1);
    case 0:
        msg = "this is child";
        n = 5;
        exit_code = 37;
        break;
    default:
        msg = "this is parent";
        n = 3;
        exit_code = 0;
        break;
    }
    for (; n > 0; --n)
    {
        puts(msg);
        sleep(1);
    }
    if (pid != 0)
    {
        int stat_val;
        pid_t child_pid;

        child_pid = wait(&stat_val);

        printf("child has finished. pid = %d\n", child_pid);

        printf("aaaaaaa %d\n", stat_val);

        if (WIFEXITED(stat_val))
            printf("child exited with code %d\n", WEXITSTATUS(stat_val));       // 37
        else
            printf("child terminated abnormally\n");
    }
    exit(exit_code);
}
```



### 僵尸进程 (waitpid)

子进程终止时,它与父进程之间的关联还会保持,直到父进程也正常终止或父进程调用wait才告结束。
因此,进程表中代表子进程的表项不会立刻释放。虽然子进程已经不再运行,
但它仍然存在于系统中,因为它的退出码还需要保存起来,以备父进程今后的wait调用使用。这时它
将成为一个死(defunct)进程或僵尸(zombie)进程。

。。对掉了 输出的次数， 及 父进程输出5次， 子进程输出3次。
。在子进程输出完后， 父进程完成前， ps -al 可以看到 子进程的 CMD 是 defunct 或 zombie

如果此时父进程异常终止,子进程将自动把PID为1的进程(即init)作为自己的父进程。子进程现
在是一个不再运行的僵尸进程,但因为其父进程异常终止,所以它由init进程接管。僵尸进程将一直
保留在进程表中直到被init进程发现并释放。进程表越大,这一过程就越慢。应该尽量避免产生僵尸
进程,因为在init清理它们之前,它们将一直消耗系统的资源。

还有另一个系统调用可用来等待子进程的结
束,它是waitpid函数。你可以用它来==等待某个特定进程的结束==。

。。是任意一个进程？ 还是 某个子进程？

```C
#include <sys/types.h>
#include <sys/wait.h>

pid_t waitpid(pid_t pid, int *stat_loc, int options);
```

pid参数指定需要等待的子进程的PID。如果它
的值为-1,waitpid将返回任一子进程的信息。与
wait一样,如果stat_loc不是空指针,waitpid将把
状态信息写到它所指向的位置。option参数可用来
改变waitpid的行为,其中最有用的一个选项是
WNOHANG,它的作用是防止waitpid调用将调用
者的执行挂起。你可以用这个选项来查找是否有子
进程已经结束,如果没有,程序将继续执行。其他
的选项和wait调用的选项相同。

如果想让父进程周期性地检查某个特定
的子进程是否已终止,就可以使用如下的调用方式:
`waitpid(child_pid, (int)* 0, WNOHANG);`

如果子进程没有结束或意外终止,它就返回0,
否则返回child_pid。如果waitpid失败,它将返
回-1并设置errno。失败的情况包括:没有子进程
(errno设置为ECHILD)、调用被某个信号中断
(EINTR)或选项参数无效(EINVAL)。


### 输入和输出重定向

已打开的文件描述符将在fork和exec调用之后
保留下来,我们可以利用对进程这方面知识的理解
来改变程序的行为。

下一个例子涉及一个过滤程
序:它从标准输入读取数据,然后向标准输出写数
据,同时在输入和输出之间对数据做一些有用的转换。

```C
#include <unistd.h>
#include <stdio.h>
#include <ctype.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
    char *filename;

    if (argc != 2)
    {
        fprintf(stderr, "use upper file\n");
        exit(1);
    }

    filename = argv[1];

    if (!freopen(filename, "r", stdin))
    {
        fprintf(stderr, "could not redirect stdin from file %s\n", filename);
        exit(2);
    }
    execl("./upper", "upper", 0);

    // 如果没有错误，下面的不会执行。
    perror("could not exec ./upper\n");
    exit(3);


    // 把下面的编译成 upper， 让上面的调用。
    // int ch;
    // while ((ch = getchar()) != EOF) // ctrl+d 退出
    // {
    //     putchar(toupper(ch));
    // }
    // exit(0);
}
```

`./a.out file.txt`


useupper程序用freopen函数先关闭标准输
入,然后将文件流stdin与程序参数给定的文件名关
联起来。接下来,它调用execl用upper程序替换掉
正在运行的进程代码。因为已打开的文件描述符会
在execl调用之后保留下来,所以upper程序的运行
情况和它在shell提示符下的运行情况完全一样



### 线程 (没用)
Linux系统中的进程可以互相协作、互相发送消
息、互相中断,甚至可以共享内存段。但从本质上
来说,它们是操作系统内各自独立的实体,要想在
它们之间共享变量并不是很容易。
在许多UNIX和Linux系统中都有一类进程叫做
线程(thread)。涉及线程的编程是比较困难的,
但它在某些应用软件(如多线程数据库服务器)中
又有很大的用处。在Linux(或UNIX)系统中编写
线程程序并不像编写多进程程序那么常见,因为Linux中的进程都是非常轻量级的,而且编写多个
互相协作的进程比编写线程要容易得多。我们将在
第12章介绍线程。



## 信号

信号是UNIX和Linux系统响应某些条件而产生的一个事件。
接收到该信号的进程会相应地采取一些行动。

我们用术语生成(raise)表示一个信号的产生,使用术语捕获(catch)表示接收到一个信号。

信号是由于某些错误条件而生成的,如内存段
冲突、浮点处理器错误或非法指令等。
它们由shell和终端处理器生成来引起中断,它们还可以作为在
进程间传递消息或修改行为的一种方式,明确地由
一个进程发送给另一个进程。
无论何种情况,它们
的编程接口都是相同的。

信号可以被生成、捕获、响应或(至少对于一些信号)忽略。

信号的名称是在头文件signal.h中定义的。它们
以SIG开头

|信号|说明|
|--|--|
|SIGABORT|进程异常终止|
|SIGALRM|超时警告|
|SIGFPE|浮点运算异常|
|SIGHUP|连接挂断|
|SIGILL|非法指令|
|SIGINT|终端中断|
|SIGKILL|终止进程(此信号不能被捕获或忽略)|
|SIGPIPE|向无读进程的管道写数据|
|SIGQUIT|终端退出|
|SIGSEGV|无效内存段访问|
|SIGTERM|终止|
|SIGUSR1|用户定义信号1|
|SIGUSR2|用户定义信号2|

如果进程接收到这些信号中的一个,但事先没
有安排捕获它,进程将会立刻终止。通常,系统将
生成核心转储文件core,并将其放在当前目录下
该文件是进程在内存中的映像,它对程序的调试很有用处。


向运行在另一个终端上的PID为512的进程发送“挂断”信号,可以使用如下命令:
`kill -HUP 512`

kill命令有一个有用的变体叫killall,它可以给运
行着某一命令的所有进程发送信号。并不是所有的
UNIX系统都支持它,但Linux系统一般都有该命令。

通知inetd程序重新读取它的配置选项,要完成这一工作,可以使用下面这条命令
`killall -HUP inetd`

。。inetd是监视一些网络请求的守护进程

程序可以用signal库函数来处理信号
```C
#include <signal.h>

void (*signal(int sig, void (*func)(int)))(int);
```

这个相当复杂的函数定义说明,signal是一个带有sig和func两个参数的函数。
准备捕获或忽略的
信号由参数sig给出,接收到指定的信号后将要调用的函数由参数func给出。
信号处理函数必须有一个
int类型的参数(即接收到的信号代码)并且返回类型为void。
signal函数本身也返回一个同类型的函
数,即先前用来处理这个信号的函数,或者也可以下面的2个值之一来代替信号处理函数
- SIG_IGN   忽略信号
- SIG_DFL   恢复默认行为


编写一个程序ctrlc.c,它将响应用
户敲入的Ctrl+C组合键,在屏幕上打印一条适当的
消息而不是终止程序的运行。当用户第二次按下
Ctrl+C时,程序将结束运行

```C
#include <signal.h>
#include <stdio.h>
#include <unistd.h>

// 信号出现时,程序调用该函数,它先打印一条
// 消息,然后将信号SIGINT(默认情况下,按下
// Ctrl+C将产生这个信号)的处理方式恢复为默认行
// 为。
void ouch(int sig)
{
    printf("got signal: %d\n", sig);
    (void) signal(SIGINT, SIG_DFL);
}

int main()
{
    (void) signal(SIGINT, ouch);
    while (1)
    {
        printf("hi\n");
        sleep(1);
    }
}
```

在信号处理函数中,调用如printf这样的函数
是不安全的。一个有用的技巧是,在信号处理
函数中设置一个标志,然后在主程序中检查该标
志,如需要就打印一条消息。在本章的结尾部
分,你将会看到一个函数列表,表中的函数都可
以在信号处理函数中被安全地调用。


==不推荐大家使用signal接口==。之所以会在
这里介绍它,是因为你可能会在许多老程序中看到
它的应用。稍后我们会介绍一个定义更清晰、
执行==更可靠的函数sigaction==,在所有的新程序中
都应该使用这个函数。


signal函数返回的是先前对指定信号进行处理的
信号处理函数的函数指针,如果未定义信号处理函
数,则返回SIG_ERR并设置errno为一个正数值。
如果给出的是一个无效的信号,或者尝试处理的信
号是不可捕获或不可忽略的信号(如SIGKILL),
errno将被设置为EINVAL。


### 发送信号 (kill)

进程可以通过调用kill函数向包括它本身在内的
其他进程发送一个信号。如果程序没有发送该信号
的权限,对kill函数的调用就将失败,失败的常见原
因是目标进程由另一个用户所拥有。这个函数和同
名的 shell命令 完成相同的功能

```C
#include <sys/types.h>
#include <signal.h>

int kill(pid_t pid, int sig);
```

kill函数把参数sig给定的信号发送给由参数pid
给出的进程号所指定的进程,成功时它返回0。要
想发送一个信号,==发送进程必须拥有相应的权限==。
这==通常意味着两个进程必须拥有相同的用户ID==
(即你只能发送信号给属于自己的进程,但超级用户可
以发送信号给任何进程)。

kill调用会在失败时返回-1并设置errno变量。失败的原因可能是:
- 给定的信号无效(errno设置为EINVAL);
- 发送进程权限不够(errno设置为EPERM);
- 目标进程不存在(errno设置为ESRCH)。

信号向我们提供了一个有用的闹钟功能。进程
可以通过调用alarm函数在经过预定时间后发送一
个SIGALRM信号
```C
#include <unistd.h>

unsigned int alarm(unsigned int seconds);
```

alarm函数用来在seconds秒之后安排发送一个
SIGALRM信号。但由于处理的延时和时间调度的
不确定性,实际闹钟时间将比预先安排的要稍微拖
后一点儿。把参数seconds设置为0将取消所有已
设置的闹钟请求。

alarm函数的返回值
是以前设置的闹钟时间的余留秒数,如果调用失败
则返回-1。


为了说明alarm函数的工作情况,我们通过使用
fork、sleep和signal来模拟它的效果。程序可以启
动一个新的进程,它专门用于在未来的某一时刻发送一个信号。

```C
#include <sys/types.h>
#include <signal.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

static int alarm_fired = 0;

void ding(int sig)
{
    alarm_fired = 1;
}

int main()
{
    pid_t pid;
    printf("alarm app starting\n");

    pid = fork();
    switch(pid)
    {
    case -1:
        perror("fork failed...\n");
        exit(1);
    case 0:
        // child
        sleep(2);
        kill(getppid(), SIGALRM);
        exit(0);
    }

    printf("ppid wait alarm\n");
    (void) signal(SIGALRM, ding);

    pause();
    if (alarm_fired)
        printf("Ding\n");
    
    printf("done\n");
    exit(0);
}
```

父进程通过一个signal调用安排好捕获SIGALRM信号的工作,然后等待它的到来

pause,它的作用很简单,就是把程序的执行挂起直到有一个信号出现为止。

```C
#include <unistd.h>

int pause(void);
```

当它被一个信号中断时,将返回-1(如果下一
个接收到的信号没有导致程序终止的话)并把
errno设置为EINTR。当需要等待信号时,一个更常
见的方法是使用稍后将要介绍的sigsuspend函数。

闹钟模拟程序通过fork调用启动新的进程。这个子进程休眠5秒后向其父进程发送一个SIGALRM信号。
父进程在安排好捕获SIGALRM信号后暂停运行,直到接收到一个信号为止。
我们并未在信号处理函数中直接调用printf,而是通过在该函数中设置标志,然后在main函数中检查该标志来完成消息的输出。

使用信号并挂起程序的执行是Linux程序设计中的一个重要部分。
这意味着程序不需要总是在执行着。程序不必在一个循环中无休止地检查某个事件是否已发生,相反,它可以等待事件的发生。


程序中信号的使用将带来一个特殊的
问题:“如果信号出现在系统调用的执行过程中会
发生什么情况?”答案是相当让人不满意的“视情
况而定”。

在编写程序中处理信号部分的代码时必须非常小心,因为在使用信号的程序中会出现各种各样
的“==竞态条件==”。例如,如果想调用pause等待一
个信号,可信号却出现在调用pause之前,就会使
程序无限期地等待一个不会发生的事件。这些竞态
条件都是一些对时间要求很苛刻的问题,许多编程
新手都有这方面的烦恼,所以在检查和信号相关的
代码时总是要非常小心。



#### 一个健壮的信号接口 (sigaction)

```C
#include <signal.h>

int sigaction(int sig, const struct sigaction *act, struct sigaction *oact);
```

sigaction结构定义在文件signal.h中,它的作用
是定义在接收到参数sig指定的信号后应该采取的行
动。该结构至少应该包括以下几个成员:
- void (*) (int) sa_handler     // function, SIG_DFL or SIG_IGN
- sigset_t sa_mask      // signal to block in sa_handler
- int sa_flags      // signal action modifiers

sigaction函数设置与信号sig关联的动作。如果
oact不是空指针,sigaction将把原先对该信号的动
作写到它指向的位置。如果act是空指针,则
sigaction函数就不需要再做其他设置了,否则将在该参数中设置对指定信号的动作。

与signal函数一样,sigaction函数会在成功时
返回0,失败时返回-1。如果给出的信号无效或者
试图对一个不允许被捕获或忽略的信号进行捕获或
忽略,错误变量errno将被设置为EINVAL


在参数act指向的sigaction结构中,sa_handler
是一个函数指针,它指向接收到信号sig时将被调用
的信号处理函数。它相当于前面见到的传递给函数
signal的参数func。我们可以将sa_handler字段设
置为特殊值SIG_IGN和SIG_DFL,它们分别表示信
号将被忽略或把对该信号的处理方式恢复为默认动
作。
sa_mask成员指定了一个信号集,在调用
sa_handler所指向的信号处理函数之前,该信号集
将被加入到进程的信号屏蔽字中。这是一组将被阻
塞且不会传递给该进程的信号。设置信号屏蔽字可
以防止前面看到的信号在它的处理函数还未运行结
束时就被接收到的情况。使用sa_mask字段可以消
除这一竞态条件。
但是,由sigaction函数设置的信号处理函数在
默认情况下是不被重置的,如果希望获得类似前面
用第二次signal调用对信号处理进行重置的效果,
就必须在sa_flags成员中包含值SA_RESETHAND。


sigaction替换signal

```C
#include <signal.h>
#include <stdio.h>
#include <unistd.h>

void ouch(int sig)
{
    printf("got signal: %d\n", sig);
    // (void) signal(SIGINT, SIG_DFL);
}

// must use ctrl+\ to end it.

int main()
{
    struct sigaction act;
    act.sa_handler = ouch;
    sigemptyset(&act.sa_mask);
    act.sa_flags = 0;
    sigaction(SIGINT, &act, 0);

    // (void) signal(SIGINT, ouch);

    while (1)
    {
        printf("hi\n");
        sleep(1);
    }
}
```
只要按下Ctrl+C组合键,就可以看到一条消息。因为sigaction函数连续
处理到来的SIGINT信号。要想终止这个程序,我们
只能按下`Ctrl+\`组合键,它在默认情况下产生SIGQUIT信号。

。。我的coredump是在：
/var/lib/systemd/coredump
。。


### 信号集

头文件signal.h定义了类型sigset_t和用来处理
信号集的函数。sigaction和其他函数将用这些信号
集来修改进程在接收到信号时的行为。

```C
#include <signal.h>

int sigaddset(sigset_t *set, int signo);
int sigemptyset(sigset_t *set);
int sigfillset(sigset_t *set);
int sigdelset(sigset_t *set, int signo);
```

sigemptyset将信号集初始化为空。
sigfillset将信号集初始化为包含所有已定义的信号。
sigaddset和sigdelset从信号集中增加或删除给定的信号(signo)。

它们在成功时返回0,失败时返回-1并
设置errno。只有一个错误代码被定义,即当给定
的信号无效时,errno将设置为EINVAL。

函数sigismember判断一个给定的信号是否是
一个信号集的成员。如果是就返回1;如果不是,
它就返回0;如果给定的信号无效,它就返回-1并
设置errno为EINVAL。
```C
#include <signal.h>

int sigismember(sigset_t *set, int signo);
```

进程的信号屏蔽字的设置或检查工作由函数
sigprocmask来完成。信号屏蔽字是指当前被阻塞
的一组信号,它们不能被当前进程接收到。
```C
#include <signal.h>

int sigprocmask(int how, cosnt sigset_t *set, sigset_t *oset);
```
sigprocmask函数可以根据参数how指定的方
法修改进程的信号屏蔽字。新的信号屏蔽字由参数
set(如果它不为空)指定,而原先的信号屏蔽字将保存到信号集oset中
how的取值：
- SIG_BLOCK     将参数set中的信号添加到 信号屏蔽字中
- SIG_SETMASK   把信号屏蔽字 设置位参数set中的信号
- SIG_UNBLOCK   从信号屏蔽字 中删除 set中的信号

如果参数set是空指针,how的值就没有意义
了,此时这个调用的唯一目的就是把当前信号屏蔽
字的值保存到oset中。

如果sigprocmask成功完成,它将返回0;如果
参数how取值无效,它将返回-1并设置errno为
EINVAL。


如果一个信号被进程阻塞,它就不会传递给进
程,但会停留在待处理状态。程序可以通过调用函
数sigpending来查看它阻塞的信号中有哪些正停留
在待处理状态。

```C
#include <signal.h>

int sigpending(sigset_t *set);
```
将被阻塞的信号中停留在
待处理状态的一组信号写到参数set指向的信号集
中。成功时它将返回0,否则返回-1并设置errno以
表明错误的原因。如果程序需要处理信号,同时又
需要控制信号处理函数的调用时间,这个函数就很
有用了。


进程可以通过调用sigsuspend函数挂起自己的执行,直到信号集中的一个信号到达为止。这是我
们前面见到的pause函数更通用的一种表现形式。
```C
#include <signal.h>

int sigsuspend(const sigset_t *sigmask);
```

sigsuspend函数将进程的屏蔽字替换为由参数
sigmask给出的信号集,然后挂起程序的执行。程
序将在信号处理函数执行完毕后继续执行。如果接
收到的信号终止了程序,sigsuspend就不会返回;
如果接收到的信号没有终止程序,sigsuspend就返
回-1并将errno设置为EINTR。


sigaction标志

用在sigaction函数里的sigaction结构中的
sa_flags字段可以包含表11-7中的取值,它们用于
改变信号的行为
- SA_NOCLDSTOP  子进程停止时不产生 SIGCHLD 信号
- SA_RESETHAND  对此信号的处理方式 在信号处理函数的入口处 置为 SIG_DEL
- SA_RESTART    重启可中断的函数而不是给出EINTR错误
- SA_NODEFER    捕获到信号时不将它添加到信号屏蔽字中


当一个信号被捕获时,SA_RESETHAND标志可
以用来自动清除它的信号处理函数,就如同我们在
前面所看到的那样。

程序中使用的许多系统调用都是可中断的。也就是
说,当接收到一个信号时,它们将返回一个错
误并将errno设置为EINTR,表明函数是因为一个信
号而返回的。使用了信号的应用程序需要特别注意
这一行为。如果sigaction调用中的sa_flags字段设
置了SA_RESTART标志,那么在信号处理函数执行
完之后,函数将被重启而不是被信号中断。

一般的做法是,信号处理函数正在执行时,新
接收到的信号将在该处理函数的执行期间被添加到
进程的信号屏蔽字中。这防止了同一信号的不断出
现引起信号处理函数的再次运行。如果信号处理函
数是一个不可重入的函数,在它结束对第一个信号
的处理之前又让另一个信号再次调用它就有可能引
起问题。但如果设置了SA_NODEFER标志,当程
序接收到这个信号时就不会改变信号屏蔽字。

信号处理函数可以在其执行期间被中断并再次
被调用。当返回到第一次调用时,它能否继续正确
操作是很关键的。这不仅仅是递归(调用自身)的
问题,而是可重入(可以安全地进入和再次执行)
的问题。Linux内核中,在同一时间负责处理多个
设备的中断服务例程就需要是可重入的,因为优先
级更高的中断可能会在同一段代码的执行期间“插
入”进来。


表11-8中列出的是可以在信号处理函数中安全调用的函数。X/Open规范保证它们都是可重入的
或者本身不会再生成信号的
。。p414.. 约70个？


表11-9中信号的默认动作都是==异常终止进程==,
进程将以_exit调用方式退出(它类似exit,但在返
回到内核之前不作任何清理工作)。但进程的结束
状态会传递到wait和waitpid函数中去,从而表明进
程是因某个特定的信号而异常终止的
|信号|说明|
|--|--|
|SIGALRM|由alarm函数设置的定时器产生|
|SIGHUP|由一个处于非连接状态的终端发送给控制进程，或者由控制进程在自身结束时发送给每个前台进程|
|SIGINT|终端敲入ctrl+c 或预先设置好的终端字符 时 产生|
|SIGKILL|因为这个信号不能被捕获或忽略，所以一般在shell中用它来强制终止异常进程|
|SIGPIPE|如果在向管道写数据时没有与之对应的读进程，就产生这个信号|
|SIGTERM|作为一个请求被发送，要求进程结束运行。unix在关机时用这个信号要求系统服务停止运行，它是kill命令默认发送的信号|
|SIGUSR1，SIGUSR2|==进程之间可以用这个信号通信==，例如让进程报告状态信息等|


默认情况下,表11-10中的信号也会引起==进程的异常终止==。但可能还会有一些与具体实现相关的
其他动作,比如创建core文件等。
|信号|说明|
|--|--|
|SIGFPE    |  浮点运算异常|
|SIGILL    |  处理器执行了一条非法指令，通常是由 崩溃的程序 或 无效的共享内存模块 引起的|
|SIGQUIT   |  终端敲入ctrl+\ 或预先设置好的退出字符 时产生|
|SIGSEGV   |  段冲突，一般是因为对 内存中的 无效地址 进行读写 导致的。如 数组越界，引用无效指针，函数返回一个非法地址，覆盖局部数据变量 和引起栈崩溃。|


默认情况下,进程接收到列在表11-11中的信
号时将会被==挂起==。

|信号|说明|
|--|--|
|SIGSTOP|停止执行(不能被捕获或忽略)|
|SIGTSTP|终端挂起信号，通常由 ctrl+z 产生|
|SIGTTIN，SIGTTOU|shell 用这2个信号 表明后台作业 因需要从 终端 读取输入或 产生输出 而暂停运行|


SIGCONT信号的作用是重启被暂停的进程,如果进程没有暂停,则忽略该信号。
SIGCHLD信号在默认情况下被忽略



# ch12 POSIX线程 (not PAXOS)

在本章中,我们将介绍以下内容:
- 在进程中创建新线程
- 在一个进程中同步线程之间的数据访问
- 修改线程的属性
- 在同一个进程中,从一个线程中控制另一个线程


## 什么是线程

在一个程序中的多个执行路线就叫做线程

线程是一个进程内部的一个控制序列

弄清楚fork系统调用和创建新线程之间的区别非常重要。
当进程执行fork调用时,将创建出该进程的一份新副本。这个新进程拥有自己的变量和自
己的PID,它的时间调度也是独立的,它的执行(通常)几乎完全独立于父进程。

当在进程中创建一个新线程时,新的执行线程将拥有自己的栈(因
此也有自己的局部变量),但与它的创建者共享全局变量、文件描述符、信号处理函数和当前目录状态。

。。进程： 独立的 变量，pid和 时间调度
。。线程： 独立的栈， 共享 全局变量，文件描述符，信号处理函数 和当前目录状态。

POSIX1003.1c规范的发布改变了这一切,线程不仅被很
好地标准化了,而且现在绝大多数Linux发行版都已支持它。

本章中的大部分代码适用于任何一种线程库,因为这些代码是基于POSIX标准的

New Generation POSIX Thread, 简写为NGPT


## 线程的优缺点

新线程的创建代价要比新进程小得多

优点
- 有时,让程序看起来好像是在同时做两件事情是很有用的。
一个经典的例子是,在编辑文档
的同时对文档中的单词个数进行实时统计。一个线
程负责处理用户的输入并执行文本编辑工作,另一
个(它也可以看到相同的文档内容)则不断刷新单
词计数变量。
另一个例子是一个多线程的数据
库服务器,这是一种明显的单进程服务多用户的情
况。它会在响应一些请求的同时阻塞另外一些请
求,使之等待磁盘操作,从而改善整体上的数据吞
吐量。对于数据库服务器来说,这个明显的多任务
工作如果用多进程的方式来完成将很难做到高效,
因为各个不同的进程必须紧密合作才能满足加锁和
数据一致性方面的要求,而用多线程来完成就比用
多进程要容易得多。

- 一个混杂着输入、计算和输出的应用程
序,可以将这几个部分分离为3个线程来执行,从
而改善程序执行的性能。当输入或输出线程等待连
接时,另外一个线程可以继续执行。因此,如果一
个进程在任一时刻最多只能做一件事情的话,线程
可以让它在等待连接之类的事情的同时做一些其他
有用的事情。
一般而言,线程之间的切换需要
操作系统做的工作要比进程之间的切换少
得多,因此多个线程对资源的需求要远小
于多个进程。



缺点
- 编写多线程程序需要非常仔细的
设计。在多线程程序中,因时序上的细微
偏差或无意造成

- 对多线程程序的调试要比对单线
程程序的调试困难得多,因为线程之间的
交互非常难于控制


## 第一个线程程序

线程有一套完整的与其有关的函数库调用,它们中的绝大多数函数名都以`pthread_`开头。为了使
用这些函数库调用,我们必须==定义宏_REENTRANT==,在程序中包含==头文件pthread.h==,
并且在编译程序时需要用==选项-1pthread来链接线程库==。

在设计最初的UNIX和POSIX库例程时,人们假
设每个进程中只有一个执行线程。一个明显的例子
就是errno,该变量用于获取某个函数调用失败后
的错误信息。在一个多线程程序里,默认情况下,
只有一个errno变量供所有的线程共享。在一个线
程准备获取刚才的错误代码时,该变量很容易被另
一个线程中的函数调用所改变。类似的问题还存在
于fputs之类的函数中,这些函数通常用一个全局
性区域来缓存输出数据

为解决这个问题,我们需要使用被称为可重入
的例程。可重入代码可以被多次调用而仍然正常工
作,这些调用可以来自不同的线程,也可以是某种
形式的嵌套调用。因此,代码中的可重入部分通常
只使用局部变量,这使得每次对该代码的调用都将
获得它自己的唯一的一份数据副本。


通过定义宏_REENTRANT来告诉编译器我们需要可重入功能,
这个宏的定义必须位于程序中的任何#include语句
之前。它将为我们做3件事情,并且做得非常优
雅,以至于我们一般不需要知道它到底做了哪些事。
- 它会对部分函数重新定义它们的可安全重入的版本,这些函数的名字一般
不会发生改变,只是会在函数名后面添加_r字符串。例如,函数名gethostbyname将变为gethostbyname_r。
- stdio.h中原来以宏的形式实现的一些函数将变成可安全重入的函数。
- 在errno.h中定义的变量errno现在将成为一个函数调用,它能够以一种多线程安全的方式来获取真正的errno值。



### 创建一个新线程 pthread_create
```C
#include <pthread.h>

int pthread_create(pthread_t *thread, pthread_attr_t *attr, void *(*start_routine)(void *), void *arg);
```
该函数调用成功时返回值是0，如果失败则返回错误代码。

看起来很复杂，其实用起来很简单。
- 第一个参数是指向pthread_t类型数据的指针。线程被创建时，这个指针指向的变量中将被写入一个标识符，我们用该标识符来引用新线程。
- 下一个参数用于设置线程的属性。我们一般不需要特殊的属性，所以只需设置该参数为NULL。
我们将在本章的后面介绍如何使用这些属性。
- 最后两个参数分别告诉线程将要启动执行的函数和传递给该函数的参数。

`void *(*start_routine)(void *)`
上面一行告诉我们必须要传递一个函数地址，
该函数以一个指向void的指针为参数，返回的也是
一个指向void的指针。
因此，==可以传递一个任一类型的参数并返回一个任一类型的指针==。用fork调用
后，父子进程将在同一位置继续执行下去，只是
fork调用的返回值是不同的；但对新线程来说，我
们必须明确地提供给它一个函数指针，新线程将在这个新位置开始执行。

pthread_create和大多数pthread_系列函数
一样，在失败时并未遵循UNIX函数的惯例返
回-1，这种情况在UNIX函数中属于一少部分。所
以除非你很有把握，在对错误代码进行检查之前
一定要仔细阅读使用手册中的有关内容。


线程通过调用`pthread_exit`函数终止执行，就如
同进程在结束时调用exit函数一样。这个函数的作
用是，终止调用它的线程并返回一个指向某个对象
的指针。注意，绝不能用它来返回一个指向局部变
量的指针，因为线程调用该函数后，这个局部变量
就不再存在了，这将引起严重的程序漏洞。

```C
#include <pthread.h>

void pthread_exit(void *retval);
```

`pthread_join`函数在线程中的作用等价于进程中
用来收集子进程信息的wait函数
```C
// pthread.h

int pthread_join(pthread_t th, void **thread_return);
```
第一个参数指定了将要等待的线程，线程通过
pthread_create返回的标识符来指定。第二个参数
是一个指针，它指向另一个指针，而后者指向线程
的返回值。与pthread_create类似，这个函数在成
功时返回0，失败时返回错误代码。



```C
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
#include <pthread.h>

void *thread_function(void *arg);

char message[] = "hi world";

int main()
{
    int res;
    pthread_t a_thread;
    void *thread_result;

    res = pthread_create(&a_thread, NULL, thread_function, (void *) message);
    // 此时 新线程已经开始执行了。如果创建成功的话。
    if (res != 0)
    {
        perror("thread create failed\n");
        exit(EXIT_FAILURE);
    }
    printf("waiting thread finish\n");
    res = pthread_join(a_thread, &thread_result);
    if (res != 0)
    {
        perror("thread join failed\n");
        exit(EXIT_FAILURE);
    }
    printf("thread join finish, it return : %s\n", (char *) thread_result);
    printf("message now is %s\n", message);
    exit(EXIT_SUCCESS);
}

void *thread_function(void *arg)
{
    printf("thread_function running, arg is %s\n", (char *)arg);
    sleep(1);
    strcpy(message, "bye");
    pthread_exit("thank you.  thread fun");
}
```
新线程修改了数组message，而原先的线程也可以访问
该数组。如果我们调用的是fork而不是pthread_create，就不会有这样的效果。

编译这个程序时，我们首先需要定义宏
_REENTRANT。在少数系统上，可能还需要定义宏
_POSIX_C_SOURCE

必须链接正确的线程库。如果使用
的是一个老的Linux发行版，默认的线程库不是
NPTL，你可能需要升级Linux发行版

`cc -D_REENTRANT -l/usr/include/nptl thread.c -o thread1 -L/usr/lib/nptl -lpthread`

。。我直接 `gcc 01_xx.c` 就可以运行了。 而且我也没有 nptl。。

如果你的系统默认使用的（很有可能）就是
NPTL线程库，那么编译程序时就无需加上-I和-L
选项
`cc -D_REENTRANT thread1.c -o thread1 -lpthread`

。。
    #ifdef _REENTRANT
    printf(" there is _reentrant\n");
    #endif

代码中加了上面的， 直接运行 `gcc 01_xx.c` 是没有这个输出的。
加上 -D_REENTRANT 后是有的。

都可以运行。

不知道是不是现在已经不需要了。

不过查询网上，并没有明确说废弃了。 而且确实 在 /usr/include/features.h 中有 _REENTRANT 的逻辑。 应该还是要的。

但不知道为什么 不加， 也可以编译。  不知道 加不加， 代码有什么区别。

和C++ 的差别好大。 std::thread .. 完全不一样。
。。



## 同时执行

。对于上面的代码的小修改。 增加一个 公共的 int， 主线程 和 新线程 都读取它， 主线程中 判断是不是2，是 就打印，并改为1，不是就sleep 1s  新线程 判断是不是1，是就打印并改为2, 不是就 sleep 1s。 每个线程 尝试一定的次数，


## 同步
控制 ==线程执行和访问代码临界区域== 的方法

- 信号量： 如同看守一段代码的看门人
- 互斥量： 如同保护代码段的一个互斥设备

它们可以互相通过对方来实现

### 用信号量进行同步

有两组接口函数用于信号量。
- 一组取自POSIX的实时扩展，用于线程。
- 另一组被称为系统V信号量，常用于进程的同步

Dijkstra首先提出了信号量的概念

信号量是一个特殊类型的变量，它可以被增
加或减少，但对其的关键访问被保证是==原子操作==，
即使在一个多线程程序中也是如此

最简单的信号量——二进制信号量，它只有0和1两种取值。
种更通用的信号量——计数信号量，它可以有更大的取值范围

信号量函数的名字都以==sem_==开头，而不像大多
数线程函数那样以pthread_开头

信号量通过sem_init函数创建
```C
#include <semaphore.h>

int sem_init(sem_t *sem, int pshared, unsigned int value);
```
初始化由sem指向的信号量对象，设置它的共享选项，并给它一个初始的整数值

pshared参数控制信号量的类型，如果其值为0，就表示这个信号量是==当前进程的局部信号量==，否则，这个信号量就可以在==多个进程之间共享==


控制信号量的值
```C
#include <semaphore.h>

int sem_wait(sem_t *sem);   // 还有 sem_trywait
int sem_post(sem_t *sem);
```
sem_post函数的作用是以==原子==操作的方式给信号量的值==加1==

sem_wait函数以==原子==操作的方式将信号量的值==减1==，
但它会==等待直到信号量有个非零值才会开始减法==操作


清理信号量
```C
#include <semaphore.h>

int sem_destory(sem_t * sem);
```
清理该信号量拥有的所有资源。如
果企图清理的信号量正被一些线程等待，就会收到
一个错误。

这些函数在成功时都返回0。


#### code semaphore
```C

#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
#include <pthread.h>
#include <semaphore.h>


void *thread_function(void *arg);
sem_t bin_sem;

// char message[] = "hi world";

#define WORK_SIZE 1024
char work_area[WORK_SIZE];

int main()
{
    #ifdef _REENTRANT
    printf(" there is _reentrant\n");
    #endif

    int res;
    pthread_t a_thread;
    void *thread_result;

    res = sem_init(&bin_sem, 0, 0);
    if (res != 0)
    {
        perror("semephore init failed\n");
        exit(EXIT_FAILURE);
    }

    res = pthread_create(&a_thread, NULL, thread_function, NULL);
    // 新thread已经开始了。如果res==0。
    if (res != 0)
    {
        perror("thread create failed\n");
        exit(EXIT_FAILURE);
    }
    // sleep(1);
    printf("intput text, 'end' to finish\n");

    while (strncmp("end", work_area, 3) != 0)
    {
        fgets(work_area, WORK_SIZE, stdin);
        sem_post(&bin_sem);
    }

    printf("wait thread end");
    res = pthread_join(a_thread, &thread_result);
    if (res != 0)
    {
        perror("thread join failed\n");
        exit(EXIT_FAILURE);
    }
    printf("thread join finish, it return : %s\n", (char *) thread_result);
    // printf("message now is %s\n", message);
    sem_destroy(&bin_sem);
    exit(EXIT_SUCCESS);
}

void *thread_function(void *arg)
{
    sem_wait(&bin_sem);
    while (strncmp("end", work_area, 3) != 0)
    {
        printf("you input %ld chars\n", strlen(work_area) - 1);
        sem_wait(&bin_sem);
    }
    pthread_exit("end of thread");
    // printf("thread_function running, arg is %s\n", (char *)arg);
    // sleep(2);
    // strcpy(message, "bye");
    // pthread_exit("thank you.  thread fun");
}
```

#### ?sem_wait 是 自旋，还是 等待唤醒?


### 使用互斥量进行同步

允许程序员锁住某个对象，使得每次
只能有一个线程访问它。为了控制对关键代码的访
问，必须在进入这段代码==之前锁住==一个互斥量，然
后在==完成操作之后解锁==它

```C
#include <pthread.h>

int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *mutexattr);
int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);
int pthread_mutex_destory(pthread_mutex_t *mutex);
```

成功时返回0，失败时将返回错误代码，但这些函数并不设置errno，你必须对函数的返回代码进行检查。

属性类型==默认为fast==，但
它有一个小缺点：如果程序试图对一个已经加了锁
的互斥量调用pthread_mutex_lock，程序就会被
阻塞，而又因为拥有互斥量的这个线程正是现在被
阻塞的线程，所以互斥量就永远也不会被解锁了，

#### 。。。fast是非可重入锁？


#### code pthread_mutex

```C
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
#include <pthread.h>
#include <semaphore.h>

void *thread_function(void *arg);
pthread_mutex_t mtx;

#define WORK_SIZE 1024
char work_area[WORK_SIZE];
int time_to_exit = 0;

int main()
{
    int res;
    pthread_t a_thread;
    void *thread_result;
    res = pthread_mutex_init(&mtx, NULL);
    if (res != 0)
    {
        perror("mutex init fail");
        exit(EXIT_FAILURE);
    }
    res = pthread_create(&a_thread, NULL, thread_function, NULL);
    if (res != 0)
    {
        perror("thread create fail");
        exit(EXIT_FAILURE);
    }
    pthread_mutex_lock(&mtx);
    printf("input some text, 'end' to quit\n");
    while (!time_to_exit)
    {
        fgets(work_area, WORK_SIZE, stdin);
        pthread_mutex_unlock(&mtx);
        while (1)
        {
            pthread_mutex_lock(&mtx);
            if (work_area[0] != '\0')
            {
    // 有值的情况下，要等另一根线程消费，所以解锁，等待。
                pthread_mutex_unlock(&mtx);
                sleep(1);
            }
            else
            {
                break;
            }
        }
    }
    pthread_mutex_unlock(&mtx);
    printf("\n wait thread finish\n");
    res = pthread_join(a_thread, &thread_result);

    if (res != 0)
    {
        perror("thread join fail");
        exit(EXIT_FAILURE);
    }
    printf("thread join success\n");
    pthread_mutex_destroy(&mtx);
}

void *thread_function(void *arg)
{
    sleep(1);
    pthread_mutex_lock(&mtx);
    while (strncmp("end", work_area, 3) != 0)
    {
        printf("you input %ld chars\n", strlen(work_area) - 1);
        work_area[0] = '\0';
        pthread_mutex_unlock(&mtx);
        while (work_area[0] == '\0')
        {
    // 如果空，那么没有东西可以消费，那么就等待。
            pthread_mutex_unlock(&mtx);
            sleep(1);
            pthread_mutex_lock(&mtx);
        }
    }
    time_to_exit = 1;
    work_area[0] = '\0';
    pthread_mutex_unlock(&mtx);
    pthread_exit(0);
}
```

。。 这个有sleep， 所以可以 连续输入的。  并不是以 回车为分隔。

#### 使用信号量，不使用mutex进行轮询

这种通过轮询来获得结果的方法通常并不是好的编程方式。在实际的
编程中，我们应该==尽可能用信号量==来避免出现这种情况



## 12.6 线程的属性

我们都在程序退出之前用pthread_join对线程再次进行同步，如果我
们想让线程向创建它的线程返回数据就需要这样做。

但有时也会有这种情况，我们既不需要第二个
线程向主线程返回信息，也不想让主线程等待它的结束。

我们可以创建这一类型的线程，它们被称为脱离线程（detached thread）。
可以通过==修改线程属性==或调用==pthread_detach==的方法来创建它们

用到的最重要的函数是pthread_attr_init，
它的作用是初始化一个线程属性对象
```C
#include <pthread.h>

int pthread_attr_init(pthread_attr_t *attr);
```
成功时返回0，失败时返回错误代码。

回收函数`pthread_attr_destroy`，它的目的是对属性对象进行清理和回收


初始化一个线程属性对象后，我们可以调用许
多其他的函数来设置不同的属性行为。我们把其中
主要的一些函数列在下面
```C
#include <pthread.h>

int pthread_attr_setdetachstate(pthread_attr_t *attr, int detachstate);
// 都有类似的getter方法，都是 setter 的参数，变成 const， 变成 指针
int pthread_attr_getdetachstate(const pthread_attr_t *attr, int *detachstate);
int pthread_attr_setschedpolicy(pthread_attr_t *attr, int policy);
int pthread_attr_setschedparam(pthread_attr_t *attr, struct sched_param *param);
int pthread_attr_setinheritsched(pthread_attr_t *attr, int isherit);
int pthread_attr_setscope(pthread_attr_t *attr, int scope);
int pthread_attr_setstacksize(pthread_attr_t *attr, int scope);
```

- detachedstate：
  这个属性允许我们无需对线程进行重新合并。函数可能用
到的两个标志分别是PTHREAD_CREATE_JOINABLE和
PTHREAD_CREATE_DETACHED。这个属性的默认标志值是
PTHREAD_CREATE_JOINABLE，所以可以允许两个线程重新合并。如果标志设置
为PTHREAD_CREATE_DETACHED，就不能调用pthread_join来获得另一个线程的
退出状态。
- schedpolicy：
  这个属性控制==线程的调度方式==。取值可以是
SCHED_OTHER、SCHED_RP和SCHED_FIFO。这个属性的默认值为
SCHED_OTHER。另外两种调度方式只能用于以超级用户权限运行的进程，因为它
们都具备实时调度的功能，但在行为上略
有区别。SCHED_RP使用循环（roundrobin）调度机制，而SCHED_FIFO使
用“先进先出”策略。
- schedparam：
  这个属性是和schedpolicy属性结合使用的，它可以对
以SCHED_OTHER策略运行的线程的调度进行控制。
- inheritsched：
  这个属性可取两个值：PTHREAD_EXPLICIT_SCHED和
PTHREAD_INHERIT_SCHED。它的默认取值是PTHREAD_EXPLICIT_SCHED，表
示调度由属性明确地设置。如果把它设置为PTHREAD_INHERIT_SCHED，新线程
将沿用其创建者所使用的参数。
- scope：
  这个属性控制一个线程调度的计算方式。由于目前Linux只支持它
的一种取值PTHREAD_SCOPE_SYSTEM
- stacksize：
  这个属性控制线程创建的栈大小，单位为字节。它属于POSIX
规范中的“可选”部分，只有在定义了宏_POSIX_THREAD_ATTR_STACKSIZE的实
现版本中才支持。Linux在实现线程时，默认使用的栈很大，所以这个功能对
Linux来说显得有些多余。


#### code detached thread

```C
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <pthread.h>

void *thread_function(void *arg);

char message[] = "hi world";
int thread_finished = 0;

int main()
{
    int res;
    pthread_t a_thread;

    pthread_attr_t thread_attr;

    res = pthread_attr_init(&thread_attr);

    if (res != 0)
    {
        perror("init attr failed");
        exit(EXIT_FAILURE);
    }

    res = pthread_attr_setdetachstate(&thread_attr, PTHREAD_CREATE_DETACHED);
    if (res != 0)
    {
        perror("set detach state failed");
        exit(EXIT_FAILURE);
    }

    res = pthread_create(&a_thread, &thread_attr, thread_function, (void *) message);
    if (res != 0)
    {
        perror("create thread failed");
        exit(EXIT_FAILURE);
    }

    (void) pthread_attr_destroy(&thread_attr);

    while (!thread_finished) {
        printf("wait thread end\n");
        sleep(1);
    }
    printf("main end, bye\n");
    exit(EXIT_SUCCESS);
}

void *thread_function(void *arg)
{
   printf("thread is running. arg: %s\n", (char *) arg);
   sleep(4);
   printf("thread fcuntion end\n");
   thread_finished = 1;
   pthread_exit(NULL);
}
```


## 12.7 取消线程 pthread_cancel

```C
#include <pthread.h>

int pthread_cancel(pthread_t thread);
```

线程可以用pthread_setcancelstate设置自己的取消状态。
```C
#include <pthread.h>

int pthread_setcancelstate(int state, int *oldstate);
```

第一个参数的取值可以是
- PTHREAD_CANCEL_ENABLE，这个值允许线程接收取消请求；
- PTHREAD_CANCEL_DISABLE，它的作用是忽略取消请求。

oldstate指针用于获取先前的取消状态。如果你对它没有兴趣，只需传递NULL给它

如果取消请求被接受了，线程就可以进入第二个控
制层次，用pthread_setcanceltype设置取消类型。
```C
#include <pthread.h>

int pthread_setcanceltype(int tpye, int *oldtype);
```
type参数可以有两种取值：
- 一个是PTHREAD_CANCEL_ASYNCHRONOUS，它将使得在接收到取消请求后立即采取行动；
- 另一个是PTHREAD_CANCEL_DEFERRED，它将使得在接收到取消请求后，一直等待直到线程执行了下述函数之一后才采取行动。具体是函数pthread_join、pthread_cond_wait、pthread_cond_timedwait、pthread_testcancel、sem_wait或sigwait。

根据POSIX标准，其他可能阻塞的系统调用，
如read、wait等也可以成为取消点。在撰写本书
时，Linux还不支持所有这些系统调用都能成为
取消点。但一些实验证明，某些阻塞调用，如
sleep确实允许取消动作的发生。为安全起见，你
可能会想在估计会被取消的代码中添加一些
pthread_testcancel调用。

oldtype参数可以保存先前的状态，如果不想知
道先前的状态，可以传递NULL给它。

默认情况下，线程在启动时的取消状态为
PTHREAD_CANCEL_ENABLE，取消类型是
PTHREAD_CANCEL_DEFERRED。


#### code: cancel a thread

```C
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <pthread.h>

void *thread_fun(void *);

int main()
{
    int res;
    pthread_t a_thread;
    void *thread_result;

    res = pthread_create(&a_thread, NULL, thread_fun, NULL);
    if (res != 0)
    {
        perror("Thread create fail");
        exit(EXIT_FAILURE);
    }

    sleep(3);
    printf("to cancel thread\n");
    res = pthread_cancel(a_thread);
    if (res != 0)
    {
        perror("cancel fail");
        exit(EXIT_FAILURE);
    }
    printf("waiting thread finish\n");
    res = pthread_join(a_thread, &thread_result);
    if (res != 0)
    {
        perror("thread join fail");
        exit(EXIT_FAILURE);
    }
    exit(EXIT_SUCCESS);
}

void *thread_fun(void *arg)
{
    int i, res;
    res = pthread_setcancelstate(PTHREAD_CANCEL_ENABLE, NULL);
    if (res != 0)
    {
        perror("set cancel state fail");
        exit(EXIT_FAILURE);
    }
    res = pthread_setcanceltype(PTHREAD_CANCEL_DEFERRED, NULL);
    if (res != 0)
    {
        perror("set cancel type fail");
        exit(EXIT_FAILURE);
    }
    printf("thread fun is running\n");
    for (i = 0; i < 10; ++i)
    {
        printf("thread fun is still running %d ..\n", i);
        sleep(1);
    }
    pthread_exit(0);
}
```



## 12.8 多线程



```C
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <pthread.h>

#define NUM_THREADS 6

void *thread_fun(void *arg);

int main()
{
    int res;
    pthread_t a_thread[NUM_THREADS];
    void *thread_result;
    int lots_of_threads;

    for (lots_of_threads = 0; lots_of_threads < NUM_THREADS; ++lots_of_threads)
    {
        res = pthread_create(&(a_thread[lots_of_threads]), NULL, thread_fun, (void *)&lots_of_threads);
        if (res != 0)
        {
            perror("thread create fail");
            exit(EXIT_FAILURE);
        }
        sleep(1);
    }
    printf("waiting thread finish\n");
    for (lots_of_threads = NUM_THREADS - 1; lots_of_threads >= 0; --lots_of_threads)
    {
        res = pthread_join(a_thread[lots_of_threads], &thread_result);
        if (res == 0)
        {
            printf("pick up a thread\n");
        }
        else
        {
            printf("pthread join fail\n");
        }
    }
    printf("all done\n");
    exit(EXIT_SUCCESS);
}

void *thread_fun(void *arg)
{
    int my_num = *(int *)arg;
    int rand_num;
    printf("thread fun is running. arg: %d\n", my_num);
    rand_num = 1 + (9.0 * rand()) / (RAND_MAX + 1.0);
    sleep(rand_num);
    printf("bye from %d\n", my_num);
    pthread_exit(NULL);
}
```

。。伪随机，所以本地 执行时的输出的顺序 和 书上的一模一样。

thread fun is running. arg: 0
thread fun is running. arg: 1
thread fun is running. arg: 2
thread fun is running. arg: 3
thread fun is running. arg: 4
bye from 1
thread fun is running. arg: 5
waiting thread finish
bye from 5
pick up a thread
bye from 0
bye from 2
bye from 3
bye from 4
pick up a thread
pick up a thread
pick up a thread
pick up a thread
pick up a thread
all done
。。。


这个程序有一个小漏洞，如果
将sleep调用从启动线程的循环中删除，它就会变得很明显
。。应该是 传给 thread_fun 的参数， 可能在 thread 获得它的值前， 被 for 的 ++ 修改。

我们可以直接传递这个参数的值
`res = pthread_create(&(a_thread[lots_of_threads]), NULL, thread_fun, (void *) lots_of_threads);`

`int my_num = (int) arg;`



# ch13 进程间通信: 管道

11章中，我们看到了一种在两个进程间发送
消息的非常简单的方法：使用信号。我们创建通知
事件，通过它引起响应，但传送的信息只限于一个信号值。

我们将介绍管道，通过它进程之间可以交换更有用的数据

- 管道的定义
- 进程管道
- 管道调用
- 父进程和子进程
- 命名管道：FIFO
- 客户/服务器架构



## 13.1 什么是管道

当从一个进程连接数据流到另一个进程时，我
们使用术语管道（pipe）。我们通常是把一个进程
的输出通过管道连接到另一个进程的输入

对于shell命令来说，命令的连接是通过管道字符来完成的
`cmd1 | cmd2`

shell负责安排两个命令的标准输入和标准输出。
- cmd1的标准输入来自终端键盘。
- cmd1的标准输出传递给cmd2，作为它的标准输入。
- cmd2的标准输出连接到终端屏幕。



## 13.2 进程管道 (popen, pclose)

可能最简单的在两个程序之间传递数据的方法就是使用popen和pclose函数了
```C
#include <stdio.h>

FILE *popen(const char *command, const char *open_mode);
int pclose(FILE *stream_to_close);
```

popen函数允许一个程序将另一个程序作为新
进程来启动，并可以传递数据给它或者通过它接收
数据。command字符串是要运行的程序名和相应
的参数。open_mode必须是“r”或者“w”。

。。启动一个新进程， 然后从这个进程 读 或写。

如果open_mode是“r”，被调用程序的输出
就可以被调用程序使用，调用程序利用popen函数
返回的FILE*文件流指针，就可以通过常用的stdio
库函数（如fread）来读取被调用程序的输出。

如果open_mode是“w”，调用程序就可以用fwrite
调用向被调用程序发送数据，而被调用程序可以在
自己的标准输入上读取这些数据。被调用的程序通
常不会意识到自己正在从另一个进程读取数据，它
只是在标准输入流上读取数据，然后做出相应的操作。

popen函数在失败时返回一个空指针。

如果想通过管道实现双向通信，最普通的解决方法是
使用两个管道，每个管道负责一个方向的数据流


用popen启动的进程结束时，我们可以用
pclose函数关闭与之关联的文件流。pclose调用只
在popen启动的进程结束后才返回。
如果调用pclose时它仍在运行，pclose调用将==等待该进程的结束==

pclose调用的返回值通常是它所关闭的文件流
所在进程的退出码。如果调用进程在调用pclose之
前执行了一个wait语句，被调用进程的退出状态就
会丢失，因为被调用进程已结束。此时，pclose将
返回-1并设置errno为ECHILD。



#### code popen pclose 捕获进程的输出

```C
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

int main()
{
    FILE *read_fp;
    char buffer[BUFSIZ + 1];

/* Default buffer size.  */
//#define BUFSIZ 8192     // stdio.h

    int chars_read;
    memset(buffer, '\0', sizeof(buffer));

    read_fp = popen("uname -a", "r");
    if (read_fp != NULL)
    {
        chars_read = fread(buffer, sizeof(char), BUFSIZ, read_fp);
        if (chars_read > 0)
        {
            printf("output was: %s\n", buffer);
        }
        pclose(read_fp);
        exit(EXIT_SUCCESS);
    }
    exit(EXIT_FAILURE);
}
```


## 13.3 code 将输出送往popen

```C
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

int main()
{
    FILE *write_fp;
    char buffer[BUFSIZ + 1];

    sprintf(buffer, "there is a word.\n");
    write_fp = popen("od -c", "w");
    if (write_fp != NULL)
    {
        fwrite(buffer, sizeof(char), strlen(buffer), write_fp);
        pclose(write_fp);
        exit(EXIT_SUCCESS);
    }
    exit(EXIT_FAILURE);
}
```

下面的命令可以获得相同的输出
`echo "there is a word." | od -c`

。。有 \n 吗？


### 传递更多数据

我们目前所使用的机制都只是将所有数据通过
一次fread或fwrite调用来发送或接收。有时，我们
可能希望能以块方式发送数据，或者我们根本就不
知道输出数据的长度。为了避免定义一个非常大的
缓冲区，我们可以用多个fread或fwrite调用来将数
据分为几部分处理。

#### code

```C
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

int main()
{
    FILE *read_fp;
    char buffer[BUFSIZ + 1];
    int chars_read;

    memset(buffer, '\0', sizeof(buffer));
    read_fp = popen("ps ax", "r");
    if (read_fp != NULL)
    {
        chars_read = fread(buffer ,sizeof(char), BUFSIZ, read_fp);
        while (chars_read > 0)
        {
            buffer[chars_read - 1] = '\0';
            printf("\n\n\n\nread %d:-\n\n\n\n %s\n", BUFSIZ, buffer);
            chars_read = fread(buffer, sizeof(char), BUFSIZ, read_fp);
        }
        pclose(read_fp);
        exit(EXIT_SUCCESS);
    }
    exit(EXIT_FAILURE);
}
```


### 如何实现popen

请求popen调用运行一个程序时，它首先启动
==shell==，即系统中的sh命令，然后将command字符
串作为一个参数传递给它。这有两个效果，一个
好，一个不太好。

在Linux（以及所有的类UNIX系统）中，所有
的参数扩展都是由shell来完成的。所以，在启动程
序之前先启动shell来分析命令字符串，就可以使各
种shell扩展（如*.c所指的是哪些文件）在程序启动
之前就全部完成。这个功能非常有用，它允许我们
通过popen启动非常复杂的shell命令。而其他一些
创建进程的函数（如execl）调用起来就复杂得多，
因为调用进程必须自己去完成shell扩展。

不太好的影响是，针对每个
popen调用，不仅要启动一个被请求的程序，还要
启动一个shell，即每个popen调用将多启动两个进
程。从节省系统资源的角度来看，popen函数的调
用==成本略高==，而且对目标命令的调用比正常方式要
慢一些。


#### code
演示popen函数的行
为。这个程序对所有popen示例程序的源文件的总
行数进行统计，方法是用cat命令显示文件的内容
并将输出通过管道传递给命令wc-l，由后者统计总
行数。如果是在命令行上完成这一任务，我们可以
使用如下命令
`cat popen*.c | wc -l`

事实上，输入命令wc -l popen*.c更简单而
且更有效率，但我们是为了通过这个例子来演示
popen函数的工作原理。

```C
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

int main()
{
    FILE *read_fp;
    char buffer[BUFSIZ + 1];
    int chars_read;

    memset(buffer, '\0', sizeof(buffer));
    read_fp = popen("cat 0*.c | wc -l", "r");
    if (read_fp != NULL)
    {
        chars_read = fread(buffer, sizeof(char), BUFSIZ, read_fp);
        while (chars_read > 0)
        {
            buffer[chars_read - 1] = '\0';
            printf("\n\n\n\nreading:\n\n\n\n %s\n", buffer);
            chars_read = fread(buffer, sizeof(char), BUFSIZ, read_fp);
        }
        pclose(read_fp);
        exit(EXIT_SUCCESS);
    }
    exit(EXIT_SUCCESS);
}
```

。。我的是 0*.c

shell在启动后将popen*.c扩展为一个文件列表，列表中的文件名都以popen开
头，以.c结尾，shell还处理了管道符（|）并将cat
命令的输出传递给wc命令。我们在一个popen调用
中启动了shell、cat程序和wc程序，并进行了一次
输出重定向。而调用这些命令的程序只看到最终的输出结果。


## 13.4 pipe调用 (不需要shell)

在看过高级的popen函数之后，我们再来看看
==底层的pipe==函数。通过这个函数在两个程序之间传
递数据不需要启动一个shell来解释请求的命令。它
同时还提供了对读写数据的更多控制。

```C
#include <unistd.h>

int pipe(int file_descriptor[2]);
```

pipe函数的参数是一个由两个整数类型的文件
描述符组成的数组的指针。**该函数在数组中填上**两
个新的文件描述符后返回0，如果失败则返回-1并
设置errno来表明失败的原因

。。数组的值 是 pipe函数 填写的。

两个返回的文件描述符以一种特殊的方式连接
起来。写到file_descriptor[1]的所有数据都可以从
file_descriptor[0]读回来。数据基于==先进先出==的原
则（通常简写为FIFO）进行处理

。。对1写 对0读。  对后面的写，对前面的读。

这与栈的处理方式不同，栈采用==后==进先出的原则，通常简写为LIFO。

这里使用的是文件描述符而不是文件流，所以我们必须用底层的read和write调
用来访问数据，而不是用文件流库函数fread和fwrite

```C
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

int main()
{
    int data_processed;
    int file_pipes[2];
    const char some_data[] = "123";
    char buffer[BUFSIZ + 1];

    memset(buffer, '\0', sizeof(buffer));

    if (pipe(file_pipes) == 0)
    {
        data_processed = write(file_pipes[1], some_data, strlen(some_data));
        // 3, 3
        printf("wrote %d bytes.... %ld...\n", data_processed, strlen(some_data));
        data_processed = read(file_pipes[0], buffer, BUFSIZ);
        printf("read %d bytes.. %s\n", data_processed, buffer);
        exit(EXIT_SUCCESS);
    }
    exit(EXIT_SUCCESS);
}
```

如果你尝试用file_descriptor[0]写数据或用
file_descriptor[1]读数据，其后果并未在文档中明确定义
在我的系统上，这样的调用将失败并返回-1，

乍看起来，这个使用管道的例子并无特别之
处，它做的工作也可以用一个简单的文件完成。

管道的真正优势体现在，当你想在两个进程之间传递数据的时候。
我们在第12章讲过，当程序用fork调
用创建新进程时，原先打开的文件描述符仍将保持
打开状态。
如果在原先的进程中创建一个管道，然
后再调用fork创建新进程，我们即可通过管道在两
个进程之间传递数据

```C
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <sys/types.h>

int main()
{
    int data_processed;
    int file_pipes[2];
    const char some_data[] = "123";
    char buffer[BUFSIZ + 1];
    pid_t fork_result;

    memset(buffer, '\0', sizeof(buffer));
    printf("\n\ntest : %ld\n\n\n", sizeof(buffer));

    if (pipe(file_pipes) == 0)
    {
        fork_result = fork();
        if (fork_result == -1)
        {
            fprintf(stderr, "fork fail");
            exit(EXIT_FAILURE);
        }
        if (fork_result == 0)
        {
            // child
            data_processed = read(file_pipes[0], buffer, BUFSIZ);
            printf("read %d byte: %s\n", data_processed, buffer);
            exit(EXIT_SUCCESS);
        }
        else
        {
            // parent
            data_processed = write(file_pipes[1], some_data, strlen(some_data));
            printf("write %d bytes\n", data_processed);
        }
    }
    exit(EXIT_SUCCESS);
}
```

。。会有shell的路径之类的前缀，并且完成后，需要 手动 回车 结束。


## 13.5 父进程和子进程

我们将学习
如何在子进程中运行一个与其父进程完全不同的另
外一个程序，而不是仅仅运行一个相同程序。我们
用exec调用来完成这一工作。这里的一个难点是，
通过exec调用的进程需要知道应该访问哪个文件描
述符。在前面的例子中，因为子进程本身有
file_pipes数据的一份副本，所以这并不成为问题。
但经过exec调用后，情况就不一样了，因为原先的
进程已经被新的子进程替换了。为解决这个问题，
我们可以将文件描述符（它实际上只是一个数字）
作为一个参数传递给用exec启动的程序。

需要使用两个程序。第一个程序是数据生产者，它负责创建管道
和启动子进程，而后者是数据消费者

```C
// producer
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <sys/types.h>

int main()
{
    int data_processed;
    int file_pipes[2];
    const char some_data[] = "asd123cvb";
    char buffer[BUFSIZ + 1];
    pid_t fork_result;

    memset(buffer, '\0', sizeof(buffer));
    
    if (pipe(file_pipes) == 0)
    {
        fork_result = fork();
        if (fork_result == -1)
        {
            fprintf(stderr, "fork fail");
            exit(EXIT_FAILURE);
        }
        if (fork_result == 0)
        {
            sprintf(buffer, "%d", file_pipes[0]);
            (void) execl("07_2", "07_2", buffer, (char *) 0);
            exit(EXIT_FAILURE);
        }
        else
        {
            data_processed = write(file_pipes[1], some_data, strlen(some_data));
            printf("%d - wrote %d bytes\n", getpid(), data_processed);
        }
    }
    exit(EXIT_SUCCESS);
}
```

```C
// consumer, gcc -o 07_2
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char *argv[])
{
    int data_processed;
    char buffer[BUFSIZ + 1];
    int file_descriptor;

    memset(buffer, '\0', sizeof(buffer));
    sscanf(argv[1], "%d", &file_descriptor);
    data_processed = read(file_descriptor, buffer, BUFSIZ);

    printf("%d - read %d bytes: %s\n", getpid(), data_processed, buffer);
    exit(EXIT_SUCCESS);
}
```



### 管道关闭后的读操作

大多数从标准输入读取数据的程序采用的却
是与我们到目前为止见到的例子非常不同的另外一
种做法。通常它们并不知道有多少数据需要读取，
所以往往采用循环的方法，读取数据——处理数据
——读取更多的数据，直到没有数据可读为止。

当没有数据可读时，read调用通常会阻塞，即
它将暂停进程来等待直到有数据到达为止。

如果管道的另一端已被关闭，也就是说，没有进程打开这
个管道并向它写数据了，这时read调用就会阻塞。
但这样的阻塞不是很有用，因此对一个已关闭写数
据的管道做read调用将返回0而不是阻塞

如果跨越fork调用使用管道，就会有两个不同
的文件描述符可以用于向管道写数据，一个在父进
程中，一个在子进程中。

只有把父子进程中的针对管道的写文件描述符都关闭，管道才会被认为是关
闭了，对管道的read调用才会失败。
我们还将深入
讨论这一问题，在学习到O_NONBLOCK标志和
FIFO时，我们将看到一个这样的例子。


### 把管道用作标准输入和标准输出

我们来看一种用管道连接两个进
程的更简洁的方法。我们把其中一个管道文件描述
符设置为一个已知值，一般是标准输入0或标准输
出1。在父进程中做这个设置稍微有点复杂，但它
使得子程序的编写变得非常简单。
这样做的最大好处是我们可以调用标准程序，
即那些不需要以文件描述符为参数的程序

```C
#include <unistd.h>

int dup(int file_descriptor);
int dup2(int file_descriptor_one, int file_descriptor_two);
```

dup调用的目的是打开一个新的文件描述符，这与open调用有点类似。
不同之处是，dup调用创建的新文件描述符与作为它的参数的那个已有文件描
述符指向同一个文件（或管道）。

对于dup函数来说，新的文件描述符总是取最小的可用值。

而对于dup2函数来说，它所创建的新文件描述符或者与
参数file_descriptor_two相同，或者是第一个大于
该参数的可用值

我们可以使用==更通用的fcnt1调用==（command参数设置为F_DUPFD）来达到与调
用==dup和dup2相同的效果==。虽然如此，但==dup调用更易于使用==，因为它是专门用于复制文件描述
符的。而且它的使用非常普遍，你可以发现，在
已有的程序中，它的使用比fcnt1和F_DUPFD更频繁。

dup是如何帮助我们在进程之间传递数据的呢？
诀窍就在于，标准输入的文件描述符总是0，而dup返回的新的文件描述符又总是使用最小可用的数字。
因此，如果我们首==先关闭文件描述符0然后调用dup，那么新的文件描述符就将是数字0==。
因为新的文件描述符是复制一个已有的文件描
述符，所以==标准输入就会改为指向一个我们传递给dup函数的文件描述符所对应的文件或管道==。我们创建了两个文件描述符，它们指向同一个文件或管
道，而且其中之一是标准输入

#### code redirect stdin(0) by dup

```C
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <sys/types.h>

int main()
{
    int data_processed;
    int file_pipes[2];
    const char some_data[] = "123qweasd";
    pid_t fork_result;

    if (pipe(file_pipes) == 0)
    {
        fork_result = fork();
        if (fork_result == (pid_t) - 1)
        {
            fprintf(stderr, "fork fail");
            exit(EXIT_FAILURE);
        }
        if (fork_result == (pid_t) 0)
        {
            // child
            close(0);           // ..
            dup(file_pipes[0]);
            close(file_pipes[0]);
            close(file_pipes[1]);

            execlp("od", "od", "-c", (char *) 0);
            exit(EXIT_FAILURE);
        }
        else
        {
            close(file_pipes[0]);
            data_processed = write(file_pipes[1], some_data, strlen(some_data));
            close(file_pipes[1]);
            printf("%d - wrote %d bytes\n", (int) getpid(), data_processed);;
        }
    }
    exit(EXIT_SUCCESS);
}
```

程序创建一个管道，然后通过fork创建一个子进程。此时，父子进程都有可以
访问管道的文件描述符，一个用于读数据，一个用于写数据，所以总共有4个打开的文件描述符

子进程先用close（0）关闭它的标准输入，然后调用dup（file_pipes[0]）
把与管道的读取端关联的文件描述符复制为文件描述符0，即标准输入。
接下来，子进程关闭原先的用来从管道读取数据的文件描述符file_pipes[0]。
因为子进程不会向管道写数据，所以它把与管道关联的写操作文件描述符file_pipes[1]也关闭了。
现在，它只有一个与管道关联的文件描述符，即文件描述符0，它的标准输入。

子进程就可以用exec来启动任何从标准输入读取数据的程序了

父进程首先关闭管道的读取端file_pipes[0]，因为它不会从管道读取数据。
接着它向管道写入数据。
当所有数据都写完后，父进程关闭管道的写入端并退出。
因为现在已没有打开的文件描述符可以向管道写数据了，
od程序读取写到管道中的3个字节数据后，后续的读操作将返回0字节，表示已到
达文件尾。
当读操作返回0时，od程序就退出运行。这类似于在终端上运行od命令，然后按下
Ctrl+D组合键发送文件尾标志。


## 13.6 命名管道：FIFO

我们还只能在相关的程序之间传递数据，即这些程序是由一个共同的祖先进程启动的。但如果我们想在不相关的进程之间交换数据，这还不是很方便。

我们可以用FIFO文件来完成这项工作，它通常也被称为命名管道（named pipe）

命名管道是一种特殊类型的文件（别忘了Linux中的所有事物
都是文件），它在文件系统中以文件名的形式存在，但它的行为却和我们已经见过的没有名字的管道类似

可以在命令行上创建命名管道，
也可以在程序中创建它。

命令行上用来创建命名管道的程序是mknod
`mknod filename p`

但mknod命令并未出现在X/Open规范的命令
列表中，所以可能并不是所有的类UNIX系统都可以这样做
我们推荐使用的命令行命令是
`mkfifo filename`


```C
#include <sys/types.h>
#include <sys/stat.h>

int mkfifo(const char *filename, mode_t mode);
int mknod(const char *filename, mode_t mode | S_IFIFO, (dev_t) 0);
```
。。mknod 的声明 有点 6 。
。。是不是 不是这样的
。。不是这样的， 不带 | S_IFIFO 的。


```C
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>

int main()
{
    int res = mkfifo("/tmp/my_fifo", 0777);
    if (res == 0)
        printf("fifo created\n");
    exit(EXIT_SUCCESS);
}
```

`ls -lF /tmp/my_fifo`
可以看到上面创建的文件。

。。
$ ls -lF /tmp/my_fifo
prwxrwxr-x 1 aa aa 0  9月 11 10:09 /tmp/my_fifo|
。。

输出结果中的第一个字符为p，表示这是一个管道。
最后的|符号是由ls命令的-F选项添加的，它也表示这是一个管道。

程序用mkfifo函数创建一个特殊的文件。
虽然我们要求的文件模式是0777，但它被用户掩
码（umask）设置（在本例中是022）给改变了，
这与普通文件的创建是一样的，所以文件的最终模
式是755

我们可以像删除一个普通文件那样用`rm命令`删除FIFO文件，
或者也可以在程序中用`unlink系统调用`来删除它


### 访问FIFO文件
这是一个文件，所以先使用 文件的命令来观察它。

`cat < /tmp/my_fifo`

必须用另一个终端来执行下面的命令，因为第一个命令现在被
挂起以等待数据出现在FIFO中
`echo "hi" > /tmp/my_fifo`

你将看到cat命令产生输出。如果不向FIFO发送
任何数据，cat命令将一直挂起，直到你中断它，
常用的中断方式是使用组合键Ctrl+C

。。是的， 执行 echo xxx 后， cat 返回了。

我们可以将第一个命令放在后台执行，这样即可一次执行两个命令
```shell
cat < /tmp/my_fifo &

echo "hi" > /tmp/my_fifo
```

与通过==pipe调用创建管道不同==，==FIFO是以命名文件的形式存在，而不是打开的文件描述符==，
所以在对它进行读写操作之前==必须先打开它==。
FIFO也用open和close函数打开和关闭，这与我
们前面看到的对文件的操作一样，但它多了一些
其他的功能。对FIFO来说，传递给open调用的
是FIFO的路径名，而不是一个正常的文件。


### 使用open打开FIFO文件

打开FIFO的一个主要限制是，程序不能以
O_RDWR模式打开FIFO文件进行读写操作，这样
做的后果并==未明确定义==。
但这个限制是有道理的，
因为我们通常使用FIFO只是==为了单向传递数据==，所
以没有必要使用O_RDWR模式。如果一个管道以
读/写方式打开，进程就会从这个管道读回它自己的输出。

如果确实需要在程序之间==双向传递数据==，最好
使用==一对FIFO或管道，一个方向使用一个==，或者
（但并不常用）采用==先关闭再重新打开FIFO==的方法
来==明确地改变数据流的方向==。

打开FIFO文件和打开普通文件的另一点区别
是，对open_flag（open函数的第二个参数）的
O_NONBLOCK选项的用法。使用这个选项不仅改
变open调用的处理方式，还会改变对这次open调
用返回的文件描述符进行的读写请求的处理方式。
O_RDONLY、O_WRONLY和O_NONBLOCK
标志共有4种合法的组合方式，我们将逐个介绍它们。

`open(const char *path, O_RDONLY);`
在这种情况下，open调用将阻塞，除非有一个进程以写方式打开同一个FIFO，否则它不会返回。这与前面第一个cat命令的例子类似

`open(const char *path, O_RDONLY | O_NONBLOCK);`
即使没有其他进程以写方式打开FIFO，这个open调用也将成功并立刻返回

`open(const char *path, O_WRONLY);`
在这种情况下，open调用将阻塞，直到有一个进程以读方式打开同一个FIFO为止

`open(const char *path, O_WRONLY | O_NONBLOCK);`
这个函数调用总是立刻返回，但如果没有进程
以读方式打开FIFO文件，open调用将返回一个错
误－1并且FIFO也不会被打开。如果确实有一个进
程以读方式打开FIFO文件，那么我们就可以通过它
返回的文件描述符对这个FIFO文件进行写操作。


请注意O_NONBLOCK分别搭配O_RDONLY
和O_WRONLY在效果上的不同，如果没有进程
以读方式打开管道，非阻塞写方式的open调用将
失败，但非阻塞读方式的open调用总是成功。
close调用的行为并不受O_NONBLOCK标志的影响。


下面我们来看，如何通过使用带O_NONBLOCK标志的open调用的行为来同步两个进程

```C
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <fcntl.h>
#include <sys/types.h>
#include <sys/stat.h>

#define FIFO_NAME "/tmp/my_fifo"

int main(int argc, char *argv[])
{
    int res;
    int open_mode = 0;
    int i;

    if (argc < 0)
    {
        fprintf(stderr, "usage: %s <some combinationof O_RDONLY O_WRONLY O_NONBLOCK\n", *argv);
        exit(EXIT_FAILURE);
    }
    for (i = 1; i < argc; ++i)
    {
        // zhege dui de, i shi bianli, suoyi meici kaishi qian, dou xuyao yidong zhizheng
        if (strncmp(*++argv, "O_RDONLY", 8) == 0)   // you dian guai. xia mian meiyou ++
            open_mode |= O_RDONLY;
        if (strncmp(*argv, "O_WRONLY", 8) == 0)
            open_mode |= O_WRONLY;
        if (strncmp(*argv, "O_NONBLOCK", 10) == 0)
            open_mode |= O_NONBLOCK;
        
        printf("\nargv: %s, %d\n", *(argv), open_mode);
    }

    printf("\n\nopen_mode is %d\n\n\n", open_mode);

    if (access(FIFO_NAME, F_OK) == -1)
    {
        res = mkfifo(FIFO_NAME, 0777);
        if (res != 0)
        {
            fprintf(stderr, "create fifo fail, %s.\n", FIFO_NAME);
            exit(EXIT_FAILURE);
        }
    }
    printf("process %d opening fifo\n", getpid());
    res = open(FIFO_NAME, open_mode);
    printf("process %d result %d\n", getpid(), res);
    sleep(5);
    if (res != -1)
        (void) close(res);
    printf("process %d finish\n", getpid());
    exit(EXIT_SUCCESS);
}
```

```shell
./a.out O_RDONLY &

./a.out O_WRONLY
```

。。可以。。pid，由于2个进程是一起新建的，所以值非常近，要注意分辨。
。。第一个进程阻塞，直到第二个进程启动。


这可能是命名管道最常见的用法了。它允许先
启动读进程，并在open调用中等待，当第二个程序
打开FIFO文件时，两个程序继续运行。注意，读进
程和写进程在open调用处取得同步。

当一个Linux进程被阻塞时，它并不消耗CPU资源，所以这种进程的同步方式对CPU来说是非常有效率的。


```shell
./a.out O_RDONLY O_NONBLOCK &

./a.out O_WRONLY
```

这两个例子可能是open模式的最常见的组合形式。你还可以用这个示例程序随意尝试其他组合方式


#### 对FIFO进行读写操作

使用O_NONBLOCK模式会影响到对FIFO的read和write调用。

对一个空的、阻塞的FIFO（即没有用O_NONBLOCK标志打开）的read调用将等待，直到有数据可以读时才继续执行。

与此相反，对一个空的、非阻塞的FIFO的read调用将立刻返回0字节。

对一个完全阻塞FIFO的write调用将等待，直到
数据可以被写入时才继续执行。如果FIFO不能接收
所有写入的数据，它将按==下面的规则==执行。

。。完全阻塞 是什么意思？ 没有使用 O_NONBLOCK， 但是 完全呢？ 难道是 数据满了？ 但是文件的 数据怎么会满？
。。根据 后续的 规则，应该是 数据满了。

- 如果请求写入的数据的长度小于等于PIPE_BUF字节，调用失败，数据不能写入。
- 如果请求写入的数据的长度大于PIPE_BUF字节，将写入部分数据，返回实际写入的字节数，返回值也可能是0。

。。？ 小于 的时候 调用失败。。wtf？


FIFO的长度是需要考虑的一个很重要的因素。
系统对任一时刻在一个FIFO中可以存在的数据长度
是有限制的。它由==#define PIPE_BUF==语句定义，通
常可以在头文件limits.h中找到它。

在Linux和许多其他类UNIX系统中，它的值通常是4096字节，但
在某些系统中它可能会小到512字节。

系统规定：在一个以O_WRONLY方式（即阻塞方式）打开的
FIFO中，如果写入的数据长度小于等于
PIPE_BUF，那么或者写入全部字节，或者一个字节都不写入。


如果几个不同的程序尝试同时向FIFO写数据，能否保证来自
不同程序的数据块不相互交错就非常关键了。也就
是说，每个写操作都必须是“原子化”的。怎样才能做到这一点呢？

如果你能保证所有的==写请求是发往一个阻塞的
FIFO==的，并且==每个写请求的数据长度小于等于
PIPE_BUF字节==，系统就可以==确保数据决不会交错==
在一起。
通常将每次==通过FIFO传递的数据长度限制为PIPE_BUF字节==是个好方法，除非你只使用一个写进程和一个读进程。


#### code: FIFO between 2 independent process

生产者
我们并不关心写入数据的内容，所以我们并未对缓冲区进行初始化
```C
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <fcntl.h>
#include <limits.h>
#include <sys/types.h>
#include <sys/stat.h>

#define FIFO_NAME "/tmp/my_fifo2"
#define BUFFER_SIZE PIPE_BUF
#define TEN_MEG (1024 * 1024 * 10)

int main()
{
    int pipe_fd;
    int res;
    int open_mode = O_WRONLY;
    int bytes_sent = 0;
    char buffer[BUFFER_SIZE + 1];

    if (access(FIFO_NAME, F_OK) == -1)
    {
        res = mkfifo(FIFO_NAME, 0777);
        if (res != 0)
        {
            fprintf(stderr, "create fifo fail %s\n", FIFO_NAME);
            exit(EXIT_FAILURE);
        }
    }
    printf("process %d opening fifo O_WRONLY \n", getpid());
    pipe_fd = open(FIFO_NAME, open_mode);
    printf("process %d result %d\n", getpid(), pipe_fd);

    if (pipe_fd != -1)
    {
        while (bytes_sent < TEN_MEG)
        {
            res = write(pipe_fd, buffer, BUFFER_SIZE);
            if (res == -1)
            {
                fprintf(stderr, "write error ont pipe");
                exit(EXIT_FAILURE);
            }
            bytes_sent += res;
        }
        (void) close(pipe_fd);
    }
    else
    {
        exit(EXIT_FAILURE);
    }
    printf("process %d finish\n", getpid());
    exit(EXIT_SUCCESS);
}
```

消费者
```C
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <fcntl.h>
#include <limits.h>
#include <sys/types.h>
#include <sys/stat.h>

#define FIFO_NAME "/tmp/my_fifo2"
#define BUFFER_SIZE PIPE_BUF

int main()
{
    int pipe_fd;
    int res;
    int open_mode = O_RDONLY;
    char buffer[BUFFER_SIZE + 1];
    int bytes_read = 0;

    memset(buffer, '\0', sizeof(buffer));

    printf("process %d opening fifo O_RDONLY\n", getpid());
    pipe_fd = open(FIFO_NAME, open_mode);
    printf("process %d result %d\n", getpid(), pipe_fd);

    if (pipe_fd != -1)
    {
        do {
            // pipe_fd 是阻塞管道
            res = read(pipe_fd, buffer, BUFFER_SIZE);
            bytes_read += res;
        } while (res > 0);
        (void) close(pipe_fd);
    }
    else
    {
        exit(EXIT_FAILURE);
    }
    printf("process %d finished, read %d bytes\n", getpid(), bytes_read);
    exit(EXIT_SUCCESS);
}
```

Linux会安排好这两个进程之间的调度，使它
们在可以运行的时候运行，在不能运行的时候阻
塞。因此，写进程将在管道满时阻塞，读进程将
在管道空时阻塞。

time命令的输出显示，读进程只运行了不到0.1
秒的时间，却读取了10MB的数据。这说明管道
（至少在现代Linux系统中的实现）在程序之间传
递数据是很有效率的


### 13.6.2 高级主题：使用FIFO的客户/服务器应用程序

通过命名管道来编写一个非常简单的客户/服
务器应用程序。我们想只用一个服务器进程来接受
请求，对它们进行处理，最后把结果数据返回给发
送请求的一方：客户。

我们想允许多个客户进程都可以向服务器发送
数据。为了使问题简单化，我们假设被处理的数据
可以被拆分为一个个数据块，每个的长度都小于
PIPE_BUF字节

因为服务器每次只能处理一个数据块，所以只
使用一个FIFO应该是合乎逻辑的，服务器通过它读
取数据，每个客户向它写数据。只要将FIFO以阻塞
模式打开，服务器和客户就会根据需要自动被阻塞。

将处理后的数据返回给客户稍微有些困难。我
们需要为每个客户安排第二个管道来接收返回的数
据。通过在传递给服务器的==原先数据中加上客户的
进程标识符（PID）==，双方就可以使用它来==为返回
数据的管道生成一个唯一的名字==。


```C
// 12_advance_topic_client.h

#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <fcntl.h>
#include <limits.h>
#include <sys/types.h>
#include <sys/stat.h>

#define SERVER_FIFO_NAME "/tmp/serv_fifo"
#define CLIENT_FIFO_NAME "/tmp/cli_%d_fifo"

#define BUFFER_SIZE 20

struct data_to_pass_st
{
    pid_t client_pid;
    char some_data[BUFFER_SIZE + 1];
};
```

```C
// server.c

#include "12_advance_topic_client.h"
#include <ctype.h>

int main()
{
    int server_fifo_fd, client_fifo_fd;
    struct data_to_pass_st my_data;
    int read_res;
    char client_fifo[256];
    char *tmp_char_ptr;

    mkfifo(SERVER_FIFO_NAME, 0777);
    server_fifo_fd = open(SERVER_FIFO_NAME, O_RDONLY);
    if (server_fifo_fd == -1)
    {
        fprintf(stderr, "server fifo fail\n");
        exit(EXIT_FAILURE);
    }
    
    sleep(10);

    do {
        read_res = read(server_fifo_fd, &my_data, sizeof(my_data));
        if (read_res > 0)
        {
            tmp_char_ptr = my_data.some_data;
            while (*tmp_char_ptr)
            {
                *tmp_char_ptr = toupper(*tmp_char_ptr);
                tmp_char_ptr++;
            }
            sprintf(client_fifo, CLIENT_FIFO_NAME, my_data.client_pid);
            client_fifo_fd = open(client_fifo, O_WRONLY);
            if (client_fifo_fd != -1)
            {
                write(client_fifo_fd, &my_data, sizeof(my_data));
                close(client_fifo_fd);
            }
        }
    } while (read_res > 0);
    close(server_fifo_fd);
    unlink(SERVER_FIFO_NAME);
    exit(EXIT_SUCCESS);
}
```

```C
// client.c

#include "12_advance_topic_client.h"
#include <ctype.h>

int main()
{
    int server_fifo_fd, client_fifo_fd;
    struct data_to_pass_st my_data;
    int times_to_send;
    char client_fifo[256];

    server_fifo_fd = open(SERVER_FIFO_NAME, O_WRONLY);
    if (server_fifo_fd == -1)
    {
        fprintf(stderr, "sorry no server");
        exit(EXIT_FAILURE);
    }

    my_data.client_pid = getpid();
    sprintf(client_fifo, CLIENT_FIFO_NAME, my_data.client_pid);

    if (mkfifo(client_fifo, 0777) == -1)
    {
        fprintf(stderr, "sorry, make %s fail\n", client_fifo);
        exit(EXIT_FAILURE);
    }

    for (times_to_send = 0; times_to_send < 5; ++times_to_send)
    {
        sprintf(my_data.some_data, "hi from %d", my_data.client_pid);
        printf("%d send %s\n", my_data.client_pid, my_data.some_data);
        write(server_fifo_fd, &my_data, sizeof(my_data));
        client_fifo_fd = open(client_fifo, O_RDONLY);
        if (client_fifo_fd != -1)
        {
            if (read(client_fifo_fd, &my_data, sizeof(my_data)) > 0)
            {
                printf("received %s \n", my_data.some_data);
            }
            close(client_fifo_fd);
        }
    }
    close(server_fifo_fd);
    unlink(client_fifo);
    exit(EXIT_SUCCESS);
}
```

```sh
./server &
for i in 1 2 3 4 5
do
    ./client &
done
```

如果这是一个真正的服务器进程，它还
需要继续等待客户的请求，我们就需要对它进行修
改，有两种方法，如下所示。
- 对它自己的服务器管道打开一个文件描述符，这样read调用将总是阻塞而不是返回0。
- 当read调用返回0时，关闭并重新打开服务器管道，使服务器进程阻塞在open调用处以等待客户的到来，就像它最初启动时那样。



## 13.7 CD数据库应用程序 (跳，有空试试)



# ch14 信号量、共享内存和消息队列

一组进程间通信的机制，它们最初由AT&T System V.2版本的UNIX引
入。由于这些机制都出现在同一个版本中并且有着
相似的编程接口，所以它们又常被称为==IPC（Inter-
Process Communication，进程间通信）==机制，或被更常见的称为System V IPC

将介绍以下几方面的内容。
- 信号量：用于管理对资源的访问。
- 共享内存：用于在程序之间高效地共享数据。
- 消息队列：在程序之间传递数据的一种简单方法。


## 14.1 信号量

我们需要确保只有一个进程（或一个执行线程）可以进入这个临界代码并拥有对资源独占式的访问权。

信号量有着复杂的编程接口，但幸运的是，我们可以很轻松地为自己提供一个更简单的接口，它足够应付大多数信号量编程的问题。

我们在本章介绍的信号量函数比在第12章看
到的用于线程的信号量函数要更通用，所以请不
要把这两者混淆。

要想编写通用的代码，以确保程序对某个特定
的资源具有独占式的访问权是非常困难的。虽然有
一个名为==Dekker算法==的解决方法，但这个算法依赖
于“忙等待”或“自旋锁”。
人们并不愿意使用
这种==浪费CPU资源==的处理方法。但==如果硬件支持独
占式访问==（一般是通过特定的CPU指令的形式），
那么情况就变得简单多了。
一个硬件支持的例子就是，用一条指令以==原子方式==访问并增加寄存器的
值，在这个读取/增加/写入操作执行的过程中不会
有其他指令（甚至一个中断）发生。

我们前面见过的一种可能的解决方法是，使用
带O_EXCL标志的open函数来创建锁文件，它提供
了原子化的文件创建方法。它允许一个进程通过获
取一个令牌（即新创建的文件）来取得成功。这个
方法比较适合于处理简单的问题，但对于更复杂的
例子，它就显得比较杂乱且缺乏效率。


荷兰计算机科学家Edsger Dijkstra提出的信号
量概念是在并发编程领域迈出的重要一步

信号量是一个特殊的变量，它只取正整数值，并且程序对其访问都是原子操作


信号量的一个更正式的定义是：它是一个特殊变量，只允许对它进行等待（wait）和发送信号（signal）这两种操作。因为在Linux编程中，“等待”和“发送信号”都已具有特殊的含义，所以我们将用原先定义的符号来表示这两种操作。
- P（信号量变量）：用于等待。
- V（信号量变量）：用于发送信号

这两个字母分别来自于荷兰语单词
passeren（传递，就好像位于进入临界区域之前的
检查点）和vrijgeven（给予或释放，就好像放弃对
临界区域的控制权）


### 信号量的定义

最简单的信号量是只能取值0和1的变量，即==二进制信号量==。这也是信号量最常见的一种形式。可以取多个正整数值的信号量被称为==通用信号量==

PV操作的定义非常简单。假设有一个信号量变量sv，则这两个操作的定义
- P(sv) 如果sv的值大于0，就-1，如果等于0，就挂起
- V(sv) 如果有其他进程因为等待sv而挂起，那么就让它恢复，如果没有进程因等待sv而挂起，就+1

当临界区域可用时，信
号量变量sv的值是true，然后P（sv）操作将它减1
使它变为false以表示临界区域正在被使用；当进程
离开临界区域时，使用V（sv）操作将它加1，使临
界区域再次变为可用

。。但是 V 的恢复被挂起的进程。。这个功能。。 而且 是没有等待的，才+1 ?


### Linux信号量机制

Linux系统中的信号量接口经过了精心设计，它提供了比通常所需更多的机制。所有
的Linux信号量函数都是针对==成组的通用信号量进行操作==，而不是只针对一个二进制信号量

在一个进程需要锁定多个资源的复杂情况中，这种能够对一组信
号量进行操作的能力是一个巨大的优势

```C
#include <sys/sem.h>

int semctl(int sem_id, int sem_num, int command, ...);
int semget(key_t key, int num_sems, int sem_flags);
int semop(int sem_id, struct sembuf *sem_ops, size_t num_sem_ops);
```

头文件sys/sem.h通常依赖于另两个头文件
sys/types.h和sys/ipc.h。一般情况下，它们都
会被sys/sem.h自动包含


参数key的作用很像一个文件名，它代表程序可
能要使用的某个资源，如果多个程序使用相同的
key值，它将负责协调工作

不同的进程可以用不同的信号量标识符来指向同一个信号量。


#### semget函数

创建一个新信号量或取得一个已有信号量的键
`int semget(key_t key, int num_sems, int sem_flags);`

- `key`参数是整数值，不相关的进程可以通过它访问同一个信号量。
程序对所有信号量的访问都是间接的，它先提供一个键，再由系统生成一个相应的信号量标识符。
只有semget函数才直接使用信号量键，所有其他的信号量函数都是使用由semget函数返回的信号量标识符。
有一个特殊的信号量键值IPC_PRIVATE，它的
作用是创建一个只有创建者进程才可以访问的信号
量，但这个键值很少有实际的用途。在创建新的信
号量时，你需要给键提供一个唯一的非零整数。

- `num_sems`参数指定需要的信号量数目。它几乎总是取值为1。

- `sem_flags`参数是一组标志，它与open函数的
标志非常相似。它低端的9个比特是该信号量的权
限，其作用类似于文件的访问权限。此外，它们还
可以和值IPC_CREAT做按位或操作，来创建一个新
信号量。即使在设置了IPC_CREAT标志后给出的键
是一个已有信号量的键，也不会产生错误。如果函
数用不到IPC_CREAT标志，该标志就会被悄悄地忽
略掉。我们可以通过联合使用标志IPC_CREAT和
IPC_EXCL来确保创建出的是一个新的、唯一的信
号量。如果该信号量已存在，它将返回一个错误。

semget函数在成功时返回一个正数（非零）
值，它就是其他信号量函数将用到的信号量标识
符。如果失败，则返回－1。


#### semop函数
用于改变信号量的值

`int semop(int sem_id, struct sembuf *sem_ops, size_t num_sem_ops);`

- sem_id是由semget返回的信号量标识符。
- sem_ops是指向一个结构数组的指针，每个数组元素至少包含以下几个成员
    ```C
    struct sembuf {
        short sem_num;
        short sem_op;
        short sem_flg;
    }
    ```
    - sem_num是信号量编号，除非你需要使用一组信号量，否则它的取值一般为0。
    - sem_op成员的值是信号量在一次操作中需要改变
    的数值（你可以用一个非1的数值来改变信号量的
    值）。通常只会用到两个值，一个是－1，也就是P
    操作，它等待信号量变为可用；一个是+1，也就是
    V操作，它发送信号表示信号量现在已可用。
    - sem_flg通常被设置为
    SEM_UNDO。它将使得操作系统跟踪当前进程对
    这个信号量的修改情况，如果这个进程在没有释放
    该信号量的情况下终止，操作系统将自动释放该进
    程持有的信号量。除非你对信号量的行为有特殊的
    要求，否则应该养成设置sem_flg为SEM_UNDO
    的好习惯。如果决定使用一个非SEM_UNDO的
    值，那就一定要注意保持设置的一致性，否则你很
    可能会搞不清楚内核是否会在进程退出时清理信号量。


semop调用的一切动作都是一次性完成的


#### semctl函数
直接控制信号量信息

`int semctl(int sem_id, int sem_num, int command, ...);`

- sem_id是由semget返回的信号量标识符。
- sem_num参数是信号量编号，当需要用到成组的信号量时，就要用到这个参数，它一般取值为0，表示这是第一个也是唯一的一个信号量。
- command参数是将要采取的动作。
- 如果还有第四个参数，它将会是一个union semun结构，根据X/OPEN规范的定义，它至少包含以下几个成员
    ```C
    union semun {
        int val;
        struct semid_ds *buf;
        unsigned short *array;
    }
    ```

大多数Linux版本会在某个头文件（一般是sem.h）中给出该结构的定义。如
果你发现确实需要自己来定义该结构，请查阅semctl的手册页

semctl函数中的command参数可以设置许多不
同的值，但只有下面介绍的两个值最常用。semctl
函数的完整细节请查阅它的手册页。
- SETVAL
  用来把信号量初始化为一个已知的值。这个值通过union semun中的val成员设置。其作用是在信号量第一次使用之前对它进行设置。
- IPC_RMID
  用于删除一个已经无需继续使用的信号量标识符。

semctl函数将根据command参数的不同而返回
不同的值。对于SETVAL和IPC_RMID，成功时返回
0，失败时返回－1。


### 14.1.4 使用信号量

大部分需要使用信号量来解决的问题只需使用一个最简单的二进制信号量即可

我们通过一个可选的参数来指定程序是负责创建信号量还是负责删除信号量

我们用两个不同字符的输出来表示进入和离开
临界区域。如果程序启动时带有一个参数，它将在
进入和退出临界区域时打印字符x；而程序的其他
运行实例将在进入和退出临界区域时打印字符o

因为在任一给定时刻，只能有一个进程可以进入临
界区域，所以字符x和o应该是成对出现的。


```C
// 这个是上面提到的。要自己写的。
union semun
{
    int val;
    struct semid_ds *buf;
    unsigned short *array;
};
```


```C
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>

#include <sys/sem.h>
#include "semun.h"

static int set_semvalue(void);
static void del_semvalue(void);
static int semaphore_p(void);
static int semaphore_v(void);

static int sem_id;

int main(int argc, char *argv)
{
    int i;
    int pause_time;
    char op_char = 'O';

    srand((unsigned int) getpid());
    sem_id = semget((key_t) 1234, 1, 0666 | IPC_CREAT);

    if (argc > 1)
    {
        if (!set_semvalue()) {
            fprintf(stderr, "failed to init semaphore\n");
            exit(EXIT_FAILURE);
        }
        op_char = 'x';
        sleep(2);
    }
    for (i = 0; i < 10; ++i)
    {
        // 调用semaphore_p函数，它在程序将进入临界区域时设置信号量以等待进入
        if (!semaphore_p())
            exit(EXIT_FAILURE);
        printf("%c", op_char);
        fflush(stdout);

        pause_time = rand() % 3;
        sleep(pause_time);
        printf("%c", op_char);
        fflush(stdout);

        // 临界区域之后，调用semaphore_v来将信号量设置为可用，然后等待一段随机的时间，再进入下一次循环。
        if (!semaphore_v())
            exit(EXIT_FAILURE);
        
        pause_time = rand() % 2;
        sleep(pause_time);
    }

    printf("\n%d - finished\n", getpid());

    if (argc > 1)
    {
        sleep(10);
        del_semvalue();
    }
    exit(EXIT_SUCCESS);
}

// 函数set_semvalue通过将semctl调用的command参数设置为SETVAL来初始化信号量。在使用信号量之前必须要这样做
static int set_semvalue(void)
{
    union semun sem_union;

    sem_union.val = 1;
    if (semctl(sem_id, 0, SETVAL, sem_union) == -1)
        return 0;
    return (1);
}

// 函数del_semvalue的形式与上面的函数几
// 乎一样，只不过它通过将semctl调用的command
// 设置为IPC_RMID来删除信号量ID
static void del_semvalue(void)
{
    union semun sem_un;
    if (semctl(sem_id, 0, IPC_RMID, sem_un) == -1)
        fprintf(stderr, "fail to delete semaphore\n");
}

// semaphore_p对信号量做减1操作（等待）
static int semaphore_p(void)
{
    struct sembuf sem_b;

    sem_b.sem_num = 0;
    sem_b.sem_op = -1;      // P()
    sem_b.sem_flg = SEM_UNDO;

    if (semop(sem_id, &sem_b, 1) == -1)
    {
        fprintf(stderr, "semaphore_p fail\n");
        return (0);
    }
    return (1);
}

// semaphore_v和semaphore_p类似，不
// 同的是它将sembuf结构中的sem_op设置为1。这
// 是一个“释放”操作，它使信号量变为可用
static int semaphore_v(void)
{
    struct sembuf sem_b;

    sem_b.sem_num = 0;
    sem_b.sem_op = 1;       // V()
    sem_b.sem_flg = SEM_UNDO;

    if (semop(sem_id, &sem_b, 1) == -1)
    {
        fprintf(stderr, "semaphore_v fail\n");
        return (0);
    }
    return (1);
}
```

```text
./a.out 1 &

./a.out
```

字符o和x是成对出现的，这表明对临界区域的处理是正确的

。。OOOOxxOOxxOOxxOOxxOOxxOOxxxxOOxxOOxxOO
。。我不知道这算不算成对。看书上，也有 4个o的

IPC_CREAT标志的作用是：如果信号量不存在，就创建它。



## 14.2 共享内存

共享内存是3个IPC机制中的第二个。它允许两
个不相关的进程访问同一个逻辑内存。共享内存是
在两个正在运行的进程之间传递数据的一种==非常有效的==方式

虽然X/Open标准并没有对它做出要
求，但大多数共享内存的具体实现，都把由不同进
程之间共享的内存安排为同一段物理内存。

。。OS把 不同程序的内存，映射到 同一块 硬件内存上。
。。不同程序的各自的逻辑地址，映射到同一块物理地址

共享内存是由IPC为进程创建的一个特殊的地址范围，
它将出现在该进程的地址空间中。其他进程
可以将同一段共享内存连接到它们自己的地址空间中。
所有进程都可以访问共享内存中的地址，就好像它们是由malloc分配的一样。
如果某个进程向共享内存写入了数据，所做的==改动将立刻==被可以访问
同一段共享内存的任何其他进程看到。

共享内存为在多个进程之间共享和传递数据提
供了一种有效的方式。由于它==并未提供同步机制==，
所以我们通常需要用其他的机制来同步对共享内存
的访问。
我们一般是==用共享内存来提供对大块内存区域的有效访问，同时通过传递小消息来同步对该内存的访问==。

在第一个进程结束对共享内存的写操作之前，
并无自动的机制可以阻止第二个进程开始对它进行
读取。==对共享内存访问的同步控制必须由程序员来负责==


```C
#include <sys/shm.h>

void *shmat(int shm_id, const void *shm_addr, int shmflg);
int shmctl(int shm_id, int cmd, struct shmid_ds *buf);
int shmdt(const void *shm_addr);
int shmget(key_t key, size_t size, int shmflg);
```

头文件sys/types.h和sys/ipc.h通常被 shm.h自动包含进程序。


### shmget函数 创建共享内存

`int shmget(key_t key, size_t size, int shmflg);`

- 参数key，它有效地为共享内存段命名，shmget函数返回一个
共享内存标识符，该标识符将用于后续的共享内存函数。
有一个特殊的键值IPC_PRIVATE，它用于创
建一个只属于创建进程的共享内存。通常你不会用
到这个值，而且你可能会发现在一些Linux系统
中，私有的共享内存其实并不是真正的私有
- 参数size以字节为单位指定需要共享的内存容量。
- 参数shmflg包含9个比特的权限标志，它
们的作用与创建文件时使用的mode标志一样。由
IPC_CREAT定义的一个特殊比特必须和权限标志按
位或才能创建一个新的共享内存段。设置
IPC_CREAT标志的同时，==给shmget函数传递一个
已有共享内存段的键并不是一个错误，如果无需用
到IPC_CREAT标志，该标志就会被悄悄地忽略掉。==

权限标志对共享内存非常有用，因为它们允许
一个进程创建的共享内存可以被共享内存的创建者
所拥有的进程写入，同时其他用户创建的进程只能
读取该共享内存

如果共享内存创建成功，shmget返回一个非负整数，即共享内存标识符；如果失败，就返回-1。


### shmat 链接内存

第一次创建共享内存段时，它不能被任何进程
访问。要想启用对该共享内存的访问，必须将其连
接到一个进程的地址空间中

`void *shmat(int shm_id, const void *shm_addr, int shmflg);`

- shm_id是由shmget返回的共享内存标识符
- shm_addr指定的是共享内存连接到
当前进程中的地址位置。它通常是一个空指针，表
示让系统来选择共享内存出现的地址。
- shmflg是一组位标志。它的两个可
能取值是SHM_RND（这个标志与shm_addr联合
使用，用来控制共享内存连接的地址）和
SHM_RDONLY（它使得连接的内存只读）。我们
很少需要控制共享内存连接的地址，通常都是让系
统来选择一个地址，否则就会使应用程序对硬件的
依赖性过高。

如果shmat调用成功，它返回一个指向共享内
存第一个字节的指针；如果失败，它就返回-1。

共享内存的读写权限由它的属主（共享内存的
创建者）、它的访问权限和当前进程的属主决定。
共享内存的访问权限类似于文件的访问权限。

一个例外是，当shmflg & SHM_RDONLY为true时的情况。此时即使该共享
内存的访问权限允许写操作，它都不能被写入。


### shmdt 分离共享内存

shmdt函数的作用是将共享内存从当前进程中分离。
它的参数是shmat返回的地址指针。
成功时它返回0，失败时返回-1


### shmctl

`int shmctl(int shm_id, int command, struct shmid_ds *buf);`

shmid_ds 至少包含以下成员
```C
struct shmid_ds {
    uid_t shm_perm.uid;
    uid_t shm_perm.gid;
    mode_t shm_perm.mode;
}
```

- 参数shm_id是shmget返回的共享内存标识符。
- 参数command是要采取的动作，它可以取3个值
  - IPC_STAT， 把 shmid_ds 中的数据设置为 共享内存的当前关联值
  - IPC_SET， 如果进程有足够的权限，就把共享内存的当前关联值设置为 shmid_ds中的值
  - IPC_RMID， 删除共享内存段
- 参数buf是一个指针，它指向包含共享内存模式和访问权限的结构

成功时返回0，失败时返回-1

X/Open规范没有定义当你试图删除一个正处于连接状态的共享内
存段时将会发生的情况。通常这个已经被删除的处
于连接状态的共享内存段还能继续使用，直到它从
最后一个进程中分离为止。但因为这个行为并未在
规范中定义，所以最好不要依赖它。


#### code: shared memory

```C
// shm_com.h

#define TEXT_SZ 2048

struct shared_use_st {
    int written_by_you;
    char some_text[TEXT_SZ];
};
```

```C
// consumer

#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

#include <sys/shm.h>

#include "shm_com.h"

int main()
{
    int running = 1;
    void *shared_memory = (void *) 0;
    struct shared_use_st *shared_stuff;
    int shmid;

    srand((unsigned int) getpid());

    // 创建共享内存段
    shmid = shmget((key_t) 1234, sizeof(struct shared_use_st), 0666 | IPC_CREAT);

    if (shmid == -1)
    {
        fprintf(stderr, "shmget fail\n");
        exit(EXIT_FAILURE);
    }

    // 让程序可以访问这个共享内存
    shared_memory = shmat(shmid, (void *) 0, 0);
    if (shared_memory == (void *) -1)
    {
        fprintf(stderr, "shmat fail\n");
        exit(EXIT_FAILURE);
    }

    // printf("memory attached at %X\n", (int) (shared_memory));
    printf("memory attached at %p\n", shared_memory);

// 循环将一直执行到在written_by_you中找到
// end字符串为止。sleep调用强迫消费者程序在临界
// 区域多待一会儿，让生产者程序等待
    shared_stuff = (struct shared_use_st *) shared_memory;
    shared_stuff->written_by_you = 0;
    while (running)
    {
        if (shared_stuff->written_by_you)
        {
            printf("you wrote: %s", shared_stuff->some_text);
            sleep(rand() % 4);
            shared_stuff->written_by_you = 0;
            if (strncmp(shared_stuff->some_text, "end", 3) == 0)
            {
                running = 0;
            }
        }
    }

    // 共享内存被分离
    if (shmdt(shared_memory) == -1)
    {
        fprintf(stderr, "shmdt failed\n");
        exit(EXIT_FAILURE);
    }

    // 共享内存被删
    if (shmctl(shmid, IPC_RMID, 0) == -1)
    {
        perror("shmctl IPC_RMID fail");
        exit(EXIT_FAILURE);
    }
    exit(EXIT_SUCCESS);
}
```


```C
// producer

#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

#include <sys/shm.h>

#include "shm_com.h"

int main()
{
    int running = 1;
    void *shared_memory = (void *) 0;
    struct shared_use_st *shared_stuff;
    char buffer[BUFSIZ];
    int shmid;

    shmid = shmget((key_t) 1234, sizeof(struct shared_use_st), 0666 | IPC_CREAT);

    if (shmid == -1)
    {
        fprintf(stderr, "shmget fail\n");
        exit(EXIT_FAILURE);
    }
    shared_memory = shmat(shmid, (void *) 0, 0);
    if (shared_memory == (void *) -1)
    {
        fprintf(stderr, "shmat fail");
        exit(EXIT_FAILURE);
    }
    printf("memory attach at %p", shared_memory);

    shared_stuff = (struct shared_use_st *) shared_memory;
    while (running)
    {
        while (shared_stuff->written_by_you == 1)
        {
            sleep(1);
            printf("waiting for client \n");
        }
        printf("enter some text: ");
        fgets(buffer, BUFSIZ, stdin);

        strncpy(shared_stuff->some_text, buffer, TEXT_SZ);
        shared_stuff->written_by_you = 1;

        if (strncmp(buffer, "end", 3) == 0)
        {
            running = 0;
        }
    }
    if (shmdt(shared_memory) == -1)
    {
        fprintf(stderr, "shmdt fail");
        exit(EXIT_FAILURE);
    }
    exit(EXIT_SUCCESS);
}
```

。。
./consumer &

./producer
。。


producer使用==相同的键1234来取得并连接同一个共享内存段==


我们只能提供自己的、非常简陋的同步
标志written_by_you，它包括一个非常缺乏效率的
忙等待（不停地循环）。
这可以使得我们的示例比较简单，但在实际编程中，我们应该==使用信号量或
通过传递消息（使用管道或IPC消息，后者我们在
下一节就会谈到）、生成信号（在第11章介绍的）
的方法==来提供应用==程序读、写==部分之间的一种==更有
效率的同步机制==。



## 14.3 消息队列

消息队列与命名管道有许多相似之处，但少了在打开和关闭管道方面的复杂性。

但使用消息队列并==未解决==我们在使用命名管道时遇到的一些问题，比如管道满时的阻塞问题。

消息队列提供了一种在==两个不相关的进程之间
传递数据的相当简单且有效的方法==。

与命名管道相比，消息队列的==优势在于==，它独立于发送和接收进
程而存在，这消除了在同步命名管道的打开和关闭时可能产生的一些困难。

消息队列提供了一种从一个进程向另一个进程
发送一个数据块的方法。而且，每个数据块都被认
为含有一个类型，==接收进程可以独立地接收含有不
同类型值的数据块==。

我们可以通过发送消息来几乎完全避免命名管道的同步和阻塞问题。
我们可以用一些方法来提前查看紧急消息

与管道一样，每个数据块都有一个最大长度的限制，系统中所有队列所包含的全部数据块的总长度也有一个上限

Linux系统有两个宏定义==MSGMAX和MSGMNB==，它们以字节为单位分别定义了一条消息的最大长度和一个队列的最大长度

```C
#include <sys/msg.h>

int msgget(key_t key, int msgflg);
int msgctl(int msqid, int cmd, struct msqid_ds *buf);
int msgrcv(int msqid, void *msg_ptr, size_t msg_sz, long int megtype, int msgflg);
int msgsnd(int msqid, const void *msg_ptr, size_t msg_sz, int msgflg);
```

头文件sys/types.h和sys/ipc.h通常被 msg.h 自动包含进程序。


### msgget (。。getOrCreate)

创建和访问一个消息队列

`int msgget(key_t key, int msgflg);`

与其他IPC机制一样，程序必须提供一个键值来命名某个特定的消息队列。
特殊键值IPC_PRIVATE用于创建私有队列，从理论上来说，它应该只能被
当前进程访问，但同信号量和共享内存的情况一
样，消息队列在某些Linux系统中事实上并非私
有。由于私有队列没有什么用处，所以这并不是一
个很严重的问题。

第二个参数msgflg由9个权限标志组成。由IPC_CREAT定义的一个特
殊位必须和权限标志按位或才能创建一个新的消息
队列。在设置IPC_CREAT标志时，如果给出的是一
个==已有消息队列的键也不会产生错误==。==如果消息队
列已有，则IPC_CREAT标志就被悄悄地忽略掉==。

成功时msgget函数返回一个正整数，即队列标识符，失败时返回-1。


### msgsnd

把消息添加到消息队列中

`int msgsnd(int msqid, const void *msg_ptr, size_t msg_sz, int msgflg);`

消息的结构受到两方面的约束。
- 首先，它的长度必须小于系统规定的上限；
- 其次，它必须以一个长整型成员变量开始，接收函数将用这个成员变量
来确定消息的类型。当使用消息时，最好把消息结构定义为下面这样：
    ```C
    struct my_message {
        long int message_type;
        // the data you wish to transfer
    }
    ```

在消息的接收中要用到message_type，所
以你不能忽略它。你必须在声明自己的数据结构时
包含它，并且最好将它初始化为一个已知值。

- 第一个参数msqid是由msgget函数返回的消息队列标识符。
- 第二个参数msg_ptr是一个指向准备发送消息的
指针，消息必须像刚才说的那样以一个==长整型成员变量开始==。
- 第三个参数msg_sz是msg_ptr指向的消息的长
度。这个长度==不能包括长整型消息类型成员变量的长度==。
- 第四个参数msgflg控制在当前消息队列满或队
列消息到达系统范围的限制时将要发生的事情。
  - 如果msgflg中设置了IPC_NOWAIT标志，函数将立刻返回，不发送消息并且返回值为-1。
  - 如果msgflg中的IPC_NOWAIT标志被清除，则发送进程将挂起以等待队列中腾出可用空间。

成功时这个函数返回0，失败时返回-1。
如果调用成功，消息数据的一份副本将被放到消息队列中。


### msgrcv

从一个消息队列中获取消息

`int msgrcv(int msqid, void *msg_ptr, size_t msg_sz, long int msgtype, int msgflg);`

- 第一个参数msqid是由msgget函数返回的消息队列标识符。
- 第二个参数msg_ptr是一个指向准备接收消息的指针，消息必须像前面msgsnd函数中介绍的那样==以一个长整型成员变量开始==。
- 第三个参数msg_sz是msg_ptr指向的消息的长
度，它==不包括长整型消息类型成员变量的长度==。
- 第四个参数msgtype是一个长整数，它可以实
现一种简单形式的接收优先级。
  - 如果msgtype的值为0，就获取队列中的第一个可用消息。
  - 如果它的值大于零，将获取具有相同消息类型的第一个消息。
  - 如果它的值小于零，将获取消息类型等于或小于msgtype的绝对值的第一个消息。
  实际应用时很简单。
  - 如果只想按照消息发送的顺序来接收它们，就把msgtype设置为0。
  - 如果只想获取某一特定类型的消息，就把msgtype设置为相应的类型值。
  - 如果想接收类型等于或小于n的消息，就把msgtype设置为-n。
- 第五个参数msgflg用于控制当队列中没有相应类型的消息可以接收时将发生的事情。
  - 如果msgflg中的IPC_NOWAIT标志被设置，函数将会立刻返回，返回值是-1。
  - 如果msgflg中的IPC_NOWAIT标志被清除，进程将会挂起以等待一条相应类型的消息到达。

成功时msgrcv函数返回放到接收缓存区中的字
节数，消息被复制到由msg_ptr指向的用户分配的
缓存区中，然后删除消息队列中的对应消息。失败
时返回-1。


### msgctl

`int msgctl(int msqid, int command, struct msqid_ds *buf);`

msqid_ds 结构至少包含以下成员
```C
struct msqid_ds {
    uid_t msg_perm.uid;
    uid_t msg_perm.gid;
    mode_t msg_perm.mode;
}
```

- 第一个参数msqid是由msgget返回的消息队列标识符。
- 第二个参数command是将要采取的动作。它可以取3个值
  - IPC_STAT，把msqid_ds结构中的数据设置为消息队列的当前关联值
  - IPC_SET，如果进程有足够权限，就把消息队列的当前关联值设置为msqid_ds结构中给出的值
  - IPC_RMID，删除消息队列

成功时它返回0，失败时返回-1。
如果删除消息队列时，某个进程正在msgsnd或msgrcv函数中等待，这两个函数将失败。


#### code: mq

```C
// producer

#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <unistd.h>

#include <sys/msg.h>

#define MAX_TEXT 512

struct my_msg_st
{
    long int my_msg_type;
    char some_text[MAX_TEXT];
};

int main()
{
    int running = 1;
    struct my_msg_st some_data;
    int msgid;
    char buffer[BUFSIZ];

    msgid = msgget((key_t) 1234, 0666 | IPC_CREAT);
    if (msgid == -1)
    {
        fprintf(stderr, "megget failed, errno: %d\n", errno);
        exit(EXIT_FAILURE);
    }

    while (running)
    {
        printf("enter some text: ");
        fgets(buffer, BUFSIZ, stdin);
        some_data.my_msg_type = 1;
        strcpy(some_data.some_text, buffer);

        if (msgsnd(msgid, (void *) &some_data, MAX_TEXT, 0) == -1)
        {
            fprintf(stderr, "msgsnd fail\n");
            exit(EXIT_FAILURE);
        }
        if (strncmp(buffer, "end", 3) == 0)
        {
            running = 0;
        }
    }
    exit(EXIT_SUCCESS);
}
```


```C
// consumer

#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <unistd.h>

#include <sys/msg.h>

struct my_msg_st
{
    long int my_msg_type;
    char some_text[BUFSIZ];
};

int main()
{
    int running = 1;
    int msgid;
    struct my_msg_st some_data;
    long int msg_to_receive = 0;

    // 建立消息队列
    msgid = msgget((key_t) 1234, 0666 | IPC_CREAT);
    
    if (msgid == -1)
    {
        fprintf(stderr, "msgget failed, errno: %d\n", errno);
        exit(EXIT_FAILURE);
    }

    // 从队列中获取消息
    while (running)
    {
        if (msgrcv(msgid, (void *) &some_data, BUFSIZ, msg_to_receive, 0) == -1)
        {
            fprintf(stderr, "msgrcv failed, errno: %d\n", errno);
            exit(EXIT_FAILURE);
        }
        printf("you write %s", some_data.some_text);
        if (strncmp(some_data.some_text, "end", 3) == 0)
        {
            running = 0;
        }
    }

    // 删除消息队列
    if (msgctl(msgid, IPC_RMID, 0) == -1)
    {
        fprintf(stderr, "msgctl IPC_RMID failed\n");
        exit(EXIT_FAILURE);
    }

    exit(EXIT_SUCCESS);
}
```

。。
./producer
    。。输入一些
./consumer
    。。可以看到之前输入的东西
。。



## 14.4 CD数据库应用程序 (跳)



## 14.5 IPC状态命令 (ipcs, ipcrm)

虽然X/Open规范并没有定义它们，但大多数
Linux系统都提供了一组命令，用于从命令行上访
问IPC信息以及清理游离的IPC机制。它们是==ipcs和
ipcrm==命令，这两个命令对于开发程序非常有用。

IPC机制一个让人烦恼的问题是：编写错误的程序或因为某些原因而执行失败的程序将把它的IPC资源（如消息队列中的数据）==遗留在系统中==，并且这些资源在程序结束后很长时间仍然在系统中游荡。
这将导致对程序的新调用执行失败，因为程序期望以一个干净的系统来启动，但事实上却发现一些遗留的资源。

状态命令（ipcs）和删除命令（ipcrm）提供了一种检查和清理IPC机制的方法。

### 显式信号量状态 (ipcs -s, ipcrm -s)

查询信号量
`ipcs -s`

删除信号量
`ipcrm -s <semid>`


### 显示共享内存状态 (ipcs -m, ipcrm -m)

查询
`ipcs -m`

删除共享内存
`ipcrm -m <shmid>`


### 显示消息队列状态 (ipcs -q, ipcrm -q)

查询
`ipcs -q`

删除队列
`ipcrm -q <semid>`


# ch15 套接字

在本章中，我们将介绍进程间通信的另一种方
法，与我们在第13、14章讨论的方法相比，它有
着明显的不同。
到目前为止，我们讨论的所有机制
都依靠==一台计算机系统==的共享资源实现。这里的资
源可以是文件系统空间、共享的物理内存或消息队
列，但只有运行在==同一台机器上的进程==才能使用它们。

Linux系统支持套接字接口。你可以通过与使用管道类似的
方法来使用套接字，但套接字还包括了计算机网络中的通信。

一台机器上的进程可以使用套接字和另外一台机器上的进程通信
同一台机器上的进程之间也可以使用套接字进行通信。

将主要介绍下面的内容：
- 套接字连接的工作原理
- 套接字的属性、地址和通信
- 网络信息和互联网守护进程（inetd/xinetd）
- 客户和服务器


## 什么是套接字

套接字（socket）是一种通信机制，
凭借这种机制，客户/服务器系统的开发工作既可以在本地
单机上进行，也可以跨网络进行。

Linux所提供的功能（如打印服务、连接数据库和提供Web页面）
和网络工具（如用于远程登录的rlogin和用于文件
传输的ftp）通常都是通过套接字来进行通信的。

套接字的创建和使用与管道是有区别的，因为
==套接字明确地将客户和服务器区分开来==。套接字机
制==可以实现将多个客户连接到一个服务器==。


## 套接字连接

首先，服务器应用程序用系统调用socket来创
建一个套接字，它是系统分配给该服务器进程的类
似文件描述符的资源，它不能与其他进程共享。

接下来，服务器进程会给套接字起个名字。本
地套接字的名字是Linux文件系统中的文件名，一
般放在/tmp或/usr/tmp目录中。对于网络套接
字，它的名字是与客户连接的特定网络有关的服务
标识符（端口号或访问点）。这个标识符允许
Linux将进入的针对特定端口号的连接转到正确的
服务器进程。例如，Web服务器一般在80端口上创
建一个套接字，这是一个专用于此目的的标识符。
Web浏览器知道对于用户想要访问的Web站点，应
该使用端口80来建立HTTP连接。我们用系统调用
bind来给套接字命名。然后服务器进程就开始等待
客户连接到这个命名套接字。系统调用listen的作
用是，创建一个队列并将其用于存放来自客户的进
入连接。服务器通过系统调用accept来接受客户的
连接。

服务器调用accept时，它会创建一个与原有的
命名套接字不同的新套接字。这个新套接字只用于
与这个特定的客户进行通信，而命名套接字则被保
留下来继续处理来自其他客户的连接。如果服务器
编写得当，它就可以充分利用多个连接带来的好
处。Web服务器就会这么做以同时服务来自许多客
户的页面请求。对一个简单的服务器来说，后续的
客户将在监听队列中等待，直到服务器再次准备就绪。

基于套接字系统的客户端更加简单。
客户首先调用socket创建一个未命名套接字，然后将服务器
的命名套接字作为一个地址来调用connect与服务器建立连接。



#### code: simple client/server

```C
// client

#include <sys/types.h>
#include <sys/socket.h>
#include <stdio.h>
#include <sys/un.h>
#include <unistd.h>
#include <stdlib.h>

int main()
{
    int sockfd;
    int len;
    struct sockaddr_un address;
    int result;
    char ch = 'A';

    // 创建一个套接字
    sockfd = socket(AF_UNIX, SOCK_STREAM, 0);

    // 根据服务器的情况给套接字命名
    address.sun_family = AF_UNIX;
    strcpy(address.sun_path, "server_socket");
    len = sizeof(address);

    // 将我们的套接字连接到服务器的套接字
    result = connect(sockfd, (struct sockaddr *) &address, len);
    if (result == -1)
    {
        perror("client connect failed");
        exit(EXIT_FAILURE);
    }

    // 通过sockfd进行读写操作
    write(sockfd, &ch, 1);
    read(sockfd, &ch, 1);
    printf("char from server: %c", ch);
    close(sockfd);
    
    exit(EXIT_SUCCESS);
}
```

#### ==listen() 会创建连接队列==

```C
// server

#include <sys/types.h>
#include <sys/socket.h>
#include <stdio.h>
#include <sys/un.h>
#include <unistd.h>
#include <stdlib.h>

int main()
{
    int server_sockfd, client_sockfd;
    int server_len, client_len;
    struct sockaddr_un server_address;
    struct sockaddr_un client_address;

    // 删除以前的套接字，为服务器创建一个未命名的套接字
    unlink("server_socket");
    server_sockfd = socket(AF_UNIX, SOCK_STREAM, 0);

    // 命名套接字
    server_address.sun_family = AF_UNIX;
    strcpy(server_address.sun_path, "server_socket");
    server_len = sizeof(server_address);
    bind(server_sockfd, (struct sockaddr *) &server_address, server_len);


    // 创建一个  连   接   队   列  ，开始等待客户
    listen(server_sockfd, 5);


    while (1)
    {
        char ch;
        printf("server waiting\n");

        // 接受一个连接
        client_len = sizeof(client_address);
        client_sockfd = accept(server_sockfd, (struct sockaddr *) &client_address, &client_len);

        // 对client_sockfd套接字上的客户进行读写操作
        read(client_sockfd, &ch, 1);
        ch++;
        write(client_sockfd, &ch, 1);
        close(client_sockfd);
    }
}
```

服务器程序一次只能为一个客户服务。它从客户那里读取一个字符，增加它的值，然后再把它写回去

```text
./server &

ls -lF server_socket

ps lx

./client

./client & ./client & ./client &
```


### socket属性

套接字的特性由3个属性确定，它们是：
- 域（domain）、
- 类型（type）
- 协议（protocol）。


套接字还用地址作为它的名字。地址的格式随域（又被称为协议族，protocol family）的不同而不同。每个协议族又可以使用一个或多个地址族来定义地址格式。

#### socket的域 (domain)

域指定套接字通信中使用的==网络介质==

最常见的套接字域是AF_INET，它指的是Internet网络
其底层的协议——网际协议（IP）只有一个地址族，它使用一种特定的方式来
指定网络中的计算机，即人们常说的IP地址。

IPv6 使用一个不同的套接字域 AF_INET6 和一个不同的地址格式

在系统内部，端口通过分配一个唯一的16位的整数来标识，
在系统外部，则需要通过IP地址和端口号的组合来确定

套接字作为通信的终点，它必须在开始通信之前绑定一个端口。

一般情况下，小于1024的端口号都是为系统服务保留的，并且所服务的进程必须具有超级用户权限。

X/Open规范在头文件netdb.h中定义了一个常量
IPPORT_RESERVED，它代表保留端口号的最大值。

标准服务都对应标准的端口号，所以计算机之间可以轻松地互连，而不需要首先协商一个正确的端口号

第一个例子中的域是UNIX文件系统域AF_UNIX，即使是一台还未联网的计算机上的套接字也可以使用这个域。
这个域的底层协议就是文件输入/输出，而它的地址就是文件名。
我们的服务器套接字的地址是server_socket，当我们运行服务器程序时，就可以在当前目录下看到这个地址


#### socket类型

一个套接字域可能有多种不同的通信方式，而每种通信方式又有其不同的特性。

AF_UNIX域的套接字没有这样的问题，它们提供了一个可靠的双向通信路径。

在网络域中，我们就需要注意底层网络的特性，以及不同的通信机制是如何受到它们的影响的。

因特网协议提供了两种通信机制：==流（stream）和数据报（datagram）==。
它们有着截然不同的服务层次。

- 流套接字
流套接字（在某些方面类似于标准的输入/输出
流）提供的是一个==有序、可靠、双向字节流的连
接==。因此，发送的数据可以==确保不会丢失、复制或
乱序到达==，并且在这一==过程中发生的错误也不会显
示出来==。大的消息将被分片、传输、再重组。这很
像一个文件流，它接收大量的数据，然后以小数据
块的形式将它们写入底层磁盘。流套接字的行为是
可预见的。
流套接字由类型 SOCK_STREAM 指定，它们是
在AF_INET域中通过 ==TCP/IP== 连接实现的。它们也是
AF_UNIX域中常用的套接字类型。在本章中，我们
将重点学习SOCK_STREAM套接字，因为它们在编
写网络程序时是最常用的。

- 数据报套接字
与流套接字相反，由类型SOCK_DGRAM指定
的数据报套接字==不建立和维持一个连接==。它对可以
发送的数据报的长度有限制。数据报==作为一个单独
的网络消息被传输==，它可能==会丢失、复制或乱序到达==。
数据报套接字是在AF_INET域中通过==UDP/IP==连
接实现的，它提供的是一种无序的不可靠服务
（UDP代表的是用户数据报协议）。但从资源的角
度来看，相对来说它们开销比较小，因为不需要维
持网络连接。而且因为无需花费时间来建立连接，
所以它们的速度也很快。
数据报适用于信息服务中的“单次”（singleshot）
查询，它主要用来提供日常状态信息或执行
低优先级的日志记录。它的优点是服务器的崩溃不
会给客户造成不便，也不会要求客户重启，因为基
于数据报的服务器通常不保留连接信息，所以它们
可以在不打扰其客户的前提下停止并重启。

。。DGRAM datagram 数据包

#### socket 协议
只要底层的传输机制允许不止一个协议来提供
要求的套接字类型，我们就可以为套接字选择一个
特定的协议。在本章中，我们将重点讨论UNIX网
络套接字和文件系统套接字，它们不需要你选择一
个特定的协议，只需要使用其默认值即可。



### 15.2.2 创建socket (socket())
socket sys call， 创建socket并返回 描述符。该描述符可以用来访问该套接字。

```C
#include <sys/types.h>
#include <sys/socket.h>

int socket(int domain, int type, int protocol);
```

创建的套接字是一条通信线路的一个端点。
- domain参数指定协议族，
- type参数指定这个套接字的通信类型，
- protocol参数指定使用的协议

domain可以指定的协议族：
- AF_UNIX：unix域协议(文件系统套接字)
- AF_INET：ARPA 因特网协议(UNIX网络套接字)
- AF_ISO：ISO标准协议
- AF_NS：施乐网络系统协议
- AF_IPX：Nowell IPX协议
- AD_APPLETALK：Appletalk DDS

最常用的套接字域是AF_UNIX和AF_INET，
前者用于通过UNIX和Linux文件系统实现的本地套接字，
后者用于UNIX网络套接字。


数type指定用于新套接字的通信特性。它的取值包括SOCK_STREAM和SOCK_DGRAM。
- SOCK_STREAM：
    有序，可靠，面向连接的 双向字节流。 对于AF_INET 域套接字来说，默认通过 一个TCP连接来提供这一特性。
- SOCK_DGRAM：
    数据报服务。可以用它来发送最大长度固定（通常比较小）的消息，但消息是否会被正确传递或消息是否不会乱序到达并没有保证。对于AF_INET域套接字来说，这种类型的通信是由UDP数据报来提供的。

protocol 一般由套接字类型和套接字域来决定，通常不需要选择。只有当需要选择时，我们才会用到protocol参数。将该参数设置为0表示使用默认协议

socket系统调用返回一个描述符，它在许多方
面都类似于底层的文件描述符。当这个套接字连接
到另一端的套接字后，我们就可以用read和write系
统调用，通过这个描述符来在套接字上发送和接收
数据了。close系统调用用于结束套接字连接。


### 套接字地址

每个套接字域都有其自己的地址格式。
对于AF_UNIX域套接字来说，它的地址由结构sockaddr_un来描述，该结构定义在头文件 sys/un.h 中。

```C
struct sockaddr_un {
    aa_family_t sun_family;     // AF_UNIX
    char sun_path[];        // pathname
};
```

在当前的Linux系统中，由X/Open规范定义的
类型sa_family_t在头文件sys/un.h中声明，它是短
整数类型。此外，sun_path指定的路径名长度也是
有限制的（Linux规定的是108个字符，其他系统可
能使用的是更清楚的常量，如UNIX_MAX_PATH）。


在AF_INET域中，套接字地址由结构sockaddr_in来指定，该结构定义在头文件
netinet/in.h中，它至少包含以下几个成员

```C
struct sockaddr_in {
    short int sin_family;       // AF_INET
    unsigned short int sin_port;    // port number
    struct in_addr sin_addr;        // internet address
};

struct in_addr {
    unsigned long int s_addr;
}
```

IP地址中的4个字节组成一个32位的值。
一个AF_INET套接字由它的域、IP地址和端口号来完全确定。
从应用程序的角度来看，所有套接字的行为就像文件描述符一样，并且通过一个唯一的整数值来区分。


### 命名套接字 (bind)

要想让通过socket调用创建的套接字可以==被其他进程使用==，服务器程序就必须给该套接字命名。
这样，AF_UNIX套接字就会关联到一个文件系统的
路径名，正如你在server1例子中所看到的。
AF_INET套接字就会关联到一个IP端口号。

```C
#include <sys/socket.h>

int bind(int socket, const struct sockaddr *address, size_t address_len);
```

bind系统调用把参数address中的地址分配给与
文件描述符socket关联的未命名套接字。地址结构
的长度由参数address_len传递。

地址的长度和格式取决于地址族。bind调用需
要将一个特定的地址结构指针转换为指向通用地址
类型（struct sockaddr *）。

bind调用在成功时返回0，失败时返回-1并设置errno为表15-2中的一个值
- EBADF: 文件描述符无效
- ENOTSOCK: 文件描述符对应的不是一个套接字
- EINVAL: 文件描述符对应的是一个已经命名的套接字
- EADDRNOTAVAIL: 地址不可用
- EADDRINUSE: 地址已经绑定了一个socket

AF_UNIX 域套接字还有其他一些错误代码
- EACCESC: 权限不足
- ENOTDIR, ENAMETOOLONG: 文件名不符合要求


### 15.2.5 创建==socket队列== (listen)

为了能够在套接字上接受进入的连接，服务器
程序必须创建一个==队列==来保存==未处理==的请求。它用
listen系统调用来完成这一工作。

```C
#include <sys/socket.h>

int listen(int socket, int backlog);
```

Linux系统可能会对队列中可以容纳的未处理连接的最大数目做出限制。
为了遵守这个最大值限制，listen函数将队列长度设置为backlog参数的值。
在套接字队列中，等待处理的进入连接的个数最多不能超过这个数字。
再往后的连接将被==拒绝==，导致客户的连接请求失败。
listen函数提供的这种机制允许当服务器程序正忙于处理前一个客户请求
的时候，将后续的客户连接放入队列等待处理。

backlog参数常用的值是5。
。。5也太低了吧。

listen函数在成功时返回0，失败时返回-1。错
误代码包括EBADF、EINVAL和ENOTSOCK，其含
义与上面bind系统调用中说明的一样。


### 接受连接 (accept)

服务器程序==创建并命名==了套接字之后，它
就可以通过==accept==系统调用来等待客户建立对该套接字的连接

```C
#include <sys/socket.h>

int accept(int socket, struct sockaddr *address, size_t *address_len);
```

accept系统调用只有当有客户程序试图连接到由socket参数指定的套接字上时才返回
。。即，服务器调用accept后会阻塞。

这里的客户是指，在套接字队列中排在第一个的未处理连接。
accept函数将==创建一个新套接字==来与该==客户==进行通信，并且返回新套接字的描述符。
新套接字的类型和服务器监听套接字类型是一样的。


套接字必须事先由bind调用命名，并且由listen
调用给它分配一个连接队列。连接客户的地址将被
放入address参数指向的sockaddr结构中。如果我
们不关心客户的地址，也可以将address参数指定
为空指针。

参数address_len指定客户结构的长度。如果客
户地址的长度超过这个值，它将被截断。所以在调
用accept之前，address_len必须被设置为预期的
地址长度。当这个调用返回时，address_len将被
设置为连接客户地址结构的实际长度


如果套接字队列中没有未处理的连接，accept
将==阻塞==（程序将暂停）直到有客户建立连接为止。
我们可以通过对套接字文件描述符设置
==O_NONBLOCK==标志来改变这一行为，使用的函数
是fcnt1，如下所示：

```C
int flags = fcntl(socket, F_GETFL, 0);

fnctl(socket, F_SETFL, O_NONBLOCK | flags);
```

当有未处理的客户连接时，accept函数将返回一个新的套接字文件描述符。
发生错误时，accept函数将返回-1。
可能的错误情况大部分与bind、listen调用类似，其他的错误有EWOULDBLOCK和EINTR。
前者是当指定了O_NONBLOCK标志，但队列中没有未处理连接时产生的错误。
后者是当进程阻塞在accept调用时，执行被中断而产生的错误。

。。没有 客户连接时， 返回 -1， 错误是 EWOULDBLOCK


### 15.2.7 请求连接 (connect)

客户程序通过在一个未命名套接字和服务器监
听套接字之间建立连接的方法来连接到服务器。它
们通过connect调用来完成这一工作。

```C
#include <sys/socket.h>

int connect(int socket, const struct sockaddr *address, size_t address_len);
```

参数socket指定的套接字将连接到参数address
指定的服务器套接字，address指向的结构的长度
由参数address_len指定。参数socket指定的套接
字必须是通过socket调用获得的一个有效的文件描述符。

成功时，connect调用返回0，失败时返回-1。
可能的错误代码见表15-4。
- EBADF: 传递给socket参数的 文件描述符无效
- EALREADY: 该套接字上已经有一个正在进行中的连接
- ETIMEOUT: 连接超时
- ECONNREFUSED: 连接请求被服务器拒绝

如果连接不能立刻建立，connect调用将阻塞一
段不确定的超时时间。一旦这个超时时间到达，连
接将被放弃，connect调用失败。

但如果connect调用被一个==信号中断==，而该信号又得到了处理，
connect调用还是会失败（errno被设置为 EINTR），但连接尝试并不会被放弃，而是以==异步方式继续建立==，程序必须在此后进行检查以查看连接是否成功建立。

与accept调用一样，connect调用的阻塞特性
可以通过设置该文件描述符的==O_NONBLOCK==标志
来改变。此时，如果连接不能立刻建立，connect
将==失败==并把errno设置为EINPROGRESS，而连接
将以==异步==方式继续进行。

虽然异步连接难于处理，但我们可以在套接字
文件描述符上，用==select调用来检查套接字是否已
处于写就绪状态==。我们将在本章的后面介绍select调用。


### 关闭套接字 (close)

调用close函数来终止服务器和客户上的套接字连接，就如同对底层文件描述符进行关闭一样。
你应该总是在连接的==两端都关闭==套接字。
对于服务器来说，应该在read调用返回0时关闭套接字，但如果套接字是一个面向连接类型的，并且设置了SOCK_LINGER选项，close调用会在该套接
字还有未传输数据时==阻塞==。
你将在本章后面的内容中学习到如何设置套接字选项。


### socket通信

示例程序中我们将尽量使用网络套接字而不是文件系统套接字。

文件系统套接字的缺点是，除非程序员使用一个绝对路径名，否则套接字
将创建在服务器程序的当前目录下。为了让它更具
通用型，你需要将它创建在一个服务器及其客户都
认可的可全局访问的目录（如/tmp目录）中。

而对网络套接字来说，你只需要选择一个未被使用的端口号即可。

其他端口号及通过它们提供的服务通常都列在系统文件==/etc/services==中。

因为UNIX计算机通常会配置了一个只包含它自身的回路
（loopback）网络。出于演示的目的，我们将使用这个回路网络

==回路网络==中只包含一台计算机，传统上它被称为==localhost==，它有一个标准的IP地址==127.0.0.1==。


#### code net socket (exist error)

```C
// client
#include <sys/types.h>
#include <sys/socket.h>
#include <stdio.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <stdlib.h>

int main ()
{
    int sockfd;
    int len;
    struct sockaddr_in address;
    int result;
    char ch = 'A';

    sockfd = socket(AF_INET, SOCK_STREAM, 0);
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = inet_addr("127.0.0.1");
    address.sin_port = 9999;
    len = sizeof(address);

    // 下面和 01_simple_client 中一样。所以复制
    
    // 将我们的套接字连接到服务器的套接字
    result = connect(sockfd, (struct sockaddr *) &address, len);
    if (result == -1)
    {
        perror("client connect failed");
        exit(EXIT_FAILURE);
    }

    // 通过sockfd进行读写操作
    write(sockfd, &ch, 1);
    read(sockfd, &ch, 1);
    printf("char from server: %c", ch);
    close(sockfd);
    
    exit(EXIT_SUCCESS);
}
```

客户程序用在头文件netinet/in.h中定义的sockaddr_in结构指定了一个AF_INET地址。
它试图连接到IP地址为127.0.0.1的主机上的服务器。
它用 `inet_addr` 函数将IP地址的文本表示方式转换为符合`套接字地址`要求的格式。
inet的手册页中有对其他地址转换函数的详细说明。


```C
// server
#include <sys/types.h>
#include <sys/socket.h>
#include <stdio.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <stdlib.h>

int main()
{
    int server_sockfd, client_sockfd;
    int server_len, client_len;
    struct sockaddr_in server_address;
    struct sockaddr_in client_address;

    server_sockfd = socket(AF_INET, SOCK_STREAM, 0);
    server_address.sin_family = AF_INET;
    server_address.sin_addr.s_addr = inet_addr("127.0.0.1");
    server_address.sin_port = 9999;

    server_len = sizeof(server_address);
    bind(server_sockfd, (struct sockaddr *) &server_address, server_len);

    // same with 01

    // 创建一个  连   接   队   列  ，开始等待客户
    listen(server_sockfd, 5);


    while (1)
    {
        char ch;
        printf("server waiting\n");

        // 接受一个连接
        client_len = sizeof(client_address);
        client_sockfd = accept(server_sockfd, (struct sockaddr *) &client_address, &client_len);

        // 对client_sockfd套接字上的客户进行读写操作
        read(client_sockfd, &ch, 1);
        ch++;
        write(client_sockfd, &ch, 1);
        close(client_sockfd);
    }
}
```

。。
./server &

./client
。。

如果想允许服务器和远程客户进行通信，就必须指定一组你允许连接的IP地址。
可以用特殊值INADDR_ANY来表示，你将接受来自计算机任何网络接口的连接。
如果你愿意，还可以通过分离如内部局域网和外部广域网连接的方式来区分不同的网络接口。
INADDR_ANY是一个32位的整数值，它可以用在地址结构的sin_addr.s_addr域中



### 15.2.10 主机字节序 和网络字节序


netstat命令来查看网络连接状况。
显示了客户/服务器连接正在等待关闭。连接将在一小段超时时间之
后关闭（具体的输出内容将随Linux版本的不同而不同）。


。。
$ netstat -A inet
激活Internet连接 (w/o 服务器)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 yiqi:40642              151.101.78.49:https     ESTABLISHED
udp        0      0 yiqi:bootpc             192.168.70.254:bootps   ESTABLISHED

$ ./1 & ./2             # 这个就是启动server,client。
[4] 9311
server waiting
char from server: B
server waiting

$ netstat -A inet
激活Internet连接 (w/o 服务器)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 yiqi:40642              151.101.78.49:https     ESTABLISHED
tcp        0      0 localhost:3879          localhost:60544         TIME_WAIT  
udp        0      0 yiqi:bootpc             192.168.70.254:bootps   ESTABLISHED
。。
。。但是我的端口不是9999吗？ 为什么是 3879。
。。看书上的例子，也不是 书上的代码中的 9734


local address一栏显示的是服务器，
foreign address一栏显示的是远程客户（即使是在同一台机器上，它仍然是通过网络连接的）。

。。local 竟然是服务器。


可是，显示的本地地址（服务器套接字）端口是1574（或者你可能会看到显示的是一个服务名mvel-lm），而我们选择的端口是9734。
为什么会不一样呢？
答案是，通过套接字接口传递的端口号和地址都是二进制数字。
不同的计算机使用==不同的字节序==来表示整数。
例如，Intel处理器将32位的整数分为4个连续的字节，并以字节序1-2-3-4存储
到内存中，这里的1表示最高位的字节。
而IBMPowerPC处理器是以字节序4-3-2-1的方式来存储
整数。如果保存整数的内存只是以逐个字节的方式
来复制，两个不同的计算机得到的整数值就会不一致。


。。
9999 -> 00100111 00001111
3879 ->     1111 00100111


9734 -> 100110 00000110
1574 ->    110 00100110

。。确实，8位一个字节。


为了使不同类型的计算机可以就通过网络传输的多字节整数的值达成一致，你需要定义一个网络字节序。
客户和服务器程序必须在传输之前，将它们的内部整数表示方式转换为网络字节序。
它们通过定义在头文件`netinet/in.h`中的函数来完成这一工作。
这些函数如下所示

```C
#include <netinet/in.h>

unsigned long int htonl(unsigned long int hostlong);
unsigned short int htons(unsigned short int hostshort);
unsigned long int ntohl(unsigned long int netlong);
unsigned short int ntohs(unsigned short int netshort);
```

这些函数将16位和32位整数在主机字节序和标准的网络字节序之间进行转换

函数名是简写：
htonl: host to network, long

```C
// server的改动
// 主机字节序, 网络字节序
// server_address.sin_addr.s_addr = inet_addr("127.0.0.1");
// server_address.sin_port = 9999;
server_address.sin_addr.s_addr = htonl(INADDR_ANY);
server_address.sin_port = htons(9999);

// client的改动
// address.sin_port = 9999;
address.sin_port = htons(9999);
```

用INADDR_ANY来允许到达服务器任一网络接口的连接。

。。现在 `netstat -A inet` 对了，是9999


## 15.3 网络信息 (vi /etc/services)
之前都是把地址和端口号编译到它们自己的内部
可以通过 网络信息函数 来决定应该使用的地址和端口。


```text
domain-s     853/tcp     # DNS over TLS [RFC7858]
domain-s     853/udp     # DNS over DTLS [RFC8094]
```
> 同一个端口 既可以udp，又可以tcp


如果你有足够的权限，也可以将自己的服务添加到/etc/services文件中的已知服务列表中，并在这个文件中为端口号分配一个名字，使用户可以使用==符号化的服务名==而不是端口号的数字。

如果给定一个计算机的名字，你可以通过调用解析地址的主机数据库函数来确定它的IP地址。
这些函数通过查询网络配置文件来完成这一工作，如/etc/hosts文件或网络信息服务

常用的网络信息服务有 `NIS`（Network Information
Service，网络信息服务，以前称为Yellow Pages，
黄页服务）和 `DNS`（Domain Name Service，域名服务）。

### gethostbyname/addr
主机数据库函数在接口头文件netdb.h中声明
```C
#include <netdb.h>

struct hostent *gethostbyaddr(const void *addr, size_t len, int type);
struct hostent *gethostbyname(const char *name);
```
如果没有，返回空指针。
如果有，返回的hostent至少包含：
```C
struct hostent {
    char *h_name;   // name of the host
    char **h_aliases;   // list of aliases (nicknames)
    int h_addrtype; // address type
    int h_length;   // length in bytes of the address
    char **h_addr_list; // list of address (network order)
}
```

### getservbyname/port
与服务及其关联端口号有关的信息也可以通过一些服务信息函数来获取
```C
#include <netdb.h>

struct servent *getservbyname(const char *name, const char *proto);
struct servent *getservbyport(int port, const char *proto);
```

proto参数指定用于连接该服务的协议，它的两
个==取值是tcp和udp==，前者用于SOCK_STREAM类
型的TCP连接，后者用于SOCK_DGRAM类型的UPD数据报。

。。datagram

servent至少包含
```C
struct servent {
    char *s_name;   // service name
    char **s_aliases;   // aliases (alternative names)
    int s_port;     // ip port number
    char *s_proto;  // service type, usually tcp or udp
}
```

如果想获得某台计算机的主机数据库信息，可
以调用gethostbyname函数并且将结果打印出来。
注意，要把返回的地址列表 转换 为正确的地址类
型，并用函数inet_ntoa将它们从网络字节序转换为
可打印的字符串。

```C
#include <arps/inet.h>

char *inet_ntoa(struct in_addr in);
```

将一个因特网主机地址转换为一个点分四元组格式的字符串。它在失败时返
回-1，但POSIX规范并未定义任何错误。
其他可用的新函数还有gethostname，它的定义如下所示：

```C
#include <unistd.h>

int gethostname(char *name, int namelength);
```

这个函数的作用是，将当前主机的名字写入
name指向的字符串中。主机名将以null结尾。参数
namelength指定了字符串name的长度，如果返回
的主机名太长，它就会被截断。gethostname在成
功时返回0，失败时返回-1，但POSIX规范中没有
定义任何错误。



#### code: get host info


```C
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <netdb.h>
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
    char *host, **names, **addrs;
    struct hostent *hostinfo;

    if (argc == 1)
    {
        char myname[256];
        gethostname(myname, 255);
        host = myname;
    }
    else
    {
        host = argv[1];
    }

    hostinfo = gethostbyname(host);
    if (!hostinfo)
    {
        fprintf(stderr, "cannot get info for host: %s\n", host);
        exit(1);
    }

    printf("result for host: %s\n", host);
    printf("name: %s\n", hostinfo -> h_name);

    printf("aliases:");
    names = hostinfo -> h_aliases;
    while (*names)
    {
        printf("  %s", *names);
        names++;
    }
    printf("\n");

    if (hostinfo -> h_addrtype != AF_INET)
    {
        fprintf(stderr, "not an ip host\n");
        exit(1);
    }

    addrs = hostinfo -> h_addr_list;
    while (*addrs)
    {
        printf("  %s", inet_ntoa(*(struct in_addr *) *addrs));
        addrs++;
    }
    printf("\n");
    exit(EXIT_SUCCESS);
}
```


你也可以用gethostbyaddr函数来查出
哪个主机拥有给定的IP地址。你可以在服务器上用
这个函数来查找连接客户的来源。

。。
./a.out XDZ-PC

result for host: XDZ-PC
name: XDZ-PC.localdomain
aliases:
  2.0.0.1
。。


#### code: 自己写的gethostbyaddr，但是不行。

```text

// 这里是最后的一些挣扎，(最后的波纹)。 应该放最后的。

man 3 gethostbyaddr
    herror(), hstrerror()
    h_errno

。。herror()：no such file or directorty。。反正就是没有这个方法
。。hstrerror() 可以

ip: 180.101.50.242, 24, 8
host: (nil)
get host error: 180.101.50.242
 -- 0 -- 1
 -- Unknown host

ip: 220.185.184.29, 24, 8
host: (nil)
get host error: 220.185.184.29
 -- 0 -- 1
 -- Unknown host

ip: 127.0.0.1, 24, 8
host: 0x7f1f3d021a00
official name: localhost
alias: 0: (null)
address type: AF_INET
ip address: 0: 127.0.0.1

。。总不可能让我在 host里加 ip host对应吧。 我。。。

。。算是个结束吧。


(int) (sizeof(ips) / sizeof(ips[0]))
。int 不加 () 会报错

```



。。自己写了一个 gethostbyaddr 的例子，只能获得127.0.0.1的。而且信息还挺怪的。。无法获得 百度，csdn的。(百度，csdn是自己ping到的ip)

。。
ip: 180.101.50.242, 24, 8
host: (nil)
get host error: 180.101.50.242
 -- 0 -- 1
 -- Operation not permitted

ip: 220.185.184.29, 24, 8
host: (nil)
get host error: 220.185.184.29
 -- 0 -- 1
 -- Operation not permitted

ip: 127.0.0.1, 24, 8
host: 0x7fc957021a00
official name: localhost
alias: 0: (null)
address type: AF_INET
ip address: 0: 127.0.0.1
。。

```C
// strerror 能解析 h_error 吗？ 感觉翻译的不对，估计要man里看下。
// 昨天是2：No such file or directory， 今天跑了下是 1：operation not permitted
// h_error 是什么？ 我看是 netdb里的， 但是 书上没有介绍。
// error 是 errno.h 里的。#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <netdb.h>
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <string.h>


// https://blog.csdn.net/weixin_42839065/article/details/131065640
int main()
{
    // char* ip = "180.101.50.242";    // baidu
    // char* ip2 = ""

    // baidu, blog.csdn
    // 2.0.0.1 是 虚拟机的 宿主机的联系IP
    // 131 是宿主机的自己的局域网IP , 1是网关，98是131 arp -a 中随便找了一台
    // 127.0.0.1 ke yi, 其他都是 unknow host，
    char* ips[] = {"180.101.50.242","220.185.184.29","127.0.0.1","2.0.0.1"
    ,"192.168.18.131","192.168.18.1","192.168.18.98"};

    struct hostent *host;
    struct sockaddr_in addr;
    for (int i = 0; i < sizeof(ips) / sizeof(ips[0]); ++i)
    {
        fprintf(stdout, "ip: %s, %lu, %lu, %d, %d\n", ips[i], sizeof(ips), sizeof(ips[i]), (int) (sizeof(ips) / sizeof(ips[0])), i);
        
        // addr.sin_addr.s_addr = inet_addr(ips[i]);
        if (!inet_aton(ips[i], &addr.sin_addr)) 
        {
            printf("inet_aton error\n");
            continue;
        }

        host = gethostbyaddr((void *) &addr.sin_addr, 4, AF_INET);
        
        printf("host: %p\n", host);     // nil
        // nil + errno==0, 确实没有找到

        if (!host)
        {
            fprintf(stderr, "get host error: %s\n", ips[i]);
            // errno - errno.h , h_errno - netdb.h
            fprintf(stderr, " -- %d -- %d\n", errno, h_errno);     // 0  2
            // 之前用的 strerror
            fprintf(stderr, " -- %s\n", hstrerror(h_errno)); // No such file or directory .. 
            continue;
        }
        printf("official name: %s\n", host -> h_name);

        for (int i = 0; i < sizeof(host -> h_aliases) / sizeof(host -> h_aliases[0]); ++i)
        {
            printf("alias: %d: %s\n", i, host -> h_aliases[i]);
        }

        printf("address type: %s\n", host -> h_addrtype == AF_INET ? "AF_INET" : "AF_INET6");

        for (int i = 0; i < sizeof(host -> h_addr_list) / sizeof(host -> h_addr_list[0]); ++i)
        {
            printf("ip address: %d: %s\n", i, inet_ntoa(*(struct in_addr *) host -> h_addr_list[i]));
        }
    }
    exit(EXIT_SUCCESS);
}
```



#### code: connect to service

连接到一个标准服务，这样就可以演示端口号的提取操作了

一项标准服务daytime，它提供系统的日期和时间

```C
#include <sys/socket.h>
#include <netinet/in.h>
#include <netdb.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
    char *host;
    int sockfd;
    int len, result;
    struct sockaddr_in address;
    struct hostent *hostinfo;
    struct servent *servinfo;
    char buffer[128];

    if (argc == 1)
        host = "localhost";
    else
        host = argv[1];
    
    // 查找主机的地址
    hostinfo = gethostbyname(host);
    if (!hostinfo)
    {
        fprintf(stderr, "no host: %s\n", host);
        exit(1);
    }

    // 检查主机上是否有daytime服务
    servinfo = getservbyname("daytime", "tcp");
    if (!servinfo)
    {
        fprintf(stderr, "no daytime service\n");
        exit(EXIT_FAILURE);
    }
    printf("daytime port is %d\n", ntohs(servinfo->s_port));

    // 创建一个套接字
    sockfd = socket(AF_INET, SOCK_STREAM, 0);

    // 构造connect调用要使用的地址
    address.sin_family = AF_INET;
    address.sin_port = servinfo->s_port;
    address.sin_addr = *(struct in_addr *) *hostinfo->h_addr_list;
    len = sizeof(address);

    // 建立连接并取得有关信息
    result = connect(sockfd, (struct sockaddr *) &address, len);
    if (result == -1)
    {
        perror("getdate error");
        exit(EXIT_FAILURE);
    }

    result = read(sockfd, buffer, sizeof(buffer));
    buffer[result] = '\0';
    printf("get %d bytes: %s\n", result, buffer);

    close(sockfd);
    exit(EXIT_SUCCESS);
}
```

。。
daytime port is 13
getdate error: Connection refused

。。

如果你看到 connection refused 或 no such file or directory。

最新版本的Linux系统在默认情况下都没有启用该服务。
在下一节中，你将学习如何启用这项服务以及其他一些服务。


### 15.3.1 因特网守护进程(xinetd/inetd)


UNIX系统通常以超级服务器的方式来提供多项网络服务。
超级服务器程序（因特网守护进程xinetd或inetd）同时监听许多端口地址上的连接。
当有客户连接到某项服务时，守护程序就运行相应的服务器。
这使得针对各项网络服务的服务器不需要一直运行着，它们可以在需要时启动。

xinetd 取代了原来的 inetd

我们通常是通过一个图形用户界面来配置xinetd以管理网络服务，但我们也可以直接修改它的配置文件。
它的配置文件通常是/etc/xinetd.conf和/etc/xinetd.d目录中的文件。

。。ubuntu 22 没有这些配置文件。
。默认情况下ubuntu没有安装xinetd服务
。。。
$ sudo apt install xinetd

/etc$ cd xinetd.d/

/etc/xinetd.d$ ls
chargen      daytime      discard      echo      servers   time
chargen-udp  daytime-udp  discard-udp  echo-udp  services  time-udp

vi daytime
可以看到下面提到的 type=internal， tcp， udp 都有。

。。有了。


每一个由xinetd提供的服务都在 `/etc/xinetd.d` 目录中有一个对应的配置文件。
xinetd将在其启动时或被要求的情况下读取所有这些配置文件。

我们的getdate程序连接的daytime服务实际上就是由xinetd自身负责处理的（它被标记为internal，即内部），
它同时支持SOCK_STREAM（tcp）和SOCK_DGRAM（udp）套接字。



ftp文件传输服务只支持SOCK_STREAM套接字，并且是由一个外部程序来提供服务的。在本例中这个程序是vsftpd，当有客户连接到ftp的端口时，守护进程就会启动它。
。。这个我没有。

为了激活服务配置的修改，你需要编辑xinetd的配置文件，然后发送一个挂起信号给守护进程，但我们建议你使用一种更加友好的方式来配置服务。
为了允许time-of-day客户进行连接，你可以使用Linux系统提供的工具来启用daytime服务。
对于SUSE和openSUSE系统来说，你可以通过SUSE控制中心来配置服务，如图15-1所示。
Red Hat的版本（包括企业版Linux和Fedora）也有一个类似的配置界面。
在图15-1中，daytime服务同时针对TCP和UDP查询进行了启用。

。。ubuntu没有这个。。
直接 root vi daytime文件，将 disable 从yes改为no。
然后
service xinetd restart

之前的代码就可以正确了，可以获得 时间：
get 26 bytes: 21 SEP 2023 13:27:42 CST
。。



### 15.3.2 套接字选项 (setsockopt)

```C
#include <sys/socket.h>

int setsockopt(int socket, int level, int option_name, const void *option_value, size_t option_len);
```

在协议层次的不同级别对选项进行设置。

如果想要在套接字级别设置选项，就必须将level参数设置为SOL_SOCKET。
如果想要在底层协议级别（如TCP、UDP等）设置选项，就必须将
level参数设置为该协议的编号（可以通过头文件
==netinet/in.h== 或 函数getprotobyname来获得）。

。。应该都是 SOL_开头的，不过好分散。
in.h, socket.h, socket-constants.h
。。

在头文件sys/socket.h中定义的套接字级别选项
- SO_DEBUG，打开调试信息
- SO_KEEPALIVE，通过定期传输 保持存活报文 来维持连接
- SO_LINGER，在close 调用返回之前完成 传输工作

SO_DEBUG和SO_KEEPALIVE用一个整数的option_value值来设置该选项的开（1）或关（0）。
SO_LINGER需要使用一个在头文件 sys/socket.h 中定义的linger结构，来定义该选项的状态以及套接字关闭之前的拖延时间。

setsockopt在成功时返回0，失败时返回-1。它的手册页介绍了更多的选项和错误。


## 15.4 多客户

服务器程序在接受来自客户的一个新连接时，会==创建==出一个新的套接字，而
==原先==的监听套接字将==被保留以继续==监听以后的连接。
如果服务器==不能立刻接受==后来的连接，它们将被==放到队列==中以等待处理。

原先的套接字仍然可用并且套接字的行为就像文件描述符，这一事实给我们提供了一种同时服务多个客户的方法。

如果服务器调用==fork==为自己创建第二份副本，打开的套接字就将被新的子进程所继承。

新的子进程可以和连接的客户进行通信，而主服务器进程可以继续接受以后的客户连接。

这些改动对我们的服务器程序来说是非常容易的，下面的实验部分将给出修改过的服务器程序。

因为我们创建子进程，但并不等待它们的完成，所以必须安排服务器忽略SIGCHLD信号以避免出现僵尸进程

#### code: server (multi client)

```C
#include <sys/types.h>
#include <sys/socket.h>
#include <stdio.h>
#include <netinet/in.h>
#include <signal.h>
#include <unistd.h>
#include <stdlib.h>

int main()
{
    int server_sockfd, client_sockfd;
    int server_len, client_len;
    struct sockaddr_in server_address;
    struct sockaddr_in client_address;

    server_sockfd = socket(AF_INET, SOCK_STREAM, 0);

    server_address.sin_family = AF_INET;
    server_address.sin_addr.s_addr = htonl(INADDR_ANY);
    server_address.sin_port = htons(9999);
    server_len = sizeof(server_address);
    bind(server_sockfd, (struct sockaddr *) &server_address, server_len);

    listen(server_sockfd, 5);

    signal(SIGCHLD, SIG_IGN);

    while (1)
    {
        char ch;
        printf("server waiting\n");

        client_len = sizeof(client_address);
        client_sockfd = accept(server_sockfd, (struct sockaddr*) &client_address, &client_len);

        if (fork() == 0)
        {
            read(client_sockfd, &ch, 1);
            printf("start to sleep...\n");
            sleep(5);
            printf("end of sleep...\n");
            ch++;
            write(client_sockfd, &ch, 1);
            close(client_sockfd);
            exit(0);
        }
        else
        {
            close(client_sockfd);
        }
    }

}
```

。。
./1 &

./2 & ./2 & ./ 2 & ps x
。。




我们真正需要的是，如何让单个服务器进程在不阻塞、不等待客户请求到达的前提下处理多个客户。
这个问题的解决方案涉及如何同时处理多个打开的文件描述符，并且它不仅仅局限于套接字应用程序，请看下一节的select系统调用。


### 15.4.1 select 系统调用

我们经常会遇到需要检查好几个输入的状态才能确定下一步行动的情况。
例如，像终端仿真器这样的通信程序，需要有效地同时读取键盘和串行口。
如果是在一个单用户系统中，运行一个“忙等待”循环还是可以接受的，它不停地扫描输入设备看是否有数据，如果有数据到达就读取它。
但这种做法很消耗CPU的时间。

select系统调用允许程序==同时在多个==底层文件描述符上==等待输入==的到达（或输出的完成）。
这意味着终端仿真程序可以一直阻塞到有事情可做为止。
类似地，服务器也可以通过==同时在多个==打开的套接字上==等待请求==到来的方法来处理多个客户。

select函数对数据结构`fd_set`进行操作，它是由==打开的文件描述符构成的集合==
有一组定义好的宏可以用来控制这些集合
```C
#include <sys/types.h>
#include <sys/time.h>

void FD_ZERO(fd_set *fdset);
void FD_CLR(int fd, fd_set *fdset);
void FD_SET(int fd, fd_set *fdset);
int FD_ISSET(int fd, fd_set *fdset);
```

FD_ZERO用于将fd_set初始化为空集合，
FD_SET和FD_CLR分别用于在集合中设置和清除由参数fd传递的文件描述符。
FD_ISSET宏中由参数fd指向的文件描述符是由参数fdset指向的fd_set集合中的一个元素，FD_ISSET将返回非零值。

fd_set结构中可以容纳的文件描述符的最大数目由常量FD_SETSIZE指定。


select函数还可以用一个超时值来防止无限期的阻塞。
这个超时值由一个timeval结构给出。
这个结构定义在头文件sys/time.h中，它由以下几个成员组成

```C
struct timeval {
    time_t tv_sec;  // second
    long tv_usec;   // micro second
}
```

```C
#include <sys/types.h>
#include <sys/time.h>

int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *errorfds, struct timeval *timeout);
```

select调用用于测试文件描述符==集合中==，==是否有一个==文件描述符已处于==可读==状态或==可写==状态或==错误==状态，它将阻塞以等待某个文件描述符进入上述这些状态。

参数nfds指定需要测试的文件描述符数目，测试的描述符范围从0到nfds-1。
3个描述符集合都可以被设为空指针，这表示不执行相应的测试。

select函数会在发生以下情况时返回：
- readfds集合中有描述符可读、
- writefds集合中有描述符可写
- errorfds集合中有描述符遇到错误条件。

如果这3种情况都没有发生，select将在timeout指定的超时时间经过后返回。
如果timeout参数是一个空指针并且套接字上也没有任何活动，这个调用将一直阻塞下去。

当select返回时，描述符集合将被修改以指示哪些描述符正处于可读、可写或有错误的状态。
我们可以用==FD_ISSET对描述符进行测试==，来==找出需要注意的描述符==。
你可以修改timeout值来表明剩余的超时时间，但这并不是在X/Open规范中定义的行为。
如果select是因为超时而返回的话，所有描述符集合都将被清空。

。。描述符集合，就是 那3个参数 readfd, writefd, errorfd 吧。
。。那么 socket返回的时候， 这3个参数 的元素个数 有没有发生变化？

select调用返回状态==发生变化的描述符总数==。
失败时它将返回-1并设置errno来描述错误。
可能出现的错误有：
- EBADF（无效的描述符）
- EINTR（因中断而返回）
- EINVAL（nfds或timeout取值错误）

Linux系统会把参数timeout指向的结构修改为剩余的超时时间，但大多数UNIX版本不会这样做


#### code: select sys-call

```C
#include <sys/types.h>
#include <sys/time.h>
#include <stdio.h>
#include <fcntl.h>
#include <sys/ioctl.h>
#include <unistd.h>
#include <stdlib.h>

int main()
{
    char buffer[128];
    int result, nread;

    fd_set inputs, testfds;
    struct timeval timeout;

    FD_ZERO(&inputs);
    FD_SET(0, &inputs);

    while(1)
    {
        testfds = inputs;
        timeout.tv_sec = 2;     // 2.5s
        timeout.tv_usec = 500000;
        result = select(FD_SETSIZE, &testfds, (fd_set *) NULL, (fd_set *) NULL, &timeout);

        switch(result)
        {
        case 0:
            printf("timeout\n");
            break;
        case -1:
            perror("select error");
            exit(EXIT_FAILURE);
        default:
            if (FD_ISSET(0, &testfds))
            {
                ioctl(0, FIONREAD, &nread);
                if (nread == 0)
                {
                    printf("keybowrd done\n");
                    exit(EXIT_SUCCESS);
                }
                nread = read(0, buffer, nread);
                buffer[nread] = 0;
                printf("read %d from keyboard: %s\n", nread, buffer);
            }
            break;
        }
    }
}
```

。。启动后，狂打字就可以了。 ctrl+d 退出。 要2下 ctrl+d。
。。timeout 是干扰。   
。。回车也可以刷新 fd， 导致 最后退出的时候 是上次 回车 到现在的 字符。


运行这个程序时，它会每隔2.5秒打印一个timeout。
如果在键盘上敲入字符，它就会从标准输入读取数据并报告敲入的内容。
对大多数shell来说，输入会在用户按下==回车键或某个控制序列==时被
发送给程序，所以这个程序将在你按下回车键时把输入内容显示出来。
注意，回车键本身也像其他字符一样被读取和处理（你可以尝试不按下回车键，
而是在敲入几个字符后按下组合键Ctrl+D，看看会怎么样）。


### 15.4.2 多客户

通过用select调用来同时处理多个客户就无需再依赖于子进程了

必须要注意，不能在处理第一个连接的客户时让其他客户等太长的时间。

用FD_ISSET来遍历所有可能的文件描述符，以检查是哪个上面有活动发生。

如果是监听套接字可读，这说明正有一个客户试图建立连接，此时就可以调用accept而不用担心发生阻塞的可能。
如果是某个客户描述符准备好，这说明该描述符上有一个客户请求需要我们读取和处理
如果读操作返回零字节，这表示有一个客户进程已结束，你可以关闭该套接字并把它从描述符集合中删除。

#### code: server + select

```C
#include <sys/types.h>
#include <sys/socket.h>
#include <stdio.h>
#include <netinet/in.h>
#include <sys/time.h>
#include <sys/ioctl.h>
// #include <signal.h>
#include <unistd.h>
#include <stdlib.h>
#include <asm-generic/ioctls.h>  // FIONREAD 的头文件。但书上没有这个头文件

// change base on 06_server_with_multi_client.c

int main()
{
    int server_sockfd, client_sockfd;
    int server_len, client_len;
    struct sockaddr_in server_address;
    struct sockaddr_in client_address;

    int result;
    fd_set readfds, testfds;
    
    server_sockfd = socket(AF_INET, SOCK_STREAM, 0);

    server_address.sin_family = AF_INET;
    server_address.sin_addr.s_addr = htonl(INADDR_ANY);
    server_address.sin_port = htons(9999);
    server_len = sizeof(server_address);
    bind(server_sockfd, (struct sockaddr *) &server_address, server_len);

    listen(server_sockfd, 5);

    // signal(SIGCHLD, SIG_IGN);
    FD_ZERO(&readfds);
    FD_SET(server_sockfd, &readfds);

    while (1)
    {
        char ch;
        int fd;
        int nread;

        testfds = readfds;

        printf("server waiting\n");

        result = select(FD_SETSIZE, &testfds, (fd_set *) NULL, (fd_set *) NULL, (struct timeval *) NULL);
        if (result < 1)
        {
            perror("server select failed");
            exit(EXIT_FAILURE);
        }

        for (fd = 0; fd < FD_SETSIZE; fd++)
        {
            if (FD_ISSET(fd, &testfds))
            {
                if (fd == server_sockfd)
                {
                    client_len = sizeof(client_address);
                    client_sockfd = accept(server_sockfd, (struct sockaddr *) &client_address, &client_len);
                    FD_SET(client_sockfd, &readfds);
                    printf("add client on fd %d\n", client_sockfd);
                }
                else
                {
                    ioctl(fd, FIONREAD, &nread);
                    if (nread == 0)
                    {
                        close(fd);
                        FD_CLR(fd, &readfds);
                        printf("remove client on fd %d \n", fd);
                    }
                    else
                    {
                        read(fd, &ch, 1);
                        sleep(5);
                        printf("serving client on fd %d \n", fd);
                        ch++;
                        write(fd, &ch, 1);
                    }
                }
            }
        }
    }
}
```

在实际应用的程序中，最好用一个变量来专门保存已连接套接字的==最大文件描述符号==（它不一定是最新连接的套接字文件描述符号）。
这可以避免循环检查数千个其实并未连接的套接字，它们根本不可能处于可读状态。出于简洁和让代码易于理解的目的，我们在这里没有这样做。

。。
./1

./2 & ./2 & ./2 & ps x

。。


socket 与 电话 的对比
|电话|socket|
|--|--|
|给公司打电话，号码5555555|连接到IP 127.0.0.1|
|接线员接听电话|建立起到远程主机的连接|
|要求转到财务部|转到指定端口|
|财务主管听电话|服务器从select调用返回|
|电话转给免费账号经理|服务器调用accept，在456编号上创建新的socket|




## 15.5 数据报

daytime服务还可以用数据报通过UDP来访问。
为了访问它，发送一个数据报给该服务，然后在响应中获取一个包含日期和时间的数据报。

当客户需要发送一个短小的查询请求给服务器，并且期望接收到一个短小的响应时，我们一般就使用由UDP提供的服务。
如果服务器处理客户请求的时间足够短，服务器就可以通过一次处理一个客户请求的方式来提供服务，从而允许操作系统将客户进入的请求放入队列。
这简化了服务器程序的编写。

实际上，UDP数据报在局域网中是非常可靠的

要用两个数据报专用的系统调用sendto和recvfrom来代替原来使用在套接字上的read和write调用。

```C
#include <sys/socket.h>
#include <netinet/in.h>
#include <netdb.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
    char *host;
    int sockfd;
    int len, result;
    struct sockaddr_in address;
    struct hostent *hostinfo;
    struct servent *servinfo;
    char buffer[128];

    if (argc == 1)
        host = "localhost";
    else
        host = argv[1];
    
    // 查找主机的地址
    hostinfo = gethostbyname(host);
    if (!hostinfo)
    {
        fprintf(stderr, "no host: %s\n", host);
        exit(1);
    }

    // 检查主机上是否有daytime服务
    // servinfo = getservbyname("daytime", "tcp");  // 。。这个也可以的。因为端口相同
    servinfo = getservbyname("daytime", "udp");
    if (!servinfo)
    {
        fprintf(stderr, "no daytime service\n");
        exit(EXIT_FAILURE);
    }
    printf("daytime port is %d\n", ntohs(servinfo->s_port));

    // 创建一个套接字
    // sockfd = socket(AF_INET, SOCK_STREAM, 0);
    sockfd = socket(AF_INET, SOCK_DGRAM, 0);

    // 构造connect调用要使用的地址
    address.sin_family = AF_INET;
    address.sin_port = servinfo->s_port;
    address.sin_addr = *(struct in_addr *) *hostinfo->h_addr_list;
    len = sizeof(address);

    // 建立连接并取得有关信息
    // result = connect(sockfd, (struct sockaddr *) &address, len);
    // if (result == -1)
    // {
    //     perror("getdate error");
    //     exit(EXIT_FAILURE);
    // }
    // result = read(sockfd, buffer, sizeof(buffer));

    result = sendto(sockfd, buffer, 1, 0, (struct sockaddr *) &address, len);
    result = recvfrom(sockfd, buffer, sizeof(buffer), 0, (struct sockaddr *) &address, &len);

    buffer[result] = '\0';
    printf("get %d bytes: %s\n", result, buffer);

    close(sockfd);
    exit(EXIT_SUCCESS);
}
```


因为我们并没有明确地建立一条到指定UDP服务的连接，所以必须用某些方式让服务器知道你需要接收一个响应。
在本例中，给服务器发送一个数据报（在这里，从准备接收响应的缓存区中发送一个字节的数据），它返回包含日期和时间的响应。

sendto系统调用从buffer缓存区中给使用指定套接字地址的目标服务器发送一个数据报
```C
int sendto(int sockfd, void *buffer, size_t len, int flags, struct sockaddr *to, socklen_t tolen);
```

flags参数一般被设置为0。

recvfrom系统调用在套接字上等待从特定地址到来的数据报，并将它放入buffer缓存区
```C
int recvfrom(int sockfd, void *buffer, size_t len, int flags, struct sockaddr *from, socklen_t *fromlen);
```
flags参数一般被设置为0。

当错误发生时，sendto和recvfrom都将返回-1并设置errno。
可能的错误：
- EBADF：传递一个无效的文件描述符
- EINTR：产生一个信号

除非用fcntl将套接字设置为非阻塞方式（正如在前面的接受TCP连接中看到的那样），否则recvfrom调用将一直阻塞。
我们可以用与前面的面向连接服务器一样的方式，通过select调用和超时设置来判断是否有数据到达套接字。
此外，还可以用alarm时钟信号来中断一个接收操作


# ch16 用GTK+进行GNOME编程 (跳)

。。GTK+ 在慢慢没落啊。

本章将涵盖以下内容：
- X视窗系统
- GNOME/GTK+简介
- GTK+构件
- GNOME构件和菜单
- 对话框
- 用GNOME/GTK+编写CD数据库GUI



# ch17 用Qt进行KDE编程 (跳)

KDE 3.5
Qt 3.3
。。太老了。

将介绍以下内容：
- Qt简介
- 安装Qt
- 开始编程
- 信号/槽机制
- Qt构件
- Qt对话框
- KDE环境下的菜单和工具栏
- 使用KDE/Qt创建CD数据库应用程序


# ch18 Linux 标准

主要介绍以下几方面内容。
- C编程语言标准。
- UNIX标准，特别是由IEEE开发的POSIX标准，以及由开放组织（Open Group）开发的单一UNIX规范。
- 由自由标准组织（Free StandardGroup）所做的工作，特别是Linux标准化规范（Linux Standard Base），它定义了标准的Linux文件系统布局。

了解Linux相关标准的一个好起点是Linux标准化规范（LSB），你可以通过访问Linux基金会网站
https://www.linuxfoundation.org/
https://refspecs.linuxfoundation.org/lsb.shtml

。。上面的2个网站是我 bing 的。 书上的是 http://www.linux-foundatjon.org
。。没这个网站的。 估计是 换行的 - 当做url的一部分了。

https://wiki.linuxfoundation.org/lsb/start


## 18.1 C语言标准

C语言是Linux编程语言事实上的标准

### GNU 编译器集

gcc最初只是一个GNU C语言的编译器，但由于目前该编译器的基本框架已支持C＋＋、Object-C、FORTRAN、Java和Ada等许多其他编程语言及其函数库，所以对gcc的定义被修改为更合适的GNU编译器集。

gcc始终是，并且看来以后也将会是Linux上的标准编译器，并且C或C＋＋语言也是Linux上程序设计的基本语言

。。哎， rust。。。


### gcc 选项

只介绍那些最重要的选项
完整的选项列表可以在gcc手册页中找到
还将简单介绍一些可以使用的#define选项

要努力确保你的代码在编译时==没有任何警告信息==出现


#### 控制标准版本的 编译选项
命令行上传递给gcc
- ansi
  最重要的标准选项，告诉编译器遵守C语言的 ISO C90 标准。
  它关闭那些与标准不兼容的gcc扩展，禁用C语言程序中的C＋＋（//）风格注释，并启用ANSI的三字母词（trigraph）特性。
  同时通过定义宏_STRICT_ANSI_来关闭在头文件中与标准不兼容的一些gcc扩展。
  未来的编译器版本可能会修改这个选项指向的C语言标准。
- std=
  - c89
  - iso9899:1999
  - gun89

。。禁用C++的注释风格 // 。。我。。


#### 控制标准版本的 常量
这些常量
- 可以通过编译器的命令行选项来设置，
- 或者通过源代码中的#define语句来定义。
我们通常建议用前者设置这些常量。

- _STRICT_ANTI_：强制使用C语言的ISO标准。这个常量在使用编译器的命令行选项-ansi时被定义。
- _P0SIX_C_S0URCE＝2：启用由IEEE Std 1003.1和1003.2标准定义的特性。我们还会在本章后面的内容中谈到这些标准。
- _BSD_SOURCE：启用BSD类型的特性。如果这些特性与POSIX定义冲突，则以BSD的定义为准。
- _GNU_SOURCE：启用大量特性，其中包括GNU扩展。如果这些特性与POSIX定义冲突，则以POSIX定义为准。


#### 编译器的警告选项
在编译器的命令行上传递

- pedantic
  这是用于检查C语言代码的功能==最强大==的编译器选项。
  它除了启用用于检查代码是否遵守C语言标准的选项外，还关闭了一些不被标准允许的传统C语言结构，并且禁用所有的GNU扩展。
  如果你希望代码能够尽可能地==做到可移植==，就需要使用这个选项。
  这个选项的缺点是，它在检查代码时显得非常挑剔，有时你不得不非常仔细地思考，以去除那些最后的警告信息。
- Wformat
  检查 `printf` 系列函数所使用的参数类型是否正确。
- Wparentheses
  检查是否总是提供了需要的圆括号，即使在某些环境中并不是必须要使用它们。
  当想要检查对一个==复杂结构的初始化是否按照预期进行==时，这个选项就很有用。
- Wswitch-default
  default：检查是否所有的switch语句都包含一个defaultcase，这通常是一个好的编码习惯。
- Wunused
  检查诸如声明静态函数但没有定义、未使用的参数和丢弃返回结果等情况
- Wall
  启用绝大多数gcc的警告选项，包括所有以-W为前缀的选项（不包括选项-pedantic），这个选项对保持代码的整洁很有用。

一般来说，我们建议使用选项 -Wall



## 18.2 接口和LSB
现在我们将讨论比C语言代码高一个层次的由==操作系统提供的接口==（系统函数）。
这一级别的标准化工作由下面两个组件构成：
- 由函数库提供的函数
- 由底层的操作系统提供的系统调用。

在这两个组件之中又分别包含两个层次的细节：提供的是哪一个接口和接口的定义。

https://wiki.linuxfoundation.org/lsb/start


Linux标准化规范（版本3.1）定义了3个需要遵守的领域。

- 核心：主要的函数库、工具和一些重要的文件系统位置。
- C＋＋：C＋＋函数库。
- 桌面：用于桌面安装的其他文件，主要是各种图形库。

我们感兴趣的主要领域是这个规范的核心部分。


LSB规范在其自身的文档中涵盖了许多领域，同时它还引用了一些针对特定接口定义的外部标准。
其涵盖的领域包括：
- 可兼容二进制程序的对象格式；
- 动态链接标准；
- 标准函数库，包括基础函数库和X视窗系统函数库；
- shell和其他命令行程序；
- 执行环境，包括用户和组；
- 系统初始化和运行级别。

我们只对标准函数库、用户和系统初始化感兴趣


### LSB标准函数库

LSB定义了必须以两种方式呈现的接口。
- 对于那些主要是由GNU C函数库实现的或试图成为Linux专属标准的函数，它定义接口及其行为。
- 对于主要来自UNIX系统的其他接口，规范只是说明必须存在一个特定接口，并且该接口的行为必须与其他标准定义的一样，这里所说的其他标准通常指的是公共应用环境（Common Application Environment，CAE）或更常见的单一UNIX规范（Single UNIX Specification）。

后者的网址为http://www.opengroup.org，其中一部分内容可以通过http://www.unix.org/online.html（需要注册）访问。


#### 针对函数库使用LSB标准

可移植的程序来说意味着什么？

需要检查你所使用的库函数是否被列在了LSB规范中。如果它不在这个规范中，你所编写的程序就可能不会那么容易地被移植，这时就需要查找一种标准的方法来执行你想要完成的工作。
你可能需要用Linux命令apropos来搜索在线手册页，以找到合适的帮助页面。

其次，也是更困难的一步，就是检查你所使用的函数行为是否是规范定义的行为，并且没有在程序中依赖系统特定的函数行为。
如果函数的用法未在LSB中定义，你可能不得不参考单一UNIX规范来检查。

用于检查未定义或可能产生错误行为的函数的
一个非常好的方法是使用Linux的在线手册。手册
中的许多页面都包含一个BUGS（漏洞）小节，它
是一个无价的信息来源。它可以告诉我们，Linux
中的某个特定函数调用可能没有完全按照标准中的
定义来实现，或者它在执行时有一些已知的漏洞或
奇怪的行为。



### 18.2.2 LSB用户和组

规范中这一部分的内容非常简明且容易理解。
下面是一些规范中的定义。
- 它告诉我们，一定==不能直接读取==如/etc/passwd这样的文件，而是应该总是使用如getpwent这样的标准库函数调用或者如passwd这样的标准工具来访问用户详细信息。
- 它告诉我们，在==root组中必须有一个名为root的用户==，这个root用户是一个拥有全部权限的管理员。同时还有一组可选的用户和组也绝对不能在标准应用程序中使用，它们由Linux发行版自身来使用。
- 它还告诉我们，用户==ID小于100==的账号是系统账号，用户ID在==100到499==之间的账号是由系统管理员和安装后脚本分配的，用户ID在500及其以上的账号用于普通用户。

一般来说，上面这些内容对大多数需要了解用户标准的Linux程序员来说已足够。


### 18.2.3 LSB系统初始化

Linux继承了类UNIX操作系统运行级别的思想，运行级别定义了在不同级别中允许启动的服务。
对于Linux来说，常见的运行级别定义见表

|运行级别|说明|
|--|--|
|0|停止。作为系统关闭时切换到的逻辑状态|
|1|单用户模式。非根目录的 目录不会被加载，网络功能被禁用，通常用于系统维护|
|2|多用户模式。但没有启用网络功能|
|3|正常的带网络功能的多用户模式，使用文本模式的登录界面|
|4|保留|
|5|正常的带网络功能的多用户模式，使用图形登录界面|
|6|用于重启系统的 伪运行级别|

LSB列出了这些运行级别，但并不要求使用它们，但实际上它们是非常常见的。

与这些运行级别相伴的是一组用于启动、关闭和重启服务的初始化脚本。
以前的Linux系统会将这些脚本放在/etc目录下的不同位置，一般是放在目录/etc/init.d或/etc/rc.d/init.d下。
这种不确定性通常让用户困惑，因为当他们更换了Linux发行
版后，他们就不能在期望的目录下找到初始化脚本
了，而且在安装程序时，当你试图将初始化脚本放
在一个错误的目录下时，也会导致安装程序失败。

LSB 3.1将这些==初始化脚本==放置的目录定义
为==/etc/init.d==，但它也允许这个目录可以是对其他
目录的一个连接。

在/etc/init.d目录中的每个脚本都有一个与其提
供的服务相关联的名字。由于这是在Linux系统中
所有服务必须共享的一个公用命名空间，所以保证
名字的唯一性是非常重要的
为了避免发生这样的冲突，我们还有另外一组标准，它就
是“Linux分配名字和数字机构”（Linux Assigned Names And Numbers Authority，简称为LANANA），它的网址为 http://www.lanana.org/


初始化脚本必须用一个参数来控制它的行为。已定义的参数见表

|start|说明|
|--|--|
|start|启动(或重启)服务|
|stop|停止服务|
|restart|重启服务，一般是通过先停止服务 再启动服务的方式 实现|
|reload|重载服务，在不停止服务的情况下 重新装载 所有的参数。 不是所有的服务都支持，不是所有脚本支持这个参数，或者 虽然脚本接受，但是没有任何效果。|
|force-reload|如果服务支持这个选项，就 重载服务，否则，就 重启服务|
|status|以文本方式打印服务的状态信息，并返回一个可以用来确定服务状态的状态码|

所有的命令在成功时返回0，失败时返回表明错误原因的错误代码。
使用status参数时，如果服务正在运行则返回0，否则返回表明服务没有运行原因的状态码。



## 18.3 文件系统层次结构标准

我们要介绍的最后一个标准是文件系统层次结构标准（Filesystem Hierarchy Standard，简称为FHS），它的网址为 http://www.pathname.com/fhs/

这个标准的目的是定义Linux文件系统的标准路径，使得开发者和用户可以在合理的位置找到需要的东西

Linux的文件布局是经过多年的合理演变才形成我们今天见到的构架的。
大体的想法是将文件和目录分为如下3组。
- 对运行Linux的某一特定系统唯一的文件和目录，例如启动脚本和配置文件。
- 可以在运行Linux的不同系统之间共享的只读文件和目录，例如可执行应用程序。
- 可以在运行Linux或其他操作系统的不同系统之间共享的可读可写的目录，例如用户家目录。


在一个由Linux机器组成的网络中，确保
只有一份主要程序目录的副本，并且可以在许多机
器之间共享是非常好的做法，但在本书中，我们对
在不同版本的Linux系统之间共享文件并不是太感
兴趣。这种做法只对无盘工作站特别有用。

FHS定义的顶级结构包含一些必须存在的子目录和一小部分可选的目录
|目录|是否需要|用途|
|--|--|--|
|/bin|是|重要的系统二进制文件|
|/boot|是|启动系统所需要的文件|
|/dev|是|设备文件|
|/etc|是|系统配置文件|
|/home|否|用于放置用户文件的目录|
|/lib|是|标准函数库|
|/media|是|用于装载可移动媒体的位置，它针对每个系统支持的媒体类型都有一个单独的子目录|
|/mnt|是|方便临时装载 如CD_ROM 和闪存 等设备的目录|
|/opt|是|其他应用程序软件|
|/root|否|root用户的文件|
|/sbin|是|在系统启动时 需要的重要的 系统二进制文件|
|/src|是|用于系统提供的服务的只读数据|
|/tmp|是|临时文件|
|/usr|是|第二级的目录层次，传统上用户的文件也可以放到这个目录下，但现在认为这是一种不好的做法，所以普通用户没有对 /usr 目录的写权限|
|/var|是|可变的数据，如日志文件|

可能还会有一些其他目录也以 lib为前缀，但这并不是很常见

你通常还会看到目录/lost+found（用于fsck命令进行文件系统的恢复）和
目录/proc，后者其实是一个伪文件系统，它提供了对当前运行系统的一个映射
当前的FHS标准提到了/proc文件系统，但并不要求它一定存在

- /bin：
  包含可以被root用户和普通用户使用的==二进制文件==，它们都可以在单用户模式下运行，即在其他一些目录结构还未装载的情况下也能单独运行。
  例如，核心命令如cat和ls都可以在这里找到，当然也包括命令sh。
- /boot：
  这个目录下放置的是==启动Linux系统==时所需使用的文件。
  这些文件通常都==比较小==，文件长度不超过100MB。
  我们通常会为这个目录单独划分一个分区，在基于PC的系统中这样做非常方便，但由于BIOS通常会对活动分区有所限制，所以需要将该分区分配在磁盘的前2G或4G空间中。
  为这个目录单独划分一个分区，可以使我们在决定如何分配剩余的磁盘空间时更灵活。
- /dev：
  这个目录下放置的是映射到硬件的特殊设备文件。
  例如/dev/hda将映射到第一个IDE磁盘。
- /etc：
  这个目录下放置的是配置文件。
  历史上有些二进制文件也放置在这个目录下，但在现在的大多数Linux系统中都不会再出现这种情况。
  在/etc目录下最有名的文件可能就是passwd文件，它包含系统中用户的信息。
  其他有用的文件有fstab（列出分区装载选项）、hosts（列出IP地址和主机名的映射关系）、httpd目录（包含Apache服务器的配置文件）。
- /home：
  这是用于放置用户文件的目录。
  正常情况下，每个用户都会在这个目录下有一个与他们的登录名相同的子目录，而这个子目录就是他们的默认登录目录。
  例如，用户rick在登录进系统后，将会发现自己位于目录/home/rick中。
- /lib：
  这个目录下放置的是==基本的共享函数库和内核模块==，特别是那些在系统启动或系统位于单用户模式时需要用到的文件。
- /media：
  这个顶级目录用于包含装载可移动媒体的其他子目录。
  其目的是消除不必要的顶级目录，如/cdrom和/floppy。
- /mnt：
  这个目录只是用来方便用户临时装载一些其他的文件系统。
  以前，一些Linux发行版还会在该目录中针对不同的设备添加一些子目录，如/mnt目录下的cdrom和floppy子目录，但现在用于装载这些设备的首选位置是在/media目录下，/mnt目录将作为一个顶级的临时装载位置。
- /opt：
  软件厂商在向系统中添加软件时会用到这个目录。
  按照惯例，Linux发行版一般不会将自己发布的软件放置在这个目录下，而是将这个目录开放给==第三方厂商来使用==。
  厂商通常会在这个目录下以它们的名字创建一个子目录，然后针对它们的应用程序，在这个子目录下==继续创建如/bin和/lib等子目录==。
  按照惯例，大多数==开放源码==的Linux软件包将==目录/usr/local==作为它们的安装点。
- /root：
  这个目录下放置的是root用户使用的文件。
  它并没有放置在/home目录下的原因是，在单用户模式下，/home目录可能未被装载进系统。
- /sbin：
  这个目录下放置的是通常只能由==系统管理员使用的命令==，以及在==系统启动==时或进入==单用户模式==时需要使用的命令。
  命令fsck、halt和swapon等就在这个目录中。
- /srv：
  这个目录用于放置站点特定的只读配置数据，但它目前还未被普遍使用。
- /tmp：
  这个目录下放置的是临时文件。
  系统通常会在（但并不总是）启动时清理这个目录。
- /usr：
  这是一个相当复杂的二级文件系统，在这个目录下，通常将包含==除==在系统启动或进入单用户模式所需要的文件以外的所有系统类的命令和函数库。
  它包含许多子目录，如/bin、/lib、/X11R6和/local
- /var：
  这个目录下放置的数据是会经常改变的，如用于打印的队列文件、应用程序的日志文件和邮件队列目录等。



# 18.4 更多标准 (autoconf, automake)

http://www.openi18n.org/


编写应用程序时，另一个需要考虑的问题是目标系统安装了哪个版本的函数库，它使用的选项是什么，等等。
幸运的是，由于我们在本章中所看到的标准化工作，这个问题已经显得不那么明显，但它仍然是一个比较困难的问题。
有一组GNU工具可以极大地帮助我们解决这一问题，它们是autoconf和automake。
虽然你可能不会直接使用它们，但当通过源代码安装软件，在命令行键入命
令./configure; make时，你几乎肯定会看到它们带来的好处。

你可以通过GNU的网站
http://www.gnu.org/software/autoconf
http://www.gnu.org/software/automake 
找到关于它们的更多信息。



---
---

22/09/2023 17:23

---






























