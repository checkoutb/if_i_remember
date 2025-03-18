
[[toc]]



---

# cpp

## new vs. malloc, delete vs. free

c 的malloc，free 是 库函数

c++ 的new，delete 是 操作符， 可以进行重载
new，delete 会调用 构造器，析构器

```c++
int main() {
    Test* p = new Test();
    return 0;
}
```

这段代码只调用了构造器，没有析构器，需要加上 `delete p;` 才会调用析构器

```c++
int main(int argc, char* argv[]) {
    Test* p = (Test*) malloc(sizeof(Test));  // 不会调用构造器
    return 0;
}
```

内存池，大概率使用 malloc来申请内存，那么如何调用构造器呢？

new 分为2步，申请内存，调用构造器

```c++
int main() {

    Test* p = (Test*) malloc(sizeof(Test));
    Test* p2 = new(p) Test();   // 调用构造器
    // p,p2 是同一个指针。

    // 合并后
    Test* p3 = new( (Test*) malloc(sizeof(Test)) ) Test();

    // 析构
    p3 -> ~Test();
    free(p3);
}
```

## 快手C++二面

https://www.bilibili.com/video/BV1QPtueuExD

### 一亿数据插入查找，unordered_map，map 怎么选？

unordered_map 优点

- 平均性能优秀， 在hash函数分布均匀 且 没有大量hash冲突 的情况下，unordered_map 的插入，删除，查找 是 O(1)
- 缓存友好， 由于数据随机分布，这可能导致在遍历 unordered_map 时 更好地利用 CPU缓存，从而提高性能。 。。。？我怎么觉得他说反了， 随机分布，遍历时 反而无法利用 局部性原理啊。

缺点

- 最坏情况性能差， 当 hash函数设计不佳 或 大量数据 映射到 同一个hash桶时， unordered_map 的性能会 急剧下降，退化到 O(n) 。 。。。baidu过，这个要看 具体的编译器/库的实现，如果像java那样 链太长就转为 RB tree 的话，应该 问题不大， 但是 codeforces 上肯定没有转为 RB tree的功能。
- 内存开销大， 为了处理hash冲突， unordered\_map需要额外的空间来 存储链表 或 RB tree， 增加了 内存开销 。。。。 但是 map红黑树 也有这个问题啊。。下面也说道了， map的 空间效率 可能比 unordered\_map 低。。。。 再后面，看到了，负载因子，但是 应该也不会导致 内存比 map 大吧。
    。。确实内存消耗大， 因为有 负载因子 (https://zhuanlan.zhihu.com/p/309991114) 所以会浪费一些 槽，但是 也没几个啊。可以忽略不计的。

map 优点

- 稳定的性能， 基于RB tree，插入，删除，查找 都是 O(logn)， 不会受到 数据集大小的 影响。
- 元素有序，按 key 升序排列。 在 需要按 key 顺序访问数据时 非常方便。

缺点

- 性能稍逊，相比 unordered\_map 的 O(1)， map在 大数据集时 可能稍逊 unordered\_map 。。。 似乎有印象，但是这个 大数据集 的大小 需要 非常非常非常大， 基本不会碰到的
- 空间效率， 虽然 map 的内存使用相对 紧凑，但是 某些情况下 (大量小对象)，内存效率可能不会 unordered_map

除了 unordered_map，map 本身的 特性，还需要考虑

- 样本重复度，样本数据集大小

### n层二叉平衡树，至少有多少个节点

平衡二叉树 的性质

- 左子树 和 右子树 的高度之差 小于1
- 左子树，右子树 也是平衡二叉树

`f(n) = f(n-1) + f(n-2) + 1, n>=3`
f(1) = 1
f(2) = 2
f(3) = 4

。。就是， 现有一棵平衡二叉树 (f(n-1))， 复制它的 上一次迭代的 平衡二叉树(f(n-2)) ( 因为 左右子树 高度可以差1，所以 使用 f(n-1-1))， 并且 新建一个 root节点 来 连接 这 2棵树。 就得到了 新 平衡二叉树的 最少节点是 `f(n-1) + f(n-2) + 1`

### 为什么使用线程池

在并发环境下，系统不能 确定 在任意时刻 需要 执行多少任务，投入多少资源，这种 不确定性 会带来一些问题

- 频繁的 创建，释放 线程， 带来 性能损耗
- 线程无限创建，可能会 耗尽系统资源
- 线程随意创建，释放，会 增加 系统的不稳定性。

线程池解决的核心问题就是资源管理问题，使用线程池的优点

- 降低资源损耗，通过复用已创建的线程，来减少 创建，释放 线程的消耗
- 提高响应速度，直接处理任务，节省了 创建线程所需要的时间
- 提高线程的可管理性， 由线程池来管理，调度。合理使用线程池，能避免线程的随意创建 释放，增加了系统稳定性；控制线程数量，避免 资源耗尽

拒绝策略
。。。java的吧。
。网上也看了下： 线程池 线程已经达到核心数，且 等待队列已满，且 线程数没有达到上限，新提交的任务 会导致 创建线程，使用 新创建的线程 来运行 该任务。
。 如果 无法创建线程，那么 会使用 拒绝策略

- AbortPolicy，抛出异常，提示 线程池已满
- DiscardPolicy， 无异常，(无事发生)
- DiscardOldSetPolicy， 将 等待队列中的 第一个任务 替换为 新任务
- CallerRunsPolicy， 调用者执行 该任务

如何配置线程数

- CPU密集型： CPU物理核心数+1
- IO密集型： 阿姆达尔定律

### 进程池 vs. 线程池

概念

- 进程池，进程池是管理资源进程的一种技术，由 资源进程 和 管理进程 组成。 管理进程 负责 创建 资源进程，并将 工作 分发给 空闲的 资源进程 处理， 还负责 回收 已经完成工作的 资源进程。
- 线程池， 多线程处理形式， 维护了一定数量的线程，并等待 任务。线程池中的线程都是 后台线程。它们从任务队列中 取出任务 并执行，执行完毕后把 结果放入 完成队列中。

资源共享与独立性

- 进程池， 每个进程 有独立的地址空间， 进程间无法直接共享数据，需要通过 IPC 才可以，比如 信号，管道，消息队列等
- 线程池， 共享同一进程的地址空间，因此它们可以方便地 共享数据和 进行通信， 这也带来了 线程安全性问题，需要使用 同步机制来保证 线程之间的 数据访问安全

创建与销毁开销

- 进程池， 进程创建和销毁 开销较大，进程池 适合那些不需要频繁 创建和销毁进程的 场景。
- 线程池， 创建和销毁 开销相对较小，但频繁创建和销毁 仍然 会 对性能产生影响。

使用场景

- 进程池， 适合那些 需要独立进程空间，计算密集型 或 IO密集型任务。
- 线程池， 适合那些需要 频繁创建和销毁线程，需要 共享数据和通信，或 对性能要求较高的 场景

安全性与稳定性

- 进程池， 进程间相互独立，所以不需要担心 线程安全问题， 但IPC 会增加 系统复杂性 和 出错的可能性
- 线程池， 池中的线程 共享一个地址空间，因此需要 特别注意 线程安全性， 通过合理的 同步机制，可以确保 线程池的稳定性和高效性

### mmap的用处

- 映射文件， 减少文件拷贝，提高IO效率。 0拷贝：mmap+write， sendfile
- 共享内存， 进程间通信， nginx，使用 accept_mutex 解决惊群问题
- 申请内存， 可动态扩缩， malloc，大于128k 直接使用 mmap。
- 动态库加载， 节省内存

### 进程间通信的方法

- 文件
- 信号
- 信号量
- pipe
- socket
- socketpair
- 消息队列
- 共享内存

第三方

- redis
- kafka
- rabbitmq

### TCP三次握手，在握手时交换哪些信息

- 序列号
    
- MSS (防止在IP层分片，重传机制在TCP层生效，如果IP层分片 并 丢包，会使得TCP 重传 整个大包)
    
- 保证可靠性 和 流量控制维护的某些状态信息
    
- 3次握手才可以 阻止 重复历史连接的初始化
    
- 3次才可以 同步双方的初始序列号
    
- 3次 可以避免资源浪费
    

### 用户数据复制到磁盘需要经历哪些缓冲区

- 用户缓冲区
- 内核缓冲区
- 磁盘控制器缓冲器 (可能涉及)
- 磁盘缓存 (如果可用)

### CPU使用率过高，如何排查

一般原因：

- 内存消耗过大，Full GC 次数过多
- 代码中有大量消耗CPU的操作 ( 如 自旋锁， 死循环)
- 由于锁 使用不当，导致死锁
- 随机出现大量线程访问接口

先确认是一台使用率过高，还是都过高， 只有一台，那么 top 察看哪个进程，然后看 对应日志。

询问业务，是否有 活动。

## move的作用和原理

C++11的特性

作用只有一个，无论输入参数 是左值还是右值，都强制 删除引用 然后转为 右值

```C++
template <class T>
LIBC_INLINE constexpr cpp::remove_reference_t<T> &&move(T &&t) {
  return static_cast<typename cpp::remove_reference_t<T>&&>(t);
}
```


## C语言的main方法和命令行之间的参数传递机制

BV1DaHTeDEaC


`int main(int argc, char** argv, char** envp)`

argc 就是 ./ 后面的单词个数，是 argv参数个数+1， 1是因为`./文件名` 也是一个参数。
argv 是 命令行参数 的字符串数组
envp 是 命令行的上下文环境参数， 是 键值对格式 的 string数组

打印所有命令行参数
```C++
int printf(const char* fmt, ...);

int main(int argc, char** argv, char** envp)
{
    int i;
    for (i = 0; i < argc; ++i) 
    {
        printf("%s\n", argv[i]);
    }
    return 0;
}
```

`int main(int argc, char* argv[], char* envp[])` 也可以。

一般envp不使用，所以可以省略 envp 这个形参


命令行执行程序的时候，会
1. fork() 一个进程
2. 新进程执行 execve(), 这个系统调用会把 ELF文件 加载到 进程的 用户态内存空间中 。。 elf: executable and linkable format
   构建程序运行上下文，栈底 -> 栈顶 分别是： envp的内容，argv的内容，argc的内容
   64位机器，参数较少的情况下，直接使用 寄存器传递(依次使用 rid,rsi,rdx,rcx,r8,r9)，而不是 栈
3. 调用 _start.s，这个方法会寻找 argc,argv,envp 的地址。并将 argc的地址放入 rdi寄存器， argv -> rsi, envp -> rdx。 这个是 Intel x64架构
   arm64架构，分别放到 x0,x1,x2 寄存器。 arm64可以传8个参数，分别直接用 x0,x1...x8
   arm32，分别是 r0,r1,r2, 可以直接传4个参数，直接用r0 .. r3


OS 只保证 布局， 具体的地址(上述的第三步) 需要 链接器来处理 


Intel X64， _start.s 位于 `/lib/_start.s` ，arm64位去 `/arm64` 中看

视频主自己写的 _start.s 的汇编代码如下
```
.text
.global _start, main

_start:
    mov %rsp, %rsi
    add $8, %rsi        # argv
    mov %rsi, %rdx
1:
    mov (%rdx), %rdi
    add $8, %rdx        # envp
    test %rdi, %rdi
    jnz 1b

    mov (%rsp), %rdi    # argc

    call main
    mov %rax, %rdi
    mov $60, %rax
    syscall
.fill 5, 1, 0
```


---


`gcc -O3 -static asd.c`

-static 静态编译 表示将 C库中 启动main函数的 代码 编译到 main函数前面

`objdump -d ./a.out > 2.txt`，进行 反汇编

在 2.txt 中可以搜索到 `_start` 标号， _start 就是 启动入口
_start 中的代码会调用 `__libc_start_main` ，这个是 C库的 启动main 的代码

`__libc_start_main` 中并没有明确跳转main方法，而是通过 函数指针 来 执行main 的。 代码很复杂。 最后通过 gdb 跟踪


`gdb ./a.out`
`b _start` 打断点到 _start 标号
`b main`
`r arg1 arg2`  执行程序
会暂停在 _start 断点上
`disassemble _start` 反汇编察看代码，没有细讲
`info registers` 察看寄存器，大多数都是0，RSP是栈顶指针，不为0；rip是代码当前位置
`x/8x 复制的rsp的值(复制的第一个，但2个值相同的，就是一个类似 0x7ffffffffdf10 的地址)`， 可以看到 值是3， 这个3 就是 argc的值
`c`
`c` 2次c，跳过 上面的2个断点 ( _start, main)
`r 11 22 33 44` 重新执行
`info registers`
`x/8x rsp的值`  可以看到是 5   。。 而且 2次rsp的值 会差 0x10 ？

rsp的值之后就是 envp 的值   。。 不是 argv ？
所以可以
`x/32c 0xffffff` 察看。 64位 需要2个地址拼接的， 后面的是 高位。   32c是指从这个地址看是察看 32个字符。

`q` 退出 gdb

---

打印 envp
envp 以 空字符串 为结尾
`for (i = 0; envp[i]; i++) { printxxxx }`


---

gcc的 _start 在下面之一。。

liba.a
liba.so
liba.so.6


## C++单例，饿汉模式

BV1EEsNeYE5s

```C++
#include <iostream>

class Singleton {
public:
    static Singleton* getInstance() {
        return instance;
    }

    // 删除 复制构造，赋值操作符   。。 但是， 移动构造需要删除吗？
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;

    void showMessage() {
        std::cout<<"asd"<<std::endl;
    }

private:
    Singleton() {}  // 私有构造器

    static Singleton* instance;
};

// 静态成员变量初始化
Singleton* Singleton::instance = new Singleton();     // 。。 感觉第一次看到这种， 主要就是 不知道 Singleton* 是必须的嘛？

int main() {
    Singleton* singelton = Singleton::getInstance();
    singleton -> showMessage();

    return 0;
}
```



## 常见的内存泄漏场景

BV1qsHCewEBU

```C++
void func() {
    int *p = new int(10);       // 泄漏
}
```

```C++
void func() {
    char *p = new char[100];        // 泄漏
    strcpy(p, "asdads");  // or strncpy

    p = new char[100];
    strcpy(p, "zxczxc");
    delete[] p;
}
```

```C++
void func() {
    struct Node {
        int* value;
        Node* next;
    };

    Node* n = new Node;
    n->value = new int(10);   // 泄漏     // 可以 new int{10} 吗？
    n->next = nullptr;

    delete n;  // 没有 delete n->value;
}
```

```C++
void func() {
    int *p;
    *p = 43;    // invalid write,  p未初始化
}
```


`man strncpy` 查看 方法文档  Linux Programmer's Mannual


`valgrind --leak-check=full ./a.out`
检查内存泄漏
在 heap summary 部分， 多少个 alloc，多少个 free 。 方法必须被调用，不然不会出现的





## 随机数，不要用rand

BV1GHpueFE3S


C版本, rand 返回 0到 RAND_MAX 之间的一个数字
```C
std::cout<<RAND_MAX;
```
获得随机数最大值

C获得随机数
```C
std::cout<<rand();
```
多运行几次，结果相同，都是同一个 数字

C修改随机数种子
```C
srand(time(nullptr));
std::cout<<rand();
```


---

C++

分为 随机数引擎，随机数分布

需要 `<random>` 库

```C++
#include <iostream>
#include <random>

int main() {
    std::random_device rd;
    std::uniform_int_distribution<int> dist(1, 9);
    std::cout<<dist(rd);  // 1-9之间的随机数
}
```

直接 `cout<< rd()` 也可以，但不推荐。  。。是一个 类似 rand() 的 大的数值

`std::uniform_real_distribution<double> dist(0.0, 1.0);`  0.0 到 1.0 之间的随机数 。修改2个地方，  int -> real， int -> double


`std::ranges::shuffle(v, rd);` 乱序 `vector<int>`


---

## C 动态数组

```C
int main(int argc, char* argv[]) {
  int *ptr = (int*) malloc(sizeof(int) * 10);
  int *maxptr = (int*) malloc(sizeof(int) * 20);

  for (int i = 0; i < 10; ++i) {
    ptr[i] = i + 1;
  }

  // 输出 ptr 1-10

  memcpy(maxptr, ptr, sizeof(10) * 10);

  // 输出 maxptr 1-20

  return 0;
}
```



## malloc 底层原理

BV1BpsTeBE8m

。被评论说的好惨，确实 这个有毛用，知道 申请内存就ok了，看底层有什么用？
。也没有 分OS， win还 brk sbrk？  mmap算底层吗？

malloc 分配的是 虚拟内存。 

只分配，==不使用的话 不会 映射到 物理内存==

`void* malloc(size_t size);`

stdlib (C库) 维持了一个 内存池
size < 128k， 从内存池中获取内存。 如果内存池中不够，则 brk 系统调用，brk会从 堆中分配内存
size > 128k， 直接 mmap 从 文件映射区 获取。


。。
https://www.man7.org/linux/man-pages/man3/malloc.3.html

Normally, malloc() allocates memory from the heap, and adjusts
the size of the heap as required, using ==sbrk==(2).  When allocating
blocks of memory ==larger than MMAP_THRESHOLD== bytes, the glibc
malloc() implementation allocates the memory as a private
anonymous mapping using ==mmap==(2).  ==MMAP_THRESHOLD is 128 kB== by
default, but is adjustable using mallopt(3).
。。
。brk是绝对值， sbrk是偏移量
。。
https://zhuanlan.zhihu.com/p/670532733
还有其他的例子，就是执行malloc后会如何，再次malloc，free

![161c3ea3c5b2a0cd9fcff37004dd4c7c.png](../_resources/161c3ea3c5b2a0cd9fcff37004dd4c7c.png)
。。


系统为每块分配的内存增加一个 16字节的头部，free时使用这个头部来确认 需要释放多少内存。
。。我记得 还有一个 脚标 但是不知道是什么地方用的了

### C++的内存划分

- 栈
  保存: 局部变量，函数参数，返回地址，栈帧
  自动分配和释放。调用函数时创建栈帧，从函数返回时 释放栈帧。
- 堆
  保存: 动态分配的内存
  由new/delete, malloc/free 手动管理
- 全局/静态存储区
  保存: 全局变量，static静态变量
  可以分为: 已初始化数据区 .data，未初始化数据区 .bss
- 常量
  保存: 字符串字面量，const修饰的全局常量和静态常量
- 代码
  保存: 程序的可执行代码


### C内存划分

- 栈
- 堆
- 全局静态
- 代码

### 虚拟地址空间划分

- 内核虚拟地址空间
- 用户虚拟地址空间
  - 栈
  - 文件映射
  - 堆
  - 未初始化数据
  - 已初始化数据
  - 代码段



## 虚函数 vs. 纯虚函数


1. 是否有方法体
2. 所在类是否可以实例化
3. 子类是否必须提供自己的实现


虚函数，适用于 需要提供默认实现，但允许 子类重写的情况

纯虚函数，用于 父类不提供实现，强制 子类实现。 如 接口，抽象基类




## safe c++, cppcon, Bjarne Stroustrup

BV1VFyAY2Er7
1.5h

不要写 C/C++， 写纯C++

类型安全 type safty: 每个对象 都是 根据它的类型 来访问的
resource safty: 每个对象 被正确地 构造 和 销毁
memory safty: 每个指针 要么指向 有效的对象，要么是 nullptr
一些运行时检查: 不能对 空指针 进行 反引用。 通过下标访问指针时 必须在范围内。

安全不仅仅是 类型安全，还有其他的错误需要处理
- 逻辑错误
- 资源泄漏
- 并发错误
- 内存corruption:  数组越界，内存已delete
- 类型错误: void*
- 溢出(下溢，上溢)，非预期的转换
- timing error
- allocation unpredictability (不可预测的分配)
- termination error


区分 safety 和 security

security是指: 获得你的源码，获得数据库数据，获得你的电脑。格式化你的磁盘。
security涉及，计算机，人，存储，物理事物。


---

不可变

const


---

RAII

1988年

---

使用 range for，span， 这样 没有变量来 犯错。


---

使用
- RAII
- const
- container
- resource management pointer
- algorithm
- range-for
- span

避免
- 源生指针
- 指针下标
- 反引用前 不检查
- 未初始化的变量




## c-style string

```C++
class MyString {
private:
  char* data;

public:
  MyString(const char* str = nullptr) {
    if (str) {
      data = new char[strlen(str) + 1];
      strcpy(data, str);   // strncpy 更好。
    } else {
      data = new char[1];
      *data = '\0';
    }
  }

  MyString& operator=(const MyString& other) {
    if (this == &other) {
      return *this;
    }

    delete[] data;

    data = new char[strlen(other.data) + 1];
    strcpy(data, other.data);

    return *this;
  }
};
```

operator= 时 判断 是否自赋值




## static的5个用处

- 局部变量上使用 static
- 全局变量 使用 static
- 全局函数使用 static
- 类数据成员 static
- 类方法成员 static

---

静态成员变量的初始化==必须在类外==，且不带static (带static是重复定义)
`int MyClass::myStaticVar = 0;`
`int MyClass::myStaticVar2 {0}; // C++11` 



---

static 局部变量

staic 局部变量 在 函数调用结束后 不会销毁。
即， static局部变量的 生命周期 是整个程序运行期间。 static局部变量的 作用域 仍然是 函数内部

```C++
void fun() {
  static int cnt = 0; // 只会创建一次。
  cnt++;
  std::cout<<cnt<<std::endl;
}
```

staic 局部变量 存储在 数据段中:
已初始化的 static 局部变量，存储在 .data 
未初始化的 static 局部变量，存储在 .bss

未初始化的 static局部变量，程序启动时 会初始化为 0值。

---

static 全局变量、全局方法

使得 该全局变量/方法 只在 本翻译单元中 可见

可以提供一定程度的封装。
但是 一般直接使用 命名空间 来封装。

---

static 类变量,方法

static 成员函数 不能访问 非静态成员。
调用 static 成员函数 不需要 初始化一个 类成员。




## 隐式转换

C可以 `int* p = 1;`， C++不行

---

内置

`int a = 1.f;`
`int a = 1L;`

---

类方法


```C++
void f(const std::string&) {}

int main() {
          // f 必须有 const 才可以 "123"， 没有const 的话，必须 传入std::string
          // 左值ref 无法ref 右值， const左值ref 可以 ref右值
  f("123");  // const char 数组。  可以通过 string的 转换构造函数 来转换为 std::string
}
```


```C++
struct X {
  operator int() { return 0; } // 转换函数 用于隐式转换
};

struct Y {
  Y(int) {}  // 转换构造函数
};

void f(Y) {}

int main() {
  int a = X{};

  f(100); // 100 隐式转为Y(100)， 等价于 f(Y{ 100 });
}
```

---

继承 也会 隐式转换

```C++
struct X {};
struct Y : X {};

int main() {
  X x = Y{};
}
```



## 命名空间，名字查找，作用域解析符

无限定名查找
有限定名查找

```C++
using namespace std;
struct pair{};

int main() {
  pair p;   // 编译失败，二义性， 无限定名查找
}
```

```C++
using namespace std;
struct pair{};
int main() {
  ::pair p;  // 可以， 有限定名查找， 只查找 全局命名空间。
}
```

---

```C++
namespace { // 匿名命名空间
  struct std();
}
int main() {
  std::cout<<"1";     // 有歧义。 gcc可以编译， clang，msvc 编译报错。
  ::std::count<<"1";  // ok， 全局查找， 不会找 匿名
}
```

---

```C++
namespace X {
  inline namespace Test {
    struct std{};
  }
}
using namespace X;

int main() {
  std::cout<<"1";  // 有歧义，可以找到 X的内联命名空间Test中的 std
  ::std::cout<<"1"; // ok
}
```


## share_ptr

BV1PWyTY2ESm

问题
- 智能指针有哪些
- shared_ptr 底层实现
- shared_ptr 引用计数如何实现
- shared_ptr 和 unique_ptr 的区别
- 如何解决 shared_ptr 引起的 循环引用问题
- shared_ptr 是否线程安全
- 为什么尽量使用 make_shared
- shared_ptr 应用场景
- 如何从类的内部返回 this 的shared_ptr，原理是什么?


野指针: 未初始化的指针
悬空指针: 指向的内容已被释放的指针


- shared_ptr
- weak_ptr
- unique_ptr


构造 shared_ptr 可以通过 make_shared 也可以 将 裸指针传递给 shared_ptr 的构造器。


从类的内部返回 this 的 shared_ptr
```C++
class T: public enable_shared_from_this<T> {
public:
  shared_ptr<T> self() {
    return shared_from_this();
  }
};
```

原理: enable_shared_from_this 维护了一个 weak_ptr， 监控 this 指针。
shared_from_this 方法中 调用了 weak_ptr 的 lock 来获得 shared_ptr

---

weak_ptr 用来解决 shared_ptr 的循环引用， 不占用 计数

---


shared_ptr 有一个控制块，创建时 会使用。

---

ref count 是线程安全的， 但是里面管理的对象不是

---

通过make_shared， 实际对象 和 引用计数 的 内存 是在一起的， 只需要分配一次。

---




## deque底层

底层的通常的实现方式是: 一系列 单独分配的 固定尺寸 数组，外加额外的记录。 这意味着 对deque 的 索引访问 需要进行 2次指针解引用。



## lambda底层实现

编译器生成类。

- 注意 operator() 是 const方法 ( 成员函数const 表示不会修改类的成员)， lambda 要使用 mutable才没有const

下面的翻译是 https://cppinsights.io/

![2992299f0f6358c9b173c4187aa75f97.png](../_resources/2992299f0f6358c9b173c4187aa75f97.png)

![c2ee768f511337e9ca5b2090230aae22.png](../_resources/c2ee768f511337e9ca5b2090230aae22.png)


## const

4个地方
- 定义常量
- 修饰指针，指针不可变，指向的值不可变
- 修饰函数参数，函数内部不会修改传入的参数
- 修饰类成员函数，不会修改类的成员变量，如果修改，编译失败


## 定时器

BV1KgATecEDk

需要数据结构 组织所有延时任务(回调函数)
触发最近将要超时的任务

数据结构，需要考虑
- 能快速找到最近要超时的任务
- 触发后要删除任务，且支持 随时删除任务   。。这个随时删除 是指 随机删除?
- 允许相同时刻触发任务


数据结构 可选 最小堆，红黑树，时间轮。 
最简单的是 红黑树，因为 multimap 的存在。 。。map也行 `map<time, vector<function<int>>>` ..似乎不行，因为 function的 模板参数不同，没办法 一个vector吧? 但是可以 variant, shared_ptr 等，反正肯定有办法 存到一个 vector 中。
最小堆不支持删除
stl没有时间轮

。。但是 multimap 很难删除啊，相同时间的话，怎么 随时删除?  总不可能对比 value (`function<int(int)>`)吧?
。。实际上，感觉 最小堆简单。
。。或者，为什么要 随时删除任务。

红黑树找最小值 O(1), 因为map2.begin()就是最小值   。。。但是真的O(1)? 这还真不知道。 就是 multimap中 有类属性 一直指向着 最小值? 感觉不可能吧， begin()应该是 O(logn) 吧。

---

任务使用 函数对象 std::function

---

延时 epoll_wait 或 timerfd

epoll_wait 的第4个参数 是 timeout。
timerfd 有3个接口，较难使用。

所以选择 epoll_wait

---

接口设计
- 添加延时任务
- 删除延时任务
- 检测触发延时任务

。。看了下， 添加任务后 返回一个 TimerTask*， 删除的时候 需要 这个 TimerTask*。
TimerTask的 run是 private (这样 拿到TimerTask* 也无法执行)， 将 Timer设置为 友元

删除是  multimap.equal_range(time) 然后 在返回的2个迭代器间 遍历。 比较 value，如果相同，那么 erase。

由于 Timer是 TimerTask的友元，所以 Timer可以调用 TimerTask的 run()。

。。如果 队列里是空的， 怎么办? 就是 Timer怎么延迟? timeout设置多少? 如果 死等的话，怎么 唤醒?  所以似乎需要一个 cv?

---

优化
1. 针对不同功能，不同对象: 设置不同定时器
2. 针对多线程: 一个事件线程一个定时器 或单独时间轮线程。  。。就是之前是在 定时器线程上执行的，现在 让定时器 调用线程池去执行。
3. 针对有规律的定时任务(大量相同间隔的延时任务): 如果队列为空，或新任务的 时间 大于 multimap.crbegin()的时间，那么 multimap.emplace_hint(multimap.end(), time, task);  。。不过感觉和前半部分的问题没什么关系。。





## 动态链接库的链接及缺少符号


```shell
# 制作 动态库
gcc -shared -fPIC mylib.c -o libmylib.so

# 调用动态库 方法1
gcc main.c -L. -lmylib -o main
export LD_LIBRARY_PATH=.:$LD_LIBRARY_PATH
./main

# 调用动态库 方法2
gcc -W1,-rpath=./ main.c -o main -L./ -lmylib
./main
```


缺少符号时
```shell
# 查看程序搜索库的路径
LD_DEBUG=libs <executable file>

# 是否有该库
sudo find / -name <lib file>

# 递归查找所依赖的库是否存在
ldd -r <lib file>

# 设置环境变量
export LD_LIBRARY_PATH=/path/to/directory:$LD_LIBRARY_PATH

# 读取elf 中rpath路径
readelf -d <lib file> | grep rpath

# 检查库的导出符号
nm -gDC <lib file> | grep -i <symbol>

# 如果存在命名修饰，获取符号的原始名称
c++filt <symbol>
```


---

评论:
- 把错误信息放 搜索引擎 搜一下，然后 动态库 引用对应的库
- 使用LSP; 未定义 一般就是 连接有问题，可能不存在，可能没连对， 也可能 引用库时 写错引用内容，或引用格式错误
- 模板类的实现也容易导致 未定义标识符，很烦
- 库路径没配好; dll没放好; 未链接到特定库





。# cpp end


# java

## threadlocal造成内存溢出，数据污染

线程完成后，不会清空 ThreadLocalMap，导致 下次复用这个线程时，可以通过 threadlocal 获得上次运行的数据。
所以造成数据污染， 可能造成内存溢出

。内存溢出是指: threadlocal 对应的是 一个map，然后每次往里面塞东西，这样迟早会用完内存。

在使用threadlocal的时候 要配合 try-finally 
```java
try {
  threadlocal.set(1);
} finally {
  threadlocal.remove();
}
```


# mysql/sql

## 运行原理

https://www.bilibili.com/video/BV1iv4y1S7nj

组件包括

- 存储引擎
- 连接池
- sql解析器
- sql优化器
- 缓存
- 集群
- 安全
- 恢复

---

InnoDB是默认的存储引擎，接收来自 mysql的命令，并返回结果。

---

sql 通过 客户端的 连接驱动 发送到 mysql 的 连接池中的某个连接上，然后 经过 查询缓存，sql解析，预处理，sql优化， 最终生成 执行计划。

执行计划，就是 发给 InnoDB 的命令

。。The buffer pool is an area in main memory where InnoDB caches table and index data as it is accessed
。。确实 buffer pool 是 innodb使用的。。。 我一直认为 它是 mysql的，不是 innodb的。。

InnoDB收到 执行计划 的命令后，对 命令进行 细化。
如果是读指令，那么会 读取 buffer pool， buffer pool 中有 自适应hash索引，可以存储热点数据。 如果 buffer pool中没有， 那么 buffer pool 负责 从 硬盘文件中 读取数据。
buffer pool 中有 free，flush，LRU 3个链表 来管理 这些 数据页 的写入位置，刷盘位置，数据页淘汰

如果是 写指令，那么先写入 buffer pool中的 undo buffer，然后写入 redo log buffer, 然后 写入 硬盘。在此之前，会判断是否需要写入 change buffer，后续 会合并到 buffer pool中。

为了 保证buffer pool中的数据 页 能完整写入 磁盘， 会先写入 double write buffer， 然后写入 磁盘的 double write 中， double write 写完后， 把数据 刷新到 磁盘，就完成了 写操作。

---

buffer pool中有 锁信息，数据字典
硬盘 有 独立表，通用表，系统表，undo表，临时表 空间

![142c35351fbf4e5e7b4c7836170095da.png](../_resources/142c35351fbf4e5e7b4c7836170095da.png)

---

## sql常用命令

![0ada1997413f9facfcff7a98c45155fa.png](../_resources/0ada1997413f9facfcff7a98c45155fa.png)


---


## 日常工作中，如何进行sql优化、explain

BV1hqSaYrE8f

- 加索引
- 避免索引不生效的场景
  - 隐式的类型转换 导致索引失效
  - 查询条件包含 or，可能导致索引失效
  - like
  - 查询条件不满足 联合索引的 最左匹配原则
  - 在索引列上 使用 mysql的 内置函数
  - 对索引进行 列(。数值?)的运算(如 + - * /)
  - 索引列上使用 !=, <>, 可能导致失效
  - 索引列上使用 is null, is not null， 可能失效
  - 左右连接，关联的字段编码格式不一样
  - 优化器选错了索引
- 避免返回不必要的数据
  - limit 减少数据
  - 指定列名，而不是 *
- 减少不必要的逻辑
  - 如果检索结果中不会有重复的记录，那么使用 union all 替换 union
- 批量
  - 写入的时候 批量写入/删除。 (。 就是insert的时候 把 几百条数据 拼成一条sql 来处理，而不是 一条数据一条sql)
- 读写分离
- 优化sql结构
  order表10万数据，customer表100条数据
  `select * from customer where id in (select customerid from order)`
  `select * from customer as a where exists (select 1 from order as b where b.customerid = b.id)`
  第一条是 先 in中的逻辑，所以 是 大表驱动小表
  第二条是 先 处理customer的逻辑，所以是 小表驱动大表。
  。。?第一条的in是 怎么处理的? 应该也是 先 in中的 计算出来，然后 每条customer 和 in的结果 对比。 也是 小表驱动大表啊。
- 分库分表
  分库分表会存在一些问题: 事务，跨库关联join，排序，分页，分布式id，  。还有 冷热数据不均匀
- 通过 explain 进行性能分析
  主要注意: type, rows, filtered, extra, key
  type 表示连接类型，性能从好到坏: system, const, eq_ref, ref, ref_or_null, index_merge, unique_subquery, index_subquery, range, index, all
  - system: db中只有一条数据，是const的 特例，一般不会出现
  - const: 通过一次 索引就可以找到数据，一般用于 ==主键或唯一索引==
  - eq_ref: 常用于 主键或唯一索引， 一般指 ==使用主键的关联查询==
  - ref: 用于 ==非 主键和唯一索引扫描==
  - ref_or_null: 类似ref，但是会额外搜索 null值得行
  - index_merge: 使用了 索引合并优化方法，查询使用了 2个以上的 索引
  - unique_subquery: 类似 eq_ref，条件使用了 in 子查询
  - index_subquery: 和unique_subquery的区别是: 使用了 非唯一索引，可以返回 重复值
  - range: 常用于范围查询，如 between and， in
  - index: 全索引扫描
  - all: 全表扫描



慢sql的排查思路
- 查看慢查询日志记录，分析慢sql
- explain 查看 执行计划
- profile 查看 执行耗时
- optimizer trace 分析详情


## 主从同步延迟

。出现于 异步复制(默认)。  不会出现于 同步复制，  还有一种 半同步复制 (至少有一个从库确认)

主从同步
- 从库 发起连接请求 到 主库，主库同意
- 主库将 binlog日志 发送给 从库 保存为 relay log。 从库后台线程处理 relay log


延迟的原因:
- 网络延迟
- 处理延迟(服务器压力大， 大事务)


查看
- 从节点 执行 `show replica status`
- `Seconds_Behind_Source` 查看 延迟状态。 0代表无延迟


解决方案
- 检查 优化 主从之间的 网络环境
- 查看 主从服务器 是否 有压力，提升硬件
- 是否有大事务，尽量拆分
- 修改 主从同步的机制
- 读取主库



## innodb行锁如何实现

BV1nvcHepEKN

通过对 索引数据页 上的记录 加锁实现，主要有3种
- record lock， 锁定单个行记录的锁
- gap lock， 锁定索引记录间隙，确保索引记录间隙不变
- next-key lock， 记录锁+间隙锁的组合，锁住数据和前后范围。

这意味着，只有通过 索引 检索数据时，才进行行锁， 否则将使用 表锁
。。但是 2个 sql 走不同的 索引呢?  所以 最终 加锁应该加载 聚簇索引上?  但我要锁 name 字段，到 id 岂不是很分散? 而且这种情况不太可能锁id吧。
。。似乎不是， 因为锁 应该是指 select * from xx for update 吧?  都是 insert的话，不太需要这里的锁，  应该只需要 写锁。

在RR隔离级别， innodb对 记录加锁 的行为 都是 先采用 next-key lock，但是 当sql 含有唯一索引时，innodb 会对 next-key lock优化，降级为 Record lock。

---

各种加锁操作 及特点
- `select ... from`， innodb使用 MVCC 实现 非阻塞读，所以 对于 普通select，innodb不加锁
- `select ... from lock in share mode`， 共享锁，使用 next-key lock进行处理，如果 发现唯一索引，降级为 record lock
- `select ... from for update`， 独占锁， next-key lock， 如果发现唯一索引，降级为 record lock
- `update ... where`， 使用 next-key lock， 如果唯一索引，将为 record lock
- `delete ... where`， 使用 next-key lock， 如果唯一索引，为 record lock
- `insert`， 将要插入的位置 设置 排他的 record lock

---

以 `update t1 set name='lisi' where id=10` 为例， 分析 innodb 对不同索引的 加锁因为， 以 RR隔离级别为例
- id是主键: 仅在 id=10 的主键索引上 加 X锁
- id是主键 且 unique(id): 在 唯一索引id 上加 X锁，在主键索引上加 X锁
- id是不唯一，id上有索引: 对索引，记录都加X锁，并增加间隙锁 ![ca359ff758457399ada52fe27211fa13.png](../_resources/ca359ff758457399ada52fe27211fa13.png)
- id上没有索引: 所有行 和 间隙都加 X锁。 (没有索引时，全表锁定)


---



## %前置模糊查询 优化

BV117w5eyEQ5


- 新建列 保存 反向字符串， 建立索引， 使用 reverse() 方法，  用的不多
- 业务限制查询数据量， 增加 where的条件，来缩小 范围。
- 走覆盖索引， 。。很奇怪，似乎是指 将 (id, name) 建立联合索引，然后 select id,name from xxx where name like '%asd' 可以走到这个索引。  。。但是没什么意义吧。。。 评论有人问了， 有人说 减少了回表次数，但是在 索引树中 还是要全量搜索。 ok 有道理。 。。 但为什么不直接建一个 name的索引呢? 但是走不了这个 name的索引啊， (id,name) 的索引 也只能走不了吧，因为 id不是查询的条件啊。。 所以 在 name上建立索引 select name from xx where name like '%asd' 也可以 不回表，直接 覆盖索引。
- 减少查询次数， 放redis里。
- 查询频率较高时， 使用分布式搜索组件，如 ES。



## 为什么不建议使用NULL作为默认值

BV1Zvw5eAEfU

1. 
搜索不能用 =null， 而是要用 is null, is not null

2. 
`select * from aa where a=20 or a<>20`
是无法搜出 a 是NULL的 列的。需要 增加 `or a is null`

3. 
NULL不会被索引。

4. 
可能违反 业务逻辑 或 数据库约束


在表定义中 增加 not null的约束 来 排除null




## 分布式主键

3个问题:
- 分布式环境，为什么不推荐自增主键?
- UUID可以做主键吗? 会存在什么问题?
- 雪花算法可以做主键吗? 原理? 优缺点? 如何解决缺点


---

单库使用 自增主键(auto_increment) 没有问题，

分布式，
- 会生成重复id，数据查询有问题
- 无法根据id分库
- 数据库同步时，可能主键冲突

---

UUID 可以用来做主键，但有点小问题: 由于 UUID 不是有序的，所以导致 插入到 B+树中间，可能导致 page分裂。
。。page 默认16k
。。话说 mysql 的 B+ balance 怎么搞? 不然不平衡啊

MySQL 8.0 有 BIN_TO_UUID()方法， 可以将 二进制转为 UUID字符串。 也可以保证 UUID有序。

。。https://dev.mysql.com/blog-archive/mysql-8-0-uuid-support/
。还真有序，这里 搜 order。可以看到
`Setting the argument to true while inserting the values: “INSERT INTO t VALUES(UUID_TO_BIN(UUID(), true));” will rearrange the time-related bits so that consecutive generated values will be ordered.`


---

雪花算法

可以做主键

---

特征
- 全局唯一
- 趋势递增 (但 不是单调递增，因为有 时钟回拨问题)
- 信息安全 ( 不规则，无法预测下一个ID)

---

原理
64位，long long
- 首位， 符号位始终0
- 41位， 时间戳(毫秒)， 一共68年， 默认从1970年1月1号开始，2038年用完，所以 一般要设置 系统上线时间，然后 时间戳是 基于 这个 上线时间的。
- 10位， 机器码， 所以支持 1024 个服务实例
- 12位， 序列号， 4096个号

---

问题
- 依赖时间戳，所以 会 受到 时钟回拨问题。

解决
- 直接抛出异常， 如果取到的时间 小于 上次时间戳，则抛出异常。
- 延迟等待， 发现 时钟回拨，就等待一段时间。
- 最好的: 将机器码拆分为 3位时钟序列 + 7位机器码， 发现时钟回拨时，增加 时钟序列。

。。为什么不是 时间戳永不回退。 并且 12位序列号满了以后，时间戳没有变的话，强制+1ms。
。而且 3位时钟序列 也会满的。

为了避免重启 引起时间序列丢失，可以将 始终序列通过DB/缓存 存储起来。
。。。不太可能啊， redis再快，也是一次 网络IO。
。。而且 重启，至少也要 几十秒，不会那么倒霉 正好 时钟回拨 的时候 重启，并且 重启后 的时间 还是 在 之前的 时间 之前吧。

。。而且 id 是主键，重复的话，DB会报错的，所以 没有太大的问题。








。# mysql end


# network

## http1, 1.1, 2, 3

https://www.bilibili.com/video/BV1h4p3eeETP

传输文本，图片，视频
还用于 API，文件传输，以及基于网络的服务

---

1996年，http从 0.9 进化到 1
http 0.9 很简单，只支持 GET，没有 header 没有 status code，服务器只能发送 html文件

http 1 增加了 header，status code，及 POST，HEAD

1.  tcp 3次握手
    1.  c->s: tcp syn client -> server
    2.  s->c: tcp syn+ack
    3.  c->s: tcp ack
2.  tls 握手，
    1.  c->s: client hello
    2.  s->c: server hello
    3.  s->c: certificate(证书)
    4.  s->c: server hello demo
3.  key 协商，
    1.  c->s: client key exchange
    2.  c->s: change cipher spec
    3.  c->s: finished
    4.  s->c: change cipher spec
    5.  s->c: finished
4.  数据传输
5.  tcp 挥手

每个资源(图片，css文件，js文件)都会 执行一遍上面的过程，所以很慢。

---

1997， http 1.1， 至今仍然在使用，因为它提供了一些新功能

1.  ==持久连接， persistent http connection==， 除非告知关闭，否则一直保持打开状态，这样就不需要在 每次 请求后关闭连接， 不需要多次tcp握手。
2.  ==管道，pipelining==， 允许客户端 通过 一个tcp连接 发送多个请求，它无需等待响应。。。 以前是 必须上一个请求 被 server回应后 才可以下一个请求，现在可以 直接发送多个请求。
3.  ==分块传输，chunk transfer encoding==， 服务器可以 以 较小的块 为单位 发送响应，不必等待整个响应 准备就绪。
4.  更好的 ==缓存控制(Cache-Conctrol) 和 条件请求(ETag)==
    1.  If-Modified-Since 等

http 1.1 有一个大问题： ==管道阻塞 pipeline blocking== 。。 大意应该是： 管道 允许 发送多个请求，无需等待响应， 但是有限制：服务器是 按顺序处理的，所以 如果 第一个请求 需要很长时间，那么会导致 后续的请求 也阻塞。 。。 不过这种 至少比 1 好啊， 1 也会阻塞吧？ 不不不，1 是 新tcp连接，所以 后续的请求不会阻塞？

这个问题，导致很多 浏览器 无法使用 管道。 开发人员找到了一些方法来 绕过这个限制

1.  ==域名分片，domain sharding==， 网站可以从 子域(static1.baidu.com, static2.baidu.com ...) 提供 静态资源， ==每个新的子域 都会 多出 6个连接==。
2.  ==捆绑资源 减少请求==， 合并图片，合并css js

---

2015 http 2

旨在解决 http 1 的性能问题

1.  ==二进制帧，binary framing layer==， http2.0(包含二进制帧) -> tls(可选) -> tcp -> ip
    ==和http1 的纯文本不同， http2 使用了 二进制格式==
    信息被划分为 更小的单元，称为帧
2.  更完整的请求和响应==多路复用==
    http 1.1 的 多路复用 是 同时请求多个资源
    http 2 借助帧，将 http消息 分解为独立的帧， 这些帧可以在 传输过程中 混合，并在 另一端 重新组合。这解决了 http 1 中的 头部对齐阻塞问题(。就是 管道阻塞，第一个请求阻塞导致后续的 阻塞)
    。。 所有请求和响应 都可以在 一个 tcp连接上进行。
3.  ==流优先级==， 资源的加载顺序非常重要， 流优先级 可以让 开发人员 设置 请求的重要性。 浏览器告知服务器 哪些资源具有 高优先级，服务器会为 这些重要请求 分配/发送 更多的帧，
4.  ==服务器推送，server push==， 允许 对客户端的请求 进行多次响应。 。例子是：之前必须 客户端请求 index.html，请求 styles.css， 现在 客户端请求 index.html,服务器可以 直接返回index.html 和 styles.css。
5.  ==头部压缩，head compress，hpack算法==。 http1 只有 body会被压缩，header是纯文本形式。 http 2 使用 hpack 来压缩 header
    hpack 会压缩 header 并 记住 之前的 header，利用这些信息 进一步压缩 未来的 header
    。。下面是网上搜的
    https://blog.csdn.net/qq_62311779/article/details/139873507
    静态索引表， 预定义的头部字段表，包含了 常见的 http头部字段，这些字段对于所有的 http2 都是相同的，所以传输是 只需要传输 `数字下标`， 而不是 以前的 `key:value`。 这样就节约了字节。 常见的 静态表条目包括：`:method: GET`, `:path: /index.html` 等
    动态索引表， 存储 连接期间 动态更新的 头部字段表。 客户端，服务器可以向 动态表 添加新的 头部字段，并在后续的 请求 或 响应中 引用这些字段。

随着 网络应用 越来越复杂，并且 移动互联网的普及，http 2 出现了一些限制

1.  标头阻塞 导致 tcp重发，导致 页面加载速度变慢。 这在 高延迟 或 有损 网络中 尤为明显。 这导致了 2022 的 http3

---

2022 http 3

使用 quic 代替 tcp

quic 由 google开发， 基于 udp

http3，减少了延迟， 改善了多路复用， 消灭了 tcp标头阻塞。

更好地处理 数据包 丢失。

在 移动网络上 表现更好，可以 无缝切换连接

客户端使用 http 3 连接 服务器时，会启动 ==quic握手==。
tcp 需要 3次握手， quic 需要2次： ==c->s: initial,hello; s->c: initial,hello,cert,fin==;
quic 和 tls 1.3 结合 以确保安全， ==tls 握手 发生在 quic 握手中 (就是上面的 cert)==， 这样可以减少 整体的 延迟。

quic非常快，在 0rtt的情况下， 服务器无需握手 即可处理请求。

http 3 还可以很好地处理 网络变化， 从 wifi切换为 蜂窝网络(，就是流量)， http3 可以继续保持连接。 ==这归功于 quic 的 connection id，这些不依赖于 ip==

---

http1.1 仍然 广泛使用(约20+%)，尤其对于 简单的网站
http2 已经广泛使用， 60% 的网络请求 都是 http2
http3 正在普及 (约 10+%)

---

## 字节-一个网页特别慢，可以从哪些维度排查

BV1eDtLewE7s

打开网页的过程

1.  客户端访问URL
2.  DNS解析
3.  TCP连接
4.  发送HTTP请求
5.  服务器处理请求
6.  返回报文
7.  浏览器解析，渲染页面
8.  TCP断开连接

可能的原因

- DNS解析慢
    - DNS服务器被DDoS攻击
    - 网络延迟
    - 域名配置问题
    - DNS缓存问题
- 客户端慢
    - 客户端生成socket连接的时候 端口选择慢， linux 偶数端口分配给客户端，奇数端口分配给服务端。如果偶数用完，再新建客户端时，会遍历 奇数端口，都被使用了，才会 分配 偶数端口。
    - 缓存过期，重新下载页面
- 代理服务器
    - 单机负载低，并高并发 处理能力差
    - LB算法不合实际
- 服务器
    - 业务负载大
    - 没有压缩payload
    - 网页资源大，没有充分使用 CDN，客户端本地缓存
    - 网页代码不优化，冗余的代码，未压缩的文件，复杂的脚本
    - 服务器耗时业务， 服务器在跑 批任务，导致CPU不够用
    - 单机，集群不够大
- 网络环境
    - 突发流量
- 慢sql
- 缓存未优化
    - 保存到 浏览器端
    - redis + db
- 资源部署
    - CDN
    - 静态资源放 Nginx
    - 合理安排 资源加载顺序( ==先加载 用户体验影响大的==)，采用 异步加载技术
    - 高清图片，媒体文件，需要消耗时间来下载
- 渲染
    - reflow
    - repain
- js执行慢
- 交互
    - 先设置 骨架界面，等待数据加载完成后 重绘



## TCP粘包

TCP协议的传输过程中，多个数据包被粘在一起，接收端一次性读取 多个数据包的情况

为什么会粘包?
- TCP是面向流的协议， 将数据作为一个字节流进行传输，所以多个数据包可能会粘合在一起
- Nagle算法， 将多个小数据包 打包成一个 大数据包 来减少网络上的数据包数量。
- TCP数据包的拆分， TCP传输的数据包大小受 MSS限制。超过这个限制的报文会被拆分为多个数据包进行传输。接收方在接收这些数据包时，可能会在缓冲区中一次性读取多个数据包，造成 粘包


解决方案
- 固定长度协议，每次发送，接收 固定长度的数据。 数据不够用0填充
- 分隔符协议，数据之间增加特定的分隔符
- 报文头部增加长度信息，
- 应用层协议设计，设计一个应用层协议，确保接收端能正确地解析接受到的字节流数据



## TCP vs. UDP

TCP:
- 面向连接
- 可靠
- 流量控制
- 拥塞控制
- 数据传输顺序: 保证 接收顺序和发送顺序一致


UDP:
- 无连接
- 不可靠传输
- 无流量控制
- 无拥塞控制
- 数据传输顺序: 不保证 接收，发送顺序一致


## proactor，reactor

Proactor 和 Reactor 是两种不同的异步 I/O 模型设计模式，它们用于处理并发操作，尤其是在网络编程中。
两者的 主要区别 在于
- 如何管理和调度 I/O 操作以及
- 何时通知应用程序 I/O 事件的发生。

---

Reactor 模式

- **同步 I/O 多路复用**：Reactor 模式通常使用同步 I/O 多路复用技术（如 `select`, `poll`, `epoll` 等）来监听多个文件描述符的就绪状态。
- **事件分发**：当某个或某些 I/O 操作准备好时（例如，数据可读、可写等），Reactor 会将这些事件分发给相应的事件处理器（Handler）。应用程序代码在事件处理器中执行具体的 I/O 操作。
- **主动调用**：应用程序需要主动调用 I/O 操作，比如读取或写入数据。
- **应用场景**：适合于 I/O 密集型的应用场景，比如 web 服务器等。

---

Proactor 模式

- **异步 I/O**：Proactor 模式依赖操作系统提供的异步 I/O 能力。在这种模式下，I/O 操作（如读写请求）被直接提交给操作系统，由操作系统完成实际的 I/O 操作。
- **回调机制**：一旦 I/O 操作完成，操作系统通过回调函数通知应用程序。这种模型允许应用程序仅关注业务逻辑，而无需关心底层的 I/O 操作细节。
- **被动接收通知**：与 Reactor 不同，Proactor 模式下应用程序是被动地接收 I/O 完成的通知，而不是主动发起 I/O 操作。
- **应用场景**：由于其高效性，Proactor 模式非常适合高性能、低延迟需求的应用程序，特别是那些能够充分利用异步 I/O 的系统。

---

总结

简单来说，Reactor 模式让应用程序等待事件发生并自行处理这些事件，而 Proactor 模式则让操作系统处理 I/O 操作并在完成后通知应用程序。选择哪种模式取决于具体的应用需求、性能考虑和可用的系统资源。在实践中，许多现代框架可能会结合这两种模式的优点以适应不同的应用场景。




。# network end

# computer/OS

## 进程与线程的区别

https://www.bilibili.com/video/BV12RHfedEFC

首先要了解 什么是程序

程序时 可执行文件， 它包含 作为文件存储于 磁盘上的 代码 或一组处理器指令

程序中的代码 被加载到 内存 并由处理器 执行时， 它便成了一个 进程

进程还包括 程序运行所需的资源，这些资源 由 OS管理，例如， 处理器，寄存器，程序计数器，堆栈指针，分配给进程的堆 和堆栈的内存页 等。

进程有一个重要的属性： 每个进程 都有自己的 内存地址空间。 进程无法影响其他进程，这意味着 一个进程出现问题时，其他进程会 继续运行。

chrome就是 多进程的， 每个标签一个进程， 所以当 一个网页有恶意代码时，不会影响其他网页。

---

线程是 进程内的执行单元， 一个进程 至少有一个 线程，被称为 主线程。

每个线程都有自己的 栈，寄存器，程序计数器

进程内的 线程 共享内存地址空间，可以 通过该 共享内存空间 在 线程间 进行通信

一个行为不当的线程 可以导致 整个进程 崩溃

---

OS 如何在 CPU上 运行线程 或进程呢？

通过上下文切换 来处理。

在上下文切换期间，一个进程会从 CPU中切换出来，以便另一个进程可以运行。
OS存储 当前正在执行的 进程的状态，以便稍候可以 恢复并继续执行该进程

上下文切换的代价是 昂贵的，它涉及 保存和加载 寄存器，切换内存页面，更新各种内核数据结构

线程之间的切换 也需要 上下文切换。 线程间上下文切换 比 进程间上下文切换 更快。 由于 线程共享 内存地址空间，所以 无需 切换 虚拟内存页面，这是 上下文切换 期间 最昂贵的操作之一。

---

上下文切换的 成本非常高，因此有其他机制可以尝试 将其最小化。比如 纤程，协程，这些机制 以复杂性 换取 更低的 上下文切换成本， 一般来说 它们是合作调度的，即 它们必须让出控制权 让其他运行， 换句话说，==应用程序本身处理任务调度==


---

## CPU中断和异常处理机制

BV1aMteeCE8Z



浏览器输入url后到显示的全过程，需要注意的知识点
- DNS，本地host缓存
- 通信协议
  - tcp握手
  - tls的密钥协商
- CDN
- 服务器
  - thread
  - memcache
  - db
  - io
- 资源解析
- js事件循环模型
- 浏览器渲染


---

键盘按下，发生了
1. 键盘硬件电路，某跟线高低电平转换
2. 键盘的芯片将按键值和电平变化，通过 USB，2.4g，蓝牙协议 发送给CPU
3. CPU怎么知道键盘被按下？
   1. 轮询
   2. 中断(外设中断，硬件中断)

CPU与OS
1. 中断处理， OS加载时 会注册 中断处理函数 到 寄存器(。应该不是寄存器吧？很贵的)

操作系统与浏览器
1. 用户程序，等待输入，即 open函数，read(0)，getc()。0是linux的stdin
2. 通过system call，转入内核态，调用驱动程序
3. 阻塞，等待输入响应(wait 信号量)
4. OS调用中断处理函数，调用 驱动程序的处理函数，来读取按键值 和事件(按下or弹起)
5. 唤醒 第三步的 输入等待 (使用信号量)
6. 拷贝按键值到用户空间
7. 第一步的 阻塞中的 用户进程 获得输入，并显示

---

中断分为
- 硬件中断
- 软件中断 (信号量)
- 定时器中断 (时间片调度)

---

网络传输，需要用到 网卡，OS，驱动，协议栈，应用程序

应用程序使用通过异常机制 系统调用

对于CPU来说， 中断和异常 非常相似。

---

RISC-V 的 异常/中断 机制

在 Privilege Levels 中，

RISC-V 中有3种 运行级别
- 00 U user/application
- 01 S supervisor
- 11 M machine

有一组专门用于控制和状态的寄存器： CSRs ( control and status registers)

。。介绍了 RISC-V 的 CSRs 的码的含义。。 pdf的书名是 The RISC-V Instruction Set Manual: Volumne II


中断可以关闭， 异常不行

。。太底层了，讲了 中断 陷入时 会 将 寄存器的值 移动 之类的。

mcause 寄存器，中断，异常。

汇编。
段 .text  .section  偏移

kernel.ld
kernel.elf.asm
start.s
。。构建 软件时 会把很多东西 编进去。 这些东西是 RISC-V 中用到的


---

中断嵌套， 在中断处理过程中，又发生了一个中断， 如果支持这种方式，那么就是 支持中断嵌套。如果不支持，那么进行中断处理前，先屏蔽中断。

异常不能屏蔽

中断优先级
1. 异常 最优先
2. 硬件中断
3. 软件中断
4. 定时器中断



---

后面都是 线路板设计了。

---



## 简述Linux启动过程

BV1sfpgeXEwM

按下电源按钮后
1. BIOS 或 UEFI 启动，
   功能是 让计算机 所有主要部分( 键盘，屏幕，硬盘等) 做好 操作准备。 
   UEFI是比较新的软件，和传统BIOS相比，UEFI提供更快的启动时间 和更好的 安全功能。  
   UEFI和BIOS的 主要区别是 它们的 磁盘存储方法
   - BIOS与 主引导记录(master boot record, MBR) 相关联，MBR将磁盘大小限制为 2TB
   - UEFI 使用 GUID分区表(GPT)，消除了 磁盘大小的限制。
2. BIOS/UEFI 进行开机自检 (POST检查)
   这个检查 确保 在完全打开所有设备之前 所有不同的硬件都正常工作
   如果POST发现问题，通常会在屏幕上 显示 错误消息
3. 自检完成，没有问题， BIOS/UEFI 查找并载入 引导加载程序。 引导程序的位置 可以自定义，通常是 先硬盘，然后USD 或CD
   1. BIOS中，引导加载程序 位于 硬盘驱动的 第一个小块中，称为 主引导记录
   2. UEFI， 有一个单独的分区 来存储 .efi 引导加载程序 等文件
   引导程序的关键工作是： 在磁盘上找到 OS，并将其加载到 计算机内存中， 开始运行内核代码。
   常见的 引导加载程序有： LILO(相当过时)，GRUB2
4. GRUB2 加载完毕，它就将 Linux 内核载入内存，并将==控制权交给内核== 以完成启动过程
5. 内核接管计算机资源 并开始 启动 所有后台进程 和服务
   1. 将自身解压到内存中，检查硬件，并加载设备驱动程序 和 其他内核模块
   2. 名为 init 的初始进程启动，在 现代 Linux中，该进程通常是 systemd。
   3. systemd 启动系统，为 使用做好准备。
      1. 检查是否由任何剩余的硬件驱动程序需要加载，挂载所有不同的文件系统和磁盘，以便访问。 
      2. 启动你需要的所有后台服务(例如，网络，声音，电源管理)。 
      3. 处理用户登录 。
      4. 将面板和菜单 加载到 你的桌面环境。
      systemd 使用目标配置文件 来决定它应该启动到哪种模式。




## Linux有哪些常见IO模型 及区别

BV14KaCe6EyX


。。我想到的是 select,poll,epoll。
。区别是： select每次需要传入全量的 监听的fd列表，我记得有3种，但只记得 读 写 了。  每次返回后，需要自己遍历 fd列表， 查看是否有新数据。
。poll， 是提供了一个 ctl 接口，自己注册/移除 需要监听的 fd，  返回后，也需要自己遍历 fd列表
。。 epoll，有 ctl接口，自己注册/移除 监听的fd，  方法会返回 有变化的 fd列表。
。。。不过我不知道 这3种方法的 具体api 的名字。
。还想起来一个最新的，  io_uring， 环形队列， 绝对异步。
。。
。。0分。。。


阻塞IO模型
非阻塞IO模型
IO多路复用模型
信号驱动模型
异步IO模型

前4个是 同步模型


### 同步IO

fd默认是阻塞的，可以通过 fctnl 来修改

==fd上的这个 设置==，就有了 下面的 同步==阻塞IO，同步非阻塞IO 的区别==


==同步异步 是指 数据拷贝 由谁执行， 用户线程执行 就是 同步，  核心执行 就是 异步==

---

#### 同步阻塞IO

用户线程发起 IO 读写操作后，线程阻塞，直到可以开始处理数据，CPU利用率 低

`int n = read(fd, buf, sz)`

每个 fd 都有一个 读缓冲区，一个写缓冲区

read 会发生一个 软中断，然后调用 系统调用 sys_read， 它会 将 内核中 读缓冲区的 数据 复制到 fd的缓冲区。


sys_read 这个系统调用，有2个阶段， ==数据准备阶段，数据拷贝阶段。== ，
在数据准备阶段，如果没有数据传过来，那么会阻塞。   
有数据后 进入 数据拷贝阶段，拷贝到 buf




---

#### 同步非阻塞IO

发起IO请求后 立即返回，如果没有就绪的数据，需要不断地 发起IO请求 直到数据就绪。 不断重复请求 浪费CPU


`int n = read(fd, buf, sz)`

如果 对方没有发给我们数据，read会立刻返回。需要 不断重复 调用。

阻塞，非阻塞，看的是 数据准备阶段。

数据拷贝阶段都是 阻塞的。


---

#### IO多路复用

单个进程/线程 就可以同时处理多个IO请求

用户将想要监视的 fd 添加到 select/poll/epoll 中，由内核监视，函数阻塞。  
一旦有fd 就绪 (读就绪 或 写就绪) 或 超时， 函数就返回。 


复用的是一个线程。

在 同步阻塞IO，同步非阻塞IO 中，每个fd 都需要一个专属线程 (。。 就是需要一个专门的线程 执行 read(fd) 这个调用 )

IO多路复用，我们用一个线程 来 管理 多条连接的IO


比如调用 epoll_wait，在 数据准备阶段是 阻塞的。在数据拷贝阶段也是阻塞的

`int n = epoll_wait(epfd, evs, max_events, timeout);`

epoll 维护了一个 监听fd 的红黑树，并在 内核态中 有一个 就绪队列

epfd 就是上面的 红黑树 + 就绪队列 组成的 epoll对象 的 fd

evs 是数组，用来 提取/保存 就绪队列中的数据

max_events 是最大事件数

timeout 超时， -1 永不超时 ， 1000是 1秒


epoll的作用就是 把 就绪队列中的 fd 复制到 evs中。


数据准备阶段， 数据拷贝阶段 都是 阻塞的

数据准备阶段的 阻塞取决于
1. epoll_wait 的timeout

数据拷贝阶段的阻塞


==IO多路复用不会操作 具体fd ( 即不进行数据拷贝)， 只是把 有事件的 fd 提取出来。  所以 IO多路复用是 在 数据准备阶段 发挥作用。==

---


#### 信号驱动模型

注册信号与回调函数 到核心， 有数据后， 核心 以信号的方式 告知， 用户线程 收到信号后  进行数据拷贝



---


### 异步IO

aio

调用 aio_read， 发起系统调用， aio_read 有一个buffer 参数， ==内核 会把数据 写入到这个 buffer中==， 写完后 通过 信号 告知我们。

用户线程 不需要 进行 数据拷贝。


---


## 怎样高效诊断 Linux 性能问题

BV13xp3eNE1d

先思考下面的问题 确定是否真的存在性能问题
- 是什么现象 导致我们认为有问题？
- 系统之前是否有更好的表现？
- 最近的哪些变化可能会影响性能？
- 我们可以量化 "慢" 的含义吗？
- 这个问题影响了许多用户 还是只有一个？

- 从什么时候开始有问题的？那个时间点 做了什么修改( 代码，服务器)

上面的问题 有助于 澄清问题，建立比较基准


一旦确认 发生了性能问题，就需要对其进行精确定义。  
不能用 缓慢，而是要具体的指标，比如 10秒、超时

需要确定 问题发生的 时间，地点。  
还需要了解其范围和影响，
- 是影响所有服务器，还是只影响一台
- 多少用户收到影响，业务后果是什么？


当看到平均负载值 升高时，就 应该使用其他工具进行 更深入的研究。
linxu cmd `uptime` 命令 的 `load average`， 给出了3个数字，如果是 上升的，说明 平均负载在升高。  
需要使用 top, vmstat, iostat, netstat 来 详细查看

。。load average 是 过去1min，5min，15min 的负载。  。。有点短啊。 可以每小时 记录下。 需要 自建一个app啊， 应该有工具可以把。 不不不，我记得 有 shell 命令可以 每x分 执行一次，然后放入 txt， 当时 jvm的 某个命令 就是这样执行的。


`top` ， 关注 CPU 或 内存 消耗 高的 进程， 这些可能就是 问题所在。

。。下面的 1 是 一秒刷新一次
`vmstat 1` ，关注 CPU队列，IO等待，交换，IO等待时间

`iostat 1`， 磁盘 的 每秒事务数， IO操作等待时间的百分比。  要确定哪个磁盘导致性能问题时，这很有用。

`netstat`
`netstat -a`， 列出所有活动连接， 有助于识别 系统上的 开放端口 和活动服务
`netstat -l`， 只显示 监听接口
`netstat -an i grep -c ':80'`， 计算特定接口的 连接数 ，看到异常高的连接数 就 可能是问题所在


最后，看 sar (system activity reporter，系统活动报告)
这是一个多功能工具，非常适合历史数据

`sar -u 1 3` cpu使用率 的历史数据

sar会自动收集数据，这样发现问题的时候可以 查看当时的情况


---

下面是更细化的工具


来自 https://www.brendangregg.com/linuxperf.html 

还有其他的一些东西

![0358ffe86eba216e98fec07ad7e3bffb.png](../_resources/0358ffe86eba216e98fec07ad7e3bffb.png)


---



## SSD 随机IO，顺序IO速度差异 及如何寻址

顺序IO， 3500mb/s - 7000mb/s
随机IO， 40mb/s

速度差异的原因
1. 数据存储方式
  - 顺序读写， 物理位置连续， SSD可以一次性读取或写入 大块数据，减少 寻址时间，提高读写效率
  - 随机读写， 数据分散在不同的物理位置，每次读写都需要 单独寻址
2. 闪存颗粒特性
  - 顺序读写， 充分利用 闪存颗粒的 并行读写能力，同时访问多个 存储单元，提高整体速度
  - 随机读写， 频繁切换 读写操作的 存储单元，导致并行读写能力 无法充分发挥。


内部架构和工作原理
- 顺序读写， SSD主控 和 闪存颗粒 可以协同工作，快速完成数据传输
- 随机读写， 主控芯片需要更多的时间 进行数据管理 和调度，增加了 读写延迟

垃圾回收和磨损均衡机制
- 随机写入， 可能触发 垃圾回收和磨损均衡机制，导致写入放大现象，进一步降低随机写入速度


---

SSD通过以下方式知道数据存储在哪里

1. 逻辑块地址 LBA
  - OS将数据以 LBA 的形式发送给 SSD，每个LBA对应一个固定的数据块
  - SSD的控制器 将 LBA转换为 物理块地址 PBA，从而 确定数据在闪存中的具体位置
2. 闪存转化层 FTL
  - FTL 是 ssd的关键固件，负责 LBA和PBA 之间的映射关系
  - 维护了一张映射表，记录每个 LBA对应的 PBA。
3. 磨损均衡和垃圾回收
  - 磨损均衡机制 通过 分散写入操作，延迟闪存寿命
  - 垃圾回收机制清理 无效数据，优化闪存空间利用率
  - 这些机制会 修改数据的 物理存储位置，但FTL会及时更新映射表，确保数据可访问性


## 逻辑块地址 LBA

是 OS 用于描述计算机存储设备中 数据所在区块的 通用机制，通常用于 硬盘，SSD。

LBA的作用
- 简化寻址方式， LBA使用简单的线性寻址模式，从0开始，依次递增。 取代了 原先OS必须面对的 存储设备的硬件构造，如CHS(磁柱-磁头-扇区)寻址模式， 使得 寻址更加直观和高效
- 抽象硬件细节， LBA隐藏了 存储设备的物理细节。 为OS 和应用程序提供了 统一的，抽象的 存储设备视图。这样 无论底层如何变化，OS和应用程序 都可以通过相同的接口来访问数据
- 支持虚拟化和集合操作。





。# computer/OS end









# high performance

。。非底层的放 # performance

## NUMA架构对计算性能有多大提升

BV1CAWQeKEyv


目前主流的 内存访问架构有 2种
- SMP，symmetric multi-processing，对称多处理架构
- NUMA，non-uniform memory access，非一致性内存访问架构

。但是baidu了下，说是 UMA 和 NUMA，这2个都是 SMP 的具体实现。
。应该网上的对。  
。UMA就是 只有一个内存，所有CPU核心 通过  总线 与 内存交互
。NUMA就是 多个内存， 每个CPU核心 直接和 某个内存交互， 要和其他内存交互时，需要 先访问 那个内存的 CPU，通过那个CPU来和 目标内存交互。

。视频后续说了， SMP架构中，所有CPU 通过一条总线 和 内存交互，这个是 UMA。  
。。SMP 是指 CPU 共享内存及总线， UMA更具体，是通过总线 共享 所有内存， NUMA 是 直接访问内存+通过其他CPU访问内存 来共享内存

。。。
。。。
。。又查到， NUMA是 非对称多处理。 我也不知道 到底哪个是哪个了。
。。
。。bing了一下，决大概率： UMA 和 NUMA 都是 SMP。
https://www.geeksforgeeks.org/difference-between-uniform-memory-access-uma-and-non-uniform-memory-access-numa/
`Both UMA and NUMA architectures are used in symmetric multiprocessing (SMP) systems where multiple processors share a common memory pool.`


随着 CPU性能的提升，内存的性能逐渐 跟不上处理器的性能  

SMP架构 对 多CPU处理器 非常不友好  。。 不过也没说出个道理。就说了 CPU核心是平等的(不存在优先级)，就可能造成 处理器 饶了一大圈 去访问内存。   。。 无法理解这个场景  。。 应该是 多核心 竞争 总线， 导致 输了的核心 需要等待一段时间 才能访问到 内存。

为了解决 SMP的问题， 就有了 NUMA架构

NUMA架构下，CPU会优先访问 临近的内存

在intel 8路服务器中，每颗 至强铂金处理器 通过3跳 UPI链 与其他处理器 进行连接

。。全链路UPI(Ultra Path Interconnect)

在NUMA架构下，每个处理器被划分为一个 NUMA，那么 8路服务器 就有 8个NUMA节点。

每个处理器会优先访问自己的内存，当一个 NUMA节点内存不够时，才会出现 跨CPU访问内存的情况

CPU和内存的延迟 得到大幅降低

NUMA中，不是 一颗核心 一定是 一个 NUMA，   霄龙(epyc)是2个核心是一个 numa， Intel E5，E7 一个核心 是 2个numa

---

### NUMA 可以带来多少速度增益？

实测，测试采用的 2款程序
- VASP， 内存密集型程序， 计算速度 和 内存带宽 基本上呈 一次方关系
- CP2K， CPU密集型程序，并行性很好，可以 实现 几千核 的并行


---

测试 NUMA架构对 CPU访问内存延迟的优化

跨CPU访问内存 vs 访问本地内存

对于 内存密集型的 VASP， 跨CPU访问内存 会 带来 60% 的损失。 。。 不过看图，vasp 本地内存是 21.8秒，跨CPU内存是54.2， 所以应该是 1.5倍啊。

对于 cpu密集型的 CP2K， 跨cpu只造成了 20%的损失。  cp2k 异地内存是 20.5， 本地内存是 16.1

---

NUMA 会优先把 程序分配给 相同的 NUMA下的 核心。

测试 单路满载 vs 双路满载一半

vsap，
AMD 128核 27秒，64核跨CPU 31秒，64核 21秒 
Intel  52核 24秒，26核跨CPU 38秒， 26核 50秒  差距很大

CP2K
AMD  128核 10秒，64核跨CPU 15秒，64核 16秒
Intel  52和 36秒， 26核跨CPU 60秒，26核 64秒


---



## 为什么 AMD CPU 需要绑定核心

BV1obvEebEnk

使用 64核 跑 cp2k 一个项目电子步 耗时 13.7s
使用 128核 跑 2个项目电子步 耗时 ， 应该也是 13.7s， 但是实际上是 81.7秒

我们要从 AMD 的 Infinity架构入手

霄龙 一共有 8个ccd ( Core Chiplet Die )



双路霄龙，有 4个连接延迟
- 0-7号核心，位于同一个CCD上，它们之间互相访问的延迟是最低的
- 0-15号核心，位于相邻的 2个 CCD上
- 0-63号核心，位于同一个CPU上
- 0-127，双路全部的128个核心


绑定 或不绑定 核心， 对于单任务 影响不大。

多任务， 是否绑定核心，对于算力的影响非常非常大， 不绑定核心时，算力只有 1/4

绑定的方法是： 2个任务，各自绑一个CPU，8个任务，各自绑到 相邻的 CCD上， 16个任务，每个任务一个CCD


AMD的CPU，只有2个任务时，就直接下降到 1/4 。
Intel， 2个 任务，下降不多， 但是 任务多了以后，也下降非常快。 6个任务的时候 也下降到了1/4


测试使用的是 非常大的模型，一个 晶胞达到了800多个原子   。。不知道这意味着什么。   反正 肯定是 绑定了更好。



。# high performance end



# redis

## redis 5种常见场景

BV1yPpwetEuq

redis 的类型： string,hash,bitmap,list,zset,set

string: session, cache, 分布式锁
int: 计数，rate limiter, 全局id
hash：购物车
bitmap：user retention
list：message queue
zset: rank/leaderboard

---

第一个用例 缓存对象以加速web应用程序  
redis将频繁请求的数据 存储在内存中
缓存分布在 redis 的服务器集群中，分片 是一种在集群中 均匀分配 缓存负载的常用技术。
将redis部署为 分布式缓存时，还要注意：
- ttl
- 冷启动时的 惊群效应

---

使用redis作为 会话存储， 以便在 无状态的服务器间共享 session

在生产中，通常使用 主备模式，这样 主redis崩溃， 可以快速切换到 备份 来接管流量。

---

分布式锁

`setnx`

有很多成熟的库 提供高质量分布式锁实现

---

速率限制器

基本的用法：对每个传入的请求， 将 请求ID或 用户ID 作为key 使用 redis中的 incr 来增加 key 的请求数量， 将当前计数 和 允许的速率限制 进行比较， 如果计数在 速率限制内，则会处理请求。如果超过，则拒绝。
这些key 在 一段时间 (1min) 后过期，来 重置 请求数

更复杂的速率限制器，如 漏桶算法，也可以通过 redis 实现


---

游戏排行榜




## redis CPU使用率太高 解决方案

BV1jnNBerE9T

解决
- 需要redis 监控平台
- 禁用 高风险，高消耗的命令，如 keys，hgetall。
- 避免redis 的短链接，要使用 连接池，如 Jedis，Lettuce
- 排查并优化key，把 热key拆分，分散到其他节点。
- 调整写磁盘的频率，但增加丢失日志的概率
- 硬件，固态。




。# redis end















# cache

## 缓存的常见陷阱和对应策略

BV1iqpGe1Edj

缓存就像一个内存层，存储频繁访问数据的副本

---

cache stampede， 缓存踩踏， 。。缓存击穿
缓存过期，大量请求访问，会压垮数据库

策略

1.  locking， 缓存未命中时，每个请求都会在重新计算过期页面之前 尝试获取 该key 的 锁。没有获得锁时，可以，
    1.  等待直到值被另一个线程计算出来
    2.  请求立即返回: 未找到， 客户端 通过 重试 来处理这种情况
    3.  server维护 缓存项的 陈旧版本，以便在 重新计算新值时 临时使用
        locking 需要对 锁进行额外的 写入操作，并且 正确实现锁 可能具有挑战性
2.  将重新计算 卸载到外部进程，此方法可以通过 多种方式激活
    1.  在缓存即将过期时 主动激活(proactive)
    2.  发生 缓存未命中时 被动激活(reactive)
        这种方法 为 架构增加了另一个动态部分，需要仔细 持续维护和监控
3.  概率提前过期，每个请求在 实际过期之前 有很小的几率 触发 cache的更新。随着时间临近，概率增加

---

cache penetration 缓存穿透

请求数据库中不存在的数据。 会导致 不必要的负载。

策略

1.  为 不存在的key 实现带ttl的占位符，后续的请求 不会继续 访问db。 需要仔细调整，防止消耗大量cache资源(。。恶意程序遍历1-10亿，导致cache中缓存10亿个key)。
2.  bloom filter 。。 话说，它有可能误报，所以还是会 走db的，所以 根据 处理时间，能不能推算出 哪个key 可以 绕过 布隆过滤器，然后 狠狠地 请求。 记得可以设置 误报率，误报率越小，那么 bit数组越大。

---

cache crash 缓存崩溃

。。不是 缓存雪崩。

如果 整个缓存系统 发生故障， 导致 每个请求都直接进入 db，会导致 db崩溃。

而且 当系统无法响应的时候，用户 会 无限刷新，导致 请求量更大。

cache crash的 近亲是 cache acalanche(缓存雪崩)

缓存雪崩有2种原因

1.  大量数据一次性过期
2.  缓存重启后，缓存里是空的。 。。 就是缓存击穿啊。似乎没什么区别

。。似乎 单个数据的原因导致 大量请求到db 就是 缓存击穿， 多个数据的原因导致 大量请求到db 就是 缓存雪崩
。。是的，视频最后说了 雪崩和击穿的区别，就是 击穿是单个数据， 雪崩是很多数据

缓存雪崩导致 大量请求到 db

策略

1.  实现断路器，当系统明显过载时，它会暂时阻止传入的请求
2.  部署冗余的高可用缓存集群
3.  缓存预热 (特别是 重启后)

## 缓存的使用场景

BV1pNpweZEhx

硬件的
CPU L1,L2,L3
TLB，transtation lookaside buffer， MMU中用来缓存 虚拟地址和 物理地址 映射。

OS
page cache， 将最近使用的 磁盘块 保存到内存中
inode cache， 文件 与 物理地址的 映射

架构中

浏览器 可以 通过 http响应中的过期时间 来 缓存http响应。

CDN 加速内容交付

一些LB 可以缓存响应，如果有相同的请求，就直接返回响应，不转发给服务器

mq中，kafka等 可以在 磁盘上 缓存大量消息

redis

elastic search

数据库

- wal write-ahead log，预写日志, 数据先陷入wal，然后在 B树中建立索引
- buffer pool， 缓存查询结果的 内存区域
- materialized view，物化视图
- transaction log
- replication log

---



## 数据库 缓存 一致性

BV12m421g7fh

。。感觉不太对劲。要用的话 得再baidu 确认

mysql-redis 同步 的5种方法

1. 同步双写
   app在更新数据时，同时更新mysql 和redis，一般是 更新mysql，更新成功后 删除 redis中缓存
2. 基于MQ异步多写
   app更新mysql后，将修改 发送给mq，其他app监听mq消息，对redis进行删除
3. 定时任务
   定期从mysql读取 上次定时任务到现在 修改的数据，并 删除对应的 redis缓存， 适用于 更新频率低的系统，比如 统计数据
4. 闪电缓存
   redis的 ttl 设置为10秒， 应用 只写入 mysql，不管redis。 用于高并发读写
5. 监听binlog
   flink-CDC监听mysql的binlog，它会删除 redis中数据







。# cache end











# LB

## 6种常见的LB算法

BV16PpuerEbR

2大类，静态算法，动态算法

静态算法, 根据算法转发，不考虑服务器的实时状况， 优点是 简单， 缺点是 适应性和精度 较差

- round robin
    某些服务器可能超载
- hash
    通常使用 ip 或 url 来确定 将请求 路由到 何处
    选择最佳 hash 是 困难的
- sticky round robin
    将 同一用户的 后续请求 转发到 同一服务器， 新用户随机分配
    很容易出现 负载不均匀的情况
- weighted round robin
    权重必须手动设置，对 实时变化的 适应性较差

动态算法，分发时主动考虑性能，服务器条件 来适应 实时状态

- least connection
    将新请求转发到 目前 活动连接或打开请求数 最少的服务器
- least time
    将请求转发到 当前 最低延迟 或 最快响应时间的 服务器

round robin 非常适合 无状态应用程序

动态算法 有助于 优化大型复杂应用程序的 响应时间和可用性

。# LB end

# data structure

## 开发人员每天常用的10种数据结构

BV1M6pPekEce

list
array
stack
queue
heap
tree
suffix tree
graph
R-tree
hash table

注意缓存友好性

---

## 数据库背后的 8种关键数据结构

mysql, redis, caaandra

skiplist
hash index 
SStable
LSM tree
B tree
Inverted Index
Suffix tree
R tree

---

skiplist 是一种概率数据结构，用于实现 有序映射 或集合
是 平衡树的 一种替代方案

可以实现 高效的 搜索，插入，删除 

redis中， skiplist 被用于实现 有序数据结构，如 zset

它可以 快速查找，==范围查询==，

---

hash index 也称为 hash table

通过使用 hash函数 为key 生成 哈希值，可以有效地将 key 映射到 value

哈希值被用于快速定位表中的值，从而实现快速 查找，插入，删除

---

SSTable，LSM tree ，相辅相成

sorted string table， 用于 按 排序顺序 在磁盘上存储数据，是一种基于文件的 数据结构， 用于 以高度压缩 和 高效的格式 存储大量数据

SSTable 是 LSM tree 的 核心组件，另一个核心组件是 MemTable  
memtable 是一种内存数据结构，被用于存储最近的 写入操作

SStable 和 memtable 协同工作 来处理 大量写入操作

LSM tree 是 NoSQL 的支柱


---

B tree
B+ tree

用于在磁盘上 高效存储和 检索大量数据

B tree 是一种平衡树，其中每个节点可以有 多个子节点，并保持数据有序

B+ tree 是 B tree 的变种， 它的所有数据都存储在 叶子节点。 非叶子节点 仅保存 key 信息。

。。mysql 是 B+ tree  。数据库都是 B+ tree


---

inverted index

倒排索引

用于从大量文本文档中 高效查询与检索数据

它创建 单词到 包含单词的文档的映射。

用于文档索引引擎，如 ElasticSearch

---

后缀树

suffix tree

在数据库中 被用于 高效的文本搜索

可以在 大量文档集合中 快速找到 搜索词 的所有 出现位置

。。就是一个 句子， 从后往前， 对每个 单词到句末 这个 后缀string 执行 trie。 这样，你搜索一个 短语 的时候，就可以 利用这个 trie 了。

---

R tree

空间索引数据结构

根据几何边界 (矩形或多边形) 组织数据

用于在 数据库中 有效地存储 和检索 空间数据


---




。# data structure end














# performance

。。# 性能 # 接口 # 优化

## 7种 10倍提升api性能的方法

BV1XvpKeXExh

不要过早优化， 过早进行优化 会导致 不必要的 复杂性

第一步始终必须是 通过压力测试和分析请求 来识别 实际瓶颈

只有在 确认api端点 存在 性能问题后 才开始优化

- caching
    最有效的方法之一，存储 昂贵计算的 结果，稍候重用，无需再计算
    大部分缓存库 只需要几行代码就可以 轻松添加
    即使短暂的缓存，也可以 显著提高速度
- connection pool
    请求通过连接池中的连接发送，而不是 每个请求 新建一个连接。 创建新连接 涉及 大量 TCP握手，协议，设置，这些会降低 api 速度
    基于连接的重用 可以 极大 提高 吞吐量
    如果你使用 ==无服务架构 serverless architecture==，连接管理可能更具挑战性。因为每个 无服务器函数实例 都会打开自己的 数据库连接，由于 无服务器 可以快速扩展，所以可能导致 大量打开的连接。 aws rds proxy, azure sql database 是 无服务器的 解决方案。
- avoid N+1 problem
    和数据库性能密切相关，
    使用join 来完成 N+1 个查询
- pagination
    分页
    api返回大量数据，会减慢速度， 使用 分页，偏移参数 将 响应分解为 更小，更易于管理的 页面。
- JSON serializers
    json序列化的 速度影响了 响应的速度。 使用更快的 json序列化器，最大程度减少 数据 与json 之间 转换的 时间
- payload compression
    减少网络传输量。
    Brotli算法，提供更好的压缩比
    许多 CDN 可以为你处理 压缩。(就是 你把原文件 上传到 CDN，CDN自己帮你压缩)
- async logging
    许多应用中，写入日志 所需的事件 可以忽略不计
    但是在 毫秒级的 高吞吐量系统中，写入日志 所需的事件 可能会增加。
    线程快速将 日志放入 内存缓冲区中， 独立的 日志记录线程 将 日志条目写入文件 或发送到 日志记录服务
    系统崩溃时，异步日志可能丢失 日志


## 接口性能优化

BV1XGfaYcE5G

后端
- 缓存， 注意 缓存穿透，击穿，雪崩， [Cache](../Arch/Cache.md), [HP](../Arch/HP.md)
- 并发调用， 原先 先调A，再调用B， 改成 同时调用A和B
  - 线程池
- 同步接口异步化， 将不影响整体流程的 接口 异步化
- 避免大事务
- 优化日志记录， 打印大量日志会占用CPU，磁盘IO


数据库
- 数据库查询优化
  - 索引
  - select 必要的字段
  - 避免深分页， 需要上一页的最后一条数据的ID
- 表设计冗余数据， 减少join。  分库分表直接 冗余一张表，防止 跨库查询
- 连接池
- 数据压缩，减少网络传输
- 加机器( 或提升硬件)
- 换数据库




## 系统设计需要知道的延迟等级

BV1jwHhebE8o

2020年的评估

纳秒 十亿分之一秒
微秒 百万分之一秒
毫秒 千分之一秒

<1纳秒： CPU寄存器，CPU时钟
1-10ns: L1 L2， 一些昂贵的CPU操作， 分支预测错误 会消耗 20个CPU时钟。
10-100ns：L3，Apple M1的内存
100-1000ns： 系统调用，linux的 system call 需要几百纳秒，这只是 陷入内核并返回的 直接成本，不包括执行； 对64位数字进行 MD5 大约要200ns
1-10us：linux线程上下文切换最好情况 几微妙。 将64k数据在内存中复制到另一个位置 也需要几微秒。
10-100us： nginx需要50微秒来处理典型的 http请求。 从内存中顺序读取 1mb数据 大约需要 50微秒， ssd读取一个 8k页需要 100微秒
100-1000us： ssd 写入延迟 比 读取延迟 慢10倍，写入一个 page需要 1ms。 现代云提供商的 区域内网络 往返 需要 几百微秒。 memcache，redis 的读取操作 需要 1ms
1-10ms： 现代云的 域间网络往返 都在这个范围内， 机械磁盘的寻道时间是 5ms
10-100ms： 美国西 与美国东， 美国东与欧洲 之间的 网络往返都在这个范围内； 从内存中 顺序读取 1GB数据
100-1000ms： bcrypt等慢速加密算法 需要300ms， tls握手在250-500ms， ssd读取1g数据
1s：同一云区域内通过网络传输1g数据需要 10s

。# performance end

# debug

## 像专业人士那样调试程序

BV1NApweFEju

在调试具有挑战性问题时 拥有正确心态。

1.  计算机是具有逻辑的，对于当前问题 总有一个合乎代码逻辑的解释
    
2.  被卡住是暂时的，只要坚持和努力，终会解决问题
    
3.  了解自己的局限性，认识到 何时 需要向 更专业的人士 寻求帮助
    
4.  并不总是需要花大量精力来解决每个问题，我们应该根据错误的潜在影响和严重性进行优先级排序，并不是每个错误都值得努力
    
5.  收到客户的错误报告时，收集尽可能多的相关信息非常重要，如果可能，请客户提供截图或录屏
    
6.  收集重现问题的详细步骤，收集所有相关日志，包括与错误相关的任何错误消息，我们的目标是获得尽可能多的信息来帮助重现问题
    
7.  如果可以隔离出现问题的环境和上下文，我们可以在临时服务器上重新创建它，看看是否可以重现问题
    
8.  拥有可重复的环境就成功了一半
    

有了可复现问题的环境后，调查错误的一些策略

1.  大量增加 log，日志可以帮助构建所发生事件的时间表，并确定我们的预期 是否和 实际打印的 相符
2.  对于某些语言，设置 debugger 可能是可行的选择，比如 Erlang/OPT 具有出色的自省功能

能重现，那么找出问题的根源 应该不会太难，虽然这需要时间和耐心

如果无法重现， 则需要 运气和 很大的耐心
一些无法重现的原因

1.  某些错误仅在生产负载下出现 。。。！！！！！！！！CMS DMTM
2.  有些错误仅在生产负载的 某些竞争条件下出现
3.  客户设备上可能存在触发错误的特定上下文和环境

我们能做什么呢？

1.  如果有一个特定的错误，那么就冲发生错误的那行开始，回溯代码向上的调用栈
2.  梳理所有日志寻找线索，通过在 整个请求生命周期中 跟踪一个失败的请求来构建时间表
3.  如果幸运的话，我们也许能够对正在发生的事情提出一个见解，然后，可以向代码添加log 以 证明 这个见解，并部署到生产

当我们完全陷入困境时 需要考虑的一些一般策略

1.  休息一下，有时，回避问题，并以全新的心态回来，可以提供新的见解
2.  有时，大声讨论问题，可能会带来 灵光乍现的时刻 。。头脑风暴
3.  写出问题，或向 想象中的导师 发送电子邮件 可能帮助产生新的想法和观点 。。文字化，具现，大脑中是跳跃的，可能遗漏一些东西，具现后，一行行看，可能发现一些东西
4.  获取帮助，不要犹豫要不要和其他人合作一起解决这个问题，全新的眼光和不同的视角 通常有助于揭示新的解决方案

。# debug end

# Arch

## 6种API架构

BV1sTpPeNEWM

- SOAP， 基于xml的企业级应用，多用于 要求高可靠，高可用的 金融服务，支付网关， 对于简单应用，过于冗余
- RESTful， 基于资源的web应用，日常应用(X,ytb)多是RESTful， 不适合 实时数据，高关联性数据
- GraphQL， 查询语句，降低网络负载，Facebook用。 由于其灵活性，效率，使其成为 复杂数据的 应用的有力选择， 有陡峭的学习曲线，对于简单应用，有点大材小用，因为它的灵活性 需要 服务器端的支持。 前端会主动推动，因为便于前端开发
- gRPC， 高性能的微服务， 更现代化，高性能，处理微服务的 巨大的 服务间通信。 由于浏览器的限制，gRPC会带来挑战。
- WebSocket， 低延迟的 双向数据交换， 实时，双向，持久连接。 如果应用不需要实时数据，那么就不要使用它，因为websocket有额外的开销
- Webhock， 异步的 事件驱动应用， 事件驱动，http回调，异步操作。

## 5种架构模式

BV1RapMeYEoQ

- 分层架构
    将系统组件分为不同的层，通常是 表示层，业务逻辑层，数据访问层
    例如，MVP(model，view，presenter)模式，模型负责数据的访问和存储，视图负责显示，逻辑处理器是model view的桥梁，确保关注点的清晰分离
    分层架构的主要目标是 促进分离，以便 其中一层的修改 不会影响其他层
- 事件驱动
    促进了 松散耦合的软件组织和服务之间 事件的产生和消费
    事件发生后，message broker 会广播事件，其他组件 订阅它们感兴趣的事件
    允许高度解耦机构
    一个场景是 命令查询职责分离(CQRS,command query responsibility segregation), 使用 CQRS，令 数据的写入操作 和 读取操作 分离，更改 通过事件来触发
    通常使用 发布/订阅 模型， 组件不会 直接相互调用，它们只是对 已发布的事件做出反应
- 微内核架构
    将核心系统功能做成小型微内核，将扩展功能做成附加组件或插件
    可扩展性，易维护性，故障隔离性
- 微服务
    将应用程序分解为小型，松散耦合的服务集合，每个服务实现特定的业务功能，有它自己的数据模型，通过 api进行通信
    这种架构促进了功能的模块化，因此可以独立开发，部署，按需扩展。 提高了敏捷性，能快速创新
    复杂度增加：管理服务间通信，维护数据一致性
- 单体架构
    简化开发和部署
    初创公司，小型应用 的首选
    模块化单体，试图 解决 单体 和 微服务 的缺点。



## data pipeline 数据管道

在如今 数据驱动的世界中，公司从各种来源收集大量数据，这些数据对于做出明智的业务决策 和 推动创新 至关重要

但原始数据通常是 混乱的，非结构化的，并且以 不同格式 存储在 多个系统中

数据管道 自动化了 收集，转换，交付数据的过程

数据管道有 多种不同形式

从广义上讲，数据管道具有如下的阶段： 收集，摄取，存储，计算/处理，使用， collect, ingest, store, compute, 


- 收集
  我们从多个来源： 数据存储(数据库，存储了用户订单，支付信息)，数据流(实时数据，kafka，用户点击，搜索)，应用程序 获取数据
- 摄取
  将数据加载到 数据管道环境中，根据数据类型，它可以直接加载到 处理管道 或 中间事件队列中， kafka, kinesis 用于 实时数据流， 批处理或 变更数据捕获工具 用于 数据库数据
- 处理
  摄取后，数据 被 存储 还是 使用 取决于 具体的用例
  - 批处理 是 按计划的 时间间隔 处理大量数据， spark 及其 分布式计算能力 是这里的关键，其他流行的批处理工具包括 hadoop，hive。
    例如，spark配置后，可以 轻松运行 以汇总每日销售数据
  - 流处理实时数据，如，flink，google cloud dataflow, apache storm, apache samza。 它们在 数据到达时 对其进行处理。 
    例如，flink可以用于通过 分析交易流 并应用复杂的事件处理规则 来 实时监测 欺诈交易。
    流处理通常 直接处理 来自数据源的数据，而不是 使用 数据湖
  - ELT或 ETL 流程 对 计算也至关重要
    apache airflow， aws glue 等 ETL 工具可以 协调数据加载，确保在 将数据加载到 存储层之前 应用 数据清理，规范化，丰富化 等转换
    在这个阶段，杂乱，非结构化，格式不一致的 数据 被转换为 适合分析的 干净的 结构化的 格式
- 存储
  处理完后，进入存储阶段，有几个选项： 数据湖，数据仓库，数据湖屋(结合 数据湖，数据仓库的 优点)
  数据湖使用 amazon s3, hdfs 等工具 存储原始数据和 处理后的数据， 数据通常是 Parquet(列式存储) 或 Avro(序列化) 等格式存储，这些格式 对于 大规模存储和查询 非常有效
  数据仓库 存储 结构化数据，使用 snowflake,amazon redshift, google bigquery
- 使用
  数据科学团队使用 数据进行预测建模， jupyter notebook，tensorflow，pytorch，pandas，scipy，numpy，matplotlib，ipython
  数据科学家 根据存储在 数据仓库中的 历史交互数据，构建模型来预测 客户转向
  tableau，power bi 等 商业智能工具 提供交互式 仪表板 和报告， 这些工具可以直接连到 数据仓库 或 数据湖屋，可视化kpi 和趋势
  looker 等 自助分析工具 是团队 无序深厚的技术知识 即可运行查询。 LookML 和look的建模语言 抽象了 sql的复杂性，允许 营销团队 分析 活动绩效
  机器学习 使用这些数据 进行持续 学习和改进，例如，银行欺诈检测系统 不断使用 新的交易数据进行训练，以适应不断变化的欺诈模式



## 7种常用的分布式系统模式

BV1bzpPeGEbq


### proxy模式

想象自己是一名忙碌的 首席执行官，有一位私人助手，负责处理你的所有的约会和沟通。这正是 proxy模式 为你的app 所做的事情， 它充当 你的app 和 通信的服务 的 中间人。

proxy会进行 日志记录，监控 或处理重试 等任务

例如，k8s 使用 Envoy 作为 proxy，来简化 服务之间的通信。

proxy模式 可以帮助减少延迟，增强安全性。

### 断路器模式 Circuit Breaker

防止分布式系统中 发生级联故障

当服务不可用时， 断路器会停止请求。

Netflix 的 Hystrix 就是这种模式


### CQRS，Command Query Responsibility Segregation

命令查询职责分离

通过 分离 写入(命令) 和读取(查询) 操作， 我们可以独立的 扩展 和 优化每个操作

电子商务平台 可能对 产品列表 有较高的 读取请求， 但对 下订单的写入请求较少。  

CQRS 允许 高效处理每项操作。

在读取和写入操作 具有 不同性能特征，不同延迟 或资源要求的 系统中，这种模式 变得很有价值




### Event Sourcing 事件溯源模式

不直接更新记录，而是保存 更改。
这种方法提供了系统的 完整历史记录，并可以 更好地 进行审核 和调试

git 版本控制

通过 事件溯源， 还可以实现 高级功能， 例如，基于时间线 调试 或重播事件 以进行分析


### leader election 领导者选举模式

分布式系统中，领导者选举模式 确保只有一个节点 负责特定任务 或资源

当领导节点发生故障时，其他结点会选举新的领导节点

zookeeper，etcd 使用 这种模式。  。。一个是 zab，一个是 raft

通过指定领导者，我们可以避免冲突并确保 整个分布式系统决策的一致性。


### pub sub 发布者订阅者模式

发布者在不知道 谁会接收事件的情况下 发出事件，

订阅者 监听 他们感兴趣的事件

这种模式 可以实现 更好的 可扩展性 和 模块化

发布订阅模式 非常适合 我们需要跨多个组件传播更改或更新的场景，比如，跨各种服务 更新用户的个人资料


### 分片模式

在系统的 多个节点间 分发数据的技术， 提高了 性能和可扩展性  

每个分片都 包含 数据的子集，从而减少 任何单个节点上的负载。


mongodb，cassandra 使用 分片来处理大量数据

分片帮助我们 实现更好的 数据局部性， 减少网络延迟， 加快查询执行速度


### 绞杀者模式

用新的系统 逐渐 代替 旧系统的方法。

新的app 逐渐替代 老的app，最终 老的系统被全部替代。


。# Arch end

# algo

## 时间轮

BV1jw48eMEpp

。。视频太冗余了。

任务最开始是 挂载在 小时上，当 走到某个小时 的时候，把 该小时挂载的任务 分配到 分钟 级别， 当运行到 某个分钟时， 把 该分钟上 挂载的任务 分配到 秒级别。

kafka更细，3层时间轮，分别是 1ms，20ms(20个1ms的格子)，400ms(20个20ms的格子) 的格子。

多线程需要考虑 锁，所以 用时间轮， 粒度小，里面内容少， 锁的范围小， 锁的个数多(一个格子就可以一个锁)， 业务逻辑复杂度小(时间片中只有很少的内容)， 高并发性能好。 而不是 list，锁整个list(实际锁 list头 或尾)。

。但是还是不知道 怎么移动。 百度了，ai： 时间轮通过一个线程推动指针转动，每当指针转到某个槽时，就会执行该槽上的所有任务 。
。。 延迟队列，如果 expireTime没有到达，那么 阻塞。



## 跳表的高度

主要由以下因素决定
- 节点层数， 节点拥有的层数越多，跳表高度越高
- 随机化算法， 插入新节点时，根据随机数的结果 来确定新节点应占据的 层数。 通常 新节点的层数 随机决定，每层概率减半。
- 元素数量， 跳表的高度通常是 O(logn)，这意味着 随着元素数量的增加，跳表的高度
- 最大层数限制， 实际实现中，会设置 跳表的 最大层数限制， 防止跳表过高。


随机化算法 是关键，使得 跳表保持 高效查找的同时，具有良好的动态适应性。




。# algo end


# mq

## kafka的5大使用场景

BV1nExPeqEt5

探讨kafka 如何解决现代软件架构中的关键挑战

kafka最初是 LinkedIn  处理日志的工具， 逐渐发展为 一个 多功能的分布式事件流平台

它的设计 利用了 具有可配置保留策略的 不可变 附加 日志。 

1. 日志分析
   现在的日志分析 不仅仅是 处理日志，而是要 实时 集中 和分析 复杂分布式系统的日志
   使现代日志分析功能强大的是 kafka 与 elasticsearch，logstash，Kibana 等工具的集成。
   logstash 从kafka 提取日志。发送给 elasticsearch
2. 实时机器学习管道
   现代 机器学习系统需要快速，持续地 处理大量数据。 kafka的流处理能力 完美地 满足这一需求
   kafka是ML管道的 中枢神经系统，它从各种来源(用户交互，物联网设备，金融交易) 摄取数据， 这些数据 通过kafka 实时流向 ML模型
   kafka 与 flink 或 spark streaming 等流处理框架的集成是 关键所在，这些工具可以从 kafka读取数据，运行复杂的计算 或 ML推断，并将结果 实时写回 kafka
   kafka stream 是kafka的原生 流处理库，它允许我们直接在 kafka上 构建可扩展，容错的 流处理应用程序
3. 实时系统监控和警报
   即时，主动的 系统健康跟踪和警报
   kafka是整个基础设施的 指标和事件的中心枢纽。它从各种来源 摄取数据，包括 应用程序的 性能指标，服务器健康状态统计，网络流量数据 等
   kafka与众不同之处在于 对这些指标的 实时处理， 当数据 流经 kafka时，流处理应用程序 会对其 进行持续分析，它可以实时计算总量，检测异常，触发警报
   kafka 的pub sub 在这里大放异彩， 多个专门的消费者 可以处理 同一个指标流，而不会相互干扰。比如，一个更新仪表盘，一个管理警报，一个为机器学习提供信息
   kafka的持久性模型 允许进行时间旅行调试。 我们可以重放指标流，以了解事件发生前的系统状态，加快根本原因的分析
4. 变更数据捕获 change data capture
   cdc是一种用于 跟踪和捕获 源数据库变化的方法。 它允许将这些变化 实时复制到 其他系统 。  。。。主从同步 可以吗？
   在这个架构中，kafka充当中心枢纽，将源数据库中的 更改 流式 传递到 各个下游系统
   为了在kafka 和其他系统之间 移动数据，我们使用 kafka connect，这个框架 允许我们 构建 和运行 各种连接器，例如，我们可以使用 elasticsearch 连接器 将数据流 传递给 elasticsearch。 。。。 那上面第一点 还走 logstash 干嘛。
5. 系统迁移
   迁移过程中，kafka的作用不仅仅是 传输数据，它在 新旧系统间 起到 缓冲作用，还可以在 它们之间 进行翻译。这样可以实现 渐进，低风险的 迁移
   kafka可以从其保留期内的任何时间点 重播消息，这是数据核对的关键。 有助于 迁移中 保持一致性。
   大规模迁移中，kafka可以充当 安全网，我们可以 并行 新旧系统。 两者 都从 kafka消费，也可以向 kafka生产。 如果出现问题，可以轻松回滚。 还可以比较 新旧系统的 输出



## 什么时候用kafka，什么时候用mq

BV1aJ4pekEH1


特点：

kafka
- 极高吞吐量
- replay(重放)
- data retention(保留、维持)
- fan out (扇形散开)

mq
- 复杂的 路由
- 消息是 某个消费者需要的
- 数据量始终， moderate data volumes

---


kafka 的 消息 会给 每个消费者一份  
mq 的 消息 只给 一个消费者

kafka ，生产者直接发送到 queue  
mq 生产者将消息 发给 exchange， exchange 根据路由 发给 queue


---

场景：

kafka
- 流数据处理
- 数据总线
- 日志系统
- 实时通信

mq
- job worker 系统
- 消息队列
- 解构微服务


---

评论：
数据以流的方式大量进入系统，且一些场景 需要进行 流事件方式处理， 那么kafka   
要求 可靠性，事务性， 那么 rabbitmq

数据采集 用kafka  
生成管理系统， 用mq

kafka 不保证 消息准备传输，也不支持事务



## 如何保证只被消费一次

有3个地方可能丢失消息
1. 生产者 -> MQ
2. MQ本身宕机
3. MQ -> 消费者
4. 消费者 宕机

生产者->MQ，需要生产者收到 MQ的确定才 确定 是 发送成功了， 有可能是 MQ的 ACK响应丢失，那么 生产者也会 重发，那么会导致 2条 不同的 消息 (但 ==业务含义相同) ， 所以 无论如何，消费者 必须做 幂等性==
。MQ宕机的时候， 生产者会 收到什么？ timeout？ 还是网络不可达？ 就是能不能区分 ，应该不能，所以 要 确认下， 重发几次？ 不然 一直卡在这里。  不。应该能区分 ( MQ宕机 无法TCP握手的，但是网络波动 也是无法 握手 )，但不管了，有可能就是网络 波动 导致不能发送。

MQ宕机， MQ开启持久化，就不怕宕机了。
MQ默认 异步刷盘， 即 先把 消息 写入 磁盘 缓冲区，但是 不 flush。
还有， MQ集群模式，  同步刷盘 会等待 从机 持久化消息。

MQ -> 消费者， 这种 需要MQ 开启ACK， 确保 消息一定被消费。

消费者宕机， 需要MQ开启ACK， 还需要注意： ==幂等性==，要用redis， redis才是 分布式唯一的， 不要用db，db可能幻读，不，用db的话，需要用 not exists， db应该也可以， 但是redis 更好，纯内存，快， 而且 幂等性的校验 不太需要 持久化到 db。  看具体情况。
主要是： redis 要设置过期时间，然后 看门狗 延长时间。  不然： ==处理到一半，宕机，那么这条消息会因为幂等性校验 而永远不能 被消费==， 所以看用看门狗， 或者 定期 清除 过期很久的任务， 或 TTL 设置合理时间。

---

重传，持久化，ack，是保证 消费者必然能拿到消息

消费者 做幂等性， 确保 只消费一次

---




## 大事务+MQ使用不当导致数据不一致

BV1cjsTekEhj

太长了。



。# mq end











# 影响计算机科学发展的25篇论文

BV13xp3eNEBm

## 分布式系统和数据库

- google file system: a scalable distributed file system for large workloads
  介绍了 高度可扩展的分布式文件系统，专为 处理海量数据密集型应用 而创建
  GFS 可以处理故障。 使用廉价的硬件，为用户提供高性能的 文件系统。
  和传统文件系统不同， 因为 
  - 它会预计 发生故障  
  - 它能对 频繁追加 和 顺序读取 的大文件 进行优化
- dynamo: amazon's highly available key-value store
- bigtable: google's distributed storage system for structured data
- cassandra: a scalable nosql database with tunable consistency
  facebook设计， cassandra结合了 亚马逊的 Dynamo，谷歌的 Bigtable，提供一个 可高度扩展的 多主复制系统，读写速度极快
- spanner: google's globally distributed and strongly consistent database
  通过 提供全球一致，高度可用 和可扩展的系统，改进了 分布式数据库
  引入了 TrueTimeAPI， 利用时间同步 实现一致快照 和 多版本并发控制，支持 非阻塞读取 和 无锁只读事务 等 强大功能
- foundationDB: a distributed key-value store with ACID transactions
  通过其 多模型 键值存储架构 引入了 一种处理 分布式事务的新方法
  它以 跨分布式系统的 ACID事务 著称，提供强大的一致性 并 支持各种数据模型
  它的层设计 支持在单个强大核心之上的 各种数据模型，具有很强的适应性
- amazon aurora: aws's high-performance relational database service
  分离存储和计算，突破高性能数据库的极限
  这种设计可实现 可扩展的 弹性存储，并能根据需要 自动增长和缩小
  还提供了 高可用性和 耐用性，在3个可用区 内进行 六向复制，来确保数据的完整性和容错性



## 数据处理和分析

- mapreduce: parallel processing framework for massive data volumes
  彻底改变大数据的处理
  实现了 在 商品级 硬件 组成的 大型集群上 并行处理海量数据集
  使得 并行化，容错，数据分发处理 变得更加容易
  hadoop 是 mapreduce 的开源版本
- flink: unified architecture for stream and batch processing
  将 流处理 和 批处理 结合在一起，实现了 实时 和 历史数据的 无缝处理
  它将 批处理 视为 流处理的 一种特殊情况，为构建数据密集型 应用程序 提供了一个强大的 框架
- kafka: internals of the distributed messaging platform
  分布式 消息传递和实时数据流的 领先平台
  创建 可靠，高扩展，容错的 数据管道
  它将 数据组织 成 主题，生产者发布数据，消费者检索数据，所有这些都有 中间人服务器管理，来确保数据的 复制和容错
  高吞吐量和 低负载延迟 使其成为 需要 实时数据 处理应用程序的 理想选择
- dapper: google's distributed systems tracing infrastructure
  论文介绍了 一个分布式跟踪系统，通过提供 低开销的 应用级 透明度 来帮助排除故障和优化复杂系统
  它强调了 抽样的使用 和 用最小的设备 来保持性能，同时提供 对复杂系统行为的 宝贵见解
- monarch: google's in-memory time series database architecture
  内存时间序列数据库，旨在高效存储和查询 海量时间序列数据
  采用区域化架构，具有 扩展性和可靠性
  通过每秒 摄取 TB级别数据 和 提供 数百万次查询，使其成为 监控大型应用程序 和系统的 理想选择


## 分布式系统中的挑战

- borg: large-scale cluster management at google
  解释了 谷歌如何管理 其大规模集群
  介绍了 容器的概念，并展示了 集中式 集群管理系统的 优势
- shard manager: understanding the generic shard management framework
  本论文为 管理分布式系统中的 分片 提供了一个通用架构
  通过 自适应调整分片位置 来应对 故障并优化资源利用率，从而简化了 大规模数据库的 扩展和管理过程
- zanzibar: google's global system for managing access control lists
  全球访问控制系统
  它能在 大规模分布式系统中 有效管理 访问控制列表
  提供了一个统一的数据模型和配置语言，用于为 各种谷歌服务 表达不同的访问控制策略
  可以扩展到 每秒处理 数万亿(。。确实，是trillion)个访问控制列表 和 数百万个授权请求
- thrift: explore the design choice behind facebook's code-generation tool
  探讨了 代码生成工具 背后的设计选择
  强调了使用通用接口定义语言 构建可扩展，可维护系统的 好处
- raft consensus algorithm: a more understandable distributed consensus protocol
  为 paxos 提供了一个更容易理解的 替代方案
  简化了 构建容错分布式系统的过程
- time clocks and ordering of events: concepts of time and event ordering
  1978年的重要论文，引入了 逻辑时钟的概念
  建立了 分布式系统中 事件的部分排序，为同步事件提供了 一个框架，无序依赖物理时钟即可解决同步问题

## 突破性的概念和架构

- attention is all you need: introduction to the transformer architecture
  2017年的论文，介绍了transformer架构， 对NLP 产生了 巨大影响
  论文展示了 自我注意力机制 的有效性
  它们允许模型权衡句子中不同词语的 重要性
  带来了GPT
- bitcoin: the pioneering peer-to-peer electronic cash system
  为 加密货币和区块链 奠定了基础
  提出了 去 中心化点对点 电子现金系统的概念，引发了 数字货币 和 去中心化应用的 新时代
- goto considered harmful: exploring the drawbacks of GoTo statement
  1968年论文，挑战了 传统智慧，引发了 关于编程语言设计的 重要讨论
  该论文反对使用 goto， 倡导 结构化编程实践
  
。。goto: 这也能有我？


## 应用和优化

- scaling memcached at facebook: complexities of large-scale caching systems
  论文展示了 大规模缓存的 复杂性
  强调了 构建分布式缓存系统 以提高应用性能 所面临的挑战 和解决方案
- myrocks: facebook's LSM-tree storage engine
  介绍了 基于 LSM-tree的数据库存储引擎
  展示了 如何优化大规模数据库的 存储和检索操作
- wtf: twitter's "who to follow" user recommendation service
  为建立有效的推荐系统提供了启示
  展示了 基于社交图谱分析 生成个性化用户推荐 所使用的算法和技术
- a comprehensive survey on vector databases: algorithms behind vector similarity search
  涵盖了 基本原理，如何构建 矢量数据库 及 矢量数据库的 各种用途
  重点讨论了 如何设计这些数据库，以高效处理和搜索 复杂的 高维数据





# 20世纪十大算法

BV1qQ2zYeE2r

。。 up是理论物理博士， 所以偏向于 数学，物理方面。
。。 不知道来源， up是解读一下这些东西 ( 就是他有没有用过，哪里用的多)


- the metropolis algorithm
  蒙特卡罗方法(量子力学)的基础
- simplex method
  单纯性方法。用于线性优化方面的优化
- krylov subspace method
  量子力学使用
- the decompositional approach to matrix computations
  矩阵计算的分解
  量子力学使用
- the fortran optimizing compiler
  fortran编程语言
- qr algorithm
  矩阵对角化
- quicksort
  快速排序(。选一个标准，将数组分为 小于，不小于，然后 递归)
- fast fourier transform
  快速傅里叶变换
- integer relation detection
  数值计算中，你怀疑是 pi， 可以用这个算法来 判断是否是 pi
  和 LLL算法有关
- fast multipole method
  快速多级展开
  天体物理


---

评论

backpropagation algorithm 。 深度学习的

kmp，dfa，nfa
。kmp的本质是构造一个 dfa
。dfa: 确定性有限状态自动机
。nfa: 不确定的有穷自动机


---

# 圆周角性质

baike

顶点在圆上，两边都和圆相交的角

圆周角定理: 在同圆或等圆中，同弧或等弧所对的圆周角都等于这条弧所对的圆心角的一半





---

# 场景

## 每天1000w订单，买家卖家都要查询最近30天的订单，如何实现

BV1xCA5eDE7F

这是典型的 大数据量+高并发 场景

30天 3亿数据。 一年36亿

。。我的
1. 分表: TX_Order_20250220, TX_Order_20250221 ... 先建几百个表，每天一个。
   1. 冗余，只要一条 单表的 select， 不需要 join，不需要 1+n。
   2. 但是，一旦表增加字段，就需要所有的表都改，不知道有没有办法 让这些表 共享一个 配置，只是名字不同?
   3. 最朴素的逻辑: 买家点击所有订单，发一个请求到后端，一般来说，都是分页的，所以只需要10条数据，所以从 今天的数据库里找，然后从昨天的，前天的。。 一直找直到 满10条 或 找完30个数据库。
   查到的cursor需要返回给 前端，然后下次前端的请求 需要带上这个 数据库的cursor 。。 但是后续想到，为什么不是 前端分页呢。
      1. bloom filter， 即redis的 hyperloglog 。。查了下，不是， hyperloglog 带 计数器，一般用于 网站UV数据。   
      查数据库前，先bloom filter一下。
      redis 有插件 RedisBloom 可以实现 bloom filter
      2. 缓存， redis
         1. blomm之后，先去redis查。 。。这里是 在查数据库时一次性全部查出来，然后放到redis里，然后 后端自己分页? 。。 就是分页的问题，可能 又买了一单， 如果分页后存redis里，新的单 会破坏分页的。  但是如果数据量很大呢。 后端每次要从redis拉取所有数据，然后分页。有点麻烦。。。。。但是 为什么不是 前端分页呢? 我后端把 所有数据传过去，前端自己想办法 慢慢展现。 因为订单 的商品图肯定是 要单独请求的( 或者单独批量请求，就是前端分页找出10条数据，然后发一个请求，要求10个图片)。
   4. 但是 应该需要展现所有订单吧。
   所有订单的话，应该是 按人分表? 一张表里 存这个人所有的订单?
   卖家 很多数据呢?

2. 订单一般不可变/修改很少，所以直接 NoSQL?，或者 sql的字段里存 前端能直接使用的 JSON， 或者redis存 JSON，这样 后端就不需要 将 数据转为JSON 了。


上面的我的问题:
1. 少分库， 会导致 单库的 查询，IO 压力很大。 单库应该无法承受
2. 查询用ES
。。


先找到业务背后的 问题/挑战
- 存储压力，如何高效存储 几十亿的数据
- 查询性能，如何快速定位到 30天内的数据
- 高并发处理，1000w订单，qps肯定高

---

解决 存储

分库分表之前要想到一个问题: 买家，卖家 的数据 存一起 还是分开     。。。应该都行吧， 一起:省数据库，分开:查询快
存一起: 开发简单，一致性较好， 但是性能问题
分开: 扩展性 灵活性好， 支持大规模的订单数据，支持高并发业务查询
。。。感觉说了点什么。

==分库分表==策略
分库策略: 按 `用户ID` 或 订单创建时间， 比如 用户ID%10， 即有10个数据库
。。分库。。 分库确实应该 不能伸缩，所以 MOD 是很好用的。 。。 分库应该伸缩吗?  不过伸缩也无所谓， 一致性hash 可以秒掉。

分表策略: 每个库 按 `月份` 或用户分表， 比如每个库中 按月份分为12张表。
。。1个月3亿， 一个库 3千万， 所以 一张表里 三千万的数据。 也可以接受。
。。确实，不分库的话， 单库的 查询压力，IO压力很大的。


一周内的热点数据， 半年前的冷数据
所以 ==冷热分离==
策略: 
- 30天内的数据 存 分库分表后的 mysql
- 30天前的，`按月 归档到 大数据`，如 TiDB，Hadoop，ClickHouse。 如果需要数据，使用离线分析，异步获取

---

==复杂业务查询==
如: 查询卖家 30天内  金额>100 且 已完成 的订单
方案: `ES搜索`。
。。所以 要把数据 存mysql 也存ES?

---

高并发下 热点数据 ES/mysql性能问题
==缓存方案==
热点数据 提前`预热`到 redis


---

评论

大公司修改数据库，直接底层支持 分库分表。 而不是 应用层分库分表
  大公司直接上 分布式数据库， 不需要关心 大表

---

方案1: 小公司，直接分库分表，冷热分离 即可。 复杂查询 放 ES集群中，多部署几台，分片多点。
  海量数据库，不用ES的情况下， 分库分表，每张表百万级别，按照 用户+日期 来分库分表，数据查询 按 天，周，月 查询。  从产品层面拒绝跨月 的需求。

方案2: 阿里的dataX + Hologre + dataComputer + etl 流批一体的方案 按天 输出到 宽表进行统计，支持 对海量数据 的实时分析 和 多维度，跨月的复杂查询

---

数据量这么大的公司 早就有专门的 数据维护和数仓，根本没开发的事情

---

这些方案 都没有串起来， 更别说 一致性 时效性 等扩展问题了。


---



## 如何实现消息实时推送

BV1R3Nre5EKf

场景: 
- 别人点赞或评论，系统会给你实时推送消息。
- 在线聊天，实时感知对方发送到消息
- 多人在线游戏，斗地主/麻将/五子棋，需要知道其他人走了什么牌
- 共享设备(自行车，充电宝)，手机点击开锁，共享设备 就解锁。


常见的方案
- 长连接
- 短连接
- SSE
- MQTT

题外话: 在大型企业中，消息推送的场景非常多，所以会构建一个 消息推送中心。 只需要往 消息推送表中 插入数据。

---

轮询: 分为长轮询，短轮询

短轮询: 前端 定时任务，每个1秒发送 ajax请求给后端。  
缺点: 资源浪费非常严重，服务器压力很大

长轮询: 前端发送请求，后端不会立刻响应，而是将线程挂起。 只有当新消息出现时，才会响应给前端
改进: 等待事件过长，前端会请求超时 -> 挂起一段时间后，后端返回 空数据 给前端。
缺点: 请求过多，服务器线程太多，压力大

所以轮询 只能处理 请求量低的场景， 不能高并发

---

SSE: 前端发请求给后端，后端在 响应中 开启 SSE长连接的协议`resp.setContentType("text/event-stream");`。 前端会和 后端建立连接，后端发生数据改变，可以实时推送消息给前端。
这个是 `单向`的。
不需要额外库，是http自带的

---

WebSocket: 前端 发送长连接的请求，后端同意后 会建立双向通道。
普遍使用的技术
通常基于 netty开发，因为netty 有NIO模型，对websocket进行了封装。

---

MQTT: 物联网，物联网的 网络不稳定，无法像http那样实时请求响应。 而且传输的数据不大。
为了保证可靠性，有3个级别
- QoS0: 最多一次交付， 发送后，不需要ack
- QoS1: 最少一次交付， 发送后，需要ack，超时就 重传
- QoS2: 只有一次交付， 发送后，等待对方ack，收到后删除消息。 如果没有收到ack，就重传。 QoS2的消息 不丢失 不重复。 。。。 有重传怎么不重复? 估计是 client那边有幂等性设置。


---



## 查询接口耗时100ms，QPS10万，怎么配置线程池线程数

BV1cwNreHEA4

业务场景的特点
- IO密集型，线程阻塞
- 大量短任务，执行时间快
- RT(response time)敏感，高并发的处理能力

解决方案重点
- 增加线程数  。。 IO时 其他线程可以CPU处理
- 核心线程数等于最大线程数  。。 减少 创建，销毁 线程的 消耗
- 监控负载 + 动态调整


网上的 坑人的方案
- IO密集型: CPU核心数 * 2
- CPU密集型: CPU核心数 + 1

线程池设置需要考虑
- 任务性质 。。 IO密集，CPU密集
- 延迟和RT时间 。。 可以容忍无限延迟的话， task queue 可以无限长
- 系统资源 。。CPU核心 影响 线程数量
- 队列类型  。。 ?这有什么用 或 这是什么?
- 任务量 。。 task queue上限
- 实时监控 和 动态调整


。。实际上就是 阿姆达尔定律。

看视频上，有个对比，机器是8核16G，使用 坑人方案的 CPU*2， 和他调整后的。 10万 100ms的任务
坑人方案是 630s
调整后是 6s

。。这个应该就是sleep的问题。
CPU*2 : 16个线程， sleep 就是等于 一个核 2个并行。 所以 16个并行， 所以 每100ms 处理 16个。
所以 100000 / 16 * 100ms = 625000ms = 625s , 所以 视频上说 630s

调整后: 看视频可以看到 核心线程数:10000。。。。 1万个线程。
。这个只能反推啊。 6秒处理10万个任务，8个CPU核，所以 每个核 6秒处理 10/8 万个任务
一个任务 0.48ms。 应该就是 直接 sleep(100ms)的， 所以 sleep的消耗是 0.48ms
。一个任务0.48ms  10w任务 6s处理完  需要多少根线程? 
。。。一个核 1.25w任务， 6秒， 每毫秒要完成 2.08333个任务。 推不出来啊。。
。。。。反推了， 1个线程: 0.48ms一个任务，6秒能完成 12500个任务。
。。。嗯，0.48 是实际的消耗，无法推出 线程数的，只能推出 多少个核。
。。知道了 100ms，实际上CPU只需要0.48ms，空闲时间可以处理其他线程， 所以 可以支持 100/0.48 = 208个线程 并发(行)。 一个核208.333个线程，8核就是 1666.666个线程。  所以 1666个线程 就可以 6s 了。
。。



## Redis与DB的一致性 如何解决

这里介绍5种方案

问题场景
1. 选择先修改DB，然后删redis， 但是 修改DB成功，删除redis失败。
2. 选择先redis，然后修改DB， 但是 缓存删除后，有用户读取该条数据，那么由于redis不存在，所以查DB，查到的是 旧数据，然后保存到redis。  最后是 修改DB。

方案
1. 双写事务， redis事务 和 DB事务，一起，同时提交，同时回滚。  
   缺点: 2个事务 终究不是 一个事务，所以依然可能出现不一致。   。。而且redis事务，岂不是很慢。
2. 延迟双删，`很常用`。 更新DB后，立刻删除redis，然后 延迟1,2秒，再次删除redis
3. 订阅更新， 修改DB后，向MQ发送消息，redis订阅MQ，将数据从缓存中删除
4. 读写分离， `高并发`场景应用很多， 修改数据库，后台有任务 将数据库中的数据 同步到 redis
5. Canal订阅Binlog同步redis。

3,4,5 的会有延迟吧?



## 订单超时自动取消如何实现

BV1cwNreHEA4

场景
- 待支付 超时
- 优惠券 过期
- 秒杀 超过有效时间
- ERP，OA系统的 定时会议

方案
1. JDK自带的DelayQueue
   缺点: 保存在内存中，重启/崩溃 就消失， 所以 每次启动都要 读取数据库中的 未支付的订单，加载到内存中。     占用内存大
   优点: 非常简单，成本低
   适合简单项目

2. MQ延时消息 RocketMQ
   优点: 使用简单;  支持分布式;  精度高，支持任意时刻
   缺点: 定时任务最长24小时;   要存储一段时间内的所有订单，MQ存储成本;   同一时刻的大量消息会导致 消息延迟。

3. 基于redis 的过期监听
   开启过期监听: `notify-keyspace-events Ex`。 java中继承 KeyExpirationEventMessageListener
   优点: 和MQ一样， 简单，分布式，任意时刻
   缺点: 不可靠， redis在过期通知时，如果app正好重启，那么通知事件可能丢失。  需要定时任务来补偿;     占据大量空间

4. 阿里内部使用 最简单的 定时任务 来进行定时的分布式`批`处理
   优点: 批处理，而不是 MQ之类的 单个任务。  可靠:app重启无所谓。    不需要中间件(成本):直接app里定时任务扫描。
   缺点: 精度差，每秒一次 压力太大。

5. 超时中心，把所有需要超时处理的 都放到 超时库中，单独的 app进行处理。


对于 精度要求高，超时时间24小时内，不会有峰值压力 的场景， 推荐 RocketMQ



## 如何防止重复下单

网络阻塞，用户重复点击下单， 出现重复下单

1. 前端，用户点击下单后，把按钮置灰
2. 后端，redis set NX 来保证 幂等性， 使用用户id作为key，3秒5秒过期时间。 也可以 用户ID+商品ID+操作



















