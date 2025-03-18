2023-06-29 09:14

[[toc]]


当发现库存为0后，通知前面的 网关或nginx，不要再转发请求了。
netty需要掌握：TCP，IO。

# 总览

涉及：
- linux
- kernel
- 系统调用
- sendfile 零拷贝
- mmap 内存映射
- jvm 调用系统内核


保护模式
用户态，内核态
用户空间，内核空间

内核中有表(GDT，全局描述符表)，表中定义了 内核空间的 范围， 非内核空间(用户空间)。对CPU增加了约束，如果其他软件读取了内核空间，那么异常。

base + offset 找到真实的地址。

代码调用库，库通过system call 调用内核。


# 中断

编码的时候，没有写：经过xx毫秒，让出CPU。
CPU中有晶振，每振一下，就产生一个事件。

## 硬中断
，比如 时间中断(晶振)，键盘按下后的 事件也会触发CPU 的中断。

## 软中断(陷阱陷入)
，程序先触发中断，然后调用内核；程序会触发一个80中断(软中断)，CPU开始切换态，从用户态 切换到 内核态。


# IO

linux： `nc -l localhost 8080`
启动nc，监听 8080端口。
这里模拟 tomcat，别人可以向8080端口发送数据

。。

https://www.cnblogs.com/aplmmy49y/p/16494786.html

nc是netcat的简写，是一个功能强大的网络工具，有着网络界的瑞士军刀美誉。nc命令在linux系统中实际命令是ncat，nc是软连接到ncat。nc命令的主要作用如下：
-    实现任意TCP/UDP端口的侦听，nc可以作为server以TCP或UDP方式侦听指定端口
-    机器之间传输文件
-    端口的扫描，nc可以作为client发起TCP或UDP连接
-    机器之间网络测速

使用示例：
- 检查服务器端口是否通
- 拷贝文件
- 终端之间的通信聊天
- 扫描端口
- 验证UDP端口
- 测网速

。。




`ps -ef | grep nc`
所有包含nc的进程， 选出 进程的 pid。

`netstat natp`
服务器才会 Listen。
可以看到 nc的信息。

`cd /proc/{nc-pid}`
关注 fd 目录。
。。file descriptor

`cd fd`
`ll`
可以看到 4个文件 0,1,2,3， 代表了 4个文件流。 标准输入，标准输出，标准错误，
3 是 socket。
服务器listen 的时候 就会消耗 一个 socket


`nc localhost 8080`
客户端，连接。

回到 fd文件，ll， 会多一个 socket：4

3是listen的 文件描述符
4是 客户端的连接的 文件描述符。


`netstat natp`
会有2个nc，一个是 listen， 一个 establish。

`man 2 socket`
manual, 2类(系统调用)

```text
RETURN VALUE
       On success, a file descriptor for the new socket is returned.   On  er‐
       ror, -1 is returned, and errno is set appropriately.
```

nc 启动的时候，进行 系统调用 调用 socket。


`man 2 bind`
里面有 Example， C编写的。

先 bind，然后 listen，然后 accept。


`man 2 accept`
```text
RETURN VALUE
       On  success,  these  system  calls return a file descriptor for the 
       ==accepted== 
       socket (a nonnegative integer).  On error, -1 is returned, errno
       is set appropriately, and addrlen is left unchanged.
```


## BIO

测试用的 java 代码

```java
// java.io.*; java.net.ServerSocket; java.net.Socket;
ServerSocket server = new ServerSocket(8090);
sysout("step 1");
while (true) {
    Socket client = server.accept();            // --阻塞
    sysout("step 2 " + client.getPort);
    new Thread(() -> {
        try {
            InputStream in = client.getInputStream();
            BufferReader reader = new BufferedReader(new InputStreamReader(in));
            while (true) {
                sysout(reader.readLine());      // --阻塞
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }).start();
}
```

linux `strace -ff -o ./out java TestSocket`
查看这个线程 使用的系统调用 及 它收到的响应。
。。java TestSocket 启动java代码
strace会输出10个左右文件，一个文件就是一个线程。 后缀是 线程ID号。

`jps`
找到 TestSocket的 pid

`cd /proc/{pid}/fd`
ll
可以看到0-5， 
0-2 是标准输入，输出，错误
3是jvm依赖的 jre/lib/rt.jar
4,5 都是 socket， 4先跳过，5就是 socket server listener。


`cd /proc/{pid}/task`
可以看到线程。
一个线程一个目录。
这里的线程 和 strace 的是一致的。


在strace的文件中，搜 端口号，socket，step 1，等。
搜端口号，然后向上找，可以看到 socket -> 5

可以在某个文件中，看到最后是 poll。

`tail -f xx.pid`


`nc localhost 8090`
mock 一个client。

可以在 tail 看到日志中，poll后面就打印了很多日志。并且有 6，在 /proc/{pid}/fd 中可以看到 新建的 6 文件。
tail 的最后又是一个 poll。 在这个 poll的上面几行，有一个 clone， 这个是 new Thread 的日志。
新建 thread， strace 也有一个 新文件，这个新文件的最后是 recvfrom(6) 。 6是文件描述符。


`tail -f xx.6的pid`
在 mock的client中输入， 可以在 server的 终端中看到 输入的东西。

0 输入
1 输出

可以看到 tail中 最后是，
write(6, xxx), 
write(1, xxx), 
write(1, xxx)
由于是 死循环，所以 最后又是 recvfrom(6)



### 总结
线程创建的时候，使用clone，需要使用内核态。
主线程阻塞等待，建立连接，每个线程一个socket，阻塞等待client。
线程栈的大小是1mb。
线程 需要进入 内核态 才能看到 有没有 数据到来。线程多了以后，频繁切换，频繁切换内核态 用户态。




## NIO

`man 2 socket`

description中，可以看到 ==SOCK_NONBLOCK== 。这个参数。

内核发生了变化，对文件描述符设置了 nonblock，调用 accept，recvfrom，read 等 的时候，会==立刻返回==，返回的可能是 到达的数据，可能是错误码。 这里==不会阻塞==。

。。感觉，比学习算法 提升还大。。。。

可以用list保存socket，遍历查看每个socket是否有数据。

`man 2 fcntl`
中查看 F_SETFL(long) 中有 O_NONBLOCK


### 弊端
。。死循环不是问题，是正确的。

每循环一次，如果socket很多(1000个)，会 1000次 通过 recvfrom/read 调用 内核，发生了 内核，用户态的切换。 读到数据的几率不大，所以很多系统调用都是 浪费的。

。。还没有看下面的 多路复用，我在想，能不能 对于读到数据的 socket，新建一个thread，thread存活几秒，，好像是 socket的生命周期 从 list 中 移动到 thread中， thread 结束的时候 会把 socket 放入到 list 中。

最理想的状态： 即使有1000个客户端， 复杂度为 O(1)。 只处理 有数据到达的 socket。 减少 不必要的 recvfrom/read 的 系统调用。 就有了 ==多路复用==。


1:22:00

## 多路复用 poll，select

一次调用，就知道 哪些socket 有数据到达。

`man 2 select`
```text
       int select(int nfds, fd_set *readfds, fd_set *writefds,
                  fd_set *exceptfds, struct timeval *timeout);

DESCRIPTION
       select() allows a program to monitor multiple file descriptors, waiting
       until one or more of the file descriptors become "ready" for some class
       of I/O operation (e.g., input possible).  A file descriptor is  consid‐
       ered  ready  if it is possible to perform a corresponding I/O operation
       (e.g., read(2), or a sufficiently small write(2)) without blocking.

       select() can monitor only file descriptors numbers that are  less  than
       FD_SETSIZE;  poll(2)  and  epoll(7)  do  not have this limitation.  See
       BUGS.


RETURN VALUE
       On success, select() and pselect() return the number of  file  descrip‐
       tors  contained in the three returned descriptor sets (that is, the to‐
       tal number of bits that are set in readfds, writefds, exceptfds).   The
       return  value  may  be  zero if the timeout expired before any file de‐
       scriptors became ready.

       On error, -1 is returned, and errno is set to indicate the  error;  the
       file descriptor sets are unmodified, and timeout becomes undefined.

```

一次性将 1000个客户端的socket，传给 内核， 用户态和内核态 只切换一次。
内核去遍历这1000个。
。。。

select 返回的是 事件。
知道有 fd可用后， 代码还是需要遍历调用 read/recvfrom。

。。。难道后面的 poll 就是 直接返回 可用的fd？



不足：
- 内核需要遍历
- 每次都要传递1000个fd给内核。
- 代码知道有socket可用后，还需要遍历所有的fd，并调用 recvfrom/read。


数据通过网卡传输过来，通过中断，来让CPU感知。

网卡通过 DMA 直接写入到内存，然后中断 CPU。

CPU被中断后，通过 中断号，进行回调。

中断 就是一个事件： 网卡有数据到达。

内核知道 哪个socket有数据到达。

内核在 内核态的内存中 缓存fd，而不是每次都要传入。

poll，select


## 基于事件的多路复用 epoll, event poll

`man epoll`

epoll_create
epoll_ctl
epoll_wait

`man 2 epoll_create`

```text
RETURN VALUE
       On success, these system calls return a file descriptor (a  nonnegative
       integer).   On  error, -1 is returned, and errno is set to indicate the
       error.
```

`man 2 epoll_ctl`
```text
       #include <sys/epoll.h>

       int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);

。。epoll_fd, operator, fd, event

DESCRIPTION
       This  system  call is used to add, modify, or remove entries in the in‐
       terest list of the epoll(7) instance referred to by the file descriptor
       epfd.   It  requests  that the operation op be performed for the target
       file descriptor, fd.
```

`man 2 epoll_wait`
```text
       #include <sys/epoll.h>

       int epoll_wait(int epfd, struct epoll_event *events,
                      int maxevents, int timeout);
       int epoll_pwait(int epfd, struct epoll_event *events,
                      int maxevents, int timeout,
                      const sigset_t *sigmask);

DESCRIPTION
       The epoll_wait() system call waits for     

       events     

       on the epoll(7)  instance
       referred  to  by  the  file  descriptor epfd.  The buffer pointed to by
       events is          

       used to return information         

       from the ready list about file de‐
       scriptors  in the interest list that have   

       some events available           

       .  Up to
       maxevents are returned by epoll_wait().  The maxevents argument must be
       greater than zero.
```


程序执行流程
1. 已有准备监听的fd: 5， 即server监听的socket 5
2. epoll_create，传入size, 返回8。 内核开辟了空间(防止每次都要传入需要监听的fd的list)，就是8。
3. epoll_ctl，传入8,5,监听accept的事件。
4. epoll_wait， 等待ing。。。等待 epoll fd 8
5. wait返回后，获得新socket的fd 9
6. 将 9 通过 epoll_ctl 添加到内核的 epfd 8 中，监听read。
7. epoll_wait

。。不知道




# nginx
netty，redis
哪些技术使用了epoll

`strace -ff -o myout /opt/nginx/sbin/nginx`

。。worker_processor 改成1 ，不然线程太多。
。。nginx 主从配置，有一个master，有一个 slave， kill掉 slave， master 会新建一个， 所以可以 热更新。

有3个文件。
一个文件，最后是 existed with 0。 已经退出了。 它创建 master 线程。
master线程 clone slave(worker)。

worker的文件的最后是 epoll_wait

`curl localhost`
nginx 默认80端口。

可以看到worker 的 文件很多epoll 的信息。



netty默认走 jdk，默认走 poll/select， 可以手动切换为 epoll。

# 0拷贝 sendfile

## kafka
是一个可持久化的队列

可持久化：数据需要存到磁盘
队列：有生产者，有消费者。

所以问题来了： 怎么把 生产者的 数据存放到磁盘， 怎么从磁盘读取 消费者需要的数据。

### 数据存入磁盘

数据通过 socket，从 生产者发送过来，发送到 内核，
kafka 底层使用了 epoll。

数据需要从 内核空间读取到 用户空间，因为 kafka 需要对 数据进行加工。
调用系统调用 write， 从用户空间写入到 内核空间，然后保存到 磁盘。



## MMAP内存映射
内核开辟的
内存中的空间，内核和用户 都可以访问。
内核可以把这块空间映射到 磁盘的某个文件。

segment file, 默认大小 1g。

kafka默认1g。这1g 是 内核和kafka 都可以访问。
kafka收到生产者的数据后，进行处理后，就放到这个 共享空间内，并且是 append。append 的时候就保存到 磁盘了。

kafka 收到 消费者的请求后，调用内核，从磁盘上 读取数据， 然后写到 网卡，发给 消费者。

## 什么时候可以 0 拷贝：数据不需要加工。


`man 2 sendfile`

```text
DESCRIPTION
       sendfile()  copies  data     
       
       between one file descriptor and another.  
       
       Be‐
       cause this copying is done within the kernel, sendfile() is 
       
       more  effi‐
       cient than 
       
       the combination of read(2) and write(2), which would require
       transferring data to and from 
       
       user space.
```





