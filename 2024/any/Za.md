Za
2022年3月8日
14:52
1


`x + ~x = -1, so ~x = -1 - x`


[[toc]]


==================

# MDC
Mapped Diagnostic Context，可以粗略的理解成是一个线程安全的存放诊断日志的容器。

```Java
public class SimpleMDC {

    private static final Logger logger = LoggerFactory.getLogger(SimpleMDC.class);

    public static final String REQ_ID = "REQ_ID";

    public static void main(String[] args) {
        MDC.put(REQ_ID, UUID.randomUUID().toString());
        logger.info("开始调用服务A，进行业务处理");
        logger.info("业务处理完毕，可以释放空间了，避免内存泄露");
        MDC.remove(REQ_ID);
        logger.info("REQ_ID 还有吗？{}", MDC.get(REQ_ID) != null);
    }
}
```

通过 %X{REQ_ID} 来打印 REQ_ID 的信息，logback.xml 文件
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <Pattern>[%t] [%X{REQ_ID}] - %m%n</Pattern>
        </layout>
    </appender>
    <root level="debug">
        <appender-ref ref="CONSOLE"/>
    </root>
</configuration>
```

![1](../_resources/440e2cc956274713bd3b86208288f09b.png)

第一：如图中红色圈住部分所示，当 logback 内置的日志字段不能满足业务需求时，便可以借助 MDC 机制，将业务上想要输出的信息，通过 logback 给打印出来；

第二：如蓝色圈住部分所示，当调用 MDC.remove(Key) 后，便可将业务字段从 MDC 中删除，日志中就不再打印请求 ID 啦；

ThreadLocal。ThreadLocalMap，后者是  前者的  静态内部类。

==================

==================

# Q: one queue or many queue
一个queue中  太多数据的话，会  导致  处理慢  ？  主要是感觉会：
所以可以分为  多个  queue。

这种，  对于一个订单的  create，update，cancel，return，支付，下单(未支付)  等一系列操作  是放一个  queue  好，还是  多个queue？   一个queue  的话，可以  FIFO。   多个queue的话，多个listener，这种  多个listener  性能好  还是不好？

==================

# DRBD
是由内核模块和相关脚本而构成，用以构建高可用性的集群。其实现方式是通过网络来镜像整个设备。您可以把它看作是一种网络RAID

==================

==================
# ssh正向代理(转发)
ssh -L 33322:202.222.222.22:443 username@111.111.11.1

所有  对  localhost:33322  的访问  会通过  111.111.11.1  转发到  202.222.222.22的443端口。

local -> sandbox server -> zozotown

==================

# 审阅多线程代码需要考虑的问题
并发访问时，哪些数据需要保护？
如何确定访问数据受到了保护？
是否会有多个线程同时访问这段代码？
这个线程获取了哪个互斥量？
其他线程可能获取哪些互斥量？
两个线程间的操作是否有依赖关系？如何满足这种关系？
这个线程加载的数据还是合法数据吗？数据是否被其他线程修改过？
当假设其他线程可以对数据进行修改，这将意味着什么？并且，怎么确保这样的事情不会发生？

==================

# RDF
resource description framework,  使用xml语法来表述  Data model,用来描述  web资源的特性，以及资源  之间的关系。

RDFS, RDF Schema

==================

==================

# 对于异步接口，开放接口来查询进度
对于有些  异步  接口，开放  额外的接口，  来让对方知道  异步事件  是什么状态。

不过需要  数据库保存  异步事件  的状态。    看得失，主要是看对面  会不会一直问  是什么状态了。   还有   异步处理的  结果。之类的。

==================

# 前沿，网站
一只IT猫

前端：看目前大众风向
后端：业务趋势得看具体内容，不同公司开发的内容不同，方向也不同，很难说有什么具体的方向。技术方向的话就是分布式，微服务。
部署：整体是云原生，serverless化。当然仅限个人认为，容器和容器编排技术带来的改变不亚于，手机从诺基亚到安卓。

了解计算机的发展：看看IEEE的会议，[https://www.ieee.org/]
了解数据库发展：[https://sigmod.org/]
科技行业应用：大科技里很多名人的推或者博客、播客
互联网&商业：很多商业的论坛网站，信息源就太多了
编程语言：看新版更新日志和未来支持的功能
某个库或者中间件：通常看官网更新日志
机器学习或者深度学习：某些开源网站、数据集&模型网站、论文等
通讯行业：3gpp会议，[https://www.3gpp.org/]
某些解决方案：头部玩家的官方博客，比如XXX使用a替换了b作为最新的数据库，这种贴文价值往往很高

Big tech通常来说指美国那几个大型科技公司，谷歌、亚马逊、苹果、Meta、微软
不过我这里的意思是泛指那些大型科技公司还有一线互联网企业等，比如奈飞、BAT TMD等等把
这些大公司往往会有很多牛人出来写一些高质量的东西，包括对某些技术的应用、前景、实际生产环境的经验等等

==================

==================

# 旋轮线

==================
# 库存
SAP->OMS, oms->各个渠道，oms->SAP
必然存在延时，必然会超卖啊。

为什么不是一个  库存数据库呢？
每次用户查看产品的时候，如果都要  走核心库存  库，那么核心的压力会大，而且地理位置的原因，会导致延迟。

但是，真正付钱的时候  肯定要  锁核心的库存啊。  并且刷新  渠道中保存的库存值。

现在情况是：
SAP的库存  生成文件后，1min内会同步到  OMS，但是看了下文件，一天也就生成几个，频率很低，不知道为什么。
OMS->各个渠道，是每隔15min同步  上次成功到现在的  数据。看起来这个数据的修改挺频繁的。
OMS->SAP，是每天  同步一次全量。

所以应该是  SAP  是  进行库存的增加。
渠道是进行  库存的  减少。  估计渠道卖出去后，会讲OMS中的库存减少，所以OMS需要通知其他的渠道。
OMS在中间是  cache。  OMS的库存就是  核心库。渠道付款的时候肯定会锁的。

这里少一个  SAP  和  工厂的库存。

==================

==================

https://www.zhihu.com/question/30196513/answer/488696885
感觉都有道理。

# 开发步骤

1) 先设计算法和数据结构，
2) 然后选择综合性能最优化的实现逻辑，包括访存、异步等等，
3) 最后再将这些实现逻辑「翻译」为C++代码。
一般来说，只要算法确定，2) 和 3) 两步都是非常自然且快速完成的。

。。感觉就是  先  文字版简单流程，  脑子里运行下，丰富下(  校验，异常，复用，细节算法，数据形式(新class or  复用)  ，内聚，解耦，数据来源，  测试案例  )  ，  然后  写代码。

。。上面缺了：  同步/异步，cache，性能，架构(哪些操作开头就执行，哪些  在需要的时候  才执行)，数据库index，具体用哪个类的哪个方法。

算法设计的核心工作就是数据结构设计
。。有一点点道理，有时候，TLE  就是因为  你不懂某个数据结构

你是拿软件当理科学的——求根，问底，逻辑推导，理论体系，但现在大部分情况下做软件应该具备工科思维——行业标准，相互配合，高效易用，粗粒度装配。

而且，为了能配合好，提高协作效率，可以不惜合理地牺牲运行效率。可读性、可维护性多重要就不说了，而运行效率其实可以被现在强大的硬件水平给找补回来的，汇编、c语言时代那种对内存、处理器资源锱铢必较的行为，不是很必要了。

==================

https://mp.weixin.qq.com/s/JBVR4W4cyd3xVvF6dAEF4g

==================

https://www.deepdh.com/ai

https://ai-bot.cn/

==================

https://www.yuque.com/jtcheng/emcpp/chap0

==================

https://aistudio.baidu.com/aistudio/index

==================

https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md

https://www.stroustrup.com/bs_faq2.html

==================

打字
https://codeflow.biaoyansu.com/

==================

https://kaifa.baidu.com/

==================

书籍
http://www.banshujiang.cn/

https://www.book123.info/

==================

# C++  主流的框架，库

https://mp.weixin.qq.com/s?__biz=MzkxNzQzNDM2Ng==&mid=2247493342&idx=1&sn=881ad7ac1caad30a2a30f95fd91815e6&source=41#wechat_redirect

见JSM - C++

==================

==================

==================

已使用 Microsoft OneNote 2016 创建。


# Nginx架构

https://zhuanlan.zhihu.com/p/396968762
图解软件架构之行业应用篇-NGINX分层

![96d0745b4e8ca250a6efc6321c95b680.png](../_resources/96d0745b4e8ca250a6efc6321c95b680.png)




# Java Unsafe 操作堆外内存



# g++支持c++11/14/17/20?

```shell
// gcc version 9.2.1 20190827 (Red Hat 9.2.1-1) (GCC) 

// 是否支持20，还可以c++11,c++14,c++17
// 下面的结果表明g++支持c++20的

[qwer@192 LeetCodeCCpp]$ g++ -std=c++2a -E - < /dev/null
cc1: warning: command line option ‘-std=c++2a’ is valid for C++/ObjC++ but not for C
# 1 "<stdin>"
# 1 "<built-in>"
# 1 "<command-line>"
# 31 "<command-line>"
# 1 "/usr/include/stdc-predef.h" 1 3 4
# 32 "<command-line>" 2
# 1 "<stdin>"


// g++ 默认支持的 g++ 版本
[qwer@192 LeetCodeCCpp]$ g++ -dM -E -x c++ /dev/null | grep -F __cplusplus
#define __cplusplus 201402L

```


# Sidecar模式，边车模式







# GnuPG
GNU Privacy Guard

一种加密软件，不仅可以用来加密和签名电子邮件，也可以用来签名和加密普通的文件

一个以GNU通用公共许可证释出的开放源码用于加密或签名的软件,可用来取代PGP



# 内存管理
ptmalloc, tcmalloc, jemalloc, buddy(伙伴)



---

# 进程，线程区别，切换了什么，共享了什么

bilibili

```C
#include <stdio.h>
int add(int a, int b);
int main()
{
    int a = 1;
    int b = 2;
    int c = a + b;
    printf("%d\n", c);
    return 0;
}
int add(int a, int b)
{
    return a + b;
}
```

获得 汇编代码
`gcc -S main.c -o main.s`

rsp,rbp
从栈读取数据到 寄存器，然后 执行 + 。  所以 寄存器 和 栈是私有的

寄存器是函数私有的，因为 main 和 add 的寄存器 使用了 同名的寄存器。


线程是 task_struct，里面包含了
- 程序计数器
- 寄存器
- 堆栈
- 状态

所以不同的线程共享了
- unix system V 信号量
- 打开的fd
- 文件系统信息
- 信号处理程序
- 信号结构体
- 虚拟内存
- 命名空间
- IO上下文


fork.c 的 copy_process, 里面 新建了一个 task_struct。 对这个 对象初始化， 有些是 新的，有些是共享的。

---
## 线程切换

core.c 中的 `__schedule` ， pick_next_task 选择下一个线程， context_switch 切换上下文,  switch_to 切换寄存器，栈， mmu_context.h中 switch_mm 切换虚拟地址空间


## 进程虚拟地址空间

- 内核空间 ZONE_HIGHMEM，高端内存，32bit 映射高于 1g的物理内存
- 内核空间 ZONE_NORMAL
- 内核空间 ZONE_DMA， 直接内存访问，加快磁盘和内存数据 交互速度
- 环境变量
- 命令行参数
- 栈区
- 共享库加载区
- 堆区
- .bss，未初始化和初始化为0 (未初始化 和初始化为0 全局变量)
- .data，初始化 全局变量
- .text，代码段






---

# no divide

bilibili

float的需要 浮点运算单元，并非所有的 ARM CPU都有 浮点运算单元(FPU)

没有FPU时，必须依赖 标准库的 浮点近似函数 来帮助计算。

ARM内核直到2004年才引入 除法 指令
大多数 小型 Cortex-M 系列处理器 仍然不支持 有符号 或 无符号的 除法。


华氏 转 摄氏， *5/9
954437177 = (2^33 / 9) + 1
将 /9 变成 乘以 

。看起来， 是 溢出了，溢出的截取，剩下的 低位 32bit 就是 结果。
。代码 & 说：低位r0, 高位r4，r0就是结果，所以直接返回。

除法太耗时，需要专门的电路来计算，会占用大量的 CPU面积。 得不偿失

---

# 提速 160万 倍


任务：文本中找出 14个不同的字符，找到就可以退出。
。。是连续的 14个，且都不相同的字符。

最简单的就是 hash，每次 insert 一个char，判断 hash 集合的 size 是不是 14

```rust
fn simple(i: &[u8]) -> usize {
    return i.windows(14).position(|w| {
        return w.iter().collection::<HashSet<_>>().len() == 14;
    }).map(|x| x+14).unwrap();
}
```
。。这个windows 是 从0开始 每次 14个。。。。。
。。正常来说，肯定是一个。。。哦，题目的问题。当初 从0开始，找到 14个字符就可以了， 实际上是 长度14个子数组，元素各不相同。。。  要用 sliding window。


## 优化1，不满足时提前退出
在插入字符时，如果重复，就退出，开始下一个windows
(运行时间可以提升92%)

```rust
fn simple(i: &[u8]) -> usize {
    return i.windows(14).position(|w| {

        let mut hash_set = HashSet::new();
        for x in w {
            if !has_set.insert(x) {
                return false;
            }
        }
        return true;

    }).map(|x| x+14).unwrap();
}
```


## 优化2，使用vector 代替hash
加速 8.9倍

```rust
fn simple(i: &[u8]) -> usize {
    return i.windows(14).position(|w| {

        let mut vec = Vec::with_capacity(14);
        for x in w {
            if vec.contains(x) { 
                return false; 
            }
            vec.push(*x);
        }
        return true;

    }).map(|x| x+14).unwrap();
}
```


## 优化3, 数组代替vector
快26倍
栈 代替 堆
缓存局部性

```rust
fn simple(i: &[u8]) -> usize {
    return i.windows(14).position(|w| {

        let mut arr = [0u8; 14];
        int mut idx = 0;
        for x in w {
            for i in 0..idx {
                if arr[i] == *x {
                    return false;
                }
            }
            arr[idx] = *x;
            idx += 1;
        }
        return true;

    }).map(|x| x+14).unwrap();
}
```


## 优化4, 位操作，使用14个bit (usize) 代替 数组
233倍

直接 ASCII码 % 32

```rust
fn simple(input: &[u8]) -> usize {
    let mut filter = 0u32;
    input.iter().take(14-1).for_each(|c| filter ^= 1 << (c % 32));  // 获得前13个字符

    input.windows(14).pisition(|w| {
        let first = w[0];
        let last = w[w.lwn() - 1];
        filter ^= 1 << (last % 32);     // + 最后一个
        let res = filter.count_ones() == 14 as u32;
        filter ^= 1 << (first % 32);    // - 第一个
        res
    })
}
```
。。这里主要是把 遍历arr 取消了。 也减去了 if， 分支预测。 应该是。


## 优化5，反向+跳过不满足的字符
提升5倍

```rust
fn simple(input: &[u8]) -> usize {
    let mut idx = 0;
    while let Some(slice) = input.get(idx..idx + 14) {
        let mut state = 0u32;
        if let Some(pos) = slice.iter().rposition(|byte| {      // 反向 遍历。
            let bit_idx = byte % 32;
            let ret = state & (1 << bit_idx) != 0;
            state |= 1 << bit_idx;
            ret
        }) {            // ? 这里是什么？。。 if let， 就是 match 的简化版， 所以这里是 匹配的， 代表 ret 返回了 true， 代表 出现了 重复
            idx += pos + 1;    // 出现了重复，那么 idx 直接跳 pos + 1 。 上面的 iter().rposition 估计是 ret 第一次 true，就会 break，把 下标 赋值给 pos， 所以 pos 拿到了 反序的 第一次 重复的 下标， 然后 下次 就从这个 下标+1 开始 遍历。
        } else {
            return Some(idx);
        }
    }
    return None;
}
```

这里也发生了 循环展开， 利用了 SIMD ， 优化4 也发生了 循环展开。 这些是在 compiler explorer 上看到的。


## 优化6, 多线程。。
。。这人 64线程。。。

800mb， 11.9ms处理完。




---



# 2024 c++ developer survey

bilibili的视频

原文也找到了：
https://isocpp.org/files/papers/CppDevSurvey-2024-summary.pdf

这个是在 isocpp 上的调查。 所以 要注意 目标人群。
一共 1200份

学C++的人 越来越少。

编程 >20年的有 40% 

行业 主流是： 游戏，硬件/物联网，开发者工具(IDE,编译器)，工程(航空，电源管理(?power management?))， 通讯软件(互联网，email)， 商业(b2b,b2e)


痛点
按多到少
- 包管理
- 构建的时间
- CICD
- 管理 CMake 工程
- 并发安全


如何管理库
按多到少
- 直接作为源码
- 单独编译库代码
- 系统包管理(apt,brew)
- 下载预构建的库
- Conan
- vckpg


使用的构建工具
- CMake
- Ninja
- Make/nmake
- MSBuild


哪些环节用到了 云
- CICD
- Test
- build
- 不用
- 部署



是否使用了 sanitizer 和/或 fuzzing
一半用，一半不用。
6% 的人 不知道。

## sanitizer & fuzzing

https://github.com/google/sanitizers

https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html
20-30个。。

---

- Address Sanitizer (ASan)：检测内存错误，如缓冲区溢出、使用已释放的内存等。使用时需在编译时添加`-fsanitize=address`选项。
- Undefined Behavior Sanitizer (UBSan)：检测代码中未定义行为（Undefined Behavior），例如整数溢出、空指针解引用等。使用时需在编译时添加`-fsanitize=undefined`选项。
- Thread Sanitizer (TSan)：检测并发问题，如数据竞争、死锁等。使用时需在编译时添加`-fsanitize=thread`选项。
- Memory Sanitizer (MSan)：检测对未初始化内存的访问以及类型不匹配的指针转换等问题。使用时需在编译时添加`-fsanitize=memory`选项。


---

https://github.com/google/fuzztest

测试框架


---
---





工作中用的版本
C++17 已经流行
。。。这里还列举了一些 特性。


## C++版本特性

- C++98/03
  exception, template, RTTI
- C++11
  auto, move, =delete/default, shared_ptr, lambda
- C++14
  generic lambda, auto return types, general constexpr function
- C++17
  if constexpr, if/switch scoped variables, structured binding, string_view, optional/any/variant, Parallel STL
- C++20
  concept, coroutine, module
- C++23
  expected, md_span




ide
- VS
- VSC
- Clion
- Vim
- emacs
- qt creator
- xcode
- sublime
- rider
- kate
- intellij IDEA
- Eclipse
- kDevelop
- android studio
- code::blocks
- webstorm
- codelite
- netbeans
- source insight
- atom
- code insight

。。上面是按比例的， 但是 有些人 根本不会选。

vsc，933人
vs, 669
vim，533
clion，331
qtcreator，281
xcode，205
emacs，165
eclipse，133
kdevelop，77
android studio，155






















