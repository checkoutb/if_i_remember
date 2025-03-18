Spring-doc
2021年11月8日
11:19

<idref>
<property>
<constructor-arg>
<null/>
<list><set><map><props>
depends-on

bean  autowire-condidate
bean  primary

Method Injection,用于  单例A  依赖  非单例B。之前是  实现ApplicationContextAware，然后  每次A需要的时候就  getBean(B)，这导致  耦合

lookup-method
replaced-method

bean  scope, 6ge，4个是web-aware applicationcontext的，singleton,prototype,request,session,application,websocket

通常，你应该对所有 有状态的bean 使用 prototype， 对 无状态的bean 使用 singleton

原型  不会调用配置的 销毁生命周期回调。客户端必须清理 原型对象 并释放原型对象的 资源..要让容器释放 prototype bean 持有的资源，请尝试使用 post-processor，这个包含 对需要清理的bean的 引用。

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

========================================

> 基于XML的 配置元信息 配置这些 bean 作为 <bean/> 元素，在 顶层的 <beans/> 中。
>
> Java配置 通常是 @Bean 注解到 方法上，在 @Configuration 类中。

========================================

IoC 也被称为 dependency injection (DI)

It is a process whereby objects define their dependencies (that is, the other objects they work with) only through constructor arguments, arguments to a factory method, or properties that are set on the object instance after it is constructed or returned from a factory method.

这是一个过程，对象 只 通过 构造器参数，工厂方法的参数，或 在构造或从工厂返回 后的实例 上设置 属性 来 定义依赖。
。。排除了 直接 new。或者说 硬编码的 依赖关系。

========================================

DI 是一个过程，对象仅通过 构造器，工厂方法的参数，对构造后或工厂返回的对象进行属性设置 来定义他们的依赖。

========================================

<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>

services.xml 必须在 同一个 目录下。
另外2个 必须在 本xml的所在目录/resources 中， 开头的/ 被忽视。

引用 上一级目录中的xml，使用 ../xxx.xml

可以，但不推荐。引用 上一级目录中的xml，使用 ../xxx.xml
特别的，不推荐 使用在 classpath中 (如 classpath:../services.xml)

全名： [file:C:/config/services.xml](file:///C:/config/services.xml)  或 classpath:/config/service.xml。

通常 最好 为这种 绝对位置 保留 间接性，如 通过在运行时 ${...} 解析 占位符

。。classpath  /  开头不会被忽视？

========================================

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

========================================

// create and configure beans

ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();

========================================

读取bean  配置

// create and configure beans

ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();

使用groovy配置，bootstrapping 看起来很类似。 它有一个不同的 上下文实现类，它支持groovy (但是也能理解 xml)

ApplicationContext context = new GenericGroovyApplicationContext("services.groovy", "daos.groovy");

最灵活的是  GenericApplicationContext 结合  reader委托
GenericApplicationContext context = new GenericApplicationContext();

new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");

context.refresh();

GenericApplicationContext context = new GenericApplicationContext();

new GroovyBeanDefinitionReader(context).loadBeanDefinitions("services.groovy", "daos.groovy");

context.refresh();

一个ApplicationContext中，reader委托可以  混用

========================================

在容器内，bean定义 被表现为 BeanDefinition 对象。

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

========================================

除了包含有关如何创建 bean的 bean定义之外，ApplicationContext 还允许注册 在 容器外部(由用户) 创建的现有对象。
这是通过 getBeanFactory() 方法 获得 ApplicationContext的 BeanFactory 来实现的，

这个方法会返回 DefaultListableBeanFactory 对象。这个对象 支持 注册，通过 registerSingleton() 和 registerBeanDefinition()

========================================

bean元数据 和 手动的单例实例 需要尽早注册，以便容器在自动装配和其他自省步骤中能正确 推导它们。

虽然 在某种程度上支持 覆盖现有 元数据和 现有单例实例，但官方不支持 在运行时注册新bean(通过对工厂进行实时访问)，这可能导致 并发访问一场，bean容器中状态不一致。

========================================

基于xml的 配置元信息，使用 ***id*** 属性， ***name*** 属性 来指明 ID。

id 属性可以让你 准确指定一个 id。 可以是 字母数字，也可以包含 特殊字符。

如果要引入别名，可以在 name 属性中指定，使用 逗号，分号，空格 分隔。

========================================

在classpath中进行 组件扫描时， spring 为 无名组件生成 名字，规则：使用 类名，并且首字母小写。
在特殊情况下，当第一第二个都是大写时，会保留 原始大小写。
这些规则 被定义在 java.beans.Introspector.decapitalize

========================================

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

========================================

bean定义，本质上是 创建一个或多个 对象的方法。
当被询问时， 容器会查看 这个bean名字 的 配方，用这个bean定义来 创建(或获取) 实际对象。

========================================

xml配置，指定 对象的 要被实例化的 类型(或类) 通过 <bean /> 的 class属性。class属性( 对应BeanDefinition 的 Class 属性(。。beanClass))通常是 强制的。

你可以通过下面2种方式之一使用 Class 属性：
1. 通常，在容器本身通过反射调用 构造器直接创建bean的情况下 指定要构造的bean类，这在某种程度上相当于 new 操作。

2. 用于指定 类，这个类包含了 static 工厂方法 用于创建对象，在不太常见的情况下，容器调用类上的 静态工厂来 创建bean。静态工厂返回的 对象类型 可能是同一个类，也可能是另一个完全不同的类。

========================================

如果你要为 内部类 配置 bean 定义， 你可以使用 内部类的 binary name 或 source name。

例如，你有一个类 com.example.SomeThing。这个类里有一个 静态内部类：OtherThing，它们通过$或. 来分隔。 此时，class属性的值可以是 com.example.SomeThing$OtherThing 或 com.example.SomeThing.OtherThing。

========================================

通过构造器创建bean时，所有普通类都可以被spring兼容，即，不需要实现额外的接口或 以特定方式进行编码。 只需要指定bean类就足够了。

但是，根据这个bean 的IoC类型，你可以需要一个 默认(空)构造器。

<bean id="exampleBean" class="examples.ExampleBean"/>
<bean name="anotherExample" class="examples.ExampleBeanTwo"/>

========================================

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

========================================

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
。。。bean的 class 是通过 实例工厂这个bean 的名字 来解析的。。 所以 工厂 必须返回 它自己的实例？？？
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

========================================

决定bean的 运行时类型

bean的运行时类型 的决定 不容易。

bean 元数据 定义 中指定的 class 只是一个 最初的class引用， 可能与 声明的工程方法 或 可能导致bean类型改变的 FactoryBean 类 结合使用，或者在 实例工厂方法 中 根本没有设置 (而是通过 指定的 工厂bean名称解析)

。。。看起来， 工厂创建的bean定义中 class 是可选的。。

此外，aop代理 可以用 基于接口的 代理 包装一个bean 实例，并 限制目标bean的 实际类型(仅其实现的接口) 的暴露

查找特定bean 的实际运行时 类型的 推荐方法是 BeanFactory.getType(bean名字)。

。。 ApplicationContext 继承了 ListableBeanFactory

========================================

========================================

DI 是一个过程，对象仅通过 构造器，工厂方法的参数，对构造后或工厂返回的对象进行属性设置 来定义他们的依赖。

容器在它创建bean时，注入这些依赖。

使用DI，代码更干净，当对象具有依赖关系时，解耦更有效。对象不查找其依赖项，也不知道依赖性的位置或类别。

测试更简单，允许mock 实现

2种DI主要变种：基于构造器的DI，基于setter的DI

========================================

ThingTwo，ThingThree  没有继承关系，下面可以用

<beans>
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg ref="beanTwo"/>
        <constructor-arg ref="beanThree"/>
    </bean>

    <bean id="beanTwo" class="x.y.ThingTwo"/>

    <bean id="beanThree" class="x.y.ThingThree"/>
</beans>

========================================

简单类型，如<value>true</value>，spring无法进行类型匹配，因为不知道类型

显示指定类型

<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>

========================================
通过index  指定  参数下标

<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>

base-0

用于构造器参数类型  相同。

========================================

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
。。。变成arg0，arg1  。。。了。。

========================================

========================================

容器调用setter方法，在无参构造器或无参static工厂返回实例 之后。

ApplicationContext 支持 构造器和 setter DI。
也支持 在已经通过构造器注入一些依赖后，再通过setter DI

你以BeanDefinition 的形式配置 依赖项，你可以将 其与PropertyEditor 实例相结合，以将属性从一种格式转换为另一种格式。

但是，大多数spring 用户，不会直接使用这些类，而是使用xml bean定义，注解的组件(@Component,@Controller等) 或 @Bean的方法(@Configuration类中)。

这些源 被转换为 BeanDefinition 实例，并用于 加载整个IoC容器实例。

========================================

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

========================================

========================================

容器执行 bean依赖解决 就像下面：
1. ApplicationContext 使用 描述所有bean 的 配置元数据 创建和初始化的。配置元数据可以通过xml，java代码，注解 指定。
2. 对于每个bean，它的依赖关系 以 属性，构造器参数，静态工厂方法参数 来表示。这些依赖项 在创建bean时 提供给bean。
3. 每个属性 和构造器参数，要 字面量作为值 设置，或 容器中另一个bean 的ref

4. 作为字面量值的 属性或构造器参数 都要进行 格式转换，从 指定格式 转换 参数的实际类型(。。应该是指 string转为 type定义的类型)。spring可以将 字符串格式 转换为 所有的内置类型。

========================================

容器校验每个bean的配置，在容器创建时。但是bean属性 不会被设置，直到bean被创建。

容器创建时，会创建 单例范围(singleton-scoped) 并设置为 pre-instantiated(默认) 的bean。其他的bean 在被request时创建(..lazy)。

依赖项之间的解析不匹配可能出现的比较晚，即在第一次创建受影响的bean时。

========================================

循环依赖
类A 通过构造器注入 类B的实例，类B通过构造器注入需要 类A的实例。
ioc容器 运行时 发现循环引用，抛出异常：BeanCurrentlyInCreationException

一种可能的方法是，修改一些类的源码，通过setter注入，而不是构造器注入。或者避免构造器注入，并仅使用setter注入。即，虽然不推荐，但是可以通过setter注入来配置 循环依赖。

========================================

容器在加载的时候 校验 配置问题，如引用到不存在的类，循环依赖等。

这意味着 已经 正确加载的容器 可能稍后，在你request 对象的时候 抛出异常，如果在创建对象或它的依赖时发生问题，如 bean在缺少属性或 非法属性时 抛出一场。

这种配置问题的 潜在延迟，是 ApplicationContext 预实例化单例bean 的原因。

========================================

如果不存在循环依赖，则当 协作bean 被注入到 依赖的bean中时， 协作bean 已经完全配置了。协作bean 已经 被实例化，被设置依赖关系，并调用相关的生命周期方法(如配置的init或InitializingBean 回调方法)

========================================

========================================

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

static 工厂方法的 参数 通过 <constructor-arg /> 提供， 和 构造器一样。
返回的类 不是必须和 包含static工厂方法的 类一致(这个例子里，是一致的)
实例工厂 使用相同的方式,除了使用factory-bean，而不是class属性

========================================

========================================

字面值(基本类型，string等)
<property /> 标签的 value属性 将属性或构造器参数 指定为 人类可读的字符串表示。
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

========================================

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

容器转换 <value /> 中的 text 到 Properties实例，通过JavaBeans PropertyEditor机制。

========================================

========================================

idref元素
idref 只是将容器中 另一个bean的id(字符串值，不是引用) 传递给 <constructor-arg> 或 <property> 的一种防错方法。

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

========================================

========================================

ref是 <constructor-arg> <property> 中的 final元素。你将 bean的属性 设置为另一个bean的 引用。

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

========================================

========================================

在<property />,<constructor-arg /> 中的 <bean> 就是 inner bean：
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

========================================

========================================

<list><set><map><props> 对应了Java的 List Set Map Properties

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

========================================

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

========================================

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

spring的类型转换机制 识别到 Map<String, Float>，所以把 value的string 转为了Float。

========================================

========================================

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

========================================

========================================

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

========================================

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

========================================

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

<!-- c-namespace index declaration -->
<bean id="beanOne" class="x.y.ThingOne" c:_0-ref="beanTwo" c:_1-ref="beanThree"
    c:_2="something@somewhere.com"/>

========================================

========================================

```xml
<bean id="something" class="things.ThingOne">
    <property name="fred.bob.sammy" value="123" />
</bean>
```
property 可以 使用 复合或嵌套的属性名称， 只要路径的所有组件除了最后一个外 不可为空。

something有一个 fred 属性，这个属性有 bob属性，bob有sammy属性。 sammy被设置为123。。 fred 和 bob 必须非null，否则NullPointerException

========================================

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

========================================

========================================

默认下，ApplicationContext 的实现 急加载和配置 所有 单例bean 作为初始化的一部分。
通常，这是可取的，因为这样 使得错误在 启动时抛出，而不是 几小时/几天 后抛出。

你可以使用 lazy-init 来 告知容器，在第一次被request时 创建bean，而不是在启动时。

```xml
<bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.something.AnotherBean"/>
```

如果lazy的bean 是 非lazy的单例bean的 依赖，那么 lazy bean 也会在 启动时创建。

========================================

全部默认 lazy：
```xml
<beans  default-lazy-init="true">
    <!-- no beans will be pre-instantiated... -->
</beans>
```

========================================

========================================

如果项目中都始终使用自动装配，那么它的效果最好。
如果一般不使用自动装配，开发者可能会混淆 使用它来 只装配一个或2个bean定义。

限制和缺点：
1. 属性和 构造器参数 中的 显式依赖 始终覆盖 自动装配。 你不能装配简单属性，如基本类型，string，Class类。 这个限制是设计使然
2. 自动装配不如 显式装配精确。spring管理的对象之间的关系 不再 明确记录。
3. 容器生成文档的工具 可能无法获得 装配信息。

4. 容器内的 多个bean定义 可能满足 要自动装配的 setter 或 构造器。对于 数组，集合，Map，这不是一个问题。但是，对于期望单个值的依赖项，这种歧义不会被解决。如果没有唯一的bean定义可用，会引发异常。

在后续场景中，你有多种选择：
1. 放弃自动装配，使用显示装配
2. 通过 设置bean定义的 autowire-condidate为false 来避免自动装配
3. 将bean的 primary属性设置为 true 来指定为 主要候选者
4. 使用注解 实现更细粒度的控制。

========================================

========================================

设置 bean标签的 autowire-candidate 属性 为false， 容器 使得这个bean 定义 不能用于 自动装配(包括 @Autowired)。

autowire-candidate 只 影响 基于类型的 自动装配。 不会影响 按名称的显示引用，即使指定的bean 没有被标记为 自动装配候选者，也会被解析。

你可以对 bean名字 进行模式匹配 来 限制 自动装配候选者。

顶层beans 标签 的 default-autowire-candidates 属性 接收 一个或多个pattern。 例如，为了限制 名字以Repository结尾的bean 的 自动装配候选者，提供一个 *Repository 值作为pattern。

多个pattern，逗号分隔。
bean定义中的 autowire-candidate 优先。

========================================

在大多数的应用场景中，容器中的bean 大多是单例的。

当一个 单例bean 要与另一个单例bean 协作，或 非单例bean 要与另一个非单例bean 协作，你通常通过 将一个bean 定义为另一个bean的属性 来处理依赖关系。

当bean 生命周期不同时，就会出现问题。
假设 单例bean A 在方法中 调用了 非单例bean B。 容器只创建一次单例A，因此只有一次机会设置属性。容器无法在每次需要时，为A提供一个新的B。

一种方法是放弃IoC。 你可以使得 A  知道(aware of)  容器，通过 实现 ApplicationContextAware  接口，然后在需要的时候 getBean("B")。

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

上面并不可取，因为业务代码 知道并耦合到 Spring框架。
方法注入是 IoC的一项高级功能，可以让你干净地处理此用例。
Method  Injection

========================================

查找方法注入 是容器的能力，用于 重写容器管理的bean的方法，并返回 容器中另一个bean的查找结果。
lookup通常设计 一个 prototype bean，就像上节描述的场景。
spring通过 cglib 库中的 字节码生成器 来动态生成 一个 重写了方法的 子类 来实现方法注入。

为了让 动态子类 起效，容器中 被子类化的类 不能是final类。被重写的方法不能是 final方法。
对具有抽象方法的类进行 单元测试 需要你自己 子类化 该类 并提供抽象方法的 存根实现。
组件扫描也需要具体的方法，这需要具体类来拾取。
一个关键限制是，查找方法不适用于工厂方法，特别不适用于 配置类中的 @Bean方法，因为在这种情况下，容器不负责创建实例，因此无法创建运行时生成的即时子类。

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

public abstract class CommandManager {

    public Object process(Object commandState) {
        Command command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup("myCommand")
    protected abstract Command createCommand();
}

public abstract class CommandManager {

    public Object process(Object commandState) {
        Command command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup
    protected abstract Command createCommand();
}
根据方法返回参数  解析bean

========================================

========================================

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

========================================

========================================

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

========================================

单例bean 只有 一个被共享的实例 被管理，容器收到的 所有 和这个bean 匹配(id或 name)的请求，都会返回这个实例。

spring的 单例bean 概念 和 GoF的设计模式中的单例模式不同。
GoF 单例是 硬编码 对象的scope，以便每个 ClassLoader 只能创建特定类的 一个 实例。
Spring 单例的scope 最好描述为 每个容器 和 每个bean

========================================

bean的 prototype scope 导致 每次对 bean的请求 都会 创建一个新实例。

通常，你应该对所有 有状态的bean 使用 prototype， 对 无状态的bean 使用 singleton

========================================

和其他范围相比，spring不管理 prototype bean的 完整生命周期。
容器 实例化，配置，组装 prototype 对象 并传递给 客户端，并不会进一步记录该原型实例。
因此，尽管 所有对象都会被调用 初始化生命周期 回调方法，但是原型，不会调用配置的 销毁生命周期回调。客户端必须清理 原型对象 并释放原型对象的 资源

要让容器释放 prototype bean 持有的资源，请尝试使用  post-processor，这个包含 对需要清理的bean的 引用。

========================================

当你使用 单例依赖 原型，请注意 依赖关系 是在实例化时解决。
因此，如果你将 原型 注入到 单例中， 会实例化一个 原型，然后注入到 单例中， 单例中的 原型 是唯一的。

但是，假设你希望 单例bean 在运行时 重复获取 原型bean的新实例。 你不能将 原型注入到 单例中，因为这个注入 仅 在 容器实例化单例时 发生一次。

如果需要多次原型bean的 新实例，参考 Method Injection

========================================

========================================

为了支持这4个scope(request,session,application,websocket)，在定义bean之前 需要一些小的初始化配置。

如何配置取决于你的 Servlet 环境。

如果你要在Spring MVC中 访问这种bean，实际上，在DispatcherServlet处理的请求中，不需要特殊设置。DispatcherServlet 已经公开了 所有相关状态。

如果你使用Servlet 2.5 web 容器，不使用DispatcherServlet 处理请求 (如使用JSF  或 Structs 时)，你需要注册spring的 RequestContextListener ServletRequestListener。

对于Servlet 3.0+，这个可以通过使用 WebApplicationInitializer 接口来 编程解决。
对于更早的容器，增加下面到 web.xml
<web-app>
    ...
    <listener>
        <listener-class>
            org.springframework.web.context.request.RequestContextListener
        </listener-class>
    </listener>
    ...
</web-app>

或者，如果你的listener 设置存在问题，考虑RequestContextListener。
这个过滤器映射 取居于周围的web应用配置，所以你可以 根据需要修改。

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

DispatcherServlet, RequestContextLisener,RequestContextFilter 都做了 同一件事情，将HTTP request 对象 绑定到 为该请求提供服务的 线程。

这使得 bean的 request，session scope 在 调用链的下游 可用。

========================================

容器使用 loginAction 的bean定义 为每个 http request 新建一个 LoginAction 实例。这样，loginAction bean 的 scope 就是 HTTP request 层级。

<bean id="loginAction" class="com.something.LoginAction" scope="request"/>

@RequestScope
@Component
public class LoginAction {
    // ...
}

========================================

容器 为每个 http session 创建一个 UserPreference bean.

<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>

@SessionScope
@Component
public class UserPreferences {
    // ...
}

========================================

为整个 web application 创建一个 实例。即appPreference 是ServletContext 层级的，保存为 ServletContext 的常规属性。

类似单例，但有2个重要的区别： 每个ServletContext 一个单例，而不是每个Applicationontext； 它通过 ServletContext 的属性来暴露。

<bean id="appPreferences" class="com.something.AppPreferences" scope="application"/>

@ApplicationScope
@Component
public class AppPreferences {
    // ...
}

========================================

WebSocket Scope

这个scope 和 WebSocket session的 生命周期相关联，适用于 基于WebSocket 的 STOMP 应用

========================================

========================================

如果你想 注入一个 http request scope 的 bean 到 另一个 长期存活的bean， 你可以选择 注入一个 AOP代理 而不是 scoped bean。

即，你需要注入一个 扩展了 和 scoped bean相同 公共接口，但也可以从相关scope(如request scope) 检索真实目标对象，并将方法调用委托给真实对象 的代理对象。

========================================

你也可以使用```<aop:scoped-proxy>``` 在单例bean 之间，引用(名词)通过可序列化的中间代理，因此能够在反序列化时重新获得单例bean。

当 对prototype bean 声明 aop:scoped-proxy 时，在共享代理上的 每次方法调用 都会导致 创建一个新的目标实例，然后将调用转发到该实例。

此外，scoped proxy 不是 唯一方式： 以生命周期安全的方式 从较短的作用域 访问bean。

你也可以 声明自己的注入点(构造器或setter或自动装配字段) 为 ObjectFactory<MyTargetBean>，这个接口的 getObject方法 调用 在每次需要时 按需检索 当前实例，不需要hold 实例或 单独存储实例。

作为扩展变体，你可以声明ObjectProvider<MyTargetBean>，提供了几个额外的访问，包括 getIfAvailable,getIfUnique.

JSR-330 变体 称为 Provider，与 Provider<MyTargetBean> 声明 和 每次检索尝试的 相应的 get() 一起使用。

========================================

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
        <aop:scoped-proxy/>
    </bean>

    <!-- a singleton-scoped bean injected with a proxy to the above bean -->
    <bean id="userService" class="com.something.SimpleUserService">
        <!-- a reference to the proxied userPreferences bean -->
        <property name="userPreferences" ref="userPreferences"/>
    </bean>
</beans>

为了创建这样的代理，将 aop:scoped-proxy 插入到 scoped bean 定义。

为什么在 request，session，自定义scope 中需要 aop:scoped-proxy ？
考虑下面的 单例 bean，并和 你需要为上述之前的scope 的定义的内容进行对比。
```xml

<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>

<bean id="userManager" class="com.something.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

这里的重点是 userManager 是 单例的： 每个容器只实例化一次，并且它的依赖项 也只注入了一次。 这意味着 userManager bean 只对完全相同的 userPerferences(即最初注入的对象) 进行操作

========================================

默认，容器创建 一个 代理 for 标记为 aop:scoped-proxy 的bean， 基于cglib的 代理 被创建。

cglib 代理 只拦截  public  的方法的调用。 不要调用 这种proxy的 非public方法，它们不会被委托到真正的 scoped 对象。

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

========================================

========================================

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

========================================

在你写且测试 自定义Scope实现后，你需要让 spring 容器 感知到你的 scope。下面的方法是注册 新的scope到 spring 容器：
```java
void registerScope(String scopeName, Scope scope);
```

这个方法在 ConfigurableBeanFactory 接口中，可以通过 ApplicationContext实现的 BeanFactory属性 来获得。

方法的第一个参数是 scope 的 唯一名字。比如spring的 singleton和 prototype。
第二个参数是 你想注册和使用的 自定义Scope 实现的 实例。

使用自定义 Scope实现，你不限于 编程方式注册scope。你也可以 声明式进行Scope 注册，通过使用 CustomScopeConfigurer：

    <bean class="org.springframework.beans.factory.config.CustomScopeConfigurer">

        <property name="scopes">
            <map>
                <entry key="thread">

                    <bean class="org.springframework.context.support.SimpleThreadScope"/>

                </entry>
            </map>
        </property>
    </bean>

========================================

当你在 实现FactoryBean接口的 bean 声明中 放置 aop:scoped-proxy， scope 的是 factory bean，而不是 从 getObject 返回的 对象。

========================================

========================================

spring 提供了一些接口，你可以用来 定制bean的 性质。
Lifecycle Callbacks
ApplicationContextAware and BeanNameAware
其他 Aware 接口。

---
Lifecycle Callbacks

为了和 容器中bean的 生命周期的 管理进行交互，你可以实现 InitializingBean，DisposableBean 接口。

容器为InitializingBean 调用 afterPropertiesSet，为DisposableBean 调用 destroy， 以让bean在 初始化和 销毁bean时 执行某些操作。

JSR-250 @PostConstruct 和 @PreDestory 注解 通常被认为是 现代spring 应用中欧给你 接收 生命周期回调的  最佳实践。 使用这些注解意味着你不会耦合到spring 的接口。

如果你不想使用JSR-250 注释，仍想移除耦合，考虑 bean标签的 init-method, destroy-method。

========================================

在内部，spring使用 BeanPostProcessor 实现 来处理它可以找到的 任何回调接口并调用适当的方法。

如果你需要自定义特性或其他生命周期行为，你可以实现 BeanPostProcessor.

对于 初始化和销毁 毁掉，spring管理的对象 也可以实现 Lifecycle 接口，这样这些对象能 参与由容器自身生命周期 驱动的 启动和关闭 过程。

========================================

spring的 InitializingBean 接口使得 bean 执行 初始化工作 在容器 为bean设置所有必要的 属性后。
接口只有一个方法：```void afterPropertiesSet() throws Exception;```

我们建议你 不 要 使用这个接口，它会使得 代码耦合到spring。

我们建议 @PostConstruct，或指定 POJO的 初始化方法。 在xml中，可以使用 init-method 来指明一个无参方法。 Java配置，你可以使用 @Bean 的 initMethod 方法。

========================================

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

========================================

<beans default-init-method="init">

beans 标签的 default-init-method 属性 导致 IoC 容器 识别出 bean 类中的 init方法 来作为 初始化方法回调。
当创建和组装bean时， 如果bean 类有这样的方法，会在合适的时候 调用。

beans 也可以设置 default-destroy-method.

bean 里的 init-method, destroy-method 会覆盖 beans 的default。

容器保证在 为bean 提供所有依赖后 立刻调用 初始化回调。
且，在原始bean 引用上 调用初始化回调，这意味着 AOP 拦截器尚未应用于bean。
先创造一个 完整的bean，然后 应用 aop代理及其拦截器链。 如果bean和代理 分开定义的，你的代码甚至可以绕过代理 与 原始bean 交互。
因此，将拦截器应用于 init 方法将是不一致的，这样做会将bean 的 生命周期与其代理或拦截器 耦合，并在你的代码直接与原始bean交互时留下奇怪的语义。

========================================

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

========================================

spring 管理的 对象可以实现 Lifecycle 接口。

当ApplicationContext 收到 开始 和 结束 标记 (例如，一个 运行时的 停止/重启 场景)，它将这些调用 级联到通过该context 中定义的所有 Lifecycle 实现。

LifecycleProcessor 继承了 Lifecycle。 增加了2个其他的方法 来对 refresh 和 close 的context 来做出反应。

常规Lifecycle 是 显式启动和停止通知的普通合同，并不意味着在context 刷新时自动启动。
要对特定bean的 自动启动(包括启动阶段) 进行细粒度控制，请考虑 使用 SmartLifecycle.

stop通知不能保证在 销毁之前发出。在常规的关闭时，所有的 Lifecycle bean 首先收到 停止通知 在 传播 销毁毁掉之前。 但是，在context的生命周期内 进行 hot 刷新 或 停止刷新尝试时， 只会调用销毁方法。

========================================

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

========================================

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

========================================

========================================

当 ApplicationContext 创建 实现了ApplicationContextAware 接口的 bean 的实例时，这个实例 会被提供一个 ApplicationContext的 ref。

这个bean 能编程式操作 创造它的 ApplicationContext，通过 ApplicationContext接口 或 强转为一个子类。

一种用途是 编程时 检索其他bean。 有时是有用的，但应该避免实现这个接口，因为会耦合，并且没有依照 IoC style(协作者应该作为属性提供给bean)。

ApplicationContext的其他方法提供 对文件资源的 访问，发布应用event，访问MessageSource。

为了更加灵活，包括自动装配字段和多参数方法的能力，请使用 基于注解的 装配功能。

如果你这样做，ApplicationContext 会被自动装配到 暴露了ApplicationContext类型的 字段，构造器参数，方法参数，如果这些带有 @Autowired 注解。

========================================

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

========================================

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

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

========================================

已使用 Microsoft OneNote 2016 创建。