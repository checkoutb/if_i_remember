
https://docs.spring.io/spring-cloud-sleuth/docs/3.0.2/reference/htmlsingle/


---

#### 2.1 Introducing Spring Cloud Sleuth

SC Sleuth 提供API for 分布式 追踪(tracing) 方案 for Spring Cloud. 它和OpenZipkin Brave 集成。

SC Sleuth 允许你 trace 你的请求 和 消息，这样你可以 correlate(相互关联) 那个 交流/信息(communication) 到 对应的 日志实体。   

你也可以 输出 trace 信息到 外部系统 进行 后续可能的可视化。SC Sleuth 支持 OpenZipkin 系统。

---
#### 2.1.1 Terminology

|术语|解释|
|:---|:---|
|Span|工作的基础单元，例如，发送一个RPC 是一个新的Span，就像发送响应到 RPC。Span也有其他数据，比如描述，时间戳事件，k-v注解(标记)，Span的ID，处理ID(一般是IP地址)，Span可以被启动和停止，它们跟踪它们的 timing 信息。一旦你创建一个span，你必须 在未来 停止它。|
|Trace|span的集合，以树状结构。例如，如果你运行一个 分布式 大数据 存储，trace可能由 PUT请求 组成|
|Annotation/Event|用来实时记录event的存在|

|Annotation/Event|解释|
|:---|:---|
|cs|Client Sent, 客户端已经生成请求，这个 注解 表示span 的开始|
|sr|Server Received, 服务器端收到请求，且已经开始处理。从此时间戳 减去 cs时间戳 等于 网络延迟|
|ss|Server Sent, 服务器已发送，注解在 请求处理完成时(当响应已经发送出去的时候)，ss-sr==服务器处理时间|
|cr|Client Received, 服务器已收到，标识了span 的结束，客户端已经收到服务器的响应。cr-cs 就是 客户端从 服务器收到响应的 整个时间。|


---
#### 2.2 Deploying Your First Spring Cloud sleuth-based Application

---
#### 2.2.1 Creating the POM

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>myproject</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <!-- Use the latest compatible Spring Boot version. You can check https://spring.io/projects/spring-cloud for more information -->
        <version>$2.4.3</version>
    </parent>

    <!-- Spring Cloud Sleuth requires a Spring Cloud BOM -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <!-- Provide the latest stable Spring Cloud release train version (e.g. 2020.0.0) -->
                <version>${release.train.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <!-- (you don't need this if you are using a GA version) -->
    <repositories>
        <repository>
            <id>spring-snapshots</id>
            <url>https://repo.spring.io/snapshot</url>
            <snapshots><enabled>true</enabled></snapshots>
        </repository>
        <repository>
            <id>spring-milestones</id>
            <url>https://repo.spring.io/milestone</url>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>spring-snapshots</id>
            <url>https://repo.spring.io/snapshot</url>
        </pluginRepository>
        <pluginRepository>
            <id>spring-milestones</id>
            <url>https://repo.spring.io/milestone</url>
        </pluginRepository>
    </pluginRepositories>
</project>
```

---
#### 2.2.2 Adding Classpath Dependencies

增加 spring-boot-starter-web

```xml
<dependencies>
    <!-- Boot's Web support -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- Sleuth with Brave tracer implementation -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-sleuth</artifactId>
    </dependency>
</dependencies>
```

---
#### 2.2.3 Writing the Code

src/main/java.Example.java
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.web.bind.annotation.*;

@RestController
@EnableAutoConfiguration
public class Example {

    private static final Logger log = LoggerFactory.getLogger(Backend.class);

    @RequestMapping("/")
    String home() {
        log.info("Hello world!");
        return "Hello World!";
    }

    public static void main(String[] args) {
        SpringApplication.run(Example.class, args);
    }

}
```

#### The @RestController and @RequestMapping Annotations

#### 2.2.4 Running the Example

由于使用了 spring-boot-starter-parent, 所以你有一个 有用的goal：run，你可以用来启动应用。   
Type SPRING_APPLICATION_NAME=backend mvn spring-boot:run 从根目录启动应用。  

你可以访问 localhost:8080. 检查日志可以看到类似：
```
2020-10-21 12:01:16.285  INFO [backend,0b6aaf642574edd3,0b6aaf642574edd3] 289589 --- [nio-9000-exec-1] Example              : Hello world!
```

你可以注意到 日志格式 增加了 ```[backend,0b6aaf642574edd3,0b6aaf642574edd3```。这个实体相当于```[application name, trace id, span id]```   
application name 是从 SPRING_APPLICATION_NAME 环境变量 读取的。

不想在处理 程序中 显示 记录请求，你可以设置 logging.level.org.springframework.web.servlet.DispatcherServlet=DEBUG.

---
#### 2.3 Next Steps
。。没什么用。

---
#### 3 Using Spring Cloud Sleuth

#### 3.1 Span Lifecycle with Spring Cloud Sleuth's API

SC Sleuth Core 在它的 api 模块 包含了 所有 必要的接口 用于 实现tracer。   
该项目带有 OpenZipkin Brave 实现。     
你可以检查 tracer 如何被 桥接到 Sleuth的API 通过 查看 org.s.c.sleuth.brave.bridge。

最常用的接口：
1. org.s.c.sleuth.Tracer: 使用一个tracer，你可以创建一个 根span 捕获 请求的 关键路径(critical path)   
2. org.s.c.sleuth.Span: Span 是一个 work的 独立单元，需要 被开始，被结束。包含 时间信息 和event 和tag。

Span 生命周期 动作：       
start, 当你开始一个span，它的名字被分配，开始时间戳被记录      
end， span结束 (span的结束时间被记录)，且 如果span是 sampled(节选的，采样的)，它有资格 收集 (比如，到Zipkin)      
continue， span 被继续，比如在另一个线程中        
create with explicit parent， 你可以创建一个新的span，设置它的 显式parent。       

SC Sleuth 为你 创建了一个 Tracer 实例，你可以autowire 它。

---
#### 3.1.1 Creating and Ending Spans

你可以手动创建 span 通过 Tracer：

```java
// Start a span. If there was a span present in this thread it will become
// the `newSpan`'s parent.
Span newSpan = this.tracer.nextSpan().name("calculateTax");
try (Tracer.SpanInScope ws = this.tracer.withSpan(newSpan.start())) {
    // ...
    // You can tag a span
    newSpan.tag("taxValue", taxValue);
    // ...
    // You can log an event on a span
    newSpan.event("taxCalculated");
}
finally {
    // Once done remember to end the span. This will allow collecting
    // the span to send it to a distributed tracing system e.g. Zipkin
    newSpan.end();
}
```

在前面的例子中，我们看到 如何创建span的 新实例。如果这个线程 已经有 span了，那么它会成为 新span的 parent。

在创建span后， 始终要 clean

如果你的span 的名字长度大于50char，会被缩短到50个char。   
名字必须 明确 且具体。 大名字会导致 延迟，有时甚至是异常。

---
#### 3.1.2 Continuing Spans

有时，你不想要 创建新span， 只想 continue 某个。   

为了continue span， 你可以保存span 到 线程中，和 传递到另一个线程：

```java
Span spanFromThreadX = this.tracer.nextSpan().name("calculateTax");
try (Tracer.SpanInScope ws = this.tracer.withSpan(spanFromThreadX.start())) {
    executorService.submit(() -> {
        // Pass the span from thread X
        Span continuedSpan = spanFromThreadX;
        // ...
        // You can tag a span
        continuedSpan.tag("taxValue", taxValue);
        // ...
        // You can log an event on a span
        continuedSpan.event("taxCalculated");
    }).get();
}
finally {
    spanFromThreadX.end();
}
```

。。。怎么从其他线程 拿呢。。。ThreadLocal？ 把当前 线程作为参数 传递给 线程池？ 但是这里有 没有 传递进去啊。

---
#### 3.1.3 Creating a Span with an explicit Parent

你可能想要 开始一个 新 span，且 提供一个 显式的 parent。   
假设 parent 在 一个线程 中，你想在 另一根 线程中 启动一个 新span。   
当你 调用 Tracer.nextSpan() 时，它创建一个 ref到 当前scope中span 的 span。   
你可以把 span 放到 scope中，然后 调用 Tracer.nextSpan()： 

```java
// let's assume that we're in a thread Y and we've received
// the `initialSpan` from thread X. `initialSpan` will be the parent
// of the `newSpan`
Span newSpan = null;
try (Tracer.SpanInScope ws = this.tracer.withSpan(initialSpan)) {
    newSpan = this.tracer.nextSpan().name("calculateCommission");
    // ...
    // You can tag a span
    newSpan.tag("commissionValue", commissionValue);
    // ...
    // You can log an event on a span
    newSpan.event("commissionCalculated");
}
finally {
    // Once done remember to end the span. This will allow collecting
    // the span to send it to e.g. Zipkin. The tags and events set on the
    // newSpan will not be present on the parent
    if (newSpan != null) {
        newSpan.end();
    }
}
```

创建这样的span， 你必须完成它。否则 它不会 报告 (比如 到Zipkin)

你也可以 使用 Tracer.nextSpan(Span parentSpan) 来显式 提供 parent span

---
#### Naming Spans

span名字应该 表述 一个操作的名字， 名字不应该包含 唯一值。   

有一些 人造的 span 名字：
1. controller-method-name, 当Controller的 名为controllerMethodName 的方法 接收时
2. async，封装在 Callable，Runnable 接口中的 异步操作。
3. @Scheduled 标注的 方法 返回 类名。

对于 异步，你可以显式 命名。

---
#### @SpanName Annotation

你可以通过 @SpanName 来 命名 span ：

```java
@SpanName("calculateTax")
class TaxCountingRunnable implements Runnable {

    @Override
    public void run() {
        // perform logic
    }

}
```

执行下面的代码， span的名字是 calculateTax：
```java
Runnable runnable = new TraceRunnable(this.tracer, spanNamer, new TaxCountingRunnable());
Future<?> future = executorService.submit(runnable);
// ... some additional logic ...
future.get();
```

---
#### toString() Method

为 Runnable，Callable 创建 单独类 是非常少的，大部分是 创建 匿名类。   
你不能注解到这些 匿名类上。   
> 为了克服 这种限制， 如果没有 @SpanName 注解， 我们会检查 类是否有一个 自定义的 toString() 的实现。   

。。直接 标注到方法上不就可以了。不过toString 或许更简单。

执行下面的代码，会生成一个 名为calculateTax 的span：
```java
Runnable runnable = new TraceRunnable(this.tracer, spanNamer, new Runnable() {
    @Override
    public void run() {
        // perform logic
    }

    @Override
    public String toString() {
        return "calculateTax";
    }
});
Future<?> future = executorService.submit(runnable);
// ... some additional logic ...
future.get();
```

---
#### 3.3 Managing Spans with Annotations

有很多 好的理由 来支持 通过注解 管理 span：
1. API-agnostic means to collaborate with a span. 与API分离
2. Reduced surface area for basic span operations. 减少入侵
3. Collaboration with runtime generated code. 与运行时生成的代码合作

。。。

---
#### 3.3.1 Creating New Spans

如果不想 手工创建 本地 span， 可以使用 @NewSpan 注解。 我们也提供了 @SpanTag 来增加tag

```java
@NewSpan
void testMethod();
```
无参方法 会创建 一个 名字等于 方法名 的span。


```java
@NewSpan("customNameOnTestMethod4")
void testMethod4();
```
可以自定义span 名字 (直接写 或通过 name参数)

```java
// method declaration
@NewSpan(name = "customNameOnTestMethod5")
void testMethod5(@SpanTag("testTag") String param);

// and method execution
this.testBean.testMethod5("test");
```
你可以组合 名字 和 tag。 被注解的方法的 参数的 ***运行时值*** 成为了 tag的 值。 这个例子中， tag key是 testTag, value是 test

```java
@NewSpan(name = "customNameOnTestMethod3")
@Override
public void testMethod3() {
}
```
@NewSpan 可以在 接口 和 类上。 如果 接口的方法 和 类的方法 都有 注解， 那么 最具体的 win。本例中，customNameOnTestMethod3 是名字。

---
#### Continuing Spans

如果你要增加 tag 和 注解 到一个 已存在的 span。你可以使用 @ContinueSpan ：

```java
// method declaration
@ContinueSpan(log = "testMethod11")
void testMethod11(@SpanTag("testTag11") String param);

// method execution
this.testBean.testMethod11("test");
this.testBean.testMethod13();
```

与@NewSpan 相比， 你还可以增加 日志 通过 log参数。

span继续 且：
1. 名字是 testMethod11.before 和 testMethod11.after 的 Log 实体 被创建
2. 如果异常抛出， 名为 testMethod11.afterFailure 的日志实体 被创建
3. 一个 key是 testTag11 ，值是 test 的 tag 被创建。

---
#### Advanced Tag Setting

3种方法 增加tag 到 span。 都通过 @SpanTag 控制， 按照优先级如下：
1. 尝试 TagValueResolver 类型的 bean 和一个提供的名字。
2. 如果bean没有提供，尝试 eval 一个表达式。搜索 TagValueExpressionResolver bean。默认实现 使用SpEL 表达式。  
3. 如果 没有找到 用于eval 的任何表达式，返回 实参的 toString()。


#### Custom Extractor

下面方法的 tag值 被 TagValueResolver 接口 的实现 计算。 它(。。应该是指 接口的实现 )的类名 被作为 resolver 属性的值 传递。

```java
@NewSpan
public void getAnnotationForTagValueResolver(
        @SpanTag(key = "test", resolver = TagValueResolver.class) String test) {
}
```

```java
@Bean(name = "myCustomTagValueResolver")
public TagValueResolver tagValueResolver() {
    return parameter -> "Value from myCustomTagValueResolver";
}
```

生成的tag 值是 Value from myCustomTagValueResolver


#### Resolving Expressions for a Value

```java
@NewSpan
public void getAnnotationForTagValueExpression(
        @SpanTag(key = "test", expression = "'hello' + ' characters'") String test) {
}
```

没有 TagValueExpressionResolver 的 自定义实现 会导致 进行SpEL 的推导。


#### Using The toString() Method

```java
@NewSpan
public void getAnnotationForArgumentToString(@SpanTag("test") Long param) {
}
```

如果用 15 调用上面的方法， 那么会增加一个 值是 "15" 的tag。
。。实参.toString

---
#### 3.4 What to Read Next


---
#### 4 Spring Cloud Sleuth Features

---
#### 4,1 Context Propagation

trace 使用 header 传播 来从服务连接到服务。   
默认格式是 B3 (https://github.com/openzipkin/b3-propagation)   

类似与 数据格式， 你可以配置 header格式，只要 trace 和 span ID 和 B3 兼容。   
这意味这 trace id 和 span id 是 小写16进制，不是UUID。

除了 trace ID，其他的属性(Baggage) 也能随 request 一起传递。

为了使用 提供的 默认， 你可以设置 spring.sleuth.propagation.type 属性。 值可以是 你会传播的tracing 头 的列表。

对于 Brave，我们支持 AWS，B3,W3C propagation 类型。

---
#### 4.2 Sampling

SC Sleuth 把 采样决策(sampling decision) 下降到 tracer 实现中。然而，某些情况下，你可以在运行时 决定 采样决策   

一种情况是 跳过某些客户端 span 的报告。你可以设置 spring.sleuth.web.client.skip-pattern，值是 需要被跳过的 path pattern。   
另一种方法是 提供你 自定义的 org.s.c.sleuth.SamplerFunction<org.s.c.sleuth.http.HttpRequest> 实现， 定义 何时 HttpRequest 会被采样。

---
#### 4.3 Baggage

分布式 tracing 工作 通过 传播字段，特别是 traceId,spanId。   

保存这些字段的上下文 可以 选择性地 推送 其他需要保持一致的字段。

这些额外字段的 名字就是 "Baggage"

Sleuth 允许你 定义 哪个baggage 可以在 trace 上下文中 存在，包括 使用什么 header 名字。

下面 设置baggage 值 通过 SC Sleuth的API：
```java
try (Tracer.SpanInScope ws = this.tracer.withSpan(initialSpan)) {
    BaggageInScope businessProcess = this.tracer.createBaggage(BUSINESS_PROCESS).set("ALM");
    BaggageInScope countryCode = this.tracer.createBaggage(COUNTRY_CODE).set("FO");
    try {
```

目前没有对 baggage 个数，尺寸的 限制。   
太多的baggage 会 降低 系统的性能。    
太多的 baggage 可能因为 超过 传输级 消息 或 header 容量 而 导致应用 崩溃

你可以使用 属性 来定义 没有特殊配置的字段， 例如名字映射：   
1. spring.sleuth.baggage.remote-fields, 从其他服务 接受 且 传播出去 的 header 名字的列表。  
2. spring.sleuth.baggage.local-fields, 本地传播的 名字列表   

这些键不会有前缀，你设置的，就是 被使用的。

这些属性 中设置的名字 会产生 同名的 Baggage。

为了自动 设置 baggage 值 到 Slf4j 的 MDC， 你需要 设置 spring.sleuth.baggage.correlation-fields 属性为 允许的 本地或远程 的key的 列表。   
比如 spring.sleuth.baggage.correlation-fields=country-code 会 设置 country-code baggage的值 到 MDC。

记住，额外的字段 被传播 和 增加到 MDC，从下一个 下游trace 上下文开始。   
为了立刻增加爱 额外字段 到 MDC 在当前 tract 上下文， 配置字段 来刷新：

```java
// configuration
@Bean
BaggageField countryCodeField() {
    return BaggageField.create("country-code");
}

@Bean
ScopeDecorator mdcScopeDecorator() {
    return MDCScopeDecorator.newBuilder()
            .clear()
            .add(SingleCorrelationField.newBuilder(countryCodeField())
                    .flushOnUpdate()
                    .build())
            .build();
}

// service
@Autowired
BaggageField countryCodeField;

countryCodeField.updateValue("new-value");
```

向MDC 添加 实体， 会急剧降低 服务器性能。

如果你想增加 baggage实体 作为tag，使得可以通过 baggage实体 搜索到 span， 你可以 设置 spring.sleuth.baggage.tag-fields 为 允许的baggage key的列表。   
禁用：spring.sleuth.propagation.tag.enabled=false.

---
#### Baggage versus Tags

像 trace ID， Baggage 被附加在 message 或request，通常作为 header。   
Tag 是 Span中发送到Zipkin的 k-v 对。   
Baggage 值 默认情况下 不会 添加到 span。

使得baggage 也是 tag，使用 spring.sleuth.baggage.tag-fields : 
```yaml
spring:
  sleuth:
    baggage:
      foo: bar
      remoteFields:
        - country-code
        - x-vcap-request-id
      tagFields:
        - country-code
```

---
#### 4.4 OpenZipkin Brave Tracer Integration

SC Sleuth 集成了 OpenZipkin Brave tracer 通过 spring-cloud-sleuth-brave 模块中的 桥接。

你可以 直接使用 Sleuth 的API 或 Brave 的API。

---
#### 4.4.1 Brave Basics

你会使用的 最核心的 类型：
1. brave.SpanCustomizer, 修改当前有效的 span
2. brave.Tracer, 获得一个 开始的新的span ad-hoc

一些OpenZipkin Brave 工程的 链接：      
https://github.com/openzipkin/brave/tree/master/brave       
https://github.com/openzipkin/brave/tree/master/brave#baggage   
https://github.com/openzipkin/brave/tree/master/instrumentation/http    

---
#### 4.4.2 Brave Sampling

采样 只在 tracing 后端 应用，比如 Zipkin。  
Trace ID 出现在 日志，无论采样率如何。  

默认的频率是 1秒10个trace，通过 spring.sleuth.sampler.rate 属性控制。   

sampler 也能通过 Java Config 设置：
```java
@Bean
public Sampler defaultSampler() {
    return Sampler.ALWAYS_SAMPLE;
}
```

你可以设置 http 头 b3 为 1, 或 在消息中 设置 spanFlags 头 为1。 这样会 强制当前 请求 被 取样。

默认下，取样器 会和 刷新scope 机制 一起工作。那意味着 你可以 运行时 修改 取样属性。   

然而，有时 围绕 sampler 创建代理 且过早地调用它(从 @PostConstruct ) 会导致 死锁。 这种情况下，要么 显式创建一个 sampler bean，要么设置 spring.sleuth.sampler.refresh.enabled 为false  来禁用 refresh scope support

---
#### 4.4.3 Brave Baggage Java configuration

如果你要做比上面更高级的事情，不要定义属性，而是为你 使用的 baggage 字段 使用 @Bean 配置。   
1. BaggagePropagationCustomizer 设置 baggage 字段
2. 增加 SingleBaggageField 来控制 Baggage的 header 名字。
3. CorrelationScopeCustomizer 设置 MDC 字段。
4. 增加 SingleCorrelationField 来改变 Baggage的 MDC 名字，或 设置更新是否刷新。

---
#### 4.4.4 Brave Customizations
brave.Tracer 对象 被 sleuth 完全管理，所以你 很少需要修改它。   
那就是说，Sleuth 支持 几个 Customizer 类型，允许你 配置 任何 Sleuth 没有通过 自动配置 或属性 完成的 事情。  

如果你定义下面的 作为 bean， Sleuth 会调用它 来 定制行为：
1. RpcTracingCustomizer, for RPC tagging 和 sampling 策略
2. HttpTracingCustomizer，HTTP tagging sampling
3. MessagingTracingCustomizer，messaging tagging sampling
4. CurrentTraceContextCustomizer， 集成decorator 比如 correlation
5. BaggagePropagationCustomizer，扩散 baggage 字段 in process and over headers
6. CorrelationScopeDecoratorCustomizer，scope decoration 比如 MDC 字段关联。


---
#### Brave Sampling Customizations

如果需要 客户端/服务器 取样，只需要注册一个 brave.sampler.SamplerFunction<HttpRequest> 类的bean，且客户端的bean的名字是 sleuthHttpClientSampler, 服务器的bean名字是 sleuthHttpServerSampler   

更方便的是 @HttpClientSampler ，@HttpServerSampler 注解，用来注入 bean 或通过 它们的 NAME 字段 来ref bean的名字。

如果你想完全重写 HttpTracing bean， 你可以 使用 SkipPatternProvider 接口 来 检索 URL Pattern for 不需要被抽样的span。   
下面是 服务器 上 SkipPatternProvider：
```java
@Configuration(proxyBeanMethods = false) 
class Config {
  @Bean(name = ***HttpServerSampler***.NAME)
  SamplerFunction<HttpRequest> myHttpSampler(SkipPatternProvider provider) {
      Pattern pattern = provider.skipPattern();
      return request -> {
          String url = request.path();
          boolean shouldSkip = pattern.matcher(url).matches();
          if (shouldSkip) {
              return false;
          }
          return null;
      };
  }
}
```

---
#### 4.4.5 Brave Messaging

Sleuth 自动配置 MessagingTracing bean，担任 消息传递工具(Kafka,JMS) 的基础

如果需要 定制 消息trace 的 生产者/消费者 采样， 只需要注册 brave.sampler.SamplerFunction<MessagingRequest> 类型的 bean，且 生产者采样器 名字是 sleuthProducerSampler, 消费者采样器的名字是 sleuthConsumerSampler。

更简单的是 @ProducerSampler, @ConsumerSampler。

下面是 每秒 取样 100 个消费者请求，除了 alerts 管道。其他请求会使用 Tracing组建的 全局速度。
```java
@Configuration(proxyBeanMethods = false)
class Config {
  @Bean(name = *-*ConsumerSampler*-*.NAME)
  SamplerFunction<MessagingRequest> myMessagingSampler() {
      return MessagingRuleSampler.newBuilder().putRule(channelNameEquals("alerts"), Sampler.NEVER_SAMPLE)
              .putRule(Matchers.alwaysMatch(), RateLimitingSampler.create(100)).build();
  }
}
```
。。ConsumerSampler ProducerSampler HttpClientSampler HttpServerSampler 怎么用的。。。。

---
#### 4.4.6 Brave Opentracing

你可以合并 Brave 和 OpenTracing，通过 io.opentracing.brave:brave-opentracing   
添加到 classpath 后， OpenTracing Tracer 会被自动 配置。

---
#### 4.5 Sending Spans to Zipkin

SC Sleuth 提供了 各种各样的 和 OpenZipkin 分布式 tracing系统 的集成。   
无论选择哪种 跟踪器 实现，只需要将 spring-cloud-sleuth-zipkin 添加到 classpath 来 开始发送span 到Zipkin 就可以了。
你可以选择 是通过HTTP 还是 messaging 

当span 关闭，它被发送到 Zipkin 通过 HTTP。这个通信是 异步的。   
你可以配置 URL 通过 spring.zipkin.baseUrl 属性：
```properties
spring.zipkin.baseUrl: https://192.168.99.100:9411/
```

如果你想 通过 服务发现 来找到 Zipkin， 你可以传递 Zipkin 的服务ID：
```properties
spring.zipkin.baseUrl: https://zipkinserver/
```

禁用： spring.zipkin.discovery-client-enabled=false

当 Discovery Client 功能 被启用，Sleuth 使用 LoadBalancerClient 来寻找 Zipkin Server的 URL。   
这意味者你可以 配置 LB。

如果 classpath中 同时有 web,rabbit,activemq,kafka， 你可能需要 选择发送span到 zipkin 的方式。   
通过设置 web,rabbit,activemq,kafka 到 spring.zipkin.sender.type 属性，下面是 设置 sender 类型为web：
```properties
spring.zipkin.sender.type: web
```

为了自定义 通过HTTP 发送 span 到 zipkin 的 RestTemplate，你可以注册一个 ZipkinRestTemplateCustomizer bean。

```java
@Configuration(proxyBeanMethods = false)
class MyConfig {
    @Bean ZipkinRestTemplateCustomizer myCustomizer() {
        return new ZipkinRestTemplateCustomizer() {
            @Override
            void customize(RestTemplate restTemplate) {
                // customize the RestTemplate
            }
        };
    }
}
```

如果你想控制 创建RestTemplate 的全过程，你可以 创建一个 zipkin2.reporter.Sender 类型：

```java
@Bean Sender myRestTemplateSender(ZipkinProperties zipkin,
        ZipkinRestTemplateCustomizer zipkinRestTemplateCustomizer) {
    RestTemplate restTemplate = mySuperCustomRestTemplate();
    zipkinRestTemplateCustomizer.customize(restTemplate);
    return myCustomSender(zipkin, restTemplate);
}
```

---
#### 4.5.1 Custom service name

默认下，Sleuth 假定：当你发送span 到zipkin，你希望 span的 服务名 等于 spring.application.name 属性的值。   
你可以明确指定 一个 service name 为这个应用发出的 所有的 span：
```properties
spring.zipkin.service.name: myService
```

---
#### 4.5.2 Host Locator

这节是关于 从 服务发现 中 发现host， 不是关于 通过 服务发现 发现 zipkin。  

为了发现 对应于某个span 的 host，我们需要 解析 host name 和 port。   
默认做法是 从 服务属性 里获得这些值。如果没有设置，我们尝试 从网络接口 检索 host name。

如果你有 服务发现客户端，并且 倾向于 从 服务注册中心 的已注册的 实例 中 找到 host 地址，你需要设置 spring.zipkin.locator.discovery.enabled 属性(它对 基于http 和 基于Stream 的 span 报告 都适用。)：

```properties
spring.zipkin.locator.discovery.enabled: true
```

---
#### 4.5.3 Customization of Reported Spans

Sleuth中， 我们生成 span with 固定的名字。 有些用户想要 根据tag 的值 来修改 名字。

Sleuth 注册一个 SpanFilter bean, 它能自动 跳过 给定名字 pattern 的 span的发送报告。   
spring.sleuth.span-filter.span-name-patterns-to-skip 包含 默认 跳过的 span 名字的pattern   
spring.sleuth.span-filter.additional-span-name-patterns-to-skip 会 追加 提供的 span 名字pattern 到 已存在的集合。   

禁用：spring.sleuth.span-filter.enabled=false


#### Brave Customization of Reported Spans

这节 只对 Brave tracer 有效。

在报告 span (比如，到zipkin) 之前，你可能希望 修改span。   
你可以通过 实现一个 SpanHandler 来完成。  

下面是，注册了2个 实现SpanHandler 的bean：

```java
@Bean
SpanHandler handlerOne() {
    return new SpanHandler() {
        @Override
        public boolean end(TraceContext traceContext, MutableSpan span, Cause cause) {
            span.name("foo");
            return true; // keep this span
        }
    };
}

@Bean
SpanHandler handlerTwo() {
    return new SpanHandler() {
        @Override
        public boolean end(TraceContext traceContext, MutableSpan span, Cause cause) {
            span.name(span.name() + " bar");
            return true; // keep this span
        }
    };
}
```

上面的例子会导致 修改 被报告的span 的名字 为 foo bar .

。。。怎么确定 哪个先？ 靠 代码先后？ 如果不同类呢？


---
#### 4.5.4 Overriding the auto-configuration of Zipkin

SC Sleuth 支持发送 trace 到多个 tracing 系统 从2.1.0开始。   
每个tracing 系统 需要一个 Reporter<Span> 和 Sender。   
如果你想 覆盖 提供的bean，你需要 给它们一个 明确的名字。   
你可以 使用 ZipkinAutoConfiguration.REPORTER_BEAN_NAME 和 ZipkinAutoConfiguration.SENDER_BEAN_NAME   

```java
@Configuration(proxyBeanMethods = false)
protected static class MyConfig {

    @Bean(ZipkinAutoConfiguration.REPORTER_BEAN_NAME)
    Reporter<zipkin2.Span> myReporter(@Qualifier(ZipkinAutoConfiguration.SENDER_BEAN_NAME) MySender mySender) {
        return AsyncReporter.create(mySender);
    }

    @Bean(ZipkinAutoConfiguration.SENDER_BEAN_NAME)
    MySender mySender() {
        return new MySender();
    }

    static class MySender extends Sender {

        private boolean spanSent = false;

        boolean isSpanSent() {
            return this.spanSent;
        }

        @Override
        public Encoding encoding() {
            return Encoding.JSON;
        }

        @Override
        public int messageMaxBytes() {
            return Integer.MAX_VALUE;
        }

        @Override
        public int messageSizeInBytes(List<byte[]> encodedSpans) {
            return encoding().listSizeInBytes(encodedSpans);
        }

        @Override
        public Call<Void> sendSpans(List<byte[]> encodedSpans) {
            this.spanSent = true;
            return Call.create(null);
        }

    }

}
```

---
#### 4.6 Log Integration

Sleuth 通过 服务名(spring.zipkin.service.name 或 spring.application.name)，span id (spanId),trace id(traceId) 等变量 来 配置 日志上下文   
这些帮助你将 日志与 分布式跟踪 联系起来，并允许你 选择 使用哪些工具 对服务 进行 故障排除。

一旦你发现 error 级别的日志，你可以 看到 trace id，复制到你的 分布式 跟踪系统 来 可视化整个 链路，不必关心 请求经过了多少服务。

```
backend.log:  2020-04-09 17:45:40.516 ERROR [backend,5e8eeec48b08e26882aba313eb08f0a4,dcc1df555b5777b3] 97203 --- [nio-9000-exec-1] o.s.c.s.i.web.ExceptionLoggingFilter     : Uncaught exception thrown
frontend.log:2020-04-09 17:45:40.574 ERROR [frontend,5e8eeec48b08e26882aba313eb08f0a4,82aba313eb08f0a4] 97192 --- [nio-8081-exec-2] o.s.c.s.i.web.ExceptionLoggingFilter     : Uncaught exception thrown
```

日志配置 是 Sleuth 自动配置的，禁用通过 spring.sleuth.enabled=false (估计少一个log吧。) 或设置 你自己的 logging.pattern.level 属性   

如果你使用一个 日志 聚合(aggregate)工具(如，Kibana，Splunk), 你可以 把事件 按发生时间 排序。   

如果你想使用 Logstash，下面是 Logstash 的 Grok pattern：
```
filter {
  # pattern matching logback pattern
  grok {
    match => { "message" => "%{TIMESTAMP_ISO8601:timestamp}\s+%{LOGLEVEL:severity}\s+\[%{DATA:service},%{DATA:trace},%{DATA:span}\]\s+%{DATA:pid}\s+---\s+\[%{DATA:thread}\]\s+%{DATA:class}\s+:\s+%{GREEDYDATA:rest}" }
  }
  date {
    match => ["timestamp", "ISO8601"]
  }
  mutate {
    remove_field => ["timestamp"]
  }
}
```

如果你想 让 Grok 和 Cloud Foundry 的日志一起使用，你必须使用下面 pattern：
```
filter {
  # pattern matching logback pattern
  grok {
    match => { "message" => "(?m)OUT\s+%{TIMESTAMP_ISO8601:timestamp}\s+%{LOGLEVEL:severity}\s+\[%{DATA:service},%{DATA:trace},%{DATA:span}\]\s+%{DATA:pid}\s+---\s+\[%{DATA:thread}\]\s+%{DATA:class}\s+:\s+%{GREEDYDATA:rest}" }
  }
  date {
    match => ["timestamp", "ISO8601"]
  }
  mutate {
    remove_field => ["timestamp"]
  }
}
```

---
#### 4.6.1 JSON Logback with Logstash

经常的，你不想 保存你的日志 到 文本文件，而是想保存到 JSON文件，Logstash 可以立刻挑选 JSON文件。   
为了这么做，你需要：

> Dependencies Setup    

1. 确保 Logback 在 classpath中 ( ch.qos.logback:logback-core)
2. 增加 Logstash Logback encode。例如 使用4.6版本： net.logstash.logback:logstash-logback-encoder:4.6


> Logback Setup    

考虑下面的 Logback 配置文件 (logback-spring.xml):
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
    <springProperty scope="context" name="springAppName" source="spring.application.name"/>
    <!-- Example for logging into the build folder of your project -->
    <property name="LOG_FILE" value="${BUILD_FOLDER:-build}/${springAppName}"/>

    <!-- You can override this to have a custom pattern -->
    <property name="CONSOLE_LOG_PATTERN"
              value="%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}"/>

    <!-- Appender to log to console -->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <!-- Minimum logging level to be presented in the console logs-->
            <level>DEBUG</level>
        </filter>
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
            <charset>utf8</charset>
        </encoder>
    </appender>

    <!-- Appender to log to file -->
    <appender name="flatfile" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_FILE}</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_FILE}.%d{yyyy-MM-dd}.gz</fileNamePattern>
            <maxHistory>7</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
            <charset>utf8</charset>
        </encoder>
    </appender>
    <!-- Appender to log to file in a JSON format -->
    <appender name="logstash" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_FILE}.json</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_FILE}.json.%d{yyyy-MM-dd}.gz</fileNamePattern>
            <maxHistory>7</maxHistory>
        </rollingPolicy>
        <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
            <providers>
                <timestamp>
                    <timeZone>UTC</timeZone>
                </timestamp>
                <pattern>
                    <pattern>
                        {
                        "timestamp": "@timestamp",
                        "severity": "%level",
                        "service": "${springAppName:-}",
                        "trace": "%X{traceId:-}",
                        "span": "%X{spanId:-}",
                        "pid": "${PID:-}",
                        "thread": "%thread",
                        "class": "%logger{40}",
                        "rest": "%message"
                        }
                    </pattern>
                </pattern>
            </providers>
        </encoder>
    </appender>
    <root level="INFO">
        <appender-ref ref="console"/>
        <!-- uncomment this to have also JSON logs -->
        <!--<appender-ref ref="logstash"/>-->
        <!--<appender-ref ref="flatfile"/>-->
    </root>
</configuration>
```

Logback 配置文件：
1. 从应用来的 日志信息 以 JSON格式 保存在 build/${spring.application.name}.json 文件
2. 已注释掉 另外2个额外 输出源： 控制台 和 标准日志文件
3. 和上一节 介绍的日志记录模式 相同。

如果你使用 自定义 logback-spring.xml, 你必须传递 spring.application.name 到 bootstrap 而不是 application 属性文件。

---
#### 4.7 What to Read Next


---
#### 5 "How-to" Guides

这节提供了 对于 一些 "how do i do that...?" 问题的 回答。

---
#### 5.1 How to Set up Sleuth with Brave?

增加 Sleuth starter 到 classpath

```xml
<dependencyManagement>
      <dependencies>
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-dependencies</artifactId>
              <version>${release.train-version}</version>
              <type>pom</type>
              <scope>import</scope>
          </dependency>
      </dependencies>
</dependencyManagement>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
```

---
#### 5.2 How to Set Up Sleuth with Brave & Zipkin via HTTP?

增加 Sleuth starter 和 Zipkin 到classpath
```xml
<dependencyManagement>
      <dependencies>
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-dependencies</artifactId>
              <version>${release.train-version}</version>
              <type>pom</type>
              <scope>import</scope>
          </dependency>
      </dependencies>
</dependencyManagement>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-sleuth-zipkin</artifactId>
</dependency>
```

---
#### How to Set Up Sleuth with Braze & Zipkin via Messaging?

如果要使用RabbitMQ，Kafka，ActiveMQ 代替HTTP，增加 spring-rabbit, spring-kafka 或 org.apache.activemq:activemq-client 依赖。   
默认 目的地名字是 Zipkin。

如果使用Kafka，增加依赖
```xml
<dependencyManagement>
      <dependencies>
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-dependencies</artifactId>
              <version>${release.train-version}</version>
              <type>pom</type>
              <scope>import</scope>
          </dependency>
      </dependencies>
</dependencyManagement>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-sleuth-zipkin</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

设置 属性 ：   
> spring.zipkin.sender.type: kafka


如果你想用RabbitMQ，增加，spring-cloud-starter-sleuth,spring-cloud-sleuth-zipkin,spring-rabbit
```xml
<dependencyManagement>
      <dependencies>
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-dependencies</artifactId>
              <version>${release.train-version}</version>
              <type>pom</type>
              <scope>import</scope>
          </dependency>
      </dependencies>
</dependencyManagement>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-sleuth-zipkin</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-rabbit</artifactId>
</dependency>
```


如果使用 ActiveMQ，增加 spring-cloud-starter-sleuth, spring-cloud-sleuth-zipkin, activemq-client   

```xml
<dependencyManagement>
      <dependencies>
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-dependencies</artifactId>
              <version>${release.train-version}</version>
              <type>pom</type>
              <scope>import</scope>
          </dependency>
      </dependencies>
</dependencyManagement>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-sleuth-zipkin</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-client</artifactId>
</dependency>
```
配置属性：
> spring.zipkin.sender.type: activemq


---
#### 5.4 how to see spans in an external system?

如果你无法 看到 span 被报告给 外部系统(如 zipkin)，可能是下面的原因：
1. 你的span 没有被 取样
2. 你忘记增加 依赖 来 报告到外部系统 (如 spring-cloud-sleuth-zipkin)
3. 你没有配置 到外部系统的 连接信息

---
#### 5.4.1 your span is not being sampled
为了检查 span 是否被 抽样，看看是否设置了 可导出标志 就足够了， 看下面的例子：
```
2020-10-21 12:01:16.285  INFO [backend,0b6aaf642574edd3,0b6aaf642574edd3,true] 289589 --- [nio-9000-exec-1] Example              : Hello world!
```

```[backend,0b6aaf642574edd3,0b6aaf642574edd3,true]``` 是true 就说明 这个span 被抽样，应该被报告。

---
#### 5.4.2 missing dependency

3.0.0 之前 spring-cloud-starter-zipkin 包含了 spring-cloud-starter-sleuth 和 spring-cloud-sleuth-zipkin 依赖。   
3.0.0 开始，spring-cloud-starter-zipkin 被移除。

---
#### 5.4.3 Connection Misconfiguration

再次确认 远程系统的地址 是否正确 (spring.zipkin.baseUrl), 如果通过 broker连接，确保你 broker的连接 被正确配置。

---
#### 5.5 How to make RestTemplate, WebClient, etc. Work？

如果你发现 tracing 上下文没有被 传播出去，可能是下面的原因：
1. 没有使用 给定的 库
2. 使用了库，但是 配置错误/没有配置


正确：
```java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration(proxyBeanMethods = false)
class MyConfiguration {
    @Bean RestTemplate myRestTemplate() {
        return new RestTemplate();
    }
}

@Service
class MyService {
    private final RestTemplate restTemplate;

    MyService(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    String makeACall() {
        return this.restTemplate.getForObject("http://example.com", String.class);
    }

}
```


错误，new 的没有 用：
```java
@Service
class MyService {

    String makeACall() {
        // This will not work because RestTemplate is not a bean
        return new RestTemplate().getForObject("http://example.com", String.class);
    }

}
```

---
#### 5.6 How to add headers to the http server response?

注册 HttpResponseParser 类型的bean， 名字是 HttpServerResponseParse.NAME

```java
import org.springframework.cloud.sleuth.http.HttpResponseParser;
import org.springframework.cloud.sleuth.instrument.web.HttpServerResponseParser;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration(proxyBeanMethods = false)
class MyConfig {

    @Bean(name = HttpServerResponseParser.NAME)
    HttpResponseParser myHttpResponseParser() {
        return (response, context, span) -> {
            Object unwrap = response.unwrap();
            if (unwrap instanceof HttpServletResponse) {
                HttpServletResponse resp = (HttpServletResponse) unwrap;
                resp.addHeader("MyCustom", "Header");
            }
        };
    }

}
```

你的span 需要 被 抽样 for parser工作。这意味这 你需要 导出 span 到 (比如)zipkin

---
#### how to customize http client spans?

注册 HttpRequestParser 类型的 bean，名字是 HttpClientRequestParser.NAME , 来增加 自定义 for 请求侧。   
注册 HttpResponseParser 的 名字是 HttpClientRequestParser.NAME 的 bean 来增加自定义 for 响应端。

```java
@Configuration(proxyBeanMethods = false)
public static class ClientParserConfiguration {

    // example for Feign
    @Bean(name = HttpClientRequestParser.NAME)
    HttpRequestParser myHttpClientRequestParser() {
        return (request, context, span) -> {
            // Span customization
            span.name(request.method());
            span.tag("ClientRequest", "Tag");
            Object unwrap = request.unwrap();
            if (unwrap instanceof feign.Request) {
                feign.Request req = (feign.Request) unwrap;
                // Span customization
                span.tag("ClientRequestFeign", req.httpMethod().name());
            }
        };
    }

    // example for Feign
    @Bean(name = HttpClientResponseParser.NAME)
    HttpResponseParser myHttpClientResponseParser() {
        return (response, context, span) -> {
            // Span customization
            span.tag("ClientResponse", "Tag");
            Object unwrap = response.unwrap();
            if (unwrap instanceof feign.Response) {
                feign.Response resp = (feign.Response) unwrap;
                // Span customization
                span.tag("ClientResponseFeign", String.valueOf(resp.status()));
            }
        };
    }

}
```

---
#### 5.8 how to customize http server spans?

也是 请求端 和 响应段，不同类型，并且 名字确定

```java
@Configuration(proxyBeanMethods = false)
public static class ServerParserConfiguration {

    @Bean(name = HttpServerRequestParser.NAME)
    HttpRequestParser myHttpRequestParser() {
        return (request, context, span) -> {
            // Span customization
            span.tag("ServerRequest", "Tag");
            Object unwrap = request.unwrap();
            if (unwrap instanceof HttpServletRequest) {
                HttpServletRequest req = (HttpServletRequest) unwrap;
                // Span customization
                span.tag("ServerRequestServlet", req.getMethod());
            }
        };
    }

    @Bean(name = HttpServerResponseParser.NAME)
    HttpResponseParser myHttpResponseParser() {
        return (response, context, span) -> {
            // Span customization
            span.tag("ServerResponse", "Tag");
            Object unwrap = response.unwrap();
            if (unwrap instanceof HttpServletResponse) {
                HttpServletResponse resp = (HttpServletResponse) unwrap;
                // Span customization
                span.tag("ServerResponseServlet", String.valueOf(resp.getStatus()));
            }
        };
    }

}
```

你的span 需要被 抽样 for parser 工作， 这意味着 你需要 导出 span 到 (比如)zipkin

---
#### how to see the application name in logs?

如果你没有修改 默认日志格式，那么把 spring.application.name 放到 bootstrap.yml， 而不是 application.yml。

使用新的 SC configuration bootstrap 很快就不再需要了， 因为 将不会有 Bootstrap Context

---
#### 5.10 how to change the context propagation mechanism?

为了使用提供的默认，你可以设置 spring.sleuth.propagation.type 属性， 值可以是 一个list，你会传播更多 tracing header。

对于 Brave，我们支持 AWS，B3,W3C 传播类型。

如果你提供了 自定义传播机制 ，设置 spring.sleuth.propagation.type 属性为 CUSTOM， 实现你自己的 bean (对于Brave是 Propagation.Factory)：

```java
@Component
class CustomPropagator extends Propagation.Factory implements Propagation<String> {

    @Override
    public List<String> keys() {
        return Arrays.asList("myCustomTraceId", "myCustomSpanId");
    }

    @Override
    public <R> TraceContext.Injector<R> injector(Setter<R, String> setter) {
        return (traceContext, request) -> {
            setter.put(request, "myCustomTraceId", traceContext.traceIdString());
            setter.put(request, "myCustomSpanId", traceContext.spanIdString());
        };
    }

    @Override
    public <R> TraceContext.Extractor<R> extractor(Getter<R, String> getter) {
        return request -> TraceContextOrSamplingFlags.create(TraceContext.newBuilder()
                .traceId(HexCodec.lowerHexToUnsignedLong(getter.get(request, "myCustomTraceId")))
                .spanId(HexCodec.lowerHexToUnsignedLong(getter.get(request, "myCustomSpanId"))).build());
    }

    @Override
    public <K> Propagation<K> create(KeyFactory<K> keyFactory) {
        return StringPropagationAdapter.create(this, keyFactory);
    }

}
```

---
#### 5.11 how to implement my own tracer?

Spring Cloud Sleuth API 包含 实现tracer 所有需要的 接口。这个工程带了 OpenZipkin Brave 实现。  
你可以 查看 2个tracer 是如何桥接到 Sleuth 的API 的，在org.s.c.sleuth.brave.bridge 模块。   



---
#### 6 Spring Cloud Sleuth customization

---
#### 6.1 Asynchronous Communication

---
#### 6.1.1 @Async annotated methods

对于所有 tracer 实现可用。  

在SC Sleuth 中，我们检测 与异步相关的组件，以便在线程之间传递跟踪信息。   
禁用这种行为： spring.sleuth.async.enabled=false

如果你 注解 你的方法 用@Async， 我们会自动 修改 存在的span：
1. 如果方法 被标注了 @SpanName, 注解的值 是 span的名字
2. 如果 方法 没有 @SpanName, span名字是 方法名
3. span 被 tag： 方法的类名 和 方法名。

由于我们不修改 现有的span，如果你想 维持它 最初的名字(比如，一个由于收到HTTP request 而创建的 span)，你应该 在你的 @Async 标注的 方法 被包装在 @NewSpan 中， 或者手动创建一个 新span。 

---
#### 6.1.2 @Scheduled annotated methods

这个功能对 所有 tracer 实现 可用。

在SC Sleuth中， 我们检测 定时方法，以便在线程间传递 跟踪信息。   
通过 spring.sleuth.scheduled.enabled=false 来禁用。

如果 用@Scheduled 注解 方法， 我们会自动创建一个 新span ：
1. span名字 是 方法名
2. span 有 tag： 方法的类名 和 方法名。

如果你想 跳过 某些@Scheduled 注解的类的 span的创建，你可以设置 spring.sleuth.scheduled.skipPattern, 值是正则，匹配 @Scheduled 注解的 类的 全名。


---
#### 6.1.3 Executor，ExecutorService, and ScheduledExecutorService

对所有 tracer 实现可用。

我们提供 LazyTraceExecutor,TraceableExecutorService,TraceableScheduledExecutorService. 这些实现 创建 span 当 新的task 被 submit，invoke,或 schedule.  

下面 是如何通过 TraceableExecutorService 来传递 跟踪信息， 当使用CompletableFuture 时：

```java
CompletableFuture<Long> completableFuture = CompletableFuture.supplyAsync(() -> {
    // perform some logic
    return 1_000_000L;
}, new TraceableExecutorService(beanFactory, executorService,
        // 'calculateTax' explicitly names the span - this param is optional
        "calculateTax"));
```

Sleuth 不能 对 parallelStream() 开箱即用。 如果你想要 跟踪信息 通过stream 传播，你 必须使用 supplyAsync() ，就像上面。

如果 有bean 实现了Executor 接口，你想要排除出 span creation， 你可以使用 spring.sleuth.async.ignored-beans 属性， 值是 bean的名字的list。

你可以禁用这种行为 通过 spring.sleuth.async.enabled=false


#### Customization of Executors

有时，你需要 设置 AsyncExecutor 的自定义 实例。 下面是 配置 自定义 Executor：
```java
@Configuration(proxyBeanMethods = false)
@EnableAutoConfiguration
@EnableAsync
// add the infrastructure role to ensure that the bean gets auto-proxied
@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
public static class CustomExecutorConfig extends AsyncConfigurerSupport {

    @Autowired
    BeanFactory beanFactory;

    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        // CUSTOMIZE HERE
        executor.setCorePoolSize(7);
        executor.setMaxPoolSize(42);
        executor.setQueueCapacity(11);
        executor.setThreadNamePrefix("MyExecutor-");
        // DON'T FORGET TO INITIALIZE
        executor.initialize();
        return new LazyTraceExecutor(this.beanFactory, executor);
    }

}
```
为了确保 你的 配置能 稍后被处理，记住 添加 @Role(BeanDefinition.ROLE_INFRASTRUCTURE) 在你的 @Configuration 类 


---
#### 6.2 HTTP Client Integration

这节的功能能通过 spring.sleuth.web.client.enabled=false 来禁用。

---
#### 6.2.1 Synchronous Rest Template

对所有 tracer 实现可用。

我们注入一个 RestTemplate 拦截器 来确保 所有的 tracing 信息 被传递到 请求。   

每次 当一个调用 被生成，一个新 span 生成。它在 收到响应后关闭。   
阻止 同步的 RestTemplate 功能，设置 spring.sleuth.web.client.enabled=false

你必须注册一个 RestTemplate 作为bean，这样 拦截器 会被注入。 new 的RestTemplate 无效。

---
6.2.2 Asynchronous Rest Template

对所有 tracer 实现 可用。

从Sleuth 2.0.0 开始，我们不再 注册 AsyncRestTemplate 类型的 bean。 你负责创建这样的bean。

阻止 AsyncRestTemplate 功能，设置 spring.sleuth.web.async.client.enabled=false。   
为了禁止 创建默认的 TraceAsyncClientHttpRequestFactoryWrapper, 设置 spring.sleuth.web.async.client.factory.enabled=false   
如果你 不想创建 AsyncRestClient， 设置 spring.sleuth.web.async.client.template.enabled=false   

#### Multiple Asynchronous Rest Templates
有时，你需要使用 多个 Asynchronous Rest Template 实现，下面是例子：如何设置一个 自定义 AsyncRestTemplate.   

```java
@Configuration(proxyBeanMethods = false)
public static class TestConfig {

    @Bean(name = "customAsyncRestTemplate")
    public AsyncRestTemplate traceAsyncRestTemplate() {
        return new AsyncRestTemplate(asyncClientFactory(), clientHttpRequestFactory());
    }

    private ClientHttpRequestFactory clientHttpRequestFactory() {
        ClientHttpRequestFactory clientHttpRequestFactory = new CustomClientHttpRequestFactory();
        // CUSTOMIZE HERE
        return clientHttpRequestFactory;
    }

    private AsyncClientHttpRequestFactory asyncClientFactory() {
        AsyncClientHttpRequestFactory factory = new CustomAsyncClientHttpRequestFactory();
        // CUSTOMIZE HERE
        return factory;
    }

}
```

#### WebClient
对所有 tracer 有用

我们注入一个 ExchangeFilterFunction 实现，来创建一个 span，通过 on-success 和 on-error 回调，注意关闭 客户端侧 span。   

阻止这个功能， spring.sleuth.web.client.enabled=false

你必须注册一个 WebClient bean，这样 跟踪 才会被应用，如果 new 创建 WebClient，不会 有用。

#### Traverson

对所有 tracer 实现可用。

如果你使用 Traverson 库，你可以注入一个 RestTemplate bean 到你的 Traverson 对象。   
由于 RestTemplate 已经 被 拦截了，在你的客户端里 你获得了跟踪的 全部支持。   

```java
@Autowired RestTemplate restTemplate;

Traverson traverson = new Traverson(URI.create("https://some/address"),
    MediaType.APPLICATION_JSON, MediaType.APPLICATION_JSON_UTF8).setRestOperations(restTemplate);
// use Traverson
```

#### Apache HttpClientBuilder and HttpAsyncClientBuilder

对所有 Brave tracer 有用

我们 创建 HttpClientBuilder 和 HttpAsyncClientBuilder 这样 tracing 上下文 被注入到 发送的request。   

阻止这个功能，spring.sleuth.web.client.enabled=false


#### Netty HttpClient

对所有 tracer 实现 有用

我们使用 Netty的 HttpClient

阻止，spring.sleuth.web.client.enabled=false

你必须注册 HttpClient 作为bean。  new的 没有用。


#### UserInfoRestTemplateCustomizer

对所有tracer 有用

使用 Spring Security 的 UserInfoRestTemplateCustomizer。

阻止， spring.sleuth.web.client.enabled=false

---
#### 6.3 HTTP Server Integration

这节的功能 能通过 spring.sleuth.web.enabled=false 来禁用。

---
#### 6.3.1 HTTP Filter

对所有 tracer 有用。

通过TracingFilter， 所有 被抽样的 入站 请求 会创建 span。你可以配置 哪些URI 被跳过，通过 spring.sleuth.web.skipPattern 属性。   
如果你有 ManagementServerProperties 在 classpath，它的 contextPath 的值 会被添加到 提供的 skip pattern。   
如果你想重用 Sleuth 的 默认的 skip pattern，然后 只加上你自己的，那么可以把 pattern放到 spring.sleuth.web.additionalSkipPattern.   

默认下， 所有 SBoot Actuator 端点 都自动 被加入到 skip pattern。 启用：spring.sleuth.web.ignore-auto-configured-skip-patterns=true

修改 tracing filter 注册顺序，设置 spring.sleuth.web.filter-order 属性。

禁用 用于 log 未捕获异常 的filter ： spring.sleuth.web.exception-throwing-filter-enabled=false 。   

---
#### 6.3.2 HandlerInterceptor

对所有 tracer 可用。

由于 我们希望 span 的名字 是精确的， 我们使用 一个 要么封装了 已存在的 HandlerInterceptor 或 直接被添加到已存在的 HandlerInterceptors 的 TraceHandlerInterceptor，   

TraceHandlerInterceptor 增加了一个 特殊的 请求属性 到 给定的 HttpServletRequest。   

如果 TracingFilter 没有看到这个属性，它创建一个 "fallback" span，这个是一个 额外的span 创建在服务器端，以便在UI中正确显示tracing。如果发生这汇总情况，可能是因为缺少 instrumentation。
这种情况下，请在SC Sleuth中提出问题。

---
#### Async Servlet Support

对所有tracer 可用。

通过 TraceWebFilter，所有 被抽样的 入站 请求 创建span。span的名字是 http: + request的路径。
例如如果request 被发送到 /this/that, span的名字是 http:/this/that.     
你可以配置 哪些URI 你想跳过，通过 spring.sleuth.web.skipPattern。   
如果你有 ManagementServerProperties 在 classpath，它的 contextPath 的值 会自动加入到 提供的 skip pattern。   
如果你 想 复用 Sleuth 的默认 skip pattern 和你自己的，by 放到 spring.sleuth.web.additionalSkipPattern。

为了获得 性能和 上下文传播 的 最佳结果，我们建议 spring.sleuth.reactor.instrumentation-type 为 MANUAL。   
为了执行 代码 with span in scope 你可以 调用 WebFluxSleuthOperators.withSpanInScope :

```java
@GetMapping("/simpleManual")
public Mono<String> simpleManual() {
    return Mono.just("hello").map(String::toUpperCase).doOnEach(WebFluxSleuthOperators
            .withSpanInScope(SignalType.ON_NEXT, signal -> log.info("Hello from simple [{}]", signal.get())));
}
```

修改 tracing filter 注册 顺序，设置 spring.sleuth.web.filter-order 属性。

---
#### 6.4 Messaging

本节的功能可以通过 spring.sleuth.messaging.enabled=false 来禁用。

---
#### 6.4.1 Spring Integration

对所有 tracer 有用。

SC Sleuth 集成了 Spring Integration 。它创建 创建span for publish 和 subscribe event。     
禁用 Spring Integration ， spring.sleuth.integration.enabled=false   

你可以提供 spring.sleuth.integration.patterns 来 明确地提供 你想tracing的 channel的名字。默认下， 除了 hystrixStreamOutput 以外的 都会 包含。

当使用 Executor 来构建 Spring Integration 的 IntegrationFlow， 你必须使用 无跟踪 版本的 Executor。   
使用 TraceableExecutorService 装饰 Spring Integration Executor Channel 会导致 span 不正确地关闭。

如果你想自定义 从 message header 中 读取/写入 tracing context 的方式， 注册下面的 bean 就足够了
1. Propagator.Setter<MessageHandlerAccessor> - 写 header 到 消息
2. Propagator.Getter<MessageHeader.Accessor> - 从 消息 读取 header


#### Spring Integration Customization

#### Customizing messaging spans

为了修改 默认的span 的名字和 tag。 只需要注册一个 MessageSpanCustomizer bean。   
你也可以 重写 现有的 DefaultMessageSpanCustomizer 来扩展 现有行为   

```java
  @Component
  class MyMessageSpanCustomizer extends DefaultMessageSpanCustomizer {
      @Override
      public Span customizeHandle(Span spanCustomizer,
              Message<?> message, MessageChannel messageChannel) {
          return super.customizeHandle(spanCustomizer, message, messageChannel)
                  .name("changedHandle")
                  .tag("handleKey", "handleValue")
                  .tag("channelName", channelName(messageChannel));
      }

      @Override
      public Span.Builder customizeSend(Span.Builder builder,
              Message<?> message, MessageChannel messageChannel) {
          return super.customizeSend(builder, message, messageChannel)
                  .name("changedSend")
                  .tag("sendKey", "sendValue")
                  .tag("channelName", channelName(messageChannel));
      }
  }
```

---
#### 6.4.2 Spring Cloud Function and Spring Cloud Stream

对所有 tracer 可用

SC Sleuth 能 使用 SC Function.   
提供一个 接受Message 作为一个参数 的 Function，Consumer，Supplier ，如 Function<Message<String>, Message<Integer>>.   
如果类型不是 Message，不会生效。   

处理 基于 Reactor 的流时，不会进行 开箱即用，如 Function<Flux<Message<String>>, Flux<Message<Integer>>>

由于 SC Stream 重用了 SC Function，你可以直接使用它。

禁用：spring.sleuth.function.enabled=false

为了能和 响应式 Stream function 使用，你可以 使用 MessagingSleuthOperators 工具类，允许你 操作 输入和输出 消息，为了 继续 tracing 上下文 且 在 tracing 上下文中 执行 自定义代码。   

```java
class SimpleReactiveManualFunction implements Function<Flux<Message<String>>, Flux<Message<String>>> {

    private static final Logger log = LoggerFactory.getLogger(SimpleReactiveFunction.class);

    private final BeanFactory beanFactory;

    SimpleReactiveManualFunction(BeanFactory beanFactory) {
        this.beanFactory = beanFactory;
    }

    @Override
    public Flux<Message<String>> apply(Flux<Message<String>> input) {
        return input.map(message -> (MessagingSleuthOperators.asFunction(this.beanFactory, message))
                .andThen(msg -> MessagingSleuthOperators.withSpanInScope(this.beanFactory, msg, stringMessage -> {
                    log.info("Hello from simple manual [{}]", stringMessage.getPayload());
                    return stringMessage;
                })).andThen(msg -> MessagingSleuthOperators.afterMessageHandled(this.beanFactory, msg, null))
                .andThen(msg -> MessageBuilder.createMessage(msg.getPayload().toUpperCase(), msg.getHeaders()))
                .andThen(msg -> MessagingSleuthOperators.handleOutputMessage(this.beanFactory, msg)).apply(message));
    }

}
```

---
#### 6.4.3 Spring RabbitMq

对 Brave tracer 有效。

使用 RabbitTemplate 来 让 tracing header 注入到 message

阻止，spring.sleuth.messaging.rabbit.enabled=false

---
#### 6.4.4 Spring Kafka

对 Brave tracer 有效

使用 Spring Kafka 的 ProducerFactory 和 ConsumerFactory， 让 tracing header 注入到 创建的Spring Kafka 的 Producer 和 Consumer。

禁用：spring.sleuth.messaging.kafka.enabled=false

---
#### 6.4.5 Spring Kafka Streams

对 Brave tracer 有效

使用 KafkaStreams， KafkaClientSupplier，使得 tracing header 注入到 Producer 和 Consumer。   
KafkaStreamTracing bean 允许更进一步的instrumentation，通过 额外的 TransformerSupplier 和 ProcessorSupplier。

禁用： spring.sleuth.messaging.kafka.streams.enabled=false

---
#### 6.4.6 Spring JMS
对Brave tracer 有用

使用 JmsTemplate，来 注入 tracing header 到 message。也支持 @JmsListener 注解到 消费端的 方法。

阻止：spring.sleuth.messaging.jms.enabled=false

不支持 JMS 的 baggage 扩散。

---
#### 6.5 OpenFeign

对所有 tracer 有用

默认。SC Sleuth 提供 对 Feign的集成 通过 TraceFeignClientAutoConfiguration.   
禁用，spring.sleuth.feign.enabled=false   

Feign 的一部分功能 通过 FeignBeanPostProcessor 来完成。   
禁用：spring.sleuth.feign.processor.enabled=false, 设置为false，SC Sleuth 不会 使用 任何 你的自定义的 Feign组建。默认的构建还在。

---
#### 6.6 OpenTracing

对所有 tracer 可用

SC Sleuth 兼容 OpenTracing。 如果你有 OpenTracing 在classpath，我们自动 注册 OpenTracing的 Tracer bean。   
禁用： spring.sleuth.opentracing.enabled=false

---
#### 6.7 Quartz

对所有 tracer 可用

我们使用 quartz job 通过 增加 Job/Trigger listener 到 Quartz Scheduler  

关闭：spring.sleuth.quartz.enabled=false

---
#### 6.8 Reactor

对所有 tracer 可用

有下列模式 来 实现基于 reactor 的应用，设置在 spring.sleuth.reactor.instrumentation-type 属性：
1. DECORATE_QUEUES, 通过新的Reactor queue wrapping 机制(Reactor 3.4.3)，我们检测Reactor 切换线程的方式。这个功能和 ON_EACH 一致，影响影响很小。
2. DECORATE_ON_EACH, 包装每个Reactor operator 在一个 trace中。在大多数情况下 传递 tracing context。可能会使得性能急剧下降。
3. DECORATE_ON_LAST, 包装 last Reactor operator 在一个 trace 中。 在某些情况下 传递 tracing context，因此 访问MDC 上下文 可能不起作用。 可能会导致 中等的 性能下降。
4. MANUAL, 包装每个 Reactor 以 侵入性最小的方式，不传递 tracing context。 取决于 用户 来完成。

当前默认是 ON_EACH 为了向后兼容，当然，我们 鼓励用户 使用 MANUAL，并且 从 WebFluxSleuthOperators 和 MessagingSleuthOperators 中获益。   
性能提升是重大的：

```java
@GetMapping("/simpleManual")
public Mono<String> simpleManual() {
    return Mono.just("hello").map(String::toUpperCase).doOnEach(WebFluxSleuthOperators
            .withSpanInScope(SignalType.ON_NEXT, signal -> log.info("Hello from simple [{}]", signal.get())));
}
```

---
#### 6.9 Redis

对 Brave tracer 有用。

我们设置 tracing 属性 到 Lettuce ClientResources 实例，来 允许 Brave tracing 构建到 Lettuce。   

禁用：spring.sleuth.redis.enabled=false

---
#### 6.10 Runnable and Callable

所有的 tracer 有用。

如果你 包装你的逻辑 到 Runnable 或 Callable。 你可以 包装这些类 在它们的 Sleuth 代表中 ： 

```java
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        // do some work
    }

    @Override
    public String toString() {
        return "spanNameFromToStringMethod";
    }
};
// Manual `TraceRunnable` creation with explicit "calculateTax" Span name
Runnable traceRunnable = new TraceRunnable(this.tracer, spanNamer, runnable, "calculateTax");
```

下面是 Callable：

```java
Callable<String> callable = new Callable<String>() {
    @Override
    public String call() throws Exception {
        return someLogic();
    }

    @Override
    public String toString() {
        return "spanNameFromToStringMethod";
    }
};
// Manual `TraceCallable` creation with explicit "calculateTax" Span name
Callable<String> traceCallable = new TraceCallable<>(tracer, spanNamer, callable, "calculateTax");
```

这样，你确保，新的span 创建 和 关闭 for 每个 执行。

---
#### 6.11 RPC

对 Brave tracer 有用

Sleuth 自动配置 RpcTracing bean，作为 gRPC 或 Dubbo 等 RPC工具的基础。

如果 需要 自定义的 客户端/服务器 RPC trace的 抽样， 只需要注册一个 SimplerFunction<RpcRequest> 类型的 bean，且 对于client名字是 sleuthRpcClientSampler, 对于server名字是 sleuthRpcServerSampler。

为了方便，还有 @RpcClientSampler, @RpcServerSampler 用于引入 正确的bean，或通过 它们的静态 String NAME字段 引用 bean 名字。

例子，1秒100条 GetUserToken 的 server 请求的 trace。 对于 health check service 的请求 不会生成 新的 trace。，其他请求会 使用 全局 采样配置。

```java
@Configuration(proxyBeanMethods = false)
class Config {
  @Bean(name = RpcServerSampler.NAME)
  SamplerFunction<RpcRequest> myRpcSampler() {
      Matcher<RpcRequest> userAuth = and(serviceEquals("users.UserService"), methodEquals("GetUserToken"));
      return RpcRuleSampler.newBuilder().putRule(serviceEquals("grpc.health.v1.Health"), Sampler.NEVER_SAMPLE)
              .putRule(userAuth, RateLimitingSampler.create(100)).build();
  }
}
```

更多，查看： github.com/openzipkin/brave/tree/master/instrumentation/rpc#sampling-policy

---
#### 6.11.1 Dubbo RPC support

通过和 Brave 的基础，SC Sleuth 支持 Dubbo。增加 brave-instrumentation-dubbo 依赖 就已经足够了。

```xml
<dependency>
    <groupId>io.zipkin.brave</groupId>
    <artifactId>brave-instrumentation-dubbo</artifactId>
</dependency>
```

你需要设置 dubbo.properties 文件，有下面的内容：
```properties
dubbo.provider.filter=tracing
dubbo.consumer.filter=tracing
```

更多 Brave - Dubbo 集成： https://github.com/openzipkin/brave/tree/master/instrumentation/dubbo-rpc

Sleuth 和 Dubbo例子： https://github.com/openzipkin/sleuth-webmvc-example/compare/add-dubbo-tracing


---
#### 6.11.2 gRPC

SC Sleuth 提供 gRPC 的支持 通过 Brave tracer.   
禁用： spring.sleuth.grpc.enabled=false

#### Variant 1
#### Dependencies
```xml
        <dependency>
            <groupId>io.github.lognet</groupId>
            <artifactId>grpc-spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>io.zipkin.brave</groupId>
            <artifactId>brave-instrumentation-grpc</artifactId>
        </dependency>
```

Gradle
```
    compile("io.github.lognet:grpc-spring-boot-starter")
    compile("io.zipkin.brave:brave-instrumentation-grpc")
```

#### Server Instrumentation
SC Sleuth 使用 grpc-spring-boot-starter 来 注册 Brave 的 gRPC 服务拦截器 with 所有@GRpcService 的服务。

#### Client Instrumentation
gRPC 客户端 使用 ManagedChannelBuilder 来 构造 一个 ManagedChannel 用来 和 gRPC服务器 通信。   
原生 ManagedChannelBuilder 提供 静态方法 作为 构造ManagedChannel 实例的 入口，然而，这个机制不受 Spring application context 的影响。

SC Sleuth 提供 SpringAwareManagedChannelBuilder 这个能通过 Spring application context 定制，并且由 gPRC client注入。   
创建 ManagedChannel 实例的时候，必须使用这个 builder。

Sleuth 创建一个 TracingManagedChannelBuilderCustomizer, 这个注入 Brave的 client 拦截器 到 SpringAwareManagedChannelBuilder.

#### Variant 2
Grpc Spring Boot Starter 自动发现 SC Sleuth 的存在 和 Brave的 为gRPC的 基础架构 和 注册必要的 客户端 和/或 服务器 工具。


---
#### 6.12 RxJava

对 所有 tracer 可用。

我们 注册自定义的 RxJavaSchedulersHook，它封装了 所有 Action0 实例 在 它们的 Sleuth 代表中，这个代表被 称为 TraceAction。   
hook 开始或继续 一个span，依赖于 在Action被 列入时间表(scheduled) 是 是否有 tracing 已经在运行。   

禁用自定义 RxJavaSchedulersHook，spring.sleuth.rxjava.schedulers.hook.enabled=false

你可以定义 线程名的正则 的列表， 这些是 你不想 span 创建的。 spring.sleuth.rxjava.schedulers.ignoredthreads 属性。    

建议使用 Reactor 支持 来进行 响应式 编程 和 Sleuth

---
#### 6.13 Spring Cloud CircuitBreaker

所有tracer 有用

如果你 有 SC CircuitBreaker 在classpath， 我们会 包装 传递的 命令 Supplier 和 fallback Function 到 他们的 trace 代表。   

禁用： spring.sleuth.circuitbreaker.enabled=false



#### Common application properties

https://docs.spring.io/spring-cloud-sleuth/docs/3.0.2/reference/htmlsingle/#common-application-properties












