
2023-08-18 14:24

https://sourceware.org/gdb/current/onlinedocs/gdb.html/

http://c.biancheng.net/view/8123.html


==DDD==



[[toc]]



---
---
---


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


# gdb的图形化界面 DDD



# gdb 命令

![8bee6e6419dd4b61b5e1746891c4ddfd.png](../_resources/8bee6e6419dd4b61b5e1746891c4ddfd.png)


# GDB bilibili

代码是 my_cpp_code/any/gdb/... 少最后一个。


编译器会优化代码
- 循环展开
- 编译期计算
- 重排序
- 消除死代码

导致我们无法看到执行的步骤。


下面的参数 让编译期不做任何优化，修改。 。。。应该是 产生 带调试信息的 可执行文件。
`g++ -g xxx.cpp`


## gdb命令


### 直接回车

运行上一条命令


### run (r)

```text
(gdb) run
Starting program: /my_github/my_cpp_code/any/gdb/zzz 

This GDB supports auto-downloading debuginfo from the following URLs:
  <https://debuginfod.fedoraproject.org/>
Enable debuginfod for this session? (y or [n]) n     /////////?????
Debuginfod has been disabled.
To make this setting permanent, add 'set debuginfod enabled off' to .gdbinit.
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib64/libthread_db.so.1".
hi
[Inferior 1 (process 16607) exited normally]
Missing separate debuginfos, use: dnf debuginfo-install glibc-2.38-7.fc39.x86_64 libgcc-13.2.1-4.fc39.x86_64 libstdc++-13.2.1-4.fc39.x86_64
```

`run arg1 arg2 < infile.in`
没太理解，估计是 参数 从文件里拿？



### break (b)
breakpoint
`break 7`       第7行
`break main`   main方法

==打印的是 下一次将执行的 行==

```text
(gdb) break main
Breakpoint 1 at 0x40114e: file 01_basic.cpp, line 6.
(gdb) run
Starting program: /my_github/my_cpp_code/any/gdb/zzz 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib64/libthread_db.so.1".

Breakpoint 1, main () at 01_basic.cpp:6
warning: Source file is more recent than executable.
6	    int i = 3;
```

#### info breakpoints (i b)
显示所有 断点

info 有很多很多功能，直接 info 会显示。

```text
(gdb) info breakpoints
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000401281 in main() 
                                                   at 04_example.cpp:34
	breakpoint already hit 1 time               // -----------
3       hw watchpoint  keep y                      x
```

#### delete (d)

```text
(gdb) info breakpoints
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000401281 in main() 
                                                   at 04_example.cpp:34
	breakpoint already hit 1 time
3       hw watchpoint  keep y                      x
(gdb) delete 3                          // ---------------------
(gdb) info breakpoints
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000401281 in main() 
                                                   at 04_example.cpp:34
	breakpoint already hit 1 time
```

```text
(gdb) delete
Delete all breakpoints? (y or n) n
(gdb) d
Delete all breakpoints? (y or n) n
```

。。感觉这的所有命令 都有缩写。。


#### next (简写n)
单步，不会进入方法

```text
6	    int i = 3;
(gdb) next
7	    int j = 4;
(gdb) 
```

#### step
单步，进入方法


#### continue (c)
执行到下一个断点


#### finish 
执行，直到 退出本方法 时(销毁堆栈) 停止

```text
(gdb) finish        // ----------------
Run till exit from #0  0x00000000004011ab in factorial (n=@0x7fffffffdd1c: 1)
    at 03_recursive.cpp:8
0x00000000004011ab in factorial (n=@0x7fffffffdd5c: 2) at 03_recursive.cpp:8
8	        return n * factorial(n - 1);
Value returned is $2 = 1
(gdb) finish            // --------------------
Run till exit from #0  0x00000000004011ab in factorial (n=@0x7fffffffdd5c: 2)
    at 03_recursive.cpp:8
0x00000000004011ab in factorial (n=@0x7fffffffdd9c: 3) at 03_recursive.cpp:8
8	        return n * factorial(n - 1);
Value returned is $3 = 2
```


#### list (l)
显示当前行的 前后的代码，而不是只显示 一行

```text
(gdb) next
7	    int j = 4;
(gdb) list
2	#include <iostream>
3	
4	int main() 
5	{
6	    int i = 3;
7	    int j = 4;
8	
9	    i += j;
10	    j = i * 2;
```

##### list 25
展示 25行 上下的 代码。


#### print (p)

```text
(gdb) print i
$1 = 3
(gdb) print j           // 还没有执行 int j = 4;
$2 = -8424
(gdb) print a
No symbol "a" in current context.
```

```text
(gdb) p j
$3 = 4
(gdb) p j+i
$4 = 11
(gdb) p j+i+i+i
$5 = 25
```

##### print address
```text
(gdb) p &i
$6 = (int *) 0x7fffffffddec
(gdb) p &j
$7 = (int *) 0x7fffffffdde8
```


### quit (q)
离开 gdb

```text
(gdb) quit
A debugging session is active.

	Inferior 1 [process 16735] will be killed.

Quit anyway? (y or n) y
```




### up / down

调用栈的上一层 下一层

```text
Program received signal SIGSEGV, Segmentation fault.
0x0000000000401136 in crash (i=0x0) at 02_example.cpp:6
warning: Source file is more recent than executable.
6	    *i = 1;
Missing separate debuginfos, use: dnf debuginfo-install glibc-2.38-7.fc39.x86_64 libgcc-13.2.1-4.fc39.x86_64 libstdc++-13.2.1-4.fc39.x86_64
(gdb) up            ///////----------------
#1  0x000000000040117b in f (i=0x7fffffffdddc) at 02_example.cpp:15
15	    crash(j);
(gdb) down      ////////----------------
#0  0x0000000000401136 in crash (i=0x0) at 02_example.cpp:6
6	    *i = 1;
```

### backtrace (bt)
调用栈

```text
(gdb) backtrace
#0  0x0000000000401136 in crash (i=0x0) at 02_example.cpp:6
#1  0x000000000040117b in f (i=0x7fffffffdddc) at 02_example.cpp:15
#2  0x0000000000401192 in main () at 02_example.cpp:21
```



### display / undisplay

跟踪 j
undisplay 序号

```text
(gdb) p j
$2 = (int *) 0x7ffff7e423c0
(gdb) display j                 // --------
1: j = (int *) 0x7ffff7e423c0
(gdb) n
13	    sophisticated(j);
1: j = (int *) 0x7fffffffdddc
(gdb)                           // 直接 回车
14	    j = complicated(j);
1: j = (int *) 0x7fffffffdddc
(gdb) n
15	    crash(j);       // 将要执行的行
1: j = (int *) 0x0          // null pointer ， 监控的 变量
(gdb) undisplay i           // ---------
```



### watch
当变量的值 变化时，暂停

```text
(gdb) watch x       // -----------
Hardware watchpoint 2: x
(gdb) c         // ---
Continuing.

Hardware watchpoint 2: x

Old value = 10
New value = 30
bar (p=@0x7fffffffdddc: 30) at 04_example.cpp:14
14	    return unknown(p);
```


### whatis

```text
(gdb) whatis x
type = int
```




## 高级功能

### target record-full
让 gdn记录下所有， 以便 撤销/回滚/回退

可能无效。

#### reverse-next (rn)

#### reverse-step

#### reverse-continue



### set var

直接修改变量的值。

```text
(gdb) set var x=1234
(gdb) p x
$4 = 1234
```


---

# gdb 命令大全

启动和退出
- 启动gdb：`gdb [程序名]`
- 启动gdb并加载核心转储文件：`gdb [程序名] [核心转储文件]`
- 退出gdb：`quit 或 q`

帮助
- 查看gdb的帮助信息：`help`
- 查看特定命令的帮助：`help [命令名]`

## 基本命令
- 运行程序：`run [参数]`
- 继续执行到下一个断点：`continue 或 c`
- 单步执行：
  - 单步进入（进入函数内部）：`step 或 s`
  - 单步跳过（不进入函数内部）：`next 或 n`
- 设置断点：
  - 在特定行设置断点：`break [行号] 或 b [行号]`
  - 在特定函数处设置断点：`break [函数名] 或 b [函数名]`
- 删除断点：
  - 删除所有断点：`delete breakpoints 或 d`
  - 删除特定编号的断点：`delete [断点编号] 或 d [断点编号]`
- 查看当前断点：`info breakpoints 或 i b`
- 查看当前执行位置：`list, l, info line, i l`
- 查看变量值：
  - 打印变量值：`print [变量名] 或 p [变量名]`
  - 打印变量地址：`print &[变量名] 或 p &[变量名]`
- 查看寄存器：`info registers 或 i r`
- 查看堆栈跟踪：`backtrace 或 bt`
- 查看内存：
  - 查看内存地址的内容：`x/[格式] [地址]`，例如 x/10xb 0x12345678 查看10个字节的十六进制内容
  - 格式包括：x（十六进制），d（十进制），u（无符号十进制），o（八进制），c（字符），s（字符串）等。

## 高级调试技术
- 条件断点：设置仅在特定条件满足时才触发断点，例如 `break func if var == value`
- 观察点（Watchpoints）：监视变量的值变化，例如 `watch var`
- 线程操作：
  - 切换线程：`thread [线程号]`
  - 查看线程列表：`info threads 或 i th`
- 信号处理：查看或处理信号，例如 `handle SIGSEGV nostop noprint pass`



