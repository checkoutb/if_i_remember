

https://docs.spring.io/spring-cloud/docs/2020.0.2/reference/htmlsingle/#spring-cloud-openfeign

为Spring Boot 应用提供 OpenFeign 集成，通过 自动配置 和绑定 到 Spring Environment 和其他Spring编程模型。   

---
#### Declarative REST Client: Feign

Feign 是一个 declarative(公告的，陈述的) web service client。   
它使得 编写 web service client 更容易。   
要使用Feign，需要创建一个 接口 并且 注解它。   
它有 pluggable(可插拔的) 注解支持， 包括 Feign 注解，和 JAX-RS 注解。 也支持可插拔的 encode，decode。   
Spring Cloud 增加了 Spring MVC注解的支持，以便使用 Spring Web中默认使用的 相同的 HttpMessageConverters   
当使用Feign是， Spring Cloud 集成了 Eureka，Spring Cloud CircuitBreaker,Spring Cloud LoadBalancer 来提供 负载均衡 http client。

---
#### How to Include Feign

org.springframework.cloud:spring-cloud-starter-openfeign   

```java
@SpringBootApplication
@EnableFeignClients
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

```java
@FeignClient("stores")
public interface StoreClient {
    @RequestMapping(method = RequestMethod.GET, value = "/stores")
    List<Store> getStores();

    @RequestMapping(method = RequestMethod.GET, value = "/stores")
    Page<Store> getStores(Pageable pageable);

    @RequestMapping(method = RequestMethod.POST, value = "/stores/{storeId}", consumes = "application/json")
    Store update(@PathVariable("storeId") Long storeId, Store store);
}
```

在 @FeignClient 注解中， 字符串"stores" 是一个 arbitrary(任意的) client name，这个用来 创建 Spring Cloud LoadBalancer client.   
你也可以 specify(指明) 一个 URL ，通过url属性。   
应用上下文中 bean的名字是 这个接口的 全名。 可以使用 @FeignClient 注解的 qualifiers 来指明 你自己的 别名。   

上面提到的 load-balancer client 会 去发现"stores"服务的物理地址，如果你的应用是一个Eureka client，那么它会 从Eureka service registry 中解析 服务。   
如果你不想使用Eureka，你可以 通过SimpleDiscoveryClient 在 你的外部配置中 配置 server列表

S C OpenFeign 支持 S C LoadBalancer 的所有 blocking mode 的功能.

---
#### Overriding Feign Defaults

Spring Cloud的 Feign 支持的 一个核心概念就是 被命名的client。   
每个feign 客户端 是 整体组件的一部分，这些组件一起工作，按需联系远程服务器，通过@FeignClient 来命名。   
Spring Cloud 为每个被命名的client 按需 通过FeignClientsConfiguration 创建一个新的整体，作为ApplicationContext。它包含了 feign.Decoder, feign.Encoder,feign.Contract。   
可以覆盖这个 新的整体的名字，通过 @FeignClient 的 contextId。

Spring Cloud 能让你 获得 feign client 的全部控制，通过 使用@FeignClient 声明附加配置 (在 FeignClientsConfiguration 之上)

```java
@FeignClient(name = "stores", configuration = FooConfiguration.class)
public interface StoreClient {
    //..
}
```
这个例子中，client 由 FeignClientsConfiguration 中已有的组件 和 FooConfiguration中的组件 组合而成(后者会覆盖前者)

FooConfiguration 不需要 @Configuration 标注。当然，如果标注了，那么要从 @ComponentScan 中排除它，否则会导致它成为 feign.Decoder,feign.Encoder,feign.Contract 等 的默认 来源。

使用 @FeignClient 的contextId 除了 能修改 ApplicationContext 的名字，还会 覆盖 client 名字的别名，

name和url属性 支持占位符
```java
@FeignClient(name = "${feign.name}", url = "${feign.url}")
public interface StoreClient {
    //..
}
```

S C OpenFeign 默认为feign 提供下面的bean 

|BeanType|BeanName|ClassName|
|:---|:---|:---|
|Decoder|feignDecoder|ResponseEntityDecoder (封装了一个SpringDecoder)|
|Encoder|feignEncoder|SpringEncoder|
|Logger|feignLogger|Slf4jLogger|
|MicrometerCapability|micrometerCapability|如果 feign-micrometer 在classpath中 且 MeterRegistry可用|
|Contract|feignContract|SpringMvcContract|
|Feign.Builder|feignBuilder|FeignCircuitBreaker.Builder|
|Client|feignClient|如果SpringCloud LoadBalancer在classpath，FeignBlockingLoadBalancerClient 被使用，如果没有，默认的feign client 被使用|

spring-cloud-starter-openfeign 支持 spring-cloud-starter-loadbalancer. 当然，这是一个可选支持，如果你要使用，那么需要确保 这个依赖 被加入到你的 工程中。

OkHttpClient,ApacheHttpClient,ApacheHC5 这些feign客户端可以被使用，通过设置 feign.okhttp.enabled, feign.httpclient.enabled 或 feign.httpclient.hc5.enabled 为true，并且在classpath中。   
你可以定制 HTTP 客户端 通过 提供 bean，使用Apache时提供org.apache.http.impl.client.CloseableHttpClient, 使用OK HTTP时提供okhttp3.OkHttpClient, 使用Apache HC5时提供 org.apache.hc.client5.http.impl.classic.CloseableHttpClient。   

Spring Cloud OpenFeign ***不会*** 为feign 提供下列 类的默认 bean，但是 在创建feign client时，会 从application context中 搜索 这些类。
```
Logger.Level
Retryer
ErrorDecoder
Request.Options
Collection<RequestInterceptor>
SetterFactory
QueryMapEncoder
Capability (MicrometerCapability is provided by default)
```

默认下，Retryer 类型的 一个 Retryer.NEVER_RETRY 的 bean会创建，这个会 禁用 retrying。   
这个retrying行为 和 Reign默认的 不同， 它会自动retry IOException，将其视为 短暂的网络相关异常，并且 从 ErrorDecoder 抛出 RetryableException。

创建上面类型的bean，并放到 @FeignClient 的配置中(比如 FooConfiguration) 会使得你 覆盖你描述的bean。

```java
@Configuration
public class FooConfiguration {
    @Bean
    public Contract feignContract() {
        return new feign.Contract.Default();
    }

    @Bean
    public BasicAuthRequestInterceptor basicAuthRequestInterceptor() {
        return new BasicAuthRequestInterceptor("user", "password");
    }
}
```

这个将 用feign.Contract.Default 替换 SpringMvcContract， 增加一个RequestInterceptor 到 RequestInterceptor 集合。   

@FeignClient 也可以通过下面的配置属性来配置：

application.yml
```yaml
feign:
    client:
        config:
            feignName:
                connectTimeout: 5000
                readTimeout: 5000
                loggerLevel: full
                errorDecoder: com.example.SimpleErrorDecoder
                retryer: com.example.SimpleRetryer
                defaultQueryParameters:
                    query: queryValue
                defaultRequestHeaders:
                    header: headerValue
                requestInterceptors:
                    - com.example.FooRequestInterceptor
                    - com.example.BarRequestInterceptor
                decode404: false
                encoder: com.example.SimpleEncoder
                decoder: com.example.SimpleDecoder
                contract: com.example.SimpleContract
                capabilities:
                    - com.example.FooCapability
                    - com.example.BarCapability
                metrics.enabled: false
```

默认配置，能被指明 在 @EnableFeignClients 的 defaultConfiguration中， 用和上面相似的方式。
不同的是，这个配置会被应用到所有 feign客户端。

如果你倾向于使用 配置属性 来配置所有的 @FeignClient， 你可以创建一个 配置属性 with default feign name   

你可以使用 feign.client.config.feignName.defaultQueryParameters 和 feign.client.config.feignName.defaultRequestHeaders 来指明 查询参数 和 header，这些会 和 feignName 名字的 client 的 每个请求 一起发送。

```yaml
feign:
    client:
        config:
            default:
                connectTimeout: 5000
                readTimeout: 5000
                loggerLevel: basic
```

如果我们 创建了 @Configuration bean 和 配置属性， 配置属性优先。   
feign.client.default-to-properties=false 来 @Configuration bean 优先

如果我们需要创建 多个 同名或同url 的 feign clients (它们指向相同server 但是每个是不同的自定义配置)，我们可以使用 @FeignClient 的 contextId 属性 来避免 命名冲突。   

```java
@FeignClient(contextId = "fooClient", name = "stores", configuration = FooConfiguration.class)
public interface FooClient {
    //..
}
```
```java
@FeignClient(contextId = "barClient", name = "stores", configuration = BarConfiguration.class)
public interface BarClient {
    //..
}
```

也可以 配置 FeignClient，不要从 父context 继承 bean。 通过 覆盖 FeignClientConfigurer 的 inheritParentConfiguration()，让它返回 false。

```java
@Configuration
public class CustomConfiguration{

@Bean
public FeignClientConfigurer feignClientConfigurer() {
            return new FeignClientConfigurer() {

                @Override
                public boolean inheritParentConfiguration() {
                    return false;
                }
            };

        }
}
```

默认下， Feign 客户端 不 encode / 字符。你可以修改这个行为，通过 feign.client.decodeSlash 到false。

---
#### SpringEncoder configuration

在 SpringEncoder中， 对于 二进制文件类型 设置 的 charset是 null， 其他数据 设置 UTF-8

可以修改这个行为，来 从Content-Type 头 获得 charset： feign.encoder.charset-from-content-type 为 true。   

---
#### Timeout Handling

我们可以配置 超时，给 默认 和 有名字的 client。 OpenFeign 有2种超时参数：   
connectTimeout, 阻止 由于 服务器处理时间太长 而阻塞 调用者
readTimeout，从 链接建立开始 计时，如果返回响应时间过长 ，就触发。

---
#### Creating Feign Clients Manually

有些情况，可能必须 定制 Feign client，但无法使用上面的方法。   
这种情况下，你需要创建 client，通过 Feign Builder API。   
下面是一个例子，创建了2个Feign client, 有着 相同的接口，但是 每个配置了一个 独特的 request interceptor

```java
@Import(FeignClientsConfiguration.class)
class FooController {

    private FooClient fooClient;

    private FooClient adminClient;

    @Autowired
    public FooController(Client client, Encoder encoder, Decoder decoder, Contract contract, MicrometerCapability micrometerCapability) {
        this.fooClient = Feign.builder().client(client)
                .encoder(encoder)
                .decoder(decoder)
                .contract(contract)
                .addCapability(micrometerCapability)
                .requestInterceptor(new BasicAuthRequestInterceptor("user", "user"))
                .target(FooClient.class, "https://PROD-SVC");

        this.adminClient = Feign.builder().client(client)
                .encoder(encoder)
                .decoder(decoder)
                .contract(contract)
                .addCapability(micrometerCapability)
                .requestInterceptor(new BasicAuthRequestInterceptor("admin", "admin"))
                .target(FooClient.class, "https://PROD-SVC");
    }
}
```

上面例子中，FeignClientsConfiguration.class 是 S C OpenFeign 提供的 默认 配置。   
PROD-SVC 是 client 发送request的 目的 服务的名字   
Feign 的 Contract 对象 定义了 接口中什么注解 和 值 是有效的。 自动注入的 Contract bean 提供了 对 SpringMVC 注解的支持，和 默认的Feign原生注解的支持。

你也可以使用 Builder 来配置 FeignClient 不集成beans 从父context，你可以 覆盖 inheritParentContext(false) 来做到。

---
#### Feign Spring Cloud CircuitBreaker Support

如果 S C CircuitBreaker 在classpath，且 feign.circuitbreaker.enabled=true, feign封装所有方法 到 circuit breaker 中。   

在每个client 上 禁用 S C CircuitBeaker， 创建 vanilla(香草) Feign.Builder with "prototype" scope.   

```java
@Configuration
public class FooConfiguration {
    @Bean
    @Scope("prototype")
    public Feign.Builder feignBuilder() {
        return Feign.builder();
    }
}
```

circuit breaker 的名字规律： <feignClientName>#<calledMethod>, 当调用一个 名字是foo的 @FeignClient 的 bar接口方法 时， circuit breaker 的名字是 foo_bar (。。真的foo_bar，估计应该是 foo#bar， _是一个合法的方法名字符)

---
#### Feign Spring Cloud CircuitBreaker Fallbacks

SCCB 支持 fallback概念： 一个默认的code path 被执行，当 断路器打开 或 有error。   

允许fallback，为@FeignClient 设置 fallback 属性 为 实现fallback 的 类名。你也需要声明 你的实现 为一个Spring Bean.

```java
@FeignClient(name = "test", url = "http://localhost:${server.port}/", fallback = Fallback.class)
protected interface TestClient {

    @RequestMapping(method = RequestMethod.GET, value = "/hello")
    Hello getHello();

    @RequestMapping(method = RequestMethod.GET, value = "/hellonotfound")
    String getException();

}

@Component
static class Fallback implements TestClient {

    @Override
    public Hello getHello() {
        throw new NoFallbackAvailableException("Boom!", new RuntimeException());
    }

    @Override
    public String getException() {
        return "Fixed response";
    }

}
```

如果 需要获得 触发fallback的 原因，可以使用 @FeignClient 中的 fallbackFactory 属性。

```java
@FeignClient(name = "testClientWithFactory", url = "http://localhost:${server.port}/",
            fallbackFactory = TestFallbackFactory.class)
protected interface TestClientWithFactory {

    @RequestMapping(method = RequestMethod.GET, value = "/hello")
    Hello getHello();

    @RequestMapping(method = RequestMethod.GET, value = "/hellonotfound")
    String getException();

}

@Component
static class TestFallbackFactory implements FallbackFactory<FallbackWithFactory> {

    @Override
    public FallbackWithFactory create(Throwable cause) {
        return new FallbackWithFactory();
    }

}

static class FallbackWithFactory implements TestClientWithFactory {

    @Override
    public Hello getHello() {
        throw new NoFallbackAvailableException("Boom!", new RuntimeException());
    }

    @Override
    public String getException() {
        return "Fixed response";
    }

}
```

---
#### Feign and @Primary

当使用 Feign with SCCB fallback，ApplicationContext中有多个 相同类型的bean。这个会导致 @Autowired 无法工作，由于 存在多个bean，且没有bean被标记为primary。   

为了能够共走，SC OpenFeign 将所有 Feign 实例 都标记为 @Primary，这样spring就知道 注入哪个bean。。。。？都是Primary还是有问题啊。   

有时，这是不可取的，要关闭上面的行为，设置@FeignClient 的 primary 属性

```java
@FeignClient(name = "hello", primary = false)
public interface HelloClient {
    // methods here
}
```

---
#### Feign Inheritance Support

Feign 支持 boilerplate(样板文件) api 通过 单继承接口。   
这允许 将常见操作 分组到 基本接口中。


UserService.java
```java
public interface UserService {
    @RequestMapping(method = RequestMethod.GET, value ="/users/{id}")
    User getUser(@PathVariable("id") long id);
}
```

UserResource.java
```java
@RestController
public class UserResource implements UserService {

}
```

UserClient.java
```java
package project.user;

@FeignClient("users")
public interface UserClient extends UserService {

}
```

不建议 在 server 和 client 间分享一个接口。 这引入了 紧耦合， 也不适用于 SpringMVC当前的模式。

---
#### Feign request/response compression

你可能考虑 允许 request 或 response GZIP 压缩 for 你的Feign请求， 你可以这么做，通过 激活下面的某个属性   

```properties
feign.compression.request.enabled=true
feign.compression.response.enabled=true
```

Feign request压缩 为你 提供 类似于 你为web服务器设置的 配置：
```properties
feign.compression.request.enabled=true
feign.compression.request.mime-types=text/xml,application/xml,application/json
feign.compression.request.min-request-size=2048
```

这些属性 允许你 选择压缩的media type，和 最小请求长度。

对于除了OkHttpClient 外的 http client， 默认 gzip decoder 能被激活 来 decode gzip 的 UTF-8编码的 响应：
```properties
feign.compression.response.enabled=true
feign.compression.response.useGzipDecoder=true
```

---
#### Feign logging

每个Feign client 被创建时，都会有一个 Logger被创建。默认下，logger的名字是 用于创建Feign client的 接口 的全名。   
Feign logging 智慧打印到 DEBUG 级别。

application.yml
```yaml
logging.level.project.user.UserClient: DEBUG
```

可以为每个client，单独配置 Logger.Level 对象， 来告诉 Feign log多少：   

NONE， no logging (default)   
BASIC， 只log request method 和 url 和 response 状态码 和执行时间   
HEADERS， 上面的基础上，再log request，response headers   
FULL， log request和response的 headers，body，metadata   

下面，设置Logger.Level 到 FULL
```java
@Configuration
public class FooConfiguration {
    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}
```

---
#### Feign Capability(能力/功能) support

Feign capabilities 暴露 核心 Feign 组件， 这样，这些组件能被修改。   
比如，capabilities 能获取 Client， decorate(装饰)它，把装饰后的实力 返还给 Feign。   
对metrics libraries的支持 是一个 很好的 现实例子。   

创建 一个或多个 Capability bean，把它们放到 @FeignClient 配置中， 使得你 注册它们并修改 涉及的client的 行为

```java
@Configuration
public class FooConfiguration {
    @Bean
    Capability customCapability() {
        return new CustomCapability();
    }
}
```

---
#### Feign metrics

如果下面的条件是true，一个 MicrometerCapability bean 被创建，被注册，这样，你的Feign client 提交 metrics 到 Micrometer：   
1. feign-micrometer 在 classpath
2. MeterRegistry bean 可用
3. feign metrics properties 被设置为 true (默认是true)
   1. feign.metrics.enabled=true (对所有client 起效)
   2. feign.client.config.{feignName}.metrics.enabled=true (对一个client起效)

禁用，只需要下面 之一:   
从classpath 移除 feign-micrometer   
设置下面属性为false： feign.metrics.enabled=false 或 feign.client.config.{feignName}.metrics.enabled=false   

可以定制 MicrometerCapability 通过 注册你自己的bean：
```java
@Configuration
public class FooConfiguration {
    @Bean
    public MicrometerCapability micrometerCapability(MeterRegistry meterRegistry) {
        return new MicrometerCapability(meterRegistry);
    }
}
```

---
#### Feign @QueryMap support

OpenFeign 的 @QueryMap 注解 提供了 支持：POJO用作GET参数map。   

遗憾的是，默认的OpenFeign QueryMap 注解 和 Spring冲突，因为它缺乏 value 属性。   

Spring Cloud OpenFeign 提供了一个等价的 @SpringQueryMap 注解，这个能用来 标注一个 POJO 或 Map 参数 作为一个查询参数map。

例如，Params类定义了 参数param1,param2：

```java
// Params.java
public class Params {
    private String param1;
    private String param2;

    // [Getters and setters omitted for brevity]
}
```
下面的feign client 使用 Params 类，通过 @SpringQueryMap : 
```java
@FeignClient("demo")
public interface DemoTemplate {

    @GetMapping(path = "/demo")
    String demoEndpoint(@SpringQueryMap Params params);
}
```

如果你需要更多关于 生成的查询参数map的 控制， 你可以实现自定义的 QueryMapEncoder bean。

---
#### HATEOAS support

Spring 提供一些 API 来创建 REST representations(陈述) 按照 HATEOAS准则，Spring Hateoas 和 Spring Data REST.   

如果你的工程 使用 org.springframework.boot:spring-boot-starter-hateoas 或 org.springframework.boot:spring-boot-starter-data-rest. Feign HATEOAS 支持 被默认启用。

当HATEOAS支持 被启用，Feign client 被允许 序列化和 反序列化 HATEOAS 陈述 模型： EntityModel, CollectionModel, PagedModel.   

```java
@FeignClient("demo")
public interface DemoTemplate {

    @GetMapping(path = "/stores")
    CollectionModel<Store> getStores();
}
```

---
#### Spring @MatrixVariable Support

SC OpenFeign 支持 Spring 的 @MatrixVariable 注解。   

如果一个map 被作为 方法参数传递， @MatrixVariable 路径段 被创建，通过 用= 来join map中的k-v对   

如果不同的 对象被传递， @MatrixVariable注解中的 name(如果定义了) 或 被注解的变量的名字 被join with 提供的方法参数 使用=   

***IMPORTANT***
虽然，在服务器端，Spring 不要求用户 命名 路径段占位符 和 matrix 变量名字一样， 因为这样做会使得 客户端 感到非常二义性。   
SC OpenFeign 要求： 你增加一个 路径段占位符 with 名字匹配 @MatrixVariable的name属性(如果提供)，或 被注解的变量的名字。   

```java
@GetMapping("/objects/links/{matrixVars}")
Map<String, List<String>> getObjects(@MatrixVariable Map<String, List<String>> matrixVars);
```
变量名 和 路径段占位符 都叫 matrixVars

```java
@FeignClient("demo")
public interface DemoTemplate {

    @GetMapping(path = "/stores")
    CollectionModel<Store> getStores();
}
```

---
#### Feign CollectionFormat support

支持 feign.CollectionFormat 通过提供 @CollectionFormat 注解，你可以标注一个 Feign client 方法 with 它 通过 传递 需要的 feign.CollectionFormat 作为注解值   

下面例子中，CSV format 被用来 代替默认的 EXPLODED 来执行 方法。
```java
@FeignClient(name = "demo")
protected interface PageableFeignClient {

    @CollectionFormat(feign.CollectionFormat.CSV)
    @GetMapping(path = "/page")
    ResponseEntity performRequest(Pageable page);

}
```
设置CSV格式 当发送 Pageable 作为一个 查询参数时， 为了 能正确 encode。

---
#### Reactive Support
OpenFeign 目前不支持 reactive client，比如 Spring WebClient.   
SC OpenFeign 也不支持。   

我们建议使用 feign-reactive for Spring WebClient support

---
#### Early Initialization Errors

当启动应用时， 取决于你 如何使用你的 Feign client，你可能会碰到 初始化错误。   
解决这个问题，你可以使用 ObjectProvider 当 autowiring 你的client时。

```java
@Autowired
ObjectProvider<TestFeginClient> testFeginClient;
```

---
#### Spring Data Support

你可能希望 启用 Jackson Module for 支持 org.springframework.data.domain.Page 和 Sort 来解码   

> feign.autoconfiguration.jackson.enabled=true









