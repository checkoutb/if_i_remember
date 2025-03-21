2023-06-25 09:55

[[toc]]


# 模型
OSI七层参考模型。 参考

TCP/IP协议,4/5层。 实际
应用层
传输控制层
网络层
链路层
物理层

。。。
## self
4层是把5层的 链路层和物理层 合并成 网络接口层。

OSI： 应用层，表示层，会话层，传输层，网络层，数据链路层，物理层
TCP/IP 5层：应用层，传输层，网络层，数据链路层，物理层
TCP/IP 4层：应用层，传输层，网络层，网络接口层。

就是 OSI的 应用层，表示层，会话层， 合并为 应用层。

应用层：HTTP, SSH, FTP
传输层： TCP, UDP
网络层： IP, ICMP

从上往下，每过一层，就在头部增加一层信息。

。。。


## 为什么要划分层次

解耦，各层独立变化。

应用层的协议：http,ftp,ssh

## 连接baidu主页

### 新建TCP连接
linux命令
`exec 9<> /dev/tcp/www.baidu.com/80`
9 约等于变量
<> 输入输出流
dev/tcp/www.baidu.com/80 路径
linux中一切皆文件，所以上面代表了 9 持有了 对百度的80端口的 输入输出流

。。Linux中的一个特殊文件: /dev/tcp 打开这个文件就类似于发出了一个socket调用,建立一个socket连接,读写这个文件就相当于在这个socket连接中传输数据。


上面的命令执行后，就拥有一个 一个连接。


### 使用应用层协议
第一行必须GET / HTTP/1.0然后换行符。
```shell
echo "GET / HTTP/1.0\n"  # 这个直接输出 \n，而不是换行

echo -e "GET / HTTP/1.0\n"  # 这个会真的换行。
```

echo 是打印到 命令行。现在 9 持有了连接。

```shell
echo -e "GET / HTTP/1.0\n" 1>& 9     # 输出重定向到 9
# 1 是文件描述符，是 输出流，默认是 当前页面。
# 由于9 是 文件描述符， 所以需要 增加 &， 不然，就输出到 文件 9 中。
```

```shell
cat 0<& 9
# 9 是双向流，所以需要从 9 中读取
# 每个程序 都有 输入流，输出流，报错流
```

可能收到 Connection reset by peer， 
这是因为 echo到9，从9 cat， 这3个命令 时间间隔有点久，导致 baidu 断开了 连接。

应该是 建立连接，后 一段时间不使用，baidu就关闭了连接。
。。确实视频中 是 新建了一个 8 来描述 /dev/tcp/xxxxx/80。 不是使用 原来的 9。 不知道这个 9 应该怎么处理，难道一直 我们这里开着，baidu关闭着 ？


## TCP: 面向连接的，可靠的传输协议
3次握手，4次挥手

### 3次握手
客户端发送 syn 给服务器
服务器返回 syn + ack 给客户端
客户端发送 ack 给服务器

完成握手后，双方才会开辟资源。开辟完后才算建立连接。 这就是 面向连接。

我发过来的东西，你必须确认。 这就是 可靠的。

连接是无力的，虚无的，比如， 3次握手后， 1年没有通信，然后这个连接 是否还有效？ 我们是不知道的。可能有效，可能无效。
通过 心跳。

### 4次挥手

linux命令： `netstat -natp`

IP是确定主机，
端口是用于确定哪个pid(进程，软件)使用这个端口。

客户端的 端口是随机生成的。
服务器的端口 是事先指定的。

挥手是为了断开连接，降低资源消耗。

客户端 fin 给服务器
服务器 ack 给客户端
服务器 fin 给客户端
客户端 ack 给服务器


### 抓包
> 00:58:00

linux： `tcpdump -nn -i eth0 port 80`

eth0 是 网卡的名字，ifconfig 中显式。

执行后，会阻塞。

再开一个终端，`curl www.baidu.com`
会经历 连接的建立，传输，连接的断开。

3次握手
    本机 [S] baidu
    baidu [S.] 本机
    本机 [.] baidu
S是syn， .是ack，代表了对之前的某个请求的 ack，所以 只有第一个请求 是不带.的，其他的都带。

本机 [P.] baidu
baidu [.] 本机
baidu [.]

P 大约代表：本次传输已结束，请处理。


。。
Flags are some combination  of  S  (SYN),  F(FIN), P (PUSH), R (RST), U (URG), W (ECN CWR), E (ECN-Echo) or '.' (ACK), or 'none' if no flags are set. 
。。

。。下面是本地虚拟机中测试的
```text
root@yiqi:/home/aa# tcpdump -nn -i ens33 port 80
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on ens33, link-type EN10MB (Ethernet), snapshot length 262144 bytes
10:44:07.066967 IP 192.168.70.128.53182 > 180.101.50.242.80: Flags [S], seq 3720278239, win 64240, options [mss 1460,sackOK,TS val 614564683 ecr 0,nop,wscale 7], length 0
10:44:07.077147 IP 180.101.50.242.80 > 192.168.70.128.53182: Flags [S.], seq 366047848, ack 3720278240, win 64240, options [mss 1460], length 0
10:44:07.077178 IP 192.168.70.128.53182 > 180.101.50.242.80: Flags [.], ack 1, win 64240, length 0
10:44:07.077317 IP 192.168.70.128.53182 > 180.101.50.242.80: Flags [P.], seq 1:78, ack 1, win 64240, length 77: HTTP: GET / HTTP/1.1
10:44:07.077664 IP 180.101.50.242.80 > 192.168.70.128.53182: Flags [.], ack 78, win 64240, length 0
10:44:07.088587 IP 180.101.50.242.80 > 192.168.70.128.53182: Flags [P.], seq 1:1441, ack 78, win 64240, length 1440: HTTP: HTTP/1.1 200 OK
10:44:07.088595 IP 192.168.70.128.53182 > 180.101.50.242.80: Flags [.], ack 1441, win 63360, length 0
10:44:07.089406 IP 180.101.50.242.80 > 192.168.70.128.53182: Flags [P.], seq 1441:2782, ack 78, win 64240, length 1341: HTTP
10:44:07.089411 IP 192.168.70.128.53182 > 180.101.50.242.80: Flags [.], ack 2782, win 63360, length 0
10:44:07.089585 IP 192.168.70.128.53182 > 180.101.50.242.80: Flags [F.], seq 78, ack 2782, win 63360, length 0
10:44:07.089761 IP 180.101.50.242.80 > 192.168.70.128.53182: Flags [.], ack 79, win 64239, length 0
10:44:07.099537 IP 180.101.50.242.80 > 192.168.70.128.53182: Flags [FP.], seq 2782, ack 79, win 64239, length 0
10:44:07.099554 IP 192.168.70.128.53182 > 180.101.50.242.80: Flags [.], ack 2783, win 63360, length 0
```

这里是 传输控制层。

![1.jpg](../_resources/11.jpg)


网络层，设置IP，port

IP & 掩码 = 网络号
IP & ~掩码 = 主机号



。。这里一段，内容很多。
。。涉及，寻址，路由，网关，路由器，路由表，网卡，arp，等，听得一知半解。
。。无所谓，感觉偏 硬件、网络了。


## ARP

arp 获得 ip 和 mac 地址的映射关系。

`tcpdump -nn -i ens33 port 80 or arp`

```text
15:02:44.261550 ARP, Request who-has 192.168.70.254 tell 192.168.70.128, length 28
15:02:44.263131 ARP, Reply 192.168.70.254 is-at 00:50:56:f8:90:f3, length 46
```

## 下面这些都没有讲，就提了一下，自己百度了

## 拥塞机制，滑动窗口，粘包，DMA驱动，缓冲区，

### 拥塞机制

一般原理：
计算机网络中的 链路容量(即带宽)，交换结点中的缓存和 处理机等，都是网络的资源。 在某段时间，如果对 网络中 某一资源的 需求 超过了 该资源的可用部分，网络的性能就要变坏。这种情况就叫做 拥塞。

网络拥塞往往是由许多因素引起的
拥塞常常趋于恶化
拥塞控制和流量控制 关系密切，但也存在一些差别。
进行拥塞控制需要付出代价

。。有点多
https://blog.csdn.net/LINZEYU666/article/details/117450799

。。就是2边都维护 拥塞窗口， 这个窗口同步地慢慢变大， 一旦出现超时， 这个窗口就 / 2。

主动队列管理 AQM






### 粘包
TCP是 面向 字节流的 传输层协议。 意味着 数据没有边界
UDP是 面向 消息 的传输服务          数据有边界

TCP发送方无法保证 每次对方接收到的 都是一个完整的 数据包。于是就有了 粘包，拆包问题。(UDP没有)

客户端 向服务器 发送2个数据包，可能：
1. 服务器正常收到 2个数据包，没有 粘包。
2. 服务器只收到一个数据包，这个包的 报文段 中包含了 2个数据包的信息，这种就是 粘包。
3. 服务器收到2个，但是包不正确，2个包去掉首部后，一个报文段中包含了 1.5个数据包，另外一个报文段中包含了 半个数据包，这种就是 粘包，拆包。

粘包：客户端发送的多个数据包，在到达服务器时 粘成一个数据包， 后一个数据包的头 禁挨着 前一个数据包的尾
拆包：客户端的数据包 被拆分为 若干部分 发送出去， 服务器收到的包 只是 数据包的一部分内容。

。。header中有 body的长度，所以这个 粘包，拆包，都是在 客户端 主动行为导致的。 并不是 传输过程中，发生的。
。。但是上面说 客户端发送多个，到达服务器时 粘成一个。。。 那就是 中间的 网关，路由器的 行为？
。。擦，下面第二点，说了， 客户端这里 就 粘一起了。我感觉下面的 可信。


#### 原因
1. app写入的数据量大于 TCP 发送缓冲区的 大小，这会发生 ==拆包==。 app调用 系统函数write()，将 app的缓冲区中的 数据拷贝到 TCP socket的 发送缓冲区。而 TCP的 发送缓冲区 有一个 SO_SNDBUF 的 大小限制， 如果 app的 缓冲区的数据大小 > TCP发送缓冲区的大小，那么 数据需要进行 拆包处理， 分多次发送。
2. app写入的数据量 小于 TCP 发送缓冲区的大小，这会发生 ==粘包==。TCP 发送缓冲区足够大，所以会等到有多个数据包时，再组装为一个 TCP报文段，然后通过网卡发送到 网络中去。
    数据包的合并，是在 socket中实现的， 应用层 无法感知到。 调用 sokcet.send()后， OS可能 先将数据 缓存起来，等待后续的数据，而不是立刻发出去。 这就是 缓存发送，可以提高 网络信道的利用率
3. app发送的数据包 大于 MSS (最大报文段长度)时， 发生 ==拆包==。 TCP在传输数据时，是以 报文段 为单位发送数据的， 待发送的数据 是以 MSS 为单位进行分割的。
    mss, maximum segment size，报文段的长度，不包含头的长度， TCP的MSS限定是因为 数据链路层的 数据传输单位： 数据帧，它有一个 最大传送长度的限制，即MTU
4. 接收方不及时读取 接收缓冲区中的数据，将会发生 ==粘包==。因为接收方 先把 收到的数据 存放在 内存接收缓冲区中， 用户进程 从 接收缓冲区 读取数据，如果 下一个数据包 达到时，前一个数据包 还没有被 用户进程取走，那么 下一个数据包 放到内核接收缓冲区时 就和 前一个数据包 粘在一起， 用户进程根据预设的 缓冲区大小从 内核接收缓冲区 读取数据，就一次读取了 多个数据包。


#### 什么时候考虑粘包问题
TCP 是长连接，且 传输的是 结构化数据时， 如：传送的 是一个 结构体类型的数据，由于不知道 结构化数据的 边界，容易导致 粘包问题的出现。

#### 什么时候不需要考虑粘包
1. 如果是 短连接。只进行一次 数据通信过程，通信完成 就关闭连接，这样就不会出现 粘包。
2. 如果传输的是 字符串，文件 等 无结构化数据时，不会出现 粘包。 因为 发送方只管 发送，接收方只管 接收存储 就可以了。

#### 粘包的解决方法
粘包的本质在于 接收方无法 分辨 消息与消息 之间的 边界在哪里。 我们可以通过以下方法给出 边界：
1. 发送定长包。 发送端 将数据包 封装为固定长度 (不够则补0), 接收端 每次 从接收缓冲区中 读取 固定长度的 数据 就 自然而然地 把每个数据包 拆分开来。 (适合定长结构的数据)
2. 包头加上包体长度。 为每个 数据包 添加 包首部。 。。？数据包，但是怎么区分数据包？ 报文头 应该肯定有 表示 报文段 长度的 属性啊。。知道了， 报文段的 第一个字节就是 这个数据包的长度，然后截取出来，然后碰到的第一个字节就是下一个数据包的长度，然后截取。
3. 包尾部设置边界标记。 每个数据包 尾部添加 边界标记，可以使用 特殊符号。但是 如果内容中有 该特殊字符，那么会 误判为 消息的边界。 。 例如，ftp协议采用 \r\n 来识别 消息的边界。。。这个应该不算问题，世界上字符太多了。不过 发送和接收方 都得确认。


## BIO，NIO，Epoll(多路复用)










































