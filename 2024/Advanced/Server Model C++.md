
2024-04-08 19:20

http://kegel.com/c10k.html

[[toc]]


---
---
---



---


# 多线程常用编程模型

https://zhuanlan.zhihu.com/p/392712498

- 每个请求创建一个线程，使用阻塞式IO操作(伸缩性不佳)
- 使用线程池，同样使用阻塞式IO操作
- 使用非阻塞IO + IO多了复用
- Leader/Follower


## one loop per thread

程序里的每个 IO 线程有一个 event loop (用作 IO 多路复用)，用于处理读写和定时时间。
event loop 代表了线程的主循环，需要让哪个线程干活，就把 timer 或 IO channel 注册到哪个线程的 loop 里即可。

## Leader/Follower

此模型会创建一个线程池，每个线程有三种状态：leading, following, processing。
Leader 线程负责监听请求，其他线程作为 follower 处于等待状态，当 leader 收到请求后，首先通知一个follower线程将其提拔为新的 leader，然后自己去处理这个请求，处理完毕后加入 follower 线程等待队列，等待下次成为 leader。
Leader/Follower 模式避免了线程动态创建和销毁的额外开销，将线程放在池中，无需交换数据，将上下文切换、同步、数据移动和动态内存管理的开销都降到了最低。


推荐的多线程编程模式：one loop per thread + 线程池。
event loop 用作 IO 多路复用，配合非阻塞 IO 和定时器；线程池用作计算，可以是任务队列或生产者消费者队列。

---
https://blog.csdn.net/woaiwojia6699/article/details/112731409

# 多线程服务器编程模型

客户端：socket、connnect、send、recv
服务端：socket、bind、listen、accept、recv、send、close
阻塞IO和非阻塞IO：没有数据的时候是不是立即返回

优化方案1：为每一条连接，创建一个线程。

优化方案2：reactor网络模型
1. 组成：非阻塞IO+IO多路复用(复用的网络线程)
  linux下的IO多路复用模型
  select：需要轮询检查每一个IO是否有事件。
  poll：将读、写、错误分成三种类型去检测
  epoll：将读、写、操作事件集中到一个阻塞中去完成，并且返回有事件的IO。
2. 特征：事件循环+事件驱动/回调
3. 缺点：容易分裂业务逻辑
4. 解决办法：协程：同步非阻塞

epoll_create, epoll_ctl, epoll_wait


one loop per thread 模型
每个线程都有一个循环
重点：
1. accept 一个线程
2. 读，写，错误 另起线程

监听线程 和 处理线程分开，每个线程创建一个 epoll，loop循环就是这个epoll

变种：one loop per thread + 队列 + 线程池 ， 会有一定的延时



---
https://blog.51cto.com/u_15346415/5172283

# Muduo之多线程服务器

使用最广泛的是 non-blocking IO + IO multiplexing, 即Reactor
程序的基本结构是 一个 事件循环 (event loop)，以 事件驱动 和 事件回调的方式 实现业务逻辑

## Reactor的优点
- 编程容易，效率也不错
- 不仅可以用于 读写 socket， 建立连接 (connect/accept)，甚至DNS解析 都可以用 非阻塞的方式进行，以提高并发度 和 吞吐量， 对IO密集型APP 也是不错的选择
- DNS解析也可以用 非阻塞方式进行，以提高 并发度 和 吞吐量

lighttpd 就是这样，它内部的fdevent 非常精妙，值得学习

## 基于事件驱动的编程模型的 缺点
- 要求事件回调函数必须是 非阻塞的
- 对于涉及网络IO的请求响应式协议，它容易割裂业务逻辑，使其分散在 多个回调函数中，不容易理解和维护
- 现代的编程语言有一些应对的方法(coroutine)

## 多线程服务器的编程模型
- 一个请求一个线程，使用阻塞式IO，伸缩性不佳
- 使用线程池，同样使用阻塞式IO操作
- 使用non-blocking IO + IO multiplexing，即java NIO 的方式
- Leader/Follower

默认情况下，一般使用第三种，即 non-blocking IO + one loop per thread。

## one loop per thread
- 这个模型下，程序里的每个IO线程 有一个 event loop (或者叫 Reactor)， 用于处理 读写 和 定时事件

好处
- 线程数目基本固定，可以在程序启动的时候设置，不会频繁创建和销毁
- 方便地在线程间调配负载
- IO事件发生的线程是固定的，同一个TCP链接不需要考虑事件并发

EventLoop代表了 线程的 主循环。需要让哪个线程干活，就把 timer 或 IO channel (如TCP连接)  注册到 那个线程的 loop中即可
- 对实时性有要求的 连接 可以单独用一个 线程
- 数据量大的 connection 可以独占一个线程，并把 数据处理任务分摊到 另外几个 计算线程中 (用线程池)
- 其他次要的辅助性 连接 可以共享一个线程

对于 non-trivial 的服务端程序，一般会采用 non-blocking IO + IO multiplexing， 每个 connection/acceptor 都会注册到 某个 event loop 上，程序里有 多个 eventloop, 每个线程最多一个  event loop

多线程程序 对 event loop 提出了更高的要求，那就是 "线程安全"
- 要允许一个线程往 别的线程的 loop 里 赛东西 ( 比如 主IO线程 收到一个 新建的连接，分配给某个子 IO线程处理)，这个loop 必须是 线程安全的

对于没有IO，只有计算任务的线程，使用 event loop 有点浪费，有一种补充方案，即 blocking queue
muduo 的 `BlockingQueue<T>` 。。 这个不对的， muduo 是C++的， `BlockingQueue<T>` 的 命名和 范型 是 java的。

muduo 推荐的C++多线程服务端 编程模式为 one (event) loop per thread + thread pool
- event loop (也叫 IO loop) 用作 IO multiplexing，配合 non-blocking IO 和 定时器
- thread pool 用来作计算，具体可以是 任务队列 或 生产者消费者队列。
- 具体使用 几个 loop，线程池的大小，应该根据app来设定，基本原则是 "阻抗匹配"，使得 CPU 和 IO 都能高效运作
- app中还有 个别特殊任务的线程，比如 logging，在分配资源时 需要算进去。

。。看这里，似乎 one loop per thread 就是 1:N:M 啊， 一个线程监听端口，有事件后，发给 N 个 event loop 的线程， event loop 线程通过 任务队列 / 消息队列 发送给 包含M根线程的 thread pool 进行 业务计算 ？



---
https://blog.csdn.net/m0_53485135/article/details/134007819

高性能的网络程序中，使用最广泛的是 `non-blocking IO + IO multiplexing`，即Reactor
- lighttpd
- libevent,libev
- ACE, Poco C++ libraries
- Java NIO, Apache Mina, Netty
- POE (Perl)
- Twisted (Python)

Boost.Asio, Windows IO Completion Ports 实现了 Proactor，应用面 窄一些。
ACE 也实现了 Proactor 

在 non-blocking IO + IO multiplexing 这种模型中，程序的 基本结构是 一个 事件循环 (event loop), 以事件驱动 (event-driven) 和 事件回调的方式 实现 业务逻辑。

。。这篇 和上一篇 一样的。这篇是 2023, 上一篇是 2022,  但是，但是，但是， 这篇 说了 "以事件驱动" ，上一篇 只有 "以事件驱" ， 少一个动了，所以上一篇 ，我只抄了 "事件回调的方式"
。。感觉还有一篇 应该在更早

。。https://www.cnblogs.com/ljygoodgoodstudydaydayup/p/5749880.html
。。16年的。。但是这里内容 很少很少。 我百度的是 "程序的基本结构是一个事件循环"
。。这篇也少东西。。 Reactor的优点的 第二条，很明显少了。
。。这些文字应该是 《Linux多线程服务端编程》 书上的。

。。和上一篇 补一起了。



---

https://www.jianshu.com/p/46d1c2830ff3

# 多线程服务器的常见编程模型

## IO 模型
IO模型分为 同步IO，异步IO
同步IO分为 Blocking IO, Non-Blocking IO, IO multiplexing

Blocking IO, 发送端send发送缓冲区满 或 接收端recv 接收缓冲区满 会阻塞当前线程
Non-Blocking IO，send, recv 直接返回错误 (需要轮询)
==IO Multiplexing，通过 select/poll/epoll 等 系统调用 监控多个连接上的 读写事件==

## 线程模型
Blocking IO + one thread per connection
acceptor 创建新线程

Blocking IO + thread pool
acceptor 转发给 线程池

IO multiplexing + thread pool
acceptor -> event queue -> pooler(select/poll/epoll) -> worker(one of thread pool)
。。这个才是 1:M:N 吧？

IO multiplexing + one thread one loop
线程数目基本固定
高并发


one thread one loop思想
一个线程 一个 事件循环流程

```C++
void CReactor::Run()
{
    //线程退出标志
    m_bShouldRun = true;
    while(m_bShouldRun)
    {
        //处理其他的任务
        handle_other_task();
        //利用select/pool/epoll等I/O多路复用监听各个连接（fd)的读写事件
        select_or_epoll_function();
        //处理各个连接(fd)的读写事件
        handle_io_event();
        //处理定时器
        check_timer();
        //处理同步或者异步事件
        dispatch_event();
    }
}
```

线程分为 AcceptReactor, TresultReactor, FrontReactor
AcceptReactor
- 监听是否有新的客户端连接 (listenfd 上是否有 读事件 发生)
- 有新的连接请求，则生成 新连接的 socket，并将 该socket 传递给 frontReactor

TresultReactor
- 订阅 tserver 发布的流水，接收到流水中的 XTP报文，保存到 TradeResult 中
。。？这个是什么？ 和 现在的 似乎没有任何关系。

FrontReactor
- 通过 epoll 或 select 监听 该线程 所负责的 socket 上的 读写事件
- 收包 (收网络数据，放缓冲区，解包并处理)，发包
- 从 TradeResult 中读取 tserver 发布的 XTP 报文，并处理









