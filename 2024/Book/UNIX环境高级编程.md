
2024-03-15 15:39

https://baike.baidu.com/item/UNIX%E7%8E%AF%E5%A2%83%E9%AB%98%E7%BA%A7%E7%BC%96%E7%A8%8B%E7%AC%AC3%E7%89%88/56396883?fr=ge_ala


[[toc]]

---
---

# ch1 unix 基础知识

OS是一种软件，控制计算机硬件资源，提供程序运行环境。通常被成为 kernel。
内核的接口被成为 system call。
公用函数库构建在 系统调用之上
应用 既可以使用 公用函数库 也可以使用 系统调用
shell是一个特殊的应用程序，它==为运行其他应用程序 提供了一个接口==。


## 1.3 登录
输入账户，密码后， 系统在 口令文件 (/etc/passwd) 中查看登录名。

用户登陆后，系统会显示一个 shell。 系统从 口令文件的 最后一列 中知道 为该用户 启动哪个shell。
。。应该是指 无图形界面的那种 服务器。  而却 ctrl+alt+t 是不是 也是从 口令文件中 知道 启动哪个shell。

shell分为：sh,bash,csh,ksh,tcsh。 
。fedora默认使用bash。


## ls(1) 的解释

`ls(1)` 是 unix 惯用方法，用以引用 unix 系统手册中的一个 特定项。
ls(1) 引用 第一部分中的 ls 项。
各部分通常用 数字 1-8 编号。

`man 1 ls` or `man -s1 ls`

。。传统unix手册分部如下
- 1, 用户层面命令
- 2, 系统调用
- 3, 库函数
- 4, 设备和设备驱动
- 5, 文件格式
- 6, 游戏
- 7, 各式零碎杂物，如宏包等
- 8, 系统维护命令
。。。


## 1.5 输入和输出

### 文件描述符
通常是一个 小的 非负整数，内核用以标识 一个特定进程 正在访问的文件。
当内核打开 一个现有文件 或 创建一个新文件时， 它都返回一个 文件描述符。


### 标准输入，标准输出，标准错误
按照惯例，每当运行一个新程序时，所有的shell 都会为其打开 3个文件描述符。
分别是 标准输入/输出/错误
不做特殊处理的话，这3个描述符都 指向 终端。
大多数 shell 都提供了 重定向的 方法
`ls > file.list` 将标准输出 重定向到 file.list 文件
`./a.out < infile > outfile` 标准输入 重定向到infile，标准输出 重定向到outfile

### 不带缓冲的IO
open,read,write,lseek,close, 不带缓冲，都使用 文件描述符

### 标准IO
为不带缓冲的IO 提供一个 带缓冲的接口。
使用标准IO函数 无需担心如何选取最佳的缓冲区大小。(。。使用 BUFFSIZE)



## 1.6 程序和进程

程序(program) 是一个 存储在磁盘上的某个目录中的 可执行文件。
内核使用 exec 函数，将 程序读入内存，并执行程序。 8.10将说明exec
exec有7种变体

程序的执行实例 被称为 进程。 unix确保每个 进程都有一个 唯一的数字标识符，称为进程ID，它总是非负数。
`getpid()`

有3个用于 控制进程的主要函数： fork,exec,waitpid

通常，一个进程只有一个控制线程 —— 某一时刻执行的一组机器命令。
一个进程内的所有线程共享同一地址空间，文件描述符，栈，以及和进程相关的属性。 因为它们能访问 同一个存储区，所以 各线程在访问 共享数据的时候 要采用 同步措施。

线程也有ID，但是 线程ID 只在它所属的 进程内起效。 一个进程中的线程ID 在另一个进程中 没有意义。
控制线程的函数在 12章， 和 控制进程的函数类似， 但是是另一套。 线程模型 是在 进程模型 建立很久以后 才引入unix的， 这2种模型之间有复杂的交互

## 1.7 出错处理
unix系统函数出错时，通常会返回一个 负数。
整型变量 errno 通常被 设置为 具有特定信息的值。

有些函数对于出错 则使用另一种约定，例如，大多数 返回 对象指针的函数，在出错时 会返回一个 null 指针

unix的 intro(2) 列出了所有的 出错常量
linux是 errno(3)

以前，errno的定义是 `extern int errno;`
但在支持 线程的 环境种，多个线程 共享 进程地址空间，每个 线程都有 属于它自己的 局部 errno，以避免 线程间 干扰。例如，linux的多线程存储errno：
```C
extern int *__errno_location(void);
#define errno (*__errno_location())
```
。。现在没有这个了，反正 github上 搜不到。

对于errno，有2点需要注意
1. 如果没有出错，其值不会被清空。因此，仅当 函数的返回值 表明出错时，才检验其值
2. 任何函数 都不会将 errno 值设置为 0

`char *strerror(int errnum);`   string.h, 将输入的错误代码转换为 出错消息字符串
`void perror(const char *msg);`  stdio.h，将当前的errno值，在标准错误上产生一条出错信息，然后返回。先输出msg，然后errno的出错消息


errno.h 中的出错 分为2类，
- 致命性的，无法执行恢复动作，最多就是 在用户屏幕上 打印出错消息 或 将出错日志写入日志文件，然后退出。
- 非致命性的，有时可以较妥善地进行处理。大多数 非致命性出错 是暂时的 (如 资源短缺)。典型的恢复操作是 延迟一段时间，然后重试。


## 1.8 用户标识

用户ID
在口令文件中。
用户不能修改其用户ID。
ID为0的用户 为 root用户 或 超级用户。

组ID
组文件 将 组名映射为 组ID，组文件通常是 /etc/group

`getuid, getpid`

附属组ID
除了口令文件中，对 登录名 指定 组ID外，大多数unix版本 还允许 一个用户 属于 另外一些组。
实际上，大多数系统至少支持 16个附属组


## 1.9 信号 signal

用于通知 进程 发生了某种情况。
例如，如果进程 执行了除法操作，除数为0,那么会将 SIGFPE (浮点异常) 信号 发给 该进程。
进程有3种处理方式
- 忽略信号
    有些信号表示 硬件异常，如，除0 或访问 进程地址空间以外的 存储单元，因为这些异常产生的后果不确定，所以不推荐 忽略信号
- 按系统默认方式处理
    对于除数为0,系统默认方式是 终止该进程。
- 提供一个函数，信号发生时调用该函数
    通过提供自编的函数，我们就能知道 什么时候 产生了型号，并按照预期的方式 处理它。



## 1.10 时间值

unix 使用过 2种不同的 时间值
- 日历时间
    自 协调世界时(1970.1.1) 以来的 秒数 (。。不是ms)
    time_t 保存了这种时间值
- 进程时间
    也称为CPU时间，用以度量 进程使用的 CPU资源。进程时间以 时种滴答计算。
    clock_t

度量进程的执行时间时(3.9节)，unix维护了 3个进程时间值
- 时钟时间，又称墙上时钟时间，是进程运行的时间总量。值与系统中同时运行的进程数有关
- 用户CPU时间，执行用户指令所用的时间量。
- 系统CPU时间，为该进程执行内核程序所经历的时间

通过 time(1) 命令就可以获得 进程的 3个时间值
`cd /usr/include`
`time -p grep _POSIX_SOURCE */*.h > /dev/null`


## 1.11 系统调用和库函数

OS提供了 多种服务的入口点，由此程序向内核请求服务。
各个版本的UNIX实现 都提供了 良好定义，数量有限，直接进入内核的 入口点，这些入口点被称为系统调用(system call)
Linux 3.2.0 提供了 380个 系统调用。
系统调用接口 在 《Unix程序员手册》的第二部分 说明，是用C语言定义的。 早期的OS按照传统方式 用机器的汇编语言 定义 内核入口点
==Unix使用的技术是 为每个系统调用 在标准C库 中设置一个 同名的函数。用户进程 用 标准C调用序列 来调用这些函数，然后 函数又用系统所要求的技术 调用 相应的内核服务。== 例如 ==函数可将一个或多个 C参数 送入通用寄存器，然后执行某个产生软中断进入内核的机器指令==。
从应用角度，可以将 系统调用 视为 C函数。

《UNIX程序员手册》 第三部分定义了 程序员可以使用的 通用库函数。虽然这些函数可能会调用1个或多个内核的 系统调用，但是它们(。应该指通用库函数)并不是内核的入口点。例如，printf函数调用 write 系统调用 以输出一个字符串，但是 strcpy，ato 并不使用任何内核的系统调用。

以==存储空间分配函数 malloc 为例==，有多种方法可以进行存储空间分配 及 相关的无用空间回收操作 (最佳适应，首次适应等)，并不存在 对所有程序 都最优的一种技术。 
==unix 系统调用中 处理存储空间分配的是 sbrk(2)==。 它不是一个 通用的存储器管理器。==它按 指定字节数 增加或减少 进程地址空间==。
==如何管理该空间 却取决于 进程==。
==存储空间分配函数 malloc(3) 实现一种特定类型的分配==。 
如果我们不喜欢其操作方式，则可以定义自己的 malloc函数，它很可能将使用 sbrk 系统调用。 
实际上，很多软件包，它们使用 sbrk 系统调用 实现自己的 存储空间分配算法。

图1-11 显示了 应用程序，malloc 函数，sbrk系统调用 的关系。 可以看到，2者的职责不同，内核中的系统调用 分配一块空间 给进程，而 库函数malloc则在 用户层次 管理这一空间。

另一个可以说明 系统调用 和 库函数 之间的 差别的例子是：unix系统提供的 判断当前时间 和日期的接口。unix系统只提供一个 系统调用，返回1970.1.1以来经过的秒数，对这个值的 解释 留给用户进程处理。

图1-12，应用程序即可以调用 系统调用 也可以调用 库函数。很多库函数 会调用 系统调用。

![365e7324fadcc07a377323224356f124.png](../_resources/365e7324fadcc07a377323224356f124.png)

系统调用 和 库函数的 另一个差别是： 系统调用 通常提供 一种最小接口，而库函数通常提供 比较复杂的功能。

进程控制 系统调用 (fork, exec, wait) 通常由 用户应用程序 直接调用，但是为了简化某些常见的情况，unix 也提供了一些库函数，比如 system, popen。



# ch2 UNIX标准及实现

按标准，将 ISO C库分为24个区。

- assert.h
- complex.h
- ctype.h  字符分配和映射支持
- errno.h
- fenv.h  浮点环境
- float.h  浮点常量及特性
- inttypes.h  整型格式变换
- iso646.h  赋值，关系，一元操作符 宏
- limits.h  实现常量, 这里包含了UNIX，POSIX的 数值的 最大最小值
- locale.h  本地化类型及相关定义
- math.h  数学函数，类型声明及常量
- setjmp.h  非局部goto
- signal.h  信号(第10章)
- stdarg.h  可变长参数表
- stdbool.h  布尔类型和值
- stddef.h  标准定义
- stdint.h  整型
- stdio.h  标准IO库
- stdlib.h  实用函数
- string.h  字符串操作
- tgmath.h  通用类型数学宏
- time.h  时间和日期
- wchar.h  扩充的多字节和宽字符支持
- wctype.h  宽字符分类和 映射支持

## POSIX.1
posix.1 在 ISO C 库的基础上 增加了 一些头文件
。。我都没有写 .h 后缀 . p23 开始
必选：aio, cpio, dirent, dlfcn, fcntl, fnmatch, glob, grp, iconv, langinfo, monetary, netdb, nl_types, poll, pthread, pwd, regex, sched, semaphore, strings, tar, termios, unistd, wordexp, 
arpa/inet, net/if, netinet/in, netinet/tcp,
sys/mman, sys/select, sys/socket, sys/stat, sys/statvfs, sys/times, sys/types, sys/un, sys/utsname, sys/wait

可选：fmtmsg, ftw, libgen, ndbm, search, utmpx,
sys/ipc, sys/msg, sys/resource, sys/sem, sys/time, sys/uio.
mqueue, spawn


## 限制
- 编译时限制，如，短整型的最大值是什么？
- 运行时限制，如，文件名有多少个字符？

编译时限制可以在头文件中定义。
运行时限制则要求进程调用一个函数获得限制值。

3种限制
- 编译时限制(头文件)
- 与文件或目录无关的运行时限制(sysconf)
- 与文件或目录有关的运行时限制(pathconf,fpathconf)


### ISO C限制(limits.h)

下图第四列是32位OS的值。
在64位OS中，long的最大值 和 long long的最大值一样。

![6515bce3ee7ef129ad5a18316458145e.png](../_resources/6515bce3ee7ef129ad5a18316458145e.png)


我们还会遇到 另一个ISO C常量，FOPEN_MAX，代表了 可以同时打开的标准IO流的最小个数，这个值在 stdio.h 中定义，最小值是8
ISO C 在 stdio.h 中定义了常量 TMP_MAX，这是由 tmpnam函数产生的唯一文件名的最大个数。
ISO C 定义了 FILENAME_MAX，但我们应该避免使用它，因为 POSIX.1 提供了更好的替代值 (NAME_MAX, PATH_MAX)

### 2.5.2 POSIX限制
POSIX.1定义了很多涉及OS实现限制的常量，这是POSIX.1 中最令人不解的部分之一。
我们只关心与基本POSIX.1 接口有关的部分，这些限制和常量分为7类
- 数值限制，LONG_BIT，SSIXE_MAX，WORD_BIT
- 最小值：
- 最大值：_POSIX_CLOCKERS_MIN
- 运行时可以增加的值：CHARCLASS_NAME_MAX,COLL_WEIGHTS_MAX,LING_MAX,NGROUPS_MAX,RE_DUP_MAX
- 运行时不变值：
- 其他不变值：
- 路径名可变值：

。。太多了

## 2.5.4 sysconf, pathconf, fpathconf
有些限制值是在编译时可用的，另一些必须是运行时确定。
有些限制值是可能更改的，它们和文件，目录有关。
运行时的限制可以调用下面的3个函数之一来获得
```C
#include <unistd.h>
long sysconf(int name);
long pathconf(const char* pathname, int name);
log fpathconf(int fd, int name);
```

。。name参数是有 常量的。


---

### 2.5.5 不确定的运行时限制

2. 最大打开文件数

守护进程 (在后台运行且不与终端相连的一种进程)中 常见的 功能是 关闭所有打开的文件。
某些程序有以下形式的代码，这段程序假定在 sys/param.h 中定义了常量 NOFILE
```C
#include <sys/param.h>
for (i = 0; i < NOFILE; ++i)
    close(i);
```

另一些使用 某些stdio.h 版本提供的  _NFILE。 有些程序直接硬编码20

我们希望用POSIX.1的OPEN_MAX确定此值以提高可移植性，但是如果这个值是不确定的，那么仍然有问题
```C
#include <unistd.h>
for (i = 0; i < sysconf(_SC_OPEN_MAX); ++i)
    close(i);
```
如果OPEN_MAX是不确定的，那么sysconf 会返回-1。



## 2.6 选项

- 编译时选项定义在 unistd.h 中
- 与文件或目录无关的运行时选项用 sysconf 判断
- 与文件或目录有关的运行时选项通过 pathconf, fpathconf 来判断

。。很多name可选值


## 2.8 基本系统数据类型
本书会用到的数据类型
clock_t
comp_t
dev_t
fd_set
fpos_t
gid_t
ino_t
mode_t
nlink_t
off_t
pid_t
pthread_t
rlim_t
sig_atomic_t
sigset_t
size_t
ssize_t
time_t
uid_t
wchar_t




# ch3 文件IO

unix系统中大多数 文件IO 只需要 5个函数：open, read, write, lseek, close

==不带缓冲 是指 每个read,write 都调用内核中的一个系统调用。==
这些不带缓冲的IO不是 ISO C的组成部分，它们是 POSIX.1，Single UNIX Specification的组成部分。

只要涉及在多个进程间共享资源，原子操作的概念就很重要。会讨论 ==多进程如何共享文件==

dup,fcntl,sync,fsync,ioctl


## 3.2 文件描述符

打开现有文件，或创建新文件，内核会返回 文件描述符 给 进程。
当读写一个文件时，使用 open或create 返回的 文件描述符 标识该文件，将其作为参数 传递给 read或write

惯例，0是标准输入，1标准输出，2标准错误
unistd.h 的 STDIN_FILENO，STDOUT_FILENO，STDERR_FILENO

文件描述符的变化范围是 0 - OPEN_MAX-1。
早期UNIX使用上限是19(最多20个文件)，现在很多是63


## 打开或创建文件 open,openat

```C
#include <fcntl.h>

int open(const char *path, int oflag, ... /* mode_t mode */);
int openat(int fd, const char *path, int oflag, ... /* mode_t mode */);
```

path是要打开或创建文件的名字。
oflag用来说明函数的选项，用下列的一个或多个常量 进行 或运算 构成 oflag参数 (这些常量在 fcntl.h中)
- O_RDONLY
- O_WRONLY
- O_RDWR
- O_EXEC，只执行
- O_SEARCH，只搜索，应用于目录
- O_APPEND，追加
- O_CLOEXEC，把FD_CLOEXEC常量设置为文件描述符标志。3.14
- O_CREAT，文件不存在就创建，使用这个选项时，open函数需要同时说明参数 mode
- O_DIRECTORY，如果path指向的不是目录，就报错
- O_EXCL，如果同时指定了 O_CREAT，而且文件已存在，就报错。
- O_NOCTTY，如果path引用的是 终端设备，则不将该设备分配作为此进程的控制终端。
- O_NOFOLLOW，如果path引用的是一个符号连接，则出错。
- O_NONBLOCK，如果path有用的是一个FIFO，一个块特殊文件 或一个字符特殊文件，则此选项为文件的==本次打开操作和后续的IO操作==设置==非阻塞方式==。
- O_SYNC，使每次write等待物理IO操作完成，包括由该write操作引起的文件属性更新的IO
- O_TRUNC，如果文件存在，且只写 或 读写 打开，则长度截断为0。
- O_TTY_INIT，如果打开一个还未打开的终端设备，设置非标准 termios 参数值。
- O_DSYNC，每次write要等待物理IO完成，但如果该写操作不影响读取刚写入的操作，则不需要等待文件属性被更新。
- O_RSYNC，使每个以文件描述符作为参数进行的 read操作等待，直至所有对文件同一部分挂起的写操作完成。

fd参数区分了 open,openat, 有3种可能
1. path参数指定的是绝对路径名，此时，fd被忽略。openat 就相当于 open
2. path是相对路径名，fd指出了相对路径名 在文件系统中的开始地址。fd参数是通过打开相对路径名所在的目录来获取的。
3. path是相对路径，fd具有特殊值 AT_FECWD，此时，路径名在当前工作目录中获取。openat 和open相似。


## 3.4 函数 creat
fcntl.h
```C
int creat(const char* path, mode_t mode);
```
这个函数等价于 `open(path, O_WRONLY | O_CREAT | O_TRUNC, mode);`

creat不足之处是，它只能以写方式打开所创建的文件。 在提供open的新版本之前，如果要创建临时文件，并要写该文件，然后读该文件，则必须先调用 creat, close，然后调用 open，现在可以 一次性： `open(path, O_RDWR | O_CREAT | O_TRUNC, mode);`

## close
关闭一个打开的文件
unistd.h
`int close(int fd);`
关闭文件会释放该进程在该文件上的所有记录锁
进程终止时，内核会自动关闭它所有打开的文件。

## lseek
每个打开的文件 都有一个 当前文件偏移量。通常是一个 非负整数，用以度量 从文件开始处计算的字节数。

系统默认情况下，打开一个文件时，除非指定 O_APPEND，否则 该偏移量为0。

unistd.h
`off_t lseek(int fd, off_t offset, int whence);`
成功返回新偏移量，失败-1

- whence是SEEK_SET，则将文件的偏移量设置为距文件开始处 offset 个字节
- whence是SEEK_CUR，则将该文件的偏移量设置为其当前值加 offset，offset可正可负。
- whence是SEEK_END，则将该文件的偏移量设置为文件长度加offset，offset可正可负

用下列方式确定打开文件的 当前偏移量
off_t currpos;
currpos = lseek(fd, 0, seek_cur);

这种方法也可以用来确定所涉及的文件 是否可以设置偏移量。如果fd指向的是一个管道，FIFO，或网络套接字，则lseek返回-1， 并将errno 设置为 ESPIPE。


## 3.7 read
从打开的文件中读取数据
unsitd.h
`ssize_t read(int fd, void *buf, size_t nbytes);`

read成功，返回读取的字节数。如果已经到尾端，则返回0。

。。带有 ==buf==

## 3.8 write
unistd.h
`size_t write(int fd, const void *buf, size_t nbytes);`

write出错的一个常见原因是 磁盘已满。 另一个是 给定进程的文件长度限制

## 3.9 IO效率

使用 read，write 复制文件

```C
#include "apue.h"

#define BUFFSIZE 4096

int main(void)
{
    int n;
    char buf[BUFFSIZE];
    while ((n = read(STDIN_FILENO, buf, BUFFSIZE)) > 0)
        if (write(STDOUT_FILENO, buf, n) != n)
            err_sys("write error");
    
    if (n < 0)
        err_sys("read error");
    
    exit(0);
}
```

- 从标准输入读，从标准输出写，这些输入，输出需要由shell安排好。一般使用shell的IO重定向功能
- 进程终止，内核会关闭所有打开的文件描述符，所以程序中没有显式关闭
- 对于 unix 内核而言， 文本文件 和 二进制文件 并无区别，所以 本程序对 这2种文件都有效。

通过测试数据，可以看到 BUFFSIZE 为4096 及之后，耗时差不多，继续增加缓冲区长度 对时间基本没有影响。 测试用的系统的磁盘块长度是4096 (由 st_blksize 表示)

大多数文件系统 为改善性能都 采用 某种 ==预读(read ahead)技术。 当检测到正在进行 顺序读取时，系统就试图读入 比应用所要求的 更多数据，并假想应用很快会用到这些数据。==


## 3.10 文件共享

unix 支持 在不同进程间 共享打开文件。 在介绍 dup函数之前，先要说明这种共享。为此先介绍内核用于所有IO的数据结构。

下面的说明是概念性的，与特定实现可能不匹配。

内核使用 3种数据结构表示打开文件，它们之间的关系决定了 在文件共享方面，一个进程 对另一个进程 可能产生的影响

1. 每个进程在进程表中都有一个记录项，它包含了 一张打开文件描述符 表，可以将其视为一个vector，每个描述符占据一项，与每个文件描述符相关联的是
   1. ==文件描述符标志== (close_on_exec，3.14节)
   2. 指向一个文件表项的指针
2. 内核为所有打开文件维持一张 文件表。每个文件表项包含
   1. ==文件状态标志== (读，写，追加，同步，非阻塞等)
   2. 当前文件偏移量
   3. 指向该文件v节点表项的指针。
3. 每个打开文件(或设置) 都有一个 v节点 (v-node) 结构。v节点包含了文件类型 和 对此文件进行各种操作函数的指针。对于 大多数文件，v节点还包含了 该文件的 i节点 (i-node，索引节点)。这些信息是在 打开文件时 从磁盘上读取到内存的，所以，文件的所有相关信息都是随时可用的。例如，i节点包含了 文件的所有者，文件长度，指向文件实际数据块在磁盘上所在位置的指针等。

文件描述符标志是 一个进程的 描述符， 文件状态标志 是指向 该文件表项的 所有进程中的 描述符

linux没有使用 v节点，而是使用了 通用i节点结构，虽然实现上有不同，但是概念上，v节点与i节点是一样的。两者都指向文件系统特有的i节点结构。

![9c8498c09121e1fa00ee3f8ab7f767eb.png](../_resources/9c8498c09121e1fa00ee3f8ab7f767eb.png)

2个独立进程各自打开 同一个文件：

![7983c6464e76c6ea35f1e50249718b4b.png](../_resources/7983c6464e76c6ea35f1e50249718b4b.png)

假定，第一个进程在 fd3上 打开该文件，另一个进程在 fd4上打开。
每个进程 有各自的 文件表项，但是对于一个给定的文件，只有一个 v节点表项。
之所以 每个进程 有自己的 文件表项，是因为 ==每个进程 都有 它自己对于 该文件的 当前偏移量==。

### ==============
p61

给出这些数据结构后，会前面所述的操作进一步说明
- 在完成每个write后，在文件表项中的当前文件偏移量即 增加所写入的字节数。如果这导致当前文件偏移量超出当前文件长度，则i节点表项中的 当前文件长度设置为 当前文件偏移量。
- 如果用 O_APPEND 打开文件，则相应标志 也被设置到 文件表项 的文件状态标志中。每次对这种具有 追加写 标志的文件 执行写操作时，文件表项中的 当前文件偏移量 首先会被设置为 i 节点表项 中的 文件长度。这就使得每次写入的数据都是 追加到 文件的当前尾端处
- 如果一个文件使用 lseek 定位到 当前文件的尾端，则文件表项 中的当前文件偏移量被设置为 i节点表象中的 当前文件长度 (这和 O_APPEND 不同)
- lseek函数只修改文件表项中的 当前文件偏移量，不执行任何IO

可能有多个 fd 指向同一个 文件表项，3.12中dup会看到。 在fork后会同样的情况，父子进程 各自的打开的文件描述符 共享同一个 文件表项。



## 3.11 原子操作

早期，不支持open的 O_APPEND选项，所以 追加被写成
```C
if (lseek(fd, 0L, 2) < 0)
    err_sys("lseek error");
if (write(fd, buf, 100) != 100)
    err_sys("write error");
```

多进程会出问题，因为lseek 和 write 不是原子的。 会覆盖其他进程内容。

open时 O_APPEND是原子的。

---
pread,pwrite
unistd.h
`ssize_t pread(int fd, void *buf, size_t nbytes, off_t offset);`
`ssize_t pwrite(int fd, const void *buf, size_t nbytes, off_t offset);`
``

pread 相当于 lseek 后 read，但
- 调用pread时，无法中断 其定位 和 读操作
- 不更新当前文件偏移量
pwrite相当于 lseek后 write，但也有如上的不同。

---
创建文件

对open 的 O_CREAT 和 O_EXCL 进行说明的时候，就已看到了 原子操作的例子。当同时指定这2个选项，而文件已存在时，open将失败。 一步做到：是否已存在+创建文件。如果没有 open 的话， 是否已存在 + 创建文件 将分为2步， 多进程也会出错。


## 3.12 dup,dup2
复制现有文件描述符
unistd.h
`int dup(int fd);`
`int dup2(int fd, int fd2);`

dup返回的一定是 当前可用文件描述符中 最小数值
dup2可以用 fd2 指定， 如果fd2已打开，就先关闭。 如果 fd==fd2，则dup2 返回 fd2，而不关闭它。 否则， fd2的 FD_CLOEXEC 文件描述符标志 被清空，这样 fd2 在进程调用 exec 时是打开的。

`newfd = dup(1);` 执行效果：

![276f7e06d6c4beb425bfed5b06c710d6.png](../_resources/276f7e06d6c4beb425bfed5b06c710d6.png)

新描述符的 执行时关闭(close on exec) 标志 总是有 dup函数清除。

调用 `dup(fd);` 等价于 `fcntl(fd, F_DUPFD, 0);`
`dup2(fd, fd2)` ~= `close(fd2); fcntl(fd, F_DUPFD, fd2);`
第二个并不完全相等，区别：
- dup2 是原子操作， close，fcntl是2个操作
- dup2 和 fcntl 有一些不同的 errno


## 3.13 sync,fsync,fdatasync

传统unix，在内核中设有 ==缓冲区高速缓存== 和 ==页高速缓存==， 大多数磁盘IO 都通过 缓冲区进行。
当我们==向文件写入数据时，内核通常先将数据复制到 缓冲区中，然后  排 入 **队 列**，晚些时候再写入磁盘，这就是 延迟写(delayed write)==

当内核需要 重用缓冲区 来存放 其他磁盘快数据时，会把所有 延迟写数据块 写入磁盘。
为了保证 磁盘数据 和 缓冲区内容一致， 提供了 sync，fsync，fdatasync
```C
unistd.h

int fsync(int fd);
int fdatasync(int fd);

void sync(void);
```
sync 只是将所有修改过的块缓冲区排入==写队列==，然后就返回，不等待 实际写磁盘操作结束。

通常，名为 update 的系统守护进程 周期性地(一般30s)调用 sync 函数。

fsync 等待 写磁盘操作结束才返回。 可以用于 数据库
fdatasync 类似 fsync， 但是 它只影响 文件的 数据部分。   fsync 还会更新 文件的属性。

## 3.14 fcntl 改变已打开文件的属性
改变已打开文件的属性
fcntl.h
`int fcntl(int fd, int cmd, ... /* int arg */);`

本书中，第3个参数总是一个整数，与上面的 arg 对应。

fcntl函数有5个功能
- 复制一个已有的描述符，cmd=F_DUPDF或F_DUPFD_CLOEXEC
- 获取/设置文件描述符标志，cmd=F_GETFD或F_SETFD
- 获取/设置文件状态标志，cmd=F_GETFL或F_SETFL
- 获取/设置异步IO所有权，cmd=F_GETOWN或F_SETOWN
- 获取/设置记录锁，cmd=F_GETLK或F_SETLK或F_SETLKW

先说明这11种CMD的前8种，14.3说明后3种，这3种和记录锁有关

p66

F_DUPFD，复制fd，返回 大于等于第三个参数的 未打开的fd中最小的。
F_DUPDF_CLOEXEC
F_GETFD
F_SETFD
F_GETFL
F_SETFL
F_GETOWN
F_SETOWN


## ioctl
不能用本章其他函数表示的IO操作通常都能用 ioctl表示。
终端IO是使用 ioctl 最多的地方
System V 使用 unistd.h
BSD,Linux 使用 sys/ioctl.h
`int ioctl(int fd, int request, ...);`

## 3.16 /dev/fd
较新的系统都提供了 /dev/fd 目录，目录项是 0,1,2 等的文件。
打开文件 /dev/fd/n 等效于 复制描述符 n (假定描述符n 是打开的)

`fd = open("/dev/fd/0", mode);` 中，大多数系统忽略 mode，另外一些系统则要求 mode 必须是 所引用的文件 (在这里是 标准输入) 初始打开时 所使用的 打开模式 的一个子集，因为上面的打开等效于
`fd = dup(0);`
所以 描述符0 和 fd 共享同一个 文件表项 (。。第二层)。 例如，如果描述符0 被只读打开，那么我们也只能对 fd 只读操作。 即使系统忽略打开模式，而且下列调用是成功的 `fd = open("dev/fd/0", O_RDWR);` 我们仍然不能 对fd 进行写操作。

也可以使用 /dev/fd 作为路径名参数 调用 creat，这与 open + O_CREAT 相同。
注意，在linux上这么做必须消息，因为 linux 实现使用 指向实际文件的 符号链接，在 /dev/fd 上使用 creat 会导致 底层 文件被截断。



# ch4 文件和目录

## stat,fstat,fstatat,lstat

### restrict C99
告诉编译器，这个指针生命周期中，没有其他指针会指向同一块区域。(到底是不是，没有人知道，只是让编译器这样认为)
C++标准不支持，各个编译器提供自己的实现。

sys/stat.h
```C
int stat(const char *restrict pathname, struct stat *restrict buf);
int fstat(int fd, struct stat *buf);
int lstat(const char *restrict pathname, struct stat *restrict buf);
int fstatat(int fd, const char *restrict pathname, struct stat *restrict buf, int flag);
```

一旦给出 pathname
- stat返回此文件有关的信息结构
- fstat 获得已在描述符fd上打开文件的 有关信息
- lstat类似stat，但是当 文件时一个 符号链接时，返回符号链接的信息，而不是引用的文件的信息
- fstatat 为一个 相对于当前打开目录(fd) 的路径名 返回文件统计信息。flag设置了 AT_SYMLINK_NOFOLLOW 标志时，fstatat不会跟随符号链接，而是返回符号链接本身的信息。

stat的基本形式
```C
struct stat
{
    mode_t st_mode; // filt type & mode (permission)
    ino_t st_ino;   // i-node number
    dev_t st_dev;   // device number
    dev_t st_rdev;  // device number for special files
    nlink_t st_nlink;   // number of link
    uid_t st_uidl;  // owner's user id
    gid_t st_gid;   // owner's group id
    off_t st_size;  // size in bytes
    struct timespec st_atime;   // time of last access
    struct timespec st_mtime;   // time of last modification
    struct timespec st_ctime;   // time of last file status change
    blksize_t st_blksize;   // best IO block size
    blkcnt_t st_blocks; // number of disk blocks allocated
};
```

timespec 结构按照 秒和纳秒 定义了时间，至少包含：
- time_t tv_sec;
- long tv_nsec;


## 4.3 文件类型

- 普通文件 regular file
- 目录文件 directory file
- 块特殊文件 block special file, 提供对设备(如磁盘) 带缓冲的访问，每次访问以固定长度进行
- 字符特殊文件 character special file, 提供对设备 不带 缓冲的访问，每次访问长度可变。 系统中所有设备 要么是 字符特殊文件，要么是 块特殊文件
- FIFO，用于==进程间通信==，有时也称为 ==命名管道==。15.5
- socket，进程间的网络通信。也可以在一台机器上的 进程间进行 非网络通信
- 符号链接

POSIX.1 允许实现 将 进程间通信 对象 (消息队列和信号量) 说明为 文件。



## 4.25 文件访问权限位



# ch05 标准IO库
库由 ISO C标准说明
有很多细节，如 缓冲区分配，以优化的块长度执行IO。


## 5.2 流和FILE对象

第三章中，所有IO函数都是围绕 文件描述符的。
对于标准IO库，它们的操作都是围绕 流(stream) 进行的。

标准IO文件流可以用于 单字节或多字节(宽) 字符集。
流的定向 决定了 读写的是 单字节还是多字节， 当一个流最初被创建时，它并没有定向。 在 未定向的流上 使用 一个多字节IO函数(wchar.h)，则该流的定向设置为宽定向。 在未定向的流上 使用 一个单字节IO函数，就设置为 字节定向。
只有2个函数可以改变流的 定向。 freopen 清除流的定向，fwide设置(不改变)流的定向

fopen 返回指向 FILE对象的 指针。该对象通常是一个struct，这个结构包含了 标准IO库 为管理该流 需要的所有信息，包括用于 实际IO的 fd，指向用于该流缓冲区的指针，缓冲区长度，当前缓冲区中的字符数，出错标志 等。


## 标准输入，标准输出，标准错误
对一个进程预定义了3个流
这3个流 引用的文件 与 文件描述符STDIN_FILENO,STDOUT_FILENO,STDERR_FILENO，所引用的相同。

这3个标准IO流 通过 预定义 的文件指针 stdin,stdout,stderr 来引用。

## 5.4 缓冲

提供缓冲的目的是 尽可能减少 read write 的调用次数。

标准IO提供了 ==3种 缓冲==
1. 全缓冲，填满标准IO缓冲区后 才进行实际IO操作。对于磁盘上的文件通常是由 标准IO库进行 全缓冲的。 在流上执行第一次IO操作时，相关标准IO函数通常调用malloc 获得需使用的缓冲区。  冲洗(flush)是 标准IO缓冲区的写操作的， 缓冲区可以由标准IO例程自动冲洗(如，填满缓冲区时)，或者调用 fflush 函数。 在标准IO库中，flush是将 缓冲区内容写到磁盘。 在终端驱动程序方面(如 18章的 tcflush)，flush表示丢弃已存储在缓冲区中的数据。
2. 行缓冲，当输入，输出 中遇到 换行符时，执行IO操作。 当流涉及终端时，通常使用 行缓冲。 行缓冲有2个限制：1.收集每一行的缓冲区的长度是固定的，所以只要填满了缓冲区，即使没有换行，也会IO操作。2.任何时候，通过标准IO 从 一个不带缓冲的流 或 一个行缓冲的流 中得到输入数据，那么就会flush所有行缓冲输出流。
3. 不带缓冲。 如 fputs。 标准错误流 stderr 通常不带缓冲。

对任何一个流，可以修改系统默认设置，下面的2个函数都可以更改缓冲类型
stdio.h
```C
void setbuf(FILE *restrict fp, char *restrict buf);
int setvbuf(FILE *restrict fp, char *restrict buf, int mode, size_t size);
```
buf必须指向一个 BUFSIZ(定义在stdio.h) 长的 缓冲区。
setbuf 之后，流就是全缓冲的， 但是如果该流和一个终端设备有关，那么有些系统也会设置为 行缓冲。  buf设置为 NULL，关闭缓冲

setvbuf可以精确说明所需的缓冲类型，mode
- _IOFBF    全
- _IOLBF    行
- _IONBF    不

如果流是缓冲，buf是NULL，标准IO库会自动分配 BUFSIZ(有些系统不是用这个，GNU C使用 stat结构中 st_blksize) 长度的缓冲区。

如果为函数分配了一个自动变量类的标准IO缓冲区，则从该函数返回之前，必须关闭流。


## 5.5 打开流
stdio.h
```C
FILE *fopen(const char *restrict pathname, const char *restrict type);
FILE *freopen(const char *restrict pathname, const char *restrict type, FILE *restrict fp);
FILE *fdopen(int fd, const char *type);
```

fopen 打开pathname 上的文件

freopen，在一个指定流上打开指定文件，如果流已打开，则先关闭，如果流已定向，则使用 freopen清除该定向。 一般用于将一个指定文件 打开为 一个预定义的流：标准输入，输出，错误。

fdopen 取一个已有的文件描述符 (可以从 open,dup,dup2,fcntl,pipe,socket,socketpair,accept 获得)，并使一个标准IO流 与 该描述符相结合。
通常用于 创建管道和网络通信通道函数 返回的描述符。 因为这些 特殊类型的文件不能能用 fopen 打开。

type是 读写方式，有15种
- r,rb，读
- w,wb，文件截断为0长，写
- a,ab，追加
- r+,r+b,rb+，读写
- w+,w+b,wb+，截断为0长，读写
- a+,a+b,ab+，文件尾读写


`int fclose(FILE *fp);`


## 5.6 读写流

打开流后，有3种不同类型的非格式化IO 读写。
1. 每次一个字符的IO
2. 每次一行的IO，fgets,fputs
3. 直接IO，fread，fwrite

一次读一个字符
stdio.h
```C
int getc(FILE *fp);
int fgetc(FILE *fp);
int getchar(void);
```
getchar 等于 getc(stdin)

getc和 fgetc的区别是：getc可以被实现为 宏， fgetc不行，这意味着
- getc的参数不能是有副作用的表达式，因为它可能被计算多次
- 因为 fgetc一定是一个函数，所以可以获得它的地址，可以作为参数传递给其他函数
- 调用 fgetc 所需的时间 很可能比 getc要长，因为 ==调用函数所需的时间 通常长于调用宏==

这3个函数，已打文件尾 或出错 都返回 -1，所以需要下面的函数来区分 文件尾 和 出错
stdio.h
```C
int ferror(FILE *fp);
int feof(FILE *fp);     // 非0真，0假
void clearerr(FILE *fp);    // 清除 FILE对象中的 2个标志：出错标志，文件结束标志
```

从流中读取数据后，可以使用  ungetc  压回去
stdio.h
`int ungetc(int c, FILE *fp);`
可以多次 压回，但是每次只能一个字符。
回送的字符 可以不是上次读到的， 但不能是 EOF。
到达文件尾时依然可以回送，下次读返回这个字符，再读 返回EOF， 一次成功的 ungetc 会清除 流的 文件结束标志。


每个输入函数 都有一个 输出函数对应
stdio.h
```C
int putc(int c, FILE *fp);
int fputc(int c, FILE *fp);
int putchar(int c);
```


## 5.7 每次一行IO
每次输入一行
stdio.h
```C
char *fgets(char *restrict buf, int n, FILE *restrict fp);
char *gets(char *buf);  // 不推荐，无法指定缓冲区长度，会缓冲区溢出。
```
将读入的行 送到 buf 中。
gets从标准输入读， fgets从指定的流中读。

gets不能将换行符存入缓冲区，fgets可以

每次输出一行
stdio.h
```C
int fputs(const char *restrict str, FILE *restrict fp);
int puts(const char *str);
```

fputs 将 一个以 null 字节终止的字符串写到指定的流，尾端的终止符null 不写出。通常，null字节之前是一个 换行符。
puts 将一个 null字符终止的 字符串写到 标准输出，终止符不写， 然后 将一个 换行符 写到 标准输出。

puts不像 gets那样不安全，但还是 少用。 以免需要记住 它会添加一个 换行。

总是使用 fgets,fputs，我们自己处理换行符。


## 5.8 标准IO的效率

。测试的代码是 从文件读，再写到其他文件 (就是复制文件)

fgets+fputs > getc+puts ~= fgetc+fgetc

![1ff05c515c708c5dcc0d5e1842a6c801.png](../_resources/1ff05c515c708c5dcc0d5e1842a6c801.png)


系统CPU时间差不多，因为这些程序对 内核提出的 读写请求书 基本相同。

根据本节 和 3.9 可知：标准IO库 和 直接调用read，write， 性能差不多。


## 5.9 二进制IO
读写一个struct的话：
getc，一次一个char，太慢
fgetc，遇到null会停止

stdio.h
```C
size_t fread(void *restrict ptr, size_t size, size_t nobj, FILE *restrict fp);
size_t fwrite(const void *restrict ptr, size_t size, size_t nobj, FILE *restrict fp);
```

2种常见用法
1. 读 写 二进制数组，将浮点数组的第2-5个元素写到一个文件上
```C
float data[10];
if (fwrite(&data[2], sizeof(float), 4, fp) != 4)
    err_sys("error");
```

2. 读写一个struct
```C
struct {
    short count;
    long total;
    char name[NAMESIZE];
} item;

if (fwrite(&item, sizeof(item), 1, fp) != 1)
    err_sys("error");
```

二进制IO的问题是，只能 读 同一个OS 写的数据。因为
1. 一个struct，在不同的 编译环境 和 OS 中 成员的偏移量 不同
2. 不同OS，存储 多字节整数 和 浮点值得 二进制格式不同。


## 5.10 定位流

3种方法
- ftell, fseek
- ftello, fseeko
- fgetpos, fsetpos

需要移植到 非UNIX，应当使用 fgetpos，fsetpos


## 5.11 格式化IO


### 格式化输出
5个printf
stdio.h
```C
int printf(const char *restrict format, ...);
int fprintf(FILE *restrict fp, const char *restrict format, ...);
int dprintf(int fd, const char *restrict format, ...);
int sprintf(char *restrict buf, const char *restrict format, ...);
int snprintf(char *restrict buf, size_t n, const char *restrict format, ...);
```

printf 写到标准输出
fprintf 写到指定的流
dprintf 写到指定的文件描述符
sprintf 将格式化的字符送入 buf，在尾端自动加一个null字节，但不算返回的长度。可能导致 缓冲区溢出
snprintf 上面的无溢出版本

### 格式控制符

`%[flags][fldwidth][precision][lenmodifier]convtype`

|标志|说明|
|--|--|
|'|按千位分组|
|-|左对齐|
|+|正负号|
|空格|如果第一个字符不是正负号，就加一个空格|
|#|指定另一种转换形式(如，对于16进制，添加0x前缀)|
|0|添加前导0|

fldwidth 最小字段宽度。  非负十进制，或 一个 *

precision，整型转换后 最少输出数字位数，浮点数转换后 小数点后的最少位数，字符串转换后的最大字节数。 精度是 一个 .  后面跟随 可选的 非负十进制数 或一个 *

lenmodifier 说明参数长度
|长度修饰符|说明|
|--|--|
|hh|将相应参数按 signed 或 unsigned char 类型输出|
|h|按 signed 或unsigned short|
|l|按 signed 或unsigned long 或 宽字符 类型输出|
|ll|按 signed 或 unsigned long long 类型输出|
|j|intmax_t 或 uintmax_t|
|z|size_t|
|t|ptrdiff_t|
|L|long double|

convtype 必选
按照顺序应用到 format 之后的参数。
也允许显式 `%n$` 来表示 第 n 个参数。 不能混用

|转换类型|说明|
|--|--|
|d,i|有符号十进制|
|o|无符号八进制|
|u|无符号10进制|
|x,X|无符号16进制|
|f,F|双精度浮点数|
|e,E|指数格式双精度浮点数|
|g,G|根据转换后的值 解释为 f,E,e 或E|
|a,A|16进制 指数格式 双精度浮点数|
|c|字符，若带长度修饰符l，为宽字符|
|s|字符串，带l，宽字符|
|p|指向void的指针|
|n|到目前为止，次printf调用输出的字符的数目 将被写到 指针所指的带符号整型中|
|%|一个 %字符|
|C|宽字符，XSI扩展，等于 lc|
|s|宽字符，XSI扩展，等于 ls|


### 变体vprintf
下面5种类似上面的5种，但是可变参数列表 变成了 arg
stdarg.h + stdio.h
```C
int vprintf(const char *restrict format, va_list arg);
int vfprintf(FILE *restrict fp, const char *restrict format, va_list arg);
int vdprintf(int fd, const char *restrict format, va_list arg);
int vsprintf(char *restrict buf, const char *restrict format, va_list arg);
int vsnprintf(char *restrict buf, size_t n, const char *restrict format, va_list arg);
```


### 格式化输入
3个scanf
stdio.h

```C
int scanf(const char *restirct foramt, ...);
int fscanf(FILE *restrict fp, const char *restrict format, ...);
int sscanf(const char *restrict buf, const char *restrict format, ...);
```

`%[*][fldwidth][m][lenmodifier]convtype`

`*` 用于抑制转换。按照转换说明的其余部分对输入进行转换，但是转换结果并不存到参数中。


## 5.12 实现细节

获得FILE的描述符
stdio.h
`int fileno(FILE *fp);`
不是ISO C， 是 POSIX.1 的扩展

如果要调用 dup，fcntl，就需要这个函数。

。。有个一页多的代码，为3个标准流，一个普通文件的流 打印有关缓冲的 状态信息

## 5.13 临时文件
ISO C 提供2个函数 来帮助创建临时文件
stdio.h
```C
char *tmpnam(char *ptr);
FILE *tmpfile(void);
```

tmpnam产生一个与现有文件名不同的 一个有效路径名 字符串。 每次调用，都产生一个不同的路径名， 最多效用 TMP_MAX (stdio.h中) 次，
如果 ptr 是NULL，产生的路径会放到 静态区，返回指针， 我们要保存 路径名，而不是 指针。
ptr 非NULL，则认为它指向的 字符串的长度 至少 L_tmpnam 个。路径名存放到 ptr中，返回ptr

tmpfile 建立一个 临时 二进制文件 (类型为 wb+)， 关闭该文件 或 程序结束时，自动删除。


## 5.14 内存流

3个函数
stdio.h
`FILE *fmemopen(void *restrict buf, size_t size, const char *restrict type);`

type的取值就是 r,rb,w,wb,a,ab,r+,r+b,rb+,w+,w+b,wb+,a+,a+b,ab+

buf如果是null，打开流进行读 写 没有任何意义。
任何时候需要添加 流缓冲区 中数据量 以及 调用 fclose,fflush,fseek,fseeko,fsetpos 都会在当前位置写入一个 null 字节。

stdio.h
`FILE *open_memstream(char **bufp, size_t *sizep);`
和fmemopen的不同：
- 创建的流只能 写打开
- 不能指定自己的缓冲区，但是可以通过 bufp,sizep 访问缓冲区地址和大小
- 关闭流后 需要 手动释放缓冲区
- 对流添加字节会增加缓冲区大小

wchar.h
`FILE *open_wmemstream(wchar_t **bufp, size_t *sizep);`



## 5.15 标准IO的替代软件

标准IO的不足之处是 效率不高，使用 每次一行的 fgets,fputs时， 通常需要2次复制，一次是 内核 和 标准IO缓冲区 之间(调用 read，write时)， 一次是 标准IO缓冲区 和 用户程序中的 行缓冲区 之间。

快速IO库，fio(3) 避免了这一点，方法是 使读一行的函数返回指向该行的指针，而不是复制到 另一个缓冲区。

另一种替代，sfio。速度上和 fio 相近，通常快于 标准IO。 sfio扩展了IO流，不仅可以代表文件，也可以代表 存储区，可以编写处理模块，以栈的方式 压入IO流。

另一个软件包，ASI (alloc stream interface) 使用了 mmap

为内存较小的系统(如 嵌入式) 的库： uClibc C库，Newlib C库



# ch6 系统数据文件和信息

## 6.2 口令文件


## 6.7 其他数据文件
前面讨论了2个系统数据文件：口令文件和组文件。
unix还有很多其他文件，例如，BSD网络软件有一个记录各网络服务器所提供服务的数据文件(/etc/services)，有一个记录协议信息的数据文件(/etc/protocols)，还有一个记录网络信息的数据文件(/etc/networks)。
这些文件的接口和 之前的口令文件和组文件相似。
一般情况下，每个数据文件至少有3个函数
- get函数，读下一个记录，如果需要，还会打开该文件。这种函数通常返回指向一个结构的指针，到达文件尾端时返回空指针。大多数get函数返回指向一个静态存储类结构的指针，如果要保存其内容，则需要复制它。
- set函数，打开相应数据文件，然后反绕该文件。如果需要在文件起始处开始处理，则调用此函数。
- end函数，关闭相应数据文件。

如果数据文件支持 某种形式的键搜索，则也提供搜索具有指定键的记录的例程。例如，对于口令文件，提供了2个按键进行搜索的程序：getpwnam寻找具有指定用户名的记录，getpwiud寻找具有指定用户ID和记录。

|说明|数据文件|头文件|结构|附加的键搜索函数|
|--|--|--|--|--|
|口令|/etc/passwd|pwd.h|passwd|getpwnam,getpwuid|
|组|/etc/group|grp.h|group|getgrnam,getgrgid|
|阴影|/etc/shadow|shadow.h|spwd|getspnam|
|主机|/etc/hosts|netdb.h|hostent|getnameinfo,getaddrinfo|
|网络|/etc/networks|netdb.h|netent|getnetbyname,getnetbyaddr|
|协议|/etc/protocols|netdb.h|protoent|getprorobyname,getprotobynumber|
|服务|/etc/services|netdb.h|servent|getservbyname,getservbyport|


## 6.9 系统表示
POSIX.1 定义了 uname， 返回与主机和OS有关的信息
sys/utsname.h
`int uname(struct utsname *name);`

传递一个 utsname 结构，此函数填充这个结构。
POSIX定义了该结构最少需要提供的字段。
```C
struct utsname {
    char sysname[];     // OS's name
    char nodename[];    // node's name
    char release[];     // OS's current release
    char version[];     // OS's current version
    char machine[];     // name of hardware type
};
```
每个字符串都以 null 字节结尾。

---
unsid.h
`int gethostname(char *name, int namelen);`

返回主机名，通常就是TCP/IP网络上的主机的名字。

hostname(1)

## 6.10 时间和日期例程

unix内核提供了 1970.1.1 以来的 秒数

> 返回当前时间和日期

time.h
`time_t time(time_t *calptr)`

时钟通过 clockid_t 类型进行标识
|标识符|选项|说明|
|--|--|--|
|CLOCK_REALTIME| |实时系统时间|
|CLOCK_MONOTONIC|`_POSIX_MONOTONIC_CLOCK`|不带负跳数的实时系统时间|
|CLOCK_PROCESS_CPUTIME_ID|`_POSIX_CPUTIME`|调用进程的CPU时间|
|CLOCK_THREAD_CPUTIME_ID|`_POSIX_THREAD_CPUTIME`|调用线程的CPU时间|


---
> 获得指定时钟的时间

timespec在4.2节中，它把时间分为 秒 和 纳秒
sys/time.h
`int clock_gettime(clockid_t clock_id, struct timespec *tsp);`

clock_gettime 比 time，可以获得更高精度的时间值


sys/time.h
`int clock_getres(clockid_t clock_id, struct timespec *tsp);`


sys/time.h
`int clock_settime(clockid_t clock_id, const structtimespec *tsp);`


---
一旦获得 秒数后，就要调用函数 将其转化为 时间结构。
localtime, mktime, strftime

```C
struct tm {
    int tm_sec;
    int tm_min;
    int tm_hour;
    int tm_mday;
    int tm_mon;
    int tm_year;
    int tm_wday;
    int tm_yday;
    int tm_isdst;
};
```
秒可以超过59，代表闰秒。
UTC的正式定义不允许双闰秒，所以秒是 `[0, 60]`


![2d5ddc3efd69233003bfd4aae5984981.png](../_resources/2d5ddc3efd69233003bfd4aae5984981.png)


---
time.h
```C
struct tm *gmtime(const time_t *calptr);
struct tm *localtime(const time_t *calptr);
```
区别是，localtime将日历时间转换为本地时间(考虑 时区 和 夏令时)， gmtime是转换为 UTC时间


time.h
`time_t mktime(struct tm *tmptr);`
本地时间 转为 time_t 值


time.h
```C
size_t strftime(char *restrict buf, size_t maxsize, const char *restrict format, const struct tm *restrict tmptr);
size_t strftime_1(char *restrict buf, size_t maxsize, const char *restrict format, const struct tm *restrict tmptr, locale_t locale);
```
类似printf，但很复杂

p154，37中ISO C的转换说明符，如%a,%A,%b,%B,%c,%C ... %z,%Z



time.h
`char *strptime(const char *restrict buf, const char *restrict format, struct tm *restrict tmptr);`
是 strftime 的反过来， 将字符串时间 转换为 tm




# ch07 进程环境

将学习：
- 程序执行时，main如何被调用。
- 命令行参数怎么传递给程序
- 典型的存储空间分布
- 如何分配另外的存储空间
- 进程如何使用环境变量
- 进程的不同终止方式
- longjmp,setjmp及它们与栈的交互
- 查看进程的资源限制


## 7.2 main函数
`int main(int argc, char *argv[]);`

内核执行C程序时 (使用一个exec函数，8.10节说明)，在调用 main前 先调用一个 特殊的 启动例程。可执行程序文件将次启动例程指定为 程序的 起始地址，这是由 连接器设置的，连接器是由C编译器调用的。 启动例程 从内核取得命令行参数 和环境变量值，然后 为 按上述方式调用 main函数做好准备


## 7.3 进程终止(8种)
共8种。5种正常终止
- 从main返回
- 调用exit
- 调用 _exit 或 _Exit
- 最后一个线程 从其启动例程返回 11.5节
- 从最后一个线程调用 pthread_exit 11.5节

异常终止3种：
- 调用abort 10.17节
- 接到一个信号 10.2节
- 最后一个线程对取消请求做出响应 11.5 和12.7节

7.2中提到的启动例程是这样的：从main返回后 立刻调用 exit 函数。

---
stdlib.h
```C
void exit(int status);
void _Exit(int status);
```

unistd.h
`void _exit(int status);`


---
ISO C规定，一个进程 最多注册 32个函数，这些函数 由 exit 自动调用。称为 exit handler。 使用 atexit 来注册

stdlib.h
`int atexit(void (*func)(void));`
执行顺序和 注册顺序 相反。 多次注册，也会多次执行。

![10c9c6e16f3be6537fec5d50f1804645.png](../_resources/10c9c6e16f3be6537fec5d50f1804645.png)


注意，内核使得程序执行的 唯一方法是 调用一个 exec函数。 
进程自愿终止的唯一方式是 显式 或 隐式(通过exit) 调用 _exit或 _Exit。
进程也可以 非自愿地 由一个信号使其终止(上图中没有显示)


## 7.4 命令行参数

`./a.out arg1 TTT arg3`



## 7.5 环境表
每个程序都有一个环境表。和参数表一样，环境表 也是一个 字符指针数组，每个指针 包含一个 以null 结束的 C字符串的地址。
全局变量 environ 包含了该指针数组的 地址
`extern char **environ;`

通常使用 7.9的 getenv, putenv 访问特定的环境变量，而不是使用 environ变量

## C程序的存储空间布局

C程序一直由下列几部分组成
- ==正文段==，CPU执行的机器指令，通常，==正文段是共享的==，所以即使执行多个程序(如 多个shell) 在存储器中也只需一个副本， 正文段是只读的。
- 初始化数据段，通常称为 ==数据段==，包含了 程序中 需明确赋初始值的变量，如，C中任何函数之外的声明：int mxcnt = 99; 使得这个变量 以其初值放在 初始化数据段中。
- 未初始化数据段，通常称为 ==bss段==，由符号开始的块(block started by symbol)，在程序开始之前，内核将此段中的数据 初始化为 0 或空指针， 函数外 声明：long sum[1000]; 使得这个变量存放在 非初始化数据段中。
- ==栈==，自动变量 及每次函数调用时所需保存的信息 都放在此段中。
- ==堆==，通常 在堆中进行动态存储分配。按惯例，==堆位于 未初始化数据段 和 栈之间。==

![9de4ec46af2d8a66d242e9d101f99ea4.png](../_resources/9de4ec46af2d8a66d242e9d101f99ea4.png)

==对于 32位 intel x86 的 linux，正文从 0x08048000 开始。栈底则在 0xC0000000 之下开始==

a.out 还有其他类型的段，如 包含符号表的段，包含调试信息的段，包含动态动向库链接表的段 等，这些不会加载到 进程执行的 程序映像中。

==未初始化数据段的 内容并不存放到 磁盘程序文件中==。 因为内核在 程序开始前 将它们设置为0， 需要存放在 磁盘程序文件中的 段 只有 正文段 和 初始化数据段。


size(1) 命令 报告 正文段，数据段，bss段的长度 (以字节为单位)


## 7.7 共享库 gcc -static 不使用共享库
不同OS中，可能使用不同的方法 说明是否要 使用 共享库。
比如， cc(1), ld(1)


![baaa8cf76a0868fbad88be98f5b06359.png](../_resources/baaa8cf76a0868fbad88be98f5b06359.png)




## 7.8 存储空间分配 malloc,calloc,realloc,free

- malloc, 分配指定字节数的存储区，初始值不确定
- calloc, 为指定数量 指定长度的 对象分配存储空间，所有bit都初始化为0
- realloc, 增加或 减少 已分配的 分配区的长度。增加时，可能需要将 之前的内容 移动到 另一块足够大的区域。新增区域 初始值不确定。

stdlib.h
```C
void *malloc(size_t size);
void *calloc(size_t nobj, size_t size);
void *realloc(void *ptr, size_t newsize);   // 新长度，不是 增量

void free(void *ptr)
```

3个分配器返回的 指针 一定是 适当对齐的，使其可以用于任何数据对象。 例如，在一个特定的系统上， 如果最 苛刻的对齐要求是 double 必须在 8的倍数地址单元处，那么这3个 函数返回的指针都是 这样对齐的。

都返回 void *，所以 如果在 程序中 包括了 #include stdlib.h (以获得函数原型)，那么 将这3个函数的 返回 赋予 一个不同类型的 指针时， 不需要 显式 执行强制转换。

这些分配例程 通常使用 sbrk(2) 系统调用实现。
虽然sbrk 可以扩大 或缩小 进程的存储空间，但是大多数 malloc 和 free 的实现 都不减小 进程的存储空间， 将它们保存在 malloc池中，而不是 返回给 内核。

大多数实现 分配的 存储空间 比所要求的 要稍大一些，额外的空间用来记录 管理信息：分配块的长度，指向下一个分配快的指针。 这就意味着，在一个已分配区 的尾端 或 起始位置之前 进行写操作，会改变 另一块的管理记录信息。

释放 已释放的块 也是致命的

由于 存储空间分配错误 很难追踪，所以 某些系统 提供了 另一种 实现版本。每次掉豫能股 这3个 分配函数 或 free时，会进行 附加的检查。

---

有很多可以代替 malloc 和 free 的函数。
1. libmalloc
2. vmalloc
3. quick-fit
4. jemalloc
5. TCMalloc，==从高速缓存中分配 缓冲区==。。。
6. alloca, 在栈帧上，而不是 堆，所以 函数返回时，自动释放。




## 7.9 环境变量
stdlib.h
`char *getenv(const char *name);`

设置环境变量
stdlib.h
putenv, setenv, unsetenv, clearenv

这些函数可能不是所有系统都支持， linux是全都支持的。


## 7.10 setjmp,longjmp

。。 p 170

setjmp.h
```C
int setjmp(jmp_buf env);

void longjmp(jmp_buf env, int val);
```



## 7.11 getrlimit,setrlimit
查看进程的 资源限制

sys/resource.h
```C
int getrlimit(int resource, struct rlimit *rlptr);
int setrlimit(int resource, const struct rlimit *rlptr);
```



# ch08 进程控制
创建进程，执行程序，终止进程


## 8.2 进程标识

非负int，唯一。
可以复用。 大多数UNIX 实现 延迟复用算法， 使得 新的进程ID 不同于 最近终止的进程的ID。 防止认错。

ID 0 的进程 通常是 调度进程。
ID 1 是init进程，/sbin/init
ID 2 页守护进程，负责支持 虚拟存储器系统的 分页操作。

unistd.h
```C
pid_t getpid(void);      // 调用进程的ID
pid_t getppid(void);        // 调用进程的 父进程ID
uid_t getuid(void);     // 调用进程的 实际用户ID
uid_t geteuid(void);       //       有效用户ID
gid_t getgid(void);     //       实际组ID
gid_t getegid(void);        //       有效组ID
```

## 8.3 fork

unistd.h
`pid_t fork(void);`
子进程返回0，父进程返回子进程ID，出错返回-1

子进程是父进程的 副本。 获得 父进程的 数据空间，堆，栈的 副本。

由于fork 之后，经常跟着 exec，所以 很多实现 并不执行 父进程的数据段，栈，堆 的 完全克隆。作为替代，使用了 写时复制(copy on write, COW) 技术。 这些区域 有 父进程，子进程 共享，而且 内核 将访问权限改为 只读。 如果 父进程，子进程 中 任意一个 试图修改这些区域时， 内核 为 修改区域 的那块 内存 做一个副本， 通常 是 虚拟存储系统中的 page。

linux 提供了 另一种 新进程创建函数 clone(2) 系统调用。 是fork 的推广，允许调用者 控制 哪些部分 由 父进程 和 子进程 共享。

fork之后，哪个先执行，是不确定的。


文件共享
fork的一个特性是，父进程的所有打开的 fd 会被 复制到 子进程中。就像 对每个 fd 执行了 dup。 共享 一个文件表项。

![84447444d9f47b60a1d391a8eb68004c.png](../_resources/84447444d9f47b60a1d391a8eb68004c.png)


如果没有任何形式的 同步，那么它们的输出 会混合。

fork后处理fd有2种情况
1. 父进程等待 子进程完成，这种情况，父进程 无需 对 fd 进行任何处理。
2. 父进程 和 子进程 各自执行不同的 程序段。 父进程 和 子进程 各自关闭 它们不需要的 fd。

fork失败 有2个主要原因
- 系统中有太多的进程 (通常意味着 出了问题)
- 实际用户ID 的进程总数 超过了 CHILD_MAX系统限制。


fork有2个用法
1. 父进程希望复制自己，使 父进程 和 子进程 同时执行 不同的 代码段。这在 网络服务进程 中是 很常见的： 父进程 等待 客户端的请求，收到请求后，fork，使子进程处理此请求，父进程 继续等待
2. 一个进程 要执行 一个不同的程序。


## 8.4 vfork
形参，返回值 和 fork相同，但 语义不同。

可移植的app 不应该使用这个 函数。

vfork 和fork一样，也是创建子进程，但是 并不 完全 复制 父进程的 地址空间， 因为 子进程 会立即调用 exec (或 exit)。 在 子进程 调用 exec/exit 之前，它在 父进程的空间中运行。  但 子进程 如果 需修改了 数据，进行函数调用，没有调用exec/exit 就返回， 会导致 未知的结果。

vfork 保证 子进程 先运行。在 子进程 调用 exec 或 exit 之后，父进程 才可能 被 调度。


## exit

终止进程有5+3种
- main中 执行return
- exit
- `_exit,_Exit`
- 最后一个线程 return
- 最后一个线程 pthread_exit

- abort
- 收到某些信号
- 最后一个线程 对 取消 请求做出响应。

不管进程如何终止，最后 都会执行 内核中的一段代码。 这段代码 为相应进程 关闭所有打开的 描述符，释放使用的存储器。

对于上述的终止，我们希望 进程能够 通知它的父进程，它是如何终止的。
- 对于3个终止函数(exit,_exit,_Exit)，将其退出状态 作为参数传递给 这3个终止函数。
- 异常终止，内核 产生一个 指示其异常终止原因的 终止状态。

在任意一种情况下，该终止进程的 父进程 都能使用 wait，waitpid 获得 终止状态。

对于父进程 已经 终止的 所有进程，它们的父进程都改成 init 进程。 操作过程大致是：一个进程终止时，内核逐个检查所有活动进程，以判断它是否是 正要终止进程的 子进程，如果是，则该子进程 的 父进程ID就改为 init进程的ID, 1。

内核为每个终止子进程 保存了一定量的信息，所以 当 已终止的进程 的父进程 调用 wait,waitpid 时，可以获得这些信息，至少包括 进程ID，终止状态，使用的CPU时间总量。

一个已经终止，但其父进程 尚未 对其进行处理 (获取子进程的有关信息，释放它仍占用的资源) 的进程 被称为 ==zombie 进程==。 ps(1) 中 僵尸进程 的状态是 Z。


## 8.6 wait,waitpid

当一个进程 正常 或 异常 终止时，内核就 向其父进程 发送 SIGCHLD 信号。
因为 子进程终止 是异步事件，所以 这种信号 也是 内核向 父进程 发的 异步通知。父进程可以选择 省略该信号，或者 提供一个 该信号 发生时 即被调用的 函数 (信号处理程序)。 对于这种信号，系统默认动作是 忽视它。

调用wait,waitpid时
- 如果其 所有子进程 都还在运行，则阻塞
- 如果一个子进程 已终止，正等待父进程 获取其终止状态，则取得该子进程的终止状态 并立刻返回
- 如果它没有任何子进程，则立刻出错返回

sys/wait.h
```C
pid_t wait(int *statloc);
pid_t waitpid(pid_t pid, int *statloc, int options);
```
区别：
- 在一个子进程终止前，wait会阻塞， waitpid 有一个选项，可以不阻塞
- waitpid 并不等待 在其调用之后的 第一个终止子进程，它有数个选项，可以控制它所等待的进程。

检查wait，waitpid 返回值 的 终止状态的 宏
WIFEXITED(status)，如果正常终止，则true，后续可以调用 WEXITSTATUS(status)，获得 子进程 传给 exit 或 _exit 参数的 低8位
WIFSIGNALED(status)，如果异常终止，则true，后续WTERMSIG(status) 获得导致终止的信号编号
WIFSTOPPED(status)，如果当前暂停子进程的返回的状态，则true，WSTOPSIG(status) 导致暂停的信号编号
WIFCONTINUED(status)，在作业控制暂停后已经继续的子进程返回了状态，则true


waitpid 的pid
- -1，等待任一子进程，和wait等效
- `>0`，等待进程ID等于pid的子进程
- 0，等待 组ID等于调用进程组ID 的任一子进程
- `<0`, 等待 组ID 等于 pid 绝对值的 任一子进程

options可选 WCONTINUED, WNOHANG, WUNTRACED



## 8.7 waitid
Single UNIX Specification 包括了 另一个取得进程终止状态的函数： waitid。
类似于 waitpid，但提供了更多的灵活性

sys/wait.h
`int waitid(idtype_t idtype, id_t id, siginfo_t *infop, int options);`

idtype: P_PID, P_PGID, P_ALL
options: WCONTINUED, WEXITED, WNOHANG, WNOWAIT, WSTOPPED


## 8.8 wait3, wait4

比 POSXI.1 的函数 wait,waitpid,waitid 提供的 功能要多一个。
多了一个参数，该参数允许 内核返回 由 终止进程 及其 所有子进程 使用的 资源情况
```C
#include <sys/types.h>
#include <sys/wait.h>
#include <sys/time.h>
#include <sys/resource.h>

pid_t wait3(int *statloc, int options, struct rusage *rusage);
pid_t wait4(pid_t pid, int *statloc, int options, struct rusage *rusage);
```

资源统计 包括 用户CPU时间总量，系统CPU时间总量，缺页次数，接受到信号的次数等。

linux 支持 这5种方法。



## 8.9 竞争条件

当多个进程都企图对 共享数据进行某种处理，而最后的结果由取决于 进程运行的顺序时，我们认为发生了 竞争条件。
如果fork之后的某种逻辑 显式或隐式地依赖于在fork之后 是父进程先运行 还是 子线程先运行，那么fork函数就是 竞争条件活跃的滋生地。

如果一个进程希望等待 一个子进程终止，则它必须调用 waitpid/wait。
如果进程等待 父进程终止，则可以使用：
```C
whlie (getpid() != 1)
    sleep(1);
```
这种循环 是 轮询(polling)， 会浪费CPU

为了避免竞争 和 轮询，在多个进程之间 需要有某种形式的 信号发送 和接收的方法。
10.16将说明信号机制的一种用法，各种形式的 IPC(进程间通信) 也可以用，在15，17章。



## 8.10 exec

unistd.h
```C
int execl(const char *pathname, const char *arg0, ... /* (char *)0 */);
int execv(const char *pathname, char *const argv[]);
int execle(const char *pathname, const char *arg0, ... /* (char *)0, char *const envp[] */);
int execve(const char *pathname, char *const argv[], char *const envp[]);
int execlp(const char *filename, const char *arg0, ... /* (char *)0 */);
int execvp(const char *filename, char *const argv[]);
int fexecve(int fd, char *const argv[], char *const envp[]);
```


![ea823f7b24c38082060029644a427f42.png](../_resources/ea823f7b24c38082060029644a427f42.png)



## 8.11 更改用户ID，组ID

unistd.h
```C
int setuid(uid_t uid);
int setgid(gid_t gid);

int setreuid(uid_t ruid, uid_t euid);
int setregid(gid_t rgid, gid_t egid);

int seteuid(uid_t uid);
int setegid(gid_t gid);
```


## 8.12 解释器文件

所有unix 都支持 解释器文件，这种文件是文本文件，其 起始行的形式：
`#! pathname [optional-argument]`

最常见的
`#! /bin/sh`

内核使得 调用 exec 函数的 进程 实际执行的不是 该解释器文件，而是 该文件 第一行的 pathname 所指定的文件。




## 8.13 system

在程序中 执行一个命令字符串很方便。
例如，假定要将 时间和日期 放到某个文件中，则可以使用 6.10 的函数实现这一点，调用time 得到当前日历时间，然后调用 localtime 将日历时间转化为 年月日时分秒，然后调用 strftime 对上面的结果进行格式化，最后写入到文件。

但是使用下面的 system 更简单
`system("data > file");`

stdlib.h
`int system(const char *cmdstring);`

如果cmdstring 是一个空指针，则仅当 命令处理程序可用时，system返回非0值，这一特征可以确定在一个 给定的OS上 是否支持 system。 unix总是支持 system的。

system的实现中 用到了 fork,exec,waitpid，所以有3种返回值
- fork失败，或 waitpid返回 除 EINTR之外的 出错，则system返回-1，并设置 errno
- 如果exec 失败(表示不能执行shell)，则返回值 如同 shell 执行了 exit(127);
- 否则3个函数都成功，system的返回值 是 shell的终止状态。


。。。有一些代码


## 8.14 进程会计 process accounting

大多数unix提供一个 选项以进行 进程会计 处理。 启用该选项后，每当进程 结束时 内核就写一个 会计记录。典型的会计记录包括 命令名，所使用的CPU时间总量，用户ID，组ID，启动时间。


## 8.15 用户标识
要获得 运行该程序用户的 登录名，可以 getpwuid(getuid()),但是如果 一个用户有多个登录名，这些登录名 对应着 同一个用户ID，该怎么办？

系统通常记录了 用户登录时使用的名字
unistd.h
`char *getlogin(void);`


# 8.16 进程调度

对进程提供的 只是 基于调度优先级的 粗粒度的控制。
调度策略 和 调度优先级 是由内核决定的。进程可以通过调整 nice值选择以更低优先级运行。 只有特权进程 允许提高 调度权限

nice值越小，优先级越高。 ( nice：友好的，进程的nice 越高，CPU占有越少，对于其他进程来说，它越友好。)

nice的范围是 0 - 2*NZERO-1， 默认NZERO

unistd.h
`int nice(int incr);`
只能调整自己的 优先级。
incr 被加到 调用进程的 nice 上。 如果incr太大，系统会降低到 最大合法值。



sys/resource.h
`int getpriority(int which, id_t who);`
可以获得一组相关进程的nice值
which可以取
- PRIO_PROCESS，进程
- PRIO_PGRP，进程组
- PRIO_USER，用户ID


sys/resource.h
`int setpriority(int which, id_t who, int value);`



## 8.17 进程时间

1.10 说明了我们可以度量的 3个事件： 墙上时钟时间，用户CPU时间，系统CPU时间。
任何进程都可以调用 times 获得 自己 及 终止子进程的上述值

sys/times.h
```C
clock_t times(struct tms *buf);

struct tms {
    clock_t tms_utime;  // user CPU time
    clock_t tms_stime;  // system CPU time
    clock_t tms_cutime; // user CPU time, terminated childrend
    clock_t tms_cstime; // system CPU time, terminated children
};
```
tms 结构中没有 墙上时钟， 墙上时钟 是返回值。





# ch09 进程关系
每个进程都有父进程， 初始的内核级进程的父进程是它自己。
子进程终止时，父进程得到通知并能取得子进程的退出状态。



## 9.4
每个进程除了有一个进程ID之外，还属于一个进程组

进程组是一个或多个进程的集合。通常，它们是在同一作业中结合起来的，同一进程组中的各进程接收来自同一终端的 各种信号。
每个进程组有一个唯一的进程组ID。 进程组ID类似进程ID，是一个正整数，可以存放在 pid_t 数据类型中。

unistd.h
`pid_t getpgrp(void);`

早期是 `pid_t getpgrp(pit_t pid);`

每个进程组 都有一个组长进程， 组长进程的进程组ID 等于其进程ID

进程组组长 可以创建 一个进程组，创建该组中的进程，然后终止。
只要进程组中有一个进程存在，这个进程组就存在，和组长进程是否终止 无关。
进程组的最后一个进程可以终止，也可以转移到 另一个进程组

进程调用 setpgid 可以 加入一个现有的进程组 或创建一个新的进程组
unistd.h
`int setpgid(pid_t pid, pid_t pgid);`

setpgid 将pid进程的进程组ID 设置为 pgid。如果这2个参数相等，则由pid指定的进程 变成进程组组长，如果pid是0，则使用调用者的进程ID，通过pgid是0，则使用pid指定的进程ID作为进程组ID

一个进程只能为它自己或它的子进程设置进程组ID，在它的子进程调用了 exec后，它就不能再更改 子进程的 进程组ID

在大多数作业控制shell中，在fork之后调用此函数，使父进程设置其子进程 的进程组ID，并且也使 子进程设置自己的进程组ID。
。。应该是 都设置为 父进程的进程组ID

这2个调用中有一个是冗余的，但可以确保 父进程和子进程中， 子进程 必然进入了 进程组 。 因为fork后 父进程，子进程的 执行顺序不确定，所以必须这样写。

在讨论信号的时候，会说明如何将一个 信号发送给 进程 或 进程组。


## 9.5 会话 session

会话是 一个或多个 进程组的 集合。

进程调用 setsid 建立一个新会话

unistd.h
`pid_t setsid(void);`

如果调用者不是进程组的组长，则此函数创建一个新会话，具体发生以下3件事
- 该进程成为 新会话的 会话首进程
- 该进程 变成一个新进程组组长
- 该进程没有控制终端，如果之前有，那么这种联系也被切断

如果调用进程 已经是一个进程组组长，函数返回出错。

unistd.h
`pid_t getsid(pid_t pid);`


## 9.6 控制终端


## tcgetpgrp,tcsetpgrp,tcgetsid
通知内核 哪个进程组 是前台进程组。  这样，终端设备驱动程序 就知道 将 终端输入 和 终端产出的信号 发往何处。


## 作业控制


## shell执行程序

![6f828a89a7d75262b320bdb982216e8a.png](../_resources/6f828a89a7d75262b320bdb982216e8a.png)


## 孤儿进程组

父进程已经终止的进程 称为 孤儿进程， 有 init进程 收养

POSIX.1 将 孤儿进程组 定义为：该组中每个成员的 父进程 要么是该组的一个成员，要么不是 该组所属会话的 成员。
所以，进程组不是 孤儿进程组的 条件是： 该组中有一个进程，它的父进程 属于 同一会话的 另一个组。


## FreeBSD实现

![db16bc4b51244a4c1be8726c81e7faa8.png](../_resources/db16bc4b51244a4c1be8726c81e7faa8.png)




# ch10 信号

信号是软件中断。
提供了一种处理异步事件的方法。

## 10.2 信号概念
每个信号都有一个名字。 以 SIG 开头。
大约有30+信号，都是 正整数
signal.h

不存在编号0 的信号。

信号出现时，可以告诉内核按下列3种方式之一进行处理，
1. 忽略此信号。大多数信号都可以这样处理。 但有==2种不能忽略：SIGKILL，SIGSTOP==。因为它们向内核 和 超级用户提供了 使进程终止 或 停止的 可靠办法。另外，忽略某些硬件异常产生的信号(如 非法内存引用，除以0)，则进程的运行行为是未定义的。
2. 捕捉信号。为了做到这一点，需要==通知内核在某种信号发生时，调用一个用户函数==。在用户函数中，执行用户希望 对这种事件进行的处理。不能捕捉 SIGKILL，SIGSTOP
3. 执行系统默认动作。下图就是默认动作，大多数信号，默认 终止该进程

终止+core 表示在进程当前工作目录的 core文件中 复制了 该进程的内存映像

linux 3.2.0 中，core文件名 通过 /proc/sys/kernel/core_pattern 配置

![4f60433a6db1973173add893f79777ce.png](../_resources/4f60433a6db1973173add893f79777ce.png)

p252 详细解释



## 10.3 signal
signal.h
`void (*signal(int signo, void(*func)(int)))(int);`

ISO C定义，由于不涉及 多进程，进程组 级 终端IO， 所以它对信号的定义非常模糊。以至于对unix系统 毫无作用。

由于signal 的语义与 实现有关，所以最好使用 sigaction 代替 signal

signo是 之前图中的 信号名， 
func的值 是常量 SIG_IGN，常量 SIG_DEL 或 接到此信号后需要调用的函数的 地址。 如果SIG_IGN，则向内核表示 忽略此信号 (SIGKIL,SIGSTOP无法忽略)。SIG_DEL表示 使用系统默认行为

signal函数的原型说明 此函数需要2个参数，==返回一个函数指针==。

上面的定义太复杂，可以：
```C
typedef void Sigfunc(int);
Sigfunc *signal(int, Sigfunc *);
```

signal.h中可以看到
```C
#define SIG_ERR (void (*)())-1
#define SIG_DEL (void (*)())0
#define SIG_IGN (void (*)())1
```
这些常量表示，指向函数的指针，该函数要求一个整型参数，而且无返回值。


---
执行一个程序时，所有信号的状态都是系统默认或忽略。通常所有信号都被设置为它们的默认动作，除非调用 exec的进程忽略该信号。 确切地讲，exec函数将原先设置为 要捕捉的信号 都更改为 默认动作，其他信号的规则不变 (一个进程 原先要捕捉的信号，当其执行一个新程序后，就不能再捕捉了，因为信号捕捉函数的地址很可能在 锁执行的 新程序文件中 无意义)


fork时，子进程继承父进程的信号处理方式，因为 子进程复制了 父进程的内存映像，所以 信号捕捉函数 在 子进程中是有意义的。

## 10.4 不可靠的信号
早期的版本中，信号是不可靠的， 它们可能丢失。


## 10.5 中断的系统调用
早期UNIX系统的 一个特性是：如果进程在执行一个 低速系统调用 而阻塞期间 捕捉到一个 信号，则该系统调用就被中断，不再继续执行。该系统调用返回出错，设置errno为 EINTR。

当捕捉到信号时，==被中断的是 内核中执行的系统调用==

为了支持这种特性，将系统调用分为2类：低速系统调用，其他系统调用。
低速系统调用时可能会使得进程永远阻塞的 系统调用，包括
- 如果某些类型文件(如读管道，终端设备，网络设备)的数据不存在，则读操作可能会使得调用者永远阻塞
- 如果这些数据不能被相同的类型文件立即接收，则写操作可能使得永远阻塞
- 在某种条件发生之前打开某些类型文件，可能发生阻塞(例如，要打开一个终端设备，需要先等待与之连接的 调制解调器应答)
- pause 函数 wait 函数
- 某些 ioctl 函数
- 某些进程间通信函数

与被中断的系统调用相关的问题是 必须显式处理出错。典型的代码序列如下( 我们希望被中断时 重启它)：
```C
again:
    if ((n = read(fd, buf, BUFSIZE)) < 0) {
        if (errno = EINTR)
            goto again;
        // handle other error
    }
```

BSD4.2,某些被中断的 系统调用会 自动重启，包括 ioctl,read,readv,write,writev,wait,waitpid。
BSD4.3,允许进程基于每个信号 禁用 上面的 自动重启。

BSD4.2 引入 自动重启的 一个理由是： 用户不知道 所使用的 输入，输出设备 是否是 低速设备。


## 10.6 可重入函数

进程捕捉到信号 并对其进行处理时，进程正在执行的 正常指令序列 就被 信号处理程序 临时中断，它首先执行 该信号处理程序中的 指令。 如果从信号处理程序返回(例如，没有调用 exit或 longjmp)，则继续处理 在捕捉到信号时 进程正在执行的 正常指令序列 (这类似于 发生硬件终端时所做的)。

但在信号处理程序中，不能判断捕捉到信号时 进程执行到哪里。
如果进程 正在执行 malloc， 此时 捕捉到信号，信号处理函数中 也进行 malloc，会发生什么？会对进程造成破坏，因为malloc为它所非配的存储区维护了一个链表，插入执行信号处理程序时，进程可能正在更改此链表。

Signle UNIX Specification 说明了 信号处理程序中 保证 调用安全的函数。
这些函数 是可重入的， 并被称为 是 异步信号安全的。
除了可重入外，在信号处理操作期间，它会阻塞任何会引起不一致的 信号发送。

下面是 异步信号安全 的函数。

![6fa3dde8e11cdcaabf5b58e3c4f50e2f.png](../_resources/6fa3dde8e11cdcaabf5b58e3c4f50e2f.png)

没有列入的大多数函数是 不可重入的，因为：
- 它们使用静态数据结构
- 它们调用malloc或 free
- 它们是标准IO函数，标准IO库的很多实现都以 不可重入方式使用全局数据结构



## 10.7 SIGCLD 语义

SIGCLD，SIGCHLD，很容易混淆
SIGCLD 是 System V 的一个信号，
SIGCHLD 是 BSD的。

SIGCHLD 信号语义 和其他信号的语义 相似。子进程状态改变后产生此信号，父进程需要嗲用 wait 函数 以检测发生了什么。

SIGCLD 不同于其他信号。对于SIGCLD的早期处理方式
1. 如果进程明确将 该信号配置为 SIG_IGN，则调用进程的 子进程 将不产生 僵尸进程。 这个和 SIG_DFL 的忽略不同。
2. 如果将SIGCLD 设置为 捕捉，则内核立即检查 是否有 子进程准备好 被等待，如果是，则调用SIGCLD处理程序。


## 10.8 可靠信号术语和语义

当造成信号的事件发生时，为进程 产生 一个信号， 事件可以是 硬件异常，软件条件，终端产生的信号，调用kill函数。
当信号产生时，内核通常在 进程表 中以某种形式设置一个标志。

当对信号采用了这种动作，我们说 向进程 递送了一个信号。在 信号产生(generation) 和递送(delivery) 之间的 时间间隔内， 信号是 未决的(pending)。

进程可以选用"阻塞信号递送"。

如果在进程解除 对某个信号的阻塞之前，这个信号发生多次，会如何？POSIX.1允许系统递送该信号一次或多次。如果递送该信号多次，则称这些信号进行了排队。但是除非支持POSIX.1 实时扩展，否则大多数UNIX 并不对信号排队，而是只递送这种信号一次。

如果多个信号要递送给进程，会如何？ POSIX.1 没有规定这些信号的递送顺序。但是 POSIX.1 建议：在其他信号之前 递送与 进程当前状态有关的信号，如SIGSEGV

每个进程都有一个 信号屏蔽字， 它规定了 当前要阻塞递送到 该进程的 信号集。对于 每种可能的信号，该屏蔽字 中都有一位与之对应。 对于某个信号，如果对应位已设置，则它当前是 被阻塞的。


## 函数kill，raise

kill函数将信号发送给进程或进程组，raise允许进程向自己发送信号。

signal.h
```C
int kill(pid_t pid, int signo);
int raise(int signo);
```

pid有4种情况
- pid>0，发给进程ID 为pid 的进程
- pid==0，发送给 与发送进程 同一进程组的所有进程
- pid<0， 发送给进程组ID 等于pid绝对值，且有权限发送信号的 所有进程
- pid==-1，发送给 有权限向它发送信号的 所有进程。


## 函数 alarm, pause

alarm 设置定时器，将来某个时刻 定时器会超时。 产生 SIGALRM 信号。
如果忽略或不捕捉此信号，则默认动作是 终止调用该 alarm函数的进程。

unistd.h
`unsigned int alarm(unsigned int seconds);`

每个进程只能有一个闹钟，如果在调用 alarm时，之前已经注册的闹钟还没有超时，则该 闹钟的余留值 作为 本次 alarm 的返回值。 之前注册的闹钟 被新值代替。

SIGALRM的默认动作 是终止进程， 但是大多数 使用闹钟的 进程 会捕捉此信号。


pause 使调用进程挂起直到捕捉到一个信号
unistd.h
`int pause(void);`


## 10.11 信号集

信号集 表示多个信号。

POSIX.1 定义 sigset_t 以包含一个信号集，并且定义了下列5个处理信号集的函数
signal.h
```C
int sigemptyset(sigset_t *set);
int sigfillset(sigset_t *set);
int sigaddset(sigset_t *set, int signo);
int sigdelset(sigset_t *set, int signo);
int sigismember(const sitset_t *set, int signo);
```


## sigprocmask
sigprocmask 可以检测，更改，或同时检测并更改 进程的信号屏蔽字

signal.h
`int sigprocmask(int how, cosnt sigset_t *restrict set, sigset_t *restrict oset);`

how可以选择： SIG_BLOCK，SIG_UNBLOCK，SIG_SETMASK

## sigpending
返回一个信号集，对于调用进程而言，其中的信号 是阻塞不能递送的，因为也是 当前未决的。


## sigaction

检查 或 修改 或 检查并修改 与指定信号 关联的 处理动作。
取代了UNIX 早期的 signal 函数。

signal.h
```C
int sigaction(int signo, consts struct sigaction *restrict act, struct sigaction *restrict oact);

struct sigaction {
    void (*sa_handler)(int);
    sigset_t sa_mask;
    int sa_flags;
    void (*sa_sigaction)(int, siginfo_t *, void *);
};
```

。。p278 很多很多


## 10.15 sigsetjmp,siglongjmp

调用 longjmp 有一个问题。当捕捉到一个信号时，进入信号捕捉函数，此时当前信号被自动加到 进程的信号屏蔽字中，这阻止了后来产生的 这种信号 中断该信号处理程序。 如果longjmp 跳出 信号处理程序，那么 此进程的 信号屏蔽字 会发生什么？
。。信号处理程序 都longjmp。。。。。

POSIX.1 没有指定 setjmp，longjmp 对信号屏蔽字的作用，而是定义了 2个函数 sigsetjmp,siglongjmp。
在信号处理程序中，进行 非局部转移时 应当使用这2个函数
setjmp.h
```C
int sigsetjmp(sigjmp_buf env, int savemask);
void siglongkmp(sigjmp_buf env, int val);
```

和 setjmp,longjmp 之间的唯一区别是， sigsetjmp 增加了一个参数，如果savemask 非0，则 sigsetjmp 在env 中保存进程的 当前信号屏蔽字。 siglongjmp 恢复

## sigsuspend


## abort
stdlib.h
`void abort(void);`

将 SIGABRT 信号发送给 调用进程 (进程不应该忽略这个信号)。
ISO C 规定，调用 abort 将向主机环境 递送一个 未成功终止的 通知，其方法是调用 raise(SIGABRT) 函数


## 函数system

POSIX.1 要求 system 忽略 SIGINT，SIGQUIT， 阻塞SIGCHLD



## sleep,nanosleep,clock_nanosleep

unistd.h
`unsigned int sleep(unsigned int seconds);`
返回0 或未休眠完的秒数

使得进程被挂起，直到
- 已经过了 seconds 所指定的墙上时钟时间
- 调用进程捕捉到一个信号 并从信号处理程序返回。


time.h
`int nanosleep(cosnt struct timespec *reqtp, struct timespec *remtp);`

time.h
`int clock_nanosleep(clockid_t clock_id, int flags, const struct timespec *reqtp, struct timespec *remtp);`


## sigqueue



## 作业控制信号
POSIX.1 认为有6个与作业控制有关
- SIGCHLD，子进程已停止或终止
- SIGCONT，如果进程已停止，则使其继续运行
- SIGSTOP，停止信号(不能被忽略或捕捉)
- SIGTSTP，交互式停止信号
- SIGTTIN，后台进程组成员 读控制终端
- SIGTTOU，后台进程组成员 写控制终端


## 信号名和编号

信号编号 与信号名之间映射
`extern char *sys_siglist[];`

signal.h
`void psignal(int signo, const char *msg);`

signal.h
`void psiginfo(const sigingo_t *info, cosnt char *msg);`

string.h
`char *strsignal(int signo);`

Solaris提供一对函数，
signal.h
```C
int sig2str(int signo, char *str);
int str2sig(const char *str, int *signop);
```



# ch11 线程

## 11.2 线程概念
典型的UNIX进程 可以看做只有一个 控制线程：一个进程在某一时刻只能做一件事情。

即使程序运行在 单处理器上，也能得到 多线程编程模型的 好处。

每个线程都包含有 表示执行环境所必须的信息，包括，==进程中标识线程的线程ID，一组寄存器值，栈，调度优先级和策略，信号屏蔽字，errno变量 及 线程私有数据。==

==一个进程的所有信息 对 该进程的所有线程 都是共享的，包括 可执行程序的代码，程序的全局内存和堆内存，栈 和 文件描述符==

我们将要讨论的线程接口来自 POSIX.1-2001， 也被称为 pthread


## 11.3 线程标识

线程ID只有在 它所属的进程上下文中 才有意义

进程ID通过 pid_t 数据类型表示， 非负整数
线程ID通过 pthread_t 表示，实现的时候可以用一个 struct 来表示，所以 不能当做 整数处理。 需要使用函数 来比较 2个线程ID 
pthread.h
`int pthread_equal(pthread_t tid1, pthread_t tid2);`

linux 3.2.0 使用 无符号长整型 表示 pthread_t 数据类型。

线程通过 pthread_self 获得自身的线程ID
pthread.h
`pthread_t pthread_self(void);`


## 11.4 线程创建
pthread.h

`int pthread_create(pthread_t *restrict tidp, const pthread_attr_t *restrict attr, void *(*start_rtn)(void *), void *restrict arg);`
成功返回0，否则返回错误编号

成功后，线程ID 被设置到 tidp 中。
新线程 从 start_rtn 函数的地址开始运行。


## 11.5 线程终止

`void pthread_exit(void *rval_ptr);`

`int pthread_join(pthread_t thread, void **rval_ptr);`

`int pthread_cancel(pthread_t tid);`

线程可以安排它退出时 需要调用的函数，这和 进程在退出时 调用 atexit 函数 类似。
一个线程可以有 多个清理退出程序，执行顺序和 注册顺序相反

```C
void pthread_cleanup_push(void (*rtn)(void *), void *arg);
void pthread_cleanup_pop(int execute);
```

线程函数 和 进程函数 的相似
![e9a29b3d576230ca6be8c567203c90d3.png](../_resources/e9a29b3d576230ca6be8c567203c90d3.png)


`int pthread_detach(pthread_t tid);`



## 11.6 线程同步

对变量+1，通常分为3步
1. 从内存单元读入到寄存器
2. 在寄存器中对变量进行增加
3. 新值写回内存单元


### 互斥量 mutex

互斥量 使用 pthread_mutex_t 数据类型表示的。 
使用前，必须初始化。 可以设置为常量 PTHREAD_MUTEX_INITIALIZER(只适合于静态分配的互斥量)， 也可以通过 pthread_mutex_init 进行初始化， 如果是 动态分配的互斥量 (如，通过malloc函数)，在释放内存前需要调用 pthread_mutex_destory

```C
// pthread.h
int pthread_mutex_init(pthread_mutex_t *restrict mutex, const pthread_mutexattr_t *restrict attr);

int pthread_mutex_destory(pthread_mutex_t *mutex);
```

互斥量加解锁
```C
int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_trylock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```
trylock 不会阻塞。 能lock就返回0， 不能lock就返回 EBUSY


### 避免死锁
如果线程试图 对一个互斥量 加锁2次， 那么它自身就会 死锁。
。。非 可递归的。

### pthread_mutex_timedlock
等于 pthread_mutex_lock，不过有时间限制，超过后会返回 ETIMEDOUT，而不是一直等待

`int pthread_mutex_timedlock(pthread_mutex_t *restrict mutex, const struct timespec *restrict tsptr);`

这个时间是 绝对值，就是 在 几点几分几秒前 等待， 而不是 等待几分钟。

### 读写锁

读写锁使用前 必须初始化，释放底层的内存之前必须销毁。
```C
int pthread_rwlock_init(pthread_rwlock_t *restrict rwlock, const pthread_rwlockattr_t *restrict attr);
int pthread_rwlock_destroy(pthread_rwlock_t *rwlock);
```

```C
int pthread_rwlock_rdlock(pthread_rwlock_t *rwlock);
int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock);
int pthread_rwlock_unlock(pthread_rwlock_t *rwlock);
```

Single UNIX Specification 定义了 读写锁原语的条件版本
```C
int pthread_rwlock_tryrdlock(pthread_rwlock_t *rwlock);
int pthread_rwlock_trywrlock(pthread_rwlock_t *rwlock);
```

### 带有超时的读写锁
```C
#include <pthread.h>
#include <time.h>

int pthread_rwlock_timedrdlock(pthread_rwlock_t *restrict rwlock, const struct timespec *restrict tsptr);

int pthread_rwlock_timedwrlock(pthread_rwlock_t *restrict rwlock, const struct timespec *restrict tsptr);
```

时间是 绝对值。


### 条件变量
条件 本身由 互斥量保护。线程在改变条件的状态之前，必须锁住互斥量。

初始化和销毁
可以用 PTHREAD_COND_INTIALIZER 给 静态分配的 条件变量 赋值。
```C
int pthread_cond_init(pthread_cond_t *restrict cond, const pthread_condattr_t *restrict attr);
int pthread_cond_destory(pthread_cond_t *cond);
```

使用 xxx_wait 等待条件变量变成真
```C
int pthread_cond_wait(pthread_cond_t *restrict cond, pthread_mutex_t *restrict mutex);
int pthread_cond_timedwait(pthread_cond_t *restrict cond, pthread_mutex_t *restrict mutex, const struct timespec *restrict tsptr);
```
时间是绝对值

调用者把 ==锁住的互斥量== 传给 函数，函数自动把 调用线程 放到 等待条件的 线程列表上，==对互斥量解锁==。 xxx_wait 返回时，==互斥量再次被锁住==。


2个函数用于通知线程 条件已满足
xxx_signal 至少唤醒一个
xxx_broadcast 唤醒所有线程

```C
int pthread_cond_signal(pthread_cond_t *cond);
int pthread_cond_broadcast(pthread_cond_t *cond);
```

### 自旋锁
在用户层，自旋锁 不是非常有用。 线程会被休眠。

很多互斥量的实现非常高效，以至于 互斥锁 的 自旋锁的 性能基本相同。
实际上，有些互斥量的 实现 在 试图获取 互斥量的时候 会 自旋一小段时间。
加上现代CPU的进步，使得上下文 切换越来越快， 使得 自旋锁 只有某些特定情况 才有用。

初始化，销毁
```C
int pthread_spin_init(pthread_spinlock_t *lock, int pshared);
int pthread_spin_destroy(pthread_spinlock_t *lock);
```

```C
int pthread_spin_lock(pthread_spinlock_t *lock);
int pthread_spin_trylock(pthread_spinlock_t *lock);
int pthread_spin_unlock(pthread_spinlock_t *lock);
```


### 11.6.8 barrier屏障

屏障是用户协调多个线程并行工作的同步机制。
屏障允许每个线程等待，直到所有合作线程都到达某一点，然后从该点继续执行。
pthread_join 就是一种屏障，允许一个线程等待，直到另一个线程退出。

初始化，销毁
```C
int pthread_barrier_init(pthread_barrier_t *restrict barrier, const pthread_barrierattr_t *restrict attr, unsigned int count);
int pthread_barrier_destroy(pthread_barrier_t *barrier);
```
count 表示 允许所有 线程继续运行之前，必须到达屏障的 线程数目。


到达，等待其他线程
`int pthread_barrier_wait(pthread_barrier_t *barrier);`

屏障一旦打开，就一直打开。除非再 destroy然后 init 



# ch12 线程控制

## 线程限制
可以通过 sysconf查询

![bcf8efe0eacd6560368cb8d4b5651225.png](../_resources/bcf8efe0eacd6560368cb8d4b5651225.png)

![229044f807b21f08dd42f7ce9716ffe2.png](../_resources/229044f807b21f08dd42f7ce9716ffe2.png)



## 线程属性

pthread 允许我们通过设置 每个对象关联的不同属性来 调整 线程和同步对象的行为。

```C
int pthread_attr_init(pthread_attr_t *attr);
int pthread_attr_destory(pthread_attr_t *attr);
```


![556ce81f504e28ff50e3e07c3a69dedd.png](../_resources/556ce81f504e28ff50e3e07c3a69dedd.png)



```C
int pthread_attr_getdetachstate(const pthread_attr_t *restrict attr, int *detachstate);
int pthread_attr_setdetachstate(pthread_attr_t *attr, int *detachstate);
```

```C
int pthread_attr_getstack(const pthread_attr_t *restrict attr, void **restrict stackaddr, size_t *restrict stacksize);
int pthread_attr_setstack(pthread_attr_t *attr, void *stackaddr, size_t stacksize);
```


```C
int pthread_attr_getstacksize(const pthread_attr_t *restrict attr, size_t *restrict stacksize);
int pthread_attr_setstacksize(const pthread_attr_t *attr, size_t stacksize);
```

```C
int pthread_attr_getguardsize(const pthread_attr_t *restrict attr, size_t *restrict guardsize);
int pthread_attr_setguardsize(pthread_attr_t * attr, size_t guardsize);
```


## 12.4 同步属性
线程的同步对象也有属性。

p 345

### 互斥量属性

### 读写锁属性

### 条件变量属性

### 屏障属性


## 12.5 重入

函数重入 和 信号处理程序 是类似的，多个控制线程在相同的时间 有可能 调用相同的函数。

如果一个函数在相同的时间点 可以被 多个线程 安全地调用，那么 这函数是 线程安全的。

![975d4d6e63f2803f39d364751abc6841.png](../_resources/975d4d6e63f2803f39d364751abc6841.png)


函数的线程安全版本，是在原函数名 后面 加 _r， 代表可重入

![9188723aba358301941ed8d5c2b60780.png](../_resources/9188723aba358301941ed8d5c2b60780.png)



POSIX.1 提供了 以线程安全的方式 管理FILE对象的方法
```C
// stdio.h
int ftrylockfile(FILE *fp);
void flockfile(FILE *fp);
void funlockfile(FILE *fp);
```
这里的 锁 是递归的。

如果标准IO例程都获取它们各自的锁，那么在做 一次一个char 的IO时 会出现严重的 性能下降，因为需要 对每个char 的读写 进行 加锁，解锁。
为了避免这种开销，提供了 不加锁的 基于字符 的标准IO例程
```C
// stdio.h

int getchar_unlocked(void);
int getc_unlocked(FILE *fp);
int putchar_unlocked(int c);
int putc_unlocked(int c, FILE *fp);
```
除非 被 flockfile, funlockfile 包围， 否则不要使用 这4个函数， 因为 会导致 未定义行为


## 12.6 线程特定数据
也称为 线程私有数据

p358

。。threadlocal

`pthread_key_create`
`pthread_key_delete`

`pthread_once`

`pthread_getspecific`
`pthread_setspecific`


## 12.7 取消选项

`pthread_setcancelstate`

pthread_cancel并不等待线程终止。 被取消的线程在 取消请求发出后 继续运行，直到线程 到达某个 取消点。

![a38fd7cc8c66f72c9e3168c3d5c5b500.png](../_resources/a38fd7cc8c66f72c9e3168c3d5c5b500.png)


`pthread_testcancel`

![a028e8c76d277da7b6a95fb2ad99d530.png](../_resources/a028e8c76d277da7b6a95fb2ad99d530.png)


`pthread_setcanceltype`


## 12.8 线程和信号

每个线程都有自己的 信号屏蔽字
信号的处理 是 进程中 所有线程 共享的。


signal.h
`pthread_sigmask`
`sigwait`
`pthread_kill`


## 线程 和 fork

当线程 调用 fork时，就 为 子进程 创建了 整个进程地址空间的副本。
8.3 写时复制，子进程和父进程是完全不同的进程，只要2者没有对内存内容进行改动，那么会共享 内存页。

子进程 继承了整个地址空间的副本， 还从父进程那里 继承了 每个互斥量，读写锁，条件变量的 状态。
如果父进程包括 一个以上的线程，子进程在 fork返回后，如果不是 马上调用 exec的话，就需要 ==清理锁状态。==

在子进程内部，只存在一个线程，它是由父进程中调用 fork的线程 的副本构成的。如果父进程的 线程 占有锁，子进程也将 占有锁。
问题是，子进程 并不包含 占有锁的线程的副本，所以 子进程 无法知道 它占有了 哪些锁，需要释放哪些锁。

如果子进程 fork后 立刻 exec，就可以避免这些问题，这种情况下， 旧的地址空间被丢弃了。

在多线程的进程中，为了避免不一致状态的问题，POSIX.1声明，在fork 返回 和 子进程 调用 exec 之间，子进程 只能调用 异步信号安全的函数。

要清除锁状态，可以通过 pthread_atfork 
```C
// pthread.h
int pthread_atfork(void (*prepare) (void), void (*parent) (void), void (*child) (void));
```
最多install 3个清理锁的函数。
prepare 由父进程在 fork 创建子进程之前调用，这个函数的 任务是 获得 父进程定义的所有锁。
parent 是在 fork 创建子进程后，返回之前，在 父进程上下文中调用，对 prepare 中获得的 所有锁 进行解锁。(释放的是父进程的锁)
child 是在 子进程上下文中调用，也是释放 prepare 获得的锁。


## 线程与IO

3.11 的 pread，pwrite 在多线程环境下 非常有用，因为进程中所有线程 共享相同的文件描述符。


# ch13 守护进程

在 系统引导装入时 启动，系统关闭时 终止。
没有控制终端

本章将说明 守护进程结构，如何编写守护进程程序，守护进程如何报告出错。

`ps -axj`




# ch14 高级IO
非阻塞IO，记录锁，IO多路复用，异步IO，readv，writev，mmap

## 14.2 非阻塞IO

非阻塞IO使我们可以发出 open,read,write 这样的 IO操作，并且这些操作不会永远阻塞。 如果操作无法完成，则调用立刻出错返回(。。就是返回值是 -1，设置errno)。

对于一个给定的 fd，有2种 指定为 非阻塞IO的方法
- 如果调用 open 获得 fd，则可以指定 O_NONBLOCK 标志
- 如果fd已经打开，则可以调用 fcntl，来设置 O_NONBLOCK标志。


## 14.3 记录锁
2个人同时编辑一个文件，后果将如何？
大多数unix系统中，文件的最终状态 取决于 写该文件的最后一个进程。
但是对于有些应用，比如数据库，需要确保 它单独写一个文件，为了 向进程提供这种功能，商用UNIX 提供了 记录锁机制。

record locking 的功能是： 当第一个进程 正在 读或写 文件的某个部分时，使用记录锁 可以阻止其他进程 修改同一文件区
一个更合适的术语可能是 字节范围锁 (byte-range locking)

![34b33d3f1660e5aaa62fcb9f38ff8047.png](../_resources/34b33d3f1660e5aaa62fcb9f38ff8047.png)


fcntl记录锁
`int fcntl(int fd, int cmd, ... /* struct flock *flockptr */);`

对于记录锁，cmd 是 F_GETLK，F_SETLK，F_SETLKW
flockptr 是指向 flock 的指针
```C
struct flock {
    short l_type;   // F_RDLCK, F_WRLCK, F_UNLCK
    short l_whence; // SEEK_SET, SEEK_CUR, SEEK_END
    off_t l_start;  // offset, 和l_whence有关
    off_t l_len;    // byte 长度，0表示锁到 EOF
    pid_t l_pid;
};
```


锁的隐含继承和释放
关于记录锁的自动继承和释放有3条规则
1. 锁与进程和文件两者相关联，即，进程终止时，它的所有锁立即释放；fd关闭时，这个进程设置的锁也会释放。
2. 由fork产生的子进程 不继承 父进程设置的锁。即，若一个进程得到一把锁，然后调用fork，那么对于父进程获得的锁而言，子进程被视为 另一个进程。子进程需要自己去获得锁。
3. exec后，新程序可以继承 原执行程序的 锁。但注意，如果对一个 fd设置了 执行时关闭，那么 exec 会 关闭该fd，并释放对应文件的 所有锁。



FreeBSD实现

![fc78e709c43f6c877bd413570649da51.png](../_resources/fc78e709c43f6c877bd413570649da51.png)



### 在文件尾加锁

。。挺繁的。。
p398

。。
加锁，append，解锁。 append 的依然有锁的。 append之前的 没有锁了
加锁的时候要用 SEEK_END，不然append的数据 是没有锁的。
。。


建议性锁 和强制性锁
。。



## 14.4 IO多路复用

> 从一个fd读，然后写到另一个fd

可以用如下的 阻塞IO
```C
while ((n = read(STDIN_FILENO, buf, BUFSIZ)) > 0)
    if (write(STDOUT_FILENO, buf, n) != n)
        err_sys("error");
```
这种形式的阻塞IO到处可见。

但如果要从 2个 fd 读，会如何呢？在这种情况下，我们不能在任一 fd 上进行 阻塞读，否则会 阻塞另一个fd上的读操作。

- 2个进程
  无法知道操作什么时候停止。因为只有 子进程 退出时 会 发信号到 父进程，父进程退出，子进程是无法感知的。

- 2个线程
  需要进行同步，增加了复杂性。

- 非阻塞IO
  即轮询，浪费CPU

- 异步IO
  利用这种技术，进程告诉内核，当 fd 准备好 进行IO时，用一个信号通知它。这种技术有2个问题，1.可移植性差， 2.信号对于进程而言只有一个(SIGPOLL 或 SIGIO)，无法对2个fd起效，进程无法知道 是哪个 fd 可读。

- IO多路复用
  先构建一张我们感兴趣的 fd列表，然后调用一个函数，直到 这个列表中 有 一个fd准备好IO时，这个函数才返回。 poll,pselect,select 可以执行 IO多路复用。

### select,pselect
传给select的参数告诉内核：
- 我们关系的 fd
- 对于每个fd 我们关系的条件 ( 关心读，还是写，还是异常条件)
- 愿意等待多长时间 ( 永远，固定时间，不等待)

select返回时，内核告诉我们
- 已经准备好的 fd 的总量
- 对于 读，写，异常， 这3个条件中的每一个，哪些描述符已经准备好。

```C
#include <sys/select.h>

int select(int maxfdp1, fd_set *restrict readfds, fs_set *restrict writefds, fs_set *restrict exceptfds, struct timeval *restrict tvptr);
```

maxfdp1的意思是 最大fd编号+1， 也可以设置为 FD_SETSIZE (最大描述符编号)，前者最好，应该可以让 内核 缩小寻找范围， FD_SETSIZE经常是1024，太大了，大多数app只使用3-10个fd。

tvptr 是愿意等待的时间长度，单位为 秒和微秒 (4.20节)， 有3种情况
- NULL，永远等待，如果捕捉到信号则中断此无限等待。如果捕捉到信号，返回-1，errno设置为 EINTR
- tv_sec == 0 && tv_usec == 0，不等待，可以用于轮询
- tv_sec != 0 || tv_usec != 0，等待指定的秒数 和微秒数。可以被捕捉到的信号中断。

readfds,writefds,exceptfds 是指向 fd集 的指针。
fd_set 数据类型 是 由 实现选择的。 我们可以认为它是一个很大的字节数组，为每个fd 保留一个 bit

对于fd_set 可以进行的操作：分配一个这种类型的变量，将这种类型的变量值 赋给 另一个变量，或使用下面的 4个函数之一
```C
#include <sys/select.h>

int FD_ISSET(int fd, fd_set *fdset);
int FD_CLR(int fd, fd_set *fdset);
int FD_SET(int fd, fd_set *fdset);
void FD_ZERO(fd_set *fdset);    // 所有bit 置0
```

select 的 3个 fd_set * 参数 可以都是 空指针。 ==如果3个指针都是 NULL，那么 select 提供了 比sleep 更精确的 定时器==

当fd上是 文件尾端，则select 认为该fd是可读的，调用read时，返回0。

POSIX.1 也定义了一个 select的变体，pselect
```C
#include <sys/select.h>

int pselect(int maxfdp1, fs_set *restrict readfds, fs_set *restrict writefds, fs_set *restrict exceptfds, const struct timespec *restrict tsptr, const sigset_t *restrict sigmask);
```
pselect,select 的区别：
- 超时，select用 timeval (秒，微秒)， pselect用 timespec，timespec使用秒和纳秒
- pselect 的tsptr 是 const， 保证 pselect 不会修改这个值
- pselect 可以选择 信号屏蔽字。 sigmask 为NULL，那么在 信号方面，pselect，select 是一样的。 如果指定了 信号屏蔽字，那么 调用 pselect时，以原子方式 安装 该信号屏蔽字，返回时，恢复以前的 信号屏蔽字


### 14.4.2 poll
类似于 select，但是 接口不同。
poll可以用于 任何类型的 fd
```C
#include <poll.h>

int poll(struct pollfd fdarray[], nfds_t nfds, int timeout);

struct pollfd {
    int fd;
    short events;   // 感兴趣的事件
    short revents;  // 发生的事件
};
```

poll不是为 每个条件 (读，写，异常) 构建一个 fd集， 而是构造一个 pollfd 数组，数组中 每个 pollfd类型的 元素 指定了 一个fd 以及我们对该 fd 感兴趣的条件。

nfds是 fdarray中的元素数量。

![c3401b6aa0f1805d69eb6d293fdd3647.png](../_resources/c3401b6aa0f1805d69eb6d293fdd3647.png)

当一个 fd 为 挂断(POLLHUP) 之后，就不能再 写 该fd。 但是有可能 从这个fd 读取到数据。

timeout是愿意等待多长时间，和select一样有3种可能
- -1，永远等待
- 0，不等待， 可以用于轮询
- `>0`，等待 timeout 毫秒

和select一样，fd是否阻塞 不影响 poll是否阻塞

---
如果信号可能会中断 select 或 poll，就要使用 signal_intr 函数



## 14.5 异步IO

在用异步IO的时候，要通过 选择 来灵活处理 多个并发操作，这会使得 app复杂化。
更简单的做法是 使用 多线程 异步，每个线程 是同步IO。

使用POSIX 异步IO，会有下列麻烦
- 每个异步操作 有 3处可能发生错误的地方：操作提交的部分，操作本身的结果，决定异步操作状态的函数中
- 与 POSIX 异步IO接口 的传统方法相比，它们本身涉及大量 额外设置 和 处理规则。
- 从错误中恢复 比较困难


### System V 异步IO

在 System V 中，异步IO时 streams系统的一部分，它只对 streams 设备 和 streams管道 起作用。
System V 的异步IO信号是 SIGPOLL。

对一个 streams 设备启动异步 IO，需要调用 ioctl，将它的第二个参数(request) 设置为 I_SETSIG。 第三个参数设置为 下表的 一个或多个 常量构成的 整数值 (在 stropts.h中定义)

与 streams 机制相关的接口 在 SUSv4中已经 标记为 废弃。

。。。那我抄什么。。


### BSD异步IO
BSD派生的系统中，异步IO 是信号 SIGIO 和 SIGURG的组合。
SIGIO是通用异步IO信号， SIGURG 只用来通知进程 网络连接上的 带外数据已经到达

为了接受SIGIO信号，需要执行以下3步
- 调用 signal 或 sigaction 为 SIGIO 信号建立 信号处理程序
- 以命令 F_SETOWN 调用 fcntl 来设置 进程ID 或 进程组ID， 用于接收该fd 的信号
- 以命令 F_SETFL 调用 fcntl设置 O_ASYNC 文件状态标志，使在该描述符上可以进行 异步IO


### POSIX 异步IO

为不同类型的文件 进行异步IO 提供一套一致的方法。
所有平台都被要求 支持这些接口

异步IO接口 使用 AIO控制块 来 描述 IO操作，aiocb 定义了 AIO控制块， 至少包括 下列字段
```C
struct aiocb {
    int aio_fildes;     // fd
    off_t aio_offset;   // file offset for IO
    volatile void *aoi_buf; // buffer for IO
    size_t aio_nbytes;  // 准备读/写的字节数
    int aio_reqprio;    // 优先级
    struct sigevent aio_sigevent;   // signal information
    int aio_lio_opcode; // operation for list IO
}
```
异步IO必须显式指定 偏移量。 异步IO接口 不影响 OS维护的 文件偏移量。 只要不在 一个进程里 把 异步IO函数 和 传统IO函数 混在一起用在同一个文件上，就不会导致什么问题。
当异步IO接口向一个 追加模式 打开的文件 写入数据时，AIO控制块中的 aio_offset 字段会被忽略。

```C
struct sigevent {
    int sigev_notify;
    int sigev_signo;
    union sigval sigev_value;
    void (*sigev_notify_function) (union sigval);
    pthread_attr_t *sigev_notify_attributes;
};
```

进行异步IO前，要先初始化AIO控制块，调用 aio_read 来进行异步读操作，或 aio_write 进行异步写操作
```C
#include <aio.h>
int aio_read(struct aiocb *aiocb);
int aio_write(struct aiocb *aiocb);
```

要强制所有等待中的 异步操作不等待 而写入持久化存储中：
```C
int aio_fsync(int op, struct aiocb *aiocb);
```

为了获得一个 异步读，写，或同步操作的 完成状态，需要调用 aio_error
```C
int aio_error(cosnt struct aiocb *aiocb);
```

如果异步操作成功，则调用 aio_return 来获得异步操作的返回值
```C
ssize_t aio_return(cosnt struct aiocb *aiocb);
```

阻塞进程自己， 等待 异步操作完成
`int aio_suspend(const struct aiocb *const list[], int nent, const struct timespec *timeout);`

取消异步IO操作
`int aio_cancel(int fd, struct aiscb *aiocb);`


lio_listio 即可以异步方式，又可以同步方式。 它提交一系列 由一个AIO控制块列表 描述的 IO请求
`int lio_listio(int mode, struct aiocb *restrict const list[restrict], int nent, struct sigevent *restrict sigev);`



## 14.6 readv,writev

用于在一次函数调用中 读，写 多个非连续缓冲区。
有时也将这2个函数 称为 散布读(scatter read)，聚集写(gather write)

```C
#include <sys/uio.h>

ssize_t readv(int fd, const struct iovec *iov, int iovcnt);
ssize_t writev(int fd, const struct iovec *iov, int iovcnt);

struct iovec {
    void *iov_base;
    size_t iov_len;
};
```


## readn,writen
管道，FIFO，某些设备(特别是 终端和网络) 有下列 2种性质
- 一次read 所返回的数据 可能少于 所要求的数据，即使没有达到文件尾。 这不是错误，应当继续 读 该设备
- 一次write的返回值 也可能少于 指定输出的字节数。可能是由于 内核输出缓冲区满。这不是错误，应当继续 写 剩下的数据。(通常，只有 非阻塞fd，或捕捉到信号时，才会发生 这种 write的中途返回)

在读写磁盘文件时，从未遇到这种情况，除非 文件系统用完了空间，或者接近配额限制，导致无法 将 写数据 全部写出

通常，在 读 写 一个管道，网络设备 或终端时，需要考虑上面的特性。
下面的函数 是读写 N个字节， 如果不满，会自动 再次 调用 read ，write 直到满 N 字节。
```C
ssize_t readn(int fd, void *buf, size_t nbytes);
ssize_t writen(int fd, void *buf, size_t nbytes);
```


## 14.8 存储映射IO

存储映射IO (memory-mapped IO) 能将 一个磁盘文件 映射到 存储空间 中的一个 缓冲区上，当从 缓冲区中取数据时，就相当于读文件中的 相应字节。 向缓冲区写入时，就相当于 写入到文件。 这样就可以在不使用read write 的情况下进行IO。

为了使用这个功能， 应首先告诉内核 将一个 文件映射到 存储区域。
```C
#include <sys/mman.h>

void *mmap(void *addr, size_t len, int prot, int flag, int fd, off_t off);
```

addr 指定 映射存储区的 (建议，受flag影响) 起始位置，通常设置为0，表示由 系统选择。
函数的返回值 是 该映射区的起始地址

fd表示 被映射文件的 fd， 在 文件映射到地址空间之前，必须先打开该文件。
len 是映射的字节数
off 是 映射的字节在文件中的 起始偏移量

prot 指定了映射存储区的 保护要求， 
- PROT_READ，映射区可读
- PROT_WRITE，映射区可写
- PROT_EXEC，映射区可执行
- PROT_NONE，映射区 不可访问
可以指定为 PROT_NONE， 或另外3个的 或。
对于映射存储区的保护 不能超过 文件 open 模式的访问权限。

flag 影响映射存储区的多种属性
- MAP_FIXED，返回值必须等于 addr，因为这不利于可移植性，所以不鼓励使用。 如果未指定这个标记，且 addr 非0，则内核把 addr 视为 一种建议。
- MAP_SHARED，描述了 本进程 对映射区 所进行的 存储操作的配置。此标志指定存储操作修改映射文件。也就是，存储操作相当于 对该文件 write。
- MAP_PRIVATE，对映射区的存储操作 导致创建 该映射文件的 一个私有副本。所有后来的对该映射区的引用 都是 引用这个副本。


下图的起始地址 是mmap的 返回值
映射存储区位于 堆和栈之间，这属于实现细节，各种实现之间可能不同

![084b338f19fd11ce55ccfac0e54bbb7a.png](../_resources/084b338f19fd11ce55ccfac0e54bbb7a.png)

每种实现 都可能还有一些 MAP_xxx 标志值，具体查看 mmap(2) 手册页

off，addr (如果指定了 MAP_FIXED) 通常被要求是 系统虚拟存储页 长度的 倍数。
虚拟存储页长可以用 带参数 _SC_PAGESIZE 或 _SC_PAGE_SIZE 的 sysconf 函数 得到。 由于 off，addr 通常都是0，所以这种要求一般并不重要。

如果映射区的长度不是页长的倍数时，会怎么样？
假定文件长 12字节，系统页长 512字节，则系统提供 512字节的映射区，其中 后500字节被设置为0. 可以修改后500字节，但是 任何变动 都不会 在文件中 反应出来。 所以 mmap 不能用于 将数据 append 到文件。 我们必须先加长文件。


与映射区有关的信号有 SIGSEGV，SIGBUS
SIGSEGV 通常表示 程序 试图 访问 对它不可用的存储区， 如果 映射存储区 被mmap指定为 只读，那么 试图 存入数据 就会 产生此信号
SIGBUS，映射区的某个部分在访问时已不存在。

子进程能通过 fork 继承 存储映射区， 因为子进程 复制了 父进程地址空间， 映射存储区 是 地址空间的一部分。  exec不能继承映射存储区

改变现有映射的权限
`int mprotect(void *addr, size_t len, int prot);`

msync将该页flush到文件中
`int msync(void *arrd, size_t len, int flags);`

进程终止时，会自动解除 映射存储区的映射， 也可以直接调用 munmap 解除 映射存储区
`int munmap(void *addr, size_t len);`

munmap 并不影响被映射的对象，即，munmap不会 flush映射区的内容 到 磁盘文件上。


# ch15 进程间通信

第八章中，进程间 交换信息 的唯一途径是 传送打开的文件，可以经由 fork 或 exec来传送，也可以通过 文件系统来传送。

![a01923410c396e117841980daedc2054.png](../_resources/a01923410c396e117841980daedc2054.png)

UDS表示 经由UNIX域套接字(socket)支持。


## 15.2 管道

UNIX系统IPC 的最古老形式。 有2种局限
- 历史上，它是 半双工的 (即数据只能在一个方向上流动)。 现在，某些OS提供了 全双工 管道，但为了移植性，我们不能假设 OS支持 全双工管道
- 管道 只能在 具有 公共祖先的 2个进程之间使用。通常一个管道 由一个进程创建，在进程 fork 之后，这个管道就可以在 父进程 和 子进程之间使用了

15.5的 FIFO 没有第二种局限。
17.2 的 socket 没有这2种局限。

尽管有这2个局限， 半双工管道 依然是 最常用的 IPC。
每当在管道中 键入一个命令序列，让shell执行时，shell都会为每一条命令 单独创建一个进程，然后用 管道将前一条命令进程的 标准输出 与 后一条命令的 标准输入 相连接。

创建管道
unistd.h
`int pipe(int fd[2]);`
fd[0]为读而打开，fd[1]为写而打开。fd[1]的输出是 fd[0]的输入。
。。意思是，进程1 往 fd[1] 写入，数据会通过管道发送到fd[0], 然后 进程2 从 fd[0] 读取

。。我看示例，直接 `int fd[2]; pipe(fd)`  就是 fd[2] 里是空的。

POSIX.1 允许实现支持全双工管道，对于这些实现，fd[0] 和 fd[1] 以 读写方式打开。

fstat 函数 对管道的每一段 都返回一个 FIFO类型的 fd，可以用 S_ISFIFO 宏来测试管道。

通常进程会==先调用 pipe，然后 fork==， 从而创建 父进程， 子进程 之间的 IPC管道。

fork之后做什么 取决于 我们想要的 数据流的方向。
如果要 父进程发往子进程，那么 ==父进程 关闭 管道的读(fd[0])，子进程关闭写(fd[1])==。

当管道的一端被关闭后，下列2条规则起效
- 当read 一个写端已关闭的 管道，在所有数据都被读取后，read 返回0，表示文件结束 (。这个是真的关闭了，是 另一个进程中 写端已关闭， 不是 fork 的时候 自己关闭的 自己的写端)
- 如果 write 一个 读端已关闭的管道，产生 信号 SIGPIPE，如果忽略该信号 或 捕捉该信号 并从其处理程序返回，则write 返回-1，errno设置为 EPIPE

在写管道 (或 FIFO) 时，常量 PIPE_BUF 规定了 内核的 管道缓冲区大小。 
对管道 write时，写得字节数 小于等于 PIPE_BUF，则此进程 不会和 其他进程 对 同一管道(或FIFO) 的write 操作 交叉进行。 
如果 多个进程 同时写 一个管道(或FIFO)，而且我们写得字节数超过 PIPE_BUF，那么 数据 可能和 其他进程 写得数据 相互交叉。
用 pathconf, fpathconf 可以确定 PIPE_BUF的值。



## popen,pclose

常见操作是 创建一个连接到 另一个进程的管道，然后 读 其输出 或 向其输入端 发送数据，为此，标准IO提供了 2个函数，用来：创建一个管道，fork一个子进程，关闭未使用的管道端，执行一个 shell 运行命令，然后等待命令终止

```C
// stdio.h

FILE *popen(const char *cmdstring, const char *type);
int pclose(FILE *fp);
```
。。就是 std::async ?

popen 先执行 fork，然后 调用 exec 执行 cmdstring， 并且返回一个 标准IO文件指针。
如果type是 r，则 返回的 文件指针 是可读的， 如果是 w，是可写的。

pclose 关闭标准IO流，等待命令终止，然后返回 shell的终止状态。如果shell不能执行，则 pclose 返回的终止状态 和 shell 执行exit(127) 一样。

cmdstring 由以下方式执行
`sh -c cmdstring`


## 协同进程 coprocess
unix 使用 过滤程序 从标准输入读，向标准输出写。 几个 过滤程序 通常在 shell管道中 线性连接。 
当一个过滤程序既 产生 某个过滤程序的输入，有读取该过滤程序的输出时，它就变成了 协同进程

popen只提供了 连接到 另一个进程的 标准输入 或标准输出的 一个 单向管道
协同进程，则有 连接到到 另一个进程的 2个 单向管道：一个 接到 标准输入，一个来自其 标准输出。  我们想 ==将 数据 写到其标准输入，经其处理后，再从其标准输出 读取数据==
。。就是 调用 方法， 异步的。  没有专门的函数


## 15.5 FIFO
有时被称为 命名管道。
未命名的管道 只能在 2个 相关的进程间使用，并且 这2个相关的进程 还有一个 共同的创建了 它们的祖先进程。

通过FIFO，不相关的 进程 也可以交换数据

创建 FIFO 类似 创建文件。 确实，FIFO的路径名 保存在 文件系统中。
```C
// sys/stat.h

int mkfifo(const char *path, mode_t mode);
int mkfifoat(int fd, const char *path, mode_t mode);
```
mode 和 3.3的open的mode 相同。

mkfifoat 可以在 fd 的目录位置创建一个 FIFO， 和其他 *at 方法一样，有3种情况
- path是 绝对路径，fd被忽略，此时 和 mkfifo 一样
- path是 相对路径，fd是一个 打开目录的 有效的 fd，路径名 和 目录有关
- path是 相对路径， fd有 特殊值 AT_FDCWD，则路径名 以当前目录开始， 和mkfifo 雷系。

当我们用 mkfifo(at) 创建FIFO时，要用 open 来打开它。

当open一个FIFO时，O_NONBLOCK 会产生下列影响
- 没有指定O_NONBLOCK时，只读open 要阻塞到 某个其他进程 为写而打开 这个FIFO为止，类似地，只写open 要阻塞到 其他进程 为读 而打开它 为止。
- 如果指定了 O_NONBLOCK，则 只读open 立即返回。 如果没有其他进程 为读 而打开一个 FIFO，那么 只写 open 将返回 -1，并且 errno 为 ENXIO

类似于管道， 如果write 一个 没有进程为读 而打开的 FIFO，则产生信号 SIGPIPE。 如果某个 FIFO的 最后一个 写进程 关闭了 该FIFO，则为 该FIFO 的读进程产生一个 文件结束标志。

一个给定的 FIFO 有多个 写进程 是 常见的。这意味着，如果不希望 多个进程 所写的 数据交叉，必须考虑 原子写操作。 和管道一样， PIPE_BUF 说明了 可被原子写到 FIFO的 最大数据量。

FIFO有2种用途
- shell命令使用 FIFO将数据 从一条管道 传送到 另一条时，无需创建中间 临时文件
- 客户进程 - 服务器进程 应用程序中， FIFO 作为 汇聚点，在 客户进程 和 服务器进程间 传递数据。


使用 FIFO 将 一个流 发送到 2个不同的进程
```shell
mkfifo fifo1
prog3 < fifo1 &
prog1 < infile | tee fifo1 | prog2
```


使用FIFO进行 客户端进程-服务器进程通信

![5e6a49357de236da0374e02e071982c4.png](../_resources/5e6a49357de236da0374e02e071982c4.png)

这种安排可以工作，但是 服务器进程 无法判断 客户进程 是否已经 崩溃终止， 这使得 客户进程专用的FIFO 会遗留在 文件系统中。
另外，服务器进程 还需要 捕获 SIGPIPE信号， 因为客户端在发送请求后，没有读取响应 就终止了，于是留下了一个 只有写进程 无读进程的 客户进程专用 FIFO



## 15.6 XSI IPC

有3种 被称为 XSI IPC 的IPC： 消息队列，信号量，共享存储器。 它们之间有很多相似之处。


### 标识符和键

每个内核中的 IPC结构 ( 消息队列，信号量，共享存储段) 都用一个 非负整数 的标识符 加以引用。
例如，要向 一个消息队列 发送消息 或 从一个消息队列 取消息，只需要知道 其队列标识符。
和fd 不同， IPC标识符 不是 小的整数。 当一个 IPC结构被创建，然后又被删除时，与这种结构相关的 标识符 连续加1，直到 整型数的 最大值，然后 有回转到 0

标识符是 IPC对象的 内部名。 为了使 多个合作进程 能在 一个IPC对象上 汇聚，需要提供一个 外部命名方案， 为此，每个 IPC对象 都与 一个 键 相关联，将这个键 作为 该对象的 外部名。

无论何时创建 IPC (msgget,semget,shmget)。 都应该指定一个键。键的类型是 key_t

有多种方法 使 客户进程 和 服务器进程 在同一个 IPC结构上 汇聚。
1. 服务器进程 指定键 IPC_PRIVATE 创建一个 新IPC结构，将返回的 标识符存放在 某处(如一个文件) 让客户进程获取。 键IPC_PRIVATE 保证 服务器进程 创建一个 新IPC结构。 缺点是， 文件系统 操作需要 服务器进程 将 整型标识符 写到文件中，伺候客户进程 要读取文件 来获得 标识符
2. 可以在一个公用header文件 定义 客户进程 和 服务器进程 都任何的 键。然后 服务器进程 用这个 键 创建一个 新的IPC结构。 缺点是，该键 可能 已经和 一个IPC结构 结合，此时，get函数(msg,sem,shm get) 出错返回。 服务器必须处理这一错误，删除 已存在的 IPC结构，然后再 创建。
3. 客户进程 和服务器进程 认同 一个路径名 和 项目ID (ID 是 0-255之间的字符值)，调用 ftok 函数 将这2个值 变成一个 键， 然后在 方法2 中使用这个键。

```C
// sys/ipc.h

key_t ftok(const char *path, int id);
```
path参数必须 引用一个现有的文件，产生键时，只使用 id 的 低8位。
通常，ftok 按照 路径名 取得 stat结构中的 st_dev, st_ino， 将它们和 id 结合。
由于i节点编号和 键通常存放在 长整型中，所以 创建键时可能丢失信息，所以 对于不同文件的2个路径名，如果 使用同一个项目ID，那么可能产生 相同的键。

3个 get函数 有着 类似的参数： 一个key， 一个整型flag。
如果key 是 IPC_PRIVSTE 或者 和当前 某种类型的 IPC结构无关，则需要指明 flag的 IPC_CREAT 标志位。
为了 引用一个 现有队列 (通常是客户进程创建)，key必须等于 队列创建时 指明的key的值，并且 IPC_CREAT 不能被指明。

不能用 IPC_PRIVATE 作为键 来引用一个现有队列， 因为 这个键 总是 用于创建一个 新队列。

如果希望创建一个新的IPC结构，并且 要确保 没有 引用具有 同一标识符的 一个现有 IPC结构，就必须在 flag 中同时指定 IPC_CREAT 和 IPC_EXCL 位。 这样，如果IPC结构已存在，则出错，返回 EEXIST ( 这和 O_CREAT + O_EXCL 的open 类似)


### 权限结构

XSI IPC 为每个 IPC 结构关联一个 ipc_oerm 结构， 这个结构规定了 权限和所有者，它至少包含下列成员
```C
// sys/ipc.h
struct ipc_perm {
    uid_t uid;  // owner
    gid_t gid;  // owner
    uid_t cuid; // creator
    gid_t cgid; // creator
    mode_t mode;   // access mode
};
```

### 结构限制
3种形式的 XSI IPC 都有内置限制。大多数限制可以通过 重新配置内核来改变。
在描述3种IPC时，会指出限制


### 优缺点
一个基本问题是：IPC结构是在 系统范围内 起作用的，没有 引用计数。
如果进程创建一个消息队列，并且在 该队列中 放入 几条消息，然后终止， 那么该消息队列 及其 内容不会被删除。 
与管道相比，当最后一个 引用管道的 程序终止时，管道就被 完全删除了。 
对于FIFO而言，最后一个 引用FIFO 的进程终止时，虽然 FIFO的名字 保留在系统中，直到被显式删除，但是 FIFO中的数据已经被删除了。

。。这不是MQ最需要的 消息不丢失吗。

另一个问题是，这些IPC结构在 文件系统中没有名字。 不能用 第3章，第4章的 函数 来访问它们 或修改它们的属性。  为了支持这些IPC对象，内核中增加了 十几个全新的 系统调用。  我们不能用 ls，rm，chmod 操作 IPC对象，于是增加了 ipcs(1), ipcrm(1)

因为这些形式的 IPC 不使用 fd，所以不能使用 IO多路复用 (select, poll)。

![eb2c222c8020a5e1eaac00a031a000ae.png](../_resources/eb2c222c8020a5e1eaac00a031a000ae.png)

无连接 是指 不需要先调用某种形式的 open函数 就可以 发送消息
流控制，是指 如果系统资源(缓冲区)短缺，或者 如果接收进程 不能再接收更多的消息，则 发送进程 就要休眠， 当 流控制条件消失时，发送进程 应自动唤醒


## 15.7 消息队列

消息队列 是 消息的 链接表，存储在 内核中，由 消息队列 标识符 标识。

msgget 用于创建一个 新队列 或打开一个 现有队列。
msgsnd 将新消息 添加到 队列尾。每个消息 包含一个 正的长整型类型的 字段，一个 非负的长度 以及 实际数据字节数。
msgrcv 取消息。 不一定要 先进先出的顺序， 可以按照 消息的类型字段 取消息。

每个队列都有一个 msqid_ds 结构与之关联
```C
struct msqid_ds {
    struct ipc_perm msg_perm;
    msgqnum_t msg_qnum;
    msglen_t msg_qbytes;
    pid_t msg_lspid;    // pid of last msgsnd()
    pid_t msg_lrpid;    // pid of last msgrcv()
    time_t msg_stime;   // last msgsnd() time
    time_t msg_rtime;   // last msgrcv() time
    time_t msg_ctime;   // last change time
};
```

```C
// sys/msg.h
int msgget(key_t key, int flag);
```

```C
// sys/msg.h
int msgctl(int msqid, int cmd, struct msqid_ds *buf);
```

```C
int msgsnd(int msqid, const void *ptr, size_t nbytes, int flag);
```
ptr 指向一个 长整型数，它包含了 正的 整型消息类型。其后 紧跟着 消息数据 (如果 nbytes 为0，则无消息数据)。如果发送的 最长消息是 512 字节的，则可以定义下列结构
```C
struct mymesg {
    long mtype;
    char mtext[512];
};
```
ptr就是 指向mymesg 的指针。

flag 可以指定 IPC_NOWAIT， 类似于 文件IO的 非阻塞IO标志。
如果消息队列已满，则指定 IPC_NOWAIT 使得 msgsnd 立即出错 返回 EAGAIN。如果没有指定 IPC_NOWAIT， 则阻塞，直到 有空间， 或 从系统删除此队列，或捕捉到信号。

对 删除消息队列的 处理不是很完善，因为每个消息队列没有维护 引用计数器，所以在 队列被删除后，仍在使用 这一队列的 进程 对队列进行操作时 出错返回。
==删除文件，要等到 使用该文件的 最后一个进程 关闭了 fd 后，才能删除。==

```C
ssize_t msgrvc(int msqid, void *ptr, size_t nbytes, long type, int flag);
```
ptr 和 msgsnd 的一样
nbytes 是 数据缓冲区的长度，如果 消息的长度大于 nbytes，且 flag 中设置了 MSG_NOERROR，那么消息会被 截断 ( 不会通知我们，被截去的部分 被丢弃)。 如果没有设置这个标志，则出错返回 E2BIG (消息仍然保留在队列中)

type指定想要哪种消息
- 0，返回队列中的第一个消息
- `>0`，返回队列中 消息类型为 type的 第一个消息
- `<0`，返回队列中 消息类型值 小于等于 |type| 的消息，如果有多个，返回 类型值最小的。

type可以用来 赋予 优先权， 或者如果是 多个客户进程，可以用type 包含 客户进程的 进程ID (只要进程ID 可以放到 长整型中)。

flag可以指定 IPC_NOWAIT， 非阻塞。 没有消息时，msgrcv 返回 -1，errno为 ENOMSG。

msgrcv 成功执行时， 内核会更新 与该 消息队列 相关的 msgid_ds 结构 的 调用者进程ID(msg_lrpid),调用时间(msg_rtime)，消息数-1 (msg_qnum)


#### 不建议使用消息队列
如果要在 客户进程，服务进程 之间 双向数据流，可以使用 消息队列 或 全双工管道
下图是 Solaris 上3种技术的比较： 消息队列，全双工(streams), socket

![7fce62fd3005eba74e73f7071f1cb992.png](../_resources/7fce62fd3005eba74e73f7071f1cb992.png)

可以看到 消息队列 原来的 目的是提供 高速的IPC，但 现在 和其他形式的 IPC相比，速度方面已经没有什么差别了。
考虑到 使用 消息队列时 遇到的 问题 (15.6.4)， 得出的结论是，在新的 应用程序中 不应当 继续使用它们。

。。Solaris 也已经死了。。



## 15.8 信号量
信号量 和 已介绍过的 其他IPC (管道，FIFO，消息队列) 不同，它是一个计数器，用于为 多个进程 提供对共享数据对象的 访问

。。和 之前知道的信号量 一样。

常用的 信号量形式 是 二元信号量 (binary semaphore)

XSI 信号量 更复杂，以下3个特性导致了 不必要的复杂性
- 信号量并非是单个非负值，而必须定义为 包含 一个或 多个 信号量值得 集合。创建信号量 时，要指定 集合中 信号量值 的数量。
- 信号量的创建 semget，是独立于 它的 初始化 semctl 的。这是一个 致命的缺点，因为 不能原子地 创建 一个信号量集合，并对 集合中 每个 信号量 赋初始值
- 即使没有进程正在使用 各种形式的 XSI IPC，它们仍然是存在的。有的程序在 终止时 没有释放 已分配给它的 信号量。

内核为 每个信号量集合 维护一个 semid_ds 结构
```C
struct semid_ds {
    struct ipc_perm sem_perm;
    unsigned short sem_nsems;
    time_t sem_otime;   // last semop
    time_t sem_ctime;   // last change
};
```

每个信号量 是一个 无名结构：
```C
struct {
    unsigned short semval;  // semaphore value, alwats >= 0
    pid_t sempid;   // pid for last operation
    unsigned short semncnt; // number of processes awaiting semval > cuval
    unsigned short semzcnt; // number process awaiting semval == 0
};
```

```C
// sys/sem.h
int semget(key_t key, int nsems, int flag);
```
15.6.1 讨论了 key 变换为 标识符的规则， 是 创建 还是 引用 一个集合。
创建 新集合时，要对 semid_ds 结构的下列成员赋初始值
- ipc_perm
- sem_otime, 0
- sem_ctime, 当前时间
- sem_nsems，设置为 nsems

nsems 是 集合中 信号数量数。 如果创建新集合，必须指定 nsems， 如果是引用现有集合，则指定为0


```C
int semctl(int semid, int semnum, int cmd, ... /* union semun arg */);
```

。。很多。 457
cmd 可以 10种。 就是代表了 对 信号量集合的 操作。主要是 get，set 某个成员的 某个字段，删除信号量集合， 等


semop 自动执行 信号量集合上的 操作数组
```C
int semop(int semid, struct sembuf semoparray[], size_t nops);
```
semoparray 指向 sembuf 结构表示的 信号量操作 数组
```C
struct sembuf {
    unsigned short sem_num;
    short sem_op;
    short sem_flg;
};
```

。。很多 458

sem_op 为正 是释放资源
负数时，不能满足时 IPC_NOWAIT



### 信号量，记录锁，互斥量的 时间比较

记录锁：先创建一个空文件，并且 用该文件的 第一个字节(无需存在) 作为 锁字节。需要资源时，先对 该字节 获得一个写锁，释放资源时，对这个字节 解锁。
记录锁的性质确保 当一个锁的 持有者进程 终止时，内核会自动释放锁。

互斥量，如果一个进程没有释放互斥量 而终止，恢复 非常难。除非使用 pthread_mutex_consistent函数

![7db1a17136a986edbbcb972b47f2c465.png](../_resources/7db1a17136a986edbbcb972b47f2c465.png)

。。互斥量 默全秒

如果我们能 单一资源加锁，并且不需要 XSI信号量的 花哨的功能， 那么 记录锁 比信号量好。 因为 它更简单，速度更快，当进程终止时 系统 管理 遗留下来的锁。

虽然 互斥量更快，但我们依然 使用 记录锁，除非特别考虑 性能，因为：
- 在多进程间 共享 的内存中 使用 互斥量 来恢复一个终止的进程 更难。
- 进程共享的互斥量 属性 还没有得到普遍支持。

Linux3.2.0 支持 进程共享的 互斥量属性。



## 15.9 共享存储

允许 2个 或多个进程 共享一个 给定的存储区
因为数据不需要在 客户进程 和 服务器进程 之间复制， 所以 这是 最快的 IPC。
唯一要注意的事， 如果有进程 在写入 存储区，那么 其他进程 不应该 读取 这些数据， 通常使用 信号量/记录锁/互斥量 来 同步 存储访问 。

之前已有一种 共享存储，就是 多个进程 将同一个 文件 映射到 它们的 地址空间。

XSI 共享存储 和 内存映射的文件 的不同之处 在于，前者没有 相关的文件，XSI共享存储段 是 内存的 匿名段。

内核为每个 共享存储段 维护了一个结构：
```C
struct shmid_ds {
    struct ipc_perm shm_perm;
    size_t shm_segsz;
    pid_t shm_lpid; // pid of last shmop()
    pid_t shm_cpid; // pid of creator
    shmatt_t shm_nattch;    // number of current attaches
    time_t shm_atime;   // last attach time
    time_t shm_dtime;   // last detach
    time_t shm_ctime;   // last change
};
```
shmatt_t 是无符号整型，至少和 unsigned short 一样大。

```C
int shmget(key_t key, size_t size, int flag);
```

15.6.1 说明了 key 变为 标识符的规则， 以及是 创建还是 引用


```C
int shmctl(int shmid, int cmd, struct shmid_ds *buf);
```
cmd 有5种命令
- IPC_STAT， 取 shmid_ds 结构，存到 buf中
- IPC_SET，按 buf中的 结构的值 设置 共享存储段的 shmid_ds的下列3个字段：shm_perm.uid, shm_perm.gid, shm_perm.code
- IPC_RMID， 删除共享存储段， 由于 共享存储段 维护了 连接计数，所以 要等到 最后一个进程 终止 或 与该段分离。
- SHM_LOCK，linux，对共享存储段加锁， 只能超级用户执行
- SHM_UNLOCK，linux，解锁， 只能超级用户执行


创建完 共享存储段后，进程可以使用 shmat 将其连接到 它的 地址空间中
```C
// sys/shm.h

void *shmat(int shmid, const void *addr, int flag);
```
addr:
- 0，内核选择可用地址，这是推荐的方式
- 非0 且 没有指定 SHM_RND， 连接到 addr 指定的地址
- 非0 且 指定了 SHM_RND，则连接到 (addr - (addr mod SHMLBA))


```C
int shmdt(const void *addr);
```


。。463
/dev/zero 的存储映射

调用 mmap 时，指定 MAP_ANON 标志，并将 fd 指定为 -1. 结果的到的 区域时 匿名的，并且创建了一个 可与 后代进程 共享的 存储区。



## 15.10 POSIX 信号量

POSIX信号量 是3种 IPC 机制 (消息队列，信号量，共享存储) 之一

POSIX 信号量 是为了 解决 XSI信号量 的几个缺陷
- POSIX考虑更高的性能
- 更简单使用
- 删除时更完美。 XSI信号量删除后， 对这个信号量的 操作会失败。 POSIX信号量 删除后，依然存货，会存活到 最后一个进程 释放引用

有2种形式，命名的，非命名的。 差异在于 创建 和 销毁 的形式上。
未命名信号量 只存在于 内存中， 并要求能使用 信号量的进程 必须可以访问内存。 这意味着 只能应用于 同一进程中的 线程，或 不同进程中 已经映射相同内存内容 到它们的地址空间 中的线程。

新建 或 引用
```C
sem_t *sem_open(const char *name, int oflag, ... /* mode_t mode, unsigned int value */);
```
引用现有命名信号量时，只需要 2个参数，信号量名字 和 oflag参数的0值。

如果 oflag 为 O_CREAT，如果不存在，则创建，如果存在，则使用，但不会有 额外的初始化发生。

当指定 O_CREAT 的时候，需要 2个 额外的参数， 
mode 指定 谁可以访问 信号量，mode的取值 和 打开文件的 权限位相同
value 用来 在 创建信号时，指定 信号量初始值， 0 - SEM_VALUE_MAX

如果要 确保是 创建，那么使用 O_CREAT | O_EXCL 。如果信号量已存在，会导致 sem_open 失败。

为了可移植性，信号量命名 要遵循一定的规则
- 以 / 开头。 
- 名字不应该包含 其他 / 
- 最大长度 不应大于 _POSIX_NAME_MAX


释放资源 ( 不 是 信号量+1)
```C
int sem_close(sem_t *sem);
```

销毁 命名信号量，真正的销毁 会延迟到 最后一个打开的引用关闭。
```C
int sem_unlink(const char *name);
```


信号量-1
```C
int sem_trywait(sem_t *sem);    // 不阻塞，没有就返回-1，errno为 EAGAIN
int sem_wait(sem_t *sem);   // 阻塞
```
```C
int sem_timedwait(sem_t *restrict sem, const struct timespec *restrict tsptr);      // 阻塞直到 超时
```

信号量+1
```C
int sem_post(sem_t *sem);
```

如果要在 单个进程中 使用 POSIX信号量， 使用 未命名信号量 更容易。 这仅仅改变 创建 和 销毁 信号量的方式 

```C
int sem_init(sem_t *sem, int pshared, unsigned int value);
```
pshared 表明 是否在 多进程中 使用信号量，如果是，则设置为 非0值

```C
int sem_destory(sem_t *sem);
```


---
检索信号量值， 实际值可能在读出来后 就被其他进程 改了
```C
int sem_getvalue(sem_t *restrict sem, int *restrict valp);
```

在linux 上，POSIX 信号量的速度是 XSI信号量 的 18倍。
因为 linux上，将 文件映射到 进程地址空间， 没有使用 系统调用 来操作 各自的信号量。



## 15.11 客户进程 - 服务器进程 属性

470
1页半 都是字



# ch16 网络IPC：套接字


## 16.2 套接字描述符

套接字描述符 在unix中 被当做 一种 fd， 许多 fd的函数(如 read，write) 都可以用于 处理 套接字描述符

```C
#include <sys/socket.h>

int socket(int domain, int type, int protocol);
```

domain (域) 确定 通信的特性，包括 地址格式。

![2b18884042fb3ccb50d92025def7f755.png](../_resources/2b18884042fb3ccb50d92025def7f755.png)

大多数系统还包含 AF_LOCAL，这是 AF_UNIX 的别名
AF是指 address family

type 确定套接字类型，经一步确定 通信特征

![e1acab70b373db43c6b0f82fb939d6a7.png](../_resources/e1acab70b373db43c6b0f82fb939d6a7.png)


protocol 通常是 0，表示 为 给定的域 和 套接字类型 选择默认协议。

![e21f561dcc7ae2caa01dc227de3b513b.png](../_resources/e21f561dcc7ae2caa01dc227de3b513b.png)


对于数据报(SOCK_DGRAM)接口，2个对等 进程之间 通信时 不需要 逻辑连接。

字节流 (SOCK_STREAM) 要求在交换数据之前，在 本地套接字 和 通信的 对等进程的 套接字 之间 建立一个逻辑连接。

SOCK_STREAM 是 字节流服务，所以 app分辨不出 报文 的界限。这意味着 从 SOCK_STREAM 读数据时，它也许不会返回 所有由 发送进程 所写的字节数。最终可以获得发送过来的所有数据，但也许需要 若干次调用。

SOCK_SEQPACKET 和 SOCK_STREAM 很类似， 只是 该套接字 得到的是 基于 报文的服务，而不是字节流。 这意味着 从 SOCKE_SEQPACKET 套接字 接收到的 数据量 和 对方 发送的 一致。 流控制传输协议(stream control transmission protocol, SCTP) 提供了 因特网域 上的 顺序数据报服务。

SOCK_RAW 提供一个 数据报接口， 直接访问下面的 网络层 (即 IP层)。 使用这个接口，app负责构建 自己的 协议头部， 因为 TCP,UDP 被绕过了。

socket函数 和 open 函数类似，  都可以获得 用于 IO的 fd，当不再需要时，调用 close 来关闭 对文件 或 套接字 的访问，并释放 该fd

![22301f2796b25e79ad41d3b4a0771e8f.png](../_resources/22301f2796b25e79ad41d3b4a0771e8f.png)


套接字通信 是双向的， 使用 shutdown 禁止 一个套接字的IO
```C
int shutdown(int sockfd, int how);
```
how可选：
- SHUT_RD，无法从socket读取
- SHUT_WR，无法使用 socket 发送数据
- SHUT_RDWE，不能读，也不能发

有了close 为什么还要 shutdown？
只有最后一个 活动引用关闭时，close 才会释放 网络端点， shutdown可以使 一个socket处于不活动状态，和它的 fd 无关。
有时 需要关闭 socket 的一个方向。


## 16.3 寻址
如何表示 一个目标通信进程
进程标识 由2部分组成，
一部分是 计算机的 网络地址，它标识了 网络上，我们想 与之通信的 计算机
另一部分时 该计算的 端口号 表示的服务，它可以帮助标识 特定的 进程。


###16.3.1 字节序
大端：最大字节地址 出现在 最低有效字节(LSB)
小端，相反，最小字节地址 出现在 最低有效字节

不管如何，最高有效字节(MSB) 总是在 左边。LSB在右边。

![8dfd2fdafd67985752e61d2ef1d65857.png](../_resources/8dfd2fdafd67985752e61d2ef1d65857.png)

对于 0x04030201, 不管字节序如何，MSB 都将包含 4， LSB是1
对于小端，cp[0] 是 1， cp[3] 是4， 
对于大端,cp[0] 是4， cp[3] 是1


![38ba919f2a648017127052e2b6e61469.png](../_resources/38ba919f2a648017127052e2b6e61469.png)


网络协议 指定了 字节序。
TCP/IP 使用 大端字节序。

对于TCP/IP 应用，有4个用来 进行 处理器字节序 和 网络字节序 之间的 转换
```C
// arpa/inet.h
uint32_t htonl(uint32_t hostint32); // 返回 网络字节序表示的 32位
uint16_t htons(uint16_t hostint16);
uint32_t ntohl(uint32_t netint32);    // 返回主机字节序表示的 32
uint16_t ntohs(uint16_t netint16);
```

h： host
n： net
l： long
s： short



### 16.3.2 地址格式

地址需要转为 通用的 地址结构 sockaddr
```C
struct sockaddr {
    sa_family_t sa_family;
    char sa_data[];
};
```

套接字实现 可以自由添加 额外成员，并定义 sa_data 的大小。
linux中
```C
struct sockaddr {
    sa_familt_t sa_family;
    char sa_data[14];
};
```

因特网地址 定义在 netinet/in.h 中，在 IPv4因特网域中，套接字地址 用 结构 sockaddr_in 表示
```C
struct in_addr {
    in_addr_t s_addr;   // IPv4 address
};
struct sockaddr_in {
    sa_family_t sin_family; // address family
    in_port_t sin_port; // port
    struct in_addr sin_addr;    // IPv4 address
};
``

in_port_t 是 uint16_t
in_addr_t 是 uint32_t
在 stdint.h 中


IPv6 因特网域 (AF_INET6) socket 地址 用 sockaddr_in6 表示
```C
struct in6_addr {
    uint8_t s6_addr[16];
};
struct sockaddr_in6 {
    sa_family_t sin6_family;
    in_port_t sin6_port;
    uint32_t sin6_flowinfo;
    struct in6_addr sin6_addr;
    uint32_t sin6_scope_id;
};
```

在linux中：
```C
struct sockaddr_in {    // 没有6？书上是没有
    sa_family_t sin_family;
    in_port_t sin_port;
    struct in6_addr sin6_addr;
    unsigned char sin_zero[8];  // 填充，全0
};
```

尽管 sockaddr_in 和 sockaddr_in6 结构相差较大，但它们均被强制 转换成 sockaddr 结构。


打印人能理解的地址
```C
// arpa/inet.h
const char *inet_ntop(int domain, const void *restrict addr, char *restrict str, socklen_t size);
int inet_pton(int domain, const char *restrict str, void *restrict addr);
```


### 地址查询
网络配置信息 被放在 /etc/hosts, /etc/services 中。
也可以由 名字服务管理，如 域名系统(DNS)，网络信息服务(NIS)

无论放哪里，都可以用 用于的函数访问

```C
// netdb.h
struct hostent *gethostent(void);   // 找到给定计算机系统的主机信息
void sethostent(int stayopen);
void endhostent(void);

struct hostent {
    char *h_name;
    char **h_aliases;
    int h_addrtype;
    int h_length;
    char **h_addr_list;
};
```
返回的地址采用 网络字节序

sethostent会打开文件，如果stayopen 非0，则 gethostent 后，文件依然打开，需要使用 endhostent关闭。


gethostbyname,gethostbyaddr 已经过时了，要使用 getaddrinfo


获得网络名字 和 网络编号
```C
// netdb.h
struct netent *getnetbyaddr(uint32_t net, int type);
struct netent *getnetbyname(const char *name);
struct netent *getnetent(void);
void setnetent(int stayopen);
void endnetent(void);

struct netent {
    char *n_name;
    char **n_alises;
    int n_addrtype;
    uint32_t n_net;
};
```
网络编号 按照 网络字节序返回


协议名字 和 协议编号之间的映射
```C
// netdb.h
struct protoent *getprotobyname();
struct protoent *getprotobynumber();
struct protoent *getprotoent();
void setprotoent(int stayopen);
void endprotoent(void);

struct protoent {
    char *p_name;
    char **p_aliases;
    int p_proto;
};
```


服务是由 地址的端口号部分表示的，每个服务由一个唯一的众所周知的端口号来支持。
```C
// netdb.h
struct servent *getservbyname(const char *name, const char *proto);
struct servent *getservbyport(int port, const char *proto);
struct servent *getservent(void);
void setservent(int stayopen);
void endservent(void);

struct servent {
    char *s_name;
    char **s_aliaese;
    int s_port;
    cahr *s_proto;
};
```


POSIX.1 定义了 若干个 函数，允许 一个应用程序 将 一个主机名和一个服务名 映射到 一个地址， 或者反之，用来代替 较老的 gethostbyname, gethostbyaddr
```C
// sys/socket.h
// netdb.h
int getaddrinfo(const char *restrict host, const char *restrict service, const struct addrinfo *restrict hint, struct addrinfo **restrict res); // res 就是结果，是一个数组

void freeaddrinfo(struct addrinfo *ai);

struct addrinfo {
    int ai_flags;
    int ai_family;
    int ai_socktype;
    int ai_protocol;
    socklen_t ai_addrlen;
    struct sockaddr *ai_addr;
    char *ai_canonname;
    struct addrinfo *ai_next;
};
```
可以提供一个可选的 hint 来选择符合特定条件的地址， hint是一个用于 过滤地址的 模板。

如果 getaddrinfo 是吧， 不能用 perror，strerror， 而是要调用 gai_strerror
```C
// netdb.h
const char *gai_strerror(int error);
```

将地址转为 主机名，服务名
```C
// sys/sockets, netdb
int getnameinfo(const struct sockaddr *restrict addr, socklen_t alen, char *restrict host, socklen_t hostlen, char *restrict service, socklen_t servlen, int flags);
```


### 将套接字与地址关联
对于服务器，需要将 接收 客户端请求的 服务器套接字 关联上 一个 众所周知的地址。
客户端 应该有一种方法来发现 连接服务器所需要的地址，最简单的方法就是 服务器保留一个 地址 并且注册在 /etc/services 或某个 名字服务中。

使用bind 来关联 地址和套接字
```C
// sys/socket.h
int bind(int sockfd, const struct sockaddr *addr, socklen_t len);
```

对使用的地址有以下一些限制
- 地址必须有效，不能指定其他机器的地址
- 地址 必须和 创建socket的 地址族 所支持的格式 相匹配
- 端口号 必须 大于等于 1024， 除非 该进程 有 相应的特权 (即 超级用户)
- 一般只能将一个 套接字端点 绑定到 一个给定地址上。

对于因特网域，如果指定IP地址为 INADDR_ANT (netinet/in.h) ，套接字端点 可以绑定到 所有的 系统网络接口上。 这意味着 可以接受 这个系统 安装的 任何一个网卡的 数据包。


发现绑定到 socket上的地址
```C
// sys/socket.h
int getsockname(int sockfd, struct sockaddr *restrict addr, socklen_t *restrict alenp);
```

如果套接字已经和 对方相连，可以使用 getpeername 找到对方的地址
```C
int getpeername(int sockfd, struct sockaddr *restrict addr, socklen_t *restrict alenp);
```


## 16.4 建立连接

```C
// sys/socket.h
int connect(int sockfd, const struct sockaddr *addr, socklen_t len);
```
如果sockfd 没有绑定到地址， connect 会给 调用者绑定一个默认地址。

```C
for (numsec = 1; numsec <= MAXSLEEP; numsec <<= 1)
{
    if (connect(sockfd, addr, alen) == 0) {
        // connect accept
        return (0);
    }
    if (numsec <= MAXSLEEP / 2)
        sleep(numsec);
}
return (-1);
```
指数补偿， exponential backoff

connect 可以用于 无连接的网络服务(SOCK_DGRAM)，如果用 SOCK_DGRAM 套接字 调用 connect， 传送的报文的 目标地址就会设置成 connect 中指定的地址，这样 每次传送报文就不需要 再提供地址了。 另外，仅能接收 来自指定地址 的报文。


服务器调用 listen 表明 它愿意 接收连接请求
`int listen(int sockfd, int backlog);`
backlog 提供了一个 提示，提示系统 该进程 所要入队的 未完成连接请求数量。 其实际值 有 系统决定，但上限是 sys/socket.h 的 SOMAXCONN

==一旦队列满，系统就会拒绝 多余的连接请求==。

一旦服务器调用量 listen，所用的socket就可以接收 连接请求，使用 accept 获得连接请求 并建立 连接
```C
int accept(int sockfd, struct sockaddr *restrict addr, socklen_t *restrict len);
```

accept 返回的 fd 是 socket fd，连接到 调用connect 的客户端。
这个新的socket fd 和 原始套接字(sockfd) 具有相同的 套接字类型 和地址族。
==传给 accept的 原始套接字 没有 关联到 这个链接， 而是继续保存 可用状态并接收 其他连接请求==

如果不关心 客户端标识， 可以将参数 addr 和 len 设置为 NULL。 否则，在调用 accept之前，将addr参数设置为 足够大的缓冲区 来存放地址，并将 len指向 这个缓冲区的字节大小的 整数。 返回时，accept 会在 缓冲区 填充 客户端的地址，并更新 len 来反应 该地址的 大小。

==如果没有 连接请求，accept 会阻塞==。 如果 sockfd 是 非阻塞模式，accept 会返回 -1， 并设置 errno 为 EAGAIN 或 EWOULDBLOCK

==服务器可以使用 poll，select 来等待 请求的 到来。==


## 16.5 数据传输
既然 套接字端点 表示为 一个 fd，那么只要 建立连接，就可以 使用 read 和write 来通过套接字 通信。

如果要 指定选项，从多个客户端接收数据包，或者 发送带外数据，就需要使用 6个专门 为数据传递设计的 套接字函数。
3个用来发，3个用来接收

```C
// sys/socket.h
ssize_t send(int sockfd, const void *buf, size_t nbytes, int flags);
```
类似 write，使用 send时，socket必须已连接。 buf，nbytes 的含义 和 write中一致。
和write不同的是，send支持 第4个参数 flags：

![17c916b476946d7aa0997b2e8d1d7b4d.png](../_resources/17c916b476946d7aa0997b2e8d1d7b4d.png)

即使 send 成功返回，也不代表 另一端的进程 一定收到了数据。 我们能保证的是 ，send成功返回时， 数据已经 无错 地发送到 网络驱动程序上。

对于支持报文边界的协议，如果尝试 发送的 单个报文长度 超过 协议所支持的 最大长度，那么 send 会失败，errno为EMSGSIZE。 对于字节流协议，send会阻塞直到 整个数据传输完成。

sendto 和send 很类似，区别是 sendto 可以在 无连接的 socket上 指定一个 目标地址
```C
ssize_t snedto(int sockfd, const void *buf, size_t bytes, int flags, const struct sockaddr *destaddr, socklen_t destlen);
```
对于面向连接的 socket，目标地址 是被忽略的。
对于 无连接的 socket，调用send前必须调用 connect设置目标地址 或者 使用 sendto。

通过socket发送数据，还有一个选择，可以调用带有 msghdr 结构的 sendmsg 来指定 多重缓冲区 传输数据，这和 writev 函数很相似
```C
// sys/socket.h
ssize_t sendmsg(int sockfd, cosnt struct msghdr *msg, int flags);

struct msghdr {
    void *msg_name;
    socklen_t msg_namelen;
    struct iovec *msg_iov;
    int msg_iovlen;
    void *msg_control;
    socklen_t msg_contorllen;
    int msg_flags;
};
```

---
函数revc 和 read相似，但是 recv 可以指定标志 来控制如何接收数据
```C
ssize_t recv(int sockfd, void *buf, size_t nbytes, int flags);
```

![540d89b6653fc5054018463aaaf5d6e3.png](../_resources/540d89b6653fc5054018463aaaf5d6e3.png)


指定 MSG_PEEK 的时候，可以查看 下一个要读取的数据但是 不真正取走它。
对于 SOCK_STREAM 套接字，接收的数据可以比 预期的少。MSG_WAITALL 会 阻止这种行为，直到 所请求的 数据全部返回， recv才返回。

如果发送者已经调用 shutdown 来结束传输，或者 网络协议 支持按默认的顺序 关闭 并且发送端 已经关闭，那么当 所有数据 接收完后， recv会返回0.


获得数据发送者的源地址
```C
// sys/socket.h
ssize_t recvfrom(int sockfd, void *restrict buf, size_t len, int flags, struct sockaddr *restrict addr, socklen_t *restrict addlen);
```
如果addr非NULL，那么它将包含 数据发送者的 套接字端点地址。
addrlen 指向一个整数，是 addr所指向的 套接字缓冲区的 字节长度。 函数返回时，它被设置为 该地址的 实际字节长度。
因为可以获得 发送者地址，所以 ==recvfrom 通常用于 无连接的 套接字==。 否则 recvfrom 等同于 recv。

为了将接受到的 数据 送入多个缓冲区，类似 readv，或者想接收 辅助数据，可以使用 recvmsg
```C
ssize_t recvmsg(int sofkfd, struct msghdr *msg, int flags);
```

![c7eacc4c9256bda8f9b81d19d07aa983.png](../_resources/c7eacc4c9256bda8f9b81d19d07aa983.png)

### 代码 494 - 502
494 - 502


## 16.6 套接字选项
提供了 2个 socket 选项接口来控制 socket 行为。
一个用来 设置选项，一个可以查询选项的 状态。
可以设置 或 获取 以下 3种选项
- 通用选项，工作在所有套接字选项上
- 在套接字层次管理的选项，但是依赖于下层协议的支持
- 特定于某协议的选项，每个协议独有的

```C
// sys/socket.h
int setsockopt(int sockfd, int level, int option, const void *val, socklen_t len);
```
level 标识了 选项应用的协议：如果是 通用的套接字层次选项，设置为 SOL_SOCKET，否则 设置为 控制这个选项的 协议编号。对于 TCP来说，level是 IPPROTO_TCP，对于IP是IPPROTO_IP。

option：
![9daca45a85b84b9d9f417015da2caeb4.png](../_resources/9daca45a85b84b9d9f417015da2caeb4.png)

val 根据 option 的不同 指向一个 数据结构 或 整数。 一些option是 on/off 开关，非0启用，0禁止选项
len 表明了 val指向的 对象的 大小。


```C
int getsocket(int sockfd, int level, int option, void *restrict val, socklen_t *restrict lenp);
```
lenp为 复制选项缓冲区的长度， 如果选项实际长度大于此值，则 选项被截断。 函数返回时，lenp 变为 实际长度。


## 16.7 带外数据
out-of-band data
是一些通信协议 支持的 可选功能，与普通数据相比， 它允许 更高优先级的 数据传输。 带外数据 先行传输，即使 传输队列 已经有值。 

TCP支持 带外数据，UDP不支持

TCP将带外数据 称为 紧急数据(urgent data)。 TCP仅支持 一个字节的 紧急数据，但是允许 紧急数据 在 普通数据传输机制 的 数据流 之外传输。
为了产生 紧急数据，可以在 3个 send函数的 任意一个中 指定 MSG_OOB标志。
如果带 MSG_OOB 标志 发送的字节 超过1个时，最后一个字节 被视为 紧急数据字节。

如果通过套接字安排了 信号的产生，那么 紧急数据被接收时，会发送 SIGURG 信号。

通过下面的函数 安排 进程 接收套接字的信号
`fcntl(sockfd, F_SETOWN, pid);`

可以获得当前套接字的 owner
`owner = fcntl(sockfd, F_GETOWN, 0);`
如果owner是正数，那么就是 进程ID
是负数，绝对值 就是 进程组的ID

TCP支持 紧急标记(urgent mark) 的概念，即在普通数据流中 紧急数据所在的 位置。
如果采用 socket选项 SO_OOBINLINE，那么可以在 普通数据中 接收 紧急数据。
为了帮助判断 是否已经到达紧急标记：
`int sockatmark(int sockfd);`
如果下一个要读取的字节 在紧急标记处，那么返回1。

当带外数据 出现在 socket 读取队列， select函数 会返回一个 文件描述符，并有一个 待处理的 异常条件。
可以在普通数据流上接收紧急数据，也可以在其中一个 recv函数中采用 MSG_OOB 标志 在其他队列数据之前 接收 紧急数据。

TCP队列 仅用一个字节的 紧急数据，如果在接收当前紧急数据字节之前，又有新的紧急数据到来，那么 已有的字节会被丢弃。


## 16.8 非阻塞和异步IO
通常，recv没有数据可用时 会阻塞。
当socket 输出队列 没有足够空间 来发送消息时，send 会阻塞。
在 socket 非阻塞模式下，行为会改变。在这种情况下，这些函数 不会阻塞 而是 失败。将 errno设置为 EWOULDBLOCK 或 EAGAIN。
当这种情况发生时，可以使用 poll 或 select 来判断 能够接受 或传输 数据。

在基于套接字的 异步IO中，当从 套接字 中读取数据时，或者当套接字 写队列中 空间变得可用时，可以安排要发送的信号 SIGIO。
启用异步IO时一个 2步的过程
1. 建立socket所有权，这样 信号可以被传递到 合适的进程
2. 通知 socket 当IO操作 不会阻塞时 发信号

第一步有3种方法
- fcntl, F_SETOWN
- ioctl, FIOSETOWN
- ioctl, SIOCSPGRP

第二步，有2种方法
- fcntl, F_SETFL, O_ASYNC
- ioctl, FIOASYNC

![d64cca02176f697142c3ed8a5bde071f.png](../_resources/d64cca02176f697142c3ed8a5bde071f.png)




# ch17 高级进程间通信

## UNIX域套接字
同一台计算机上运行的 进程之间的 通信。
因特网域socket 也可以做到，但是 UNIX域socket 更高效。
UNIX域socket 仅复制数据，不需要 执行协议处理，不必添加/删除 网络报头，无需计算校验和，不用产生 顺序号，无需发送确认报文

有 流， 数据报 2种接口。
UNIX域 数据报 是可靠的， 不会丢失，不会传错。
UNIX域套接字就像 socket 和 管道的 混合。

可以使用 面向网络的 域套接字接口， 或 使用 socketpair 创建一对 无名，相互连接的 UNIX域套接字
```C
// sys/socket.h
int socketpair(int domain, int type, int protocol, int sockfd[2]);
```

一对相互连接的 UNIX域套接字 可以起到 全双工管道的 作用。我们称为 fd管道 (fd-pipe)


命名UNIX域套接字
。。bind
511


## 17.3 唯一连接
服务器进程可以使用 标注bind，listen，accept 函数，为客户进程 安排一个 唯一的 UNIX域连接。

客户进程使用 connect 与服务器进程 联系。

服务器进程 接受 connect 请求后，在服务器进程 和 客户进程之间 就存在了 唯一连接。

我们将 开发 3个函数，使用这些函数 可以在 运行于同一台计算机上的 2个无关进程 之间 创建 唯一连接。

![017a76b4ea7c9eebe1e3aa2d46a48ed3.png](../_resources/017a76b4ea7c9eebe1e3aa2d46a48ed3.png)


```C
// apue.h
int serv_listen(const char *name);
int serv_accept(int listenfd, uid_t *uidptr);
int cli_conn(const char *name);
```

513 - 517 代码实现


## 17.4 传送文件描述符

在2个进程间 传送打开fd 的技术 非常有用。
可以对 客户进程 - 服务器进程 应用进行不同的设计
它使 一个进程 (通常是服务器进程) 能够处理 打开一个文件所要做的一切操作 (包括将 网络名翻译为网络地址，拨号调制解调器，协商文件锁 等) 以及 向调用进程 送回 一个fd，该fd用于 之后所有的IO函数。

![8eac6149a018761d694fd4148f99e41a.png](../_resources/8eac6149a018761d694fd4148f99e41a.png)


```C
// apue.h   // .. 自己写的
int send_fd(int fd, int fd_to_send);
int send_err(int fd, int status, const char *errmsg);
int recv_fd(int fd, ssize_t (*userfunc)(int, const void *, size_t));
```

519 - 527

定义3个宏，用于访问控制数据
```C
#include <sys/socket.h>
unsigned char *CMSG_DATA(struct cmsghdr* cp);
struct cmsghdr *CMSG_FIRSTHDR(struct msghdr *mp);
struct cmsghdr *CMSG_NXTHDR(struct msghdr *mp, struct cmsghdr *cp);

unsigned int CMSG_LEN(unsigned int nbytes); // 自定义
```




## 17.5 open服务器进程 第一版
527 - 533
代码
## 17.6 open服务器进程 第二版
533 - 542
代码

unistd.h
`int getopt(int argc, char *const argv[], const char *options);`



# ch18 终端IO

# ch19 伪终端

# ch20 数据库函数库
dbm
ndbm


# ch21 与网络打印机通信





# 附录A 函数原型
pdf 695
# 附录B.2  标准出错例程




2024-03-22 23:24



