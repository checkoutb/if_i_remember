

htt://docs.spring.io/spring-cloud/docs/2020.0.2/reference/htmlsingle/#spring-cloud-gateway

提供了一个构建在Spring 生态系统上的 API 网关，包括Spring 5, Spring Boot 2 和 Project Reactor。   

Gateway 致力于提供 一个简单 有效的 方式 来route to APIs 并提供 跨领域的 横切关注点，如security，monitoring/metrics,resiliency.


#### How to Include Spring Cloud Gateway

使用starter，org.springframework.cloud:spring-cloud-starter-gateway    

如果你包含starter，但你不想激活 gateway，设置 spring.cloud.gateway.enabled=false

Spring Cloud Gateway 基于 Spring Boot 2, Spring WebFlux, Project Reactor.   

Spring Cloud Gateway 需要 SpringBoot 和SpringWebFlux 提供的 Netty runtime。 Netty runtime 不会在 传统的 Servlet Container 或 打包成WAR 时 work。

---
#### Glossary (术语)

Route，网关的基本模块，通过 一个ID，一个目标URI，predicate集合，filter集合 来定义。 一个route 是匹配的，如果 aggregate predicate (聚合谓语) 是true。

Predicate，是JDK8的 Predicate函数式，输入类型是Spring的 ServerWebExchange，使你匹配 on HTTP Request中的任何东西，比如headers，parameters。

Filter，GatewayFilter 的实例，实例通过具体的工厂构建。在这里，你可以修改 request和response before or after 发送 downstream request。

---
#### How It Works

下面的图标提供了一个 高层的概述： SpringCloudGateway 如何工作：   
。。。图404了。。。  https://docs.spring.io/spring-cloud/docs/2020.0.2/reference/htmlsingle/images/spring_cloud_gateway_diagram.png

客户端发送请求到 Gateway，如果 Gateway Handler Mapping 确定了 request 匹配一个route，它被发送到 Gateway Web Handler。这个handler 运行 request，through 特定于这个request的filter chain，
filter能在发送代理请求 之前和之后 运行。所有的pre filter都执行，然后proxy request 创建，创建后，post filter 执行。   

route中的URI，如果没有端口，就默认http->80, https->443

---
#### Configuring Route Predicate Factories and Gateway Filter Factories

2种方法配置 predicate 和 filter： 快捷方式，和 完全扩展的参数 (shortcut, fully expanded arguments).   
下面的大部分例子都是 快捷方式。

---
#### Shortcut Configuration

通过 filter名字，然后 =，然后 逗号分隔的参数值。   

application.yml
```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - Cookie=mycookie,mycookievalue
```
上面定义了 "Cookie" route predicate factory with 2个参数，cookie 名字：mycookie, 并且值是 mycookievalue   

---
#### Fully Expanded Arguments

更像标准的yaml 配置，通过 name/value 对。   
通常，是 name key 和 args key， args 是一个map 来配置predicate 或filter

application.yml
```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - name: Cookie
          args:
            name: mycookie
            regexp: mycookievalue
```

---
#### Route Predicate Factories

Gateway的 route匹配 是 Spring WebFlux 的 HandlerMapping结构的 一部分。   
Gateway包含许多 内置的 route predicate factories。   
你可以组合多个 route predicate factories 通过 逻辑 and

---
#### The After Route Predicate Factory

After 路由谓语工厂 接受一个参数，参数是一个 datetime (java的 ZonedDateTime)。   
这个谓语 匹配 那些发生在这个时间之后的request   

application.yml
```
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - After=2017-01-20T17:42:47.789-07:00[America/Denver]
```
匹配那些 在 Jan 20, 2017 17:42 Mountain Time (Denver) 之后 创建的 request。

---
#### The Before Route Predicate Factory

接受一个 ZonedDateTime 的参数， 匹配那些 发生在 这个时间 之前的 request。   

application.yml
```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: before_route
        uri: https://example.org
        predicates:
        - Before=2017-01-20T17:42:47.789-07:00[America/Denver]
```
matches any request made before Jan 20, 2017 17:42 Mountain Time (Denver).

---
#### The Between Route Predicate Factory

2个时间参数，都是 ZonedDateTime类型。 匹配那些 在 第一个参数后，第二个参数前 的request。 第二个参数必须在第一个之前。   

application.yml
```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: between_route
        uri: https://example.org
        predicates:
        - Between=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[America/Denver]
```
。。不知道 等于 行不行，看样子 不行，但是 sql的between是包含的。

---
#### The Cookie Route Predicate Factory

Cookie 路由谓语工厂 接受2个参数，cookie的 name 和 regexp (java RE)。   
匹配那些 cookies中有这个name 且 值匹配 RE 的 request。   

application.yml
```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: cookie_route
        uri: https://example.org
        predicates:
        - Cookie=chocolate, ch.p
```
匹配那些 包含 名字是chocolate，值匹配ch.p模式 的cookie 的 request。   

---
#### The Header Route Predicate Factory

Header 路由谓语工厂接受2个参数，分表代表header 的 name 和 regexp。   

application.xml
```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: header_route
        uri: https://example.org
        predicates:
        - Header=X-Request-Id, \d+
```

匹配 包含一个名为X-Request-Id 的 header且值是 \d+ 模式 的 request

---
#### The Host Route Predicate Factory

Host，接受一个参数，host名字pattern的列表    
是Ant-style pattern， .作为分隔符。   
匹配那些 Host 头 匹配pattern的。   

application.yml
```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: https://example.org
        predicates:
        - Host=**.somehost.org,**.anotherhost.org
```

URI template 变量 (如 {sub}.myhost.org) 也被支持。

匹配，如果 request 有 Host header，并且值是 www.somehost.org or beta.somehost.org or www.anotherhost.org 。

这个谓语 提取 URI模板参数 (如上面的 sub) 作为 name-value的 map，并放到 ServerWebExchange.getAttributes() 中，key是 ServerWebExchangeUtils.URI_TEMPLATE_VARIABLES_ATTRIBUTE。   
这些值，后续在 Gateway Filter 工厂中 会被用到。

---
#### The Method Route Predicate Factory

Method, 接受一个参数，是一个或多个HTTP方法。   

application.yml
```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: method_route
        uri: https://example.org
        predicates:
        - Method=GET,POST
```

匹配，如果request 方法是 GET 或 POST

---
#### The Path Route Predicate Factory

Path, 接受2个参数， Spring PathMatcher的列表， 和 一个可选择标记 matchTrailingSlash (默认true)   

application.yml
```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: path_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment},/blue/{segment}
```

匹配，如果request path，是 /red/1 or /red/1/ or /red/blue or /blue/green .等

如果matchTrailingSlash 被设置为 false， 那么 /red/1/ 不被匹配。

这个predicate 抽取 URI 模板变量 (如上面定义中的 segment) 形成一个 map，并放到 ServerWebExchange.getAttributes(), key是 ServerWebExchangeUtils.URI_TEMPLATE_VARIABLES_ATTRIBUTE   
这些值 会在 Gateway Filter 工厂 中被用到。

公共方法(get) 用来 获得这些 变量。
```
Map<String, String> uriVariables = ServerWebExchangeUtils.getPathPredicateVariables(exchange);

String segment = uriVariables.get("segment");
```
。。。？？？

---
#### The Query Route Predicate Factory

Query, 接受2个参数， 一个param， 一个可选的 regexp。

application.yml
```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: query_route
        uri: https://example.org
        predicates:
        - Query=green
```
匹配，如果request 包含一个 green的 查询参数

application.yml
```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: query_route
        uri: https://example.org
        predicates:
        - Query=red, gree.
```
如果包含 red查询参数，且参数的值 匹配 gree.  如果green， greet。

---
#### The RemoteAddr Route Predicate Factory

RemoteAddr, 接受 source的列表(至少1个)，列表里是 CIDR标记(IPv4 or IPv6) 的string， 比如 192.168.0.1/16   

application.yml
```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: remoteaddr_route
        uri: https://example.org
        predicates:
        - RemoteAddr=192.168.1.1/24
```

匹配，如果 request的 远程地址是 192.168.1.10 ..等。

---
#### The Weight Route Predicate Factory

Weight, 2个参数， group， weight(一个int)。 权重按组计算。   

application.yml
```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: weight_high
        uri: https://weighthigh.org
        predicates:
        - Weight=group1, 8
      - id: weight_low
        uri: https://weightlow.org
        predicates:
        - Weight=group1, 2
```

80% 的转发到 weighthign.org, 20%的转发到 weightlow.org

---
#### Modifying the Way Remote Addresses Are Resolved

默认下，RemoteAddr 路由谓语工厂 使用 request的 远程地址，这可能并不匹配真实客户端IP，如果Gateway在 代理层后面。   

你可以自定义 如何解析远程地址，通过设置一个自定义的 RemoteAddressResolver。   
Gateway 有一个 非默认的 远程地址解析，基于 X-Forwarded-For header, XForwardedRemoteAddressResolver   

XForwardedRemoteAddressResolver 有2个 静态构造方法，针对不同的security：   
XForwardedRemotedAddressResolver::trustAll 返回一个 RemoteAddressResolver，它总会获得 在 X-Forwarded-For 头中找到的第一个IP。它容易受到欺骗，比如一个恶意客户端设置了 X-Forwarded-For 的初始值，这个初始值会被 这个解析器获得。    
XForwardedRemotedAddressResolver::maxTrustedIndex 获得一个索引，它是 Gateway前面运行的 受信任的设施的数量 相关的。如果Gateway只通过HAProxy 被访问，那么索引值是1。如果Gateway前面有2跳来通过受信任的服务，那么值是2。   

考虑下面的 header：   
X-Forwarded-For: 0.0.0.1, 0.0.0.2, 0.0.0.3

maxTrustedIndex 是 [INT_MIN, 0], 初始化的时候抛出 IllegalArgumentException   
1 0003   
2 0002   
3 0001   
[4, INT_MAX] 0001   

下面展现了如何在Java中获得相同的配置：   

GatewayConfig.java
```
RemoteAddressResolver resolver = XForwardedRemoteAddressResolver
    .maxTrustedIndex(1);

...

.route("direct-route",
    r -> r.remoteAddr("10.1.1.1", "10.10.1.1/24")
        .uri("https://downstream1")
.route("proxied-route",
    r -> r.remoteAddr(resolver, "10.10.1.1", "10.10.1.1/24")
        .uri("https://downstream2")
)
```

---
---
#### GatewayFilter Factories

Route filters 允许 修改 HTTP request 或 response。   
Route filters 的作用范围是一个 特定的route。   

更多详细的例子，看 测试案例： https://github.com/spring-cloud/spring-cloud-gateway/blob/main/spring-cloud-gateway-server/src/test/java/org/springframework/cloud/gateway/filter/factory/DedupeResponseHeaderGatewayFilterFactoryUnitTests.java

---
#### The AddRequestHeader GatewayFilter Factory

AddRequestHeader 接受 name，value，2个参数：   
application.yml
```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_header_route
        uri: https://example.org
        filters:
        - AddRequestHeader=X-Request-red, blue
```
会 对所有匹配的 request 增加 X-Request-red:blue header 到 请求下游的headers 中。   

AddRequestHeader 知道 URI中用来匹配 路径或host的 变量，URI变量可以被当作值来使用，并且在运行时扩展。   
下面配置了一个 AddRequestHeader，里面使用了变量   
application.yml
```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_header_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment}
        filters:
        - AddRequestHeader=X-Request-Red, Blue-{segment}
```
。。没有 predicate 的话，就不会有 {segment} 啊。

---
#### AddRequestParameter

接受2个参数， name，value   

application.yml
```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_parameter_route
        uri: https://example.org
        filters:
        - AddRequestParameter=red, blue
```
对所有匹配的 request，当往下游发送的时候，增加 red=blue 这个 query参数 到 uri

AddRequestParameter 知道 URI中用于 匹配 path 或 host的 变量。 这个变量可以当作value使用，并且在运行时扩展。   

application.yml
```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_parameter_route
        uri: https://example.org
        predicates:
        - Host: {segment}.myhost.org
        filters:
        - AddRequestParameter=foo, bar-{segment}
```

---
#### AddResponseHeader

接受2个参数，name value。   

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_response_header_route
        uri: https://example.org
        filters:
        - AddResponseHeader=X-Response-Red, Blue
```

对匹配的request， 下游的response 的header里 增加 X-Response-Foo:Bar    

知道 URI中 路径 or host的 变量。   
```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_response_header_route
        uri: https://example.org
        predicates:
        - Host: {segment}.myhost.org
        filters:
        - AddResponseHeader=foo, bar-{segment}
```

---
#### DedupeResponseHeader

接受一个 name参数，一个可选的 strategy参数。   
name可以包含 空格分隔的 header的name。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: dedupe_response_header_route
        uri: https://example.org
        filters:
        - DedupeResponseHeader=Access-Control-Allow-Credentials Access-Control-Allow-Origin
```

这个会移除 response 的 Access-Control-Allow-Credentials，Access-Control-Allow-Origin header中重复值。   
用于 当 网关CORS逻辑 和 下游逻辑 都添加了它们。

可选的 strategy 参数，可选的值是 RETAIN_FIRST(默认), RETAIN_LAST, RETAIN_UNIQUE

---
#### Spring Cloud CircuitBreaker GatewayFilter Factory

这个 factory 使用 Spring Cloud CircuitBreaker API 来 包装 Gateway routes 在一个circuit breaker中。   
SpringCloud CircuitBreaker 支持多个 libraries 能和 Gateway 连用。   
Cloud 支持 Resilience4J 开箱即用。   

激活 Spring Cloud CircuitBreaker filter， 你需要把 spring-cloud-starter-circuitbreaker-reactor-resilience4j 放到classpath。   

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: circuitbreaker_route
        uri: https://example.org
        filters:
        - CircuitBreaker=myCircuitBreaker
```

S C CircuitBreaker filter 也接受一个可选的 fallbackUri 参数，目前，只有 forward: 模式的URI 支持。   
如果fallback 被调用，request 会被 forward 到 匹配URI的 controller中。
```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: circuitbreaker_route
        uri: lb://backing-service:8088
        predicates:
        - Path=/consumingServiceEndpoint
        filters:
        - name: CircuitBreaker
          args:
            name: myCircuitBreaker
            fallbackUri: forward:/inCaseOfFailureUseThis
        - RewritePath=/consumingServiceEndpoint, /backingServiceEndpoint
```

下面是相同效果，在java   
Application.java
```
@Bean
public RouteLocator routes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("circuitbreaker_route", r -> r.path("/consumingServiceEndpoint")
            .filters(f -> f.circuitBreaker(c -> c.name("myCircuitBreaker").fallbackUri("forward:/inCaseOfFailureUseThis"))
                .rewritePath("/consumingServiceEndpoint", "/backingServiceEndpoint")).uri("lb://backing-service:8088")
        .build();
}
```

这个例子里会 forward 到 /inCaseOfFailureUseThis URI，当 断路器的 fallback 被调用。   
这个例子也演示了 (可选的) SpringCloud LB 的负载均衡 (在目标URI 前面加lb 前缀)

主要场景是，使用 fallbackUri 来定义一个 gateway 应用 内部的controller 或 handler。   
当然，你有可以 reroute 请求到 外部应用的 controller 或 handler。   

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: ingredients
        uri: lb://ingredients
        predicates:
        - Path=//ingredients/**
        filters:
        - name: CircuitBreaker
          args:
            name: fetchIngredients
            fallbackUri: forward:/fallback
      - id: ingredients-fallback
        uri: http://localhost:9994
        predicates:
        - Path=/fallback
```
。。。fallbackUri 转发给 /fallback, 然后 /fallback 会发送给 localhost:9994 

上面的例子中，gateway应用没有 fallback 的 端点 或 handler。 而是 其他应用有一个，注册在 localhost:9994   

万一 request 被转发到 fallback，S C CircuitBreaker Gateway filter 也提供 导致fallback的 Throwable。   
Throwable 会被添加到 ServerWebExchange，作为 ServerWebExchangeUtils.CIRCUITBREAKER_EXECUTION_EXCEPTION_ATTR 属性，在gateway应用的处理fallback中能获得。

对于外部的 controller/handler, 能把exception详情添加到 header。 FallbackHeaders GatewayFilter Factory.

---
#### Tripping(触发/断开) The Circuit Breaker On Status Codes

有时，你可能想要 trip 断路器，基于 它包装的route返回的 状态码。   
断路器配置 接受 状态码列表，如果这些状态码返回，会导致 断路器 打开。      
状态码可以使用 int 或 string   

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: circuitbreaker_route
        uri: lb://backing-service:8088
        predicates:
        - Path=/consumingServiceEndpoint
        filters:
        - name: CircuitBreaker
          args:
            name: myCircuitBreaker
            fallbackUri: forward:/inCaseOfFailureUseThis
            statusCodes:
              - 500
              - "NOT_FOUND"
```
Application.java
```
@Bean
public RouteLocator routes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("circuitbreaker_route", r -> r.path("/consumingServiceEndpoint")
            .filters(f -> f.circuitBreaker(c -> c.name("myCircuitBreaker").fallbackUri("forward:/inCaseOfFailureUseThis").addStatusCode("INTERNAL_SERVER_ERROR"))
                .rewritePath("/consumingServiceEndpoint", "/backingServiceEndpoint")).uri("lb://backing-service:8088")
        .build();
}
```

---
#### The FallbackHeaders GatewayFilter Factory

FallbackHeaders factory 让你增加 S C CircuitBreaker 执行异常明细 到 转发给 外部应用的 fallbackUri 的request的 headers中。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: ingredients
        uri: lb://ingredients
        predicates:
        - Path=//ingredients/**
        filters:
        - name: CircuitBreaker
          args:
            name: fetchIngredients
            fallbackUri: forward:/fallback
      - id: ingredients-fallback
        uri: http://localhost:9994
        predicates:
        - Path=/fallback
        filters:
        - name: FallbackHeaders
          args:
            executionExceptionTypeHeaderName: Test-Header
```

在这个例子中， 断路器中 发生异常后，request被转发到 运行在localhost:9994 端口的 应用的 fallback 端点/handler。   
通过FallbackHeaders filter，header包含 异常的类型和信息，以及可能有的 导致这个异常的异常(。。cause by)的类型和信息。   

你可以覆盖 header的名字，通过设置下面的 参数(括号里为 默认值)   

```
executionExceptionTypeHeaderName ("Execution-Exception-Type")
executionExceptionMessageHeaderName ("Execution-Exception-Message")
rootCauseExceptionTypeHeaderName ("Root-Cause-Exception-Type")
rootCauseExceptionMessageHeaderName ("Root-Cause-Exception-Message")
```

更多关于 断路器 和 网关的信息 看 上面的  Spring Cloud CircuitBreaker Factory

---
#### MapRequestHeader

接受2个参数，fromHeader, toHeader.   
在incoming request, 创建一个新的名字为 toHeader值 的header， value是 一个已存在的header (fromHeader)。   
如果 输入header 不存在，filter 不会有作用。   
如果 toHeader 已存在，它的值会 加上 新的value。   

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: map_request_header_route
        uri: https://example.org
        filters:
        - MapRequestHeader=Blue, X-Request-Red
```
增加 X-Request-Red:<value> 到 发送给下游的request中， 值是 incoming request的 Blue 头。

---
#### PrefixPath

接收一个参数，prefix   

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: prefixpath_route
        uri: https://example.org
        filters:
        - PrefixPath=/mypath
```
前缀 /mypath 会增加到 匹配的request 的路径。 所以 访问/hello 会发送到 /mypath/hello

---
#### PreserveHostHeader

没有参数。   
设置request attribute，routing filter 会检查这个attribute，来决定 是否 发送 原始host头，而不是 HTTP客户端确定的 host头。   

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: preserve_host_route
        uri: https://example.org
        filters:
        - PreserveHostHeader
```

---
#### RequestRateLimiter

使用 RateLimiter 的实现类 来决定 当前request 是否 允许继续执行。 如果不允许，默认返回一个 HTTP 429 Too Many Requests。   

接受一个可选的 keyResolver 参数，和 一个特定于rate limiter的参数。   

keyResolver 是一个 bean，实现了 KeyResolver接口。在配置中，通过 SpEL 来确定bean的。#{@myKeyResolver} 是一个SpEL表达式，ref了 名字是myKeyResolver 的bean。   

下面是 KeyResolver 接口：
```
public interface KeyResolver {
    Mono<String> resolve(ServerWebExchange exchange);
}
```

这个接口让 可插拔策略 派生出 限制请求的 key。   

默认实现是 PrincipalNameKeyResolver，它从 ServerWebExchange 检索 Principal，然后调用 Principal.getName()   

默认，如果KeyResolver 找不到key，request 会被拒绝。   
你可以调整行为，通过设置 spring.cloud.gateway.filter.request-rate-limiter.deny-empty-key (true 或 false) 和 
spring.cloud.gateway.filter.request-rate-limiter.empty-key-status-code

RequestRateLimiter 没有 shortcut形式。 下面是 非法的：   
```
# INVALID SHORTCUT CONFIGURATION
spring.cloud.gateway.routes[0].filters[0]=RequestRateLimiter=2, 2, #{@userkeyresolver}
```

---
#### Redis RateLimiter

Redis实现基于 Stripe (https://stripe.com/blog/rate-limiters), 需要 spring-boot-starter-data-redis-reactive 这个starter   
使用的算法是 令牌桶算法   

redis-rate-limiter.replenishRate (。。补充速度) 属性是 你希望一个用户 每秒执行多少个request，放弃的request不计入。令牌桶 fill的速度    
redis-rate-limiter.burstCapacity 是 一秒内 一个用户 允许的 最多请求。令牌桶最多令牌数，设置0 阻塞所有request    
redis-rate-limiter.requestedTokens 是 一个请求 消耗多少个令牌，这个是每个请求从桶中获得的令牌数，默认 1    

设置 replenishRate 和 burstCapacity 的值 相同，则获得一个 稳定的速度。   
burstCapacity > replenishRate 则允许 暂时的突发，这种情况下，需要允许速率限制器在突发 之间有一段时间(根据replenishRate),因为2个连续的突发将导致请求丢失 (HTTP 429, Too Many Request)   

下面配置了一个 redis-rate-limiter,     
低于 1请求/s 的限制 是可以的，通过设置 replenishRate 为 想要的请求数， requestedTokens 为 时间跨度(单位秒)，burstCapacity 是 replenishRate 和 requestedTokens 的积。   
比如，replenishRate=1, requestedTokens=60 and burstCapacity=60 会有限制： 1 request/min.

application.yml
```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: requestratelimiter_route
        uri: https://example.org
        filters:
        - name: RequestRateLimiter
          args:
            redis-rate-limiter.replenishRate: 10
            redis-rate-limiter.burstCapacity: 20
            redis-rate-limiter.requestedTokens: 1
```

配置一个 KeyResolver 在 java：
```
@Bean
KeyResolver userKeyResolver() {
    return exchange -> Mono.just(exchange.getRequest().getQueryParams().getFirst("user"));
}
```

这定义了一个 请求速率： 每个用户最多10个，突发可以是20,但是突发后下一秒只能10个请求。   
KeyResolver 是简单的例子，获得 user 这个请求参数。   

你也可以定义一个 速度限制 是一个实现 RateLimiter 的bean。   
在配置中，你可以通过SpEL #{@myRateLimiter} 来ref 名字是myRateLimiter 的 bean。   

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: requestratelimiter_route
        uri: https://example.org
        filters:
        - name: RequestRateLimiter
          args:
            rate-limiter: "#{@myRateLimiter}"
            key-resolver: "#{@userKeyResolver}"
```
。。key-resolver 就是 以什么作为 是一个 user ？。 就是 每个用户 的 速率限制是 分开的。

---
#### RedirectTo

接受2个参数， status 和url。   
status应该是 300 系列的 HTTP code，如301   
url参数 是一个 合法的URL。 这是 Location 头的值。   
对于 相对 重定向，你应该使用 uri: no://op 作为 你 route定义中的 uri。   

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: prefixpath_route
        uri: https://example.org
        filters:
        - RedirectTo=302, https://acme.org
```

发送 带有 Location:https://acme.org 这个头的 状态302 以执行重定向。   

---
#### RemoveRequestHeader

接受一个参数：name， 是将被移除的header的 名字。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: removerequestheader_route
        uri: https://example.org
        filters:
        - RemoveRequestHeader=X-Request-Foo
```

会移除 X-Request-Foo 头，在发送到下游前。

---
#### RemoveResponseHeader

接受一个参数，name

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: removeresponseheader_route
        uri: https://example.org
        filters:
        - RemoveResponseHeader=X-Response-Foo
```

移除 response的 X-Response-Foo 头，在 response 返回 网关客户端前。

要移除 任何形式的 敏感头，你应该配置这个 filter for 所有route。   
另外，你可以只配置 这个filter 一次，通过 使用 spring.cloud.gateway.default-filters. 这样，这个filter会应用到所有route

---
#### RemoveRequestParameter

接受一个参数，name，是 将被移除的 query parameter的名字 。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: removerequestparameter_route
        uri: https://example.org
        filters:
        - RemoveRequestParameter=red
```
在 往下游发送前，会删除 red 查询参数。

---
#### RewritePath

接受一个 regexp参数，和一个replacement参数。   
用java RE 完成 请求路径的 重写。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: rewritepath_route
        uri: https://example.org
        predicates:
        - Path=/red/**
        filters:
        - RewritePath=/red/?(?<segment>.*), /$\{segment}
```

request /red/blue， 在 生成往下游发送的request 时，会变成 /path。   
$ 应该用 $\ 来代替，这是YAML的特例。

---
#### RewriteLocationResponseHeader

修改 response中的 Location 的值，通常是为了摆脱 后端的细节    
接受 stripVersionMode, locationHeaderName, hostValue, protocolsRegex 参数。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: rewritelocationresponseheader_route
        uri: http://example.org
        filters:
        - RewriteLocationResponseHeader=AS_IN_REQUEST, Location, ,
```

对于一个 POST api.example.com/some/object/name 的请求， 响应的header中有一个 Location:object-service.prod.example.net/v2/some/object/id, 会被重写为 api.example.com/some/object/id.   

stripVersionMode 支持 以下值：   
NEVER_STRIP: version 不会被 strip(撕掉)，即使 原始request path 不包含 version    
AS_IN_REQUEST: 默认， 当原始request path 不带version时， 会撕掉 version   
ALWAYS_STRIP: 始终撕掉 version   

hostValue 参数，如果提供了，就用来替换 response的 Location头的 host:port 部分。如果没有提供，就使用request的 Host头 的值。   

protocolsRegex,必须是一个 正则string，来匹配 protocol 名字。如果不匹配，不会做任何事情。默认是 http|https|ftp|ftps   

---
#### RewriteResponseHeader

<<<<<<< HEAD
接受3个参数，name，regexp，replacement。使用java RE 来重写 response header 值。   

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: rewriteresponseheader_route
        uri: https://example.org
        filters:
        - RewriteResponseHeader=X-Response-Red, , password=[^&]+, password=***
```

对于一个值为 /42?user=ford&password=omg!what&flag=true 的header，会被重写为 /42?user=ford&password=***&flag=true，在生成下有请求后   
需要使用 $\ 而不是 $

---
#### SaveSession

强制一个 WebSession::save 操作，在 转发到下游前。   
很有用，当你使用 诸如 Spring Session with a lazy data store， 你需要确保 session状态 被保存，在 调用下游前。   

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: save_session
        uri: https://example.org
        predicates:
        - Path=/foo/**
        filters:
        - SaveSession
```

如果你集成了 Spring Security with Spring Session，并且想要确保 security 数据转发到远程，这是很重要的。

---
#### SecureHeaders

增加几个头到response。   
```
X-Xss-Protection:1 (mode=block)

Strict-Transport-Security (max-age=631138519)

X-Frame-Options (DENY)

X-Content-Type-Options (nosniff)

Referrer-Policy (no-referrer)

Content-Security-Policy (default-src 'self' https:; font-src 'self' https: data:; img-src 'self' https: data:; object-src 'none'; script-src https:; style-src 'self' https: 'unsafe-inline)'

X-Download-Options (noopen)

X-Permitted-Cross-Domain-Policies (none)
```

要改变这些默认值，设置 spring.cloud.gateway.filter.secure-headers 中的属性:   
```
xss-protection-header

strict-transport-security

x-frame-options

x-content-type-options

referrer-policy

content-security-policy

x-download-options

x-permitted-cross-domain-policies
```

禁用默认值，通过 spring.cloud.gateway.filter.secure-headers.disable, 值是 逗号分隔的列表：   

spring.cloud.gateway.filter.secure-headers.disable=x-frame-options,strict-transport-security   

使用的是 小写的 secure header。

---
#### SetPath

接受一个参数 template.   
提供了一种简单的方法 来 操作 request path 通过 允许路径模板。    
这个使用了 URI templates from Spring Framework   
允许多个匹配segment   

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setpath_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment}
        filters:
        - SetPath=/{segment}
```

请求 /red/blue， 设置路径为 /blue，在创建访问下游的request前。   

---
#### SetRequestHeader

接受 name，value 参数   

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setrequestheader_route
        uri: https://example.org
        filters:
        - SetRequestHeader=X-Request-Red, Blue
```

替换(而不是增加) 所有 给出的 头。
=======
接受 name，regexp，replacement 参数，使用java RE 来重写response header 值。   

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: rewriteresponseheader_route
        uri: https://example.org
        filters:
        - RewriteResponseHeader=X-Response-Red, , password=[^&]+, password=***
```

对于个一个 值是 /42?user=ford&password=omg!what&flag=true 的header，在发送下游request 后，会被设置为 /42?user=ford&password=***&flag=true。   

需要使用 $\ 来代替 $   

---
#### SaveSession

强制一个 WebSession::save 操作，在 调用下游前。    
有用的：当使用类似 Spring Session with a lazy data store，你需要 确保session状态 被保存，在调用下游前。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: save_session
        uri: https://example.org
        predicates:
        - Path=/foo/**
        filters:
        - SaveSession
```

如果你集成 Spring Security with Spring Session. 并且想确保 security 明细 被转发到 远程处理，这很重要。

---
#### SecureHeaders

加几个header 到response。

```
X-Xss-Protection:1 (mode=block)
Strict-Transport-Security (max-age=631138519)
X-Frame-Options (DENY)
X-Content-Type-Options (nosniff)
Referrer-Policy (no-referrer)
Content-Security-Policy (default-src 'self' https:; font-src 'self' https: data:; img-src 'self' https: data:; object-src 'none'; script-src https:; style-src 'self' https: 'unsafe-inline)'
X-Download-Options (noopen)
X-Permitted-Cross-Domain-Policies (none)
```

要修改默认值，通过spring.cloud.gateway.filter.secure-headers.*   
```
xss-protection-header
strict-transport-security
x-frame-options
x-content-type-options
referrer-policy
content-security-policy
x-download-options
x-permitted-cross-domain-policies
```

spring.cloud.gateway.filter.secure-headers.disable 加上 逗号分隔的 需要 禁用的 属性 列表

```
spring.cloud.gateway.filter.secure-headers.disable=x-frame-options,strict-transport-security
```

禁用的时候，使用 全小写。

---
#### SetPath

接受一个 template 参数， 提供一个简单的方法来 操作 request path 通过 允许path 的 模板化。   
使用Spring框架的 URI template，多匹配段是可以的。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setpath_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment}
        filters:
        - SetPath=/{segment}
```

如果访问 /red/blue, 在往下游发送前，路径会变成 /blue

---
#### SetRequestHeader

接收 name 和 value 2个参数。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setrequestheader_route
        uri: https://example.org
        filters:
        - SetRequestHeader=X-Request-Red, Blue
```

替换 (不是add) 给定名字的所有header。   
如果有一个header： X-Request-Red:1234, 这个会被替换为 X-Request-Red:Blue，这个会是下游服务器收到的。

SetRequestHeader 知道 用来匹配path 或 host 的 URI变量。运行时，能使用这些URI 变量。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setrequestheader_route
        uri: https://example.org
        predicates:
        - Host: {segment}.myhost.org
        filters:
        - SetRequestHeader=foo, bar-{segment}
```

---
#### SetResponseHeader

接受 name，value 2个参数

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setresponseheader_route
        uri: https://example.org
        filters:
        - SetResponseHeader=X-Response-Red, Blue
```

替换(不是add) 给定名字的 所有header。   
如果下游返回一个header，X-Response-Red:1234， 这个会被替换为 X-Response-Red:Blue， 这个是网关客户端收到的。

SetResponseHeader 知道 匹配path 或 host 的 URI变量。 能在运行时使用。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setresponseheader_route
        uri: https://example.org
        predicates:
        - Host: {segment}.myhost.org
        filters:
        - SetResponseHeader=foo, bar-{segment}
```

---
#### SetStatus

接受一个参数，status。必须是一个合法的Spring的 HttpStatus，可以是 数字404 或 字符串 NOT_FOUND。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setstatusstring_route
        uri: https://example.org
        filters:
        - SetStatus=BAD_REQUEST
      - id: setstatusint_route
        uri: https://example.org
        filters:
        - SetStatus=401
```

你可以配置SetStatus 来返回 原始HTTP状态码 来自 被代理request 在response的一个header里。   
下面的配置，增加一个header到response。   

```yaml
spring:
  cloud:
    gateway:
      set-status:
        original-status-header-name: original-http-status
```

---
#### StripPrefix

一个参数，parts。表示在发送request到下游钱，从request中剥离的 part 的数量。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: nameRoot
        uri: https://nameservice
        predicates:
        - Path=/name/**
        filters:
        - StripPrefix=2
```

当request 通过网关 到 /name/blue/red，发送给nameservice 的 request 看起来像 nameservice/red

---
#### Retry

支持下面的参数：   
```
retries, 尝试多少次retry   
statuses, 需要retry的HTTP状态码，使用org.springframework.http.HttpStatus 表示    
methods, 需要retry的 HTTP method。使用 org.springframework.http.HttpMethod 表示    
series, 需要retry的 状态码系列，使用 org.springframework.http.HttpStatus.Series 表示( 类似1xx,2xx,3xx,4xx,5xx)    
exceptions, 需要retry的 异常 列表    
backoff, 配置 retry的 指数 backoff。 retry的执行是 间隔 firstBackoff * (factor ^ n)， n代表第几次。如果配置了 maxBackoff,最大backoff 会被限制到 maxBackoff。如果 basedOnPreviousValue 是true，backoff 会通过 prevBackOff * factor 来计算。   
```

下面是Retry的 默认配置，如果起效：
```
retries: Three times
series: 5XX series
methods: GET method
exceptions: IOException and TimeoutException
backoff: disabled
```

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: retry_test
        uri: http://localhost:8080/flakey
        predicates:
        - Host=*.retry.com
        filters:
        - name: Retry
          args:
            retries: 3
            statuses: BAD_GATEWAY
            methods: GET,POST
            backoff:
              firstBackoff: 10ms
              maxBackoff: 50ms
              factor: 2
              basedOnPreviousValue: false
```

当使用 Retry with forward: 前缀的 URL，目标端点 需要仔细编写，因为如果出现错误，它不会做任何可能导致response被发送到client并commit的事情。   
例如，如果 目标端点是一个 注解形式的Controller，目标controller方法 不应该return 带有 错误状态码的 ResponseEntity。它应该抛出一场，或 signal an error (例如，通过Mono.error(ex) 返回值)，这样Retry能按照配置的进行处理。

当使用 Retry 处理 有body的 任何HTTP method，body会被cache，网关会变得内存受限。   
body被缓存在 request attribute： ServerWebExchangeUtils.CACHED_REQUEST_BODY_ATTR 这个里。 对象的类型是 Spring框架的 DataBuffer。

---
#### RequestSize

当request size 大于 限定，RequestSize 可以限制请求 到达下游服务。   
一个参数 maxSize， 是一个 DataSize 类型。所以可以定义为  数字 + 数据单元(DataUnit)的后缀(如 KB，MB)。 默认是B (byte)。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: request_size_route
        uri: http://localhost:8080/upload
        predicates:
        - Path=/upload
        filters:
        - name: RequestSize
          args:
            maxSize: 5000000
```

当由于大小而拒绝request时，设置 response 的状态是 413 Payload Too Large, 以及一个额外的头：errorMessage。   
一个errorMessage的例子：
```yaml
errorMessage : Request size is larger than permissible limit. Request size is 6.0 MB where permissible limit is 5.0 MB
```

默认request size 是 5MB， 如果route 定义中没有提供 filter。


---
#### SetRequestHostHeader

有时 需要覆盖host 这个header。这种情况下，SetRequestHostHeader 能替换 已存在的 host 这个header with 一个特定值。   
一个参数 host    

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: set_request_host_header_route
        uri: http://localhost:8080/headers
        predicates:
        - Path=/headers
        filters:
        - name: SetRequestHostHeader
          args:
            host: example.org
```

替换 host 这个header 的值 为 example.org

---
#### Modify a Request Body GatewayFilter Factory

使用 ModifyRequestBody 来修改 request body 在通过网关 发送到下游前。

只能使用 Java DSL 来配置。

```
@Bean
public RouteLocator routes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("rewrite_request_obj", r -> r.host("*.rewriterequestobj.org")
            .filters(f -> f.prefixPath("/httpbin")
                .modifyRequestBody(String.class, Hello.class, MediaType.APPLICATION_JSON_VALUE,
                    (exchange, s) -> return Mono.just(new Hello(s.toUpperCase())))).uri(uri))
        .build();
}
static class Hello {
    String message;
    public Hello() { }
    public Hello(String message) {
        this.message = message;
    }
    public String getMessage() {
        return message;
    }
    public void setMessage(String message) {
        this.message = message;
    }
}
```

如果request没有body，RewriteFilter 会被传入一个null，Mono.empty() 应该被返回 来表示 request里没有 body

---
#### Modify a Response Body GatewayFilter Factory

使用 ModifyResponseBody 来修改 response body 在 它返回给客户端前。

只能通过 Java DSL 配置

```
@Bean
public RouteLocator routes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("rewrite_response_upper", r -> r.host("*.rewriteresponseupper.org")
            .filters(f -> f.prefixPath("/httpbin")
                .modifyResponseBody(String.class, String.class,
                    (exchange, s) -> Mono.just(s.toUpperCase()))).uri(uri))
        .build();
}
```

如果Response 没有body， RewriteFilter 会被传入一个null，Mono.empty() 应该被返回，来表示 response没有 body。

---
#### Token Relay GatewayFilter Factory

Token Relay (令牌中继) 是 OAuth2消费者 充当客户端并将传入令牌转发到传出资源请求的地方。   
消费者可以是 纯客户端(比如SSO应用) 或一个 资源服务器

S C Gateway能 将OAuth2 访问令牌下游 转发到 它正在代理的服务。要增加这个功能到Gateway，你需要增加 TokenRelayGatewayFilterFactory，就像下面：

```
@Bean
public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
    return builder.routes()
            .route("resource", r -> r.path("/resource")
                    .filters(f -> f.tokenRelay())
                    .uri("http://localhost:9000"))
            .build();
}
```

或

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: resource
        uri: http://localhost:9000
        predicates:
        - Path=/resource
        filters:
        - TokenRelay=
```

它会 (还会登录用户并获取令牌) 传递 身份令牌 到下游服务 (这里是 /resource)

在Gateway激活这个令牌中继功能，需要增加下列依赖： org.springframework.boot:spring-boot-starter-oauth2-client   

这个如何工作？ filter 提取 访问令牌 从 当前已认证的user，把这个放在 发送到下游的 request 的header 中。   

TokenRelayGatewayFilterFactory 被创建，只有当 恰当的 spring.security.oauth2.client.* 属性被设置 (这些属性会 触发 创建一个 ReactiveClientRegistrationRepository bean)。   

TokenRelayGatewayFilterFactory 使用的 ReactiveOAuth2AuthorizedClientService 的默认实现 使用了 内存数据存储。如果你需要一个更健壮的方案，你需要自己实现 ReactiveOAuth2AuthorizedClientService.   

---
#### Default Filters

要增加一个filter，并且应用这个filter到所有的 route。 你可以使用 spring.cloud.gateway.default-filters. 这个属性接受 filter的列表。   
```yaml
spring:
  cloud:
    gateway:
      default-filters:
      - AddResponseHeader=X-Response-Default-Red, Default-Blue
      - PrefixPath=/httpbin
```

---
---

#### Global Filters

GlobalFilter 接口签名 和 GatewayFilter 一致。   

这里有些 filter 按条件应用到 所有 route

> 在未来，此界面及用法可能发生变化。

---
#### Combined Global Filter and GatewayFilter Ordering

当一个request 匹配一个 route， filtering web handler 会增加 GlobalFilter 的所有实例 和 这个route的 所有 GatewayFilter 实例 到 一个 filter chain。   
这个合并后的 filter chain 排序，按照 org.springframework.core.Ordered 接口，你可以通过 实现getOrder()方法来设置。   

S C Gateway 对于 filter 逻辑执行 区分为 pre 和 post 2块。 优先级高的 在pre中 先执行，在post中 后执行。

```
@Bean
public GlobalFilter customFilter() {
    return new CustomGlobalFilter();
}

public class CustomGlobalFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("custom global filter");
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return -1;
    }
}
```

---
#### Forward Routing Filter

搜索URI 在 exchange attribute ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR中。   
如果 URL 是forward 格式 (如 forward://localendpoint ), 它使用Spring的 DispatcherHandler 来处理这个请求。   
request URL的 path部分 被 forward URL的path 覆盖。   
未修改的原URL 增加到 ServerWebExchangeUtils.GATEWAY_ORIGINAL_REQUEST_URL_ATTR 这个list中。   

---
#### The LoadBalancerClient Filter

搜索 URI， 在名为 ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR 的 exchange attribute 中。   
如果URL 是 lb格式 (如 lb://myservice), 使用Spring Cloud LoadBalancerClient 来解析 名字(这里是myservice)，获得一个真实 host和端口，来替换URI中的 host，端口。   
未修改的原URL 添加在 ServerWebExchangeUtils.GATEWAY_ORIGINAL_REQUEST_URL_ATTR 这个list中。   
这个filter 也搜索 ServerWebExchangeUtils.GATEWAY_SCHEME_PREFIX_ATTR 属性来看 它是否等于 lb。如果等于，也会执行上面的规则。   

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: myRoute
        uri: lb://service
        predicates:
        - Path=/service/**
```

默认下，如果负载均衡找不到服务实例，返回503. 你可以让Gateway返回404,通过 spring.cloud.gateway.loadbalancer.use404=true   

负载均衡返回的 服务实例 的 isSecure 值 会覆盖 发送到Gateway的 request 定义的 scheme。   
例如，如果 发送到Gateway的 是HTTPS，但是 服务实例 表明它是不 secure的，发送给 下游的请求会是 HTTP。 相反的情况也适用。   
然而，如果 在Gateway的配置 中 为route 指明了 GATEWAY_SCHEME_PREFIX_ATTR，前缀会被丢弃，并且来自route URL的 scheme 覆盖了 ServiceInstance 配置。

。。它的LB 是怎么配置的？ 怎么警用这个filter？

---
#### ReactiveLoadBalancerClientFilter

搜索URI 在 名为 ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR 的exchange attribute中。   
如果URL 是 lb scheme(如 lb://myservice), 使用 S C ReactorLoadBalancer 来 检索名字(这里是myservice) 来获得 真实的host 和 port，并且替换URI中的host port。   
修改前的 原URL 被append 到 ServerWebExchangeUtils.GATEWAY_ORIGINAL_REQUEST_URL_ATTR 这个list。   
filter也搜索 ServerWebExchangeUtils.GATEWAY_SCHEME_PREFIX_ATTR 属性 来判断是否是 lb。如果是lb，也执行上面的规则

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: myRoute
        uri: lb://service
        predicates:
        - Path=/service/**
```

默认下，如果 ReactorLoadBalancer 找不到 服务实例，返回503, 可以返回404通过 spring.cloud.gateway.loadbalancer.use404=true   

ReactiveLoadBalancerClientFilter 返回的 服务实例的 isSecure 值 会覆盖 发送给Gateway的request的 scheme。   
如，如果发送到Gateway的 request 是HTTPS，但是 服务实例 表示它是不 secure，那么发送给下游的request是HTTP。 反之也是。   
如果网关配置中为route制定了 GATEWAY_SCHEME_PREFIX_ATTR，则 前缀将被剥离，并且 来自route URL的 scheme将覆盖 服务实例的配置   

---
#### The Netty Routing Filter

Netty routing filter 执行， 如果 位于 ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR exchange attribute 的 URL有一个 http 或 https scheme。   
它使用Netty的 HttpClient 来 发出下游代理请求。   
response 被放到 ServerWebExchangeUtils.CLIENT_RESPONSE_ATTR exchange attribute， 以便后续filter 使用。   

也有一个实验性的 WebClientHttpRoutingFilter，执行相同的功能，但是不使用 Netty

---
#### The Netty Write Response Filter

NettyWriteResponseFilter 执行，如果 ServerWebExchangeUtils.CLIENT_RESPONSE_ATTR 中有一个 Netty的 HttpClientResponse。   
它在 所有其他filter 执行完后 执行，并且 将 proxy response 写回 网关客户端response。   

也有一个实验性的 WebClientWriteResponseFilter 完成相同的功能，但不需要 Netty。

---
#### RouteToRequestUrl

如果 ServerWebExchangeUtils.GATEWAY_ROUTE_ATTR exchange attribute 有 一个 Route 对象， RouteToRequestUrlFilter 执行。   
它会创建一个新URI，基于 request URI 但是 使用 Route 对象的 URI attribute 进行更新。    
新URI 放到 ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR exchange attribute。   

如果 URI 有 scheme 前缀，如 lb:ws://serviceid, lb scheme 会从URI中 剥离，并放到 ServerWebExchangeUtils.GATEWAY_SCHEME_PREFIX_ATTR 供 filter链中后续的filter 使用。   

---
#### The Websocket Routing Filter

如果 在 ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR exchange attribute 中的 URL 有一个 ws 或 wss scheme，那么 websocket routing filter 执行。
它使用Spring WebSocket 来转发 websocket 请求 到 下游

可以使用 load-balance websockets 通过 为URI 增加前缀 lb，如 lb:ws://serviceid   

如果你使用SockJS 作为 fallback over 普通HTTP, 你应该配置一个 普通HTTP route 和一个 websocket route。

```yaml
spring:
  cloud:
    gateway:
      routes:
      # SockJS route
      - id: websocket_sockjs_route
        uri: http://localhost:3001
        predicates:
        - Path=/websocket/info/**
      # Normal Websocket route
      - id: websocket_route
        uri: ws://localhost:3001
        predicates:
        - Path=/websocket/**
```

---
#### The Gateway Metrics Filter

要激活 gateway metrics, 增加 spring-boot-starter-actuator 依赖。默认下，gateway metrics filter 执行，只要 spring.cloud.gateway.metrics.enabled 没有被设置为false。   
这个filter 增加 一个名为 gateway.requests 的计时器指标，并带有以下标签：    
```
routeId: the route id    
routeUri: api route 的 URL   
outcome: 结果(outcome), 由 HttpStatus.Series 分类   
status: 返回给客户端的 http status   
httpStatusCode: 返回给客户端的 http status   
httpMethod: request的 http method 
```

可以从 /actuator/metrics/gateway.requests 抓取这些 metric，能被简单地 集成到 Prometheus 来创建 Grafana dashboard   

要激活 Prometheus 端点， 增加 micrometer-registry-prometheus 依赖   

---
#### Marking An Exchange As Routed

在 gateway route 一个 ServerWebExchange后，它标记这个 exchange 是 "routed" 通过增加 gatewayAlreadyRouted 到 exchange attributes   

一旦一个请求 被标记为 routed，其他routing filter 不会 再 route它， 本质上是跳过filter。   

有一些简单方法 用来 标记exchange 是routed 或者检查 exchange 是否是 routed

ServerWebExchangeUtils.isAlreadyRouted, 接受一个ServerWebExchange 对象 并检查 是否 routed

ServerWebExchangeUtils.setAlreadyRouted， 接受 ServerWebExchange，标记为 routed

---
---
#### HttpHeadersFilters

应用到request，在发送下游前，比如 NettyRoutingFilter。

---
#### Forward Headers Filter

创建一个Forwarded 头。它增加 当前request的 host头，scheme，port 到 任何已存在的 Forwarded 头。

---
#### RemoveHopByHop Headers Filter

从 forwarded request 移除 header。 默认会移除下面的头：
```
Connection
Keep-Alive
Proxy-Authenticate
Proxy-Authorization
TE
Trailer
Transfer-Encoding
Upgrade
```

spring.cloud.gateway.filter.remove-hop-by-hop.remove-hop-by-hop.headers 设置需要移除的 头 列表。

---
#### XForwarded Headers Filter

创建 X-Forwarded-* 头 发送到 下游服务。 它使用 当前request的 Host header，scheme，port和path 来创建 头。   

下面的 boolean 属性 来控制 是否 创建头，默认 true
```
spring.cloud.gateway.x-forwarded.for-enabled
spring.cloud.gateway.x-forwarded.host-enabled
spring.cloud.gateway.x-forwarded.port-enabled
spring.cloud.gateway.x-forwarded.proto-enabled
spring.cloud.gateway.x-forwarded.prefix-enabled
```

下面的 boolean 属性来控制是否 append 到头 ，默认 true
```
spring.cloud.gateway.x-forwarded.for-append
spring.cloud.gateway.x-forwarded.host-append
spring.cloud.gateway.x-forwarded.port-append
spring.cloud.gateway.x-forwarded.proto-append
spring.cloud.gateway.x-forwarded.prefix-append
```

---
#### TLS and SSL

Gateway 能监听 HTTPS 的请求，通过 遵循常规Spring server 配置。下面展现了如何做：

application.yml
```yaml
server:
  ssl:
    enabled: true
    key-alias: scg
    key-store-password: scg1234
    key-store: classpath:scg-keystore.p12
    key-store-type: PKCS12
```

你可以 route gateway routes to HTTP 和 HTTPS 后端。   
如果你 route 到一个 HTTPS 后端，你可以配置 gateway 来信任 所有的 下游证书

```yaml
spring:
  cloud:
    gateway:
      httpclient:
        ssl:
          useInsecureTrustManager: true
```
使用一个 insecure trust manager 不适用于 生产。   
对于 生产环境，你可以配置 一系列 信任的 证书：
```yaml
spring:
  cloud:
    gateway:
      httpclient:
        ssl:
          trustedX509Certificates:
          - cert1.pem
          - cert2.pem
```

如果 S C Gateway 没有被提供 受信任的证书，默认的 trust store 被使用。 可以通过 设置 javax.net.ssl.trustStore 系统变量 来修改。

---
#### TLS Handshake

Gateway 维持一个 client pool，用来 route 到后端。   
当通过 HTTPS 通信时，客户端初始化一个 TLS 握手。这个握手有超时限制。你可以配置：(下面的值是默认值)    

```yaml
spring:
  cloud:
    gateway:
      httpclient:
        ssl:
          handshake-timeout-millis: 10000
          close-notify-flush-timeout-millis: 3000
          close-notify-read-timeout-millis: 0
```

---
#### Configuration

SpringCloud Gateway的 配置 被 一群 RouteDefinitionLocator 实例 驱动。   
下面是 RouteDefinitionLocator 的接口定义
```java
public interface RouteDefinitionLocator {
    Flux<RouteDefinition> getRouteDefinitions();
}
```

默认，PropertiesRouteDefinitionLocator 加载属性 通过 使用 SpringBoot 的 @ConfigurationProperties 机制。    

之前的配置例子，都使用了 缩写，使用位置而不是名字 来定位参数。   

下面2个是等价的
```
spring:
  cloud:
    gateway:
      routes:
      - id: setstatus_route
        uri: https://example.org
        filters:
        - name: SetStatus
          args:
            status: 401
      - id: setstatusshortcut_route
        uri: https://example.org
        filters:
        - SetStatus=401
```

对于 gateway的 某些用途，properties 足够了。   
但是有时，生产 需要 从外部source (比如数据库) 加载配置。未来，会使用 基于 Spring Data Repositories (比如Redis，MongoDB，Cassandra) 的 RouteDefinitionLocator 实现

---
#### Route Metadata Configuration

你可以 为每个route 配置 额外的参数，通过 metadata ：

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: route_with_metadata
        uri: https://example.org
        metadata:
          optionName: "OptionValue"
          compositeObject:
            name: "value"
          iAmNumber: 1
```

可以获得这些 metadata ：
```
Route route = exchange.getAttribute(GATEWAY_ROUTE_ATTR);
// get all metadata properties
route.getMetadata();
// get a single metadata property
route.getMetadata(someKey);
```

---
#### Http timeouts configuration

Http timeouts (response and connect) 能被 配置给 所有的route，并且 每个route可以有自己的定义(覆盖全局的)。

---
#### Global timeouts

单位是： 毫秒，java.time.Duration

```yaml
spring:
  cloud:
    gateway:
      httpclient:
        connect-timeout: 1000
        response-timeout: 5s
```

---
#### Per-route timeouts

都是毫秒

```
      - id: per_route_timeouts
        uri: https://example.org
        predicates:
          - name: Path
            args:
              pattern: /delay/{timeout}
        metadata:
          response-timeout: 200
          connect-timeout: 200
```

java 为单独 route 配置 timeout。
```
import static org.springframework.cloud.gateway.support.RouteMetadataUtils.CONNECT_TIMEOUT_ATTR;
import static org.springframework.cloud.gateway.support.RouteMetadataUtils.RESPONSE_TIMEOUT_ATTR;

      @Bean
      public RouteLocator customRouteLocator(RouteLocatorBuilder routeBuilder){
         return routeBuilder.routes()
               .route("test1", r -> {
                  return r.host("*.somehost.org").and().path("/somepath")
                        .filters(f -> f.addRequestHeader("header1", "header-value-1"))
                        .uri("http://someuri")
                        .metadata(RESPONSE_TIMEOUT_ATTR, 200)
                        .metadata(CONNECT_TIMEOUT_ATTR, 200);
               })
               .build();
      }
```

---
#### Fluent Java Routes API

为了允许 用Java的方式 进行简单的配置， RouteLocatorBuilder bean有一个 API

```
// static imports from GatewayFilters and RoutePredicates
@Bean
public RouteLocator customRouteLocator(RouteLocatorBuilder builder, ThrottleGatewayFilterFactory throttle) {
    return builder.routes()
            .route(r -> r.host("**.abc.org").and().path("/image/png")
                .filters(f ->
                        f.addResponseHeader("X-TestHeader", "foobar"))
                .uri("http://httpbin.org:80")
            )
            .route(r -> r.path("/image/webp")
                .filters(f ->
                        f.addResponseHeader("X-AnotherHeader", "baz"))
                .uri("http://httpbin.org:80")
                .metadata("key", "value")
            )
            .route(r -> r.order(-1)
                .host("**.throttle.org").and().path("/get")
                .filters(f -> f.filter(throttle.apply(1,
                        1,
                        10,
                        TimeUnit.SECONDS)))
                .uri("http://httpbin.org:80")
                .metadata("key", "value")
            )
            .build();
}
```

这种方式 允许 更自定义的谓词断言。通过使用 Java API，你可以使用 and(),or(),negate() 操作 来组合 Predicate类对象。

---
#### The DiscoveryClient Route Definition Locator

你可以配置 gateway 来 创建 基于 向 DiscoveryClient兼容的 服务注册中心注册的 服务 的 route。

激活，spring.cloud.gateway.discovery.locator.enabled=true， 并确保 有 DiscoveryClient实现类(如 Eureka，Consul，Zookeeper) 在 classpath并且被激活着 

---
#### Configuring Predicates and Filters For DiscoveryClient Routes

默认，gateway 定义了一个 单独的predicate 和 filter for 使用DiscoveryClient创建的 route。   

默认predicate是 定义了/serviceId/**模式的 path predicate， serviceId 是来自DiscoveryClient的 服务的ID。

默认filter 是 使用正则 /serviceId/?(?<remaining>.*) 及 替代品/${remaining} 的 rewrite path filter。这个从path 剥离service ID，在发给下游前

如果你要自定义 DiscoveryClient route使用的 predicate或 filter。设置 spring.cloud.gateway.discovery.locator.predicates[x], spring.cloud.gateway.discovery.locator.filters[y]   
这么做时，如果你想要使用前面的默认谓词和filter的功能，你需要确保包含了 前面的默认 谓词和filter。

```properties
spring.cloud.gateway.discovery.locator.predicates[0].name: Path
spring.cloud.gateway.discovery.locator.predicates[0].args[pattern]: "'/'+serviceId+'/**'"
spring.cloud.gateway.discovery.locator.predicates[1].name: Host
spring.cloud.gateway.discovery.locator.predicates[1].args[pattern]: "'**.foo.com'"
spring.cloud.gateway.discovery.locator.filters[0].name: CircuitBreaker
spring.cloud.gateway.discovery.locator.filters[0].args[name]: serviceId
spring.cloud.gateway.discovery.locator.filters[1].name: RewritePath
spring.cloud.gateway.discovery.locator.filters[1].args[regexp]: "'/' + serviceId + '/?(?<remaining>.*)'"
spring.cloud.gateway.discovery.locator.filters[1].args[replacement]: "'/${remaining}'"
```

---
#### Reactor Netty Access Logs

要激活 Reactor Netty access logs，设置 -Dreactor.netty.http.server.accessLogEnabled=true   
这必须是一个 Java 系统属性， 不能是 SpringBoot property。

你可以配置 日志系统 来获得一个 单独的 access log 文件。
```
    <appender name="accessLog" class="ch.qos.logback.core.FileAppender">
        <file>access_log.log</file>
        <encoder>
            <pattern>%msg%n</pattern>
        </encoder>
    </appender>
    <appender name="async" class="ch.qos.logback.classic.AsyncAppender">
        <appender-ref ref="accessLog" />
    </appender>

    <logger name="reactor.netty.http.server.AccessLog" level="INFO" additivity="false">
        <appender-ref ref="async"/>
    </logger>
```

---
#### CORS(跨域资源共享) Configuration

配置gateway 来控制 CORS 行为，"global" CORS 配置是 一个 map，URL pattern 映射到 SpringFramework 的 CorsConfiguration    

```yaml
spring:
  cloud:
    gateway:
      globalcors:
        cors-configurations:
          '[/**]':
            allowedOrigins: "https://docs.spring.io"
            allowedMethods:
            - GET
```

在上面的例子中，来自 docs.spring.io的 且是 GET 请求的 CORS请求 被允许。   

要提供相同的 CORS 配置 到 某些 没有被gateway route谓词处理的 请求， 设置 spring.cloud.gateway.globalcors.add-to-simple-url-handler-mapping 为true。   
这是有用的，当你尝试支持 CORS 预检请求，但是 由于 HTTP method 是 options，你的 route predicate没有计算出true，

---
#### Actuator API

/gateway actuator 端点 使得你监控和操作 S C Gateway 应用。   
为了让远程能访问，endpoint 需要通过 应用属性 来 激活 且 通过 HTTP或 JMX 暴露：
```properties
management.endpoint.gateway.enabled=true # default value
management.endpoints.web.exposure.include=gateway
```

---
#### Verbose Actuator Format

新的，更冗长的 格式 被添加到 SpringCloudGateway。   
它增加了更多的细节 到 每个route，使你能看到 每个route 的 predicate 和filter，以及任何可用的配置。   
下面是 /actuator/gateway/routes 的一个例子：
```
[
  {
    "predicate": "(Hosts: [**.addrequestheader.org] && Paths: [/headers], match trailing slash: true)",
    "route_id": "add_request_header_test",
    "filters": [
      "[[AddResponseHeader X-Response-Default-Foo = 'Default-Bar'], order = 1]",
      "[[AddRequestHeader X-Request-Foo = 'Bar'], order = 1]",
      "[[PrefixPath prefix = '/httpbin'], order = 2]"
    ],
    "uri": "lb://testservice",
    "order": 0
  }
]
```

默认是激活的; 禁用：spring.cloud.gateway.actuator.verbose.enabled=false

---
#### Retrieving Route Filters

这里会描述 如何 检索 route filter. 包括 Global Filter 和 Gateway Route Filter

---
#### Global Filters

要检索应用到所有route 的 global filter， 发送一个 GET 请求 到 /actuator/gateway/globalfilters.   
响应类似：
```
{
  "org.springframework.cloud.gateway.filter.LoadBalancerClientFilter@77856cc5": 10100,
  "org.springframework.cloud.gateway.filter.RouteToRequestUrlFilter@4f6fd101": 10000,
  "org.springframework.cloud.gateway.filter.NettyWriteResponseFilter@32d22650": -1,
  "org.springframework.cloud.gateway.filter.ForwardRoutingFilter@106459d9": 2147483647,
  "org.springframework.cloud.gateway.filter.NettyRoutingFilter@1fbd5e0": 2147483647,
  "org.springframework.cloud.gateway.filter.ForwardPathFilter@33a71d23": 0,
  "org.springframework.cloud.gateway.filter.AdaptCachedBodyGlobalFilter@135064ea": 2147483637,
  "org.springframework.cloud.gateway.filter.WebsocketRoutingFilter@23c05889": 2147483646
}
```

包含了 global filer 的明细，对于每个 global filter,有一个string 形式的 filter object 和 对应的order。

---
#### Route Filters

为了检索 应用到 route的 GatewayFilter factories， 发送 GET 请求到 /actuator/gateway/routefilters    
响应类似：
```
{
  "[AddRequestHeaderGatewayFilterFactory@570ed9c configClass = AbstractNameValueGatewayFilterFactory.NameValueConfig]": null,
  "[SecureHeadersGatewayFilterFactory@fceab5d configClass = Object]": null,
  "[SaveSessionGatewayFilterFactory@4449b273 configClass = Object]": null
}
```

响应包含了 应用到 任何特定route的 GatewayFilter factories 的明细。每个factory 是一个 string形式 对象。   
null 是因为 端点controller 是一个不完整的实现，因为 它尝试设置 filter chain中对象的 order，这个不适用于 GatewayFilter factory 对象。

---
#### Refreshing the Route Cache

要清空 route cache，发送一个POST 到 /actuator/gateway/refresh。 会返回一个200,不带body。

---
#### Retrieving the Routes Defined in the Gateway

为了检索 定义在gateway 中的 route，发送一个GET 请求到 /actuator/gateway/routes。   
响应类似：
```
[{
  "route_id": "first_route",
  "route_object": {
    "predicate": "org.springframework.cloud.gateway.handler.predicate.PathRoutePredicateFactory$$Lambda$432/1736826640@1e9d7e7d",
    "filters": [
      "OrderedGatewayFilter{delegate=org.springframework.cloud.gateway.filter.factory.PreserveHostHeaderGatewayFilterFactory$$Lambda$436/674480275@6631ef72, order=0}"
    ]
  },
  "order": 0
},
{
  "route_id": "second_route",
  "route_object": {
    "predicate": "org.springframework.cloud.gateway.handler.predicate.PathRoutePredicateFactory$$Lambda$432/1736826640@cd8d298",
    "filters": []
  },
  "order": 0
}]
```
响应 包含定义在gateway中的所有route的 详情。   
下面的表 描述了 response中 每个route元素的 结构

|Path|Type|Description|
|:---|:---|:---|
|route_id|String|route id|
|route_object.predicate|Object|route 的 predicate|
|route_object.filters|Array|应用到这个route的 GatewayFilter factories|
|order|Number|route 次序|


---
#### Retrieving Information about a Particular Route

要获得 某个route 的信息，发送 GET请求到 /actuator/gateway/routes/{id}   
响应类似：
```
{
  "id": "first_route",
  "predicates": [{
    "name": "Path",
    "args": {"_genkey_0":"/first"}
  }],
  "filters": [],
  "uri": "https://www.uri-destination.org",
  "order": 0
}
```

下面是响应的结构

|Path|Type|Description|
|:---|:---|:---|
|id|String|route id|
|predicates|Array|route谓词的集合，每项定义了给定谓词的名字和参数|
|filters|Array|应用到这个route的 filter集合|
|uri|String|route的目标URI|
|order|Number|route 次序|


---
#### Creating and Deleting a Particular Route

create route, 发送 POST 到 /gateway/routes/{id_route_to_create} with JSON body。   
delete， 发送 DELETE 到 /gateway/routes/{id_route_to_delete}   

---
#### Recap: The List of All endpoints

下表描述了 S C Gateway actuator 端点 (每个端点都是 /actuator/gateway 前缀)

|ID|HTTP Method|Description|
|:---|:---|:---|
|globalfilters|GET|展示应用到route的 global filter 列表|
|routefilters|GET|展示应用到每个route的GatewayFilter factories列表|
|refresh|POST|清除route cache|
|routes|GET|展示定义在gateway中的 route列表|
|routes/{id}|GET|展示特定route的信息|
|routes/{id}|POST|增加一个新route 到gateway|
|routes/{id}|DELETE|删除一个route|

---
#### Troubleshooting

这节覆盖了 你在使用 Gateway 中会碰到的 普通问题。

---
#### Log Levels

下面的logger 可能包含有价值的 故障排除信息，在 DEBUG，TRACE 级别
```
org.springframework.cloud.gateway
org.springframework.http.server.reactive
org.springframework.web.reactive
org.springframework.boot.autoconfigure.web
reactor.netty
redisratelimiter
```

---
#### Wiretap(窃听)

Reactor Netty 的 HttpClient 和 HttpServer 能被窃听。   

当把reactor.netty日志级别设置为 debug或trace时，可以打印信息，比如 发送和接受到的 header，body。   

要激活窃听，设置 spring.cloud.gateway.httpserver.wiretap=true，spring.cloud.gateway.httpclient.wiretap=true 分别为了 HttpServer 和 HttpClient。

---
#### Developer Guide

这个是基础手册，用来写 gateway的自定义组件

---
#### Writing Custom Route Predicate Factories

要写一个 route predicate，你需要实现 RoutePredicateFactory 接口。   
有一个 抽象类 AbstractRoutePredicateFactory 你可以扩展。   

```java
public class MyRoutePredicateFactory extends AbstractRoutePredicateFactory<HeaderRoutePredicateFactory.Config> {

    public MyRoutePredicateFactory() {
        super(Config.class);
    }

    @Override
    public Predicate<ServerWebExchange> apply(Config config) {
        // grab configuration from Config object
        return exchange -> {
            //grab the request
            ServerHttpRequest request = exchange.getRequest();
            //take information from the request to see if it
            //matches configuration.
            return matches(config, request);
        };
    }

    public static class Config {
        //Put the configuration properties for your filter here
    }

}
```

---
#### Writing Custom GatewayFilter Factories

需要实现 GatewayFilterFactory 接口， 你可以扩展 AbstractGatewayFilterFactory。

```java
public class PreGatewayFilterFactory extends AbstractGatewayFilterFactory<PreGatewayFilterFactory.Config> {

    public PreGatewayFilterFactory() {
        super(Config.class);
    }

    @Override
    public GatewayFilter apply(Config config) {
        // grab configuration from Config object
        return (exchange, chain) -> {
            //If you want to build a "pre" filter you need to manipulate the
            //request before calling chain.filter
            ServerHttpRequest.Builder builder = exchange.getRequest().mutate();
            //use builder to manipulate the request
            return chain.filter(exchange.mutate().request(builder.build()).build());
        };
    }

    public static class Config {
        //Put the configuration properties for your filter here
    }

}
```
>>>>>>> refs/remotes/origin/main

上面pre，下面post

```java
public class PostGatewayFilterFactory extends AbstractGatewayFilterFactory<PostGatewayFilterFactory.Config> {

    public PostGatewayFilterFactory() {
        super(Config.class);
    }

    @Override
    public GatewayFilter apply(Config config) {
        // grab configuration from Config object
        return (exchange, chain) -> {
            return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                ServerHttpResponse response = exchange.getResponse();
                //Manipulate the response in some way
            }));
        };
    }

    public static class Config {
        //Put the configuration properties for your filter here
    }

}
```

---
#### Naming Custom Filters And References In Configuration

自定义filter 的类名 应该以 GatewayFilterFactory 结束。

不以GatewayFilterFactory 结尾，也可以在配置文件中 被 ref。 但是这是不推荐的命名规则，并且未来可能移除。

---
#### Writing Custom Global Filters

实现 GlobalFilter 接口，会应用这个filter到所有request。

```java
@Bean
public GlobalFilter customGlobalFilter() {
    return (exchange, chain) -> exchange.getPrincipal()
        .map(Principal::getName)
        .defaultIfEmpty("Default User")
        .map(userName -> {
          //adds header to proxied request
          exchange.getRequest().mutate().header("CUSTOM-REQUEST-HEADER", userName).build();
          return exchange;
        })
        .flatMap(chain::filter);
}

@Bean
public GlobalFilter customGlobalPostFilter() {
    return (exchange, chain) -> chain.filter(exchange)
        .then(Mono.just(exchange))
        .map(serverWebExchange -> {
          //adds header to response
          serverWebExchange.getResponse().getHeaders().set("CUSTOM-RESPONSE-HEADER",
              HttpStatus.OK.equals(serverWebExchange.getResponse().getStatusCode()) ? "It worked": "It did not work");
          return serverWebExchange;
        })
        .then();
}
```

---
#### Building a Simple Gateway by Using Spring MVC or Webflux

下面描述了 另一种风格的 gateway。 
> 先前的文档不适用于下面的内容。

Gateway 提供了一个工具类：ProxyExchange。   
你可以使用它作为方法参数 在一个常规Spring web handler 中。   
它通过反映HTTP动词的方法 支持基本的下游HTTP交换。。(估计是指，里面有get()，post()方法)。。It supports basic downstream HTTP exchanges through methods that mirror the HTTP verbs.   
在MVC中，也支持 转发到本地handler 通过 forward()方法。   
为了使用 ProxyExchange，包含正确的模块到你的classpath (spring-cloud-gateway-mvc 或 spring-cloud-gateway-webflux)

下面是 MVC 的例子，把向/test下游的请求 代理到远程服务器

```java
@RestController
@SpringBootApplication
public class GatewaySampleApplication {

    @Value("${remote.home}")
    private URI home;

    @GetMapping("/test")
    public ResponseEntity<?> proxy(ProxyExchange<byte[]> proxy) throws Exception {
        return proxy.uri(home.toString() + "/image/png").get();
    }

}
```

下面是相同的功能在 webflux中

```java
@RestController
@SpringBootApplication
public class GatewaySampleApplication {

    @Value("${remote.home}")
    private URI home;

    @GetMapping("/test")
    public Mono<ResponseEntity<?>> proxy(ProxyExchange<byte[]> proxy) throws Exception {
        return proxy.uri(home.toString() + "/image/png").get();
    }

}
```

ProxyExchange 中的方法 允许 handler 方法 去 发现和增强 收到的request的URI路径。   
例如，你可能想要 提炼路径的尾随元素，以将它们传递到下游：

```java
@GetMapping("/proxy/path/**")
public ResponseEntity<?> proxyPath(ProxyExchange<byte[]> proxy) throws Exception {
  String path = proxy.path("/proxy/path/");
  return proxy.uri(home.toString() + "/foos/" + path).get();
}
```

Spring MVC 和 Webflux 的所有功能 都能在 gateway handler方法中使用。这样， 你可以注入 request header和query参数。   
例如，你可以通过 在 @RequestMapping 注解中的声明 来约束 传入的请求。

你可以增加header到 下游响应，通过 使用 ProxyExchange 的 header() 方法。

你可以操作 response header (或response中任何东西) 通过增加一个 mapper 到 get()方法(或其他方法)。这个mapper是一个 Function，接受传入的ResponseEntity，转换成输出的ResponseEntity。   

为敏感头(默认是cookie，authorization)提供一流的支持，敏感头不能被传递到 下游，不能放到代理(x-forwarded-*)头

---









