

https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop

这里会包含 第五(使用spring面向切面编程) 第六章(AOP-API) 



#### 5. Aspect Oriented Programming with Spring

aspect-oriented programming (AOP) 补充 Object-oriented Programming (OOP) by 提供另一种思考程序结构的方式。OOP中模块化的关键单元是 class，而AOP中模块化的单元是 aspect。   
切面支持跨多种类型和对象的关注点(例如 事物管理) 的模块化。 ( 这种关注点在AOP 文献中通常被称为 "crosscutting 横切" 关注点。)

spring的关键框架之一是AOP框架。虽然spring IoC容器 不依赖 AOP，AOP 补充了IoC 以提供非常强大的中间件解决方案。

spring 通常使用基于schema的方式 或 @AspectJ 注解的方式 提供了编写自定义切面的 简单且强大的方法。这2种方法都提供了完全类型化的advice和使用AspectJ pointcut 语言，同时仍然使用 spring aop 进行编织。

本章讨论基于模式和@AspectJ 的 AOP 支持。较低级别的AOP支持在下一章中讨论。

spring框架使用AOP 来：

1. 提供声明式企业服务。最重要的此类服务是 声明式 事物管理。
2. 让用户实现自定义切面，用AOP补充他们对OOP的使用。

如果你只对通用声明式服务 或 其他预打包的声明式中间件服务( 如 池) 感兴趣，则无需直接使用Spring AOP，并且可以跳过本章的大部分内容。



---

5.1 AOP Concepts 概念

先定义一些核心AOP 概念和术语。这些术语不是spring特定的。不幸的是，AOP术语并不是特别直观。

Aspect：跨多个类的关注点的模块化，事物管理是企业java 应用中横切关注点的一个 很好的例子。在spring AOP中，切面是通过使用常规类(基于schema) 或 @AspectJ 注解的常规类 来实现的。

Join point：程序执行过程中的一个点，例如，方法的执行或异常的处理。在spring AOP 中，一个连接点总是代表一个方法执行。

Advice：切面在特定连接点采取的操作。advice的类型包括 around，before，after 等。包括spring 在内的许多AOP框架将advice 建模为拦截器，并在连接点周围维护一个 拦截器链。

Pointcut：匹配连接点的谓词，advice 和 切入点表达式 相关联，并在 与切入点匹配的 任何 连接点处 运行。切入点表达式匹配的 连接点是 AOP的核心，spring 默认使用 AspectJ pointcut expression language。

Introduction：在类型的 behalf(代表/利益) 上声明额外的方法或属性。AOP允许你向任何 advised 对象 引入新的接口( 和相应的实现)。例如，你可以使用 引入 来使 bean 实现 IsModified 接口，来简化缓存。(introduction 在 AspectJ 社区中 被称为 类型间声明)

Traget object：被一个或多个切面 advice的对象。也称为"advised object"。由于AOP 是使用 运行时代理实现的，所以该对象始终是代理对象。

AOP proxy：AOP框架创建的一个对象，用于实现 切面 协定 ( advise 方法等)。在spring中，AOP 代理是 jdk动态代理 或 cglib代理。

Weaving：将切面与其他应用类型 或对象 链接 以创建 advised 对象。这个可以在 编译时( 例如，使用AspectJ编译器)，加载时，或运行时 完成。Spring AOP和其他 纯java AOP 一样，在运行时进行weaving。



Spring AOP 包含下面的 advice 类型：

1. before advice: 在连接点之前运行但不能阻止 后续连接点执行 ( 除非它抛出异常)
2. after returning advice: 在连接点正常完成后执行
3. after throwing advice: 在方法 通过 抛出异常的方式 exit 后 执行
4. after (finally) advice: 在连接点退出后执行，不管是正常退出 还是 异常退出
5. around advice: 围绕连接点的advice，这是最有力的 advice。around advice 可以在方法调用 之前 和之后 执行自定义行为。它还负责 选择 是继续执行连接点 还是 通过返回它自己的返回值/抛出异常 来缩短 被advice 的方法。



around advice 是最一般的建议。由于 AOP 和AspectJ 一样，提供了 全方位的 advice 类型，因此我们建议你使用 可以满足所需行为的 最不强大的 advice type。    
例如，如果你只需要 使用方法的返回值 来更新缓存，那么你最好实现一个 after returning advice，而不是 around advice。使用最具体的 advice type 提供了一个更简单的编程模型，并且出错的可能性更小。

所有 advice 参数 都是静态类型的，因此你可以使用适当类型的 advice参数，而不是Object数组。

使用 pointcut 匹配 join point 的 概念 是 AOP的关键，这将它们与仅提供拦截的 旧技术区分开来。pointcut 允许 advice 独立于 OOP。例如 你可以使用 around advice 提供 声明式 事务管理 到 一组 跨越多个object的 方法上。



---

5.2 Spring AOP 能力和目标

spring aop 是 纯java 实现的。不需要特殊的编译过程。aop 不需要控制类加载器 层次结构，因此使用与servlet 容器 或 应用服务器。

aop目前只支持 方法执行 的 join point ( advice spring bean 上方法的执行)。没有Field拦截，尽管可以在不破坏核心spring aop api 的情况下添加对 field 拦截的 支持。如果你需要 advice 属性访问 和 更新连接点，请考虑使用 AspectJ 等语言。

spring aop 的 aop 方法 不同于大多数其他aop框架。目的不是提供最完整的 aop 实现。目的是 提供aop实现和 ioc之间的 紧密集成，以帮助解决企业应用程序中常见的问题。

因此，aop通常和 ioc 容器 结合使用。切面是通过 普通bean定义语法 来配置的( 尽管这允许强大的"自动代理"功能)。这是与其他aop 实现的关键区别。你无法使用 spring aop 轻松或高效 地做一些事情，例如advice 非常细粒度的对象( 通常是，domain 对象)。这种情况下，AspectJ 是最佳选择。然而，我们的经验是spring aop 为企业java 应用程序 的大多数 aop问题 提供了出色的解决方案。

spring aop 和 AspectJ 不是竞争。。。。

spring框架的核心原则之一是 非侵入性。这就是不应该强迫你将特定于框架的类 和接口 引入你的 业务或领域模型。   
但，在某些地方，spring 确实为你提供了 将spring 特定的依赖 引入到你的代码中的 选项。提供这些选项的理由是，在某些情况下，以这种方式阅读或编写某些特定功能可能更容易。但spring总是为你提供选择：你可以自由地就哪个选项最适合你的特定用例或场景作出明智的选择。

与本章相关的一个选择是： 选择哪种AOP框架( 以及哪种AOP风格)。你可以选择AspectJ，Spring AOP 或两者兼而有之。你还可以选择 注解 或 xml 配置。本章先介绍 注解配置，并不代表spring 团队更喜欢 注解配置。



---

5.3 AOP Proxies

aop默认使用 标准jdk代理 for aop 代理。这允许 任何接口 被代理。

aop 也可以使用 CGLIB。这个是代理类 而不是 接口。默认情况下，如果业务对象 未实现任何接口，则使用cglib。由于对接口进行编程是一个 很好的做法，所以业务类通常实现了 一个或多个业务接口。如果你需要 advice 一个 不在接口中声明的 方法 或 需要将代理对象作为具体类型传递给方法 的情况下，你可以强制使用cglib。

。。。我记得 IOC 默认 cglib吧。。 不，IOC 应该不算是 代理， 里面用的 还是 aop:scoped-proxy，还是算 aop的， 不过这个默认是 cglib。

重要的掌握，spring aop 是基于代理的这一事实。



---

5.4 @AspectJ 支持

@AspectJ 代表了一种 声明切面的 风格，这种风格使用 注解。spring使用了 aspectJ 的注解，使用aspectj 提供pointcut 的转换和匹配 。aop 运行时 仍然是 纯 spring aop。不依赖 aspectJ compiler 或 weaver。



---

5.4.1 激活@AspectJ 支持

为了在spring 配置中使用 @AspectJ 切面，你需要 激活 基于@AspectJ 切面的 spring aop 配置的支持 和 基于是否被这些且面 advice 而进行 自动代理的 bean。通过 自动代理，我们意味着，如果spring决定 bean 被 advice by 一个或多个切面，它自动生成一个代理 来拦截方法 调用并确保根据需要 运行advice。

可以使用 xml 或 java风格 来启用 @AspectJ 支持。 无论哪种情况，你还需要确保 aspectJ的 aspectjweaver.jar 库 在你的app的 classpath中 (1.8或更高版本 )。



---

启用@AspectJ 支持 with java 配置

要使@Configuration 启用 @AspectJ 支持， 请添加 @EnableAspectJAutoProxy

```java
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {

}
```



---

通过xml 启用@AspectJ 支持

要使用 基于xml 的@AspectJ 配置

```
<aop:aspectj-autoproxy/>
```



---

5.4.2 声明一个Aspect

当 @AspectJ 的支持 启用后，你的 app ctx 中定义的 任何 bean，且bean 的类是一个 @AspectJ 切面，会被spring自动发现 且用于 配置 spring aop。下面的 展示了 不太有用的 切面的 最小定义：

```xml
<bean id="myAspect" class="org.xyz.NotVeryUsefulAspect">
    <!-- configure properties of the aspect here -->
</bean>
```

下面是 NotVeryUsefulAspect 的定义，被注解了@Aspect：

```java
package org.xyz;
import org.aspectj.lang.annotation.Aspect;

@Aspect
public class NotVeryUsefulAspect {

}
```

切面( 被@Aspect 注解的 类 ) 可以有 方法 和 属性，和其他类一样。 他们也可以包含 pointcut，advice，introduction 声明。

通过组件扫描自动侦测切面

你可以通过@Configuration 类的 @Bean 方法 将 切面类 注册为 spring xml 配置中的常规 bean，或者让 spring 通过 classpath 扫描 自动检测它们 - 和其他spring管理的bean 一样。   
但是，@Apsect 注解 不足以 让自动检测 发现。为此，需要一个 @Component 注解。



在spring aop中， 切面不能成为 其他切面的 advice的 目标。类上的@Aspect注解 标记它为 切面，从自动代理中排除出去。



---

5.4.3 声明切点

切点决定 连接点，使我们能够控制 advice 何时运行。spring aop 仅支持spring bean 的 方法执行的连接点，所以你可以将切点视为 对bean 上的执行的方法 进行 匹配。   
切入点声明有 2部分：一个包含名称和任何参数的签名，一个切点表达式用于准确决定哪些是我们感兴趣的方法执行。   
在aop的@AspectJ 风格中，切点签名由 常规方法定义 提供，并且 切点表达式通过@Pointcut来指明。(作为切点签名的方法必须返回 void)。

一个例子可以有助于明确切点签名 和 切点表达式 之间的区别。下面定义了一个名为anyOldTransfer 的切点，它和任何名为transfer的方法的 执行想匹配：

```java
@Pointcut("execution(* transfer(..))") // the pointcut expression
private void anyOldTransfer() {} // the pointcut signature
```

@Pointcut 的值 是一个常规的 AspectJ 切点表达式。



---

支持的切点指示符

spring aop 支持以下的 aspectj pointcut designators (PCD) 在 切点表达式中使用：

1. execution: 用于匹配方法执行 连接点。spring aop 使用的 主要切点指示符。
2. within: 限制 匹配特定类型的连接点。( 使用spring aop 时执行 匹配的类型 中的方法执行)
3. this: 限制匹配到 连接点(使用spring aop 时方法的执行)，其中bean 引用 (这个是spring aop的 代理对象) 是给定类型的实例。
4. target: 限制匹配的连接点( 使用spring aop时方法的执行)，其中目标对象(被代理的对象) 是给定类型的实例
5. args: 限制匹配的连接点，参数(argument 是实参还是形参，实参，不清楚了，这里是argument，下面的第7点，是actual argument)必须是给定类型的实例
6. @target: 限制匹配的连接点，执行对象(。。被代理对象) 有被 给定的注解 注解。
7. @args: 限制匹配的连接点，传递的实参的运行时类型具有给定的注解。
8. @within: 限制匹配的连接点 需要被给定的注解 注解。
9. @annotation: 限制匹配的连接点，连接点的subject 有 指定的注解。



其他切点类型

AspectJ 切点语言 还支持 spring不支持的 其他 切点designator：call, get, set, preinitialization, staticinitialization, initialization, handler, adviceexecution, withincode,cflow, cflowbelow, if, @this, @withincode。在spring aop 中使用这些 会导致 IllegalArgumentException。



由于spring aop 限制了 只匹配方法执行 join points，因此前面的 spring的 designator 比 ApsectJ 的更窄。   
此外，aspectj 本身具有基于类型的语义，并且在 执行连接点处，this 和 target 都指向同一个对象：执行该方法的对象。 

> 而，spring aop 是基于代理的系统，它区别 代理本身(this)，被代理的对象(target)



由于spring aop框架基于代理的特性，根据定义，目标对象内的调用不会被拦截。对于jdk代理，只能拦截代理上的公共接口方法的调用。使用cglib，代理上的 公共和 protected 方法 调用被拦截( 如果需要，甚至包含可见的方法)。 但是，通过代理的常见交互应始终通过公共前面进行设计。

。。。拦截的是 代理对象 的方法，而不是 业务类的方法。所以自己ref自己，就可以了？

注意，切点定义通常与 任何拦截 的方法进行匹配。如果切点严格来说只是public的，即使在cglib代理场景中，通过代理进行潜在的非公开交互，也需要相应地定义它。

如果你的拦截需要包括目标类中的方法调用甚至构造器，请考虑使用spring 驱动的 原生aspectj 编织，而不是spring 的基于代理的 aop 框架。



spring aop 还支持一个 额外的 pcd：bean。这个pcd允许你将 连接点限制为 特定命名的spring bean 或一组spring bean (使用通配符时)，bean PCD具有以下形式：

```bean(idOrNameOfBean)```

idOrNameOfBean 可以是 任何spring bean的 name，支持 * 通配符，所以你可以为spring bean 建立一些 命名约定。 和其他 切点designator 一样，bean pcd 可以和 &&，||，!  连用。

bean pcd 是 spring aop 的，AspectJ 不支持，所以 在 @Aspect 声明的切面 中 不能使用。

bean pcd 在 实例级别(基于spring bean的概念构建 ) 操作 而不是只在 类型级别( 基于织入的aop 被闲置)。   
基于实例的切入点designator 是 spring 基于代理的aop框架的一种特殊功能，它和 spring bean factory 紧密结合，通过 name 来识别特定的bean 是自然而直接的。



---

组合切点表达式

可以通过 &&， ||， ! 来组合 切点表达式。 你可以 通过名字 ref 切点表达式。：

```java
@Pointcut("execution(public * *(..))")
private void anyPublicOperation() {} 

@Pointcut("within(com.xyz.myapp.trading..*)")
private void inTrading() {} 

@Pointcut("anyPublicOperation() && inTrading()")
private void tradingOperation() {} 
```

anyPublicOperation 匹配 如果 方法执行join point 代表 任何public方法的执行。

inTrading 匹配，如果 方法执行 是在 trading 模块中。

tradingOperation 匹配，如果 方法执行 代表了 trading模块中的 任何 publiic 方法。

如前所述，使用较小的命名组件 构建更复杂的切点表达式 是最佳实践。当通过名字 ref 切点，要遵守java可见性规则 ( 你可以看到 同类的 private 切点，父类的protected 切点，任何类的 public 切点 )。 可见性不影响切点匹配。



---

分享通用的切点定义

企业级应用，开发者通常希望从 多个切面 引入  应用模块和特定操作集。我们建议为此目的定义一个 捕获 常见切点表达式的 名叫 CommonPointcuts 的 切面：

```java
package com.xyz.myapp;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;

@Aspect
public class CommonPointcuts {

    /**
     * A join point is in the web layer if the method is defined
     * in a type in the com.xyz.myapp.web package or any sub-package
     * under that.
     */
    @Pointcut("within(com.xyz.myapp.web..*)")
    public void inWebLayer() {}

    /**
     * A join point is in the service layer if the method is defined
     * in a type in the com.xyz.myapp.service package or any sub-package
     * under that.
     */
    @Pointcut("within(com.xyz.myapp.service..*)")
    public void inServiceLayer() {}

    /**
     * A join point is in the data access layer if the method is defined
     * in a type in the com.xyz.myapp.dao package or any sub-package
     * under that.
     */
    @Pointcut("within(com.xyz.myapp.dao..*)")
    public void inDataAccessLayer() {}

    /**
     * A business service is the execution of any method defined on a service
     * interface. This definition assumes that interfaces are placed in the
     * "service" package, and that implementation types are in sub-packages.
     *
     * If you group service interfaces by functional area (for example,
     * in packages com.xyz.myapp.abc.service and com.xyz.myapp.def.service) then
     * the pointcut expression "execution(* com.xyz.myapp..service.*.*(..))"
     * could be used instead.
     *
     * Alternatively, you can write the expression using the 'bean'
     * PCD, like so "bean(*Service)". (This assumes that you have
     * named your Spring service beans in a consistent fashion.)
     */
    @Pointcut("execution(* com.xyz.myapp..service.*.*(..))")
    public void businessService() {}

    /**
     * A data access operation is the execution of any method defined on a
     * dao interface. This definition assumes that interfaces are placed in the
     * "dao" package, and that implementation types are in sub-packages.
     */
    @Pointcut("execution(* com.xyz.myapp.dao.*.*(..))")
    public void dataAccessOperation() {}

}
```

你可以引用 定义在这个切面中的 切点，当你需要切点表达式时。例如，使service 层 事物化，你可以：

```xml
<aop:config>
    <aop:advisor
        pointcut="com.xyz.myapp.CommonPointcuts.businessService()"
        advice-ref="tx-advice"/>
</aop:config>

<tx:advice id="tx-advice">
    <tx:attributes>
        <tx:method name="*" propagation="REQUIRED"/>
    </tx:attributes>
</tx:advice>
```



---

例子

spring aop 用户喜欢 经常使用 execution 切点designator。execution 表达式：

```
execution(modifiers-pattern? ret-type-pattern declaring-type-pattern?name-pattern(param-pattern) throws-pattern?)
```

除了 返回类型pattern (上面的 ret-type-pattern )，name-pattern, param-pattern 外 都是 可选的。   
ret-type-pattern 是 join point 返回值 必须匹配的。 * 经常用作 ret-type，它匹配任何返回类型。 当返回类型匹配时，全限定类型名才匹配。   
name-pattern 匹配方法名字。你可以使用 * 作为 name-pattern 整体或一部分。如果你指定 declaring-type-pattern，那么 包含使用 . 来连接 declaring-type-pattern 和 name-pattern。   
param-pattern 复杂一些：() 匹配不带参数的方法，(..) 匹配任意数量(0或多个) 参数。(\*) 匹配 单参数(任何类型) 。(\*, String) 匹配 2个形参，第一个可以是任何类型，第二个必须是String。

下面的例子展示了一些通用的切点表达式。

任何public方法的执行：

```
execution(public * *(..))
```

任何 方法名以 set 作为 开头的方法的执行：

```
execution(* set*(..))
```

任何定义在AccountService 接口中的方法：

```
execution(* com.xyz.service.AccountService.*(..))
```

任何定义在 service 包中的方法：

```
execution(* com.xyz.service.*.*(..))
```

任何定义在 service 包 或它子包 中的方法：

```java
execution(* com.xyz.service..*.*(..))
```

service包中的 任意连接点 ( 只在spring aop 中的 方法执行)：

```
 within(com.xyz.service.*)
```

service 包或它子包 中的 任意连接点 (方法执行 只在 spring aop 中)：

```
within(com.xyz.service..*)
```

代理 实现了 AccountService 接口 的 连接点(方法执行 只在spring aop 中)：

```
this(com.xyz.service.AccountService)
```

this 更常以 bingding  形式 被使用

被代理的对象 实现了 AccountService 接口 的 任意连接点( ff执行只在aop中)：

```
target(com.xyz.service.AccountService)
```

target 更常以 bingding  形式 被使用

接受单形参，且运行时 传递的实参 是Serializable 的 任何 连接点 (ff 执行只在aop中)：

```
args(java.io.Serializable)
```

args 更常以 bingding  形式 被使用

这个例子的 切入点 和 execution(* *(java.io.Serializable)) 不同。args版本的 匹配 运行时是 Serializable，execution版本 匹配 方法签名中声明了一个 单参数的 Serializable。

target对象有 @Transactional 的 任何连接点 ( 方法执行只在aop中)：

```
@target(org.springframework.transaction.annotation.Transactional)
```

你可以用 binding 形式 来使用 @target

target对象 的声明的类型 有 @Transactional 注解 的 连接点 (ff aop):

```
@within(org.springframework.transaction.annotation.Transactional)
```

你可以用 binding 形式 来使用 @within

执行的方法有 @Transactional 注解 的 连接点(ff aop)：

```
@annotation(org.springframework.transaction.annotation.Transactional)
```

你可以用 binding 形式 来使用 @annotation

单参数，且实参的运行时类型 被 @Classified 的 连接点(ff aop)：

```
@args(com.xyz.security.Classified)
```

你可以用 binding 形式 来使用 @args

spring bean的名字是 tradeService 的 任何连接点(ff aop)：

```
bean(tradeService)
```

spring bean 的名字 匹配 *Service 的任何连接点(ff aop):

```
bean(*Service)
```



---

编写好的切入点

编译期间，AspectJ 处理切点 来 优化 匹配时的性能。检查代码并确定 每个连接点 是否 ( 静态或动态) 匹配给定的切点是一个 高代价的过程。( 动态匹配意味着无法从静态分析中完全确定匹配，并在代码中放置测试以确定代码运行时是否存在实际匹配)。   
在第一次遇到切点声明时， aspectj 将其重写为匹配过程的最佳形式。这是什么意思？基本上，切点在DNF( Disjunctive Normal Form 析取范式 ) 中被重写，切点的组件被排序，以便首先检查那些评估成本较低的组件。这意味着你不必担心 各种 切点designator的性能，并且可以在切点声明中 以任何顺序提供。

。。。切点 表达式 被重写，是表达式被重写。

然而，aspectJ只能使用它被告知的内容。为了获得最佳匹配性能，你应该考虑他们试图实现的目标，并在定义中仅可能缩小匹配的搜索空间。现有的 designator 可以很自然地分为3组：kinded，scoping，contextual：

1. kinded designator：选择特定类型的连接点：execution,get,set,call,handler
2. scoping designator：选择 一组连接点 of interest (可能有多种)：within,withincode
3. contextual designator：基于context 的匹配( 和可选地绑定)：this,target,@annotation

一个写的好的 切点 至少要包含 前2点 ( 种类 和 范围)。你可以包含 上下文 designator  以根据 连接点上下文进行匹配，或绑定 该上下文 以在 advice 中使用。   
只提供一个 kinded 或 contextual，是可以的，但是会影响性能。 scoping 的匹配速度非常快。好的切点应该 始终包含一个 scoping。



---

5.4.4 声明 Advice

advice 和 切点表达式 相关联，并在 切点匹配的 方法 before，after ，around 运行。切点表达式可以是 对 命名的切点的 引用，也可以是 原地声明的切点表达式。



---

before advice

声明 before advice 在 切面中，通过使用 @Before 注解：

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class BeforeExample {

    @Before("com.xyz.myapp.CommonPointcuts.dataAccessOperation()")
    public void doAccessCheck() {
        // ...
    }
}
```

。。这个是 ref。。。。。

如果我们使用 原地 切点表达式，我们可以重写上面的例子：

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class BeforeExample {

    @Before("execution(* com.xyz.myapp.dao.*.*(..))")
    public void doAccessCheck() {
        // ...
    }
}
```



---

after returning advice

@AfterReturning，在方法 正常返回后 执行

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterReturning;

@Aspect
public class AfterReturningExample {

    @AfterReturning("com.xyz.myapp.CommonPointcuts.dataAccessOperation()")
    public void doAccessCheck() {
        // ...
    }
}
```

有时，你需要访问 在advice body 中访问 返回的值。你可以使用 绑定 返回值的 形式来获得 这个访问：

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterReturning;

@Aspect
public class AfterReturningExample {

    @AfterReturning(
        pointcut="com.xyz.myapp.CommonPointcuts.dataAccessOperation()",
        returning="retVal")
    public void doAccessCheck(Object retVal) {
        // ...
    }
}
```

returning 属性中的 名字 必须对应 advice 方法中的 参数名。当方法返回时，返回值作为相应的参数值 传递给 advice方法。returning  还可以限制 匹配 连接点，只有那些 返回 形参的类型的 连接点 才 匹配 ( 本例中是Object，所以匹配任何返回值。。。也不是吧，int？)。

无法使用 after returning advice 来 返回一个 完全不同的 ref



---

after throwing advice

当匹配的方法 抛出异常 而退出 时 执行。@AfterThrowing

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterThrowing;

@Aspect
public class AfterThrowingExample {

    @AfterThrowing("com.xyz.myapp.CommonPointcuts.dataAccessOperation()")
    public void doRecoveryActions() {
        // ...
    }
}
```

通常，你想要 advice 只有当 给定异常抛出时 执行，你也需要 在 advice body 里 获得 抛出的异常。你可以使用 throwing 属性 来 进一步匹配 和 绑定异常到 advice 的参数：

```java
@Aspect
public class AfterThrowingExample {

    @AfterThrowing(
        pointcut="com.xyz.myapp.CommonPointcuts.dataAccessOperation()",
        throwing="ex")
    public void doRecoveryActions(DataAccessException ex) {
        // ...
    }
}
```

。。方法 形参的类型。

throwing 属性 使用的名字 必须和 advice 方法的 参数的名字 匹配。当一个方法执行 由于 异常而 exit，异常被传递到 advice 方法。

> throwing 属性也用于 进一步匹配 到 那些 抛出指定异常 的方法执行。

注意，@AfterThrowing 并不表示 一般的 异常处理回调。



---

After (Finally) Advice

当一个匹配的方法执行退出时 执行。@After。after advice 必须可以处理 正常 和 异常 退出的情况。通常用来 释放资源 和类似的目的：

```java
@Aspect
public class AfterFinallyExample {

    @After("com.xyz.myapp.CommonPointcuts.dataAccessOperation()")
    public void doReleaseLock() {
        // ...
    }
}
```



---

Around Advice

最后一种 advice 是 around advice。它可以在 方法运行 前和 后 执行，并确定 该方法 何时，如何，以及是否真正开始运行。如果你需要以线程安全的方式 在方法执行 之前 或 之后 共享状态 (例如，启动和停止计时器)，则通常使用around advice。

@Around 的方法应该 声明 Object 作为 返回类型。并且第一个形参是 ProceedingJoinPoint。   
在advice body 中，调用 ProceedingJoinPoint 的 proceed() 来让 真正的方法 执行。直接调用proceed() 不带参数  会让调用者的 原始参数 在调用时 提供给 真正的方法。对于高级用例，proceed 有一个 接受 Object[] 的重载函数，调用时， Object[] 的值 将作为 底层方法的 参数。

around advice 返回的值是 调用者收到的 返回值。例如，cache 的切面，返回cache的值 或者 调用 proceed()。

如果 around advice 方法的 返回值是 void， 会把null 返回给调用者，从而忽略了proceed() 的结果。因此，建议将 around advice 的返回值设置为 Object。advice 方法 通常应该返回 proceed() 的返回值，即使 底层方法的 返回类型是 void。但是，根据用例，建议可以选择 返回 缓存值，包装值，或其他值。

```java
@Aspect
public class AroundExample {

    @Around("com.xyz.myapp.CommonPointcuts.businessService()")
    public Object doBasicProfiling(ProceedingJoinPoint pjp) throws Throwable {
        // start stopwatch
        Object retVal = pjp.proceed();
        // stop stopwatch
        return retVal;
    }
}
```



---

advice parameters

spring提供完全类型化的advice，意味着你可以在 advice 签名中 声明所需的参数 ( 正如我们之前在 returning，throwing 例子中看到的那样)，而不是一直使用 Object[] 数组。本节后面 将 看到 如何使 参数和其他context 值 可用于advice 主体。



---

访问当前 JoinPoint

任何advice 方法 可以声明 JoinPoint 类型的参数 作为 第一个参数。 around advice 需要的 ProceedingJoinPoint 是 JoinPoint 的 子类。

JoinPoint接口提供了一些有用的方法：

1. getArgs(): 返回方法参数
2. getThis(): 代理
3. getTarget(): 被代理对象
4. getSignature(): 被advice 的方法的 描述
5. toString(): 被advice 的方法的 有用的描述



---

传递参数到advice

我们已经看到 如何绑定 返回的值 或异常的值。为了使参数值 在 advice body里可以获得，你可以使用 args 的绑定形式。如果你使用一个参数名字 而不是 类型名字，在args 表达式中，对应的实参被 作为 形参值传递，当advice 被调用。    
比如，你想advice  DAO操作的执行，且 DAO操作的第一个参数是 Account对象，且你想在 advice body 中访问这个 account：

```java
@Before("com.xyz.myapp.CommonPointcuts.dataAccessOperation() && args(account,..)")
public void validateAccount(Account account) {
    // ...
}
```

切点表达式的 args(account,..) 部分 有2个目的，第一，进一步限制了匹配，只有那些 至少1个参数，且实参 是Account 实例 的方法的  方法执行。 第二，使得真正的 Account 对象 在 advice 中可用 通过 account参数。

另一种方式是 声明一个 切点 来 "提供" Account 对象值 当它匹配一个join point时，然后 从 advice里 ref 命名的切点。

```java
@Pointcut("com.xyz.myapp.CommonPointcuts.dataAccessOperation() && args(account,..)")
private void accountDataAccessOperation(Account account) {}

@Before("accountDataAccessOperation(account)")
public void validateAccount(Account account) {
    // ...
}
```



代理对象(this)，目标对象(target)，注解(@within,@target,@annotation,@args) 能可以以类似的方式绑定。接下来的2个例子战士了 如何匹配使用 @Auditable 注解注解的方法的执行 并提取审计代码。

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Auditable {
    AuditCode value();
}
```



```java
@Before("com.xyz.lib.Pointcuts.anyPublicMethod() && @annotation(auditable)")
public void audit(Auditable auditable) {
    AuditCode code = auditable.value();
    // ...
}
```



---

advice 参数和 泛型

spring aop 可以处理 类定义 和 方法参数中的 泛型。 假设你有下面的泛型类型：

```java
public interface Sample<T> {
    void sampleGenericMethod(T param);
    void sampleGenericCollectionMethod(Collection<T> param);
}
```



你可以 进一步限制 拦截 方法类型 为 指定参数类型 by 绑定 advice 参数 到 要拦截的方法参数类型

```java
@Before("execution(* ..Sample+.sampleGenericMethod(*)) && args(param)")
public void beforeSampleMethod(MyType param) {
    // Advice implementation
}
```

这个不对泛型起效，所以你 无 法 定 义 ：

```java
@Before("execution(* ..Sample+.sampleGenericCollectionMethod(*)) && args(param)")
public void beforeSampleMethod(Collection<MyType> param) {
    // Advice implementation
}
```

要完成这项工作，我们必须检查集合的每个元素，这是不合理的，因为我们也无法决定如何处理null。要实现类似的效果，你必须将参数设置为 Collection<?> 并手动检查元素类型。



---

决定实参名字

advice 调用中的 参数绑定 依赖于 切点表达式中使用的 名称与 advice和切点方法签名中声明的 参数名称的匹配。参数名不能通过 java  reflection 可用，所以 spring aop 使用如下策略 来决定 参数名：

1. 如果用户已明确指定参数名字，则使用指定的参数名称。advice 和 pointcut注解 都有 可选的 argNames 属性，你可以用这个 argNames 来 指定 被注解的方法的 参数名称。这些参数名字在 运行时可用，：

```java
@Before(value="com.xyz.lib.Pointcuts.anyPublicMethod() && target(bean) && @annotation(auditable)",
        argNames="bean,auditable")
public void audit(Object bean, Auditable auditable) {
    AuditCode code = auditable.value();
    // ... use code and bean
}
```

。。？感觉是 argNames 来 给 实参 取个代号， value的表达式 和 advice中 就使用这个代号 来ref 到实参。这种应该是 靠 index 的吧？

如果第一个参数是 JoinPoint，ProceedingJoinPoint，或 JoinPoint.StaticPart 类型，你可以 在argNames 属性 中 忽略，这个参数的名字：

```java
@Before(value="com.xyz.lib.Pointcuts.anyPublicMethod() && target(bean) && @annotation(auditable)",
        argNames="bean,auditable")
public void audit(JoinPoint jp, Object bean, Auditable auditable) {
    AuditCode code = auditable.value();
    // ... use code, bean, and jp
}
```

对JoinPoint,ProceedingJoinPoint,JoinPoint.StaticPart 类型的 第一个参数的 特殊处理 对于不收集 任何其他连接点 上下文的 advice 实例特别方便。这种情况下，你可以省略argNames：

```java
@Before("com.xyz.lib.Pointcuts.anyPublicMethod()")
public void audit(JoinPoint jp) {
    // ... use jp
}
```



2. 使用argNames 属性 有一点笨拙，所以如果argNames 属性没有被制定，spring aop 查看 class的 debug信息，尝试决定 参数名字 从 local variable table。这些信息 需要 类在编译时选择 debug信息( 至少-g:vars )。使用这个标志进行编译的后果是：1.你的代码非常容易理解( 逆向工程)，2.类文件大小非常大，3.删除未使用的本地变量 这个优化 没有被使用。 不管怎么样，你使用这个标记 进行构建 是不会有问题的。

如果使用AspectJ 编译器 (ajc) 编译 @AspectJ 切面，即使没有 debug 信息，你也不需要添加 argNames 属性，因为 编译器会保留所需的信息。



3. 如果代码在没有必须的 debug 信息的情况下 被编译了，spring aop 会尝试 推断 绑定变量 和 参数的 配对。如果在给定可用信息的情况下 变量的绑定 不能明确，会抛出 AmbiguousBindingException。
4. 上面的策略都失败，IllegalArgumentException。



---

Proceeding with Arguments

我们之前提过，我们将描述 如何写一个 proceed 调用  with 在spring aop 和 aspectJ 中一致 工作的参数。解决方案是 确保 advice 签名 按照顺序绑定 每个 方法参数：

```java
@Around("execution(List<Account> find*(..)) && " +
        "com.xyz.myapp.CommonPointcuts.inDataAccessLayer() && " +
        "args(accountHolderNamePattern)")
public Object preProcessQueryPattern(ProceedingJoinPoint pjp,
        String accountHolderNamePattern) throws Throwable {
    String newPattern = preProcess(accountHolderNamePattern);
    return pjp.proceed(new Object[] {newPattern});
}
```

。。。？



---

advice 顺序

当有多个 advice 在 一个 join point 上时 会发生什么？spring aop 遵循 与 AspectJ 相同的优先级规则 来确定 advice 执行的顺序。最高优先级 advice 在进入 连接点 时 先运行，从连接点退出时， 最后运行。

。。但是哪个优先？

当2个不同切面的 advice 都需要在同一个 join point 上 运行，除非你明确指定，否则 执行顺序是 未定义的。   
你可以控制执行顺序 by 指定优先级。 可以通过 在 切面类上 实现 spring 的 Ordered 接口，或者 使用 @Order 注解。 Ordered.getOrder() 返回值 小的 优先。

。。。应该能 一个切面 多个 相同切点的 advice 吧？

一个特定切面的每个不同advice 类型 在概念上意味着 直接应用于 连接点。因此，@AfterThrowing advice 方法不支持 从 随同的 @After @AfterReturning 方法 接受 异常

从spring 5.2.7 开始，在同一连接点 运行的 且 定义在 同一个@Aspect 类中的 advice 方法，按照它们的 advice type 分配优先级，从高到低：@Around，@Before,@After,@AfterReturning,@AfterThrowing。但注意，@After advice 方法 将在 同一个切面的 @AfterReturning 或 @AfterThrowing 之后 有效地调用，来遵循AspectJ 的 @After 的 "after finally advice" 语义 。

当 同一个 @Aspect类中的 2个 相同类型的 advice 在同一个 连接点运行时，顺序是 未定义的 ( 因为没有办法 通过 反射 从 class文件中获得 源码中的声明顺序)。考虑在每个 @Aspect 类中将此类 advice 方法 折叠成 一个 advice 方法，或者 将片段 重构为 单独的 @Aspect 类，然后通过Ordered ，@Order 进行排序。



---

5.4.5 Introductions

introduction ( 在AspectJ 中称为 类型间声明 (inter-type declaration)) 允许 切面 声明 被advice的对象 实现了 指定接口，并 代表这些对象 提供该接口 的实现。    
。。。感觉好像是：aop知道 被advice 对象的 接口，然后 aop 提供了这些接口的 实现，导致 对 被advice 对象的 调用 真正调用的是 aop提供的 对象。。。？。。不，是提供了默认方法，等于就是 为现有类，在不改变类的源文件的情况下，增加一个它实现的接口。。。应该只能用于 为 第三方增加接口吧，自己的代码 直接改就是了。也不是，第三方的，可以继承，，或者 final 的第三方类。

你可以创造一个 introduction by使用 @DeclareParents 。这个注解用来 声明 匹配的类型 有一个新的 parent (因此得名)。例如，给定一个名为 UsageTracked 的接口 和一个 DefaultUsageTracked 的实现，以下切面声明：所有实现 service 接口的 也实现 UsageTracked 接口。 (用于 JMX 统计)。

```java
@Aspect
public class UsageTracking {

    @DeclareParents(value="com.xzy.myapp.service.*+", defaultImpl=DefaultUsageTracked.class)
    public static UsageTracked mixin;

    @Before("com.xyz.myapp.CommonPointcuts.businessService() && this(usageTracked)")
    public void recordUsage(UsageTracked usageTracked) {
        usageTracked.incrementUseCount();
    }
}
```

被实现的接口 被决定 by 被注解的属性的类型。@DeclareParent 的 value 属性是一个 AspectJ 类型 pattern。匹配的type的 bean 都实现了 UsageTracked 接口。注意，之前的 before advice 例子，service bean 能直接用作 UseageTracked :

```java
UsageTracked usageTracked = (UsageTracked) context.getBean("myService");
```

。。这个属性的 static 有没有必要？



---

5.4.6 Aspect Instantiation Models

高级用法，刚开始使用AOP，可以跳过。

默认，在app ctx 中 每个 切面 都有一个 实例。aspectJ 称它为 the singleton instantiation model。   
它可以定义 具有 各种 生命周期的 aspect。spring支持 aspectJ 的 perthis,pertarget 实例化模型。percflow,percflowbelow,pertypewithin 目前不支持。

你可以定义一个 perthis 切面 通过 指明@Aspect注解中的 perthis。

```java
@Aspect("perthis(com.xyz.myapp.CommonPointcuts.businessService())")
public class MyAspect {

    private int someState;

    @Before("com.xyz.myapp.CommonPointcuts.businessService()")
    public void recordServiceUsage() {
        // ...
    }
}
```

上面的例子中，perthis 的作用是： 一个切面实例 被创建 for 每个单独的 执行业务服务的 service 对象 ( 每个在 连接点匹配 时 通过 切点表达式 绑定到 this 的 独立对象)。切面实例被创建，当service 对象 中的方法 第一次被调用时。切面超出scope 当 service 对象超出scope。在切面实例创建前，其中的任何advice 都不会运行。一旦创建了切面实例，声明的advice 就会在 匹配的连接点处运行，但仅当service 对象是 与此切面 关联的对象时。

pertarget 和 perthis 类似，不过 它创建 切面实例 for 每个 唯一 target 对象 在匹配的连接点处。

。。this 是代理对象， target 是被代理的对象。



---

5.4.7 AOP 例子

现在你已经了解了所有组成部分的工作原理，我们来组合在一起做一些有用的事情。

由于并发问题(如，死锁失败者) 会导致业务服务的 执行 失败。如果 操作被重试，那么可能在下次尝试中成功。   
对于适合在这种情况下 重试的业务 ( 不需要返回给用户解决冲突的 幂等操作)，我们希望透明地重试该操作以避免客户端看到 PessimisticLockingFailureException。这是一个明确跨服务层中多个服务的需求，所以非常适合使用 切面。

```java
@Aspect
public class ConcurrentOperationExecutor implements Ordered {

    private static final int DEFAULT_MAX_RETRIES = 2;

    private int maxRetries = DEFAULT_MAX_RETRIES;
    private int order = 1;

    public void setMaxRetries(int maxRetries) {
        this.maxRetries = maxRetries;
    }

    public int getOrder() {
        return this.order;
    }

    public void setOrder(int order) {
        this.order = order;
    }

    @Around("com.xyz.myapp.CommonPointcuts.businessService()")
    public Object doConcurrentOperation(ProceedingJoinPoint pjp) throws Throwable {
        int numAttempts = 0;
        PessimisticLockingFailureException lockFailureException;
        do {
            numAttempts++;
            try {
                return pjp.proceed();
            }
            catch(PessimisticLockingFailureException ex) {
                lockFailureException = ex;
            }
        } while(numAttempts <= this.maxRetries);
        throw lockFailureException;
    }
}
```

实现了Ordered 接口，我们可以设置这个切面的 优先级高于 事物切面 ( 我们需要在 重试时使用 新的事务)。我们对 businessService() 进行重试逻辑。

对应的spring配置：

```xml
<aop:aspectj-autoproxy/>

<bean id="concurrentOperationExecutor" class="com.xyz.myapp.service.impl.ConcurrentOperationExecutor">
    <property name="maxRetries" value="3"/>
    <property name="order" value="100"/>
</bean>
```



为了细化切面 以使其 仅重试 幂等操作，我们可以Idempotent 注解：

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface Idempotent {
    // marker annotation
}
```

使用这个注解到 service 操作。改变切面 只重试 幂等操作，修改切点表达式，以便仅匹配 @Idempotent。

```java
@Around("com.xyz.myapp.CommonPointcuts.businessService() && " +
        "@annotation(com.xyz.myapp.service.Idempotent)")
public Object doConcurrentOperation(ProceedingJoinPoint pjp) throws Throwable {
    // ...
}
```



---

5.5 基于Schema 的AOP

spring 提供了 对定义切面的支持，通过 aop 命名空间。   
支持和 @AspectJ 相同的 切点表达式 和 advice 类型。因此，在本书中，我们将重点放在 该语法上。阅读上一节来了解 如何编写切入点表达式 和 advice 参数的绑定。

需要导入 spring-aop schema 来使用 aop 命名空间。

在spring配置中，所有的 切面 和 advisor 元素 都必须 放在 aop:config 元素中 (你可以有多个 aop:config 元素)。

> 一个 aop:config 可以包含 pointcut, advisor, aspect 元素。 注意，必须按照这个顺序 声明。

aop:config 格式 重度使用 spring的 自动代理 机制，这可能导致问题( 比如 advice 没有被 织入)， 如果你已经使用了 显式 自动代理 通过 使用BeanNameAutoProxyCreator 或 类似的。   
推荐的使用模式是 仅使用aop:config 或 仅使用 AutoProxyCreator ，永远不要混合使用。

。。。显示 自动代理 是解决什么的？ 毕竟 aop 几乎必用的。。



---

5.5.1 声明切面

切面就是一个 spring的 常规java bean 定义。状态和行为 通过 对象的 属性和方法 来获得，xml中获得 切点和 advice 信息。

你可以使用 aop:aspect 元素声明 一个 切面，并使用 ref 属性 引用到 支持bean。

```xml
<aop:config>
    <aop:aspect id="myAspect" ref="aBean">
        ...
    </aop:aspect>
</aop:config>

<bean id="aBean" class="...">
    ...
</bean>
```

bean支持 切面 (本例中是 aBean) 能 像其他bean一样 进行配置和 依赖注入。



---

5.5.2 声明切点

你可以声明一个 命名的 切点 在 aop:config 中，使得 切点定义 在 多个切面 和 advisor 分享。

切点代表 任何service 层的 业务服务的 执行 的 定义：

```xml
<aop:config>
    <aop:pointcut id="businessService"
        expression="execution(* com.xyz.myapp.service.*.*(..))"/>
</aop:config>
```

你可以在xml 中 ref 到 类中定义的 切点表达式：

```xml
<aop:config>
    <aop:pointcut id="businessService"
        expression="com.xyz.myapp.CommonPointcuts.businessService()"/>
</aop:config>
```

假设你有一个 CommonPointcuts 切面 (在 sharing common pointcut definitions) 中定义。

然后声明一个 切点 到 切面中 就像 声明 顶级切点：

```xml
<aop:config>
    <aop:aspect id="myAspect" ref="aBean">
        <aop:pointcut id="businessService"
            expression="execution(* com.xyz.myapp.service.*.*(..))"/>
        ...
    </aop:aspect>
</aop:config>
```

和@AspectJ 切面非常类型，使用基于schema 定义的 切点可以收集 连接点上下文。下面 收集this 作为 连接点上下文 且传递它到 advice：

```xml
<aop:config>
    <aop:aspect id="myAspect" ref="aBean">
        <aop:pointcut id="businessService"
            expression="execution(* com.xyz.myapp.service.*.*(..)) &amp;&amp; this(service)"/>
        <aop:before pointcut-ref="businessService" method="monitor"/>
        ...
    </aop:aspect>
</aop:config>
```

advice 必须声明 接收 收集到的 连接点上下文的 参数，并且 参数的名称和 this(xxx) 的匹配

```java
public void monitor(Object service) {
    // ...
}
```

当组合 切点 子表达式时，```&amp;&amp; ``` 在xml 中很尴尬，所以你可以使用 and,or,not 等关键字分别代替 amp amp, || 和 ! 。

```xml
        <aop:pointcut id="businessService"
            expression="execution(* com.xyz.myapp.service.*.*(..)) and this(service)"/>
```



这种方式定义的切点 由它们的xml id引用，不能用作 命名的切入点 来形成复合切入点。因此，xml的命名切点支持 比 @ApsectJ 中的 更有限。



---

5.5.3 声明 Advice

使用5种 advice， 和@AspectJ 中的 有相同的语义。



---

Before Advice

在 aop:aspect 中使用 aop:before 。

```xml
<aop:aspect id="beforeExample" ref="aBean">
    <aop:before
        pointcut-ref="dataAccessOperation"
        method="doAccessCheck"/>
    ...
</aop:aspect>
```

dataAccessOperation 是 定义在顶层 ( aop:config ) 的 切点的id。可以内联定义 切点，使用pointcut 代替 pointcut-ref:

```xml
<aop:aspect id="beforeExample" ref="aBean">
    <aop:before
        pointcut="execution(* com.xyz.myapp.dao.*.*(..))"
        method="doAccessCheck"/>
    ...
</aop:aspect>
```

就像 @AspectJ 中讨论的，使用命名切点 可以增加可读性。

method 属性 指定了一个 方法( doAccessCheck ) ，来提供 advice 的 body。必须为包含advice 的 切面元素 ref 的bean 定义这个方法。在执行dao的操作前，doAccessCheck 方法会被调用。



---

After Returning Advice

匹配的方法 正常返回以后。

```xml
<aop:aspect id="afterReturningExample" ref="aBean">
    <aop:after-returning
        pointcut-ref="dataAccessOperation"
        method="doAccessCheck"/>
    ...
</aop:aspect>
```



returning 属性的名字 必须是 方法的形参名，传递 业务方法 返回的值。  类型的 限制 和 @AfterReturning 一样。

```xml
<aop:aspect id="afterReturningExample" ref="aBean">
    <aop:after-returning
        pointcut-ref="dataAccessOperation"
        returning="retVal"
        method="doAccessCheck"/>
    ...
</aop:aspect>
```

```java
public void doAccessCheck(Object retVal) {...}
```



---

After Throwing Advice

匹配的方法 由于抛出异常而退出时。定义在 aop:aspect 中，通过 after-throwing 元素

```xml
<aop:aspect id="afterThrowingExample" ref="aBean">

    <aop:after-throwing
        pointcut-ref="dataAccessOperation"
        method="doRecoveryActions"/>

    ...
</aop:aspect>
```

使用throwing 属性 来指明 方法形参 的名称，异常会通过这个参数来传递：

```xml
<aop:aspect id="afterThrowingExample" ref="aBean">
    <aop:after-throwing
        pointcut-ref="dataAccessOperation"
        throwing="dataAccessEx"
        method="doRecoveryActions"/>
    ...
</aop:aspect>
```

doRecoveryActions 方法必须声明 dataAccessEx 参数，类型的限制和 @AfterThrowing 一样。

```java
public void doRecoveryActions(DataAccessException dataAccessEx) {...}
```

。。throwing 的类型 是进一步匹配的作用



---

After (Finally) Advice

匹配的方法 退出。使用 after 元素

```xml
<aop:aspect id="afterFinallyExample" ref="aBean">
    <aop:after
        pointcut-ref="dataAccessOperation"
        method="doReleaseLock"/>
    ...
</aop:aspect>
```



---

Around Advice

最后是 around advice。around 方法执行 来执行，可以完成 before ，after，可以决定 when，how，甚至方法是否真正被执行。    
around advice 被经常使用于：如果你需要以线程安全的方式在方法之心前 和之后共享状态 (例如，启动和停止计时器)。

总是使用 最小权限原则 来匹配你的需求。

通过aop:around 元素 来声明 around advice。advice 方法应该声明 Object 作为它的返回值，且 第一个参数必须是 ProceedingJoinPoint。在around advice 方法的body中，你必须调用 proceed() 方法。   
更高级的用例：有一个重载版本的 proceed() 方法 接受 参数数组( Object[])。值会被用作 底层方法的 实参。

```xml
<aop:aspect id="aroundExample" ref="aBean">

    <aop:around
        pointcut-ref="businessService"
        method="doBasicProfiling"/>

    ...
</aop:aspect>
```

```java
public Object doBasicProfiling(ProceedingJoinPoint pjp) throws Throwable {
    // start stopwatch
    Object retVal = pjp.proceed();
    // stop stopwatch
    return retVal;
}
```



---

Advice Parameters

基于schema 的声明 和 @AspectJ 的方式 都支持 完全类型化的 advice -- 通过按名称匹配 切点参数和 advice 方法参数。   
如果你希望为advice 方法 显式指定参数名称，你可以使用 advice 元素的 arg-names 属性 来这么做，这个和注解中的argNames 处理方式相同。

```xml
<aop:before
    pointcut="com.xyz.lib.Pointcuts.anyPublicMethod() and @annotation(auditable)"
    method="audit"
    arg-names="auditable"/>
```

arg-names 接受 逗号分隔的 参数名 list。



下面是一个 更复杂的 基于 XSD的 例子，展示了 与强类型参数结合使用的advice：

```java
package x.y.service;

public interface PersonService {
    Person getPerson(String personName, int age);
}

public class DefaultPersonService implements PersonService {
    public Person getPerson(String name, int age) {
        return new Person(name, age);
    }
}
```

下面是切面，注意profile() 方法 接受 多个 强类型参数。第一个参数是 ProceedingJoinPoint，这个参数的存在表明 profile 会作为 around advice 使用。

```java
package x.y;

import org.aspectj.lang.ProceedingJoinPoint;
import org.springframework.util.StopWatch;

public class SimpleProfiler {

    public Object profile(ProceedingJoinPoint call, String name, int age) throws Throwable {
        StopWatch clock = new StopWatch("Profiling for '" + name + "' and '" + age + "'");
        try {
            clock.start(call.toShortString());
            return call.proceed();
        } finally {
            clock.stop();
            System.out.println(clock.prettyPrint());
        }
    }
}
```

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- this is the object that will be proxied by Spring's AOP infrastructure -->
    <bean id="personService" class="x.y.service.DefaultPersonService"/>

    <!-- this is the actual advice itself -->
    <bean id="profiler" class="x.y.SimpleProfiler"/>

    <aop:config>
        <aop:aspect ref="profiler">

            <aop:pointcut id="theExecutionOfSomePersonServiceMethod"
                expression="execution(* x.y.service.PersonService.getPerson(String,int))
                and args(name, age)"/>

            <aop:around pointcut-ref="theExecutionOfSomePersonServiceMethod"
                method="profile"/>

        </aop:aspect>
    </aop:config>

</beans>
```



```java
public final class Boot {

    public static void main(final String[] args) throws Exception {
        BeanFactory ctx = new ClassPathXmlApplicationContext("x/y/plain.xml");
        PersonService person = (PersonService) ctx.getBean("personService");
        person.getPerson("Pengo", 12);
    }
}
```



输出：

```
StopWatch 'Profiling for 'Pengo' and '12': running time (millis) = 0
-----------------------------------------
ms     %     Task name
-----------------------------------------
00000  ?  execution(getFoo)
```



---

Advice Ordering

一个连接点 多个 advice方法。顺序就和前面描述的一样。切面之间的优先级通过 aop:aspect 的 order 属性 来决定，或 增加 @Order 到 切面背后的 bean，或者 bena 实现Ordered 接口。

和定义在 同一个@Aspect 类中的 advice 方法的 优先级规则 相 反：当用一个aop:aspect 元素中定义的2条advice 都需要在同一个连接点运行时，aop:aspect 元素中的 advice 元素的优先级 从高到低。

例如，给定在同一个aop:aspect 元素中 适用于同一个连接点的 around 和 before advice，为了确保 around 的优先级高于 before， aop:around 必须在 aop:before 之前声明。

。。。？around 高于before？ 那before 怎么运行。。around 进去以后，就不可能再before了吧？ 还是说 proceed() 调用 实际方法的时候 还能before？ 这样不会 再来一次 around？

作为一般经验法则，若你发现在同一个 aop:aspect 中定义了 多条适用于 同一个连接点的advice，考虑将这些advice 合并成一个advice ，或将 片段重构为单独的 aop:aspect，你可以在 切面上进行排序。

。。。切面怎么排序？。。。就是 第一句。。order属性( 这个是 xml的order，不是java的，java的应该好似 Ordered接口的额外产出)，@Order，Ordered接口



---

Introductions

introduction( aspectJ 中的 inter-type声明) 使得切面 声明被 advice的对象 实现了 给定的接口，并提供了一个 该接口的 实现 为这些对象。

你可以make 一个introduction 通过 使用 aop:declare-parents ，包含在 aop:aspect 中。 你可以使用 aop:declare-parents 元素来声明 匹配的类型 有一个 新的 parent。例如，一个接口UsageTracked，实现DefaultUsageTracked，下面的 切面 声明了 所有实现 service 包中的所有类也实现了 UsageTracked 接口。( 为了暴露统计信息 通过 JMX)。

```xml
<aop:aspect id="usageTrackerAspect" ref="usageTracking">

    <aop:declare-parents
        types-matching="com.xzy.myapp.service.*+"
        implement-interface="com.xyz.myapp.service.tracking.UsageTracked"
        default-impl="com.xyz.myapp.service.tracking.DefaultUsageTracked"/>

    <aop:before
        pointcut="com.xyz.myapp.CommonPointcuts.businessService()
            and this(usageTracked)"
            method="recordUsage"/>

</aop:aspect>
```

useageTracking 这个bean 包含下面的方法：

```java
public void recordUsage(UsageTracked usageTracked) {
    usageTracked.incrementUseCount();
}
```

实现的接口通过 implement-interface 属性来声明。types-matching 的值 是ApsectJ 类型pattern。任何类型匹配的bean 实现了 UsageTracked 接口。在上面的 before advice 中，service包中 bean能直接用作 UsageTracked 的实现。

```java
UsageTracked usageTracked = (UsageTracked) context.getBean("myService");
```



---

5.5.5 Aspect Instantiation Models

基于schema 定义的 切面 的唯一被支持的 实例化模型是 单例模型。未来可能支持其它实例化模型。



---

5.5.6 Advisors

spring aop支持的 advisors 概念，在AspectJ 中没有 直接等价物。advisor 就像一个小的独立的 只有一条advice 的切面。advice 本身通过 bean 表示，并且必须实现 spring 的 advice type 中的一种。advisors 可以使用 aspectJ 切点表达式。

spring 通过 aop:advisor 支持 advisors 概念。你最常看到的是 它和 事物advice 一起使用，后者也有自己的命名空间：

```xml
<aop:config>

    <aop:pointcut id="businessService"
        expression="execution(* com.xyz.myapp.service.*.*(..))"/>

    <aop:advisor
        pointcut-ref="businessService"
        advice-ref="tx-advice"/>

</aop:config>

<tx:advice id="tx-advice">
    <tx:attributes>
        <tx:method name="*" propagation="REQUIRED"/>
    </tx:attributes>
</tx:advice>
```

pointcut-ref 也可以用 pointcut 定义内联的 切点表达式。

要定义advisort 的顺序，使用order 属性。

```xml
		<aop:advisor pointcut-ref="serviceOperation"
			advice-ref="txAdvice" order="2"/>
```



---

5.5.7 一个AOP Schema 例子

本节展示，如何将 并发锁失败时进行重试 的例子 用schema 重写。

下面是基本的切面实现：

```java
public class ConcurrentOperationExecutor implements Ordered {

    private static final int DEFAULT_MAX_RETRIES = 2;

    private int maxRetries = DEFAULT_MAX_RETRIES;
    private int order = 1;

    public void setMaxRetries(int maxRetries) {
        this.maxRetries = maxRetries;
    }

    public int getOrder() {
        return this.order;
    }

    public void setOrder(int order) {
        this.order = order;
    }

    public Object doConcurrentOperation(ProceedingJoinPoint pjp) throws Throwable {
        int numAttempts = 0;
        PessimisticLockingFailureException lockFailureException;
        do {
            numAttempts++;
            try {
                return pjp.proceed();
            }
            catch(PessimisticLockingFailureException ex) {
                lockFailureException = ex;
            }
        } while(numAttempts <= this.maxRetries);
        throw lockFailureException;
    }
}
```

注意 切面 实现了Ordered 接口，我们可以设置优先级 高于 事物advice ( 每次retry 都需要一个 新事物)。

这个切面和 @AspectJ 的例子一样，只不过移除了所有 注解。

下面是 spring配置：

```xml
<aop:config>

    <aop:aspect id="concurrentOperationRetry" ref="concurrentOperationExecutor">

        <aop:pointcut id="idempotentOperation"
            expression="execution(* com.xyz.myapp.service.*.*(..))"/>

        <aop:around
            pointcut-ref="idempotentOperation"
            method="doConcurrentOperation"/>

    </aop:aspect>

</aop:config>

<bean id="concurrentOperationExecutor"
    class="com.xyz.myapp.service.impl.ConcurrentOperationExecutor">
        <property name="maxRetries" value="3"/>
        <property name="order" value="100"/>
</bean>
```

我们暂时假设所有 业务服务都是 幂等的。如果不是，那么可以引入 @Idempotent 注解：

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface Idempotent {
    // marker annotation
}
```

修改切点

```xml
<aop:pointcut id="idempotentOperation"
        expression="execution(* com.xyz.myapp.service.*.*(..)) and
        @annotation(com.xyz.myapp.service.Idempotent)"/>
```



---

5.6 选择AOP 声明方式



5.6.1 spring aop 还是 full aspectj ?

使用 能起效的 最简单的。

spring aop 比 full aspectj 更简单，不需要引入 aspectJ 编译器/weaver 到你的 开发和构建 过程。   
如果你只需要 advice spring bean 上的 操作的执行，spring aop 是一个好的选择。如果你需要 advice 不是 spring 管理的 对象( 比如 domain 对象)，你需要使用 AspectJ。   
如果你想 advice 连接点 而不是 简单的 方法执行 ( 比如 属性获取 或设置连接点等)，你还需要使用 AspectJ。



5.6.2 @AspectJ 还是 xml 来完成 spring aop

如果你选择了 spring aop，你还需要选择 是 @AspectJ 还是 Xml 风格。需要权衡各种信息。

xml可能对于 已有spring 用户 更熟悉，它的背后是 POJO。当使用 aop 作为 配置 企业级 服务的工具，xml 是一个好的选择 ( 一个很好的测试是 你是否认为切点表达式 是你可能要独立更改的配置的一部分)。使用xml格式，可以说从你的配置中 更清楚系统中存在那些方面。

xml有2个缺点，首先，它没有把 它 所解决的需求的 实现 完全封装在一个地方。DRY原则说，系统内的任何知识都应该有一个 单一的，明确的，权威的 表示。使用xml 时，如何实现需求的 知识 被拆分成 bean的声明 和 xml配置文件。当你使用@AspectJ 时，信息被封装在一个单独模块(切面)中。   
其次，和@AspectJ 相比，xml在表达 方面稍有 限制：仅支持"单例"切面实例化模型，不能在xml定义的 切点中 使用 命名的切点。例如，你可以写下面3个@AspectJ的， 但是 只 有 前 2 种 是有 等价的xml配置的：

```java
@Pointcut("execution(* get*())")
public void propertyAccess() {}

@Pointcut("execution(org.xyz.Account+ *(..))")
public void operationReturningAnAccount() {}

@Pointcut("propertyAccess() && operationReturningAnAccount()")
public void accountPropertyAccess() {}
```

```xml
<aop:pointcut id="propertyAccess"
        expression="execution(* get*())"/>

<aop:pointcut id="operationReturningAnAccount"
        expression="execution(org.xyz.Account+ *(..))"/>
```

@AspectJ 支持额外的实例化模型，更丰富的切点组合。将切面保持为模块化单元的优点。还有以下优点：spring aop 和 AspectJ 都可以理解 @AspectJ 切面。因此，如果你以后决定需要AspectJ 的功能，你可以轻松迁移到经典的AspectJ 设置。总体来说，spring团队，更喜欢@AspectJ 风格的 切面。



---

5.7 混合切面类型

通过使用自动代理支持，schema定义的 aop:aspect，aop:advisor，甚至在同一配置中 其他样式的 代理和 拦截，完全可以混合 @AspectJ 风格切面。所有这些都是使用相同的底层实现的，所以可以共存。



---

5.8 代理机制

aop 使用 jdk动态代理 或 cglib 来创建 给定的目标对象的 代理。   

> 如果目标对象实现了 至少1个接口，jdk动态代理 会被使用。

对象实现的所有接口都会被 代理实现。   
如果没有实现接口，使用cglib。



如果要强制cglib 代理 ( 例如，代理 目标对象的 每个方法，而不是 在接口中声明的方法)，你可以这么做。但是，你应该先考虑下面的问题：

1. 使用cglib，final 方法 不能被advice，因为它们不能在 运行时生成的类中 被 覆盖。
2. spring4.0后，代理对象构造器不会再被调用2次，因为cglib 代理实例是通过 Objenesis创建的。仅当你的jvm不允许绕过构造器时，你可能会看到来自spring 的aop支持的 双重调用和调试日志的条目。



强制cglib，在aop:config 标签内设置 proxy-target-class 属性为 true：

```xml
<aop:config proxy-target-class="true">
    <!-- other beans defined here... -->
</aop:config>
```

要在使用@AspectJ 自动代理支持时 强制cglib代理，请将 aop:aspectj-autoproxy 标签的 proxy-target-class 属性设置为true：

```xml
<aop:aspectj-autoproxy proxy-target-class="true"/>
```



多个aop:config 部分在运行时被折叠成一个 统一的自动代理创建器，它应用任何 aop:config 部分 指定的 strongest 代理设置。(通常来自不同的xml bean定义文件)。这也适用于 tx:annotation-driven, aop:aspectj-autoproxy。

需要明确的是，在 tx:annotation-driven， aop:aspectj-autoproxy 或 aop:config 标签上 使用 proxy-target-class="true" 会强制对 所有3个元素 使用cglib 代理。



---

5.8.1 理解AOP 代理

spring aop 是基于代理的。

先，考虑你有一个 普通的，未代理的，没有什么特别的，直接的对象引用场景：

```java
public class SimplePojo implements Pojo {
    public void foo() {
        // this next method invocation is a direct call on the 'this' reference
        this.bar();
    }

    public void bar() {
        // some logic...
    }
}
```

如果你通过一个 对象引用 来调用方法，方法在对象引用上直接被调用。

![](https://docs.spring.io/spring-framework/docs/current/reference/html/images/aop-proxy-plain-pojo-call.png)





当客户端代码的引用 是 代理时，情况会发生变化。

![](https://docs.spring.io/spring-framework/docs/current/reference/html/images/aop-proxy-call.png)

```java
public class Main {

    public static void main(String[] args) {
        ProxyFactory factory = new ProxyFactory(new SimplePojo());
        factory.addInterface(Pojo.class);
        factory.addAdvice(new RetryAdvice());

        Pojo pojo = (Pojo) factory.getProxy();
        // this is a method call on the proxy!
        pojo.foo();
    }
}
```

关键是要理解，main 方法中的客户端代码 是一个 对代理的 引用。这意味着 对 对象引用的 方法调用 会调用到 代理上。因此，代理可以委托给 与特定方法调用相关的所有拦截器(advice)。然而，一旦调用最终达到 目标对象，它可能对自身进行的任何方法调用，如this.bar(),this.foo()， 都将直接调用这个引用，而不是代理。这意味着自调用不会导致 与方法 相关的advice 被运行。

一个好方法是重构你的代码。这样自调用就不会发生。这确实需要你做一些工作，但它是最好的，侵入性最小的方法。   
另一种方法很可怕，你可以将你的类中的逻辑 完全绑定到spring aop：

```java
public class SimplePojo implements Pojo {

    public void foo() {
        // this works, but... gah!
        ((Pojo) AopContext.currentProxy()).bar();
    }

    public void bar() {
        // some logic...
    }
}
```

将你的代码 和 aop 完全耦合，它是的类自己 知道 在aop 中使用。

当代理被创建时，还需要一些额外的配置：

```java
    public static void main(String[] args) {
        ProxyFactory factory = new ProxyFactory(new SimplePojo());
        factory.addInterface(Pojo.class);
        factory.addAdvice(new RetryAdvice());
        factory.setExposeProxy(true);				// 。

        Pojo pojo = (Pojo) factory.getProxy();
        // this is a method call on the proxy!
        pojo.foo();
    }
```

。。这个类也看下，，ProxyFactory。。。应该是 代码式 aop。



最后，AspectJ 不存在这个 自调用问题，因为它不是基于代理的 aop 框架。



---

5.9 编程式创建 @ApsectJ 代理

除了使用 aop:config 或 aop:aspectj-autoproxy 在配置中声明 切面外，还可以以编程的方式创建 advice 目标对象的代理。

使用 AspectJProxyFactory 类来创建 被advice的 目标对象 的 代理。

```java
// create a factory that can generate a proxy for the given target object
AspectJProxyFactory factory = new AspectJProxyFactory(targetObject);

// add an aspect, the class must be an @AspectJ aspect
// you can call this as many times as you need with different aspects
factory.addAspect(SecurityManager.class);

// you can also add existing aspect instances, the type of the object supplied must be an @AspectJ aspect
factory.addAspect(usageTracker);

// now get the proxy object...
MyInterfaceType proxy = factory.getProxy();
```

。。。这里 addAspect ，上面的 ProxyFactory 是 addAdvice 。。。 感觉差不多啊



---

5.10 在spring应用中使用AspectJ

目前位置，介绍的内容都是 纯spring aop。 本节中，如果你的需求超出了spring aop的功能，我们将了解如何使用 aspectJ 编译器或 编织器 来代替spring aop 或 作为spring aop 的补充。



。。。我感觉这节应该不用看吧。。。随便记录点 配置，，，文字就不翻了，，头大。。



增加spring-aspects.jar

@Configurable(autowire=Autowire.BY_TYPE)

@Configurable(autowire=Autowire.BY_NAME)



@Configurable(autowire=Autowire.BY_NAME,dependencyCheck=true)

。。这个是check 什么？ 非空？而且看了下( org.springframework.beans.factory.annotation.AnnotationBeanWiringInfoResolver.buildWiringInfo(Object, Configurable) )，autowire 不是 no(默认) 的情况下，才有用。

如果设置为true，spring 在配置后 校验，所否所有 属性( 非 原始类型或集合类型) 都被设置了。



如果你希望 依赖 在 构造器body 执行 前 被注入，并且在 构造器body 中可以使用 依赖。

```java
@Configurable(preConstruction = true)
```



为了这个能工作，被注解的 类必须被 AspectJ weaver 织入。



add `@EnableSpringConfigured` to any `@Configuration` class

```xml
<context:spring-configured/>
```



```xml
<bean id="myService"
        class="com.xzy.myapp.service.MyService"
        depends-on="org.springframework.beans.factory.aspectj.AnnotationBeanConfigurerAspect">

    <!-- ... -->

</bean>
```



如果@Configurable 没有被 织入，单元测试中 这个注解没有用。被织入的话，可以在容器外 单元测试。



```
AnnotationBeanConfigurerAspect
```

@EnableSpringConfigured

多个 App Ctx



拦截 @Transactional 的是 AnnotationTransactionAspect。

拦截 javax.transaction.Transactional 的是 JtaAnnotationTransactionAspect。

```
AbstractBeanConfigurerAspect
```

```
AbstractTransactionAspect
```



。。这个有点吊，和java根本不一样。 AbstractBeanConfigurerAspect 。最开始的 public aspect

```java
public aspect DomainObjectConfiguration extends AbstractBeanConfigurerAspect {

    public DomainObjectConfiguration() {
        setBeanWiringInfoResolver(new ClassNameBeanWiringInfoResolver());
    }

    // the creation of a new bean (any object in the domain model)
    protected pointcut beanCreation(Object beanInstance) :
        initialization(new(..)) &&
        CommonPointcuts.inDomainModel() &&
        this(beanInstance);
}
```

当你在spring 应用中使用 AspectJ 切面时， 自然希望能使用spring 来配置切面。aspectJ 运行时负责切面创建，通过spring配置aspectJ 创建的切面的 方式 取决于切面 使用的 aspectJ 实例化模型(per-xxx 子句)。

大多数AspectJ 切面都是 单例切面。这种切面配置很简单。你可以创建一个 ref 切面 的bean ，并包括 factory-method="aspectOf"，这确保 spring 通过想 AspectJ 请求它 而不是尝试自己 创建实例来获取切面。

```xml
<bean id="profiler" class="com.xyz.profiler.Profiler"
        factory-method="aspectOf"> 

    <property name="profilingStrategy" ref="jamonProfilingStrategy"/>
</bean>
```



非单例切面更难配置。但是，可以通过创建 原型 bean 定义 并使用spring-asepcts.jar 支持的 @Configuration，来配置 aspect 实例。



如果你想 一些@AspectJ 切面 你想使用AspectJ 织入( 如 对于 domain模型类型 使用 load-time weaving)，其他的@AspectJ 切面 使用 spring aop， 且这些切面 都通过 spring 配置，你需要告知spring aop @AspectJ 自动代理 支持 来 对某些@AspectJ 切面 进行自动代理。   
aop:include 中是 name pattern，只有 bean 名字匹配的，才会 spring-aop 自动代理。

```xml
<aop:aspectj-autoproxy>
    <aop:include name="thisBean"/>
    <aop:include name="thatBean"/>
</aop:aspectj-autoproxy>
```



```
EnableLoadTimeWeaving
```

```
<context:load-time-weaver/>
```



```java
@Aspect
public class ProfilingAspect {

    @Around("methodsToBeProfiled()")
    public Object profile(ProceedingJoinPoint pjp) throws Throwable {
        StopWatch sw = new StopWatch(getClass().getSimpleName());
        try {
            sw.start(pjp.getSignature().getName());
            return pjp.proceed();
        } finally {
            sw.stop();
            System.out.println(sw.prettyPrint());
        }
    }

    @Pointcut("execution(public * foo..*.*(..))")
    public void methodsToBeProfiled(){}
}
```



META-INF/aop.xml

```xml
<!DOCTYPE aspectj PUBLIC "-//AspectJ//DTD//EN" "https://www.eclipse.org/aspectj/dtd/aspectj.dtd">
<aspectj>

    <weaver>
        <!-- only weave classes in our application-specific packages -->
        <include within="foo.*"/>
    </weaver>

    <aspects>
        <!-- weave in just this aspect -->
        <aspect name="foo.ProfilingAspect"/>
    </aspects>

</aspectj>
```



```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <!-- a service object; we will be profiling its methods -->
    <bean id="entitlementCalculationService"
            class="foo.StubEntitlementCalculationService"/>

    <!-- this switches on the load-time weaving -->
    <context:load-time-weaver/>
</beans>
```



```java
public final class Main {

    public static void main(String[] args) {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml", Main.class);

        EntitlementCalculationService entitlementCalculationService =
                (EntitlementCalculationService) ctx.getBean("entitlementCalculationService");

        // the profiling aspect is 'woven' around this method execution
        entitlementCalculationService.calculateEntitlement();
    }
}
```

```
java -javaagent:C:/projects/foo/lib/global/spring-instrument.jar foo.Main
```



也可以的：

```java
public final class Main {

    public static void main(String[] args) {
        new ClassPathXmlApplicationContext("beans.xml", Main.class);

        EntitlementCalculationService entitlementCalculationService =
                new StubEntitlementCalculationService();

        // the profiling aspect will be 'woven' around this method execution
        entitlementCalculationService.calculateEntitlement();
    }
}
```



AspectJ load-time weaving 通过 一个或多个 META-INF/aop.xml 文件来配置。



spring 的 LTW 支持 的关键组件 是 org.springframework.instrument.classloading.LoadTimeWeaver 接口。



```java
@Configuration
@EnableLoadTimeWeaving
public class AppConfig {
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:load-time-weaver/>

</beans>
```



| Runtime Environment                                          | `LoadTimeWeaver` implementation |
| ------------------------------------------------------------ | ------------------------------- |
| Running in [Apache Tomcat](https://tomcat.apache.org/)       | `TomcatLoadTimeWeaver`          |
| Running in [GlassFish](https://eclipse-ee4j.github.io/glassfish/) (limited to EAR deployments) | `GlassFishLoadTimeWeaver`       |
| Running in Red Hat’s [JBoss AS](https://www.jboss.org/jbossas/) or [WildFly](https://www.wildfly.org/) | `JBossLoadTimeWeaver`           |
| Running in IBM’s [WebSphere](https://www-01.ibm.com/software/webservers/appserv/was/) | `WebSphereLoadTimeWeaver`       |
| Running in Oracle’s [WebLogic](https://www.oracle.com/technetwork/middleware/weblogic/overview/index-085209.html) | `WebLogicLoadTimeWeaver`        |
| JVM started with Spring `InstrumentationSavingAgent` (`java -javaagent:path/to/spring-instrument.jar`) | `InstrumentationLoadTimeWeaver` |
| Fallback, expecting the underlying ClassLoader to follow common conventions (namely `addTransformer` and optionally a `getThrowawayClassLoader` method) | `ReflectiveLoadTimeWeaver`      |



```java
@Configuration
@EnableLoadTimeWeaving
public class AppConfig implements LoadTimeWeavingConfigurer {

    @Override
    public LoadTimeWeaver getLoadTimeWeaver() {
        return new ReflectiveLoadTimeWeaver();
    }
}
```

```xml
    <context:load-time-weaver
            weaver-class="org.springframework.instrument.classloading.ReflectiveLoadTimeWeaver"/>
```



```xml
<scanning xmlns="urn:jboss:scanning:1.0"/>
```



































---

---

---

---



### 6. Spring AOP APIs

上一章 是 spring 对aop 的 @AspectJ 支持 和 基于schema 的切面定义。   
本章，底层的spring aop api。



---

6.1 Pointcut API in Spring



6.1.1 Concepts

切点模型 允许 切点 重用 独立的advice type。你可以在 一个切点中 target 不同advice。

```java
public interface Pointcut {

    ClassFilter getClassFilter();

    MethodMatcher getMethodMatcher();
}
```

把Pointcut 分为2部分，允许重用 class和method匹配 ，为了细粒度的组合操作。



ClassFilter 用来限制 切点 到 制定的类。如果 matches 始终返回true，那么 所有class都会被匹配。

```java
public interface ClassFilter {

    boolean matches(Class clazz);
}
```

MethodMatcher 通常更重要：

```java
public interface MethodMatcher {

    boolean matches(Method m, Class<?> targetClass);

    boolean isRuntime();

    boolean matches(Method m, Class<?> targetClass, Object... args);
}
```

matches(Method,Class) 用来测试 是否这个切点 和 目标类的 给定方法 匹配。这个 推导 能在aop代理创建 时执行，来避免 对每个方法调用进行测试。   
如果2参数的 matches 返回 true，isRuntime 方法返回true，那么 3参数的 matches 在每个 方法 调用上执行。这让切点 在 advice 开始前 看到 传递给方法的参数。

大部分MethodMatcher 实现是静态的，意味着它们的 isRuntime() 始终返回 false。这种情况下，3参数的matches 不会被调用。

如果可以，尽量让 切点静态，这样aop 会缓存 切点eval 结果，当 aop 代理创建时。



---

6.1.2 切点上的操作

spring 支持 对切点进行操作 (尤其是 并集 和 交集)

你可以组合切点使用 Pointcuts 中的静态方法 或 使用 ComposablePointcut 类。

当然，使用 切点表达式 更简单。

---

6.1.3 AspectJ Expression Pointcuts

从2.0 开始，spring中最重要的 切点类型是 AspectJExpressionPointcut。



---

6.1.4 简单切点实现

spring 提供 多个方便的切点实现。一部分你可以直接使用，其他的用于 生成 应用级别的 切点。



static pointcuts

static 切点 基于方法和目标类，不能获得 方法的参数。   
static 切点 适用于大部分用例，且非常合适。spring 能eval 一个 static 切点 只一次，当方法被第一次调用时。之后的方法调用，就不需要eval pointcut了。



本节的后续是描述 spring 中 static 切点实现



Regular Expression Pointcuts

指定静态切点的一种 显式方法是 RE。JdkRegexpMethodPointcut 是一个 通用的RE 切点。

在JdkRegexpMethodPointcut 类中，你可以提供 pattern list，如果任何一个满足，则切点 eval 为 true。

```xml
<bean id="settersAndAbsquatulatePointcut"
        class="org.springframework.aop.support.JdkRegexpMethodPointcut">
    <property name="patterns">
        <list>
            <value>.*set.*</value>
            <value>.*absquatulate</value>
        </list>
    </property>
</bean>
```



spring提供一个方便的类：RegexpMethodPointcutAdvisor，这个使得我们 ref Advice ( 记住 Advice 可以是一个 拦截器，before advice，throws advice ，等)。在幕后，spring 使用了 JdkRegexpMethodPointcut。

```xml
<bean id="settersAndAbsquatulateAdvisor"
        class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
    <property name="advice">
        <ref bean="beanNameOfAopAllianceInterceptor"/>
    </property>
    <property name="patterns">
        <list>
            <value>.*set.*</value>
            <value>.*absquatulate</value>
        </list>
    </property>
</bean>
```



---

attribute-driven 切点

static 切点的一个 重要类型是 metadata-driven 切点。这个使用了 metadata attributes 的值 ( 通常，是source-level metadata)



---

dynamic pointcuts

动态切点 比 静态切点 的eval 成本更高。它们考虑 方法参数 和 静态信息。这意味着必须在每次方法调用时，进行评估，并且结果不能缓存，因为参数 会有所不同。

主要例子是 control flow 切点。



---

Control Flow Pointcuts

spring 的control flow 切点 概念上类似 AspectJ 的cflow 切点，但是 功能更少。



---

6.1.5 切点父类。

spring 提供了 有用的父类 来帮助你 实现自己的切点。

由于静态切点是最有用的，你应该 继承 StaticMethodMatcherPointcut。这样只需要实现一个 抽象方法。

```java
class TestStaticPointcut extends StaticMethodMatcherPointcut {

    public boolean matches(Method m, Class targetClass) {
        // return true if custom criteria match
    }
}
```



---

6.1.6 自定义切点



---

6.2 spring的 Advice api



6.2.1 Advice 生命周期

每个advice 都是 spring bean。一个advice 实例能在 所有 被advice 对象间共享，或对 每个被advice 独占。这就是 per-class 或 per-instance advice。

per-class advice 最常用。它适用于通用advice，例如 事物advisor。这些不依赖 被代理对象的状态 或添加新状态。它们仅作用于方法 和 参数。

per-instance advice 是适合于 introduction, 来支持 mixin。这种情况下，advice 增加状态 到 被代理对象。

在一个aop 代理中，可以混合使用 per-class,per-instance advice。



---

6.2.2 Advice Types in Spring

spring支持多个 advice 类型，并且可以扩展支持任何advice 类型。本节表述 基础概念和 标准 advice 类型



---

Interception Around Advice

spring中最基本的 advice type 是 interception around advice。

spring 使用 AOP Alliance (。。aopalliance包) 接口 for 使用方法拦截的 around advice。类实现MethodInterceptor 和实现 around advice 的类也应该 实现以下接口：

```java
public interface MethodInterceptor extends Interceptor {

    Object invoke(MethodInvocation invocation) throws Throwable;
}
```

invoke方法的 MethodInvocation 参数暴露的 将会调用的方法，目标连接点，aop代理，和方法的实参。





















。。。。。随便抄点吧。。。



```java
public class DebugInterceptor implements MethodInterceptor {

    public Object invoke(MethodInvocation invocation) throws Throwable {
        System.out.println("Before: invocation=[" + invocation + "]");
        Object rval = invocation.proceed();
        System.out.println("Invocation returned");
        return rval;
    }
}
```



before advice 不需要 MethodInvocation 对象。

```java
public interface MethodBeforeAdvice extends BeforeAdvice {

    void before(Method m, Object[] args, Object target) throws Throwable;
}
```



```java
afterThrowing([Method, args, target], subclassOfThrowable)
```

如果 RemoteException 抛出，会调用这个advice

```java
public class RemoteThrowsAdvice implements ThrowsAdvice {

    public void afterThrowing(RemoteException ex) throws Throwable {
        // Do something with remote exception
    }
}
```

这种可以访问 被调用的 方法，实参，目标对象。

```java
public class ServletThrowsAdviceWithArguments implements ThrowsAdvice {

    public void afterThrowing(Method m, Object[] args, Object target, ServletException ex) {
        // Do something with all arguments
    }
}
```



```java
public static class CombinedThrowsAdvice implements ThrowsAdvice {

    public void afterThrowing(RemoteException ex) throws Throwable {
        // Do something with remote exception
    }

    public void afterThrowing(Method m, Object[] args, Object target, ServletException ex) {
        // Do something with all arguments
    }
}
```

如果 throws-advice 方法抛出异常， 那么这个异常 会覆盖 原异常。



```java
public interface AfterReturningAdvice extends Advice {

    void afterReturning(Object returnValue, Method m, Object[] args, Object target)
            throws Throwable;
}
```



不能调用 proceed()

```java
public interface IntroductionInterceptor extends MethodInterceptor {

    boolean implementsInterface(Class intf);
}
```



```java
public interface IntroductionAdvisor extends Advisor, IntroductionInfo {

    ClassFilter getClassFilter();

    void validateInterfaces() throws IllegalArgumentException;
}

public interface IntroductionInfo {

    Class<?>[] getInterfaces();
}
```







```
DefaultPointcutAdvisor
```



```
ProxyFactoryBean
```



```
ProxyConfig
```



```xml
<bean id="personTarget" class="com.mycompany.PersonImpl">
    <property name="name" value="Tony"/>
    <property name="age" value="51"/>
</bean>

<bean id="myAdvisor" class="com.mycompany.MyAdvisor">
    <property name="someProperty" value="Custom string property value"/>
</bean>

<bean id="debugInterceptor" class="org.springframework.aop.interceptor.DebugInterceptor">
</bean>

<bean id="person"
    class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="proxyInterfaces" value="com.mycompany.Person"/>

    <property name="target" ref="personTarget"/>
    <property name="interceptorNames">
        <list>
            <value>myAdvisor</value>
            <value>debugInterceptor</value>
        </list>
    </property>
</bean>
```



```xml
<bean id="personUser" class="com.mycompany.PersonUser">
    <property name="person"><ref bean="person"/></property>
</bean>
```





```xml
<bean id="myAdvisor" class="com.mycompany.MyAdvisor">
    <property name="someProperty" value="Custom string property value"/>
</bean>

<bean id="debugInterceptor" class="org.springframework.aop.interceptor.DebugInterceptor"/>

<bean id="person" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="proxyInterfaces" value="com.mycompany.Person"/>
    <!-- Use inner bean, not local reference to target -->
    <property name="target">
        <bean class="com.mycompany.PersonImpl">
            <property name="name" value="Tony"/>
            <property name="age" value="51"/>
        </bean>
    </property>
    <property name="interceptorNames">
        <list>
            <value>myAdvisor</value>
            <value>debugInterceptor</value>
        </list>
    </property>
</bean>
```



```xml
<bean id="proxy" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target" ref="service"/>
    <property name="interceptorNames">
        <list>
            <value>global*</value>
        </list>
    </property>
</bean>

<bean id="global_debug" class="org.springframework.aop.interceptor.DebugInterceptor"/>
<bean id="global_performance" class="org.springframework.aop.interceptor.PerformanceMonitorInterceptor"/>
```



---

6.5 Concise Proxy Definitions



```xml
<bean id="txProxyTemplate" abstract="true"
        class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">
    <property name="transactionManager" ref="transactionManager"/>
    <property name="transactionAttributes">
        <props>
            <prop key="*">PROPAGATION_REQUIRED</prop>
        </props>
    </property>
</bean>
```



```xml
<bean id="myService" parent="txProxyTemplate">
    <property name="target">
        <bean class="org.springframework.samples.MyServiceImpl">
        </bean>
    </property>
</bean>
```



```xml
<bean id="mySpecialService" parent="txProxyTemplate">
    <property name="target">
        <bean class="org.springframework.samples.MySpecialServiceImpl">
        </bean>
    </property>
    <property name="transactionAttributes">
        <props>
            <prop key="get*">PROPAGATION_REQUIRED,readOnly</prop>
            <prop key="find*">PROPAGATION_REQUIRED,readOnly</prop>
            <prop key="load*">PROPAGATION_REQUIRED,readOnly</prop>
            <prop key="store*">PROPAGATION_REQUIRED</prop>
        </props>
    </property>
</bean>
```



```java
ProxyFactory factory = new ProxyFactory(myBusinessInterfaceImpl);
factory.addAdvice(myMethodInterceptor);
factory.addAdvisor(myAdvisor);
MyBusinessInterface tb = (MyBusinessInterface) factory.getProxy();
```



```java
Advised advised = (Advised) myObject;
Advisor[] advisors = advised.getAdvisors();
int oldAdvisorCount = advisors.length;
System.out.println(oldAdvisorCount + " advisors");

// Add an advice like an interceptor without a pointcut
// Will match all proxied methods
// Can use for interceptors, before, after returning or throws advice
advised.addAdvice(new DebugInterceptor());

// Add selective advice using a pointcut
advised.addAdvisor(new DefaultPointcutAdvisor(mySpecialPointcut, myAdvice));

assertEquals("Added two advisors", oldAdvisorCount + 2, advised.getAdvisors().length);
```



##### `BeanNameAutoProxyCreator`是BPP，自动创建AOP 代理。



```xml
<bean class="org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator">
    <property name="beanNames" value="jdk*,onlyJdk"/>
    <property name="interceptorNames">
        <list>
            <value>myInterceptor</value>
        </list>
    </property>
</bean>
```



```
DefaultAdvisorAutoProxyCreator
```

非常适合用来 应用同样的advice 到 许多业务对象。一旦基础定义完成，以后增加 新的业务对象时，不需要再包含特殊的代理配置。

```xml
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"/>

<bean class="org.springframework.transaction.interceptor.TransactionAttributeSourceAdvisor">
    <property name="transactionInterceptor" ref="transactionInterceptor"/>
</bean>

<bean id="customAdvisor" class="com.mycompany.MyAdvisor"/>

<bean id="businessObject1" class="com.mycompany.BusinessObject1">
    <!-- Properties omitted -->
</bean>

<bean id="businessObject2" class="com.mycompany.BusinessObject2"/>
```



```
TargetSource
```

用于返回 target object。 每次aop proxy处理 方法调用时，TargetSource 会被询问以获得 target 实例。

你没有指定，就会有默认的，默认的已经够了。



```
HotSwappableTargetSource
```

```java
HotSwappableTargetSource swapper = (HotSwappableTargetSource) beanFactory.getBean("swapper");
Object oldTarget = swapper.swap(newTarget);
```

让AOP代理的目标被替换，同时让调用者保留对它的ref。

线程安全

```xml
<bean id="initialTarget" class="mycompany.OldTarget"/>

<bean id="swapper" class="org.springframework.aop.target.HotSwappableTargetSource">
    <constructor-arg ref="initialTarget"/>
</bean>

<bean id="swappable" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="targetSource" ref="swapper"/>
</bean>
```



pooling target sources 管理 一池的实例。

```xml
<bean id="businessObjectTarget" class="com.mycompany.MyBusinessObject"
        scope="prototype">
    ... properties omitted
</bean>

<bean id="poolTargetSource" class="org.springframework.aop.target.CommonsPool2TargetSource">
    <property name="targetBeanName" value="businessObjectTarget"/>
    <property name="maxSize" value="25"/>
</bean>

<bean id="businessObject" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="targetSource" ref="poolTargetSource"/>
    <property name="interceptorNames" value="myInterceptor"/>
</bean>
```

```xml
<bean id="poolConfigAdvisor" class="org.springframework.beans.factory.config.MethodInvokingFactoryBean">
    <property name="targetObject" ref="poolTargetSource"/>
    <property name="targetMethod" value="getPoolingConfigMixin"/>
</bean>
```



prototype target sources

```xml
<bean id="prototypeTargetSource" class="org.springframework.aop.target.PrototypeTargetSource">
    <property name="targetBeanName" ref="businessObjectTarget"/>
</bean>
```



ThreadLocal target source

对每个进入的请求 一个 对象。

```xml
<bean id="threadlocalTargetSource" class="org.springframework.aop.target.ThreadLocalTargetSource">
    <property name="targetBeanName" value="businessObjectTarget"/>
</bean>
```

ThreadLocal 实例有严重的问题 当在 多线程，多classloader 环境中 不当使用。

记得封装ThreadLocal。

记得ThreadLocal.set(null)







---























































































