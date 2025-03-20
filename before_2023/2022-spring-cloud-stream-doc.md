

从 spring-cloud-sleuth 复制的。。 htmlsingle 里没有sleuth。。


https://docs.spring.io/spring-cloud/docs/2020.0.2/reference/htmlsingle/#spring-cloud-sleuth-reference-documentation


To extend this to Data Integration workloads, Spring Integration and Spring Boot were put together into a new project. Spring Cloud Stream was born.

Spring Cloud Stream 能：
1. 独立地 构建，测试，发布 以数据为中心的 应用。
2. 应用现代的 微服务 架构模式，包括 通过消息传递 来进行组合
3. Decouple(解耦) 应用责任 with 以事件为中心的思考。 一个event能代表 及时发生的事情，下游消费者 应用 可以对其作出反应，而不需要知道 源于何处，生产者的身份。
4. Port(移植) 业务逻辑 到 消息代理 (如 RabbitMQ, Apache Kafka, Amazon Kinesis)
5. 依靠 框架的自动 content-type 支持 for 一般用例。可以扩展到不同的数据转换类型。
6. 等等

---
#### Quick Start

访问 Spring Initializr
1. 依赖部分，输入 stream，选择 Cloud Stream
2. 输入 kafka 或 rabbit
3. 选择 Kafka 或 RabbitMQ
4. Artifact 部分，输入 logging-consumer, 这个值会成为应用名字。

点击创建工程，下载后解压

导入到自己的IDE中。导入后，不会有error，并且src/main/java 包含 com.example.loggingconsumer.LoggingConsumerApplication.

修改 LoggingConsumerApplication 类 成下面：
```java
@SpringBootApplication
public class LoggingConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(LoggingConsumerApplication.class, args);
    }

    @Bean
    public Consumer<Person> log() {
        return person -> {
            System.out.println("Received: " + person);
        };
    }

    public static class Person {
        private String name;
        public String getName() {
            return name;
        }
        public void setName(String name) {
            this.name = name;
        }
        public String toString() {
            return this.name;
        }
    }
}
```
使用了 函数式编程模型(Spring Cloud Function) 来定义 一个 message handler 作为 消费者。   
依靠框架公约来绑定这个handler 到 输入的目的地。

这样做以后，你可以看到 框架的 核心功能之一： 尝试自动转换 输入的消息 为 Person对象。

你现在有了一个 完全函数式 的 Spring Cloud Stream 应用，这个应用监听消息。   
。。。我记得 还要 @Select @Input 之类的吧，现在自动了？

我们假设你使用了RabbitMQ    
启动APP，查看日志：
```
--- [ main] c.s.b.r.p.RabbitExchangeQueueProvisioner : declaring queue for inbound: input.anonymous.CbMIwdkJSBO1ZoPDOtHtCg, bound to: input
--- [ main] o.s.a.r.c.CachingConnectionFactory       : Attempting to connect to: [localhost:5672]
--- [ main] o.s.a.r.c.CachingConnectionFactory       : Created new connection: rabbitConnectionFactory#2a3a299:0/SimpleConnection@66c83fc8. . .
. . .
--- [ main] o.s.i.a.i.AmqpInboundChannelAdapter      : started inbound.input.anonymous.CbMIwdkJSBO1ZoPDOtHtCg
. . .
--- [ main] c.e.l.LoggingConsumerApplication         : Started LoggingConsumerApplication in 2.531 seconds (JVM running for 2.897)
```

去RabbitMQ 管理控制台 或 任何 RabbitMQ客户端， 来发送消息到 *input.anonymous.CbMIwdkJSBO1ZoPDOtHtCg*   
anonymous.CbMIwdkJSBO1ZoPDOtHtCg 代表了 group 的名字。   
spring.cloud.stream.bindings.input.group=hello 来指定名字。

消息的内容 应该是 JSON 形式的 Person对象：
```
{"name":"Sam Spade"}
```

然后，在你的 控制台，你应该看到：
```
Received: Sam Spade
```

---
#### What's New in 3.0?

#### New Features and Enhancements
Routing Function   
Multiple bindings with functions   
Functions with multiple inputs/outputs   
Native support for reactive programming

---
#### Notable(值得注意的，令人尊敬的) Deprecations
Reactive module (spring-cloud-stream-reactive) 停产了，不再分发，支持通过spring-cloud-function 的 源生支持。 为了兼容，你仍然可以使用 spring-cloud-stream-reactive.

Test support binder spring-cloud-stream-test-support with MessageCollector 支持一个新test binder。

@StreamMessageConverter deprecated,不再需要

original-content-type 头 已经被移除。

BinderAwareChannelResolver deprecated，如果提供了 spring.cloud.stream.sendto.destination 属性。主要好似为了 基于函数式 的编程模型。对于StreamListener，它依然是需要的，直到 我们 终止 StreamListener 和 基于注解的 编程模型。

---
#### Introducing Spring Cloud Stream

S C Stream 是一个框架，for 构建 消息驱动的 微服务应用。   
S C Stream 基于 Spring Boot, 来创建 standalone(独立运行), 生产级别的 Spring 应用， and 使用 Spring Integration 来连接消息代理。   
它提供了 来自多个供应商的 中间件的 opinionated(自以为是的) 配置，包括 持久化发布-订阅，消费者组，分区 的概念。

通过 spring-cloud-stream 依赖，你可以立刻连接到 由spring-cloud-stream binder提供的 消息代理， 你可以实现自己的 函数式需求，这个会通过 java.util.function.Function执行

```java
@SpringBootApplication
public class SampleApplication {

    public static void main(String[] args) {
        SpringApplication.run(SampleApplication.class, args);
    }

    @Bean
    public Function<String, String> uppercase() {
        return value -> value.toUpperCase();
    }
}
```

```java
@SpringBootTest(classes =  SampleApplication.class)
@Import({TestChannelBinderConfiguration.class})
class BootTestStreamApplicationTests {

    @Autowired
    private InputDestination input;

    @Autowired
    private OutputDestination output;

    @Test
    void contextLoads() {
        input.send(new GenericMessage<byte[]>("hello".getBytes()));
        assertThat(output.receive().getPayload()).isEqualTo("HELLO".getBytes());
    }
}
```

---
#### Main Concepts

SCStream 提供了许多 抽象 和 原语，来 简化 消息驱动的 微服务 应用。
1. Spring Cloud Stream 的 应用模型
2. Binder 抽象
3. 持久化 发布-订阅
4. 消费者组
5. 分区
6. 可插拔的Binder SPI

---
#### Application Model

SC Stream 应用包括一个 中间件核心。   
应用 和外部交流是通过 建立绑定到目标，目标是 外部broker和 你代码中的 input/output 参数 暴露的。   
broker 定义了 建立 绑定的 详细要求。

---
#### Fat JAR
可以在IDE中 以 stand-alone 运行。   
生产，需要创建一个 executable(or "fat") JAR， 通过 Maven或Gradle 提供的 标准 Spring Boot 工具

---
#### The Binder Abstraction

SC Stream 提供了 Binder 实现 为 Kafka 和 RabbitMQ。 框架也包含一个 test binder for 集成 你的测试。

Binder 抽象 也是 框架的 扩展点， 你可以实现你自己的 binder。

Stream 使用 Spring Boot for 配置，Binder 抽象 使得 SCStream 应用 在 如何连接 中间件 方面是 易变的。   
比如，部署人员可以 动态选择，在运行时，外部目的地(如 Kafka topics 或 RabbitMQ exchanges)的 映射。和 message handler 的 input，output (如，函数式的 input 参数 和 它返回的值)。   
这些配置可以通过 Spring Boot支持的外部属性 提供 (应用参数，环境变量，application.yml/properties)。

SCStream 自动发现 和使用 在classpath中的 binder，你可以使用 不同的 中间件 with 相同的代码。只需要在 build的时候 包含不同的binder。   
对于更复杂的用例，你可以打包 多个binder 到 你的app， 并且在 运行时 决定 用哪个binder。

---
#### Persistent Publish-Subscribe Support

应用间的 通信 遵循 发布-订阅 模型， 数据被广播 through 共享的topics。

数据被 sensor(传感器) 报告到 一个HTTP 端点，被送到一个 普通的 名为 raw-sensor-data的 目的地。   
从这个目的地， 它被独立处理 by 微服务应用，一个会计算 时间窗口的 平均值 ，另一个会 保存 raw data 到 HDFS。   
为了处理数据，所有的 应用 声明了 topic，并作为 他们的 input。

发布-订阅 通信模型 降低了 生产者和消费者的 复杂度， 使得新app加入到拓扑，而不需要 中断已有的flow。   
比如，计算 平均值 应用的下游， 你可以增加一个 app 来计算 最高温度。

通过 共享的topic 通信，而不是 point-to-point 队列， 降低了 微服务的耦合。

当 发布-订阅 消息 不是最新的， SC Stream 通过 额外的步骤 使得它 成为 它的应用模型的 opinionated 选择。

通过使用 源生的 中间件支持， SCStream 也 简化了 跨平台的 发布-订阅 模型的 使用。

---
#### Consumer Groups

当 发布-订阅 模型 使得 通过共享topic 连接 不同app 更加容易， 通过创建给定app的多个实例来使得 伸缩性更高。   
一个app的多个实例 是 竞争关系，一条消息 只能被 一个实例 消费。

SCStream 为这种行为建模，通过 消费者组 的概念。(SCStream 消费者组 类似 Kafka 消费者组)。   
每个消费者绑定 能使用 spring.cloud.stream.bindings.<bindingName>.group 属性来制定一个 组名。

所有 订阅指定目的地的 的组 收到一个 发布的消息的副本，但是 每个组里 只有一个成员 收到 消息。   
默认下， 当组没有被指定，SCStream 为 应用 分配 一个 匿名的 独立的 只有一个成员的 消费者组。

---
#### Consumer Types

支持2种 消费者：
1. 消息驱动 (有时称为异步)
2. Polled (有时称为同步)

2.0 以前，只有异步消费者 被支持。 消息一可用就会被传递，并且有一个线程可以处理它。

当你希望 控制 消息被处理的 速度，你可以希望使用一个 同步的消费者。

---
#### Durability

与 SCStream 的 应用模型一样，消费者组 订阅 是 持久的。   
即，一个binder实现 确保 组订阅 是 持久的， 并且一旦为一个组创建了至少一个订阅，该组就会收到消息，即使这些消息是在该组中的所有应用程序 都停止时发送的。

自然而然，匿名订阅 是 非持久的。对于一些 binder实现(如RabbitMQ)，可以有 非持久的 组订阅。

一般来说，更倾向于 指定一个消费者组，当绑定应用 到 给定的目的地。   
当 新增 SCStream 应用的 实例时，你必须指定一个 消费者组 for 它的每个 input 绑定。 这样做 防止 app实例收到 重复消息。


---
#### Partitioning Support

SCStream 支持 在 给定app的多个实例 之间对数据进行分区。   
在一个分区的设想中，物理通信介质(如 broker topic) 被视为 构造为 多个 分区。 一个或多个 生产者应用实例 发送 数据到 多个 消费者应用实例，并且 确保 特征值相同 的数据 会由 同一个消费者实例处理。

SCStream 提供了一个 常见的抽象 for 以统一的方式 实现 分区处理 用例。   
无论 broker 它自己 是 天生 支持分区的(Kafka)，还是 天生 不支持的(Rabbit), 都能使用 分区。

分区是一个关键的概念 在 有状态的处理中， 它是关键的(因为 性能或一致性 原因) 用来确保，所有 有关联的数据 被同时处理。   
例如，在 时间窗口 平均值计算 这个例子中，下面是重要的：对给定传感器的 所有测量数据 必须在 同一个 app实例中 处理。

要设置分区处理方案，你必须配置 数据生产端 和 数据消费端

---
#### Programming Model

为了理解 编程模型，你应该熟悉一下 核心 概念：
1. Destination Binders, 组建用于负责提供 与外部消息系统的 集成
2. Bindings，外部消息系统 和 应用之间的 桥梁，提供 消息(由Destination Binders生成) 的生产者和消费者。
3. Message， 典型数据结构，被 生产者，消费者 使用，用于 通过 Destination Binders 通信

---
#### Destination Binders

是 SCStream的 扩展组件 用于 提供必要的配置 和 实现，用于 和外部消息系统 集成。   
这个集成 负责 连接、delegate(委托)、route 到生产者和 消费者， 数据类型的转换，用户代码的调用 等。

Binder 有很多 模板，这些模板 本会由你来处理。然而，为了实现这一点，binder 仍需要一些来自用户的 简约的指令，这些指令通常以某种类型的 绑定配置的形式出现。

---
#### Bindings

在上面提到，Bindings 提供了 外部消息系统(如 queue，topic) 和 应用之间的桥梁。

下面例子，展现了一个 完整配置的 函数式的 SCStream 应用，它接收 消息的payload 以String类型，log它到控制台，转换成大写，然后 发送到下游。

```java
@SpringBootApplication
public class SampleApplication {

    public static void main(String[] args) {
        SpringApplication.run(SampleApplication.class, args);
    }

    @Bean
    public Function<String, String> uppercase() {
        return value -> {
            System.out.println("Received: " + value);
            return value.toUpperCase();
        };
    }
}
```

上面的例子 看起来和 普通spring-boot应用 没有什么两样。   
它定义了一个 Function类型的 bean，那么它是如何变成 spring-cloud-stream 应用的？

它变成spring-cloud-stream 应用 简单地 基于 spring-cloud-stream 的存在 和 binder依赖 和 自动配置类，有效地 设置 你的 boot 应用的 context 为 spring-cloud-stream 应用。   
在这个 context 中，Supplier，Function，Consumer 类型的 bean 被当作 事实上的 消息 handler 触发 到目的地的绑定，目的地通过某些 命名约定 和规则 来确定， 避免额外的配置。

---
#### Binding and Binding names

Binding 是一个 抽象，代表了 binder 和 user code 暴露的 source 和 target 之间的 桥梁，   
这个抽象有一个名字，and 当我们 尝试 限制运行 spring-cloud-stream 应用 所需的配置，知道这样的名字 是必要的，当你需要增加 对每个 binding 增加配置时。

你会看到一些配置的例子，如spring.cloud.stream.bindings.input.destination=myQueue, 这个属性名称中的 input 段就是我们说所的 binding name (。。结合下面，这个input 是一个占位符)， 它可以通过多种机制获得。

下面将描述 scs 用于控制 binding name 的 命名约定 和 配置元素。

#### Functional binding names
和 之前SCS版本中使用的 基于注解的 要求显示命名 不同， 函数式编程模型 默认 一个简单的 约定 来 生成 binding name，从而简化配置。

```java
@SpringBootApplication
public class SampleApplication {

    @Bean
    public Function<String, String> uppercase() {
        return value -> value.toUpperCase();
    }
}
```
上面的例子中，我们有一个应用，这个应用有一个 函数式，这个函数式 担当 message handler。   
作为Function类型，它有 input 和 output， 用于 命名 input 和 output binding 的命名约定：
```
input:<functionName> + -in- + <index>
output:<functionName> + -out- + <index>
```

in,out 对应了 binding 的类型。index 是 input 或output 的 下标， 对于 传统的 单 input/output 函数 ，index始终是0.

如果你想 映射这个 input 到一个 称为my-topic的 远程目的地 (如 topic，queue)，你可以配置：
```
--spring.cloud.stream.bindings.uppercase-in-0.destination=my-topic
```
uppercase-in-0 被作为 property name 的一个段。

#### Descriptive Binding Names
有时为了提高 可读性，你可能需要 给予 你的 binding 一个 更描述性的名字(比如 account，orders等)。   
一种方法，你可以 map 一个 implicit(含蓄的) binding name 为 一个 explicit(确切的) binding name。   
你可以通过 spring.cloud.stream.function.bindings.<binding-name> 属性。   
这个属性还为依赖于需要显式名称的 基于自定义接口的绑定的现有应用提供 迁移路径。
```
--spring.cloud.stream.function.bindings.uppercase-in-0=input
```
在上面的例子中，你映射和 有效地重命名 ‘uppercase-in-0’ binding name 到 ‘input’。现在所有的配置 属性 可以ref 到 input 这个binding name。(如，--spring.cloud.stream.bindings.input.destination=my-topic)

描述性的binding name 增强了 配置的可读性，也引入了另一种的 误导。并且由于所有后续配置属性 都将使用 显式绑定名称，因此你必须始终引用这个绑定属性 来关联它实际对应的功能。   
我们认为大多数情况下(函数式除外)，它可能是一种 矫枉过正，因此，我们建议完全避免使用它。

#### Pollable Destination Binding
尽管前面的 描述的 bindings 支持 基于event的 消息消费，有时，你需要 更多控制，比如，消费的速率。

从2.0开始，你可以绑定一个 pollable consumer:
```java
public interface PolledBarista {

    @Input
    PollableMessageSource orders();
    . . .
}
```
这个例子中，PollableMessageSource 的一个 实现 被绑定到 orders 管道。

---
#### Producing and Consuming Messages
你可以写一个 SCStream 应用 通过 简单的 写函数式 并 暴露它们 为 @Bean。   
你也可以 使用 Spring Integration 的基于注解的配置 或 SCStream的 基于注解的配置。从spring-cloud-stream 3.x 开始，建议使用 函数式实现。

---
#### Spring Cloud Function support

从 SCStream 2.1开始，另一种定义 stream handler 和 source 的方法是 使用 内置的对 SCFunction 的 支持，可以 表达为 Supplier，Function，Consumer 类型的 bean。

要指明哪个 函数式bean 绑定到 bindings 公开的 外部目的地，你必须提供 spring.cloud.function.definition 属性。

如果你只有一个 Supplier/Function/Consumer 类型的 bean， 你可以跳过 spring.cloud.function.definition 属性，因为这样的 函数式bean 会被自动发现。
当然，使用这个属性 来避免混淆 是最好的做法。

例子：应用暴露 消息处理者 为 Function，有效地支持 pass-thru 语义，by 行为就像消息的 消费者 和 生产者。
```java
@SpringBootApplication
public class MyFunctionBootApp {

    public static void main(String[] args) {
        SpringApplication.run(MyFunctionBootApp.class);
    }

    @Bean
    public Function<String, String> toUpperCase() {
        return s -> s.toUpperCase();
    }
}
```
上面的例子中，定义了一个 Function 类型的bean，名字是 toUpperCase 来作为 消息处理者，它的 input 和output 必须被绑定到 目的地binder 暴露的 外部目的地。   
默认下， input 和output 的binding name 是 toUpperCase-in-0, toUpperCase-out-0

下面是 一个简单的 函数式应用 支持 其他语义：   
这是一个 source 语义的例子，暴露为 Supplier：
```java
@SpringBootApplication
public static class SourceFromSupplier {

    @Bean
    public Supplier<Date> date() {
        return () -> new Date(12345L);
    }
}
```

这是一个 sink 语义的 例子，暴露为 Consumer
```java
@SpringBootApplication
public static class SinkFromConsumer {

    @Bean
    public Consumer<String> sink() {
        return System.out::println;
    }
}
```

#### Suppliers(Sources)

https://docs.spring.io/spring-cloud/docs/2020.0.2/reference/htmlsingle/#suppliers-sources

Function 和 Consumer 都非常直观 在 如何触发它们。 它们被基于 数据/事件 被送到 它们绑定的目的地 而触发。这是经典的 事件驱动 组件。

然而，Supplier 在触发方面 有它自己的类别。因为它是 数据源，它不需要订阅 任何 入站的目的地(。。其他的是消息进入就触发，但这个是数据源)，所以它需要一些其他机制来触发。   
Supplier的实现 也有一个问题，它可能是命令式(imperative)的，也可能是反应式(reactive)的，这与这些Supplier的触发 直接相关。

```java
@SpringBootApplication
public static class SupplierConfiguration {

    @Bean
    public Supplier<String> stringSupplier() {
        return () -> "Hello from Supplier";
    }
}
```
上面的Supplier bean 生产string，当它的 get() 方法被调用时。 然而，谁调用这个方法？几时？

框架提供了一个默认的 polling 机制( 回答了 谁调用 这个问题)， 会触发 Supplier的 调用，默认下，会 每秒一次 (回答了 几时 )。

所以上面的 配置， 每条生产一条消息，每条消息 被送到 output 目的地。

查看 Polling Configuration Properties 来学习 如何自定义。


```java
@SpringBootApplication
public static class SupplierConfiguration {

    @Bean
    public Supplier<Flux<String>> stringSupplier() {
        return () -> Flux.fromStream(Stream.generate(new Supplier<String>() {
            @Override
            public String get() {
                try {
                    Thread.sleep(1000);
                    return "Hello from Supplier";
                } catch (Exception e) {
                    // ignore
                }
            }
        })).subscribeOn(Schedulers.elastic()).share();
    }
}
```
上面的 Supplier bean 采用 响应式 编程模式。

通常，和命令式(imperative) Supplier 不同，它应该 只被触发一次，因为 对get() 方法的调用 会产生 连续的 消息流 而不是 单个消息。

框架能识别出 这2种编程模式的不同，并且 触发这个 Supplier 只有一次。

然而，想象一个用例：你想要 poll 一些数据源，并且 返回 一个 有限流 作为 结果集。 响应式 是非常完美的机制 在这种场景。但是 因为 是有限流，所以Supplier 会被 周期性调用。
```java
@SpringBootApplication
public static class SupplierConfiguration {

    @PollableBean
    public Supplier<Flux<String>> stringSupplier() {
        return () -> Flux.just("hello", "bye");
    }
}
```
上面就是一个 有限流 的Supplier。   
bean 被 @PollableBean 注解(是 @Bean的 子集)，告诉 框架，尽管这个Supplier 是 reactive，但依然需要被 poll。

@PollableBean 有一个 splittable 属性，这个属性 会告知 这个注解的 处理者，生产的结果 需要被 拆分。 默认 true。   
这意味这 框架 会拆分 返回 的 需要发送出去的item 为 独立的消息。   
你可以设置为 false， 这样 会直接 返回 生产的 Flux 而不拆分。

---
#### Supplier & threading
就像你在上面学到的， 和 Function，Consumer 这种 被event 触发的 不同，Supplier 没有 input，而是 被 不同的机制 触发，这可能导致 不可预知的 线程机制。   
虽然大多数时候 线程机制 的细节 和 函数的下游执行 无关，但某些时候可能会出现问题，特别是 对于 线程敏感的 框架。     
例如，SC Sleuth 会依赖于 ThreadLocal 中存储的 tracing数据。   
对于这些情况，我们有另一种 机制 通过 StreamBridge， 用户可以对 线程机制 有更多的控制。

---
#### Consumer (Reactive)

响应式 Consumer 有点特殊，因为 它返回类型是 void，让框架没有要订阅的引用。   
很可能 你不需要写 Consumer<Flux<?>>，而是写Function<Flux<?>, Mono<Void>> 调用 then 操作 作为 你的stream的 最后一个 操作。
```java
public Function<Flux<?>, Mono<Void>>consumer() {
    return flux -> flux.map(..).filter(..).then();
}
```

如果你 必须写 Consumer<Flux<?>>， 记得 订阅到 输入的 Flux (remember to subscribe to the incoming Flux)

---
#### Polling Configuration Properties

下面的配置 被 org.springframework.cloud.stream.config.DefaultPollerProperties 暴露，并且前缀 是 spring,cloud.stream.poller.

|属性|描述|
|:---|:---|
|fixedDelay|默认poller的 固定 延迟，单位毫秒，默认1000L|
|maxMessagesPerPoll|默认poller的 每次polling 的 最大消息数，默认 1L|
|cron|Cron 表达式， 默认空|
|initialDelay|定时器的 延迟初始化， 默认 0|
|timeUnit|delay 值的 TimeUnit, 默认 MILLISECONDS|

例如，--spring.cloud.stream.poller.fixed-delay=2000 ，设置 poller 每隔2秒 poll 一次。


---
#### Sending arbitrary(任意的，专制的) data to an output (e.g. Foreign event-driven sources)

有些情况下，真实的 数据源 可能是 外部系统(不是binder)。   
例如，数据源可能是一个 REST 端点。 我们如何 使用 spring-cloud-stream 连接 这种 数据源 和 函数式机制？

SCStream 提供了 2种机制。

2个例子中，我们使用 标准MVC 端点方法：delegateToSupplier 绑定到 root web context，委托 输入的请求 到stream，通过 2种 机制： imperative(通过StreamBridge) 和 reactive(通过EmitterProcessor)

#### Using StreamBridge

```java
@SpringBootApplication
@Controller
public class WebSourceApplication {

    public static void main(String[] args) {
        SpringApplication.run(WebSourceApplication.class, "--spring.cloud.stream.source=toStream");
    }

    @Autowired
    private StreamBridge streamBridge;

    @RequestMapping
    @ResponseStatus(HttpStatus.ACCEPTED)
    public void delegateToSupplier(@RequestBody String body) {
        System.out.println("Sending " + body);
        streamBridge.send("toStream-out-0", body);
    }
}
```
这里，注入一个StreamBridge bean, 这个允许 我们 发送数据到 output binding，有效地 将 non-stream 应用 和 spring-cloud-stream 连接起来。   
上面的例子中，没有 任何 source function (如Supplier bean)，使得框架 没有触发器 来创建 源绑定，这个对于 包含 function bean的 配置来说 是非常典型的。   
所以 为了触发 source binding 的 创建，我们使用 spring.cloud.stream.source 属性，你可以指明你的source 的名字。   
被提供的名字会被 用作 触发器，来创建一个 source 绑定。 所以 上面的例子中output binding 的名字是 toStream-out-0。   
你可以使用 ; 来指明多个 源 (如：--spring.cloud.stream.source=foo;bar)

streamBridge.send(...) 方法接受一个 Object，这意味着你可以发送 POJO 或 Message，当发送给 output 时， 经过的流程 和 Object来自Function 或Supplier 一样。   
这意味着 output 类型转换，分区 等，使得 它就像 被 函数式 生成的一样。

#### StreamBridge and Dynamic Destinations
StreamBridge 也能被用于： 无法提前知道 输出 目的地 的情况

```java
@SpringBootApplication
@Controller
public class WebSourceApplication {

    public static void main(String[] args) {
        SpringApplication.run(WebSourceApplication.class, args);
    }

    @Autowired
    private StreamBridge streamBridge;

    @RequestMapping
    @ResponseStatus(HttpStatus.ACCEPTED)
    public void delegateToSupplier(@RequestBody String body) {
        System.out.println("Sending " + body);
        streamBridge.send("myDestination", body);
    }
}
```
上面的例子 和 之前的例子很像，除了 通过 spring.cloud.stream.source 属性的 显示绑定。   
这里，我们发送数据到 myDestination，这个并不存在 一个 binding。因此 这样的名字 会视为 动态目的地。在 Routing FROM Consumer 中介绍。


#### Using reactor API
另一种 发送 任意数据 到 output的 方式 是使用 Reactor API

我们需要做的只是：声明一个 Supplier<Flux<?>>，返回 EmitterProcessor 从 响应式API 来 高效地提供 一个 真实事件源 和 spring-cloud-stream 之间的 桥梁。   
你需要做的只是：通过 EmitterProcessor#onNext(data) 操作 来为 EmitterProcessor 添加 数据。

```java
public class SampleApplication {

    public static void main(String[] args) {
               SpringApplication.run(SampleApplication.class);
    }

    EmitterProcessor<String> processor = EmitterProcessor.create();

    @Bean
    public ApplicationRunner runner() {
        Message<String> msg1 = MessageBuilder.withPayload("foo")
                .setHeader("*.events", "test1.events.billing")
                .build();
        Message<String> msg2 = MessageBuilder.withPayload("bar")
                .setHeader("*.events", "test2.events.messages")
                .build();
        return args -> {
            this.processor.onNext(msg1);
            this.processor.onNext(msg2);
        };
    }

    @Bean
    public Supplier<Flux<String>> supplier() {
               return () -> this.processor;
    }
}
```

上面的例子中，我们使用 ApplicationRunner 作为一个 外部 源 来 给 stream 数据。

一个更真实的例子：外部源 是 一个 REST 端点：
```java
@SpringBootApplication
@Controller
public class WebSourceApplication {

    public static void main(String[] args) {
        SpringApplication.run(WebSourceApplication.class);
    }

    EmitterProcessor<String> processor = EmitterProcessor.create();

    @RequestMapping
    @ResponseStatus(HttpStatus.ACCEPTED)
    public void delegateToSupplier(@RequestBody String body) {
        processor.onNext(body);
    }

    @Bean
    public Supplier<Flux<String>> supplier() {
        return () -> this.processor;
    }
}
```

和我们之前 声明一个 返回 Flux<String> 的 Supplier bean 一样。   
不过这里是 REST 端点,我们可以发送消息 通过 简单地 POST 到 REST端点。
> curl -H "Content-Type: text/plain" -X POST -d "hello from the other side" http://localhost:8080/

通过这2个例子，我们看到了，能和 任何类型的 外部源 一起工作。

#### Reactive Functions support
由于 Spring Cloud Function 建立在 Project Reactor, 因此你不需要做太多事 就 可以在 实现 Supplier,Function,Consumer 时 享受到 响应式编程模型的 好处。

```java
@SpringBootApplication
public static class SinkFromConsumer {

    @Bean
    public Function<Flux<String>, Flux<String>> reactiveUpperCase() {
        return flux -> flux.map(val -> val.toUpperCase());
    }
}
```

#### Functional Composition
使用 函数式变成模型 你也可以 从 functional composition 获得好处，通过 functional composition 你可以 从简单function集合中 动态 组合 复杂的handler。

作为一个例子：我们添加下面的 function bean 到 应用：
```java
@Bean
public Function<String, String> wrapInQuotes() {
    return s -> "\"" + s + "\"";
}
```
修改 spring.cloud.function.definition 属性 来 体现你的 从 toUpperCase 和 wrapInQuotes 组合成 一个 新的 function 的意图。   
SC Function 靠 | 来 这么做。：
> --spring.cloud.function.definition=toUpperCase|wrapInQuotes

SCFunction 提供的 函数式组合 的一个 好处是 你可以组合 响应式 和 命令式 函数。

组合的结果 是一个 单独的 函数，可以有一个相当长的名字(如 foo|bar|baz|xyz...) 带来了不便，当它涉及其他配置属性 时。   
在 Functional binding names 中的 descriptive binding names 功能 能帮助 解决这个问题。

例如，如果我们想给 toUpperCase|wrapInQuotes 一个更具描述性的名字，我们可以    
spring.cloud.stream.function.bindings.toUpperCase|wrapInQuotes=quotedUpperCase 来允许 其他配置属性 refer 到这个 binding name:    
spring.cloud.stream.bindings.quotedUpperCase.destination=myDestination

---
#### Functions with multiple input and output arguments

从 SCStream 3.0 开始，支持 多个 input，多个 output。

这意味着什么？它致力于解决什么样的问题？
1. Big Data: 你在处理的数据 的源 是无组织的，包含各种数据类型，你想要 有效地 排序。
2. Data aggregation(聚合): 从 多个数据源 合并数据。
   等等

这里对 流的概念 的着重点 略有不同。   
假设 function 只有当 它能接触到 数据流，而不是独立的数据 时才是有价值的。   
所以 我们以来 Project Reactor 提供的抽象(如Flux，Mono)。这些已经被SCFunctions 包含。

另一个重要的方面是 多input，多output 的 表达。   
Java提供了 不同种类的 抽象 来 表达 multiple of something: unbounded,lack arity,lack type information 是很重要的。   
比如，Collection 或数组 只允许我们 描述 一个类型的 多个 或者 把所有元素 都转为Object， 影响了SCStream的 类型传递。

为了包含这些需求，最初的支持 是 依赖于 Project Reactor - Tuples 提供的 另一种抽象的 签名。

```java
@SpringBootApplication
public class SampleApplication {

    @Bean
    public Function<Tuple2<Flux<String>, Flux<Integer>>, Flux<String>> gather() {
        return tuple -> {
            Flux<String> stringStream = tuple.getT1();
            Flux<String> intStream = tuple.getT2().map(i -> String.valueOf(i));
            return Flux.merge(stringStream, intStream);
        };
    }
}
```

上面的例子 中的 function 接受 2个input (String类型 和 Integer类型) 生成一个 String类型的output   
上面例子的 2个 input bindings 是 gather-in-0, gather-in-1, output binding 是 gather-out-0

下面覆盖了 gather-in-0 绑定的 content-type:
```
--spring.cloud.stream.bindings.gather-in-0.content-type=text/plain
```

```java
@SpringBootApplication
public class SampleApplication {

    @Bean
    public static Function<Flux<Integer>, Tuple2<Flux<String>, Flux<String>>> scatter() {
        return flux -> {
            Flux<Integer> connectedFlux = flux.publish().autoConnect(2);
            UnicastProcessor even = UnicastProcessor.create();
            UnicastProcessor odd = UnicastProcessor.create();
            Flux<Integer> evenFlux = connectedFlux.filter(number -> number % 2 == 0).doOnNext(number -> even.onNext("EVEN: " + number));
            Flux<Integer> oddFlux = connectedFlux.filter(number -> number % 2 != 0).doOnNext(number -> odd.onNext("ODD: " + number));

            return Tuples.of(Flux.from(even).doOnSubscribe(x -> evenFlux.subscribe()), Flux.from(odd).doOnSubscribe(x -> oddFlux.subscribe()));
        };
    }
}
```
上面是 1个input：scatter-in-0， 2个output:scatter-out-0, scatter-out-1

可以测试它：
```java
@Test
public void testSingleInputMultiOutput() {
    try (ConfigurableApplicationContext context = new SpringApplicationBuilder(
            TestChannelBinderConfiguration.getCompleteConfiguration(
                    SampleApplication.class))
                            .run("--spring.cloud.function.definition=scatter")) {

        InputDestination inputDestination = context.getBean(InputDestination.class);
        OutputDestination outputDestination = context.getBean(OutputDestination.class);

        for (int i = 0; i < 10; i++) {
            inputDestination.send(MessageBuilder.withPayload(String.valueOf(i).getBytes()).build());
        }

        int counter = 0;
        for (int i = 0; i < 5; i++) {
            Message<byte[]> even = outputDestination.receive(0, 0);
            assertThat(even.getPayload()).isEqualTo(("EVEN: " + String.valueOf(counter++)).getBytes());
            Message<byte[]> odd = outputDestination.receive(0, 1);
            assertThat(odd.getPayload()).isEqualTo(("ODD: " + String.valueOf(counter++)).getBytes());
        }
    }
}
```

---
#### Multiple functions in a single application

需要在 一个应用中 分组 几个 消息处理者。 你可以通过定义多个function 来完成：
```java
@SpringBootApplication
public class SampleApplication {

    @Bean
    public Function<String, String> uppercase() {
        return value -> value.toUpperCase();
    }

    @Bean
    public Function<String, String> reverse() {
        return value -> new StringBuilder(value).reverse().toString();
    }
}
```
上面，我们定义了2个 function：uppercase 和 reverse。   
我们需要注意到 这里有一个 冲突(more then one function)， 我们需要提供 spring.cloud.function.definition 指定 我们想绑定的 function。   
我们可以使用 ; 来指定 2个function。

测试：
```java
@Test
public void testMultipleFunctions() {
    try (ConfigurableApplicationContext context = new SpringApplicationBuilder(
            TestChannelBinderConfiguration.getCompleteConfiguration(
                    ReactiveFunctionConfiguration.class))
                            .run("--spring.cloud.function.definition=uppercase;reverse")) {

        InputDestination inputDestination = context.getBean(InputDestination.class);
        OutputDestination outputDestination = context.getBean(OutputDestination.class);

        Message<byte[]> inputMessage = MessageBuilder.withPayload("Hello".getBytes()).build();
        inputDestination.send(inputMessage, "uppercase-in-0");
        inputDestination.send(inputMessage, "reverse-in-0");

        Message<byte[]> outputMessage = outputDestination.receive(0, "uppercase-out-0");
        assertThat(outputMessage.getPayload()).isEqualTo("HELLO".getBytes());

        outputMessage = outputDestination.receive(0, "uppercase-out-1");
        assertThat(outputMessage.getPayload()).isEqualTo("olleH".getBytes());
    }
}
```

---
#### Batch Consumers

当使用 支持batch listener 的 MessageChannelBinder， 并且 consumer binding的 功能被激活 时，你可以设置   
spring.cloud.stream.bindings.<binding-name>.consumer.batch-mode 为 true 来 启用： 通过List 传递 一批消息 到 function
```java
@Bean
public Function<List<Person>, Person> findFirstPerson() {
    return persons -> persons.get(0);
}
```

---
#### Spring Integration flow as functions

当你实现一个 function，你可能有 符合Enterprise Integration Patterns(EIP)的 复杂的需求，最好使用 框架(如 Spring Integration(SI)，这个是EIP的参考实现) 来处理这些问题

由于SI 已经提供了 通过Integration flow as gateway 来 exposing integration flows as function 的支持.
```java
@SpringBootApplication
public class FunctionSampleSpringIntegrationApplication {

    public static void main(String[] args) {
        SpringApplication.run(FunctionSampleSpringIntegrationApplication.class, args);
    }

    @Bean
    public IntegrationFlow uppercaseFlow() {
        return IntegrationFlows.from(MessageFunction.class, "uppercase")
                .<String, String>transform(String::toUpperCase)
                .logAndReply(LoggingHandler.Level.WARN);
    }

    public interface MessageFunction extends Function<Message<String>, Message<String>> {

    }
}
```

我们定义了 IntegrationFlow 类型的 bean，在这个bean中，我们声明了 一个 integration flow，我们暴露它为 名字是uppercase的 Function<String, String> (使用 SI DSL)。   
MessageFunction 接口 使得我们 明确声明了 input，output 的类型 for proper(正确/严格) 类型转换。   
为了接收 原始input 你可以使用 from(Function.class, ...)

最终的function 被绑定到 target binder 暴露的input，output destination

---
#### Using Polled Consumers

当使用 polled consumers时， 你按要求 poll PollableMessageSource.   
为了定义 polled consumer 的绑定，你需要提供 spring.cloud.stream.pollale-source 属性。
```
--spring.cloud.stream.pollable-source=myDestination
```

pollable-source 名叫 myDestination，会导致 myDestination-in-0 这个binding name 来 和 函数式编程模型 保持一致。

在上面的 polled consumer 的基础上， 你可以这样使用：
```java
@Bean
public ApplicationRunner poller(PollableMessageSource destIn, MessageChannel destOut) {
    return args -> {
        while (someCondition()) {
            try {
                if (!destIn.poll(m -> {
                    String newPayload = ((String) m.getPayload()).toUpperCase();
                    destOut.send(new GenericMessage<>(newPayload));
                })) {
                    Thread.sleep(1000);
                }
            }
            catch (Exception e) {
                // handle failure
            }
        }
    };
}
```

一个更少手动，且更像Spring 的替代方案是 配置 定时任务 bean：
```java
@Scheduled(fixedDelay = 5_000)
public void poll() {
    System.out.println("Polling...");
    this.source.poll(m -> {
        System.out.println(m.getPayload());

    }, new ParameterizedTypeReference<Foo>() { });
}
```

PollableMessageSource.poll() 方法 接受一个 MessageHandler 参数(通常是lambda表达式，就像这个例子中)，它返回true 如果 消息被接收 且 成功处理。

作为 消息驱动 的消费者，如果MessageHandler 抛出异常，消息会被发布到 error channel。

通常，poll()方法 ACK 消息，当MessageHandler 退出时。   
如果 方法 异常退出，message 被拒绝 (而不是 重新入队)，但请参照 Handling Errors。   
你可以覆盖这种行为通过 获得 ACK的责任：
```java
@Bean
public ApplicationRunner poller(PollableMessageSource dest1In, MessageChannel dest2Out) {
    return args -> {
        while (someCondition()) {
            if (!dest1In.poll(m -> {
                StaticMessageHeaderAccessor.getAcknowledgmentCallback(m).noAutoAck();
                // e.g. hand off to another thread which can perform the ack
                // or acknowledge(Status.REQUEUE)

            })) {
                Thread.sleep(1000);
            }
        }
    };
}
```

你必须 ACK 或 NACK 消息 在某个地方，来防止消息丢失。

一些 消息系统(如 Apache Kafka) 在日志中 维护一个 偏移量。 如果传递失败，且 通过StaticMessageHeader.Accessor.getAcknowledgmentCallback(m).acknowledge(Status.REQUEUE) 重新入队，所有后续 成功ACK的 消息 被重新投递。

也有一个 覆盖的 poll 方法，定义如下：
```
poll(MessageHandler handler, ParameterizedTypeReference<?> type)
```

type 是一个 转换提示，表示 允许 输入消息负载 被转换的 类型。
```
boolean result = pollableSource.poll(received -> {
            Map<String, Foo> payload = (Map<String, Foo>) received.getPayload();
            ...

        }, new ParameterizedTypeReference<Map<String, Foo>>() {});
```

---
#### Handling Errors

默认下，为 pollable source 配置一个 error channel，如果 callback 抛出异常，一个ErrorMessage 被发送到 error channel (<destination>.<group>.errors).   
这个error channel 也 桥接到了 全局的 Spring Integration 的 errorChannel。

你可以 订阅 任一 有@ServiceActivator来处理错误的error channel。   
没有订阅的话，错误被 log， 消息被认为成功处理，被ACK。

如果 error channel 的 service activator 抛出一个异常，消息被拒绝(默认), 不会 被重发。  
如果 service activator 抛出 RequeueCurrentMessageException, 消息会被 requeue。

如果 listener 抛出 RequeueCurrentMessageException，消息会被 requeue，不会被发送到 error channel

---
#### 106.4 Event Routing
Event Routing, 在SCStream 的上下文中，代表：   
route events 到 某个事件订阅者， 或 route 事件订阅者生产的 event 到 某个目的地。   
在这里，我们将其称为 route 'TO', route 'FROM'

---
#### Routing TO Consumer
Routing 能被完成 通过 基于 SCFunction 3.0可用的 RoutingFunction.   
你需要的只是激活它通过 --spring.cloud.stream.function.routing.enabled=true , 或 提供 spring.cloud.function.routing-expression 属性。   
一旦激活， RoutingFunction 会被 绑定到 input 目的地，接收所有消息，route它们 到 其他function， 基于提供的 instruction(用法说明/指令)。

为了绑定，route 目的地 的 绑定的名字是 functionRouter-in-0 (查看RoutingFunction.FUNCTION_NAME 和 binding name 约定)

Instruction(用法说明/指令) 可以和 单独的消息 以及 properties 一起提供。

使用 message headers
```java
@SpringBootApplication
public class SampleApplication {

    public static void main(String[] args) {
        SpringApplication.run(SampleApplication.class,
                       "--spring.cloud.stream.function.routing.enabled=true");
    }

    @Bean
    public Consumer<String> even() {
        return value -> {
            System.out.println("EVEN: " + value);
        };
    }

    @Bean
    public Consumer<String> odd() {
        return value -> {
            System.out.println("ODD: " + value);
        };
    }
}
```

通过发送消息到 binder暴露的 functionRouter-in-0 目的地，这条消息会被 route 到 合适的(even or odd) Consumer。

默认，RoutingFunction 会寻找 spring.cloud.function.definition 或 spring.cloud.function.routing-expression header, 如果 找到，这个属性的值 就会被当作 routing instruction。

比如，设置 spring.cloud.function.routing-expression 头为 T(java.lang.System).currentTimeMillis() % 2 == 0 ? 'even' : 'odd', 会导致 一半 随机 routing request 到odd，一半到even。

对于SpEL，eval 的 根对象 是 Message，所以你可以 对个别的header(或 message) 进行eval ： ....routing-expression=headers['type']

。。。这个是找 request的 header？长的很想属性啊。。。

---
#### Using application properties

spring.cloud.function.routing-expression 和/或 spring.cloud.function.definition 能被当作 应用属性 传递 (如，spring.cloud.function.routing-expression=headers['type'])

```java
@SpringBootApplication
public class RoutingStreamApplication {

  public static void main(String[] args) {
      SpringApplication.run(RoutingStreamApplication.class,
      "--spring.cloud.function.routing-expression="
      + "T(java.lang.System).nanoTime() % 2 == 0 ? 'even' : 'odd'");
  }
  @Bean
  public Consumer<Integer> even() {
    return value -> System.out.println("EVEN: " + value);
  }

  @Bean
  public Consumer<Integer> odd() {
    return value -> System.out.println("ODD: " + value);
  }
}
```
通过应用属性传递 instruction 对于 响应式函数 尤其 重要，因为响应式函数 仅仅被调用一次，因此对单个item的 访问 收到限制。


#### Routing FROM Consumer

除了 静态 目的地外，SCStream 允许 应用发送消息 到 动态绑定的目的地。   
有用的，比如需要在 运行时 决定 目标目的地   
应用有2种方法来完成。

#### BinderAwareChannelResolver
BinderAwareChannelResolver 是一个特殊bean，被框架自动注册。   
你可以 autowire 这个bean 到你的应用，使用 它来 运行时 解析 output目的地。

spring.cloud.stream.dynamicDestinations 属性 被用来 限制 动态目的地名字 为 一个 已知的集合。

下面的例子是一个普通的场景：REST controller 使用 path变量 来决定 target 目的地：
```java
@SpringBootApplication
@Controller
public class SourceWithDynamicDestination {

    @Autowired
    private BinderAwareChannelResolver resolver;

    @RequestMapping(value="/{target}")
    @ResponseStatus(HttpStatus.ACCEPTED)
    public void send(@RequestBody String body, @PathVariable("target") String target){
        resolver.resolveDestination(target).send(new GenericMessage<String>(body));
    }
}
```

```
curl -H "Content-Type: application/json" -X POST -d "customer-1" http://localhost:8080/customers

curl -H "Content-Type: application/json" -X POST -d "order-1" http://localhost:8080/orders
```

目的地 customers，orders，已经在 broker(RabbitMQ的 exchange，Kafka的 topic) 中创建，数据会被发送到 对应的 目的地。

#### spring.cloud.stream.sendto.destination

你也可以委托框架 来 动态 解析 output 目的地 通过 指定 spring.cloud.stream.sendto.destination header 设置为 需要被解析的 目的地的名字。
```java
@SpringBootApplication
@Controller
public class SourceWithDynamicDestination {

    @Bean
    public Function<String, Message<String>> destinationAsPayload() {
        return value -> {
            return MessageBuilder.withPayload(value)
                .setHeader("spring.cloud.stream.sendto.destination", value).build();};
    }
}
```
我们的输出是一个 Message，有着spring.cloud.stream.sendto.destination header，这个header 设置了 输入参数的 值。   
框架会 查询 这个header，并 尝试 创建或发现 这个名字的 目的地，然后发送到这个目的地。

如果 目的地名字 预先知道，你可以配置 生产者属性 就像 配置其他目的地。   
或者，如果你注册了一个 NewDestinationBindingCallback<> bean，它只有在 binding创建前 被调用。   
callback 接受 binder 使用的 生产者属性 扩展的 通用类型，只有一个方法：
```java
void configure(String destinationName, MessageChannel channel, ProducerProperties producerProperties,
        T extendedProducerProperties);
```

下面的例子，展示了 如何使用 RabbitMQ binder：
```java
@Bean
public NewDestinationBindingCallback<RabbitProducerProperties> dynamicConfigurer() {
    return (name, channel, props, extended) -> {
        props.setRequiredGroups("bindThisQueue");
        extended.setQueueNameGroupOnly(true);
        extended.setAutoBindDlq(true);
        extended.setDeadLetterQueueName("myDLQ");
    };
}
```

如果你需要支持 动态目的地 with 多个binder type， 使用 Object 作为通用类型，且 转换extended 参数 是必须的。

---
#### 106.5 Error Handling

这节 解释 error handling 背后的 一般概念。   
我们使用 Rabbit binder 作为例子，因为 各个binder 为特定于底层broker 的某些 机制 定义了不同的 属性集。

错误发生，SCStream 提供 数个 灵活的机制来处理它们。技术 依赖于 binder的具体实现，和底层消息中间件的 功能，和编程模型。

当Message handler (function) 抛出异常，它被扩散到 binder，binder 扩散 这个错误到 消息系统。 框架会 重试 这个消息(默认3次) 通过 Spring Retry 提供的 RetryTemplate。

在那以后，依赖于 消息系统的能力， 消息系统可能 drop消息+requeue消息 或 发送失败的消息到DLQ。   
Rabbit，Kafka 都支持这些概念。   
其他的binder 或许不支持，所以 查看你的 binder的 文档，for 支持的error handling 的 明细。

记住： 响应式function 并不能作为一个 合格的 Message handler，因为 它不能 处理 单独的(individual)消息， 而是提供了 一个连接stream(如Flux)的方法。   
换种方式来看，它是 Message handler (如 命令式function) 被调用 for 每个消息。   
响应式 只被调用一次，在初始化时调用，来连接 2个stream，此时 框架将控制权 交给 响应式 API。

为什么是重要的？ 因为你稍后读到的东西 关于Retry Template，dropping失败的消息，retrying，DLQ，配置属性，这些 只能应用到 Message handler(如 命令式 function)。

响应式API 提供了 丰富的 库 关于它的 操作和机制 来 帮助你 进行error 处理。   
比如 你可以在 reactor.core.publisher.Flux 中找到 public final Flux<T> retryWhen(Retry retrySpec);

```java
@Bean
public Function<Flux<String>, Flux<String>> uppercase() {
    return flux -> flux
            .retryWhen(Retry.backoff(3, Duration.ofMillis(1000)))
            .map(v -> v.toUpperCase());
}
```

---
#### 106.5.1 Drop Failed Messages
默认下，如果没有额外的 系统级别的 配置，消息系统 drop 失败的message。某些情况下是可接受的，但是大多数情况下，我们需要一些恢复机制 来避免消息丢失。

---
#### 106.5.2 DLQ - Dead Letter Queue

可能是最常见的机制，DLQ允许 失败的消息 被发送到一个 特殊的目的地： the Dead Letter Queue

配置后，失败的信息 被发送到 这个目的地，for 后续的 re-processing 或 auditing(审计) 和 reconciliation(和解)

```java
@SpringBootApplication
public class SimpleStreamApplication {

    public static void main(String[] args) throws Exception {
        SpringApplication.run(SimpleStreamApplication.class,
          "--spring.cloud.function.definition=uppercase",
          "--spring.cloud.stream.bindings.uppercase-in-0.destination=uppercase",
          "--spring.cloud.stream.bindings.uppercase-in-0.group=myGroup",
          "--spring.cloud.stream.rabbit.bindings.uppercase-in-0.consumer.auto-bind-dlq=true"
        );
    }

    @Bean
    public Function<Person, Person> uppercase() {
        return personIn -> {
           throw new RuntimeException("intentional");
          });
        };
    }
}
```

这个例子中， 配置属性的 uppercase-in-0 段 代表了 input destination binding 的名字。 consumer段 表示 它是一个消费者属性。

当使用DLQ，至少 要提供 group 属性，来 设置为DLQ的名字。当然，group经常 和 destination 属性一起使用。

除了标准属性，我们也设置了 auto-bind-dlq 来 指引 binder 去 create 和 配置 DLQ 目的地 for uppercase-in-0 binding， 这个 binding 对应了 uppercase 目的地，   
这会导致一个额外的 Rabbit queue，名字是 uppercase.myGroup.dlq.

配置完成后，所有失败的消息 会 route 到这个 目的地，保存原始消息，for 进一步处理。

你可以看到 error message 包含 关于错误的信息：
```. . . .
x-exception-stacktrace: org.springframework.messaging.MessageHandlingException: nested exception is
      org.springframework.messaging.MessagingException: has an error, failedMessage=GenericMessage [payload=byte[15],
      headers={amqp_receivedDeliveryMode=NON_PERSISTENT, amqp_receivedRoutingKey=input.hello, amqp_deliveryTag=1,
      deliveryAttempt=3, amqp_consumerQueue=input.hello, amqp_redelivered=false, id=a15231e6-3f80-677b-5ad7-d4b1e61e486e,
      amqp_consumerTag=amq.ctag-skBFapilvtZhDsn0k3ZmQg, contentType=application/json, timestamp=1522327846136}]
      at org.spring...integ...han...MethodInvokingMessageProcessor.processMessage(MethodInvokingMessageProcessor.java:107)
      at. . . . .
Payload: blah
```

你也可以设置：不重试，一次失败直接 发送到 DLQ 通过设置 max-attempts 为 1：
```
--spring.cloud.stream.bindings.uppercase-in-0.consumer.max-attempts=1
```

---
#### Retry Template
这节讲述 retry 的配置   
RetryTemplate 是 Spring Retry 的一部分。 这里不会讲述所有的 RetryTemplate的能力。   
我们会提到 下面的 RetryTemplate 的 消费者属性：

|属性|描述|
|:---|:---|
|maxAttempts|尝试处理消息的次数， 默认3|
|backOffInitialInterval|retry时 backoff 初始间隔， 默认1000ms|
|backOffMaxInterval|backoff 最大间隔， 默认10000ms|
|backOffMultiplier|backoff 系数， 默认 2.0|
|defaultRetryable|listener抛出的异常 不在 retryableExceptions 时，是否retry？， 默认 true|
|retryableExceptions|map,异常类名字是key，boolean是value，指明这些异常(和子类) 会/不会 触发 retry。例子：spring.cloud.stream.bindings.input.consumer.retryable-exceptions.java.lang.IllegalStateException=false, 默认 空|

上面的配置 满足 大多数 自定义的需求。   
它们可能无法满足 某些 复杂需求，此时，你需要 提供你自己的 RetryTemplate 的实例。   
在你的应用中，把它配置成一个 bean， 应用提供的 实例，会 覆盖 框架提供的实例。   
为了避免冲突，你必须将 由binder 使用的RetryTemplate实例 限定为 @StreamRetryTemplate。

```java
@StreamRetryTemplate
public RetryTemplate myRetryTemplate() {
    return new RetryTemplate();
}
```
你可以看到，你不需要 @Bean，因为 @StreamRetryTemplate 就是 一个 @Bean

如果你需要更精确地使用 RetryTemplate，你可以制定 bean的 名字 在你的 ConsumerProperties 中：
```
spring.cloud.stream.bindings.<foo>.consumer.retry-template-name=<your-retry-template-bean-name>
```

---
#### 107 Binders
SCStream 提供了一个 Binder抽象，来用在 连接 外部中间件的 物理目的地。   
这节介绍 Binder SPI 的主要概念，主要组成，实现细节。

---
#### 107.1 Producers and Consumers
下图展现了一般情况下 生产者和消费者的 关系。
。。。。图裂了。。。

一个生产者 是任意组件，只要它发送消息到 binding 目的地。    
binding 目的地 可以被 绑定到 一个外部 消息broker 通过 该broker 的 一个Binder 实现。   
当调用 bindProducer()方法时，第一个参数是 broker中目的地的 名字，第二个参数是 生产者向其发送消息的本地目的地的实例，
第三个参数是 包含属性(比如一个 partition key expression) 被 为binding 目的地创建的 adapter 使用。

一个消费者 是任意组件，只要它接受消息 从 绑定的目的地。   
和生产者一样，消费者能被绑定到 外部message broker。   
当调用 bindConsumer() 时，第一个参数 是目的地名字，第二个是 消费者的逻辑组的名字，当生产者发送数据到 某个目的地，这个目的地的 消费者 绑定的每个组 都会 收到 消息的副本。(即，遵循正常的发布订阅语义)    
。。。是不是：一个目的地，N个消费者 分成M组 (N>=M)， 然后 消息到这个 目的地后， 会有 M 条消息 到 M 个组， 组里是随机选一个 消费者来处理。。。是的，下面说了。。     
如果 多个 消费者实例 绑定了 同一个组名， 消息会被 负载均衡 这些 消费者实例。这样，保证 每条消息 只被 每个组的 一个消费者实例 消费。(也就是说，它遵循正常的 queueing语义)

---
#### 107.2 Binder SPI
Binder SPI 包含 一些 接口，开箱即用的工具类，发现策略(提供了一个 用于连接外部中间件的 可插拔的 机制)。

SPI接口的 关键是 Binder 接口，这个是 连接input 和output 到 外部中间件 的策略。   
下面是Binder 接口的 定义：
```java
public interface Binder<T, C extends ConsumerProperties, P extends ProducerProperties> {
    Binding<T> bindConsumer(String bindingName, String group, T inboundBindTarget, C consumerProperties);

    Binding<T> bindProducer(String bindingName, T outboundBindTarget, P producerProperties);
}
```

接口是参数话的，提供了 一些扩展点：
1. input 和output 绑定目标。
2. 扩展的 消费者 和 生产者属性， 允许 特定的Binder 实现 添加 可以以类型安全 方式 支持的 补充属性。

典型的 binder 实现 包含下面：
1. 一个实现 Binder 接口的类
2. 一个 Spring @Configuration 类，它创建一个 Binder 类型的 bean，以及 中间件连接基础结构。
3. 一个classpath中的 META-INF/spring.binders 文件 包含一个或多个 binder定义：
```
kafka:\
org.springframework.cloud.stream.binder.kafka.config.KafkaBinderConfiguration
```

---
#### 107.3 Binder Detection
SCStream 依赖 Binder SPI 的实现 来 执行 连接 user code 到 消息broker 的任务。   
每个Binder 实现，通常 连接 一种 消息系统。

---
#### 107.3.1 Classpath Detection
默认，SCStream 依赖与 SpringBoot 的 自动配置 来配置 绑定过程。   
如果一个 Binder 实现 在 classpath中 被发现，SCStream 自动使用它， 例如，一个SCStream 工程，想要 只绑定到RabbitMQ，能增加下列依赖：
```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
</dependency>
```

---
#### 107.4 Multiple Binders on the Classpath

当classpath中 有多个binder，应用必须 表明 使用哪个binder for 每个 destination binding。   
每个binder 配置 包含一个 META-INF/spring.binders 文件：
```
rabbit:\
org.springframework.cloud.stream.binder.rabbit.config.RabbitServiceAutoConfiguration
```

其他的binder 实现 也有 类似的文件，自定义binder实现 也被期望 提供它们。   
key 代表了 一个 唯一的名字 for binder 实现， value 是一个 逗号分隔的 list 包含 配置类，每个配置类 有且仅有 一个 Binder 类型的 bean。

Binder 选择 能 全局执行，或 individually(单独地)        
全局是通过 使用 spring.cloud.stream.defaultBinder 属性(如 spring.cloud.stream.defaultBinder=rabbit)       
单独执行是，为每个 binding 配置 binder。

比如，一个 处理者应用( 有binding，名字是input 和 output)，从Kafka读，写到RabbitMQ：
```
spring.cloud.stream.bindings.input.binder=kafka
spring.cloud.stream.bindings.output.binder=rabbit
```

---
#### 107.5 Connecting to Multiple Systems
默认，binders 使用 SB 的自动配置，所以 classpath 中 发现的每个binder 会创建一个 实例。   
如果你的应用 应该连接 同一类型的消息中间件的 多个 broker，你可以指明 多个 binder 配置，每个 with 不同的 环境配置。

使用 明确的 binder 配置 会 禁用 默认的 binder 配置。如果你这样做， 你必须把 所有 用到的 binder 都包含在 配置中。   
intend(准备，打算) 使用SCStream 的框架 可能会创建 binder 配置，这个配置能 通过 name 被引用，但他们 不会影响默认的 binder 配置。为了这么做，
binder 配置可以有它自己的 defaultCandidate 标记 设置为false (例如，spring.cloud.stream.binders.<configurationName>.defaultCandidate=false)，这表示 配置 独立于 默认的 binder配置。

下面是 典型的配置 for 一个 处理者应用，包含了 2个RabbitMQ broker 实例：
```yaml
spring:
  cloud:
    stream:
      bindings:
        input:
          destination: thing1
          binder: rabbit1
        output:
          destination: thing2
          binder: rabbit2
      binders:
        rabbit1:
          type: rabbit
          environment:
            spring:
              rabbitmq:
                host: <host1>
        rabbit2:
          type: rabbit
          environment:
            spring:
              rabbitmq:
                host: <host2>
```

environment 属性 也能被用于 任何 SBoot 属性，包括 spring.main.sources，这个可以 有用 for 增加 额外的配置 for 特定的binder，比如覆盖自动配置的bean。

例如：
```yaml
    spring:
        main:
           sources: com.acme.config.MyCustomBinderConfiguration
```

为了激活一个明确的 profile for 特定的binder 环境，你应该使用一个 spring.profiles.active 属性。
```yaml
environment:
    spring:
        profiles:
           active: myBinderProfile
```
。。？ 上上面的 environment 怎么用？ 不是这个激活吧。。 上上面 是 同时激活的啊。。。 但是上面这个 写了干什么。。

---
#### 107.6 Customizing binders in multi binder applications

当一个应用有多个binder，想定制binder，能通过 BinderCustomizer 实现 来 完成。   
如果 应用只有一个 binder， 这个特殊的 customizer 并不需要，因为 binder context 能直接 访问 自定义bean。   
当然，多binder的 场景并不适用，因为 binder 存在于 不同的 application context中。   
通过提供一个 BinderCustomizer 实现，binders，尽管驻留在不同的application context中，会收到 定制信息。   
SCStream 确保，定制信息 起效 before 应用使用binder。   
user必须检查 binder类型，然后应用 必要的 定制化。

BinderCustomizer bean 的例子：
```java
@Bean
public BinderCustomizer binderCustomizer() {
    return (binder, binderName) -> {
        if (binder instanceof KafkaMessageChannelBinder) {
            ((KafkaMessageChannelBinder) binder).setRebalanceListener(...);
        }
        else if (binder instanceof KStreamBinder) {
            ...
        }
        else if (binder instanceof RabbitMessageChannelBinder) {
            ...
        }
    };
}
```

当 同一个binder类型 有多个实例时，binder 名字 能用来 过滤 定制化。

---
#### Binding visualization and control

从2.0 开始，SCStream 支持 binding的 可视化 和控制，通过 Actuator 端点。

从2.0 开始，actuator和 web 是可选的，你必须 手动 增加 一个web依赖 和actuator 依赖。

增加web 框架依赖：
```xml
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

增加WebFlux框架依赖：
```xml
<dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

你可以增加Actuator 依赖：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

为了在 Cloud Foundry 上运行 SCStream 2.0, 你必须增加 spring-boot-starter-web, spring-boot-starter-actuator。否则，应用不会启动，由于 health check 失败。

你必须 也 激活 bindings actuator 端点，通过设置：
```
--management.endpoints.web.exposure.include=bindings
```

一旦这些 先觉条件 被满足，你应该能看到 下面的 log，在应用启动时：
```
: Mapped "{[/actuator/bindings/{name}],methods=[POST]. . .
: Mapped "{[/actuator/bindings],methods=[GET]. . .
: Mapped "{[/actuator/bindings/{name}],methods=[GET]. . .
```

要可视化 当前的 bindings，访问： <host>:<port>/actuator/bindings

或者要 查看 单个binding，访问：<host>:<port>/actuator/bindings/<bindingName>

你可以停止，开始，暂停，恢复 个别binding，通过 POST 到 上面的URL，发送一个JSON，提供一个 state 状态。
```
curl -d '{"state":"STOPPED"}' -H "Content-Type: application/json" -X POST http://<host>:<port>/actuator/bindings/myBindingName
curl -d '{"state":"STARTED"}' -H "Content-Type: application/json" -X POST http://<host>:<port>/actuator/bindings/myBindingName
curl -d '{"state":"PAUSED"}' -H "Content-Type: application/json" -X POST http://<host>:<port>/actuator/bindings/myBindingName
curl -d '{"state":"RESUMED"}' -H "Content-Type: application/json" -X POST http://<host>:<port>/actuator/bindings/myBindingName
```

PAUSED，RESUMED 起效， 只有当 相应的 binder，和它的底层技术 支持它。   
否则 你会看到警告消息。   
目前，只有Kafka binder 支持 PAUSED，RESUMED。

---
#### Binder Configuration Properties

下面的属性 可用于 定制 binder 配置。   
这些属性 通过 org.springframework.cloud.stream.config.BinderProperties 暴露。

属性的前缀都是   
```spring.cloud.stream.binders.<configurationName>```

type
binder类型，一般指向classpath中 找到的 binders 中的一个，特别是 MATE-INF/spring.binders 文件中的 一个key。
默认，和 配置的名字相同
inheritEnvironment
配置是否 基础 应用的环境。
默认，true
environment
属性集 的根，能用来 定制 binder的 环境。当这个属性设置，会被创建的binder所在的 context 不是 application context的 child。这个甚至允许 binder组件 和 应用组件 完全分离
默认，空
defaultCandidate
这个binder 配置 是不是 默认binder的 候选者？还是只能用于 显式引用。这个配置允许增加 binder配置，不干扰默认处理。
默认，true

---
#### 107.9 Implementing Custom Binders

为了实现自定义Binder，你需要：
1. 增加需要的依赖
2. 提供一个 ProvisioningProvider 实现
3. 提供一个 MessageProducer 实现
4. 提供一个 MessageHandler 实现
5. 提供一个 Binder 实现
6. 创建一个 Binder配置
7. 定义你的 binder 到 META-INF/spring.binders

增加 spring-cloud-stream 依赖
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream</artifactId>
    <version>${spring.cloud.stream.version}</version>
</dependency>
```

提供一个 ProvisioningProvider 实现

ProvisioningProvider 负责 provisioning(供应/准备) 消费者 和 生产者 目的地，用于转换 application.yml/properties 中的 逻辑目的地 到 物理目的地。

下面是一个 ProvisioningProvider 实现的例子，简单地 trim input/output bindings 配置 提供的 目的地。
```java
public class FileMessageBinderProvisioner implements ProvisioningProvider<ConsumerProperties, ProducerProperties> {

    @Override
    public ProducerDestination provisionProducerDestination(
            final String name,
            final ProducerProperties properties) {

        return new FileMessageDestination(name);
    }

    @Override
    public ConsumerDestination provisionConsumerDestination(
            final String name,
            final String group,
            final ConsumerProperties properties) {

        return new FileMessageDestination(name);
    }

    private class FileMessageDestination implements ProducerDestination, ConsumerDestination {

        private final String destination;

        private FileMessageDestination(final String destination) {
            this.destination = destination;
        }

        @Override
        public String getName() {
            return destination.trim();
        }

        @Override
        public String getNameForPartition(int partition) {
            throw new UnsupportedOperationException("Partitioning is not implemented for file messaging.");
        }

    }

}
```

提供 MessageProducer 实现

MessageProducer 负责 消费 event，将它们作为消息 发送到 根据配置应该消费这些event的 客户端应用。

一个MessageProducer 实现的例子，扩展了 MessageProducerSupport，为了poll on a file that 匹配 trimmed 目的地名字 并且 位于 project path，也实现了 读取消息，丢弃后续的 相同的消息：
```java
public class FileMessageProducer extends MessageProducerSupport {

    public static final String ARCHIVE = "archive.txt";
    private final ConsumerDestination destination;
    private String previousPayload;

    public FileMessageProducer(ConsumerDestination destination) {
        this.destination = destination;
    }

    @Override
    public void doStart() {
        receive();
    }

    private void receive() {
        ScheduledExecutorService executorService = Executors.newScheduledThreadPool(1);

        executorService.scheduleWithFixedDelay(() -> {
            String payload = getPayload();

            if(payload != null) {
                Message<String> receivedMessage = MessageBuilder.withPayload(payload).build();
                archiveMessage(payload);
                sendMessage(receivedMessage);
            }

        }, 0, 50, MILLISECONDS);
    }

    private String getPayload() {
        try {
            List<String> allLines = Files.readAllLines(Paths.get(destination.getName()));
            String currentPayload = allLines.get(allLines.size() - 1);

            if(!currentPayload.equals(previousPayload)) {
                previousPayload = currentPayload;
                return currentPayload;
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }

        return null;
    }

    private void archiveMessage(String payload) {
        try {
            Files.write(Paths.get(ARCHIVE), (payload + "\n").getBytes(), CREATE, APPEND);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

}
```
当实现一个自定义binder，这一步 不是强制的，你可以总是 使用现有的 MessageProducer 实现。


Provide a MessageHandler implementation

MessageHandler 提供 生成 event 的逻辑。    
一个例子：
```java
public class FileMessageHandler implements MessageHandler{

    @Override
    public void handleMessage(Message<?> message) throws MessagingException {
        //write message to file
    }

}
```
在实现一个自定义binder时，这一步不是强制的，你总可以使用现有的MessageHandler 实现。


Provide a Binder implementation

你现在可以提供 你自己的 Binder 的实现，非常简单：
1. 继承 AbstractMessageChannelBinder 类
2. 指定你的 ProvisioningProvider 作为 AbstractMessageChannelBinder 的 通用参数
3. 重写 createProducerMessageHandler 和 createConsumerEndpoint 方法。

```java
public class FileMessageBinder extends AbstractMessageChannelBinder<ConsumerProperties, ProducerProperties, FileMessageBinderProvisioner> {

    public FileMessageBinder(
            String[] headersToEmbed,
            FileMessageBinderProvisioner provisioningProvider) {

        super(headersToEmbed, provisioningProvider);
    }

    @Override
    protected MessageHandler createProducerMessageHandler(
            final ProducerDestination destination,
            final ProducerProperties producerProperties,
            final MessageChannel errorChannel) throws Exception {

        return message -> {
            String fileName = destination.getName();
            String payload = new String((byte[])message.getPayload()) + "\n";

            try {
                Files.write(Paths.get(fileName), payload.getBytes(), CREATE, APPEND);
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        };
    }

    @Override
    protected MessageProducer createConsumerEndpoint(
            final ConsumerDestination destination,
            final String group,
            final ConsumerProperties properties) throws Exception {

        return new FileMessageProducer(destination);
    }

}
```

Create a Binder Configuration

这是必须的   
你需要创建一个 Spring Configuration 来初始化 bean for 你的binder 实现(和其他你可能需要的bean)。

```java
@Configuration
public class FileMessageBinderConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public FileMessageBinderProvisioner fileMessageBinderProvisioner() {
        return new FileMessageBinderProvisioner();
    }

    @Bean
    @ConditionalOnMissingBean
    public FileMessageBinder fileMessageBinder(FileMessageBinderProvisioner fileMessageBinderProvisioner) {
        return new FileMessageBinder(null, fileMessageBinderProvisioner);
    }

}
```

定义你的binder 到 META-INF/spring.binders, 指明 binder的名字 和 Binder配置类的 全名：
```
myFileBinder:\
com.example.springcloudstreamcustombinder.config.FileMessageBinderConfiguration
```

---
#### 108 Configuration Options
SCStream 支持通用配置选项以及binding和binder的配置。   
一些binder 允许额外的绑定属性支持特定于中间件的功能。

可以通过SBoot支持的任何机制为SCStream 应用程序提供配置选项，包括 应用程序参数，环境变量，yaml，properties。

---
#### Binding Service Properties

这些属性通过 org.springframework.cloud.stream.config.BindingServiceProperties 暴露

**spring.cloud.stream.instanceCount**   
应用部署的实例数。必须为生产者端的partitioning 设置。必须在消费者端设置，如果使用RabbitMQ和Kafka 且 autoRebalanceEnabled=false。   
默认 1

**spring.cloud.stream.instanceIndex**       
应用的实例的下标。数值范围[0, instanceCount-1]，用于RabbitMQ和Kafka的 partitioning，如果autoRebalanceEnabled=false。在Cloud Foundry 自动设置为对应的应用的实例下标

**spring.cloud.stream.dynamicDestinations         
目的地列表，能被动态绑定(例如，在一个动态routing场景)，如果设置，只有 列觉的 目的地能被绑定             
默认 空(使得任何目的地能被绑定)

**spring.cloud.stream.defaultBinder**       
如果配置了多个binder，默认的binder     
默认 空

**spring.cloud.stream.overrideCloudConnectors**     
只有当 cloud profile被激活，且应用中有 SCConnectors 时，这个属性会启用。      
如果值是false(默认), binder检测合适的bound服务，使用这个服务来创建连接。      
如果是true，binder完全无视 bound 服务，基于SBoot属性(如 spring.rabbitmq.*)      
默认 false

**spring.cloud.stream.bindingRetryInterval**        
创建binding时 retry的间隔(单位秒)。设置为0会立刻失败，阻止应用启动。      
默认 30

---
#### 108.2 Binding Properties

binding属性通过 ```spring.cloud.stream.bindings.<bindingName>.<property>=<value>```来支持。

例如，对于下面的 function
```java
@Bean
public Function<String, String> uppercase() {
    return v -> v.toUpperCase();
}
```
有2个 binding 名字， uppercase-in-0 用于input， uppercase-out-0用于输出。

为了避免重复，SCStream支持 对于全部binding 的设置，以```spring.cloud.stream.default.<property>=<value>``` 和 ```spring.cloud.stream.default.<producer|consumer>.<property>=<value>``` 的格式 for 公共的binding 属性。

为了避免重复 扩展binding属性，使用```spring.cloud.stream.<binder-type>.default.<producer|consumer>.<property>=<value>```
。。估计是 kafka，rabbit 的属性。。不是，是指 消费者，生产者。

---
#### 108.2.1 Common Binding Properties

这些属性通过 org.springframework.cloud.stream.config.BindingProperties 暴露

下面的 binding 属性 对于 input output binding 都 可用。前缀必须是 ```spring.cloud.stream.bindings.<bindingName>``` (比如 spring.cloud.stream.bindings.uppercase-in-0.destination=ticktock)

默认值 能通过 spring.cloud.stream.default 前缀。(如，spring.cloud.stream.default.contentType=application/json)

> destination       
> binding的 在中间件的 目标目的地。如果binding代表了一个 消费者binding，可以绑定到 多个 目的地，目的地的名字通过 逗号分隔的字符串。如果不是，需要使用真实的 binding名字。     
> 这个属性的默认值 不能被 覆盖。

> group     
> binding的 消费者组，只应用于 inbound binding        
> 默认 null (表示匿名消费者)

> contentType       
> binding的 content type       
> 默认 application/json

> binder      
> 这个binding使用的binder。     
> 默认 null

---
#### Consumer Properties
这些属性 被 org.springframework.cloud.stream.binder.ConsumerProperties 暴露。

下面的binding属性 用于 input binding。 前缀 ```spring.cloud.stream.bindings.<bindingName>.consumer.```

默认值通过 ```spring.cloud.stream.default.consumer``` 前缀设定。

> autoStartup       
> 是否需要自动启动
> 默认 true


> concurrency       
> inbound consumer 的 并发     
> 默认 1

> partitioned       
> 消费者是否 从 一个 partitioned 生产者 接收消息
> 默认 false

> headerMode        
> 设置为none，disable header parsing on input(input 不带header？还是 input的 header不解析)。仅对 本身不支持消息header但需要 header嵌入 的 消息中间件 有效。
> 有用的，当消费数据 从 非SCStream 应用 when 源生头不支持。     
> 设置为headers，使用中间件 自己的 header 机制    
> 设置为embeddedHeaders, 嵌入 header 到 消息 payload    
> 默认，依赖于 binder 实现

> maxAttempts       
> 如果处理失败，尝试处理的次数(包括第一次)， 设置为1,禁用retry       
> 默认 3

> backOffInitialInterval        
> retry时，backoff 初始 间隔      
> 默认 1000

> backOffMaxInterval        
> 最大间隔      
> 默认 10000

> backOffMultiplier      
> 系数        
> 默认 2.0

> defaultRetryable      
> 当listener 抛出的异常 不在 retryableExceptions 中，是否 可以重试      
> 默认 true

> instanceCount     
> 如果设置一个 >= 0 的值，允许定制 这个消费者的 实例数 (如果和 spring.cloud.stream.instanceCount 不同)。 负数，就和 spring.cloud.stream.instanceCount 一样。        
> 默认 -1

> instanceIndexList     
> 与不支持 源生partitioning(如 RabbitMQ) 的 binder 一起使用。允许 应用实例 从 多个partition 消费。       
> 默认 空

> retryableExceptions       
> map，异常类名是key，boolean 是value。指明这些异常(和子类) 是否会 retry。如spring.cloud.stream.bindings.input.consumer.retryable-exceptions.java.lang.IllegalStateException=false                 
> 默认 空

> useNativeDecoding     
> 设置为true，inbound 消息 直接被 client library 反序列化，这个client library 必须是 对应的 (比如，设置一个合适的 Kafka producer value deserializer).
> 这个配置被使用时，inbound 消息 不会基于 binding的 contentType 进行 unmarshalling (数据编出)。    
> 当原生decoding被使用，生产者有责任使用合适的编码器来序列化 输出的消息。
> 如果 原始的 编码，解码 被使用， headerMode=embeddedHeaders 属性被忽略，headers不会被 嵌入到消息。          
> 默认 false

> multiplex             
> 设置为true，底层的binder 会在 同一输入binding 上进行 复用       
> 默认 false

---
#### 108.2.3 Advanced Consumer Configuration

为了进一步配置 底层的 消息监听容器 for 消息驱动的消费者，增加一个ListenerContainerCustomizer bean到 应用context。    
它会被调用 after 上面的属性(。。应该是本章的前面部分)已经被应用，能被用于 设置额外属性。   
类似的，for polled消费者，增加一个 MessageSourceCustomizer bean。

下面是RabbitMQ binder 的例子：
```java
@Bean
public ListenerContainerCustomizer<AbstractMessageListenerContainer> containerCustomizer() {
    return (container, dest, group) -> container.setAdviceChain(advice1, advice2);
}

@Bean
public MessageSourceCustomizer<AmqpMessageSource> sourceCustomizer() {
    return (source, dest, group) -> source.setPropertiesConverter(customPropertiesConverter);
}
```
。。advice1 代表了什么？

---
#### 108.2.4 Producer Properties
通过 org.springframework.cloud.stream.binder.ProducerProperties 暴露。

下面的binding 属性 只用于 output bindings, 前缀必须是 ```spring.cloud.stream.bindings.<bindingName>.producer```。如 spring.cloud.stream.bindings.func-out-0.producer.partitionKeyExpression=payload.id

默认值能被设置，通过前缀 spring.cloud.stream.default.producer. 如 spring.cloud.stream.default.producer.partitionKeyExpression=payload.id

> autoStartup       
> 是否这个 消费者(？生产者吧？) 需要自动启动       
> 默认：true

> partitionKeyExpression    
> SpEL表达式，决定如何 对出站数据 进行分区。如果设置，这个binding上的 出站数据 是被分区的。partitionCount 必须被设置为>1的值 才能生效。   
> 默认 null

> partitionSelectorName     
> 实现 PartitionSelectorStrategy 的 bean的名字，用来基于 partition key 决定 partition id. 和 partitionSelectorExpression 属性 互斥    
> 默认 null

> partitionSelectorExpression       
> SpEL 表达式 for 定制 partition selection. 如果没有设置，partition 被 select as hashCode(key) % partitionCount, 这里的key 是通过 partitionKeyExpression 计算出来      
> 默认 null

> partitionCount    
> 数据的 目标分区数，如果分区被激活。必须设置为 >1，如果 生产者是 分区的。 在Kafka，这个被当作 hint。目标topic的分区数 和 这个属性的 较大值 被使用  
> 默认 1

> requiredGroups
> 逗号分隔的 group的 列表，生产者必须确保 消息被传递到 这些group，即使它们是创建后开始的。(比如，通过RabbitMQ的 预先创建持久队列)
> 默认 依赖于binder实现

> useNativeEncoding     
> 当设置为true，出站消息 被 client library 直接 序列化，这个必须被 对应配置。(比如，设置一个合适的 Kafka 生产者value serializer)。
> 当这个属性被使用，出站消息的 marshall 不是基于 binding的 contentType。当原生encoding被使用，consumer负责使用 对应的 decoder来反序列化 入站消息。
> 当原生编码 和解码被使用， headerMode=embeddedHeaders 属性被忽视，header不能嵌入到message。        
> 默认 false

> errorChannelEnabled       
> 当设置为true，如果binder 支持 异步发送 结果，发送失败将发送到 目标的 error channel。      
> 默认 false

---
#### 109 Content Type Negotiation(谈判，协商)

数据转换 是 任何消息驱动 微服务架构 的核心功能之一。   
在SCStream中，数据被表示为一个 Spring的 Message对象。一个消息可能必须被转换到一个 所需的形状或大小， before 达到它的目的地。这是必须的，因为2个原因：
1. 转换 入站消息的 内容 以匹配 应用提供的 handler的 签名
2. 转换 出站消息 到 wire format

wire format 是 传统的 byte[]，但是它被 binder实现 管理。

在SCStream，消息转换 通过 MessageConverter类 来完成。

---
#### 109.1 Mechanics(机械学，力学，运作方式)

为了更好理解 运作方式 和 content-type negotiation 的必要性，我们来看一个 简单用例：使用下面的 消息处理者：
```
public Function<Person, Person(?下面说输出String)> personFunction {..}
```
为了简便，我们假设应用里只有这 一个 handler。

上面的例子，期望一个 Person 对象 作为参数， 产生一个 String 类型 作为输出。   
为了 让框架能 把 入站的 Message 传递给这个 处理者，它必须 以某种方式 将 Message类型的 payload 转为 Person类型。   
换句话说，框架必须 找到并应用 合适的 MessageConverter。 为了达到这种效果，框架需要 从 用户那里 获得一些指示。   
一个指示已经提供了，通过 handler方法的 签名 (Person 类型)。理论上( 且 对于某些用例)来说，这已经足够了。   
但是 对于大部分用例，为了 选择合适的 MessageConverter，框架需要一个 额外的信息，就是 contentType

SCStream 提供3种机制 来定义 contentType (按照优先级顺序)：
1. HEADER，contentType 能通过 消息它自己 来传递，通过提供一个 contentType 头，你声明了 用于定位和应用 合适的MessageConverter 的 content type。
2. BINDING，能为每个 目的地binding 设置 它的 contentType,通过 spring.cloud.stream.bindings.input(这个是目的地的真实名字，本用例中是 input。。但是我怎么感觉不是呢。。).content-type 属性。
3. DEFAULT，如果 contentType 不在 Message的header 和 binding 中存在，默认的 application/json 被用来 定位和应用 合适的 MessageConverter

当 handler 返回的不是 void，如果返回值已经是Message对象，那么这个就会成为 payload。
如果返回值不是Message，会构造一个Message，以返回值作为payload，同时从 输入Message 继承的header 会减去 SpringIntegrationProperties.messageHandlerNotPropagatedHeaders 定义或过滤的 header。默认情况下，只有一个header，就是contentType。这意味着 新的Message 没有contentType 头。
你始终可以 选择不从处理程序方法返回Message，你可以在其中注入你希望的任何头。

如果有一个内部的 pipeline， Message 被发送到 下一个 handler，经历相同的转换过程。
如果没有 internal pipeline 或已经到达了 结尾， Message 被发送到 output 目的地。

---
#### 109.1.1 Content Type versus Argument Type
上面提到，为了 框架 选择 合适的 MessageConverter， 它需要 参数类型 和 可选的content type信息。   
选择 合适的 MessageConverter 的逻辑 在 参数解析中( HandlerMethodArgumentResolvers), 这个 触发 before 用户定义的 handler方法 (此时 框架已经知道 实参)。  
如果 参数类型 和 当前payload的类型 不匹配，框架 委托 预先配置的 MessageConverters 栈 来查看 是否有 一个 能用于 转换 payload。   
你可以看到 MessageConverter 的 Object fromMessage(Message<?> message, Class<?> targetClass); 操作 接收 targetClass 作为它的参数。    
框架也确保 提供的 Message 总有一个 contentType header。 如果没有 contentType，那么会注入 binding的 contentType 或者 默认的 contentType。  
contentType 参数类型的组合 是框架 确定消息是否可以转换为目标类型的 机制。    
如果没有合适的 MessageConverter，异常抛出，你可以 处理这个异常，通过 增加自定义的 MessageConverter。

> 如果 payload类型 和 handler声明的类型 匹配，不会转换，直接传递payload。      
> 注意那些 接受 Message<?> 或 Object 作为参数的 handler方法。  
> 通过声明 类型为Object (instanceof java中所有)，你本质上丧失了 转换过程。

不要期望 Message 只靠 contentType 就能被转为 其他类型。  
记住 contentType 是 目标类型的补充， 如果你希望，你可以提供一个 hint，MessageConvert 可能会/可能不会 参考。

---
#### 109.1.2 Message Converters
MessageConverters 定义了 2个方法：
```
Object fromMessage(Message<?> message, Class<?> targetClass);

Message<?> toMessage(Object payload, @Nullable MessageHeaders headers);
```

理解 这些方法的 contract(合同。。格式？定义？) 和 用途 很重要，特别是在 SCStream 上下文。

fromMessage 方法 转换一个 输入的 Message 到一个 参数类型。Message 的 payload 可以是任何类型，它由 MessageConverter 的实现 来决定 支持多种类型。
例如，一些 JSON converter 可以支持 byte[],String 等 的 payload type。   
这是重要的，当应用包含 internal pipeline (input->handler1->handler2->...->output), 且 上一个handler的输出 可能不是 初始的 wire format

然而，toMessage 方法有一个 更 严格的 contract，必须转换 Message 到 wire format: byte[]

所以，对于所有的 意图和目的 (特别是当你实现你自己的 converter时) 你你认为这2种方法具有下面的方法签名：
```java
Object fromMessage(Message<?> message, Class<?> targetClass);

Message<byte[]> toMessage(Object payload, @Nullable MessageHeaders headers);
```
。。<byte[]>  toMessage.


---
#### 109.2 Provided MessageConverters
就像上面提到的， 框架提供了一个 MessageConverters 的 stack 来处理 最常见的用例。   
下面的列表 描述了 提供的 MessageConverters。按照 优先级顺序：
1. ApplicationJsonMessageMarshallingConverter，MappingJackson2MessageConverter 的变种，支持 把 Message的 payload 转换 to/from POJO，当contentType 是 application/json (默认)
2. ByteArrayMessageConverter，支持 把 Message 的payload 从 byte[] 转换为 byte[]，当 contentType 是 application/octet-stream。它本质上是一种传递，主要是为了向后兼容而存在。
3. ObjectStringMessageConverter，支持 转换任何类型到 String，当contentType 是text/plain。它调用 Object的 toString()，或者如果payload 是byte[], 那么 new String(byte[])。
4. JsonUnmarshallingConverter，类似ApplicationJsonMessageMarshallingConverter。它支持 转换任何类型 当 contentType 是 application/x-java-object。它期望 真实类型 被嵌入在 contentType 中 作为一个 属性 (例如，application/x-java-object;type=foo.bar.Cat)

当没有 合适的 converter，框架抛出异常。 如果发生这种情况，你应该检查你的 代码和配置， 确保你没有遗漏 (比如，确保你提供了 contentType 通过binding 或 header)。   
当然，最有可能的是，你使用了一些 不常见的 用例 (比如一个自定义的 contentType)，且 目前的 MessageConverters 栈 不知道如何 转换。 此时，你需要添加自己的 MessageConverter。

---
#### 109.3 User-defined Message Converters

SCStream 暴露一个机制 来 定义和注册 额外的 MessageConverters。   
为了使用这个机制： 实现MessageConverter接口，配置为 @Bean。 它会自动被添加到 现有的 MessageConverter 栈

重要的：理解 自定义的MessageConverter 会被 添加到 现有栈的 头上。 因此，自定义MessageConverter 实现 优先于 现有的实现，这使得你可以 覆盖 现有的转换器。

。。。如果多个 自定义 MessageConverter 呢。。 或许 @Order？

下面是 创建了一个 MessageConverter，支持 application/bar 的 content type:
```java
@SpringBootApplication
public static class SinkApplication {

    ...

    @Bean
    public MessageConverter customMessageConverter() {
        return new MyCustomMessageConverter();
    }
}

public class MyCustomMessageConverter extends AbstractMessageConverter {

    public MyCustomMessageConverter() {
        super(new MimeType("application", "bar"));
    }

    @Override
    protected boolean supports(Class<?> clazz) {
        return (Bar.class.equals(clazz));
    }

    @Override
    protected Object convertFromInternal(Message<?> message, Class<?> targetClass, Object conversionHint) {
        Object payload = message.getPayload();
        return (payload instanceof Bar ? payload : new Bar((byte[]) payload));
    }
}
```

SCStream 也提供了 对 Avro-based converters 和 schema evolution 的支持。

SCStream 允许 应用间的 通信。 Inter-application 通信 是一个复杂的问题，涉及多个问题：
1. Connecting Multiple Application Instances
2. Instance Index and Instance Count
3. Partitioning

---
#### 109.4 Connecting Multiple Application Instances

SCStream 使得 单体的SBoot应用 连接 消息系统 更容易，SCStream的传统场景是 多应用 pipeline的创建，使得微服务应用可以互相发送数据。   
你可以实现这个场景 通过 关联 相邻应用的 input 和 output 目的地。

假设 一个设计： Time Source 应用 发送数据到 Log Sink应用。 你可以使用一个 公共目的地 名为 ticktock for 2个应用中的 binding。

Time Source (binding name 是output) 设置：
```properties
spring.cloud.stream.bindings.output.destination=ticktock
```

Log Sink (binding name 是input) 设置：
```properties
spring.cloud.stream.bindings.input.destination=ticktock
```

---
#### 109.5 Instance Index and Instance Count

当 伸缩 SCStream 应用时， 每个 实例 可以收到信息：本应用有多少个实例，本实例的下标。
SCStream 通过 spring.cloud.stream.instanceCount, spring.cloud.stream.instanceIndex 属性 来 这么做。   
例如，如果一个 HDFS 有3个实例，3个实例中的 spring.cloud.stream.instanceCount 都是3, 每个实例的 spring.cloud.stream.instanceIndex 分别是0,1,2

当SCStream 应用 通过 SC Data Flow 部署，这些属性 被自动配置。
> 如果SCStream 被独立部署， 这些属性 必须被 正确设置，默认instanceCount 是1, instanceIndex 是0.

在一个 伸缩的场景中，这2个属性的正确配置 是非常重要的 for 解决分区行为。 某些binder (如Kafka binder) 始终需要这2个属性 来确保 数据 在多个消费者实例 中 正确拆分。

---
#### Partitioning
SCStream 中的 partitioning 包含2个任务：
1. configuring output bindings for partitioning
2. configuring input bindings for partitioning

---
#### Configuring Output Bindings for Partitioning

你可以配置一个 output binding 来发送 分区后的数据，通过 设置 它的 partitionKeyExpression 或 partitionKeyExtractorName 属性(这2个互斥)，还有 partitionCount 属性。

例如，下面的是 有效的，典型的：
```properties
spring.cloud.stream.bindings.func-out-0.producer.partitionKeyExpression=payload.id
spring.cloud.stream.bindings.func-out-0.producer.partitionCount=5
```

基于上面的配置，数据被发送到 目标 partition ，by 使用下面的逻辑：

每条 被发送到 分区的 output binding 的 消息的 分区key 被基于 partitionKeyExpression 计算。
partitionKeyExpression 是SpEL 表达式，对 出站Message 进行 eval 来 提取 分区key。

如果SpEL 表达式 不能满足你的 要求，你可以 自己计算 分区key， 通过提供一个 PartitionKeyExtractorStrategy 的实现，且 配置它为 @Bean。  
如果你有多个 PartitionKeyExtractorStrategy 的bean 在 application context 中 可用，你可以 通过 partitionKeyExtractorName 属性 来 唯一指定一个：
```
--spring.cloud.stream.bindings.func-out-0.producer.partitionKeyExtractorName=customPartitionKeyExtractor
--spring.cloud.stream.bindings.func-out-0.producer.partitionCount=5
. . .
@Bean
public CustomPartitionKeyExtractorClass customPartitionKeyExtractor() {
    return new CustomPartitionKeyExtractorClass();
}
```

之前版本的 SCStream中，你可以指定PartitionKeyExtractorStrategy 的 实现类
通过spring.cloud.stream.bindingds.output.producer.partitionKeyExtractorClass 属性   
从3.0开始，移除了这个 属性。

一旦 message 的key 被计算出来，分区选择过程 决定 目标分区 as 在 0 到 partitionCount-1 之间的值。  
默认的计算，也是可以用在大多数场景的，基于下面的公式： key.hashCode()%partitionCount.   
这个能在 binding 上自定义，通过 partitionSelectorExpression 属性的 SpEL表达式 或 配置一个 PartitionSelectorStrategy 的实现 作为bean。   
和PartitionKeyExtractorStrategy 类似，当有多个 该类型的 bean 在应用上下文中 可用时，可以通过 partitionSelectorName 属性来指定一个。
```
--spring.cloud.stream.bindings.func-out-0.producer.partitionSelectorName=customPartitionSelector
. . .
@Bean
public CustomPartitionSelectorClass customPartitionSelector() {
    return new CustomPartitionSelectorClass();
}
```

之前版本，可以通过 spring.cloud.stream.bindings.output.producer.partitionSelectorClass 属性 来 指定 PartitionSelectorStrategy 的 类名。 3.0开始，移除。

---
#### Configuring Input Bindings for Partitioning

一个intput binding (binding name 是 uppercase-in-0) 被配置 用于 接收 分区后的数据， by 设置它的 partitioned 属性，还有 instanceIndex，instanceCount 属性：
```
spring.cloud.stream.bindings.uppercase-in-0.consumer.partitioned=true
spring.cloud.stream.instanceIndex=3
spring.cloud.stream.instanceCount=5
```

instanceCount 是 该应用的所有 实例       
instanceIndex 是 用于 区分 多个实例 的值，必须是 0 到 instanceCount - 1。   
instanceIndex 帮助每个 应用实例 确认 它从哪个分区 接受数据。   
如果原生不支持 分区的话，就必须需要这个配置。   
比如，在RabbitMQ中，每个partition 有一个 queue，queue的名字 包含了 实例下标。   
在Kafka中，如果 autoRebalanceEnabled 是true(默认)，kafka负责跨实例分配分区，所以不需要 这些属性。   
如果autoRebalanceEnabled 是false，instanceCount，instanceIndex 被 binder 使用 来决定 实例订阅哪个 分区。   
binder 代理Kafka来 分配分区，这可以用于 你希望 一个 分区的消息 都被同一个 实例消费。

多实例 for 分区数据处理 在 standalone的情况下 是复杂的。 SpringCloud Dataflow 能处理这种 情况，通过 扩散 input 和output 值值，让你依赖 runtime infrastructure 来提供 关于 instanceCount 和 instanceIndex 的信息。

---
#### 110 Testing

支持测试微服务，而不需要连接到 消息系统。

---
#### 110.1 Spring Integration Test Binder

老的测试binder 定义在 spring-cloud-stream-test-support 模块，用于 促进 unit testing。   
这种轻量级的 对于 很多情况都够用了， 但是 它依然要求 真实的binder， 所以 我们deprecate it。

为了 打破 unit测试 和 integration 测试 之间的 空隙，开发了一个新的 test binder，这个binder 使用了 Spring Integration 框架 作为 JVM内置的 消息代理。

---
#### 110.1.1 Test Binder configuration

启动 Spring Integration Test Binder ，你需要做：
1. 增加依赖
2. 移除 spring-cloud-stream-test-support 的依赖

增加依赖：
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream</artifactId>
    <version>${spring.cloud.stream.version}</version>
    <type>test-jar</type>
    <scope>test</scope>
    <classifier>test-binder</classifier>
</dependency>
```

或者 build.gradle.kts:
```
---
testImplementation("org.springframework.cloud:spring-cloud-stream") {
    artifact {
        name = "spring-cloud-stream"
        extension = "jar"
        type ="test-jar"
        classifier = "test-binder"
    }
}
---
```

---
#### 110.1.2 Test Binder usage

现在你可以测试你的 微服务 就像一个简单 unit test：
```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class SampleStreamTests {

    @Autowired
    private InputDestination input;

    @Autowired
    private OutputDestination output;

    @Test
    public void testEmptyConfiguration() {
        this.input.send(new GenericMessage<byte[]>("hello".getBytes()));
        assertThat(output.receive().getPayload()).isEqualTo("HELLO".getBytes());
    }

    @SpringBootApplication
    @Import(TestChannelBinderConfiguration.class)
    public static class SampleConfiguration {
        @Bean
        public Function<String, String> uppercase() {
            return v -> v.toUpperCase();
        }
    }
}
```

如果你需要更多的控制 或想测试 多个配置 在相同的 test 单元，你可以：
```java
@EnableAutoConfiguration
public static class MyTestConfiguration {
    @Bean
    public Function<String, String> uppercase() {
            return v -> v.toUpperCase();
    }
}

. . .

@Test
public void sampleTest() {
    try (ConfigurableApplicationContext context = new SpringApplicationBuilder(
                TestChannelBinderConfiguration.getCompleteConfiguration(
                        MyTestConfiguration.class))
                .run("--spring.cloud.function.definition=uppercase")) {
        InputDestination source = context.getBean(InputDestination.class);
        OutputDestination target = context.getBean(OutputDestination.class);
        source.send(new GenericMessage<byte[]>("hello".getBytes()));
        assertThat(target.receive().getPayload()).isEqualTo("HELLO".getBytes());
    }
}
```

对于那些 你有多个bindings and/or 多个input 和out， or 想要明确 你发送和接收的 目的地的名字， InputDestination和 OutputDestination 中的 send()和 receive() 方法 被重载 来允许你 提供 input 和 output 目的地的 名字。

考虑下面的代码
```java
@EnableAutoConfiguration
public static class SampleFunctionConfiguration {

    @Bean
    public Function<String, String> uppercase() {
        return value -> value.toUpperCase();
    }

    @Bean
    public Function<String, String> reverse() {
        return value -> new StringBuilder(value).reverse().toString();
    }
}
```
测试：
```java
@Test
public void testMultipleFunctions() {
    try (ConfigurableApplicationContext context = new SpringApplicationBuilder(
            TestChannelBinderConfiguration.getCompleteConfiguration(
                    SampleFunctionConfiguration.class))
                            .run("--spring.cloud.function.definition=uppercase;reverse")) {

        InputDestination inputDestination = context.getBean(InputDestination.class);
        OutputDestination outputDestination = context.getBean(OutputDestination.class);

        Message<byte[]> inputMessage = MessageBuilder.withPayload("Hello".getBytes()).build();
        inputDestination.send(inputMessage, "uppercase-in-0");
        inputDestination.send(inputMessage, "reverse-in-0");

        Message<byte[]> outputMessage = outputDestination.receive(0, "uppercase-out-0");
        assertThat(outputMessage.getPayload()).isEqualTo("HELLO".getBytes());

        outputMessage = outputDestination.receive(0, "reverse-out-0");
        assertThat(outputMessage.getPayload()).isEqualTo("olleH".getBytes());
    }
}
```

对于 要增加额外的 mapping properties 比如destination。   
比如，考虑一个测试，我们 明确 map uppercase 函数的 input，output 到 myInput,myOutput bindings name:

```java
@Test
public void testMultipleFunctions() {
    try (ConfigurableApplicationContext context = new SpringApplicationBuilder(
            TestChannelBinderConfiguration.getCompleteConfiguration(
                    SampleFunctionConfiguration.class))
                            .run(
                            "--spring.cloud.function.definition=uppercase;reverse",
                            "--spring.cloud.stream.bindings.uppercase-in-0.destination=myInput",
                            "--spring.cloud.stream.bindings.uppercase-out-0.destination=myOutput"
                            )) {

        InputDestination inputDestination = context.getBean(InputDestination.class);
        OutputDestination outputDestination = context.getBean(OutputDestination.class);

        Message<byte[]> inputMessage = MessageBuilder.withPayload("Hello".getBytes()).build();
        inputDestination.send(inputMessage, "myInput");
        inputDestination.send(inputMessage, "reverse-in-0");

        Message<byte[]> outputMessage = outputDestination.receive(0, "myOutput");
        assertThat(outputMessage.getPayload()).isEqualTo("HELLO".getBytes());

        outputMessage = outputDestination.receive(0, "reverse-out-0");
        assertThat(outputMessage.getPayload()).isEqualTo("olleH".getBytes());
    }
}
```

---
#### 110.1.3 Test Binder and PollableMessageSource

Spring Integration Test Binder 也允许你 写测试，当 使用 PollableMessageSource.

重要的事情是，需要理解，polling 不是事件驱动的， PollableMessageSource 是一个策略，暴露了发布消息的操作。   
poll的频率 和 使用多少线程 或 从哪里poll 是完全 取决于你的。  
换句话说，你负责 配置 Poller 或 Threads 或 消息的真实来源。

```java
@Test
public void samplePollingTest() {
    ApplicationContext context = new SpringApplicationBuilder(SamplePolledConfiguration.class)
                .web(WebApplicationType.NONE)
                .run("--spring.jmx.enabled=false", "--spring.cloud.stream.pollable-source=myDestination");
    OutputDestination destination = context.getBean(OutputDestination.class);
    System.out.println("Message 1: " + new String(destination.receive().getPayload()));
    System.out.println("Message 2: " + new String(destination.receive().getPayload()));
    System.out.println("Message 3: " + new String(destination.receive().getPayload()));
}

@Import(TestChannelBinderConfiguration.class)
@EnableAutoConfiguration
public static class SamplePolledConfiguration {
    @Bean
    public ApplicationRunner poller(PollableMessageSource polledMessageSource, StreamBridge output, TaskExecutor taskScheduler) {
        return args -> {
            taskScheduler.execute(() -> {
                for (int i = 0; i < 3; i++) {
                    try {
                        if (!polledMessageSource.poll(m -> {
                            String newPayload = ((String) m.getPayload()).toUpperCase();
                            output.send("myOutput", newPayload);
                        })) {
                            Thread.sleep(2000);
                        }
                    }
                    catch (Exception e) {
                        // handle failure
                    }
                }
            });
        };
    }
}
```

上面的例子，生产 3条消息 以2秒的间隔，发送它们到 输出目的地。   
我们会收到：
```
Message 1: POLLED DATA
Message 2: POLLED DATA
Message 3: POLLED DATA
```

你可以看到 数据是相同的，这是因为 这个binder 定义了一个 默认的MessageSource 的实现- 从这里拿消息是通过 poll()操作。   
对于大多数场景够了， 有些场景 你想要 定义你自己的 MessageSource，这样做，只需要 配置一个 MessageSource类型的 bean，在你的 测试配置中 来提供你自己的 message sourcing 的实现。

```java
@Bean
public MessageSource<?> source() {
    return () -> new GenericMessage<>("My Own Data " + UUID.randomUUID());
}
```

输出就会变成：
```
Message 1: MY OWN DATA 1C180A91-E79F-494F-ABF4-BA3F993710DA
Message 2: MY OWN DATA D8F3A477-5547-41B4-9434-E69DA7616FEE
Message 3: MY OWN DATA 20BF2E64-7FF4-4CB6-A823-4053D30B5C74
```

不要命名这个bean 为 messageSource，这会和 SBoot提供的 同名(不同类型) 的 bean 发生冲突。

---
#### 111 Health Indicator
SCStream 提供了一个 health indicator for binders。它以binders 的名字注册，能启用或禁用 通过 management.health.binders.enabled 属性，默认true。

为了启动 health indicator，你需要 启用 web 和 actuator 依赖。

/actuator/health   
默认，你只收到 顶层的应用状态。为了收到 全部明细，你需要 management.endpoint.health.show-details=ALWAYS。

health indicator 是特定于 binder的，某些binder实现可能不一定提供 health indicator

完全禁用： management.health.binders.enabled=false， 提供你自己的 HealthIndicator bean。 因为SBoot的 健康指标 仍然会 获取这些自定义bean。

当你有多个 binders，禁用某一部分，设置 management.health.binders.enabled=false 到 binder 配置的环境中。

如果有多个 binder 在classpath，但不是 全都被使用了， 可能会导致 健康指标的 上下文中 的一些问题。

如果 classpath中有 Kafka 和 Kafka Streams 的binder，但是 只使用了 Kafka Streams binder 在代码中。
由于Kafka binder 没有被使用， 健康指标 查看 它是否有 注册的目的地，会失败， 顶层的应用的健康检查状态 就是 DOWN。
此时，你需要 移除依赖

---
#### Samples

https://github.com/spring-cloud/spring-cloud-stream-samples

---
#### Deploying Stream Applications on CloudFoundry







