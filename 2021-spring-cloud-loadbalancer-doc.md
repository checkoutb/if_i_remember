
https://docs.spring.io/spring-cloud/docs/2020.0.2/reference/htmlsingle/#spring-cloud-loadbalancer

Spring Cloud LoadBalancer

spring cloud 提供了 客户端侧的 负载均衡器 的抽象 和实现。   
为了 负载均衡机制， ReactiveLoadBalancer 接口被增加。   
要为选定的服务 或 全部服务 切换实现， 你可以使用 自定义 LoadBalancer 配置机制 (href: https://docs.spring.io/spring-cloud/docs/2020.0.2/reference/htmlsingle/#custom-loadbalancer-configuration ， 本章的后面)   

例如，下面的配置 通过 @LoadBalancerClient 传递，来 切换到 使用 RandomLoadBalancer
```
public class CustomLoadBalancerConfiguration {
    @Bean
    ReactorLoadBalancer<ServiceInstance> randomLoadBalancer(Environment environment,
            LoadBalancerClientFactory loadBalancerClientFactory) {
        String name = environment.getProperty(LoadBalancerClientFactory.PROPERTY_NAME);
        return new RandomLoadBalancer(loadBalancerClientFactory
                .getLazyProvider(name, ServiceInstanceListSupplier.class),
                name);
    }
}
```

这个类，作为 @LoadBalancerClient 或 @LoadBalcnerClients 配置参数， 不应该被 @Configuration注解， 应该在 组件扫描之外。

---

#### Spring Cloud LoadBalancer integrations (结合，融合)

为了 使得 使用 Spring Cloud LoadBalancer 更容易， 我们提供了 **ReactorLoadBalancerExchangeFilterFunction**, 这个能 和 WebClient一起使用， **BlockingLoadBalancerClient**这个和RestTemplate一起。   

更多信息和例子在下面：   
Spring RestTemplate as a Load Balancer Client   
https://docs.spring.io/spring-cloud/docs/2020.0.2/reference/htmlsingle/#rest-template-loadbalancer-client

Spring WebClient as a Load Balancer Client   
https://docs.spring.io/spring-cloud/docs/2020.0.2/reference/htmlsingle/#webclinet-loadbalancer-client

Spring WebFlux WebClient with ReactorLoadBalancerExchangeFilterFunction   
https://docs.spring.io/spring-cloud/docs/2020.0.2/reference/htmlsingle/#webflux-with-reactive-loadbalancer


---

#### Spring Cloud LoadBalancer Caching
除了基本的 **ServiceInstanceListSupplier** 的实现，这个实现用于 检索实例 通过 DiscoveryClient，每次它不得不选择一个实例。 我们还提供了2个缓存实现。

#### Caffeine-backed LoadBalancer Cache Implementation
如果你有一个 com.github.ben-manes.caffeine:caffeine 在 classpath。 基于Caffeine 的实现 会被使用。   
查看下面网址来获得 如何配置：   
https://docs.spring.io/spring-cloud/docs/2020.0.2/reference/htmlsingle/#loadbalancer-cache-configuration

如果你在使用Caffeine，你也可以覆盖默认的 Caffeine Cache 设置， 通过 spring.cloud.loadbalancer.cache.caffeine.spec 属性来 传递自己的 Caffeine Specification(https://static.javadoc.io/com.github.ben-manes.caffeine/caffeine/2.2.2/com/github/benmanes/caffeine/cache/CaffeineSpec.html) 来生成 LoadBalancer

警告：传递你自己的Caffeine规则 会 覆盖 LoadBalancer 的 默认的 Caffeine Cache 设置，包括 General LoadBalancer Cache Configuration(https://docs.spring.io/spring-cloud/docs/2020.0.2/reference/htmlsingle/#loadbalancer-cache-configuration) 的属性，比如 ttl，capacity

---

#### Default LoadBalancer Cache Implementation
如果你没有Caffeine 在classpath， DefaultLoadBalancerCache 会被使用，这个来自 spring-cloud-starter-loadbalancer.   
查看下面网址来了解 如何配置   
https://docs.spring.io/spring-cloud/docs/2020.0.2/reference/htmlsingle/#loadbalancer-cache-configuration

#### LoadBalancer Cache Configuration
你可以设置你自己的 ttl 值(写入后过多久需要被清除)，通过传递一个 符合 Spring Boot String to Duration converter syntax(https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-external-config-conversion-duration) 的 string 到 spring.cloud.loadbalancer.cache.ttl。   
也可以设置你自己的 LoadBalancer cache 初始容量通过 spring.cloud.loadbalancer.cache.capacity   

默认ttl 是35s， 初始容量是256

可以禁止 负载均衡缓存 设置 spring.cloud.loadbalancer.cache.enabled 为false

基础的，无缓存的 实现 在 原型设计，测试 是 有用的， 但是 比缓存版本 效率低， 我们建议 生产 始终使用缓存版本

---
#### Zone-Based Load-Balancing

为了激活 基于zone的 负载均衡，我们提供了 ZonePreferenceServiceInstanceListSupplier.    
我们使用 DiscoveryClient规格的 zone 配置(如，euraka.instance.mertadata-map.zone) 来 挑选 zone that 客户端尝试 filter 有效的服务实例 for。

你可以覆盖 DsicoveryClient 规格的zone 配置 通过设置 spring.cloud.loadbalancer.zone   

目前，只有 Eureka Discovery Client 能被 instrument(仪表化) 来设置 LoadBalancer zone。      
其他的 discovery client，设置 spring.cloud.loadbalancer.zone。   
更多的 instrumentation (仪表化) 会 很快加入。   
。。。根据是 自动配置？  或者  可视化的配置？

。。。好像 zone 不是软件，而是指 地区， 根据地区 来 选择一个 合适的服务实例。。。

为了确定 检索到的ServiceInstance 的zone。 我们检查 它的metadata map 中的 "zone" 这个key。

**ZonePreferenceServiceInstanceListSupplier** filter 检索 实例，只 返回 相同 zone 的 实例。   
如果zone 是null 或 相同zone中没有 实例。则返回 所有检索到的 实例。

为了使用 基于zone的 负载均衡， 你必须 实例化一个 ZonePreferenceServiceInstanceListSupplier bean 在配置中(类上使用@LoadBalancerClient，这个注解有个 configuration 传递自己的 配置，https://docs.spring.io/spring-cloud/docs/2020.0.2/reference/htmlsingle/#custom-loadbalancer-configuration )。   

我们使用 delegate(代表，应该指 委托设计模式)，通过 ServiceInstanceListSupplier bean。      
我们建议传递一个 DiscoveryClientServiceInstanceListSupplier delegate 到 ZonePreferenceServiceInstanceListSupplier 构造器 and, in turn(相反), 通过CachingServiceInstanceListSupplier封装后者 来 影响 负载均衡缓存机制。   

可以使用这个简单配置来设置：
```
public class CustomLoadBalancerConfiguration {

    @Bean
    public ServiceInstanceListSupplier discoveryClientServiceInstanceListSupplier(
            ConfigurableApplicationContext context) {
        return ServiceInstanceListSupplier.builder()
                    .withDiscoveryClient()
                    .withZonePreference()
                    .withCaching()
                    .build(context);
    }
}
```

---

#### Instance Health-Check for LoadBalancer

可以允许一个 定时的HealthCheck for 负载均衡器。   
HealthCheckServiceInstanceListSupplier 就是为这个而提供的。它定期验证： 一个 delegate ServiceInstanceListSupplier 提供的 实例 是否存活， 并且只返回 健康的实例s，除非没有，那么它返回所有 检索到的 实例。    

这种架构是特别有用的， 当使用SimpleDiscoveryClient时。 因为 client 被一个 真实的Service Registry 支持， 它不需要使用， 由于我们已经获得健康的实例 在查询外部ServiceDiscovery 后。
。。。怎么觉得 前后矛盾， 前面特别有用， 后面就不需要使用。

这个Supplier 建议 在每个服务上 配置 几个实例。防止 在一个失败的实例 上 重试。

如果使用 任何 Service Discovery-backed suppliers, 增加这个 健康检查机制 通常不是必须的， 因为我们 直接从Service Registry中 检索 实例的健康状态

spring.cloud.loadbalancer.health-check.refetch-instances   
spring.cloud.loadbalancer.health-check.refetch-instances-interval   
spring.cloud.loadbalancer.repeat-health-check   
。。一大段。。就是讲述， 很少情况下，你想 不刷新 委托对象。

HealthCheckServiceInstanceListSupplier 使用 属性前缀： spring.cloud.loadbalancer.health-check      
可以设置 定时器的 initialDelay 和 interval      
通过spring.cloud.loadbalancer.health-check.path.default 设置 健康检查url 的path。   
可以使用 spring.cloud.loadbalancer.health-check.path.[service_id], [service_id]是服务ID， (设置这个服务的path)。   
如果path没有设置，/actuator/health 是默认的

如果你依赖于 默认path (/actuator/health)， 那么确保 你增加了 spring-boot-starter-actuator 到 你的 collaborator(合作者) 的dependencies， 除非你准备增加一个这样的端点在你自己上。   

为了使用 健康检查定时器，你必须 实例化 HealthCheckServiceInstanceListSupplier bean 在 custom configuration(@LoadBalancerClient的配置 https://docs.spring.io/spring-cloud/docs/2020.0.2/reference/htmlsingle/#custom-loadbalancer-configuration)

我们使用 delegates 来和ServiceInstanceListSupplier bean 一起工作。   
我们建议传递一个 DiscoveryClientServiceInstanceListSupplier delegate 到 HealthCheckServiceInstanceListSupplier 的构造器中。

可以使用下面的配置来设置
```
public class CustomLoadBalancerConfiguration {

    @Bean
    public ServiceInstanceListSupplier discoveryClientServiceInstanceListSupplier(
            ConfigurableApplicationContext context) {
        return ServiceInstanceListSupplier.builder()
                    .withDiscoveryClient()
                    .withHealthChecks()
                    .build(context);
        }
    }
}
```

for non-reactive stack, 创建这个 supplier 通过 withBlockingHealthChecks()。 你也可以传递你自己的 WebClient 或 RestTemplate 实例 来被 check 用到。

HealthCheckServiceInstanceListSupplier 有它自己的缓存架构，基于Reactor Flux replay()， 因此，如果它被使用，你可能希望跳过 那个supplier的封装，通过 CachingServiceInstanceListSupplier。

---

#### Same instance preference(偏好) for LoadBalancer

你可以设置 LoadBalancer 以这样的方式： 如果上一次选择的实例可用，负载均衡器会选择这个实例。

为了这个目标，你需要使用 SameInstancePreferenceServiceInstanceListSupplier 。   
可以配置它通过 设置 *spring.cloud.loadbalancer.configuration* 为 same-instance-preference 或 提供你自己的 ServiceInstanceListSupplier. 如下面的：
```
public class CustomLoadBalancerConfiguration {

    @Bean
    public ServiceInstanceListSupplier discoveryClientServiceInstanceListSupplier(
            ConfigurableApplicationContext context) {
        return ServiceInstanceListSupplier.builder()
                    .withDiscoveryClient()
                    .withSameInstancePreference()
                    .build(context);
        }
    }
}
```
这也是 Zookeeper StickyRule 的一个替代品

---
#### Request-based Sticky Session for LoadBalancer

设置 LB 以这样的方式： 它优先 通过 request cookie中 instanceId 来决定 选择哪个 实例。   
我们现在支持这个，如果request 通过 ClientRequestContext 或 ServerHttpRequestContext 传递给LB， SC LoadBalancer 交换过滤功能 和 过滤 来使用它们。

你需要使用 RequestBasedStickySessionServiceInstanceListSupplier. 你可以配置它 通过设置 *spring.cloud.loadbalancer.configurations* 为 request-based-sticky-session, 或 提供你自己的 ServiceInstanceListSupplier bean，如下：
```
public class CustomLoadBalancerConfiguration {

    @Bean
    public ServiceInstanceListSupplier discoveryClientServiceInstanceListSupplier(
            ConfigurableApplicationContext context) {
        return ServiceInstanceListSupplier.builder()
                    .withDiscoveryClient()
                    .withRequestBasedStickySession()
                    .build(context);
        }
    }
```

对于该功能，经常用来 在 发送request forward(转发请求) 之前 更新 选择的服务实例(这个可能不是原始request cookie中的，因为cookie中的而不可用)。   
要做到这一点，设置 spring.cloud.loadbalancer.sticky-session.add-service-instance-cookie 为 true。

默认情况下，cookie的名字是 sc-lb-instance-id, 可以通过 spring.cloud.loadbalancer.instance-id-cookie-name 来制定。

---
#### Spring Cloud LoadBalancer Hints
Spring Cloud LB 能让你 设置 String类型的 hint，这个hint 包含于Request对象中， 传递给LB， 这个稍后被用在 ReactiveLoadBalancer 实现中 被处理。   

你可以设置一个默认 hint 对于所有 服务， 通过 spring.cloud.loadbalancer.hint.default 属性。   
你也可以设置 spring.cloud.loadbalancer.hint.[service_id]， 给 服务ID 所指的这个 服务 设置 hint。   

---
#### Hint-Based Load-Balancing
提供了一个 **HintBasedServiceInstanceListSupplier**， 这个是 **ServiceInstanceListSupplier** 基于 hint 的 实例选择 的 实现。

HintBasedServiceInstanceListSupplier 检查 request的header中的 hint(默认header-name是 X-SC-LB-Hint, 可以通过 spring.cloud.loadbalancer.hint-header-name 来自定义header-name)， 如果找到，使用这个header中的hint 来filter 服务实例。

如果header中 没有hint，HintBasedServiceInstanceListSupplier 使用 属性中的 hint(就是loadbalancer.hint.default/hint.[service_id]) 来 过滤服务实例

如果没有 在 header 或property 中设置 hint， 返回 delegate 提供的所有 服务实例。

在过滤时， HintBasedServiceInstanceListSupplier 查找 服务实例，这个服务实例 在它的 metadataMap 中 "hint"作为key 得到的值 匹配hint。   
如果没有匹配的， delegate提供的所有实例 被返回。

下面是配置：
```
public class CustomLoadBalancerConfiguration {

    @Bean
    public ServiceInstanceListSupplier discoveryClientServiceInstanceListSupplier(
            ConfigurableApplicationContext context) {
        return ServiceInstanceListSupplier.builder()
                    .withDiscoveryClient()
                    .withHints()
                    .withCaching()
                    .build(context);
    }
}
```

---
#### Transform the load-balanced HTTP request

可以使用选择的 ServiceInstance 来 转换 负载均衡HTTP请求   

对于RestTemplate，你需要实现和定义 LoadBalancerRequestTransformer 如下：
```
@Bean
public LoadBalancerRequestTransformer transformer() {
    return new LoadBalancerRequestTransformer() {
        @Override
        public HttpRequest transformRequest(HttpRequest request, ServiceInstance instance) {
            return new HttpRequestWrapper(request) {
                @Override
                public HttpHeaders getHeaders() {
                    HttpHeaders headers = new HttpHeaders();
                    headers.putAll(super.getHeaders());
                    headers.add("X-InstanceId", instance.getInstanceId());
                    return headers;
                }
            };
        }
    };
}
```
。。似乎是把 普通请求，转为了 负载均衡请求。。就是往header里增加一些东西。 但是 X-InstanceId。。没有见过啊，

对于WebClient， 你需要 实现和定义 LoadBalancerClientRequestTransformer:
```
@Bean
public LoadBalancerClientRequestTransformer transformer() {
    return new LoadBalancerClientRequestTransformer() {
        @Override
        public ClientRequest transformRequest(ClientRequest request, ServiceInstance instance) {
            return ClientRequest.from(request)
                    .header("X-InstanceId", instance.getInstanceId())
                    .build();
        }
    };
}
```
如果有多个 transformer， 按照 bean中定义的 顺序(?@Order?) 来使用。   
或者 你可以使用 LoadBalancerRequestTransformer.DEFAULT_ORDER 或 LoadBalancerClientRequestTransformer.DEFAULT_ORDER 来指定 顺序。

---
#### Spring Cloud LoadBalancer Starter
提供了一个starter，为了让你能 在一个SpringBoot应用中方便地添加 Spring Cloud LoadBalancer.   
只需要将 org.springframework.cloud:spring-cloud-starter-loadbalancer 添加到 你的 依赖中。

spring cloud loadbalancer starter 包含 Spring Boot Caching 和 Evictor。

---
#### Passing Your Own Spring Cloud LoadBalancer Configuration

可以使用 ***@LoadBalancerClient*** 来传递 你自己的 负载均衡客户端配置，传递 负载均衡客户端的**名字**(。。谁的名字？不能通用？) 和 配置类：   
下面的value参数("stores") 指明了 服务ID， 我们会将 request 发送到这个服务，并且将 自定义配置的内容放到request中。   
```
@Configuration
@LoadBalancerClient(value = "stores", configuration = CustomLoadBalancerConfiguration.class)
public class MyConfiguration {

    @Bean
    @LoadBalanced
    public WebClient.Builder loadBalancedWebClientBuilder() {
        return WebClient.builder();
    }
}
```

为了更轻松地处理 你的LoadBalancer配置， 我们增加了一个 builder() 方法到 ServiceInstanceListSupplier类。

你也可以使用我们 可替代的 预定义的配置 代替 默认的配置，   
通过 设置 spring.cloud.loadbalancer.configurations 为 zone-preference 来使用 有cache的 ZonePreferenceServiceInstanceListSupplier，    
或者设置为 health-check 来使用 带cache的 HealthCheckServiceInstanceListSupplier。   

你能使用这个特点 来 实例化 ServiceInstanceListSupplier或 ReactorLoadBalancer 的不同实现。   
这些实现可以由你编写，也可以由我们的替代方案提供(如 ZonePreferenceServiceInstanceListSupplier) 以覆盖默认设置。

一个例子(这个也是 Zone-Based Load-Balancing 中的)：
```
public class CustomLoadBalancerConfiguration {

    @Bean
    public ServiceInstanceListSupplier discoveryClientServiceInstanceListSupplier(
            ConfigurableApplicationContext context) {
        return ServiceInstanceListSupplier.builder()
                    .withDiscoveryClient()
                    .withZonePreference()
                    .withCaching()
                    .build(context);
    }
}
```

你也可以传递 多个配置 (为了 多个 负载均衡客户端) 通过 @LoadBalancerClient***s***：
```
@Configuration
@LoadBalancerClients({@LoadBalancerClient(value = "stores", configuration = StoresLoadBalancerClientConfiguration.class), @LoadBalancerClient(value = "customers", configuration = CustomersLoadBalancerClientConfiguration.class)})
public class MyConfiguration {

    @Bean
    @LoadBalanced
    public WebClient.Builder loadBalancedWebClientBuilder() {
        return WebClient.builder();
    }
}
```
你在@LoadBalancerClient 或 @LoadBalancerClients的参数 传递的类， should either not be annotated with @Configuration or be outside component scan scope. 
。。。这个是说  要么 没有@Configuration， 如果有，那么就需要在Scan 外？  就是不能作为 spring的 组件 啊。

---
#### Spring Cloud LoadBalancer Lifecycle

使用自定义LB配置注册时，可能有用的类是 LoadBalancerLifecycle  

LoadBalancerLifecycle bean提供了 回调函数：onStart(Request<RC> request), onStartRequest(Request<RC> xx, Response<T> xxx) 和 onComplete(CompletionContext<RES,T,RC> xx)   
你应该实现这些方法来 制定 loadbalancing 前 和后 应该做什么。   

onStart(Request<RC> xx) 接受一个 Request 对象作为参数， 它包含了 用于选择合适实例 的数据，包括 (将发送)下游客户端请求 和 hint。   
onStartRequest 接受 Request 和 Response 作为参数   

onComplete 需要一个 CompletionContext 作为参数，它包含了 LB response，(LB Response) 包括了 选择的服务实例，在那个服务实例上 request执行的状态，下游客户端返回的response(如果有返回的话)，相应的异常(如果有异常的话)

supports(Class requestContextClass, Class responseClass, Class serverTypeClass) 方法 用来 决定 所讨论的处理器 是否能 处理所提供类型的 对象。 如果没有被重写，默认返回true。   

在前面的方法调用中， RC 意味 RequestContext 类型， RES意味着 client response type, T 意味着 返回的服务类型

---
#### Spring Cloud LoadBalancer Statistics

我们提供了一个 LoadBalancerLifecycle bean， 叫做 MicrometerStatsLoadBalancerLifecycle，这个使用 Micrometer 来提供 负载均衡调用 的统计数据。   

为了把这个 bean 加入到 你的应用上下文， 设置 spring.cloud.loadbalancer.stats.micrometer.enabled 为 true。并且有一个可用的 MeterRegistry(比如，通过增加SpringBootActuator 到工程)

MicrometerStatsLoadBalancerLifecycle 注册如下的 计量标准 在MeterRegistry
loadbalancer.requests.active    允许你监控 任何服务实例 的当前激活的request(通过tags能获得服务实例数据)   
loadbalancer.requests.success   一个计时器测量 任何负载均衡的request的执行 到 被调用客户端返回response 的 时间   
loadbalancer.requests.failed    一个计时器，测量，任何 负载均衡的request的执行 到 发生异常 的时间   
loadbalancer.requests.discard   一个计数器，测量，丢弃的负载均衡的request的数量，比如，LB没有找到运行这个request的服务实例。   

当可用时，通过tags 添加关于 服务实例，请求数据，返回数据 的额外信息

对于一些实现，比如 BlockingLoadBalancerClient，请求和响应数据可能 不可用， 因为我们 参数的泛型，可能无法决定 data的type，无法读取数据。   

计量标准 被登记到登记处，当至少有一条记录被添加到 该计量标准时。  

你可以进一步配置 这些计量标准的 行为 通过 增加 MeterFilters.  
































































