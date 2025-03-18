TCP
2022年11月29日
13:33

nocode，高层，底层

transmission control protocol




[[toc]]

========================


# TCP头，IP头，UDP头

![8e62449b2a0ab024838321a312b334ac.png](../_resources/8e62449b2a0ab024838321a312b334ac.png)

---

![3da52b3233b6a90696a20c6f1a2cdb98.png](../_resources/3da52b3233b6a90696a20c6f1a2cdb98.png)

![fafa6587b96d225e7f29bbd106d1d203.png](../_resources/fafa6587b96d225e7f29bbd106d1d203.png)

---

![6af25e2e697c5b5927dd9de7f852bf68.png](../_resources/6af25e2e697c5b5927dd9de7f852bf68.png)

伪首部是一个虚拟的数据结构，其中的信息是从数据报所在IP分组头的分组头中提取的，既不向下传送也不向上递交，而仅仅是为计算校验和。
这样的校验和，既校验了TCP&UDP用户数据的源端口号和目的端口号以及TCP&UDP用户数据报的数据部分，又检验了IP数据报的源IP地址和目的地址。
伪报头保证TCP&UDP数据单元到达正确的目的地址。因此，伪报头中包含IP地址并且作为计算校验和需要考虑的一部分。
最终目的端根据伪报头和数据单元计算校验和以验证通信数据在传输过程中没有改变而且到达了正确的目的地址。

# 通信模型


![fee9c5d3432c66b9d025a250437075d2.png](../_resources/fee9c5d3432c66b9d025a250437075d2.png)

![b8c6e926808da0c7b24c96696976eaaf.png](../_resources/b8c6e926808da0c7b24c96696976eaaf.png)





# 拥塞控制算法

cwnd: congestion window 发送方通过保证自己的 发送窗口 小于等于 拥塞窗口 以避免网络拥塞的发生

- 慢启动/慢开始
  每过1个rtt，cwnd大小加倍，直到 预先定义的阈值 或 发生网络拥塞 。 达到 阈值 或 发生网络拥塞，进入线性增长，每过1个rtt，增加一个 mss
- 拥塞避免
  网络出现拥塞后，将 cwnd重置，并缓慢增加 cwnd 的大小。 cwnd 线性增长，而不是 指数增长。每个rtt增加 1个mss
- 快速重传
  检测到3个重复ACK后， 立刻重传
- 快速恢复
  收到3个重复ACK后，进入快速恢复状态，发送方将 cwnd 减半，发送窗口设置为 丢包前的大小。 重传 丢失的包 与 (正常的)后续的数据包。 每收到一个ack，将窗口翻倍，直到达到 原始的 拥塞窗口，然后，TCP进入 拥塞避免状态。

## Tahoe (慢启动，拥塞避免，快速重传，没有快速恢复)

TCP的早期版本。

核心思想：让cwnd 以指数增长的方式迅速逼近可用信道容量，然后慢慢接近均衡。

包括3个基本的拥塞控制算法
- 慢启动
- 拥塞避免
- 快速重传

不包括 快速恢复算法

在收到 3个重复ACK 或 超时的情况下， 将 cwnd 设置为 1, 然后进入 慢启动阶段。

会导致 网络的激烈振荡，也会降低 网络的利用率

没有快速恢复算法，在恢复丢失数据包期间，不能发送新的数据包，这段时间的吞吐量为0。
如果利用这段时间 来发送 适量的 新数据包，可以大大提高丢包的传输效率，这就是 快速恢复 名称的由来。



## Reno (增加快速恢复阶段，不能有效地处理多个分组从同一数据窗口丢失)

增加了 快速恢复阶段，即，完成 快速重传后，进入拥塞避免阶段 而 不是慢启动阶段。

在快速恢复阶段，收到重复的ACK，则 cwnd + 1, 收到 非重复的ACK，cwnd = sshthresh，转入 拥塞避免； 如果发生 超时重传，则设置 ssthread 为当前 cwnd 的一半，cwnd设置为1, 进入 慢启动阶段。




## NewReno (有效地处理多个分组从同一个数组窗口丢失，不支持SACK)

reno 不能有效处理 多个分组从同一个数据窗口丢失。

new reno 丢改了 reno 的快速恢复算法，处理 一个窗口中 多个报文段 同时丢失 的 出现的 部分确认 (partial ack， 它在快速恢复阶段 到达 并 确认新数据，但它 只确认进入快速重传之前发送的 一部分数据) 
这种情况下，reno 会退出 快速恢复状态，等待 定时器溢出 或 重复的ack到达， 
但是 new reno 并不退出 快速恢复状态，而是
- 重传紧接着那个 部分ack 之后的报文段，拥塞窗口等于 其减去 部分ack的部分
- 对于ack的新数据，cwnd++
- 对于每个 部分ack，复位 重传定时器，且 减半 cwnd




## SACK

使得接收方 可以告知 发送方，哪些报文段丢失，哪些报文重传，哪些报文 提前收到。
根据这些信息，发送方只 重传 真正丢失的报文段。

只有收到失序的分组时才会可能会发送SACK

需要在 TCP 包头 增加2个选项，一个是 开启选项，一个是 sack 选项。


---

# ==拥塞控制算法==

BV1aPBCYCESR


慢开始
拥塞避免
快重传
快恢复

![b42eb0ec1bbfcf1bbf0a513d764296a9.png](../_resources/b42eb0ec1bbfcf1bbf0a513d764296a9.png)

。。拥塞窗口，用于控制 发送窗口。
。。拥塞窗口指的是在收到对端 ACK 之前自己还能传输的最大 MSS 段数。

。。ack 的值 是 表示 小于这个序号的 都已经收到，即 期望下次收到 这个序号

1. 最开始的阶段是 ==慢开始==。
2. 慢开始的特点是， 拥塞窗口从1开始，拥塞窗口的大小 和 往返时延(rtt) 呈 指数次方 (1,2,4,8,16)。
3. 达到 慢开始 门限(ssthresh) 后 停止增长， 开始执行 ==拥塞避免==
4. 拥塞避免时， 每经过一个 rtt， 拥塞窗口+1
5. 超时(指定时间内没有收到ack)后，
  1. 调整 慢开始的 门限 为当前 拥塞窗口的 一半 ( 上图中 3 的 y轴就是 12，是 24的 一半) 
  2. 进入 ==慢开始==
6. 慢开始 拥塞窗口 1-2-4-8， 由于16大于12，所以 窗口是 12， 并且进入 ==拥塞避免==
7. 拥塞避免， 每过一个rtt，拥塞窗口+1
8. 如果在拥塞避免状态下，碰到 ==3-ack== 这种情况 (即 收到3个重复的 ack)，(。。说明数据包丢失了，而不是拥塞 )，此时 要进行 ==快重传，快恢复==。
   1. 发送方 发送 接收方 期望的 数据包 ==快重传==
   2. 慢开始 门限(ssthresh) 变成当前窗口值的一半 (16->8)。
   3. 拥塞窗口的大小 变成 门限值。 这就是 ==快恢复==， 避免了 慢开始的 从1开始。
9.  继续执行 拥塞避免


。。超时，3ack 时， ssthresh 都是变成当前拥塞窗口的 一半。
。。超时后 进入 慢重传， 拥塞窗口从1开始
。。3ack，进入快重传快恢复， 拥塞窗口从 ssthresh开始 (即 之前拥塞窗口的一半)， 直接跳过了 慢开始。 ( 慢开始 是从1开始，指数增长到 ssthresh，然后进入 拥塞避免， 现在 拥塞窗口直接是 ssthresh，直接进入 拥塞避免。)

。。不过没有说 在 慢开始阶段，收到3ack，怎么处理，应该 也是 快重传，快恢复。 不值得再次慢开始。



---

https://zhuanlan.zhihu.com/p/430799766
https://juejin.cn/post/7028003193502040072

# 3次握手，4次挥手

TCP传输分为3个阶段
1. 建立连接
2. 数据传输
1. 释放连接

TCP头部格式

![1](../_resources/29ddf076b16848708a9615b07a010817.png)

源端口，16bit，源端口号，用来识别  发送  该TCP报文段的  应用进程。

目的端口，16bit，目的端口号，用来识别  接受  该TCP报文段的  应用进程。

序号，32bit，取值范围[0,2^32-1]，序号增加到最后一个后，下一个序号又回到0。指出  本TCP报文段  数据载荷  的第一个字节的  序号。

确认号，32bit，uint，最大+1变成0。  指出  期望  收到对方下一个TCP  报文段的  数据载荷的  第一个字节的  序号，同时  也是对  之前收到的  所有数据的确认。如果确认号是  n，那么说明  <= n-1  的数据都已经被正确接收，希望获得  序号为  n  的数据。

确认标志位ACK，取值为1  时，  确认号  字段才有效。TCP规定，在连接建立后，所有传送的  TCP报文段  都应该将  ACK  设置为  1

数据偏移，占4bit，并以4字节为单位。用来指出  TCP  报文段的  数据载荷部分的  起始处  距离  TCP  报文段的  起始处有多远。  这个字段实际上  是指出  TCP报文段的  首部长度。

窗口，16bit，以字节为单位。指出发送  本报文段的  一方的  接收窗。

同步标志位SYN，在TCP连接  建立时  用来同步序号。

终止标志位FIN：用来释放TCP连接。

复位标志位RST：用来复位TCP连接。

推送标志位PSH：接收方的  TCP  收到  该标志位为1  的报文  会  尽快上交应用程序，而不必等到接受缓存  都填满后  再向上交付。

校验和：16bit，检查范围  包括  TCP  报文段的  首部  和  数据载荷  2部分。在计算校验和时，要在  TCP  报文段的  前面加上  12字节的  伪首部。

紧急指针：16bit，以字节为单位，用来指明  紧急数据的长度。

填充，由于  选项的  长度可变，因此使用  填充来  确保报文段首部  能被4整除  (因为  数据偏移  字段，  也就是  首部长度字段，是以  4字节  为单位的)

## TCP的连接建立
建立连接的过程叫做握手，需要在  客户端  和  服务器  之间  交换  3个TCP  报文段，称为  三报文握手。
采用三报文握手，主要是为了防止  已  失效的  连接  请求  报文段  突然  又传送到了，而产生错误。

TCP的连接建立  需要解决以下  3个问题
1. 使  TCP  双方都能  确知  对方的  存在
2. 使  TCP  双方  能够  协商  一些  参数  (  最大窗口值是否使用窗口扩大选项和时间戳选项，以及  服务质量等)
3. 使  TCP  双方  都能  对  运输实体资源  (例如  缓存大小  连接表中的项目  等)  进行分配。

主动发起TCP连接建立  的称为  客户端
被动等待  TCP连接建立  的称为  服务器

1. 最开始，2端  的  TCP  进程都处于  关闭状态。

2. 服务器TCP进程  创建  传输控制块，用来  存储  TCP连接中的  一些重要信息，例如，TCP连接表，指向发送  和  接受  缓存的  指针，指向  重传队列的  指针，当前的  发送  和  接受序号  等。之后  就  准备  接受  TCP客户端进程  的  连接请求，此时  TCP服务器进程  进入  监听状态，等待  TCP客户端进程的  连接请求。

3. TCP客户单  也需要  首先  创建  传输控制块，然后  再打算建立。TCP服务器进程  是  被动等待  来自TCP客户端进程的  连接请求，所以被称为  被动打开连接。

4. TCP连接时  向TCP服务器  进程  发送  TCP连接请求  报文段，并进入  同步已发送状态。
    1. TCP连接请求报文首部中的  同步位SYN  被设置为  1，表明这是一个  TCP  连接请求报文段。
    2. 序号字段seq  被设置为  一个  初始值  x  作为TCP客户端进程  所选择的  初始序号。

5. 由于TCP连接建立  是由  TCP  客户端进程主动发起的，因此成为  主动打开连接。  注意TCP规定  SYN被设置为1  的报文段  不能携带数据  但要  消耗一个  序号。

6. TCP服务器进程  收到  TCP  连接请求报文段后，如果  同意建立连接，则向  客户端进程  发送  TCP  连接  请求  确认报文段，并进入到  同步已接收状态。

    1. 该报文段  首部中的  同步位  SYN  和  确认位ACK  都设置为1，  表明这是一个  TCP连接请求。
    2. 序号字段SEQ被设置了一个  初始值  y，作为  TCP服务器进程  所选择的  初始序号。
    3. 确认号  字段ACK  的值  被设置为  x+1，  这是对TCP  客户进程  所选择的  初始序号SEQ  的确认。
注意，这个报文段  也不能携带数据，因为  SYN是1，但同样要消耗一个  序号。

1. TCP客户进程  收到  TCP  连接请求  确认报文段后，还要向  TCP服务器  进程  发送一个  普通的TCP确认报文段  并进入  连接已建立状态。

    1. 该报文段首部中的  确认位ACK  为1，表明是一个  普通的TCP确认报文段

    2. 序号字段SEQ  被设置为  x+1，  这是因为  TCP客户端  发送的第一个TCP  报文的序号是  x，并且  不携带数据，所以  第二个报文段的  序号是  x + 1

    3. 确认号字段ACK  被设置为  y + 1，  这是对  TCP服务器进程  所选择的  初始序号的  确认。

2. TCP服务器收到  该确认报文段后  也进入  连接已建立状态，现在  TCP双方都进入了  连接已建立  状态，它们可以基于  已建立好  的TCP  连接进行  可靠的数据传输了。

![1](../_resources/4fb4e810e7ed46bb89b61fe8786b5435.png)

注意，TCP规定，普通的TCP  确认报文段  可以携带数据。但如果不携带数据  则不消耗  序号，在这种情况下，下一个数据报文段的序号仍然是  x + 1。
。。。？不消耗序号的话，下一个  不应该  还是  x  ？

两次握手的问题
客户端发送  建立TCP连接请求，
但是超时，客户端重试，这次  服务器收到，并返回  连接建立  报文给  客户端，
数据传输，
挥手释放连接。

客户端发送的第一个请求  送达服务器，服务器  返回  连接建立报文  给客户端，  但是客户端已经关闭了，所以不会有响应，  服务器就一直维持着  这个连接，等待  客户端的数据。

在不可靠的信道中想要模拟出可靠的，双向的传输最少也得是三次通信，两次只能建立可靠的单向传输。

-----------------------------------

https://juejin.cn/post/7132499826771656734

# 四次挥手
1. 客户端主动停止连接，向服务器发送  FIN=1  的报文
2. 服务器收到后，返回一个  确认报文
3. 服务器  确定自己也没有数据  要发送给  客户端时，服务器会向  客户端  发送  FIN=1  的报文，并等待确认
4. 客户端在收到  服务器的  FIN=1  的报文是，会  立即返回  一个  确认报文，并进入  2MSL  等待阶段。

在2MSL  时间范围内，如果客户端收到服务器端  超时重传的  FIN=1  的报文，则说明  客户端发出的确认，服务端  没有收到，所以此时客户端  会重新发送  确认并  重置计时器，在2MSL只有，没有收到服务端的  超时重传  FIN=1  的报文，则客户端正常关闭。服务器在收到  确认后也是正常关闭的。

2MSL MSL是Maximum Segment Lifetime英文的缩写,中文可以译为“报文最大生存时间”

-----------------------------

https://juejin.cn/post/7075945250115551239

|     |     |     |
| --- | --- | --- |
| TCP/IP模型 | OSI模型 | 作用  |
| 应用层http | 应用层 | 为应用程序提供服务 |
| 应用层http | 表示层 | 数据格式转化，数据加密 |
| 应用层http | 会话层 | 建立，管理，维护会话 |
| 传输层TCP | 传输层 | 建立，管理，维护  端对端的连接 |
| 网络层IP | 网络层 | IP选址  及  路由选择 |
| 链路层(物理网卡) | 链路层 | 提供介质访问  和  链路管理 |
| 链路层(物理网卡) | 物理层 | 物理传输 |

TCP/IP  协议族

TCP/IP  是  Internet  最基本的协议，Internet  国际互联网络的基础，由网络层的  IP  协议  和  传输层的  TCP  协议组成。

|     |     |     |
| --- | --- | --- |
| TCP/IP概念层模型 | 功能  | TCP/IP协议族 |
| 应用层 | 文件传输，电子邮件，文件服务，虚拟终端 | TFTP,HTTP,SNMP,FTP,SMTP,DNS,Telnet |
| 应用层 | 数据格式化，代码转换，数据加密 | 没有协议 |
| 应用层 | 解除  或  建立  与别的接点  的联系 | 没有协议 |
| 传输层 | 提供端对端的接口 | TCP UDP |
| 网络层 | 为数据包选择路由 | IP,ICMP,RIP,OSPF,BGP,IGMP |
| 链路层 | 传输地址的帧以及错误检测功能 | SLIP,CSLIP,PPP,ARP,RARP,MTU |
| 链路层 | 以二进制数据形式在物理媒体上传输数据 | ISO2110,IEEE802,IEEE802.2 |

一次完整的http请求的过程
DNS域名解析（本地浏览器缓存，操作系统缓存或者DNS服务器）
三次握手建立TCP连接
客户端发起HTTP请求
服务器响应HTTP请求
客户端解析html代码，并请求HTML代码中的资源
客户端渲染展示内容
关闭TCP连接
1.0 连接不可以复用    2.0  IO多路复用，一次TCP连接可以进行多次http请求

为什么需要四次挥手?
TCP是全双工（即客户端和服务端可以相互发送和接收请求）所以需要双方都确认关闭连接

为什么需要TIME_WAIT状态?
客户端的最后一次应答报文可能在网络传输中会丢失，所以客户端还得进行一次重传，确保可靠的终止TCP/IP  保证迟来的TCP报文有足够的时间，被识别并丢弃

========================

========================

========================

========================

========================

========================

========================

========================

========================

========================

已使用 Microsoft OneNote 2016 创建。



# Code a TCP/IP stack

https://github.com/saminiir/level-ip


## Ethernet(以太网) & ARP

http://www.saminiir.com/lets-code-tcp-ip-stack-1-ethernet-arp/

TCP 已经超过40年了(1981)，累积了很多的规范。
但，最核心的规范看起来很简单：最重要的部分是 TCP header parse， state machine，congestion control(拥塞控制)，retransmission timeout computation(超时重发)

大部分2层，3层(Ethernet, IP层)的协议，和 TCP 比较起来是 非常简单的。

在本系列的blog中，我们会在 linux 上实现 最小的 用户空间的 TCP/IP 栈。


### TUN/TAP devices

。。 TUN/TAP是操作系统内核中的虚拟网络设备,可以完成用户空间与内核空间的数据的交互
。。 linux支持的虚拟网络设备中,tun/tap设备相对特殊,其为用户空间程序提供了网络数据包的发送和接收能力

。。在计算机网络中，TUN 与 TAP 是操作系统内核中的虚拟网络设备。不同于普通靠硬件网路板卡实现的设备，这些虚拟的网络设备全部由软件实现，并向运行于操作系统上的软件提供与硬件的网络设备完全相同的功能。 TAP 等同于一个以太网设备，它操作第二层数据包如以太网数据帧。TUN 模拟了网络层设备，操作第三层数据包比如 IP 数据封包。
。。操作系统通过 TUN/TAP 设备向绑定该设备的==用户空间的==程序发送数据，反之，用户空间的程序也可以像操作硬件网络设备那样，通过 TUN/TAP 设备发送数据。在后种情况下，TUN/TAP 设备向操作系统的网络栈投递（或“注入”）数据包，从而模拟从外部接受数据的过程。
。。https://zhuanlan.zhihu.com/p/388742230


为了从 linux kernel 拦截 底层的网络流量，我们会使用 Linux TAP device。
networking userspace application 使用 TUN/TAP device 来操作 L3/L2 的traffic(流量)
常见的例子是 ==tunneling==，一个packet 被封装在 另一个packet 中。

TUN/TAP device 的优势是，在用户态程序中 容易配置，并且已经得到了广泛的 应用，如 OpenVPN

从L2 开始构建 networking stack，需要一个 TAP device。代码如下：
```C
/*
 * Taken from Kernel Documentation/networking/tuntap.txt
 */
int tun_alloc(char *dev)
{
    struct ifreq ifr;
    int fd, err;

    if( (fd = open("/dev/net/tap", O_RDWR)) < 0 ) {
        print_error("Cannot open TUN/TAP dev");
        exit(1);
    }

    CLEAR(ifr);

    /* Flags: IFF_TUN   - TUN device (no Ethernet headers)
     *        IFF_TAP   - TAP device
     *
     *        IFF_NO_PI - Do not provide packet information
     */
    ifr.ifr_flags = IFF_TAP | IFF_NO_PI;
    if( *dev ) {
        strncpy(ifr.ifr_name, dev, IFNAMSIZ);
    }

    if( (err = ioctl(fd, TUNSETIFF, (void *) &ifr)) < 0 ){
        print_error("ERR: Could not ioctl tun: %s\n", strerror(errno));
        close(fd);
        return err;
    }

    strcpy(dev, ifr.ifr_name);
    return fd;
}
```

返回的 文件描述符 fd 可以用来 对 虚拟设备 的 ethernet buffer 进行读写。

在这里，IFF_NO_PI 是 关键，不然 我们需要在 ethernet frame 中预处理不必要的数据包信息

你可以查看 tun-device driver 的源码 ( https://github.com/torvalds/linux/blob/v4.4/drivers/net/tun.c#L1306 )，并自己验证


### ethernet frame format

各种不同的 以太网网络技术 (ethernet networking technologies) 是 LANs (local area networks，局域网) 中 连接 计算机的 backbone (骨干，基础，支柱)

ethernet的 第一个版本 非常慢：大约10Mb/s，使用了 半双工通信，意味着 你 要么发送，要么接受 数据，不能同时 发送，接受。 
这就是 为什么 必须使用 MAC(media access control) 协议 来组织数据流。 
现在，如果在 半双工模式下 运行 ethernet 接口，需要使用 CSMA/CD 作为 MAC method。

使用双绞线的 100BASE-T 以太网标准 实现了 全双工 和 更高的吞吐量。
并且，由于以太网交换机 的普及，导致 CSMA/CD 在很大程度上 被淘汰了。

IEEE 802.3 工作组 维护着 各种以太网标准。

我们来看下 以太网帧头， 被如下的 C struct 描述

```C
#include <linux/if_ether.h>

struct eth_hdr
{
    unsigned char dmac[6];
    unsigned char smac[6];
    uint16_t ethertype;
    unsigned char payload[];
} __attribute__((packed));
```

==dmac, smac== 是一目了然的，它们包含了 通信双方的 MAC地址。( destination, source)

==ethertype== 是一个 2字节字段，根据值不同，它可能表示 payload 的 长度 或 类型。具体来说，如果值 >= 1536, 就包含了 payload的 类型 (如 IPv4， ARP)。如果 值 小于 1536，那么它包含了 payload的 长度。
。。。？

类型字段后，以太网帧 可能有 几个不同的 ==tags==。 这些 tag 用来 描述 帧的 Virtual LAN (VLAN) 或 Quality of Service (QoS) 类型。
本次实现中，没有使用到 ethernet frame tag，所以 上面的代码中 没有它们。

==payload==，包含了一个指针 (。。数组==指针)，指向了 以太网帧的 负载。 
在我们的例子中，它会包含 一个 ARP 或 IPv4 包。
如果 payload 长度 小于 最低要求 48byte (不包含 tags)，需要在尾部 填充字节。
。。想起来，TCP/IP详解 中提到过，模糊了。 就是在 包的大小 的时候，IP头中的长度 不包含 attribute ？ 。 IPv6 是60 byte ？

代码中，使用了 if_ether.h 头文件，提供了 ethertypes 与 它们的 16进制值 之间的 映射。

以太网帧格式 也包含了 ==Frame Check Sequence 字段==，使用 CRC (cyclic redundancy check) 来校验 帧的完整性。 在本次实现中，忽略了。


### Ethernet Frame Parsing

在之前的代码中的 packed 属性 是一个 实现方面的 细节，用来指示 GNU C 编译器 不要优化 struct 内存布局 for 带字节填充的 数据对齐。

。。`__attribute__((packed))`，告诉编译器取消结构在编译过程中的优化对齐,按照实际占用字节数进行对齐,是GCC特有的语法

我们对于 data buffer 的转换 只是 type cast，所以需要这个属性。 (。。乱翻的。 The use of this attribute stems purely out of the way we are “parsing” the protocol buffer, which is just a type cast over the data buffer with the proper protocol struct:)

```C
struct eth_hdr *hdr = (struct eth_hdr *) buf;
```

一种 可移植的方式是 手动序列化 协议数据，通过这种方式， 编译器可以 添加 填充字节，来 满足不同 处理器的 数据对齐的要求。

解析和处理 incoming 以太网帧的 场景非常简单：
```C
if (tun_read(buf, BUFLEN) < 0) {
    print_error("ERR: Read from tun_fd: %s\n", strerror(errno));
}

struct eth_hdr *hdr = init_eth_hdr(buf);

handle_frame(&netdev, hdr);
```

handle_frame 只是 查看 以太网头的 ethertype，根据这个值 决定下一步行为


### Address Resolution Protocol ARP

ARP 用来 动态 映射 48-bit 的 以太网地址 (MAC地址) 到 协议地址 (如. IPv4 地址)。
这里的关键是，with ARP，多个不同的 L3 协议可以使用，不只是 IPv4，还有其他的，如 CHAOS，它声明了 16bit 的协议地址。

。。就是 IP地址，是IP协议的， 以太网层/物理链路层，不知道 IP地址是什么，它只知道 MAC，因为MAC是出厂就设置的，每个设备唯一。 所以 通过ARP，将 MAC地址 和 IP 地址进行了 关联， 以后 IP 地址解析为 MAC地址后，才能真正的传输。 
。。IP是可变的， MAC不可变。

通常的情况是，你知道 LAN 中一些设备的 IP地址，但是 要建立 真正的 通信，你还需要知道 硬件地址(MAC)。
因此，ARP 被用来 广播和查询 网络，要求 IP地址的拥有者 报告它的 硬件地址。

ARP包格式：
```C
struct arp_hdr
{
    uint16_t hwtype;
    uint16_t protype;
    unsigned char hwsize;
    unsigned char prosize;
    uint16_t opcode;
    unsigned char data[];
} __attribute__((packed));
```

hwtype, 2个8位字节，决定了使用的 link层type。 我们的案例中是 以太网层，值是 0x0001.
protype，2个8位字节，指明了 协议type。 我们的案例中是 IPv4，值是 0x0090
hwsize，prosize，都是 1字节的，分别 包含了 硬件和协议 字段的 size。 我们的案例中是，6 byte的 MAC地址，4 byte的 IP地址
opcode，2个8位字节，声明了 ARP消息的 type。 它可以是：
- 1: arp request
- 2: arp reply
- 3: rarp request
- 4: rarp reply

data，包含了 ARP消息的 实际负载，我们的例子中，它会包含 IPv4 特定消息:
```C
struct arp_ipv4
{
    unsigned char smac[6];
    uint32_t sip;
    unsigned char dmac[6];
    uint32_t dip;
} __attribute__((packed));
```

smac, dmac 是 sender，receiver的 6byte的 MAC地址
sip,dip 是sender，receiver的 IP地址


### Address Resolution Algorithm

https://tools.ietf.org/html/rfc826
上面的文档简单描述了 address resolution 的算法

```text
?Do I have the hardware type in ar$hrd?
Yes: (almost definitely)
  [optionally check the hardware length ar$hln]
  ?Do I speak the protocol in ar$pro?
  Yes:
    [optionally check the protocol length ar$pln]
    Merge_flag := false
    If the pair <protocol type, sender protocol address> is
        already in my translation table, update the sender
        hardware address field of the entry with the new
        information in the packet and set Merge_flag to true.
    ?Am I the target protocol address?
    Yes:
      If Merge_flag is false, add the triplet <protocol type,
          sender protocol address, sender hardware address> to
          the translation table.
      ?Is the opcode ares_op$REQUEST?  (NOW look at the opcode!!)
      Yes:
        Swap hardware and protocol fields, putting the local
            hardware and protocol addresses in the sender fields.
        Set the ar$op field to ares_op$REPLY
        Send the packet to the (new) target hardware address on
            the same hardware on which the request was received.
```

translation table 用来 保存 ARP的结果，作为cache，避免冗余的 ARP 请求。

这个算法的实现在下面。
https://github.com/saminiir/level-ip/blob/e9ceb08f01a5499b85f03e2d615309c655b97e8f/src/arp.c#L53



最后，对 自己的ARP 实现 进行测试，观察是否 正确的 对 arp 请求进行 reply

```shell
[saminiir@localhost lvl-ip]$ arping -I tap0 10.0.0.4
ARPING 10.0.0.4 from 192.168.1.32 tap0
Unicast reply from 10.0.0.4 [00:0C:29:6D:50:25]  3.170ms
Unicast reply from 10.0.0.4 [00:0C:29:6D:50:25]  13.309ms

[saminiir@localhost lvl-ip]$ arp
Address                  HWtype  HWaddress           Flags Mask            Iface
10.0.0.4                 ether   00:0c:29:6d:50:25   C                     tap0
```

kernel 的 网络栈 识别出来 我们的自定义的网络栈 的 arp reply，并且 存储了 arp cache

。。。？写得代码 放哪里？ 怎么启动？。。



## IPv4 && ICMPv4

http://www.saminiir.com/lets-code-tcp-ip-stack-2-ipv4-icmpv4/

这章，将在 我们的用户空间的 TCP/IP 栈中 实现 最简单的 IP层，并通过 ICMP echo request(即 ping) 来测试它。

我们会看到 IPv4 和 ICMPv4 的格式，我们会描述 如何检查它们的完整性。
一些功能，如 IP fragmentation(碎片，片段)，留给读者作为练习

我们选择 IPv4，而不是IPv6， 因为 v4 目前还是 Internet 默认的 网络协议

内容：
- Internet Protocol version 4
  - header format
  - Internet checksum
- Internet Control Message Protocol version4
  - header format
  - message and their procesing
- testing the implementation


### Internet Protocol version 4

在 实现Ethernet帧后，我们来实现 L3 层，它处理 向目的地的 数据传输。
即，IP 是为了向 传输层协议(transport protocol，比如 TCP，UDP) 提供一个 基础 而发明的。
它是 无连接的，在 网络栈中，所有的 数据报(datagram) 都是 独立处理的，和其他数据报没有关系。这也意味着 IP 数据报 可能 乱序到达。

此外，IP 不保证 传递成功，这是 设计者有意为之的，因为 IP也需要为 不保证传递成功的 协议提供一个基础，比如UDP。

如果通信需要保证 可靠性，可以在IP的 基础上 使用 TCP。 更高层的 协议 负责 检测丢失的数据，确保所有的数据 都被传递。


#### header format

IPv4 头 通常有 20个字节。 头还包含了一些 后续的 option，本次实现 我们忽略了这些 option。这意味着 字段非常简单：

```C
struct iphdr {
    uint8_t version : 4;
    uint8_t ihl : 4;
    uint8_t tos;
    uint16_t len;
    uint16_t id;
    uint16_t flags : 3;
    uint16_t frag_offset : 13;
    uint8_t ttl;
    uint8_t proto;
    uint16_t csum;
    uint32_t saddr;
    uint32_t daddr;
} __attribute__((packed));
```

- version
  4bit, 表明 internet header的 格式，我们的例子中会是 4，代表 IPv4.
- ihl
  4bit，internet header length，表明 IP header 中 word (一个word 32bit) 的数量。因为4bit，所以最大15，所以 IP header 最大 32 * 15 / 8 = 60 byte
- tos
  type of service，第一版IP规范 中就有，后续的规范中 被分解为 更小的 字段。 简单起见，我们使用 第一版中的定义。 这个字段表达 IP数据报的 服务质量
- len
  total length，代表 整个IP 数据报 的长度，由于是16 bit的，所以 最大长度是 65535 byte。大的 IP数据报 会被 fragmentation，切分成 更小的 数据报 以满足 MTU (maximum transmission unit)。
- id
  用于 对 数据报建立索引，最终用于 将 fragmented的 IP数据报 重组。 值是一个 简单的计数器，发送一部分就会增加。接收端 就知道 如何 排序 收到的 fragment。
- flags
  定义了 数据报的 控制标志。 比如，发送者 可以规定 是否允许 数据报 被 fragment， 它是不是最后一个 fragment。
- frag_offset
  fragment offset, 表明 数据报中 fragment的位置。 通常，第一个数据报 是0
- ttl
  time to live，是一个 普通属性，用于 倒计时 数据报的 生存时间。最初的发送者 通常设置64，每个 接收者 会降低 1。当 变成0时，数据报会被 丢弃，可能会返回一个 ICMP消息 来报告错误。
- proto
  为数据报提供了一种能力：在payload 中携带其他协议。通常使 16(UDP) 或 6(TCP)，将数据的实际类型 告知 接收者。
- csum
  check sum, 用来校验 IP头的 完整性。它的算法相对简单，我们稍后会解释这个算法。
- saddr，daddr
  数据报的 source，destination的 address。 32bit的，大约43亿个地址，将会被用完。 (。。目前已用完)。 地址的范围会被扩展。 IPv6的地址是 128bit


#### internet checksum

internet checksum 用来 检查 IP数据报的完整性 (。。看下面的，应该是 IP 头)

checksum是 header中 所有16位word的 补码和 的 16位1的补码。 为了计算方便， checksum字段被认为0

The checksum field is the 16 bit one’s complement of the one’s complement sum of all 16 bit words in the header. For purposes of computing the checksum, the value of the checksum field is zero.
。。。翻不出，baidu翻译的

算法 代码如下：
```C
uint16_t checksum(void *addr, int count)
{
    /* Compute Internet Checksum for "count" bytes
     *         beginning at location "addr".
     * Taken from https://tools.ietf.org/html/rfc1071
     */

    register uint32_t sum = 0;
    uint16_t * ptr = addr;

    while( count > 1 )  {
        /*  This is the inner loop */
        sum += * ptr++;
        count -= 2;
    }

    /*  Add left-over byte, if any */
    if( count > 0 )
        sum += * (uint8_t *) ptr;

    /*  Fold 32-bit sum to 16 bits */
    while (sum>>16)
        sum = (sum & 0xffff) + (sum >> 16);

    return ~sum;
}
```

比如，IP头：`45 00 00 54 41 e0 40 00 40 01 00 00 0a 00 00 04 0a 00 00 05`

1. 把字段加到一起 得到二的补码和 `01 1b 3e`
2. 为了转换为1 的补码，需要将进位增加到第一个16bit，`1b 3e` + `01` = `1b 3f`
3. 最后，获得 1的补码：`e4c0`

IP头变为：`45 00 00 54 41 e0 40 00 40 01    e4 c0    0a 00 00 04 0a 00 00 05`

。。two’s complement sum， one’s complement 是什么？
。。看代码+例子，第一步是 word 相加，word是16bit，所以 是 45 00 这种，确实和 是 01 1b 3e
。。1b 3f -> e4c0 是 取反。 不是补码。
。。感觉 2' complement 是指 2个2个 相加？

检验规则：对 转换后的 IP头进行 checksum，如果结果是0，那么是ok的。
。。是的， 1b3f + e4c0 = ffff, 取反就是0


### internet control message protocol version 4

Internet Protocol 缺乏 保证 可靠性的 机制，在出错的时候 通知另一端 是有必要的。
ICMP 用于网络中的诊断措施。
一个例子是：当 gateway 不可达到时， 识别到这个问题的 网络栈 会发送 ICMP "Gateway Unreachable" 到 origin。

#### header format

ICMP 头存放在 对应的 ==IP包的payload中==，结构如下

```C
struct icmp_v4 {
    uint8_t type;
    uint8_t code;
    uint16_t csum;
    uint8_t data[];
} __attribute__((packed));
```

- type
  表示了 消息的用途，一共有42种不同的值，只有8种是常用的，我们的实现中，只用到了0(echo reply)，3(destination unreachable)，8(echo request)。
- code
  进一步描述了 消息的含义。例如，当类型是3时，code保存 原因。一个通常的错误是 包无法被路由到 网络，这种情况下，origin会收到 一个ICMP，它的type是3，code是0(Net Unreachable)
- csum
  和IPv4中的类似，可以使用同一个算法来计算。 不过，ICMPv4中，checksum是 端到端的，意味着 计算 checksum的时候，payload 也包含在内。


#### message and their processing
实际的ICMP payload 包含：query/informational message 和 error message。
首先，我们查看 Echo request/reply 消息，在互联网中通常 指 ping
```C
struct icmp_v4_echo {
    uint16_t id;
    uint16_t seq;
    uint8_t data[];
} __attribute__((packed));
```
- id
  由发送端设置，已确定 echo reply 要发送到哪个进程。比如，进程id 会被设置到这个字段
- seq
  是echo的 sequence number，是一个从 0开始的 值，每产生一个新的 echo request，就+1。 用于 检测 echo消息是否消失 或 传输中是否被重排序
- data
  可选的，通常包含一些信息，如：echo的时间戳，能用来 估计 主机间的 rtt (round-trip time)


常见的 ICMPv4 错误消息： Destination Unreachable，有如下的格式
```C
struct icmp_v4_dst_unreachable {
    uint8_t unused;
    uint8_t len;
    uint16_t var;
    uint8_t data[];
} __attribute__((packed));
```

第一个字节没有使用。
- len
  表示初始数据报的长度，IPv4是 4个字节一个单位。
- var
  依赖于 ICMP code
- data
  保存 导致 目的地不可达的 尽可能多的 原始 IP分组



### testing the implementation

在shell中，可以验证 我们的 用户态的 网络栈 对于 ICMP echo request的 响应：

```shell
[saminiir@localhost ~]$ ping -c3 10.0.0.4
PING 10.0.0.4 (10.0.0.4) 56(84) bytes of data.
64 bytes from 10.0.0.4: icmp_seq=1 ttl=64 time=0.191 ms
64 bytes from 10.0.0.4: icmp_seq=2 ttl=64 time=0.200 ms
64 bytes from 10.0.0.4: icmp_seq=3 ttl=64 time=0.150 ms

--- 10.0.0.4 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1999ms
rtt min/avg/max/mdev = 0.150/0.180/0.200/0.024 ms
```


### Conclusion

最小可用的 能处理 以太网帧，ARP，IP的 网络栈 很容易就能创建。
但是，原始的规范 已经被扩展了很多次。在本文中，我们跳过了 IP的部分功能，如 option，fragmentation，header DCN，DS字段。

。。DCN网络，专为数据传输而设计的网络，它支持网络七层协议栈中的第一层（物理层）、第二层（数据链路层）和第三层（网络层）。DCN的主要功能是为传送平面、控制平面和管理平面之间以及它们之间的管理信息和控制信息的通信提供传送途径

。。DS，IPv4头部的第3和第4字段分别是DS(DiffServ)字段(区分服务)和ECN字段，用于支持Internet上不同类型的服务。以前这两个字段称为ToS字段。




## TCP Basics & Handshake

transmission control protocol

目前，我们的 用户态 TCP/IP 栈已经实现了 以太网 和 IPv4 的最小功能， 现在是时候开始 TCP了

TCP，在 OSI网络模型的 第四层工作，负责 修复 包传递过程中的 错误连接 和 故障。
TCP是互联网中 重要的 一环，提供了 可靠的连接。

TCP的 第一版规范在 1974年颁发。后续做了很多的修改。


内容
- reliablity machanisms
- TCP basics
- TCP header format
- TCP handshake
- TCP options


### Reliablity mechanisms

数据的可靠性问题，看起来很简单，实际上很复杂。
主要是以下的问题
- sender 应该等待 receiver的ack 多久
- receiver 不能及时处理数据，应该怎么办
- 网络不能及时传输数据，应该怎么办

。。流量控制，拥塞控制

在所有的场景中， packet-switched 网络的 潜在错误都存在： receiver的 ack 在网络中丢失了，sender的处境就很棘手。

为了解决这些问题，需要使用一些机制。
最常用的就是 ==滑动窗口==，它维护了 account of the transmitted data。

使用 滑动窗口 也可以减轻 ==流量控制 flow control== 的 问题。
当 receiver 不能及时处理数据时，流量控制是必须的。这种情况下， 滑动窗口的 size 通过协商后 会降低，来减少 sender的 输出。

==拥塞控制 congestion control==，帮助 sender 和 receiver 之间的 网络栈避免 拥塞。
有2种方法
1. 显式控制：协议中有一个 字段 (..ECN) 来将 拥挤状态 告知 sender。
2. 隐式：sender 尝试猜测 网络拥塞会在什么时候发生， 并降低 输出。

总的来说，拥塞控制 是复杂的，反复发生的 网络问题。


### TCP basics

TCP的底层机制 比其他的协议(如 UDP，IP) 更复杂。
TCP是一个 connection-oriented protocol 面向连接的协议，这意味着 TCP连接的 第一步就是 在 2侧 (各)建立一个 单播通道。
双方都在努力维护着这个 连接： 建立连接(握手)，告知另一方 数据的状态 和可能的问题。

TCP另一个重要的属性是， 它是 ==streaming 协议==。 不同于 UDP， TCP不保证 数据发送 和接收时，数据的chunk 是一致的。
TCP 实现 不得不 buffer data (。。对数据进行缓冲)，当 packet 丢失，重排序或 损坏时，TCP 不得不 等待 并在组织buffer中的data。
只有当数据 被认为是 完整的时候，TCP才会把 数据交给 应用的socket。

由于 TCP将data 作为 stream 处理， 来自stream的 chunk 会被转换为 IP能携带的 packet。 这被称为 ==packetization==，TCP header 包含了 当前index(。。应该是数据吧) 在 stream 中的 sequence number。这个属性提供了方便 来：stream 可以切分为 多个 可变大小的段( variable-seized segment)，TCP知道如何 re-packetize 它们。

类似于IP，TCP也检查 消息的完整性。 通过和 IP中相同的 checksum算法 来完成，但增加了细节。
checksum 是 end-to-end的，意味着 checksum 的范围包含 头和data。另外，还包括 由IP header 创建的 伪header。

。。？TCP怎么计算到 IP的头？ 所以用了 伪头？ 但还是很难想象 有什么意义。。
。。IP 的 checksum 只包含头， TCP的 包含 头和body？。 yes，卷1，P420，原文: "校验和字段覆盖了TCP的头部和数据以及头部中的一些字段" 。。所以这里没有问题。估计书上的 "头部中的一些字段" 等同于这里的 IP伪头
。。确实， IP的校验和 只校验 IP头，P127。
。。但是无法理解，TCP怎么假设 IP的伪头。。卷1似乎没有特别说明，估计要卷2中 看实现。。。卷2一共900页。。。没有TCP checksum的计算逻辑，只看到了头的struct，估计要自己看 源码的。TCP那章开头有说 .h文件的名称。

如果 TCP 收到 损坏的 segment，TCP会丢弃它们，不会告知 sender。 这种错误 由 sender的 定时器 来纠正，在定时器结束时，没有收到 receiver的 ack，那么会重发那个 segment。

TCP是一个 full-duplex (全双工) 系统，2个方向可以同时传输。 这意味着 通信双方 需要 在内存中 维护 2个方向的 数据的 sequence。更深入地说，TCP会在 它发送的 segment中 包含 对 对方流量的 ACK 信息。

本质上，TCP的主要 原理 是 数据流的 sequencing。 保持同步 并不不简单。 (。。估计就是说 确认信息 是否已被送达，这种 同步)


#### TCP header format

TCP header 看起来很简单，但是它包含了很多 关于 通信状态的 信息

TCP header 是 20个字节 (20 octets)

```text
        0                            15                              31
       -----------------------------------------------------------------
       |          source port          |       destination port        |
       -----------------------------------------------------------------
       |                        sequence number                        |
       -----------------------------------------------------------------
       |                     acknowledgment number                     |
       -----------------------------------------------------------------
       |  HL   | rsvd  |C|E|U|A|P|R|S|F|        window size            |
       -----------------------------------------------------------------
       |         TCP checksum          |       urgent pointer          |
       -----------------------------------------------------------------
```

- source port 和 destination port
用于建立 host的 多个连接。
伯克利socket 是 app 绑定到 TCP网络的 接口。
通过端口，网络栈 知道将数据 交给哪个进程。
16个bit，所以端口范围是 0 到 65535

- sequence number
stream中的 每个byte 都是有编号的，squence number 代表了 TCP segment的 window index。
握手时，它包含了 ISN (initial sequence number)

- ack number
ack number 包含了 sender (。。这个sender是 ack的sender) 希望收到的 下一个byte 的 window index。
握手后，ACK 必须一直 填充。

- header length
header的长度，单位是 32bit的 word。

- rsvd
4bit，没有使用。

- C
congestion window reduced，用来告知 sender 降低发送速率

- E
ECN Echo，通知sender 收到 拥塞通知

- U
urgent pointer， 表明 segment 中包含 需要优先处理的数据

- A
ack，用于传达 TCP握手的 状态，它在 连接的剩余部分 保持打开

- P
PSH，用来表示 receiver 应该 尽可能快地 推送 数据 到 app

- R
RST，重置 TCP连接

- S
SYN，在初始的握手时，用于 同步 sequence number

- F
FIN，表示 sender已经完成发送

- Window size
用于 告知 window size。即，这是 receiver 将 会收到的 数据的 byte 数。
16bit，所以 最大的window size 是 65535 byte

- TCP checksum
校验 TCP segment的 完整性。和IP的算法一样，但是 输入包含了 TCP数据 和 IP数据报的一个 伪头。

- Urgent Pointer
当 U 标记被设置的时候，有效。指针代表了 stream 中的 加急数据的 位置。


头之后，可以添加一些 option。 比如 MSS ( maximum segment size，用于 sender告知另一方 segment的最大size)。

在可选的 option之后，就是 实际的数据。 数据 不是必须的，比如握手时。


### TCP handshake

TCP 连接，通常会经历3个阶段：
- 连接建立(握手)
- 传输数据
- 连接关闭

下面是 常规的 TCP握手
```text
          TCP A                                                TCP B
    	  
    1.  CLOSED                                               LISTEN
    	
    2.  SYN-SENT    --> <SEQ=100><CTL=SYN>               --> SYN-RECEIVED
    	  
    3.  ESTABLISHED <-- <SEQ=300><ACK=101><CTL=SYN,ACK>  <-- SYN-RECEIVED
    			
    4.  ESTABLISHED --> <SEQ=101><ACK=301><CTL=ACK>       --> ESTABLISHED
    			  
    5.  ESTABLISHED --> <SEQ=101><ACK=301><CTL=ACK><DATA> --> ESTABLISHED
```

1. 主机A的 socket是关闭状态，意味着 它不会接受连接。 主机B的socket 被绑定到 指定端口 监听 新连接
2. 主机A尝试 初始化一个 到 主机B的连接。 主机A发送了一个 TCP段，其中，设置了SYN标记，sequence号被设置了 100
3. 主机B 回应了一个 TCP段，设置了 SYN和 ACK，ACK的值是 收到的sequence(100) + 1，sequence(300)是主机B 生成的。
4. 主机A 发送ACK 代表着 3-way handshake (3次握手) 完成。ACK的值代表了 下次期望收到的 sequence 值。
5. 数据开始传输，因为 双方都 ACK 了对方的 segement。

还有一些问题：
1. 最初的sequence number 应该选择什么？
2. 如果双方 同时 请求建立 到对方的请求，会发生什么？
3. 如果 segment 延迟 或 丢失，会发生什么？

ISN(initial sequence number) 是在第一次接触时，双方 各自 独立选择的，是用于 识别不同connection 的 重要因素，ISN必须尽可能唯一且难以猜测。
TCP sequence number attack 是 攻击者 复制了 TCP连接，并冒充受信任的主机 发送数据

第一版规范 建议 ISN 通过 一个 每4微秒自增一次 的 计数器 来设置。 但是，攻击者可以猜测到这个值。
实际上，现代网络栈 以更复杂的方式 生成 ISN

双方同时收到 对方的 连接请求(SYN) 的情况 被称为 Simultaneous Open (同时打开)。 这通过 TCP握手中的 额外消息交换 来解决： 双方都 发送 ACK ( 不知道对方也这样做)，双方都 SYN-ACK 请求。然后 开始传输数据。
。。不太理解。 主要是 双方都SYN-ACK请求，是指 前面的 SYN，然后ACK 还是说 一个新的步骤？ 如果是一个新的 SYN-ACK，有什么意义？
Both sides send an ACK (without knowing that the other side has done it as well), and both sides SYN-ACK the requests.

TCP的实现中 包含一个 计时器，知道什么时候 放弃建立 连接。 尝试重新建立连接，通常采用 指数退避，retry次数满或 时间阈值到达 以后，认为 连接不存在。


### TCP options

TCP头的最后一部分是 留给 可能的 TCP option。
初版规范提供了3个 option，后续版本进行了添加。
我们来看下最常用的option

- MSS
  maximum segement size，告知 TCP实现 它会收到的 最大 TCP segment size。通常 IPv4是 1460byte

- SACK
  selective ack，优化了 当许多packet丢失，导致了很多 窗口出现了很多 洞的时候。
  为了防止吞吐量降低，TCP实现 会在 SACK 中告知 sender，它没有收到哪些packet。sender就发送 丢失的 packet，而不是 从上次ack的地方全部重发。

- window scale
  用于突破 window size 的 16bit限制。 当双方 在 握手时 都带有这个 option，那么 window size 就要乘以 这个值。 传输大量数据时，更大的 window size 是非常重要的。

- Timestamps
  这个选项 允许 sender 放置 时间戳 到 TCP segment，可以用来计算 RTT。这个信息可以用来计算 重发超时 的超时时间。


### test TCP handshake

现在，我们有了 普通的 TCP handshake 的mock，它监听了每个端口， 让我们来测试它：

```shell
[saminiir@localhost ~]$ nmap -Pn 10.0.0.4 -p 1337

Starting Nmap 7.00 ( https://nmap.org ) at 2016-05-08 19:02 EEST
Nmap scan report for 10.0.0.4
Host is up (0.00041s latency).
PORT     STATE SERVICE
1337/tcp open  waste

Nmap done: 1 IP address (1 host up) scanned in 0.05 seconds
```

由于 nmap 没有 SYN-scan (它只是等待 SYN-ACK 来决定目标端口是否打开)， 所以很容易 通过 返回一个 SYN-ACK TCP段 来欺骗它，让它以为 我们有一个app在 端口上监听。

。。？没有代码啊。怎么测试？


### 总结

最小可行的TCP握手，只需要 选择 sequence number，设置 SYN-ACK标志，计算TCP segment的 checksum 就可以。

接下来，我们会看 TCP最重要的责任：可靠性。stream 的窗口的 管理，对于 TCP 传输数据 至关重要，它的逻辑可能变得复杂

将app 绑定到 TCP实现， 是通过 socket 完成的。我们会 查看 伯克利的 socket api，来看 我们是否可以 为我们的app mock 它，使得它们(。。socket) 使用 我们自定义的 TCP 实现



## TCP data flow & socket api

http://www.saminiir.com/lets-code-tcp-ip-stack-4-tcp-data-flow-socket-api/

上一篇，我们了解了 TCP header，及 如何建立连接

现在，我们看 TCP 数据传输，及 如何管理。

另外，我们会 从 网络栈 提供一个 接口，应用可以使用这个 接口来进行 网络通信。
我们会使用 这个 socket api，来向 网站发送 简单的 http 请求。


内容
- transmission control block
- TCP data communication
- TCP connection termination
- Socket api
- test
- conclusion
- source

### transmission control block

。。中断好几个月了。。

让我们先看下 记录了 数据流状态 的 变量

TCP 必须跟踪 发送 和 收到的ack 的数据的sequence。为了实现这个，需要对每个 opened 连接 初始化一个 Transmission Control Block (TCB)。


发送端的变量是
- SND.UNA  send unacknowledged
- SND.NXT  send next
- SND.WND  send window
- SND.UP   send urgent pointer
- SND.WL1  segment sequence number used for last window update
- SND.WL2  segment acknowlegement number used for last window update
- ISS  initial send sequence number


接收端的变量
- RCV.NXT  receive next
- RCV.WND  receive window
- RCV.UP   receive urgent pointer
- IRS  initial receive sequence number


还有一些 当前正在处理的 segment 的 helper variable
- SEG.SEQ  segment sequence number
- SEG.ACK  segment acknowledgment number
- SEG.LEN  segment length
- SEG.WND  segment window
- SEG.UP   segment urgent pointer
- SEG.RPC  segment precedence value


### TCP Data Communication

一旦连接建立，开始处理数据流。 TCB 中3个变量 对于 状态的 基础追踪 是非常重要的
- SND.NXT  发送端 需要跟踪 下一个使用的 sequence number
- RCV.NXT  接收端 继续 下一个想要收到的 sequence number
- SND.UNA  发送端 记录 最早的 un-ack 的 sequence number

没有传输发生后，经过一段足够的时间后， 这3个变量的值 会相同。  

。。SND.UNA 有点。 确实相等。 但是之前想的是 unack，那肯定已经发出去了， 就是 已经发生了， 怎么会等于 未发生的 下一个使用 。  但是确实， 在 全部ack的情况下， UNA 是多少？ 就应该是 NXT， 所以 3个能相等。  所以 UNA 和 发送无关， 只是 收到 ACK 后， UNA变成 被ack的 sequence number + 1。  不会和 发送有任何的 关联。


例如，当A 发送数据给 B 时，以下发生
- A 发送 一个 segment，更新 它的 TCB 的 SND.NXT
- B 收到 segment，并 发送 ack，更新 RCV.NXT
- A 收到 ack，更新 SND.UNA      。。确实，和发送无关。

这些变量的 更新值 是 segment 中 data 的 长度。  
。。 ==不是 +1==

上面就是 TCP 控制逻辑的 基础。  

现在，来看下 `tcpdump` 会展示的值

```shell
[saminiir@localhost level-ip]$ sudo tcpdump -i any port 8000 -nt
IP 10.0.0.4.12000 > 10.0.0.5.8000: Flags [S], seq 1525252, win 29200, length 0
IP 10.0.0.5.8000 > 10.0.0.4.12000: Flags [S.], seq 825056904, ack 1525253, win 29200, options [mss 1460], length 0
IP 10.0.0.4.12000 > 10.0.0.5.8000: Flags [.], ack 1, win 29200, length 0
```

10.0.0.4(主机A) 初始化一个连接 从它的 12000端口 到 10.0.0.5(主机B) 监听中的 8000端口

在3次握手之后，连接被建立。 它们内部的 TCP socket 状态变成 ESTABLISHED。  
A的 最初的 sequence number 是 1525252， B的是 825056904


---

```text
IP 10.0.0.4.12000 > 10.0.0.5.8000: Flags [P.], seq 1:18, ack 1, win 29200, length 17
IP 10.0.0.5.8000 > 10.0.0.4.12000: Flags [.], ack 18, win 29200, length 0
```

A 发送一个 包含 17 byte的数据 的 segment， B 发送 ACK segment 表示 ack。  
默认情况下，`tcpdump` 显示的是 相对 sequence number，所以 `ack 18` 实际上是 1525253 + 17

接收端(B) 的 TCP 使用 17 来修改 RCV.NXT

---

```text
IP 10.0.0.4.12000 > 10.0.0.5.8000: Flags [.], ack 1, win 29200, length 0
IP 10.0.0.5.8000 > 10.0.0.4.12000: Flags [P.], seq 1:18, ack 18, win 29200, length 17
IP 10.0.0.4.12000 > 10.0.0.5.8000: Flags [.], ack 18, win 29200, length 0
IP 10.0.0.5.8000 > 10.0.0.4.12000: Flags [P.], seq 18:156, ack 18, win 29200, length 138
IP 10.0.0.4.12000 > 10.0.0.5.8000: Flags [.], ack 156, win 29200, length 0
IP 10.0.0.5.8000 > 10.0.0.4.12000: Flags [P.], seq 156:374, ack 18, win 29200, length 218
IP 10.0.0.4.12000 > 10.0.0.5.8000: Flags [.], ack 374, win 29200, length 0
```

继续 发送数据 和 ACK。  
我们可以看到， length 0 的 segment 只有 ACK 标记， ack sequence number 基于 上次收到的 segment 的 length 来 增加。

---

```text
IP 10.0.0.5.8000 > 10.0.0.4.12000: Flags [F.], seq 374, ack 18, win 29200, length 0
IP 10.0.0.4.12000 > 10.0.0.5.8000: Flags [.], ack 375, win 29200, length 0
```

通过 FIN segment ( 方括号中的F )，B 告知 A ：B没有更多的数据需要发送。 然后 A ack  
。。但是 为什么 A 的ack 会是 375 ？  FIN 的 length 是0啊， ack不应该是 前面的 ack 374 吗？

为了 结束 连接， A 也需要发送 FIN 来告知B A没有数据需要发送。

### TCP Connection Termination

关闭连接，有2种
- RST  强制中断
- FIN  协商中断

基础的场景
- 主动关闭方 发送一个 FIN segment
- 被动关闭方 通过 发送 ACK segment 来 ack它
- 被动关闭方 开始 它自己的 关闭操作 (当没有更多数据需要发送时), 它转化为 主动关闭方 进行连接的关闭
- 一旦 两方都发送了 FIN 到对方 且 都 ack了对方的FIN， 连接关闭

显然，关闭 TCP连接 需要 4个 segment。

TCP 是双向连接的， 只关闭 一个方向的连接， 成为 TCP half-close

packet-switched (分组交换) 网络的 不可靠性，增加了 关闭连接的 难度， FIN可能丢失，使得 连接进入尴尬的状态。linux中 tcp_fin_timeout 控制 在 强制关闭 连接前，它需要等待多少秒。 这个违反了 TCP标准， 但是为了防止 DoS，它是必须的。     
。。 就是 我要关闭，但是对方 ==恶意地 不ACK==，那么 我不可能无限等待的，只能强制关闭。


---

可以通过 RST 标记 来 中断连接。 RST 的原因有很多，比如
- 连接请求 发送到了 不存在的 端口 或 接口
- 其他 TCP 崩溃了，导致 out-of-sync 状态
- 尝试 干扰 已有的 连接

正常的 TCP 数据交换 不会 包含 RST


### Socket API

为了使用 网络栈，为 app 提供了一些接口。   
BSD Socket API 是最著名的。  

传递 socket的类型，协议 来 调用`socket` 方法，可以从 网络栈中 获得 一个socket。  
通常，类型是 AF_INET， domain是 SOCK_STREAM。 这会生成 一个 基于IPv4 的 TCP socket

在获得 socket 后，使用 `connect` 方法 将它 连接到 远程端点。 

连接成功后，就可以使用 `write，read` 来 操作 socket

网络栈 会处理 入队，重传，错误检查，数据重组 。  
对于app， TCP的内部行为是 不可见的。app 可以确信的事情 是 TCP 会负责 发送和接收 数据，当异常行为发生时，会通过 socket api 通知 app。


---

让我们来观察 curl 会进行的 系统调用

```text
$ strace -esocket,connect,sendto,recvfrom,close curl -s example.com > /dev/null
...
socket(AF_INET, SOCK_STREAM, IPPROTO_TCP) = 3
connect(3, {sa_family=AF_INET, sin_port=htons(80), sin_addr=inet_addr("93.184.216.34")}, 16) = -1 EINPROGRESS (Operation now in progress)
sendto(3, "GET / HTTP/1.1\r\nHost: example.co"..., 75, MSG_NOSIGNAL, NULL, 0) = 75
recvfrom(3, "HTTP/1.1 200 OK\r\nCache-Control: "..., 16384, 0, NULL, NULL) = 1448
close(3)                                = 0
+++ exited with 0 +++
```

- 使用 `socket方法` 打开 socket，类型是 TPv4/TCP
- `connect方法` 进行 TCP握手。传递 目的地 和 端口 给这个方法。
- 连接建立后， `sendto方法` 用来 向 socket 写入数据，在这个例子中，是 HTTP GET 请求
- 最后，使用`recvfrom方法` 读取数据

读者 可能注意到，这里每一偶 read，write 系统调用。  
因为，实际上 socket API 不包含这些方法，但是 在socket上可以使用 普通的 IO操作(read,write)。

另外，标准IO操作，如 write,writev,sendfile,read,readv 也可以用于 读写数据

Socket API 包含多个 只用于读写数据的方法。 IO函数也可以用于 读写 socket 的 fd，这增加了 复杂性



## TCP Retransmission

http://www.saminiir.com/lets-code-tcp-ip-stack-5-tcp-retransmission/


现在，我们有了一个 可以和网络上其他host通信的 TCP/IP 栈。  
这个实现 缺少一个 功能： 可靠性

我们目前实现的TCP 不能保证 数据的完整。 甚至 可能由于 握手包的丢失 导致 建立连接失败。


目录
- auto repeat request
- TCP retransmission
- Karn's algorithm
- Managing the RTO timer
- requesting retransmission
- trying it out
- conclusion
- source


### auto repeat request

许多可靠的协议的基础是 ARQ (auto repeat reQuest)

在ARQ中， 接收者需要 为每个收到的数据 发送 ack ， 发送者 如果没有收到 某些数据的 ACK，那么需要重发这些数据。

TCP 维护了 已发送的数据的 sequence number 和 已ack的 sequence number。  
已发送的数据 被放到 重发队列中，并且 对这个数据 启动一个计时器。 在计数器 超时之前，如果没有收到 ack，那么 会进行重发。

TCP 是基于 ARQ 来完成 可靠性的。  
但是还有很多 细节问题，如：发送者应该等待ACK等待多久？这很难回答。  
TCP的一些扩展，如 selective acknowledgement (SACK) ，通过 告知 未收到的 data 来避免 不必要的 通信，来提高性能。


### TCP retransmission

在最初的规范中，TCP的 retranmission 的描述如下：
```text
When the TCP transmits a segment containing data, it puts a copy on a retransmission queue and starts a timer; when the acknowledgment for that data is received, the segment is deleted from the queue. If the acknowledgment is not received before the timer runs out, the segment is retransmitted.
```

但是 最初的 重传超时时间的 计算公式 无法满足所有的 网络环境。  
现在的方法是 RFC6298 的

基本算法很简单。 对于 TCP 发送者，定义下列 变量 来计算 timeout
- srtt  segment 的 RTT(round-trip time) 的 平均值 (smoothed RTT)
- rttvar  RTT的变化值
- rto  重传的超时，毫秒

srtt 就像一个 对于连续RTT的 低通过率的 filter。  
由于可能有 较大的RTT值， rttvar 用于 侦测这些修改 并 阻止它们 影响 averaging function   
另外，假设 时钟的 粒度是 G秒

下面的步骤 来自 RFC6298
1. 在第一个 RTT 测量前： `rto = 1000ms`
2. 对于第一个RTT测量值 `R`：
  ```text
  srtt = R
  rttvar = R/2
  rto = srtt + max(G, 4*rttvar)
  ```
3. 后续的 RTT测量值 r：
  ```text
  alpha = 0.125
  beta = 0.25
  rttvar = (1-beta) * rttvar + beta * abs(srtt - r)
  srtt = (1-alpha) * srtt + alpha * r
  rto = srtt + max(g, 4*rttvar)
  ```
4. 计算出 rto后，如果小于 1秒，则改成 1秒。 可以设定 rto最大值，但这个最大值 必须大于60秒


以前 TCP 的 时钟粒度 通常非常高，从 500ms 到 1秒。  
现在 linux的 TCP 时钟粒度是 1ms

RTO至少1秒，是为了防止 spurious retransmissions (伪重传) ， 如果 太频繁地 重传，会导致 网络 拥塞。  
实践中，许多实现 是 小于1秒的， 比如 Linux 使用 200ms。


### Karn's algorithm

为了防止 RTT 测量值 错误，有一个 强制的算法： Karn'n algorithm ：对于 重传的包，不计算 RTT样本。

即， TCP 发送者 维护了 segment 是否是 重传的，在收到 它们的 ACK时 跳过 它们的RTT计算。

利用 TCP的 timestamp 选项， 对每个 ACK 都可以计算出 RTT。


### Managing the RTO timer

重传timer 的管理非常简单， RFC6289 建议如下的算法
1. 当发送 data segment 时， RTO timer 还没有运行的话， 那就激活它，并设置 timeout 为 rto
2. 当所有 未完成的 data segment 都被 ACk时， 关闭 RTO timer
3. 当 收到 新数据 的ACK 时，重启 RTO timer，值为 rto

当 RTO timer 超时 时
1. 重发 最早的 未ack的 segment
2. RTO timer 超时时间 * 2。
3. 启动 RTO timer

当 RTO timer 超时时间 * 2 ，后续的 测量值 成功生成 时， RTO 会收缩。 当超时时间*2 时，TCP 实现 可能清空 srtt 和 rttvar，等待 下一个ACK


### Requesting retransmission

TCP 通常不会 仅仅 依赖 TCP发送者 的 timer 来 修复 包丢失。  
接收者 也可以 主动告知 发送者 需要重传的 segment

Duplication acknowledgement 是一个算法：`out-of-sequence segments are acknowledged, but by the sequence number of the latest in-order segment`   
。。。感觉就是 不是必须按照 顺序 ack，而是可以 多个segment 一起ack？ 。。不是，下面的 例子，就知道了，  是  收到包就ack，但是 ack的值 是 最近有序的 (就是 中间不能有 空洞的)

在 3个 重复ACK 后，发送者应该 意识到： 它需要 从 重复ACK ack的 sequence number 开始 重传一些数据。

更进一步， SACK 是 重复ACK 的 更复杂版本。  
它是一个 TCP option， 接收者 可以 将 它收到的 sequence number 编码到 ACK 中， 发送者 会注意到 缺失的 segment，重发。



### try it out

。只记录了下命令

```shell
$ iptables -I FORWARD --in-interface tap0 \
	-m conntrack --ctstate ESTABLISHED \
	-j DROP
$ ./tools/level-ip curl news.ycombinator.com
curl: (56) Recv failure: Connection timed out
```

```shell
$ sudo tcpdump -i tap0 host 10.0.0.4 -n -ttttt
```


---

```shell
$ iptables -I FORWARD --in-interface tap0 \
	-m connbytes --connbytes 6 \
	--connbytes-dir original --connbytes-mode packets \
	-m quota --quota 6000 -j DROP
```

```shell
$ ./tools/level-ip curl -X POST http://httpbin.org/post \
	-d "payload=$(perl -e "print 'lorem ipsum ' x500")"
```


2024.06.08 20:57



---




# Linux 网络IO的结构

2024.06.08 20:58

https://zhuanlan.zhihu.com/p/448502965



## TCP 如何发送数据


。。会将 用户数据 切分成 sk_buff， 所以这里就是 分包，粘包的 源头吧。
![8d2c5b3ea7c76657852715fdc3e3b732.png](../_resources/8d2c5b3ea7c76657852715fdc3e3b732.png)


![283e771420c0c88d87e4f917da09ba34.png](../_resources/283e771420c0c88d87e4f917da09ba34.png)

1. 程序调用 write/send，进入 ==内核空间==
2. 内核根据 发送的数据 创建 sk_buff 组成的链表，单个sk_buff 最多包含 ==MSS== 字节。即 使用 sk_buff 将数据切割。 这个 sk_buffer 组成的链表，就是 常说的 ==socket 发送缓冲区==
3. 检测 拥塞窗口 和 接收窗口，判断 接收方是否可以接收新数据。 
  创建数据包 (称为 packet 或 TCP segment)，添加 TCP 头，进行 TCP校验
4. 执行IP路由选择，添加IP头，进行IP校验。 
  通过 QDisc (排队规则) 队列 将 数据包 缓存起来，用来 控制 网络收发的速度
5. 经过排队，数据包 被发送到 驱动，被放入 Ring Buffer 输出队列
6. 网卡驱动 调用 ==DMA== engine 将数据 ==从 系统内存== 拷贝到它自己的内存中
7. NIC 会向数据包 增加 帧间隙 (IFG, inter-frame gap)，同步码(preamble)，CRC校验和。
  当 NIC 发送了数据包，NIC会在主机的 CPU上产生 中断，使得内核确认已发送。

。。NIC 网卡

## TCP 如何接收数据

![95f069284b15bf6d82d7653a59330d96.png](../_resources/95f069284b15bf6d82d7653a59330d96.png)


![f68979c8d0e7964a78f239a396d8ebed.png](../_resources/f68979c8d0e7964a78f239a396d8ebed.png)


1. 收到报文时，NIC 把数据包 写入它自身的内存
  NIC通过CRC校验和 检查数据包是否有效，之后 通过 DMA 把数据包发送到 主机的内存缓冲区，这是 驱动程序 提前向 内核申请好的 一块内存区域。 (sk_buff 的数据缓冲区)
2. 数据包的实际大小，checksum 和其他信息 会保存在 独立的 Ring Buffer 中， Ring Buffer 接收之后， NIC 向主机发送 中断，告知内核有新数据到达。收到中断，驱动(。。这里应该是内核吧)会把 数据包 包装成 指定的数据结构 (sk_buff) 并发往上一层
3. 链路层会检查数据包 是否有效 并解析出 上层的协议 (网络协议)
4. IP层 同样会检查 数据包 是否有效。检查 IP 校验和
5. TCP层 检查 TCP 校验和
6. 根据 TCP 控制块 中的端口号，找到对应的 socket，数据会被增加到 socket 的接收缓冲区，socket接收缓冲区的大小就是 TCP 接收窗口大小
7. 当应用程序 调用 read 系统调用时，程序会切换到 内核区，并把 socket 接收缓冲区中的数据 拷贝到 用户区，拷贝后的数据 从 socket缓冲区中移除。


---

## 缓冲区

socket buffer 是发送缓冲区，接收缓冲区的统称
- 发送缓冲区
  进程调用 send后，内核会将数据 拷贝到 socket的 发送缓冲区中。不管 它们有没有到达目标机器，也不管 何时 发送到 网络，这些都是 TCP 负责的
- 接收缓冲区
  接收缓冲区被TCP 和 UDP 用来缓存 网络上来的数据，一直保存 直到 应用进程 独走位置。 recv 就是把 接收缓冲区 中的数据 拷贝到 应用的内存中，并返回。


socket buffer 的设计应该符合 2个要求
- 保持实际在网络中传输的数据
- 数据在 各协议层传输的过程中，尽量减少拷贝


每个socket被创建后，内核都为其分配一个 socket buffer (抽象概念)， socket buffer 指的是 sk_buff 链表。
sk_buff 是 TCP数据包 的一个 抽象表示， 所以最大不能 超过 MSS。

。。有点。。。
。。应该有 2个 链表吧， 读写各一个， 读的链表是 NIC/内核 append，app从头读。 写的链表是 app append，NIC/内核从头读。


只有 2种情况下 创建sk_buff
- app 往 socket 中写入数据时
- 数据包 达到 NIC 时

数据会拷贝2次
- 用户空间 与 内核空间 之间
- sk_buff 与 NIC 之间 。。。 就是  内核空间 和 硬件 之间



## QDisc

排队规则
是 queueing discipline 的间写。
位于 IP 层 与 网卡的Ring Buffer 之间，是 ==IP层流量控制 的基础==。
QDisc 的队列长度 有 txqueuelen 设置，和网卡关联

内核如果需要通过 某个网络接口发送 数据包，它都需要 按照 为这个接口配置的 QDisc 将 数据包加入队列。  内核会尽可能多地 从 QDisc 中取出 数据包，将它们交给 网络适配器 驱动模块。

物理设备发送数据是有上限的， IP 层需要约束 传输层的行为， 避免 数据 大量堆积， 平滑 数据的发送。


## Ring Buffer

固定尺寸，头尾相连 的缓冲区。
本质是一个 FIFO 队列， 为了解决某些特殊情况下 的竞争问题 提供了 一种免锁的方法，可以避免 频繁的申请/释放内存， 避免内存碎片

这里的 Ring buffer，是指 NIC的驱动程序队列， 位于 NIC 和 协议栈 之间

有2个重要作用
- 可以平滑 生产者，消费者的 速度
- 通过 NAPI 机制，合并以减少 IRQ 次数 (。。IRQ 中断请求)

ring buffer 不存储数据 (。估计是存一个sk_buff的指针) 。不会发生数据拷贝


## 总结

1. 网络IO中， 内核中 只有一个 地方保存 数据，就是 socket buffer
2. socket buffer 就是 sk_buff 的链表，只有 往socket写入 或 数据到达 NIC 时 创建
3. sk_buff 是一个 线性的数据缓冲区， 通过 alloc_skb 和 skb_reserve 申请 和 释放
4. 每个 sk_buff 是固定大小的，和 MTU 有关
5. 数据只有 2次拷贝， 用户空间 与 sk_buff，  sk_buff 与 NIC



## 网络IO中的 零拷贝

### DPDK

处理数据包的 传统方式是 CPU中断方式。
网卡驱动 接收数据包后 通过 中断通知 CPU 处理， 数据通过协议栈，保存在 socket buffer，最终用户态程序 再通过 中断取走数据。 会产生大量 CPU中断， 性能低

DPDK 使用 轮询方式 实现 数据包处理过程。 DPDK 在用户态 重载了 网卡驱动，该驱动在收到 数据包 后不中断 通知 CPU，而是通过 DMA 直接将数据 拷贝到 用户空间， 节省了 CPU中断，内存拷贝 时间。

。。核心管理 硬件， 所以 用户空间应用 只知道 核心，不知道 硬件(驱动)， 硬件 只知道 核心，不知道 用户空间应用。  就是 OS 隔离了 app 和 硬件。 所以 每次 app 需要调用 硬件，就必须 经过 linux核心。 就需要 中断，并且 复制内存
。。dpdk 就直接在 用户态 执行 硬件的驱动， 就是  app 直接调用 硬件， 不需要 核心。 减少 CPU中断 和 不必要的 内存复制。

为了让驱动运行在 用户态， linux 提供 UIO (userspace IO) 机制，使用 UIO 可以通过 read 感知中断， 通过 mmap 实现 和 网卡的通讯


#### 缺点

开发量太大，相当于 要将 整个IP协议底层 实现一遍。




## 磁盘IO的网络IO 的零拷贝

### read + write

。。场景是 从磁盘read，然后 write到 socket

`read(file_fd, tmp_buf, len)`
`write(socket, tmp_buf, len)`

4次 上下文切换， 2次CPU拷贝， 2次DMA拷贝

1. read，write，2次系统调用，每次 从 用户态 切换为 内核态， 再从 内核态 切换为 用户态， 所以 4次 上下文切换
2. 数据由 page cache 拷贝到 用户空间， 再由 用户空间 拷贝到 socket buffer， 2次 cpu拷贝
3. 目前，磁盘和网卡 都是支持 DMA的，所以 从磁盘 到内存， 从 网卡到内存 都是 DMA 拷贝。

。。第三步 指的是  内核 和 硬件(磁盘，NIC) 之间的操作，都是DMA的
。第二步 是 内核 和 用户态 之间的操作。


### mmap + write

`tmp_buf = mmap(file, len);`
`write(socket, tmp_buf, len);`

4次 上下文切换，1次CPU拷贝。
对于大文件 性能高， 小文件需要内存对齐，所以浪费内存 。。这个和下面说的 相反了。这行应该不准。这里是作者在网上搜到的总结。

1. mmap，write， 2次系统调用，4次上下文切换
2. MMU支持下， 数据从 虚拟地址 到 socket buffer 的 拷贝，实际是 page cache 到 socket buffer 的拷贝，所以 1次 CPU拷贝
3. mmap 读取过程中，会触发多次 缺页异常，造成 上下文切换， 所以 越大的文件，性能越差
4. 不存在 浪费内存， mmap本质也是 buffer io， 本来也是 page 对齐的。

。。DMA 没有提到， 这个得 2次，  一次从 磁盘 到 内核， 一次 内核 到 NIC
。。 由于 mmap 的存在， 所以 内核 和 app 是 同一块 内存， 所以 只有1次。  之前是 内核的 page cache 到 app， app 到 socket buffer ，现在  内核的 page cache 和 app  是同一块内存， 所以 page cache 到 app 就少了一次 复制。 直接 app/page cache 到 socket buffer




### sendfile

`sendfile(socket_fd, file_fd, len);`

2次上下文切换，1次 CPU拷贝

1. 一次系统调用，所以 2次上下文切换
2. 2次DMA， 硬盘 -> page cache,  socket buffer -> NIC
3. 1次CPU拷贝， 从 page cache 到 socket buffer
4. sendfile 适合大文件，所有工作 都由 内核完成， 效率高


#### sendfile, splice, tee 区别

- sendfile
  在内核态 从 in_fd 读取数据到 一个内部 pipe，然后 从 pipe 写入 out_fd中
  in_fd 不能是 socket 类型，因为根据 函数原型，必须提供 随机访问
- splice
  类似 sendfile ，但更通用
  需要 fd_in 或 fd_out， 至少有一个是 pipe
- vmsplice
  fd_in 必须是 pipe
  如果是写端 把 iov 部分数据挂载到这个pipe中(不拷贝数据)，并通知 reader有数据需要读取
  如果是读端 则从 pipe中 copy数据到 用户空间
- tee
  需要 fd_in, fd_out 都必须是 pipe，从 fd_in pipe 中读取数据 并挂载到 fd_out 中


sendfile 是否可以用于 https 传输？
很难实现。 http，https 是 应用层协议，位于 用户空间。
http可以在用户空间写入 http 头信息，文件内容的拷贝 由 内核空间完成
https 的加解密 必须在 用户空间完成， 必须进行数据拷贝


### sendfile + DMA gather copy

0次 CPU拷贝

网上总结
在硬件的支持下，sendfile 拷贝方式 不再从 内核缓冲区的数据 拷贝到 socket 缓冲区，而是仅仅 拷贝 缓冲区文件描述符 和 数据长度的 。
这样 DMA 引擎 直接利用 gather 操作 将 页缓存 中的 数据 打包发送到 网络中即可。

笔者认为不太可能。 DMA gather 是指 DMA 运行在 一次单一的 DMA 处理中 传输 数据到 多个 内存区域， 就是 批量操作。
不可能是 磁盘 DMA 到 page cache，然后 page cache 通过 DMA gather 到 网卡。socket buffer 结构是 很复杂的，它 负担 数据跨层传递的 作用，如果 传递过程中 page cache 中的数据被回收 怎么办。
如果可能，也是 直接 磁盘 DMA 到 socket buffer，这种 需要 内核有明确的 api 支持 socket_sendfile。
















































































