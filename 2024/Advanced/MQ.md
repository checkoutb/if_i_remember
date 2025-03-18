MQ

2024-04-25 17:20

[[toc]]

---
---




# 常见MQ比较
bilibili

- 比较古老的 ActiveMQ
- 用的最多的 RabbitMQ
- ali的 RocketMQ
- 大数据 kafka

## 性能

都是单机 吞吐量qps

ActiveMQ，  6000
RabbitMQ,  12000
RocketMQ, 十万
kafka, 百万

## 数据持久化
都支持
ActiveMQ，RabbitMQ，开启 持久化，性能会下降。
RocketMQ，Kafka 天生支持

## 多语言支持

ActiveMQ，RabbitMQ，kafka 主流语言都支持
RocketMQ，只支持 java

## 综合
ActiveMQ，适合 业务简单，系统老；没有大规模应用的案例。 一般不推荐

RabbitMQ，高可用(erlang)。管理界面好； 很难了解 内部机制, 集群不支持 动态扩展。

RocketMQ，模型简单(只是 生产者-消费者，没有 AMQP协议)， 接口易用， 在ali有大规模应用。性能较好，功能多(定时消息，死信)； 只支持java

kafka，天生分布式，性能最好；运维难度大，不像RocketMQ 只需要几个简单的API 就可以 生产消费。 对带宽 有一定要求，因为多副本集群。 大数据支持(有流式处理)； 没有 定时消息。



---

# kafka工作中使用

bilibili

java，spring boot
maven依赖：spring-cloud-starter-stream-kafka

yaml配置：
- bootstrap-servers: 111.11.11.11:1111,22.22.22.22:2222,33.33.33.33:333
- ack-mode: manual_immediate
- concurrency， 设置为 分区数？
- missing-topics-fatal: false, 主题不存在时，不抛出异常
- key-serializer, 消息的 key的序列化器
- value-serializer, 消息的 value 的序列化器
- acks: 1
- auto-offset-reset: earliest 从最早的可用消息开始消费，即从分区的起始位置开始读取消息
- enable-auto-commit: false  禁用自动提交 偏移量
- key-deserializer, 反序列化器
- value-deserializer, 反序列化器



---
kafka 创建主题
1. 自动创建，不推荐
在kafka 的安装目录 .conf 下找到 server.properties，配置
```property
num.partitions=3  # 默认3个分区
auto.create.topics.enabled=true  # 开启自动创建topic
default.replication.factor=3  # 默认3个副本
```


2. 手动创建
在kafka的安装目录的 bin 目录下执行
```text
// 创建3个分区，3个副本的 名为 zhouye 的主题
./kafka-topics.sh --create --bootstrap-server localhost:9002 --replication-factor 3 --partitions 3 --topic zhouye
```


---
java 发送的时候 指定 topic 和 消息 就可以了。

消费者用 @KafkaListener(topic="xx", groupId="xxx")





---
---


# 如何回答消息丢失，重复，积压问题

bilibili

场景：购买商品时，选择使用积分抵扣金额。
。。有点逗， 交易服务发送 扣减积分msg 到 MQ，积分服务消费消息，扣减积分。
。。。。？一致性不要了？ 超卖 不管了？
。这个看业务了， 100个积分 值不值钱。 但 不可能不值钱啊。


## 消息丢失
1. 怎么知道消息有没有丢失
2. 哪个环节最可能丢失消息
3. 怎么确保不丢失

3个可能丢失数据的地方
1. 生产者 -> MQ，消息在半路丢失
2. MQ，mq重启
3. MQ -> 消费者

解决方法
1. ack
2. MQ 持久化， 副本， 同步flush

检查消息是否丢失
1. 生产端，生成消息时，设置一个 全局唯一/全局递增的 版本/ID
2. 消费端，作对应的版本校验
  可以使用 拦截器，spring 的 AOP。 通过拦截器 检测 版本号的连续性。  多生产者，多消费者 要注意


## 重复消费

ACK 返回时丢失，导致 消息再次出现在 MQ中。

幂等性

数据库保存 消息ID，消费执行状态。


## 积压

消费能力不够。

- 为什么会积压， 积压是性能问题
- 考虑整个链路
- 解决

和生产者没有问题，MQ的性能也绝对够的，所以 肯定是 消费者 出问题。


关闭其他app。
扩容
消费端的逻辑优化




---



# rocket mq (介绍，和kafka对比，架构) 7min

BV1m7421Z7fN: 什么是RocketMQ，和kafka区别，架构


延迟处理，比如定时外卖，到时间点，才会将订单投递给商家，让商家处理。

rocket mq 接收来自生产者的消息，并对每个消息进行分类，每一类是一个 topic。 消费者根据需要订阅topic

rocket mq的架构参考了 kafka的架构，并对架构做了简化，但增加了 功能。

## (对比kafka)在架构上做减法

为了提高性能，kafka将 单个topic拆分为 多个 partition。  
为了提高扩展性，kafka 将 同一个topic的 partition 部署到不同的 broker上。  
为了提升可用性，为partition增加了 副本  
为了协调和管理 kafka集群的数据消息，使用 zookeeper 来协调节点


rocket mq
1. 简化协调节点
   kafka中，zk会和 broker通信，维护kafka集群消息； 当添加一个broker时，其他结点能立刻感知到 新添加的broker  
   kafka有很多功能，所以较重，在 rocket mq中，移除了 kafka，使用了 NameServer，用更轻量的方式管理消息队列的集群信息。  
   kafka在 2.8后 也可以移除 zk，使用raft 来进行一致性 ，这就是 kraft 或 Quorum 模式

2. 简化分区
   kafka中，将topic 拆分为多个 partition，以提高并发度。  
   rocket mq 也将 topic拆分，拆分为 queue (而不是 partition)  
   kafka的partition中保存了 完整的信息，而 rocketmq 的queue只保存一些简要信息 (比如消息偏移offset)， 消息完整数据 放到了 名为 CommitLog 的 文件上， 通过offset可以定位到 CommitLog 上的 某条消息  
   所以 kafka的消费者只需要读一次即可， rocket mq 需要2次 (一次offset，一次真正的数据)


3. 底层存储
   kafka的 partition 下面还有 segment，每个segment 可以认为是 一个小文件，将数据写入到 partition，本质上是将数据写入到 segment。  
   kafka对 单个segment文件是顺序写， 但 topic变多，partition变多，segment也变多，所以 会同时向多个文件 写入，会从 顺序写 劣化为 随机写。  
   rocketmq 将单个 broker 下的 多个 topic的数据 全部写到一个 逻辑文件 CommitLog 上，消除 随机写。


4. 简化备份模型
   kafka将partition分散到多个 broker中，并为 partition配置副本。将partition分为 leader 和 follower ，主从会进行同步，本质上是 同步 partition下的 ssegment 文件。   
   。。对啊，小文件同步比较简单，commitlog 怎么同步？  
   rocketmq将所有数据 写到了 一个CommitLog上，所以不能向 kafka那样 按 partition 粒度同步。  
   rocketmq直接同步 commitlog文件。以broker为粒度 区分主从。



## (对kafka)在功能上做加法

1. 消息过滤
   kafka支持通过 topic 将数据进行分类，无法再进一步划分。即 消费者必须得到 topic中的所有数据，然后消费者自己进行过滤。  
   rocket支持 对消息打标记tag， 消费者可以根据tag 过滤所需要的数据。


2. 事务
   kafka支持事务：多个消息，要么一起成功发送到 broker，要么一起失败。
   rocket 的事务功能更强，可以： 多条数据的 业务逻辑校验+发送 要么 一起成功，要么一起失败。

3. 增加延迟队列
   kafka无
   rocketmq 通过 half消息 来实现

4. 增加死信队列
   kafka源生不支持

5. 消息回溯
   kafka可以通过调整offset 来让 消费者从某个地方开始消费
   rocketmq 可以调整 offset 和时间


---

## kafka性能优于rocketmq

BV1Zy411e7qY

kafka每秒17w
rocketmq 每秒10w

因为0拷贝，  
kafka使用了 sendfile， 而rocketmq使用的是 mmap，所以kafka更快。

### map

mmap需要3次拷贝，比最原始的 少了一次 (节约了 内核缓冲功能区 -> 用户空间)
1. 磁盘 -> 内核缓冲区
2. 内核缓冲区 -> socket缓冲区
3. socket缓冲区 -> 网卡

2次系统调用： mmap + write
4次 内核态，用户态的切换


### sendfile

2次拷贝
1. 磁盘 -> 内核缓冲区
2. 内核缓冲区 -> 网卡  (使用SG-DMA)

1次系统调用
2次 空间切换



0拷贝是指 0 CPU拷贝， 发生的拷贝都是 DMA执行的，CPU不需要操作。

sendfile 返回了 发送了多少字节
mmap 返回了 内容的指针

rocket的部分功能需要 了解具体的消息内容，进行额外的处理(如 死信队列)，所以 无法使用 sendfile


kafka的优化还有：批处理，数据压缩， 这些 rocket 也有。


### 如何选择

大数据场景，(接触到 spark，flink) 使用kafka
其他 使用 rocketmq















