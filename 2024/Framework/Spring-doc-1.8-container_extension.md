Spring-doc-1.8-容器扩展点
2022年2月15日
13:44

=================================

=================================

---

#### 1.8. Container Extension Points

一般，开发者 不需要 实现 ApplicationContext 接口。 相反，可以通过插入特殊集成接口的 实现来扩展 spring ioc容器。

---

#### 1.8.1. Customizing Beans by Using a BeanPostProcessor

BeanPostProcessor 接口定义了 回调方法，你可以实现 来提供你的 实例化逻辑,依赖resolution逻辑，等。
如果你想要 实现一些逻辑，在 容器完成 实例化，配置，初始化bean后，你可以plug in 一个或多个自定义的 BeanPostProcessor 实现。

你可以配置多个 BeanPostProcessor 实例，你可以控制顺序通过 order属性。你可以设置这个属性 只有 BeanPostProcessor 实例继承了 Ordered 接口。

如果你要写自己的 BeanPostProcessor，你应该考虑 实现 Ordered接口。

BeanPostProcessor 实例 在 bean(or object) 实例上进行操作。即，容器 实例化 bean实例，然后BeanPostProcessor实例工作。

BeanPostP 实例的scope 每个容器。 这仅在你使用容器hierarchy(层次结构)时才相关。
如果你在 一个容器里定义了 BeanPP,它只会对该容器中的bean进行 post-process.

换句话说，在一个容器中定义的bean不会由另一个容器中定义的 BPP进行 post-process，即使2个容器是同一hierarchy(层次结构)的一部分。

要更改实际bean定义(即，定义bean的蓝图)，你需要使用 BeanFactoryPostProcessor。

BeanPostP 接口包含2个 回调函数。

当这样的类注册为容器的 后处理器时，对于容器创建的每个 bean实例， 后处理器 从容器获得回调 before 容器初始化方法(如InitializingBean.afterProperties, init方法) 被调用， after bean初始化callback。

后处理器 可以对 bean实例采取任何行为，包括 完全忽略回调。
后处理器通常 检查 回调 接口，或 它可以用代理包装一个bean。
一些Spring AOP 基础结构类 被实现为 bean 后处理器，以提供 代理包装 逻辑。

记住，当在 配置类上使用@Bean工厂方法 声明 BeanPP 时，工厂方法返回的类型 应该是 实现类本身 或至少是 BPP接口，明确表明该bean是 后处理器。否则，AppCtx 在完全创建它之前 是无法按照类型 自动感知到它的。由于需要尽早实例化BPP 以应用于 context 中其他bean的初始化，所以这种早期类型检测 至关重要。

编程方式注册BPP实例

实现BPP注册的 建议的方式是 通过 AppCtx 自动侦测， 你可以以编程的方式注册 ConfigurableBeanFactory 通过 addBeanPostProcessor 方法。 当你需要在注册之前评估条件逻辑 甚至在层次结构中跨上下文复制bean 后处理器时，这可能很有用。

但是请注意，以编程方式添加的 BPP实例 不遵守Ordered 接口。在这里，注册的顺序决定了执行的顺序。另外注意，以编程方式注册的BPP实例 始终在通过自动检测注册的实例之前处理，无论如何显式排序。

BeanPostProcessor 实例 和 AOP 自动代理

实现BPP接口的类是特殊的，被容器特殊对待。所有BBP实例和这些BBP实例直接引用的bean在启动时被实例化，作为AppCtx的特殊启动阶段的一部分。 然后，所有的BPP实例都 有序注册，并应用于容器中其他所有bean。因为AOP 自动代理本身就是 BPP，所以BBP实例 和它们直接引用的bean 都没有 自动代理，所以它们没有aspect织入。

对于这样的bean，你应该看到一条日志：Bean xxx is not eligible for getting processed by all BeanPostProcessor interfaces (for example: not eligible for auto-peoxying).

如果你有bean 注入到你的BPP 通过 自动装配 或 @Resource (这个可能回到自动装配) 的方式，spring 在搜索 类型匹配的 依赖候选者时，可能会访问 意外的 bean，因此，会导致它们(。候选者)没有资格进行自动代理 或其他类型的bean 后处理。例如，如果你有一个使用@Resource 的依赖，其中字段或setter名称 不直接对于bean的声明名称 并且没有使用name属性，spring会访问其他bean 以按类型匹配它们。

。。BPP是先于AOP 代理的，所以BPP引用的bean 也会先于AOP 代理。而且看起来 先byName，然后byType。这里的例子就是 无法根据name找到bean，然后 退化到 根据类型，导致 其他的bean 也先于 AOP代理 了。

---
下面展示了 如何写，注册，使用BPP实例，在AppCtx 中。

第一个例子是 基础用法。展示了一个自定义 BPP 实现，在每个被容器创建的bean 上调用 toString 方法，打印到控制台。

```java
package scripting;

import org.springframework.beans.factory.config.BeanPostProcessor;

public class InstantiationTracingBeanPostProcessor implements BeanPostProcessor {

    // simply return the instantiated bean as-is

    public Object postProcessBeforeInitialization(Object bean, String beanName) {

        return bean; // we could potentially return any object reference here...

    }

    public Object postProcessAfterInitialization(Object bean, String beanName) {

        System.out.println("Bean '" + beanName + "' created : " + bean.toString());

        return bean;
    }
}
```

下面的beans 标签 使用 InstantiationTracingBeanPostProcessor

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:lang="http://www.springframework.org/schema/lang"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
         https://www.springframework.org/schema/beans/spring-beans.xsd
         http://www.springframework.org/schema/lang
         https://www.springframework.org/schema/lang/spring-lang.xsd">

    <lang:groovy id="messenger"

            script-source="classpath:org/springframework/scripting/groovy/Messenger.groovy">

        <lang:property name="message" value="Fiona Apple Is Just So Dreamy."/>
    </lang:groovy>

    <!--
    when the above bean (messenger) is instantiated, this custom
    BeanPostProcessor implementation will output the fact to the system console
    -->
    <bean class="scripting.InstantiationTracingBeanPostProcessor"/>

</beans>
```

注意，InstantiationTracingBeanPostProcessor 仅仅是被define。它没有name。由于它是一个bean，它可以被 依赖注入到 你想要注入的bean。

下面的代码是测试 上面的类和配置
```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.scripting.Messenger;

public final class Boot {

    public static void main(final String[] args) throws Exception {

        ApplicationContext ctx = new ClassPathXmlApplicationContext("scripting/beans.xml");

        Messenger messenger = ctx.getBean("messenger", Messenger.class);
        System.out.println(messenger);
    }

}
```
输出：
```

Bean 'messenger' created : org.springframework.scripting.groovy.GroovyMessenger@272961

org.springframework.scripting.groovy.GroovyMessenger@272961
```

---
例子：AutowiredAnnotationBeanPostProcessor
将回调接口 或注解 与自定义BPP实例 结合使用 是扩展IoC容器的 常用方法。

一个例子是 Spring的 AutowiredAnnotationBeanPostProcessor，它是BPP实现，随着spring发行版一起提供，并自动连接带注释的字段，setter 和 任意配置方法。

---
1.8.2 定制配置元数据 通过 BeanFactoryPostProcessor
下一个扩展点是 BeanFactoryPostProcessor。
这个接口的语义 和 BPP的予以类似，但有一个主要区别：BFactoryPP对 bean配置元数据进行操作。
也就是说，spring ioc 容器 允许 BFPP读取 配置元数据，并可能在容器 实例化 其他bean之前 更改它。

你可以配置多个 BFPP 实例，你可以控制 它们的顺序 通过 设置order属性，但是只有当实现了Ordered接口时，这个属性才有用。 如果你写你自己的BFPP，你应该考虑实现 Ordered 接口。

如果你想修改 真实的bean实例(就是通过配置元数据创建的对象)，你需要使用 BPP。

虽然 技术上可以在 BFactoryPP 中使用bean实例(例如，通过BeanFactory.getBean())，但这样做会导致bean过早实例化，从而违反标准容器生命周期。这可能导致负面影响，例如，绕过bean post-process。

同样，BFPP实例 的scope 也是 每个容器。只有在你使用容器层次结构的时候有影响。如果你在一个容器中定义了 BFPP，它只应用到 这个容器中 bean定义。

BFPP 自动运行 当它被声明到 AppCtx 中，以便将更改应用于 定义定义容器的 配置元数据。

spring包含了许多预定义的 BFPP，比如，PropertyOverrideConfigurer 和 PropertySourcesPlaceholderConfigurer。你也可以使用自定义的 BFPP--例如，注册自定义属性编辑器。

AppCtx自动 检测 部署到其中并实现 BFPP 接口的任何 bean。它在适当的时候，使用这些bean作为 bean factory post-processor。 你可以部署这些BFPP bean 就像 其他bean 一样。

和BPP一样，你通常不希望将BFPP 配置为 延迟初始化。如果没有其他bean 引用 B(F)PP，则那个后处理器根本不会被实例化。因此，把它标记为lazy 将被忽略，B(F)PP会被 急加载，即使你在 beans 标签中 声明 default-lazy-init 为 true。

---
例子： 类名替代 PropertySourcesPlaceholderConfigurer

你可以使用 PropertySourcesPlaceholderConfigurer 来 使用标准Java Properties 格式 将bean定义中的 属性值外部化 到单独文件中。

这样做 可以让 部署应用的人员能够自定义 特定于环境的属性，例如数据库url和密码，而无需修改容器的 主要的xml定义文件。

考虑下面的基于xml的 配置元数据片段，其中定义了具有占位符值的 DataSource。
```xml

<bean class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">

    <property name="locations" value="classpath:com/something/jdbc.properties"/>

</bean>

<bean id="dataSource" destroy-method="close"
        class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="${jdbc.driverClassName}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>
```

这个例子展示了 从外部 Properties 文件 读取配置。
运行时，PropertySourcesPlaceholderConfigurer 被应用，来替换 DataSource的某些属性的 元数据。

```properties
jdbc.driverClassName=org.hsqldb.jdbcDriver
jdbc.url=jdbc:hsqldb:hsql://production:9002
jdbc.username=sa
jdbc.password=root
```
这样，${jdbc.username} 这个string 在运行时被替换为 值 'sa'.
PropertySourcesPlaceholderConfigurer 检查 bean 定义的大多数属性和 属性中的占位符。
此外，你还可以自定义占位符前缀，后缀。

通过spring 2.5 的 context 命名空间，你可以配置 属性占位符 通过专用配置标签。你可以提供 一个或多个逗号分隔的list 到 location属性：

```xml

<context:property-placeholder location="classpath:com/something/jdbc.properties"/>

```

PropertySourcesPlaceholderConfigurer 不仅查看 你指明的 properties文件中的属性。
默认下，如果在 你指定的 属性文件中 找不到属性，它会检查 Spring Environment 属性 和常规Java System属性。

你可以使用 PropertySourcesPlaceholderConfigurer 来 代替 class name，这有时是有用的，当你必须 在运行时 获得 特定的类。

```xml

<bean class="org.springframework.beans.factory.config.PropertySourcesPlaceholderConfigurer">

    <property name="locations">
        <value>classpath:com/something/strategy.properties</value>
    </property>
    <property name="properties">
        <value>custom.strategy.class=com.something.DefaultStrategy</value>
    </property>
</bean>

<bean id="serviceStrategy" class="${custom.strategy.class}"/>
```

如果该类在运行时无法解析为有效类，则该类在即将创建时解析失败，这个阶段是 AppCtx 为 非lazy bean的 preInstantiateSingletons() 阶段。

---
例子 PropertyOverrideConfigurer

PropertyOverrideConfigurer 是另一个 BFPP，类似 PropertySourcesPlaceholderConfigurer，但是与后者的区别是， 对于bean属性 原始定义 可以有默认值或根本没有值。如果 overriding Properties文件 没有 特定的bean属性的条目，则使用默认 context 定义。

。。感觉是 上一个是需要占位符${}， 而这里是不需要的，就是普通的设置， 只不过如果外部文件有配置，那么会更新进去。

请注意，bean定义 不知道被覆盖，因此从xml定义文件 中不能立即看出 被使用的覆盖配置器。
如果有多个PropertyOverrideConfigurer实例 为同一个bean 属性定义不同的值，由于覆盖机制，最后一个会获胜。

properties文件的格式：
```
beanName.property=value
```

所以
```properties
dataSource.driverClassName=com.mysql.jdbc.Driver
dataSource.url=jdbc:mysql:mydb
```
用于定义 名为dataSource 的bean 的 driver 和 url属性。

支持复合属性名称，只要路径中的每个组件(除了要覆盖的最终值) 都已经非null。
下面tom这个bean的 fred属性的 bob属性的 sammy属性 设置为 123：
```properties
tom.fred.bob.sammy=123
```

指定的覆盖值始终是 字面量，不会被解释成 bean ref。当xml bean定义 中的 原始值是 bean ref时，这个约定也适用。

通过spring 2.5 中的 context 命名空间，可以使用专用的标签 来配置 property overriding。
```xml
<context:property-override location="classpath:override.properties"/>
```

---
1.8.3 定制实例化逻辑 通过 FactoryBean

你可以为 那些工厂对象 实现 FactoryBean 接口

FactoryBean 接口是 IoC 容器的 实例化逻辑的 一个可插入点。 如果你有复杂的初始化代码，用java能更好的表达，而不是冗长的xml，你可以创建自己的FactoryBean，在该类中编写复杂的初始化，然后将您的自定义FactoryBean插入到容器中。

FactoryBean<T> 接口提供 3个方法：
1. T getObject(): 返回 工厂创建的 实例对象。这个实例可能会被共享，取决于该工厂 是返回 单例还是 原型。

2. boolean isSingleton()： 如果FactoryBean 返回的是 单例，则true，其他则false。这个方法的默认实现是返回true。

3. Class<?> getObjectType()： 返回getObject方法返回的对象的类型，如果事先不知道类型，返回null。

FactoryBean 概念和接口 在spring 中很多地方使用，spring本身就有 50 多个FactoryBean接口的实现。

=================================

=================================

=================================

=================================

=================================

=================================

=================================

=================================

=================================

=================================

=================================

=================================

=================================

=================================

=================================

=================================

=================================

=================================

=================================

=================================

=================================

=================================

=================================

=================================

已使用 Microsoft OneNote 2016 创建。