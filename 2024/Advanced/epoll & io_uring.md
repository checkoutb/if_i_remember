epoll
2023年4月8日
9:45

select，pselect，pool，ppoll

[[toc]]


------------------------

# Linux epoll (bing)

。。bing 搜索 linux epoll， bing给了一段介绍，整合自 wiki，man7，等网站。

epoll 是 linux kernel 的一个 scalable的 IO event notification 机制。
它可以监控 一个 file descriptor，来 查看 是否可以对其中一个fd 进行 IO 操作。
它是为了 ==代替 老的 POSIX **select 和 poll** 系统调用==， 并有更好的性能。
epoll 操作是 O(1) 时间 的，老的 系统调用 是 O(n) 的。
epoll API 即可以被用作 edge-triggered 接口，也可以被用作 level-triggered 接口，可以很容易地 scale成 监控大量 file descriptor。
epoll 在 linux 2.6 中引入， 其他 类UNIX 系统没有这个API。
epoll 是一个 blocking operation，你会 block 线程 直到 一些事件发生，然后你 分发 event 到 你的代码的 不同的 步骤/函数/分支。


------------------------

# epoll 4
https://linux.die.net/man/4/epoll
。在man7.org 中只有 epoll(7)， 没有4。。
。。网上 没有找到 epoll 4 和 epoll 7 的具体概念，只有epoll，没有 epoll 4/7，不知道这里4 和7 代表什么。。

## name
epoll: I/O event notification facility

## synopsis
`#include <sys/epoll.h>`

## description
epoll 是 poll(2) 的变种，用于 edge 或 level triggered interface，可以 监控大量 fds。
提供了 3个 系统调用，来 设置 和 控制 epoll set：epoll_create(2), epoll_ctl(2), epoll_wait(2)

epoll set 被连接到 epoll_create 创建的 file descriptor 上。 然后通过 epoll_ctl 注册 感兴趣的 file descriptor。最后，epoll_wait 启动 等待。

## notes
epoll 事件分发接口 (event distribution interface )  能表现为 edge triggered (ET) 和 level triggered (LT)。
ET 和 LT 事件分发机制的 不同如下：
假设发生这种情况
1. 代表了 pipe 的 读侧 的 file descriptor (RFD) 被添加到 epoll 装置。
2. pipe writer 在 pipe 的写侧 写入 2kb 数据
3. 执行 完成 一次 epoll_wait，它会返回 RFD，作为 ready file descriptor。
4. pipe reader 从 RFD 中读取 1kb 数据
5. 执行完成一次 epoll_wait

如果 RFD file descriptor 已经被添加到 使用 EPOLL==ET== 标记的 epoll 接口，第 5 步 完成的 epoll_wait 调用 可能会 挂起，因为 在文件的 input buffer 中依然有 可用数据，且 远程 peer 可能 在等待 针对它刚发送的数据的 响应。这样做的原因是：只有当 event 在 监控的file 中发生时， edge triggered event distribution 才会提供 event。所以，在 第 5 步中，caller 可能 最终 会等待 已经在 input buffer 中的 数据。
在上面的例子中， RFD 上的 event 会 因为 第 2 步 的 write 行为 而被创建，这个 event 在 第 3 步 中被 消费掉。由于 第4 步 的 read 操作 没有消费 整个 buffer 数据，所以 第 5步 中的 epoll_wait 可能 永远 锁住。
使用 EPOLLET 标记 的 epoll 接口， 应该使用 非阻塞的 file descriptor 来避免 阻塞read 或 write，处理 多个 fd 的任务 会由于 阻塞IO 而 饥饿。

以下是 使用 epoll 作为 edge triggered (EPOLLET) 时的 推荐用法，避免可能的陷阱
1. 使用 非阻塞 fd
2. 只有在 read  或 write 返回 EAGAIN 后 去等待 event

当用作 level triggered interface 时， epoll 是一个 更快的 poll，能在 任何使用 poll 的地方使用 epoll， 因为它们的==语义相同==。
由于 即使使用 edge triggered epoll，在收到 数据的 多个chunk 时，(。。也) 会 生成 多个event，(所以) caller 可以指定 EPOLLONESHOT标记，来告诉 epoll 在收到 epoll_wait 的 一个 event 后 禁用 关联的 fd。 当 EPOLLONWSHOT 被指定时，caller 需要 使用 具有 EPOLL_CTL_MOD 的 epoll_ctl 来 re-arm fd


## Example for Suggested Usage
level triggered interface 的 epoll 有着 和 poll 相同的语义。
edge triggered 的 epoll 需要更多的 清晰性 来避免 event loop 中的 停滞。

在这个例子中，listener 是一个 非阻塞的socket，在这个 socket上 已经调用了 listen。
do_use_fd() 使用 new ready fd，直到 read 或 write 返回 EAGAIN。
一个 事件驱动的 状态机 app 应该 在 收到 EAGAIN后， 记录当前的状态， 下次调用 do_use_fd 时，它 从 之前停止的地方 继续 read  或 write。

``` C++
struct epoll_event ev, *events;
for(;;) {
    nfds = epoll_wait(kdpfd, events, maxevents, -1);
    for(n = 0; n < nfds; ++n) {
        if(events[n].data.fd == listener) {
            client = accept(listener, (struct sockaddr *) &local,
                            &addrlen);
            if(client < 0){
                perror("accept");
                continue;
            }
            setnonblocking(client);
            ev.events = EPOLLIN | EPOLLET;
            ev.data.fd = client;
            if (epoll_ctl(kdpfd, EPOLL_CTL_ADD, client, &ev) < 0) {
                fprintf(stderr, "epoll set insertion error: fd=%d0,
                        client);
                return -1;
            }
        }
        else
            do_use_fd(events[n].data.fd);
    }
}
```

当用作 edge triggered interface， 由于 性能因素，可以 指定 ( EPOLLIN | EPOLLOUT) 在 epoll 接口中 (EPOLL_CTL_ADD) 添加 fd。 这可以 避免 在使用 EPOLL_CTL_MOD 调用 epoll_ctl 时， 在 EPOLLIN 和 EPOLLOUT 之间 切换。

## Q & A (from linux-kernel)
- 重复添加 fd 到 epoll_set会发生什么？
	- 你可能获得 EEXIST。但是 也可能 2根线程 add 同一个fd 2次。这是一个 无害的condition
- 2 个epoll set 可以等待 同一个 fd 吗？如果可以，那么 event 会 被 reposted 到 2个 epoll set？
	- 是的，但这不推荐。 是的，会 同时 report 给 2个 epoll set。
- epoll fd 它自己是 poll / epoll / selectable ?
	- yes
- 如果 epoll fd 被 添加到 它自己的 fd set， 会发生什么？
	- 会失败。 但 你可以 添加 epoll fd  到另一个 epoll fd set 中
- 可以通过 unix socket 发送 epoll fd 到 另一个 进程？
	- no
- fd close时，会自动 从 所有 epoll set 中 移除吗？
	- yes
- 在 epoll_wait 调用中，有 多个事件 到来， 它们会被组合起来， 还是 单独 report？
	- 它们会被组合起来
- 在 fd 上的操作 会 影响 已经被收集，但是还没有被 report 的event 吗？
	- 在已经存在的 fd 上，你可以做 2 件事情，remove 是毫无意义的，modify 会 重新读取 可用的 IO
- 当使用 EPOLLET 时，我必须 不断地 read / write 一个 fd 直到 EAGAIN 吗？
	- 不，Receiving an event from epoll_wait(2) should suggest to you that such file descriptor is ready for the requested I/O operation. You have simply to consider it ready until you will receive the next EAGAIN. When and how you will use such file descriptor is entirely up to you. Also, the condition that the read/write I/O space is exhausted can be detected by checking the amount of data read/write from/to the target file descriptor. For example, if you call read(2) by asking to read a certain amount of data and read(2) returns a lower number of bytes, you can be sure to have exhausted the read I/O space for such file descriptor. Same is valid when writing using the write(2) function.



## Possible Pitfalls and Ways to Avoid Them
### Starvation ( Edge Triggered ) 饥饿
If there is a large amount of I/O space, it is possible that by trying to drain it the other files will not get processed causing starvation. This is not specific to epoll.

The solution is to maintain a ready list and mark the file descriptor as ready in its associated data structure, thereby allowing the application to remember which files need to be processed but still round robin amongst all the ready files. This also supports ignoring subsequent events you receive for fd's that are already ready. 

### If using an event cache...
If you use an event cache or store all the fd's returned from epoll_wait(2), then make sure to provide a way to mark its closure dynamically (ie- caused by a previous event's processing). Suppose you receive 100 events from epoll_wait(2), and in eventi #47 a condition causes event #13 to be closed. If you remove the structure and close() the fd for event #13, then your event cache might still say there are events waiting for that fd causing confusion.

One solution for this is to call, during the processing of event 47, epoll_ctl(EPOLL_CTL_DEL) to delete fd 13 and close(), then mark its associated data structure as removed and link it to a cleanup list. If you find another event for fd 13 in your batch processing, you will discover the fd had been previously removed and there will be no confusion.

## Conforming to
epoll(4) is a new API introduced in Linux kernel 2.5.44. Its interface should be finalized in Linux kernel 2.5.66. 





----------------------


# epoll(7) - Linux man page

https://linux.die.net/man/7/epoll

## Name
epoll - I/O event notification facility

## Synopsis
`#include <sys/epoll.h>`

## Description
epoll API 的功能类似 poll， 监控多个 fd 来查看 是否 发生IO。
可以 用作 edge triggered interface， 或 level triggered interface。可以监控 很多 fd。

### 下面的 system call 用来 创建和 管理 epoll 实例
- epoll_create 创建一个 epoll 实例，并返回 指向那个实例的 fd。( 更新的 epoll_create1 扩展了 epoll_create 的功能 )
- 然后 通过 epoll_ctl 注册 感兴趣的 fd。当前注册在 epoll 实例上的 fd 集合 有时被称为 epoll set
- epoll_wait 等待 IO事件，如果当前没有 event 可用，则阻塞 calling线程。

### Level-triggered and edge-triggered

。。这个 epoll7 和 epoll4 差不多一样。都跳了
。。除了下面的，epoll7 就多了下面的， 还有就是 example 更复杂。

### /proc interfaces
下面的接口可以用来 限制 epoll 消费的 kernel 内存 上限：
`/proc/sys/fs/epoll/max_user_watches (since Linux 2.6.28) `

这个指定了 用户可以在系统的所有epoll中 注册的 fd 的最大数量总和。
这个限制是 对 每个user ID 的。
每个注册的 fd 在 32位kernel上消耗 90byte，在64位kernel上消耗 160 byte
目前，max_user_watches 的默认值是 available low memory 的 1/25，除以 注册消耗 ( 即，32位90byte，64位160byte)







-------------

https://www.cnblogs.com/zhaoosheLBJ/p/9268532.html

# epoll,lt,et

epoll 是 select/poll的 加强版，即 enhancement poll。
epoll模型可以显著提高 程序 在 大量并发连接中 只有少量活跃CPU 系统的 CPU利用率。
它把 用户关系的 文件描述符 上的时间放在内核的 一个事件表中，无需像 select 和 poll 那样 每次调用 都 重复传入 文件描述符集。
在获取事件的时候，无需遍历整个被监听的文件描述符集，而是遍历那些被 内核IO 事件 异步唤醒 而加入ready队列的 fd集合。
所以 epoll 是linux 大规模高并发网络程序的 首选模型。

## epoll api
### epoll_create / epoll_create1

```C++
#include <sys/epoll.h>

int epoll_create(int size);     // since 2.6
        // create a new epoll instance
        // 参数被忽略了，但是必须大于0

int epoll_create1(int flags);   // since Linux 2.6.27
        // 返回 指向 new epoll instance 的 file descriptor
        // 这个fd 被后续的所有 epoll 接口用到。
        // 如果 flags 是0， 和 epoll_create 一模一样。
        // 下面的值可以用在 flags，获得不同的行为
        //      EPOLL_CLOEXEC：在新的fd上设置 close-on-exec flag。这个等同于 open() 的 O_CLOEXEC标记。

```

创建一个epoll句柄(epoll特有)，用来唯一标识内核中 这个事件表，这个特有的epoll 文件描述符 将是 其他所有epoll 系统调用的 第一个参数 (epollfd)，以指定要访问的内核事件表。


### epoll_ctl
```C++
#include <sys/epoll.h>

int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
// 成功返回0，出错返回-1
```

- epfd, 就是 epoll_create创建的 epoll句柄，唯一
- op：3种操作：
  - EPOLL_CTL_ADD：向epfd注册fd上的event
  - EPOLL_CTL_MOD：修改fd已注册的event
  - EPOLL_CTL_DEL：从epfd上删除 fd的event
- fd：操作的文件描述符
- event：指定内核要监听的事件，它是 struct epoll_event 的指针

```C++
struct epoll_event {
    uint32_t  　　events; /* Epoll events */ 
    epoll_data_t　data;　 /* User data variable */        
};
```
    - events成员: 描述了事件类型，将以下宏定义 通过 bit& 组合：
      - EPOLLIN：表示对应的fd 可以读(包括对端socket正常关闭)
      - POLLOUT：fd 可以写
      - EPOLLPRI：fd 有紧急的数据可读(这里应该表示有带外数据到来)
      - EPOLLERR：fd 发生错误
      - EPOLLHUP：fd 被挂断
      - EPOLLET：将epoll设置为 edge triggered
      - EPOLLONESHOT：只监听一次事件，监听完这次事件后，如果还需要 继续监听这个socket的话，需要再次把这个socket加入到epoll队列中。

    - data：用户数据遍历，用于存储用户数据，是 epoll_data_t 结构类型。 epoll_data_t 是一个联合体，fd指定事件从属 目标文件描述符，ptr可以用来指定 fd相关的用户数据，但两者不能同时使用

```C++
typedef union epoll_data {
　　void 　　　　*ptr;
　　int 　　　　　fd;
　　uint32_t　　 u32;
　　uint64_t 　　u64;
} epoll_data_t;
```
。。。union。。 一个时间点，只能有一个值。 占用的内存是 最大的成员。

### epoll_wait
等待 监听的fd上的 事件发生
```C++
#include <sys/epoll.h>

int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
// 成功返回就绪的文件描述符个数，若出错返回 -1，超时返回0
```

- epfd： epoll_create 创建的句柄，唯一
- events： 是一个 传入传出函数， 是一个 epoll_event 结构指针，用来从 内核 获得 事件集合
- maxevents: 告知内核 events 的大小，但不能大于 epoll_create() 时创建的size。
- timeout： -1 为阻塞，0位立即返回， 大于0 是等待多少 微秒。

。。size 不是废弃的吗。


## LT ET

### level triggered 水平触发
当被监控的文件描述符上有可读写事件发生时，epoll_wait()会通知处理程序去读写。
如果这次没有把数据一次性全部读写完(如读写缓冲区太小)，    
那么下次调用epoll_wait()时，它还会通知你在上没读写完的文件描述符上继续读写，
当然如果你一直不去读写，它会一直通知你！！！
如果系统中有大量你不需要读写的就绪文件描述符，  而它们每次都会返回
这样会大大降低处理程序检索自己关心的就绪文件描述符的效率！！！

### edge triggered 边缘触发
当被监控的文件描述符上有可读写事件发生时，epoll_wait()会通知处理程序去读写。
如果这次没有把数据全部读写完(如读写缓冲区太小)，
那么下次调用epoll_wait()时，它不会通知你
也就是它只会通知你一次，直到该文件描述符上出现第二次可读写事件才会通知你 
这种模式比水平触发效率高  
系统不会充斥大量你不关心的就绪文件描述符！！！

### 阻塞IO
当你去读一个阻塞的文件描述符时，如果在该文件描述符上没有数据可读，那么它会一直阻塞(通俗一点就是一直卡在调用函数那里)，直到有数据可读。
当你去写一个阻塞的文件描述符时，如果在该文件描述符上没有空间(通常是缓冲区)可写，那么它会一直阻塞，直到有空间可写。
以上的读和写我们统一指在某个文件描述符进行的操作，不单单指真正的读数据，写数据，还包括接收连接accept()，发起连接connect()等操作...


### 非阻塞IO
当你去读写一个非阻塞的文件描述符时，不管可不可以读写，它都会立即返回， 
返回成功说明读写操作完成了，返回失败会设置相应errno状态码，根据这个errno可以进一步执行其他处理它不会像阻塞IO那样，卡在那里不动！！！



## 几种IO模型的触发方式
select,poll 模型都是 LT。
信号驱动IO 是 ET。
epoll 支持LT，ET， 默认LT


## demo
这里讨论 epoll的 LT 和ET，以及阻塞IO 和非阻塞IO 对它们的影响。
对应监听的socket 文件描述符用 sockfd
对于 accept返回的 文件描述符 (即要读写的 文件描述符) 用 connfd 表示。

场景
- 水平触发的非阻塞 sockfd
- ET的非阻塞 sockfd
- LT的阻塞 connfd
- LT的 非阻塞 connfd
- ET的 阻塞 connfd
- ET的 非阻塞 connfd

以上没有验证 阻塞的 sockfd，因为 epoll_wait() 返回 必定是 已就绪的连接，设不设阻塞， accept()都会立即返回。

例外：UNP里面有个例子，在BSD上，使用select()模型。设置阻塞的监听sockfd时， 
当客户端发起连接请求，由于服务器繁忙没有来得及accept()，此时客户端自己又断开，当服务器到达accept()时，会出现阻塞。
本机测试epoll()模型没有出现这种情况，我们就暂且忽略这种情况！！！


。。代码有点多。。

https://www.cnblogs.com/zhaoosheLBJ/p/9268532.html



------------------------



# io_uring

https://zhuanlan.zhihu.com/p/636097492

2019年5.1引入

一个使用简单、兼容性强的异步IO框架


io_uring和eBPF这两个子系统经常一起被提及，并称为改变Linux底层的两个技术革命，因为它们都改变了用户空间应用程序与 Linux 内核交互的方式
这2者属于不同子系统，并有一个本质区别
- eBPF对于用户来说是看不到的，只需要升级内核，app无需任何改造。
- io_uring 提供了 新的系统调用 和 用户空间交互的API，需要修改 app

u是指 user
ring 是指 环形队列

通俗来说，就是==将 io 放到 user 可以管理的 一个 环形队列中==

## 异步IO发展史

### 基于fd的阻塞式io

```C
size_t read(int fd, void *buf, size_t count);
size_t write(int fd, const void *buf, size_t count);
```

阻塞式 系统调用。

### 非阻塞式IO: select/poll/epoll

不会阻塞，立刻返回，可以获得已经ready的 fd列表

致命缺点：只支持 network socket，pipe， 不支持文件的读取。

epoll是用于管理 读和写的，用来检测 io 是否可读写，但是并没有对 io 进行操作。检查完后，还需要用 阻塞的 read/write 来进行读写。


### 线程池方式

主线程将 io 分发给worker 线程， worker线程进行阻塞式读写， 主线程不会阻塞。

线程上下文切换 开销可能很大


### Direct IO (数据库软件)

有时候并不想使用操作系统的page cache，提交给内核处理。内核会通知IO设备独立进行操作

- 需要指定 O_DIRECT
- app自己管理缓存
- 是 0拷贝 io，应用的缓冲数据 直接发送到 设备，或直接从 设备读取


### 异步IO (AIO)

现在市场上的一些设备，把读取文件的异步操作提交给内核处理。内核会通知IO设备独立进行操作。

Linux 原生AIO 处理流程
- app 调用 io_submit 系统调用 发起一个 异步IO 后，会向内核的IO任务队列中添加一个 IO 任务，并返回成功
- 内核会在 后台处理 IO任务队列中的 IO，然后把 处理结果存到 IO任务中
- app 调用 io_getevents 系统调用 来获取 异步IO的处理结果，如果还没有完成，就返回失败信息，否则返回 IO处理结果

从上面的流程可以看出，Linux的异步IO主要分为2个步骤
- 调用 io_submit 发起一个 异步IO操作
- 调用 io_getevents 获取异步IO的结果

但还存在问题
- 仅支持 direct IO，无法借助 文件系统缓存 来缓存当前 IO请求，还有 size对齐。
- 依然可能被阻塞。
- 系统调用 开销大
- 数据拷贝 开销大
- API不友好，一个IO 至少需要2次 系统调用
- IOPOLL 支持不好。

按io完成方式，分为2种
- 基于中断进行通知
- 用户轮询IO事件，这就是 IOPOLL。 适合于 高速设备。


## io_uring 原理和接口

### 基本原理

io_uring 为了尽量减少系统调用的发生，采用了用户态与==内核==态“==共享内存==”的方式来通信。
用户进程可以向共享内存提交要发起的I/O操作，而内核线程可以从共享内存中读取I/O 操作，并且进行相关的I/O操作。
用户态对共享内存进行读写操作是不需要使用系统调用的，所以==不会发生上下文切换==的情况。

3个问题
- 用户空间 和 内核空间 应该如何实现 对共享内存的管理？
- IO的请求 及 完成后，用什么样的数据结构传输？
- 如果完成 iopoll，以及负责的io操作？


### 基本概念

用户 和 内核 通过提交队列 和 完成队列 来进行 IO 任务的 提交 和 交割。

- SQ，submission queue，提交队列，一整块连续的 内存空间存储的 环形队列。用于存放 执行的 操作数据项
- CQ，completion queue，完成队列，一整块连续的 内存空间存储的 环形队列。用于存放执行完成后的结果。
- SQE，submission queue entry，提交队列项，存储在 SQ中的数据项
- CQE，completion queue entry，完成队列项，存储在 CQ中的数据项
- Ring，环，SQ和CQ 都是 环形队列结构，Ring用来代表一个 io_uring 的实体。包括 队列数据，队列大小，丢失项等信息


### 提交队列和完成队列

io_uring 通过用户态与内核态共享内存的方式，来免去了使用系统调用发起 I/O 操作的过程。io_uring 主要创建了 3 块共享内存：SQ、CQ和SQE。

![1ec35aa5a31075c5a9b121989f9a1cec.png](../_resources/1ec35aa5a31075c5a9b121989f9a1cec.png)

SQ和CQ 中每个节点 保存的都是 SQEs 数组的 索 引。
提交的时候，可以提交 SQE 数组上 不连续的 请求。

#### 提交队列SQ

内核中，用 io_sq_ring 来表示 提交队列

```C
struct io_sq_ring {
    struct io_uring {
        u32 head;//环形队列的头指针
        u32 tail;//环形队列的尾指针
    } r; // 使用head和tail指针来模拟环形操作
    ...
    u32 ring_entries; // 队列中的提交项总数
    ...
    u32 flags;
    u32 array[]; // 环形队列数组（指向提交队列项数组的索引）
};
```

io_sq_ring 结构 array 字段只是一个整形类型的数组，用于存储指向 提交队列项数组 的的索引

#### SQE

在内核中，提交队列项 使用 io_uring_sqe 结构表示

```C
struct io_uring_sqe {
    __u8 opcode; //I/O 操作码，主要用于表示当前的 I/O 操作是什么类型，如读、写或者同步等。

    __u8 flags;// 包含了跨命令类型的常见修饰符标志
    __u16 ioprio; //：I/O 操作的优先级，可以通过此字段来把一些重要的 I/O 操作提前执行。
    __s32 fd; //I/O 操作对应的文件句柄。
    __u64 off; //当前 I/O 操作的偏移量。
    __u64 addr; //用于指向当前 I/O 操作所关联的内存地址。如写操作，指向的是要写入到文件的内容的内存地址。
    __u32 len; //表示当前 I/O 操作的数据长度
    union {
        __kernel_rwf_t rw_flags;
        __u32 fsync_flags;
        __u16 poll_events;
        __u32 sync_range_flags;
        __u32 msg_flags;
    };
    __u64 user_data;//可以适用于所有操作码，并且不会被内核修改。当该请求的完成事件发布时（请求完成），复制到完成事件 cqe 中
    union {
        __u16 buf_index;
        __u64 __pad2[3];
    };
};
```

io_uring_setup() 系统调用，创建一个 io_ring 对象时，内核会创建一个类型为 io_uring_sqe 结构的数组。
内核会会将此数组映射到应用程序的内存空间，这样应用程序就可以直接操作这个数组。

应用程序提交 I/O 操作时，先要从提交队列项数组中获取一个空闲的项，然后向此项填充数据（如 I/O 操作码、要进行 I/O 操作的文件句柄等），然后将此项在提交队列项数组的索引写入提交队列中。


#### 完成队列

当内核完成 I/O 操作后，会将 I/O 操作的结果保存到完成队列中。
与提交队列相比，完成队列的结构要简单很多，因为不需要像IO提交那样，携带较多的IO描述性信息。
内核使用 io_cq_ring 结构来表示

```C

struct io_cq_ring {
    struct io_uring {
        u32 head; //环形队列的头指针。
        u32 tail; //环形队列的尾指针。
    };
...
    u32 ring_entries; //已完成的 I/O 操作总数。

...
    struct io_uring_cqe cqes[]; //用于保存 I/O 操作结果的环形队列数组，其元素类型为 io_uring_cqe 结构。
    // 这里没有额外开辟空间，和io的操作逻辑也有关系；io的执行顺序是无法控制的；但是读取已完成的io，一般都是按序的；
};

struct io_uring_cqe {
    __u64 user_data; // 指向 I/O 操作返回的数据
    __s32 res; // I/O 操作的结果
...
};
```

内核也会将完成队列映射到应用程序的内存空间，这样应用程序就可以通过读取完成队列来获取 I/O 操作的结果。
而不用通过使用系统调用来获取，从而避免了不必要的上下文切换。


#### SQ线程
app 将 IO操作 提交到 提交队列后， 内核什么时候 从 提交队列 中获取要进行的 IO操作，并发起IO请求呢？

当用户使用 SQPOLL 模式 (指定了 IORING_SETUP_SQPOLL 标志) 创建io_uring时，内核将会 创建一个 名为 io_uring-sq 的内核线程， 这个内核线程 会不断从 提交队列中读取 IO操作，并发起 IO请求。


![761291e78e25c069596f3124e1faa12b.png](../_resources/761291e78e25c069596f3124e1faa12b.png)

- app 通过向 io_uring 的 提交队列 提交 IO操作
- SQ内核线程 从 提交队列 读取IO操作
- SQ内核线程 发起 IO请求
- IO请求完成后，SQ内核线程会 将IO请求的结果 写入到 io_uring 的完成队列中
- app 从 完成队列中读取到 IO操作的结果



### io_uring 接口

io_uring 提供了 3个系统调用
- io_uring_setup，建立一个 io_uring 实体，初始化环境和上下文
- io_uring_enter，将 SQE 提交到 SQ中； 或从 CQ中 提取已处理好的 IO
- io_uring_register，用于 预注册读写文件 等，提供一些高级功能


io_uring_setup 是对 io_uring_create 的封装。
创建一个 上下文结构 io_ring_ctx 来管理 整个会话，随后 创建 SQ 和 CQ 内存区，并将 2个偏移带给外层

CQ 默认长度是 SQ 的2倍， 因为 SQ中的 SQE 一旦被 内核发现，就会被消耗掉，SQE的生命周期很短。  请求的完成事件放到 CQ环中。 内核会在 内部 存储 溢出的 事件，直到CQ 有空闲空间。


io_uring_enter 主要4步
- 将IO请求交给 SQE
- 通知内核 消费 SQE
- 内核完成 SQE后，写入CQ
- 用户获取 CQE


io_uring_register
开启一些高级功能
- IORING_SETUP_SQPOLL
  ==创建==一个==内核线程==，主动进行sqe的处理（io提交）。
  该功能几乎完全能消除了内核态上下文的切换开销，并且真正将IO逻辑offload，实现业务逻辑与IO逻辑的分离。
- IORING_SETUP_IOPOLL
  对于==非常快的设备==（如现在常用的NVMe SSD），处理IO完成时的中断通知开销是比较大的。
  该功能允许用户关闭中断，通过轮询来实现对已完成的IO消息捕捉。（当开启SQPOLL特性时，该功能由sqthread同时负责）；
- IORING_REGISTER_FILES & IORING_REGISTER_BUFFERS
  每次发起对一个指定文件的操作时，内核都需要花费一些时钟周期将文件描述符映射到内核中，对于那些会针对同一个文件进行重复操作的场景，io_uring支持==提前注册这些文件==，后面直接使用文件就可以了；
  与文件注册类型，direct IO的场景中，内核需要map/unmap memory areas，io_uring支持==提前注册这些缓冲区==。SQEs也是通过register_buffer，放到page中的
- IORING_FEAT_FAST_POLL
  传统的epoll流程中，会等到网络包到了之后，再去通知用户读取数据。
  FAST_POLL可以实现，当socketfd有数据的时候， 会==直接从内核空间中拿到数据==（可以理解为，在==内核态直接下发read操作==，该新特性对传统的epoll有较大的冲击）；
- IOSQE_IO_LINK
  ==通常== sqe 会被单独的==异步执行==。也就是说一个sqe的执行，并不会影响环中后续sqe的执行和顺序，这使操作具有充分的灵活性，并且能够并行的执行和完成，以获得最大效率和性能。
  但有些场景，我们==对io的执行顺序有一定要求==。



### liburing库介绍及使用

由于io_uring的三个系统调用接口过于精简，需要业务做很多处理，还要手动建立用户态和内核态的队列映射关系，以及调整各种输入参数。
liburing 提供了一个简单的高层 API， 可用于一些基本场景，应用程序避免了直接使用更底层的系统调用。 
此外，这个 API 还避免了一些重复操作的代码，如设置 io_uring 实例。




# archlinux's io_uring man

https://man.archlinux.org/man/io_uring_setup.2.en


https://man.archlinux.org/man/io_uring.7

sq: submission queue
cq: completion queue
sqe: sq entry
cqe: cq entry

1. io_uring_setup; mmap映射 SQ 和 CQ 到 用户空间 shared buffer

2. 当你要执行IO (读文件，写文件，socket操作) 时， 创建一个 SQE 来描述 IO操作， 把这个SQE 添加到 sq tail 。

3. 添加完后，调用 io_uring_enter 来告知 kernel 出队 SQE 并执行

4. 对于你提交的 每个SQE，一旦执行完成，kervel 会在 CQ tail 添加 CQE 。
    每个 SQE 都有 有且仅有一个 CQE
    你收到 CQE 后，你 需要检查 CQE 的 res 字段， 它保存了 系统调用的 返回值。你可以直接使用它， 不需要再 通过 io_uring
    举例来说： 通过 io_uring 进行 读操作 时， 以 IORING_OP_READ 操作 开始，等同于 read 系统调用。 实际上，如果有 偏移量的话，它 混合了 pread, preadv 的语义， -1的 偏移量值 代表 当前文件位置。

    io_uring 是 异步接口，所以 errno 没有用来 传递 错误信息。  使用 CQE 的 res 来传递， 如果成功，res是 对应系统调用的返回， 如果失败，res 是 -errno 。
        例如， 如果 普通的read 系统调用 返回 -1, 并设置 errno 为 EINVAL，那么 res 会是 -EINVAL。 如果 read成功，返回了 读到的字节数为 1024, 那么 res 是 1024。

5. 可选的，io_uring_enter 也可以 等待 指定数量的 request 被 kernel 处理完成后 再返回。 kernel 会至少放置 你指定的 数量 个 CQE 。你可以在 io_uring_enter 之后，直接 读。

6. 要记住，提交给 kernel 的 IO请求 可以 以任意顺序 完成。 你不能要求 kernel 在完成某个请求后 才能处理 另一个请求。
    当你 出队 CQE 的时候，你应该 总是 检查 这个CQE 对应的 请求。
    最常用的方法是 利用 SQE的 user_data 字段， 这个字段 会被 赋值到 CQE 上。


对于SQ，CQ
- 你在 SQ tail 上添加 SQE， kernel 在 头上读取
- kernel 在 CQ tail 上 添加 CQE，你在 头上读取


io_uring 支持 polling 模式， 这样 你就不需要调用  io_uring_enter 了。
通过 SQ Polling， io_uring 启动一根 内核线程 ，它会 poll 所有的IO请求。  启动SQ POlling 后，你就不需要 调用 io_uring_enter 了。 这样 减少了 系统调用的 开销。

还有一些 代码及注释，  使用io_uring的 代码片段,  枚举值的 说明

## my_cpp_code/test

有 io_uring 的example










