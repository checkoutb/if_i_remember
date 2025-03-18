Java
2021年8月25日
11:18

[[toc]]


package-info.java

****
https://docs.oracle.com/javase/specs/index.html

*****
https://docs.oracle.com/en/java/javase/17/

https://docs.oracle.com/en/java/javase/17/docs/api/index.html

https://docs.oracle.com/en/java/javase/17/docs/specs/security/standard-names.html

https://docs.oracle.com/en/java/javase/17/docs/specs/serialization/index.html

https://docs.oracle.com/en/java/javase/17/vm/java-virtual-machine-technology-overview.html

https://docs.oracle.com/en/java/javase/17/gctuning/introduction-garbage-collection-tuning.html

已使用 Microsoft OneNote 2016 创建。




# Java并发编程常识-梁飞
https://www.cnblogs.com/thisiswhy/p/14329726.html

写中间件经常要做的2件事：
- 延迟加载，在内存缓存已加载项
- 统计调用次数，拦截并发量

团队中十有八九是错的。所以写了 Java并发编程常识。

## JVM内存模型
- 堆
  - 所有对象全部放在共享堆空间中
  - 对象的属性在共享堆空间中
    - 堆内存单字节对齐，short不变
- 栈
  - 每个线程有独立的线程栈空间
  - 线程栈只存基本类型和对象地址
    - 栈内存4字节对齐，short变int
    - 对象地址4字节，引用堆空间
  - 方法中局部变量在线程栈空间内
    - 局部变量不会竞争，线程安全
    - 方法参数在栈顶交叉，不拷贝        。。交叉，看图
    - 栈顶寄存，减少中间状态读取
  - PC指针记录当前执行位置

-Xss1M  栈大小          。。 线程栈
-Xmx1g  堆大小



## 原子性
- 对象类型：
  - 对象地址原子读写，线程安全
  - 并发读 不可变状态，线程安全
  - 并发读写 可变状态，非线程安全
- 基本类型：
  - int,char数值读写，线程安全          。。？ 那为什么有 AtomicInt？ 应该是 byte是安全的，4个byte的int不安全吧。 但是 上面的 对象地址 原子读写。 对象地址就是 4byte的 指针吧。怎么安全。。。 或者说，这里是 一次性 读写4个byte的？ bus总线的关系？ 但是还是那句：为什么有 AtomicInt。。。
  - long,double 高低位，非线程安全
  - i++ 等组合操作，非线程安全

## 可见性
- final
  - 初始化final字段 确保可见性
- volatile
  - 读写 volatile字段 确保可见性
- synchronized
  - 同步块内读写字段确保可见性
- happen before
  - 遵守 happen before次序 可见性


## 可排序性
- happen before法则
  - 程序次序法则
    - 如果A一定在B之前发生，则 happen before
  - 监视器法则
    - 对一个 监视器的 解锁 一定发生在 后续对该监视器的 加锁之前
  - volatile变量法则
    - 对volatile变量的写 一定发生在 后续 对该变量的 读之前
  - 线程启动法则
    - Thread.start() 一定发生在线程中的 动作之前
  - 线程终结法则
    - 线程中的任何动作一定发生在括号中的动作之前 (其他线程检测到这个线程已经终止，从Thread.join调用中成功返回，Thread.isAlive()返回false)
  - 中断法则
    - 一个线程调用另一个线程的 interrupt 一定发生在 另一个线程发现中断之前
  - 终结法则
    - 一个对象的构造器结束 一定发生在对象的 finalizer之前
  - 传递性
    - A发生在B之前，B发生在C之前，A一定发生在C之前


## 系统内存
MESI协议
- Modified
  - 本CPU写，则直接写到Cache，不产生总线事务；
  - 其它CPU写，则不涉及本CPU的Cache
  - 其他CPU读，则本CPU需要把Cache line中的数据提供给它，而不是让它去读内存
- Exclusive
  - 只有本CPU有该内存的Cache，而且和 内存一致。
  - 本CPU的写操作会导致 转到 Modified 状态
- Shared
  - 多个CPU都对该内存有 Cache，而且内存一致。
  - 任何一个CPU写自己的这个Cache，都必须通知其他CPU
- Invalid
  - 一旦Cache line进入这个状态，CPU读数据就必须发出总线事务，从内存中读。


## 内存栅栏

![2414a00a6ca2d34b66de3bcfbe797c07.png](../_resources/2414a00a6ca2d34b66de3bcfbe797c07.png)

读
`volatile int a, b; if (a == 1 && b == 2)`
JIT通过 load acquire 依赖保证读顺序：
- 0x2000001de819c: adds r37=597,r36;; ;...84112554
- 0x2000001de81a0: ==la1.acq== r38=[r37];; ;...0b30014a a010

写
`volatile A a; a = new A();`
JIT通过 lock addl 使CPU的 cache line失效：
- 0x01a3de1d: movb $0x0,0x1104800(%esi);
- 0x01a3de24: ==lock addl== $0x0,(%esp);


## 查询JIT编译结果

`java -XX:+UnlockDiagnosticVMOptions -XX:PrintAssemblyOptions=hsdis-print-bytes -XX:CompileCommand=print, *AtomicInteger.incrementAndGet`

。。
https://blog.csdn.net/LNF568611/article/details/113716155

反汇编命令：
-XX:+UnlockDiagnosticVMOptions
-XX:+PrintAssembly -Xcomp
-XX:CompileCommand=print,*AtomicInteger.incrementAndGet

把这些参数设置到jvm启动参数，但一般首次执行会报错：
Java HotSpot™ 64-Bit Server VM warning: PrintAssembly is enabled; turning on DebugNonSafepoints to gain additional output

这是因为缺少hsdis-amd64.dylib导致的，把hsdis-amd64.dylib放到$JAVA_PATH/jre/lib/server/目录下即可：
。。


## 对齐
```Java
// LinkedTransferQueue
static final class PaddedAtomicReference<T> {
    Object p0,p1,p2,p3,p4,p5,p6,p7,p8,p9,pa,pb,pc,pd,pe;
    PaddedAtomicReference(T r) {
        super(r);
    }
}
```

16个地址的长度，正好占满==一个 cache line的长度。==
确保2个 引用，不再同一个 cache line上，防止 多锁竞争。

。。Common cache line sizes are 32, 64 and 128 bytes. 
。。bing的 feedback 中显示 64 bytes。

。。
缓冲行对齐，这个应该是一个需要掌握的知识点，属于关于程序优化的奇技淫巧。
偶现于面试环节。
对齐的目的是为了解决伪共享(False Sharing)的发生。
不知道伪共享的朋友，建议去了解一下。
PPT 中的例子的源码来源是：
com.google.code.yanf4j.util.LinkedTransferQueue.PaddedAtomicReference
。。

。。
在java8中，有更优雅的解决方案， @Contented 注解
。。


## 引用
```Java
private Channel channel;

public void setChannel(Channel channel) {
    this.channel = channel;
}

public void run() {
    Channel channel = this.channel;             // localed reference    。。此行标红
    if (channel != null && channel.isConnected()) {
        // do sth
    }
}

public void check() {
    if (channel != channel)                 // 此行标红
        throw new Error("check error!");
}
```

。。 感觉意思是，要使用的时候， 本地化， 防止并发 导致 中间状态？


## 单例
![5e32c5f13af9dd6ba32077b64aa1948e.png](../_resources/5e32c5f13af9dd6ba32077b64aa1948e.png)

。。基于类初始化的单例模式解决方案

## 多锁

![aaf9aa0ac49d318094ed74386fc2d9de.png](../_resources/aaf9aa0ac49d318094ed74386fc2d9de.png)

## 计数
![40e7710316bb6ee54bf6d2b0927b9436.png](../_resources/40e7710316bb6ee54bf6d2b0927b9436.png)

![819117719b4141e662ce07a0c9162cd2.png](../_resources/819117719b4141e662ce07a0c9162cd2.png)


## 缓存

![632d32e1be5d3f05ebda9bac14a1c65d.png](../_resources/632d32e1be5d3f05ebda9bac14a1c65d.png)

![69d5881352d0a4e0797dee48fd3556ce.png](../_resources/69d5881352d0a4e0797dee48fd3556ce.png)

## 线程安全策略

![fbe006b4f5cdd6af2ef85e6a9bfa3da8.png](../_resources/fbe006b4f5cdd6af2ef85e6a9bfa3da8.png)


## 习惯
输入每个 点号时：
- 会不会出现空指针
- 有没有异常抛出
- 是不是在热点区域
- 在哪个线程执行
- 有没有并发锁间隙
- 会不会并发修改不可见










=========================



=========================



=========================



=========================



=========================



# Graal VM

https://www.graalvm.org/latest/docs/


GraalVM 把你的java代码 编译为 可以独立运行的二进制文件。
比起在JVM上运行应用，这些二进制文件 更小，启动速度是100倍，不需要warmup就可以达到最高性能，使用更少的内存和CPU。

GraalVM减少的你的应用的攻击面(attack surface)。
从app的二进制文件中，排除了 未使用的class，method，field。
将反射和其他的动态特性 限制到 构建时，在运行时 不会加载任何未知代码。

Popular ==microservices frameworks== such as Spring Boot, Micronaut, Helidon, and Quarkus, and ==cloud platforms== such as Oracle Cloud Infrastructure, Amazon Web Services, Google Cloud Platform, and Microsoft Azure all support GraalVM.

通过优化和G1 gc， 和在JVM上运行对比，你可以获得 更低的延迟，相同甚至更好的峰值性能和吞吐量。

You can use the GraalVM JDK just like any other Java Development Kit in your IDE.



## native image
native image 是一项技术，用来 将 java代码编译为 二进制文件， native executable。
native executable 只包含了 运行时 需要的代码，即 app class, 标准库 class，运行时，从jdk静态链接的native code。


可以从以下东西中构建 native executable
- class 。。.class, not .java
- jar
- module 。。 java9的module










