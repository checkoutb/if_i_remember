

2024-03-25 11:15

[[toc]]

---
---



# 概述

基于zookeeper

## 角色
- producer，消息生产者
- consumer，消息消费者
- topic，消息主题
- partition，主题内分区
- broker，消息服务器
- group，消费者组

## 持久化

持久化数据保存在 kafka的日志文件中。

## topic
topic 是消息的逻辑分类
1. 每条消息 属于且仅属于 一个topic
2. producer发布时，必须指定发布到哪个topic
3. consumer消费时，必须指定消费哪个topic


## partition
topic是逻辑的概念，partition是物理的概念(即物理上进行了分离，分布在不同的机器上)

如果没有partition，一个topic对应的消息 在分布式集群服务组中，就会分布不均匀，导致某个服务器压力大。
有partition后，假设一个 topic分为10个分区，kafka内部根据一定的算法 将 10个分区 分散到 不同的服务器上，来 均衡 压力。

producer发送消息的时候，如果没有指定发送到哪个分区，kafka就会根据 负载均衡算法 计算 目标分区
consumer 消费消息的时候，也可以指定分区
`consumer.assign(Arrays.asList(new TopicPartition(topic, partition)));`

partition的目的是：通过多分区实现负载均衡的效果，提高kafka访问吞吐率。

kafka是基于文件存储的，配置多个 partition来将 消息分散到 多个broker上，可以避免 文件尺寸 达到 单机磁盘的上限。


将一个topic 分为 多个 partition，可以提高 吞吐率，partition越多，可以容纳的consumer 也越多。

## consumer
consumer 必须自己 从 topic 的 partition中 拉取(poll)消息。
一个consumer 连接到 一个broker 的 partition，从中读取消息

拉取消息的时候可以选择 批量拉取 或 单条拉取。

一个消费者可以订阅 多个 topic

### 订阅topic
3种
- subscribe 传入topic，可以一个或多个
- subscribe 可以接受正则
- assign 订阅时 指定分区

subscribe 会 自动再均衡，分区分配关系会自动调整。
assign 不会自动再均衡


## group
consumer group

同一个group中的 consumer 不能订阅 相同的 partition

当group 中只有一个 consumer时， 这个consumer 将收到 所有 partition 的消息。
当group 有2个consumer时， kafka会 将 一半的 partition 的消息发送给 一个 consumer，另一半的 partition的消息 发送给 另一个 consumer
当group 的 consumer 大于 这个topic的 partition 数量时，  多余的 consumer 是闲置的， 不会收到消息。


2个group之间 没有任何 影响。


# 一致性

## 读是顺序一致性





# partition

一般来说，partition是3-10个， 很多企业是2或3个。

理论上，一个topic可以有无数个 partition

实际上有以下限制
- 集群的分区总数限制: kafka集群中所有的partition的总数是有限制的，由 broker 的 `num.partition` 参数决定， 默认 `1000`。 即意味着 如果集群中 某个topic有500个partition，那么其他topic的 partition加起来不能超过500.
- 单个topic的partition数量: 理论上可以很大，但是实际存在限制: 单个topic的partition数量通常建议在 几千以内，因为 管理大量partition会增加 元数据的复杂性 和 网络开销。
- 性能和资源: 过多的partition 会对性能产生负面影响，尤其是 broker资源(CPU,内存,网络带框)有限的情况下。 每个partition都意味着更多的IO和数据管理开销
- kafka不同版本: 不同版本 在处理大量partition时 性能和稳定性不同。 新版的 对大规模的partition支持更好。


## 设置/修改 partition数

创建topic时指定 partition数

```shell
kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 3 --partitions 10 --topic my-topic
```

---

修改topic的partition数

**减少partition数 会导致数据丢失**

```shell
kafka-topics.sh --alter --bootstrap-server localhost:9092 --topic my-topic --partitions 20
```






# ACK机制

## 生产者

生产者发送消息后，ACK机制 决定了 何时认为消息已经成功发送到 kafka。

==生产者可以在 发送时指定==ACK的级别

ACK级别分为3种
- acks=0， 异步发送，消息发送后offset增加，生产者不会等待kafka的确认，消息发送后就被视为已送达。
- acks=1， leader收到 一个replica 的ack后增加offset，然后返回ack给生产者，生产者继续处理
- 默认: acks=-1 (或 acks=all)， leader收到 所有replica 的ack后增加offset，然后返回ack给生产者，生产者继续处理


---

## 消费者

通过 AckMode 控制消息确认的方式

。。这里和 下面的代码不同。。 感觉代码更对。
- 默认: AckMode.COUNT, 每次从kafka拉取指定数量的消息后进行确认。适用于大量数据流
- AckMode.BATCH, 拉取消息，直到 达到指定的批量大小 或 时间限制 后ACK。 适用于 小批量数据
- AckMode.TIME, 拉取消息，直到 指定时间限制 后ACK。适用于处理 实时数据流
- AckMode.CALLBACK, 通过回调函数 实现消息确认逻辑。



- 每条记录处理完后 ack
- poll的每批记录处理完后 ack
- poll的每批记录处理完后，距离上次ack 大于 TIME时 ack
- 处理的record大于等于COUNT时 ack
- 处理的记录大于某个值 或 距离上次ack大于TIME时 ack
- 处理完后 手动ack
- 手动立刻ack
```Java
public enum AckMode {
    // 当每一条记录被消费者监听器（ListenerConsumer）处理之后提交
    RECORD,
    // 当每一批poll()的数据被消费者监听器（ListenerConsumer）处理之后提交
    BATCH,
    // 当每一批poll()的数据被消费者监听器（ListenerConsumer）处理之后，距离上次提交时间大于TIME时提交
    TIME,
    // 当每一批poll()的数据被消费者监听器（ListenerConsumer）处理之后，被处理record数量大于等于COUNT时提交
    COUNT,
    // TIME |　COUNT　有一个条件满足时提交
    COUNT_TIME,
    // 当每一批poll()的数据被消费者监听器（ListenerConsumer）处理之后, 手动调用Acknowledgment.acknowledge()后提交
    MANUAL,
    // 手动调用Acknowledgment.acknowledge()后立即提交
    MANUAL_IMMEDIATE,
}
```



# kafka的消费

每个kafka消息都有一个 唯一的 键key 和值value 和 一个时间戳。

kafka允许消费者从指定partition读取数据。

消费者可以订阅一个或多个主题，并从这些主题的一个或多个分区中读取数据。

消费者组: 多个消费者可以组成一个消费者组 来 共同消费 一个或多个 topic的数据。 每个分区只能有 同一个消费者组内的 一个消费者消费。

==批量处理==: 消费者通常 batch 读取消息，而不是 逐条。 即 ==poll()== 的时候，消费者 一次可以获得 多条消息。  批次大小可以通过 消费者的 配置的参数 `fetch.min.bytes` `fetch.max.wait.ms` 决定

性能优化， 为了提高消费性能，kafka提供了多种配置
- `max.poll.records`，每次调用 poll 时，消费者从 服务器获得的最大记录数
- `max.poll.interval.ms`， 连续2次poll之间的 最大时间间隔。 如果超过这个时间间隔， 则认为 消费者 挂起，可能会被视为 失效 并重新平衡




# rebalance

对消费者组 内的消费者 重新 分配 partition

## 触发

以下情况，触发rebalance
- 消费者组 内 消费者 个数发生变化
- cg(consumer group) 消费的 topic的 分区个数发生变化
- cg 消费的 topic个数发生变化


## 影响

- 延迟， rebalance过程中，消费者需要重新连接到 新的分区， 这会导致 消息处理 的延迟
- 重复消费和消息丢失， rebalance过程中，可能出现 重复消费 和 消息遗漏
- 集群负载， 消费者 重连导致的


## 实践经验

- 合理规划cg， 根据实际需求，合理规划消费者组 的数量和 消费者数量，避免频繁rebalance
- 使用合适的 分区分配策略， kafka提供了 range，round-robin，sticky assignor 等分区分配策略。
- 优化心跳机制，调整 消费者 和 coordinator 之间的心跳间隔 和 会话超时时间，以确保 消费者在 rebalance时 能及时感知 组内的变化
- 监控和预警，监控 kafka集群的各种指标，如 cg状态，分区分配情况等，及时发现 潜在的 rebalance问题，并 采取相应的 措施。







