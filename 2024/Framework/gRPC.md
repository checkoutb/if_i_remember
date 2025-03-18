
2024.06.07 09:27

[[toc]]


---
---

15.1


# 官方简易教程

https://grpc.io/docs/


## introduction

介绍 gRPC 和 protocol buffer

gRPC 使用 protocol buffer 作为 接口定义语言(IDL, interface definition languate) 和 它底层的消息交换格式


gRPC中， client app 可以 像调用本地方法一样 调用 其他机器上的 server app 的方法， 可以使你更方便地 构造 分布式app。

gRPC 基于： 如何定义 service， 如何指明 一个可以被 远程调用的 方法 及 方法的参数和返回值。

在server 端，server 实现接口，并运行一个 gRPC server 来处理 client 的请求。
在client 端，client 有一个 stub，这个 stub(客户端) 提供了 和 像server端一样的方法

。。下面 是由 3个 不同语言构成的。
![07716923439aeec25480646b1e34d02c.png](../_resources/07716923439aeec25480646b1e34d02c.png)


---

### protocol buffer

默认情况下， gRPC 使用 protocol buffer， 这是一个 成熟的开源的组件 用于 序列化 已结构化的数据。
下面是一个快速介绍。

第一步是定义 需要序列化的 数据的结构 到 proto文件， 这是一个 普通的文本文件。

```text
message Person {
  string name = 1;
  int32 id = 2;
  bool has_ponycopter = 3;
}
```

有了 proto 文件后，你需要使用 protocol buffer compiler `protoc` 来 根据 proto文件 生成 你指定的语言的 data access class。 
为每个 field 生成 简单的 accessor 比如 name()，set_name() 。

如果你选择使用 C++，对上面的例子使用protoc 会得到 Person类。

---

你可以定义 gRPC service 在 普通的protoc文件中， 方法的 参数和返回值 必须是 被定义的 protocol buffer message

```text
// The greeter service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

gRPC 使用 带有gRPC插件的 protoc 来生成 代码。 你会生成 gRPC client，server 的代码。



#### protocol buffer version

教程大部分例子都是 proto3

。。proto3 2016年的


## 核心概念，架构，生命周期


### service 定义

gRPC 基于 如何定义 service，指明方法 及 形参 和返回值。

默认使用 protocol buffer 作为 接口定义语言 来描述 service 接口 和 payload 的结构
也可以使用其他

```text
service HelloService {
  rpc SayHello (HelloRequest) returns (HelloResponse);
}

message HelloRequest {
  string greeting = 1;
}

message HelloResponse {
  string reply = 1;
}
```

gRPC 可以定义4种 service method
- unary(一元) RPC， client 发送 一个request 到 server，获得 一个response， 就像 普通的 方法调用
  `rpc SayHello(HelloRequest) returns (HelloResponse);`
- Server streaming RPC， client 发送请求到 server， 获得 一个stream 用于 读取返回的 消息序列。client 从stream 一直读取 直到 没有消息。  grpc 保证 消息是有序的。
  `rpc LotsOfReplies(HelloRequest) returns (stream HelloResponse);`
- Client streaming RPC， client 写入一系列消息，并将它们 发送到 server，复用提供的流。 client 完成写入后，它等待 server 读取它们 并 返回 响应。 gRPC保证 消息 是有序的。
  `rpc LotsOfGreetings(stream HelloRequest) returns (HelloResponse);`
- Bidirectional streaming RPC， 两端 通过 read-write stream 都发送 消息的序列。 这2个stream 操作是独立的。所以 client 和 server 可以 任意安排 读写。 比如，server 可以一直等待 直到 收到所有的 消息，然后开始 返回response， 也可以 获得一个request 就返回一个 response。
  `rpc BidiHello(stream HelloRequest) returns (stream HelloResponse);`


### using api

server 定义在 proto文件中，所以 gRPC 提供 protocol buffer compiler plugin 来 生成 client端 和 server 端 的 代码。

- 在server端，server 实现方法，运行 gRPC server 来处理 client 的请求。 gRPC 解码request，执行service method，编码 response
- 在client端，client有一个 本地对象 叫 stub，它实现了 和 server相同的method。 client只需要 调用 本地的stub上的方法，这些方法 会 将参数封装为 proto message，发送 请求到 server，返回 server 的 proto response。

### sync vs. async

大部分语言的 gRPC 都支持 sync 和 async， 查看 具体语言的 教程。


### RPC life cycle

本节可以 近距离观看：当client 调用 server 时 会发生什么。

unary rpc
最简单的 RPC， 发送一个请求，收到一个请求
1. client 调用 stub方法， server 被通知：RPC 被调用了，并且 server 获得 client的 metadata，方法名，可能的 deadline
2. server 可以 返回 它的最初的metadata (必须在 任何 response 之前发送) 或 等待 client 的 request message。 哪个先发生 是 app 特定的。
3. 一旦 server 获得 client 的 request message， 它就执行必要的代码 来 新建 response。 如果方法成功，就返回 response，并带上 status detail (状态码 和可选的 状态信息) 和 可选的 trailing metadata。
4. 如果 响应的状态是 OK，那么 client 收到 响应，这就完成了 调用。

server streaming rpc
和 unary rpc 类似， 除了 server 返回 消息流 作为 response 给 client。
发送完 所有 消息后， 发送 server 的 status detail (状态码 和 可选的 状态信息) 和 可选的 trailing metadata 给 client。

client streaming rpc
和 unary rpc 类似，除了， client 发送 消息流 到 server。
server 返回 一个消息(并带上 status detail 和 可选的 tailing metadata) ， 通常，并==不是== 必须在收到 client 的所有消息后 才可以返回。

bidirectional streaming rpc
在 双向streaming RPC 中， 调用被初始化 by： client 调用方法，server接受到 client的 metadata，方法名，deadline。
server 可以选择 返回 它的 原始 metadata 或 等待 client 开始 streaming message

客户端，服务器 的 stream 处理 是 独立于 app 的，由于 2个 stream 是独立的， client 和 server 可以 任意 顺序读写。



### deadline/timeout

gRPC 允许 client 指定 它愿意等多久。 超时的话，RPC 被中断 ，返回 DEADLINE_EXCEEDED 错误。
在服务器端，server 可以查看 某个RPC 是否已超时，或 多久后超时。

指定 超时 是 各语言不同的。  一些语言 是 指定等待的 时间段， 一些语言是 指定时间点，且 有些语言 有默认的 deadline 有些没有。


---

### RPC termination 

gRPC中，client 和 server 是 独立地，本地地 决定 调用 是否 成功， 它们的结论 可能并不一致。
这意味着，可能 服务器 认为 RPC 调用成功， client 认为 RPC调用失败。
也可能：server 在 client 发送完 request前 就 决定 完成调用。


### cancel an rpc

client 和 server 都可以 在任意时刻 cancel ， 会立刻中断RPC ， cancel之前的修改 ==不会 rollback==


### metadata

是有关 一个 RPC call 的 信息。 以 k-v对 的列表的 格式。 k是字符串， v 是字符串或 二进制数据

key 是大小写敏感的，只能包含 ascii 字母，数字，`-` `_` `.` ，不能以 grpc- 开头。
value 为 二进制数据时， key 以 `-bin` 结尾 。

访问 metadata 是 各语言不同的

### channel

gRPC channel 提供了一个 到 gRPC server 的 连接，通过 指定 host 和 端口。
在创建 client stub 时 使用。 client 可以 指定 channel 的参数来修改 grpc 的默认行为，比如 是否开启 消息压缩。
channel 是有状态的，包括 connected 和 idle。

gRPC 怎么关闭 channel  是各语言不同的。 一些语言 允许 查询 channel size




## c++ basic tutorial

https://grpc.io/docs/languages/cpp/basics/


。。代码太长了。。





















# proto3

https://protobuf.dev/programming-guides/proto3/


















































































































