
spring core 5.3.15

https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#spring-core

### Chapter 1. The IoC Container

Inversion of Control

IoC 也被称为 dependency injection (DI)

It is a process whereby objects define their dependencies (that is, the other objects they work with) only through constructor arguments, arguments to a factory method, or properties that are set on the object instance after it is constructed or returned from a factory method. 
这是一个过程，对象 只 通过 构造器参数，工厂方法的参数，或 在构造或从工厂返回 后的实例 上设置 属性 来 定义依赖。   
。。排除了 直接 new。或者说 硬编码的 依赖关系。

然后容器在创建bean 时 注入这些依赖。

这个过程是 bean自己控制 实例化和依赖 的 反转。


org.springframework.beans 和 org.springframework.context 包是 IoC容器 基础

BeanFactory 提供了 能管理任何类型的 对象的 高级配置机制


ApplicationContext 是 BeanFactory 的子接口，它增加了：
1. 和 Spring AOP 功能更简单的集成
2. 消息资源处理(用于国际化)
3. Event publication(发布)
4. 应用特定context，如WebApplicationContext 用于web 应用

。。 ListableBeanFactory extends BeanFactory {
。。 ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
MessageSource, ApplicationEventPublisher, ResourcePatternResolver {

简单来说，BeanFactory 提供了 配置的框架 和基础功能，ApplicationContext 增加了 企业级的功能。

本章使用 ApplicationContext

---

ApplicationContext 接口 代表 Spring IoC 容器，负责 实例化，配置，组装 bean。

容器 阅读 配置元信息 来知道 如何实例化，配置，组装。

配置元信息可以通过 xml，注解，java 代码 表示。

Spring 也有一些 ApplicationContext 接口的实现。

在stand-alone(独立)应用，通常创建ClassPathXmlApplicationContext 或 FileSystemXmlApplicationContext.

图： Sprint Container 接收： 你的业务对象(POJOs) 和 配置元信息， 生成：配置好的系统。 

---

IoC 容器消费 配置元信息。 配置元信息 代表了 你(开发者) 告诉 容器 怎么实例化，配置，组装。

spring配置 包含了 容器必须管理的 至少一个，通常是多个 bean 定义。

> 基于XML的 配置元信息 配置这些 bean 作为 <bean/> 元素，在 顶层的 <beans/> 中。
> 
> Java配置 通常是 @Bean 注解到 方法上，在 @Configuration 类中。

bean的定义 对应了 构成应用 中的 真实 对象。

可以使用 Spring 和 AspectJ 的集成 来 配置 IoC 容器控制之外 创建的 对象。

xml的配置：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">  
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```


初始化容器

提供给ApplicationContext 构造其的 一个或多个 位置路径 是 资源string，允许容器 加载 配置元数据 从 各种外部资源(如，本地文件，Java的 CLASSPATH等)

> ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

services.xml:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->

    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for services go here -->

</beans>
```

property 标签的 name 元素 表示 Bean的属性的名字， ref表示 另一个bean定义的 名字。

daos.xml:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for data access objects go here -->

</beans>
```


组合 xml配置

多个xml 文件 是有用的，通常，每个文件 代表了 一个逻辑层或模块。

使用 ```<import />``` 标签 来 从其他文件 加载 bean 定义：

```xml
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```

services.xml 必须在 同一个 目录下。  
另外2个 必须在 本xml的所在目录/resources 中， 开头的/ 被忽视。

可以，但不推荐。引用 上一级目录中的xml，使用 ../xxx.xml  
特别的，不推荐 使用在 classpath中 (如 classpath:../services.xml)  
。。classpath 不会忽略 前缀 /

全名： file:C:/config/services.xml  或 classpath:/config/service.xml。  
通常 最好 为这种 绝对位置 保留 间接性，如 通过在运行时 ${...} 解析 占位符

命名空间自己提供了 import 指令。

spring提供了 更多配置特性，比如 context， util 命名空间。

---

#### Groovy 定义 bean
bean定义能被 表达为 Spring的 Groovy Bean Definition DSL。
```
beans {
    dataSource(BasicDataSource) {
        driverClassName = "org.hsqldb.jdbcDriver"
        url = "jdbc:hsqldb:mem:grailsDB"
        username = "sa"
        password = ""
        settings = [mynew:"setting"]
    }
    sessionFactory(SessionFactory) {
        dataSource = dataSource
    }
    myService(MyService) {
        nestedBean = { AnotherBean bean ->
            dataSource = dataSource
        }
    }
}
```
配置风格 和 xml类似，甚至支持xml 配置命名空间。还允许 通过 importBeans 导入 xml配置。

---

ApplicationContext 是高级工厂的接口，能够维护不同bean 及其依赖项 的注册表。

```xml
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

使用groovy配置，bootstrapping 看起来很类似。 它有一个不同的 上下文实现类，它支持groovy (但是也能理解 xml)
> ApplicationContext context = new GenericGroovyApplicationContext("services.groovy", "daos.groovy");

最灵活的形式是 GenericApplicationContext 结合 reader委托，如 结合xml文件的 XmlBeanDefinitionReader：
```
GenericApplicationContext context = new GenericApplicationContext();
new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
context.refresh();
```

groovy：GenericApplicationContext
```
GenericApplicationContext context = new GenericApplicationContext();
new GroovyBeanDefinitionReader(context).loadBeanDefinitions("services.groovy", "daos.groovy");
context.refresh();
```

可以在一个 ApplicationContext 中 混用 reader 委托

使用 getBean 方法来 检索 你的bean的实例。 ApplicationContext接口还有其他方法 用于检索bean，但是，理想情况下，你的应用不应该使用它们。   
实际上，你的代码根本就不应该调用 getBean

---

#### 1.3 Bean Overview

IoC 容器管理 一个或多个 bean。 bean 通过 你提供的配置元数据(如xml的<bean />) 创建。

在容器内，bean定义 被表现为 BeanDefinition 对象，包含下列元数据：
1. 带包名的 类全名：通常，被定义为 bean的实际实现类。
2. bean 行为配置元素，说明了容器中 bean的行为方式( 范围，生命周期回调 等)
3. bean 所依赖的 其他bean的 ref。这些ref 也被成为 协作者 或 依赖项。
4. 新创建的对象 需要的其他配置，如，池的大小限制，管理连接池的bean中使用的连接数。

这些元数据转换为 每个bean定义 中的 一组属性，下面是这些属性：  
Class  
Name  
Scope  
Constructor arguments  
Properties  
Autowiring mode  
Lazy initialization mode  
Initialization method  
Destruction method  

除了包含有关如何创建 bean的 bean定义之外，ApplicationContext 还允许注册 在 容器外部(由用户) 创建的现有对象。   
这是通过 getBeanFactory() 方法 获得 ApplicationContext的 BeanFactory 来实现的，   
这个方法会返回 DefaultListableBeanFactory 对象。这个对象 支持 注册，通过 registerSingleton() 和 registerBeanDefinition()

bean元数据 和 手动的单例实例 需要尽早注册，以便容器在自动装配和其他自省步骤中能正确 推导它们。   
虽然 在某种程度上支持 覆盖现有 元数据和 现有单例实例，但官方不支持 在运行时注册新bean(通过对工厂进行实时访问)，这可能导致 并发访问一场，bean容器中状态不一致。

---
#### 1.3.1 Naming Beans

每个bean 有一个或多个 ID，这些ID 必须在 包含这个bean的 容器中 唯一。   
Bean 通常只有一个 ID，如果需要多个，额外的 被认为是 别名  

基于xml的 配置元信息，使用 ***id*** 属性， ***name*** 属性 来指明 ID。  

id 属性可以让你 准确指定一个 id。 可以是 字母数字，也可以包含 特殊字符。   

如果要引入别名，可以在 name 属性中指定，使用 逗号，分号，空格 分隔。

id唯一性 由 容器执行，不再由 xml parser 确保。

id，name 不是必录的。 都没有的时候，容器生成 唯一的名字 for bean。   
但是，如果你想 引用 bean 通过名字，通过 ref 或 Service Locator， 你必须提供 一个 名字。

Bean Naming Conventions   
对 实例属性名称 使用标准Java 约定，在命名bean时。 即，bean名称以小写字母开头，并且是 驼峰。

在classpath中进行 组件扫描时， spring 为 无名组件生成 名字，规则：使用 类名，并且首字母小写。   
在特殊情况下，当第一第二个都是大写时，会保留 原始大小写。   
这些规则 被定义在 java.beans.Introspector.decapitalize 

---

有时 需要为 在别处定义的bean 引入别名。 使用 xml的 <alias /> ：
```xml
<alias name="fromName" alias="toName"/>
```

这种情况下，fromName名字的bean 在这个alias定义后，也可以被称为 toName

例如，子系统A通过 subsystemA-dataSource 来ref 数据源，  
子系统B通过 subsystemB-dataSource 来ref   
主系统 提供 myApp-dataSource. 配置：
```xml
<alias name="myApp-dataSource" alias="subsystemA-dataSource"/>
<alias name="myApp-dataSource" alias="subsystemB-dataSource"/>
```
这样，每个子系统 和 主系统 都可以通过 唯一的名称 来引用数据源，并且不会和其他定义冲突，而且引用的是同一个bean。

如果你使用 java 配置， @Bean 也能用于提供 别名。

---

实例化bean

bean定义，本质上是 创建一个或多个 对象的方法。   
当被询问时， 容器会查看 这个bean名字 的 配方，用这个bean定义来 创建(或获取) 实际对象。

xml配置，指定 对象的 要被实例化的 类型(或类) 通过 ```<bean />``` 的 class属性。class属性( 对应BeanDefinition 的 Class 属性(。。beanClass))通常是 强制的。   
你可以通过下面2种方式之一使用 Class 属性：
1. 通常，在容器本身通过反射调用 构造器直接创建bean的情况下 指定要构造的bean类，这在某种程度上相当于 new 操作。
2. 用于指定 类，这个类包含了 static 工厂方法 用于创建对象，在不太常见的情况下，容器调用类上的 静态工厂来 创建bean。静态工厂返回的 对象类型 可能是同一个类，也可能是另一个完全不同的类。

嵌入类的名字   
如果你要为 内部类 配置 bean 定义， 你可以使用 内部类的 binary name 或 source name。   
例如，你有一个类 com.example.SomeThing。这个类里有一个 静态内部类：OtherThing，它们通过$或. 来分隔。 此时，class属性的值可以是 com.example.SomeThing$OtherThing 或 com.example.SomeThing.OtherThing。

---

通过 构造器 实例化
通过构造器创建bean时，所有普通类都可以被spring兼容，即，不需要实现额外的接口或 以特定方式进行编码。 只需要指定bean类就足够了。   

但是，根据这个bean 的IoC类型，你可以需要一个 默认(空)构造器。

```xml
<bean id="exampleBean" class="examples.ExampleBean"/>
<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```

---

通过 静态工厂方法 实例化。

当定义一个使用静态工厂方法 创建的bean，使用class 属性来 指定包含 static 工厂方法的类，用factory-method 属性来指定 工厂方法的名字。   
你应该可以直接调用这个方法( with 可选的参数)，返回一个存活的对象，后续 就和 构造器创建的对象一样。   
一种用法 是 使用遗留的代码中的 静态工厂 来 新建 bean定义。   

定义不需要指定 返回的对象的类。

```xml
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
```
。。。。。 这个class 既是 bean类名， 也是 工厂所在类的名字。。。。。

```java
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```

---

通过 实例工厂方法 实例化

类似静态工厂实例化， 使用实例工厂方法的实例化 从容器中的 已存在的 bean的 非静态方法 来创建bean。    

让 class 属性为空。 factory-bean 指定 当前容器中 bean的名字。 factory-method 属性。

```xml
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```

```java
public class DefaultServiceLocator {
    private static ClientService clientService = new ClientServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```

一个工厂类可以有多个工厂方法：
```xml
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>

<bean id="accountService"
    factory-bean="serviceLocator"
    factory-method="createAccountServiceInstance"/>
```
。。。bean的 class 是通过 实例工厂这个bean 的名字 来解析的。。 所以 工厂 必须返回 它自己的实例？？？
```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    private static AccountService accountService = new AccountServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }

    public AccountService createAccountServiceInstance() {
        return accountService;
    }
}
```

---

决定bean的 运行时类型

bean的运行时类型 的决定 不容易。   
bean 元数据 定义 中指定的 class 只是一个 最初的class引用， 可能与 声明的工程方法 或 可能导致bean类型改变的 FactoryBean 类 结合使用，或者在 实例工厂方法 中 根本没有设置 (而是通过 指定的 工厂bean名称解析)   
。。。看起来， 工厂创建的bean定义中 class 是可选的。。   

此外，aop代理 可以用 基于接口的 代理 包装一个bean 实例，并 限制目标bean的 实际类型(仅其实现的接口) 的暴露

查找特定bean 的实际运行时 类型的 推荐方法是 BeanFactory.getType(bean名字)。
。。 ApplicationContext 继承了 ListableBeanFactory


---
#### 1.4 Dependencies

典型的企业级应用不包含单个对象(或spring术语中的 bean)。   
即使最简单的应用 也有一些 对象协同工作。   

---
#### 1.4.1 Dependency Injection

DI 是一个过程，对象仅通过 构造器，工厂方法的参数，对构造后或工厂返回的对象进行属性设置 来定义他们的依赖。   

容器在它创建bean时，注入这些依赖。   

使用DI，代码更干净，当对象具有依赖关系时，解耦更有效。对象不查找其依赖项，也不知道依赖性的位置或类别。

测试更简单，允许mock 实现

2种DI主要变种：基于构造器的DI，基于setter的DI

---
#### Constructor-based Dependency Injection

基于构造器DI 是 通过 容器调用构造器 with 参数s，每个参数代表以来。   
带参数 调用 static 工厂方法 和 构造器 差不多。  

下面的例子 的类 只能通过 构造器注入 依赖：
```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private final MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

---
构造器参数解决

发生在使用参数类型时。   
如果bean定义的构造器参数 不存在潜在歧义时，则构造器中参数的顺序 是在实例化bean时将这些参数提供给 适当构造器的顺序。   
考虑下面的类：   
```java
package x.y;

public class ThingOne {

    public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
        // ...
    }
}
```

如果 ThingTwo 和 ThingThree 没有 继承关系， 不会有 潜在的歧义。下面的配置能工作，你不需要 在<constructor-arg /> 中指定 构造器参数下标或类型：
```xml
<beans>
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg ref="beanTwo"/>
        <constructor-arg ref="beanThree"/>
    </bean>

    <bean id="beanTwo" class="x.y.ThingTwo"/>

    <bean id="beanThree" class="x.y.ThingThree"/>
</beans>
```

当另一个bean被引用，类型会被知道，并且能够进行匹配。   
当一个简单类型被使用，比如```<value>true</value>```，spring不能确定 值的类型，不能进行匹配，考虑下面的类：
```java
package examples;

public class ExampleBean {

    // Number of years to calculate the Ultimate Answer
    private final int years;

    // The Answer to Life, the Universe, and Everything
    private final String ultimateAnswer;

    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

在上面的场景中，容器无法对 简单类型 进行 类型匹配，除非你通过type 指定类型：
```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```

构造器参数是相同类型。可以使用 index 来制定 构造器中的参数 下标：
```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```

指定构造器参数名字。
```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```
记住，要使这项工作开箱即用，你的代码必须在 启用调试标志的情况下 编译，以便spring可以从构造函数中 查找参数名称。   
如果你不想，或不能 使用 debug 标记 来编译。可以使用 @ConstructorProperties 注解，显示命名构造器参数：
```java
package examples;

public class ExampleBean {

    // Fields omitted

    @ConstructorProperties({"years", "ultimateAnswer"})
    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

---

#### 基于setter的 DI

容器调用setter方法，在无参构造器或无参static工厂返回实例 之后。

下面的class 只能通过 setter来DI。是一个POJO，没有 容器级别的 接口，类，注解
```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on the MovieFinder
    private MovieFinder movieFinder;

    // a setter method so that the Spring container can inject a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

ApplicationContext 支持 构造器和 setter DI。  
也支持 在已经通过构造器注入一些依赖后，再通过setter DI

你以BeanDefinition 的形式配置 依赖项，你可以将 其与PropertyEditor 实例相结合，以将属性从一种格式转换为另一种格式。

但是，大多数spring 用户，不会直接使用这些类，而是使用xml bean定义，注解的组件(@Component,@Controller等) 或 @Bean的方法(@Configuration类中)。   
这些源 被转换为 BeanDefinition 实例，并用于 加载整个IoC容器实例。


基于构造器 or 基于setter
由于可以混用 构造器，setter 的DI，所以将 构造器 用于强制依赖， setter用于可选状态 是一个好的经验法则。   
注意，setter方法上的 @Required 注解 声明这个属性 强制。当然，带编程验证的参数的构造器更可取。  
。。xml + @Required？。。。 xml怎么配？

spring团队提倡 构造器注入，因为它允许你将组件 实现为不可变对象，并确保需要的依赖项不为空。   
此外，构造器注入的组建 总是 以完全初始化的状态返回给 客户端的。   
另外，大量的 构造器参数 是一种不好的 代码气味，意味着这个类可能有太多的职责，应该重构以更好地解决适当的关注点分离问题。

setter注入 主要只用于 可以在类中分配合理默认值的 可选依赖项。否则，必须在代码使用依赖项的任何地方执行非空校验。   
setter注入的一个好处是 setter使该类的对象可以在以后 重新分配或重新注入。因此 通过JMX MBean进行管理 是 setter注入的一个用例。

对特定类使用有意义的DI形式。有时，在处理没有源码的第三方类时，选择已经决定了。例如，如果第三方类没有公开setter方法，那么构造器注入 可能是DI的唯一形式。


---
依赖解决过程

容器执行 bean依赖解决 就像下面：
1. ApplicationContext 使用 描述所有bean 的 配置元数据 创建和初始化的。配置元数据可以通过xml，java代码，注解 指定。
2. 对于每个bean，它的依赖关系 以 属性，构造器参数，静态工厂方法参数 来表示。这些依赖项 在创建bean时 提供给bean。
3. 每个属性 和构造器参数，要 字面量作为值 设置，或 容器中另一个bean 的ref
4. 作为字面量值的 属性或构造器参数 都要进行 格式转换，从 指定格式 转换 参数的实际类型(。。应该是指 string转为 type定义的类型)。spring可以将 字符串格式 转换为 所有的内置类型。

容器校验每个bean的配置，在容器创建时。但是bean属性 不会被设置，直到bean被创建。   
容器创建时，会创建 单例范围(singleton-scoped) 并设置为 pre-instantiated(默认) 的bean。其他的bean 在被request时创建(..lazy)。
依赖项之间的解析不匹配可能出现的比较晚，即在第一次创建受影响的bean时。


循环依赖   
类A 通过构造器注入 类B的实例，类B通过构造器注入需要 类A的实例。   
ioc容器 运行时 发现循环引用，抛出异常：BeanCurrentlyInCreationException  

一种可能的方法是，修改一些类的源码，通过setter注入，而不是构造器注入。或者避免构造器注入，并仅使用setter注入。即，虽然不推荐，但是可以通过setter注入来配置 循环依赖。

容器在加载的时候 校验 配置问题，如引用到不存在的类，循环依赖等。   
这意味着 已经 正确加载的容器 可能稍后，在你request 对象的时候 抛出异常，如果在创建对象或它的依赖时发生问题，如 bean在缺少属性或 非法属性时 抛出一场。   
这种配置问题的 潜在延迟，是 ApplicationContext 预实例化单例bean 的原因。

如果不存在循环依赖，则当 协作bean 被注入到 依赖的bean中时， 协作bean 已经完全配置了。协作bean 已经 被实例化，被设置依赖关系，并调用相关的生命周期方法(如配置的init或InitializingBean 回调方法)


下面的例子，使用 基于xml的配置元数据 for setter DI。
```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- setter injection using the nested ref element -->
    <property name="beanOne">
        <ref bean="anotherExampleBean"/>
    </property>

    <!-- setter injection using the neater ref attribute -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

EmampleBean 类定义：
```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public void setBeanOne(AnotherBean beanOne) {
        this.beanOne = beanOne;
    }

    public void setBeanTwo(YetAnotherBean beanTwo) {
        this.beanTwo = beanTwo;
    }

    public void setIntegerProperty(int i) {
        this.i = i;
    }
}
```

下面是 构造器DI：
```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- constructor injection using the nested ref element -->
    <constructor-arg>
        <ref bean="anotherExampleBean"/>
    </constructor-arg>

    <!-- constructor injection using the neater ref attribute -->
    <constructor-arg ref="yetAnotherBean"/>

    <constructor-arg type="int" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

对应的 ExampleBean：
```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public ExampleBean(
            AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {
        this.beanOne = anotherBean;
        this.beanTwo = yetAnotherBean;
        this.i = i;
    }
}
```

使用 static 工厂方法：
```xml
<bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
    <constructor-arg ref="anotherExampleBean"/>
    <constructor-arg ref="yetAnotherBean"/>
    <constructor-arg value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```
```java
public class ExampleBean {

    // a private constructor
    private ExampleBean(...) {
        ...
    }

    // a static factory method; the arguments to this method can be
    // considered the dependencies of the bean that is returned,
    // regardless of how those arguments are actually used.
    public static ExampleBean createInstance (
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {

        ExampleBean eb = new ExampleBean (...);
        // some other operations...
        return eb;
    }
}
```

static 工厂方法的 参数 通过 ```<constructor-arg />``` 提供， 和 构造器一样。   
返回的类 不是必须和 包含static工厂方法的 类一致(这个例子里，是一致的)     
实例工厂 使用相同的方式,除了使用factory-bean，而不是class属性


---
#### 1.4.2 依赖和配置detail

字面值(基本类型，string等)   
```<property />``` 标签的 value属性 将属性或构造器参数 指定为 人类可读的字符串表示。
spring 提供转换服务，用于将这些值 从string 转为实际类型：
```xml
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <!-- results in a setDriverClassName(String) call -->
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
    <property name="username" value="root"/>
    <property name="password" value="misterkaoli"/>
</bean>
```

下面使用 p命名空间：
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
          destroy-method="close"
          p:driverClassName="com.mysql.jdbc.Driver"
          p:url="jdbc:mysql://localhost:3306/mydb"
          p:username="root"
          p:password="misterkaoli"/>

</beans>
```

也可以配置 java.util.Properties 实例：
```xml
<bean id="mappings"
    class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">

    <!-- typed as a java.util.Properties -->
    <property name="properties">
        <value>
            jdbc.driver.className=com.mysql.jdbc.Driver
            jdbc.url=jdbc:mysql://localhost:3306/mydb
        </value>
    </property>
</bean>
```
容器转换 ```<value />``` 中的 text 到 Properties实例，通过JavaBeans PropertyEditor机制。


idref元素   
idref 只是将容器中 另一个bean的id(字符串值，不是引用) 传递给 ```<constructor-arg> 或 <property>``` 的一种防错方法。

```xml
<bean id="theTargetBean" class="..."/>

<bean id="theClientBean" class="...">
    <property name="targetName">
        <idref bean="theTargetBean"/>
    </property>
</bean>
```

上面的bean 定义 和下面的 在运行时 一样：
```xml
<bean id="theTargetBean" class="..." />

<bean id="client" class="...">
    <property name="targetName" value="theTargetBean"/>
</bean>
```

第一种更好，因为使用idref，使得容器在 部署时 验证引用的bean 确实存在。   
第二种，不对传递给客户端bean的 targetName属性的值 执行验证。只有在 实际实例化bean时 才发现拼写错误。

idref的 local属性 在 4.0 bean XSD 中不再受支持，因为它不再提供常规bean 引用的值。   
升级到4.0时，把 idref local 改为 idref bean 。

idref 的带来价值的常见地方(至少在Spring 2.0之前的版本中) 是在 ProxyFactoryBean bean定义中的AOP拦截器配置中。   
使用 idref 防止 拼写错 拦截器的id


---
引用其他bean   

ref是 ```<constructor-arg> <property>``` 中的 final元素。你将 bean的属性 设置为另一个bean的 引用。   

所有引用最终都是 对另一个对象的引用。scope和 validation 取决与 你 使用 bean 还是 parent 属性 制定 其他对象的id还是名字。

通过 ref标签 中的 bean属性 指定目标bean 是最通用的形式，它允许创建 对 同一容器或父容器 中的 任何bean的 引用，而不管它是否在同一个xml文件中。   
bean属性的值 可以与目标bean的 id属性 相同，也可以与目标bean的 name属性 之一相同：
```xml
<ref bean="someBean"/>
```

通过parent 属性 来指定 目标bean， 创建一个引用 指向 当前容器的 父容器 中的bean。    
parent的值 可以是目标bean的 id，或 目标bean的 name属性值之一。   

```xml
<!-- in the parent context -->
<bean id="accountService" class="com.something.SimpleAccountService">
    <!-- insert dependencies as required here -->
</bean>
```

```xml
<!-- in the child (descendant) context -->
<bean id="accountService" <!-- bean name is the same as the parent bean -->
    class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target">
        <ref parent="accountService"/> <!-- notice how we refer to the parent bean -->
    </property>
    <!-- insert other configuration and dependencies as required here -->
</bean>
```

ref的 local 不再支持 从4.0 beans XSD 开始。 使用 ref bean 代替 ref local。


---

Inner Beans
https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-inner-beans

在```<property />,<constructor-arg />``` 中的 ```<bean>``` 就是 inner bean：
```xml
<bean id="outer" class="...">
    <!-- instead of using a reference to a target bean, simply define the target bean inline -->
    <property name="target">
        <bean class="com.example.Person"> <!-- this is the inner bean -->
            <property name="name" value="Fiona Apple"/>
            <property name="age" value="25"/>
        </bean>
    </property>
</bean>
```

内部bean不需要 id 或name。如果定义了，容器也不会使用这个值作为 身份标识。   
容器也忽视 scope ， 因为 内部bean 总是匿名的，且 和 外部bean一起创建。
不能单独访问 内部bean，或者注入到 其他bean中。

极端情况，可以从自定义范围接收 销毁回调，例如，在单例bean中包含 请求范围的 内部bean。   
内部bean实例的创建 与 外部bean 相关联，但是 销毁回调 让它参与请求范围的生命周期。

---
Collections   
```<list><set><map><props>``` 对应了Java的 List Set Map Properties

```xml
<bean id="moreComplexObject" class="example.ComplexObject">
    <!-- results in a setAdminEmails(java.util.Properties) call -->
    <property name="adminEmails">
        <props>
            <prop key="administrator">administrator@example.org</prop>
            <prop key="support">support@example.org</prop>
            <prop key="development">development@example.org</prop>
        </props>
    </property>
    <!-- results in a setSomeList(java.util.List) call -->
    <property name="someList">
        <list>
            <value>a list element followed by a reference</value>
            <ref bean="myDataSource" />
        </list>
    </property>
    <!-- results in a setSomeMap(java.util.Map) call -->
    <property name="someMap">
        <map>
            <entry key="an entry" value="just some string"/>
            <entry key="a ref" value-ref="myDataSource"/>
        </map>
    </property>
    <!-- results in a setSomeSet(java.util.Set) call -->
    <property name="someSet">
        <set>
            <value>just some string</value>
            <ref bean="myDataSource" />
        </set>
    </property>
</bean>
```

map的k，v，set的值，可以是下面的任意元素：   
bean | ref | idref | list | set | map | props | value | null

---
Collection Merging
容器也支持 merging 集合。   
开发者可以 定义 父<list><set><map><props>，有子<list map set props>，并且 覆盖父 的值。   
即，子集合的值 是 父集合+子集合，并且子集合的元素覆盖父集合同名元素。


```xml
<beans>
    <bean id="parent" abstract="true" class="example.ComplexObject">
        <property name="adminEmails">
            <props>
                <prop key="administrator">administrator@example.com</prop>
                <prop key="support">support@example.com</prop>
            </props>
        </property>
    </bean>
    <bean id="child" parent="parent">
        <property name="adminEmails">
            <!-- the merge is specified on the child collection definition -->
            <props merge="true">
                <prop key="sales">sales@example.com</prop>
                <prop key="support">support@example.co.uk</prop>
            </props>
        </property>
    </bean>
<beans>
```
administrator=administrator@example.com
sales=sales@example.com
support=support@example.co.uk


---
Strongly-typed collection

jdk5 开始有了 泛型。

```java
public class SomeClass {

    private Map<String, Float> accounts;

    public void setAccounts(Map<String, Float> accounts) {
        this.accounts = accounts;
    }
}
```
```kotlin
class SomeClass {
    lateinit var accounts: Map<String, Float>
}
```
```xml
<beans>
    <bean id="something" class="x.y.SomeClass">
        <property name="accounts">
            <map>
                <entry key="one" value="9.99"/>
                <entry key="two" value="2.75"/>
                <entry key="six" value="3.99"/>
            </map>
        </property>
    </bean>
</beans>
```

spring的类型转换机制 识别到 Map<String, Float>，所以把 value的string 转为了Float。

---
Null and Empty String Values

spring 把 property 的空参数 视为 ""。下面的xml将设置 email 属性 为 "".
```xml
<bean class="ExampleBean">
    <property name="email" value=""/>
</bean>
```
上面等同于：
```
exampleBean.setEmail("");
```

```<null />``` 用于处理 null：

```xml
<bean class="ExampleBean">
    <property name="email">
        <null/>
    </property>
</bean>
```
等价于：   
```exampleBean.setEmail(null);```


---
XML Shortcut with the p-namespace

p命名空间让你 使用 bean元素的 属性 (代替嵌套的 property 标签)。

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="classic" class="com.example.ExampleBean">
        <property name="email" value="someone@somewhere.com"/>
    </bean>

    <bean name="p-namespace" class="com.example.ExampleBean"
        p:email="someone@somewhere.com"/>
</beans>
```

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="john-classic" class="com.example.Person">
        <property name="name" value="John Doe"/>
        <property name="spouse" ref="jane"/>
    </bean>

    <bean name="john-modern"
          class="com.example.Person"
          p:name="John Doe"
          p:spouse-ref="jane"/>

    <bean name="jane" class="com.example.Person">
        <property name="name" value="Jane Doe"/>
    </bean>
</beans>
```
p:spouse-ref="jane"

p命名空间不如xml灵活，例如 声明属性引用的-ref格式 和 ref结尾的属性冲突。

---
XML Shortcut with the c-namespace

类似p空间，c空间始于3.1,允许 用于配置 构造器参数的 内联属性。

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="beanTwo" class="x.y.ThingTwo"/>
    <bean id="beanThree" class="x.y.ThingThree"/>

    <!-- traditional declaration with optional argument names -->
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg name="thingTwo" ref="beanTwo"/>
        <constructor-arg name="thingThree" ref="beanThree"/>
        <constructor-arg name="email" value="something@somewhere.com"/>
    </bean>

    <!-- c-namespace declaration with argument names -->
    <bean id="beanOne" class="x.y.ThingOne" c:thingTwo-ref="beanTwo"
        c:thingThree-ref="beanThree" c:email="something@somewhere.com"/>

</beans>
```

```xml
<!-- c-namespace index declaration -->
<bean id="beanOne" class="x.y.ThingOne" c:_0-ref="beanTwo" c:_1-ref="beanThree"
    c:_2="something@somewhere.com"/>
```

由于xml语法，属性不能以数字开头，下标符号以_开头。

---
Compound Property Names

```xml
<bean id="something" class="things.ThingOne">
    <property name="fred.bob.sammy" value="123" />
</bean>
```
property 可以 使用 复合或嵌套的属性名称， 只要路径的所有组件除了最后一个外 不可为空。

something有一个 fred 属性，这个属性有 bob属性，bob有sammy属性。 sammy被设置为123。。 fred 和 bob 必须非null，否则NullPointerException

---
Using depends-on

如果 bean 是 另外一个bean的依赖，通常意味着 这个bean 是另外一个bean 的属性。   
传统的，你可以用 property标签的 ref属性。   
但有时 依赖关系不那么直接。 比如，在需要触发类中的 静态初始化程序时，比如 用于 数据库驱动程序注册。   
depends-on 属性 显式强制 一个或多个bean 必须在 本bean 之前初始化。

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager"/>
<bean id="manager" class="ManagerBean" />
```
```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
    <property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```

depends-on 可以指定 初始化顺序，也可以指定 销毁顺序。 定义depends-on 的bean 先销毁。

---
Lazy-initialized Beans

默认下，ApplicationContext 的实现 急加载和配置 所有 单例bean 作为初始化的一部分。   
通常，这是可取的，因为这样 使得错误在 启动时抛出，而不是 几小时/几天 后抛出。   

你可以使用 lazy-init 来 告知容器，在第一次被request时 创建bean，而不是在启动时。

```xml
<bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.something.AnotherBean"/>
```
如果lazy的bean 是 非lazy的单例bean的 依赖，那么 lazy bean 也会在 启动时创建。

全部默认 lazy：
```xml
<beans default-lazy-init="true">
    <!-- no beans will be pre-instantiated... -->
</beans>
```

---
#### 1.4.5. Autowiring Collaborators

容器可以自动装配 协作bean之间的关系。 自动装配的优点：
1. 显著减少指定属性或构造器参数 的需要
2. 随着对象发展，自动装配可以更新配置。例如，如果你要增加一个依赖，自动装配可以自动满足这个依赖项，不需要修改配置。

使用 bean 标签的 autowire 属性来 指明 自动装配模式。有4个模式
1. no: 默认， 没有自动装配。bean的引用必须ref。 对于大项目，不建议更改默认配置，因为明确指定协作者 可以提供更大的控制力 和 清晰度。在某种程度上，它记录了系统的结构。
2. byName： 自动装配 通过 属性名字，spring查找 和属性名相同的 bean 来自动装配。 例如，如果bean定义 设置了 通过名字 自动装配，且 它有一个 master 属性(也就是说，它有一个setMaster(,,)方法)，spring查找 一个名字是 master 的 bean定义，并且设置到 这个属性
3. byType： 让属性被自动装配，如果 容器中 有且只有 一个 该属性类型的bean。如果多于1个，抛出异常，意味着你可能不能使用byType。如果没有匹配，不会发生任何事 (属性不会被设置)
4. constructor： 类似byType，但是引用到 构造器参数。 如果不是 有且只有一个，抛出异常

使用 byType 或 constructor 模式，你可以 wire 数组和 tpyed集合。这种情况下，将提供容器中 所有与预期类型匹配的所有 自动装配候选者 来满足依赖关系。   
你可以自动装配 强类型的Map ，如果期望的key类型是 string。   

---
Limitations and Disadvantages of Autowiring 限制和缺点

如果项目中都始终使用自动装配，那么它的效果最好。   
如果一般不使用自动装配，开发者可能会混淆 使用它来 只装配一个或2个bean定义。

限制和缺点：
1. 属性和 构造器参数 中的 显式依赖 始终覆盖 自动装配。 你不能装配简单属性，如基本类型，string，Class类。 这个限制是设计使然
2. 自动装配不如 显式装配精确。spring管理的对象之间的关系 不再 明确记录。
3. 容器生成文档的工具 可能无法获得 装配信息。
4. 容器内的 多个bean定义 可能满足 要自动装配的 setter 或 构造器。对于 数组，集合，Map，这不是一个问题。但是，对于期望单个值的依赖项，这种歧义不会被解决。如果没有唯一的bean定义可用，会引发异常。

在后续场景中，你有多种选择：
1. 放弃自动装配，使用显示装配
2. 通过 设置bean定义的 autowire-condidate为false 来避免自动装配(。。autowiring 不是 autowired)
3. 将bean的 primary属性设置为 true 来指定为 主要候选者
4. 使用注解 实现更细粒度的控制。


---
Excluding a Bean from Autowiring

排除bean from 自动装配。  
设置 bean标签的 autowire-candidate 属性 为false， 容器 使得这个bean 定义 不能用于 自动装配(包括 @Autowired)。

autowire-candidate 只 影响 基于类型的 自动装配。 不会影响 按名称的显示引用，即使指定的bean 没有被标记为 自动装配候选者，也会被解析。

你可以对 bean名字 进行模式匹配 来 限制 自动装配候选者。   
顶层beans 标签 的 default-autowire-candidates 属性 接收 一个或多个pattern。 例如，为了限制 名字以Repository结尾的bean 的 自动装配候选者，提供一个 *Repository 值作为pattern。   
多个pattern，逗号分隔。   
bean定义中的 autowire-candidate 优先。   


---
Method Injection

在大多数的应用场景中，容器中的bean 大多是单例的。

当一个 单例bean 要与另一个单例bean 协作，或 非单例bean 要与另一个非单例bean 协作，你通常通过 将一个bean 定义为另一个bean的属性 来处理依赖关系。

当bean 生命周期不同时，就会出现问题。   
假设 单例bean A 在方法中 调用了 非单例bean B。 容器只创建一次单例A，因此只有一次机会设置属性。容器无法在每次需要时，为A提供一个新的B。

一种方法是放弃IoC。 你可以使得 A 知道 容器，通过 实现 ApplicationContextAware 接口，然后在需要的时候 getBean("B")。

```java
// a class that uses a stateful Command-style class to perform some processing
package fiona.apple;

// Spring-API imports
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class CommandManager implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    public Object process(Map commandState) {
        // grab a new instance of the appropriate Command
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    protected Command createCommand() {
        // notice the Spring API dependency!
        return this.applicationContext.getBean("command", Command.class);
    }

    public void setApplicationContext(
            ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```

上面并不可取，因为业务代码 知道并耦合到 Spring框架。   
方法注入是 IoC的一项高级功能，可以让你干净地处理此用例。

---
Lookup Method Injection

查找方法注入 是容器的能力，用于 重写容器管理的bean的方法，并返回 容器中另一个bean的查找结果。   
lookup通常设计 一个 prototype bean，就像上节描述的场景。   
spring通过 cglib 库中的 字节码生成器 来动态生成 一个 重写了方法的 子类 来实现方法注入。

为了让 动态子类 起效，容器中 被子类化的类 不能是final类。被重写的方法不能是 final方法。   
对具有抽象方法的类进行 单元测试 需要你自己 子类化 该类 并提供抽象方法的 存根实现。   
组件扫描也需要具体的方法，这需要具体类来拾取。   
一个关键限制是，查找方法不适用于工厂方法，特别不适用于 配置类中的 @Bean方法，因为在这种情况下，容器不负责创建实例，因此无法创建运行时生成的即时子类。

对于上节的 CommandManager 类，容器动态重写 createCommand 方法的实现。CommandManager没有任何spring依赖项。

```java
package fiona.apple;

// no more Spring imports!

public abstract class CommandManager {

    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```

在包含要注入的方法 的类中( 本例为CommandManager)，需要注入的方法 以如下的形式：
```java
<public|protected> [abstract] <return-type> theMethodName(no-arguments);
```

如果方法是 abstract，动态生成的子类 实现这个方法。否则 动态生成的子类将覆盖原始类中定义的具体方法。
```xml
<!-- a stateful bean deployed as a prototype (non-singleton) -->
<bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype">
    <!-- inject dependencies here as required -->
</bean>

<!-- commandProcessor uses statefulCommandHelper -->
<bean id="commandManager" class="fiona.apple.CommandManager">
    <lookup-method name="createCommand" bean="myCommand"/>
</bean>
```

commandManager 这个bean 在需要 myCommand bean的新实例时调用 它自己的createCommand() 方法。   
你必须小心地 把myCommand 设置为 prototype。如果是单例，那么每次都返回相同的myCommand bean实例。


或者，使用基于注解的组件模型，你可以声明一个 查找方法，通过 @Lookup。
```java
public abstract class CommandManager {

    public Object process(Object commandState) {
        Command command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup("myCommand")
    protected abstract Command createCommand();
}
```

或，更加惯用的，你可以依赖 lookup方法 声明的返回类型解析目标bean
```java
public abstract class CommandManager {

    public Object process(Object commandState) {
        Command command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup
    protected abstract Command createCommand();
}
```

注意，你通常应该使用具体的存根实现 来声明这个被注解的lookup方法，以使它们与默认情况下 忽略抽象类的 spring组建扫描规则兼容。   
这个限制不适用于显示注册或显式导入的bean类。


访问不同scope的bean的另一种方式是 ObjectFactory/Provider 注入点。 你也会发现ServiceLocatorFactoryBean 是有用的。


---
Arbitrary Method Replacement 随意方法替换

和lookup相比，一种不太有用的 method injection 是 替换托管的bean的任意方法。

replaced-method

```java
public class MyValueCalculator {

    public String computeValue(String input) {
        // some real code...
    }

    // some other methods...
}
```

```java
public class ReplacementComputeValue implements MethodReplacer {

    public Object reimplement(Object o, Method m, Object[] args) throws Throwable {
        // get the input value, work with it, and return a computed result
        String input = (String) args[0];
        ...
        return ...;
    }
}
```

```xml
<bean id="myValueCalculator" class="x.y.z.MyValueCalculator">
    <!-- arbitrary method replacement -->
    <replaced-method name="computeValue" replacer="replacementComputeValue">
        <arg-type>String</arg-type>
    </replaced-method>
</bean>

<bean id="replacementComputeValue" class="a.b.c.ReplacementComputeValue"/>
```

为了方便 arg-type 中可以是全民的 substr，下面都是 java.lang.String
java.lang.String
String
Str


---
### 1.5 Bean Scopes

不仅可以控制 依赖和配置值，还可以控制 scope。   

Spring支持 6个scope，4个只有 当你使用 web-aware ApplicationContext 时 可用。

你也可以创建自定义scope。。

singleton: 默认，每个IoC容器中 一个bean定义 只有一个 实例。

prototype： 一个bean定义 有任意多的 实例

以下4个仅在 web-aware ApplicationContext 中有效

request： bean定义的生命周期是 一个HTTP request。即，每个HTTP request，对于一个bean定义 有它自己的实例。

session： bean定义的 生命周期是 HTTP session。

application： bean定义的生命周期是 ServletContext

websocket： 生命周期是 WebSocket

从3.0开始， thread scope 可用，但默认情况下未注册。


---
The Singleton Scope

单例bean 只有 一个被共享的实例 被管理，容器收到的 所有 和这个bean 匹配(id或 name)的请求，都会返回这个实例。

spring的 单例bean 概念 和 GoF的设计模式中的单例模式不同。   
GoF 单例是 硬编码 对象的scope，以便每个 ClassLoader 只能创建特定类的 一个 实例。   
Spring 单例的scope 最好描述为 每个容器 和 每个bean

```xml
<bean id="accountService" class="com.something.DefaultAccountService"/>

<!-- the following is equivalent, though redundant (singleton scope is the default) -->
<bean id="accountService" class="com.something.DefaultAccountService" scope="singleton"/>
```

---
The Prototype Scope   

bean的 prototype scope 导致 每次对 bean的请求 都会 创建一个新实例。

>> 通常，你应该对所有 有状态的bean 使用 prototype， 对 无状态的bean 使用 singleton


DAO 通常不被定义为 prototype，因为 传统的DAO 不会保持 任何 conversational(会话式的) 状态。

```xml
<bean id="accountService" class="com.something.DefaultAccountService" scope="prototype"/>
```

和其他范围相比，spring不管理 prototype bean的 完整生命周期。   
容器 实例化，配置，组装 prototype 对象 并传递给 客户端，并不会进一步记录该原型实例。   
因此，尽管 所有对象都会被调用 初始化生命周期 回调方法，但是原型，不会调用配置的 销毁生命周期回调。客户端必须清理 原型对象 并释放原型对象的 资源

要让容器释放 prototype bean 持有的资源，请尝试使用 post-processor，这个包含 对需要清理的bean的 引用。

在某些方面，容器对于 prototype bean 的 作用 是替代 java的new。


---
单例 依赖了 原型

当你使用 单例依赖 原型，请注意 依赖关系 是在实例化时解决。   
因此，如果你将 原型 注入到 单例中， 会实例化一个 原型，然后注入到 单例中， 单例中的 原型 是唯一的。

但是，假设你希望 单例bean 在运行时 重复获取 原型bean的新实例。 你不能将 原型注入到 单例中，因为这个注入 仅 在 容器实例化单例时 发生一次。   
如果需要多次原型bean的 新实例，参考 Method Injection


---
#### 1.5.4. Request, Session, Application, and WebSocket Scopes

只在 web-aware ApplicationContext 中有用。(如 XmlWebApplicationContext)。  

如果你在 常规的IoC 容器中使用 这些scope，会抛出 IllegalStateException。

---
初始化Web配置

为了支持这4个scope，在定义bean之前 需要一些小的初始化配置。   

如何配置取决于你的 Servlet 环境。  

如果你要在Spring MVC中 访问这种bean，实际上，在DispatcherServlet处理的请求中，不需要特殊设置。DispatcherServlet 已经公开了 所有相关状态。

如果你使用Servlet 2.5 web 容器，不使用DispatcherServlet 处理请求 (如使用JSF  或 Structs 时)，你需要注册spring的 RequestContextListener ServletRequestListener。   
对于Servlet 3.0+，这个可以通过使用 WebApplicationInitializer 接口来 编程解决。   
对于更早的容器，增加下面到 web.xml
```xml
<web-app>
    ...
    <listener>
        <listener-class>
            org.springframework.web.context.request.RequestContextListener
        </listener-class>
    </listener>
    ...
</web-app>
```

或者，如果你的listener 设置存在问题，考虑RequestContextListener。   
这个过滤器映射 取居于周围的web应用配置，所以你可以 根据需要修改。   

```xml
<web-app>
    ...
    <filter>
        <filter-name>requestContextFilter</filter-name>
        <filter-class>org.springframework.web.filter.RequestContextFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>requestContextFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    ...
</web-app>
```

DispatcherServlet, RequestContextLisener,RequestContextFilter 都做了 同一件事情，将HTTP request 对象 绑定到 为该请求提供服务的 线程。   
这使得 bean的 request，session scope 在 调用链的下游 可用。


---
Request scope

```xml
<bean id="loginAction" class="com.something.LoginAction" scope="request"/>
```

容器使用 loginAction 的bean定义 为每个 http request 新建一个 LoginAction 实例。这样，loginAction bean 的 scope 就是 HTTP request 层级。

注解组件或Java配置， @RequestScope注解能被用来 指定组件 是 request scope。

```java
@RequestScope
@Component
public class LoginAction {
    // ...
}
```

---
Session Scope

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>
```

容器 为每个 http session 创建一个 UserPreference bean.

@SessionScope  
```java
@SessionScope
@Component
public class UserPreferences {
    // ...
}
```

---
Application Scope

```xml
<bean id="appPreferences" class="com.something.AppPreferences" scope="application"/>
```

为整个 web application 创建一个 实例。即appPreference 是ServletContext 层级的，保存为 ServletContext 的常规属性。   
类似单例，但有2个重要的区别： 每个ServletContext 一个单例，而不是每个Applicationontext； 它通过 ServletContext 的属性来暴露。

@ApplicationScope
```java
@ApplicationScope
@Component
public class AppPreferences {
    // ...
}
```

---
WebSocket Scope

这个scope 和 WebSocket session的 生命周期相关联，适用于 基于WebSocket 的 STOMP 应用

---
Scoped Beans as Dependencies

IcO 容器不仅管理 bean的 实例，也 装配协作者(依赖)。   
如果你想 注入一个 http request scope 的 bean 到 另一个 长期存活的bean， 你可以选择 注入一个 AOP代理 而不是 scoped bean。   
即，你需要注入一个 扩展了 和 scoped bean相同 公共接口，但也可以从相关scope(如request scope) 检索真实目标对象，并将方法调用委托给真实对象 的代理对象。   


你也可以使用```<aop:scoped-proxy>``` 在单例bean 之间，引用(名词)通过可序列化的中间代理，因此能够在反序列化时重新获得单例bean。   

当 对prototype bean 声明 aop:scoped-proxy 时，在共享代理上的 每次方法调用 都会导致 创建一个新的目标实例，然后将调用转发到该实例。

此外，scoped proxy 不是 唯一方式： 以生命周期安全的方式 从较短的作用域 访问bean。   
你也可以 声明自己的注入点(构造器或setter或自动装配字段) 为 ObjectFactory<MyTargetBean>，这个接口的 getObject方法 调用 在每次需要时 按需检索 当前实例，不需要hold 实例或 单独存储实例。   

作为扩展变体，你可以声明ObjectProvider<MyTargetBean>，提供了几个额外的访问，包括 getIfAvailable,getIfUnique.

JSR-330 变体 称为 Provider，与 Provider<MyTargetBean> 声明 和 每次检索尝试的 相应的 get() 一起使用。   


下面的配置只有一行，但是是重要的 用于 理解 why 和 how

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- an HTTP Session-scoped bean exposed as a proxy -->
    <bean id="userPreferences" class="com.something.UserPreferences" scope="session">
        <!-- instructs the container to proxy the surrounding bean -->
      。。。。。。。。。。。。。。。。。  <aop:scoped-proxy/> 
    </bean>

    <!-- a singleton-scoped bean injected with a proxy to the above bean -->
    <bean id="userService" class="com.something.SimpleUserService">
        <!-- a reference to the proxied userPreferences bean -->
        <property name="userPreferences" ref="userPreferences"/>
    </bean>
</beans>
```

为了创建这样的代理，将 aop:scoped-proxy 插入到 scoped bean 定义。   
为什么在 request，session，自定义scope 中需要 aop:scoped-proxy ？   
考虑下面的 单例 bean，并和 你需要为上述之前的scope 的定义的内容进行对比。   
```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>

<bean id="userManager" class="com.something.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

在上面的例子中，单例bean userManager 被注入了 一个 指向 session scope 的 bean userPreferences 的引用。   
这里的重点是 userManager 是 单例的： 每个容器只实例化一次，并且它的依赖项 也只注入了一次。 这意味着 userManager bean 只对完全相同的 userPerferences(即最初注入的对象) 进行操作

这不是你想要的长生命bean 注入了一个 短生命bean 的 行为，你想要的是 userManager 单例，在 http session 生命周期中，你需要 这个session的 userPreferences.   

容器创建了一个 和UserPerferences 相同公共接口的 的对象，这个对象可以检索真正的 UserPerferences 对象 从 scope 机制。   
对于 userManager bean, 它不知道 UserPreferences 引用是一个 代理。   
这个例子中，当 UserManager实例 调用 依赖注入的 UserPreferences 对象的方法时，它是调用了 代理的方法。代理会检索 真正的 userPreferences 对象 从 http Session，并且委托方法调用到 检索到的 真正的 UserPreferences 对象。


---
选择创建的代理的类型

默认，容器创建 一个 代理 for 标记为 aop:scoped-proxy 的bean， 基于cglib的 代理 被创建。

cglib 代理 只拦截 public 的方法的调用。 不要调用 这种proxy的 非public方法，它们不会被委托到真正的 scoped 对象。

或者，你可以配置 容器，使得它创建 标准jdk 基于接口的代理 for scoped bean，通过 设置 aop:scoped-proxy 标签的 proxy-target-class 属性为 false。   
使用 jdk的基于接口的代理 意味着，你不需要额外 library。但是也意味着 scoped bean 的类 必须实现 至少一个接口，并且注入 这个scoped bean的 bean都是通过 scoped bean 的某个接口 来引用的。

下面是 基于接口的 代理：
```xml
<!-- DefaultUserPreferences implements the UserPreferences interface -->
<bean id="userPreferences" class="com.stuff.DefaultUserPreferences" scope="session">
    <aop:scoped-proxy proxy-target-class="false"/>
</bean>

<bean id="userManager" class="com.stuff.UserManager">
<property name="userPreferences" ref="userPreferences"/>
</bean>
```


---
### 1.5.5. Custom Scopes

你可以定义你自己的 scope 或 重定义已有的scope，但是重定义已有scope 是非常糟糕的，并且 你不能重写 内置的 singleton 和 prototype。

---
创建自定义scope

实现spring的 Scope 接口，可以查看 已有的 spring 的 Scope 接口的实现 和 它的Doc。   

Scope接口有4个方法，来从scope 中获得 对象，从scope 移除对象，使得它们被销毁。   

session scope实现，例如，返回 session-scoped bean (如果不存在，方法会返回一个 新的 实例，after 绑定到session for 后续 ref )。   
```
Object get(String name, ObjectFactory<?> objectFactory)
```

session scope实现，移除 session-scoped bean 从 底层的 session中。移除的对象应该被return，但你可以return null 如果指定的名字不存在。   
```
Object remove(String name)
```

下面的方法注册一个 callback，scope 应该调用这个callback 当它被 销毁 或 当 在scope中的 指定的对象 被销毁时。
```
void registerDestructionCallback(String name, Runnable destructionCallback)
```

下面的方法 获得 scope 的 标识符。
```
String getConversationId()
```
每个scope的 标识符是不同的， 对于 session scoped 实现，标识符可以是 session的标识符。


---
使用自定义scope

在你写且测试 自定义Scope实现后，你需要让 spring 容器 感知到你的 scope。下面的方法是注册 新的scope到 spring 容器：
```java
void registerScope(String scopeName, Scope scope);
```

这个方法在 ConfigurableBeanFactory 接口中，可以通过 ApplicationContext实现的 BeanFactory属性 来获得。   
方法的第一个参数是 scope 的 唯一名字。比如spring的 singleton和 prototype。   
第二个参数是 你想注册和使用的 自定义Scope 实现的 实例。  

下面的例子是 SimpleThreadScope, 这是spring的类，但是没有注册。
```
Scope threadScope = new SimpleThreadScope();
beanFactory.registerScope("thread", threadScope);
```
```
<bean id="..." class="..." scope="thread">
```

使用自定义 Scope实现，你不限于 编程方式注册scope。你也可以 声明式进行Scope 注册，通过使用 CustomScopeConfigurer：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean class="org.springframework.beans.factory.config.CustomScopeConfigurer">
        <property name="scopes">
            <map>
                <entry key="thread">
                    <bean class="org.springframework.context.support.SimpleThreadScope"/>
                </entry>
            </map>
        </property>
    </bean>

    <bean id="thing2" class="x.y.Thing2" scope="thread">
        <property name="name" value="Rick"/>
        <aop:scoped-proxy/>
    </bean>

    <bean id="thing1" class="x.y.Thing1">
        <property name="thing2" ref="thing2"/>
    </bean>
</beans>
```

当你在 实现FactoryBean接口的 bean 声明中 放置 aop:scoped-proxy， scope 的是 factory bean，而不是 从 getObject 返回的 对象。

---
### 1.6. Customizing the Nature of a Bean

spring 提供了一些接口，你可以用来 定制bean的 性质。   
Lifecycle Callbacks   
ApplicationContextAware and BeanNameAware   
其他 Aware 接口。

---
Lifecycle Callbacks

为了和 容器中bean的 生命周期的 管理进行交互，你可以实现 InitializingBean，DisposableBean 接口。   
容器为InitializingBean 调用 afterPropertiesSet，为DisposableBean 调用 destroy， 以让bean在 初始化和 销毁bean时 执行某些操作。

JSR-250 @PostConstruct 和 @PreDestory 注解 通常被认为是 现代spring 应用中欧给你 接收 生命周期回调的 最佳实践。 使用这些注解意味着你不会耦合到spring 的接口。

如果你不想使用JSR-250 注释，仍想移除耦合，考虑 bean标签的 init-method, destroy-method。 

在内部，spring使用 BeanPostProcessor 实现 来处理它可以找到的 任何回调接口并调用适当的方法。   

如果你需要自定义特性或其他生命周期行为，你可以实现 BeanPostProcessor.

对于 初始化和销毁 毁掉，spring管理的对象 也可以实现 Lifecycle 接口，这样这些对象能 参与由容器自身生命周期 驱动的 启动和关闭 过程。


---
Initialization Callbacks

spring的 InitializingBean 接口使得 bean 执行 初始化工作 在容器 为bean设置所有必要的 属性后。   
接口只有一个方法：```void afterPropertiesSet() throws Exception;```

我们建议你 不 要 使用这个接口，它会使得 代码耦合到spring。   
我们建议 @PostConstruct，或指定 POJO的 初始化方法。 在xml中，可以使用 init-method 来指明一个无参方法。 Java配置，你可以使用 @Bean 的 initMethod 方法。   
```
<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
```

或者
```
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>

public class AnotherExampleBean implements InitializingBean {

    @Override
    public void afterPropertiesSet() {
        // do some initialization work
    }
}
```

---
Destruction Callbacks

实现 DisposableBean 接口，使得 bean 进行回调 当 包含它的 容器 被销毁时。
```
void destroy() throws Exception;
```
。。。？ 是容器被销毁，而不是这个bean? 不过bean 似乎就是和 容器一起的，除了。。不好像只有 singleton 是和容器一起的，其他的都是不是。。

建议你 不要使用这个，因为耦合。 而是使用 @PreDestroy 或指定 一个普通方法。 xml的使用 bean标签的 destroy-method, Java配置，使用@Bean 的 destroyMethod。

```
<bean id="exampleInitBean" class="examples.ExampleBean" destroy-method="cleanup"/>
```



```java
public class AnotherExampleBean implements DisposableBean {

    @Override
    public void destroy() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

你可以为 bean 标签的 destroy-method 属性分配一个 特殊的值：inferred， 该值 指示spring 自动检测特定bean 上的 公共的 close，shutdown 方法(实现了AutoCloseable, Closeable 接口)。   
inferred 也可以设置到 beans标签的 default-destroy-method 上。

java配置，是默认有这个行为的。


---
默认初始化和销毁方法

当你编写 初始化和销毁 的毁掉方法时，如果不依靠InitializingBean，DisposableBean 接口， 你通常会命名为 init,initialize,dispose 等。   
理想情况下，此类生命周期回调方法的名称在整个项目中是标准化的，以便所有开发人员使用相同的方法名称确保一致性。   

你可以配置 spring 容器 来 "查看" 每个bean 的 有名字的初始化和 销毁方法。   
这意味着 作为开发者，你可以写你的类，并使用init() 作为 初始化回调方法，不需要 配置init-method="init" 属性。   

IoC 容器调用这个方法，当bean被创建时。

支持这种自动回调的方法名是 init， destroy。

```java
public class DefaultBlogService implements BlogService {

    private BlogDao blogDao;

    public void setBlogDao(BlogDao blogDao) {
        this.blogDao = blogDao;
    }

    // this is (unsurprisingly) the initialization callback method
    public void init() {
        if (this.blogDao == null) {
            throw new IllegalStateException("The [blogDao] property must be set.");
        }
    }
}
```

```xml
<beans default-init-method="init">

    <bean id="blogService" class="com.something.DefaultBlogService">
        <property name="blogDao" ref="blogDao" />
    </bean>

</beans>
```

beans 标签的 default-init-method 属性 导致 IoC 容器 识别出 bean 类中的 init方法 来作为 初始化方法回调。   
当创建和组装bean时， 如果bean 类有这样的方法，会在合适的时候 调用。

beans 也可以设置 default-destroy-method.

bean 里的 init-method, destroy-method 会覆盖 beans 的default。

容器保证在 为bean 提供所有依赖后 立刻调用 初始化回调。   
且，在原始bean 引用上 调用初始化回调，这意味着 AOP 拦截器尚未应用于bean。   
先创造一个 完整的bean，然后 应用 aop代理及其拦截器链。 如果bean和代理 分开定义的，你的代码甚至可以绕过代理 与 原始bean 交互。   
因此，将拦截器应用于 init 方法将是不一致的，这样做会将bean 的 生命周期与其代理或拦截器 耦合，并在你的代码直接与原始bean交互时留下奇怪的语义。


---
Combining Lifecycle Mechanisms

spring 2.5 开始，有3个选项来控制 bean 生命周期行为：  
InitializingBean 和 DisposableBean 回调接口。  
自定义 init(), destroy() 方法   
@PostConstruct,@PreDestroy。   

你可以混合这些机制来控制bean。

如果bean有多个生命周期被配置，且每个机制被配置在不同的方法名，那么每个被配置的方法都会执行，按照下面的顺序。但是，如果配置了相同的方法名，这个方法 执行一次。   

一个bean 多个生命周期，且配置了不同了 初始化方法，按照下面的顺序：   
@PostConstruct    
InitializationBean 的 afterPropertiesSet()   
自定义的 init() 方法  

销毁的顺序：   
@PreDestroy   
DisposableBean 的 destroy()   
自定义的 destroy 方法


---
Startup and Shutdown Callbacks

Lifecycle 接口 为任何有自己的生命周期要求(例如 启动和停止某些后台进程)的对象定义了基本方法：
```java
public interface Lifecycle {

    void start();

    void stop();

    boolean isRunning();
}
```

spring 管理的 对象可以实现 Lifecycle 接口。   
当ApplicationContext 收到 开始 和 结束 标记 (例如，一个 运行时的 停止/重启 场景)，它将这些调用 级联到通过该context 中定义的所有 Lifecycle 实现。   
通过委托给 LifecycleProcessor 来完成，就像下面：
```java
public interface LifecycleProcessor extends Lifecycle {

    void onRefresh();

    void onClose();
}
```

LifecycleProcessor 继承了 Lifecycle。 增加了2个其他的方法 来对 refresh 和 close 的context 来做出反应。   

常规Lifecycle 是 显式启动和停止通知的普通合同，并不意味着在context 刷新时自动启动。   
要对特定bean的 自动启动(包括启动阶段) 进行细粒度控制，请考虑 使用 SmartLifecycle.

stop通知不能保证在 销毁之前发出。在常规的关闭时，所有的 Lifecycle bean 首先收到 停止通知 在 传播 销毁毁掉之前。 但是，在context的生命周期内 进行 hot 刷新 或 停止刷新尝试时， 只会调用销毁方法。


启动和关闭 调用的 顺序是重要的，如果2个对象间存在依赖关系，被依赖的 在 依赖者前面启动，在后面关闭。   
但是，有时，直接依赖 是未知的。你可能只知道某种类型的对象应该先于另一种类型的对象开始。在这些情况下，SmartLifecycle 接口定义了另一个选项，即 在其 父接口Phased 上定义一个 getPhase() 方法。

启动时，最低的 phase 先开始。停止时，反过来。   
因此， 实现 SmartLifecycle 接口的 对象，且 getPhase 返回 Integer.MIN_VALUE 的 会是 第一个启动，最后一个 结束。   
在考虑 phase 的值时，知道 任何 未实现 SmartLifecycle 的 对象的生命周期的 默认phase 是 0 也很重要。   
因此，任何 负数phase 表示 对象应该在 标准组件之前启动，之后停止。

SmartLifecycle 的 停止方法 接受一个 回调。任何实现 必须调用 回调的 run() 方法 在 该实现的 关闭过程 完成后。 这会在必要时 启用异步关闭，因为 LifecycleProcessor 的默认实现：DefaultLifecycleProcessor 会等待每个phase内的对象组调用该回调 直到超时。默认每个phase 超时是 30s。   
你可以在context 中定义一个 名字为 lifecycleProcessor 的bean，来覆盖默认的 lifecycle processor instance。

```xml
<bean id="lifecycleProcessor" class="org.springframework.context.support.DefaultLifecycleProcessor">
    <!-- timeout value in milliseconds -->
    <property name="timeoutPerShutdownPhase" value="10000"/>
</bean>
```

LifecycleProcessor 接口 还定义了 用于 refresh 和 close context的 回调方法。后者 驱动 关闭过程，就好像 stop()已经被显式调用，但它发生在 context 关闭时。   
refresh 回调，启用了 SmartLifecycle bean 的另一个功能。刷新context时(在所有对象已经实例化和初始化后)，回调被调用。此时，默认生命周期 处理器 检查 每个 SmartLifecycle 对象的 isAutoStartup() 返回的bool。 如果true，则该对象启动，而不是等待 context 或 它自己的start() 方法 的显式调用(不同于 context refresh，对于标准context实现，上下文启动不会自动发生)。   
阶段值 和 依赖关闭 决定了启动顺序。


---
Shutting Down the Spring IoC Container Gracefully(优雅地) in Non-Web Applications

本节只适用于 非web应用。   
spring的基于 web的 ApplicationContext 实现 已经有代码可以在 相关web 应用程序 关闭时优雅地关闭spring IoC 容器。

如果你在 非web应用中 使用spring的 IoC 容器(如，在 富客户端桌面应用)，注册一个 shutdown hook 到 jvm。   
这样做可以保证正常关闭并在 单例bean 上调用 相关的 销毁方法，以释放所有资源。 你需要正确配置和实现这些 销毁回调。

为了注册 shutdown hook，调用 ConfigurableApplicationContext 接口的 registerShutdownHook() 方法。

```java

public final class Boot {
    public static void main(final String[] args) throws Exception {
        ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");
        // add a shutdown hook for the above context...
        ctx.registerShutdownHook();
        
        // app runs here...
        
        // main method exits, hook is called prior to the app shutting down...
    }
}
```
。。里面： Runtime.getRuntime().addShutdownHook(this.shutdownHook); 。 给一个Thread到 Runtime


---
#### 1.6.2. ApplicationContextAware and BeanNameAware

当 ApplicationContext 创建 实现了ApplicationContextAware 接口的 bean 的实例时，这个实例 会被提供一个 ApplicationContext的 ref。   

这个bean 能编程式操作 创造它的 ApplicationContext，通过 ApplicationContext接口 或 强转为一个子类。   
一种用途是 编程时 检索其他bean。 有时是有用的，但应该避免实现这个接口，因为会耦合，并且没有依照 IoC style(协作者应该作为属性提供给bean)。   
ApplicationContext的其他方法提供 对文件资源的 访问，发布应用event，访问MessageSource。  

autowiring 是获得 ApplicationContext ref 的另一种方法。   
传统的 构造器 和 byType 自动装配 可以为 构造器参数，setter参数 提供 ApplicationContext 类型的依赖。   
为了更加灵活，包括自动装配字段和多参数方法的能力，请使用 基于注解的 装配功能。   
如果你这样做，ApplicationContext 会被自动装配到 暴露了ApplicationContext类型的 字段，构造器参数，方法参数，如果这些带有 @Autowired 注解。


当ApplicationContext 实例化 实现了 BeanNameAware 接口的 bean 定义时，这个类被提供了一个 spring中 这个bean的 名称。。

```java
public interface BeanNameAware {
    void setBeanName(String name) throws BeansException;
}
```
这个回调 在普通 bean 属性设置完后， 初始化回调(如InitializingBean.afterPropertiesSet() 或 自定义init-method) 之前。

---
其他 Aware 接口
ApplicationContextAware: 注入了声明的 ApplicationContext   
ApplicationEventPublisherAware： 注入 enclosing ApplicationContext的 事件发布者   
BeanClassLoaderAware： 加载bean的类 的 类加载器   
BeanFactoryAware： 声明的 BeanFactory   
BeanNameAware： 声明的bean的名字   
LoadTimeWeaverAware： 定义的编织器，用于在加载时处理类定义。   
MessageSourceAware： 用于解析消息的配置策略(支持参数化和国际化)  
NotificationPublisherAware： spring JMX 通知 publisher  
ResourceLoaderAware： 为 对资源进行低级访问 而配置的加载程序。   
ServletConfigAware： 容器运行的当前 ServletConfig，仅在 bean-aware AppCtx 有效  
ServletContextAware: 容器运行的当前 ServletContext，只在 bean-aware AppCtx 有效。  

使用这些aware，违反了 IoC。 建议用于 需要以编程方式访问容器的 基础设置bean。   

---
#### 1.7. Bean Definition Inheritance

bean定义 可以包含许多配置信息，包括构造器参数，属性值，容器特定信息(如，初始化方法，静态工厂方法名等)。   
子 bean定义 继承 配置信息 从 父定义。 子定义能重写 或增加一些 值。   

如果你 以编程的方式 使用 AppCtx接口，则 子bean定义 由 ChildBeanDefinition 类表示。   
绝大部分用户 用不到这一层。   
相反，它们在 ClassPathXmlApplicationContext 等类中 配置bean定义。   
当你使用xml时，可以通过 parent 来 表明 子定义，parent的值是 parent bean。  

```xml
<bean id="inheritedTestBean" abstract="true"
        class="org.springframework.beans.TestBean">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithDifferentClass"
        class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBean" init-method="initialize">  
    <property name="name" value="override"/>
    <!-- the age property value of 1 will be inherited from parent -->
</bean>
```

如果子bean定义没有指明 bean class，则使用 父定义中的。 如果指明了bean class，则 子bean 必须和 父bean 兼容(就是需要有同名属性值)。
。。。可以不同类啊。

子定义从 父定义 继承 scope，构造器参数值，属性值，方法覆盖，并可以添加新值。 你指定的 scope，init方法，destroy方法，static工厂方法 会覆盖 父定义。   
其他设置，总是取自 子定义：依赖，自动装配模式，依赖检查，单例，lazy init。

上面的例子中，显式标记 父bean 是 abstract。   
如果父定义 没有指定 class，那么 必须 显式设置为 abstract。

```xml
<bean id="inheritedTestBeanWithoutClass" abstract="true">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithClass" class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBeanWithoutClass" init-method="initialize">
    <property name="name" value="override"/>
    <!-- age will inherit the value of 1 from the parent bean definition-->
</bean>
```

这个父bean 不能被实例化，因为它是不完整的，且它被显式标记为 abstract。   
当一个 定义是 abstract，它只能用作单纯的模板bean定义，作为子定义的父定义。  
abstract bean定义，不能被 ref， 不能被getBean。   
且容器内部 preInstantiateSingletons() 方法 忽略 抽象bean定义。  

AppCtx 默认 预实例化所有单例，因此，如果你有一个父bean定义，且只用来作为 模板，并且为该类指定了一个class，那么 必须保证 abstract 是true，否则 app ctx将 尝试 预初始化 这个父bean。


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


---
---
---

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
。。g了啊。。。那岂不是5.3 @Required 无用了？  这个 被 deprecated 了。。。   
。。。cao，5.3.5 也说了。。只要多看几行。。。   
。。5.3 建议使用 @Autowired(required=false) 

context:annotation-config，仅在 定义它的 应用上下文中 查找bean的 注解。   
这意味着，如果你将 context:annotation-config 放到 DispatcherServlet 的 WebApplicationContext 中，它只检查 controller 的  @Autowired 的bean。   

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
1.9.2  @Autowired   

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
这些接口和它们的子接口 (如，ConfigurableApplicationContext, ResourcePatternResolver) 被自动解析，无需特殊设置。 下面是 自动装配 ApplicationContext的例子：
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
。。好像也没有问题，  没有name的情况下，就 先by-type，然后用 默认的name 进行筛选。。  
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

spring提供了一个 默认的宽松嵌入式值解析器。它将尝试解析property值，如果无法解析，property name(如 ${catalog.name}) 将作为值注入。   
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



spring boot 配置了默认的 PropertySourcesPlaceholderConfigurer bean，这个bean会从 application.properties, application.yml 文件中 读取 property。



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
    ScopedProxyMode proxyMode() default ScopedProxyMode.TARGET_CLASS;

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
public class AppConfig  {
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



>  使用 context:component-scan 会隐式启用 context:annotation-config 的功能。



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

| Filter Type         | Example Expression         | 描述                                        |
| ------------------- | -------------------------- | ------------------------------------------- |
| annotation(default) | org.example.SomeAnnotation | 在目标组件的 类型级别 存在或 元-存在的 注解 |
| assignable          | org.example.SomeClass      | 目标组件 实现/继承 的 接口/类               |
| aspectj             | org.example..*Service+     | 一个AspectJ 类型表达式，被目标组件匹配      |
| regex               | org\\.example\\.Default.*  | 一个RE，被目标组件的类名匹配                |
| custom              | org.example.MyTypeFilter   | spring的 TypeFilter接口的 自己的实现。      |



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



如前所述，自动装配的 属性和 方法 也被支持  with 额外支持@Bean 方法的自动装配。

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

这个例子 自动装配 string 方法参数 country 为 一个名为iprivateInstance 的 bean 的 age属性的值。#{}是SpEL; @Value注解，表达式解析器 被预先配置 在为解析表达式文本时 查询bean的名字。

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



常规spring component 中 @Bean 方法的处理方式 和 Spring @Configuration类中 对应方法不同。不同之处在于 @Component 没有通过 cglib 增强来拦截方法和字段的 调用。cglib 代理是调用@Configuration 类中@Bean方法中的方法或字段创建协作对象的bean 元数据引用的方法。此类方法不是使用正常的Java语义调用的，而是通过容器来提供spring bean的常规生命周期管理和代理，即使当 ref到其他bean 通过编程方式调用 @Bean方法。相反，在普通的@Component类中 调用@Bean方法中的方法或字段 具有标准的java语义，没有特殊的cglib处理或其他约束。

。。好像是 cglib 不会增强 @Component， 会增强@Configuration



你可以声明@Bean 方法为 static，从而 允许 它们被call 而不需要 创建 它们包含的配置class(their containing configuration class) 为实例。这在定义 post-processor bean (如，BFPP 或 BeanPostProcessor) 是特别有意义，因为此类bean在容器生命周期的早期被初始化，并且应该避免在那个时候触发配置的其他部分。

由于技术限制，对static @Bean 方法的调用 永远不会被 容器拦截，即使在@Configuration 类中 也不会被拦截：cglib 子类化 只能覆盖 非static方法。因此，直接调用另一个 @Bean 方法 具有标准的java 语义，从而导致直接从 工厂方法 本身返回一个独立实例。

。。没有办法拦截，就不能 cache， 所以 就看 这个工厂 是每次 return new，还是 单例设计模式。

@Bean方法的Java语言可见性 不会立刻影响spring容器中生成的bean定义。你可以在非@Configuration 类中自由声明 你认为合适的工厂方法，也可以在任何地方声明静态方法。但是@Configuration 类中的 常规@Bean 方法 需要是可覆盖的，也就是说，它们不能被声明为private 或final。

。。。？后半段应该是说 cglib 通过 继承来增强，所以需要 能被覆盖。 前半段呢？ 非@Configuration 的 工厂有什么用。。@Bean 必须在 @Configuration中吧？难道可以不是？不是，必须是@Component中。看网上的例子，@Configuration 的 被cglib 了， @Component 的没有被cglib。。

在给定组件或配置类的基类，或给定组件或配置类 实现的 接口的 default方法 的 @Bean 会被发现。这为复杂的配置安排 提供了很大的灵活性，甚至从4.2起 可以通过 jdk8的 default方法 实现多重继承。

最后，单个类 可以为 同一个 bean保存多个@Bean方法，作为多个工厂方法的排列，根据运行时可用的依赖关系使用。这与在其他配置场景中选择"最贪婪"的构造器或工厂方法 的算法相同：在构造时 选择具有最多可满足依赖项的 变体，类似于容器如何在多个 @Autowired构造器之间进行选择。



---

1.10.6 Naming Autodetected Components

当一个组件 在 扫描过程中 被自动侦测到，它的bean name 通过 scanner知道的BeanNameGenerator 策略生成。默认下，任何spring 原型注解(Component,Repository,Service,Controller)  包含的 name属性的值 被用作 bean的名字。

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

要为范围解析提供自定义策略而不是依赖基于注解的方法，你可以实现ScopeMetadataResolver 接口。确保包含一个默认的 无参构造器。然后你可以在配置 scanner时提供完全限定的类名：

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



当使用某些 非单例 scope时，可能需要为 scoped 对象 生成代理。为此，在component-scan中有 scoped-proxy 属性，3个可能值是 no，interfaces,targetClass。例如，下面会生成标准jdk动态代理：

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



---

#### 1.11 Using JSR 330 标准注解

从3.0 开始，spring提供了对 JSR-330 标准注解 的支持。这些注解被扫描 如同Spring的注解一样。 要使用这些注解，你需要导入相关的jar。

---
1.11.1 Dependency Injection with @Inject and @Named

你可以代替@Autowired，使用 @javax.inject.Inject:
```java
import javax.inject.Inject;

public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    public void listMovies() {
        this.movieFinder.findMovies(...);
        // ...
    }
}
```

和@Autowired 一样，你可以使用@Inject在 属性，方法，构造器参数级别。   
更进一步，你可以声明你的 注入点 as Provider，允许通过 Provider.get() 调用 按需访问 更短scope的bean 或 lazy访问其他bean。  

。。 @Autowired 可以吗？

```java
import javax.inject.Inject;
import javax.inject.Provider;

public class SimpleMovieLister {

    private Provider<MovieFinder> movieFinder;

    @Inject
    public void setMovieFinder(Provider<MovieFinder> movieFinder) {
        this.movieFinder = movieFinder;
    }

    public void listMovies() {
        this.movieFinder.get().findMovies(...);
        // ...
    }
}
```

如果你想使用 限定名 for 应该注入的依赖，你应该使用 @Named   
```java
import javax.inject.Inject;
import javax.inject.Named;

public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(@Named("main") MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

和@Autowired 一样，@Inject 也可以使用 Optional，@Nullable。这在这里更适用，因为 @Inject 没有 required 属性。
```java
public class SimpleMovieLister {

    @Inject
    public void setMovieFinder(Optional<MovieFinder> movieFinder) {
        // ...
    }
}
```
```java
public class SimpleMovieLister {

    @Inject
    public void setMovieFinder(@Nullable MovieFinder movieFinder) {
        // ...
    }
}
```

---
1.11.2 @Named 和 @ManagedBean: @Component的标准等价物

取代@Component,你可以使用 @Named 或 @ManagedBean。
```java
import javax.inject.Inject;
import javax.inject.Named;

@Named("movieListener")  // @ManagedBean("movieListener") could be used as well
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

很常见：使用@Component 而不指定组件名称。@Named可以用于类似的方式。
```java
import javax.inject.Inject;
import javax.inject.Named;

@Named
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

当你使用 @Named 或 @ManagedBean 时，你可以使用 组件扫描 就像你使用 spring注解一样。

```java
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    // ...
}
```

和@Component相比，JSR-330的 @Named，JSR-250 的@ManagedBean 并不是可组合的。  
你应该使用spring的 原型模型来构建自定义组件注解。

---
1.11.3 JSR 330 标准注解的限制

你需要知道一些重大功能不可用。

|spring|javax.injext.*|javax限制/注释|
|---|---|---|
|@Autowired|@Inject|Inject没有'required'属性，能通过Optional 来代替|
|@Component|@Named/@ManagedBean|JSR 330 不支持 可组合模型，只有一种方法来识别命名的组件|
|@Scope("singleton")|@Singleton|JSR330默认scope是类似spring的prototype，但是，为了和spring的默认值一致，在spring容器中声明的JSR 330 bean默认为单例。要使用非单例的scope，你应该使用spring的@Scope注解。javax还提供了一个@Scope，然而，这个仅用于创建你自己的注解|
|@Qualifier|@Qualifier/@Named|javax的@Qualifier 只是用于构建自定义限定符的元注解。具体的字符串限定符(如，带有值的spring的@Qualifier) 可以通过 @Named 关联|
|@Value|-|无等价的|
|@Required|-|无等价的|
|@Lazy|-|无等价的|
|ObjectFactory|Provider|Provider是spring的ObjectFactory的直接替代品。可以和@Autowired和 无注解的构造器 和 setter 结合使用|


---
#### 1.12 基于Java的 容器配置

本节覆盖 如何使用注解在你的java代码中 来配置spring 容器。包含以下主题：   
基础概念：@Bean 和 @Configuration   
实例化spring 容器 by AnnotationConfigurationApplicationContext   
使用@Bean 注解  
使用@Configuration注解  
组合 基于java的配置  
bean 定义 profile   
PropertySource 抽象   
使用@PropertySource  
statement中的 占位符 解析   

---

1.12.1 基础概念：@Bean，@Configuration  
spring的java-配置 支持的 中央功能是 @Configuration注解的类 和 @Bean注解的方法  
@Bean用来指明 一个方法 实例化，配置，初始化 一个由spring ioc 容器管理的新对象。  
@Bean 和 xml的bean 标签 作用相同。  
你可以使用 @Bean注解的方法 在任何spring的 @Component中，当然，最常用于 @Configuration 类。  

用@Configuration注解一个类 表明 它的主要作用是作为 bean 定义的 源。   
更进一步，@Configuration 类使得 bean间的依赖 被定义 by 调用同一个类中的 @Bean方法。最简单的@Configuration类：

```java
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```

上面的AppConfig 类等同于下面的 beans xml：

```xml
<beans>
    <bean id="myService" class="com.acme.services.MyServiceImpl"/>
</beans>
```



Full @Configuration vs "lite" @Bean mode？

当@Bean方法 被声明在 没有@Configuration 的class 中，它们被 以"lite精简"模式 处理。  
在@Component 或 普通类中 声明的 Bean method 被认为是 "lite"，包含类的 不同的 主要目的，@Bean是其中的一种意外收获/奖励(bonus)。例如，service 组件可能通过在每个适用的组件类上的 附加@Bean 方法 向容器公开管理视图。在这种情况下，@Bean方法是一个 通用的工厂方法机制。  

不同于full @Configuration，lite @Bean method 不可以声明 bean间的依赖，相反，它们对其包含的组件的内部状态进行操作，并且 optional 对它们声明的参数 进行操作。  
因此这样的@Bean方法 不应该调用 其他 @Bean 方法。   
每个这样的方法只是特定 bean ref的工厂方法，没有任何特殊运行时语义。这里正向副作用是运行时不必cglib子类化，因此在类的设计方面没有限制( 类可以是final的)   

在常见情况下，@Bean方法 在@Configuration 类中声明，确保使用 full 模式，并且跨方法引用被重定向到容器的生命周期管理。这可以防止通过常规java调用意外调用相同的@Bean方法，这有助于减少在lite模式下操作时 难以追踪的细微错误。   
。。感觉是 防止 自我调用?   

后续还有@Bean，@Configuration 深入讨论。我们先看通过 基于java的配置 创建 spring容器的 多种方法



---

初始化spring 容器 by AnnotationConfigApplicationContext  

AnnotationConfigApplicationContext 从3.0 引入。是一个多功能的ApplicationContext实现。不仅能处理@Configuration类，也能处理@Component类，和JSR330的类。  

当提供@Configuration类作为输入时，@Configuration类自身 被注册为一个 bean定义，该类中所有声明的@Bean 方法也被 注册为 bean定义。  

当@Component，JSR-330 作为输入，它们被注册为 bean定义，并且假定在必要时 在这些类中使用了诸如@Autowired,@Inject 之类的 DI 元数据。



Simple Construction

和将xml作为输入 来实例化 ClassPathXmlApplicationContext 相似，你可以在实例化 AnnotationConfigApplicationContext时 使用@Configuration类作为输入。这允许spring容器 以无xml的方式 使用：

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

就像前面提到的，AnnotationConfigApplicationContext 不限于 @Configuration。任何@Component，JSR330 都可以作为 输入导入到 容器：

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(MyServiceImpl.class, Dependency1.class, Dependency2.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

上面的例子假设 MyServiceImpl,Dependency1,Dependency2 使用spring依赖注入注解，如@Autowired。



---

建造容器 以编程的方式 by register(Class..)   

你可以实例化 AnnotationConfigApplicationContext by使用 无参构造器，然后配置通过 register方法。

这个方法特别有用，当编程式构建一个AnnotationConfigApplicationContext:

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.register(AppConfig.class, OtherConfig.class);
    ctx.register(AdditionalConfig.class);
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```



启用组件扫描 scan(string...)

为了启用组件扫描，可以注解你的@Configuration：

```java
@Configuration
@ComponentScan(basePackages = "com.acme") 
public class AppConfig  {
    // ...
}
```

xml中等价

```xml
<beans>
    <context:component-scan base-package="com.acme"/>
</beans>
```

上面的例子中，com.acme 包被扫描，寻找 @Component 注解的类，这些类被注册为 容器中的 bean定义。  
AnnotationConfigApplicationContext 暴露了 scan(string...) 方法来 允许 相同的组件扫描功能。

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.scan("com.acme");
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
}
```

记住，@Configuration  类是被@Component 元注解注解的注解，所以它们是 component-scanning的候选者。在上面的例子中，假设AppConfig被声明在 com.acme包(或它的子包)中，它被获取 在调用scan()的时候。在refresh() 的时候，它所有的@Bean方法都被处理并被注册为容器中的 bean定义。



通过AnnotationConfigWebApplicationContext 支持 web app

WebApplicationContext 关于 AnnotationConfigApplicationContext 的变体是 AnnotationConfigWebApplicationContext。你可以使用这个实现，当配置spring ContextLoaderListener servlet listener，spring mvc的 DispatcherServlet 等 时。下面的 web.xml 配置了一个 传统的 spring mvc web app (关注 contextClass 这个context-param 的使用 和 初始init-param)：

```xml
<web-app>
    <!-- Configure ContextLoaderListener to use AnnotationConfigWebApplicationContext
        instead of the default XmlWebApplicationContext -->
    <context-param>
        <param-name>contextClass</param-name>
        <param-value>
            org.springframework.web.context.support.AnnotationConfigWebApplicationContext
        </param-value>
    </context-param>

    <!-- Configuration locations must consist of one or more comma- or space-delimited
        fully-qualified @Configuration classes. Fully-qualified packages may also be
        specified for component-scanning -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>com.acme.AppConfig</param-value>
    </context-param>

    <!-- Bootstrap the root application context as usual using ContextLoaderListener -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- Declare a Spring MVC DispatcherServlet as usual -->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- Configure DispatcherServlet to use AnnotationConfigWebApplicationContext
            instead of the default XmlWebApplicationContext -->
        <init-param>
            <param-name>contextClass</param-name>
            <param-value>
                org.springframework.web.context.support.AnnotationConfigWebApplicationContext
            </param-value>
        </init-param>
        <!-- Again, config locations must consist of one or more comma- or space-delimited
            and fully-qualified @Configuration classes -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>com.acme.web.MvcConfig</param-value>
        </init-param>
    </servlet>

    <!-- map all requests for /app/* to the dispatcher servlet -->
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>
</web-app>
```



---

1.12.3 使用 @Bean 注解

@Bean 是 方法级别的注解，是xml 的 bean的直接模拟。注解支持 bean标签里的 部分属性，比如，init-method,destory-method,autowiring,name

你可以在@Configuration 或 @Component 的类中 使用 @Bean

---

声明一个bean

为了声明bean，你可以使用 @Bean 来注解方法。你使用这个方法来注册一个 类型是方法返回值的类型 的bean定义 到ApplicationContext中。默认下，bean名称和方法名称相同。下面是一个@Bean 方法定义：

```java
@Configuration
public class AppConfig {

    @Bean
    public TransferServiceImpl transferService() {
        return new TransferServiceImpl();
    }
}
```

上面配置等价于下面的xml：

```xml
<beans>
    <bean id="transferService" class="com.acme.TransferServiceImpl"/>
</beans>
```

上面的2种定义都 制造了一个 名字是 transferService的bean 到ApplicationContext，绑定了TransferServiceImpl 类型的 实例。

你也可以使用 default 方法来定义bean。这允许通过在default方法上实现带有bean定义的多个接口来组合bean配置。

```java
public interface BaseConfig {

    @Bean
    default TransferServiceImpl transferService() {
        return new TransferServiceImpl();
    }
}

@Configuration
public class AppConfig implements BaseConfig {

}
```

你也可以声明的你@Bean方法，返回类型是 一个接口(或基类)，就像下面：

```java
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }
}
```

但是，这将 高级类型预测的可见性限制为 指定的接口类型。然后，只有实例化受影响的单例bean后，容器才知道完整类型。非lazy 单例bean 根据它们声明顺序进行实例化，因此你可能会看到不同的类型匹配结果，具体取决于另一个组件 何时尝试 by 未声明的类型 进行匹配。( 比如@Autowired TransferServiceImpl，只有在transferService bean已经被实例化 时才 进行注入)。

如果你始终通过声明的 服务接口 来引用你的类型，则你的@Bean 返回类型 可以安全地加入该设计决策(。。应该指返回接口作为bean 类型)。但是，对于实现多个接口的组件 或可能直接引用到实现类的 组件，声明最具体的返回类型可能更安全。(至少 与 引用bean时 用到的类型 一样具体)。

---

Bean Dependencies

> 一个@Bean 注解的方法 可以有任意数量的参数 来描述构建该bean 所需的依赖项。

```java
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}
```

解析机制 和基于构造器的注入 几乎相同。

---

接受生命周期回调

使用@Bean注解 定义的任何类都支持常规的生命周期回调，能使用JSR-250 的 @PostConstruct, @PreDestory。  
。。结合下面的以及@Bean无法用于类，感觉这里说的是，@Bean 工厂方法返回的 类 支持回调，并且返回的 这个类可以通过 @PostConstruct/PreDestory 来 标注回调方法。   

常规spring 生命周期也支持。如果bean实现了 InitializingBean，DisposableBean，Lifecycle，它们的方法会被 容器调用。

*Aware 接口的 标准集合(如 BeanFactoryAware，BeanNameAware,MessageSourceAware,ApplicationContextAware 等) 也都支持。

@Bean 注解支持 指明 任意的 初始化和 析构 回调方法，就像xml的 bean标签的 init-method, destory-method。

```java
public class BeanOne {

    public void init() {
        // initialization logic
    }
}

public class BeanTwo {

    public void cleanup() {
        // destruction logic
    }
}

@Configuration
public class AppConfig {

    @Bean(initMethod = "init")
    public BeanOne beanOne() {
        return new BeanOne();
    }

    @Bean(destroyMethod = "cleanup")
    public BeanTwo beanTwo() {
        return new BeanTwo();
    }
}
```

默认下，使用Java 配置的 具有 public的 close 或 shutdown 方法的 bean 会自动进行 回调。   
如果你有一个 公共的 close /shutdown，且 你不希望被 回调，那么@Bean(destoryMethod="") 到你的bean定义。  

默认情况下，你可能希望对使用 JNDI 获取的资源执行此操作，因为它的生命周期在app之外进行管理。特别是，确保始终为 DataSource 执行此操作，因为已知它在java EE 应用服务器上会出现问题。  
下面是 阻止 对DataSource  的自动回调：

```java
@Bean(destroyMethod="")
public DataSource dataSource() throws NamingException {
    return (DataSource) jndiTemplate.lookup("MyDS");
}
```

此外，使用@Bean method，你通常使用 编程 JNDI 查找，通过使用spring 的 JndiTemplate 或 JndiLocatorDelegate 帮助程序 或直接 使用 JNDI 的 InitialContext，而不是 JndiObjectFactoryBean变体(这个会迫使你将返回类型声明为 FactoryBean ，而不是实际目标类型，使其更难用于其他@Bean 方法中的 交叉调用。)。

下面是 之前的 @Bean(initMethod = "init") public BeanOne beanOne() { xxx }的等价：

```java
@Configuration
public class AppConfig {

    @Bean
    public BeanOne beanOne() {
        BeanOne beanOne = new BeanOne();
        beanOne.init();
        return beanOne;
    }

    // ...
}
```



---

指定bean scope

spring有 @Scope 注解，所以你可以制定 bean的 scope

你可以指明 你的用@Bean注解的 bean 的 scope。

默认scope 是单例。

```java
@Configuration
public class MyConfiguration {

    @Bean
    @Scope("prototype")
    public Encryptor encryptor() {
        // ...
    }
}
```



---

##### @Scope 和 scoped-proxy

spring提供了一种简单的方法(scoped proxies(。。在长生命周期的bean中 注入 短生命周期的bean，一般是注入 proxy)) 来使用 scoped 依赖。  
最简单的方式是 创建一个代理 当使用xml配置时用aop:scoped-proxy 标签。  
配置java中的bean 用@Scope注解的 proxyMode属性，提供了 等价支持。默认没有scope 代理，除非 component-scan 这层进行了配置。你可以指定 ScopedProxyMode.TARGET_CLASS, INTERFACES, NO。

如果你使用java 将(。。1.5中，反正很早的 ) xml 中的 配置进行 转换，它类似下面：

```java
// an HTTP Session-scoped bean exposed as a proxy
@Bean
@SessionScope
public UserPreferences userPreferences() {
    return new UserPreferences();
}

@Bean
public Service userService() {
    UserService service = new SimpleUserService();
    // a reference to the proxied userPreferences bean
    service.setUserPreferences(userPreferences());
    return service;
}
```

。。。下面是 xml 的配置

```xml
    <bean id="userPreferences" class="com.something.UserPreferences" scope="session">
        <!-- instructs the container to proxy the surrounding bean -->
        <aop:scoped-proxy/> 
    </bean>

    <!-- a singleton-scoped bean injected with a proxy to the above bean -->
    <bean id="userService" class="com.something.SimpleUserService">
        <!-- a reference to the proxied userPreferences bean -->
        <property name="userPreferences" ref="userPreferences"/>
    </bean>
```



---

自定义 Bean 命名

默认下，配置文件使用@Bean的 方法的名字 就是bean的名字。

```java
@Configuration
public class AppConfig {

    @Bean("myThing")
    public Thing thing() {
        return new Thing();
    }
}
```



---

Bean Aliasing

就像最开始讨论的，有时，需要给一个 bean 多个name，被称为bean 别名。@Bean注解的 name属性 接受 一个string[]。

```java
@Configuration
public class AppConfig {

    @Bean({"dataSource", "subsystemA-dataSource", "subsystemB-dataSource"})
    public DataSource dataSource() {
        // instantiate, configure and return DataSource bean...
    }
}
```



---

Bean描述

有时，给bean一个文字描述是有用的。当bean被暴露(可能通过JMX) 用于监视目的时，这可能特别有用。

为了增加@Bean的描述，你可以使用@Description注解：

```java
@Configuration
public class AppConfig {

    @Bean
    @Description("Provides a basic example of a bean")
    public Thing thing() {
        return new Thing();
    }
}
```



---

1.12.4 使用@Configuration

是一个类级别的注解，表示这个对象是 bean定义的 源。  
@Configuration 类 声明bean 通过 @Bean 注解方法。调用 @Configuration 类中 @Bean 方法 可以用来定义 bean间依赖。



注入bean间依赖

当bean依赖另外的bean，表达这种依赖非常简单就像让一个bean方法调用另一个方法

```java
@Configuration
public class AppConfig {

    @Bean
    public BeanOne beanOne() {
        return new BeanOne(beanTwo());
    }

    @Bean
    public BeanTwo beanTwo() {
        return new BeanTwo();
    }
}
```

在上面的例子中，beanOne 收到一个 beanTwo的 ref。

这种声明bean间依赖的方法 只能用于 @Configuration 类中 @Bean 方法。你不能 通过 @Component类 声明bean间依赖。



---

查找Method注入

查找方法注入是一个高级功能，你应该很少使用。当 单例bean依赖 原型bean 很有用。使用java进行这种配置 为实现这种模式 提供了一种自然的方式。

```java
public abstract class CommandManager {
    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```

通过使用 java 配置，你可以 创建了一个 CommandManager 的子类，重写createCommond 方法，从而查找新的(prototype)命令对象。

```java
@Bean
@Scope("prototype")
public AsyncCommand asyncCommand() {
    AsyncCommand command = new AsyncCommand();
    // inject dependencies here as required
    return command;
}

@Bean
public CommandManager commandManager() {
    // return new anonymous implementation of CommandManager with createCommand()
    // overridden to return a new prototype Command object
    return new CommandManager() {
        protected Command createCommand() {
            return asyncCommand();
        }
    }
}
```



---

基于java的配置的如何工作的 更多信息

考虑下面的例子，展示了@Bean 注解的方法，被调用了2次：

```java
@Configuration
public class AppConfig {

    @Bean
    public ClientService clientService1() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientService clientService2() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientDao clientDao() {
        return new ClientDaoImpl();
    }
}
```

spring中，实例化的bean 默认是单例。所有 @Configuration类 在启动时都使用 CGLIB 子类化。在子类中，子方法在调用父方法 并创建 新实例之前，首先检查容器中是否有任何 缓存的 bean。

由于cglib在启动时动态添加功能，因此存在一些限制。特别是，配置类不能是 final。但是，从4.3开始，配置类上允许使用 任何构造器，包括@Autowired 或单个 非默认构造其 声明 进行默认注入。

如果你希望比买你任何cglib的限制，请考虑在非@Configuration 类上声明你的 @Bean 方法(例如，改为在普通的 @Component 上)。@Bean方法之间的跨方法调用不会被拦截，因此你必须完全以来构造器 或方法级别的以来注入。

---

1.12.5 组合基于java的配置

使用@Import注解

就像xml中 import 标签 来帮助模块化配置一样，@Import 注解允许从另一个配置类加载@Bean 定义。

。。@Configuration的 @Bean 调用 本类的@Bean 会自动进行依赖注入， 调用其他@Configuration类的 @Bean 方法呢？。。而且之前没有@Import。。   
。。。。。pu，其他@Configuration类，你都没有实例。不过 @Configuration 确实是一个 @Component，所以可以注入进来？ 然后调用这个 bean的 @Bean 方法？。。。还是 同一个 @Configuration 吧。

```java
@Configuration
public class ConfigA {

    @Bean
    public A a() {
        return new A();
    }
}

@Configuration
@Import(ConfigA.class)
public class ConfigB {

    @Bean
    public B b() {
        return new B();
    }
}
```

在实例化context时，只需要指明 ConfigB。

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(ConfigB.class);

    // now both beans A and B will be available...
    A a = ctx.getBean(A.class);
    B b = ctx.getBean(B.class);
}
```

简化了容器实例化，因为只需要写一个类，而不是要求你记住所有的@Configuration类。

从4.2开始，@Import还支持对常规component类的引用，类似 AnnotationConfigApplicationContext.register 方法。如果你想通过使用一些配置类作为入口点来显示定义所有组建来避免组件扫描，这将特别有用。



---

注入依赖在导入的@Bean 定义上

前面的示例有用，但过于简单。大多数场景中，bean 跨配置类 相互依赖。  
当使用xml时，这不是问题，因为不涉及编译器，你可以声明 ref="someBean" 并 信任spring在容器初始化时能解决依赖。   
当使用@Configuration类时，java编译器会对配置模型施加约束，因为对其他bean的引用必须是 有效的java语法。

幸运的是，解决这个问题很简单。@Bean方法可以有任意数量的参数 来描述bean 依赖关系。   
考虑下面的场景，其中包含 多个 @Configuration类，每个类都依赖于其他类中声明的bean：

```java
@Configuration
public class ServiceConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}

@Configuration
public class RepositoryConfig {

    @Bean
    public AccountRepository accountRepository(DataSource dataSource) {
        return new JdbcAccountRepository(dataSource);
    }
}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }
}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

这是另一个方法来 达到相同的目的，记住，@Configuratio类 最终只是容器中的 一个bean：这意味这它们可以利用 @Autowired 和@Value 注入 以及其他bean相同的特性。    
。。。但是，@Configuration A注入到 @Configuration B 后， B里的 @Bean 方法 调用 A.@Bean XXX方法，这种能声明依赖么？

确保你以这种方式注入的依赖只是最简单的类型。@Configuration类在 context初始化期间 很早被处理，并且强制以这种方式注入依赖项可能导致 意外的 早期初始化。 尽可能 使用基于参数的注入。

此外，特别注意通过 @Bean 定义的 BeanPostProcessor, BeanFactoryPostProcessor。这些通常应该被声明为 static @Bean 方法，而不是 触发包含它们的配置类的实例化。否则，@Autowired 和@Value 可能无法在配置类本身上工作，因为可能在 AutowiredAnnotationBeanPostProcessor 之前将其创建为bean实例。

下面是 一个bean如何自动装配到另一个bean

```java
@Configuration
public class ServiceConfig {

    @Autowired
    private AccountRepository accountRepository;

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(accountRepository);
    }
}

@Configuration
public class RepositoryConfig {

    private final DataSource dataSource;

    public RepositoryConfig(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }
}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }
}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

从4.3开始，才支持 @Configuration 类的 构造器注入。另外注意，如果目标bean 仅定义了一个构造器，则不需要显式@Autowired。

---

Fully-qualifying imported beans for ease of navigation
。。。感觉是 高质量的导入，为了更简单的查找。

在上面的场景中，使用 @Autowired 工作的很好，且提供了 所需的模块，但是 要确定在哪里声明了 装配的bean 定义仍然有些 模糊。   
例如，当一个开发者 查看 ServiceConfig类，你如何确定 @Autowired AccountRepository bean在哪里被定义？代码中没有明确说明。记住，STS 提供了工具可以 渲染显示 所有连接方式的图形，这可能就是你所需要的。另外，你的IDE 可以轻松找到 AccountRepository 类型的 所有声明和使用，并快速向您显示返回该类型的 @Bean方法的位置。

如果这种歧义不可接受，并且你希望在IDE中 从一个 @Configuration 直接导航到另一个，请考虑自动装配配置类本身：

```java
@Configuration
public class ServiceConfig {

    @Autowired
    private RepositoryConfig repositoryConfig;

    @Bean
    public TransferService transferService() {
        // navigate 'through' the config class to the @Bean method!
        return new TransferServiceImpl(repositoryConfig.accountRepository());
    }
}
```

。。。谜团解开了。。可以 跨@Configuration 调用 @Bean 方法 来提供 依赖。。。别弄那种 一个接口 N个类的。。不然注入了接口 也完蛋，不知道是哪个类。。

在上面的例子中， AccountRepository 被显式定义。但是，现在ServiceConfig 和 RepositoryConfig 紧耦合。通过使用 基于接口 或基于抽象类的 @Configuration 类可以在一定程度上 缓解这种紧密耦合。

```java
@Configuration
public class ServiceConfig {

    @Autowired
    private RepositoryConfig repositoryConfig;

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(repositoryConfig.accountRepository());
    }
}

@Configuration
public interface RepositoryConfig {

    @Bean
    AccountRepository accountRepository();
}

@Configuration
public class DefaultRepositoryConfig implements RepositoryConfig {

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(...);
    }
}

@Configuration
@Import({ServiceConfig.class, DefaultRepositoryConfig.class})  // import the concrete config!
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return DataSource
    }

}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

。。。接口 和 类 都必须要 @Configuration 和 @Bean ？

现在ServiceConfig 与具体的 DefaultRepositoryConfig 松耦合，内置的IDE工具依然游泳：你可以轻松获得 RepositoryConfig 实现的 类层次结构。通过这种方式，导航@Configuration 类及其依赖项与导航基于接口的代码的过程没有什么不同。

如果你想影响某些bean的创建顺序，考虑将其中一些声明为@Lazy 或 @DependsOn 。



---

有条件地包含 @Configuration 类 或 @Bean 方法

有用的：有条件地 启用或 禁用 一整个@Configuration 类 或 单独的@Bean 方法，基于一些系统状态。

一个普通的例子是 使用 @Profile注解 来 当一个特殊的profile在spring的Environment中被启用时，激活bean。

@Profile 是通过 @Conditional 来实现的。@Conditional 指明了一个spring的 Condition接口的 实现，这个实现应该在 @Bean 被注册前 先咨询下。

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(ProfileCondition.class)
public @interface Profile {

	/**
	 * The set of profiles for which the annotated component should be registered.
	 */
	String[] value();

}
```

```java
class ProfileCondition implements Condition {

	@Override
	public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
		MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
		if (attrs != null) {
			for (Object value : attrs.get("value")) {
				if (context.getEnvironment().acceptsProfiles(Profiles.of((String[]) value))) {
					return true;
				}
			}
			return false;
		}
		return true;
	}

}
```

---

组合Java 和 xml 配置

@Configuration 并不是旨在 成为 xml的100%替换。一些设施，比如spring xml 命名空间，仍然是配置容器的理想方式。在xml方便或必要的情况下，你有一个选择：要么通过使用 ClassPathXmlApplicationContext 以 "以xml为中心" 的方式 实例化容器，要么通过 AnnotationConfigApplicationContext 和 @ImportResource 根据需要导入xml。

。。xml 或注解 都可以作为 中心。



---

以XML为中心时 使用 @Configuration

可能是完美的：使用xml引导spring 容器，并以特别的方式包含@Configuration 类。  
例如，在使用xml的大型现代代码库中，根据需要 创建@Configuration 类，并从现有xml 文件中 包含它们 会更容易。在本节后面，我们会介绍这种 "以XML为中心"的情况下，使用@Configuration。

---

将@Configuration类 声明为 普通 spring bean 标签。  

记住，@Configuration类是 容器中的 bean定义。在这一系列的例子中，我们创建@Configuration 类 名字是AppConfig，包含它到 system-text-config.xml 作为一个 bean标签。因为context:annotation-config 被打开，容器意识到 @Configuration 注解 并处理 定义在AppConfig 中的 @Bean 方法声明。

```java
@Configuration
public class AppConfig {

    @Autowired
    private DataSource dataSource;

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }

    @Bean
    public TransferService transferService() {
        return new TransferService(accountRepository());
    }
}
```

下面是system.test-config.xml:

```xml
<beans>
    <!-- enable processing of annotations such as @Autowired and @Configuration -->
    <context:annotation-config/>
    
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>

    <bean class="com.acme.AppConfig"/>

    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```

jdbc.properties 文件：

```properties
jdbc.url=jdbc:hsqldb:hsql://localhost/xdb
jdbc.username=sa
jdbc.password=
```

```java
public static void main(String[] args) {
    ApplicationContext ctx = new ClassPathXmlApplicationContext("classpath:/com/acme/system-test-config.xml");
    TransferService transferService = ctx.getBean(TransferService.class);
    // ...
}
```

在system-test-config.xml 文件中，AppConfig bean标签没有声明id属性。虽然可以增加一个id属性，但是这不是必要的，因为没有其他bean 引用它，并且不太可能通过名字从容器中显示获取。同样，DataSource bean仅by-type 进行autowire(。。被@Autowired)，所以明确的id 也不是严格要求。

。。这里是开启了注解配置，没有开启扫描，所以需要自己把 注解的类 引入到 xml中。

---

使用 context:component-scan 来获得 @Configuration 类

为@Configuration 被 @Component注解，所以@Configuration的类 也自动被 扫描。在上面的场景中，我们可以重新定义 system-test-config.xml 来使用 component-scanning。记住，这里，我们不需要显式声明 context:annotation-config, 因为context:component-scan 启用相同的功能。

下面是system-test-config.xml

```xml
<beans>
    <!-- picks up and registers AppConfig as a bean definition -->
    <context:component-scan base-package="com.acme"/>
    
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>

    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```

---

以@Configuration 为中心，通过@ImportResource 来使用 xml

在@Configuration 类是配置容器的主要机制的app中，仍然可能需要一些xml。在这些场景中，你可以使用 @ImportResource 导入需要的xml。  
这样做实现了"以java为中心" 的方式 来配置容器并将xml保持在最低限度。   
以下示例(包括一个配置类，一个定义bean的xml文件，一个属性文件和主类) 展示了如何使用@ImportResource 注解来实现"以java为中心"的配置，并根据需要使用xml：

```java
@Configuration
@ImportResource("classpath:/com/acme/properties-config.xml")
public class AppConfig {

    @Value("${jdbc.url}")
    private String url;

    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;

    @Bean
    public DataSource dataSource() {
        return new DriverManagerDataSource(url, username, password);
    }
}
```

properties-config.xml

```xml
<beans>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>
</beans>
```

jdbc.properties

```properties
jdbc.url=jdbc:hsqldb:hsql://localhost/xdb
jdbc.username=sa
jdbc.password=
```

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    TransferService transferService = ctx.getBean(TransferService.class);
    // ...
}
```



















---

#### 1.13 Environment Abstraction

Environment 接口是 集成在容器中的 抽象，它对 app的环境的 2个关键方面进行建模：profiles 和 properties。

profile 是 named，逻辑组 of bean定义，这些bean定义 会被注册到容器 只有当  profile被激活。  
bean(无论xml还是注解) 可能被关联到了 profile。和profile有关的 Environment的作用是 确定 哪些profile 是当前激活的，哪些profile是默认激活的

properties 在几乎所有的应用中都扮演者重要的角色，并且可能有多个来源：properties文件，JVM系统参数，系统环境变量，JNDI，servlet context参数，ad-hoc Properties对象，Map对象 等。Environment对象与属性相关的作用 是为用户提供一个方便的服务接口，用于配置属性源并从中解析属性。



---

1.13.1 Bean定义 profile

bean定义 profile 在核心容器中 提供了一个机制，允许在不同环境中 注册不同bean。"environment" 这个词，可能对不同用户 有不同的含义，这个功能可以帮助许多用例，包括：

1. 在开发中处理内存中的数据源，而不是像在QA或生产中那样通过JNDI来查找数据源。
2. 仅在部署app到 性能环境时，才注册 基础监控架构。
3. 为客户A和客户B 注册定时的bean实现。

考虑第一个 数据源的 用例。在测试环境中，配置可能类似下面：

```java
@Bean
public DataSource dataSource() {
    return new EmbeddedDatabaseBuilder()
        .setType(EmbeddedDatabaseType.HSQL)
        .addScript("my-schema.sql")
        .addScript("my-test-data.sql")
        .build();
}
```

现在考虑如何发布应用到 QA或生产环境，假设app的数据源 已在生产的应用服务器的 JNDI 目录中注册，我们的dataSource bean：

```java
@Bean(destroyMethod="")
public DataSource dataSource() throws Exception {
    Context ctx = new InitialContext();
    return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
}
```

问题是，如何根据当前的环境 切换这2个数据源。随着时间推移，spring用户 设计了许多方法来完成此操作，通常依赖系统环境变量和 使用${placeholder}标记的xml的 import标签的组合，这些标记根据值被解析为 正确的配置文件路径的一个环境变量。bean定义 profile是 核心容器的一个特性，它为这个问题提供了解决方案。

如果我们概括前面的 环境特定的bean定义 的用例，我们最终需要在某些context中注册 某些bean定义，但其他context 中不需要。   
你可以说，你想在情况A中注册特定的bean定义配置文件，在情况B中注册不同的配置文件。我们首先更新配置以反映这种需求。



---

使用 @Profile

这个注解让你指示 一个组件 是否可以注册，当一个或多个profile 激活时。   
我们重写dataSource 配置：

```java
@Configuration
@Profile("development")
public class StandaloneDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }
}
```

```java
@Configuration
@Profile("production")
public class JndiDataConfig {

    @Bean(destroyMethod="")
    public DataSource dataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```

如前所述，使用@Bean方法，你通常选择使用编程JNDI查找，通过使用spring的JndiTemplate 或 JndiLocatorDelegate，或前面的直接 jndi InitialContext，而不是 JndiObjectFactoryBean 变体，这将强制你将返回类型 声明为 FactoryBean 类型。



profile 可以包含简单profile name(如 production) 或一个 profile 表达式。profile 表达式允许更复杂的 profile 逻辑(如， production & us-east) ，下面是可以用于 profile 表达式的 操作符：

1. !
2. &
3. |

混合 & | 操作符时，必须带()。

你可以使用 @Profile 作为元注解 来创建自定义组合注解。下面的定义了@Production，你可以用来替换@Profile("production")

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Profile("production")
public @interface Production {
}
```

如果@Configuration 类被 标记为 @Profile，则与该类关联的 所有的@Bean方法 和 @Import 注解都将被绕过，除非 一个或多个指定的 profile 的处于激活状态。   
如果@Component 或 @Configuration 类被 标记为 @Profile({"p1","p2"})，这个类不会被注册或处理 除非 profile 'p1' 或 'p2' 被激活。如果一个 给定的profile 的前缀是 NOT操作符( ! )，那么被注解的元素 被注册，只有当profile 没有被激活。例如，@Profile({"p1","!p2"})，会进行注册，只有当 p1激活 或 p2没有激活时。

@Profile 也可以被声明在 配置类的 @Bean 方法层：

```java
@Configuration
public class AppConfig {

    @Bean("dataSource")
    @Profile("development") 
    public DataSource standaloneDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }

    @Bean("dataSource")
    @Profile("production") 
    public DataSource jndiDataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```

。。这个能 @Profile 在 类 且 方法上吗？ 2个 @Profile 是 &&的关系？

通过将 @Profile 放到 @Bean 方法，可能会应用一种特殊情况：在重载具有相同Java 方法名称的 @Bean 方法的情况下(类似于构造器重载)，  

> 所有的重载方法需要声明相同的 @Profile 条件。  

如果条件不一致，则仅重载方法中 第一个声明的条件。因此，@Profile不能用于选择具有特定参数签名的重载方法。同一bean的所有工厂方法之间的解析在创建时遵循spring的 构造器解析算法。

如果你要定义具有 不同配置profile条件的 bean，请使用不同的java方法名，这些方法名通过使用 @Bean 的name 属性来指定 相同的 bean名字。

---

XML Bean 定义 profile

xml中是 beans标签的 profile 属性。

。。。beansssssssssss

```xml
<beans profile="development"
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xsi:schemaLocation="...">

    <jdbc:embedded-database id="dataSource">
        <jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
        <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
    </jdbc:embedded-database>
</beans>
```

```xml
<beans profile="production"
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xsi:schemaLocation="...">

    <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
</beans>
```

也可以避免分为2个文件，通过 嵌套的 beans 标签。

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xsi:schemaLocation="...">

    <!-- other bean definitions -->

    <beans profile="development">
        <jdbc:embedded-database id="dataSource">
            <jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
            <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
        </jdbc:embedded-database>
    </beans>

    <beans profile="production">
        <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
    </beans>
</beans>
```

spring-bean.xsd 已经被限制：这些元素必须是文件的 最后部分。这有助于提供灵活性，而不会在xml中产生混乱。

xml不支持前面描述的 profile 表达式。但是可以 negate 一个profile 通过 ! 操作符。也可以使用 逻辑的"and" 通过 嵌套profile：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xsi:schemaLocation="...">

    <!-- other bean definitions -->

    <beans profile="production">
        <beans profile="us-east">
            <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
        </beans>
    </beans>
</beans>
```

上面的例子中，只有 production 和 us-east 都激活时，才有 dataSource。



---

激活profile

我们已经更新了配置，我们仍然需要指导spring：那个profile是激活的。如果我们开始我们的app，会收到NoSuchBeanDefinitionException，因为容器找不到dataSource。

多种方法来激活profile。最简单的是编程式调用Environment的API，Environment可以通过AppCtx 获得：

```java
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
ctx.getEnvironment().setActiveProfiles("development");
ctx.register(SomeConfig.class, StandaloneDataConfig.class, JndiDataConfig.class);
ctx.refresh();
```

另外，你也可以声明激活的profile 通过 spring.profiles.active 属性，这个可以放在 系统环境变量，jvm参数，web.xml的servlet context parameters，或作为JNDI 的entry。  
在集成测试中，激活的profile 可以使用 @ActiveProfiles 到spring-test模块。

记住，profiles不是 非此即彼的。你可以同时激活多个profile，编程式，你可以提供多个profile name 到 setActiveProfiles()，这个接受String... 参数。

```java
ctx.getEnvironment().setActiveProfiles("profile1", "profile2");
```

声明式，spring.profiles.active 可以接受一个 逗号分隔的 profile name的列表：

```
-Dspring.profiles.active="profile1,profile2"
```



---

Default Profile

default profile 表示 这个profile 被默认激活：

```java
@Configuration
@Profile("default")
public class DefaultDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .build();
    }
}
```

如果没有profile 被激活，dataSource 被创建。这种一种提供一个默认定义的方法。如果有其他profile被激活，default profile 就不会被激活。

你可以修改 默认profile 的名字 by Environment.setDefaultProfiles()，或通过spring.profiles.default 属性来声明。   
。。。Environment 没有这个方法，这个方法是 ConfigurableEnvironment 的。ConfigurableApplicationContext 才有 getEnvironment 方法，返回一个 ConfigurableEnvironment。



---

1.13.2 PropertySource 抽象

spring的Environment 抽象提供了搜索操作 对一个可配置的property sources 继承结构。

```java
ApplicationContext ctx = new GenericApplicationContext();
Environment env = ctx.getEnvironment();
boolean containsMyProperty = env.containsProperty("my-property");
System.out.println("Does my environment contain the 'my-property' property? " + containsMyProperty);
```

在上面的例子中，我们看到一个 高层 的方法来 询问spring：当前环境中 my-property 属性是否被定义。   
为了回答这个问题，Environment对象执行一个搜索 对 PropertySource 集合。PropertySource 是一个简单抽象 对任何k-v对的 源，spring的 StandardEnvironment 被配置 2个 PropertySource 对象，一个是 jvm系统参数(System.getProperties())，一个是系统环境变量(System.getenv())

这些默认属性源存在于StandardEnvironment中，用于独立应用程序。StandardServletEnvironment填充了其他默认属性源，包括servlet配置和servlet上下文参数。它可以选择启用 JndiPropertySource

具体来说，当你使用StandardEnvironment，对env.containsProperty("my-property") 的调用会返回 true，如果在运行时 存在一个 my-property 的系统属性，或 my-property的环境变量。

search是分层执行的。默认下，先搜索系统属性，然后环境变量。所以，如果my-property 在2个地方都存在，那么系统属性win 且被返回。 属性值不会合并，只是返回 高优先的。

对于一个普通的 StandardServletEnvironment，全部的继承如下，高优先的 先：

1. ServletConfig 参数，如果存在，例如一个DispatcherServlet 上下文。
2. ServletContext 参数，web.xml中的 content-param 
3. JNDI 环境变量，java:comp/env/ 
4. jvm系统属性，-D 命令行参数
5. jvm系统环境(os环境变量)

。。。上面说了半天 系统变量，结果是 jvm的启动参数，环境变量参数os的。。。-Xmx是哪层的。

最重要的，整个机制是可配置的。可能你有一个自定义property source，你想集成到这个搜索中。你可以：实现和实例化你的PropertySource，然后加入到 当前 Environment的 PropertySource 集合：

```java
ConfigurableApplicationContext ctx = new GenericApplicationContext();
MutablePropertySources sources = ctx.getEnvironment().getPropertySources();
sources.addFirst(new MyPropertySource());
```

上面的代码中，MyPropertySource 被添加为 最高优先 在search时。



---

1.13.3 使用  @PropertySource

这个注解提供了一个 简单的和声明机制 for 增加一个 PropertySource 到 spring的 Environment。  

一个文件名叫 app.properties，包含了k-v对：testbean.name=myTestBean， 下面的使用了 @PropertySource 的 @Configuration 类中使用 testBean.getName() 返回 myTestBean:

```java
@Configuration
@PropertySource("classpath:/com/myco/app.properties")
public class AppConfig {

    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
}
```

@PropertySource 资源地址中的 ${} 会被解析 通过 已经注册到当前环境的 property sources。

```java
@Configuration
@PropertySource("classpath:/com/${my.placeholder:default/path}/app.properties")
public class AppConfig {

    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
}
```

假设 my.placeholder 存在于 一个已经注册的 property source中，这个占位符会被解析为对应的值。如果没有，那么使用 defualt/path。如果没有 默认的，且属性无法被解析，抛出 IllegalArgumentException。

根据java8约定，@PropertySource 是可重复的。但是，所有@PropertySource 需要被声明在 同一级，或直接在配置类上，或作为 元注解到 同一个自定义注解。不建议混合使用 注解 和元注解，因为 注解覆盖了 元注解。



---

1.13.4 Placeholder Resolution in Statements

从历史上看，元素中占位符的值只能根据jvm系统属性或环境变量 来解析。现在不是了。   
因为Environment 抽象 被集成到容器中，所以很容易通过它来确定 占位符的值。这意味着你可以以任何你喜欢的方式配置解析过程。你可以更改搜索 系统属性和 环境变量的 优先级或完全删除他们。你可以根据需要添加自己的属性源。

具体来说，无论 customer 在哪里定义，只要它在Environment 中可用，以下语句有效：

```xml
<beans>
    <import resource="com/bank/service/${customer}-config.xml"/>
</beans>
```

















---

1.14 Registering a LoadTimeWeaver

spring使用 LoadTimeWeaver 来动态transform 类，当他妈被加载到 jvm时。

为了启用 load-time weaving，你需要增加 @EnableLoadTimeWeaving 到 你的某个@Configuration 类。

```java
@Configuration
@EnableLoadTimeWeaving
public class AppConfig {
}
```

xml形式：

```xml
<beans>
    <context:load-time-weaver/>
</beans>
```

一旦为 AppCtx 配置，这个AppCtx中的 所有bean 都可以实现 LoadTimeWeaverAware，从而接收对 load-time weaver 实例的ref。这在 结合spring的JPA 支持时非常有用，JPA类转换可能需要 load-time weaving。查看LocalContainerEntityManagerFactoryBean 获得更多信息。







---

1.15 AppCtx的额外能力

org.springframework.beans.factory 包提供了 基础功能 来管理和 操作bean，包括编程的方式。   
org.springframework.context 包增加了 ApplicationContext 接口，这个扩展了 BeanFactory 接口，增加接口来提供额外的能力 以一个更加面向框架编程的方式。   
许多人以完全声明的方式使用 AppCtx，甚至没有以编程的方式创建它，而是依赖 ContextLoader 等支持类 来自动实例化AppCtx，作为Java EE Web 应用程序 正常启动过程的一部分。

为了以更加面向框架的风格 增强 BeanFactory 功能，context 包 还包含下面的功能：

1. 通过 MessageSource 接口以 i18n style 访问消息。
2. 通过 ResourceLoader 接口访问资源，例如URL 和文件。
3. Event publication，通过使用 ApplicationEventPublisher 接口发布到 实现 ApplicationListener 接口的bean。
4. 加载多个( 分层 )上下文，通过HierarchicalBeanFactory 接口，让每个上下文 都专注于 一个特定的层，例如app的 web 层。



---

1.15.1 Internationalization 使用 MessageSource

AppCtx 接口扩展了 MessageSource 接口，所以提供了 internationalization 能力。spring 也提供了 HierarchicalMessageSource 接口，这个可以 hierarchically(体系地，分层地) 解析消息。这些接口共同提供了spring影响消息解析的基础，这些接口提供的方法：

```
String getMessage(String code, Object[] args, String default, Locale loc)
```

基础方法，用来检索消息 从 MessageSource。如果对指定的 locale 找不到 消息，使用default参数作为消息。使用标准库提供的MessageFormat功能，传入的任何参数都将成为替换值。

```
String getMessage(String code, Object[] args, Locale loc)
```

和前一个方法基本相同，区别：不能只能默认消息，找不到就NoSuchMessageException

```
String getMessage(MessageSourceResolvable resolvable, Locale locale)
```

之前的方法中用到的 所有属性 也都被封装在 MessageSourceResolvable 类中，你可以将其与这个方法一起使用。



加载AppCtx时，它会自动搜索context中 定义的 MessageSource bean。这个bean的名字必须是 messageSource。如果有这样的bean，任何 对于 之前的方法的调用 都 委托给 message source。   
如果没有message source，AppCtx 尝试查找 包含同名bean 的 父级。如果找到，它将使用这个bean 作为 MessageSource。如果AppCtx找不到任何 消息源，则实例化一个空的 DelegateingMessageSource 以便能够接受对上面定义的方法的调用。



spring 提供了3个MessageSource实现，ResourceBundleMessageSource，ReloadableResourceBundleMessageSource，StaticMessageSource。   
它们都实现了 HierarchicalMessageSource 以进行 嵌套消息传递。StaticMessageSource 很少使用，但是提供了编程方式 将 消息添加到 源。下面是ResourceBundleMessageSource:

```xml
<beans>
    <bean id="messageSource"
            class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basenames">
            <list>
                <value>format</value>
                <value>exceptions</value>
                <value>windows</value>
            </list>
        </property>
    </bean>
</beans>
```

上面的例子假设你 有3个 resource bundle(资源包) ，分别是 format，exception，windows，定义在你的 classpath。任何解析消息的请求 都以jdk标准的方式处理，即通过ResourceBundle对象解析消息。   
假设，上述2个资源包文件内容如下：

```properties
    # in format.properties
    message=Alligators rock!
```

```properties
    # in exceptions.properties
    argument.required=The {0} argument is required.
```

下面展现了代码 来使用 MessageSource 功能。记住所有 ApplicationContext 也是 MessageSource，所以可以强转：

```java
public static void main(String[] args) {
    MessageSource resources = new ClassPathXmlApplicationContext("beans.xml");
    String message = resources.getMessage("message", null, "Default", Locale.ENGLISH);
    System.out.println(message);
}
```

输出是：

```
Alligators rock!
```



总之，MessageSource 在 beans.xml 文件中 定义，这个文件存在于classpath的 根目录中。messageSource bean定义 引用了许多资源包 通过 basenames 属性。在列表中传递给basenames 属性的3个文件存放在 classpath 根目录，分别是 foramt.properties, exceptions.properties, windows.properties.

下面展示了传递给消息查找的参数。这些参数被转换为String对象并插入到 消息中的占位符中。

```xml
<beans>

    <!-- this MessageSource is being used in a web application -->
    <bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basename" value="exceptions"/>
    </bean>

    <!-- lets inject the above MessageSource into this POJO -->
    <bean id="example" class="com.something.Example">
        <property name="messages" ref="messageSource"/>
    </bean>

</beans>
```

```java
public class Example {

    private MessageSource messages;

    public void setMessages(MessageSource messages) {
        this.messages = messages;
    }

    public void execute() {
        String message = this.messages.getMessage("argument.required",
            new Object [] {"userDao"}, "Required", Locale.ENGLISH);
        System.out.println(message);
    }
}
```

execute() 方法的输出：

```
The userDao argument is required.
```

关于i18n，spring的各种MessageSource 实现 遵循与标准JDK ResourceBundle 相同的语言环境解析和回退规则。简而言之，继续前面定义的例子，如果你想针对 英语(en-GB) 环境解析消息，你将创建名为 format_en_GB.properties, exceptions_en_GB.properties, windows_en_GB.properties 文件。

通常，区域解析 由 应用程序的周围环境 管理。在下面的例子中，手动指定 解析消息 的区域。

```properties
# in exceptions_en_GB.properties
argument.required=Ebagum lad, the ''{0}'' argument is required, I say, required.
```

```java
public static void main(final String[] args) {
    MessageSource resources = new ClassPathXmlApplicationContext("beans.xml");
    String message = resources.getMessage("argument.required",
        new Object [] {"userDao"}, "Required", Locale.UK);
    System.out.println(message);
}
```

上面的结果是：

```
Ebagum lad, the 'userDao' argument is required, I say, required.
```



你也可以使用 MessageSoureAware 接口 来获取 对已定义的 任何MessageSource 的引用。在创建和配置bean时，在AppCtx中 所有 实现MessageSoureAware 接口的 bean 都会被注入 context 的 MessageSource。

因为spring的 MessageSource 基于 java的 ResourceBundle，它不会合并 相同的 base name的 bundle，但只使用 找到的 第一个bundle。后续的相同的 base name 的 bundle 会被 忽略。

作为ResourceBundleMessageSource 的另一种，spring 提供了一个 ReloadableResourceBundleMessageSource 类。这个支持相同的 包文件格式，但是比 基于标准jdk的 ResourceBundleMessageSource 更灵活。特别是，它允许从任何spring资源位置(不仅从classpath) 读取文件，并支持属性文件的 热重载(同时 在它们之间有效地缓存它们)。



---

1.15.2 标准和自定义事件

AppCtx 中的 事件处理 是通过 ApplicationEvent 类 和 ApplicationListener 接口 提供的。   
如果将实现ApplicationListener 接口的 bean 部署到 context 中，则每次将ApplicationEvent 发布到 ApplicationContext 时，都会通知该bean。本质上，这是标准的观察者设计模式。

从4.2开始，事件基础机构得到显著改进，并提供了基于注释的模型和发布任意事件的能力(即，不一定从ApplicationEvent 扩展的对象)。当这样的对象发布时，我们会为你将其包装在一个事件中。

下面是spring 提供的 标准事件的描述：

| Event                      | Explanation                                                  |
| -------------------------- | ------------------------------------------------------------ |
| ContextRefreshedEvent      | 发布，当AppCtx被初始化 或 刷新(例如，使用ConfigurableApplicationContext接口的 refresh() 方法)。"被初始化" 意味着所有的bean 都加载了，post-processor bean被检测并激活，单例已预先实例化，ApplicationContext 对象已准备好使用。只要context没有被关闭，refresh 能被触发多次，前提是所选的 AppCtx 实际上支持这种 "热"刷新。例如，XmlWebApplicationContext 支持热刷新，但GenericApplicationContext 不支持。 |
| ContextStartedEvent        | 发布，当AppCtx 被启动 通过 ConfigurableApplicationContext 接口的 start() 方法。这里"started" 是指 所有的 Liftcycle bean 收到了一个 显式的 启动信号。通常，此型号用于在显式停止后重新启动bean，但它也可以用于启动尚未配置为自动启动的组件( 例如，尚未在初始化时启动的组件)。 |
| ContextStoppedEvent        | 发布，当AppCtx被停止 通过 ConfigurableAppCtx 的 stop()。这里 停止 意味着所有的 Lifecycle bean 收到一个 显式stop 信号。一个 stopped 的context 可以通过 start() 来 重启。 |
| ContextClosedEvent         | 发布，当AppCtx 将被关闭 通过ConfigurableAppCtx 的 close() 或 jvm的 shutdown hook。closed 意味着所有单例已销毁。 一旦context是closed，它就达到了生命的终点，不能 refresh 和 restart。 |
| RequestHandledEvent        | 一个特定于web的事件，告诉所有bean，一个http请求 已得到服务。此事件在请求完成后发布。此事件仅适用于spring的 DispatcherServlet的 web app |
| ServletRequestHandledEvent | RequestHandledRequest的子类，增加了 servlet特定的 上下文信息。 |



你也可以创建和发布自己的自定义事件。下面是一个简单类，扩展了ApplicationEvent：

```java
public class BlockedListEvent extends ApplicationEvent {

    private final String address;
    private final String content;

    public BlockedListEvent(Object source, String address, String content) {
        super(source);
        this.address = address;
        this.content = content;
    }

    // accessor and other methods...
}
```

为了发布自定义 ApplicationEvent，调用 ApplicationEventPublisher的 publishEvent()。通常，这是通过创建一个 实现ApplicationEventPublisherAware 的类并将其注册为 spring bean 来完成的:

```java
public class EmailService implements ApplicationEventPublisherAware {

    private List<String> blockedList;
    private ApplicationEventPublisher publisher;

    public void setBlockedList(List<String> blockedList) {
        this.blockedList = blockedList;
    }

    public void setApplicationEventPublisher(ApplicationEventPublisher publisher) {
        this.publisher = publisher;
    }

    public void sendEmail(String address, String content) {
        if (blockedList.contains(address)) {
            publisher.publishEvent(new BlockedListEvent(this, address, content));
            return;
        }
        // send email...
    }
}
```

在配置时，spring容器检测到EmailService 实现了 ApplicationEventPublisherAware，就通过 setApplicationEventPublisher 注入了ApplicationEventPublisher。   

> 实际上注入的是 容器本身。

你通过 容器实现的 ApplicationEventPublisher 接口的方法 与 context 交互。

要接受自定义的ApplicationEvent，你可以创建一个实现了 ApplicationListener的类 并将其注册为 spring bean：

```java
public class BlockedListNotifier implements ApplicationListener<BlockedListEvent> {

    private String notificationAddress;

    public void setNotificationAddress(String notificationAddress) {
        this.notificationAddress = notificationAddress;
    }

    public void onApplicationEvent(BlockedListEvent event) {
        // notify appropriate parties via notificationAddress...
    }
}
```

注意，泛型，意味着onApplicationEvent 可以保证类型安全，避免向下强转。   
注意，默认下，event listeners 会同步地接受事件，这意味着 publishEvent 方法阻塞，直到所有 监听器都处理完事件。这种同步和单线程 方法的一个优点是：当监听器收到事件时，如果事务上下文可用，它会在 发布者的事务中运行。   
如果需要其他的事件发布策略，请看 ApplicationEventMulticaster 接口 和 SimpleApplicationEventMulticaster 实现。   
。。SimpleAppEventMulticaster 如果注入一个 Executor，那么它会使用这个线程池来 广播event，否则，用调用者线程广播。

下面的例子是上面的例子的 注册和配置 的bean 定义

```xml
<bean id="emailService" class="example.EmailService">
    <property name="blockedList">
        <list>
            <value>known.spammer@example.org</value>
            <value>known.hacker@example.org</value>
            <value>john.doe@example.org</value>
        </list>
    </property>
</bean>

<bean id="blockedListNotifier" class="example.BlockedListNotifier">
    <property name="notificationAddress" value="blockedlist@example.org"/>
</bean>
```

总而言之，当调用 emailService bean的sendEmail 方法时，如果有任何email消息应该被阻止，那么会发布一个 BlockedListEvent 类型的 自定义事件。blockedListNotifier bean注册为 ApplicationListener 并接受 BlockedListEvent，此时它可以通知适当的各方。

spring的事件机制是为 同一个application context 中的 bean 之间的 简单通信而设计的。   
对于复杂的企业集成需求，单独维护的 spring Integration 项目 为构建基于众所周知的 spring 编程模型的 轻量级，面向模式，事件驱动的框架提供了 完整的支持。



---

基于注解的Event listener

使用 @EvnetListener 注解到 一个bean的 任意方法上，这样可以注册一个 事件监听器。BlockedListNotifier 能重写为：

```java
public class BlockedListNotifier {

    private String notificationAddress;

    public void setNotificationAddress(String notificationAddress) {
        this.notificationAddress = notificationAddress;
    }

    @EventListener
    public void processBlockedListEvent(BlockedListEvent event) {
        // notify appropriate parties via notificationAddress...
    }
}
```

方法签名 声明了 它所监听的 事件类型，但是这次使用了一个灵活的名称 且没有实现特定的 监听器接口。只要实际事件类型 在其实现层次结构中解析你的泛型参数，也可以通过泛型来缩小事件类型。

如果你的方法应该监听多个事件，或者你想在没有参数的情况下定义它，也可以在注解上指定事件类型：

```java
@EventListener({ContextStartedEvent.class, ContextRefreshedEvent.class})
public void handleContextStart() {
    // ...
}
```

也可以增加运行时过滤 通过使用 condition 属性的 SpEL，这个应该与 实际调用特定事件的方法匹配。

下面是 只有 事件的content属性是 my-event 的才会 调用这个方法。

```java
@EventListener(condition = "#blEvent.content == 'my-event'")
public void processBlockedListEvent(BlockedListEvent blEvent) {
    // notify appropriate parties via notificationAddress...
}
```

每个SpEL 对 context进行 eval。下表列出了 context中可用的 item，你能使用这些item来 条件化 事件处理。

| name            | location           | desc.                                                        | example                                                      |
| --------------- | ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Event           | root object        | 实际的 ApplicationEvent                                      | #root.event 或 event                                         |
| Arguments array | root object        | Object[]形式实参                                             | #root.args，args；args[0]是地一个参数                        |
| Argument name   | evaluation context | 方法参数的名字。如果由于某些原因，名字不可获得(比如，由于在字节码中没有debug 信息)，单个参数也可以使用#a<#arg> 语法获得，其中<#arg> 代表参数索引( 从0 开始) | #blEvent , #a0 (你也可以使用#p0 或 #p<#arg> 参数符号作为 一个别名) |

记住，#root.event 让你访问 底层的event，即使你的方法签名实际上是指已发布的任意对象。

如果你需要发布一个事件 作为处理另一个时间的结果，你可以更改方法签名以返回该发布的事件：

```java
@EventListener
public ListUpdateEvent handleBlockedListEvent(BlockedListEvent event) {
    // notify appropriate parties via notificationAddress and
    // then publish a ListUpdateEvent...
}
```

异步监听器中不支持这个功能。

handleBlockedListEvent() 方法发布了一个新的 ListUpdateEvent for 每个被他处理的BlockedListEvent。如果你需要发布多个事件，你可以返回一个 集合 或 数组。



---

Asynchronous Listeners

如果你希望特定监听器 异步处理事件，则可以使用常规的 @Async：

```java
@EventListener
@Async
public void processBlockedListEvent(BlockedListEvent event) {
    // BlockedListEvent is processed in a separate thread
}
```

使用 异步事件的时候，了解下面的限制：

1. 如果 异步事件监听器 抛出一个 Exception，不会传播到 caller。
2. 异步事件监听器 方法 不能通过返回值来 发布后续事件。如果你需要 发布另一个事件作为 事件处理的结果，请注入一个 ApplicationEventPublisher 以手动发布。



---

Listeners 排序

如果你需要 一个listener 在 另一个listener 之前被调用，可以增加 @Order 注解到 方法上：

```java
@EventListener
@Order(42)
public void processBlockedListEvent(BlockedListEvent event) {
    // notify appropriate parties via notificationAddress...
}
```



---

泛型事件

你可以使用 泛型 来进一步 定义你的event 结构。考虑使用一个 ```EntityCreatedEvent<T>```，T是实际被创建的对象的类型。例如，你可以使用下面的定义来 只接收 创建Person 而引发的 EntityCreatetEvent：

```java
@EventListener
public void onPersonCreated(EntityCreatedEvent<Person> event) {
    // ...
}
```

由于类型擦除，这仅在 触发的事件 解析了 事件监听器过滤的泛型参数 时 才有效 (即，类似于class PersonCreatedEvent extends EntityCreatedEvent《Person》{ ... })。

在某些情况下，如果所有的事件都遵循相同的结构，这可能变得非常乏味。在这种情况下，你可以实现ResolvableTypeProvider 来指导框架 超出 运行时环境时提供的范围：

```java
public class EntityCreatedEvent<T> extends ApplicationEvent implements ResolvableTypeProvider {

    public EntityCreatedEvent(T entity) {
        super(entity);
    }

    @Override
    public ResolvableType getResolvableType() {
        return ResolvableType.forClassWithGenerics(getClass(), ResolvableType.forInstance(getSource()));
    }
}
```

这不仅可以 工作 for ApplicationEvent，也可以为 任何你 作为 event send的 对象。



---

1.15.3 方便地访问 低层资源

为了更好地使用 和 理解 应用上下文，你应该熟悉spring的 Resource 抽象。

应用上下文是一个 ResourceLoader，可以用于加载 Resource 对象。实际上，Resource 的实现 封装了 java.net.URL 实例。   
Resource 可以以透明的方式从几乎任何位置 获得 低级资源，包括 从类路径，文件系统位置，可以使用标准URL描述的任何地址以及其他一些变体。如果资源位置字符串是没有任何 特殊前缀的 简单路径，则这些资源的来源是特定的，并且适用于实际的应用程序上下文类型。

你可以配置 bean 到 应用上下文 来实现 特殊的回调接口，ResourceLoaderAware，以便在初始化时自动回调，并将AppCtx 本身作为 ResourceLoader 传入。   
你还可以公开Resource 类型的属性，用于访问静态资源。它们像其他属性一样被注入其中。你可以将这些Resource属性 指定为 简单的String 路径，并在部署bean 时 依赖 对这些string 到实际Resource对象的 自动转换。

AppCtx 构造器的一个或多个位置路径实际上是 资源字符串，根据特定的上下文实现 进行适当的处理。例如，ClassPathXmlApplicationContext 将简单的 位置路径视为 类路径位置。你还可以使用带有 特殊前缀的 位置路径 来强制从类路径 或 URL加载定义，而不管实际的上下文类型如何。



---

1.15.4 Application Startup Tracking

AppCtx 管理了 spring 应用的 生命周期，且 围绕 组件 提供了一个 丰富的编程模型。因此，复杂应用可以具有同样复杂的 组件图 和 启动阶段。

使用特定指标 追踪应用的启动步骤可以帮助 理解在启动阶段的 时间话费，也可以用作 更好地了解整个context 生命周期 的一种方式。

AbstractApplicationContext (及其子类) 使用 ApplicationStartup 进行检测，它收集有关各种启动阶段的 startup数据：

1. app ctx 生命周期 (基础包扫描，配置类管理)
2. bena 生命周期 (实例化，智能初始化，后处理)
3. 应用事件处理

。。。这个ApplicationStartup 是 5.3开始的。。

下面是AnnotationConfigApplicationContext 中的 基础代码：

```java
// create a startup step and start recording
StartupStep scanPackages = this.getApplicationStartup().start("spring.context.base-packages.scan");
// add tagging information to the current step
scanPackages.tag("packages", () -> Arrays.toString(basePackages));
// perform the actual phase we're instrumenting
this.scanner.scan(basePackages);
// end the current step
scanPackages.end();
```

application context 被分解为多步。记录后，可以使用特定工具手机，显示，分析这些启动步骤。启动步骤的完整列表，可以查看专用的附录部分。

默认的ApplicationStartup 实现是一个 无操作的 变体，用于最小的开销。这意味着，默认下，应用启动期间不会收集任何指标。spring 有一个使用 Java Flight Recorder 跟踪启动步骤的实现：FlightRecorderApplicationStartup。要使用这个，你必须在创建后立刻 将其实例配置到ApplicationContext   
开发者也可以使用 ApplicationStartup 基础结构，如果他们提供他们自己的AbstractApplicationContext 子类，或 如果他们希望收集更多的 精确数据。

ApplicationStartup 仅在 应用启动 期间 且 在核心容器中 使用。这不是 Java profilers 或 metrics 库( 如Micrometer)的 替代品。

要开始收集自定义StartupStep，组件可以直接从 应用上下文 获取 ApplicationStartup 实例，使其组件实现 ApplicationContextAware，或者在任何注入点请求 ApplicationStartup类型。

开发者在创建自定义 启动步骤时，不应该使用 "spring.*" 这个命名空间。这个空间是为内部的spring 使用 而保留的，并且可能发生变化。



---

1.15.5 方便的ApplicationContext 实例 for web应用

你可以 声明式 创建一个 AppCtx 实例 通过使用 例如 ContextLoader。 当然，你也可以编程式创建 ApplicationContext 实例 通过使用 ApplicationContext 的某个 实现。

通过ContextLoaderListener 来注册一个 ApplicationContext :

```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/daoContext.xml /WEB-INF/applicationContext.xml</param-value>
</context-param>

<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

listener 检查 contextConfigLocation 参数，如果参数不存在，默认使用/WEB-INF/applicationcontext.xml。   
当参数存在，切分 string 通过预定义的分隔符(逗号，分号，空格)，且使用这些值做为地址，当app ctx 进行搜索时。   

Ant-风格的path pattern 也支持。例如 /WEB-INF/*Context.xml 和 /WEB-INF/\*\*/\*Context.xml，前者是在WEB-INF 目录下的 所有以 Context.xml 为结束的 文件， 后者是 WEB-INF 及其 子文件夹中的。



---

1.15.6 部署AppCtx 作为一个 Java EE RAR 文件

。。。这节漏了很多。

可以把AppCtx 部署为一个 RAR文件，将上下文和所必须的bean类和库 封装到 Java EE RAR 部署单元中。

RAR 非常适合不需要http 入口点，仅包含消息端点和 计划作业的 app ctx。

查看 SpringContextResourceAdapter 的注释来获得 有关RAR部署中涉及的配置详细信息。   
将AppCtx 简单部署为 Java EE RAR文件的步骤：

	1. 将所有应用类打包成一个 RAR文件 (这是一个具有不同扩展名的 标准jar 文件)
	1. 将所有必须的库 添加到 RAR 存档的 根目录中
	1. 添加 META/INF/ra.xml 部署描述，和spring xml bean定义文件 (通常是 META-INF/applicationContext.xml)
	1. 将生成的RAR文件放到应用服务器的部署目录中。













---

1.16 BeanFactory

BeanFactory API 为 Spring 的IoC 功能 提供了基础。它的具体契约主要用于 与spring 的其他部分 和相关第三方框架的集成，它的 DefaultListableBeanFactory 实现是更高级别的 GenericApplicationContext 容器中的 关键委托。

BeanFactory 和相关接口( 如BeanFactoryAware，InitializingBean，DisposableBean) 是其他框架组件的重要集成点。通过不需要任何注释甚至反射，它们允许容器与其组件之间非常有效地交互。应用程序级别的bean可以使用相同的回调接口，但通常更喜欢声明性 依赖注入，或通过注释或通过编程配置。

注意，核心BeanFactory API级别机器DefaultListableBeanFactory 实现不对配置格式或 要使用的任何组件注解作出接设。所有这些风格都通过扩展(例如XmlBeanDefinitionReader,AutowiredAnnotationBeanPostProcessor) 出现，并在共享BeanDefinition 对象上作为 核心元数据 进行操作。这就是 spring容器 灵活和可扩展的本质。



---

1.16.1 BeanFactory or ApplicationContext ?

本节说明BeanFactory 和ApplicationContext 容器级别之间的差异以及对 bootstrap 的影响。

除非有充分理由不这样做，否则应该使用ApplicationContext，将GenericApplicationContext 及其子类AnnotationConfigApplicationContext 作为自定义bootstrap 的常规实现。   
这些是spring核心容器的主要入口点， 用于常见目的：加载配置文件，触发classpath扫描，编程方式注册bean定义 和 被注解的类，(从5.0开始的)注册functional bean 定义。

由于AppCtx 包含BeanFactory的 所有功能，通常建议使用 BeanFactory，除非需要控制bean 处理。在AppCtx( 比如GenericAppCtx 实现) 中，按照约定(by-name / by-type -- 特别是 post-processor ) 检测 几种bean，而普通 DefaultListableBeanFactory 对任何特殊bean 是不可知的。

对于许多扩展容器特性，例如 注解处理和AOP代理，BeanPostProcessor 是必不可少的。如果你仅使用DefaultListableBeanFactory，则默认情况下不会检测和激活 post-processor。这种情况下，需要额外的设置来完全引导容器。

下面是 BeanFactory 和 AppCtx 的功能列表：

| feature                             | BeanFactory | AppCtx |
| ----------------------------------- | ----------- | ------ |
| Bean instantiation/wiring           | T           | T      |
| Integrated lifecycle management     | F           | T      |
| Auto BeanPostProcessor registration | F           | T      |
| Auto BeanFPP registration           | F           | T      |
| 方便访问MessageSrouce( for 国际化)  | F           | T      |
| 内置 AppEvent 发布机制              | F           | T      |



要显式注册 bean post-processor 到 DefaultListableBeanFactory，你需要编程方式调用 addBeanPostProcessor：

```java
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
// populate the factory with bean definitions

// now register any needed BeanPostProcessor instances
factory.addBeanPostProcessor(new AutowiredAnnotationBeanPostProcessor());
factory.addBeanPostProcessor(new MyBeanPostProcessor());

// now start using the factory
```

要应用BFPP 到 DefaultListableBF，需要调用 postProcessBeanFactory:

```java
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);
reader.loadBeanDefinitions(new FileSystemResource("beans.xml"));

// bring in some property values from a Properties file
PropertySourcesPlaceholderConfigurer cfg = new PropertySourcesPlaceholderConfigurer();
cfg.setLocation(new FileSystemResource("jdbc.properties"));

// now actually do the replacement
cfg.postProcessBeanFactory(factory);
```

在上面的2个例子中，显式注册步骤 不方便，这就是为什么 基于spring的应用 使用AppCtx 而不是 DefaultListableBF 的原因，特别是在典型企业设置中依赖 BFPP 和 BPP 来扩展容器功能时。

AnnotationConfigAppCtx 注册了所有常见的 post-processor，且可以通过注解(如 @EnableTransactionManagement) 来引入额外的 处理器。在spring的基于注解的配置模型的抽象级别上，bean后处理器的概念变成了单纯的内部容器细节。











