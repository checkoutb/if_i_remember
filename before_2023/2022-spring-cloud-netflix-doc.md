

https://docs.spring.io/spring-cloud/docs/2020.0.2/reference/htmlsingle/#spring-cloud-netflix

提供了SpringBoot 和 Netflix OOS 的集成，通过 自动配置，和绑定到Spring Environment 和其他Spring programming model idioms。   

通过一些简单的注解，你可以 快速启用和配置 应用中的通用模式，构建大型分布式系统，通过 久经考验的 Netflix 组件。

这个模式提供了： Service Discovery(Eureka), Circuit Breaker(Hystrix), Intelligent Routing(Zuul), Client Side Load Balancing(Ribbon)

---
#### Service Discovery: Eureka Clients

服务发现 是 基于微服务的架构 的 关键信条之一。   
手工配置每个客户端 是非常困难的。   
Eureka 是 Netflix Service Discovery Server and Client   
server能被配置，和部署，来获得 高可用，with 每个server 从其他server 复制已注册的service 的状态。   

---
#### How to Include Eureka Client

使用 org.springframework.cloud:spring-cloud-starter-netflix-eureka-client，来包含 Eureka Client 到工程。

---
#### Registering with Eureka

当一个客户端在Eureka注册时，它提供 它自己的meta-data,比如 host，port，health indicator URL，home page, 和其他信息。   
Eureka 从一个服务的 每个实例 接收 心跳信息。如果 心跳失败，并且持续失败了 配置的时间，实例从注册中心移除。

下面是一个迷你 Eureka 客户端应用：
```java
@SpringBootApplication
@RestController
public class Application {

    @RequestMapping("/")
    public String home() {
        return "Hello world";
    }

    public static void main(String[] args) {
        new SpringApplicationBuilder(Application.class).web(true).run(args);
    }

}
```

上面是 普通的 SpringBoot 应用，通过放置spring-cloud-starter-netflix-eureka-client 到classpath，你的应用会自动注册到Eureka Server。   

需要配置来定位 Eureka Server：
```yaml
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```
defaultZone 是一个 magic string fallback value，为所有client提供了 service URL。   
defaultZone 是大小写敏感的，并且需要驼峰，因为serviceUrl 属性是一个 Map<String, String>。所以，defaultZone 没有遵循 普通SpringBoot的 snake-case，如default-zone

默认的application name (同时也是 service ID), virtual host, non-secure port (token from the Environment), 分别是 ${spring.application.name}, ${spring.application.name} and ${server.port} 

在classpath中有 spring-cloud-starter-netflix-eureka-client，会使得 应用 既是一个 Eureka实例(它会注册它自己)，也是一个 Eureka客户端(它能查询注册中心以定位其他服务的地址)。   
实例行为通过 eureka.instance.* 驱动。但是，如果你的应用定义了spring.application.name (这个是Eureka service ID or VIP 的默认值)，那么默认定义 已经足够了。   

禁用 Eureka Discovery Client， eureka.client.enabled=false 或者 spring.cloud.discovery.enabled=false    

---
#### Authenticating with the Eureka Server

如果eureke.client.serviceUrl.defaultZones 中的URL 嵌入了证明 (curl风格, user:password@localhost:8761/eureka)，HTTP basic authentication 被自动添加到 你的 Eureka Client   
如果有更复杂的需求，可以创建一个 DiscoveryClientOptionalArgs类的 bean，并且注入一个ClientFilter实例。   

当Eureka server 要求 client 的身份证明时，client的证书和trust store 能通过 属性配置：   

application.yml
```yaml
eureka:
  client:
    tls:
      enabled: true
      key-store: <path-of-key-store>
      key-store-type: PKCS12
      key-store-password: <key-store-password>
      key-password: <key-password>
      trust-store: <path-of-trust-store>
      trust-store-type: PKCS12
      trust-store-password: <trust-store-password>
```

eureka.client.tls.enabled 设置为 true 来启用 Eureka client的 TLS。   
如果eureka.client.tls.trust-store 被忽略，使用JVM 默认的trust store   
eureka.client.tls.key-store-type, eureka.client.tls.trust-store-type 的 默认值是 PKCS12.      
password 不配置，就使用 空   

由于Eureka的限制，不可能支持 每台server 的 basic auth credentials，所以只有第一个被找到的配置集合 被使用。   

如果你要自定义 Eureka HTTP client 使用的 RestTemplate，你可以需要创建一个 EurekaClientHttpRequestFactorySupplier类的 bean，提供你自己的 生成 ClientHttpRequestFactory 实例的逻辑。

---
#### Status Page and Health Indicator

Eureka实例的默认 status page, health indicator 是 /info, /health，它们是spring boot actuator中有用端点的默认位置。。。。(是说冲突，还是说属于actuator？，应该是冲突)    

如果你使用了 非默认的 context path 或 servlet path (如，server.servletPath=/custom), 你需要修改配置，即使你是actuator应用。

```yaml
eureka:
  instance:
    statusPageUrlPath: ${server.servletPath}/info
    healthCheckUrlPath: ${server.servletPath}/health
```

这些link 出现在 被client消费的 metadata 中，用在某些场景：需要确定是否发送到你的app，所以如果信息是准确的，那么是有帮助的。   

---
#### Registering a Secure Application

如果你的app需要 通过 https 通信，你可以设置 2个标记 到 EurekaInstanceConfigBean 中：
```yaml
eureka.instance.[nonSecurePortEnabled]=[false]
eureka.instance.[securePortEnabled]=[true]
```

这么做 会使得 Eureka 发布instance info，这些info 展现了 对 安全链接的 明确偏好。   
对于这种配置的server， Spring Cloud DiscoveryClient 总是 返回一个 https 开头的URI   
类似的，当一个server这样配置，Eureka (native) instance info 有一个 安全协议的 health check URL。   

由于Eureka 内部工作的方式，它仍然会发布一个 非安全协议的 URL for status和home page，除非你也显示 覆盖了这些。   
你可以使用 占位符 来配置 Eureka 实例URLs，就像下面：   
```yaml
eureka:
  instance:
    statusPageUrl: https://${eureka.hostname}/info
    healthCheckUrl: https://${eureka.hostname}/health
    homePageUrl: https://${eureka.hostname}/
```
。。上面是 httpsssss

${eureka.hostname} 是一个native(。。估计是eureka自己的)占位符，只在高版本的Eureka中有效，你可以使用 Spring占位符(如，使用 ${eureka.instance.hostname}) 达到相同的效果。   

如果你的app在代理后运行，且SSL终止在代理上(比如，你在Cloud Foundry 或其他platforms as a service 上运行)，你需要确定 代理 "forwarded" 头被 应用 截获和处理。   
如果嵌入在SpringBoot应用中的 Tomcat容器 对于 'X-Forwarded-*' 头 有显示配置，那么这个会自动发生。   
如果 你的应用呈现给自身的链接(错误的host，post，protocol)是错误的，那么表明你没有正确地配置。   

---
#### Eureka's Health Checks

默认，Eureka 使用 客户端心跳 来决定 是否 客户端存活。  
根据SpringBootActuator，除非特别说明，DiscoveryClient 不会传播 应用的当前健康检查状态。    
最后，在成功注册后，Eureka 总是宣称：应用是'UP'状态。这个行为能被修改，通过 激活Eureka health check, 这个会导致 传播app状态到 Eureka。作为一个结果，每个其他app不需要 发送信息到 那些不是UP状态的应用。   
激活health check for the client:
```yaml
eureka:
  client:
    healthcheck:
      enabled: true
```

eureka.client.healthcheck.enabled=true 应该被放到 application.yml中，如果放到bootstrap中，会导致不受欢迎的边际效应，如注册到Eureka with UNKNOW 状态。

如果你需要更多的health check的控制，考虑实现你自己的 com.netflix.appinfo.HealthCheckHandler.   

---
#### Eureka Metadata for Instances and Clients

花时间理解 Eureka metadata 如何工作 是值得的，这样，你可以使用的时候更加清晰。   
标准metadata包含：hostname,IP,port,status page,health check. 这些信息被发布到服务注册中心，能被client用来 以直接了当的方式 访问服务。   
额外的metadata可以被增加到 实例注册，在eureka.instance.metadataMap，这个 metadata 能被 远程client访问到。   
一般情况，附加metadata 不会改变 client的行为，除非 client知道 metadata的 含义。   
稍后，有几个特殊例子，这些例子是 Spring Cloud 已经定义了 metadata map 的含义。   

---
#### Using Eureka on Cloud Foundry

Cloud Foundry 有一个全局router，这样相同app的 所有实例 有相同的 hostname。这不一定是使用Eureka的屏障。
当然，如果你使用router，你需要 明确地设置 hostname 和port，这样它们才能使用route。   
你可能也想使用 实例 metadata 这样 你可以 在client 区分 不同的实例。   
默认情况下，eureka.instance.instanceId 是 vcap.application.instance_id。就像下面
```yaml
eureka:
  instance:
    hostname: ${vcap.application.uris[0]}
    nonSecurePort: 80
```

根据你的Cloud Foundry 实例 的 安全策略的不同，你可以能够 注册和使用 主机VM的 IP地址 来 直接 进行 服务对服务的 调用。   
这个功能在 Pivotal Web Services (PWS) 上不可用。

---
#### Using Eureka on AWS
如果应用 准备发布到 AWS 云，Eureka 实例必须被配置成 AWS-aware (AWS感知)。   
你可以通过 自定义 EurekaInstanceConfigBean 来完成：
```java
@Bean
@Profile("!default")
public EurekaInstanceConfigBean eurekaInstanceConfig(InetUtils inetUtils) {
  EurekaInstanceConfigBean bean = new EurekaInstanceConfigBean(inetUtils);
  AmazonInfo info = AmazonInfo.Builder.newBuilder().autoBuild("eureka");
  bean.setDataCenterInfo(info);
  return bean;
}
```

---
#### Changing the Eureka Instance ID

一个 vanilla(香草) 的 Netflix Eureka instance 注册 with ID，这个ID等于 它的 host name (所以每个host 只有一个服务)   
Spring Cloud Eureka 提供了一个 默认：
> ${spring.cloud.client.hostname}:${spring.application.name}:${spring.application.instance_id:${server.port}}}

比如： myhost:myappname:8080

通过使用Spring Cloud, 你可以覆盖这个值 通过一个 唯一标识： eureka.instance.instanceId :
```yaml
eureka:
  instance:
    instanceId: ${spring.application.name}:${vcap.application.instance_id:${spring.application.instance_id:${random.value}}}
```

通过上面例子中的 metadata，以及在localhost 部署多个服务实例，随机值被用来 确保实例唯一。   
在Cloud Foundry, vcap.application.instance_id 是被自动输入到SpringBoot应用的，所以 random value 不是必须的。

---
#### Using the EurekaClient

一旦你有一个 服务发现客户端 的应用，你可以使用它 来发现服务实例， 从 Eureka Server。   
一种方式是 使用 原生的 com.netflix.discovery.EurekaClient (和 SpringCloud 的 DiscoveryClient 相对)   
```java
@Autowired
private EurekaClient discoveryClient;

public String serviceUrl() {
    InstanceInfo instance = discoveryClient.getNextServerFromEureka("STORES", false);
    return instance.getHomePageUrl();
}
```

不要使用 EurekaClient 在 @PostConstruct 方法 或 @Scheduled方法 (或任何 ApplicationContext 还没有启动完成的 地方)。   
它被初始化在一个 SmartLifecycle (with phase = 0), 所以你最早可以依赖 可用的它 是在另一个具有 更高phase 的 SmartLifecycle 中。

---
#### EurekaClient without Jersey

默认下。EurekaClient 使用 Spring的 RestTemplate 来进行HTTP 连接。   
如果你想要使用 Jersey，你需要 增加Jersey依赖 到你的 classpath：
```xml
<dependency>
    <groupId>com.sun.jersey</groupId>
    <artifactId>jersey-client</artifactId>
</dependency>
<dependency>
    <groupId>com.sun.jersey</groupId>
    <artifactId>jersey-core</artifactId>
</dependency>
<dependency>
    <groupId>com.sun.jersey.contribs</groupId>
    <artifactId>jersey-apache-client4</artifactId>
</dependency>
```

---
#### Alternatives to the Native Netflix EurekaClient

你不必使用 原始的Netflix EurekaClient。此外，在某个wrapper后面 使用它通常更方便。   
Spring Cloud 支持 Feign 和Spring RestTemplate 通过 逻辑Eureka服务标识符(VIPs) 而不是 物理URL。   
要使用固定的 物理服务器列表配置Ribbon，你可以设置 <client>.ribbon.listOfServers 为一个 逗号分隔的物理地址(或hostname) 列表。<client>是客户端的ID。

你也可以使用 org.springframework.cloud.client.discovery.DiscoveryClient，这个提供了一个简单API (不是特定于Netflix) 来获得 discovery clients：

```java
@Autowired
private DiscoveryClient discoveryClient;

public String serviceUrl() {
    List<ServiceInstance> list = discoveryClient.getInstances("STORES");
    if (list != null && list.size() > 0 ) {
        return list.get(0).getUri();
    }
    return null;
}
```

---
#### Why Is It so Slow to Register a Service?
实例还涉及 用于注册的 周期心跳，默认间隔30s。   
一个服务不能被client发现，直到 instance,server,client 都有相同的 metadata 在它们本地cache(所以 可能需要3次心跳)。   
你可以修改 间隔：eureka.instance.leaseRenewalIntervalInSeconds。   
在生产，使用默认值 更好，因为server 内部的计算 假设了 lease renewal period

---
#### Zones
如果你部署Eureka client 到 多个区域，你可以希望 client 优先 使用 同一个区域中的 服务。你需要配置Eureka client来达到这个目的   

第一，你需要确保 在每个 zone 都部署了 Eureka server，并且这些server都是相同规格的。   
接下来，你需要告诉 Eureka 它的service在哪个 zone，你可以通过使用 metadataMap 属性 来达到这个目的。   
比如，如果service1 被部署在 zone1 和 zone2, 你需要 在service1 设置 下面的 Eureka属性

Service1 in zone1
```properties
eureka.instance.metadataMap.zone = zone1
eureka.client.preferSameZoneEureka = true
```

Service1 in zone2
```properties
eureka.instance.metadataMap.zone = zone2
eureka.client.preferSameZoneEureka = true
```

---
#### Refreshing Eureka Clients
默认下，EurekaClient bean 是可以刷新的，意味着 Eureka client 属性可以被修改 和 刷新。   
当 刷新发生时，client 会从 Eureka server 注销，这样，可能有一段时间，某个服务的 所有实例不可用。   
防止这种不可用的一种方法是 禁用 Eureka client 刷新： eureka.client.refresh.enable=false

---
#### Using Eureka with Spring Cloud LoadBalancer

提供了 对于 Spring Cloud LoadBalancer 的支持： ZonePreferenceServiceInstanceListSupplier。   
从Eureka 实例 metadata (eureka.instance.metadataMap.zone) 获得的 zone值 被用来 设置 spring-cloud-loadbalancer-zone属性，这个熟悉用来 通过 zone过滤 service实例。   

如果那个没有，且 如果 spring.cloud.loadbalancer.eureka.approximateZoneFromHostname 是 true。 它能使用 server hostname 的domain name 作为 zone的代理。

如果没有其他的 zone数据的来源，那么会生成一个 猜测，基于 client 配置 (与实例配置相反)。获得 eureka.client.availabilityZones, 这个是一个map，从region name 到 list of zones，获得 实例的region (eureka.client.region, 默认us-east-1)的 zones list的 第一个zone。   

---
#### Service Discovery: Eureka Server

描述如何建立Eureka Server

---
#### How to Include Eureka Server

使用starter，org.springframework.cloud:spring-cloud-starter-netflix-eureka-server   

如果你的工程已经使用了 Thymeleaf 作为模板引擎，那么Eureka server的 Freemarker模板可能无法正确加载。这种情况下，你需要配置模板加载器：
```yaml
spring:
  freemarker:
    template-loader-path: classpath:/templates/
    prefer-file-system-access: false
```

---
#### How to Run a Eureka Server

下面的代码启动了一个 迷你 Eureka server：

```java
@SpringBootApplication
@EnableEurekaServer
public class Application {

    public static void main(String[] args) {
        new SpringApplicationBuilder(Application.class).web(true).run(args);
    }

}
```

这个server 有一个主页 和 普通eureka功能的 HTTP API 端点 在 /eureka/*    

由于Gradle 的依赖 解析规则 和 缺少父bom功能，依赖 spring-cloud-starter-netflix-eureka-server 可能导致失败，在应用启动时。   
作为补救，增加Spring Boot Gradle plugin 和导入 Spring cloud starter parent bom : 

```
buildscript {
  dependencies {
    classpath("org.springframework.boot:spring-boot-gradle-plugin:{spring-boot-docs-version}")
  }
}

apply plugin: "spring-boot"

dependencyManagement {
  imports {
    mavenBom "org.springframework.cloud:spring-cloud-dependencies:{spring-cloud-version}"
  }
}
```

---
#### High Availability, Zones and Regions

Eureka server 没有 后端存储，但是注册中心 中的 服务实例 会发送心跳来 维持他们的注册信息 是最新的(这个能做在内存中)。   
客户端也有一个 Eureka 注册信息 在 内存cache中 (这样，就不必每次都访问注册中心来获得服务)。

默认，每个Eureka server 也是一个 Eureka client 和 requires (至少一个) service URL 来 定位 peer(同龄人)。   
如果你没有提供它，服务 run and work, 但它会向 日志中 写入 很多信息 来表达 自己无法在 对等端注册。   

---
#### Standalone Mode

2个cache(client，server) 的组合 以及 心跳，使得 一个 独立运行的 Eureka server 对于错误有非常高的容忍性，只要 存在某种监控 或 弹性运行时 (如 Cloud Foundry) 来保持它存活。   
在独立运行的模式，你可能 更喜欢 关闭 客户端侧的行为，这样，它就不需要 一直 尝试 获得peer 和失败。   
下面关闭了 客户端侧的行为：

```yaml
server:
  port: 8761

eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

serviceUrl 指向了 本地实例的 host。

---
#### Peer Awareness

Eureka 能变得 更具容错性，更高可用性，如果运行多个 实例，并且让它们相互注册。实际上，这是一种默认行为，所以 你需要做的只是 增加一个 有效的 serviceUrl 到一个peer：

```yaml
---
spring:
  profiles: peer1
eureka:
  instance:
    hostname: peer1
  client:
    serviceUrl:
      defaultZone: https://peer2/eureka/

---
spring:
  profiles: peer2
eureka:
  instance:
    hostname: peer2
  client:
    serviceUrl:
      defaultZone: https://peer1/eureka/
```

上面的例子中，我们有一个 yaml文件，能被用来在 2个host 上运行相同的服务，通过 不同的Spring profile.   
你可以使用这个配置 来测试 peer awareness(同伴意识) 在一台主机上 通过 操作 /etc/hosts 来解析 主机名。   
实际上，eureka.instance.hostname 并不需要，如果你 运行在一台 知道它自己hostname的 机器上。(默认下，hostname可以通过 java.net.InetAddress 来获得)

你可以增加多个peer 到一个系统，只要 它们能相互访问，它们会同步 注册信息。   
如果peer 是物理隔离的 (一个数据中心 或 多个数据中心)，那么 原则上，系统可以 熬过 "脑分裂"失败。   

application.yml (Three Peer Aware Eureka Servers)
```yaml
eureka:
  client:
    serviceUrl:
      defaultZone: https://peer1/eureka/,http://peer2/eureka/,http://peer3/eureka/

---
spring:
  profiles: peer1
eureka:
  instance:
    hostname: peer1

---
spring:
  profiles: peer2
eureka:
  instance:
    hostname: peer2

---
spring:
  profiles: peer3
eureka:
  instance:
    hostname: peer3
```

---
#### When to Prefer IP Address
一些场景下，更希望Eureka 登记 服务的IP地址，而不是 hostname。   
设置 eureka.instance.preferIpAddress=true，这样 应用向eureka注册时，使用IP地址，而不是 hostname

如果hostname 无法通过Java 来确定，那么 IP地址 被发送到 Eureka。   
设置hostname的 唯一显示方式 是 eureka.instance.hostname    
你可以在 运行时 设置你的 hostname 通过 一个 环境变量： eureka.instance.hostname=${HOST_NAME}

---
#### Securing The Eureka Server

你可以 secure 你的Eureka server，通过 增加Spring Security 到你的 server 的classpath 通过 spring-boot-starter-security。

默认下，当SpringSecurity在classpath，它会要求 每次访问app的请求 都要有一个合法的 CSRF令牌。   
Eureka 客户端 通常不会拥有 合法的 Cross Site Request Forgery 令牌，你需要禁用这个功能 在 /eureka/** 端点。   

```java
@EnableWebSecurity
class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().ignoringAntMatchers("/eureka/**");
        super.configure(http);
    }
}
```


---
#### JDK 11 Support

Eureka server 依赖的 JAXB模块 在JDK11 中被移除了。   
如果你想在JDK11 上执行 Eureka server， 你需要 手动添加 依赖：

```xml
<dependency>
    <groupId>org.glassfish.jaxb</groupId>
    <artifactId>jaxb-runtime</artifactId>
</dependency>
```









