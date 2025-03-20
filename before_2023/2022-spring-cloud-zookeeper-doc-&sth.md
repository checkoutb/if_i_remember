
https://docs.spring.io/spring-cloud/docs/2020.0.2/reference/htmlsingle/#spring-cloud-zookeeper



...2020.0.2 这个版本的 文档太烂了。。 坑。。
主要是 Sleuth，结果全是 Stream。。。。不是。应该是 工具的问题。 看起来是 每个module一个工程。但是我用的是 htmlsingle，转出来的根本不对。。
用html/ 的就可以了。。。


https://docs.spring.io/spring-cloud/docs/2020.0.2/reference/html/

https://docs.spring.io/spring-cloud-zookeeper/docs/3.0.2/reference/html/

。。确实，spring是 spring-cloud-zookeeper 一个工程，所以 这个工程有 独立的 文档。

有工具 把 这些文档全部加一起，生成一个 单页的。。 

我从未见过如此

---

https://docs.spring.io/spring-cloud-zookeeper/docs/3.0.2/reference/html/

为SBoot 应用提供了 Zookeeper 集成，通过自动配置 和绑定 到 Spring Environment 和其他Spring 变成模型。   

通过一些注解，你可以 在你的应用中 快速 启用和 配置 一般模式，构造大型分布式系统 with 基于zookeeper的组件。   
提供的模式包括 Service Discovery 和 配置， 工程也提供了 客户端的 LB 通过集成 SC LoadBalancer

---
#### 1 Quick Start
通过 SCZookeeper 来进行 服务发现 和分布式配置。   

首先，运行Zookeeper 在你的 机器。 然后你可以 访问它，使用它 作为一个 服务注册 和 配置源。

#### 1.1 Discovery Client Usage
依赖 spring-cloud-zookeeper-core, spring-cloud-zookeeper-discovery. 最简单的方式是增加：org.springframework.cloud:spring-cloud-starter-zookeeper-discovery   
我们建议使用 依赖管理 和 spring-boot-starter-parent:

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
      <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
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

gradle 配置
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
  implementation 'org.springframework.cloud:spring-cloud-starter-zookeeper-discovery'
  testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
dependencyManagement {
  imports {
    mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
  }
}
```

需要调整 Zookeeper 版本。

现在你可以创建一个 标准SBoot 应用， 比如下面的 HTTP Server：

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

当这个 HTTP Server 运行，它 默认连接到 在本地2181端口的 Zookeeper

application.yml
```yaml
spring:
  cloud:
    zookeeper:
      connect-string: localhost:2181
```

你现在可以使用 DiscoveryClient, @LoadBalanced RestTemplate, @LoadBalanced WebClient.Builder 来 检索服务 和实例数据 从 Zookeeper：

```java
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

---
#### 1.2 Distributed Configuration Usage

依赖 spring-cloud-zookeeper-core, spring-cloud-zookeeper-config.  
导入 org.springframework.cloud:spring-cloud-starter-zookeeper-config  

创建标准 SBoot 应用：
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

这个应用从 Zookeeper 检索 配置。

如果你使用了 Spring Cloud Zookeeper Config， 你需要设置 spring.config.import 属性 来 绑定到 Zookeeper。


---
#### 2 Install Zookeeper

SCZ 使用 Apache Curator。

如果你使用Zookeeper 3.4,你需要：
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zookeeper-all</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.12</version>
    <exclusions>
        <exclusion>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

3.5.x 不需要

---
#### 3 Service Discovery with Zookeeper

微服务架构的关键原则之一 就是 服务发现。   

Curator( java library for Zookeeper)， 提供了 服务发现。

---
#### 3.1 Activating
包含依赖： org.springframework.cloud:spring-cloud-starter-zookeeper-discovery。 允许自动配置 来设置 SCZ Discovery。   

对于web功能，你依然需要spring-boot-starter-web

---
#### Registering with Zookeeper

但一个客户端 注册到 zk， 它提供 它的metadata (比如host，post，id，name)    

下面是 一个 zk client 的例子：
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
上面是一个普通的 SBoot 应用。

如果zk 地址不是 localhost:2181 , 需要在 application.yml 指定。

application.yml
```yaml
spring:
  cloud:
    zookeeper:
      connect-string: localhost:2181
```

> 如果你使用 SCZ Config， 上面的配置 需要在 bootstrap.yml 中。

默认的 服务名，实例id，端口，是 ${spring.application.name}, Spring Context ID, ${server.port}

在classpath中 有 spring-cloud-starter-zookeeper-discovery 使得 app 既是 zk service(会注册自己) 也是 zk client(能查询zk 来定位其他服务)。

如果要禁用 zk discovery client， 设置 spring.cloud.zookeeper.discovery.enabled=false。

---
#### Using the DiscoveryClient

SC 支持 Feign， RestTemplate，WebFlux，使用 逻辑服务名 而不是 物理地址。

你可以使用 DiscoveryClient， 这个提供了一个 简单 API for discovery client 这不是Netflix特有的：

```java
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

---
#### 4 Using Spring Cloud Zookeeper with Spring Cloud Components

Feign, SC Gateway, SC LoadBalancer 都能 和 SCZ 一起工作。

---
#### 4.1 Spring Cloud LoadBalancer with Zookeeper

SCZ 提供了一个 SC LoadBalancer ServiceInstanceListSupplier 的实现。   
当你使用 spring-cloud-starter-zookeeper-discovery, SC LB 被自动配置 成 默认使用 ZookeeperServiceInstanceListSupplier。   

如果你之前使用 Zookeeper 中的 StickyRule， 它在当前 stack 中的替换是 SC LoadBalancer 的 SameInstancePreferenceServiceInstanceListSupplier

---
#### 5 Spring Cloud Zookeeper and Service Registry

SCZ 实现 ServiceRegistry接口， 使得 开发者 以编程的方式 注册任意服务。

ServiceInstanceRegistration 类提供了一个 builder() 方法 来创建 一个 Registration 对象，这个能被 ServiceRegistry 使用：

```java
@Autowired
private ZookeeperServiceRegistry serviceRegistry;

public void registerThings() {
    ZookeeperRegistration registration = ServiceInstanceRegistration.builder()
            .defaultUriSpec()
            .address("anyUrl")
            .port(10)
            .name("/a/b/c/d/anotherservice")
            .build();
    this.serviceRegistry.register(registration);
}
```
。。那么实际上会调用什么呢？ 就是 这个是注册，注册了什么？注册了 本地:ip/anyUrl 。？ 还是说 url 包含了 地址端口， 应该是 包含了地址端口。 这样的话，能 第三方代码 提供这种信息啊。 但是有port，所以感觉 应该是 ip 必须是本机， 所以 anyUrl 应该是 后缀。

---
#### Instance Status

Netflix Eureka 支持 有实例 是 OUT_OF_SERVICE 注册到 server。   
这些实例 不会返回 活动服务实例。   
这是对某些行为是有用的，比如 blue/green 发布。(Curator 服务发现 没有提供这种行为)   
灵活的payload的 优势 使得 SCZ 实现 OUT_OF_SERVICE 通过 更新一些 特定的 metadata ，然后 在SC LB的 ZookeeperServiceInstanceListSupplier 中 根据 那个 metadata 进行filter。  
ZookeeperServiceInstanceListSupplier 过滤 掉 所有 不 等 于 UP 的 非null 的实例状态。      
如果 实例状态 是 空，为了向后兼容，它被认为是 UP。   
要改变 实例的状态，发送POST with OUT_OF_SERVICE 到 ServiceRegistry 实例状态 actuator 端点：
```
$ http POST http://localhost:8081/service-registry status=OUT_OF_SERVICE
```

---
#### 6 Zookeeper Dependencies

下面包含了 如何使用 SCZ Dependencies
。。。SCZ Dependencies 应该是 一个 module。。。

---
#### 6.1 Using the Zookeeper Dependencies
SCZ 使你可以 通过 properties 提供 应用的 依赖。   
作为依赖，你可以了解 在zk中注册的其他应用，以及你希望通过Feign RestTemplate，WebFlux 调用的 应用。   

你可以使用 Zookeeper Dependency Watchers 功能来 控制和监控 你的依赖的状态。

---
#### 6.2 Activating Zookeeper Dependencies

包含 spring-cloud-starter-zookeeper-discovery ，来允许自动配置，来设置 SCZ Dependencies.   

即使你在 属性中提供了 依赖项，你可以关闭 依赖项，通过 spring.cloud.zookeeper.dependency.enabled 为false。 (默认 true)

---
#### Setting up Zookeeper Dependencies

在yaml中，默认 headers 展现为 headers map。   
有事，对依赖项的每次调用 都需要设置一些默认 header。 不需要在代码中做，你可以设置它们 在yaml 文件中，就像下面：
```yaml
headers:
    Accept:
        - text/html
        - application/xhtml+xml
    Cache-Control:
        - no-cache
```
上面的配置导致 增加 Accept，Cache-Control 头 及适当的 值列表。

---
#### 6.3.6 Required Dependencies

需要的 依赖，在yaml 中 表现为 required 属性  

如果你的应用启动时，需要 启动你的依赖项之一，你可以在 yaml 中 设置 required: true 属性。  

如果你的应用不能 把需要的依赖 地方化 在 boot时间内，它抛出一个异常。Spring Context 配置失败。   
换句话说， 你的应用 不能启动，如果 所需要的依赖 没有注册到 zk中。
。。。？就是 把 依赖注册到 zk，然后 其他应用启动的时候 拉去？ 就是类似 maven repository?

---
#### Stubs
你可以 为 包含依赖项存根的 jar 提供 以冒号分隔的 路径：

```
stubs: org.springframework:myApp:stubs
```

org.springframework 是 groupId   
myApp 是 artifactId      
stubs 是一个 classifier(分类器)。(stubs 是一个默认值)

因为 stubs 是默认的 classifier，上面的例子 也能写为：
```
stubs: org.springframework:myApp
```

---
#### 6.4 Configuring Spring Cloud Zookeeper Dependencies

你可以设置下面的 属性 来 启用或禁用 zk dependencies 功能：   
1. spring.cloud.zookeeper.dependencies: 如果不设置这个属性，你不能使用 zk Dependencies
2. spring.cloud.zookeeper.dependency.loadbalancer.enabled: 默认启用，开启 为zk 定制的 LB 策略，包括 ZookeeperServiceInstanceListSupplier,和基于依赖的 LB RestTemplate 配置。
3. spring.cloud.zookeeper.dependency.headers.enabled: 默认启用，注册一个 FeignBlockingLoadBalancerClient,用于自动根据它们在 Dependency 配置中的 版本 追加 合适的 header 和content type。没有这个配置，这2个参数 不起效
4. spring.cloud.zookeeper.dependency.resttemplate.enabled: 默认启用，激活时，这个属性修改 @LoadBalanced 注解的 RestTemplate 的 请求头，它会根据 dependency 配置中的 版本 来 传递 头和 content type。 没有这个配置，这2个参数不起效。

---
#### 7 Spring Cloud Zookeeper Dependency Watcher
Dependency Watcher 机制 让你 注册 listener 到你的 依赖。   
这个 观察者模式的 实现。   
当一个 依赖 改变了， 它的状态 (变成UP 或者 DOWN)，一些 自定义逻辑 可以被应用

---
#### 7.1 Activating

SCZ Dependencies 功能 需要被启用 以便让你 使用 Dependency Watcher 机制。

---
#### 7.2 Registering a Listener

为了注册listener，你需要实现 DependencyWatcherListener 接口， 注册它为 bean。   
接口里有一个方法：
```
void stateChanged(String dependencyName, DependencyState newState);
```

如果你想注册一个 listener for 特定的依赖， dependencyName 就是鉴别器。 newState提供你 关于 是否你的依赖被改为CONNECTED 或 DISCONNECTED 的信息

---
#### 7.3 Using the Presence Checker

和 Dependency Watcher 绑定在一起的 功能是 Presence Checker. 它让你 提供定制的行为，当你的应用启动时，来 根据你的 依赖的状态 作出反应。

抽象类 DependencyPresenceOnStartupVerifier 的默认实现是 DefaultDependencyPresenceOnStartupVerifier, 它的工作方式：
1. 如果 依赖被标记为 required，且不在 zk中， 当你的app 启动，它会抛出一个异常 且 关闭
2. 如果 依赖不是required，LogMissingDependencyChecker 打印日志 在WARN 级别。

由于 DefaultDependencyPresenceOnStartupVerifier 只有当 没有 DependencyPresenceOnStartupVerifier 类型的bean 存在时，才会注册，所以 这个功能 能重写。

---
#### 8 Distributed Configuration with Zookeeper

zk 提供了一个 hierarchical namespace，使得客户端 保存任意数据。   

SCZ Config 是一个 可选的 Config Server 和 Client   
配置在 bootstrap 阶段 被加载到 Spring Environment.   
默认，配置被存储在 /config 命名空间。   
多个 PropertySource 实例 被创建，基于 应用的名字 和 激活的 profile，来 模仿SC Config 解析属性的顺序。   
例如，一个 名为 testApp 的应用 有着 dev profile，会有下列的 property source 创建：
1. config/testApp,dev
2. config/testApp
3. config/application,dev
4. config/application

上面按照 优先级顺序。

配置在 应用的启动阶段 读取。 发送POST 到 /refresh 导致 配置 重新加载。   

观察配置命名空间 (zk支持的) 目前还没有实现。

---
#### 8.1 Activating

增加 spring-cloud-starter-zookeeper-config 依赖 来 启用 自动配置 来 配置 SCZ Config

---
#### 8.2 Spring Boot Config Data Import

SBoot 2.4 引入了 新方式 来导入 配置数据，通过 spring.config.import 属性。 这个是现在 从zk 获得配置的 默认方式   

zk的可选的连接：       
application.properties
```
spring.config.import=optional:zookeeper:
```

这个会连接zk 在默认地址 localhost:2181。   
修改zk的连接属性，通过设置 spring.cloud.zookeeper.connect-string 或者 增加连接string 到 spring.config.import，比如 spring.config.import=optional:zookeeper:myhost:2818。import 属性 优先于 content-string。

zk Config 尝试加载值 从4个 自动的上下文， 上下文基于 spring.cloud.zookeeper.config.name(默认 spring.application.name), 和 spring.cloud.zookeeper.config.default-context (默认 application)。        
如果你想要制定 上下文，你可以增加 信息到 spring.config.import 中。
application.properties
```properties
spring.config.import=optional:zookeeper:myhost:2181/contextone;/context/two
```
只会从 /contextone 和 /context/two 加载配置。

通过spring.config.import 导入 不需要在 bootstrap文件中 声明。

---
#### 8.3 Customizing
zk Config 可以自定义 通过下面的属性：
```yaml
spring:
  cloud:
    zookeeper:
      config:
        enabled: true
        root: configuration
        defaultContext: apps
        profileSeparator: '::'
```

|property|description|
|:---|:---|
|enable|设置为false，就 禁用 zk Config|     
|root|设置 基础命名空间 for 配置值|
|defaultContext|为所有应用使用的context设置名字|
|profileSeparator|设置分隔符 来分隔 profile name 在 有profile的property source 中|

如果你设置 spring.cloud.bootstrap.enabled=true 或 spring.config.use-legacy-processing=true.或 包含 spring-cloud-starter-bootstrap。那么上面的值 需要放在 bootstrap.yml 中。

---
#### 8.4 Access Control Lists (ACLs)

可以增加 验证信息 for zk ACLs 通过 调用 CuratorFramework bean 的 addAuthInfo 方法。   

一种方法是，提供你自己的 CuratorFramework bean，就像下面：
```java
@BoostrapConfiguration
public class CustomCuratorFrameworkConfig {

  @Bean
  public CuratorFramework curatorFramework() {
    CuratorFramework curator = new CuratorFramework();
    curator.addAuthInfo("digest", "user:password".getBytes());
    return curator;
  }

}
```

或者，你可以 往 现有的 CuratorFramework bean 中增加 证明：
```java
@BoostrapConfiguration
public class DefaultCuratorFrameworkConfig {

  public ZookeeperConfig(CuratorFramework curator) {
    curator.addAuthInfo("digest", "user:password".getBytes());
  }

}
```

这个bean 必须在 bootstrapping 阶段 创建。 通过@BootstrapConfiguration 和 把这些类变成 逗号分隔的list 作为 org.springframework.cloud.bootstrap.BootstrapConfiguration属性 放到 resources/META-INF/spring.factories 文件 可以 注册配置类 在 bootstrapping 阶段：

resources/META-INF/spring.factories
```properties
org.springframework.cloud.bootstrap.BootstrapConfiguration=\
my.project.CustomCuratorFrameworkConfig,\
my.project.DefaultCuratorFrameworkConfig
```






