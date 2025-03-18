Spring-doc-1.9-Annotation
2022年2月15日
13:45

## 1.9 基于注解的容器配置

注解是否优于xml？
简短的答案是：视情况而定。
长答案是，每种方法都有其优点和缺点，通常由开发者决定 哪种策略更适合他们。
由于它们的定义方式，注释在其声明中提供了大量上下文，从而使配置更简洁。
xml擅长在不触及源代码或重新编译它们的情况下连接组件。
一些开发人员更喜欢在源代码附近进行装配，而另一些开发人员则认为带注释的类不再是POJO，此外，配置变得分散且更难控制。

无论如何选择，spring可以同时适应这2种风格，甚至可以混合在一起。

值得指出的是，通过JavaConfig(1.12章)选项，spring允许以非侵入性的方式使用注解，而无需触及目标组件的源代码，并且在工具方面，spring tool for eclipse 支持所有配置样式。

基于注解的配置提供了 xml配置的替代方案，它依赖字节码元数据来 装配组件，而不是尖括号声明。
开发人员不使用xml来描述bean连接，而是通过在相关类，方法或字段声明上使用注解将配置移动到组件类本身。

AutowiredAnnotationBeanPostProcessor，将BeanPostProcessor 和 注解 结合使用是 扩展IoC 容器的常用方法。

> 注解注入在xml注入之前执行， 因此，如果2种注入都有，xml配置 覆盖 注解注入。

你可以注册 post-processors 就像独立的bean定义， 也可以通过基于xml的 标签的 隐式注册(注意context 命名空间)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
         https://www.springframework.org/schema/beans/spring-beans.xsd
         http://www.springframework.org/schema/context
         https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

</beans>
```

context:annotation-config, 隐式注册了下面 post-processors:
ConfigurationClassPostProcessor
AutowiredAnnotationBeanPostProcessor
CommonAnnotationBeanPostProcessor
PersistenceAnnotationBeanPostProcessor
EventListenerMethodProcessor

。。5.3.5 没有 PersistenceAnnotationBeanPostProcessor。这个类应该不是spring-core 的。。

。。而且下面的 @Required需要的 RequiredAnnotationBeanPostProcessor, 看类注释，是这个标签自动注册的，但是 5.1开始 deprecated了，不知道 deprecated以后 是否还自动注册。。

。。5.3.5 不注册 RequiredAnnotationBeanPostProcessor 了。。

。。https://docs.spring.io/spring-framework/docs/5.0.1.RELEASE/spring-framework-reference/core.html#beans-annotation-config

。。里面写了： (The implicitly registered post-processors include AutowiredAnnotationBeanPostProcessor, CommonAnnotationBeanPostProcessor, PersistenceAnnotationBeanPostProcessor, as well as the aforementioned RequiredAnnotationBeanPostProcessor.)

。。g了啊。。。那岂不是5.3 @Required 无用了？  这个 被 deprecated 了。。。
。。。cao，5.3.5 也说了。。只要多看几行。。。
。。5.3 建议使用 @Autowired(required=false)

context:annotation-config，仅在 定义它的 应用上下文中 查找bean的 注解。

这意味着，如果你将 context:annotation-config 放到 DispatcherServlet 的 WebApplicationContext 中，它只检查 controller 的  @Autowired 的bean。

---
1.9.1 @Required

这个注解应用到 bean 属性的setter方法：
。。这个只能 注解在方法上。。自己看源码

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Required
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

这个注解 表明 必须在配置时通过bean 定义中的 显式属性值 或通过自动装配 来填充 受影响的bean属性。如果受影响的bean 属性没有被填充，则容器将抛出异常。

这允许 急切和明确的失败，避免了稍后的 NullPointer。我们仍然建议将 断言放入bean类本身(如init方法)。即使你在容器外部使用类，这样做也会强制执行这些必须的引用和值。

RequiredAnnotationBeanPostProcessor 必须被注册为bean，用来支持 @Required 注解。

@Required 和 RequiredAnnotationBeanPostProcessor， 从5.1开始 被 Deprecated 了。

推荐使用 构造器注入 for required 设置 (或自定义的 InitializingBean.afterPropertiesSet,或自定义的 @PostConstruct 方法。 或者setter里判断null)

---
1.9.2  @Autowired

JSR 330 的 @Inject 能用于替换 spring的 @Autowired 注解。

你可以应用 到 构造器：
```java
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

从4.3开始，构造器上的 @Autowired 注解 不再是必须的，如果 目标bean 只有一个构造器。
但是，如果有多个构造器可用，并且没有设置 主/默认 构造器，则必须使用 @Autowired 对至少一个 构造器进行注解，以指示容器使用哪一个。

你可以应用 @Autowired 到 传统的 setter：
```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

应用到 方法，有着 任意名字(。。估计指方法名)和 多个参数的方法：
```java
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

应用到 属性，甚至可以将其与构造器混合使用。
```java
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    private MovieCatalog movieCatalog;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

。。。如果这4个地方 都是同一个 属性，最终 是谁的值？。。不，好像注入的都是同一个东西。。。

确保你的 目标组件 (如 MovieCatalog，CustomerPreferenceDao) 始终 和你 @Autowired 注入点的类型 一致。否则，注入时，可能由于 运行时由于 "no type match found" 错误而失败。

对于通过xml定义的bean 或通过路径扫描找到的 组件类，容器通常预先知道 具体类型。但是对于 @Bean 工厂方法，你需要确保 声明的返回类型 有足够的 表现力。 对于 实现多个接口的组件 或 可能由其实现类型引用的组件，请考虑在你的工程方法中 声明 最具体的返回类型 (至少与引用你的bean的注入点所要求的一样具体)

将@Autowired 添加在 数组 的 属性或方法 来指示 spring从 AppCtx提供特定类型的 所有bean：
```java
public class MovieRecommender {

    @Autowired
    private MovieCatalog[] movieCatalogs;

    // ...
}
```

也可以应用于 有类型的集合：
```java
public class MovieRecommender {

    private Set<MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Set<MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```

你的目标bean 可以实现 Ordered 接口，或使用 @Order 或 @Priority (。。javax的)，如果你希望 数组或list 中的 item 被排序。 否则，它们的顺序是 容器中 bean的 注册顺序。

你可以在 目标类级别 和 @Bean 方法 上使用 @Order，可能用于单个bean定义(在使用相同bean类的 多个定义的情况下)。 @Order 值 可能会影响注入点的 优先级，但不会影响 单例启动顺序，这个顺序是由 依赖关系 和 @DependsOn 来决定的。

标准的 javax.annotation.Priority 注解 在 @Bean 上不可用，因为它不能声明在方法上。。它的语义可以通过 @Order 结合 @Primary 对每个类型的 单个bean进行 建模。

只要预期的 键类型是 字符串，即使是 有类型的 Map 也可以自动装配。map的value 包括预期类型的 所有bean，key包括相应的bean名字：
```java
public class MovieRecommender {

    private Map<String, MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Map<String, MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```
。。。？但是这里 不是数组啊，怎么包括所有bean？。。bean id 是唯一的。所以可以

默认下，自动装配失败，当没有匹配的 候选bean可用时。 在声明的 数组，集合或map的情况下，至少需要一个匹配的元素。

默认行为是 对待 被注解的方法 和 属性 就像 指示了必须的依赖。你可以改变这种行为，通过required 属性设置为false。spring能跳过 找不到依赖的 注入点。

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired(required = false)
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

一个 非required 的方法 不会被调用，如果 找不到它需要的某个依赖。
这种情况下，非required 的属性 也不会填充字段，而是保留它的默认值。

注入到 构造器的和工厂方法的参数 是特殊的情况，因为 @Autowired中的 required属性 具有不同的 含义，由于spring的 构造器解析算法 可能会处理多个构造器。

默认情况下，构造器和工厂方法 参数是 required 的，但是 在 单构造器的场景下 有一些特殊规则，例如 多元素的注入点(数组，集合，map) 解析为 空实例 如果没有bean 可用。 这允许一种通用的实现模式，其中所有依赖都可以在 唯一的 多参数构造器中声明---例如，声明为没有@Autowired注解的 单个公共构造器。

。。。我...

只有一个给定bean类的 构造器可以声明为 @Autowired 并且required是true， 指明 这个构造器用于spring的自动装配。

因此，如果 required 保持默认值(true)，只有一个 构造器可以被注解。 如果多个 构造器 被注解，它们必须被声明 required=false 才能被视为 自动装配的候选者( 类似xml中的 autowire=constructor)。

容器中 该bean 可用的依赖项数量最多的 构造函数 将被选择。 如果没有候选者，则将使用 主/默认 构造器(如果存在)。
类似地，如果一个类声明了多个构造器，但都没有@Autowired，将使用 primary/default 构造器。
如果一个类只声明了一个构造器，即使没有注解，也会一直使用这个构造器。
注意：被注解的构造器 不是必须public的。

> 建议使用 @Autowired的 required 属性，而不是 在setter上使用已经废弃的 @Required。

你也可以通过Java8的 java.util.Optional 表达特定依赖项 是非required的。
```java
public class SimpleMovieLister {

    @Autowired
    public void setMovieFinder(Optional<MovieFinder> movieFinder) {
        ...
    }
}
```

从5.0开始，你也可以使用 @Nullable
```java
public class SimpleMovieLister {

    @Autowired
    public void setMovieFinder(@Nullable MovieFinder movieFinder) {
        ...
    }
}
```

还可以将 @Autowired 用于 众所周知的 可解析依赖项的接口： BeanFactory,AppCtx,Environment,ResourceLoader,ApplicationEventPublisher,MessageSource。

这些接口和它们的子接口 (如，ConfigurableApplicationContext,  ResourcePatternResolver) 被自动解析，无需特殊设置。 下面是 自动装配 ApplicationContext的例子：

```java
public class MovieRecommender {

    @Autowired
    private ApplicationContext context;

    public MovieRecommender() {
    }

    // ...
}
```

@Autowired，@Inject，@Value，@Resource 通过 BeanPostProcessor 实现 来处理。 这意味这你不能应用这些 注解 到你自己的 BeanPostProcessor，BFactoryPP 。这些类型必须通过 xml 或@Bean 方法 显示 装配。

---
1.9.3 Fine-tuning(微调) Annotation-based Autowiring with @Primary

因为 自动装配 by type 会导致 多个 候选者，不得不在选择时 做更多控制。

一种方法是 spring 的 @Primary 注解。 @Primary 表示当多个bean成为自动装配 到 单值依赖项的 候选者时，应该优先考虑的bean。如果多个候选者中存在 确切的一个Primary bean，这个bean 会成为 自动装配的 值。

```java
@Configuration
public class MovieConfiguration {

    @Bean
     @Primary
    public MovieCatalog firstMovieCatalog() { ... }

    @Bean
    public MovieCatalog secondMovieCatalog() { ... }

    // ...
}
```

。。。没有看到是 默认 byType 还是 byName， 看起来 是byType， 但是 xml 的autowire 默认 no。。

。。看 AutowiredAnnotationBeanPostProcessor ,里面有 findAutowireCandidates方法, 参数是 Class，但是 没有地方调用这个方法啊。。。

。。。byName 怎么会有多个 候选者呢。。。haha。。

with上面的配置，下面的 MovieRecommender 被注入了 firstMovieCatalog
```java
public class MovieRecommender {

    @Autowired
    private MovieCatalog movieCatalog;

    // ...
}
```

相应的 xml 定义：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
         https://www.springframework.org/schema/beans/spring-beans.xsd
         http://www.springframework.org/schema/context
         https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog" primary="true">
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

---
1.9.4 微调基于注解的自动装配 通过 Qualifiers

@Primary 是一个有效的方法用于 有多个 by type 的自动装配的 候选者时 能决定一个 Primary候选者。
当你需要在 进行选择时 进行更多的控制，你可以使用spring的 @Qualifier 。
你可以 将 qualifier 值 与特定参数相关联，缩小类型匹配的范围，以便为每个参数选择特定的bean。
最简单的情况是，一个简单的描述性值。

```java
public class MovieRecommender {

    @Autowired
    @Qualifier("main")
    private MovieCatalog movieCatalog;

    // ...
}
```

@Qualifier 也可以用于 单个构造器参数，方法参数上。
```java
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(@Qualifier("main") MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

下面是 等价的 xml配置
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
         https://www.springframework.org/schema/beans/spring-beans.xsd
         http://www.springframework.org/schema/context
         https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier value="main"/>

        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier value="action"/>

        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```
。。上面2个是把 具有 main/action 的 bean 注入到 构造器参数。
。。。不不不，上面是 为所在bean 设置 这个bean的 qualifier 值，用于 被匹配。
。。注解怎么 声明 bean 自己的 qualifier？ ... @Qualifier 可以放在type 上。
。。@Qualifier   有没有  声明当前bean的  别名的  意思？

对于 fallback match，bean名字被视为 默认qualifier 值。因此，你可以使用 main 作为id 来代替 嵌入的 qualifier 标签，也有相同的匹配结果。

但是，尽管你可以使用这个约定 来通过 name 来引用 一个特定的bean，但@Autowired 从根本上来说是 带有可选 语义限定符 的 类型驱动注入。

这意味着，限定符值，即使使用bean 名称fallback，也总是在类型匹配的集合上具有缩小的语义。它们不会在语义上表达 对唯一bean id 的引用。好的限定符值是 main,EMEA,persistent,这种独立于 bean id 的 string，这可以在匿名bean定义 的情况下自动生成。

限定符也使用于 有类型的集合，如 ```Set<MovieCatalog>```。在这种情况下，根据声明的限定符，所有匹配的bean 都注入到集合中。这意味着限定符不是必须唯一的。相反，它们构成过滤标准。例如，你可以定义多个 MovieCatalog bean，它们有着相同的 限定符 "action"，所有这些bean 都会被注入到 @Qualifier("action") 注释的 ```Set<MovieCatalog>``` 中。

---
在类型匹配的候选者中，让限定符针对bean的name 进行选择，不需要在注入点使用 @Qualifier。

如果没有其他解析指示(如qualifier，primary标记)，对于非唯一依赖的情况，spring将注入点名称(字段名称或参数名称) 与目标bean名称匹配，并选择同名的候选者(如果有的话)。

---

也就是说，如果你打算使用 by name 的 注解驱动注入，不要 主要使用@Autowired，即使它能够在类型匹配候选者中 按bean名称进行选择。
相反，使用 JSR-250 @Resource，它在语义上 定义为 通过其唯一名称 标识 目标组件，声明的类型与匹配过程无关。
@Autowired 有相当不同的语义： 在 类型选择 候选者后， 限定符的值 仅在候选者中考虑。

对于本身定义为 集合，map，数组 的bean，@Resource是一个好的方案，它通过唯一name 引用 特定的集合或数组 bean。

从4.3开始，你也 可以匹配 集合，map，数组， 通过spring的 @Autowired 类型匹配算法，只要元素类型信息 保留在 @Bean 返回类型签名 或集合继承层次结构中即可。这种情况下，你可以使用 qualifier 值 在 类型相同的 集合中 进行选择。

从4.3开始，@Autowired 还考虑注入的 自引用。注意，自注入 是一种 fallback。对其他组件的常规依赖始终具有优先权。 从这个意义上说，自引用 不参与 常规的候选者选择，因此不可能是primary。相反，它们总是以最低的优先级结束。

在实践中，你应该仅将 自ref 用作最后的手段(例如，通过bean的食物代理在同一个实例上调用其他方法)。
在这种情况下，考虑将受影响的方法分解为单独的委托bean。 或者，你可以使用 @Resource，它可以通过唯一名称 获取 代理到当前bean的 代理。

尝试在同一个配置类上 注入 @Bean方法的结果实际上也是一种 自ref 方案。 要么在实际需要的方法前面中 lazy解析 这些ref，那么将受影响的 @Bean 方法 声明为 static，将它们与包含的配置类实例 及其生命周期 解耦。 否则，仅在 fallback 阶段 考虑此类bean，而将其他配置类上的 匹配bean 做为 主要候选者(如果可用)。

@Autowired 适用于 属性，构造器，多参数方法，允许在参数级别通过限定符缩小范围。
相反，@Resource 仅支持 字段和 单参数的setter。
因此，如果你的注入目标 是构造器 或多参数方法，你应该坚持使用 限定符。

你可以创建你自己的自定义限定符注释。为了这么做，你需要定义一个 注解 并在你的定义中 使用 @Qualifier。
```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Genre {

    String value();
}
```

使用自定义标识符到 自动装配的 属性和参数：
```java
public class MovieRecommender {

    @Autowired
    @Genre("Action")
    private MovieCatalog actionCatalog;

    private MovieCatalog comedyCatalog;

    @Autowired
    public void setComedyCatalog(@Genre("Comedy") MovieCatalog comedyCatalog) {
        this.comedyCatalog = comedyCatalog;
    }

    // ...
}
```

然后，你可以提供 信息 for 候选者bean定义。你可以增加 qualifier 标签 作为 bean标签的 子元素。然后指定 type 和 value 来匹配你的 自定义 限定符注解。 type 需要使用 类的全民，或者，如果不存在name冲突，为了方便，你可以使用 短类名。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
         https://www.springframework.org/schema/beans/spring-beans.xsd
         http://www.springframework.org/schema/context
         https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="Genre" value="Action"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="example.Genre" value="Comedy"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

在Classpath Scanning and Managed Components 中， 你可以看到基于注释的替代方法，以在xml中提供 限定符 元数据。

有时，使用 不带value 的注解 可能已经够了。 当注解服务于更通用的目的并且可以应用于多种不同类型的依赖项时，这可能很有用。例如，你可以提供一个离线目录，当没有可用的Internet连接时 可以搜索改目录。

首先，定义简单的注解：
```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Offline {

}
```

增加注解到 自动装配的 属性。
```java
public class MovieRecommender {

    @Autowired
    @Offline
    private MovieCatalog offlineCatalog;

    // ...
}
```

现在，bean定义只需要一个 限定符type。
```xml
<bean class="example.SimpleMovieCatalog">
    <qualifier type="Offline"/>
    <!-- inject any dependencies required by this bean -->
</bean>
```

除了 附加或替代 简单value 属性，你还可以定义 接受 命名的属性 的自定义限定符注释。
如果随后要在自动装配的 属性或参数 上指定多个属性值。bean定义必须匹配所有的属性 才会被认为是 候选者。
例如，考虑以下注解定义：
```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface MovieQualifier {

    String genre();

    Format format();
}
```

这里的Format 是 枚举：
```java
public enum Format {
    VHS, DVD, BLURAY
}
```

要自动装配的属性 使用 自定义限定符进行注释，并包括2个属性：genre，format。

```java
public class MovieRecommender {

    @Autowired
    @MovieQualifier(format=Format.VHS, genre="Action")
    private MovieCatalog actionVhsCatalog;

    @Autowired
    @MovieQualifier(format=Format.VHS, genre="Comedy")
    private MovieCatalog comedyVhsCatalog;

    @Autowired
    @MovieQualifier(format=Format.DVD, genre="Action")
    private MovieCatalog actionDvdCatalog;

    @Autowired
    @MovieQualifier(format=Format.BLURAY, genre="Comedy")
    private MovieCatalog comedyBluRayCatalog;

    // ...
}
```

最终，bean定义应该包含 匹配的限定符值。
这个例子还演示了你可以使用bean 标签的 meta 子标签 而不是 qualifier标签。
如果可用，qualifier 标签 和它的属性 优先，如果没有qualifier 标签，自动装配机制回退到 meta标签 提供的值。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
         https://www.springframework.org/schema/beans/spring-beans.xsd
         http://www.springframework.org/schema/context
         https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="MovieQualifier">
            <attribute key="format" value="VHS"/>
            <attribute key="genre" value="Action"/>
        </qualifier>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="MovieQualifier">
            <attribute key="format" value="VHS"/>
            <attribute key="genre" value="Comedy"/>
        </qualifier>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <meta key="format" value="DVD"/>
        <meta key="genre" value="Action"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <meta key="format" value="BLURAY"/>
        <meta key="genre" value="Comedy"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

</beans>
```

---
1.9.5 使用泛型作为 自动装配的 限定符。

除了@Qualifier 注解之外，你还可以使用 Java泛型类型 作为限定 的隐式形式。
假如，你有如下配置：

```java
@Configuration
public class MyConfiguration {

    @Bean
    public StringStore stringStore() {
        return new StringStore();
    }

    @Bean
    public IntegerStore integerStore() {
        return new IntegerStore();
    }
}
```

假设前面的bean 实现了一个 泛型接口(即 Store<String> 和 Store<Integer>), 你可以 @Autowired 到 Store 接口，并将泛型作为限定符。

```java
@Autowired
private Store<String> s1; // <String> qualifier, injects the stringStore bean

@Autowired

private Store<Integer> s2; // <Integer> qualifier, injects the integerStore bean

```

泛型限定符 也可以应用到 自动装配的 list，map，数组。
```java
// Inject all Store beans as long as they have an <Integer> generic
// Store<String> beans will not appear in this list
@Autowired
private List<Store<Integer>> s;
```

---

#### 1.9.6 使用 CustomAutowireConfigurer

是一个 BeanFactoryPostProcessor，使得你注册你自己的 自定义 限定符注解类型，即使 它们没有被@Qualifier。

```xml
<bean id="customAutowireConfigurer"

        class="org.springframework.beans.factory.annotation.CustomAutowireConfigurer">

    <property name="customQualifierTypes">
        <set>
            <value>example.CustomQualifier</value>
        </set>
    </property>
</bean>
```

AutowireCandidateResolver 决定 自动装配 候选者 by：
1. 每个bean定义的 autowire-candidate 值。
2. beans 标签的 任何 default-autowire-candidates pattern
3. @Qualifier 和 任何 注册在CustomAutowireConfigurer 中的自定义注解。

当多个bean 有资格成为 自动装配候选者，按后续的规则决定一个"primary": 如果有一个且只有一个bean 的 primary 属性是 true，就被选择。

---

### 1.9.7 通过@Resource 注入

spring 支持 JSR-250 的 @Resource 在 属性 或 属性的setter。

@Resource 接受一个 name 属性，默认下，spring 将该值 解释为 要注入的bean的 name。换句话说，它是 by-name 的语义。

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource(name="myMovieFinder")
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

如果name 没有显式声明，默认的名字 源自 属性名 或setter名。
下面会获得 名为 movieFinder 的bean 注入到 setter
```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

注解提供的 name 通过 CommonAnnotationBeanPostProcessor 知道的 AppCtx 解析为 bean名字。
如果你显式配置了spring的 SimpleJndiBeanFactory，name 可以通过 JDNI 解析。
但是，我们建议 你依赖默认的行为，并为使用 spring的 jndi 查找功能 来保留间接级别。

@Resource没有显式 name的排他情况下， 和 @Autowired 类似，@Resource 查找类型匹配 而不是 name，并解析 众所周知的 可解析依赖：BeanFactory,AppCtx,ResourceLoader,ApplicationEventPublisher,MessageSource 接口。

。。。？上面说了 没有name，就 属性的名字，这里又说 没有name的情况下，类型匹配。。
。。好像也没有问题，  没有name的情况下，就 先by-type，然后用 默认的name 进行筛选。。
。。不对，name是唯一的， 直接用name就可以了。
。。看网上， 写name，就 用name，找不到就炸。
。。。。。。不写name，就用默认 name，找不到就 by type。

In the exclusive case of @Resource usage with no explicit name specified,
估计是 exclusive case 。但是找不到这种。。 难道是 Resource的 shareable ？,看注释，也不像啊。。。。

> 下面的例子，customerPreferenceDao 属性 先搜索 名字是customPreferenceDao 的bean，然后 回退成 primary type match。

```java
public class MovieRecommender {

    @Resource
    private CustomerPreferenceDao customerPreferenceDao;

    @Resource
    private ApplicationContext context;

    public MovieRecommender() {
    }

    // ...
}
```

---

### 1.9.8 使用 @Value

通常用来注入 外部properties。

```java
@Component
public class MovieRecommender {

    private final String catalog;

    public MovieRecommender(@Value("${catalog.name}") String catalog) {
        this.catalog = catalog;
    }
}
```

with 下面的配置：
```java
@Configuration
@PropertySource("classpath:application.properties")
public class AppConfig { }
```
with 下面的 application.properties 文件：
```properties
catalog.name=MovieCatalog
```

spring提供了一个  默认的宽松嵌入式值解析器(。。我在想这个是不是类名。。)。它将尝试解析property值，如果无法解析，property name(如  ${catalog.name}) 将作为值注入。

如果要严格 控制不存在的值，则应该声明 PropertySourcesPlaceholderConfigurer bean
```java
@Configuration
public class AppConfig {

    @Bean

    public static PropertySourcesPlaceholderConfigurer propertyPlaceholderConfigurer() {

        return new PropertySourcesPlaceholderConfigurer();
    }
}
```

通过 JavaConfig 配置 PropertySourcesPlaceholderConfigurer, @Bean 的方法 必须 static。

使用上面的配置，确保 spring 初始化时失败，如果${} 无法被解析。它也可以使用 setPlaceholderPrefix,setPlaceholderSuffix,setValueSeparator 方法来自定义 placeholder。

spring  boot 配置了默认的 PropertySourcesPlaceholderConfigurer bean，这个bean会从 application.properties, application.yml 文件中 读取 property。

spring的内置的converter 允许 自动处理简单类型转换(例如 转为Integer 或 int)。逗号分隔的多个值 可以自动转为 String数组。

可以提供默认值：

```java
@Component
public class MovieRecommender {

    private final String catalog;

    public MovieRecommender(@Value("${catalog.name:defaultCatalog}") String catalog) {

        this.catalog = catalog;
    }
}
```

spring 的 BeanPostProcessor 使用 ConversionService 来处理 @Value 中的string 值转换为目标类型。

如果你想为你自己的自定义类型提供转换支持，你可以提供你自己的 ConversionService bean。

```java
@Configuration
public class AppConfig {

    @Bean
    public ConversionService conversionService() {

        DefaultFormattingConversionService conversionService = new DefaultFormattingConversionService();

        conversionService.addConverter(new MyCustomConverter());
        return conversionService;
    }
}
```

当@Value 包含一个 SpEL，值会在运行时动态计算。

```java
@Component
public class MovieRecommender {

    private final String catalog;

    public MovieRecommender(@Value("#{systemProperties['user.catalog'] + 'Catalog' }") String catalog) {

        this.catalog = catalog;
    }
}
```

SpEL 也支持 更复杂的数据结构

```java
@Component
public class MovieRecommender {

    private final Map<String, Integer> countOfMoviesPerCatalog;

    public MovieRecommender(

            @Value("#{{'Thriller': 100, 'Comedy': 300}}") Map<String, Integer> countOfMoviesPerCatalog) {

        this.countOfMoviesPerCatalog = countOfMoviesPerCatalog;
    }
}
```

---

1.9.9 使用 @PostConstruct 和 @PreDestory

CommonAnnotationBeanPostProcessor 不仅可以识别 @Resource 注解，还可以识别 JSR-250 生命周期注解：PostConstruct, PreDestory。

在Spring 2.5引入，对这些注解的支持 提供了 生命周期回调机制 的替代方案，在 initialization callbacks 和 destruction callbacks 中。

如果在AppCtx中 注册了 CommonAnnotationBeanPostProcessor，则在生命周期中与对应的spring生命周期接口方法或显示声明的回调方法相同的点 调用带有这些注解的方法。

下面是 缓存在初始化时预先填充，并在销毁时清除：

```java
public class CachingMovieLister {

     @PostConstruct
    public void populateMovieCache() {
        // populates the movie cache upon initialization...
    }

     @PreDestroy
    public void clearMovieCache() {
        // clears the movie cache upon destruction...
    }
}
```

like @Resource， @PostConstruct 和 @PreDestory 注解 是jdk6 to 8 的 标准java lib 的一部分。

但是在jdk9，整个javax.annotation 包 从 核心java 分离出去，jdk11 核心java 删除了 javax.annotation。 如果需要，javax.annotation-api 现在在maven可以用。

===========================

---

1.10 Classpath 扫描和 管理的组件

本章的大部分例子 使用xml来配置 用于生成 BeanDefinition 的配置元数据。

上节展示了如何 提供配置元数据 通过 源代码层的注解。但是，bean定义还是xml的，注解只是依赖注入。

本节描述 通过扫描classpath 来隐式检测候选者 的选项。这消除了使用xml来进行 bena定义的 需要。你可以使用注解( 如@Component )，AspectJ 类型表达式，或你自定义的过滤条件 来选择 哪些类有注册到容器的 bean 定义。

从spring 3.0开始，Spring JavaConfig 工程 提供的许多功能是 core spring的一部分。

---

1.10.1 @Component 和 进一步/更多 Stereotype(刻板印象/老一套/原型) 注解

@Repository 是marker for 任何满足 存储库 角色或原型的 类 (就像Data Access Object)。这个marker的用途之一是 自动翻译异常。

spring提供了更多的原型注解：@Component，@Service，@Controller。

@Component是 spring管理的组件的 通用原型。

@Repository，@Service，@Controller 是 @Component 针对更具体的用例(持久层，服务层，表示层) 的特化。使用这3个，比@Component，可以被工具或 切面 更好的适配。例如，这些注解是切入点的理想目标。 这3个，在未来，spring可以会加以额外的语义。

如果你为你的服务层 在@Component 和@Service 间做选择，@Service 显然是一个更好的选择。同样，如前所述，@Repository 已被支持作为 持久层中 自动转换异常的 标记。

---

1.10.2 使用 meta-annotation 和 组合后的 注解

spring的许多注解 可以 在你自己的代码中 用作 元-注解。元-注解是可以注解到其他注解的注解。

例如，@Service 被 @Component 注解：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Service {

    // ...
}
```

@Component 导致 @Service 被和@Component 相同的方式 处理。

你还可以组合 元注解来创建 组合注解。例如 spring mvc 中的 @RestController 由 @Controller 和 @ResponseBody 组成。

另外，组合注解可以选择 从元注解中 重新声明属性 以允许 自定义。当你只想公开 元注解属性的 子集时，这可能很有用。例如，spring的@SessionScope 注解将 scope的名字 硬编码为 session，但依然允许自定义 proxyMode。下面是 @SessionScope 的定义：

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Scope(WebApplicationContext.SCOPE_SESSION)
public @interface SessionScope {

    /**
     * Alias for {@link Scope#proxyMode}.
     * <p>Defaults to {@link ScopedProxyMode#TARGET_CLASS}.
     */
    @AliasFor(annotation = Scope.class)
    ScopedProxyMode proxyMode()  default ScopedProxyMode.TARGET_CLASS;

}
```

你可以使用 @SessionScope 不需要声明proxyMode

```java
@Service
@SessionScope
public class SessionScopedService {
    // ...
}
```

你也可以重写 proxyMode 的值：

```java
@Service
@SessionScope(proxyMode = ScopedProxyMode.INTERFACES)
public class SessionScopedUserService implements UserService {
    // ...
}
```

---

1.10.3 自动检测类 和 注册bean定义

spring可以自动 detect 原型的类，且注册成 AppCtx中 对应的 BeanDefinition 实例。

例如，下面2个类可以被 自动侦测。

```java
@Service
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

```java
@Repository
public class JpaMovieFinder implements MovieFinder {
    // implementation elided for clarity
}
```

为了自动侦测到这些类 且 注册对应的bean，你需要增加 @ComponentScan 到你的 @Configuration 类，basePackages 属性 是 这2个类的 公共父包。(或者你可以 指定一个 逗号/分号/空格 分隔的包名的列表)

```java
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    // ...
}
```

为了简洁，上面的例子可以使用 value 属性。( 即 @ComponentScan("org.example") )

下面是对应的xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
         https://www.springframework.org/schema/beans/spring-beans.xsd
         http://www.springframework.org/schema/context
         https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.example"/>

</beans>
```

>  使用 context:component-scan 会隐式启用 context:annotation-config 的功能。

classpath package的扫描 需要 在classpath中存在对应的目录。当你使用Ant构建jar时，确保 你没有激活jar 任务的 files-only 开关。

此外，在某些环境下，类路径目录可能 不会根据安全策略公开。

在jdk9的 module path (Jigsaw) 上，spring的类路径扫描按照预期工作。但是，确保你的组件类 在你的 module-info 描述符中导出。如果你希望spring 调用你的类的 非public 成员，请确保它们是 'opened' (即，它们在你的 module-info 描述中 使用 opens 声明 而不是 exports 声明)

此外，当你使用 component-scan 元素时，AutowiredAnnotationBeanPostProcessor, CommonAnnotationBeanPostProcessor 都被隐式包含。这意味着，这2个组件会被自动检测并装配在一起 --- 这些不需要xml中提供任何配置元数据。

你可以禁止注册 AutowiredAnnotationBeanPostProcessor, CommonAnnotationBeanPostProcessor，通过 包含 值为false的 annotation-config 属性

---

1.10.4 使用Filter来自定义扫描

默认下，@Component，@Repository,@Service,@Controller,@Configuration，或 被@Component注解的自定义注解 注解的 类 是only被检测到的候选组件。

但是，你可以修改和扩展这种行为，通过自定义filter。增加它们作为@ComponentScan 注解的 includeFilters 或 excludeFilters。(或作为 context:include-filter 或 context:exclude-filter 子标签 of context:component-scan ).

每个filter元素 需要 type 和 expression 属性，下面是FilterType类的描述：

| Filter Type         | Example Expression         | 描述                                        |
| ------------------- | -------------------------- | ------------------------------------------- |
| annotation(default) | org.example.SomeAnnotation | 在目标组件的 类型级别 存在或 元-存在的 注解 |
| assignable          | org.example.SomeClass      | 目标组件 实现/继承 的 接口/类               |
| aspectj             | org.example..*Service+     | 一个AspectJ 类型表达式，被目标组件匹配      |
| regex               | org\\.example\\.Default.*  | 一个RE，被目标组件的类名匹配                |
| custom              | org.example.MyTypeFilter   | spring的 TypeFilter接口的 自己的实现。      |

下面的例子展示了 配置 忽略所有@Repository注解，使用"stub" repositories

```java
@Configuration
@ComponentScan(basePackages = "org.example",

        includeFilters = @Filter(type = FilterType.REGEX, pattern = ".*Stub.*Repository"),

        excludeFilters = @Filter(Repository.class))
public class AppConfig {
    // ...
}
```

下面是等价的xml：

```xml
<beans>
    <context:component-scan base-package="org.example">
        <context:include-filter type="regex"
                expression=".*Stub.*Repository"/>
        <context:exclude-filter type="annotation"
                expression="org.springframework.stereotype.Repository"/>
    </context:component-scan>
</beans>
```

你也可以禁用 默认的filter 通过设置 useDefaultFilters=false 到@ComponentScan 或 将use-default-filters="false" 作为属性放置到 component-scan 标签。 这会禁用 @Component,@Repository,@Service,@Controller,@RestController,@Configuration 注解或元注解的 自动检测。

---

1.10.5 定义bean metadata within Components

spring组件 也可以贡献 bean定义元数据到容器。你可以这样做 with 在 @Configuration 类中 @Bean 用来定义bean 元数据。

```java
@Component
public class FactoryMethodComponent {

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

    public void doWork() {
        // Component method implementation omitted
    }
}
```

上面的类是一个 spring 组件，并且在doWork 方法中有 特定于app的代码。

但是，它还提供了一个bean定义，该定义具有 工厂方法 引用到方法publicInstance()。@Bean注解标识工厂方法和其他bean定义属性，例如通过@Qualifier 的限定符值。其他方法层级可以用的注解有@Scope，@Lazy 和自定义限定符注解。

@Lazy 除了用于组件初始化之外，还可以将 @Lazy 放到 @Autowired或@Inject 的注入点上。这种情况下，它会导致 延迟解析的 代理 的注入。然后，这种代理方法非常有限。对于复杂的lazy交互，特别是结合optional依赖，我们建议使用 ```ObjectProvider<MyTargetBean>```

如前所述，自动装配的 属性和 方法 也被支持  with 额外支持@Bean 方法的自动装配。

```java
@Component
public class FactoryMethodComponent {

    private static int i;

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

    // use of a custom qualifier and autowiring of method parameters
     @Bean
    protected TestBean protectedInstance(
            @Qualifier("public") TestBean spouse,
            @Value("#{privateInstance.age}") String country) {
        TestBean tb = new TestBean("protectedInstance", 1);
        tb.setSpouse(spouse);
        tb.setCountry(country);
        return tb;
    }

    @Bean
    private TestBean privateInstance() {
        return new TestBean("privateInstance", i++);
    }

    @Bean
    @RequestScope
    public TestBean requestScopedInstance() {
        return new TestBean("requestScopedInstance", 3);
    }
}
```

这个例子 自动装配 string 方法参数 country 为 一个名为privateInstance  的 bean 的 age属性的值。#{}是SpEL; @Value注解，表达式解析器 被预先配置 在为解析表达式文本时 查询bean的名字。

从spring 4.3 开始，你还可以声明 一个工厂方法参数，它的类型是 InjectionPoint (或它的子类：DependencyDescriptor) 来访问 触发当前bean创建 的 请求注入点。请注意，这仅适用于bean实例的实际创建，不适用于已有实例的注入。因此，此功能对于prototype scope 是非常有意义的。对于其他scope，工程方法只看到 在给定scope中 触发bean实例创建的 注入点。

```java
@Component
public class FactoryMethodComponent {

    @Bean @Scope("prototype")
    public TestBean prototypeInstance(InjectionPoint injectionPoint) {

        return new TestBean("prototypeInstance for " + injectionPoint.getMember());

    }
}
```

常规spring component 中 @Bean 方法的处理方式 和 Spring @Configuration类中 对应方法不同。不同之处在于 @Component 没有通过 cglib  增强来拦截方法和字段的 调用。cglib 代理是调用@Configuration 类中@Bean方法中的方法或字段创建协作对象的bean 元数据引用的方法。此类方法不是使用正常的Java语义调用的，而是通过容器来提供spring bean的常规生命周期管理和代理，即使当 ref到其他bean 通过编程方式调用 @Bean方法。相反，在普通的@Component类中 调用@Bean方法中的方法或字段 具有标准的java语义，没有特殊的cglib处理或其他约束。

。。好像是 cglib 不会增强 @Component， 会增强@Configuration

你可以声明@Bean 方法为 static，从而 允许 它们被call 而不需要 创建 它们包含的配置class(their containing configuration class) 为实例。这在定义 post-processor bean (如，BFPP 或 BeanPostProcessor) 是特别有意义，因为此类bean在容器生命周期的早期被初始化，并且应该避免在那个时候触发配置的其他部分。

由于技术限制，对static  @Bean 方法的调用 永远不会被 容器拦截，即使在@Configuration 类中 也不会被拦截：cglib 子类化 只能覆盖 非static方法。因此，直接调用另一个 @Bean 方法 具有标准的java 语义，从而导致直接从 工厂方法 本身返回一个独立实例。

。。没有办法拦截，就不能 cache， 所以 就看 这个工厂 是每次 return new，还是 单例设计模式。

@Bean方法的Java语言可见性 不会立刻影响spring容器中生成的bean定义。你可以在非@Configuration 类中自由声明 你认为合适的工厂方法，也可以在任何地方声明静态方法。但是@Configuration 类中的 常规@Bean 方法 需要是可覆盖的，也就是说，它们不能被声明为private 或final。

。。。？后半段应该是说 cglib 通过 继承来增强，所以需要 能被覆盖。 前半段呢？ 非@Configuration 的 工厂有什么用。。@Bean 必须在 @Configuration中吧？难道可以不是？不是，必须是@Component中。看网上的例子，@Configuration 的 被cglib 了， @Component 的没有被cglib。。

在给定组件或配置类的基类，或给定组件或配置类 实现的 接口的 default方法 的 @Bean 会被发现。这为复杂的配置安排 提供了很大的灵活性，甚至从4.2起 可以通过 jdk8的 default方法 实现多重继承。

最后，单个类 可以为 同一个 bean保存多个@Bean方法，作为多个工厂方法的排列，根据运行时可用的依赖关系使用。这与在其他配置场景中选择"最贪婪"的构造器或工厂方法 的算法相同：在构造时 选择具有最多可满足依赖项的 变体，类似于容器如何在多个 @Autowired构造器之间进行选择。

---

1.10.6 Naming Autodetected Components

当一个组件 在 扫描过程中 被自动侦测到，它的bean name 通过 scanner知道的BeanNameGenerator 策略生成。默认下，任何spring 原型注解(Component,Repository,Service,Controller)  包含的 name属性的值 被用作 bean的名字。

如果注解不包含 name属性 或任何其他检测到的组件( 比如由自定义过滤器发现的组建)，默认bean name generator 返回 非首字母大写的 非限定类名。例如，if下面组件被自动发现，名字会是 myMovieLister, movieFinderImpl:

```java
@Service("myMovieLister")
public class SimpleMovieLister {
    // ...
}
```

```java
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```

如果你不想依赖 默认 bean-naming 策略，你可以提供一个自定义 bean命名策略。第一，实现BeanNameGenerator接口，且确保有一个 默认的 无参构造器。然后提供 类的全名 在 配置scanner时，就像下面

```java
@Configuration

@ComponentScan(basePackages = "org.example", nameGenerator = MyNameGenerator.class)

public class AppConfig {
    // ...
}
```

```xml
<beans>
    <context:component-scan base-package="org.example"
        name-generator="org.example.MyNameGenerator" />
</beans>
```

如果你有命名冲突，由于多个自动检测的组件 有相同的 非限定类名 (如，相同的类名，不同的包)，你可能需要配置一个 BeanNameGenerator，它默认对生成的bean 使用 全限定名。从5.2.3开始，FullyQualifiedAnnotationBeanNameGenerator 能被用于 这个场景。

作为一个一般规则，考虑指明 注解的 name，当其他组件可能 明确引用到它时。另一方面，只要容器负责装配，自动生成的名字就足够了。

---

1.10.7 Providing a Scope for Autodetected Components

与一般spring 管理的组件一样，自动检测组件的默认 和最常见的scope 是singleton。但是，有时你需要不同的scope，可以通过@Scope。你可以在注解中提供scope 的名字。

```java
@Scope("prototype")
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```

@Scope 仅在具体的bean类(for 注解的组件) 或 工厂方法( for @Bean 方法) 上 进行自省(introspected)。和xml bean定义 相比，没有bean定义 继承的概念，并且类级别的继承层次和 元数据无关。

与为这些scope 而 预构建的注解(。。request,session,application,websocket)一样，你也可以使用spring的 元注解 方法来编写自己的 scope 注解： 例如 用 @Scope("prototype") 元注解 注解的 自定义注解，也可能声明自定义范围代理模式。

要为scope解析提供自定义策略而不是依赖基于注解的方法，你可以实现ScopeMetadataResolver 接口。确保包含一个默认的 无参构造器。然后你可以在配置 scanner时提供完全限定的类名：

```java
@Configuration

@ComponentScan(basePackages = "org.example", scopeResolver = MyScopeResolver.class)

public class AppConfig {
    // ...
}
```

```xml
<beans>

    <context:component-scan base-package="org.example" scope-resolver="org.example.MyScopeResolver"/>

</beans>
```

当使用某些 非单例  scope时，可能需要为 scoped 对象 生成代理。为此，在component-scan中有 scoped-proxy 属性，3个可能值是 no，interfaces,targetClass。例如，下面会生成标准jdk动态代理：

```java
@Configuration

@ComponentScan(basePackages = "org.example", scopedProxy = ScopedProxyMode.INTERFACES)

public class AppConfig {
    // ...
}
```

```java
<beans>

    <context:component-scan base-package="org.example" scoped-proxy="interfaces"/>

</beans>
```

---

1.10.8 Providing Qualifier Metadata with Annotations

@Qualifier 被讨论过了。那节中的例子是 使用@Qualifier注解 和自定义限定符注解 来解析 自动装配候选者时 提供细粒度的控制。因为这些例子是基于xml bean定义，所以通过使用 xml的 bean标签的 qualifier 或 meta 子标签 在候选bean定义上提供 限定符元数据。当依赖 classpath scanning for 组件的自动侦测时，你可以在 候选类上为限定符元数据 提供 类型级别的注解。以下3个例子就是这种技术：

```java
@Component
@Qualifier("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
```

```java
@Component
@Genre("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
```

```java
@Component
@Genre("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
```

与大多数基于注解的替代方案一样，请记住 注解元数据 绑定到 类定义本身，而xml允许相同类型的 多个bean 提供限定符元数据的变体，因为元数据 是根据 每个实例 而不是每个类 提供的。

---

1.10.9 Generating an Index of Candidate Components

虽然 classpath 扫描非常快，但可以通过在 编译时 创建静态候选列表 来提高大型app的 启动性能。在这种模式下， 作为 组件扫描的所有模块都必须使用这种机制。

你现有的@ComponentScan或 context:component-scan 指令必须保持不变才能请求context 以扫描某些包中的候选者。当AppCtx检测到这种的索引时，它会自动使用它，而不是扫描类路径。

为了生成索引，增加一个额外的依赖到每个包含组件扫描指令目标的组建的模块。

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-indexer</artifactId>
        <version>5.3.15</version>
        <optional>true</optional>
    </dependency>
</dependencies>
```

gradle 4.5及之前

```
dependencies {
    compileOnly "org.springframework:spring-context-indexer:5.3.15"
}
```

gradle 4.6 及之后

```
dependencies {
    annotationProcessor "org.springframework:spring-context-indexer:5.3.15"
}
```

spring-context-indexer 生成一个 META-INF/spring.components 文件 到jar中。

当在你的IDE中以这种模式工作，spring-context-indexer 必须被注册为一个 注解 processor 来确保 idnex是 最新的，当候选者组建更新时。

当在classpath 中找到 META-INF/spring.components 文件时，索引被自动激活。如果索引对某些库(或用例) 部分可用，但无法为整个应用构建，你可以通过设置spring.index.ignore 为true 来回退到常规的 路径安排( 就像根本没有index 一样)，这个spring.index.ignore可以作为 JVM 系统属性 或 通过SpringProperties 机制

已使用 Microsoft OneNote 2016 创建。