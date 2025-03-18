
来自k8s

2023-09-25 17:10

https://etcd.io/docs/v3.5/dev-guide/


[[toc]]


# ETCD介绍—etcd概念及原理方面分析

https://zhuanlan.zhihu.com/p/405811320


## 介绍
etcd作为一个受到ZooKeeper与doozer启发而催生的项目，除了拥有与之类似的功能外，更专注于以下四点。
- 简单：基于HTTP+JSON的API让你用curl就可以轻松使用。
- 安全：可选SSL客户认证机制。
- 快速：每个实例每秒支持一千次写操作。
- 可信：使用Raft算法充分实现了分布式。

分布式系统中的数据分为 控制数据 和 应用数据。
etcd的使用场景默认处理的数据都是控制数据，对于应用数据，只推荐数据量很小，但是更新访问频繁的情况。

应用场景有如下几类：
场景一：服务发现（Service Discovery）
场景二：消息发布与订阅
场景三：负载均衡
场景四：分布式通知与协调
场景五：分布式锁、分布式队列
场景六：集群监控与Leader竞选

举个最简单的例子，如果你需要一个分布式存储仓库来存储配置信息，并且希望这个仓库读写速度快、支持高可用、部署简单、支持http接口，那么就可以使用etcd。
目前，cloudfoundry使用etcd作为hm9000的应用状态信息存储，kubernetes用etcd来存储docker集群的配置信息等。

提供配置共享和服务发现的系统比较多，其中最为大家熟知的是[Zookeeper]（后文简称ZK），而ETCD可以算得上是后起之秀了。在项目实现，一致性协议易理解性，运维，安全等多个维度上，ETCD相比Zookeeper都占据优势。

## etcd vs zookeeper

- 一致性协议： ETCD使用[Raft]协议， ZK使用ZAB（类PAXOS协议），前者容易理解，方便工程实现；
- 运维方面：ETCD方便运维，ZK难以运维；
- 项目活跃度：ETCD社区与开发活跃，ZK已经快死了；
- API：ETCD提供HTTP+JSON, gRPC接口，跨平台跨语言，ZK需要使用其客户端；
- 访问安全方面：ETCD支持HTTPS访问，ZK在这方面缺失；


## ETCD的使用场景

和ZK类似，ETCD有很多使用场景，包括：
- 配置管理
- 服务注册与发现
- 选主
- 应用调度
- 分布式队列
- 分布式锁


## 工作原理

ETCD使用Raft协议来维护集群内各个节点状态的一致性。
简单说，ETCD集群是一个分布式系统，由多个节点相互通信构成整体对外服务，每个节点都存储了完整的数据，并且通过Raft协议保证每个节点维护的数据是一致的。

每个ETCD节点都维护了一个状态机，并且，任意时刻至多存在一个有效的主节点。
主节点处理所有来自客户端写操作，通过Raft协议保证写操作对状态机的改动会可靠的同步到其他节点。



## ETCD接口
ETCD提供HTTP协议，在最新版本中支持Google gRPC方式访问。具体支持接口情况如下：
- ETCD是一个高可靠的KV存储系统，支持PUT/GET/DELETE接口；
- 为了支持服务注册与发现，支持WATCH接口（通过http long poll实现）；
- 支持KEY持有TTL属性；
- CAS（compare and swap）操作;
- 支持多key的事务操作；
- 支持目录操作





# Etcd详解(作用原理及应用场景)

https://mikechen.cc/22705.html

Etcd 是云原生架构中重要的基础组件，由 CNCF 孵化托管，Etcd 在微服务和 Kubernates 集群中不仅可以作为==服务注册与发现==，还可以作为 ==key-value 存储的中间件==。


Etcd 的目标是构建一个高可用的分布式键值数据库，具有以下特点：
1. 简单：安装配置简单，而且提供了 HTTP API 进行交互，使用也很简单；
2. 键值对存储：将数据存储在分层组织的目录中，如同在标准文件系统中；
3. ==监测变更==：监测特定的键或目录以进行更改，并对值的更改做出反应；
4. 安全：支持 SSL 证书验证；
5. 快速：根据官方提供的 benchmark 数据，单实例支持每秒 2k 读操作；
6. 可靠：采用 Raft算法，实现分布式系统数据的可用性和一致性；


## 架构

![446409501649ea3b7004d4518575a9eb.png](../_resources/446409501649ea3b7004d4518575a9eb.png)


从架构图中我们可以看到etcd 主要分为四个部分：

1. HTTP Server
    用于处理用户发送的 API 请求，以及其它 Etcd节点的同步与心跳信息请求。
2. Store
    用于处理 Etcd支持的各类功能的事务，包括数据索引、节点状态变更、监控与反馈、事件处理与执行等等。
3. Raft
    Raft 是强一致性算法的具体实现，是 Etcd的核心。
4. WAL
    Write Ahead Log，预写日志系统，是 Etcd的数据存储方式，Etcd就通过 WAL 进行持久化存储，这个与MySQL持久化存储机制类似。


## 工作流程
Etcd的工作流程，大致分为如下4步：
1. 通常一个用户的请求发送过来，会经由 HTTP Server 转发给 Store 进行具体的事务处理;
2. 如果涉及到节点的修改，则交给 Raft 模块进行状态的变更、日志的记录;
3. 然后再同步给别的 Etcd节点以确认数据提交;
4. 最后进行数据的提交，再次同步。


## 应用场景

和Zookeeper类似Etcd有很多使用场景，主要包括以下5类：

1. 服务注册与发现
2. 消息发布订阅
    生产者可以往 Etcd中注册 topic 并发送消息，消费者从Etcd中订阅 topic，来获取生产者发送至 Etcd中的消息。
3. 负载均衡
    后端多组相同的服务提供者可以经自己服务注册到 Etcd中，Etcd并且会与注册的服务进行监控检查。
    服务请求这首先从 Etcd中获取到可用的服务，然后对此多组服务发送请求，Etcd在其中充当了负载均衡的功能。
4. 分布式锁
5. 分布式队列





# doc

## data model

https://etcd.io/docs/v3.5/learning/data_model/

etcd被设计用来提供一个： 偶尔修改数据，提供可靠的watch query 的 可靠的 存储。

etcd 提供了 k-v对的 历史版本，来支持 简单的 snapshot 和 查询历史数据 功能。

上面的情况，非常适合 persistent, multi-version, concurrenct-control 数据模型。

etcd 存储数据 到 multiversion persistent k-v store.

k-v 对是不能修改的， 只能新增一个。 (。。才能保证 多版本)

为了防止 由于维护历史版本 导致 数据量增长过快， store 会压缩历史版本。
。。这个不太清楚 怎么压缩的。
the store may be compacted to shed the oldest versions of superseded data. 。。翻译不出。。


### logical view

store 的逻辑视图 是一个 平坦的 二进制key 空间。
key 空间 有 按照 key的 字典顺序 排序的 index， 所以 range query 消耗不大。

key space 维护这 多个 ==revision==。
key space 创建的时候，初始 revision 是1。
每个原子的 变更操作 ( 比如，一个包含多个操作的 事务操作 ) 创建一个 新的 revision 。 历史数据不会被修改，依然可以被访问。

revision 也有索引， range 是高效的。

如果 store 被压缩 来节约 空间， 在 压缩 revision 之前的 revision 会被移除。
If the store is compacted to save space, revisions before the compact revision will be removed

。。 ？ 那么 我要的 历史版本，其不是 没了？
。。 好像也是，etcd 不是数据库。 而且数据库也不会 提供历史数据啊。
。。 历史数据 已经毫无意义了， 比如 3天前的 app注册信息 已经没有意义了。(。。当然，服务注册中心 的 数据 只有 现在起效的 才有用)

---------------

一个 key 从 create 到 delete 的跨度 被称为 一个 generation， 每个key 可能有多个 generation (至少有一个)

==创建==一个 key 会增加 key 的 ==version== 
。。啊，上午的 create-revision, mod-revision,version 的含义搞清楚了。
。。好像不是， 我 一直 put ，version 会增加了， 并不是 创建吧， 因为 etcd 没有修改啊， 都是创建。 

version 从1 开始。

如果当前 revision 中不存在 key，那么会 增加 key 的 version 。
。。但是上午，一直 put 相同的 k-v， k的 version 还是增长了啊。
。。下面有 ： 修改。

==删除== key 会导致 key墓碑， 将 version 充值为0，来结束 key的当前 generation，

key的每次==修改== 都会增加 version。所以，在 key的一个 generation中，version 是 递增的。

压缩时， 在被压缩的 revision 之前的 generation 都会被 remove， 在 revision 之前 设置的 value，除了最后一个以外，都会被删除


### physical view

存储k-v对 到 b+树

store 的 每个 revision 只包含了 和 上一个 revision 相比的 ==delta== 。

一个 revision 可能对应了 树中的 多个key。

k-v对的 key 是一个 3元组，(major,sub,type)
- major 是 这个key 的 store revision
- sub 区分同一个 revision 中的 不同key
- type 是 对于特殊值的一个可选前缀

k-v对的 value 是 对于上一个 revision 的 差异。所以是 delta

b+树 按照 key的 二进制 顺序 排序。

通过 revision delta 进行 range lookup 是非常快的。这样就可以 快速找到 2个 revision 之间的 修改。

压缩会移除过时的 k-v对

-----------
etcd 在内存中 还有一个 辅助 btree index ，来加速 对key的 range 查询。

btree index 中的 key 是 用户用到的 key。 value 是 指针，指向 b+树中的 modification。
压缩会移除 dead pointer

总的来说， etcd 从 b树 获得 revision 信息，然后 使用 revision 作为key 从 B+树 fetch value。

![6dd6b73055dc7078baeaf07465c390ad.png](../_resources/6dd6b73055dc7078baeaf07465c390ad.png)


----------------

## etcd client design

etcd server 经过多年的注入错误测试 证明了它的 健壮性。

client 和 server 间 需要复杂的协议 来确保 故障条件下的 正确性 和可用性。

理想情况下， etcd server 提供 一个 包含许多物理机器的 逻辑 集群视图， 客户端 实现 failover 。


----------
术语

- clientv3
- clientv3-grpc1.0
- clientv3-grpc1.7
- clientv3-grpc1.23， 都是官方客户端，只不过 底层 grpc 不同。
- balancer，客户端 load balancer, 实现了 retry 和 failover。 etcd client 应该 在 多个 endpoint 间 自动 负载均衡。
- endpoints， client可以访问的 etcd server 组成的list。 通常是 一个集群中的 3或5个 url地址
- pinned endpoint，当配置多个 endpoint时， <=v3.3 的 client balancer 只选择一个 endpoint 并建立 TCP 连接，来减少 对于 cluster 的 open 的connection 的数量。 在 3.4 中， balancer 轮询 pinned endpoint，这样 负载更加平均。
- client connection， 和 etcd server 已建立的 TCP 连接， 比如通过 gRPC Dial。
- sub connection， gRPC SubConn 接口，每个 sub connection 包含了 地址组成的list， balancer 从已解析的地址中 创建一个 SubConn。 gRPC ClientConn 可以映射为 多个 SubConn (比如 example.com 解析为 2个 sub-connection 的 10.10.10.1 和 10.10.10.2 ) 。 etcd 3.4 balancer 使用 内部解析器 来 为 每个 endpoint 建立一个 sub-connection
- transient disconnect，当 gRPC 服务器 返回 code unavailable 状态时。


-------------
客户端的需求
- Correctness，正确，
- liveness，活性，(。健壮)，server有问题时，client应该能后备方案，不能 死锁等待 一个已经下线的服务器。
- effectiveness，高效，使用最少资源完成。 endpoint切换后， 之前的 TCP 应该被 优雅地关闭。 failover机制 应该能快速地 找到下一个 连接的 副本，而不是 在 已经失败的 node 上 retry。
- portability，可移植， 需要有 清晰的文档， 并且 对于其他语言 是可实现的。 不同语言间的 error handling应该 一致。 etcd完全 致力于 gRPC，所以 实现 应该和 gRPC的 设计目标 一致。  新版本客户端应该 兼容老版本。


-------------
### client overview

etcd client 实现了 下面的 组件：
- balancer, 建立 gRPC连接 到 etcd 集群
- API client, 发送 RPC 到 etcd server
- error handler, 决定 是 在失败的node上 retry 还是 切换endpoint

不同的语言有 很大的差异，比如： 
- 如何建立 初始的连接 (如 配置TLS) 
- 如何 encode 并发送 Protocol Buffer message 到 server
- 如何处理 stream RPC
- 等

但是，从服务器返回的 error 是相同的， 所以 错误处理 和 retry策略 应该是相同的。
比如，server返回：`rpc error: code = Unavailable desc = etcdserver: request timed out`， 这是一个 临时错误 (transient error), 需要retry。
如果server 返回 `rpc error: code = InvalidArgument desc = etcdserver: key is not provided`， 这意味着 请求是 非法的， 不应该 retry。

go client 可以通过 `google.golang.org/grpc/status.FromError` 来转换错误。
java 使用 `io.grpc.Status.fromThrowable`


-----------------
### clientv3-grpc1.0: Balancer Overview

。。这个就不需要了吧。
。。简单看下。

配置多个 etcd endpoint 时， 维持 多个TCP conn
选择一个 地址， 使用它 来完成 所有的 client request。
pinned addredd 会一直维持， 直到 client close。
当client 收到 error， 随机选一个 其他的，然后 retry。

。被选择的 address 被称为 pinned address
。。 所以如果 这个 地址有问题， 那么 每次请求 都要 2次？
。。就是 即使 error， 也不会 切换 pinned server， 确实， error，有些是业务错误， 不需要切换 server 啊。

。。不过看图上， balancer 连接 非 pinned server 是 reserved(保留的) TCP conn
。。没问题， 搞成 reversed 了， 这个是 反转的， 上面是 保留的，   所以有点搞不懂了。

局限
打开了 多个 TCP 连接， 可以提供 更快的 failover，但是需要 更多的资源。
balancer 不理解 node 的 health 状况 和 它的 集群成员身份。所以 它可能被 一个故障 或 分区 的 节点卡住。


------------
### clientv3-grpc1.7: balancer overview

只维持 一个 TCP 连接，连到 选择的 etcd server 上。

当 有多个 集群endpoint时， client 先尝试 连接到 它们全部。 只要有一个 conn 成功建立， balancer pin 它的地址， 关闭其他的 TCP conn。
pinned address 会一致维持到 client close。
来自 服务器 或 client 网络 fault 的 error 会被发送到 error handler.

client error handler 获得 gRPC 服务器的 失败后， 根据 error code 和message 决定： 在相同endpoint上retry 还是 切换endpoint。
transient 的错误， 意味着 服务器稍后可以恢复， 所以 在 同一个endpoint 上 retry。
如果错误 不是 transient 的，那么 切换 endpoint。
。。这个切换 看起来是 永久的。 并不是这个 请求的。 不知道。

Stream RPC (比如，Watch 和 KeepAlive)，通常 request 的时候 不带timeout。
相反，客户端可以 定期发送 HTTP/2 ping 来检查 pinned endpoint 的 状态。
如果 server 没有对 ping 进行 响应， 那么 balancer 会切换 endpoint

。。
当客户端与服务器长时间保持链接而不传输消息时客户端为了检测服务端是否还在连接就发送ping帧进行询问，服务器也必须返回一个响应ping帧，这个ping帧同时还可以计算服务器到客户端的延迟。
。。。



局限
发送 http2 keepalives 来 从 streaming request 中检测 disconnection。
如果是一个 简单的 gRPC server ping 机制，没有对 集群成员关系 进行推理，那么 无法检测到 网络分区。 由于 分区的grpc server 能响应 ping，所以 balancer 会被这个 分区node 卡住。
理想情况下， keepalive ping 发现分区，然后触发 endpoint switch， 在 request timeout 之前。

balancer 维持者 不健康的endpoint 的list。 断开的 address 会被加入到 不健康的 list 中，并且 在 等待时间 结束前 被认为是 不健康的， 等待时间 被 硬编码为 dial timeout，默认5秒。
balancer 对于 是否健康，可能误判，比如 endpoint可能在 被加入 不健康list后 立刻恢复了，但是 5秒内依然不可用。

。。图上有一种是， 节点A 失败后，立刻恢复了，但是进入了5秒的 等待期， 结果 B和C 也都失败了， 导致 没有 节点可用( 虽然 实际上 A 是可用的，但是在5秒的等待期中) 


------
gRPC Go 已经切换到了新的 balancer 接口。
etcd客户端收到 一些轻微 影响。 为了获得更好额定支持 ，最好和 最新的 gRPC版本同步



-------------------
### clientv3-grpc1.23: Balancer Overview

clientv3-grpc1.7 和 旧的 gRPC接口 耦合很紧密，导致 gRPC的更新 会 破坏 client 的行为。

clientv3-grpc1.23 的主要目标是 ==简化== balancer failover 逻辑： 只要 client和 当前endpoint 失去连接，就 轮询 下一个endpoint， 而不是 维护 一个 不健康endpoint 的list。
它不会使用 endpoint status，所以 不需要复杂的 状态追踪。

当获得 多个 endpoint时，1.23会创建多个 sub-connection (一个 endpoint一个 sub-connection)，  1.7只创建一个 连到 pinned endpoint 的 connection。

在一个 5node 的集群中， 1.7只需要一个 TCP conn， 1.23需要5个。

由于 维持了 TCP conn Pool， 1.23 需要更多资源， 但是也提供了 更灵活的 负载均衡，有着更好的 failover 性能。

默认的 均衡策略是 轮询， 非常容易 就可以扩展为 其他类型。( 如 power of two, pick leader 等)

1.23 使用了 gRPC 解析组 和实现 balancer picker policy， 以便 将复杂的 均衡工作 委托给 上游的 gRPC。
1.7 手工处理 每个gRPC连接 和 balancer failover，这增加了 复杂性。

1.23 在 gRPC interceptor chain 中 实现了 retry， 这个 chain 会自动处理 gRPC 内部 error，并 使用 更高级的 retry 策略，如 backoff。
1.7 手动 处理 gRPC错误。


局限
缓存 每个endpoint 的状态，可以 提升性能。比如，balancer 可以ping 每个 server 来 维护一个 health list，在轮询的时候 使用这个list。
客户端 侧的 keepalive ping 不会 处理 网络分区， streaming request 可能被 分区node 卡住。
目前，retry逻辑 是通过 拦截器手工处理的， 可以通过 gRPC 官方的retry 来简化。



## etcd learner design

https://etcd.io/docs/v3.5/learning/design-learner/

通过 配置成员关系 来解决常见挑战。

-----------
背景
成员关系的重新配置是 最大的操作挑战 之一。

1. 新的集群成员 使得leader 过载
新加入的 etcd成员 是没有数据的，所以 需要从 leader 获取更新。 leader的 网络可能因此而过载， 降低 甚至阻塞 leader 传递心跳 给follower。 导致 follower 选举超时，启动leader选举。

2. 网络分区
网络分区会发生什么？
取决于 leader 分区。 如果 leader 所在分区依然维持了 法定人数，集群可以继续提供服务。

2.1 leader 孤立
leader 和 其他node 都断开连接，会发生什么？
leader 监控了 每个follower。 当leader 失去了 法定人数 的连接，它会 恢复为 follower，这会影响 集群的可用性。


2.2 集群分裂3+1
集群本来有3个node，现在新增一个node，新增后发生了网络分区，变成了3+1。会发生什么？
3节点的集群 增加一个节点后， 法定人数 是3。 如果新node在 leader的分区中，那么 leader 维持者 法定人数，所以 集群可以继续工作，不会影响可用性。

2.3 集群分裂2+2
本来3节点的集群，加入一个新node，然后网络分区，分成 2+2 ，会发生什么？
leader需要 2个follower，以满足 法定人数3，但是由于网络分区，只有一个follower，所以leader 恢复为 follower。
。。但是这样应该永远无法恢复了啊。

2.4 法定人数丢失
如果 先分区，然后 新node加入，会发生什么？
3节点集群，已经分区成了2+1，当一个新node加入时，法定人数从2变成3。现在，这个集群只有2个节点，不满足法定人数，所以开启 投票。
由于 增加成员操作 会修改 法定人数个数，所以 ==如果要替换不健康的node，要先移除node，而不是先增加node==

1节点的集群，增加一个node，法定人数从1变成2， 当之前的leader 发现不满足法定人数，会立即触发 leader选举。 这是因为 添加成员 是 2步过程， 首先 执行添加命令，然后启动新节点进程。


3. 集群错误配置(cluster misconfiguration)
更糟的情况是， 一个新增的集群的 配置是错误的。
成员关系重新配置 是 2步的： 执行`etcdctl member add`，然后 启动一个 指定URL的 etcd server 进程。 member add 命令的 处理是 和url 无关的，即使 url的值 是错误的。
如果第一步的时候，使用的是 无效的 url，第二步 不可能 启动 新etcd。
一但 集群丢失的 法定人数， 没有办法 恢复之前的 成员关系的。

一个错误配置 可以导致 集群不可用。
此时，需要手动 重新创建集群，通过 `--force-new-cluster` 标记。
etcd 是 k8s 的关键服务，短暂的停机 也会对 用户造成巨大的影响。
怎么样才能使得这些操作更简单？
从上面的场景可以看到，leader 选举 是 集群可用性的最大影响因素，我们能否不改变 法定人员的数量 来减少 成员重配置 带来的破坏性？
一个新节点是否可以是 空闲的 直到它的数据和 leader 一致为止？
成员关系错误配置 是否可以 恢复？ 是否可能 错误的 member add 不会导致 集群失败？
添加新成员时，用户是否应该担心网络拓扑？
成员和api 的工作 是否可以 不受 分区影响？

### Rafe learner
为了减轻 可用性gap， raft 引入了 新的node状态： ==Learner==， 它没有vote权 (不会记入法定人数)，直到它的数据和leader一致为止 (即使一致了，也需要手动转换)。

v3.4中的功能
`member add --learner` 命令 用来增加一个 新的 learner。

当learner 追上了 leader的进度，learner 可以被 提升为一个 voting member，通过 `member promote` API，然后会 计入 法定人数。
etcd server 会检查 promote 请求，确保 learner的 数据 和 leader 是同步的，不同步 会拒绝。

learner 没有选举权，没有被选举权，不会给client提供 读写服务 (client 的 balancer 不能 路由 请求到 learner)，允许状态检查。 这意味着 learner 不需要 发送 read index 请求到 leader。

etcd 限制了 一个集群 中 learner 的数量，避免 太多的日志复制 导致 leader 过载。
learner 不会自动 升级为 follower， 必须 手动提升。
etcd 提供了 learner 的状态信息 和 安全检查。



### 未来版本中的计划功能
- Make learner state only and default: (默认只能从learner开始？)
    新成员的状态 默认为 learner 可以极大提升 成员关系重配置的安全性，因为 learner不会修改 法定人数。 错误的配置可以被 回滚，而不会失去 法定人数

- Make voting-member promotion fully automatic
    一但learner 的数据 和 leader一致了，那么集群可以 自动提升 learner。
    etcd 要求 用户 指明 阈值，一但 满足，learner 可以提升自己为 follower。

- Make learner standby failover node:
    learner 作为备用节点加入到 集群， 当 集群可用性受到影响时，自动升级为follower。 (。。就是出现2+2的情况，但是不可能配置4个节点，可能需要多个learner)

- Make learner read-only:
    在弱一致性模型中，learner只能 从leader 接收数据， 不能处理写请求，在没有共识开销的情况下，处理 读请求，可以极大降低 leader的负载，但可能提供 过时的数据
    在强一致性模型中，learner 从 leader 请求 read index，以提供 最新的数据 来处理读请求， 不能处理 写请求。


### learner vs. Mirror Maker
etcd 通过 watch API 实现了 "mirror maker" 来 持续转发 key的创建和更新 到 指定的集群。
mirroring 在完成初始化后 ，通常只有 较低的延迟开销。
learner 和 mirroring 的 共同之处 在于， 都可以用来复制现有的只读数据
但是 mirroring 不能保证 线性化。 由于网络断开，之前的k-v 可能会被丢弃，客户端需要 验证 watch response 以获得 正确的排序。 所以 mirror 中不能保证 ordering。

如果要 低延迟 和 低一致性开销， 用 mirror。
要保留 所有历史数据 和 顺序， 用 learner。


### 附录：v3.4中 learner 的实现

etcd客户端 为 learner node 增加了一个 `MemberAdd` API。
etcd server 处理 `pb.ConfChangeAddLearnerNode`  类型的 成员修改
一但命令被应用， 一个带有 `etcd --initial-cluster-state=existing`标记的server 加入到 集群。
learner 没有 投票权，没有 被投票权。
etcd server 限制 learner 的数量 为 1： 因为learner越多，leader需要传播的 数据也越多。
client 可以发送请求到 learner， 但是 learner会拒绝 除了 serializable read 和 memeber status 以外的所有请求。 这是为了简化实现。  在未来，learner 可能被 扩展为 一个 只读server。
client balancer 必须提供 方法 来排除 learner endpoint。
client sync member 调用 应该考虑 learner 节点。所以应该 client endpoint update 调用。

`MemberList` and `MemberStatus` responses should indicate which node is learner.



## etcd v3 authentication design
https://etcd.io/docs/v3.5/learning/design-auth-v3/

为什么不复用 v2的 auth system？
v3 使用了 gRPC 用来 传输， 而不是 v2的 RESTful。

新的协议提供了 更好的设计可能， 例如，v3可以对 connection 进行 auth， 而 v2 只能 对每个 request 进行 auth。


功能要求
- 对每个conn进行auth，而不是每个request
  - gRPC的auth 是通过 用户id + 密码 实现的
  - auth策略更新后，auth必须刷新。
- 功能要尽可能 像v2一样 简单和易用
  - v3提供了一个 flat key space，v2提供的是 字典结构。 内部match需要权限检查
- 要比 v2有更强的 一致性保证

主要的变更
- client 在 发送 已认证的请求前， 必须创建 仅用于 auth的 专用conn
- 增加 许可信息 ( user id 和 已认证的revision ) 到 raft command ( `etcdserverpb.InternalRaftRequest`)
- 每个请求 在 状态机层 检查 许可，而不是 api 层。


### Permission metadata consistency
。。简单抄几个概念。。

对metadata 的读写 需要所有node 同意。 一个node的失败 可以阻止整个集群。 所以 有node down了，就不可能完成 读写。

v2的时候，如果压力很大，配置的修改 可能无法立刻扩散出去。


> Inconsistent permissions are unsafe for linearized requests

Therefore, the permission checking logic should be added to the state machine of etcd


### design 和实现

#### Authentication
开始时，client必须创建一个 gRPC conn， 专门用来 验证 它的 user id和密码。
etcd server 会返回一个 authentication reply。 验证成功，response包含了 token， 验证失败，则是一个 error。

用来获取 token 的conn 用完后，被关闭。 不能用来携带 token凭证，因为 gRPC 无法在 conn 创建 (grpc.Dial() )后 附加 RPC凭证 到conn上。


#### Authenticate() RPC 的Impl 的 笔记
Authenticate() RPC 基于 user name 和password 生成 token。
etcd 通过 go的 `bcrypt` 包 来 存储 和比较 密码。 在设计中，bcrypt 的 密码校验 需要消耗大量CPU，在传统的服务器上需要 100ms。 所以在 状态机上 进行 校验，会导致 性能问题： etcd集群 每秒 只能处理 10个 Authenticate() 请求。

为了更好的性能，v3 auth机制 在 etcd的 API层 校验 密码，它可以在 raft外 并行。
但是这可能导致 潜在的 time-of-check/time-of-use (TOCTOU) 权限失效：
1. client A 发送 Authenticate()
2. API层 处理完了 Authenticate() 的 密码校验 部分。
3. client B 发送了 ChangePassword(), server 处理完成
4. 状态机层 继续 A 的 Authenticate() 剩下部分。
5. server 返回 成功 给 A
6. 现在 A 的密码是过时的，但是 通过了 认证。

为了防止这个问题， API层 基于 auth store的 revision 执行 version number validation。
在 密码校验期间，API层 保存了 auth store 的 revision， 密码校验成功后，API层 对比 之前的 revision 和 auth store 最新的 revision， 如果 不同，则说明 密码被人修改了， server 会 再次 校验密码。



### 在API层 决定(resolve) token
在 Authenticate() 的 身份认证后， client 可以像 不使用 auth一样 创建一个 conn。
除了现有的初始化外，client 必须 将 token 和 新创建的conn 进行关联。 `grpc.WithPerRPCCredentials()` 提供了这样的功能。

客户端的 每个 认证后的请求 都有一个 token。 server端 通过 `grpc.metadata.FromIncomingContext()` 来获取 token。 server可以 看到 谁发出的请求，用户是几时 验证通过的。 这些信息 在 API 层 放到了 raft log (`etcdserverpb.InternalRaftRequest`) 的 header (`etcdserverpb.RequestHeader.Username and etcdserverpb.RequestHeader.AuthRevision`) 中


### checking permission in the state machine
在 状态机的 apply 阶段，`etcdserverpb.RequestHeader` 中的 auth 信息 被检查。
这一步检查 用户是否有权限 访问 auth store中 最新 revision 的 请求的key。
。。感觉就是检查是不是有权限访问 key。


### token的 2种类型： simple 和 JWT
simple token 不是为 生产环境 设计的。它的token 没有 加密签名，server 必须跟踪 token-user 的对应关系。

生产环境应该使用 JWT token，因为它 有加密前面和校验。
JWT是无状态的，它的token 包含了 metadata,如 username, revision, 所以服务器不需要 记录 token 和 metadata 之间的 映射关系。


### KVS models 和 file system models 的不同

v3 是 KVS，不是 文件系统。 所以 权限可以看作是 对 某个key， 或某个 key range。 这意味着， 允许 对不存在的key的权限 是可能的。
用户需要注意 无意中导致的 权限许可。
在文件系统中(如 Chubby 或 Zookeeper)， inode (索引节点) 包含了 权限信息。所以 对不存在的 key 的 权限许可 是不可能的。

和 文件系统不同，v3 的模型 需要 多次查询 metadata。 最坏的情况下，会查询完 用户的所有 权限 (key 和 interval)。
这个消耗无法避免，因为 v3 的 flat key space， 不同于 file system(每个 inode包含 权限信息)。
实际使用中， 这个消耗 不是问题，因为 metadata 非常小， cache的 代价很低。



---------------
## ==etcd API==

https://etcd.io/docs/v3.5/learning/api/


### gRPC service

发送给 etcd server的 每个 API request 都是 一个 gRPC remote procedure call.
etcd 中的 gRPC 按照 功能分类。

处理 etcd key space 的 service：
- KV， 创建，更新，fetch，删除 k-v对
- Watch， 监控 key上发生的修改
- Lease， 处理 client 的 keep-alive 消息

管理 集群的 service：
- Auth， 基于role 的 auth 机制
- Cluster， 提供 成员关系 信息 和 配置
- Maintenance， 获取恢复快照，对存储进行 碎片整理，返回每个 node的状态信息



### request and response

etcd中的 所有 RPC 都有相同的格式。
每个RPC 都有一个 `Name` 方法，接收 NameRequest 作为参数，返回 NameResponse。
例如， `Range` RPC:
```go
service KV {
  Range(RangeRequest) returns (RangeResponse)
  ...
}
```

#### response header
etcd API 的 所有 response 都有一个 附加的 response header，包含了 集群元数据
```go
message ResponseHeader {
  uint64 cluster_id = 1; // 生成这个response的 集群id
  uint64 member_id = 2; // 生成这个response的 memeber id
  int64 revision = 3; // 生成这个response时的 k-v存储的 revision
  uint64 raft_term = 4; // 生成这个response时的 member的 raft term
}
```

app可以读取 Cluster_ID 或 Member_ID，来确保 它连接到了 期望的 集群/机器

可以使用 Revision，来知道 k-v store的 最新的 revision。 在 指定历史版本进行查询时 非常有用。

可以使用 raft_term 来检测 集群在 何时完成了 leader选举


---------
### key-value API

k-v api 操作 存储在 etcd 中的 k-v 对。

#### key-value pair
k-v 对时 k-v api 可以操作的 最小单位。
每个k-v对都有 多个字段，定义在 protobuf format中：
```go
message KeyValue {
  bytes key = 1;       // byte格式的key，不允许空
  int64 create_revision = 2;    // key的最后一次创建时的 revision
  int64 mod_revision = 3;   // key的最后一次修改时的 revision
  int64 version = 4;    // key的 version，删除操作会 重置为0。任何修改都会增加version
  bytes value = 5;  // byte格式的 value
  int64 lease = 6;  // 附加到这个 key上的 lease的 id，如果 lease 是0，那么没有 lease 附加。
}
```

除了key和value， etcd 还附加了 额外的 revision metadata 作为 key message 的一部分。这个revision信息 使得 按照 创建和修改的 时间来 对key 进行排序， 对于管理分布式的并发的sync 很有用
etcd client 的 分布式共享锁 使用 创建的 revision 来等待 锁定所有权。
类似的，mod revision 用来 检测 软件事务内存读 的冲突，以及 等待 leader选举的更新。


#### Revision
etcd 维护了 一个 64位的 集群生效的 计数器： store revision， 当每次 key space 被修改时，它会增加。
revision 就像一个 全局逻辑锁， 将所有的 update 排序，

当涉及到 etcd 的 ==MVCC== 时， revision 更加有用。
mvcc模型使得 k-v store可以查看 某个 revision的 历史数据。
集群的拥有者 可以 配置 retention 策略。
通常 etcd 通过 timer 来 丢弃 老数据， 通常， etcd 集群 保存 被代替的 key 数小时 (。就是 数小时后删除 那么 不是最新的数据 )。 这 提供了 对于 client 长时间的 断开连接 的可靠处理。。。(。。一堆，，就是 可以从上次 断开时的 revision 继续读)


#### key range

etcd 数据模型 对 flat binary key space 上的 所有 key 进行了 index。
使用区间 来标识 key list。
[a, a+1)，比如`['a', 'a\x00')` 查询单个key
`['a', 'b')` 查询所有 a 开头的 key

按照约定，request 的 range 是 `key` 到 `range_end` 。`key`是range的 第一个key，； `range_end` 是 range的 最后一个key 的 下一个。
如果 range_end 空或没有， range就变成了 只包含 key 。
`\0` 代表无限， 所以 key 和 range_end 都是 `\0` 时， range 包含 所有的key，
如果range_end 是`\0`，range 包含 所有>= `key` 的 key



#### Range

通过 Range API调用 来 从 k-v store 中 获得 key
Range API 接受 一个 RangeRequest:
```go
message RangeRequest {
  enum SortOrder {
	NONE = 0; // default, no sorting
	ASCEND = 1; // lowest target value first
	DESCEND = 2; // highest target value first
  }
  enum SortTarget {
	KEY = 0;
	VERSION = 1;
	CREATE = 2;
	MOD = 3;
	VALUE = 4;
  }

  bytes key = 1;    // range start
  bytes range_end = 2;  // range end + 1
  int64 limit = 3;  // key的最多数量， 0无限
  int64 revision = 4;   // k-v store 的 时间点， <=0 就是最新， 如果这个 revision 已经被压缩了，那么返回 ErrCompacted 作为 response
  SortOrder sort_order = 5; // 有序请求的 顺序
  SortTarget sort_target = 6;   // 排序用到的 k-v的 属性
  bool serializable = 7;    // 使用 serializable member-local read. 默认，Range 是 linearizable。。。 一大堆。。
  bool keys_only = 8;   // 只返回key，不要value
  bool count_only = 9;  // 只返回 个数
  int64 min_mod_revision = 10;  // 最小的mod revision， 移除所有<它的。
  int64 max_mod_revision = 11;  // >
  int64 min_create_revision = 12;   // <
  int64 max_create_revision = 13;   // >
}
```

Range call 返回一个 RangeResponse 给 client

```go
message RangeResponse {
  ResponseHeader header = 1;
  repeated mvccpb.KeyValue kvs = 2; // 匹配range 的 k-v对 的 list，如果 Count_Only 设置了，那么 这里是 空
  bool more = 3;    // 表明是否有 更多的 key， 如果 limit 被设置了的话
  int64 count = 4;  // 匹配range 的 key的 总数。
}
```

#### ==Put==
Put() 保存 k-v 到 k-v store。
```go
message PutRequest {
  bytes key = 1;    // key
  bytes value = 2;  // value
  int64 lease = 3;  // lease 的id， 0代表没有
  bool prev_kv = 4; // 设置后，response会返回 更新前的 k-v
  bool ignore_value = 5; // 设置后，更新key，不更新value，如果key不存在，error
  bool ignore_lease = 6; // 设置后，更新key，不更新lease，如果key不存在，error
}
```

```go
message PutResponse {
  ResponseHeader header = 1;
  mvccpb.KeyValue prev_kv = 2;  // 如果request 中设置了 prev_kv，这个就有值
}
```


#### delete range
使用 `DeleteRange` call
```go
message DeleteRangeRequest {
  bytes key = 1;
  bytes range_end = 2;
  bool prev_kv = 3; // 返回被删除的 k-v 对
}
```

```go
message DeleteRangeResponse {
  ResponseHeader header = 1;
  int64 deleted = 2;        // 被删除的key的个数
  repeated mvccpb.KeyValue prev_kvs = 3;
}
```


#### Transaction
transaction 是在 k-v store 上的 原子的 If/Then/Else 的构造。
用于防止 并发更新，构建 ==CAS== 操作，开发 高层的 并发控制。

transaction 可以自动处理 一个请求中的 多个操作。
对于 k-v store的修改，这意味着， store的 revision 只增加一次， 这个 事务产生的 所有 event 的 revision 都是一样的。
但是，在一个 事务中 对于 一个key 的 多次修改 是被 禁止的。

事务 由一组 类似if 的 comparison 连词 保护，
每个 comparison 检查 store 中的 一个 key。 可以检查：
- value 是否存在
- 和给定的 value进行对比
- 检查 key 的 revision 或 version。
2个不同的 comparison 可以对 相同 或不同的 key 操作。
所有的 comparison 被 原子方式 应用。
如果 所有的 comparison 都是 true，那么 事务成功，etcd 执行 事务的 then 模块， 否则是 事务是失败，执行 else 模块。

每个 comparison 被 编码为 一个 Compare 消息
```go
message Compare {
  enum CompareResult {
    EQUAL = 0;
    GREATER = 1;
    LESS = 2;
    NOT_EQUAL = 3;
  }
  enum CompareTarget {
    VERSION = 0;
    CREATE = 1;
    MOD = 2;
    VALUE= 3;
  }
  CompareResult result = 1;
  // target is the key-value field to inspect for the comparison.
  CompareTarget target = 2;
  // key is the subject key for the comparison operation.
  bytes key = 3;
  oneof target_union {
    int64 version = 4;
    int64 create_revision = 5;
    int64 mod_revision = 6;
    bytes value = 7;
  }
}
```

在 comparison 模块 处理完后， 事务 使用 request 模块。 是 RequestOp 的list
```go
message RequestOp {
  // request is a union of request types accepted by a transaction.
  oneof request {
    RangeRequest request_range = 1;
    PutRequest request_put = 2;
    DeleteRangeRequest request_delete_range = 3;
  }
}
```

完成后， 使用 Txn API call 发布事务，
```go
message TxnRequest {
  repeated Compare compare = 1;
  repeated RequestOp success = 2;
  repeated RequestOp failure = 3;
}
```

```go
message TxnResponse {
  ResponseHeader header = 1;
  bool succeeded = 2;
  repeated ResponseOp responses = 3;
}
```

```go
message ResponseOp {
  oneof response {
    RangeResponse response_range = 1;
    PutResponse response_put = 2;
    DeleteRangeResponse response_delete_range = 3;
  }
}
```


### Watch API
Watch API 提供了 基于event的接口 来完成 异步 监控 key的修改。
etcd watch 从指定的 revision(当前 或 历史都可以 ) 开始 watch key，然后将 key的 update 用stream 返回给 client。

#### Event
对每个 key 的修改事件 被表现为 `Event`。
Event 提供了 更新事件 和 更新类型
```go
message Event {
  enum EventType {
    PUT = 0;
    DELETE = 1;
  }
  EventType type = 1;   // PUT， DELTET
  KeyValue kv = 2;  // 事件的 KeyValue， PUT时是传入的k-v对，DELETE时是被删除的k-v对。
  KeyValue prev_kv = 3; // 在这个 event 之前的 k-v对。
}
```


#### Watch stream
watch 是一个长时间运行的 request， 使用 gRPC stream 来 stream event data。
watch stream 是 双向的。 client 可以 写入到stream 也可以从stream读取
一个 watch stream 可以 通过 将event打上 watch id 来进行多路复用。

watch 对 event 有 3个保证
- ordered， event 按照 revision 排序。
- reliable， event的序列 不会丢失 任何 event。
- atomic， event的list 保证包含了 完整的 revision。 同一个 revision 中的 对不同key的 更新 不会被 分开。


client 通过 `WatchCreateRequest` 在 `Watch`返回的 stream 上创建 watch

```go
message WatchCreateRequest {
  bytes key = 1;
  bytes range_end = 2;
  int64 start_revision = 3; // 可选，不设置的话，从 watch creation response header revision 开始。
  bool progress_notify = 4; // 设置后，watch 会周期性地收到 WatchResponse，但是可能没有event (如果真的没有的话)。 如果 client 想从 最近的 已知的 revision 恢复 断开的 watcher，这很有用。。。。。 etcd server 根据 当前 server 负载 来 决定 发送notification 的 频率

  enum FilterType {
    NOPUT = 0;
    NODELETE = 1;
  }
  repeated FilterType filters = 5; // 在服务器端， 将这些 event type 过滤掉。
  bool prev_kv = 6; // 设置后，watch 收到 event 发生前的 k-v 数据。
}
```

```go
message WatchResponse {
  ResponseHeader header = 1;
  int64 watch_id = 2;   // 和这个 response 关联的 watch的 id
  bool created = 3; // 设置为true，如果是一个 create watch 请求 的response， client应该保存这个 id，然后等待 从 stream 接收 event。 发给 已创建的 watcher 的 所有 event 都有 相同的 watch_id。
  bool canceled = 4;    // 设置为true，如果 是 cancel watch 请求的 response， cancel 后的 watcher 不会有 event 传过来。
  int64 compact_revision = 5; // 如果 watch 的是 压缩的 revision, 则设置 最早/小 可用的 历史 revision。 这个 watcher 会被拒绝， 需要 新开一个 watch。

  repeated mvccpb.Event events = 11; // 和 watch id 对应的顺序的新event 列表
}
```

##### stop watch

```go
message WatchCancelRequest {
   int64 watch_id = 1;
}
```


### Lease(租约) API
lease 是一个机制 来 检测 client 的liveness。
集群 给予 lease 一个 time-to-live (TTL)。 如果etcd 集群 在 给定的TTL中 没有收到 keepAlive ，则 lease 过期。

为了将 lease 绑定到 k-v store， 每个 key 可以 被 attach 最多 一个 lease。
当一个 lease 过期或 被撤销， 这个lease 的所有 key 会被删除。
每个过期的 key 会生成 一个 delete event 到 event history。

#### Obtain lease
`LeaseGrant` API call

```go
message LeaseGrantRequest {
  int64 TTL = 1;    // ttl, time to live, second
  int64 ID = 2; // the requesting lease 's id, if 0, etcd choose an id
}
```

```go
message LeaseGrantResponse {
  ResponseHeader header = 1;
  int64 ID = 2; // lease 的 id
  int64 TTL = 3;    // ttl， second
}
```

------------

```go
message LeaseRevokeRequest {        // revoke request
  int64 ID = 1;
}
```

#### keep alive
使用 `LeaseKeepAlive` API call 创建的 双向 stream 来 刷新 lease。

想要 刷新 lease 的 时候，发送：
```go
message LeaseKeepAliveRequest {
  int64 ID = 1;
}
```
。。这里是 refresh， 但是 是 keepAlive, 一次刷新多少秒？

```go
message LeaseKeepAliveResponse {
  ResponseHeader header = 1;
  int64 ID = 2;
  int64 TTL = 3;
}
```



## KV API guarantees

etcd 是一个 一致的，持久的， 支持小事务的 k-v store 
k-v store 通过 KV API 暴露。
etcd 尝试 确保 分布式系统的 强一致性 和 持久性 

这章列举 KV API 的 保证。

我们会考虑如下API
- read api
  - range
  - watch
- write api
  - put
  - delete
- 组合api (read-modify-write)
  - txn
- lease api
  - grant
  - revoke
  - put (attach a lease to a key)


-------
etcd定义

> operation completed

> revision

-----------
保证提供

> atomicity

> durability

> isolation level and consistency of replicas

--------
Granting, attaching and revoking leases


## etcd versus other key-value stores

https://etcd.io/docs/v3.5/learning/why/

etcd名字来源于 linux的 /etc  和 distributed system

。。。跳了。
主要是和 zk， consul， NewSQL 对比。




## Glossary 词汇表
https://etcd.io/docs/v3.5/learning/glossary/


Member
A logical etcd server that participates in serving an etcd cluster

Peer
Peer is another member of the same cluster.




























































