https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#null-safety



java已经提供了 null安全 在它的类型系统中(。。指Optional？)。 spring 提供了注解，让你 声明api 和 属性 是否可以为null：

1. @Nullable，指明 参数，返回值，属性 可以 为null。
2. @NonNull，指示 参数，返回值，属性 不可以为null ( 在@NonNullApi，@NonNullFields 中不需要 这个注解)
3. @NonNullApi，在package层声明 non-null 是 参数 和 返回值 的默认语义。
4. @NonNullFields，在package层 声明 non-null 是 属性的默认语义。

。。package-info.java ?



---

8 Data Buffers and Codecs

java NIO 提供 ByteBuffer, 但许多库构建自己的字节缓冲区，如Netty的ByteBuf，Undertow 的 XNIO，Jetty使用池化字节缓冲区 等。

spring-core 提供了一组抽象来处理字节缓冲区api：

1. DateBufferFactory，抽象了 data buffer 的创建
2. DataBuffer，代表 byte buffer，它可能是池化的
3. DataBufferUtils，提供工具方法 for data buffers
4. Codecs，decode 或 encode data buffer streams 到 更高层对象。



---

8.1 DataBufferFactory

以下面2种方式之一来创建 data buffer：

1. 分配一个新的data buffer，可选地指定容量上限。
2. 包装一个已有的 byte[], java.nio.ByteBuffer，使用 DataBuffer 装饰给定数据 ，不涉及分配。

WebFlux 应用不会直接创建 DataBufferFactory，但通过 ServerHttpResponse 或 ClientHttpRequest 访问 DataBufferFactory。工厂的类型取决于底层的 client 或server，如 NettyDataBufferFactory for Reactor Netty， DefaultDataBufferFactory for others。



---

8.2 DataBuffer

DataBuffer 接口提供 类似 java.nio.ByteBuffer 的操作，但也 带有一些额外的好处，其中一些是受 Netty的 ByteBuf 启发的，下面是一部分好处：

1. 在独立位置读和写，不需要调用 flip() 来切换读和写
2. 和StringBuilder 一样 按需扩容
3. 通过 PooledDataBuffer 池化缓冲区和引用计数
4. 以java.nio.ByteBuffer, InputStream, OutputStream 的形式查看缓冲区。
5. 确定给定字节的索引或最后一个索引。



---

8.3 PooledDataBuffer

就像java 的ByteBuffer 的JavaDoc，byte buffer 可以是直接 或 非直接的。直接buffer可以驻留在java 堆之外，这消除了本地I/O 操作进行复制的需要。这使得直接缓冲区对于通过套接字接受和发送数据特别有用，但它们的创建和释放成本也更高，这导致了 池化缓冲区的想法。

retain()，release()

不要直接在PooledDataBuffer 上操作，大多时候使用 DataBufferUtils 提供的简单的方法来 release 或 retain 。

---



8.4 DataBufferUtils

提供了一些工具方法来操作data buffer：

1. 如果底层byte buffer api支持，则将 byte buffer stream 加入到 单个缓冲区 通过可能的0拷贝。例如，通过复合缓冲区。
2. 将 InputStream 或 NIO Channel 转换为 ```Flux<DataBuffer>```，反之亦然。将Publisher《DateBuffer》 转为OutputStream 或 NIO Channel
3. 如果缓冲区是 PooledDataBuffer 的实例，则release 或 retain DataBuffer。
4. 跳过或从字节流中获取，直到特定的字节数。



---

8.5 Codecs

org.springframework.core.codec 包提供下面的 策略接口：

1. Encoder 来将 ```Publisher<T>``` 编码为 date buffer stream。
2. Decoder 来将 ```Publisher<DataBuffer>``` 解码为 更高级别的对象流。



spring-core 模块提供了 byte[], ByteBuffer, DataBuffer, Resource, String 的 encoder 和 decoder。   

spring-web 提供了 Jackson JSON, Jackson Smile, JAXB2, Protocol Buffers 和其他的 encoder 和 decoder。



---

8.6 使用DataBuffer

当使用data buffer，特别小心确保 buffer 被release，由于它们可能是pooled的。

Decoder是在创建更高级别对象之前最后读取输入数据缓冲区的，因此它必须按照如下方式释放它们：

1. 如果Decoder 只是简单的读取 每个input buffer，并且现在准备释放，那么可以通过 DataBufferUtils.release(dataBuffer).
2. 如果Decoder 使用Flux 或 Mono 操作，如flatMap，reduce，和其它的在内部 预取 和 缓存数据项，或者正在使用诸如filter,skip 和其他省略项的操作符，则必须将 doOnDiscard(PooledDataBuffer.class, DateBufferUtils::release) 添加到 组合链中 以确保此类缓冲区 在被丢弃前释放，丢弃可能是由于错误或取消信号导致的。
3. Decoder 以其他任何方式保留一个或多个数据缓冲区，它必须确保在完全读取时释放它们，或者在缓存数据缓冲区被读取和释放之前发生错误或取消信号的情况下。



DateBufferUtils.join 提供了一种将数据缓冲区流 聚合到单个数据缓冲区中的 安全有效的方法，同样，skipUntilByteCount, takeUntilByteCount 是 decoder使用的附加安全方法。



```java
DataBuffer buffer = factory.allocateBuffer();
boolean release = true;
try {
    // serialize and populate buffer..
    release = false;
}
finally {
    if (release) {
        DataBufferUtils.release(buffer);
    }
}
return buffer;
```

Encoder 的消费者有责任 释放它收到的 data buffer。





---

9 Logging

从5.0开始，spring有自己的 Common Logging bridge 实现，在 spring-jcl模块中。

这个实现检查 log4j 2.X 和 slf4j 1.7 是否存在于 classpath，使用 先找到的那个实现。

如果2个都没有，fall back是 java平台的 核心logging ( 被称为JUL，java.util.logging)。



通过org.apache.commons.logging.LogFactory:

```java
public class MyBean {
    private final Log log = LogFactory.getLog(getClass());
    // ...
}
```





---

10 附录

---

10.1 XML Schemas

---

10.1.1 util Schema

##### `util:constant`

```xml
<bean id="..." class="...">
    <property name="isolation">
        <util:constant static-field="java.sql.Connection.TRANSACTION_SERIALIZABLE"/>
    </property>
</bean>
```



##### `util:property-path`

```xml
<!-- target bean to be referenced by name -->
<bean id="testBean" class="org.springframework.beans.TestBean" scope="prototype">
    <property name="age" value="10"/>
    <property name="spouse">
        <bean class="org.springframework.beans.TestBean">
            <property name="age" value="11"/>
        </bean>
    </property>
</bean>

<!-- results in 10, which is the value of property 'age' of bean 'testBean' -->
<bean id="testBean.age" class="org.springframework.beans.factory.config.PropertyPathFactoryBean"/>
```

```xml
<!-- target bean to be referenced by name -->
<bean id="testBean" class="org.springframework.beans.TestBean" scope="prototype">
    <property name="age" value="10"/>
    <property name="spouse">
        <bean class="org.springframework.beans.TestBean">
            <property name="age" value="11"/>
        </bean>
    </property>
</bean>

<!-- results in 10, which is the value of property 'age' of bean 'testBean' -->
<util:property-path id="name" path="testBean.age"/>
```



###### `util:property-path`

```xml
<!-- target bean to be referenced by name -->
<bean id="person" class="org.springframework.beans.TestBean" scope="prototype">
    <property name="age" value="10"/>
    <property name="spouse">
        <bean class="org.springframework.beans.TestBean">
            <property name="age" value="11"/>
        </bean>
    </property>
</bean>

<!-- results in 11, which is the value of property 'spouse.age' of bean 'person' -->
<bean id="theAge"
        class="org.springframework.beans.factory.config.PropertyPathFactoryBean">
    <property name="targetBeanName" value="person"/>
    <property name="propertyPath" value="spouse.age"/>
</bean>
```

```xml
<!-- results in 12, which is the value of property 'age' of the inner bean -->
<bean id="theAge"
        class="org.springframework.beans.factory.config.PropertyPathFactoryBean">
    <property name="targetObject">
        <bean class="org.springframework.beans.TestBean">
            <property name="age" value="12"/>
        </bean>
    </property>
    <property name="propertyPath" value="age"/>
</bean>
```

```xml
<!-- results in 10, which is the value of property 'age' of bean 'person' -->
<bean id="person.age"
        class="org.springframework.beans.factory.config.PropertyPathFactoryBean"/>
```

```xml
<bean id="..." class="...">
    <property name="age">
        <bean id="person.age"
                class="org.springframework.beans.factory.config.PropertyPathFactoryBean"/>
    </property>
</bean>
```



##### `util:properties`

```xml
<!-- creates a java.util.Properties instance with values loaded from the supplied location -->
<bean id="jdbcConfiguration" class="org.springframework.beans.factory.config.PropertiesFactoryBean">
    <property name="location" value="classpath:com/foo/jdbc-production.properties"/>
</bean>
```

```xml
<!-- creates a java.util.Properties instance with values loaded from the supplied location -->
<util:properties id="jdbcConfiguration" location="classpath:com/foo/jdbc-production.properties"/>
```



##### `util:list`

```xml
<!-- creates a java.util.List instance with values loaded from the supplied 'sourceList' -->
<bean id="emails" class="org.springframework.beans.factory.config.ListFactoryBean">
    <property name="sourceList">
        <list>
            <value>pechorin@hero.org</value>
            <value>raskolnikov@slums.org</value>
            <value>stavrogin@gov.org</value>
            <value>porfiry@gov.org</value>
        </list>
    </property>
</bean>
```

```xml
<!-- creates a java.util.List instance with the supplied values -->
<util:list id="emails">
    <value>pechorin@hero.org</value>
    <value>raskolnikov@slums.org</value>
    <value>stavrogin@gov.org</value>
    <value>porfiry@gov.org</value>
</util:list>
```

```xml
<util:list id="emails" list-class="java.util.LinkedList">
    <value>jackshaftoe@vagabond.org</value>
    <value>eliza@thinkingmanscrumpet.org</value>
    <value>vanhoek@pirate.org</value>
    <value>d'Arcachon@nemesis.org</value>
</util:list>
```



##### `<util:map/>`

```xml
<!-- creates a java.util.Map instance with values loaded from the supplied 'sourceMap' -->
<bean id="emails" class="org.springframework.beans.factory.config.MapFactoryBean">
    <property name="sourceMap">
        <map>
            <entry key="pechorin" value="pechorin@hero.org"/>
            <entry key="raskolnikov" value="raskolnikov@slums.org"/>
            <entry key="stavrogin" value="stavrogin@gov.org"/>
            <entry key="porfiry" value="porfiry@gov.org"/>
        </map>
    </property>
</bean>
```

```xml
<!-- creates a java.util.Map instance with the supplied key-value pairs -->
<util:map id="emails">
    <entry key="pechorin" value="pechorin@hero.org"/>
    <entry key="raskolnikov" value="raskolnikov@slums.org"/>
    <entry key="stavrogin" value="stavrogin@gov.org"/>
    <entry key="porfiry" value="porfiry@gov.org"/>
</util:map>
```

```xml
<util:map id="emails" map-class="java.util.TreeMap">
    <entry key="pechorin" value="pechorin@hero.org"/>
    <entry key="raskolnikov" value="raskolnikov@slums.org"/>
    <entry key="stavrogin" value="stavrogin@gov.org"/>
    <entry key="porfiry" value="porfiry@gov.org"/>
</util:map>
```



##### `<util:set/>`

```xml
<!-- creates a java.util.Set instance with values loaded from the supplied 'sourceSet' -->
<bean id="emails" class="org.springframework.beans.factory.config.SetFactoryBean">
    <property name="sourceSet">
        <set>
            <value>pechorin@hero.org</value>
            <value>raskolnikov@slums.org</value>
            <value>stavrogin@gov.org</value>
            <value>porfiry@gov.org</value>
        </set>
    </property>
</bean>
```

```xml
<!-- creates a java.util.Set instance with the supplied values -->
<util:set id="emails">
    <value>pechorin@hero.org</value>
    <value>raskolnikov@slums.org</value>
    <value>stavrogin@gov.org</value>
    <value>porfiry@gov.org</value>
</util:set>
```

```xml
<util:set id="emails" set-class="java.util.TreeSet">
    <value>pechorin@hero.org</value>
    <value>raskolnikov@slums.org</value>
    <value>stavrogin@gov.org</value>
    <value>porfiry@gov.org</value>
</util:set>
```



---

10.1.2 aop Schema

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- bean definitions here -->

</beans>
```



---

10.1.3 context Schema

##### `<property-placeholder/>`

```
${…}
```



##### `<annotation-config/>`

自动发现bean 的class 的注解：

1. spring的 @Configuration 模型
2. @Autowired / @Inject, @Value, @Lookup
3. JSR-250: @Resource,@PostConstruct,@PreDestroy
4. JAX-WS @WebServiceRef, EJB 3 @EJB
5. JPA @PersistenceContext, @PersistenceUnit
6. @EvenListener



##### `<component-scan/>`



##### `<load-time-weaver/>`

##### `<spring-configured/>`

##### `<mbean-export/>`



---

10.1.4 beans Schema

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="foo" class="x.y.Foo">
        <meta key="cacheName" value="foo"/> 
        <property name="name" value="Rick"/>
    </bean>

</beans>
```



---

10.2 XML Schema Authoring

```xml
<myns:dateformat id="dateFormat"
    pattern="yyyy-MM-dd HH:mm"
    lenient="true"/>
```



```xml
<?xml version="1.0" encoding="UTF-8"?>
<xsd:schema xmlns="http://www.mycompany.example/schema/myns"
        xmlns:xsd="http://www.w3.org/2001/XMLSchema"
        xmlns:beans="http://www.springframework.org/schema/beans"
        targetNamespace="http://www.mycompany.example/schema/myns"
        elementFormDefault="qualified"
        attributeFormDefault="unqualified">

    <xsd:import namespace="http://www.springframework.org/schema/beans"/>

    <xsd:element name="dateformat">
        <xsd:complexType>
            <xsd:complexContent>
                <xsd:extension base="beans:identifiedType"> 		<!-- 。。。 -->
                    <xsd:attribute name="lenient" type="xsd:boolean"/>
                    <xsd:attribute name="pattern" type="xsd:string" use="required"/>
                </xsd:extension>
            </xsd:complexContent>
        </xsd:complexType>
    </xsd:element>
</xsd:schema>
```



```xml
<bean id="dateFormat" class="java.text.SimpleDateFormat">
    <constructor-arg value="yyyy-MM-dd HH:mm"/>
    <property name="lenient" value="true"/>
</bean>
```



---

10.2.2 `NamespaceHandler`



10.2.3 `BeanDefinitionParser`

```java
AbstractSingleBeanDefinitionParser
```



10.2.4 注册处理器和schema

##### `META-INF/spring.handlers`

```
http\://www.mycompany.example/schema/myns=org.springframework.samples.xml.MyNamespaceHandler
```

##### META-INF/spring.schemas

```
http\://www.mycompany.example/schema/myns/myns.xsd=org/springframework/samples/xml/myns.xsd
```



---

#### 10.2.5. Using a Custom Extension in Your Spring XML Configuration

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:myns="http://www.mycompany.example/schema/myns"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.mycompany.example/schema/myns http://www.mycompany.com/schema/myns/myns.xsd">

    <!-- as a top-level bean -->
    <myns:dateformat id="defaultDateFormat" pattern="yyyy-MM-dd HH:mm" lenient="true"/> 

    <bean id="jobDetailTemplate" abstract="true">
        <property name="dateFormat">
            <!-- as an inner bean -->
            <myns:dateformat pattern="HH:mm MM-dd-yyyy"/>
        </property>
    </bean>

</beans>
```



---

##### Nesting Custom Elements within Custom Elements

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:foo="http://www.foo.example/schema/component"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.foo.example/schema/component http://www.foo.example/schema/component/component.xsd">

    <foo:component id="bionic-family" name="Bionic-1">
        <foo:component name="Mother-1">
            <foo:component name="Karate-1"/>
            <foo:component name="Sport-1"/>
        </foo:component>
        <foo:component name="Rock-1"/>
    </foo:component>

</beans>
```

```java
public class Component {

    private String name;
    private List<Component> components = new ArrayList<Component> ();

    // mmm, there is no setter method for the 'components'
    public void addComponent(Component component) {
        this.components.add(component);
    }

    public List<Component> getComponents() {
        return components;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

```java
public class ComponentFactoryBean implements FactoryBean<Component> {

    private Component parent;
    private List<Component> children;

    public void setParent(Component parent) {
        this.parent = parent;
    }

    public void setChildren(List<Component> children) {
        this.children = children;
    }

    public Component getObject() throws Exception {
        if (this.children != null && this.children.size() > 0) {
            for (Component child : children) {
                this.parent.addComponent(child);
            }
        }
        return this.parent;
    }

    public Class<Component> getObjectType() {
        return Component.class;
    }

    public boolean isSingleton() {
        return true;
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>

<xsd:schema xmlns="http://www.foo.example/schema/component"
        xmlns:xsd="http://www.w3.org/2001/XMLSchema"
        targetNamespace="http://www.foo.example/schema/component"
        elementFormDefault="qualified"
        attributeFormDefault="unqualified">

    <xsd:element name="component">
        <xsd:complexType>
            <xsd:choice minOccurs="0" maxOccurs="unbounded">
                <xsd:element ref="component"/>
            </xsd:choice>
            <xsd:attribute name="id" type="xsd:ID"/>
            <xsd:attribute name="name" use="required" type="xsd:string"/>
        </xsd:complexType>
    </xsd:element>

</xsd:schema>
```



```java
public class ComponentNamespaceHandler extends NamespaceHandlerSupport {

    public void init() {
        registerBeanDefinitionParser("component", new ComponentBeanDefinitionParser());
    }
}
```



```java
public class ComponentBeanDefinitionParser extends AbstractBeanDefinitionParser {

    protected AbstractBeanDefinition parseInternal(Element element, ParserContext parserContext) {
        return parseComponentElement(element);
    }

    private static AbstractBeanDefinition parseComponentElement(Element element) {
        BeanDefinitionBuilder factory = BeanDefinitionBuilder.rootBeanDefinition(ComponentFactoryBean.class);
        factory.addPropertyValue("parent", parseComponent(element));

        List<Element> childElements = DomUtils.getChildElementsByTagName(element, "component");
        if (childElements != null && childElements.size() > 0) {
            parseChildComponents(childElements, factory);
        }

        return factory.getBeanDefinition();
    }

    private static BeanDefinition parseComponent(Element element) {
        BeanDefinitionBuilder component = BeanDefinitionBuilder.rootBeanDefinition(Component.class);
        component.addPropertyValue("name", element.getAttribute("name"));
        return component.getBeanDefinition();
    }

    private static void parseChildComponents(List<Element> childElements, BeanDefinitionBuilder factory) {
        ManagedList<BeanDefinition> children = new ManagedList<BeanDefinition>(childElements.size());
        for (Element element : childElements) {
            children.add(parseComponentElement(element));
        }
        factory.addPropertyValue("children", children);
    }
}
```

```
# in 'META-INF/spring.handlers'
http\://www.foo.example/schema/component=com.foo.ComponentNamespaceHandler
# in 'META-INF/spring.schemas'
http\://www.foo.example/schema/component/component.xsd=com/foo/component.xsd
```

##### 

```xml
<bean id="checkingAccountService" class="com.foo.DefaultCheckingAccountService"
        jcache:cache-name="checking.account">
    <!-- other dependencies here... -->
</bean>
```

```java
public class JCacheInitializer {

    private String name;

    public JCacheInitializer(String name) {
        this.name = name;
    }

    public void initialize() {
        // lots of JCache API calls to initialize the named cache...
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>

<xsd:schema xmlns="http://www.foo.example/schema/jcache"
        xmlns:xsd="http://www.w3.org/2001/XMLSchema"
        targetNamespace="http://www.foo.example/schema/jcache"
        elementFormDefault="qualified">

    <xsd:attribute name="cache-name" type="xsd:string"/>

</xsd:schema>
```

```java
public class JCacheNamespaceHandler extends NamespaceHandlerSupport {

    public void init() {
        super.registerBeanDefinitionDecoratorForAttribute("cache-name",
            new JCacheInitializingBeanDefinitionDecorator());
    }

}
```





```java
public class JCacheInitializingBeanDefinitionDecorator implements BeanDefinitionDecorator {

    private static final String[] EMPTY_STRING_ARRAY = new String[0];

    public BeanDefinitionHolder decorate(Node source, BeanDefinitionHolder holder,
            ParserContext ctx) {
        String initializerBeanName = registerJCacheInitializer(source, ctx);
        createDependencyOnJCacheInitializer(holder, initializerBeanName);
        return holder;
    }

    private void createDependencyOnJCacheInitializer(BeanDefinitionHolder holder,
            String initializerBeanName) {
        AbstractBeanDefinition definition = ((AbstractBeanDefinition) holder.getBeanDefinition());
        String[] dependsOn = definition.getDependsOn();
        if (dependsOn == null) {
            dependsOn = new String[]{initializerBeanName};
        } else {
            List dependencies = new ArrayList(Arrays.asList(dependsOn));
            dependencies.add(initializerBeanName);
            dependsOn = (String[]) dependencies.toArray(EMPTY_STRING_ARRAY);
        }
        definition.setDependsOn(dependsOn);
    }

    private String registerJCacheInitializer(Node source, ParserContext ctx) {
        String cacheName = ((Attr) source).getValue();
        String beanName = cacheName + "-initializer";
        if (!ctx.getRegistry().containsBeanDefinition(beanName)) {
            BeanDefinitionBuilder initializer = BeanDefinitionBuilder.rootBeanDefinition(JCacheInitializer.class);
            initializer.addConstructorArg(cacheName);
            ctx.getRegistry().registerBeanDefinition(beanName, initializer.getBeanDefinition());
        }
        return beanName;
    }
}
```



```
# in 'META-INF/spring.handlers'
http\://www.foo.example/schema/jcache=com.foo.JCacheNamespaceHandler
# in 'META-INF/spring.schemas'
http\://www.foo.example/schema/jcache/jcache.xsd=com/foo/jcache.xsd
```





---

### 10.3. Application Startup Steps



| Name                                           | Description                                                  | Tags                                                         |
| ---------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `spring.beans.instantiate`                     | Instantiation of a bean and its dependencies.                | `beanName` the name of the bean, `beanType` the type required at the injection point. |
| `spring.beans.smart-initialize`                | Initialization of `SmartInitializingSingleton` beans.        | `beanName` the name of the bean.                             |
| `spring.context.annotated-bean-reader.create`  | Creation of the `AnnotatedBeanDefinitionReader`.             |                                                              |
| `spring.context.base-packages.scan`            | Scanning of base packages.                                   | `packages` array of base packages for scanning.              |
| `spring.context.beans.post-process`            | Beans post-processing phase.                                 |                                                              |
| `spring.context.bean-factory.post-process`     | Invocation of the `BeanFactoryPostProcessor` beans.          | `postProcessor` the current post-processor.                  |
| `spring.context.beandef-registry.post-process` | Invocation of the `BeanDefinitionRegistryPostProcessor` beans. | `postProcessor` the current post-processor.                  |
| `spring.context.component-classes.register`    | Registration of component classes through `AnnotationConfigApplicationContext#register`. | `classes` array of given classes for registration.           |
| `spring.context.config-classes.enhance`        | Enhancement of configuration classes with CGLIB proxies.     | `classCount` count of enhanced classes.                      |
| `spring.context.config-classes.parse`          | Configuration classes parsing phase with the `ConfigurationClassPostProcessor`. | `classCount` count of processed classes.                     |
| `spring.context.refresh`                       | Application context refresh phase.                           |                                                              |









Version 5.3.16
 Last updated 2022-02-17 07:33:24 UTC

。。我记得是5.3.15 。。。

暂时告一段落吧，虽然中间跳过了很多。revol4 <

















