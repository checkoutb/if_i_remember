

https://docs.spring.io/spring-cloud/docs/2020.0.2/reference/htmlsingle/#spring-cloud-commons-common-abstractions

模板(如 服务发现，负载均衡，断路器) 提取它们自己到 一个公共抽象层，这样 所有 SpringCloud 客户端 都可以使用，独立于实现 (比如，服务发现是使用Eureka 还是Consul)

---
#### @EnableDiscoveryClient

这个注解 通过META-INF/spring.factories 搜索 DiscoveryClient 和 ReactiveDiscoveryClient 的实现.   
服务发现client 的实现 增加一个 配置类到 spring.factories，通过 ***org.springframework.cloud.client.discovery.EnableDiscoveryClient***。  
DiscoveryClient 的实现有 SpringCloudNetflixEureka，SpringCloudConsulDiscovery，SpringCloudZookeeperDiscovery。   

默认情况下，SpringCloud提供 blocking 和 reactive 服务发现客户端。 你可以 禁止通过：
> spring.cloud.discovery.blocking.enabled=false   
> spring.cloud.discovery.reactive.enabled=false

禁止全部：
> spring.cloud.discovery.enabled=false

默认情况下，DiscoveryClient的实现 自动把 本地SpringBoot服务 注册到 远程 服务发现服务器。   
可以通过  @EnableDiscoveryClient 注解的 **autoRegister=false**

@EnableDiscoveryClient 已经不再需要了， 你添加了一个 DiscoveryClient的实现到classpath，SpringBoot 会自动 注册到 服务发现服务器

---
#### Health Indicators 健康指示

Commons 自动配置了 下列的 SpringBoot 健康指示

##### DiscoveryClientHealthIndicator
基于 当前注册的 DiscoveryClient 实现   
禁止该功能： spring.cloud.discovery.client.health-indicator.enabled=false
禁止description： spring.cloud.discovery.client.health-indicator.include-description=false
禁止服务检索： spring.cloud.discovery.client.health-indicator.use-services-query=false. 默认情况下indicator 会调用client的 getServices 方法。可能有许多注册的服务，消耗太大。 跳过服务检索，使用 client的 probe方法

##### DiscoveryCompositeHealthContributor
这个组合的健康指示 基于 所有注册的 DiscoveryHealthIndicator。   
禁止： spring.cloud.discovery.client.composite-indicator.enabled=false

---

#### 排序 DiscoveryClient实例
DiscoveryClient接口继承了 Ordered。 当有多个discovery client时，允许你定义 返回的 discovery client的 顺序。   
默认下，所有的 DiscoveryClient 的order 都是0。 如果你要修改 你自定义的 DiscoveryClient 实现的 顺序，那么 重写 getOrder()。   
你也可以 使用 属性 来设置 SpringCloud 提供的 DiscoveryClient实现 ， 如 ConsulDiscoveryClient, EurekaDiscoveryClient,ZookeeperDiscoveryClient. 你需要设置 spring.cloud.{clientIdentifier}.discovery.order 或者 对于eureka设置 eureka.client.order

---
#### SimpleDiscoveryClient
如果classpath中 没有 Service-Registry-backed DiscoveryClient， 一个 SimpleDiscoveryClient实例会被使用，这个实例会使用 property 去获得 服务和实例的 信息。

可用实例的信息通过： spring.cloud.discovery.client.simple.instances.service1[0].url=http://s11:8080 .
spring.cloud.discovery.client.simple.instances 是一个公共前缀， service1 是 服务的ID，[0] 代表实例的下标。url就是 实例所在的url。

---
#### ServiceRegistry
Commons 提供了 ServiceRegistry 接口， 这个接口提供了 register(Registration) 和 deregister(Registration) 方法,  使你能提供自定义注册的服务，Registration也是一个接口。   

下面展示了 ServiceRegistry 的用法：
```
@Configuration
@EnableDiscoveryClient(autoRegister=false)
public class MyConfiguration {
    private ServiceRegistry registry;

    public MyConfiguration(ServiceRegistry registry) {
        this.registry = registry;
    }

    // called through some external process, such as an event or a custom actuator endpoint
    public void register() {
        Registration registration = constructRegistration();
        this.registry.register(registration);
    }
}
```

每个 ServiceRegistry实现  有它自己的 Registry实现
ZookeeperRegistration - ZookeeperServiceRegistry
EurekaRegistration - EurekaServiceRegistry
ConsulRegistration - ConsulServiceRegistry

当你使用 ServiceRegistry 接口的时候， 要传递 正确的 Registry实现。

---

#### ServiceRegistry 自动注册

默认情况下，ServiceRegistry实现 自动注册 正在运行的服务。   
禁止： @EnableDiscoveryClient(autoRegister=false)  或 spring.cloud.service-registry.auto-registration.enabled=false 

ServiceRegistry 自动注册Event
当一个服务自动注册时， 有2个事件被激发，    
InstancePreRegisteredEvent 在服务被注册 前 执行，   
InstanceRegisteredEvent 在服务被注册 后 执行   
你可以注册一个 ApplicationListener 来 监听和处理 这些事件。

如果spring.cloud.service-registry.auto-registration.enabled=false，那么上面的事件不会被激发

---

#### Service Registry Actuator Endpoint
Commons 是提供了 /service-registry 端点，这个端点依赖 Spring Application Context 的 Registration bean。   
GET 调用 /service-registry 返回 Registration 的状态。   
POST 发 JSON， 修改当前 Registration 的状态 到 新值。 JSON必须包含 status 属性。 看 ServiceRegistry 实现的文档，查看 更新状态时允许的值

---

#### Spring RestTemplate 作为一个 Load Balancer Client
可以配置 一个 RestTemplate 作为一个 负载均衡客户端。   
为了创建一个 负载均衡的 RestTemplate， 创建一个 RestTemplate bean并且使用 @LoadBalanced。

```
@Configuration
public class MyConfiguration {

    @LoadBalanced
    @Bean
    RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

public class MyClass {
    @Autowired
    private RestTemplate restTemplate;
    
    public String doOtherStuff() {
        String results = restTemplate.getForObject("http://stores/stores", String.class);
        return results;
    }
}
```

RestTemplate 已经不能 通过 自动配置 创建。 应用必须显示创建它。

URI 需要使用 虚拟主机名字 ( 服务名，而不是 主机名)， BlockingLoadBalancerClient 被用来 创建一个 全的物理地址

为了使用 负载均衡的 RestTemplate， 你需要把 负载均衡实现 放在 classpath中， 增加 Spring Cloud LoadBalancer starter   

---

#### Spring WebClient 作为 负载均衡客户端

配置 WebClient 来 自动使用一个负载均衡客户端。要创建 负载均衡的WebClient，需要创建WebClient.Builder bean 并使用 @LoadBalanced
```
@Configuration
public class MyConfiguration {

    @Bean
    @LoadBalanced
    public WebClient.Builder loadBalancedWebClientBuilder() {
        return WebClient.builder();
    }
}

public class MyClass {
    @Autowired
    private WebClient.Builder webClientBuilder;

    public Mono<String> doOtherStuff() {
        return webClientBuilder.build().get().uri("http://stores/stores")
                        .retrieve().bodyToMono(String.class);
    }
}
```

URI需要是一个 虚拟主机名称 (即服务名)。  SpringCloudLoadBalancer 用来创建 物理全名。   

如果要使用 @LoadBalanced 的 WebClient.Builder， 在classpath中，你需要有一个 负载均衡器的实现，添加 SpringCloudLoadBalancer starter， 底层使用 ReactiveLoadBalancer

---

#### 重试失败的请求

负载均衡的 RestTemplate 通过配置，能进行重试。默认情况下，是不开启重试机制的。   
对于 非响应式 ( RestTemplate)， 你可以启用重试，通过增加 Spring Retry 到应用的 classpath。   
对于 响应式 ( WebTestClient)，你需要 spring.cloud.loadbalancer.retry.enabled=true   

禁止Spring Retry 或 Reactive Retry 重试： spring.cloud.loadbalancer.retry.enabled=false   

对于 非响应式的实现， 如果你想要 实现一个 BackOffPolicy 在重试中， 你需要创建一个 LoadBalancedRetryFactory 的bean，并且重写 createBackOffPolicy() 方法

对于 响应式的实现， 如果想要 BackOffPolicy，只需要设置 spring.cloud.loadbalancer.retry.backoff.enabled 到 false

你可以设置：
spring.cloud.loadbalancer.retry.maxRetriesOnSameServiceInstance: 在同一个ServiceInstance上 最多重试多少次。   
spring.cloud.loadbalancer.retry.maxRetriesOnNextServiceInstance: 在新的 ServiceInstance 上 重试多少次。   
spring.cloud.loadbalancer.retry.retryableStatusCodes: 这些状态码 需要总是被重试。   

对于 响应式 实现， 你可以 额外设置    
spring.cloud.loadbalancer.retry.backoff.minBackoff, 设置 最小的补偿间隔 (backoff duration) (默认：5 毫秒)，   
spring.cloud.loadbalancer.retry.backoff.maxBackoff, 最大 backoff duration (默认是 LONG_MAX 毫秒)   
spring.cloud.loadbalancer.retry.backoff.jitter, 设置 jitter(晃动) ，用来计算 每个call时的 实际 backoff duration。 (默认 0.5)

对于响应式实现， 你也可以实现自己的 LoadBalancerRetryPolicy, 以 获得 负载均衡的retry的 更多细节控制   

为了 负载均衡retry， 默认下，我们 包装ServiceInstanceListSupplier bean with RetryAwareServiceInstanceListSupplier 来 选择一个和 上一次使用的实例 不同的 实例，如果可用的话。   
禁止这种行为通过 设置 spring.cloud.loadbalancer.retry.avoidPreviousInstance 到 false

```
@Configuration
public class MyConfiguration {
    @Bean
    LoadBalancedRetryFactory retryFactory() {
        return new LoadBalancedRetryFactory() {
            @Override
            public BackOffPolicy createBackOffPolicy(String service) {
                return new ExponentialBackOffPolicy();
            }
        };
    }
}
```

如果你想要一个或多个 RetryListener实现 在你的 retry 功能中， 你需要创建一个 LoadBalancedRetryListenerFactory类型的bean，并返回 RetryListener 数组。
```
@Configuration
public class MyConfiguration {
    @Bean
    LoadBalancedRetryListenerFactory retryListenerFactory() {
        return new LoadBalancedRetryListenerFactory() {
            @Override
            public RetryListener[] createRetryListeners(String service) {
                return new RetryListener[]{new RetryListener() {
                    @Override
                    public <T, E extends Throwable> boolean open(RetryContext context, RetryCallback<T, E> callback) {
                        //TODO Do you business...
                        return true;
                    }

                    @Override
                     public <T, E extends Throwable> void close(RetryContext context, RetryCallback<T, E> callback, Throwable throwable) {
                        //TODO Do you business...
                    }

                    @Override
                    public <T, E extends Throwable> void onError(RetryContext context, RetryCallback<T, E> callback, Throwable throwable) {
                        //TODO Do you business...
                    }
                }};
            }
        };
    }
}
```

---

#### 多个RestTemplate 对象

如果要一个 非负载均衡的 RestTemplate，那么就 创建bean并且注入。   
要使用 负载均衡的RestTemplate， 就需要 @LoadBalanced 到 你的bean上

```
@Configuration
public class MyConfiguration {

    @LoadBalanced
    @Bean
    RestTemplate loadBalanced() {
        return new RestTemplate();
    }

    @Primary
    @Bean
    RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

public class MyClass {
@Autowired
private RestTemplate restTemplate;

    @Autowired
    @LoadBalanced                           // ！！！！！！！！！！！！！！！！！！
    private RestTemplate loadBalanced;

    public String doOtherStuff() {
        return loadBalanced.getForObject("http://stores/stores", String.class);
    }

    public String doStuff() {
        return restTemplate.getForObject("http://example.com", String.class);
    }
}
```

注意 @Primary 在 普通的RestTemplate 上， 来消除 @Autowired 的歧义。

如果你遇到 IllegalArgumentException: Can not set org.springframework.web.client.RestTemplate field com.my.app.restTemplate to com.sun.proxy.$Proxy89, 尝试 注入 RestOperations 或设置 spring.aop.proxyTargetClass=true    

---

#### 多个 WebClient 对象

如果想要一个 不带 负载均衡的 WebClient，那么 创建一个 bean。   
如果要 带负载均衡的 WebClient，那么使用 @LoadBalanced。

```
@Configuration
public class MyConfiguration {

    @LoadBalanced
    @Bean
    WebClient.Builder loadBalanced() {
        return WebClient.builder();
    }

    @Primary
    @Bean
    WebClient.Builder webClient() {
        return WebClient.builder();
    }
}

public class MyClass {
    @Autowired
    private WebClient.Builder webClientBuilder;

    @Autowired
    @LoadBalanced               // ！！！！！！！！！！！！！
    private WebClient.Builder loadBalanced;

    public Mono<String> doOtherStuff() {
        return loadBalanced.build().get().uri("http://stores/stores")
                        .retrieve().bodyToMono(String.class);
    }

    public Mono<String> doStuff() {
        return webClientBuilder.build().get().uri("http://example.com")
                        .retrieve().bodyToMono(String.class);
    }
}
```

---

#### Spring WebFlux WebClient 做为一个 负载均衡客户端

Spring WebFlux 能同时在 响应式 和 非响应式 WebClient 下工作。 就像 topic描述的：
> Spring WebFlux WebClient with ReactorLoadBalancerExchangeFilterFunction 链接指向  https://docs.spring.io/spring-cloud/docs/2020.0.2/reference/htmlsingle/#webflux-with-reactive-loadbalancer
> load balancer exchange filter function 链接指向 https://docs.spring.io/spring-cloud/docs/2020.0.2/reference/htmlsingle/#load-balancer-exchange-filter-functionload-balancer-exchange-filter-function

---
#### Spring WebFlux WebClient with ReactorLoadBalancerExchangeFilterFunction

可以配置 WebClient 来使用 ReactiveLoadBalancer。   
如果你增加Spring Cloud LoadBalancer starter 到 工程 且 如果spring-webflux 在classpath中， ReactorLoadBalancerExchangeFilterFunction 会自动配置。   
下面的例子展示了 如何配置 WebClient 来使用 响应式负载均衡
```
public class MyClass {
    @Autowired
    private ReactorLoadBalancerExchangeFilterFunction lbFunction;

    public Mono<String> doOtherStuff() {
        return WebClient.builder().baseUrl("http://stores")
            .filter(lbFunction)
            .build()
            .get()
            .uri("/stores")
            .retrieve()
            .bodyToMono(String.class);
    }
}
```

URI 使用 虚拟主机名称 (服务名)， ReactorLoadBalancer 用来生成 真正的物理地址。

---

#### Spring WebFlux WebClient 具有(with)  非响应式 负载均衡客户端

如果 spring-webflux 在classpath中， LoadBalancerExchangeFilterFunction 自动配置。 不过这个 底层 使用了 非响应式客户端。   
下面的例子展示了 如何配置一个 WebClient 来使用 负载均衡
```
public class MyClass {
    @Autowired
    private LoadBalancerExchangeFilterFunction lbFunction;

    public Mono<String> doOtherStuff() {
        return WebClient.builder().baseUrl("http://stores")
            .filter(lbFunction)
            .build()
            .get()
            .uri("/stores")
            .retrieve()
            .bodyToMono(String.class);
    }
}
```

URI 使用 服务名， LoadBalancerClient 用来生成 真正的物理地址。

这个方法已经 deprecated， 推荐使用  WebFlux with 响应式负载均衡。

---

#### Ignore Network Interfaces

有时，忽略 具名的网络接口 是有用的， 这样它们能被 服务发现注册 功能排除 (比如，在Docker中执行)。   
能设置 正则列表 来忽略 网络接口。   
下面的配置 忽略 docker0 接口 和所有 以veth开始的接口

application.yml
```
spring:
  cloud:
    inetutils:
      ignoredInterfaces:
        - docker0
        - veth.*
```

也能 强制 只使用某些网络地址， 通过 正则列表：

bootstrap.yml
```
spring:
  cloud:
    inetutils:
      preferredNetworks:
        - 192.168
        - 10.0
```

也能强制 使用 本站 地址：

application.yml
```
spring:
  cloud:
    inetutils:
      useOnlySiteLocalInterfaces: true
```
查看 Inet4Address.html.isSiteLocalAddress() 来获得 构成 site-local address 的 东西。

---

#### HTTP Client Factories

Spring Cloud Commons 提供了 bean， 用来 创建 Apache HTTP client(ApacheHttpClientFactory) 和 OK HTTP client(OkHttpClientFactory)。   
OkHttpClientFactory bean 只有当 OK HTTP jar在 classpath中时 创建。   
另外，Commons 提供了 bean 来创建 连接管理器， ApacheHttpClientConnectionManagerFactory, OkHttpClientConnectionPoolFactory   
如果你想自定义： 在下游工程中如何创建HTTP Client，你可以提供你 对于这些bean的 自己的实现。   
另外，如果你提供了一个 HttpClientBuilder 或 OkHttpClient.Builder 类 的 bean， 默认工厂使用这些builder 来返回给 下游工程。   
你也可以 禁止 这些bean的创建，通过设置 spring.cloud.httpclientfactories.apache.enabled 或 spring.cloud.httpclientfactories.ok.enabled 到false

---

#### Enabled Features

Commons 提供了一个 /features actuator 端点。 这个端点 返回 在 classpath 中可用的 feature (不管是否已激活)。   
返回的信息包含 feature type,name,version,vendor

---

#### Feature types

feature 有2种类型： abstract， named

abstract feature： 定义了接口或抽象类，并且有实现，如 DiscoveryClient， LoadBalancerClient， LockService。 这个抽象类或接口 用来 从上下文中 找到 该类型的 bean。 显示的版本是 bean.getClass().getPackage().getImplementationVersion().

named feature： 没有实现类。 包含 Circuit Breaker， API Gateway, Spring Cloud Bus. 这些功能需要 一个名字 和一个 bean 类型。

---

#### 声明 feature
任何模块都能声明 任意数量的 HasFeature bean: 
```
@Bean
public HasFeatures commonsFeatures() {
  return HasFeatures.abstractFeatures(DiscoveryClient.class, LoadBalancerClient.class);
}

@Bean
public HasFeatures consulFeatures() {
  return HasFeatures.namedFeatures(
    new NamedFeature("Spring Cloud Bus", ConsulBusAutoConfiguration.class),
    new NamedFeature("Circuit Breaker", HystrixCommandAspect.class));
}

@Bean
HasFeatures localFeatures() {
  return HasFeatures.builder()
      .abstractFeature(Something.class)
      .namedFeature(new NamedFeature("Some Other Feature", Someother.class))
      .abstractFeature(Somethingelse.class)
      .build();
}
```

这些bean中的 每个都应该放在 被适当保护的 @Configuration 中。

---

#### Spring Cloud Compatibility Verification  兼容性验证

由于一个事实： 一些用户 在 配置 Spring Cloud应用时有问题。 我们决定增加一个 兼容性校验机制。   
如果你现在的 设置 和 SpringCloud 要求的 不兼容， 会break，返回一个 报告，展示错了什么：

```
***************************
APPLICATION FAILED TO START
***************************

Description:

Your project setup is incompatible with our requirements due to following reasons:

- Spring Boot [2.1.0.RELEASE] is not compatible with this Spring Cloud release train


Action:

Consider applying the following actions:

- Change Spring Boot version to one of the following versions [1.2.x, 1.3.x] .
You can find the latest Spring Boot versions here [https://spring.io/projects/spring-boot#learn].
If you want to learn more about the Spring Cloud Release train compatibility, you can visit this page [https://spring.io/projects/spring-cloud#overview] and check the [Release Trains] section.
```

spring.cloud.compatibility-verifier.enabled 设置为 false， 禁用这个功能。
spring.cloud.compatibility-verifier.compatible-boot-versions 设置 逗号分隔的 spring boot 版本列表  来作为 兼容的版本。

---





























