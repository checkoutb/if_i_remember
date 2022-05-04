



https://docs.spring.io/spring-cloud/docs/2020.0.2/reference/htmlsingle/#spring-cloud-consul

为SpringBoot应用提供Consul集成，通过自动配置和绑定到Spring Environment 和其他 Spring编程模型习语(Spring programming model idioms)   
通过一些注解，你可以 在你的app中 快速启用和配置 常见模式， 可以构建分布式系统 with 基于Consul的组件。   
提供的 模式包括 Service Discovery, Control Bus, Configuration.  智能路由(Zuul)，客户端的LB(Ribbon),断路器(Hystrix) 通过集成 SpringCloudNetflix 来提供。

---
#### Quick Start
使用Consul 来获得 服务发现，和分布式配置

首先，启动 Consul Agent 在你的机器上，然后 你可以访问它，并把它当作 SpringCloudConsul的 服务注册中心 和 配置源。 

---
#### Discovery Client Usage

为了在app中使用这些功能，你可以build 它 当作一个 依赖spring-cloud-consul-core 的 SpringBoot应用   
最简单的方式是增加一个SpringBoot starter： org.springframework.cloud:spring-cloud-starter-consul-discovery   
我们建议使用 依赖管理 和 spring-boot-starter-parent。   
下面是 典型的Maven配置：
```xml
<project>
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>{spring-boot-version}</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>

  <dependencies>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-consul-discovery</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dependencies</artifactId>
        <version>${spring-cloud.version}</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>
  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
```
下面是 典型的 Gradle设置：
```
plugins {
  id 'org.springframework.boot' version ${spring-boot-version}
  id 'io.spring.dependency-management' version ${spring-dependency-management-version}
  id 'java'
}

repositories {
  mavenCentral()
}

dependencies {
  implementation 'org.springframework.cloud:spring-cloud-starter-consul-discovery'
  testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
dependencyManagement {
  imports {
    mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
  }
}
```

现在你可以创建一个标准的SpringBoot 应用，比如下面的HTTP服务器：
```java
@SpringBootApplication
@RestController
public class Application {

    @GetMapping("/")
    public String home() {
        return "Hello World!";
    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

当这个HTTP服务器启动，它连接 Consul Agent 在默认的 local 8500端口。   
可以通过 property 来修改 Consul Agent的地址：
```yaml
spring:
  cloud:
    consul:
      host: localhost
      port: 8500
```

你现在可以使用 DiscoveryClient， @LoadBalanced RestTemplate 或 @LoadBalanced WebClient.Builder 来 检索服务和实例信息 从Consul，比如：
```
@Autowired
private DiscoveryClient discoveryClient;

public String serviceUrl() {
    List<ServiceInstance> list = discoveryClient.getInstances("STORES");
    if (list != null && list.size() > 0 ) {
        return list.get(0).getUri().toString();
    }
    return null;
}
```

。。上面是服务发现， 下面是 配置,  都是作为客户端的。

---
#### Distributed Configuration Usage

为了在app中使用这些功能，你可以build it 做为一个依赖 spring-cloud-consul-core 和 spring-cloud-consul-config 的SpringBoot应用      
最简单的方法是 增加依赖 org.springframework.cloud:spring-cloud-starter-consul-config.   
建议使用 dependency management 和 spring-boot-starter-parent   
下面是 典型的Maven配置
```xml
<project>
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>{spring-boot-version}</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>

  <dependencies>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-consul-config</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dependencies</artifactId>
        <version>${spring-cloud.version}</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>
  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
```

典型的Gradle 配置：
```
plugins {
  id 'org.springframework.boot' version ${spring-boot-version}
  id 'io.spring.dependency-management' version ${spring-dependency-management-version}
  id 'java'
}

repositories {
  mavenCentral()
}

dependencies {
  implementation 'org.springframework.cloud:spring-cloud-starter-consul-config'
  testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
dependencyManagement {
  imports {
    mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
  }
}
```

现在你可以创建一个标准 SB应用，比如下面的HTTP服务器：
```java
@SpringBootApplication
@RestController
public class Application {

    @GetMapping("/")
    public String home() {
        return "Hello World!";
    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

这个应用会从Consul 检索配置信息

如果你使用Spring Cloud Consul Config, 你需要设置 spring.config.import 属性来绑定 Consul   

---
#### Install Consul

https://www.consul.io/intro/getting-started/install.html

---
#### Consul Agent

必须有一个 Consul Agent 客户端 对所有 SpringCloudConsul 应用可用。   
默认下，会去 localhost:8500 寻找 Consul Agent 客户端。   
查看 https://consul.io/docs/agent/basics.html 获得 如何启动Agent 客户端，如何连接到 Consul Agent Server 集群   
对于开发，安装consul 后，你可以启动 consul agent 使用下面的命令：
```
./src/main/bash/local_run_consul.sh
```
这个会开启 一个agent 以 server 模式，在8500端口， 可以浏览器访问 http://localhost:8500/  

---
#### Service Discovery with Consul
服务发现是 微服务架构的 一个基础原则。   
Consul 提供了 Service Discovery 服务 通过 HTTP API和 DNS。   
SpringCloudConsul 通过 HTTP API 来 服务注册 与发现。这不会阻止 非SpringCloud应用 利用DNS接口。   
Consul Agents servers 跑在集群模式， 通过 gossip protocol 和 使用 Raft consensus protocol 来通信。

---
#### How to activate

为了激活 Consul Service Discovery， 使用starter： org.springframework.cloud:spring-cloud-starter-consul-discovery    

---
#### Registering with Consul
当一个客户端向Consul注册时，它提供 关于它自己的meta-data，比如 host，port，id，name，tags。   
默认情况下，一个HTTP Check 被创建， Consul 访问/actuator/health 每10秒。   
如果health check 失败，那么 服务实例 被标记为 critical

Consul客户端例子：
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

如果 Consul 客户端不是 localhost:8500， 可以配置：
application.yml
```yaml
spring:
  cloud:
    consul:
      host: localhost
      port: 8500
```

如果你使用 spring cloud consul config, 且你设置了 spring.cloud.bootstrap.enabled=true or spring.config.use-legacy-processing=true or 使用了spring-cloud-starter-bootstrap, 那么，上面的配置需要放在bootstrap.yml中，而不是 application.yml

默认的 service name, instance id and port, 从 Environment 中获得，分别是 ${spring.application.name}, SpringContextID 和 ${server.port}   

禁用Consul Discovery Client，你可以设置 spring.cloud.consul.discovery.enabled 到 false。 或者 spring.cloud.consul.discovery.register 为 false。

禁用服务注册，设置 spring.cloud.consul.discovery.register 到 false。

---
#### Registering Management as a Separate Service  (注册管理作为一个单独服务)

通过 management.server.port 属性 将 management server port 设置为 和 应用端口不同。   
management service 会被注册为一个 单独的 (和应用程序服务不同的) 服务。 

application.yml
```
spring:
  application:
    name: myApp
management:
  server:
    port: 4452
```
上面的配置，会注册2个服务：   
> Application Service: ID:myApp, Name:myApp   
> Management Service: ID:myApp-management, Name:myApp-management

management service 会从 application service 集成 instanceId，serviceName，如：

application.yml
```yaml
spring:
  application:
    name: myApp
management:
  server:
    port: 4452
spring:
  cloud:
    consul:
      discovery:
        instance-id: custom-service-id
        serviceName: myprefix-${spring.application.name}
```

上面的配置会注册2个服务：
> Application Service: ID:custom-service-id, Name:myprefix-myApp   
> Management Service: ID:custom-service-id-management, Name:myprefix-myApp-management   

进一步的配置可以通过下面的属性：
```
/** Port to register the management service under (defaults to management port) */
spring.cloud.consul.discovery.management-port

/** Suffix to use when registering management service (defaults to "management" */
spring.cloud.consul.discovery.management-suffix

/** Tags to use when registering management service (defaults to "management" */
spring.cloud.consul.discovery.management-tags
```

---
#### HTTP Health Check

Consul实例的 健康检查默认是 /actuator/health ，这个是 spring boot actuator 应用 的默认的 health endpoint    
如果你使用非默认的上下文路径 或servlet路径(server.servletPath=/foo) 或管理端点路径(management.server.servlet.context-path=/admin) ，你需要修改它，即使你的应用是 actuator应用。

健康检查的间隔也可以配置， 10s  1m 都是有效的。 一个10秒，一个1分钟。

查看 spring.cloud.consul.discovery.health-check-* 来获得更多信息

application.yml
```yaml
spring:
  cloud:
    consul:
      discovery:
        healthCheckPath: ${management.server.servlet.context-path}/actuator/health
        healthCheckInterval: 15s
```

你可以禁用 HTTP 健康检查， spring.cloud.consul.discovery.register-health-check=false

---
#### Applying Headers

Header 能被应用到 健康检查request中，例如，如果你尝试注册 使用Vault的 SpringCloudConfig server：   
application.yml
```yaml
spring:
  cloud:
    consul:
      discovery:
        health-check-headers:
          X-Config-Token: 6442e58b-d1ea-182e-cfa5-cf9cddef0722
```

根据HTTP标准，每个header 可以有多个值，这种情况，可以用数组：   
application.yml
```
spring:
  cloud:
    consul:
      discovery:
        health-check-headers:
          X-Config-Token:
            - "6442e58b-d1ea-182e-cfa5-cf9cddef0722"
            - "Some other value"
```

---
#### Actuator Health Indicator(s)

如果服务实例是 SpringBootActuator 应用，可能提供以下的 Actuator health indicator：

> DiscoveryClientHealthIndicator   

当consul service discovery 激活， 一个 DiscoverClientHealthIndicator 被配置，并且在 actuator 的健康端点 可用。

> ConsulHealthIndicator   

一个指标被配置，来验证 ConsulClient 的健康。   
默认，它检索 Consul leader node status 和所有 已注册的 服务。   
在prod，可能有许多 已注册的服务，消耗很大。要跳过 服务检索，只校验 leader node status，可以通过：spring.cloud.consul.health-indicator.include-services-query=false   
management.health.consul.enabled=false 来禁用这个 indicator。

当应用运行在 bootstrap context mode (即默认)， 这个indicator 被加载到 bootstrap context，对于actuator 健康端点不可用。   

---
#### Metadata

Consul 支持 metadata 在 服务上。   
SpringCloud 的 ServiceInstance 有一个 Map<String, String> metadata 属性。这里的信息 来自于 服务的 meta 属性。   
要设置meta 属性，可以通过： spring.cloud.consul.discovery.metadata 或 spring.cloud.consul.discovery.management-metadata 属性。   
application.yaml
```yaml
spring:
  cloud:
    consul:
      discovery:
        metadata:
          myfield: myvalue
          anotherfield: anothervalue
```

上面的配置会 使得 服务的 meta属性中包含 myfiled->myvalue 和 anotherfield->anothervalue.   

---
#### Generated Metadata
Consul Auto Registration 会自动生成下面的Metadata：   

|key|value|
|---|---|
|group|spring.cloud.consul.discovery.instance-group的值，如果这个值只有当instance-group非空时才创建|
|secure|如果spring.cloud.consul.discovery.scheme等于https 则true， 否则false|
|spring.cloud.consul.discovery.default-zone-metadata-name 的值，默认zone|spring.cloud.consul.discovery.instance-zone 的值，只有instance-zone非空才创建|

老版本的SpringCloudConsul 给ServiceInstance.getMetadata() 填充数据，从 SpringCloudCommons，通过转换spring.cloud.consul.discovery.tags属性， 这将不会再支持。要使用spring.cloud.consul.discovery.metadata   

---
#### Making the Consul Instance ID Unique

默认下，consul实例 通过 ID来注册，这个ID等于 Spring应用的Context ID。   
默认下，spring app context id 是 ${spring.application.name}:逗号分隔的profiles:${server.port}   
大多数情况下，一台机器上 可以运行 一个服务的多个实例。如果需要进一步的唯一性，可以通过spring.cloud.consul.discovery.instanceId   
application.yml
```yaml
spring:
  cloud:
    consul:
      discovery:
        instanceId: ${spring.application.name}:${vcap.application.instance_id:${spring.application.instance_id:${random.value}}}
```

通过这个metadata，本地部署多个实例， random值可以使得 instance 唯一。   
在 Cloud Foundry, vcap.application.instance_id 会被自动传入到 SpringBoot， 所以并不需要 random值。   

---
#### Looking up services

#### Using Load-balancer

SpringCloud 支持Feign， 和 RestTemplate， 来搜索服务，通过 服务逻辑名/id 而不是 物理url。   
Feign，和 具有服务发现功能的RestTemplate 利用 SpringCloud-LB 来做 客户端的负载均衡   

如果你想访问 STORES 服务 通过 RestTemplate：
```
@LoadBalanced
@Bean
public RestTemplate loadbalancedRestTemplate() {
     return new RestTemplate();
}
```
就像下面这样使用 (注意我们使用了 STORES 服务名/id )
```
@Autowired
RestTemplate restTemplate;

public String getFirstProduct() {
   return this.restTemplate.getForObject("https://STORES/products/1", String.class);
}
```

如果你有Consul 集群 在多个 数据中心，你希望访问一个服务 在另一个数据中心的， 单独的一个服务名/id 是不够的。   
你需要使用 spring.cloud.consul.discovery.datacenters.STORES=dc-west， STORES是服务名/id，dc-west是STORES服务所在的数据中心   

。。就是没有办法 在代码里写访问那个 数据中心，只能在配置里， 而且似乎独占的， 不可能 部分请求访问 本数据中心的STORES，其他的访问 另一个数据中心的STORES。   

---
#### Using the DiscoveryClient
你也可以使用 DiscoveryClient， 这个提供了一个简单的API for discovery clients，这些discovery客户端不必是 Netflix的   
```
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
#### Consul Catalog Watch

ConsulCatalogWatch 利用consul的能力 来 watch services。   
CatalogWatch 创建一个 阻塞Consul HTTP API 调用 来确定 是否有服务发生了变更。   
如果有新的 服务数据，一个 Heartbeat Event 被 发布。

修改config watch 的频率，spring.cloud.consul.config.discovery.catalog-services-watch-delay, 默认是1000， 单位ms。   

禁用： spring.cloud.consul.discovery.catalogServicesWatch.enabled=false   

catalog watch 使用 Spring的 TaskScheduler 来 调度 对于Consul的调用。   
默认下，它是一个 ThreadPoolTaskScheduler，它的poolSize是 1。   
要修改 TaskScheduler，创建 TaskScheduler类型的bean，并且bean的名字是 ConsulDiscoveryClientConfig.CATALOG_WATCH_TASK_SCHEDULER_NAME 这个常量的值。   

---
#### Distributed Configuration with Consul

Consul 提供 Key/Value Store 来存储配置 和其他 metadata。   
spring cloud consul config 是 ConfigServer/Client 的另一种选择。   
在特殊的"bootstrap"阶段，配置被加载到 Spring Environment。   
配置默认被存储在 /config 文件夹。   
多个PropertySource 实例被创建，基于 应用名字和激活的profiles that 模仿SpringCloudConfig 顺序 来解析属性。   

例如，一个应用名字是testApp，dev的profile，会有下面的 property source 被创建：   
config/testApp,dev/   
config/testApp   
config/application,dev/   
config/application   

最具体的 property source 在最上面，最不具体的在下面。   
config/application 文件夹中的属性 适用于 所有通过Consul 获得配置的应用     
config/testApp 文件夹中的属性 只对 服务的名字为testApp的 实例有用   

配置现在 在应用启动时 被读取， 发送一个 HTTP POST 到 /refresh 会导致 配置重新加载。   
Config Watch 自动 探测修改，并且reload 应用context   

---
#### How to activate

使用Consul Configuration， 需要加入starter， org.springframework.cloud:spring-cloud-starter-consul-config 

---
#### Spring Boot Config Data Import
Boot2.4 引入了 新的方法 来导入 配置数据，通过 spring.config.import， 现在，这个是默认的方式 从Consul获得 配置。  

Consul的 可选 连接配置：   
application.properties
```properties
spring.config.import=optional:consul:
```
这个会连接 Consul Agent在默认的地址 http://localhost:8500   
移除 optional: 前缀， 如果无法连接到Consul，会导致Consul Config 失败。   
改变Consul Config的连接配置： spring.cloud.consul.host, spring.cloud.consul.port, 或者 增加 host/port 对 到 spring.config.import 中，比如，spring.config.import=optional:consul:myhost:8500   
import中的地址 比 host port属性 优先

Consul Config 会尝试加载value 从 基于spring.cloud.consul.config.name (值默认是spring.application.name的值) 和spring.cloud.consul.config.default-context(默认是application) 的 4个 自动context 。

。。估计就是上面的   
config/testApp,dev/   
config/testApp   
config/application,dev/   
config/application
。。。

如果你要指明 context，你可以增加 信息 到 spring.config.import 中：   
application.properties
```properties
spring.config.import=optional:consul:myhost:8500/contextone;/context/two
```
这个会加载配置 只从 /contextone 和 /context/two

通过 spring.config.import 加载 SpringBootConfigData 的话，不需要 bootstrap.yml/properties 文件。   

---
#### Customizing

Consul Config 能自定义 通过下面的 属性：
```yaml
spring:
  cloud:
    consul:
      config:
        enabled: true
        prefix: configuration
        defaultContext: apps
        profileSeparator: '::'
```

如果你设置 spring.cloud.bootstrap.enabled=true 或 spring.config.use-legacy-processing=true 或 包含 spring-cloud-starter-bootstrap, 上面的配置 需要放在 bootstrap.yml，而不是application.yml   

enabled: 设置false，禁用Consul Config   
prefix: 设置 base文件夹 for 配置值   
defaultContext: 设置 所有应用使用的 文件夹名字   
profileSeparator: 设置 separator 的值，用来 分割 profiles   


---
#### Config Watch
Consul Config Watch 利用 Consul的能力 来 watch a key prefix。    
Config Watch 创建 阻塞Consul HTTP API 调用 来决定 是否有 和当前应用相关的配置数据 被修改。   
如果有新的配置数据， Refresh Event 被发布。这个等同于调用 /refresh actuator 端点。  

修改Config Watch 的频率，通过 spring.cloud.consul.config.watch.delay .默认 1000,单位ms。是 上次调用完以后 等待1s，然后开始下次调用。。。。 不是 上次开始后1秒后 再开始   

禁用 spring.cloud.consul.watch.enabled=false   

Watch 使用Spring TaskScheduler 来调度 对Consul的调用。 默认是一个 ThreadPoolTaskScheduler，且poolSize是1.   
要修改 TaskScheduler，创建TaskScheduler 类型的 bean，并且bean的名字是 ConsulConfigAutoConfiguration.CONFIG_WATCH_TASK_SCHEDULER_NAME 的值   

---
#### YAML or Properties with Config

保存大量properties，使用yaml或properties 格式 比 k-v对 更容易。   
设置 spring.cloud.consul.config.format 到 YAML 或 PROPERTIES。   
```yaml
spring:
  cloud:
    consul:
      config:
        format: YAML
```

如果你设置了 spring.cloud.bootstrap.enabled=true or spring.config.use-legacy-processing=true, or included spring-cloud-starter-bootstrap, 需要在 bootstrap.yml中设置

YAML 必须设置 Consul 中 合适的 data key。 默认值就像下面：
```
config/testApp,dev/data
config/testApp/data
config/application,dev/data
config/application/data
```

你可以保存一个 YAML 文档 在上面的任意一个key中。

你可以修改 data key，通过 spring.cloud.consul.config.data-key   

---

#### git2consul with Config

git2consul 是一个 Consul 社区工程，它会加载文件 从一个 git仓库，到 Consul中独立的key。   
默认下，key的名字就是文件的名字。   
.yml, .properties 文件 是支持的。   
需要设置 spring.cloud.consul.config.format 为 FILES   
```yaml
spring:
  cloud:
    consul:
      config:
        format: FILES
```

现有，下面的 keys 在 /config中， development profile， 一个名叫foo的 app：
```
.gitignore
application.yml
bar.properties
foo-development.properties
foo-production.yml
foo.properties
master.ref
```
下面的property source 会被创建：
```
config/foo-development.properties
config/foo.properties
config/application.yml
```

---
#### Fail Fast

如果consul 不可用，spring.cloud.consul.config.fail-fast=false 会导致 配置模块 log一条warn，而不是抛出异常   

如果使用了spring.cloud.bootstrap.enabled=true or spring.config.use-legacy-processing=true, or included spring-cloud-starter-bootstrap，上面的需要在bootstrap.yml中。

---
#### Consul Retry
如果Consul Agent 可以不可用，当你的app启动时， 你可以让你的app 失败后重新try。   

需要add spring-retry, spring-boot-starter-aop 到 classpath。   
默认行为是 重试6次， 初始间隔1000ms， 以后每次间隔在上次间隔上 *1.1。   
可以通过 spring.cloud.consul.retry.* 来配置。这个对 SpringCloudConsulConfig 和 Discovery registration 都有效

全面控制retry行为，通过 创建 RetryOperationsInterceptor 类型的 Bean，并且id是 consulRetryInterceptor。 SpringRetry 有一个 RetryInterceptorBuilder 来使得创建更简单。   

---
#### Spring Cloud Bus with Consul

#### How to Active

要使用Consul Bus，需要使用starter ： org.springframework.cloud:spring-cloud-starter-consul-bus

看SpringCloudBus (https://cloud.spring.io/spring-cloud-bus/) 文档，了解 有效的actuator端点， 如何发送自定义消息。

---

#### Circuit Breaker with Hystrix

能使用 SpringCloudNetflix 提供的 Hystrix Circuit Breaker, 通过增加 starter(spring-cloud-starter-hystrix) 到pom.xml 文件。     
Hystrix 不依赖 Netflix Discovery Client. @EnableHystrix 应该被标记到配置文件(通常是main class)。   
然后，被@HystrixCommand标记的 方法 会被 断路器 保护。    

查看文档了解更多：https://projects.spring.io/spring-cloud/spring-cloud.html#_circuit_breaker_hystrix_clients

---
#### Hystrix metrics aggregation with Turbine and Consul

Turbine (由SpringCloudNetflix 工程提供) 聚合多个 Hystrix metrics stream 实例，所以dashboard 能展现一个 聚合视图。   
Turbine 使用 DiscoveryClient 接口 来查找 相关实例。   
要使用Turbine with SpringCloudConsul，配置Turbine应用，类似下面：
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-netflix-turbine</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-consul-discovery</artifactId>
</dependency>
```

记住，Turbine 依赖 不是一个starter。  turbine-starter 包含了 对 Netflix Eureka的支持。

application.yml
```yaml
spring.application.name: turbine
applications: consulhystrixclient
turbine:
  aggregator:
    clusterConfig: ${applications}
  appConfig: ${applications}
```

clusterConfig, appConfig 这2个必须匹配，所以可以把它们的内容提取到一个 独立的 逗号分隔的 保存ServiceID的列表 的 属性   

Turbine.java
```
@EnableTurbine
@SpringBootApplication
public class Turbine {
    public static void main(String[] args) {
        SpringApplication.run(DemoturbinecommonsApplication.class, args);
    }
}
```

---
#### Configuration Properties

在附录可以看到所有的 Consul的配置属性










