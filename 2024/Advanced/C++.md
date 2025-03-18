C++
2023年4月8日
18:21

[[toc]]


==================
6个内存序，3种类型

--------------
https://blog.csdn.net/wxj1992/article/details/103649056

# C++ memory order循序渐进（一）—— 多核编程和memory model

C++ 11标准正式引入的memory order

一个典型的多核CPU架构：

![在这里插入图片描述](../_resources/b67d0532cfd14639a0a14f498ffcbc00.png)

。。不过我baidu  了下，主要是  上图中  它就  2级cache。。
下面这个应该更准。。而且上面的  RAM 512m  。。

![在这里插入图片描述](../_resources/5d2b73f3e786472bb221b841dff390ab.png)

。。

现代cpu使用的是一套包括MESI协议、store buffer、invalid queue等技术在内的数据同步方式，以及流水线乱序的执行模式，数据在各个核之间的一致性和可见性并不是那么理想，再加上编译器也会做优化，最终的结果就是各个核的指令执行顺序和各个变量值的可见性变得不确定，从某种意义上都可以统称为重排

。。MESI：  modify, exclusive, share, invalid。修改，独占，共享，无效，代表了  高速缓存行  的4种状态。

cpu和编译器也是有底线的，无论产生什么样的重排，都会保证对于单线程内部的执行结果不会有任何区别，但是进入多核时代后，原本人畜无害的重排可能会出现大问题，因为单个核的内部的执行和优化并不会考虑和其他的核互动

例如：
// thread 1
p.init();
ready = true;

// thread 2
if (ready) { p.bar(); }

thread 1  中，p  和  ready  没有关联，所以可以被重排序。

锁，很方便，可以在不了解太多底层的情况下  写出正确的并发代码。但是   并发高时，容易成为  瓶颈。

多核编程中的临界区保护
并发编程中最核心的技术：临界区保护
对于临界区保护通常可以分为三个级别：互斥、lock-free和wait-free。

互斥锁
悲观锁算法
阻塞等待
阻塞传播

lockfree
Lock free programing
非阻塞算法
乐观锁算法
有着不同的表述方式

如果涉及到共享内存的多线程代码在多线程执行下不可能互相影响导致被hang住，不管OS如何调度线程，至少有一个线程在做有用的事，那么就是lock-free。前面讲到了使用了锁的代码肯定不是lock free，因为一个线程加锁后如果被系统切出去了其他所有线程都处于等待中。

lock-free需要处理更多更复杂的race condition和ABA等问题，编写出合理的lock-free代码也需要更深厚的功底，需要对底层有更多地了解，完成相同目的的代码会比用锁更复杂，执行时间可能更长，代码也更难理解。很多场景合理地使用锁就能很好的胜任，lock-free和锁之间在应用场景上更多的是一种互补的关系。lock-free算法的价值在于其保证了一个或所有线程始终在做有用的事，而不是绝对的高性能。但lock-free相较于锁在并发度高（竞争激烈导致上下文切换开销变得突出）的某些场景下会有很大的性能优势，比如实现一个多线程的lock-free queue，总的来说，在多核环境下，lock-free是很有意义的。

wait-free
解决了临界区内的阻塞传播问题
但是本质上，依然是多个线程排队顺序经过临界区
wait-free级别和lock-free的主要区别也就体现在这个吞吐的问题上
虽然理论角度存在不少有Wait Free级别的算法，但大多并不具备工业使用价值

比较有实际意义的Wait-Free Population Oblivious级别来说，额外限制了每个参与线程必须要在预先可给出的明确执行周期内完成，且这个周期不能和与参与线程数相关。这一点明确拒绝了一些类似线程间协作的方案（这些方案往往引起较大的缓存竞争），以及一些需要很长很长的有限步来完成的设计。时至今日各种数据结构上工业可用的wait-free算法依旧是一项持续探索中的领域。

lock-free和wait-free编程既然要抛弃传统意义上的锁，那么往更底层走自然成为必然，其中最重要的两个相关技术就是原子操作和控制memory order。

原子操作主要包括赋值原子操作、Read-Modify-Write（比如c++11里的fetch_add）、Compare-And-Swap(比如c++11里的compare_exchange_strong)。原子操作保证了各线程在进行共享内存的存取的时候能读到完整的值。

对于cpu来说，执行代码对内存的操作就形成了memory order，而这个order受编译器和cpu运行时重排的影响，当然这两种重排是有方法可以限制的。，显示的锁在底层实现上往往通过手段保证了一些顺序和可见性，因为无锁算法没有显式的锁，将会直接观察到这些和代码顺序不一致的重排，给无锁算法的实现带来了挑战。

c++的memory oder更多则是一种约定和方法，给程序员提供了一种跨平台的通用方法来限制上述两种重排，这也极大地方便了我们实现无锁算法，在此之前，c++程序员如果想要限制重排，通常需要手动根据不同的平台选择使用特定的内存屏障指令。

关于程序运行时的memory order，重点提一下sequential consistency，假设没有重排，那么就是sequential consistency的内存序，也就是顺序一致，所有的线程都能看到一样的内存修改顺序，同时这个顺序和程序的源码是一致的，也就是各线程的内存操作是按顺序交织的，这种memory order下，程序的行为很好预期，但目前没有任何主流cpu默认提供sequential consistency保证，也就是都可能会进行不同程度的指令重排，最终会出现什么样的memory oder，则是由对应的memory model决定的。


## Memory model

某个处理器或者工具链对于代码的重排会严格遵循对应的memory model，什么情况可以重排、什么情况不能重排。这里强调一点，这里讨论的重排只是针对单个线程内部在单个核内的指令的执行顺序问题。本质上，可以理解为我们指定memory_order，就是通过限制重排来来保证共享数据的可见性和正确同步。

reorder类型和Memory model的强弱
对内存的操作可以概括为读和写，也就是load和store，因此reorder也就可以整体上分为以下四种类型：
（1）Loadload reorder：两个读操作之间重排
（2）Loadstore reorder：原来在写操作之前的读操作重排到之后
（3）Storeload reorder：原来在读操作之前的写操作重排到之后
（4）Storestore reorder：两个写操作之间重排

DEC Alpha，四种reorder都有可能发生，只要不改变单线程内部的执行正确性。
。查了下  DEC Alpha，上世纪的，  RISC CPU。  技术很强，但是商业。。。

ARM架构的cpu则是在此基础上额外保证了数据依赖顺序。

X86/X64这种就属于强memory model的范畴了，此类平台只可能发生Storeload reorder。

### Compiler Barrier  和  Runtime Memory Barrier

无论是哪种memory model，hardware memory model还是 software memory model，上面说到的这些可能的重排都是指的在没有其他限制的情况，为了能够保证程序的正确性，CPU和编译器（语言）的设计者都给我们留了手段来改变这些重排，这类手段可以抽象成一个统一的概念barrier，屏障。无论上面提到的那些memory model是定义在哪个层面的，要想限制重排，具体到最后的程序的编写、编译、运行步骤，总是需要程序员用代码来限制编译阶段和运行阶段的重排，因此可以分为Compiler Barrier和runtime memory barrier。

Compiler Barrier，顾名思义，编译器层面的屏障，可以防止编译器在将源码转换成机器码的过程中重排，一个例子如下
int a, b;
void foo()
{
a = b + 1;
b = 0;
}
使用GCC 4.6.1  不开启优化  获得的汇编：
gcc -S -masm=intel foo.c
cat foo.s

mov eax, DWORD PTR _b (redo this at home…)
add eax, 1
mov DWORD PTY _a, eax
mov DWORD PTY _b, 0

开启优化后：
gcc -O2 -S -masm=intel foo.c
cat foo.s

mov eax, DWORD PTY b
mov DWORD PTR b, 0
add eax, 1
mov DWORD PTR a, eax

出现了重排。
。。不过，这个重排有什么意义？

不想重排，就可以使用  Compiler Barrier，  GCC中，  asm volatile("" ::: "memory")  就是一个  compiler barrier

int a, b;
void foo()
{
a = b + 1;
asm volatile("" ::: "memory");
b = 0;
}
开启优化  和  不开启优化，一样。

asm volatile("" ::: “memory”)是显式的Compiler Barrier，它只能保证编译阶段不重排，在多核系统里，光做到这一点还不够，因为它没法对cpu核心运行时的重排做出限制

runtime memory barrier是限制运行时重排的手段，通常是通过特定的CPU 指令来实现，上面提到了重排基本可以分为四种类型，因此对应的barrier也有四种

（1）Loadload barrier：保证barrier前的读操作比barrier后的读操作先完成。
（2）Loadstore barrier：保证barrier前的读操作比barrier后的写操作先完成。
（3）Storeload barrier：保证barrier前的写操作比barrier后的读操作先完成。
（4）Storestore barrier：保证barrier前的写操作比barrier后的写操作先完成。

以GCC为例，GCC对于PowerPC，定义了一个宏：
#define RELEASE_FENCE() asm volatile(“lwsync” ::: “memory”)

lwsync是PowerPC平台的一个cpu指令，可以限制运行时重排，它同时具有LoadLoad barrier, LoadStore barrier StoreStore barrier的效果。具有runtime memory barrier的效果，而且需要重点注意的是，一旦在代码里插入了RELEASE_FENCE()，除了会插入一条lwsync指令限制运行时重排之外，它也会阻止编译器编译阶段重排，也就是说，RELEASE_FENCE()不光是runtime memory barrier ，也是隐式的Compiler Barrier。

----------------------

https://blog.csdn.net/wxj1992/article/details/103656486

C++ memory order基本定义和形式化描述所需术语关系详解

## C++的六种memory_order

### memory_order_relaxed

宽松的内存序，使用这个内存序的原子操作就真的仅仅是保证自身的原子性，不会对任何其他变量的读写产生影响。如果确实不需要其他的内存同步，那么这是最好的选择，比如原子计数器。

### memory_order_consume

memory_order_consume适用于load operation（原子读操作），对于采用此内存序的load operation，我们可以称为consume operation

对周围内存序的影响是：当前线程中该consume operation后的依赖该consume operation读取的值的load 或strore不能被重排到该consume operation前，其他线程中所有对M的release operation（下面memory_order_release讲到的）及其之前的对数据依赖变量的写入都对当前线程从该consume operation开始往后的操作可见

相比较于下面讲的memory_order_acquire，memory_order_consume只是阻止了之后有依赖关系的重排。

绝大部分平台上，这个内存序只会影响到编译器优化，依赖于dependency chain。但实际上很多编译器都没有正确地实现consume，导致等同于acquire。

### memory_order_acquire

memory_order_acquire适用于load operation，对于采用此内存序的load operation，我们可以称为acquire operation

对周围内存序的影响是：当前线程中该acquire operation后的load 或strore不能被重排到该acquire operation前，结合下面的memory_order_release我们能推导出从而会有其他线程中所有对M的release operation及其之前的写入都对当前线程从该acquire operation开始往后的操作可见。

。这个不需要  依赖load  的值。  所以是  所有的  后续  load store  都不能  重排到这个前面。

### memory_order_release

memory_order_release适用于store operation（原子写操作），对于采用此内存序的store operation，我们可以称为release operation

对周围内存序的影响是：该release operation前的内存读写都不能重排到该release operation之后。结合memory_order_acquire的作用从而有：

当前线程截止到该release operation的所有内存写入都对另外线程对M的acquire operation以及之后的内存操作可见，这就是release acquire 语义。

当前线程截止到该operation的所有M所依赖的内存写入都对另外线程对M的consume operation以及之后的内存操作可见，这就是release consume语义。

### memory_order_acq_rel

memory_order_acq_rel适用于read-modify-write operation，对于采用此内存序的read-modify-write operation，我们可以称为acq_rel operation，既属于acquire operation 也是release operation.

设有一个原子变量M上的acq_rel operation：自然的，因为同时具有两种属性，所以该acq_rel operation之前的内存读写都不能重排到该acq_rel operation之后，该acq_rel operation之后的内存读写都不能重排到该acq_rel operation之前. 其他线程中所有对M的release operation及其之前的写入都对当前线程从该acq_rel operation开始的操作可见，并且截止到该acq_rel operation的所有内存写入都对另外线程对M的acquire operation以及之后的内存操作可见。

### memory_order_seq_cst

memory_order_seq_cst 可以用于 load operation，release operation, read-modify-write operation三种操作，
用于 load operation的时候有acquire operation的特性，
用于 store operation的时候有release operation的特性,
用于 read-modify-write operation的时候有acq_rel operation的特性，
除此之外，有个很重要的附加特性，一个单独全序，也就是所有的线程会观察到一致的内存修改顺序，也就是第一篇博客提到的顺序一致性的强保证。

这里所谓的指定内存序，指的是对执行语句所在的 **线程内部** 的限制，也就是只影响一个cpu核心，但是这些对**单线程**内部的限制**组合**起来就能实现**多线程**之间数据同步的效果，我们利用的就是各种内存序之间的组合效果来保证程序在实际运行时候的正确memory order


## 形式化定义用到的术语和关系

### Sequenced-before
定义的是同一个线程内的一种根据表达式求值顺序来的一种关系，完整的规则定义很复杂，可以参考 http://en.cppreference.com/w/cpp/language/eval_order ，其中最直观常用的一条规则简单来说如下：**每一个完整表达式的值计算和副作用都Sequenced-before于下一个完整表达式的值计算和副作用**。从而也就有以分号结束的语语句Sequenced-before于下一个以分号结束的语句

```
	r2 = x.load(std::memory_order_relaxed); // C 
	y.store(42, std::memory_order_relaxed); // D
```
有 C Sequenced-before D。


### Carries dependency
字面意思是携带依赖，描述的是有sequenced-before关系的求值操作之间的一种**依赖关系**，对于求值A和求值B，A Carries dependency into B的意思是B对A有依赖，在A sequenced-before B的前提下，如果：
- A是B的一个操作数
- A写入了某个标量对象（float、int、指针枚举等）M，B从M里读
- 求值X依赖A，B依赖X

那么我们说B依赖A。

### Modification order
字面意思是修改顺序，这个是针对原子变量的一个概念，对于c++里的原子变量，有个很重要的保证，那就是：对于任意特定的原子变量的所有修改操作都会以一个特定于该原子变量的全序发生，换句话说，不管是什么序的原子变量操作，针对原子变量本身是会有一个全序的，因此也保证了**原子变量的操作**在各线程之间有如下**四个一致性**（为了描述简洁下面说的读写都是针对某一个特定的原子变量）：
- 写-写一致性：如果写A happens-before 写 B ，那么在Modification order中A 比B早出现
- 读-读一致性：如果读A happens-before 读B, 并且如果 A 读到的是来自写X， 那么B读到的要么也是写X的值，要么是Modification order中比X后出现的一个写Y 的值.
- 读-写一致性：如果读A happens-before 写B, 那么A读到的值来自于modification order 中的一个早于B出现的写X
- 写-读一致性：如果一个写X happens-before 读B , 那么B读取的值可能来自写X或者modification order 中在X之后出现的写Y。

### Release sequence
Release sequence，指的是某个原子变量M上由一个release operation开始的一个序列，正式定义如下：

原子变量M上的release sequence headed by A指的是M的Modification order上的一个最大连续子序列，该序列由A开始，当前线程内对M的写操作（这个在c++20被去掉了，这里先忽略），以及其他线程对M的read-modify-write操作（test-and-set, fetch-and-add, compare-and-swap等）组成。

对应的，有一个比较重要的release sequence rule，能够保证特定情况下多个acquire线程和个release操作形成Synchronizes-with关系，比如线程A release store，B fetch_sub，后面展开讲release acquire语义的时候会详细讲下，暂时了解Release sequence这个概念就行。


### Synchronizes-with
同步关系，用来描述对内存的修改操作（包括原子和非原子操作）对其他线程可见，有一点需要注意，这是一个运行时的关系，也就是说不是代码层面的，代码只是让程序在运行的时候有可能建立这种关系，并且一旦建立了就有可见性的保证。简单来说，如果A和B建立了Synchronizes-with关系，那么A之前的内存修改对B之后都可见

release 和 acquire就能够建立Synchronizes-with关系。

。。但是怎么建立？这里看图，还是靠时序，需要 store 先发生，然后 load。如果 load 先发生，store 不可能同步。 。

### Dependency-ordered before
这个关系是针对consume来的，对于分属不同线程的赋值A和赋值B，如果他们之间有以下关系之一：
- A对某个原子变量M做release操作，B对M做consume操作，并且B读到了release sequence headed by A中的任意一个值。这条就是上面说的release sequence rule的关键。
- A is dependency-ordered before X，并且 X carries dependency into B。前面说过了，carries dependency into是线程内的一种关系，这里就是X和B属于同一线程，并且B对X有依赖。

我们就说 A dependency-ordered before B。


### Inter-thread happens-before
定义了线程间赋值操作之间的关系，假如有线程1中的赋值操作A，线程2中的赋值操作B，如果满足以下**任意一条**，那么有A inter-thread happens before B：
-    A synchronizes-with B
-    A dependency-ordered before B
-    A synchronizes-with 某个 evaluation X, 并且 X sequenced-before B
-    A sequenced-before 某个 evaluation X, and X inter-thread happens-before B
-    A inter-thread happens-before 某个evaluation X,并且X inter-thread happens-before B

虽然看上去有点繁琐，其实每一条都是很自然的，核心就是其中的**传导关系**，多线程编程中，两个操作如果能推导出Inter-thread happens-before的关系，那么内存修改的可见性是有保证的，换句话说就是不用担心重排带来的影响。


### Happens-before
特别重要的一个概念，定义的是两个evaluation（可以通俗地理解为包括读写在内地一些值操作）之间的一种关系，不管是线程内部还是线程之间，evaluation A和evaluation B如果满足下面两者条件之一，我们就说A Happens-before B：
-    A sequenced-before B
-    A inter-thread happens before B
	
如果A Happens-before B，那么A对内存的修改将会在B操作之前对B可见，也就是下面讲到的Visible side-effects，**我们利用memory_order就是为了确保某些evaluation操作之间有Happens-before关系**

### Visible side-effects
先解释下涉及到的一个概念Scalar，指的是以下类型：
-    arithmetic (integral, float)
-    pointers: T * for any type T
-    enum
-    pointer-to-member
-    nullptr_t

可见副作用，通俗地讲就是写能被读看见，正式定义如下：
有对同一Scalar类型的M的写A和读B，如果满足同时满足下述两个条件那么A的副作用对B可见：
-    A happens-before B。
-    不存在一个对M有副作用的写X， A happens-before X 并且X happens-before B。

换句话说，就是A和B有happens-before 关系，并且二者之间没有其他的写，否则可能会读到其他写的内容，从而A的副作用不保证对B可见。


------------------
https://blog.csdn.net/wxj1992/article/details/103843971
# 原子变量上组合应用memory order实现不同的内存序

有六种memory order，要达到数据同步的效果，需要进行组合使用，我们可以在两个地方指定memory order，一是**atomic**变量（原子变量）的操作，二是**fence**，利用fence可以不依赖原子变量进行同步，
本篇文章只介绍atomic上的memory order应用，关于不同memory order的fence使用后面引入fence的概念后再介绍。
总的来说，利用六种memory order的原子操作组合，主要可以得到以下几种内存序。 


## Relaxed ordering
除了保证操作的原子性和对应原子变量的modification order，不会有任何额外的数据同步限制，也就是在保证 ==单线程== 执行效果一致的情况下，编译器在编译时和cpu在运行时可以进行各种重排
对于以下代码，在c++ 11的标准下，是允许出现r1=42并且r2=42的，注意这里说的是标准允许，但是实际在不同的编译器和平台上可能并不能复现。
```
// Thread 1:
r1 = y.load(std::memory_order_relaxed); // A
x.store(r1, std::memory_order_relaxed); // B
// Thread 2:
r2 = x.load(std::memory_order_relaxed); // C 
y.store(42, std::memory_order_relaxed); // D
```
。。就是D 被重排到最前面？ 。 这种怎么能算 单线程 执行效果一致？ 是指 重排后 效果一致？ 但是CPU重排呢？二进制文件只是 编译器重排后的。
。。不，这个是 Thread 1， Thread 2。。
。。所以 D 被重排到 C 前面。 然后 D-A-B-C

Relaxed序 通常 适用于一些 纯计数器类的场景。

## Release-Acquire ordering
Release-Acquire 序，也就是我们通常说的 Release-Acquire 语义，也是 最常用的一种序。
能保证 必要的数据同步，开销也不会太大。
各个线程间的数据同步，最常见的 述求 就是 在 某个线程观察到了 另一个线程 表明某种情况的flag后，与之相关的一些数据 也要确保对 该线程ready。

memory_order_release和memory_order_acquire 结合起来，容易得出下列结论： 如果线程1 中有一个 对原子变量的 memory_order_release 的写入A，线程2 对同一个原子变量 的 order_acquire 的读取B， 那么 所有 happend-before A 的内存写入 对 线程2 B 之后的操作可见，也就是A之前的内存写操作将会成为线程2从B开始的visible side-effects。

对于release sequence有着额外的同步保证，那就是 synchronizes-with 可以扩展到 整个 release sequence 和 acquire load，acquire load到release sequence headed by A中的任意一个写操作写入的值，仍然可以确保对A之前的内存操作的可见性。

```
#include <thread>
#include <atomic>
#include <cassert>
#include <string>
 
std::atomic<std::string*> ptr;
int data;
 
void producer()
{
    std::string* p  = new std::string("Hello");
    data = 42;
    ptr.store(p, std::memory_order_release);
}
 
void consumer()
{
    std::string* p2;
    while (!(p2 = ptr.load(std::memory_order_acquire)))
        ;
    assert(*p2 == "Hello"); // never fires
    assert(data == 42); // never fires
}
 
int main()
{
    std::thread t1(producer);
    std::thread t2(consumer);
    t1.join(); t2.join();
}

```

consumer线程通过循环等待ptr初始化，一旦读到了非null的ptr，因为memory_order_release和memory_order_acquire的同步效果，producer里p和data的内存写入都对consumer中后续两个assert可见了。


```
#include <thread>
#include <atomic>
#include <cassert>
#include <vector>
 
std::vector<int> data;
std::atomic<int> flag = {0};
 
void thread_1()
{
    data.push_back(42);
    flag.store(1, std::memory_order_release); //A
}
 
void thread_2()
{
    int expected=1;
    while (!flag.compare_exchange_strong(expected, 2, std::memory_order_acq_rel)) { //B
        expected = 1;
    }
}
 
void thread_3()
{
    while (flag.load(std::memory_order_acquire) < 2) //C
        ;
    assert(data.at(0) == 42); // will never fire
}
 
int main()
{
    std::thread a(thread_1);
    std::thread b(thread_2);
    std::thread c(thread_3);
    a.join(); b.join(); c.join();
}
```
这个例子展示了Synchronizes-with的传递性，flag初始值是0，在thread_1的A将flag置为1之前，thread_2和thread_3都会一直处于循环中，thread_2中B属于read-modify-write operation，使用的是memory_order_acq_rel order，因此既是release operation，也是acquire operation。一旦A将flag置为1，对于thread_3没有变化，但是thread2里B acquire读到1之后能够成功，这里也就建立了 A Synchronizes-with B，A inter-thread happens-before B ；同时B也将flag release写为2，循环结束。随后C读到2后，也有 B inter-thread happens-before C，根据传递性有A inter-thread happens-before C，因此跳出循环后去访问普通变量data能保证data[0]一定为42。


## Release-Consume ordering
release-consume 和 release-acquire 很类似，区别在于，只是保证dependency-ordered-before release写的写操作的数据同步
比较完整的定义如下：
如果线程1中有一个对原子变量的memory_order_release memory order的写入A，线程2中有一个对同一原子变量memory_order_consmue memory order的读取B，那么所有dependency-ordered-before A的内存写入都对线程2 B 之后**依赖B**的操作可见。

```
#include <thread>
#include <atomic>
#include <cassert>
#include <string>
 
std::atomic<std::string*> ptr;
int data;
 
void producer()
{
    std::string* p  = new std::string("Hello");  //A
    data = 42;//B
    ptr.store(p, std::memory_order_release); //C
}
 
void consumer()
{
    std::string* p2;
    while (!(p2 = ptr.load(std::memory_order_consume))) //D
        ;
    assert(*p2 == "Hello"); // E    never fires: *p2 carries dependency from ptr
    assert(data == 42); // F   may or may not fire: data does not carry dependency from ptr
}
 
int main()
{
    std::thread t1(producer);
    std::thread t2(consumer);
    t1.join(); t2.join();
}
```


但是因为Release-Consume需要跟踪依赖链，目前在很多编译器实现上是直接等同于Release-Acquire的，并且从c++17标准开始， release-consume order还在修订，并不推荐使用。


## Sequentially-consistent ordering
使用memory_order_seq_cst可以获得顺序一致，是atomic变量默认的内存序，也是==最强的保证==，在如果形成了该内存序，除了有release acquire的效果，还会在所有的以这个为内存序的原子操作序列上外加一个单独全序，也就是保证所有的线程观察一组内存操作的顺序完全一样，这组内存操作是可以发生在不同的原子变量上的。这也是代价很大的一种内存序，尤其是在arm等weak order平台上，因为要实现Sequentially-consistent需要施加很强的内存屏障从而==影响性能==，因此如果对性能有较高要求，不是特别必要不要使用这个内存序。

关于原子变量上的顺序一致形式化定义如下：
对于从原子变量M中读取的 memory_order_seq_cst load B , 会观察到以下情况之一:
1. 单独全序中B之前最近的一个对M的memory_order_seq_cst store A。
2. 如果有（1）中的这么一个 A，那么B可能观察到一个不是happen-before A 的非memory_order_seq_cst的修改C
3. 如果没有（1）中的这么一个 A，那么B 可能观察到某个M的非memory_order_seq_cst的无关联修改的结果。

一个必须要Sequentially-consistent的例子如下：
```
#include <thread>
#include <atomic>
#include <cassert>
 
std::atomic<bool> x = {false};
std::atomic<bool> y = {false};
std::atomic<int> z = {0};
 
void write_x()
{
    x.store(true, std::memory_order_seq_cst);
}
 
void write_y()
{
    y.store(true, std::memory_order_seq_cst);
}
 
void read_x_then_y()
{
    while (!x.load(std::memory_order_seq_cst))
        ;
    if (y.load(std::memory_order_seq_cst)) {
        ++z;
    }
}
 
void read_y_then_x()
{
    while (!y.load(std::memory_order_seq_cst))
        ;
    if (x.load(std::memory_order_seq_cst)) {
        ++z;
    }
}
 
int main()
{
    std::thread a(write_x);
    std::thread b(write_y);
    std::thread c(read_x_then_y);
    std::thread d(read_y_then_x);
    a.join(); b.join(); c.join(); d.join();
    assert(z.load() != 0);  // will never happen
}
```

该例是要确保z在运行完后不为0，如果要使得z为0，那么必须同时满足以下两个条件：
1. read_x_then_y()依次观察到x=true，y=false。
2. read_y_then_x()依次观察到y=true，x=false。

这在Sequentially-consistent下是不可能的，因为是全局顺序一致，各个线程观察到的顺序一致，如果有（1），那么说明x先比y被置为true，那么read_y_then_x()在观察到y=true之后必然会观察到x=true，从而（2）不可能。反之如果有（2）那么（1）肯定不可能，因此z必不为0。

-----------------------
https://blog.csdn.net/wxj1992/article/details/103917093
在std::atomic_thread_fence 上应用std::memory_order实现不同的内存序

fence可以和原子操作组合进行同步，也可以fence之间进行同步，fence不光可以不依赖原子操作进行同步，而且相比较于同样memory order的原子操作，具有更强的内存同步效果

最常用的两种就是release acquire和Sequentially-consistent ordering

c++的memory_order属于逻辑上的内存变量的可见性和顺序的限制，并不等同于底层的memory barrier，但在实际的底层实现上就是选择合适的具有特定memory barrier效果的cpu指令，换句话说，指定memory_order可以得到特定的memory barrier效果。


## atomic_thread_fence分类和效果
和atomic变量类似，atomic_thread_fence也可以指定六种memory order，指定不同memory order的fence可以分为以下几类：
1. std::atomic_thread_fence(memory_order_relaxed)，没有任何效果。
2. std::atomic_thread_fence(memory_order_acquire) 和 std::atomic_thread_fence(memory_order_consume) 属于acquire fence。
3. std::atomic_thread_fence(memory_order_release)属于release fence。
4. std::atomic_thread_fence(memory_order_acq_rel)既是acquire fence 也是release fence，为了方便这里称为full fence。
5. std::atomic_thread_fence(memory_order_seq_cst)额外保证有单独全序的full fence。

也就是说，如果不考虑单独全序，那么有release fence、acquire fence 和full fence三种。下面就根据以前介绍过的四种重排来介绍下这三种fence的效果。


## Release fence
Release fence可以防止fence前的内存操作重排到fence后的任意store之后，即阻止loadstore重排和storestore重排。
`std::atomic_thread_fence(std::memory_order_release)`
。这条语句 **前的 load/store 不会 重排到 这条语句后的 store 之后**。
。。应该是不能排到这条语句后面吧。 不然 前面的load 排到这条语句后，但是 任意的store 之前，岂不是也可以？ 没有这么智能吧。
。。可以的。看下面第二页

## acquire fence
acquire fence可以**防止fence后的内存操作重排到fence前的任意load之前**，即阻止loadload重排和loadstore重排
`std::atomic_thread_fence(std::memory_order_acquire)`

。。不，看图，和之前的 loadstore, storeload 的定义。 这里应该是 阻止 loadload 和 storeload。
。之前，storeload 的定义是， store之前的load 被重排到 store 之后。
。。要看下 cppreference了。
。。cppreference 没有 loadload reorder。
。。搜索的时候 搜到一篇 《Non-Speculative Load-Load Reordering in TSO》
。。TSO ： TCP Segment Offload，利用 网卡的少量处理能力，降低CPU发送数据包负载 的计数，需要 网卡硬件 和 驱动的支持。
。。cpp就没有 loadload reorder，只有 happen before 等。



## full fence
full fence是release fence和acquire fence的组合，所以也就是防止loadload、loadstore、storestore重排
。反正只允许 前面的store重排到后面的load之后。

std::atomic_thread_fence(memory_order_acq_rel)和std::atomic_thread_fence(memory_order_seq_cst)都是full fence。

在c++的标准定义里，full fence并没有规定一定要阻止storeload重排，即便是std::atomic_thread_fence(memory_order_seq_cst)也一样，只是需要额外保证单独全序，但是在实际的实现上为了实现这个全序编译器大都是采用了硬件层面的能够阻止storeload重排的full barrier指令


## fence和同样memory order的原子操作同步效果的区别
基于atomic_thread_fence（外加一个任意序的原子变量操作）的同步和基于原子操作的同步很类似
比如最常用的，都可以形成release acquire语义
但是从上面的描述可以看出，fence的效果要比基于原子变量的效果更强，在weak memory order平台的开销也更大。

以release为例，对于基于原子变量的release opration，仅仅是阻止前面的内存操作重排到该release opration之后，而release fence则是阻止重排到fence之后的任意store operation之后
。。。看起来 load 真可以 重排到 这条fence语句 和 store 之间。
。。看下面的 代码，感觉没有问题。 确实 可以重排到 fence 和 store之间。
。。因为 对比的是 store，store 是 屏障+store 的混合。 保护 store。
。。那么 fence 也是 保护store，等于就是 在 后续的 store上 加上 这个 memory order。  是给 后续的 **每个 store 都加上**。 

一个简单的例子
```
std::string* p  = new std::string("Hello");
ptr.store(p, std::memory_order_release);
```

以下代码具有相同的效果
```
std::string* p  = new std::string("Hello");
std::atomic_thread_fence(memory_order_release);
ptr.store(p, std::memory_order_relaxed);
```

但是，比如
依赖 ptr1 的线程 永远 可以读到正确值，但是 依赖 ptr2 的 不一定
```
std::string* p  = new std::string("Hello");
ptr1.store(p, std::memory_order_release);
ptr2.store(p, std::memory_order_relaxed);
```

依赖ptr1 和 ptr2 的都可以 永远读到正确值
```
std::string* p  = new std::string("Hello");
std::atomic_thread_fence(memory_order_release);
ptr1.store(p, std::memory_order_relaxed);
ptr2.store(p, std::memory_order_relaxed);
```


## 利用atomic_thread_fence进行release acquire同步
fence的同步效果 和 原子操作上的同步效果比较相似，可以互相结合。
1. fence-atomic同步
2. fence-fence同步
3. atomic-fence同步


## release fence - atomic acquire 同步
线程A有一个 release fence FA，线程B有一个acquire operation Y ，如果满足以下三个条件，那么会形成release acquire语义进而形成synchronizes-with 关系，从而线程A的FA之前的所有写入都会happen-before Y之后的读，也就是对Y可见：
1. 有一个任意memory order的 atomic store X 。
2. Y读到了X写入的值 (或者如果x是release operation ，读到了release sequence headed by X 写入的值)。
3. FA sequenced-before X 。


## atomic release- acquire fence 同步
线程A有一个release operation X，线程B中有一个acquire fence FB，如果满足以下三个条件，那么会形成release acquire语义进而形成synchronizes-with 关系，从而 thread A 中所有sequenced-before X的内存写入都会happen-before 线程B中FB后的读：
1. B中有一个任意memory order的 atomic read Y。
2. Y 读到了 X或者release sequence headed by X写入的值 。
3. Y sequenced-before FB 。


## release fence - acquire fence 同步
有线程A中的 release fence FA，和线程B中的 acquire fence FB, 如果满足以下条件，那么会形成release acquire语义进而形成synchronizes-with 关系，从而线程A中所有 sequenced-before FA的写入都会 happen-before 线程B中FB之后的所有读取：
1. 有一个原子变量 M。
2. 线程A中有一个任意memory order 的对M的原子写入X。
3. FA sequenced-before X。
4. 线程B中有一个任意memory order 的对M的原子读Y读到了 X 写入的值 (或者如果x是release operation ，读到了release sequence headed by X 写入的值)。
5. Y sequenced-before FB。


## 利用atomic_thread_fence进行Sequentially-consistent 同步
。。跳


## fence同步实例
fence-fence同步
``` C++
//Global
std::string computation(int);
void print( std::string );
 
std::atomic<int> arr[3] = { -1, -1, -1 };
std::string data[1000] //non-atomic data
 
// Thread A, compute 3 values
void ThreadA( int v0, int v1, int v2 )
{
	//assert( 0 <= v0, v1, v2 < 1000 );
	data[v0] = computation(v0);
	data[v1] = computation(v1);
	data[v2] = computation(v2);
	std::atomic_thread_fence(std::memory_order_release);
	std::atomic_store_explicit(&arr[0], v0, std::memory_order_relaxed);
	std::atomic_store_explicit(&arr[1], v1, std::memory_order_relaxed);
	std::atomic_store_explicit(&arr[2], v2, std::memory_order_relaxed);
}
 
// Thread B, prints between 0 and 3 values already computed.
void ThreadB()
{
	int v0 = std::atomic_load_explicit(&arr[0], std::memory_order_relaxed);
	int v1 = std::atomic_load_explicit(&arr[1], std::memory_order_relaxed);
	int v2 = std::atomic_load_explicit(&arr[2], std::memory_order_relaxed);
	std::atomic_thread_fence(std::memory_order_acquire);
	// v0, v1, v2 might turn out to be -1, some or all of them.
	// otherwise it is safe to read the non-atomic data because of the fences:
	if( v0 != -1 ) { print( data[v0] ); }
	if( v1 != -1 ) { print( data[v1] ); }
	if( v2 != -1 ) { print( data[v2] ); }
}
```

atomic fence同步
```C++
const int num_mailboxes = 32;
std::atomic<int> mailbox_receiver[num_mailboxes];
std::string mailbox_data[num_mailboxes];
 
// The writer threads update non-atomic shared data 
// and then update mailbox_receiver[i] as follows
mailbox_data[i] = ...;
std::atomic_store_explicit(&mailbox_receiver[i], receiver_id, std::memory_order_release);
 
// Reader thread needs to check all mailbox[i], but only needs to sync with one
for (int i = 0; i < num_mailboxes; ++i) {
    if (std::atomic_load_explicit(&mailbox_receiver[i], std::memory_order_relaxed) == my_id) {
        std::atomic_thread_fence(std::memory_order_acquire); // synchronize with just one writer
        do_work( mailbox_data[i] ); // guaranteed to observe everything done in the writer thread before
                    // the atomic_store_explicit()
    }
 }
```


-------------------------
https://blog.csdn.net/wxj1992/article/details/104266983
C++ memory order在编译器和cpu层面的具体实现

以x86-64和ARMv8两个典型平台为例，这也分别是最常见的两个strong memory model和weak memory model。 

## x86-64 C++ memory_order和指令对照

|C++11 操作|对应x86-64指令|
|--|--|
|Load Relaxed	|MOV (from memory)
|Load Consume	|MOV (from memory)
|Load Acquire	|MOV (from memory)
|Load Seq_Cst	|MOV (from memory)
|Store Relaxed	|MOV (into memory)
|Store Release	|MOV (into memory)
|Store Seq Cst	|(LOCK) XCHG // alternative: MOV (into memory),MFENCE
|Consume Fence	|ignore
|Acquire Fence	|ignore
|Release Fence	|ignore
|Acq_Rel Fence	|ignore
|Seq_Cst Fence	|MFENCE

根据上表可以看到，因为x86-64本身就不会进行storestore loadstore loadload重排，除了Seq_Cst相关的操作，均不需要添加额外的cpu指令，实际上，对于x86-64平台，实现Sequential Consistency可以有四种方式
1. 纯LOAD 和STORE + MFENCE
2. 纯LOAD 和LOCK XCHG
3. MFENCE + LOAD 和纯STORE
4. LOCK XADD ( 0 ) 和纯STORE


## ARMv8（AArch64部分） C++ memory_order和指令对照
|C++ 操作|对应AArch64 指令|
|--|--|
|Load Relaxed	|LDR|
|Load Consume	|LDR + preserve dependencies until next kill_dependency OR LDAR|
|Load Acquire	|LDAR|
|Load Seq Cst	|LDAR|
|Store Relaxed	|STR |
|Store Release	|STLR|
|Store Seq Cst	|STLR|
|Cmpxchng Relaxed	|_loop: ldxr roldval, [rptr]; cmp roldval, rold; b.ne _exit; stxr rres, rnewval, [rptr]; cbnz rres, _loop; _exit |
|Cmpxchng Acquire	|_loop: ldaxr roldval, [rptr]; cmp roldval, rold; beq _exit; stxr rres, rnewval, [rptr]; cbnz rres, _loop; _exit |
|Cmpxchng Release	|_loop: ldxr roldval, [rptr]; cmp roldval, rold; b.ne _exit; stlxr rres, rnewval, [rptr]; cbnz rres, _loop; _exit|
|Cmpxchng AcqRel	|_loop: ldaxr roldval, [rptr]; cmp roldval, rold; b.ne _exit; stlxr rres, rnewval, [rptr]; cbnz rres, _loop; _exit|
|Cmpxchng SeqCst	|_loop: ldaxr roldval, [rptr]; cmp roldval, rold; b.ne _exit; stlxr rres, rnewval, [rptr]; cbnz rres, _loop; _exit|
|Acquire Fence	|DMB ISH LD|
|Release Fence	|DMB ISH|
|AcqRel Fence	|DMB ISH|
|SeqCst Fence	|DMB ISH|







==================

https://javaforall.cn/132631.html
# 无锁编程介绍

原文
https://preshing.com/20120612/an-introduction-to-lock-free-programming/


## 无锁编程是什么
如果你的代码的一部分符合如下的条件，就可以认为是 lock free。

![7c49dff20d6c587458c00c5066b5ff01.png](../_resources/7c49dff20d6c587458c00c5066b5ff01.png)

无锁的锁并不是直接指互斥量，而是指 锁定 整个应用的所有可能的方式，不论是死锁，活锁，线程调度等。

下面这个例子没有使用mutex，但是仍然 不是无锁的。 X初始值为0。考虑如何调度2个线程，才能使得2个都不退出循环。
```
while (X == 0)
{
	X = 1 - X;
}
```

|Thread 1|Thread 2|
|--|--|
|判断X==0||
||判断X==0|
|X=1-X,X变成1||
||X=1-X,X变成0|


没人期待整个大型的应用是完全无锁的。通常，我们可以从整个代码库中识别出一系列无锁操作。例如，在一个无锁队列中有少许的无锁操作，像push、pop，或许还有isEmpty等等

《多处理器编程的艺术》提出了一个简单的无锁的定义：**在一个无限的执行过程中，会不停地有调用结束**。换句话说，**程序能不断地调用这些无锁操作的同时，许多调用的也是不断地在完成**。从算法上来说，在这些操作的执行过程中，系统锁定是不可能的。

无锁编程的一个重要的特性就是，**如果挂起一个单独的线程，不会阻碍其它线程执行**。作为一组线程，他们使用特定的无锁操作来完成这个特性。这揭示了无锁编程在中断处理程序和实时系统方面的价值。因为在这些情况下，特定的操作必须在特定的时间内完成，不论程序的状态如何。
。。但是 其他线程都在CAS等待 本线程设置flag，如果本线程被挂起，其他线程就无法继续执行了。也不能说无法继续执行，而是死循环。
。。所以后面说 必须在 特定的时间内完成。
。。话说，thread timeout 我还不知道怎么实现的呢。

最后的精确描述：设计用于阻塞的操作不会影响算法运行，这种代码才是无锁的。例如在无锁队列的操作中，当队列为空时，队列的弹出操作会有意地阻塞。但其它的代码路径仍然被认为是无锁的。

## 无锁编程技术
当你试图满足无锁编程的无阻塞条件时，会出现一系列技术：==原子操作、内存屏障、避免ABA问题==，等等。

这些技术之间的关联：

![b987e15d92f1c0acb495f1a90555b243.png](../_resources/b987e15d92f1c0acb495f1a90555b243.png)

## 原子的 Read-Modify-Write 操作
原子操作是指，通过一种看起来不可分割的方式来操作内存：**线程无法看到原子操作的中间过程**。
在现代的处理器上，有很多操作本身就是的原子的。例如，对简单类型的对齐的读和写通常就是原子的。

**Read-Modify-Write**（RMW）操作更进一步，它允许你按照原子的方式，操作更复杂的事务。当一个无锁的算法必须支持多个写入者时，原子操作会尤其有用，因为多个线程试图在同一个地址上进行RMW时，它们会按“一次一个”的方式排队执行这些操作。我已经在我的博客中涉及了RMW操作了，如实现 轻量级互斥量、递归互斥量 和 轻量级日志系统。

RMW 操作的例子包括：Win32上 的 _InterlockedIncrement，iOS 上的 OSAtomicAdd32 以及 C++11 中的 `std::atomic<int>::fetch_add`。需要注意的是，C++11 的原子标准**不保证**其在每个平台上的实现都是无锁的，因此最好要清楚你的平台和工具链的能力。你可以调用 **`std::atomic<>::is_lock_free`** 来确认一下。

不同的 CPU 系列支持 RMW 的方式也是不同的。例如，PowerPC 和 ARM 提供 load-link/store-conditional 指令，这实际上是允许你实现你自定义的底层 RMW 操作。常用的 RMW 操作就已经足够了。


## Compare-And-Swap 循环
最常讨论的 RMW 操作是 compare-and-swap (CAS)。在Win32上，CAS 是通过如 _InterlockedCompareExchange 等一系列指令来提供的。通常，程序员会在一个事务中使用 CAS 循环。这个模式通常包括：复制一个共享的变量至本地变量，做一些特定的工作（改动），再试图使用 CAS 发布这些改动。
``` C++
void LockFreeQueue::push(Node* newHead)
{
    for (;;)
    {
        // Copy a shared variable (m_Head) to a local.
        Node* oldHead = m_Head;

        // Do some speculative work, not yet visible to other threads.
        newHead->next = oldHead;

        // Next, attempt to publish our changes to the shared variable.
        // If the shared variable hasn't changed, the CAS succeeds and we return.
        // Otherwise, repeat.
        if (_InterlockedCompareExchange(&m_Head, newHead, oldHead) == oldHead)
            return;
    }
}
```
这样的循环仍然有资格作为无锁的，因为如果一个线程检测失败，意味着有其它线程成功。尽管某些架构提供一个 较弱的CAS变种 。无论何时实现一个CAS循环，都必须十分小心地避免 ABA 问题。


## 顺序一致性
顺序一致性（Sequential consistency）意味着，所有线程就内存操作的顺序达成一致。这个顺序是和操作在**程序源代码中的顺序是一致**的。在顺序一致性的要求下，像我 另一篇文章 演示的那样的有意的**内存重排序不再可能**了。

一种实现顺序一致性的简单（但显然不切实际）的方法是禁用编译器优化，并强制所有线程在单个处理器上运行。 即使线程在任意时间被抢占和调度，处理器也永远不会看到自己的内存影响。

某些编程语言甚至可以为在多处理器环境中运行的优化代码提供顺序一致性。 在C ++ 11中，可以将所有共享变量声明为具有默认内存排序约束的C ++ 11原子类型。 在Java中，您可以将所有共享变量标记为volatile。下面是 我之前一个案例 用C++11风格重写的例子。
。。java volatile无法提供原子性啊，前面说 C++ atomic，后面说 java volatile。这2个应该并不等价。 。 不，但是 java volatile可以提供 顺序一致性。应该吧，我也不清楚。
```C++
std::atomic X(0), Y(0);
int r1, r2;
 
void thread1()
{ 
   
    X.store(1);
    r1 = Y.load();
}
 
void thread2()
{ 
   
    Y.store(1);
    r2 = X.load();
}
```
因为 C ++ 11 原子类型保证顺序一致性，所以结果 r1 = r2 = 0 是不可能的。 为此，编译器会在后台输出其他指令——通常是 内存围栏 和/或 RMW 操作。 与程序员直接处理内存排序的指令相比，那些额外的指令可能会使实现效率降低。

## Memory Ordering
如果你的环境不能保证顺序一致性，你都必须考虑如何来**防止 内存重新排序**。

在当今的架构中，增强内存保序性的工具通常分为三类，它们既防止 **编译器重新排序** 又防止 **处理器重新排序**：
1.    一个轻型的同步或屏障指令，以后的文章会详述；
2.    一个完全的内存屏障指令，如之前所述；
3.    提供获取或释放语义的内存操作。
。。
a lightweight sync or fence instruction
a full memory fence instruction
memory operation which provide acquire or release semantics
。。
acquire防止后续的操作 重排序到它前面
release防止前面的操作 重排序到它后面
特别适合于 生产者，消费者。


## Different Processors Have Different Memory Models
。。跳
As a result, it’s been common in the past to write lock-free code which works on x86/64, but fails on other processors.




==================
https://learn.microsoft.com/zh-cn/windows/win32/dxtecharts/lockless-programming
这篇文章是上面的preshing的无锁编程介绍中提到了。

# 《Xbox 360 和 Microsof Windows 的无锁编程注意事项》

无锁编程是一种安全地共享多个线程之间更改数据的方法，无需获取和释放锁的成本
无锁编程是复杂而微妙
无锁编程是多线程编程的有效技术，但不应轻用。 在使用它之前，必须了解复杂性，并且应该仔细衡量，以确保它实际上是给你预期的收益。 

## 使用锁编程
编写多线程代码时，通常需要在线程之间共享数据。 如果多个线程同时读取和写入共享数据结构，则可能会出现内存损坏。 解决此问题的最简单方法是使用锁。 例如，如果一次只能由一个线程执行 ManipulateSharedData，则可以使用CRITICAL_SECTION来保证这一点，如以下代码所示：

```
// Initialize
CRITICAL_SECTION cs;
InitializeCriticalSection(&cs);

// Use
void ManipulateSharedData()
{
    EnterCriticalSection(&cs);
    // Manipulate stuff...
    LeaveCriticalSection(&cs);
}

// Destroy
DeleteCriticalSection(&cs);
```

使用锁编程具有几个潜在的缺点
- 如果两个线程尝试获取相同的两个锁，但以不同的顺序获取它们，你可能会获得死锁
- 如果程序持有锁定时间过长（由于设计不佳或线程已由优先级较高的线程交换），则其他线程可能会长时间被阻止
- 如果延迟的过程调用或中断服务例程尝试获取锁，可能会获得死锁。


尽管存在这些问题，但同步基元（如关键部分）**通常**是协调多个线程的**最佳方法**。 如果同步基元太慢，最佳解决方案通常是**使用它们的频率降低**。 但是，对于那些能够承受额外复杂性的人来说，另一个选项是无锁编程。


## 无锁编程
无锁编程（顾名思义）是一系列技术，用于在不使用锁的情况下安全地操作共享数据。 有**一些无锁算法**可用于**传递消息、共享列表和数据队列和其他任务**。

执行无锁编程时，必须处理两个难题：==非原子操作和重新排序==。

## 非原子操作
原子操作对于无锁编程非常重要，因为如果没有它们，其他线程可能会看到半写的值，否则状态不一致。

在所有新式处理器上，可以假定自然对齐的本机类型的读取和写入是原子的。 只要内存总线的宽度至少与读取或写入的类型一样宽，CPU 会在单个总线事务中读取和写入这些类型，从而使其他线程无法以半完成状态查看它们。 
在 x86 和 x64 上，不能保证读取和写入大于 8 个字节是原子的。 这意味着流式 SIMD 扩展的 16 字节读取和写入 (SSE) 寄存器和字符串操作可能不是原子操作。

不自然对齐的类型读取和写入（例如，写入跨四字节边界的 DWORD）不能保证是原子的。 CPU 可能必须以多个总线事务的形式执行这些读取和写入操作，从而允许另一个线程在读取或写入中间修改或查看数据。

复合操作（如递增共享变量时发生的读-修改-写序列）不是原子操作。
在 x86 和 x64 上，有一个指令 (inc) 可用于递增内存中的变量。 如果使用此指令，递增变量在单处理器系统上是原子的，但在多处理器系统上它仍然不是原子的。
```
// This write is not atomic because it is not natively aligned.
DWORD* pData = (DWORD*)(pChar + 1);
*pData = 0;

// This is not atomic because it is three separate operations.
++g_globalCounter;

// This write is atomic.
g_alignedGlobal = 0;

// This read is atomic.
DWORD local = g_alignedGlobal;
```


## 保证原子性
可以通过以下组合来==确保==使用原子操作：
- 自然原子操作
- 用于包装复合操作的锁
- 实现常用复合操作原子版本的操作系统函数


## 重组
更微妙的问题是重新排序。 读取和写入并不总是按照你在代码中编写它们的顺序发生，这可能会导致非常令人困惑的问题。 
在许多多线程算法中，线程会写入一些数据，然后写入一个标志，告知其他线程数据已准备就绪。 这称为写入发布。 如果重新排序写入，则其他线程可能会在看到写入数据之前设置标志。
同样，在许多情况下，线程从标志读取，然后读取一些共享数据（如果标志表示线程已获取对共享数据的访问权限）。 这称为读取获取。 如果重新排序读取，则可以在标志前从共享存储读取数据，并且看到的值可能不是最新的。

## 了解写入的 CPU 重新排列

### Xbox 360
虽然Xbox 360 CPU 没有重新排序指令，但它会重新排列写入操作，这些操作在指令本身之后完成。 PowerPC内存模型专门允许重新排列写入。
Xbox 360上的写入操作**不会直接转到 L2 缓存**。 相反，**为了改进 L2 缓存写入带宽**，它们通过存储队列，然后存储收集缓冲区。 存储收集缓冲区允许在一个操作中将 64 字节块写入 L2 缓存。 有八个**存储收集缓冲区**，允许高效写入到多个不同内存区域。
存储收集缓冲区通常以先出先出 (FIFO) 顺序写入 L2 缓存。 但是，如果写入的目标缓存行不在 L2 缓存中，则在从内存提取缓存行时，该写入可能会延迟。
即使以严格的 FIFO 顺序将存储收集缓冲区写入 L2 缓存，这不能保证将单个写入按顺序写入 L2 缓存。 例如，假设 CPU 写入位置0x1000，然后写入位置0x2000，然后写入位置0x1004。 第一个写入分配存储收集缓冲区，并将其置于队列的前面。 第二个写入分配另一个存储收集缓冲区，并将其放在队列中。 第三次写入会将数据添加到第一个存储收集缓冲区，该缓冲区保留在队列的前面。 因此，第三个写入最终将转到第二个写入前的 L2 缓存。
存储收集缓冲区引起的重新排序是根本不可预知的，特别是因为核心上的两个线程共享存储收集缓冲区，使存储收集缓冲区的分配和清空高度可变。
。。编译器重排序，CPU重排序，存储收集缓冲区重排序。。
。。不过只有Xbox 有这个 存储收集缓存区吧？

### x86 和 x64
即使 x86 和 x64 CPU 执行重新排序指令，但它们通常不会对其他写入操作重新排序。 写入组合内存存在一些异常。 此外， (MOVS 和 STOS) 和 16 字节 SSE 写入的字符串操作可以在内部重新排序，但否则，写入不会相互重新排序。

## 了解读取的 CPU 重新排列

### Xbox 360
**缓存未命中**可能会导致某些读取延迟，这实际上会导致读取按顺序来自共享内存，并且这些缓存未命中时间从根本上不可预知。 预提取和分支预测还可能导致数据按顺序来自共享内存。 这些只是有关如何重新排序读取的几个示例。 可能存在其他可能性。 PowerPC内存模型专门允许重新排列读取。

### x86 和 x64
即使 x86 和 x64 CPU 执行重新排序指令，但它们通常不会相对于其他读取重新排序读取操作。 字符串操作 (MOVS 和 STOS) 和 16 字节 SSE 读取可以内部重新排序，但否则，读取不会相互重新排序。


## 其他重新排序
即使 x86 和 x64 CPU 不会相对于其他写入重新排序写入，或者对相对于其他读取重新排序读取，但它们也可以对相对于写入的读取重新排序。 

。。这句就是，X86,X64 没有 loadload reorder, storestore reorder,但是有 loadstore,storeload reorder 。。 不太清楚 相对于写入的读取 是 loadstore 还是 storeload。  应该是 代码先 store 然后load， 这种会被重排序。   那 load-store 呢？。 
。。下面说了，是 store 然后 load。但是 是 其他位置load，是指 其他线程？还是 读取了 cache？ 重排序 不是和多线程无关么，只是 单线程中的 效果啊。 对，后续的"数据来自共享内存"，读取的内存位置 不对。但是，单线程，怎么可能读取的位置不对呢？
。。后续的 Dekker，应该是， 。。不知道
。。Dekker互斥算法是由荷兰数学家Dekker提出的一种解决并发进程互斥与同步的软件实现方法。

具体而言，如果程序写入到一个位置，然后从其他位置读取，则读取数据可能来自共享内存，然后写入数据就在那里。 

这种重新排序可能会中断某些算法，例如 ==Dekker== 的相互排除算法。 在 Dekker 算法中，每个线程设置一个标志以指示它想要进入关键区域，然后检查另一个线程的标志，以查看另一个线程是否位于关键区域或尝试输入它。 初始代码遵循。

。。进入天书环节。

``` C++
volatile bool f0 = false;
volatile bool f1 = false;

void P0Acquire()
{
    // Indicate intention to enter critical region
    f0 = true;
    // Check for other thread in or entering critical region
    while (f1)
    {
        // Handle contention.
    }
    // critical region
    ...
}


void P1Acquire()
{
    // Indicate intention to enter critical region
    f1 = true;
    // Check for other thread in or entering critical region
    while (f0)
    {
        // Handle contention.
    }
    // critical region
    ...
}
```

问题是，P0Acquire 中 f1 的读取可以在写入 f0 之前从共享存储读取到共享存储。 同时，在写入 f1 之前，P1Acquire 中的 f0 读取可以从共享存储读取，然后写入 f1 即可将其写入共享存储。 净效果是，两个线程将标志设置为 TRUE，两个线程将另一个线程的标志视为 FALSE，因此它们都进入关键区域。 因此，虽然在基于 x86 和 x64 的系统上重新排序的问题与Xbox 360相比不太常见，但它们肯定仍可能发生。 Dekker 的算法在上述任何平台上都不会使用硬件内存屏障。

x86 和 x64 CPU 不会在上一次读取之前重新排序写入。 如果 x86 和 x64 CPU 面向不同的位置，则仅重新排序之前写入。

PowerPC CPU 可以在写入之前重新排序读取，并且可以在读取之前对写入重新排序，只要它们与不同的地址相同。


## 重新排序摘要
与 x86 和 x64 CPU 相比，Xbox 360 CPU 重新排序比 x86 和 x64 CPU 更积极，如下表所示。 有关更多详细信息，请参阅处理器文档。
|重新排序活动 	|x86 和 x64 	|Xbox 360|
|--|--|--|
|读取操作在读取之前移动 	|否 	|是|
|写入操作在写入之前移动 	|否 	|是|
|写入操作在读取之前移动 	|否 	|是|
|读取操作在写入之前移动 	|是 	|是|


## Read-Acquire和Write-Release障碍
用于防止重新排序读取和写入的主要构造称为读取获取和写入释放屏障。 
读取获取是一个标志或其他变量的读取，用于获取资源的所有权，加上阻止重新排序的障碍。 
同样，写发布是标记或其他变量的写入，用于放弃资源的所有权，加上阻止重新排序的障碍。

赫布·萨特的正式定义包括：
- 读取获取在按程序顺序遵循该线程的所有读取和写入之前执行。
- 写入释放在按程序顺序排列之前的所有读取和写入操作之后执行。

。。赫布·萨特 是C++专家
。读取 在 后续所有 读/写 之前执行
。写入 在 前面所有 读/写 之后执行


最简单的方式是将读取获取和写入释放屏障视为单个操作。 但是，它们有时必须从两个部分构造：读取或写入以及不允许读取或写入的屏障在读取或写入之间移动。 在这种情况下，屏障的位置至关重要。 
对于读取获取屏障，**先读取标志，再读取屏障，然后读取和写入共享数据**。 
对于写入释放屏障，**共享数据的读取和写入先是屏障，然后是标志的写入。**

``` C++
// Read that acquires the data.
if( g_flag )
{
    // Guarantee that the read of the flag executes before
    // all reads and writes that follow in program order.
    BarrierOfSomeSort();

    // Now we can read and write the shared data.
    int localVariable = sharedData.y;
    sharedData.x = 0;

    // Guarantee that the write to the flag executes after all
    // reads and writes that precede it in program order.
    BarrierOfSomeSort();
    
    // Write that releases the data.
    g_flag = false;
}
```

读取获取和写入释放之间的唯一区别是内存屏障的位置。 
读取获取在**锁定操作后具有屏障**，
写入**释放之前具有屏障**。 
在这两种情况下，屏障位于对锁定内存的引用和对锁的引用之间。

若要了解获取数据时和发布数据时为何都需要屏障，最好 (和最准确的) 将这些屏障==视为保证与共享内存的同步==，而不是与其他处理器同步。 如果一个处理器使用写版本将数据结构发布到共享内存，另一个处理器使用读取获取从共享内存访问该数据结构，则代码将正常工作。 如果任一处理器未使用适当的屏障，则数据共享可能会失败。

使用正确的屏障防止平台的编译器和 CPU 重新排序至关重要。

使用操作系统提供的同步基元的优点之一是，所有这些基元都包含适当的内存屏障。


## 阻止编译器重新排序

使用 Visual C++，可以使用编译器内部 `_ReadWriteBarrier`来阻止编译器重新排序。 在代码中插入 `_ReadWriteBarrier` 的位置，编译器不会移动读取和写入。

``` C++
#if _MSC_VER < 1400
    // With VC++ 2003 you need to declare _ReadWriteBarrier
    extern "C" void _ReadWriteBarrier();
#else
    // With VC++ 2005 you can get the declaration from intrin.h
#include <intrin.h>
#endif
// Tell the compiler that this is an intrinsic, not a function.
#pragma intrinsic(_ReadWriteBarrier)

// Create a new sprite by filling in a previously empty entry.
g_sprites[nextSprite].x = x;
g_sprites[nextSprite].y = y;
// Write-release, barrier followed by write.
// Guarantee that the compiler leaves the write to the flag
// after all reads and writes that precede it in program order.
_ReadWriteBarrier();
g_sprites[nextSprite].alive = true;
```

在以下代码中，另一个线程从子画面数组读取：

``` C++
// Draw all sprites.
for( int i = 0; i < numSprites; ++i )
{

    // Read-acquire, read followed by barrier.
    if( g_sprites[nextSprite].alive )
    {
    
        // Guarantee that the compiler leaves the read of the flag
        // before all reads and writes that follow in program order.
        _ReadWriteBarrier();
        DrawSprite( g_sprites[nextSprite].x,
                g_sprites[nextSprite].y );
    }
}
```

请务必了解 ，`_ReadWriteBarrier` 不会插入任何其他指令，并且它不会阻止 CPU 重新排列读取和写入-它只阻止编译器重新排列读取和写入。
因此，当你在 x86 和 x64 (上实现写释放屏障时， `_ReadWriteBarrier` 就足够了，因为 x86 和 x64 **不重新排序写入，并且正常写入足以释放锁**) ，但在大多数情况下，还需要防止 CPU 重新排序读取和写入。

还可以在写入不可缓存的写入组合内存时使用 `_ReadWriteBarrier` ，以防止重新排序写入。 在这种情况下 ，`_ReadWriteBarrier` 通过保证写入以处理器的首选线性顺序进行，从而帮助**提高性能**。

还可以使用 `_ReadBarrier 和 _WriteBarrier` 内部函数更精确地控制编译器重新排序。 编译器不会在 `_ReadBarrier`之间移动读取，也不会在 `_WriteBarrier`之间移动写入。

## 防止 CPU 重新排序
CPU 重新排序比编译器重新排序更微妙。 你永远看不到它直接发生，你只会看到难以理解的 bug。 
为了防止对读取和写入进行 CPU 重新排序，需要在某些处理器上使用==内存屏障指令==。 内存屏障指令（Xbox 360和Windows）的所有用途名称为 MemoryBarrier。 此宏是针对每个平台正确实现的。

在Xbox 360，**MemoryBarrier 定义为 lwsync (轻型同步)** ，也可以通过 ppcintrinsics.h 中定义的`__lwsync`内部函数获得。 `__lwsync` 还充当编译器内存屏障，防止编译器重新排列读取和写入。

lwsync 指令是Xbox 360上的内存屏障，用于将**一个处理器核心与 L2 缓存同步**。 
它保证 lwsync 之前的所有写入都会在后续写入之前将其写入 L2 缓存。 
它还保证以下 lwsync 的任何读取不会从 L2 获取早于以前读取的数据。 
它不会阻止的一种类型的重新排序是在写入到其他地址之前进行读取。 
因此， lwsync 强制实施与 x86 和 x64 处理器上默认内存排序匹配的内存排序。 
若要获取完整内存排序，需要更昂贵的同步指令 (也称为重量级同步) ，但在大多数情况下，这不是必需的。 

下表显示了Xbox 360上的内存重新排序选项。

|Xbox 360重新排序 	|无同步 	|lwsync 	|sync|
|--|--|--|--|
|读取操作在读取之前移动 	|是 	|否 	|否|
|写入操作在写入之前移动 	|是 	|否 	|否|
|写入操作在读取之前移动 	|是 	|否 	|否|
|读取操作在写入之前移动 	|是 	|是 	|否|

PowerPC还有同步指令 isync 和 eieio (用于控制缓存抑制内存) 的重新排序。 出于正常同步目的，不需要这些同步说明。

在**Windows**中，==MemoryBarrier== 在 `Winnt.h` 中定义，并提供不同的内存屏障指令，具体取决于是要编译 x86 还是 x64。 
内存屏障指令充当完整屏障，防止对屏障上的读取和写入进行重新排序。 
因此，Windows上的 MemoryBarrier 提供比Xbox 360**更强的重新排序保证**。

在Xbox 360和其他许多 CPU 上，可以阻止 CPU 进行读取重新排序的**另一种方法**。 
==如果读取指针，然后使用该指针加载其他数据，则 CPU 保证指针的读取时间不早于指针的读取。== 
如果**锁标志是指针**，并且**共享数据的所有读取都脱离了指针**，则可以省略 MemoryBarrier ，以节省适度的性能。

``` C++
Data* localPointer = g_sharedPointer;
if( localPointer )
{
    // No import barrier is needed--all reads off of localPointer
    // are guaranteed to not be reordered past the read of
    // localPointer.
    int localVariable = localPointer->y;
    // A memory barrier is needed to stop the read of g_global
    // from being speculatively moved ahead of the read of
    // g_sharedPointer.
    int localVariable2 = g_global;
}
```

MemoryBarrier 指令仅阻止对可缓存内存的读取和写入重新排序。 如果将==内存分配为PAGE_NOCACHE或PAGE_WRITECOMBINE==，则对于设备驱动程序作者和Xbox 360上的游戏开发人员来说，MemoryBarrier 对访问此内存没有任何影响。 大多数开发人员不需要同步不可缓存的内存。 这超出了本文的范围。
。。普通的C++没有 PAGE_NOCACHE或PAGE_WRITECOMBINE 吧。


## 互锁函数和 CPU 重新排序

有时，获取或释放资源的读取或写入是使用 InterlockedXxx 函数之一完成的。 在Windows，这简化了操作;因为在Windows，InterlockedXxx 函数都是**全内存屏障**。 它们实际上在 CPU 内存屏障之前和之后都有一个 CPU 内存屏障，这意味着它们是完全读取获取或写入释放屏障本身。

在Xbox 360上，InterlockedXxx 函数不包含 CPU 内存屏障。 它们可防止编译器重新排序读取和写入，但不会对 CPU 重新排序。 因此，在大多数情况下，在Xbox 360上使用 InterlockedXxx 函数时，应先使用__lwsync遵循它们，使其成为读取获取或写入释放屏障。 为方便起见，并且更易于阅读，有许多 InterlockedXxx 函数的 Acquire 和 Release 版本。 它们附带内置内存屏障。 例如， InterlockedIncrementAcquire 执行互锁增量，后跟 `__lwsync` 内存屏障，以提供完整的读取获取功能。

此示例演示了一个线程如何使用 InterlockedXxxSList 函数的 Acquire 和 Release 版本将任务或其他数据传递给另一个线程。 InterlockedXxxSList 函数是一系列函数，用于维护不带锁的共享单声链接列表。 请注意，这些函数的获取和发布变体在Windows不可用，但这些函数的常规版本是Windows的完整内存屏障。

``` C++
// Declarations for the Task class go here.

// Add a new task to the list using lockless programming.
void AddTask( DWORD ID, DWORD data )
{
    Task* newItem = new Task( ID, data );
    InterlockedPushEntrySListRelease( g_taskList, newItem );
}

// Remove a task from the list, using lockless programming.
// This will return NULL if there are no items in the list.
Task* GetTask()
{
    Task* result = (Task*)
        InterlockedPopEntrySListAcquire( g_taskList );
    return result;
}
```


## 可变变量和重新排序

==C++ 标准版表示，无法缓存易失变量的读取，易失性写入无法延迟，并且无法相互移动易失读取和写入。 这足以与硬件设备通信，这是 C++ 标准中的易失性关键字的用途。==

但是，==标准保证不足以对多线程使用易失性==。 C++ 标准不会阻止编译器重新排序相对于易失性读取和写入的非易失性读取和写入，并且它不说明阻止 CPU 重新排序。

。。volatile 是 立即读取，写入的，  但是 重排序 会把 非volatile 重排到 volatile 前面 或 后面。

Visual C++ 2005 超越标准 C++ ，以定义多线程友好语义，以便进行可变变量访问。 从 Visual C++ 2005 开始，从易失变量读取定义为具有读取获取语义，对易失变量的写入定义为具有写发布语义。 这意味着编译器不会重新排列任何读取和写入，并且Windows它将确保 CPU 不会这样做。
。。挂不得移植性很垃。 各个厂商都是各自的。

## Lock-Free数据管道的示例

管道是一个构造，允许一个或多个线程写入数据，然后由其他线程读取。 ==管道的无锁版本可以是将工作从线程传递到线程的优雅高效方法==。 DirectX SDK 提供 LockFreePipe，这是 DXUTLockFreePipe.h 中提供的单读取器单编写器无锁管道。 AtgLockFreePipe.h 中的 Xbox 360 SDK 中提供了相同的 LockFreePipe。

当两个线程具有生成者/使用者关系时，可以使用 LockFreePipe。
如果管道填满，写入会失败
如果管道清空，读取失败
如果两个线程==平衡良好==，并且管道足够大，则管道允许它们顺利传递数据，且不会延迟或块。

## Xbox 360性能

Xbox 360上的同步指令和函数的性能因运行其他代码而异。 
如果另一个线程当前拥有锁，获取锁需要更长的时间。
如果其他线程写入同一缓存行，则 InterlockedIncrement 和关键节操作需要更长的时间。 
存储队列的内容也会影响性能。 
因此，所有这些数字只是从非常==简单的测试生成的近似值==：
- lwsync 被测量为采用 33-48 个周期。
- InterlockedIncrement 测量为采用 225-260 个周期。
- 获取或释放关键部分被测量为大约 345 个周期。
- 获取或释放互斥体被测量为大约 2350 个周期。


## Windows 性能
Windows上的同步指令和函数的性能因处理器类型和配置以及运行其他代码而异。 
多核和多套接字系统通常需要更长的时间来执行同步指令，如果另一个线程当前拥有锁，获取锁需要更长的时间。
但是，即使是从非常简单的测试生成的一些度量值也很有用：
-    MemoryBarrier 被测量为采用 20-90 个周期。
-    InterlockedIncrement 测量为采用 36-90 周期。
-    获取或释放关键部分被测量为采用 40-100 个周期。
-    获取或释放互斥体测量为大约 750-2500 个周期。

这些测试是在一系列不同处理器上的 Windows XP 上完成的。

。。
1 应该是 内存屏障
2 应该是 原子加
3 。。。关键部分，原文： critical section， 临界区
4 mutex

CRITICAL_SECTION不是针对于资源的，而是针对于不同线程间的代码段的！我们能够用它来进行所谓资源的“锁定”，其实是因为我们在任何访问共享资源的地方都加入了EnterCriticalSection和 LeaveCriticalSection语句，使得同一时间只能够有一个线程的代码段访问到该共享资源而已
。。


## 性能思考
获取或释放==关键部分==包括**内存屏障**、 **InterlockedXxx** 操作和**一些额外的检查来处理递归**，并在**必要时回退到互斥体**。 你应该谨慎地实现自己的关键部分，因为旋转在循环中等待锁释放，而不回退到互斥体，可能会浪费相当大的性能。 对于大量竞争但未保留很长时间的关键部分，应考虑使用 InitializeCriticalSectionAndSpinCount ，以便操作系统会在等待关键部分可用时旋转一段时间，而不是在尝试获取关键部分时立即延迟到互斥体。 若要确定可从旋转计数中获益的关键部分，必须测量典型等待特定锁的长度。

如果**共享堆用于内存分配（默认行为）**，则每个内存分配和可用都涉及获取锁。 随着线程数和分配数的增加，性能级别会降低，最终开始减少。 ==使用线程堆或减少分配数==可以避免这种锁定瓶颈。
如果一个线程正在生成数据，另一个线程正在使用数据，则它们最终可能会频繁共享数据。 如果一个线程正在加载资源，另一个线程正在呈现场景，则可能会出现这种情况。 如果呈现线程引用每个绘图调用上的**共享数据**，则锁定开销会很高。 如果每个线程都有**专用数据结构，则性能要好得多**，然后每个帧或更短的时间同步一次。

不保证无锁算法比使用锁的算法更快。 在尝试避免锁之前，应检查锁是否确实导致问题，并且应该衡量无锁代码是否确实提高了性能。

## 平台差异摘要
-    InterlockedXxx 函数可防止在Windows上重新排序 CPU 读/写，但不能在Xbox 360上重新排序。
-    使用 Visual Studio C++ 2005 读取和写入易失变量可防止对Windows进行 CPU 读/写重新排序，但在Xbox 360，它只会阻止编译器读/写重新排序。
-    写入在Xbox 360上重新排序，但不在 x86 或 x64 上重新排序。
-    读取在Xbox 360上重新排序，但在 x86 或 x64 上，它们仅相对于写入重新排序，并且仅当读取和写入面向不同位置时。

## ==建议==

-    尽可能使用锁，因为它们更易于正确使用。
-    避免锁定过于频繁，因此锁定成本不会变得显著。
-    避免长时间持有锁，以避免长时间停止。
-    适当时使用无锁编程，但请确保增益证明复杂性是正当的。
-    在禁止其他锁的情况下使用无锁编程或旋转锁，例如，在延迟过程调用和正常代码之间共享数据时。
-    仅使用已证明正确的==标准无锁编程算法==
-    执行无锁编程时，请务必根据需要使用可变标志变量和内存屏障指令。
-    在 Xbox 360 上使用 InterlockedXxx 时，请使用“获取”和“发布”变体。









==================
https://mirrors.edge.kernel.org/pub/linux/kernel/people/paulmck/perfbook/perfbook.2011.01.02a.pdf

。。纯英文，500页。
。这个也是在 《an-introduction-to-lock-free-programming》 中提到的。不过只提到了 这个pdf的 附录C，用来了解 硬件如何处理内存重排序 的细节。


==================
https://zhuanlan.zhihu.com/p/608637620
# 浅谈：C++ 11 无锁并发——风险指针

## 风险指针的形象比喻
风险指针用来辅助无锁栈的内存回收，它可以做到线程安全地释放一个已经pop的节点。

```C++
/*以下代码摘自《C++并发编程实战》*/
template<typename T>
class lock_free_stack
{
private:
struct node
{
  T data;
  node* next;
  node(T const&data_):data(data_){}
};
std::atomic<node*>head;
public:
  void push(T const& data)
 {
   node* const new_node = new node(data)
   new_node->next=head.load();
   while(!head.compare_exchange_weak(new_node->next,new_node));
 }
/*我们的关注点放在下面的pop方法*/
  void pop(T& result)
 {
   node* old_head=head.load();  //operate 1
   while(!head.compare_exchange_weak(old_head,old_head->next)); //operate 2
   result=old_head->data;
 }
};
```

上面的push方法中，使用 new来申请堆内存，但没有在pop 方法中使用delete 来释放，存在内存泄露。
如果在 pop中delete，单线程没有问题。
多线程时，假设 多根线程 持有栈顶指针，当其中的一根线程执行delete后，其他线程在 operate 2 这行代码时，old_head已经被delete，会出现未定义行为。

风险指针
多根线程持有栈顶，其中一根线程想要进行delete时，会询问 其他线程，风险指针 会反对 delete。
使用一个风险指针数组 用于记录，如果这个风险指针数组中有多个 当前栈顶指针的记录，就可以确定，回收操作 是存在风险的，那我们就延后进行delete操作。

## 风险指针的具体实现和工作原理

。。跳
。。就是 新建一个数组，每次 delete 的时候就
。。不，挺复杂的，2个数组，一个保存<thread id, node pointer>，一个保存 等待删除的node。
。每次pop的时候，遍历数组，看下 是否有 线程持有 结点，没有就直接delete，有的话，就进入 等待删除的数组中，并且在pop 的最后 都会调用 方法来 删除 等待删除的数组中的 结点，删除的时候，也要遍历 数组，查看是否有线程 持有该结点。

。。很复杂的。

------------------------
=============
# Lock Free Algo

------------------
https://blog.csdn.net/lijinbinlaa/article/details/122150837
# 无锁编程技术（一）
。。没有后续了。

## 锁为什么低效
首先，上锁和释放锁，都需要**模式切换**，本身就是开销。上锁：由用户空间切换到内核空间，上锁后再切回到用户空间，这就是两次切换；释放锁也同样需要两次模式切换，一共是四次模式切换。

另一个开销，在于**进程上下文的切换**。==线程切换有两种场景==：**CPU时间片用尽**、**因阻塞被提前切换**。竞争资源失败导致阻塞，会使得调度单元放弃未使用完的CPU时间片而切换到其他调度单元。也就是说，因为锁的缘故，导致上下文的切换更多了。无锁并发和CAS并不是说避免了上下文切换，而是减少了上下文切换。即，无锁并发则只有时间片用完后才进行线程切换。

上下文切换需要保存、回复现场，这些都是CPU开销。

## lock free
无锁编程具体使用和考虑到的 技术包括：
- 原子操作
- 内存栅栏
- 内存顺序
- 指令序列一致性
- 顺ABA现象
- read modify write

最常用的原子操作是 compare and swap， 几乎所有 CPU 指令集 都支持 CAS 的原子操作。

## lock free 缺点
并不能在所有场景下都带来极大提升。
循环CAS 会占用大量CPU。所以 基于 CAS 的 各种自旋锁 不适合做 操作和等待时间太长的 并发操作。 而 通过 对 **锁** 程序 进行合理的设计和优化，很多场景下 更容易 使得 程序 实现高度并发。
无锁编程 开发和维护的 成本很高。 
类似ABA 问题 也很难 直观的探测和解决
实现细节 与 硬件平台 相关性很强。目前 理论和实践上 只有 少数 数据结构 可以支持 无锁编程，比如 队列，栈，链表，词典等。 要在 业务场景下 大规模 无锁编程 较困难， 更多 是在 部分特殊场景解决 锁竞争 等问题。


---------------------

https://zhuanlan.zhihu.com/p/444193849

# 无锁编程

## 活锁，死锁
活锁、死锁本质上是一样的，原因是在获取临界区资源时，并发多个进程/线程声明资源占用(加锁)的顺序不一致，
死锁是加不上就死等，
活锁是加不上就放开已获得的资源重试

## 饥饿
饥饿是指如果线程T1占用了资源R，线程T2又请求封锁R，于是T2等待。T3也请求资源R，当T1释放了R上的封锁后，**系统首先批准了T3的请求**，T2仍然等待。然后T4又请求封锁R，当T3释放了R上的封锁之后，系统又批准了T4的请求......，**T2可能永远等待**。

## 优先级反转
优先级反转是指一个低优先级的任务持有一个被高优先级任务所需要的共享资源。高优先任务由于因资源缺乏而处于受阻状态，一直等到低优先级任务释放资源为止。而低优先级获得的CPU时间少，如果此时有优先级处于两者之间的任务，并且不需要那个共享资源，则该中优先级的任务反而超过这两个任务而获得CPU时间。如果高优先级等待资源时不是阻塞等待，而是忙循环，则可能永远无法获得资源，因为此时低优先级进程无法与高优先级进程争夺CPU时间，从而无法执行，进而无法释放资源，造成的后果就是高优先级任务无法获得资源而继续推进。

### 解决方案
- 设置优先级上限，给临界区一个高优先级，进入临界区的进程都将获得这个高优先级，如果其他试图进入临界区的进程的优先级都低于这个高优先级，那么优先级反转就不会发生。
- 优先级继承，当一个高优先级进程等待一个低优先级进程持有的资源时，低优先级进程将暂时获得高优先级进程的优先级别，在释放共享资源后，低优先级进程回到原来的优先级别。嵌入式系统VxWorks就是采用这种策略。
- 第三种方法就是使用中断禁止，通过禁止中断来保护临界区，采用此种策略的系统只有两种优先级：可抢占优先级和中断禁止优先级。前者为一般进程运行时的优先级，后者为运行于临界区的优先级。

## 护航现象（Lock Convoys）
Lock Convoys是在多线程并发环境下由于锁的使用而引起的性能退化问题。 当多个相同优先级的线程频繁地争抢同一个锁时可能会引起lock convoys问题，一般而言，lock convoys并不会像deadlock或livelock那样造成应用逻辑停止不前，相反地，遭受lock convoys的系统或应用程序仍然往前运行，但是，由于线程们频繁地争抢锁而导致过多的线程环境切换，==从而使得系统的运行效率大为降低==，而且，若存在同等优先级下不参与锁争抢的线程，则它们可以获得相对较多的处理器资源，从而造成系统调度的不公平性。


## 自旋锁
自旋锁是一种非阻塞锁，也就是说，如果某线程需要获取自旋锁，但该锁已经被其他线程占用时，该==线程不会被挂起==，而是在不断的消耗CPU的时间，不停的试图获取自旋锁。
互斥量是阻塞锁，当某线程无法获取互斥量时，该==线程会被直接挂起==，该线程不再消耗CPU时间，当其他线程释放互斥量后，操作系统会激活那个被挂起的线程，让其投入运行。

两种锁适用于不同场景：
如果是多核处理器，如果预计线程**等待锁的时间很短**，短到比线程两次上下文切换时间要少的情况下，使用==自旋锁是划算==的。
如果是多核处理器，如果预计线程等待锁的时间较长，至少比两次线程上下文切换的时间要长，建议==使用互斥量==。
如果是单核处理器，一般建议**不要使用自旋锁**。因为，在同一时间只有一个线程是处在运行状态，那如果运行线程发现无法获取锁，只能等待解锁，但因为**自身不挂起**，所以那个获取到锁的线程没有办法进入运行状态，==只能等到运行线程把操作系统分给它的时间片用完==，才能**有机会**被调度。这种情况下使用自旋锁的代价很高。
如果加锁的代码经常被调用，但竞争情况很少发生时，应该优先考虑使用自旋锁，自旋锁的开销比较小，互斥量的开销较大。


## pthread与tbb中各种锁的对比测试
pthread 是 linux 下多线程API，头文件 pthread.h。。。C的
std::thread 是C++标准库中的线程库，可以跨平台
tbb 是 TBB(Intel® Threading Building Blocks ) 。。

pthread中提供的锁有：
### pthread_mutex_t, pthread_spinlock_t, pthread_rwlock_t。

- pthread_mutex_t :
是互斥锁，同一瞬间只能有一个线程能够获取锁，其他线程在等待获取锁的时候会进入休眠状态。因此pthread_mutex_t消耗的CPU资源很小，但是性能不高，因为会引起线程切换。

- pthread_spinlock_t :
是自旋锁，同一瞬间也只能有一个线程能够获取锁，不同的是，其他线程在等待获取锁的过程中并不进入睡眠状态，而是在CPU上进入“自旋”等待。自旋锁的性能很高，但是只适合对很小的代码段加锁（或短期持有的锁），自旋锁对CPU的占用相对较高。

- pthread_rwlock_t :
是读写锁，同时可以有多个线程获得读锁，同时只允许有一个线程获得写锁。其他线程在等待锁的时候同样会进入睡眠。读写锁在互斥锁的基础上，允许多个线程“读”，在某些场景下能提高性能。

诸如pthread中的pthread_cond_t, pthread_barrier_t, semaphone等，更像是一种同步原语，不属于单纯的锁。

### TBB中提供的锁有：
- mutex 互斥锁，等同于pthread中的互斥锁（实际上就是对pthread_mutex_t进行封装）
- recurisive_mutex 可重入的互斥锁，在pthread_mutex_t的基础上加了一个可重入的属性
- spin_metux 自旋锁，与pthread_spinlock_t类似，但是性能比pthread_spinlock_t低28%
- queuing_metux 公平的互斥锁，严格按照等待锁的先后顺序获得锁
- spin_rw_mutex 读写自旋锁，功能与pthread_rwlock_t一致，但是性能比pthread_rwlock_t高很多
- queuing_rw_mutex 公平的读写读写锁，也是严格按照等待锁的先后顺序获得锁

以下是一个拥有3667527个节点的HASH表进行读操作所花费的时间，可以说明各种锁的性能：
多线程的环境为：4CPU的电脑上使用四个线程进行同样的度操作，然后取四个线程读取的平均时间·
- 单线程不加锁：0.818845s·
- 多线程使用pthread_mutex_t：120.978713s (很离谱吧…………我也吓了一跳)·
- 多线程使用pthread_rwlock_t：10.592172s （多个线程加读锁）·
- 多线程使用pthread_spinlock_t：4.766012s·
- 多个线程使用tbb::spin_mutex：6.638609s （从这里可以看出pthread的自旋锁比TBB的自旋锁性能高出28%）·
- 多个线程使用tbb::spin_rw_mutex：3.471757s （并行读的环境下，这是所有锁中性能最高的）


## 无锁
无锁，英文一般翻译为lock-free，是利用处理器的一些特殊的原子指令来避免传统并行设计中对锁的使用。 如果一个共享数据结构的操作不需要互斥，那么它是无锁的。如果一个进程或线程在操作中间被中断，其他进程或线程的操作不受影响。[Herlihy 1991] 笔者对于无锁的实践都是在一个进程下关于多线程并发的，所以后面我们只讨论多线程。

### 为什么要无锁?（界定问题）
#### 首先是性能考虑
通信项目一般对性能有极致的追求，这是我们使用无锁的重要原因。当然，无锁算法如果实现的不好，性能可能还不如使用锁，所以我们选择比较擅长的数据结构和算法进行lock-free实现，比如Queue，对于比较复杂的数据结构和算法我们通过lock来控制，比如Map（虽然我们实现了无锁Hash，但是大小是限定的，而Map是大小不限定的）。 对于性能数据，后续文章会给出无锁和有锁的对比。（通过实验进行对比）

#### 其次是避免锁的使用引起的错误和问题：
死锁（dead lock）、 活锁（live lock）、 锁护送（lock convoy）、 优先级反转（priority inversion）

- 死锁(dead lock)：两个以上线程互相等待
- 锁护送(lock convoy)：多个同优先级的线程反复竞争同一个锁，抢占锁失败后强制上下文切换，引起性能下降
- 优先级反转(priority inversion)：低优先级线程拥有锁时被中优先级的线程抢占，而高优先级的线程因为申请不到锁被阻塞


### 如何无锁?（界定问题）
在现代的 CPU 处理器上，很多操作已经被设计为原子的，比如对齐读（Aligned Read）和对齐写（Aligned Write）等。Read-Modify-Write（RMW）操作的设计让执行更复杂的事务操作变成了原子操作，当有多个写入者想对相同的内存进行修改时，保证一次只执行一个操作。

RMW 操作在不同的 CPU 家族中是通过不同的方式来支持的：
- x86/64 和 Itanium 架构通过 Compare-And-Swap (CAS) 方式来实现
- PowerPC、MIPS 和 ARM 架构通过 Load-Link/Store-Conditional (LL/SC) 方式来实现

笔者都是在x64下进行实践的，用的是CAS操作，CAS操作是lock-free技术的基础，我们可以用下面的代码来描述：

``` C++
template <class T> bool CAS(T* addr, T expected, T value)
{   
    if (*addr == expected)   
    {      
        *addr = value;      
        return true;   
        
    }   
    return false;
}
```

在GCC中，CAS操作如下所示：
``` C++
bool __sync_bool_compare_and_swap (type *ptr, type oldval type newval, ...)
type __sync_val_compare_and_swap (type *ptr, type oldval type newval, ...)
```
==这两个函数提供原子的比较和交换，如果ptr == oldval，就将newval写入ptr，第一个函数在相等并写入的情况下返回true，第二个函数的内置行为和第一个函数相同，只是它返回操作之前的值。==
。。这文章也是 markdown啊。

后面的可扩展参数(...)用来指出哪些变量需要memory barrier，因为目前gcc实现的是full barrier，所以可以略掉这个参数。

除过CAS操作，GCC还提供了其他一些原子操作，可以在无锁算法中灵活使用：

``` C++
type __sync_fetch_and_add (type *ptr, type value, ...)
type __sync_fetch_and_sub (type *ptr, type value, ...)
type __sync_fetch_and_or (type *ptr, type value, ...) 
type __sync_fetch_and_and (type *ptr, type value, ...) 
type __sync_fetch_and_xor (type *ptr, type value, ...) 
type __sync_fetch_and_nand (type *ptr, type value, ...) 

type __sync_add_and_fetch (type *ptr, type value, ...) 
type __sync_sub_and_fetch (type *ptr, type value, ...) 
type __sync_or_and_fetch (type *ptr, type value, ...) 
type __sync_and_and_fetch (type *ptr, type value, ...) 
type __sync_xor_and_fetch (type *ptr, type value, ...) 
type __sync_nand_and_fetch (type *ptr, type value, ...)
```

`_sync*`系列的built-in函数，用于提供加减和逻辑运算的原子操作。这两组函数的区别在于第一组返回更新前的值，第二组返回更新后的值。

笔者开发无锁算法感触最深的是复杂度的分解，比如多线程对于一个双向链表的插入或删除操作，如何能==一步一步分解成一个一个串联的原子操作，并能保证事务内存的一致性==。


### CAS等原子操作
在开始说无锁队列之前，我们需要知道一个很重要的技术就是CAS操作——Compare & Set，或是 Compare & Swap，现在几乎所有的CPU指令都支持CAS的原子操作，X86下对应的是 CMPXCHG 汇编指令。有了这个原子操作，我们就可以用其来实现各种无锁（lock free）的数据结构。

这个操作用C语言来描述就是下面这个样子：（代码来自Wikipedia的Compare And Swap词条）意思就是说，看一看内存`*reg`里的值是不是oldval，如果是的话，则对其赋值newval。

``` C++
int compare_and_swap (int* reg, int oldval, int newval)
{
  int old_reg_val = *reg;
  if (old_reg_val == oldval)
     *reg = newval;
  return old_reg_val;
}
```

这个操作可以变种为返回bool值的形式（返回 bool值的好处在于，可以调用者知道有没有更新成功）：

```
bool compare_and_swap (int *accum, int *dest, int newval)
{
  if ( *accum == *dest ) {
      *dest = newval;
      return true;
  }
  return false;
}
```

与CAS相似的还有下面的原子操作：（这些东西大家自己看Wikipedia吧）

Fetch And Add，一般用来对变量做 +1 的原子操作 Test-and-set，写值到某个内存位置并传回其旧值。汇编指令BST Test and Test-and-set，用来低低Test-and-Set的资源争夺情况 
注：在实际的C/C++程序中，CAS的各种实现版本如下：
#### GCC的CAS
```
bool __sync_bool_compare_and_swap (type *ptr, type oldval type newval, ...)
type __sync_val_compare_and_swap (type *ptr, type oldval type newval, ...)
```

#### Windows的CAS
```
InterlockedCompareExchange ( __inout LONG volatile *Target,
                                __in LONG Exchange,
                                __in LONG Comperand);
```

#### C++11中的CAS
```
template< class T >
bool atomic_compare_exchange_weak( std::atomic* obj,
                                   T* expected, T desired );
template< class T >
bool atomic_compare_exchange_weak( volatile std::atomic* obj,
                                   T* expected, T desired );
```


### 无锁队列的链表实现

```
EnQueue(x) //进队列
{
    //准备新加入的结点数据
    q = new record();
    q->value = x;
    q->next = NULL;
 
    do {
        p = tail; //取链表尾指针的快照
    } while( CAS(p->next, NULL, q) != TRUE); //如果没有把结点链在尾指针上，再试
 
    CAS(tail, p, q); //置尾结点
}
```

这里有一个潜在的问题——如果T1线程在用CAS更新tail指针的之前，线程停掉或是挂掉了，那么其它线程就进入死循环了。下面是改良版的EnQueue()

```
EnQueue(x) //进队列改良版
{
    q = new record();
    q->value = x;
    q->next = NULL;
 
    p = tail;
    oldp = p
    do {
        while (p->next != NULL)
            p = p->next;
    } while( CAS(p.next, NULL, q) != TRUE); //如果没有把结点链在尾上，再试
 
    CAS(tail, oldp, q); //置尾结点，不准了。
}
```

我们让每个线程，自己fetch 指针 p 到链表尾。但是这样的fetch会很影响性能。而通实际情况看下来，99.9%的情况不会有线程停转的情况，所以，更好的做法是，你可以接合上述的这两个版本，如果retry的次数超了一个值的话（比如说3次），那么，就自己fetch指针。

好了，我们解决了EnQueue，我们再来看看DeQueue的代码：（很简单，我就不解释了）

```
DeQueue() //出队列
{
    do{
        p = head;
        if (p->next == NULL){
            return ERR_EMPTY_QUEUE;
        }
    while( CAS(head, p, p->next) != TRUE );
    return p->next->value;
}
```

DeQueue的代码操作的是 head->next，而不是head本身。这样考虑是因为一个边界条件，我们需要一个dummy的头指针来解决链表中如果只有一个元素，head和tail都指向同一个结点的问题，这样EnQueue和DeQueue要互相排斥了。


### ABA问题

ABA问题最容易发生在lock free 的算法中的，CAS首当其冲，因为CAS判断的是==指针的地址==。如果这个地址被重用了呢，问题就很大了。（地址被重用是很经常发生的，一个内存分配后释放了，再分配，很有可能还是原来的地址）
。。地址不变，但是里面的值变了。

#### 解决ABA的问题
维基百科上给了一个解——使用double-CAS（双保险的CAS），例如，在32位系统上，我们要检查64位的内容
1. 一次用CAS检查双倍长度的值，前半部是指针，后半部分是一个计数器。
2. 只有这两个都一样，才算通过检查，赋新的值。并把计数器累加1。

这样一来，ABA发生时，虽然值一样，但是计数器就不一样（但是在32位的系统上，这个计数器会溢出回来又从1开始的，这还是会有ABA的问题）
。。溢出。。21亿呢。

我们这个队列的问题就是不想让那个内存重用，这样明确的业务问题比较好解决，论文《Implementing Lock-Free Queues》给出一这么一个方法——使用结点内存引用计数refcnt！

```
SafeRead(q)
{
    loop:
        p = q->next;
        if (p == NULL){
            return p;
        }
 
        Fetch&Add(p->refcnt, 1);
 
        if (p == q->next){
            return p;
        }else{
            Release(p);
        }
    goto loop;
}
```

其中的 Fetch&Add和Release分是是加引用计数和减引用计数，都是原子操作，这样就可以阻止内存被回收了。

### 用数组实现无锁队列

https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.53.8674&rep=rep1&type=pdf

### 小结
以上基本上就是所有的无锁队列的技术细节，这些技术都可以用在其它的无锁数据结构上。
1. 无锁队列主要是通过CAS、FAA这些原子操作，和Retry-Loop实现。
2. 对于Retry-Loop，我个人感觉其实和锁什么什么两样。只是这种“锁”的粒度变小了，主要是“锁”HEAD和TAIL这两个关键资源。而不是整个数据结构。

。。FAA： fetch and add

















------------------








------------------

------------------


------------------





==================

https://mp.weixin.qq.com/s?__biz=MzkxNzQzNDM2Ng==&mid=2247493342&idx=1&sn=881ad7ac1caad30a2a30f95fd91815e6&source=41#wechat_redirect

# 值得学习的C语言开源项目
Webbench  网站压测工具
Tinyhttpd  轻量http server
cJSON    JSON编解码器
CMockery  C单元测试
Libev    事件驱动库，基于epoll，kqueue
Memcached   高性能分布式内存对象缓存系统
Lua
SQLite   嵌入式关系数据库
UNIX v6
NETBSD   UNIX-like OS

C++标准库
C++ Standard Library：是一系列类和函数的集合，使用核心语言编写，也是C++ISO自身标准的一部分。
Standard Template Library：标准模板库
C POSIX library ：POSIX系统的C标准库规范
ISO C++ Standards Committee ：C++标准委员会

C++通用框架和库
    Apache C++ Standard Library：是一系列算法，容器，迭代器和其他基本组件的集合
    ASL ：Adobe源代码库提供了同行的评审和可移植的C++源代码库。
    Boost ：大量通用C++库的集合。
    BDE ：来自于彭博资讯实验室的开发环境。
    Cinder：提供专业品质创造性编码的开源开发社区。
    Cxxomfort：轻量级的，只包含头文件的库，将C++ 11的一些新特性移植到C++03中。
    Dlib：使用契约式编程和现代C++科技设计的通用的跨平台的C++库。
    EASTL ：EA-STL公共部分
    ffead-cpp ：企业应用程序开发框架
    Folly：由Facebook开发和使用的开源C++库
    JUCE ：包罗万象的C++类库，用于开发跨平台软件
    libPhenom：用于构建高性能和高度可扩展性系统的事件框架。
    LibSourcey ：用于实时的视频流和高性能网络应用程序的C++11 evented IO
    LibU ：C语言写的多平台工具库
    Loki ：C++库的设计，包括常见的设计模式和习语的实现。
    MiLi ：只含头文件的小型C++库
    openFrameworks ：开发C++工具包，用于创意性编码。
    Qt ：跨平台的应用程序和用户界面框架
    Reason ：跨平台的框架，使开发者能够更容易地使用Java，.Net和Python，同时也满足了他们对C++性能和优势的需求。
    ROOT ：具备所有功能的一系列面向对象的框架，能够非常高效地处理和分析大量的数据，为欧洲原子能研究机构所用。
    STLport：是STL具有代表性的版本
    STXXL：用于额外的大型数据集的标准模板库。
    Ultimate++ ：C++跨平台快速应用程序开发框架
    Windows Template Library：用于开发Windows应用程序和UI组件的C++库
    Yomm11 ：C++11的开放multi-methods.

人工智能
    btsk ：游戏行为树启动器工具
    Evolving Objects：基于模板的，ANSI C++演化计算库，能够帮助你非常快速地编写出自己的随机优化算法。
    Neu：C++11框架，编程语言集，用于创建人工智能应用程序的多用途软件系统。

异步事件循环
     Boost.Asio：用于网络和底层I/O编程的跨平台的C++库。
    libev ：功能齐全，高性能的时间循环，轻微地仿效libevent，但是不再像libevent一样有局限性，也修复了它的一些bug。
    libevent ：事件通知库
    libuv ：跨平台异步I/O。

音频，声音，音乐，数字化音乐库
    FMOD ：易于使用的跨平台的音频引擎和音频内容的游戏创作工具。
    Maximilian ：C++音频和音乐数字信号处理库
    OpenAL ：开源音频库—跨平台的音频API
    Opus：一个完全开放的，免版税的，高度通用的音频编解码器
    Speex：免费编解码器，为Opus所废弃
    Tonic：C++易用和高效的音频合成
    Vorbis：Ogg Vorbis是一种完全开放的，非专有的，免版税的通用压缩音频格式。

生物信息，基因组学和生物技术
    libsequence：用于表示和分析群体遗传学数据的C++库。
    SeqAn：专注于生物数据序列分析的算法和数据结构。
    Vcflib ：用于解析和处理VCF文件的C++库
    Wham：直接把联想测试应用到BAM文件的基因结构变异。

压缩和归档库
    bzip2：一个完全免费，免费专利和高质量的数据压缩
    doboz：能够快速解压缩的压缩库
    PhysicsFS：对各种归档提供抽象访问的库，主要用于视频游戏，设计灵感部分来自于Quake3的文件子系统。

    KArchive：用于创建，读写和操作文件档案（例如zip和 tar）的库，它通过QIODevice的一系列子类，使用gzip格式，提供了透明的压缩和解压缩的数据。

    LZ4 ：非常快速的压缩算法
    LZHAM ：无损压缩数据库，压缩比率跟LZMA接近，但是解压缩速度却要快得多。
    LZMA ：7z格式默认和通用的压缩方法。
    LZMAT ：及其快速的实时无损数据压缩库
    miniz：单一的C源文件，紧缩/膨胀压缩库，使用zlib兼容API，ZIP归档读写，PNG写方式。
    Minizip：Zlib最新bug修复，支持PKWARE磁盘跨越，AES加密和IO缓冲。
    Snappy ：快速压缩和解压缩
    ZLib ：非常紧凑的数据流压缩库
    ZZIPlib：提供ZIP归档的读权限。

并发执行和多线程
    Boost.Compute ：用于OpenCL的C++GPU计算库
    Bolt ：针对GPU进行优化的C++模板库
    C++React ：用于C++11的反应性编程库
    Intel TBB ：Intel线程构件块
    Libclsph：基于OpenCL的GPU加速SPH流体仿真库
    OpenCL ：并行编程的异构系统的开放标准
    OpenMP：OpenMP API
    Thrust ：类似于C++标准模板库的并行算法库
    HPX ：用于任何规模的并行和分布式应用程序的通用C++运行时系统
    VexCL ：用于OpenCL/CUDA 的C++向量表达式模板库。

容器
    C++ B-tree ：基于B树数据结构，实现命令内存容器的模板库
    Hashmaps：C++中开放寻址哈希表算法的实现

密码学
    Bcrypt ：一个跨平台的文件加密工具，加密文件可以移植到所有可支持的操作系统和处理器中。
    BeeCrypt：
    Botan：C++加密库
    Crypto++：一个有关加密方案的免费的C++库
    GnuPG：OpenPGP标准的完整实现
    GnuTLS ：实现了SSL，TLS和DTLS协议的安全通信库
    Libgcrypt
    libmcrypt
    LibreSSL：免费的SSL/TLS协议，属于2014 OpenSSL的一个分支
    LibTomCrypt：一个非常全面的，模块化的，可移植的加密工具
    libsodium：基于NaCI的加密库，固执己见，容易使用
    Nettle 底层的加密库
    OpenSSL ：一个强大的，商用的，功能齐全的，开放源代码的加密库。
    Tiny AES128 in C ：用C实现的一个小巧，可移植的实现了AES128ESB的加密算法

数据库，SQL服务器，ODBC驱动程序和工具
    hiberlite ：用于Sqlite3的C++对象关系映射
    Hiredis：用于Redis数据库的很简单的C客户端库
    LevelDB：快速键值存储库
    LMDB：符合数据库四大基本元素的嵌入键值存储
    MySQL++：封装了MySql的C API的C++ 包装器
    RocksDB：来自Facebook的嵌入键值的快速存储
    SQLite：一个完全嵌入式的，功能齐全的关系数据库，只有几百KB，可以正确包含到你的项目中。

调试库， 内存和资源泄露检测，单元测试
    Boost.Test：Boost测试库
    Catch：一个很时尚的，C++原生的框架，只包含头文件，用于单元测试，测试驱动开发和行为驱动开发。
    CppUnit：由JUnit移植过来的C++测试框架
    CTest：CMake测试驱动程序
    googletest：谷歌C++测试框架
    ig-debugheap：用于跟踪内存错误的多平台调试堆
    libtap：用C语言编写测试
    MemTrack —用于C++跟踪内存分配
    microprofile- 跨平台的网络试图分析器
    minUnit ：使用C写的迷你单元测试框架，只使用了两个宏
    Remotery：用于web视图的单一C文件分析器
    UnitTest++：轻量级的C++单元测试框架

游戏引擎
    Cocos2d-x ：一个跨平台框架，用于构建2D游戏，互动图书，演示和其他图形应用程序。
    Grit ：社区项目，用于构建一个免费的游戏引擎，实现开放的世界3D游戏。
    Irrlicht ：C++语言编写的开源高性能的实时#D引擎
    Polycode：C++实现的用于创建游戏的开源框架（与Lua绑定）。

图形用户界面
    CEGUI ：很灵活的跨平台GUI库
    FLTK ：快速，轻量级的跨平台的C++GUI工具包。
    GTK+：用于创建图形用户界面的跨平台工具包
    gtkmm ：用于受欢迎的GUI库GTK+的官方C++接口。
    imgui：拥有最小依赖关系的立即模式图形用户界面
    libRocket ：libRocket 是一个C++ HTML/CSS 游戏接口中间件
    MyGUI ：快速，灵活，简单的GUI
    Ncurses：终端用户界面
    QCustomPlot ：没有更多依赖关系的Qt绘图控件
    Qwt ：用户与技术应用的Qt 控件
    QwtPlot3D ：功能丰富的基于Qt/OpenGL的C++编程库，本质上提供了一群3D控件
    OtterUI ：OtterUI 是用于嵌入式系统和互动娱乐软件的用户界面开发解决方案
    PDCurses 包含源代码和预编译库的公共图形函数库
    wxWidgets C++库，允许开发人员使用一个代码库可以为widows， Mac OS X，Linux和其他平台创建应用程序

图形
    bgfx：跨平台的渲染库
    Cairo：支持多种输出设备的2D图形库
    Horde3D 一个小型的3D渲染和动画引擎
    magnum C++11和OpenGL 2D/3D 图形引擎
    Ogre 3D 用C++编写的一个面向场景，实时，灵活的3D渲染引擎（并非游戏引擎）
    OpenSceneGraph 具有高性能的开源3D图形工具包
    Panda3D 用于3D渲染和游戏开发的框架，用Python和C++编写。
    Skia 用于绘制文字，图形和图像的完整的2D图形库
    urho3d 跨平台的渲染和游戏引擎。

图像处理
    Boost.GIL：通用图像库
    CImg ：用于图像处理的小型开源C++工具包

    CxImage ：用于加载，保存，显示和转换的图像处理和转换库，可以处理的图片格式包括 BMP, JPEG, GIF, PNG, TIFF, MNG, ICO, PCX, TGA, WMF, WBMP, JBG, J2K。

    FreeImage ：开源库，支持现在多媒体应用所需的通用图片格式和其他格式。
    GDCM：Grassroots DICOM 库
    ITK：跨平台的开源图像分析系统
    Magick++：ImageMagick程序的C++接口
    MagickWnd：ImageMagick程序的C++接口
    OpenCV ：开源计算机视觉类库
    tesseract-ocr：OCR引擎
    VIGRA ：用于图像分析通用C++计算机视觉库
    VTK ：用于3D计算机图形学，图像处理和可视化的开源免费软件系统。

国际化
    gettext ：GNU  gettext
    IBM ICU：提供Unicode 和全球化支持的C、C++ 和Java库
    libiconv ：用于不同字符编码之间的编码转换库

Json
    frozen ：C/C++的Json解析生成器
    Jansson ：进行编解码和处理Json数据的C语言库
    jbson ：C++14中构建和迭代BSON data,和Json 文档的库
    JeayeSON：非常健全的C++ JSON库，只包含头文件
    JSON++ ：C++ JSON 解析器
    json-parser：用可移植的ANSI C编写的JSON解析器，占用内存非常少
    json11 ：一个迷你的C++11 JSON库
    jute ：非常简单的C++ JSON解析器
    ibjson：C语言中的JSON解析和打印库，很容易和任何模型集成。
    libjson：轻量级的JSON库
    PicoJSON：C++中JSON解析序列化，只包含头文件
    qt-json ：用于JSON数据和 QVariant层次间的相互解析的简单类
    QJson：将JSON数据映射到QVariant对象的基于Qt的库
    RapidJSON：用于C++的快速JSON 解析生成器，包含SAX和DOM两种风格的API
    YAJL ：C语言中快速流JSON解析库

日志
    Boost.Log ：设计非常模块化，并且具有扩展性
    easyloggingpp：C++日志库，只包含单一的头文件。
    Log4cpp ：一系列C++类库，灵活添加日志到文件，系统日志，IDSA和其他地方。
    templog：轻量级C++库，可以添加日志到你的C++应用程序中

机器学习
    Caffe ：快速的神经网络框架
    CCV ：以C语言为核心的现代计算机视觉库
    mlpack ：可扩展的C++机器学习库
    OpenCV：开源计算机视觉库
    Recommender：使用协同过滤进行产品推荐/建议的C语言库。
    SHOGUN：Shogun 机器学习工具
    sofia-ml ：用于机器学习的快速增量算法套件

数学
    Armadillo ：高质量的C++线性代数库，速度和易用性做到了很好的平衡。语法和MatlAB很相似
    blaze：高性能的C++数学库，用于密集和稀疏算法。
    ceres-solver ：来自谷歌的C++库，用于建模和解决大型复杂非线性最小平方问题。
    CGal：高效，可靠的集合算法集合
    cml ：用于游戏和图形的免费C++数学库
    Eigen ：高级C++模板头文件库，包括线性代数，矩阵，向量操作，数值解决和其他相关的算法。
    GMTL：数学图形模板库是一组广泛实现基本图形的工具。
    GMP：用于个高精度计算的C/C++库，处理有符号整数，有理数和浮点数。

多媒体
    GStreamer ：构建媒体处理组件图形的库
    LIVE555 Streaming Media ：使用开放标准协议(RTP/RTCP, RTSP, SIP) 的多媒体流库
    libVLC ：libVLC (VLC SDK)媒体框架
    QtAv：基于Qt和FFmpeg的多媒体播放框架，能够帮助你轻而易举地编写出一个播放器
    SDL ：简单直控媒体层
    SFML ：快速，简单的多媒体库

网络
    ACE：C++面向对象网络变成工具包
    Boost.Asio：用于网络和底层I/O编程的跨平台的C++库
    Casablanca：C++ REST SDK
    cpp-netlib：高级网络编程的开源库集合
    Dyad.c：C语言的异步网络
    libcurl :多协议文件传输库
    Mongoose：非常轻量级的网络服务器
    Muduo ：用于Linux多线程服务器的C++非阻塞网络库
    net_skeleton ：C/C++的TCP 客户端/服务器库
    nope.c ：基于C语言的超轻型软件平台，用于可扩展的服务器端和网络应用。对于C编程人员，可以考虑node.js
    Onion :C语言HTTP服务器库，其设计为轻量级，易使用。
    POCO：用于构建网络和基于互联网应用程序的C++类库，可以运行在桌面，服务器，移动和嵌入式系统。
    RakNet：为游戏开发人员提供的跨平台的开源C++网络引擎。
    Tuf o ：用于Qt之上的C++构建的异步Web框架。
    WebSocket++ ：基于C++/Boost Aiso的websocket 客户端/服务器库
    ZeroMQ ：高速，模块化的异步通信库

动力学仿真引擎
    Box2D：2D的游戏物理引擎。
    Bullet ：3D的游戏物理引擎。
    Chipmunk ：快速，轻量级的2D游戏物理库
    LiquidFun：2D的游戏物理引擎
    ODE ：开放动力学引擎-开源，高性能库，模拟刚体动力学。
    ofxBox2d：Box2D开源框架包装器。
    Simbody ：高性能C++多体动力学/物理库，模拟关节生物力学和机械系统，像车辆，机器人和人体骨骼。

机器人学
    MOOS-IvP ：一组开源C++模块，提供机器人平台的自主权，尤其是自主的海洋车辆。
    MRPT：移动机器人编程工具包
    PCL ：点云库是一个独立的，大规模的开放项目，用于2D/3D图像和点云处理。
    Robotics Library (RL)：一个独立的C++库，包括机器人动力学，运动规划和控制。
    RobWork：一组C++库的集合，用于机器人系统的仿真和控制。
    ROS ：机器人操作系统，提供了一些库和工具帮助软件开发人员创建机器人应用程序。

科学计算
    FFTW :用一维或者多维计算DFT的C语言库。
    GSL：GNU科学库。

脚本
    ChaiScript ：用于C++的易于使用的嵌入式脚本语言。
    Lua ：用于配置文件和基本应用程序脚本的小型快速脚本引擎。
    luacxx：用于创建Lua绑定的C++ 11 API
    SWIG ：一个可以让你的C++代码链接到JavaScript，Perl，PHP，Python，Tcl和Ruby的包装器/接口生成器
    V7：嵌入式的JavaScript 引擎。
    V8 ：谷歌的快速JavaScript引擎，可以被嵌入到任何C++应用程序中。

序列化
    Cap’n Proto ：快速数据交换格式和RPC系统。
    cereal ：C++11 序列化库
    FlatBuffers ：内存高效的序列化库
    MessagePack ：C/C++的高效二进制序列化库，例如 JSON
    protobuf ：协议缓冲，谷歌的数据交换格式。
    protobuf-c ：C语言的协议缓冲实现
    SimpleBinaryEncoding：用于低延迟应用程序的对二进制格式的应用程序信息的编码和解码。
    Thrift ：高效的跨语言IPC/RPC，用于C++，Java，Python，PHP，C#和其它多种语言中，最初由Twitter开发。

视频
    libvpx ：VP8/VP9编码解码SDK
    FFmpeg ：一个完整的，跨平台的解决方案，用于记录，转换视频和音频流。
    libde265 ：开放的h.265视频编解码器的实现。
    OpenH264：开源H.364 编解码器。
    Theora ：免费开源的视频压缩格式。

虚拟机
    CarpVM：C中有趣的VM，让我们一起来看看这个。
    MicroPython ：旨在实现单片机上Python3.x的实现
    TinyVM：用纯粹的ANSI C编写的小型，快速，轻量级的虚拟机。

Web应用框架
    Civetweb ：提供易于使用，强大的，C/C++嵌入式Web服务器，带有可选的CGI，SSL和Lua支持。
    CppCMS ：免费高性能的Web开发框架（不是 CMS）.
    Crow ：一个C++微型web框架（灵感来自于Python Flask）
    Kore :使用C语言开发的用于web应用程序的超快速和灵活的web服务器/框架。
    libOnion：轻量级的库，帮助你使用C编程语言创建web服务器。
    QDjango：使用C++编写的，基于Qt库的web框架，试图效仿Django API，因此得此名。
    Wt ：开发Web应用的C++库。

XML
XML就是个垃圾，xml的解析很烦人，对于计算机它也是个灾难。这种糟糕的东西完全没有存在的理由了。-Linus Torvalds

    Expat ：用C语言编写的xml解析库
    Libxml2 ：Gnome的xml C解析器和工具包
    libxml++ ：C++的xml解析器
    PugiXML ：用于C++的，支持XPath的轻量级，简单快速的XML解析器。
    RapidXml ：试图创建最快速的XML解析器，同时保持易用性，可移植性和合理的W3C兼容性。
    TinyXML ：简单小型的C++XML解析器，可以很容易地集成到其它项目中。
    TinyXML2：简单快速的C++CML解析器，可以很容易集成到其它项目中。
    TinyXML++：TinyXML的一个全新的接口，使用了C++的许多许多优势，模板，异常和更好的异常处理。
    Xerces-C++ ：用可移植的C++的子集编写的XML验证解析器。

多项混杂
一些有用的库或者工具，但是不适合上面的分类，或者还没有分类。
    C++ Format ：C++的小型，安全和快速格式化库
    casacore ：从aips++ 派生的一系列C++核心库
    cxx-prettyprint：用于C++容器的打印库
    DynaPDF ：易于使用的PDF生成库
    gcc-poison ：帮助开发人员禁止应用程序中的不安全的C/C++函数的简单的头文件。
    googlemock：编写和使用C++模拟类的库
    HTTP Parser ：C的http请求/响应解析器
    libcpuid ：用于x86 CPU检测盒特征提取的小型C库
    libevil ：许可证管理器
    libusb：允许移动访问USB设备的通用USB库
    PCRE：正则表达式C库，灵感来自于Perl中正则表达式的功能。
    Remote Call Framework ：C++的进程间通信框架。
    Scintilla ：开源的代码编辑控件
    Serial Communication Library ：C++语言编写的跨平台，串口库。
    SDS：C的简单动态字符串库
    SLDR ：超轻的DNS解析器
    SLRE：超轻的正则表达式库
    Stage ：移动机器人模拟器
    VarTypes：C++/Qt4功能丰富，面向对象的管理变量的框架。
    ZBar：‘条形码扫描器’库，可以扫描照片，图片和视频流中的条形码，并返回结果。
    CppVerbalExpressions ：易于使用的C++正则表达式
    QtVerbalExpressions：基于C++ VerbalExpressions 库的Qt库
    PHP-CPP：使用C++来构建PHP扩展的库
    Better String ：C的另一个字符串库，功能更丰富，但是没有缓冲溢出问题，还包含了一个C++包装器。

C/C++编译器列表
    Clang :由苹果公司开发的
    GCC：GNU编译器集合
    Intel C++ Compiler ：由英特尔公司开发
    LLVM ：模块化和可重用编译器和工具链技术的集合
    Microsoft Visual C++ ：MSVC，由微软公司开发
    Open WatCom ：Watcom，C，C++和Fortran交叉编译器和工具
    TCC ：轻量级的C语言编译器

在线C/C++编译器列表
    codepad ：在线编译器/解释器，一个简单的协作工具
    CodeTwist：一个简单的在线编译器/解释器，你可以粘贴的C,C++或者Java代码，在线执行并查看结果
    coliru ：在线编译器/shell， 支持各种C++编译器
    Compiler Explorer：交互式编译器，可以进行汇编输出
    CompileOnline：Linux上在线编译和执行C++程序
    Ideone ：一个在线编译器和调试工具，允许你在线编译源代码并执行，支持60多种编程语言。

C/C++调试器列表
    Comparison of debuggers ：来自维基百科的调试器列表
    GDB ：GNU调试器
    Valgrind：内存调试，内存泄露检测，性能分析工具。

构建系统
    Bear ：用于为clang工具生成编译数据库的工具
    Biicode：基于文件的简单依赖管理器。
    CMake ：跨平台的免费开源软件用于管理软件使用独立编译的方法进行构建的过程。
    CPM：基于CMake和Git的C++包管理器
    FASTBuild：高性能，开源的构建系统，支持高度可扩展性的编译，缓冲和网络分布。
    Ninja ：专注于速度的小型构建系统
    Scons ：使用Python scipt 配置的软件构建工具
    tundra ：高性能的代码构建系统，甚至对于非常大型的软件项目，也能提供最好的增量构建次数。
    tup：基于文件的构建系统，用于后台监控变化的文件。

提高质量，减少瑕疵的代码分析工具列表
    Cppcheck ：静态C/C++代码分析工具
    include-what-you-use ：使用clang进行代码分析的工具，可以#include在C和C++文件中。
    OCLint ：用于C，C++和Objective-C的静态源代码分析工具，用于提高质量，减少瑕疵。
    Clang Static Analyzer：查找C，C++和Objective-C程序bug的源代码分析工具
    List of tools for static code analysis ：来自维基百科的静态代码分析工具列表

==================

==================

brpc

==================

==================

已使用 Microsoft OneNote 2016 创建。