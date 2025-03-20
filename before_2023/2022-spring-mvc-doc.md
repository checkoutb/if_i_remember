https://docs.spring.io/spring-framework/docs/current/reference/html/web.html

version 5.3.16



### Web on Servlet Stack

包含了对 Servlet-stack web app 的支持，来构造 Servlet API 和 部署到 Servlet 容器。

包含了 Spring MVC, View Technologies, CORS Suppoer, WebSocket Support

响应式请查看spring-web-reactive (https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#spring-web-reactive)



#### Spring Web MVC

spring web mvc 是基于Servlet API 的原生 web框架，从一开始就包含在spring 框架中。

平行于 mvc，spring框架5.0 引入了 reactive-stack web 框架：Spring WebFlux，这个也是基于 它的源代码模块。

这节讲 Spring Web MVC，下节Spring WebFlux。



---

1.1 DispatcherServlet

mvc和其他web框架一样，围绕这 前端控制器模式 设计，其中中央Servlet：DispatcherServlet，提供了共享算法 for 处理请求，实际工作由可配置的委托组件执行。非常灵活，支持多种工作流程。

DispatcherServlet 和其他Servlet 一样，需要根据 Servlet 规范 通过使用 Java 配置 或在web.xml 中进行声明和映射。

DisSer 使用 spring配置来 发现 用于请求映射，视图解析，异常处理等 所需的委托组件。

下面是使用java配置的 注册，初始化 DisSer，这个会被 Servlet 容器 自动发现：

```java
public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext servletContext) {

        // Load Spring web application configuration
        AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
        context.register(AppConfig.class);

        // Create and register the DispatcherServlet
        DispatcherServlet servlet = new DispatcherServlet(context);
        ServletRegistration.Dynamic registration = servletContext.addServlet("app", servlet);
        registration.setLoadOnStartup(1);
        registration.addMapping("/app/*");
    }
}
```



为了直接使用 ServletContext API，你也可以扩展 AbstractAnnotationConfigDispatcherServletInitializer。

对于编程方式的用例，可以使用GenericWebApplicationContext 替换 AnnotationConfigWebApplicationContext



web.xml 配置来 注册和初始化 DispatcherServlet

```xml
<web-app>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/app-context.xml</param-value>
    </context-param>

    <servlet>
        <servlet-name>app</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value></param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>app</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>

</web-app>
```



spring boot 使用不同的初始化 顺序。boot 不是使用 hook 来进入 Servlet 容器的生命周期，而是使用 spring 配置来引导自身和嵌入式servlet 容器。在spring 配置中检测filter 和 servlet 声明，并在servlet 容器中注册。



---

1.1.1 Context Hierarchy

DispatcherServlet 期望一个 WebApplicationContext 作为配置。WebAppCtx 有一个link 指向 ServletContext 和 关联的 Servlet。它还绑定到ServletContext 以便 应用可以使用 RequestContextUtils 的静态方法 来查找 WebApplicationContext。

对于许多应用，单个WebApplicationContext 是简单且够用的。也可以有一个context hierarchy，其中一个 root WebAppCtx 在多个 DisSer (或其他Servlet) 实例中共享，每个实例有它自己的 子WebAppCtx 配置。

root WebAC 通常包含 基础bean，例如，需要在多个Servlet 实例间共享的 data repository 和 业务服务。这些bena 是有效继承的，可以在 Servlet 的 子WebAC 中被覆盖。子WAC通常包含 特定于给定Servlet的 本地bean。



![](https://docs.spring.io/spring-framework/docs/current/reference/html/images/mvc-context-hierarchy.png)



下面配置了一个 WebAppCtx hierarchy：

```java
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class<?>[] { RootConfig.class };
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[] { App1Config.class };
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/app1/*" };
    }
}
```

如果app ctx 不需要 hierarchy，那么 getRootConfigClasses 返回全部的 配置，getServletConfigClasses 返回 null。

xml等价物：

```xml
<web-app>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/root-context.xml</param-value>
    </context-param>

    <servlet>
        <servlet-name>app1</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/app1-context.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>app1</servlet-name>
        <url-pattern>/app1/*</url-pattern>
    </servlet-mapping>

</web-app>
```

。。。DispatcherServlet 的 WebAC 中是否 包含了 AppCtx 的内容？ 主要是 /WEB-INF/root-context.xml ，这个在外部的。 外部的话，应该是 AppCtx 的。看样子是 外部的话，共用。



---

1.1.2 Special Bean Types

DisSer 委托特殊的bean 来处理请求并 响应。特殊bean 是指 实现框架契约的spring 管理的 Object 实例。这些通常有内置的契约，但你可以自定义它们的属性并扩展或替换它们。

下面是DispatcherServlet 检测到的特殊bean：

| Bean type                                                    | Explanation                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `HandlerMapping`                                             | 把 请求及用于预处理和后处理的拦截器列表 映射 到 handler。映射基于一些标准，细节取决于 HandlerMapping 实现。 2个主要的实现是 RequestMappingHandlerMapping (支持@RequestMapping )，SimpleUrlHandlerMapping (维护从URL到handler的显式注册) |
| `HandlerAdapter`                                             | 帮助DisSer 来调用 映射到request的 handler，不需要考虑handler 实际怎么被调用。例如，调用 被注解的controller 需要 解析注解。HandlerAdapter 的主要目的是保护 DisServ 免受这些细节的影响 |
| [`HandlerExceptionResolver`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-exceptionhandlers) | 解决异常的策略，可能将它们映射到 处理程序，HTML 错误视图 或其他目标。 |
| [`ViewResolver`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-viewresolver) | 将从 handler返回的 基于string的 视图名称解析为 用于呈现响应的实际试图。 |
| [`LocaleResolver`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-localeresolver), [LocaleContextResolver](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-timezone) | 根据客户的地区和可能的时区，以便提供国际化视图。             |
| [`ThemeResolver`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-themeresolver) | 解析你的web app 可以使用的 theme，例如提供个性化布局。       |
| [`MultipartResolver`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-multipart) | multi-part 请求的 解析的抽象                                 |
| [`FlashMapManager`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-flash-attributes) | 存储和检索可用于将属性从一个请求传递到另一个请求的 输入，输出 的FlashMap，通常通过重定向。 |



---

1.1.3 Web MVC Config

你可以声明 mvc 所需要的 特殊bean，如果DisSer 在WebAppCtx 中找不到 特殊bean，就会使用DispatcherServlet.properties 中列出的默认的类型



1.1.4 Servlet Config

在Servlet 3.0+ 环境中，你可以通过 编程 来配置 Servlet 容器，

```java
import org.springframework.web.WebApplicationInitializer;

public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext container) {
        XmlWebApplicationContext appContext = new XmlWebApplicationContext();
        appContext.setConfigLocation("/WEB-INF/spring/dispatcher-config.xml");

        ServletRegistration.Dynamic registration = container.addServlet("dispatcher", new DispatcherServlet(appContext));
        registration.setLoadOnStartup(1);
        registration.addMapping("/");
    }
}
```

。。构建Servlet 容器， 不是 DisSer。 不，就是构建 DisSer。那么这个怎么 声明root 配置 和 子配置？。。。擦，自己先配置好 WebAppCtx，取那些文件 自己定制好，然后 传给DisSer 就可以了。。。



WebApplicationInitializer 是mvc提供的接口，确保你的 实现 被发现和自动使用，用来初始化 任何 Servlet 3 容器。这个接口的一个抽象类是 AbstractDispatcherServletInitializer，使用它 注册DisSer 更简单，只需要覆盖掉方法 来指明 servlet mapping 和 DisSer 的配置。

推荐

```java
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return null;
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[] { MyWebConfig.class };
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }
}
```

如果使用基于xml的spring 配置，你可以扩展 AbstractDispatcherServletInitializer

```java
public class MyWebAppInitializer extends AbstractDispatcherServletInitializer {

    @Override
    protected WebApplicationContext createRootApplicationContext() {
        return null;
    }

    @Override
    protected WebApplicationContext createServletApplicationContext() {
        XmlWebApplicationContext cxt = new XmlWebApplicationContext();
        cxt.setConfigLocation("/WEB-INF/spring/dispatcher-config.xml");
        return cxt;
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }
}
```



AbstractDispatcherServletInitializer 也提供了 简单方法 来增加 Filter 实例，让它们自己映射到 DisServ

```java
public class MyWebAppInitializer extends AbstractDispatcherServletInitializer {

    // ...

    @Override
    protected Filter[] getServletFilters() {
        return new Filter[] {
            new HiddenHttpMethodFilter(), new CharacterEncodingFilter() };
    }
}
```

filter 的名字基于 它的类型，被自动映射到DisSer



AbstractDispatcherServletInitializer 有一个 protected 方法 isAsyncSupported，这个可以启用 对 DisSer 和 映射到它的所有filter 的异步支持。默认，是true。

最后，如果你要进一步自定义 DisSer， 可以重写 createDispatcherServlet 方法。



---

1.1.5 Processing

DisSer 处理请求的步骤：

1. 在请求中 搜索并绑定 WebAppCtx 作为controller 和 处理中其他元素可以使用的 属性，默认绑定在 DispatcherServlet.WEB_APPLICATION_CONTEXT_ATTRIBUTE 下。
2. locale resolver 被绑定到 request，以让流程中的元素解析处理请求( 呈现视图，准备数据等) 时要使用的 区域设置。如果不需要 区域解析，就不需要设置 区域解析器。
3. theme resolver 被绑定到 request，让视图 等元素决定使用哪个theme。如果你不使用theme，可以忽略
4. 如果你指定multipart file resolver，会检查请求的 multipart。如果找到，request 被封装到 MultipartHttpServletRequest 中，以供流程的其他元素处理。
5. 搜索合适的handler，如果找到，则运行 与handler 相关的 执行链 (预处理器，后处理器和控制器) 以准备渲染模型。或者对于带注解的 controller，可以呈现相应(在 HandlerAdpater 内) 而不是返回视图。
6. 如果模型返回，则呈现视图，如果没有模型返回( 可能是由于 预处理器 或后处理器 拦截了请求，可能是处于安全的原因)，则不会呈现试图，因为请求可能已经完成。



WebAppCtx 中的 HandlerExceptionResolver bean 用于解析 请求处理期间的 异常。这些异常解析器允许自定义处理异常的逻辑。

对于HTTP caching 支持，handler可以使用 WebRequest的 checkNotModified 方法，以及 带注解的控制器(后面会有)。

你可以自定义 DisSer 通过 增加 Servlet 初始化参数( init-param ) 到 web.xml 的 servlet 声明中，下面是支持的参数

| Parameter                        | Explanation                                                  |
| -------------------------------- | ------------------------------------------------------------ |
| `contextClass`                   | 实现了 ConfigurableWebApplicationContext 的类，由该servlet 实例化和本地配置。默认情况下，使用XmlWebApplicationContext。 |
| `contextConfigLocation`          | 字符串，传递到context 实例 ( 被 contextClass 指定的) 来 表明 在哪里能找到 contexts。可以逗号分隔 支持多个context。如果多个 上下文地址 中定义了重复的 bean，则最后的位置 优先。 |
| `namespace`                      | Namespace of the `WebApplicationContext`. Defaults to `[servlet-name]-servlet`. |
| `throwExceptionIfNoHandlerFound` | 当无法为request 找到 handler 时，是否抛出 NoHandlerFoundException。异常能被 HandlerExceptionResolver 捕获 并处理 (例如，使用一个 @ExceptionHandler controller 方法)。     默认，false，这种情况下，DispatcherServlet 设置 response 状态为404，而不是异常。      如果默认servlet handling 也被配置了，那么无法解析的request 总是 转发到 默认的servlet ，而不是404. |



---

1.1.6 Path Matching

servlet aip 将完整 request path 公开为 requestURI，并进一步细分为 contextPath, servletPath, pathInfo，其值根据servlet 的 mapping方式不同 而不同。根据这些输入，spring mvc 需要确定 用于 映射handler 的 查找路径，它是 DisSer 本身映射中的路径，不包括 contextPath 和任何 servletMapping 前缀。

servletPath 和 pathInfo 被解码，这使得它们无法直接与完整的requestURI 进行比较 以派生 lookupPath，并使得它需要解码requestURI。但是，这引入了它自己的问题，因为路径可能包含编码的保留字符，例如/ 或 ; 这会在解码后改变路径的结构，可能导致安全问题。因此，servlet容器可能会在不同程度上规范化servletPath，这使得更不可能针对requestURI 进行 startsWith 比较。

这就是为什么要避免基于前缀的servletPath 映射类型 附带的servletPath。如果DisSer 被映射为 带有 / 或没有前缀 /* 的默认servlet，并且servlet 容器是4.0+，则spring mvc 能检测servlet 映射类型并完全避免使用servletPath 和 pathInfo。在 3.1 Servlet 容器中，假设相同的 servlet mapping 类型，可以通过在 mvc配置中通过 Path Matching 提供 带有 alwaysUseFullPath=true的 UrlPathHelper 来实现等价。

幸运的是，默认的servlet 映射 / 是一个不错的选择。但是仍存在一个问题，及需要对requestURI 进行decode才能与控制器映射进行比较。这也是不可取的，因为可能会解码改变路径结构的保留字符。如果不需要这些字符，那么你可以拒绝它们( 如 spring security http)， 或者你可以使用urlDecode=false 配置 UrlPathHelper 但 控制器映射将需要与编码路径匹配，这可能不是一直运行良好。此外，有时DisSer 需要与另一个Servlet 共享URL 空间。，可能需要通过前缀进行map

上面的issue 可以被解决，通过从PathMatcher 切换到5.3 或更高版本中可用的 已解析的 PathPattern。不同于需要 lookup path decoded 或 controller mapping encoded 的 AntPathMatcher，parsed PathPattern 与 被称为RequestPath 的 路径解析表示匹配，一次 一个path段。这允许单独 解码 和 清理 路径段值，而不会有改变路径结构的风险。parsed PathPattern 也支持 使用servletPath 前缀 映射，只要前缀保持简单 并且 没有任何需要 encode 的字符。

。。。我也不知道在说什么。。。就是 机翻了。。



---

1.1.7 Interception

所有 HandlerMapping 接口的实现 都支持 handler interceptor，当你想要 对 特定的请求 应用特殊的功能 时很有用。拦截器必须实现 HandlerInterceptor，提供3个方法，来提供足够的灵活性来进行 pre-process 和 post-process。

1. preHandle(..)，在真正handler执行前 执行
2. postHandle(..)，handler执行后 执行
3. afterCompletion(..)，整个request 已经 结束。

preHandler 返回 boolean。你可以使用这个方法 来 break 或 continue 执行链的 处理。当这个方法返回 true，handler链继续执行。返回false，DisServ 假设 拦截器 已经处理了请求 (例如，呈现了适当的视图) 并且不会继续执行 其他拦截器 和执行链 中的 实际处理程序。

如何配置，看具体的 Intercepors 章节，你还可以在 各个HandlerMapping 实现 上使用 setter 来直接注册它们。

注意，postHandler 对于 @ResponseBody 和 ResponseEntity 方法的用处 不大，因为这些 响应 在 postHandle 之前 就被 HandlerAdapter 写入和提交。此时已经太晚了，无法对 response 做出任何修改，比如增加一个额外的header。对于这种场景，你可以实现 ResponseBodyAdvice，并将其声明为 Controller Advice bean 或直接在 RequestMappingHandlerAdapter 上进行配置。



---

1.1.8 Exceptions

如果异常发生在 request mapping 或 被 request handler 抛出(如 @Controller)，DisSer 委托给 HandlerExceptionResolver bean 链 来 resolve 异常，并提供 处理，一般来说是一个 错误响应。

| `HandlerExceptionResolver`                                   | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `SimpleMappingExceptionResolver`                             | 异常类名 和 错误视图名称 之间的映射。用于在浏览器上呈现错误页面。 |
| [`DefaultHandlerExceptionResolver`](https://docs.spring.io/spring-framework/docs/5.3.16/javadoc-api/org/springframework/web/servlet/mvc/support/DefaultHandlerExceptionResolver.html) | 解决Spring MVC引发的异常，并映射到http 状态码。参阅 可替代的 ResponseEntityExceptionHandler 和 REST API exception |
| `ResponseStatusExceptionResolver`                            | 使用 @ResponseStatus(。。这个应该是在异常类上的？。类和方法都可以。而且不是异常类，是controller类或它的方法) 注解 解决异常，根据注解的值 将异常映射到 http 状态码。 |
| `ExceptionHandlerExceptionResolver`                          | 通过 调用 @Controller 或 @ControllerAdvice 类中的 @ExceptionHandler 方法来解决异常。 |



---

Chain of Resolvers

你可以形成 一个 异常处理链，通过 声明多个 HandlerExceptionResolver bean  到你的 spring 配置并且 如果有必要可以设置 order属性。 order越大，异常解析链 中的位置就越晚。

HandlerExceptionResolver 的 合约 规定它可以返回：

1. 指向错误试图的 ModeAndView
2. 如果在解析器中处理了异常，则 empty ModeAndView
3. null，如果异常没有被解决，以让后续的 解析器 尝试处理，如果最终异常仍然存在，则允许 抛出到 servlet 容器。

MVC Config 自动为默认spring mvc 异常，@ResponseStatus 注解的异常，对@ExceptionHandler方法的支持 声明 内置解析器。你可以自定义该列表或替换它。



---

Container Error Page

如果没有 HandlerExceptionResolver 能解析异常，导致 继续传播，或者如果响应状态设为错误状态码(4xx，5xx)，则servlet 容器可以在html中呈现错误页面。要自定义容器中的默认错误页面，可以在web.xml 中声明一个错误页面映射：

```xml
<error-page>
    <location>/error</location>
</error-page>
```

根据上面的例子，当一个异常 冒出 或 response有 error status，servlet 容器 在容器内 向配置的URL( 如/error) 发送dispatch。然后由 DisSer 处理，可能将其映射到 @Controller，这个可能实现为 返回带有模型的 错误视图名称 或 呈现JSON response：

```java
@RestController
public class ErrorController {

    @RequestMapping(path = "/error")
    public Map<String, Object> handle(HttpServletRequest request) {
        Map<String, Object> map = new HashMap<String, Object>();
        map.put("status", request.getAttribute("javax.servlet.error.status_code"));
        map.put("reason", request.getAttribute("javax.servlet.error.message"));
        return map;
    }
}
```



servlet api 不提供在java 中创建 错误页面映射的方法。但是你可以同时使用 WebApplicationinitializer 和一个 最小的 web.xml。



---

1.1.9 View Resolution

mvc 定义 ViewResolver 和 View 接口，让你 在浏览器中 渲染模型，而不需要使用特定的视图技术。

ViewResolver 提供一个 view name 和 真实view 之间的 映射。view解决了 移交给特定视图技术 之前的 数据准备问题

| ViewResolver                     | Description                                                  |
| -------------------------------- | ------------------------------------------------------------ |
| `AbstractCachingViewResolver`    | AbstractCachingViewResolver 的子类 cache 它们解析的 视图实例。cache提高了某些视图技术的性能。你可以通过将 cache 属性设置为false 来关闭。此外，如果你必须在运行时刷新某个视图( 如，修改FreeMarker 模板时)， 你可以使用 removeFromCache(viewName, locale) 方法。 |
| `UrlBasedViewResolver`           | ViewResolver 的简单实现，直接将逻辑视图名称 直接解析为 URL without 显式的mapping 定义。如果你的逻辑名称 能直接 和 视图资源的名称匹配 不需要任何映射，则这个是合适的。 |
| `InternalResourceViewResolver`   | UrlBasedViewResolver 的方便的子类，支持 InternalResourceView (实际上是 Servlet 和 JSP) 以及 JstlView 和 TilesView 等子类。 你可以使用 setViewClass(..) 为该解析器生成的所有视图 指定视图类。 |
| `FreeMarkerViewResolver`         | UrlBasedViewResolver 的便捷子类，支持 FreeMarkerView 和它的自定义子类 |
| `ContentNegotiatingViewResolver` | ViewResolver 的实现，它根据请求文件名 或Accept header 解析视图。 |
| `BeanNameViewResolver`           | ViewResolver 的实现，把view 名字解释为当前上下文中的 bean 名字。这是一个非常灵活的变体，它允许基于不同的视图名称 混合和匹配 不同的view 类型。每个这样的View 都可以定义个 bean。 |



---

Handling

你可以 chain view resolver by 声明 多个 resolver bean，并在必要时 通过设置order 属性来指定 顺序。记住，order 越大，视图解析器 链中位置 越晚。

ViewResolver 的约定 指定 它可以返回 null 以指示 无法找打视图。但是 对于JSP 和 InternalResourceViewResolver， 判断JSP 是否存在的 唯一方法是通过 RequestDispatcher 执行分派。因此，你必须始终将 InternalResourceViewResolver 配置为 视图解析器链 的最后一个。

配置 view 解析 就是 增加 ViewResolver bean 到你的spring 配置。mvc config 提供了 专门的配置api for View Resolvers 和 添加无逻辑View Controller，这对于没有controller逻辑的HTML 模板渲染很有用。



---

Redirecting

视图名的 特殊前缀 redirect: ，使得你执行一个 重定向。UrlBasedViewResolver (和它的子类) 识别 它 为需要重定向的 指令。view 名字的剩余部分就是 重定向的URL

最终效果 与控制器返回 一个 RedirectView 相同，但现在控制器本身可以根据逻辑视图名称进行操作。一个逻辑视图名( 如 redirect:/myapp/some/resource) 重定向 是 相对于 当前servlet context。 redirect:https://myhost.com/some/arbitiarry/path 重定向 是 绝对路径。

注意，如果 controller 的方法被 @ResponseStatus 注解，注解值 优先于 RedirectView 设置 的 响应状态。



---

Forwarding

你还可以为 最终由 UrlBasedViewResolver 和它子类 解析的 视图名称使用特殊的 forward: 前缀。这将创建一个 InternalResourceView，它执行 RequestDispatcher.forward()。 因此，这个前缀 对 InternalResourceViewResolver 和 InternalResourceView (for JSP) 没有用处，但如果你使用另一种视图技术 但仍希望强制 forward/转发 资源 来让 Servlet/JSP 引擎来处理， 它很有用。 注意，你可以 chain 多个view resolvers。



---

Content Negotiation

ContentNegotiatingViewResolver 不会resolve view，而是委托给其它 view resolver 和 选择接近客户端要求的的表达/格式 的视图。客户端要求的表达/格式 可以通过 Accept 头 或 一个查询参数( 如，/path?format=pdf)

ContentNegxxVR 选择一个合适的 view 来处理请求 by 比较 请求的media type 和 它的每个ViewResolvers 关联 的 View支持的 media type (通常是 Content-Type)。list中 地一个满足的 View 会 返回 representation 到客户端。如果 ViewResolver 链 无法找到 合适的View，则查询 DefaultViews 属性指定的 Views。后者适合于 可以呈现当前资源的 合适的 表现 的 单例 Views，不管逻辑view name。Accept 头可以包含 通配符 (如 text/* )，这种情况下，一个 Content-Type 是 text/xml 的 View 是 匹配的。



---

1.1.10 Locale

spring 架构中的大部分都支持 国际化。MVC也是。

DispatcherServlet 让你 自动 解析 message by 使用 客户端的locale。这个是通过 LocaleResovler 来完成的。

当请求进来，DisSer 查找 locale resovler ，如果找到，它尝试 使用这个 resolver 来设置 locale。通过使用 RequestContext.getLocale() 方法，你可以看到 locale resovler 用于 解析的 locale。

除了自动locale resolution，你也可以 添加一个 interceptor 来处理 mapping 来修改 locale 在特殊情况下。

一般情况下，被定义在 org.springfw.web.servlet.i18n 包中 的 locale resolver 和 拦截器 ，被配置到你的 app ctx。

下面是spring提供的 locale revolers

Time Zone

Header Resolver

Cookie Resolver

Session Resolver

Locale Intercaptor



---

Time Zone

除了获得客户端的 locale(语言环境)，了解其时区 通常也很有用。LocaleContextResolver 接口提供了对LocaleResolver 的扩展，它允许解析器 提供 更丰富的 LocaleContext，这个可能包含时区信息。

当可用时，用户的 TimeZone 能被 获得 通过 RequestContext.getTimeZone() 。时区信息 自动被 任何 注册在spring的 ConversionService 中的 时间 Converter 和 Formatter 对象 使用。



---

Header Resolver

locale resovler 检查 客户端发送的 request 中的 accept-language 头。通常，这个头 包含了 客户端的 OS 的 locale。 注意，这个 resolver 不支持 时区信息。



---

Cookie Resolver

这个 locale resolver 检查 可能存在于客户端上的 Cookie，查看是否有 指定 Locale 或 TimeZone。如果有，那就用指定的。 通过 这个 语言环境解析器 的 属性，你可以制定  cookie 名字 和 最长存货。

```xml
<bean id="localeResolver" class="org.springframework.web.servlet.i18n.CookieLocaleResolver">

    <property name="cookieName" value="clientlanguage"/>

    <!-- in seconds. If set to -1, the cookie is not persisted (deleted when browser shuts down) -->
    <property name="cookieMaxAge" value="100000"/>

</bean>
```

下面是 CookieLocaleResolver 的属性

| Property       | Default                   | Description                                                  |
| -------------- | ------------------------- | ------------------------------------------------------------ |
| `cookieName`   | classname + LOCALE        | The name of the cookie                                       |
| `cookieMaxAge` | Servlet container default | The maximum time a cookie persists on the client. If `-1` is specified, the cookie will not be persisted. It is available only until the client shuts down the browser. |
| `cookiePath`   | /                         | Limits the visibility of the cookie to a certain part of your site. When `cookiePath` is specified, the cookie is visible only to that path and the paths below it. |



---

Session Resolver

SessionLocaleResolver 使你 获得 Locale 和 TimeZone 从 (用户请求中可能关联的) session中。和 CookieLocaleResolver 相比，这个策略 将 本地选择的locale( 。。估计是说 客户端选择的locale) 存储在 Servlet 容器的 HttpSession 中。 因此，这些设置 是临时的，session结束就没有了。

注意，这个和 外部session 管理机制( 如Spring Session 项目) 没有直接关系。SessionLocaleResolver eval 和 修改 当前HttpServletRequest 中 相应的 HttpSession 属性。



---

Locale Interceptor

你可以 允许 locale 的修改， 通过 增加 LocaleChangeInterceptor 到 某个 HandlerMapping 定义中。   

它检查请求中的参数，并相应地修改 locale，通过 调用 dispatcher 的 app ctx 中的 LocaleResolver 的 setLocale 方法。

下面的例子展示了 包含一个 siteLanguage 参数的 对所有 *.view 资源的 调用，会修改locale。比如 `https://www.sf.net/home.view?siteLanguage=nl`, changes the site language to Dutch.

```xml
<bean id="localeChangeInterceptor"
        class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor">
    <property name="paramName" value="siteLanguage"/>
</bean>

<bean id="localeResolver"
        class="org.springframework.web.servlet.i18n.CookieLocaleResolver"/>

<bean id="urlMapping"
        class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
    <property name="interceptors">
        <list>
            <ref bean="localeChangeInterceptor"/>
        </list>
    </property>
    <property name="mappings">
        <value>/**/*.view=someController</value>
    </property>
</bean>
```



---

1.1.11 Themes

你可以应用 Spring Web MVC框架 theme 来设置 app的整体外观，从而增强用户体验。

theme 是 静态资源的集合，通常是 style sheet 和 images，它们会影响app的 视觉样式。



---

Defining a theme

为了在 web app 中使用 theme，你 必 须 配置一个 ThemeSource 接口的实现。   

WebAppCtx 扩展了ThemeSource 接口( 。。 看了下，没有扩展，最多就是  GenericWebApplicationContext 直接实现了 ThemeSource 接口。)，但 将职责 委托给 专门的实现。默认，委托给 ResourceBundleThemeSource，这个类 从 classpath 的root 加载 properties 文件。   

要使用 自定义 ThemeSource 实现，或配置 ResourceBundleThemeSource 的 基础name 前缀， 你可以注册 一个名叫 themeSource  的bean 到 app ctx。web app ctx 自动根据名字 发现这个bean 并使用。

当你使用 ResourceBundleTS ，theme 被定义在 简单的properties 文件中。properties 文件列举 构成这个theme的资源：

```
styleSheet=/themes/cool/style.css
background=/themes/cool/img/coolBg.jpg
```

properties 的key 是 名字 是 view code 中引用 主题元素的名称。 对于JSP，你可以使用 spring:theme 自定义tag，这个类似与 spring:message tag。下面的jsp 使用了 上面定义的 theme

```xml
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags"%>
<html>
    <head>
        <link rel="stylesheet" href="<spring:theme code='styleSheet'/>" type="text/css"/>
    </head>
    <body style="background=<spring:theme code='background'/>">
        ...
    </body>
</html>
```



默认下，ResourceBundleTS 使用 空的base name 前缀，导致，从classpath 的root 加载properties 文件。

你可以把 cool.properties theme定义放到 classpath 的root 的 目录中( 如，/WEB-INF/classes) 。ResourceBunleTS 使用标准Java 资源包加载机制，允许主题的完全国际化，比如，我们可以有一个 /web-inf/classes/cool_nl.properties ，使用荷兰语。



---

Resolving Themes

在你定义 theme后，你决定使用哪个。

DisServ 搜索 名字是 themeResolver 的bean 来找到 要使用的 ThemeResolver 实现。

theme resolver 和 LocaleResovler 类似。它检测 用于特定请求的 theme，还可以更改请求的 theme。

下面是spring 提供的 主题解析器

| Class                  | Description                                                  |
| ---------------------- | ------------------------------------------------------------ |
| `FixedThemeResolver`   | Selects a fixed theme, set by using the `defaultThemeName` property. |
| `SessionThemeResolver` | The theme is maintained in the user’s HTTP session. It needs to be set only once for each session but is not persisted between sessions. |
| `CookieThemeResolver`  | The selected theme is stored in a cookie on the client.      |

。。固定，session，cookie

spring 也提供了 ThemeChangeInterceptor 来 通过一个简单的 request 参数 修改 request的 theme。



---

1.1.12 Multipart Resolver

MultipartResolver 是一个策略 for 转换 包含文件的multipart request。

一个实现 基于 Commons FileUpload ， 一个基于 Servlet 3.0 multipart request parsing。

为了允许 处理 multipart，你需要 声明一个 MultipartResolver bean 到你的 DispatcherServlet 的spring 配置，并且 bean 名字是 multipartResolver。 DisServ 发现它，并 应用它 到 输入的request。 当一个POST 带了 content type是 multipart/form-data 到达，resolver 转换 content，将当前 HttpServletRequest 包装成 MultipartHttpSrevletRequest，来提供 对已解析文件的访问，以及 将 part 暴露为 request 参数。



---

Apache Commons FileUpload

为了使用 apache commons 的 FileUpload，你可以配置一个 CommonsMultipartResolver 类型 && 名字是 multipartResolver 的 bean 。 你也需要有 commons-fileupload jar包。

这个解析器 委托 给 app中欧给你的 本地库，提供 跨Servlet 容器的 最大可移植性。

作为替代方案，考虑 通过容器自己的 parser 的 标准Servlet multipart 解析。

Commons FileUpload 通常 只 应用到 带`multipart/` Content-Type 的 POST。查看 CommonsMultipartResolver 的javadoc for 详细配置参数



---

Servlet 3.0

Servlet 3.0 multipart parsing 需要 通过 Servlet 容器配置 来激活：

1. 在java中，在Servlet 注册上 设置一个 MultipartConfigElement。
2. web.xml 中，增加 ``<multipart-confg>`` 到 servlet 声明

下面是如何设置 MultipartConfigElement 在 Servlet 注册上：

```java
public class AppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    // ...

    @Override
    protected void customizeRegistration(ServletRegistration.Dynamic registration) {

        // Optionally also set maxFileSize, maxRequestSize, fileSizeThreshold
        registration.setMultipartConfig(new MultipartConfigElement("/tmp"));
    }

}
```



一旦Servlet 3.0 配置ok，你可以添加一个 StandardServletMultipartResolver 类型的，multipartResolver名字的 bean。

这个解析器使用 Servlet 容器的 multipart parser，可能会将 app 暴露给 容器实现 差异。默认下，它将尝试 转换任何 content type 是 `multipart/` 的 任何 HTTP method，但这可能不是被 所有 Servlet 容器支持。查看 StandardServletMultipartResolver 的javadoc for 详细配置参数



---

1.1.13 Logging

MVC中的 DEBUG 级别日志 设计为 紧凑，最小，人性化。它侧重于 反复的 有用的 高价值信息，而不是仅 在调试特定问题时 有用的信息。

TRACE 级别 遵循和 DEBUG相同的原则，但可用于调试任何问题。



---

Senstitive  Data

debug 和trace 会打印很多 敏感信息。这就是为什么，请求参数 和header 默认 被屏蔽，必须通过 DispatcherServlet 上的 enableLoggingRequestDetails 属性来 启用。

```java
public class MyInitializer
        extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return ... ;
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return ... ;
    }

    @Override
    protected String[] getServletMappings() {
        return ... ;
    }

    @Override
    protected void customizeRegistration(ServletRegistration.Dynamic registration) {
        registration.setInitParameter("enableLoggingRequestDetails", "true");
    }

}
```



---

1.2 Filters

spring-web 模块提供了一些游泳的filter：

Form Data

Forwarded Headers

Shallow ETag

CORS



---

1.2.1 Form Data

浏览器只能通过 http get/post 提交 form data (表单数据)，但 非浏览器可以使用 http put patch delete。 Servlet api 需要 ServletRequest.getParameter*() 方法来 支持 仅对 http post 的表单字段访问。

spring-web 模块提供了 FormContentFilter 来 拦截 带有 application/x-www-form-urlencoded 的 http put,patch,delete 请求，从请求的body 读取 form data，包装ServletRequest 来 使得 表单数据 能通过 ServletRequest.getParameter*() 方法s 获得。



---

1.2.2 Forwarded Headers

当请求通过 代理(如Load balancer) 时，host，port,scheme 可能改变，这使得 从客户端的角度 创建 指向正确 host，port，scheme 的link 成为了一项挑战。

RFC 7239 定义了 Forwarded http header，代理可以使用这个 头 来提供 关于 原始请求的信息。还有其它非标准的 header，如 x-Forwarded-Host, x-Forwarded-Port, x-Forwarded-Proto，x-Forwarded-Ssl,X-Forwarded-Prefix。

ForwardedHeaderFilter 是一个 Servlet filter，修改 request 为了：

1. 基于 Forwarded 头 ，修改 host,post,scheme
2. 移除这些头 来 消除影响。

filter 依赖于 包装请求，因此 它必须在 其他filter 之前，比如 RequestContextFilter ，它适用于修改后的请求，而不是原始请求。

forwarded header 存在 安全考虑，因为 app 无法知道 header 是 proxy 添加的，还是 恶意客户端添加的。这就是为什么应该 配置 信任边界的 代理 以删除 来自外部的 不受信任的 Forwarded header。你也可以 用 removeOnly=true 来配置 FoewardedHeaderFilter ，这样 只会移除，而不会使用 header

为了支持 异步请求 和 错误分派，这个filter 应该映射到 DispatcherType.ASYNC 和 DispatcherTypeERROR。如果使用spring 框架的 AbstractAnnotationConfigDispatcherServletInitializer ，所有filter 都被自动注册 所有的 dispatcher type。但是 如果通过 web.xml  或 boot中通过 FilterRegistrationBean 注册 filter，确保 除了 DispatcherType.REQUEST 外 还需要 DispatcherType 的 ASYNC 和 ERROR。



---

1.2.3 Shallow( 肤浅，弱) ETag

ShallowEtagHeaderFilter 创建一个 "shallow" ETag 通过 缓存写入响应的内容 并计算MD5。 下次client 发送时，它执行相同的操作，但它还会将 计算值 与 If-None-Match 头比较，如果两者相等，则返回 304 ( NOT_MODIFIED)

这个策略 节约 带宽，不节约CPU，因为必须为 每个请求计算 完整的响应。controller 层的其他策略 可以避免计算，查看HTTP Caching

这个过滤器 有一个 writeWeakETag 参数，该参数 将过滤器配置 为写入类似于以下内容的 weak ETag： `W/"02a2d595e6ed9a0b24f027f2b63b134d6"` ，如 RFC 7232 中 2.3节定义的。

为了支持 异步请求，这个过滤器必须映射为 DispatcherType.ASYNC， 以便过滤器可以延迟 并成功生成一个 ETag 到 最后一个异步调度的末尾。 如果使用spring框架的 AbstractAnnotationConfigDispatcherServletInitializer， 所有filter 被自动注册 所有的 dispatch type。但是如果 通过 web.xml 或 boot的 FilterRegistrationBean 注册filter，则必须确保 包含 DispatcherType.ASYNC



---

1.2.4 CORS

mvc 通过 controller 的注解 为 CORS 提供 细粒度的配置。但是，当和spring security 一起使用时，我们建议 依赖的内置CoreFilter 必须在 spring security 过滤链 之前。



---

### 1.3 Annotated Controllers

mvc 提供了 基于注解的 编程模型，@Controller ，@RestController 组件 使用 注解 来表达 request mapping，request input，异常处理，等。

被注解的controller 有 灵活的方法签名，不必扩展 基类，也不必实现特定接口。

```java
@Controller
public class HelloController {

    @GetMapping("/hello")
    public String handle(Model model) {
        model.addAttribute("message", "Hello World!");
        return "index";
    }
}
```

上面的例子中，方法接受一个 Model ，返回一个 string类型的 视图名。



---

1.3.1 Declaration

你可以定义 controller bean 通过使用标准 spring bean 定义 ，在WebAppCtx 中。

@Controller stereotype(刻板印象，模式化观念，铅板，原型) 允许 自动检测，和spring 在classpath 中 detect @Component 类 并自动注册 bean定义 的 支持 保持一致。它还可以作为 注解类的 stereotype，表明它是一个 web component。

为了启用 @Controller bean 的自动发现，你可以增加 component scanning ：

```java
@Configuration
@ComponentScan("org.example.web")
public class WebConfig {

    // ...
}
```

xml 的等价：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.example.web"/>

    <!-- ... -->

</beans>
```



@RestController 是一个 复合注解，被注解了 @Controller 和 @ResponseBody 来指明 一个controller ，它的每个方法 都集成了 @ResponseBody 注解，因此，直接写入到 response body，而不是 通过 view 解析 来使用HTML 模板 来呈现。



---

AOP Proxies

有些情况下，你可能需要 在运行时 使用AOP 代理 来装饰 controller。一个例子是：如果你直接在controller 上 @Transactional。这种情况下，对于controller，我们建议使用 基 于 类 的代理。这通常是 controller 的 默认选择。但是如果 一个controller 必须 实现 一个 非Spring Context callback (InitializingBean，*Aware等) 的接口，你可能需要 显式 配置 基于类的 代理。例如 使用 `<tx:annotation-driven/>` 你可以修改为 `<tx:annotation-driven proxy-target-class="true" />` ，使用 @EnableTransactionManagement 你可以修改为 @EnableTransactionManagement(proxyTargetClass = true)。

。。spring 默认： 有任何一个接口，就 jdk代理， 没有任何接口 就 cglib。所以 一般情况下，controller 是没有接口的，所以是 cglib代理， 但有接口就 jdk了, 那些 @RequestMapping 方法 就不会被代理了，除非提取到接口中。但是ABCControllerImpl implements ABCController 。。好像真没有见过。说好的面向接口编程呢。。或者说 可以把 这种视为 配置，，确实是配置，@Controller 是一个组件， 只不过 很多情况下，这个组件 和 业务 耦合了。@RequestMapping 也是一种映射，(无)关业务



---

1.3.2 Request Mapping

你可以使用 @RequestMapping 注解 来 映射 请求到 controller 方法。 它有各种属性来匹配 URL，http方法，请求参数，头，media type。 你可以使用它 在 class 级别 来 表示 共享的 映射 (。。记得就是个公共前缀)，或者在方法级别 使用它来 缩小 到 特定的 端点映射。

@RequestMapping变体：

@GetMapping

@PostMapping

@PutMapping

@DeleteMapping

@PatchMapping

这些 快捷方式/shortcut 是自定义注解，被提供是因为：可以说，大部分controller 方法 应该被 映射到 特定的 HTTP method，而不是使用 @RequestMapping，这个默认会匹配所有的HTTP method。在类级别，依然需要使用 @RequestMapping 来表达 共享映射。

下面是 类型 和 方法级别的 mapping

```java
@RestController
@RequestMapping("/persons")
class PersonController {

    @GetMapping("/{id}")
    public Person getPerson(@PathVariable Long id) {
        // ...
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public void add(@RequestBody Person person) {
        // ...
    }
}
```



---

URI patterns

@RequestMapping 注解的 方法 能通过 URL pattern 来映射。2种选择：

1. PathPattern：与URL 进行匹配的 pre-parsed pattern 也被 pre-parsed 为 PathContainer。为web使用而设计，这个方法 有效解决 encoding 和 path parameters，比有效地匹配。
2. AntPatMatcher：通过 String pattern 与 字符串路径 进行匹配。这是 最初方案 用在spring 配置中， 来选择 classpath，filesystem，其他位置 上的 资源。效率低，并且 string path 输入 是一个挑战 for 有效地处理 URL encoding 和 其他问题



PathPattern 是 web app 的推荐方案， 它在 Spring WebFlux 中 是 唯一选择。5.3之前，AntPathMatcher 是mvc的唯一选择，并且 现在 还是 默认设置。

PathPattern 支持 AntPathMatcher 一样的 pattern 语法。此外，它还支持 capturing pattern/捕获模式，比如 `{*string}`，用于 匹配 path 末尾的 0或多个 path段。PathPattern 也限制了 `**` 的使用 for 匹配多个路径段，导致 它只被允许 在 pattern的末尾。这个消除了 许多模棱两可的情况，当 为给定的请求 选择最佳匹配 pattern时。

一些例子：

`/resources/img?e.png`，匹配 一个path段的 一个char

`/resurces/*.png`，匹配一个path段的 0 或多个 char 

`/resources/**`，匹配多个 path segement

`/projects/{project}/version`，匹配 一个path 段，捕获它作为一个 变量

`/projects/{project:[a-z]+}/versions`， 使用 正则 来匹配 和 捕获 变量。



被捕获的 URI 变量 通过 @PathVariable 来 访问：

```java
@GetMapping("/owners/{ownerId}/pets/{petId}")
public Pet findPet(@PathVariable Long ownerId, @PathVariable Long petId) {
    // ...
}
```



你可以声明URI 变量 在 类和 方法级别：

```java
@Controller
@RequestMapping("/owners/{ownerId}")
public class OwnerController {

    @GetMapping("/pets/{petId}")
    public Pet findPet(@PathVariable Long ownerId, @PathVariable Long petId) {
        // ...
    }
}
```



> URI 变量 被自动 转为 合适的 类型，或者 TypeMismatchException 抛出。

简单类型(int, long, Date 等) 被默认支持，你可以注册 其他类型的 支持。 查看 Type Conversion章节 和 DataBinder 类。

> 你可以显式命名 URI 变量 (如 @PathVariable("customId") )，如果 名称相同，且你的代码编译时 使用了 debug 信息，或 使用了Java8的 -parameters 编译flag， 你可以省略。

语法 `{varName:regex}` 声明了 URI 变量 with 一个 具有 {varName:regex} 语法的 RE (regular expression) 。例如，给定URL `/spring-web-3.0.5.jar` ，下面的方法 提取了 name，version，文件扩展名：

```java
@GetMapping("/{name:[a-z-]+}-{version:\\d\\.\\d\\.\\d}{ext:\\.[a-z]+}")
public void handle(@PathVariable String name, @PathVariable String version, @PathVariable String ext) {
    // ...
}
```

。。。。

> URI path pattern 也可以 嵌入 ${...} 占位符

这些占位符 在启动时 解析 by 使用 PropertyPlaceHolderConfigurer  对 local,system,environment,和其他 property source 。 例如，你可以使用这个 来根据外部配置 参数化 URL。



---

Pattern Comparison ( 比较)

当多个 pattern 匹配 一个URL时， 必须选出一个 最合适的。这使用下面2种方法之一 来做到，选哪种方法 取决于 是否 可以使用 parsed PathPattern。

1. PathPattern.SPECIFICITY_COMPARATOR。。。 这个是 Comparator 对象。
2. AntPathMatcher.getPatternComparator 。。。这个是方法，形参 String path，返回一个Comparator 对象。

两种都提供 帮助 来 sort pattern，更具体的 在上面。 一个pattern 更 不 具 体( 特定性低)，如果 它 的 URI变量(记为1)，单个通配符( 计为1)，双通配符(计为2 ) 数量 更少。

> 如果分数相同，选择长的pattern；如果分数，长度 都相同，则选择 URI变量 多于 通配符 的 pattern。



默认的 映射pattern ( `/**` )  被排除在 评分之外，并且始终 排在最后，此外 前缀模式 ( 如 `/public/**` ) 被认为比 没有双通配符的 其他模式 更不具体。

更多信息，看那 2种方法的 javadoc



---

Suffix Match

从5.3 开始，默认下，mvc 不在 执行 `.*` 的后缀 pattern 匹配，其中 映射到 `/person` 的controller 也隐式 映射到 `/person.*` 。因此，不再使用 path 扩展 来解释 请求的 响应内容类型，如 `/person.xml`, `/person.pdf` 等。

这种 文件扩展名 是必要的， 当浏览器 发送了 Accept header，但 难以一致解释。目前，这不再是一个 必须项，使用 Accept 应该是 首选。

随着时间推移，文件扩展名的 使用 已经被证明了存在 多种问题。 它可以导致 歧义 当 使用URI变量，path parameters，URI encoding 覆盖( overlain )时。基于URL 推理 授权 和 安全性 也变得更加困难。

为了在5.3 之前的 版本中 ，彻底禁用 path 扩展：

1. PathMatcherConfigurer 的 useSuffixPatternMatching( false )
2. ContentNegotiationConfigurer 的 favorPathExtension ( false )

除了"Accept" 头外，还有一种 很有用的 方法 来 获得 content type，例如在浏览器中输入 URL时。路径扩展的 安全的替代方案是 使用 query parameter 策略。 如果你必须使用 文件扩展名，请考虑通过 ContentNegotiationConfigurer 的mediaTypes 属性 限制它们 为 显式注册的 扩展名列表。



---

Suffix Match and RFD

reflected file download (RFD) 攻击 类似 XSS，因为它 依赖于响应中 反映的 请求输入 ( 例如，查询参数 和 URI 变量)。但是 RFD 攻击不是将 JS 插入到HTML，而是依赖于 浏览器 切换 来 执行下载，并在稍后 双击时 将响应是为可执行脚本。

mvc中，@ResponseBody 和 ResponseEntity 方法 存在风险，因为它们可以呈现不同的内容类型。客户端可以通过 URL path extension 来 请求。禁用 后缀 pattern匹配，并 使用 path extension 进行 内容协商 会降低风险，但不足以防止 RFD 攻击。

为了防止 RFD 攻击，在渲染 响应正文之前，spring mvc 添加一个 Content-Disposition:inline;filename=f.txt 头 来 建议/表明 一个固定 且安全的下载文件。仅当URL 路径包含 即不安全 也不明确注册 用于内容协商的文件扩展名时，才会这样做。但是，当直接浏览器中输入URL时，它可能会产生副作用。

默认下，许多常见的路径扩展都是安全的。具有 自定义 HttpMessageConverter 实现的 app 可以显式注册 文件扩展名 以进行内容协商，以避免为这些扩展名 添加 Content-Disposition header。



---

Consumable Media Types

你可以根据 请求的 Content-Type 缩小 请求映射：

```java
@PostMapping(path = "/pets", consumes = "application/json") 
public void addPet(@RequestBody Pet pet) {
    // ...
}
```

consumes 属性 也支持 否定表达式，如 `!text/plain` 意味 除了 `text/plain` 的任何 content type。

你可以声明一个 共享的 consumes 在 类 级别。但是，不像 大部分其他 request-mapping 属性，当它被使用在 class 级别时，方法级别的 consumes 属性 覆盖 而不是 扩展 类级别 的声明。

。。consumes 是一个 String[]



MediaType 提供了常量 for 常用 media type，如 APPLICATION_JSON_VALUE, APPLICATION_XML_VALUE。



---

Producible Media Type

你可以缩小 request mapping 根据 Accept 请求头 和 controller 方法 注解的 produces 属性的列表。

```java
@GetMapping(path = "/pets/{petId}", produces = "application/json") 
@ResponseBody
public Pet getPet(@PathVariable String petId) {
    // ...
}
```

media type 可以指定 一个 character set。支持否定表达式。

你可以在 类级别声明 共享的 produces 属性，并且 方法级别的 produces 会 覆盖 而不是 扩展 类级别的 produces ， request-mapping 的很多 其他属性 是 扩展。



---

parameters， headers

你可以缩小 request mapping 根据 request parameter condition。你可以测试 请求参数 (myParam) 是否存在，是否不存在 (!myParam) 或 特定值 (myParam=myValue) 。

```java
@GetMapping(path = "/pets/{petId}", params = "myParam=myValue") 
public void findPet(@PathVariable String petId) {
    // ...
}
```

测试 myParam 是否等于 myValue

```java
@GetMapping(path = "/pets", headers = "myHeader=myValue") 
public void findPet(@PathVariable String petId) {
    // ...
}
```

这里是 headers 属性。 上面是 params

你可以在 headers condition 中 匹配 Content-Type, Accept， 但是 使用 consumes，produces 更好。



---

HTTP DEAD, OPTIONS

@GetMapping ( 和 @RequestMapping(method=HttpMethod.GET) ) 透明地支持 http HEAD 以进行 request mapping。controller 方法不需要修改。在 javax.xx.xx.HttpServlet 中应用的 响应包装器 确保将 Content-Length 头 设置为 写入的字节数 ( 而不会 真正的响应)。

。。http HEAD 就是 不需要服务器 返回 body。。其他都要。

@GetMapping( 和 同上) 被隐式 映射 到 并支持 HTTP HEAD。一个 http HEAD 请求 被处理 就像 它是一个 http get， 除了：不写入正文，而是 计算字节数， 并设置 Content-Length 头。

。。。第一段是 透明地/transparently，这里是 隐式/implicitly。。其他内容重复的啊。

默认，http OPTIONS 被处理 by 设置 Allow 响应 头 为 匹配URL pattern的 所有 @RequestMapping 方法的 http method 列表。

对于没有 http method 声明的 @RequestMapping， Allow 头设置 get,head,post,put,patch,delete,options。controller 方法应该 始终声明 支持的 http method (例如，通过 @GetMapping， @PostMapping)

你可以显式 映射 @RequestMapping 方法 到 http head 和 http options， 但这在常见情况下 不是必须的。



---

Custom Annotations

mvc 支持使用 组合注解 for request mapping。 这些注解本身就是用 @RequestMapping 进行 meta-注解的，并组合 成重新声明 @RequestMapping 属性的 子集(或全部)， 具有更窄，更具体的 用途。

@GetMapping,@PostMapping,@PutMapping,@DeleteMapping,@PatchMapping 之组合注解的 例子。

mvc 还支持 具有 自定义 request-mapping 逻辑的 自定义request-mapping属性。这是更高级的选项，需要继承 RequestMappingHandlerMapping 并覆盖 getCustomMethodCondition 方法。你可以在其中 检查自定义属性 并返回你自己的 RequestCondition。



---

Explicit Registrations 显式注册

你可以以 编程方式 注册 handler 方法，你可以将其用于 动态注册 或高级案例，比如 不同URL 下 同一个 handler 的 不同实例。 下面注册了一个 handler 方法：

```java
@Configuration
public class MyConfig {

    @Autowired
    public void setHandlerMapping(RequestMappingHandlerMapping mapping, UserHandler handler) 
            throws NoSuchMethodException {   // 注入目标handler 和 controller 的handler mapping

        RequestMappingInfo info = RequestMappingInfo
                .paths("/user/{id}").methods(RequestMethod.GET).build();   // 准备 request mapping 元数据

        Method method = UserHandler.class.getMethod("getUser", Long.class);   // 获得 handler 方法

        mapping.registerMapping(info, handler, method);   // 增加注册
    }
}
```



---

1.3.3 Handler Methods

@RequestMapping handler 方法 有一个 灵活的 签名 且 能从一系列 受支持的 controller 方法 参数 和 返回值 中进行选择。



---

Method Arguments

下表显示了 受支持的 controller 方法参数，任何参数都不支持 响应式类型。

jdk8 的 java.util.Optional 被支持为 方法参数 与 具有`required` 属性的 注解( 如 @RequestParam， @RequestHeader ) 结合使用，并等效于 required=false。



。。。有点多。。

| Controller method argument                                   | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `WebRequest`, `NativeWebRequest`                             | 通用的 对 request parameters 和 request 和 session attributes 的 访问，不需要直接使用 Servlet API |
| `javax.servlet.ServletRequest`, `javax.servlet.ServletResponse` | 选择任何 特定的 request 或 response 类型，例如ServletRequest, HttpServletRequest,或spring 的 MultipartRequest, MultipartHttpServletRequest。 |
| `javax.servlet.http.HttpSession`                             | 强制 session 的存在。因此，这个参数 永远不会 null。注意，session 访问 不是 threa-safe 的。 如果多个request 被允许 并发访问 session， 考虑将 RequestMappingHandlerAdapter 实例的 synchronizeOnSession 设置为 true。 |
| `javax.servlet.http.PushBuilder`                             | Servlet 4.0 的 push builder api for 编程的方式 push HTTP/2 的资源。注意，根据Servlet 规范，如果客户端不支持 HTTP/2 功能，则注入的 PushBuilder 实例 可以为空。 |
| `java.security.Principal`                                    | 当前 经过身份验证的 用户 - 如果已知，可能是 特定的 Principal 实现类。        注意，如果这个参数 不会立刻被解析，如果它被注解 以允许 自定义解析器 来解析它 before 回退到默认的解析 通过 HttpServletRequest#getUserPrincipal。 例如，spring security authrntication 实现了 Principal 并且通过将 HttpServletRequest#getUserPrincipal注入，除非 它也使用 @AuthenticationPrincipal 注解，这种情况下，它由自定义的 spring security 解析器 通过 Authentication#getPrincipal 解析。。。？？？。。。 |
| `HttpMethod`                                                 | The HTTP method of the request。                             |
| `java.util.Locale`                                           | 当前请求的 语言环境，通过可用的 最 明确的 可用的 LocaleResolver 来决定。 (实际上，配置的 LocaleResovler 或 LocaleContextResolver ) |
| `java.util.TimeZone` + `java.time.ZoneId`                    | 当前request 关联的 时区，被 LocaleContextResolver 决定。     |
| `java.io.InputStream`, `java.io.Reader`                      | 用来访问 Servlet API 公开的 raw request body。               |
| `java.io.OutputStream`, `java.io.Writer`                     | 用来访问 Servlet API 公开的 raw response body。              |
| `@PathVariable`                                              | 用来访问 URI 模板 变量。                                     |
| `@MatrixVariable`                                            | 用于访问 URI 路径段 中的 name-value 对。                     |
| `@RequestParam`                                              | 用于 访问 Servlet 请求参数，包括 multipart 文件。参数值 被转换为 方法的形参的类型。  注意，@RequestParam 对于简单 参数值 是可选的。 |
| `@RequestHeader`                                             | For access to request headers. Header values are converted to the declared method argument type. See [`@RequestHeader`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestheader). |
| `@CookieValue`                                               | For access to cookies. Cookies values are converted to the declared method argument type. See [`@CookieValue`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-cookievalue). |
| `@RequestBody`                                               | For access to the HTTP request body. Body content is converted to the declared method argument type by using `HttpMessageConverter` implementations. See [`@RequestBody`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestbody). |
| `HttpEntity<B>`                                              | For access to request headers and body. The body is converted with an `HttpMessageConverter`. See [HttpEntity](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-httpentity). |
| `@RequestPart`                                               | For access to a part in a `multipart/form-data` request, converting the part’s body with an `HttpMessageConverter`. See [Multipart](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-multipart-forms). |
| `java.util.Map`, `org.springframework.ui.Model`, `org.springframework.ui.ModelMap` | For access to the model that is used in HTML controllers and exposed to templates as part of view rendering. |
| `RedirectAttributes`                                         | 指定 要在重定向情况下 使用的属性 (即，被附加到query string 的) 和 要临时存储的 flash 属性 直到重定向完成后的 请求。    Specify attributes to use in case of a redirect (that is, to be appended to the query string) and flash attributes to be stored temporarily until the request after redirect. See [Redirect Attributes](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-redirecting-passing-data) and [Flash Attributes](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-flash-attributes). |
| `@ModelAttribute`                                            | For access to an existing attribute in the model (instantiated if not present) with data binding and validation applied. See [`@ModelAttribute`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-modelattrib-method-args) as well as [Model](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-modelattrib-methods) and [`DataBinder`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-initbinder). Note that use of `@ModelAttribute` is optional (for example, to set its attributes). See “Any other argument” at the end of this table. |
| `Errors`, `BindingResult`                                    | For access to errors from validation and data binding for a command object (that is, a `@ModelAttribute` argument) or errors from the validation of a `@RequestBody` or `@RequestPart` arguments. You must declare an `Errors`, or `BindingResult` argument immediately after the validated method argument. |
| `SessionStatus` + class-level `@SessionAttributes`           | For marking form processing complete, which triggers cleanup of session attributes declared through a class-level `@SessionAttributes` annotation. See [`@SessionAttributes`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-sessionattributes) for more details. |
| `UriComponentsBuilder`                                       | For preparing a URL relative to the current request’s host, port, scheme, context path, and the literal part of the servlet mapping. See [URI Links](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-uri-building). |
| `@SessionAttribute`                                          | For access to any session attribute, in contrast to model attributes stored in the session as a result of a class-level `@SessionAttributes` declaration. See [`@SessionAttribute`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-sessionattribute) for more details. |
| `@RequestAttribute`                                          | For access to request attributes. See [`@RequestAttribute`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestattrib) for more details. |
| Any other argument                                           | If a method argument is not matched to any of the earlier values in this table and it is a simple type (as determined by [BeanUtils#isSimpleProperty](https://docs.spring.io/spring-framework/docs/5.3.16/javadoc-api/org/springframework/beans/BeanUtils.html#isSimpleProperty-java.lang.Class-), it is resolved as a `@RequestParam`. Otherwise, it is resolved as a `@ModelAttribute`. |



---

Return Values

下表描述了 支持的 controller 方法的 返回值。所有 返回值都支持 Reactive 类型

| Controller method return value                               | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `@ResponseBody`                                              | The return value is converted through `HttpMessageConverter` implementations and written to the response. See [`@ResponseBody`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-responsebody). |
| `HttpEntity<B>`, `ResponseEntity<B>`                         | The return value that specifies the full response (including HTTP headers and body) is to be converted through `HttpMessageConverter` implementations and written to the response. See [ResponseEntity](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-responseentity). |
| `HttpHeaders`                                                | For returning a response with headers and no body.           |
| `String`                                                     | A view name to be resolved with `ViewResolver` implementations and used together with the implicit model — determined through command objects and `@ModelAttribute` methods. The handler method can also programmatically enrich the model by declaring a `Model` argument (see [Explicit Registrations](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestmapping-registration)). |
| `View`                                                       | A `View` instance to use for rendering together with the implicit model — determined through command objects and `@ModelAttribute` methods. The handler method can also programmatically enrich the model by declaring a `Model` argument (see [Explicit Registrations](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestmapping-registration)). |
| `java.util.Map`, `org.springframework.ui.Model`              | Attributes to be added to the implicit model, with the view name implicitly determined through a `RequestToViewNameTranslator`. |
| `@ModelAttribute`                                            | An attribute to be added to the model, with the view name implicitly determined through a `RequestToViewNameTranslator`. Note that `@ModelAttribute` is optional. See "Any other return value" at the end of this table. |
| `ModelAndView` object                                        | The view and model attributes to use and, optionally, a response status. |
| `void`                                                       | A method with a `void` return type (or `null` return value) is considered to have fully handled the response if it also has a `ServletResponse`, an `OutputStream` argument, or an `@ResponseStatus` annotation. The same is also true if the controller has made a positive `ETag` or `lastModified` timestamp check (see [Controllers](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-caching-etag-lastmodified) for details). If none of the above is true, a `void` return type can also indicate “no response body” for REST controllers or a default view name selection for HTML controllers. |
| `DeferredResult<V>`                                          | Produce any of the preceding return values asynchronously from any thread — for example, as a result of some event or callback. See [Asynchronous Requests](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async) and [`DeferredResult`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-deferredresult). |
| `Callable<V>`                                                | Produce any of the above return values asynchronously in a Spring MVC-managed thread. See [Asynchronous Requests](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async) and [`Callable`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-callable). |
| `ListenableFuture<V>`, `java.util.concurrent.CompletionStage<V>`, `java.util.concurrent.CompletableFuture<V>` | Alternative to `DeferredResult`, as a convenience (for example, when an underlying service returns one of those). |
| `ResponseBodyEmitter`, `SseEmitter`                          | Emit a stream of objects asynchronously to be written to the response with `HttpMessageConverter` implementations. Also supported as the body of a `ResponseEntity`. See [Asynchronous Requests](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async) and [HTTP Streaming](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-http-streaming). |
| `StreamingResponseBody`                                      | Write to the response `OutputStream` asynchronously. Also supported as the body of a `ResponseEntity`. See [Asynchronous Requests](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async) and [HTTP Streaming](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-http-streaming). |
| Reactive types — Reactor, RxJava, or others through `ReactiveAdapterRegistry` | Alternative to `DeferredResult` with multi-value streams (for example, `Flux`, `Observable`) collected to a `List`. For streaming scenarios (for example, `text/event-stream`, `application/json+stream`), `SseEmitter` and `ResponseBodyEmitter` are used instead, where `ServletOutputStream` blocking I/O is performed on a Spring MVC-managed thread and back pressure is applied against the completion of each write. See [Asynchronous Requests](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async) and [Reactive Types](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-async-reactive-types). |
| Any other return value                                       | Any return value that does not match any of the earlier values in this table and that is a `String` or `void` is treated as a view name (default view name selection through `RequestToViewNameTranslator` applies), provided it is not a simple type, as determined by [BeanUtils#isSimpleProperty](https://docs.spring.io/spring-framework/docs/5.3.16/javadoc-api/org/springframework/beans/BeanUtils.html#isSimpleProperty-java.lang.Class-). Values that are simple types remain unresolved. |



---

Type Conversion

一些表示 基于string 的请求输入 的 带注解的 controller 方法 参数 ( 如 @RequestParam，@RequestHeader，@PathVariable, @MatrixVariable, @CookieValue ) 。如果参数 声明 为 非string 类型，则 可能 需要 类型转换。

对于这些例子，类型转换 被自动 应用 基于 配置的 converter。 默认下，简单类型 ( int，long , Data 等) 被支持。你可以自定义类型转换 通过 WebDataBinder 或 注册 Formatters 通过 FormattingConversionService。

类型转换的 一个问题是，空string 的 处理。如果 类型转换的结果 是null，那么这个值 被认为是 缺失。Long ，UUID 和其他目标类型 可能就是这种情况。 如果要允许注入 null，请在参数 注解上 使用 required 标志，或声明 参数 是 @Nullable 。

5.3开始，非null 实参 会被强制( 。。要求)，即使是 类型转换后。如果你的 handler方法 也可以接受null值，那么 要么 声明参数为@Nullable 或  在对应的 @RequestParam 中标记required=false 。

或者，你可以进行专门的处理， 比如 @PathVariable 产生的 MissingPathVariableException 的问题。类型转换后的 null 会被是为 空原始值，因此会抛出 相应的 Missing...Exception。



---

Matrix Variables

RFC 3986 讨论了 path段中的 name-value 对。在mvc中，我们根据 Tim Berners-Lee 的 "旧帖子" 将它们称为 "matrix variables"，但它们也可以被称为 URI 路径参数。

矩阵变量可以出现在 任何 路径段中，每个变量 用 分号 相隔，多个值 用 逗号相隔 ( 如 `/cars;color=red,green;year=2012`)。 multiple value 也可以通过 重复的 变量名称 来指定 ( 如 `color=red;color=green;color=blue` )

如果URL 预期包含 矩阵变量，则 controller 方法 的 request mapping 必须使用 URI 变量 来 mask(隐藏 ) 这个变量 内容，并 确保 请求 能被匹配成功，而且 与 矩阵变量顺序和 存在 无关。

```java
// GET /pets/42;q=11;r=22

@GetMapping("/pets/{petId}")
public void findPet(@PathVariable String petId, @MatrixVariable int q) {

    // petId == 42
    // q == 11
}
```

所有的路径段 有可以包含 矩阵变量，你有时需要 分辨 矩阵变量是哪段的。

```java
// GET /owners/42;q=11/pets/21;q=22

@GetMapping("/owners/{ownerId}/pets/{petId}")
public void findPet(
        @MatrixVariable(name="q", pathVar="ownerId") int q1,
        @MatrixVariable(name="q", pathVar="petId") int q2) {

    // q1 == 11
    // q2 == 22
}
```

矩阵变量可以 被定义为 可选的，且 指定默认值

```java
// GET /pets/42

@GetMapping("/pets/{petId}")
public void findPet(@MatrixVariable(required=false, defaultValue="1") int q) {

    // q == 1
}
```

获得所有 矩阵变量，你可以使用 MultiValueMap 

```java
// GET /owners/42;q=11;r=12/pets/21;q=22;s=23

@GetMapping("/owners/{ownerId}/pets/{petId}")
public void findPet(
        @MatrixVariable MultiValueMap<String, String> matrixVars,
        @MatrixVariable(pathVar="petId") MultiValueMap<String, String> petMatrixVars) {

    // matrixVars: ["q" : [11,22], "r" : 12, "s" : 23]
    // petMatrixVars: ["q" : 22, "s" : 23]
}
```



你需要 启用 矩阵变量的 使用。mvc java配置中，你需要设置一个 带有removeSemicolonContent=false 的UriPathHelper 通过 Path Matching。 mvc xml配置中，你可以设置 `<mvc:annotation-driven enable-matrix-variables="true"/>`



---

@RequestParam

你可以使用 @RequestParam 注解 来绑定 Servlet request parameters ( 即， 查询参数 或 表单数据) 到 controller 的 方法参数。

```java
@Controller
@RequestMapping("/pets")
public class EditPetForm {

    // ...

    @GetMapping
    public String setupForm(@RequestParam("petId") int petId, Model model) { 
        Pet pet = this.clinic.loadPet(petId);
        model.addAttribute("pet", pet);
        return "petForm";
    }

    // ...

}
```

默认下，被注解的 方法参数 是 required 。不可以指定 可选，通过设置 @RequestParam 的 required 为false。 或 在 Optional 中声明 参数。

如果参数类型不是 string，类型转换被自动应用。

声明参数类型为 数组 或list 允许 解析 相同参数name 的 多个参数值。

当 @RequestParam 注解到 `Map<String, String>` 或 `MultiValueMap<String, String>` ，如果在注解中没有指定参数名字，则map 将填充每个参数名称的 请求参数值。

> 注意 @RequestParam 是可选的。默认下，任何 简单类型的( 通过BeanUtils#isSimpleProperty 决定) 参数，且没有 被其他 参数解析器 解析， 会被视为 带有 @RequestParam 注解



---

@RequestHeader

绑定 request header 到 controller 的 方法 参数

考虑下面的header

```
Host                    localhost:8080
Accept                  text/html,application/xhtml+xml,application/xml;q=0.9
Accept-Language         fr,en-gb;q=0.7,en;q=0.3
Accept-Encoding         gzip,deflate
Accept-Charset          ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive              300
```

下面的例子获得 Accept-Encoding 和 Keep-Alive 头

```java
@GetMapping("/demo")
public void handle(
        @RequestHeader("Accept-Encoding") String encoding, 
        @RequestHeader("Keep-Alive") long keepAlive) { 
    //...
}
```

如果方法参数类型不是 string，自动进行 类型转换。

当 @RequestHeader 被注解到 `Map<String, String>` 或 `MultiValueMap<String, String>` 或 HttpHeaders 参数，会被填充所有 header值。

内置的支持可以 转换 逗号分隔 的string 到 数组 或 集合 of string 或其他 类型转换 知道的类。例如，如果方法参数被注解了 @RequestHeader("Accept") ，这个参数可以是 string 也可以是 String[], `List<String>`。



---

@CookieValue

绑定 HTTP Cookie 到 controller 的方法参数。

考虑request 有如下 cookie

```
JSESSIONID=415A4AC178C59DACE0B2C9CA727CDD84
```

下面是 获得 cookie 值

```java
@GetMapping("/demo")
public void handle(@CookieValue("JSESSIONID") String cookie) { 
    //...
}
```

如果方法参数类型不是string ，自动进行类型转换。



---

@ModelAttribute

注解到 方法参数上 来访问 model中的 属性 或 如果它不存在就实例化。model 属性 还 覆盖 名字和 字段名称匹配的 http servlet 请求参数的值。这称为 数据绑定，它使你不必处理 解析 和转换 单个查询参数 和 表单字段。

```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@ModelAttribute Pet pet) {
    // method logic...
}
```

Pet 实例 来源于 以下几个方式之一：

1. 从可能由 @ModelAttibute 方法添加的 model 中检索。
2. 如果 模型属性 被列举在 类级别 的 @SessionAnnotation 中，那么 从 http session 中检索。
3. 通过 Converter 获得，其中 模型属性 名称 与request value (如 路径变量和请求参数) 的名字 匹配，
4. 使用默认构造器实例化。
5. 通过 带有 与servlet 请求参数 匹配参数 的 "primary 构造器" 实例化。参数名字 通过 JavaBeans @ConstructorProperties 或 字节码 中运行时保留的 参数名称 确定。

使用 @ModelAttribute 方法 提供它 或 依赖框架创建 模型属性的 一种替代方案 是 使用 `Converter<String, T>` 来提供实例。这被应用 当模型属性名字 和 request 值的名字( 如 路径变量 或 请求参数 ) 匹配 ，并且 有一个 Converter 来进行 string 到 模型属性类型 的转换。

下面的例子中，模型属性名称account ,与 URI path 变量 account 匹配，有一个注册过的 `Converter<String, Account>` 这个能用来 加载 Account。

```java
@PutMapping("/accounts/{account}")
public String save(@ModelAttribute("account") Account account) {
    // ...
}
```

在模型实例 获得后，进行类型绑定。

WebDataBinder 将 Servlet请求参数名称 ( 查询参数 和表单数据 ) 与目标 Object 上的 字段名称 进行匹配。在必要时，使用 类型转换后的 值 进行填充。更多关于 类型绑定( 和 校验) 查看 Validation。 更多自定义data binding，看 DataBinder。

数据绑定可能导致 error， 默认，抛出 BindException。但是为了 检查这些error 在 controller 方法中，你可以增加一个 BindingResult 参数，***紧跟着*** @ModelAttribute 

```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@ModelAttribute("pet") Pet pet, BindingResult result) { 
    if (result.hasErrors()) {
        return "petForm";
    }
    // ...
}
```



有时，你可能想要访问一个 model attribute 不需要 data binding。这种情况下， 你可以注入 Model 到 controller，且直接访问， 或 设置 `@ModelAttribute(binding=false)`。

```java
@ModelAttribute
public AccountForm setUpForm() {
    return new AccountForm();
}

@ModelAttribute
public Account findAccount(@PathVariable String accountId) {
    return accountRepository.findOne(accountId);
}

@PostMapping("update")
public String update(@Valid AccountForm form, BindingResult result,
        @ModelAttribute(binding=false) Account account) { 
    // ...
}
```

。。主要是 前2个方法 是什么？第三个方法的 第一个参数 是怎么注入的？ 就是查询 setUpXXX ？

你可以自动 应用 校验 after 数据绑定 by 增加 javax.xx.Valid 注解 或 spring 的@Validated 。

```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@Valid @ModelAttribute("pet") Pet pet, BindingResult result) { 
    if (result.hasErrors()) {
        return "petForm";
    }
    // ...
}
```

注意，使用 @ModelAttirbute 是可选的，默认下，任何 ***非*** 简单 类型(通过 BeanUtils#isSimpleProperty 决定) 的 参数，并且不能其他 参数解析器 解析 ， 会被 认为 被注解了 @ModelAttribute



---

@SessionAttributes

用于在 请求之间的 http servlet session 中存储 模型属性。 是类级别注解，用于声明 特定控制器使用的 session 属性。这通常会 列出 模型属性的名字或 模型属性的类型，这些属性 应该 透明地 存储到 session 中，供后续请求 访问。

```java
@Controller
@SessionAttributes("pet") 
public class EditPetForm {
    // ...
}
```

第一个请求，当一个名为 pet 的模型属性 被添加到 模型中时，它会自动提升/推进(promote) 并保存在 http servlet session 中。它一直存在，直到 另一个 控制器 使用 SessionStatus 方法参数 来清楚 存储：

```java
@Controller
@SessionAttributes("pet") 		// store
public class EditPetForm {

    // ...

    @PostMapping("/pets/{id}")
    public String handle(Pet pet, BindingResult errors, SessionStatus status) {
        if (errors.hasErrors) {
            // ...
        }
        status.setComplete(); 		// clear
        // ...
    }
}
```



---

@SessionAttribute

如果你需要访问 全局管理的 预先存在的 session 属性( 即，在控制器外部--如，通过filter)， 且 可能存在也可能不存在， 你可以在 方法参数上使用 @SessionAttribute 注解。

```java
@RequestMapping("/")
public String handle(@SessionAttribute User user) { 
    // ...
}
```

对于需要增加或移除 session 属性的情况，考虑注入 org.springframework.xxx.WebRequest 或 javax...HttpSession 到 控制器方法。

对于在session 中临时存放的 作为 controller workflow一部分 的 模型属性，考虑使用 @SessionAttirbutes。(。。这个 多个 s )



---

@RequestAttribute

类似 @SessionAttribute， 你可以使用这个注解 来 访问 预先存在的 request attribute ( 比如通过 Servlet Filter 或 HandlerInterceptor 创建的)

```java
@GetMapping("/")
public String handle(@RequestAttribute Client client) { 
    // ...
}
```



---

Redirect Attributes

默认，所有 模型属性 都被视为 在 重定向URL 中 作为 URI模板变量公开。在剩余的属性中，那些是 原始类型 或 原始类型的集合或数组的 属性会被自动附加为 查询参数。

如果为重定向 准备了 模型实例，则将原始类型属性作为 查询参数 可能是想要的结果。但，在被注解的控制器中，模型可以包含 为呈现而添加的 其他属性 ( 如 下啦字段值)。为了避免此类 属性出现在 URL 中，@RequestMapping 可以声明 RedirectAttributes 类型的参数，并使用它来 指定 可用于 RedirectView 的确切属性。如果方法 执行重定向，RedirectAttributes 的内容 被使用。否则，model 的内容 被使用。

RequestMappingHandlerAdapter 提供了一个 名为 ignoreDefaultModelOnRedirect 的标志，你可以使用这个来指示：如果controller 方法 重定向，则不应该使用默认Model的内容。相反，controller 方法应该声明 RedirectAttributes 类型 的属性，或者，如果不这样做，则不应该将 任何属性传递给 RedirectView。mvc命名空间和 mvc java配置 都将这个 标志设置为false，以保持向后兼容。但是 对于新应用，我们建议设置为 true。

注意，当前请求的 URI 模板变量 在 扩展重定向URL 时 自动可用，你无需通过 Model 或 RedirectAttributes 显式添加它们。

```java
@PostMapping("/files/{path}")
public String upload(...) {
    // ...
    return "redirect:files/{path}";
}
```

另一种 传递数据 到 重定向目标的方法 是 使用 flash attributes。不同于 其他 重定向属性，flash 属性 被保存在 http session中 ( 因此，不会出现在 URL 中)



---

Flash Attributes

flash attributes 提供了 一个方法 for 一个请求存储 那些 打算在 另一个请求中 使用的 属性。这在 重定向时最常需要的--例如，Post-Redirect-Get 模式。flash属性 在重定向之前 ( 通常在session中) 临时保存，以便在重定向之后 对请求可用 并立即删除。

mvc 有2个主要的抽象 来支持 flash 属性。FlashMap用于保存flash 属性，FlashMapManager 用来 保存，检索，管理 FlashMap 实例。

flash attribute 的支持 总是 打开的，且 不需要 显式启用。但，如果不使用，它永远不会导致 http session创建。在每个请求上，都有一个 "input" FlashMap，其属性从 前一个请求 (如果有) 传递过来，还有一个 "ouput" FlashMap，其保存 后续请求的属性。这2个 FlashMap 都可以通过 RequestContextUtils 中的 静态方法 从 spring mvc 的 任何位置 访问。

被注解的控制器 通常不需要 直接使用 FlashMap。相反，@RequestMapping 方法可以接受 RedirectAttributes 类型的 参数，并使用它 为重定向场景 添加 flash属性。通过 RedirectAttributes 添加的 flash 属性 会自动 传播到 "output" FlashMap。类似的，在重定向之后，来自 "input" FlashMap 的属性 会自动添加到 为 目标URL 提供服务的 controller 的Model。



Matching requests to flash attributes

flash 属性的概念 存在于 许多其他 web 框架中，并且 被证明 有时会遇到并发问题。这是因为，根据定义，flash属性 被存储 直到 下一个请求。然后 下一个请求 可能不会预期的接受者，而是另一个 异步请求 (例如，轮询 或 资源请求)，这种情况下， flash attributes 被 过早删除。

为了减少这类问题，RedirectView 会 自动使用 目标重定向URL 的路径 和 查询参数 来 标记 FlashMap。反过来，默认的 FlashMapManager 在 查找 "input" FlashMap 时 会将 该信息与传入请求 进行匹配。

这并不能完全 消除 并发问题，但可以通过 重定向URL 中已有的信息 来 大大减少发生的几率。因此，我们建议 你 将flash 属性 主要 用于 重定向场景。



---

Multipart

在 MultipartResolver 被启用后，POST 的 multipart/form-data 的 内容 被解析，可以想 一般 request 参数 一样 被 访问。

```java
@Controller
public class FileUploadController {

    @PostMapping("/form")
    public String handleFormUpload(@RequestParam("name") String name,
            @RequestParam("file") MultipartFile file) {

        if (!file.isEmpty()) {
            byte[] bytes = file.getBytes();
            // store the bytes somewhere
            return "redirect:uploadSuccess";
        }
        return "redirect:uploadFailure";
    }
}
```

声明 参数类型 为 `List<MultipartFile>` 允许 解析 相同参数名称 的 多个文件。

当 @RequestParam 注解 在 `Map<String, MultipartFile>` 或 `MultiValueMap<String, MultipartFile>` 上，如果 没 有 在 注解中 指定 名字，那么 map被 每个指定参数名称 的 multipart 文件 填充。



使用Servlet 3.0 的 Multipart 解析，你还可以声明 javax..Part 而不是 spring 的 MultipartFile ，作为 方法参数 或 集合 的类型。

你还可以使用 multipart 内容 作为绑定到 command object ( 。。链接指向 上面的 @ModelAttribute)的 数据的一部分。例如，前面例子中的 表单字段和 文件 可以是 表单对象上的 字段：

```java
class MyForm {

    private String name;

    private MultipartFile file;

    // ...
}

@Controller
public class FileUploadController {

    @PostMapping("/form")
    public String handleFormUpload(MyForm form, BindingResult errors) {
        if (!form.getFile().isEmpty()) {
            byte[] bytes = form.getFile().getBytes();
            // store the bytes somewhere
            return "redirect:uploadSuccess";
        }
        return "redirect:uploadFailure";
    }
}
```



multipart 请求 也可以被 提交 从非浏览器，在restful 场景中。下面是 一个json 文件

```
POST /someUrl
Content-Type: multipart/mixed

--edt7Tfrdusa7r3lNQc79vXuhIIMlatb7PQg7Vp
Content-Disposition: form-data; name="meta-data"
Content-Type: application/json; charset=UTF-8
Content-Transfer-Encoding: 8bit

{
    "name": "value"
}
--edt7Tfrdusa7r3lNQc79vXuhIIMlatb7PQg7Vp
Content-Disposition: form-data; name="file-data"; filename="file.properties"
Content-Type: text/xml
Content-Transfer-Encoding: 8bit
... File Data ...
```

你可以访问 "meta-data" 部分 通过 @RequestParam 作为一个string， 但你可能希望它从 json 反序列化 ( 类似@RequestBody )。 使用HttpMessageConverter 转换 multipart 后， 使用 @RequestPart 注解访问 multipart

```java
@PostMapping("/")
public String handle(@RequestPart("meta-data") MetaData metadata,
        @RequestPart("file-data") MultipartFile file) {
    // ...
}
```



你可以使用 @RequestPart 组合 javax.Valie 或 spring的 @Validated，这2个都会导致 应用 标准 Bean 校验。默认下，校验错误 导致 MethodArgumentNotValidException, 这个被转换为 400 (bad request) 响应。或者，你可以通过 Errors 或 BindingResult 参数 在 controller 中 本地处理 验证错误：

```java
@PostMapping("/")
public String handle(@Valid @RequestPart("meta-data") MetaData metadata,
        BindingResult result) {
    // ...
}
```



---

@RequestBody

使用这个注解，来让 request body 读取并反序列化到 Object 通过 HttpMessageConverter

```java
@PostMapping("/accounts")
public void handle(@RequestBody Account account) {
    // ...
}
```

你可以通过 mvc 配置 message oconverter 来 自定义/配置 消息转换。

你可以使用 @RequestBody 结合 javax..Valid 或 spring 的@Validated ，这2个都会导致 标准Bean 校验 被应用。默认下， 校验失败，会导致 MethodArgumentNotValidException，这个会转换为 400 响应。另一方面，你可以通过 Errors 或 BindingResult 参数 来 在 controller 方法 本地 处理这些 校验错误。

```java
@PostMapping("/accounts")
public void handle(@Valid @RequestBody Account account, BindingResult result) {
    // ...
}
```



---

HttpEntity

HttpEntity 类似 @RequestBody，但是它是基于 公开 request header 和body 的 容器对象 的。

```java
@PostMapping("/accounts")
public void handle(HttpEntity<Account> entity) {
    // ...
}
```



---

@ResponseBody

注解到方法上，通过 HttpMessageConverter 将 返回 序列化为 响应体。

```java
@GetMapping("/accounts/{id}")
@ResponseBody
public Account handle() {
    // ...
}
```

@ResponseBody 也可以在 类级别，这种情况下，它 被 所有 controller 方法 继承。这就是 @RestController 的效果。

你可以使用 @ResponseBody 和 reactive 类型 一起。

你可以使用 mvc 的 message converter 来 配置 或自定义 消息转换。

你可以将 @ResponseBody 方法 和 Json 序列化视图 结合使用。



---

ResponseEntity

类似 @ResponseBody， 但它有 status 和 header

```java
@GetMapping("/something")
public ResponseEntity<String> handle() {
    String body = ... ;
    String etag = ... ;
    return ResponseEntity.ok().eTag(etag).build(body);
}
```

mcv支持，使用 单值 reactive 类型 来 异步 生成 ResponseEntity ，且/或 为body 生成 单值和多值 的reactive type。这允许以下类型的异步响应：

1. `ResponseEntity<Mono<T>>` 或 `ResponseEntity<Flux<T>>` 可以立刻知道 响应状态 和 header，稍后异步提供 body。如果body包含 0到1个值，那么使用 Mono， 如果可以生成多值，那么使用 Flux。
2. `Mono<ResponseEntity<T>>` 稍后 异步提供 全部的 3个部分：status，header，body。这允许响应状态 和header 根据异步请求处理 的结果 而变化。



---

Jackson JSON

spring 提供了对 Jackson Json 库的支持

---

JSON Views

mvc 提供内置支持 for  Jackson's Serialization Views， 这个只 允许 渲染 Object中所有filed 的 子集。将它和 @ResponseBody 或 ResponseEntity 控制器方法 一起使用， 你可以使用 Jackson 的 @JsonView 注解 来激活 序列化 view class

```java
@RestController
public class UserController {

    @GetMapping("/user")
    @JsonView(User.WithoutPasswordView.class)
    public User getUser() {
        return new User("eric", "7!jd#h23");
    }
}

public class User {

    public interface WithoutPasswordView {};
    public interface WithPasswordView extends WithoutPasswordView {};

    private String username;
    private String password;

    public User() {
    }

    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }

    @JsonView(WithoutPasswordView.class)
    public String getUsername() {
        return this.username;
    }

    @JsonView(WithPasswordView.class)
    public String getPassword() {
        return this.password;
    }
}
```



@JsonView 允许 view class 的数组，但 每个控制器方法只能制定一个。如果要激活 多个视图，可以使用 复合接口。



如果你想 以编程方式 执行上述操作，而不是声明 @JsonView 注解，用MappingJacksonView 包装 返回值 并使用它来提供 序列化 view：

```java
@RestController
public class UserController {

    @GetMapping("/user")
    public MappingJacksonValue getUser() {
        User user = new User("eric", "7!jd#h23");
        MappingJacksonValue value = new MappingJacksonValue(user);
        value.setSerializationView(User.WithoutPasswordView.class);
        return value;
    }
}
```



对于 依赖 view解析的 controller，你可以增加 序列化 view class 到 model：

```java
@Controller
public class UserController extends AbstractController {

    @GetMapping("/user")
    public String getUser(Model model) {
        model.addAttribute("user", new User("eric", "7!jd#h23"));
        model.addAttribute(JsonView.class.getName(), User.WithoutPasswordView.class);
        return "userView";
    }
}
```



---

#### 1.3.4 Model

你可以使用 @ModelAttirbute 注解：

1. 在@RequestMapping方法 参数上，用于从Model 创建或访问 Object，并通过 WebDataBinder 来绑定到请求上。
2. 作为@Controller 或 @ControllerAdvice 类中的 方法注解，有助于 在 任何@RequestMapping 方法 调用前 初始化 模型
3. 标注在 @RequestMapping 方法上，标记它的返回值 是一个 模型属性。



这节讨论 @ModelAttribute 的方法，前面列表中的 第二项。 一个controller 可以有任意数量的 @ModelAttribute 方法，所有这些方法 被调用 before  同一个controller 的 @RequestMapping 方法。@ModelAttribute 的方法 也可以通过 @ControllerAdvice 跨controller 共享。



> @ModelAttribute 方法 具有灵活的方法签名。它们支持 许多与 @RequestMapping方法相同的参数，除了 @ModelAttribute 本身 或与请求body 相关的任何内容。

。。就是 @ModelAttribute 注解的方法 和 @RequestMapping 的方法 所能 接受的 参数 差不多， 除了 本身 及 请求body。

```java
@ModelAttribute
public void populateModel(@RequestParam String number, Model model) {
    model.addAttribute(accountRepository.findAccount(number));
    // add more ...
}
```

```java
@ModelAttribute
public Account addAccount(@RequestParam String number) {
    return accountRepository.findAccount(number);
}
```

当没有显式指定名字，基于 对象 类型的 默认名字 会被选择，就像 Convertions 的 javadoc中 描述的一样。你可以通过 重载的 addAttribute 方法 或 通过 @ModelAttributes 的name 属性 来 指定一个 显式的名字。



你可以使用 @ModeAttributes 作为 @RequestMapping 方法的 方法级别注解，这种情况下，

> @RequestMapping 的返回值 被解释为 model attribute。这通常不是必须的，因为它是HTML controller 中的 默认行为，除非 返回值是一个字符串 这种会被解释为view name 
>
> This is typically not required, as it is the default behavior in HTML controllers, unless the return value is a `String` that would otherwise be interpreted as a view name. ``



@ModelAttribute 也可以 自定义 model attribute name：

```java
@GetMapping("/accounts/{id}")
@ModelAttribute("myAccount")
public Account handle() {
    // ...
    return account;
}
```



---

#### 1.3.5 DataBinder

@Controller 或 @ControllerAdvice 注解的类 可以有 @InitBinder 方法，用来初始化 WebDataBinder 实例，这些可以：

1. bind 请求参数 ( 表单或 查询 数据) 到 模型对象
2. 转换 基于string 的 请求值 ( 如 请求参数，路径变量，头，cookie，等) 到 controller 方法 参数的 类型。
3. 呈现HTML表单时，将模型对象值格式化为 字符串值。



@InitBinder 方法可以注册 特定于 controller 的 java.beas.PropertyEditor 或 spring的 Converter 和 Formatter 组件。此外，你可以使用 mvc 配置 在 全局共享的 FormattingConversionService 中 注册 Converter 和 Formatter类型。



@InitBinder 方法 支持 许多和 @RequestMapping 方法相同的参数，但 @ModelAttribute (command object) 除外。通常，它们 声明 WebDataBinder 参数( 为了注册) 和 void 返回值。

```java
@Controller
public class FormController {

    @InitBinder 
    public void initBinder(WebDataBinder binder) {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        dateFormat.setLenient(false);
        binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false));
    }

    // ...
}
```

或者，当你 通过 一个共享的 FormattingConversionService 使用 基于Formatter 的配置，你可以 重用 相同的方法 且 注册 方法级别的 Formatter 实现

```java
@Controller
public class FormController {

    @InitBinder 
    protected void initBinder(WebDataBinder binder) {
        binder.addCustomFormatter(new DateFormatter("yyyy-MM-dd"));
    }

    // ...
}
```



---

1.3.6 Exceptions



@Controller 和 @ControllerAdvice 类可以有 @ExceptionHandler 方法 来处理 来自 controller 方法的 异常：

```java
@Controller
public class SimpleController {

    // ...

    @ExceptionHandler
    public ResponseEntity<String> handle(IOException ex) {
        // ...
    }
}
```



异常可以匹配 正在传播的顶级异常 ( 如 抛出直接IOException ) 或与 被包装的 异常的 内嵌的异常 匹配 (如 IllegalStateException 中嵌套的 IOException )。 从5.3开始，这可以匹配任意的 cause level，之前只考虑 直接原因。

为了 匹配异常类型，最好将目标异常 声明为 方法参数，就像上面的例子。 当有多个异常方法匹配时，root异常 匹配 优先于 比 cause异常匹配。更具体的说，ExceptionDepthComparator 用来 排序 根据抛出的异常类型中的深度。

注解可以缩小 匹配的异常：

```java
@ExceptionHandler({FileSystemException.class, RemoteException.class})
public ResponseEntity<String> handle(IOException ex) {
    // ...
}
```

甚至你可以 使用 非常通用的 方法参数 with 特定的异常列表

```java
@ExceptionHandler({FileSystemException.class, RemoteException.class})
public ResponseEntity<String> handle(Exception ex) {
    // ...
}
```



root 和 cause 异常 匹配 之间的区别可能令人惊讶。

在前面显示的 IOException 变体中，通常使用 实际的 FileSystemException 或 RemoteException 实例作为 参数调用该方法，因为它们都是从 IOException 扩展而来的。但是如果任何 匹配的异常 被本身是IOException 的 异常封装 并传播，则传入的异常实例就是 该包装异常。

。。。。这个是说 异常被封装为 IOException 会被捕获，并且 方法 获得 封装后的IOException。 毕竟方法形参是IOException。

handle(Exception ) 行为更简单。这总是在 包装场景中 使用 包装异常调用，在这种情况下 使用ex.getCause 找到实际匹配的 异常。只有当这些异常作为顶级异常抛出时，传入的异常才是实际的 FileSystemException 或 RemoteException 实例。



我们通常建议你在参数签名中 尽可能具体，以减少 root 和 cause 异常 类型 间的 不匹配。考虑将一个 多匹配方法 分解为 单独的 @ExceptionHandler 方法，每个方法通过签名匹配一个特定的异常类型。

在多@ControllerAdvice 情况下，我们建议在 @ControllerAdvice 上声明 你的 主root 异常映射，并按照相应的顺序进行优先级排序。虽然root 异常 匹配 优先于 cause 异常，但是这是在 给定controller 或 @ControllerAdvice 类中 定义的。这意味这 较高优先级 @ControllerAdvice bean 上的 cause 匹配 优先于 较低优先级的 @ControllerAdvice 上的 任何匹配。

。。就是 先看 @ControllerAdvice 的 优先级， 然后再看 root 还是 cause。

@ExceptionHandler 方法实现可以 选择 退出处理给定异常实例，方法是 以原始形态 重新抛出它。这在 你只对root 级别匹配 或无法静态确定的 特定上下文中 的匹配感兴趣的情况下很有用。重新抛出的异常通过 剩余的 解析链 传播，就像 给定的 @ExceptionHandler 方法 一开始 就没有匹配到一样。



spring mvc 中 对 @ExceptionHandler 方法的支持 构建在 DispatcherServlet 级别，HandlerExceptionResovler 机制。



---

MEthod Arguments

@ExceptionHandler 方法 支持下面的参数：

| Method argument                                              | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Exception type                                               | For access to the raised exception.                          |
| `HandlerMethod`                                              | For access to the controller method that raised the exception. |
| `WebRequest`, `NativeWebRequest`                             | Generic access to request parameters and request and session attributes without direct use of the Servlet API. |
| `javax.servlet.ServletRequest`, `javax.servlet.ServletResponse` | Choose any specific request or response type (for example, `ServletRequest` or `HttpServletRequest` or Spring’s `MultipartRequest` or `MultipartHttpServletRequest`). |
| `javax.servlet.http.HttpSession`                             | Enforces the presence of a session. As a consequence, such an argument is never `null`.  Note that session access is not thread-safe. Consider setting the `RequestMappingHandlerAdapter` instance’s `synchronizeOnSession` flag to `true` if multiple requests are allowed to access a session concurrently. |
| `java.security.Principal`                                    | Currently authenticated user — possibly a specific `Principal` implementation class if known. |
| `HttpMethod`                                                 | The HTTP method of the request.                              |
| `java.util.Locale`                                           | The current request locale, determined by the most specific `LocaleResolver` available — in effect, the configured `LocaleResolver` or `LocaleContextResolver`. |
| `java.util.TimeZone`, `java.time.ZoneId`                     | The time zone associated with the current request, as determined by a `LocaleContextResolver`. |
| `java.io.OutputStream`, `java.io.Writer`                     | For access to the raw response body, as exposed by the Servlet API. |
| `java.util.Map`, `org.springframework.ui.Model`, `org.springframework.ui.ModelMap` | For access to the model for an error response. Always empty. |
| `RedirectAttributes`                                         | Specify attributes to use in case of a redirect — (that is to be appended to the query string) and flash attributes to be stored temporarily until the request after the redirect. See [Redirect Attributes](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-redirecting-passing-data) and [Flash Attributes](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-flash-attributes). |
| `@SessionAttribute`                                          | For access to any session attribute, in contrast to model attributes stored in the session as a result of a class-level `@SessionAttributes` declaration. See [`@SessionAttribute`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-sessionattribute) for more details. |
| `@RequestAttribute`                                          | For access to request attributes. See [`@RequestAttribute`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestattrib) for more details. |



---

Retuen Values

@ExceptionHandler 支持的 返回值：

| Return value                                    | Description                                                  |
| ----------------------------------------------- | ------------------------------------------------------------ |
| `@ResponseBody`                                 | The return value is converted through `HttpMessageConverter` instances and written to the response. See [`@ResponseBody`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-responsebody). |
| `HttpEntity<B>`, `ResponseEntity<B>`            | The return value specifies that the full response (including the HTTP headers and the body) be converted through `HttpMessageConverter` instances and written to the response. See [ResponseEntity](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-responseentity). |
| `String`                                        | A view name to be resolved with `ViewResolver` implementations and used together with the implicit model — determined through command objects and `@ModelAttribute` methods. The handler method can also programmatically enrich the model by declaring a `Model` argument (described earlier). |
| `View`                                          | A `View` instance to use for rendering together with the implicit model — determined through command objects and `@ModelAttribute` methods. The handler method may also programmatically enrich the model by declaring a `Model` argument (descried earlier). |
| `java.util.Map`, `org.springframework.ui.Model` | Attributes to be added to the implicit model with the view name implicitly determined through a `RequestToViewNameTranslator`. |
| `@ModelAttribute`                               | An attribute to be added to the model with the view name implicitly determined through a `RequestToViewNameTranslator`. Note that `@ModelAttribute` is optional. See “Any other return value” at the end of this table. |
| `ModelAndView` object                           | The view and model attributes to use and, optionally, a response status. |
| `void`                                          | A method with a `void` return type (or `null` return value) is considered to have fully handled the response if it also has a `ServletResponse` an `OutputStream` argument, or a `@ResponseStatus` annotation. The same is also true if the controller has made a positive `ETag` or `lastModified` timestamp check (see [Controllers](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-caching-etag-lastmodified) for details). If none of the above is true, a `void` return type can also indicate “no response body” for REST controllers or default view name selection for HTML controllers. |
| Any other return value                          | If a return value is not matched to any of the above and is not a simple type (as determined by [BeanUtils#isSimpleProperty](https://docs.spring.io/spring-framework/docs/5.3.16/javadoc-api/org/springframework/beans/BeanUtils.html#isSimpleProperty-java.lang.Class-)), by default, it is treated as a model attribute to be added to the model. If it is a simple type, it remains unresolved. |



---

REST API exceptions

rest服务的 常见要求是 在 响应body中 包含 错误详细信息。spring 框架 不会自动执行此操作，因为响应正文中错误信息是特定于应用的。但是 @RestController 可以使用 带有ResponseEntity 返回 的@ExceptionHandler 方法 来设置 响应的状态 和 body，这些方法也可以声明到 @ControllerAdvice 中 以全局使用它们。

在响应body中使用错误信息 实现 全局异常处理的 app 应该考虑扩展 ResponseEntityExceptionHandler， 它为 spring mvc 引发的异常提供了 自定义响应正文的 hook。为了使用它，创建一个 ResponseEntityExceptionHandler 的子类，并注解 @ControllerAdvice，覆盖必要的方法，并声明为 bean。



---

1.3.7 Controller Advice

@ExceptionHandler, @InitBinder,@ModelAttribute 的方法 只能 声明它们的 @Controller 类中，或类hierarchy中 使用。 如果它们在 @ControllerAdvice 或 @RestControllerAdvice 中 声明，则它们适用于任何 controller。 从5.3开始，@CoontrollerAdvice 中的 @ExceptionHandler 能用来处理 来自 任何 @Controller 或其他 处理程序中的 异常。

@ControllerAdvice 被 @Component 注解注解，因此可以注册为 spring bean ，通过 component scanning。@RestControllerAdvice 被 @ControllerAdvice 和 @ResponseBody 注解注解，这意味着@ExceptionHandler 方法的返回值 将通过 响应body 的消息转换 而不是 通过 html视图 呈现。

启动时，RequestMappingHandlerMapping 和 ExceptionHandlerExceptionResolver 检测 controller advice bean，并在运行时 应用它们。来自 @ControllerAdvice 的全局 @ExceptionHandler 方法 在 来自 @Controller 的 本地@ExceptionHandler方法 之后应用。 全局的 @ModelAttribute 和 @InitBinder 方法 在 @Controller 的 之前 应用。



@ControllerAdvice 注解 有属性 让你 缩小 它们应用的 controller 和 handler

```java
// Target all Controllers annotated with @RestController
@ControllerAdvice(annotations = RestController.class)
public class ExampleAdvice1 {}

// Target all Controllers within specific packages
@ControllerAdvice("org.example.controllers")
public class ExampleAdvice2 {}

// Target all Controllers assignable to specific classes
@ControllerAdvice(assignableTypes = {ControllerInterface.class, AbstractController.class})
public class ExampleAdvice3 {}
```

例子中的 选择器 是在 运行时 eval 的，如果广泛使用，可能会性能造成影响。更多信息，看 @ControllerAdvice 的 javadoc。



---

1.4 Functional Endpoints

spring web mvc 包含了 WebMvc.fn， 一个轻量级 functional programming model，用来 route 和 handle 请求，并且为 不变性 设计了 合约，它是基于注解的 编程模型的替代方案，但在相同的 DisServ 上运行。



---

1.4.1 Overview

WebMvc.fn 中， 一个http 请求 被 HandlerFunction 处理：一个接受ServerRequest 并返回 ServerResponse 的 function。请求 和 响应 对象 都有不可变的 契约，提供对http请求和响应的 jdk8 友好 的访问。HandlerFunction是 基于注解的编程模型 中的 @RequestMapping 方法body 的 等价。

传入的请求 通过 RouterFunction 进行路由： 一个function 接受ServerRequest ，返回一个可选的 HandlerFunction ( 如 `Optional<HandlerFunction>`) 。当 router function 匹配，返回一个 handler function。否则 返回空的 Optional 对象。 RouterFunction 等价于 @RequestMapping 注解，但主要区别是，router function 不仅提供数据，还提供行为。

RouterFunction.route() 提供了 router builder ，用来 帮助 router 的创建：

```java
import static org.springframework.http.MediaType.APPLICATION_JSON;
import static org.springframework.web.servlet.function.RequestPredicates.*;
import static org.springframework.web.servlet.function.RouterFunctions.route;

PersonRepository repository = ...
PersonHandler handler = new PersonHandler(repository);

RouterFunction<ServerResponse> route = route()
    .GET("/person/{id}", accept(APPLICATION_JSON), handler::getPerson)
    .GET("/person", accept(APPLICATION_JSON), handler::listPeople)
    .POST("/person", handler::createPerson)
    .build();


public class PersonHandler {

    // ...

    public ServerResponse listPeople(ServerRequest request) {
        // ...
    }

    public ServerResponse createPerson(ServerRequest request) {
        // ...
    }

    public ServerResponse getPerson(ServerRequest request) {
        // ...
    }
}
```

。。。？ 这个怎么在 class 外 声明变量。。



如果你注册 RouterFunction 作为bean，例如通过 @Configuration 类中公开它，它将被servlet 自动检测到。



---

1.4.2 HandlerFunction

ServerRequest, ServerResponse 是不可变接口，提供了 jdk8 友好的 接口 来访问 http request 和 response，包括 header，body,method,status code。



---

ServerRequest

提供了 对 http method, uri,headers,query paramerters 的访问， 同时 通过 body 方法来访问 body

```java
String string = request.body(String.class);
```

提取body 到 `List<Person>`， Person 对象 从序列化的形式( 如JSON ，xml )中 解码。

```java
List<Person> people = request.body(new ParameterizedTypeReference<List<Person>>() {});
```

下面是访问 parameter

```java
MultiValueMap<String, String> params = request.params();
```



---

ServerResponse

提供了 对 http response 的访问，由于它是 不可变的，你可以使用 build 方法 来创建它。你可以使用 builder 来设置 响应status，增加 响应 header，或提供一个 body。

下面创建 200 响应 with JSON 内容

```java
Person person = ...
ServerResponse.ok().contentType(MediaType.APPLICATION_JSON).body(person);
```

下面是 创建 201 with Location 头。没有body

```java
URI location = ...
ServerResponse.created(location).build();
```

使用 异步结果作为 body，以 CompletableFuture，Publisher 格式，或其他 任何 被 ReactiveAdapterRegistry 支持的类型

```java
Mono<Person> person = webClient.get().retrieve().bodyToMono(Person.class);
ServerResponse.ok().contentType(MediaType.APPLICATION_JSON).body(person);
```

如果不止body，header 和status 也要 异步，你可以使用 ServerResponse 的 静态的 async 方法，这个接受 `CompletableFuture<ServerResponse>` ，`Publisher<ServerResponse>` 或 任何其他的 被ReactiveAdapterRegistry 支持的异步类。

```java
Mono<ServerResponse> asyncResponse = webClient.get().retrieve().bodyToMono(Person.class)
  .map(p -> ServerResponse.ok().header("Name", p.name()).body(p));
ServerResponse.async(asyncResponse);
```



Server-Sent Events 能被发布 通过 ServerResponse 的静态 sse 方法。这个方法提供的 builder 允许你 发送 string 或其他 对象 (如 JSON)

```java
public RouterFunction<ServerResponse> sse() {
    return route(GET("/sse"), request -> ServerResponse.sse(sseBuilder -> {
                // Save the sseBuilder object somewhere..
            }));
}

// In some other thread, sending a String
sseBuilder.send("Hello world");

// Or an object, which will be transformed into JSON
Person person = ...
sseBuilder.send(person);

// Customize the event by using the other methods
sseBuilder.id("42")
        .event("sse event")
        .data(person);

// and done at some point
sseBuilder.complete();
```



---

Handler Classes

我们可以编写一个handler function 作为一个 lambda：

```java
HandlerFunction<ServerResponse> helloWorld =
  request -> ServerResponse.ok().body("Hello World");
```



。。跳过。。。

























---

1.5 URI Links

介绍 spring 框架中 用于处理 URI 的各种选项。



---

1.5.1 UriComponents

UriComponentBuilder 帮助我们 来创建 URI 从 带有变量的 URI模板中。

```java
UriComponents uriComponents = UriComponentsBuilder
        .fromUriString("https://example.com/hotels/{hotel}")  
        .queryParam("q", "{q}")  
        .encode() 
        .build(); 

URI uri = uriComponents.expand("Westin", "123").toUri();  
```

上面的能被合成一个链，并且使用 buildAndExpand 来缩短

```java
URI uri = UriComponentsBuilder
        .fromUriString("https://example.com/hotels/{hotel}")
        .queryParam("q", "{q}")
        .encode()
        .buildAndExpand("Westin", "123")
        .toUri();
```

进一步缩短 by 直接转到URI ( 这意味着编码)

```java
URI uri = UriComponentsBuilder
        .fromUriString("https://example.com/hotels/{hotel}")
        .queryParam("q", "{q}")
        .build("Westin", "123");
```

再短一点

```java
URI uri = UriComponentsBuilder
        .fromUriString("https://example.com/hotels/{hotel}?q={q}")
        .build("Westin", "123");
```



---

1.5.2 UriBuilder

UriComponentsBuilder 实现了 UriBuilder。反过来，你可以使用 UriBuilderFactory 来创建 UriBuilder。

UriBuilderFactory 和 UriBuilder 一起提供了一种可插入的机制，以基于共享配置( 例如基本URL，编码首选项 和其他详细信息 ) 从URI模板构建URI

下面是配置 RestTemplate

```java
// import org.springframework.web.util.DefaultUriBuilderFactory.EncodingMode;

String baseUrl = "https://example.org";
DefaultUriBuilderFactory factory = new DefaultUriBuilderFactory(baseUrl);
factory.setEncodingMode(EncodingMode.TEMPLATE_AND_VALUES);

RestTemplate restTemplate = new RestTemplate();
restTemplate.setUriTemplateHandler(factory);
```

配置 WebClient

```java
// import org.springframework.web.util.DefaultUriBuilderFactory.EncodingMode;

String baseUrl = "https://example.org";
DefaultUriBuilderFactory factory = new DefaultUriBuilderFactory(baseUrl);
factory.setEncodingMode(EncodingMode.TEMPLATE_AND_VALUES);

WebClient client = WebClient.builder().uriBuilderFactory(factory).build();
```

也可以直接使用 DefaultUriBuilderFactory 。 类似与 使用 UriComponentsBuilder，但 它不是静态工厂方法，而是保存配置和首选项的 实际实例。

```java
String baseUrl = "https://example.com";
DefaultUriBuilderFactory uriBuilderFactory = new DefaultUriBuilderFactory(baseUrl);

URI uri = uriBuilderFactory.uriString("/hotels/{hotel}")
        .queryParam("q", "{q}")
        .build("Westin", "123");
```



---

1.5.3 URI Encoding

UriComponentsBuilder 开放 encoding 选项 在2个级别：

1. UriComponentsBuilder#encode()，先对URI模板进行 预编码，然后在扩展时是 URI 变量进行 严格编码。
2. UtiComponents#encode()，在URI变量扩展后 对URI 组件 进行编码。

这2个都使用转义的八位字节替换 给 AScii 和 非法字符。但是 第一个选项也会替换URI 变量中 出现的 具有保留含义的 字符。

考虑";"，它在路径中是合法的，但是具有 保留的含义，第一个选项替换 ";"，在URI 变量中使用 %3B，但是在URI木板中 没有。相比之下， 第二种永远 不会 替换 ; 。因为它是路径中的合法字符。



对于大多数情况，第一个选项可能会给出预期的结果，因为它将uri 变量视为 要完全编码的 不透明数据，而如果uri 变量确实包含 保留字符，则第二个选项有用。当根本不扩展uri 变量时，第二个选项也很有用。因为这也会对任何看起来想URI 变量的 东西进行编码。

```java
URI uri = UriComponentsBuilder.fromPath("/hotel list/{city}")
        .queryParam("q", "{q}")
        .encode()
        .buildAndExpand("New York", "foo+bar")
        .toUri();

// Result is "/hotel%20list/New%20York?q=foo%2Bbar"
```

```java
URI uri = UriComponentsBuilder.fromPath("/hotel list/{city}")
        .queryParam("q", "{q}")
        .build("New York", "foo+bar");
```

```java
URI uri = UriComponentsBuilder.fromUriString("/hotel list/{city}?q={q}")
        .build("New York", "foo+bar");
```



WebClient 和 RestTemplate 内部使用 UriBuilderFactory 扩展和 编码 URI 模板。 2者都能配置 自定义策略

```java
String baseUrl = "https://example.com";
DefaultUriBuilderFactory factory = new DefaultUriBuilderFactory(baseUrl)
factory.setEncodingMode(EncodingMode.TEMPLATE_AND_VALUES);

// Customize the RestTemplate..
RestTemplate restTemplate = new RestTemplate();
restTemplate.setUriTemplateHandler(factory);

// Customize the WebClient..
WebClient client = WebClient.builder().uriBuilderFactory(factory).build();
```



DefaultUriBuilderFactory 实现 内部 使用 UriComponentBuilder 来扩展 和 编码 URI 模板。作为一个工厂，它提供了一个单一的位置 来配置编码方式，基于以下的编码模式之一：

1. TEMPLATE_AND_VALUES，使用 UriConponentsBuilder#encode()，对应前面列表中(。。1.5.3刚开始的)的 第一个选项，对URI模板进行 预编码，并在 扩展时 严格编码URI 变量。
2. VALUES_ONLY，不对URI 模板进行编码，而是通过UriUtils#encodeUriVariables 对 URI变量 应用严格编码，然后再将它们展开到 模板中。
3. URI_COMPONENT，使用 UriComponents#encode()，对应前面列表中的第二个选项，在 URI变量 扩展后 对 URI 组件进行编码
4. NONE，不应用编码

RestTemplate 被设置为 EncodingMode.URI_COMPONENT 由于 历史原因 和 向后兼容。WebClient 依赖DefaultUriBuilderFactory 中的 默认值，该默认值 从5.0的 EncodingMode.URI_COMPONENT 变为 5.1的 TEMPLATE_AND_VALUES



---

1.5.4 Relative(相对) Servlet Requests

你可以使用 ServletUriComponentsBuilder 来创建 相对于向前请求的 URI

```java
HttpServletRequest request = ...

// Re-uses scheme, host, port, path, and query string...

URI uri = ServletUriComponentsBuilder.fromRequest(request)
        .replaceQueryParam("accountId", "{id}")
        .build("123");
```

相对于 context path

```java
HttpServletRequest request = ...

// Re-uses scheme, host, port, and context path...

URI uri = ServletUriComponentsBuilder.fromContextPath(request)
        .path("/accounts")
        .build()
        .toUri();
```

想对于servlet (例如 /main/*)

```java
HttpServletRequest request = ...

// Re-uses scheme, host, port, context path, and Servlet mapping prefix...

URI uri = ServletUriComponentsBuilder.fromServletMapping(request)
        .path("/accounts")
        .build()
        .toUri();
```



5.1开始，ServletUriComponentsBuilder 忽略 来自 Forwarded 和 X-Forwarded-* 头的 信息，这些header 指定了 客户端发起的 地址。考虑使用 ForwardedHeaderFilter 来提取和使用 或 丢弃 此类header。



---

1.5.5 Links to Controllers

mvc 提供了 机制 来准备 controller 方法的链接，例如，下面的 mvc controller 允许 创建 link

```java
@Controller
@RequestMapping("/hotels/{hotel}")
public class BookingController {

    @GetMapping("/bookings/{booking}")
    public ModelAndView getBooking(@PathVariable Long booking) {
        // ...
    }
}
```

你可以准备一个 link 通过 name 指向方法

```java
UriComponents uriComponents = MvcUriComponentsBuilder
    .fromMethodName(BookingController.class, "getBooking", 21).buildAndExpand(42);

URI uri = uriComponents.encode().toUri();
```

上面的例子中，我们提供了 实际的 方法参数值( 本例中是 21) 用做路径变量 并插入到URL 中。此外，我们提供 42 来填充 任何剩余的 URI 变量，比如从 类级别 request mapping 继承而来的 hotel 。如果该方法有更多参数，我们可以为URL 不需要的 参数 提供 null，通常 只有 @PathVariable 和 @RequestParam 参数 需要 用于 构造URL。

还有其他使用MvcUriComponentsBuilder 的方法，例如，你可以使用 类似于通过 代理 进行模拟测试的 技术 来避免 按名称 引用 控制器方法( 假定静态导入了 MvcUriComponentsBuilder.on)：

```java
UriComponents uriComponents = MvcUriComponentsBuilder
    .fromMethodCall(on(BookingController.class).getBooking(21)).buildAndExpand(42);

URI uri = uriComponents.encode().toUri();
```



 controller方法 签名 的设计 收到限制，当它们被认为 可以用于 通过 fromMethodCall 创建链接时。除了需要正确的 参数签名外，返回类型 还有 技术限制 (即， 生成一个运行时代理 for link builder 的调用)，所以 返回 类型 不能是 final。特别是，返回 普通 string 作为 视图名称 在这里不起作用。你应该使用 ModelAndView 或 甚至是普通的 Object ( 带有 string 返回值)

前面的例子使用 MvcUriComponentsBuilder 中的 静态方法，在内部，它们依靠 ServletUriComponentsBuilder 从当前请求的 schema，host，post，context path，servlet path 中 准备一个 基本URL。这在大多数情况下 都很有效。 但，有时，它可能不够。例如，你可能在请求的 上下文之外 ( 例如，准备链接的批处理) 或 你可能需要 插入路径前缀 ( 例如，从请求路径中 删除 并需要 重新插入链接)

对于这种情况，你可以使用 接受 UriComponentsBuilder 的 静态 fromXxx 重载方法 来使用 基本URL。或者，你可以使用基本URL 创建 MvcUriComponentsBuilder 的实例，然后使用 基于实例的 withXxx 方法：

```java
UriComponentsBuilder base = ServletUriComponentsBuilder.fromCurrentContextPath().path("/en");
MvcUriComponentsBuilder builder = MvcUriComponentsBuilder.relativeTo(base);
builder.withMethodCall(on(BookingController.class).getBooking(21)).buildAndExpand(42);

URI uri = uriComponents.encode().toUri();
```



从5.1 开始，MvcUriComponentsBuilder 忽略 来自 Forwarded 和 X-Forwarded-* 头，这些指明了 客户端的 发起地址。考虑使用 ForwardedHeaderFilter 来提取 和 使用或丢弃 这类header。



---

1.5.6 Links in Views

在view ( 如 Thymeleaf，Freemarker，JSP) ,你可以 构建 link 到 注解的 contoller by 引用 为每个 request mapping 隐式 或显示分配的 名称。

```java
@RequestMapping("/people/{id}/addresses")
public class PersonAddressController {

    @RequestMapping("/{country}")
    public HttpEntity<PersonAddress> getAddress(@PathVariable String country) { ... }
}
```

JSP中：

```jsp
<%@ taglib uri="http://www.springframework.org/tags" prefix="s" %>
...
<a href="${s:mvcUrl('PAC#getAddress').arg(0,'US').buildAndExpand('123')}">Get Address</a>
```



上面依赖 声明在spring tag 库( 即 META-INF/spring.tld ) 的 mvcUrl 函数，但很容易定义自己的函数 或 为其他木板准备一个 类型的函数。



工作原理：在启动时，通过 HandlerMethodMappingNamingStrategy 为每个 @RequestMapping 分配一个 默认名称，其默认实现 使用类的 大写字母和方法名称 ( 如，ThingController 的getThing 方法 变成 TC#getThing )。 如果名称冲突，你可以使用 @RequestMapping(name="xxx") 来显式分配 名称 或 实现你自己的 HandlerMethodMappingNamingStragegy.



---

1.6 Asynchronous Requests

spring mvc 广泛集成了 Servlet 3.0 的 异步请求处理：

1. controller 方法 返回的 DeferredResult 和 Callable  提供了 对 单个异步返回值的 基本的支持
2. controller 能 stream multiple value，包括 SSE 和 raw data
3. controller 能使用 reactive client，且 返回 reactive type for 响应处理。



---

1.6.1 DeferredResult

一旦 在 servlet 容器中 启用 异步请求处理后，controller 方法 可以 使用 DeferredResult 包装 任何 支持的 controller 方法返回值。

```java
@GetMapping("/quotes")
@ResponseBody
public DeferredResult<String> quotes() {
    DeferredResult<String> deferredResult = new DeferredResult<String>();
    // Save the deferredResult somewhere..
    return deferredResult;
}

// From some other thread...
deferredResult.setResult(result);
```

controller 可以 异步生成 返回值，从其他线程--例如，响应外部事件( JMS 消息)，scheduled task，或其他事件。



---

1.6.2 Callable

控制器可以用 java.util.xxx.Callable 来封装 任何 支持的返回值

```java
@PostMapping
public Callable<String> processUpload(final MultipartFile file) {

    return new Callable<String>() {
        public String call() throws Exception {
            // ...
            return "someView";
        }
    };
}
```

可以通过 配置i的 TaskExecutor 执行指定 任务 来或的 返回值。





---

1.6.3 Processing

下面是对 Servlet 异步请求处理的 一个 简单概述：

1. 通过调用 request.startAsync() 将 ServletRequest 变成 异步 模式。这样做的主要效果是 Servlet( 及任何filter) 可以退出，但响应 保持打开状态，以便稍后 完成处理。
2. 对request.startAsync() 的调用 返回了 AsyncContext，你可以用 AsyncContext 来 进一步控制 异步处理。例如，它提供了 dispatch 方法，这个类似于 从 Servlet API 进行 转发，不同的是 它允许 app 在 servlet 容器 线程上 恢复 request processing
3. ServletRequest 提供了 对当前 DispatcherType 的访问，你可以用来 区分 原始请求，异步dispatch，转发，和其他 dispatcher 类型。



DeferredResult 处理流程：

1. controller 返回 DeferedResult，且 保存到 它可以访问的 内存中的 queue / list。
2. mvc 调用 request.startAsync()
3. 同时 , DispatcherServlet 和 所有配置的filter 退出 请求处理线程，但响应保持打开状态。
4. app 从某个线程 设置 DeferredResult，然后 mvc dispatch 请求 回到 servlet 容器。
5. DisServ 被再次调用，处理 异步生成的 返回值。



Callable 流程：

1. 容器返回 Callable
2. mvc 调用 request.startAsync()，submit Callable 到 TaskExecutor，让任务 在单独的线程中 运行。
3. 同时，DisServ 和 所有filter  退出Servlet 容器线程，但 响应保持打开。
4. 最终 Callable 产生一个 结果，mvc 将 请求 dispatch/分派 回 servet容器 来完成处理
5. DisSer 被再次调用，处理 从 Callable 生成的 异步结果。



---

Exception Handling

当你使用 DeferredResult，你可以选择 调用 setResult 还是 使用异常来setErrorResult。2种情况下，mvc 都把请求 分派回 servlet 容器 来完成处理。然后根据 控制器方法返回的 正常值 或 异常 来进行处理。异常会通过常规异常处理机制 处理( 如，调用 @ExceptionHandler 方法)。

当你使用 Callable，处理逻辑类似，主要的区别是： callable 返回结果 还是 callable 引发异常。



---

Interception

HandlerInterceptor 实例 可以是 AsyncHandlerInterceptor 类型，来 接受 启动异步处理的 原始请求 的 afterConcurrentHandlingStarted 回调。 (而不是 postHandle， afterComplete)

HandlerInterceptor 实现可以注册 CallableProcessingInterceptor 或 DeferredResultProcessingInterceptor，以更深入地和 异步请求 的生命周期 集成 (例如，处理超时事件)。

DeferredResult 提供 onTimeout(Runnable) 和 onComplete(Runnable) 回调。查看 DeferredResult 的 javadoc。Callable 可以被 WebAsyncTask 替换，后者提供了 timeout ，complete 的回调。



---

和 WebFlux 对比

。。。。

mvc 和 webflux 都支持 异步 和响应式 作为 controler 的返回值。

mvc甚至支持 流式传输，包括 响应式back pressure. 但对响应的单个写入 仍然是阻塞的 ( 并且在单独的线程上执行)， webflux 依赖于 非阻塞IO，并且每次写入都不需要 额外的线程

另一个根本区别是，mvc 不支持 控制器方法参数(。。参数，不是返回值 )中的  异步 或 响应式 (如 @RequestBody, @RequestPart 等) ， 也不明确支持 异步和额响应式 作为 模型属性。 webflux 支持这些。

。。。。



---

1.6.4 Http Streaming

你可以使用 DeferredResult 和 Callable 用于 单个异步返回值。如果你想生成 多个 异步值 并写入它们 到响应中，应该怎么办？本节介绍。



---

Objects

你可以使用 ResponseBodyEmitter 返回值 来生成 object 流，流中 每个对象 被 HttpMessageConverter 序列化，然后写入到 响应：

```java
@GetMapping("/events")
public ResponseBodyEmitter handle() {
    ResponseBodyEmitter emitter = new ResponseBodyEmitter();
    // Save the emitter somewhere..
    return emitter;
}

// In some other thread
emitter.send("Hello once");

// and again later on
emitter.send("Hello again");

// and done at some point
emitter.complete();
```



你也可以使用 ResponseBodyEmitter 作为 ResponseEntity 中的 body， 让你可以 自定义 响应的 http status 和 header。

当emitter 抛出IOException ( 如，远程客户端消失了)，app 不负责 清理connection，不应该调用 emitter.complete 或 emitter.completeWithError。 相反，servlet 容器会自动启动 AsyncListener 错误通知，其中 spring mvc 会调用 completeWithError。 反过来，此调用 对 app 执行一个 final ASYNC 分派，在此期间 mvc 调用配置的 exception resolver 并 完成请求。



---

SSE

SseEmitter ( ResponseBodyEmitter 的子类) 提供了 对 Server-Sent Events 的支持，其中从服务器发送的 事件根据W3C SSE 规范 进行 规格化。要从 controller 生成SSE 流，请返回 SseEmitter：

```java
@GetMapping(path="/events", produces=MediaType.TEXT_EVENT_STREAM_VALUE)
public SseEmitter handle() {
    SseEmitter emitter = new SseEmitter();
    // Save the emitter somewhere..
    return emitter;
}

// In some other thread
emitter.send("Hello once");

// and again later on
emitter.send("Hello again");

// and done at some point
emitter.complete();
```

虽然SSE 是流式 传输到 浏览器的 主要选项，但请注意 IE 浏览器 不支持 Server-Sent Events。(...IE:???)。考虑使用 spring 的 WebSocket messaging 和 SockJS fallback 传输 ( 包括SSE) 一起使用，该传输 针对各种浏览器。

异常处理 看上节



---

Raw Data

有时，绕过消息转换，并直接 流式传输到 响应OutputStream ( 如，用于文件下载) 很有用。你可以使用 StreamingResponseBody 返回值类型 来这么做

```java
@GetMapping("/download")
public StreamingResponseBody handle() {
    return new StreamingResponseBody() {
        @Override
        public void writeTo(OutputStream outputStream) throws IOException {
            // write...
        }
    };
}
```

你可以使用 StreamingResponseBody 作为 ResponseEntity 的body 来 自定义 响应的 http status 和 headers。



---

1.6.5 Reactive Types

mvc 支持 在controller 中使用 响应式客户端 库。这包含了 从spring-webflux 的 WebClient  和 其他的类 (比如，spring data 响应式数据存储库)。这种情况下，能够 方便地从 controller 方法 返回 响应式类型。



响应式返回值 被如下处理：

1. 一个单值的承诺 类似于 使用 DeferredResult。例如 Mono (Reactor) 或 Single (RxJava)
2. 带有 流式media type (如 application/x-ndjson, text/event-stream) 的 multi-value stream 类似于使用ResponseBodyEmitter 或 SseEmitter。 例如 Flux(Reactor) ,Observable(RxJava)。app 也能返回 `Flux<ServerSentEvent>` 或 `Observable<ServerSentEvent>` 。
3. 其他 media type ( 如 application/json ) 的 其他 multi-value stram 类似 使用 `DeferredResult<List<?>>`



mvc 通过 spring-core 的 ReactiveAdapterResigtry 来支持 Reactor  和 RxJava.

对于streaming 到 响应，响应式back pressure 被支持，但 写入到 响应 仍然是 阻塞的，并且 通过配置的 TaskExecutor 在 单独的线程上 运行，以避免阻塞 上游源 (例如 从 WebClient 返回的 Flux)。默认下，SimpleAsyncTaskExecutor 用于 阻塞写入，但在 负载下 不合适。 如果你计划使用 响应式类型 stream，你应该 使用 mvc的配置 来 配置一个 task executor。



---

1.6.6 Disconnects

servlet api 不提供 任何 通知 ，当远程客户端消失。 因此，当streaming 到 响应时，无论是通过 SseEmitter 或 reactive types， 定期发送数据很重要，因为如果客户端 断开连接，写入将失败。发送可以 采用 空的( 仅注释) SSE 事件 或 任何其他数据的 形式，对方必须 将其解释为心跳 并忽略。

或者，考虑使用 内置心跳的 web 消息传递解决方案 ( 例如，STOMP over WebSocket 或 WebSocket with SockJS)。



---

#### 1.6.7 Configuration

异步消息处理 功能必须在 servlet 容器 级别 启用。mvc配置还为 异步请求公开了几个选项。



---

Servlet Container

filter 和 servlet 声明 有一个 asyncSupported 标志，需要设置为true 以 启动 异步请求处理。此外，filter mapping 应该被声明为 处理 ASYNC

java配置中，当你使用 AbstractAnnotationConfigDispatcherServletInitializer 来初始化 Servlet 容器，这个会自动做。

web.xml 配置中，你可以增加 `<async-supported>true</async-supported>`  到DispatcherServlet 和 Filter 声明，增加 `<dispatcher>ASYNC</dispatcher>`  到 filter mapping



---

Spring MVC

mvc配置 公开了下面 与异步请求处理 相关的选项：

1. java配置：在 WebMvcConfigurer 上使用 configureAsyncSupport 回调。
2. xml命名空间： 在 `<mvc:annotation-driven>` 中使用 `<async-support>`

你可以配置下面的：

1. 默认的 异步请求的 timeout，不设置的话，依赖于 底层的 servlet 容器。
2. AsyncTaskExecutor 用来 阻塞式写入，当 使用Reactive 类型 和 从controller 方法 返回的 Callable 实例 进行 streaming 时。我们 强烈建议 配置 这个属性 如果你 通过 响应式类型 来 streaming 或 控制器方法 返回了 Callable，因为默认下，它是 SimpleAsyncTaskExecutor.
3. DeferredResultProcessingInterceptor 实现 和 CallableProcessingInterceptor 实现。



注意，你也可以在 DeferredResult, ResponseBodyEmitter, SseEmitter 的 默认 timeout。 对于 Callable，你可以使用 WebAsyncTask 提供 timeout 值。



---

1.7 CORS 跨域资源共享

mvc 让你 处理 cors (cross-origin resource sharing)。 本节描述如何做



---

1.7.1 Introduction

因为安全原因，浏览器 禁止 ajax 调用 当前源之外的 资源。例如，你可以将你的 bank account 放在一个 选项卡(tab)中，而将 evil.com 放在 另一个 选项卡(tab)中。从 evil.com 的 脚本 不应该能够  使用你的 凭证 发出 ajax 请求 到你的 bank api。

CORS 是大多数浏览器实现的  W3C 规范， 它允许你 指定 授权哪种类型的 跨域请求，而不是基于 IFRAME 或 JSONP 的 不太安全的 且功能较弱的 解决方案。



---

1.7.2 Processing

CORS 规范区分了 preflight/预检，simple，actual 请求。

mvc 的 HandlerMapping 实现 提供了 对 CORS 的内置 支持。在成功映射request 到 handler 后，HandlerMaping 会检查 给定请求和handler的 CORS 配置 并采取进一步操作。preflight 请求被直接处理，而simple 和 actual 的 CORS 请求 被拦截，验证并设置了 所需的CORS 响应 header。

为了启用 cross-origin request ( 跨域请求，即存在 Origin 头 且 和 request的 host 不同)， 你需要有一些 明确声明的 COSR 配置。如果没有找到匹配的CORS 配置，则会拒绝 preflight 请求。simple 和actual CORS 请求的 响应  没有被添加 CORS 头，因此，浏览器会拒绝它们。

每个HandlerMapping 都可以使用 基于URL pattern的 CorsConfiguration 映射  来进行 单独配置。在大多数情况下，app 使用 mvc java 配置 或 xml 命名空间 来 声明这种映射，这会导致 单个全局 映射 被传递到 所有 HandlerMapping 实例。

你可以将 HandlerMapping 级别的 全局CORS 配置 和 更细粒度的，handler级别的 CORS 配置 相结合。例如，带注解的controller 可以使用 类或 方法级别的 @CrossOrigin 注解。( 其他handler 可以实现 CorsConfigurationSource)。

组合全局和本地配置 的规则 通常是 相加的，例如，所有 全局 和 所有本地 源。对于那些只能接受 单个值的 属性，例如 allowCredentials 和 maxAge，本地覆盖 全局值。 更多信息，看 CorsConfiguration#combine(CorsConfiguration)

要从源代码中了解更多信息 或进行高级自定义，请查看下面的源码：

1. CorsConfiguration
2. CorsProcessor, DefaultCorsProcessor
3. AbstractHandlerMapping



---

1.7.3 @CrossOrigin

这个注解 启用 跨域请求 在 被注解的 controller 方法：

```java
@RestController
@RequestMapping("/account")
public class AccountController {

    @CrossOrigin
    @GetMapping("/{id}")
    public Account retrieve(@PathVariable Long id) {
        // ...
    }

    @DeleteMapping("/{id}")
    public void remove(@PathVariable Long id) {
        // ...
    }
}
```

默认，@CrossOrigin 允许：

所有 源

所有 header

这个controller方法 映射的 所有 http method



默认下，allowCredentials 没有被启用，因为它建立了一个 信任级别，该级别会公开敏感的用户特定信息( 如，cookie，CSRF 令牌) ，并且只能在适当的情况下 使用。 启用后，allowOrigins 必须设置为 一个 或多个 特定域 ( 但不是 特殊值 "*")，或者 allowOriginPatterns 属性 可以用于 匹配一组动态 源。

maxAge 是 30分钟。



@CrossOrigin 也可以放在 类级别，被所有方法 继承：

```java
@CrossOrigin(origins = "https://domain2.com", maxAge = 3600)
@RestController
@RequestMapping("/account")
public class AccountController {

    @GetMapping("/{id}")
    public Account retrieve(@PathVariable Long id) {
        // ...
    }

    @DeleteMapping("/{id}")
    public void remove(@PathVariable Long id) {
        // ...
    }
}
```

可以在 类 和 方法 上 都用

```java
@CrossOrigin(maxAge = 3600)
@RestController
@RequestMapping("/account")
public class AccountController {

    @CrossOrigin("https://domain2.com")
    @GetMapping("/{id}")
    public Account retrieve(@PathVariable Long id) {
        // ...
    }

    @DeleteMapping("/{id}")
    public void remove(@PathVariable Long id) {
        // ...
    }
}
```



---

1.7.4 Global Configuration

除了 细粒度的 控制器方法 级别配置之外，你可能还想定义一些全局 cors 配置。你可以在任何 HandlerMapping 上单独 配置 基于URL的 CorsConfiguration 映射。但是，大多数app，使用java配置 或 xml命名空间 来这样做。

默认下，全局配置 启用下面：

所有 源

所有 头

get, head, post 方法



默认下，allowCredentials 没有被启用，因为它建立了一个 信任级别，该级别会公开敏感的用户特定信息( 如，cookie，CSRF 令牌) ，并且只能在适当的情况下 使用。 启用后，allowOrigins 必须设置为 一个 或多个 特定域 ( 但不是 特殊值 "*")，或者 allowOriginPatterns 属性 可以用于 匹配一组动态 源。

maxAge 是 30分钟。



---

Java Configuration

要通过 mvc java 配置 启用 CORS，你可以使用 CorsRegistry 回调：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {

        registry.addMapping("/api/**")
            .allowedOrigins("https://domain2.com")
            .allowedMethods("PUT", "DELETE")
            .allowedHeaders("header1", "header2", "header3")
            .exposedHeaders("header1", "header2")
            .allowCredentials(true).maxAge(3600);

        // Add more mappings...
    }
}
```



---

XML Configuration

要启用 CORS 在 xml 命名空间， 你可以使用 `<mvc:cors>` 

```xml
<mvc:cors>

    <mvc:mapping path="/api/**"
        allowed-origins="https://domain1.com, https://domain2.com"
        allowed-methods="GET, PUT"
        allowed-headers="header1, header2, header3"
        exposed-headers="header1, header2" allow-credentials="true"
        max-age="123" />

    <mvc:mapping path="/resources/**"
        allowed-origins="https://domain1.com" />

</mvc:cors>
```



---

1.7.5 CORS Filter

应用core 支持 通过 内置的 CorsFilter

如果你尝试 使用 CorsFilter with spring security，记住：spring security 内置了 对 cors 的支持。

要配置 filter， 传递一个 CorsConfigurationSource 到 它的 构造器

```java
CorsConfiguration config = new CorsConfiguration();

// Possibly...
// config.applyPermitDefaultValues()

config.setAllowCredentials(true);
config.addAllowedOrigin("https://domain1.com");
config.addAllowedHeader("*");
config.addAllowedMethod("*");

UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
source.registerCorsConfiguration("/**", config);

CorsFilter filter = new CorsFilter(source);
```





---

#### 1.8 Web Security

spring security 工程 为 保护web app 免受恶意攻击 提供了 支持。请看 spring security 文档，包括：

spring mvc security

spring mvc test support

csrf protection

HDIV 是另一个 和 spring mvc 集成的 web 安全框架。



---

#### 1.9 Http Caching

http caching 可以显著提高 web app 性能。 http caching 围绕 Cache-Control 响应头 及 后续的 条件请求头 ( 如 Last-Modified ,ETag )。Cache-Control 建议 私有 (如，浏览器) 和 公共 (如代理) 缓存 如何缓存和 重用响应。ETag用于 发出条件请求，如果 内容未更改，则可能导致 没有 body的 304 (not_modified) ，ETag 可以看做是 Last-Modified 头的 更复杂的 继承者。

本节描述 mvc 中可用的 http caching 选项。



---

1.9.1 CacheControl

CacheControl 提供了 对Cache-Control 头的 配置 的 支持，在许多地方 通过 参数 接收：

1. WebContentInterceptor
2. WebContentGenerator
3. Controllers
4. Static Resources



虽然 RFC 7234 描述了 Cache-Control 响应头 的 所有可能指令，但 CacheControl 采用 面向case 的方法，专注于 常见场景：

```java
// Cache for an hour - "Cache-Control: max-age=3600"
CacheControl ccCacheOneHour = CacheControl.maxAge(1, TimeUnit.HOURS);

// Prevent caching - "Cache-Control: no-store"
CacheControl ccNoStore = CacheControl.noStore();

// Cache for ten days in public and private caches,
// public caches should not transform the response
// "Cache-Control: max-age=864000, public, no-transform"
CacheControl ccCustom = CacheControl.maxAge(10, TimeUnit.DAYS).noTransform().cachePublic();
```



WebContentGenerator 也接受 更简单的 cachePeriod 属性 ( 按秒) ：

1.  -1： 不会生成 Cache-Control 响应头
2. 0： 阻止缓存 通过使用 `Cache-Control: no-store` 
3. n > 0 ： 缓存指定 响应 n秒，通过 `Cache-Control: max-age=n`



---

1.9.2 Controllers

controller 可以添加对 http 缓存的 显示支持。我们建议这样做 (。。指 前面一句)，因为需要先计算资源的 lastModified 或 ETag，然后才能将其 与 conditional request header 对比。控制器可以将 etag 和 cache-control 设置 添加到 ResponseEntity ：

```java
@GetMapping("/book/{id}")
public ResponseEntity<Book> showBook(@PathVariable Long id) {

    Book book = findBook(id);
    String version = book.getVersion();

    return ResponseEntity
            .ok()
            .cacheControl(CacheControl.maxAge(30, TimeUnit.DAYS))
            .eTag(version) // lastModified is also available
            .body(book);
}
```



上面的例子，会发送 不带body的 304(not_modified) 响应，如果 与 conditional request header 的比较 表明内容未更改。 否则，ETag 和Cache-Control 头被添加到 响应中。



你还可以在 controller 中检查 conditional request header：

```java
@RequestMapping
public String myHandleMethod(WebRequest request, Model model) {

    long eTag = ... 		// 特定于app 的计算

    if (request.checkNotModified(eTag)) {
        return null; 		// 响应被 设置为 304 (not_modified) ，没有后续 处理
    }

    model.addAttribute(...); 		// 继续 request 处理。
    return "myViewName";
}
```



有3个变体 用来根据 etag，lastModified 或 2者一起 来检查 conditional request。 对于 get，head 请求，你可以将 响应设置为 304. 对于 post,put,delete，你可以设置为 412 (precondition_failed)， 防止并发修改。



---

1.9.3 Static Resources

你应该使用 Cache-Control 和 conditional response header 为 static resource 提供最佳性能。



1.9.4 ETag filter

你可以使用 ShallowEtagHeaderFilter 来增加 "shallow" eTag 值( 这个值 通过 response 内容 计算得出)，节约流量，但不会节约cpu。



---

---

---



1.10 View Technologies

mvc 中的 view technologies 是可插拔的。 无论你决定使用 Thymleaf，Groovy Markup Template ，JSP，或其它技术，主要取决于 配置的修改。

本章介绍 和spring mvc 集成的 view technologies。

spring mvc 应用的 视图 应该位于该 app的 内部信息 边界内。视图可以访问 app ctx 中所有的 bean。因此，不建议在 模板可以被外部源 编辑的 app 内 使用spring mvc 的模板支持，这会 产生安全隐患。



---

1.10.1 Thymeleaf

是现代的服务器端的 java 模板引擎，它强调自然html 木板，可以通过 双击 在浏览器中 预览，这对于 UI 模板的独立工作 非常有帮助，无需运行服务器。

thymeleaf 和spting mvc 的集成 由 thymeleaf 项目管理。配置 涉及一些 bean 的声明，如 ServletContextTemplateResolver, SpringTemplateEngine,ThymeleafViewResolver ，查看

 https://www.thymeleaf.org/documentation.html 获得更多信息。



---

1.10.2 FreeMarker

---

View Configuration



```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.freeMarker();
    }

    // Configure FreeMarker...

    @Bean
    public FreeMarkerConfigurer freeMarkerConfigurer() {
        FreeMarkerConfigurer configurer = new FreeMarkerConfigurer();
        configurer.setTemplateLoaderPath("/WEB-INF/freemarker");
        return configurer;
    }
}
```

等价物 在 xml

```xml
<mvc:annotation-driven/>

<mvc:view-resolvers>
    <mvc:freemarker/>
</mvc:view-resolvers>

<!-- Configure FreeMarker... -->
<mvc:freemarker-configurer>
    <mvc:template-loader-path location="/WEB-INF/freemarker"/>
</mvc:freemarker-configurer>
```

或者你也可以声明 FreeMarkerConfigueer bean 用来 完全控制 所有 属性：

```xml
<bean id="freemarkerConfig" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
    <property name="templateLoaderPath" value="/WEB-INF/freemarker/"/>
</bean>
```

你的模板需要 存储在 配置的目录中。根据上面的配置，如果你的 控制器 返回 welcome 这个 view name， resolver 会寻找 /WEB-INF/freemarker/welcome.ftl 模板



---

FreeMarker Configuration

可以 传递 FreeMarker 'setting' 和 'sharedVariables' 到 FreeMarker 的 Configuration 对象( 这个被spring 管理)，by 在FreeMarkerConfigurer bean 上 设置 适当的 bean 属性。

freemarkerSettings 属性需要 java.util.Properties 对象， freemarkerVariables 需哟啊 java.util.Map 。下面是 如何使用 FreeMarkerConfigurer:

```xml
<bean id="freemarkerConfig" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
    <property name="templateLoaderPath" value="/WEB-INF/freemarker/"/>
    <property name="freemarkerVariables">
        <map>
            <entry key="xml_escape" value-ref="fmXmlEscape"/>
        </map>
    </property>
</bean>

<bean id="fmXmlEscape" class="freemarker.template.utility.XmlEscape"/>
```

查看FreeMarker 文档 ，查看 应用到 Configuration 对象 settings  variables 的信息



---

Form Handling

spring提供了一个 tag 库，用于 JSP 中，包含了 `<spring:bind />` 元素。这个元素 主要让 form 显示 表单支持的 对象的 值，且 显示来自 web 或 业务层 中 校验失败的 结果。 spring 还支持 freemarker 中想共同的功能，并带有 额外的 便利宏，用于自己生成表单输入元素。



---

The Bind Macros

spring-webmvc.jar 中维护了 用于 freemarker 的一组标准宏，所以它们始终 可用于适当配置的 app。

spring 模板库中 定义的 一些宏 被认为是 内部的( 私有的)，但是 宏定义中不存在这样的 作用域，使得所有宏对 调用代码和 用户模板可见。以下部分仅关注 你需要从模板中直接调用的宏。 

> 如果你需要看所有宏代码，可以看 位于 org.springframework.web.servlet.view.freemarker 包中的 spring.ftl。



---

Simple Binding

在你的 基于 FreeMarker 模板 ( 充当mvc controller 的 view) 的 html form 中，你可以使用类似于下面的 代码绑定到字段值 并以与jsp 等效的方式 类似的方式 显示 每个输入字段的 错误消息。

```xml
<!-- FreeMarker macros have to be imported into a namespace.
    We strongly recommend sticking to 'spring'. -->
<#import "/spring.ftl" as spring/>
<html>
    ...
    <form action="" method="POST">
        Name:
        <@spring.bind "personForm.name"/>
        <input type="text"
            name="${spring.status.expression}"
            value="${spring.status.value?html}"/><br />
        <#list spring.status.errorMessages as error> <b>${error}</b> <br /> </#list>
        <br />
        ...
        <input type="submit" value="submit"/>
    </form>
    ...
</html>
```

`<@spring.bind>` 需要 path 参数，它有 命令对象的名称 后跟一个 句号 和 字段名称 组合 你希望绑定的命令对象。你还可以使用 嵌套字段，例如，command.address.street。 bind 宏采用了 web.xml 中ServletContext 参数 defaultHtmlEscape 指定的 默认 html 转义行为。

宏的另一种形式称为 `<@spring.bindEscaped>` 采用第二个参数，这个参数明确指定 是否应该 在 状态错误消息或 值 中使用 html 转义。你可以根据需要将其设置为 true / false。额外的 处理form 的宏 简化了html 转义，你应该尽可能使用这些宏。 它们在 下一节中 解释。



---

Input Macro

FreeMarker 的 其他便利宏 简化了 绑定 和 表单生成 ( 包括 显示 校验错误信息)。从来没有必要 使用这些 宏来生成 表单输入字段，你可以将它们与简单 html 混合和匹配，或直接调用我们之前强调的 spring bind 宏。

下面是 FreeMarker Template 可用的宏 定义和 参数列表

| macro                                                        | FTL definition                                               |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `message` (output a string from a resource bundle based on the code parameter) | <@spring.message code/>                                      |
| `messageText` (output a string from a resource bundle based on the code parameter, falling back to the value of the default parameter) | <@spring.messageText code, text/>                            |
| `url` (prefix a relative URL with the application’s context root) | <@spring.url relativeUrl/>                                   |
| `formInput` (standard input field for gathering user input)  | <@spring.formInput path, attributes, fieldType/>             |
| `formHiddenInput` (hidden input field for submitting non-user input) | <@spring.formHiddenInput path, attributes/>                  |
| `formPasswordInput` (standard input field for gathering passwords. Note that no value is ever populated in fields of this type.) | <@spring.formPasswordInput path, attributes/>                |
| `formTextarea` (large text field for gathering long, freeform text input) | <@spring.formTextarea path, attributes/>                     |
| `formSingleSelect` (drop down box of options that let a single required value be selected) | <@spring.formSingleSelect path, options, attributes/>        |
| `formMultiSelect` (a list box of options that let the user select 0 or more values) | <@spring.formMultiSelect path, options, attributes/>         |
| `formRadioButtons` (a set of radio buttons that let a single selection be made from the available choices) | <@spring.formRadioButtons path, options separator, attributes/> |
| `formCheckboxes` (a set of checkboxes that let 0 or more values be selected) | <@spring.formCheckboxes path, options, separator, attributes/> |
| `formCheckbox` (a single checkbox)                           | <@spring.formCheckbox path, attributes/>                     |
| `showErrors` (simplify display of validation errors for the bound field) | <@spring.showErrors separator, classOrStyle/>                |

在freemarker 模板中， formHiddenInput 和 formPasswordInput 实际上是不需要的。因为你可以使用普通 formInput 宏，指定 hidden 或 password  作为 fieldType 的值。



上述宏 中的参数的 含义是一致的：

1. path：将绑定的 属性的名字 ( 如 command.name)
2. options：Map，输入字段中 的 所有可用值。 key代表 从表单 post 并绑定到 command 对象的值。。。xxxxxx。。可以使用任何map实现，如果有 顺序要求，使用 带合适Comparator的 SortedMap ( 如 TreeMap)，对于应该按插入顺序返回值的 map，应该使用 LinkedHashMap 或 LinkedMap。
3. separator：在多个 选项可用时，做为 discreet (谨慎的，周到的) 元素 ( 单选按钮或 复选框)，用来分隔列表中每个选项的 字符序列 ( 如 `<br />`)
4. attributes：要包含在 html tag 本身中的 任意标记或文本的 附加字符串。这个string 由macro 逐字 echo。例如，在 textarea 属性中，你可能提供 属性 ( 如 `rows="5" cols="60"`)，或你可以传递 style 信息，如 `style="border:1px solid sliver"`。
5. classOrStyle：对于 showErrors 宏，(classOrStyle 的值是 ) 包含每个错误的 span 元素使用 的CSS类的名称。如果没有提供任何 信息 ( 或值为空)，则错误将被包含在 `<b></b>` 中。



下面部分概述了 宏的例子

Input Fields

`formInput` 宏 接受 path 参数 ( command.name ) 和 额外的 attributes 参数 (下面的例子中是空的)。该宏和所有其他表单生成宏 一起 对路径参数 执行 隐式 spring 绑定。绑定在 新的绑定 发生前一直有效，应次 showErrors 宏 不需要再次传递 path 参数 -- 它对上次创建的绑定的字段 进行操作。

showErrors 接受一个 分隔符参数 ( 用于分隔 给定字段上的 多个错误的 字符。)，还接受第二个参数 -- 这次是类名或 样式属性。注意，FreeMarker 可以为 attributes 参数 指定 默认值。

```xml
<@spring.formInput "command.name"/>
<@spring.showErrors "<br>"/>
```



下面的例子展示了 form 部分 的输出，生成了 name属性 且展现了 检验错误的信息，

```jsp
Name:
<input type="text" name="name" value="">
<br>
    <b>required</b>
<br>
<br>
```



formTextarea 宏 和 formInput 工作方式相同，接受 同样的 参数列表。

一般，第二个参数( attributes ) 用来传递 style 信息 或 textarea的 rows 和 cols 属性。



Selection Fields

你可以使用4个 selection field 宏 来 生成 普通UI 值选择器输入，在你的 html 表单中。

formSingleSelect

formMultiSelect

formRadioButtons

formCheckboxes

上面的每个 都接受 Map 作为选项，包含 表单字段的值 和 对应该值的 label。 值 和label 可以相同。

下面的例子是 radio bottons in FTL，表单背后的对象 指定了 默认的 `London` for 这个字段，因此无需验证。呈现表单时，表单中可选的所有城市列表 作为模型中的ref 数据，名字是 cityMap ：

```jsp
...
Town:
<@spring.formRadioButtons "command.address.town", cityMap, ""/><br><br>
```

上面的清单 呈现了一行单选按钮，每个cityMap的值 有一个按钮，并使用了 "" 作为 分隔符。没有提供其他属性( 因为没有提供宏的最后一个参数)。cityMap 使用 map中每个 k-v 对 相同的 String。map的 key 是 表单 POST提交的内容。map的value 是用户看到的。上面的例子中，给出3个城市的列表 和 表单支持对象的默认值，html类似：

```jsp
Town:
<input type="radio" name="address.town" value="London">London</input>
<input type="radio" name="address.town" value="Paris" checked="checked">Paris</input>
<input type="radio" name="address.town" value="New York">New York</input>
```



如果你的app 希望通过内部代码 处理城市， 你可以使用合适的 key 来创建代码映射：

```java
protected Map<String, ?> referenceData(HttpServletRequest request) throws Exception {
    Map<String, String> cityMap = new LinkedHashMap<>();
    cityMap.put("LDN", "London");
    cityMap.put("PRS", "Paris");
    cityMap.put("NYC", "New York");

    Map<String, Object> model = new HashMap<>();
    model.put("cityMap", cityMap);
    return model;
}
```

下面是html，用户看到的 没有变

```jsp
Town:
<input type="radio" name="address.town" value="LDN">London</input>
<input type="radio" name="address.town" value="PRS" checked="checked">Paris</input>
<input type="radio" name="address.town" value="NYC">New York</input>
```



---

HTML Escaping

前面表述的 表单宏 的默认使用 会导致 使用 html 4.01 标准的 元素，并且 使用 web.xml 中定义的 html 转义字符的默认值，就像 spring 的bind 支持 使用的。

要 使用 XHTML 标准 或 覆盖默认的 转义字符，你可以在 你的 模板 内 指定 2个变量 ( 或在模型中，它们对模板可见)。在木板中指定的好处是，它们可以在 木板处理的 后期 更改为不同的值，以便为 表单中的不同字段提供 不同的行为。

将你的 tag 切换为 XHTML，指定 xhtmlCompliant 为true， 这个 xhtmlCompliant 可以放在 model 或 context 中。

```jsp
<#-- for FreeMarker -->
<#assign xhtmlCompliant = true>
```

你可以指定 为每个字段指定  html 转义字符

```jsp
<#-- until this point, default HTML escaping is used -->

<#assign htmlEscape = true>
<#-- next field will use HTML escaping -->
<@spring.formInput "command.name"/>

<#assign htmlEscape = false in spring>
<#-- all future fields will be bound with HTML escaping off -->
```



---

1.10.3 Groovy Markup

。。随便抄点代码。。



```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.groovy();
    }

    // Configure the Groovy Markup Template Engine...

    @Bean
    public GroovyMarkupConfigurer groovyMarkupConfigurer() {
        GroovyMarkupConfigurer configurer = new GroovyMarkupConfigurer();
        configurer.setResourceLoaderPath("/WEB-INF/");
        return configurer;
    }
}
```

```xml
<mvc:annotation-driven/>

<mvc:view-resolvers>
    <mvc:groovy/>
</mvc:view-resolvers>

<!-- Configure the Groovy Markup Template Engine... -->
<mvc:groovy-configurer resource-loader-path="/WEB-INF/"/>
```

```groovy
yieldUnescaped '<!DOCTYPE html>'
html(lang:'en') {
    head {
        meta('http-equiv':'"Content-Type" content="text/html; charset=utf-8"')
        title('My page')
    }
    body {
        p('This is an example of HTML contents')
    }
}
```



---

1.10.4 Script Views

。。有点吊。

spring 提供了 内置集成，这样 spring mvc 可以使用 任何 可以运行在JSR-223 java scripting 引擎 上的 模板库。

| Scripting Library                                            | Scripting Engine                                      |
| ------------------------------------------------------------ | ----------------------------------------------------- |
| [Handlebars](https://handlebarsjs.com/)                      | [Nashorn](https://openjdk.java.net/projects/nashorn/) |
| [Mustache](https://mustache.github.io/)                      | [Nashorn](https://openjdk.java.net/projects/nashorn/) |
| [React](https://facebook.github.io/react/)                   | [Nashorn](https://openjdk.java.net/projects/nashorn/) |
| [EJS](https://www.embeddedjs.com/)                           | [Nashorn](https://openjdk.java.net/projects/nashorn/) |
| [ERB](https://www.stuartellis.name/articles/erb/)            | [JRuby](https://www.jruby.org)                        |
| [String templates](https://docs.python.org/2/library/string.html#template-strings) | [Jython](https://www.jython.org/)                     |
| [Kotlin Script templating](https://github.com/sdeleuze/kotlin-script-templating) | [Kotlin](https://kotlinlang.org/)                     |



```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.scriptTemplate();
    }

    @Bean
    public ScriptTemplateConfigurer configurer() {
        ScriptTemplateConfigurer configurer = new ScriptTemplateConfigurer();
        configurer.setEngineName("nashorn");
        configurer.setScripts("mustache.js");
        configurer.setRenderObject("Mustache");
        configurer.setRenderFunction("render");
        return configurer;
    }
}
```



```xml
<mvc:annotation-driven/>

<mvc:view-resolvers>
    <mvc:script-template/>
</mvc:view-resolvers>

<mvc:script-template-configurer engine-name="nashorn" render-object="Mustache" render-function="render">
    <mvc:script location="mustache.js"/>
</mvc:script-template-configurer>
```



```java
@Controller
public class SampleController {

    @GetMapping("/sample")
    public String test(Model model) {
        model.addAttribute("title", "Sample title");
        model.addAttribute("body", "Sample body");
        return "template";
    }
}
```

```html
<html>
    <head>
        <title>{{title}}</title>
    </head>
    <body>
        <p>{{body}}</p>
    </body>
</html>
```





```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.scriptTemplate();
    }

    @Bean
    public ScriptTemplateConfigurer configurer() {
        ScriptTemplateConfigurer configurer = new ScriptTemplateConfigurer();
        configurer.setEngineName("nashorn");
        configurer.setScripts("polyfill.js", "handlebars.js", "render.js");
        configurer.setRenderFunction("render");
        configurer.setSharedEngine(false);   // 。。好像是要使用自定义 js，就要设置
        return configurer;
    }
}
```



---

1.10.5 JSP and JSTL

```xml
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

```xml
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
```



```xml
<form:form>
    <table>
        <tr>
            <td>First Name:</td>
            <td><form:input path="firstName"/></td>
        </tr>
        <tr>
            <td>Last Name:</td>
            <td><form:input path="lastName"/></td>
        </tr>
        <tr>
            <td colspan="2">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form:form>
```

```xml
<form method="POST">
    <table>
        <tr>
            <td>First Name:</td>
            <td><input name="firstName" type="text" value="Harry"/></td>
        </tr>
        <tr>
            <td>Last Name:</td>
            <td><input name="lastName" type="text" value="Potter"/></td>
        </tr>
        <tr>
            <td colspan="2">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form>
```

```xml
<form:form modelAttribute="user">     // .. qu bie
    <table>
        <tr>
            <td>First Name:</td>
            <td><form:input path="firstName"/></td>
        </tr>
        <tr>
            <td>Last Name:</td>
            <td><form:input path="lastName"/></td>
        </tr>
        <tr>
            <td colspan="2">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form:form>
```



input 标签

checkbox 标签



```java
public class Preferences {

    private boolean receiveNewsletter;
    private String[] interests;
    private String favouriteWord;

    public boolean isReceiveNewsletter() {
        return receiveNewsletter;
    }

    public void setReceiveNewsletter(boolean receiveNewsletter) {
        this.receiveNewsletter = receiveNewsletter;
    }

    public String[] getInterests() {
        return interests;
    }

    public void setInterests(String[] interests) {
        this.interests = interests;
    }

    public String getFavouriteWord() {
        return favouriteWord;
    }

    public void setFavouriteWord(String favouriteWord) {
        this.favouriteWord = favouriteWord;
    }
}
```



```xml
<form:form>
    <table>
        <tr>
            <td>Subscribe to newsletter?:</td>
            <%-- Approach 1: Property is of type java.lang.Boolean --%>
            <td><form:checkbox path="preferences.receiveNewsletter"/></td>
        </tr>

        <tr>
            <td>Interests:</td>
            <%-- Approach 2: Property is of an array or of type java.util.Collection --%>
            <td>
                Quidditch: <form:checkbox path="preferences.interests" value="Quidditch"/>
                Herbology: <form:checkbox path="preferences.interests" value="Herbology"/>
                Defence Against the Dark Arts: <form:checkbox path="preferences.interests" value="Defence Against the Dark Arts"/>
            </td>
        </tr>

        <tr>
            <td>Favourite Word:</td>
            <%-- Approach 3: Property is of type java.lang.Object --%>
            <td>
                Magic: <form:checkbox path="preferences.favouriteWord" value="Magic"/>
            </td>
        </tr>
    </table>
</form:form>
```



```xml
<tr>
    <td>Interests:</td>
    <td>
        Quidditch: <input name="preferences.interests" type="checkbox" value="Quidditch"/>
        <input type="hidden" value="1" name="_preferences.interests"/>
        Herbology: <input name="preferences.interests" type="checkbox" value="Herbology"/>
        <input type="hidden" value="1" name="_preferences.interests"/>
        Defence Against the Dark Arts: <input name="preferences.interests" type="checkbox" value="Defence Against the Dark Arts"/>
        <input type="hidden" value="1" name="_preferences.interests"/>
    </td>
</tr>
```



checkboxes 标签

```xml
<form:form>
    <table>
        <tr>
            <td>Interests:</td>
            <td>
                <%-- Property is of an array or of type java.util.Collection --%>
                <form:checkboxes path="preferences.interests" items="${interestList}"/>
            </td>
        </tr>
    </table>
</form:form>
```



radiobutton

```xml
<tr>
    <td>Sex:</td>
    <td>
        Male: <form:radiobutton path="sex" value="M"/> <br/>
        Female: <form:radiobutton path="sex" value="F"/>
    </td>
</tr>
```



radiobuttons

```xml
<tr>
    <td>Sex:</td>
    <td><form:radiobuttons path="sex" items="${sexOptions}"/></td>
</tr>
```

```xml
<tr>
    <td>Password:</td>
    <td>
        <form:password path="password"/>
    </td>
</tr>
```

```xml
<tr>
    <td>Password:</td>
    <td>
        <form:password path="password" value="^76525bvHGq" showPassword="true"/>
    </td>
</tr>
```



```xml
<tr>
    <td>Skills:</td>
    <td><form:select path="skills" items="${skills}"/></td>
</tr>
```

```xml
<tr>
    <td>Skills:</td>
    <td>
        <select name="skills" multiple="true">
            <option value="Potions">Potions</option>
            <option value="Herbology" selected="selected">Herbology</option>
            <option value="Quidditch">Quidditch</option>
        </select>
    </td>
</tr>
```



```xml
<tr>
    <td>House:</td>
    <td>
        <form:select path="house">
            <form:option value="Gryffindor"/>
            <form:option value="Hufflepuff"/>
            <form:option value="Ravenclaw"/>
            <form:option value="Slytherin"/>
        </form:select>
    </td>
</tr>
```



```xml
<tr>
    <td>House:</td>
    <td>
        <select name="house">
            <option value="Gryffindor" selected="selected">Gryffindor</option> 
            <option value="Hufflepuff">Hufflepuff</option>
            <option value="Ravenclaw">Ravenclaw</option>
            <option value="Slytherin">Slytherin</option>
        </select>
    </td>
</tr>
```

```xml
<tr>
    <td>Country:</td>
    <td>
        <form:select path="country">
            <form:option value="-" label="--Please Select"/>
            <form:options items="${countryList}" itemValue="code" itemLabel="name"/>
        </form:select>
    </td>
</tr>
```



```xml
<tr>
    <td>Country:</td>
    <td>
        <select name="country">
            <option value="-">--Please Select</option>
            <option value="AT">Austria</option>
            <option value="UK" selected="selected">United Kingdom</option> 
            <option value="US">United States</option>
        </select>
    </td>
</tr>
```



```xml
<tr>
    <td>Notes:</td>
    <td><form:textarea path="notes" rows="3" cols="20"/></td>
    <td><form:errors path="notes"/></td>
</tr>
```



```xml
<form:hidden path="house"/>
```



```xml
<input name="house" type="hidden" value="Gryffindor"/>
```



```java
public class UserValidator implements Validator {

    public boolean supports(Class candidate) {
        return User.class.isAssignableFrom(candidate);
    }

    public void validate(Object obj, Errors errors) {
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "firstName", "required", "Field is required.");
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "lastName", "required", "Field is required.");
    }
}
```

```xml
<form:form>
    <table>
        <tr>
            <td>First Name:</td>
            <td><form:input path="firstName"/></td>
            <%-- Show errors for firstName field --%>
            <td><form:errors path="firstName"/></td>
        </tr>

        <tr>
            <td>Last Name:</td>
            <td><form:input path="lastName"/></td>
            <%-- Show errors for lastName field --%>
            <td><form:errors path="lastName"/></td>
        </tr>
        <tr>
            <td colspan="3">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form:form>
```



```xml
<form method="POST">
    <table>
        <tr>
            <td>First Name:</td>
            <td><input name="firstName" type="text" value=""/></td>
            <%-- Associated errors to firstName field displayed --%>
            <td><span name="firstName.errors">Field is required.</span></td>
        </tr>

        <tr>
            <td>Last Name:</td>
            <td><input name="lastName" type="text" value=""/></td>
            <%-- Associated errors to lastName field displayed --%>
            <td><span name="lastName.errors">Field is required.</span></td>
        </tr>
        <tr>
            <td colspan="3">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form>
```



```xml
<form:form>
    <form:errors path="*" cssClass="errorBox"/>
    <table>
        <tr>
            <td>First Name:</td>
            <td><form:input path="firstName"/></td>
            <td><form:errors path="firstName"/></td>
        </tr>
        <tr>
            <td>Last Name:</td>
            <td><form:input path="lastName"/></td>
            <td><form:errors path="lastName"/></td>
        </tr>
        <tr>
            <td colspan="3">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form:form>
```

```xml
<form method="POST">
    <span name="*.errors" class="errorBox">Field is required.<br/>Field is required.</span>
    <table>
        <tr>
            <td>First Name:</td>
            <td><input name="firstName" type="text" value=""/></td>
            <td><span name="firstName.errors">Field is required.</span></td>
        </tr>

        <tr>
            <td>Last Name:</td>
            <td><input name="lastName" type="text" value=""/></td>
            <td><span name="lastName.errors">Field is required.</span></td>
        </tr>
        <tr>
            <td colspan="3">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form>
```



HTTP Method Conversion

```xml
<form:form method="delete">
    <p class="submit"><input type="submit" value="Delete Pet"/></p>
</form:form>
```

```xml
<filter>
    <filter-name>httpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>

<filter-mapping>
    <filter-name>httpMethodFilter</filter-name>
    <servlet-name>petclinic</servlet-name>
</filter-mapping>
```

```java
@RequestMapping(method = RequestMethod.DELETE)
public String deletePet(@PathVariable int ownerId, @PathVariable int petId) {
    this.clinic.deletePet(petId);
    return "redirect:/owners/" + ownerId;
}
```



spring form tag 库 可以 输入动态属性，这意味着你可以 输入任何 html5 的属性。



---

1.10.6 Tiles

```xml
<bean id="tilesConfigurer" class="org.springframework.web.servlet.view.tiles3.TilesConfigurer">
    <property name="definitions">
        <list>
            <value>/WEB-INF/defs/general.xml</value>
            <value>/WEB-INF/defs/widgets.xml</value>
            <value>/WEB-INF/defs/administrator.xml</value>
            <value>/WEB-INF/defs/customer.xml</value>
            <value>/WEB-INF/defs/templates.xml</value>
        </list>
    </property>
</bean>
```

```xml
<bean id="tilesConfigurer" class="org.springframework.web.servlet.view.tiles3.TilesConfigurer">
    <property name="definitions">
        <list>
            <value>/WEB-INF/defs/tiles.xml</value>
            <value>/WEB-INF/defs/tiles_fr_FR.xml</value>
        </list>
    </property>
</bean>
```

```xml
<bean id="viewResolver" class="org.springframework.web.servlet.view.UrlBasedViewResolver">
    <property name="viewClass" value="org.springframework.web.servlet.view.tiles3.TilesView"/>
</bean>
```

```xml
<bean id="tilesConfigurer" class="org.springframework.web.servlet.view.tiles3.TilesConfigurer">
    <property name="definitions">
        <list>
            <value>/WEB-INF/defs/general.xml</value>
            <value>/WEB-INF/defs/widgets.xml</value>
            <value>/WEB-INF/defs/administrator.xml</value>
            <value>/WEB-INF/defs/customer.xml</value>
            <value>/WEB-INF/defs/templates.xml</value>
        </list>
    </property>

    <!-- resolving preparer names as Spring bean definition names -->
    <property name="preparerFactoryClass"
            value="org.springframework.web.servlet.view.tiles3.SpringBeanPreparerFactory"/>

</bean>
```



---

1.10.7 RSS and Atom

```java
public class SampleContentAtomView extends AbstractAtomFeedView {

    @Override
    protected void buildFeedMetadata(Map<String, Object> model,
            Feed feed, HttpServletRequest request) {
        // implementation omitted
    }

    @Override
    protected List<Entry> buildFeedEntries(Map<String, Object> model,
            HttpServletRequest request, HttpServletResponse response) throws Exception {
        // implementation omitted
    }
}
```

```java
public class SampleContentRssView extends AbstractRssFeedView {

    @Override
    protected void buildFeedMetadata(Map<String, Object> model,
            Channel feed, HttpServletRequest request) {
        // implementation omitted
    }

    @Override
    protected List<Item> buildFeedItems(Map<String, Object> model,
            HttpServletRequest request, HttpServletResponse response) throws Exception {
        // implementation omitted
    }
}
```



---

1.10.8 PDF and Excel

```java
public class PdfWordList extends AbstractPdfView {

    protected void buildPdfDocument(Map<String, Object> model, Document doc, PdfWriter writer,
            HttpServletRequest request, HttpServletResponse response) throws Exception {

        List<String> words = (List<String>) model.get("wordList");
        for (String word : words) {
            doc.add(new Paragraph(word));
        }
    }
}
```



```
AbstractXlsView
```



---

1.10.9 Jackson

```
MappingJackson2JsonView
```

```
extractValueFromSingleKeyModel
```

```
ObjectMapper
```

```
MappingJackson2XmlView
```

```
XmlMapper
```

```
modelKey
```



---

1.10.10 XML Matshalling

```
MarshallingView
```

```
Marshaller
```

```
modelKey
```



---

1.10.11 XSLT Views

```java
@EnableWebMvc
@ComponentScan
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Bean
    public XsltViewResolver xsltViewResolver() {
        XsltViewResolver viewResolver = new XsltViewResolver();
        viewResolver.setPrefix("/WEB-INF/xsl/");
        viewResolver.setSuffix(".xslt");
        return viewResolver;
    }
}
```

```java
@Controller
public class XsltController {

    @RequestMapping("/")
    public String home(Model model) throws Exception {
        Document document = DocumentBuilderFactory.newInstance().newDocumentBuilder().newDocument();
        Element root = document.createElement("wordList");

        List<String> words = Arrays.asList("Hello", "Spring", "Framework");
        for (String word : words) {
            Element wordNode = document.createElement("word");
            Text textNode = document.createTextNode(word);
            wordNode.appendChild(textNode);
            root.appendChild(wordNode);
        }

        model.addAttribute("wordList", root);
        return "home";
    }
}
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">

    <xsl:output method="html" omit-xml-declaration="yes"/>

    <xsl:template match="/">
        <html>
            <head><title>Hello!</title></head>
            <body>
                <h1>My First Words</h1>
                <ul>
                    <xsl:apply-templates/>
                </ul>
            </body>
        </html>
    </xsl:template>

    <xsl:template match="word">
        <li><xsl:value-of select="."/></li>
    </xsl:template>

</xsl:stylesheet>
```

```html
<html>
    <head>
        <META http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>Hello!</title>
    </head>
    <body>
        <h1>My First Words</h1>
        <ul>
            <li>Hello</li>
            <li>Spring</li>
            <li>Framework</li>
        </ul>
    </body>
</html>
```

















----

---

---

---

### 1.11 MVC Config



java 配置和 xml命名空间 提供了 适用于大多数app的 默认配置 和一个 配置api 来定制。

更多的高级定制，不在 这个 配置api 中，查看 Advanced Java Config , 和 Advanced XML Config，这2个分别是 1.11.13 ， 1.11.14

你不需要了解 java 配置 和 xml 命名空间 创建的 底层bean。如果你想，那么看 Special Bean Types, Web MVC Config. 分别是 1.1.2, 1.1.3  (。。。早过了啊。。。)



---

1.11.1 Enable MVC Configuration

@EnableWebMvc 来启用 mvc 的java 配置

```java
@Configuration
@EnableWebMvc
public class WebConfig {
}
```



`<mvc:annotation-driven>` 开启xml

。。mvc 前缀

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:annotation-driven/>

</beans>
```

上面会注册 一些 mvc的 基础bean，并适应 classpath 上可用的 依赖 ( 如，JSON，XML 等的 payload converter)



---

1.11.2 MVC Config API

java配置，你可以实现 WebMvcConfigurer 接口：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    // Implement configuration methods...
}
```



xml中，你可以 查看 `<mvc:annotation-driven>` 的子元素。



---

1.11.3 Type Conversion

默认，安装了 各种数字 和 日期个是的 formatter，并支持 通过 属性 上的 @NumberFormat 和 @DateTimeFormat 进行自定义。

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        // ...
    }
}
```

等价：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:annotation-driven conversion-service="conversionService"/>

    <bean id="conversionService"
            class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        <property name="converters">
            <set>
                <bean class="org.example.MyConverter"/>
            </set>
        </property>
        <property name="formatters">
            <set>
                <bean class="org.example.MyFormatter"/>
                <bean class="org.example.MyAnnotationFormatterFactory"/>
            </set>
        </property>
        <property name="formatterRegistrars">
            <set>
                <bean class="org.example.MyFormatterRegistrar"/>
            </set>
        </property>
    </bean>

</beans>
```



默认，mvc考虑请求的 locale，当转换和 格式化 时间时。这适用于 日期表示为 带有 "input" 表单字段 的 string 的 表单。对于 "date" 和 "time" 表单字段，浏览器使用 html规范中定义的 固定格式。对于这种情况，可以按如下方式 自定义 日期和时间的格式：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        DateTimeFormatterRegistrar registrar = new DateTimeFormatterRegistrar();
        registrar.setUseIsoFormat(true);
        registrar.registerFormatters(registry);
    }
}
```

查看 FormatterRegister ， FormattingConversionServiceFactoryBean 了解更多。



---

1.11.4 Validation

默认，如果 Bean Validation 在 classpath 上可用 (如 Hibernate Validator) ，LocalValidatorFactoryBean 被注册为一个 全局Validator 来 在controller 方法 参数上使用 @Valid 和 Validated

定制 全局 Validator 实例

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public Validator getValidator() {
        // ...
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:annotation-driven validator="globalValidator"/>

</beans>
```



注意，你也可以 本地 注册 Validator ：

```java
@Controller
public class MyController {

    @InitBinder
    protected void initBinder(WebDataBinder binder) {
        binder.addValidators(new FooValidator());
    }
}
```



如果你需要 LocalValidatorFactoryBean 注入到其它地方， 创建一个bean 标记为 @Primary， 来避免和 mvc 配置中 声明的 冲突。



---

1.11.5 Interceptors

java配置，你可以注册 拦截器 来 应用到 输入的request：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LocaleChangeInterceptor());
        registry.addInterceptor(new ThemeChangeInterceptor()).addPathPatterns("/**").excludePathPatterns("/admin/**");
        registry.addInterceptor(new SecurityInterceptor()).addPathPatterns("/secure/*");
    }
}
```



```xml
<mvc:interceptors>
    <bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor"/>
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <mvc:exclude-mapping path="/admin/**"/>
        <bean class="org.springframework.web.servlet.theme.ThemeChangeInterceptor"/>
    </mvc:interceptor>
    <mvc:interceptor>
        <mvc:mapping path="/secure/*"/>
        <bean class="org.example.SecurityInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```



---

1.11.6 Content Types

你可以配置 mvc如何决定 request 的 media type ( 如，Accept 头，URL 路径扩展，查询参数，等)。

默认，只检查 Accept 头。

如果你必须 使用 基于URL 的 content type 方案，考虑 使用 查询参数 策略 而不是 路径扩展。 查看 Suffix Match 和 Suffix Match and RFD 。

java配置，定制 请求的 内容类型解析：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
        configurer.mediaType("json", MediaType.APPLICATION_JSON);
        configurer.mediaType("xml", MediaType.APPLICATION_XML);
    }
}
```

```xml
<mvc:annotation-driven content-negotiation-manager="contentNegotiationManager"/>

<bean id="contentNegotiationManager" class="org.springframework.web.accept.ContentNegotiationManagerFactoryBean">
    <property name="mediaTypes">
        <value>
            json=application/json
            xml=application/xml
        </value>
    </property>
</bean>
```



---

1.11.7 Message Converters

定制 HttpMessageConverter 在 java 配置中 通过 覆盖 configureMessageConverters()方法 ( 来替换 mvc创建的默认的 converter) 或 通过 覆盖 extendMessageConverters() 方法 ( 来定制 默认的 converter 或 增加 额外的 converter 到 默认的)。

下面是 通过自定义的 ObjectMapper 来 添加 xml 和 jackson json converter 而不是 用默认的。

```java
@Configuration
@EnableWebMvc
public class WebConfiguration implements WebMvcConfigurer {

    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        Jackson2ObjectMapperBuilder builder = new Jackson2ObjectMapperBuilder()
                .indentOutput(true)
                .dateFormat(new SimpleDateFormat("yyyy-MM-dd"))
                .modulesToInstall(new ParameterNamesModule());
        converters.add(new MappingJackson2HttpMessageConverter(builder.build()));
        converters.add(new MappingJackson2XmlHttpMessageConverter(builder.createXmlMapper(true).build()));
    }
}
```



上面的例子中， Jackson2ObjectMapperBuilder 用来 创建 通用配置 for MappingJackson2HttpMessageConverter 和 MappingJackson2XmlHttpMessageConverter ，启用 缩进，自定义日期格式 和 注册 jackson-module-parameter-names，这个增加了 对访问参数名称的 支持 (这个功能在jdk8 中添加)。



此builder 定制 Jackson 的 默认属性，如下：

DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES 被禁用

MapperFeature.DEFAULT_VIEW_INCLUSION 被禁用



也自动注册下面的 众所周知的 模块，如果它们 在classpath 中 被发现：

jackson-datatype-joda，支持 Joda-Time

jackson-datatype-jsr310，支持 java 8  日期和时间 api 类

jackson-datatype-jdk8，支持 java8 的类型 如 Optional

jackson-module-kotlin，支持 Kotlin 类和 数据类



除了jackson-dataformat-xml 之外，启用 带有 Jackson XML 支持的 缩进 还需要 woodstox-core-asl 依赖。



其他可用的 有吸引力的 Jackson模块 ：

jackson-datatype-money，支持 javax.money类型( 非官方模块)

jackson-datatype-hibernate，支持特定于 hibernate 的类型 和属性 ( 包括延迟加载 aspect)。



xml中相同配置

```xml
<mvc:annotation-driven>
    <mvc:message-converters>
        <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
            <property name="objectMapper" ref="objectMapper"/>
        </bean>
        <bean class="org.springframework.http.converter.xml.MappingJackson2XmlHttpMessageConverter">
            <property name="objectMapper" ref="xmlMapper"/>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>

<bean id="objectMapper" class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean"
      p:indentOutput="true"
      p:simpleDateFormat="yyyy-MM-dd"
      p:modulesToInstall="com.fasterxml.jackson.module.paramnames.ParameterNamesModule"/>

<bean id="xmlMapper" parent="objectMapper" p:createXmlMapper="true"/>
```



---

1.11.8 View Controllers

快捷的 定义 ParameterizableViewController，这个控制器 在被调用时，立刻转发到 view。 你可以在 static case 中 使用，此时 没有 java controller 逻辑 运行 before view 生成 response。

java配置，转发 对/的请求 到 home 这个view

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("home");
    }
}
```

等价

```xml
<mvc:view-controller path="/" view-name="home"/>
```



如果 @RequestMapping 方法 被映射为 接受任何 http method (。。感觉好像是 只要接受一个 就可以。。) 的URL，则 view controller 不能被用来 处理相同的URL。这是因为 一个 URL 和 注解的controller 的 匹配 被认为 足够指明 端点的所有权了，所以 会 405( method not allowed), 415(unsupported media type) 或相似的响应 会发送到 client。 因为这个原因，建议避免 拆分URL的处理 到 一个注解的controller 和 一个 view controller。

。。应该是指 URL 优先被 controller 匹配， 然后才会被 view controller 匹配。  不过 any http method 是指 任一 还是 全部。。 主要时 405,那么应该是 任一。



---

1.11.9 View Resolvers

mvc 配置 简化了 view resolvers 的注册

下面是java配置， 配置了 内容协商视图 解析 (content negotiation view resolution ) by 使用JSP 和 Jackson 作为 一个 默认的 View 用于 呈现json。

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.enableContentNegotiation(new MappingJackson2JsonView());
        registry.jsp();
    }
}
```

等价

```xml
<mvc:view-resolvers>
    <mvc:content-negotiation>
        <mvc:default-views>
            <bean class="org.springframework.web.servlet.view.json.MappingJackson2JsonView"/>
        </mvc:default-views>
    </mvc:content-negotiation>
    <mvc:jsp/>
</mvc:view-resolvers>
```



但 FreeMaker，Tiles，Groovy Markup，和 script template  也需要 配置底层的 view technology。

mvc 命名空间 提供了 专用元素，下面例子 适用于 FreeMarker：

```xml
<mvc:view-resolvers>
    <mvc:content-negotiation>
        <mvc:default-views>
            <bean class="org.springframework.web.servlet.view.json.MappingJackson2JsonView"/>
        </mvc:default-views>
    </mvc:content-negotiation>
    <mvc:freemarker cache="false"/>
</mvc:view-resolvers>

<mvc:freemarker-configurer>
    <mvc:template-loader-path location="/freemarker"/>
</mvc:freemarker-configurer>
```

java配置，你可以增加 相应的 Configurer bean：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.enableContentNegotiation(new MappingJackson2JsonView());
        registry.freeMarker().cache(false);
    }

    @Bean
    public FreeMarkerConfigurer freeMarkerConfigurer() {
        FreeMarkerConfigurer configurer = new FreeMarkerConfigurer();
        configurer.setTemplateLoaderPath("/freemarker");
        return configurer;
    }
}
```



---

1.11. 10 Staic Resources

这个选项提供了 方便的方法来 提供 静态资源 从 基于Resource 的 地址 列表中。

下一个例子中，给定 以 /resources 开头的请求，使用 相对路径 来查找和提供 静态资源，这些资源 想对于 web app root的 /public  或 classpath 的 /static。资源存放一年，来最大限度使用 浏览器缓存，并减少从浏览器发出的 http 请求。 Last-Modified 信息 是从 Resource#lastModified 推导出来的，所以 http conditional request 支持 Last-Modified 头。

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**")
                .addResourceLocations("/public", "classpath:/static/")
                .setCacheControl(CacheControl.maxAge(Duration.ofDays(365)));
    }
}
```

下面是 xml

```xml
<mvc:resources mapping="/resources/**"
    location="/public, classpath:/static/"
    cache-period="31556926" />
```



资源handler 也支持 ResourceResolver 实现 和 ResouceTransformer 实现的 链式， 你可以使用它们 来创建一个 工具链来 处理 优化的资源。



你可以使用 VersionResourceResolver 用于 基于 从 内容，固定app版本或其他 计算出来的 MD5 的 版本化资源URL。 ContentVersionStrategy 是一个不错的选择-- 除了一些明显的例外：如 与模块加载器 一起使用的 js 资源。



java配置 VersionResourceResolver

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**")
                .addResourceLocations("/public/")
                .resourceChain(true)
                .addResolver(new VersionResourceResolver().addContentVersionStrategy("/**"));
    }
}
```

xml等价

```xml
<mvc:resources mapping="/resources/**" location="/public/">
    <mvc:resource-chain resource-cache="true">
        <mvc:resolvers>
            <mvc:version-resolver>
                <mvc:content-version-strategy patterns="/**"/>
            </mvc:version-resolver>
        </mvc:resolvers>
    </mvc:resource-chain>
</mvc:resources>
```



你可以使用 ResourceUrlProvider 来 重写 URL，且 应用 resolver 和 transformer 的 full/完整 chain -- 例如，来增加 版本。mvc配置 提供了 ResourceUrlProvider bean，这样，它可以被注入到 其他。你也可以 使用 ResourceUrlEncodingFilter 来 为 Thymeleaf，JSP，FreeMarker 等 依赖于HttpServletResponse#encoding 的URL tag  重写 transparent。

注意，当同时使用 EncodedResourceResolver ( 例如，用于提供 gzip的 或 brotli编码的 资源) 和 VersionResourceResolver 时，你必须按此顺序注册它们，这可以确保 基于内容的版本 可以对 未编码的文件 可靠地计算。

WebJars 也被支持 通过 WebJarsResourceResolver，这个会被自动注册，当 org.wegjars:webjars-locator-core 库 出现在 classpath中。解析器可以重写 URL 以包含 jar 的版本，还可以匹配 没有版本的传入的 URL，例如，从 /jquery/jquery.min.js 到 /jqury/1.2.0/jquery.min.js。

基于ResourceHandlerRegistry 的java 配置 为 细粒度控制 提供了 更多选项，例如 last-modifier 行为 和 优化的资源解析。



---

1.11.11 Default Servlet

mvc 允许 映射 DisSer 到 / ，(从而覆盖容器默认servlet 的映射)， 同时仍然允许由容器的默认 Servlet 处理 静态资源请求。它配置了一个 映射/** url路径的 ，并且是 最低优先级mapping 的 DefaultServletHttpRequestHandler。

这个handler 把 所有请球 转发到默认的servlet。因此，它必须处在 所有 URL HandlerMapping 的 最后。如果你使用 `<mvc:annotation-driven>` 就是这样的情况。或者，如果你设置自己的 自定义 HandlerMapping 实例，请确保 它的order 属性 设置为 低于 DefaultServletHttpRequestHandler 的值 ( 它是 Integer.MAX_VALUE)。

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }
}
```

等价

```xml
<mvc:default-servlet-handler/>
```



覆盖 / 的 servlet mapping 的警告 是 默认servlet 的 RequestDispatcher 必须 按名字 而不是 路径检索。在启动时，DefaultSrevletHttpRequestHandler 尝试 自动发现 容器的默认的Servlet，使用大多数主要Servlet 容器( tomcat,jetty,glassfish,jboss,resin,weblogic,websphere)的 已知的名字列表。如果默认 servlet 使用不同的名称 进行自定义配置，或者 如果在默认servlet 名称 未知的情况下 使用 不同的 servlet 容器，则必须显示提供 默认 servlet 的名字：

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable("myCustomDefaultServlet");
    }
}
```



```xml
<mvc:default-servlet-handler default-servlet-name="myCustomDefaultServlet"/>
```



---

1.11.12 Path Matching

你可以自定义 与路径匹配 和 url 处理相关的 选项。有关各个选项的 详细信息，看 PathMatchConfigurer 的 javadoc。

下面是 自定义 path matching

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        configurer
            .setPatternParser(new PathPatternParser())
            .addPathPrefix("/api", HandlerTypePredicate.forAnnotation(RestController.class));
    }

    private PathPatternParser patternParser() {
        // ...
    }
}
```

```xml
<mvc:annotation-driven>
    <mvc:path-matching
        trailing-slash="false"
        path-helper="pathHelper"
        path-matcher="pathMatcher"/>
</mvc:annotation-driven>

<bean id="pathHelper" class="org.example.app.MyPathHelper"/>
<bean id="pathMatcher" class="org.example.app.MyPathMatcher"/>
```



---

1.11.13 Advanced Java Config

@EnableWebMvc 导入了 DelegatingWebMvcConfiguration，这个类：

提供默认 spring 配置 for mvc app

发现 和委托 WebMvcConfigurer 来 定制 配置。



对于 高 级 模 式，你可以删除 @EnableWebMvc，直接继承 DelegatingWebMvcConfiguration，而不是 实现 WebMvcConfigurer:

```java
@Configuration
public class WebConfig extends DelegatingWebMvcConfiguration {

    // ...
}
```

你可以保留 WebConfig中 已存在的方法，但你现在也可以 覆盖 基类的 bean 声明，并且你依然可以拥有 任意数量的 其他 WebMvcConfigurer 实现 在classpath中。



---

1.11.14 Advcanced XmL Config

mvc 命名空间没有 高级模式，如果你要定制 你不能修改的bean的 属性，你可以使用 BeanPostProcessor  lifecycle hook。。

```java
@Component
public class MyPostProcessor implements BeanPostProcessor {

    public Object postProcessBeforeInitialization(Object bean, String name) throws BeansException {
        // ...
    }
}
```

注意 你需要 声明 MyPostProcessor 为一个 bean，在xml中 显式声明，或 通过 `<component-scan>` 自动扫描到。



---

1.12 HTTP/2

servlet 4 容器 需要支持 http/2, spring5 和 servlet 4 兼容。从编程模型考虑，app 不需要做任何 特定的事情。但，有一些与 server 配置 相关的 注意事项。 更多信息，看 http/2 wiki。。。。

servlet api 公开了 一个与 http/2 相关的 结构，你可以使用 javax.servlet.http.PushBuilder 来主动 向客户端 推送资源，它支持作为 @RequestMapping 方法的 方法参数。







---

2 Rest Clients

描述 客户端 访问 rest 端点的 选项。



---

2.1 RestTemplate

sync 客户端 用来 执行 http request。它是原始的spring rest 客户端，并在底层 http 客户端库 上公开了 一个 简单的 模板方法 API。

从5.0开始， RestTemplate 处于 维护模式，只有较小的 更改请求 和 错误修复 被接受。请，考虑使用 提供了更先到的API ，并支持 同步，异步 ，stream 传输方案的  WebClient。



---

2.2 WebClient

非阻塞，响应式 client  来 执行 http请求， 在5.0引入，提供 RestTemplate 的 现代替代方案，有效支持 同步，异步，流。

和RestTemplate 相比，WebClient支持：

1. 非阻塞io
2. 响应式 stream back pressure
3. 更少硬件资源下的 高并发
4. 使用java 8 lambda 的函数式风格，
5. 同步和异步 交互
6. stream 上传到/下载from 服务器。

更多信息：https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-client

。。。这个还有个 web-reactive 项目。。。我这个是 web 。。然后 WebClient 的介绍 在 web-reactive 中。。





---

3 测试

列举了 spring-test 中 可用于 mvc app 的 选项：

1. Servlet API Mocks，mock Servlet API for 测试 controller，filter，和其他web组件。
2. TestContext Framework，支持 加载 spring 配置。 MockServletContext
3. Spring MVC Test，框架，被称为 MockMvc，测试 注解的controller 通过 DispatcherServlet。完成spring mvc 基础架构，而不需要 http server。
4. Client-side REST，MockRestServiceServer，你可以用做 mock server 来 测试 客户端测的使用RestpTemplate的 代码。
5. WebTestClient，构建测试用的 WebFlux 应用，也能用于 端到端 的集成测试，非阻塞，响应式 客户端 适合用来测试 异步和 streaming 的场景。





---

4 WebSockets

这部分 覆盖 对以下的支持： Servlet stack，包含原始WebSocket交互的WebSocket 消息传递，通过SockJS 的WebSocket 模拟，和 通过STOMP作为 WebSocket上的 子协议的 发布-订阅消息传递。



---

4.1 介绍 WebSocket

websocket 协议，RFC 6455, 提供了 标准的方式 来 通过单个TCP连接 建立 客户端和服务器 间 全双工双向通信通道。它是 tcp协议，不同于http协议，但旨在通过 http工作，使用80,443端口，并运行重用现有防火墙规则。

websocket 交互以 http 请求开始，该请求 使用 http Upgrade header 来进行升级，在这种情况下，切换到 websocket 协议，下面显示了这样的交互：

```yaml
GET /spring-websocket-portfolio/portfolio HTTP/1.1
Host: localhost:8080
Upgrade: websocket     // Upgrade 头
Connection: Upgrade     // 使用 Upgrade 连接
Sec-WebSocket-Key: Uc9l9TMkWGbHFD2qnFHltg==
Sec-WebSocket-Protocol: v10.stomp, v11.stomp
Sec-WebSocket-Version: 13
Origin: http://localhost:8080
```



支持 websocket 的 服务器 不返回200,而是 返回下面的：

```yaml
HTTP/1.1 101 Switching Protocols      // 协议切换
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: 1qVdfYHU9hPOl4JYYNXF623Gzn0=
Sec-WebSocket-Protocol: v10.stomp
```



成功握手后，http upgrade 请求的 底层的 TCP socket 保持打开状态，供 client 和server 继续 接受，发送 消息。

完整介绍 看 RFC 6455, HTML5的 WebSocket 章节，等

注意，如果 WebSocket 服务器在 Web服务器 ( 如nginx ) 后面运行，你可能需要将其配置为 将websocket upgrade请求 传递到 websocket 服务器。同样，如果app 在 云中运行，请查看 提供商对于 WEbsocket 的支持。



---

4.1.1 Http vs WebSocket

尽管websocket 被设计为 与http 兼容 并以 http 请求开始，但重要的是 要了解 这2种协议 会导致 不同的 架构 和 app 编程模型。

在 http 和rest 中，app 被建模为 多个 url。 为了与app交互，客户端 以 请求-响应 的方式 访问这些 url。server 根据 http url，method，header 将 请求 route 到合适的 handler。

相比之下，在websocket中，初始连接 通常只有一个 url，随后 所有app 消息都在同一个 tcp 上流动。这指向了一个 完全不同的 异步，事件驱动的  消息传递架构。

websocket 也是一种 低级传输协议，和http不同，它不为 消息的内容规定任何语义。这个意味着 除非客户端 和服务器在 消息语义上达成一致，否则无法路由或处理消息。

websocket 客户端 和服务器 可以通过 http 握手请求 的 Sec-WebSocket-Protocol 头 协商使用 更高级的消息传输协议( 如，STOMP) ，如果没有，它们需要提出自己的约定。





---

4.1.2 When to Use WebSockets

websocket 可以使得页面动态和交互。但是，在许多情况下，Ajax 和 HTTP stream 或 长轮询的组合 可以提供 有效 简单 的解决方案。

例如，新闻，邮件 和社交订阅源 需要 动态更新，但每隔几分种 更新一次 可能完全没有问题。另一方面，多人协作，游戏，金融 的app 需要 更接近 实时。

延迟本身并不是决定性因素。如果消息量较少，http stream 或 轮询 可以提供有效的解决方案。低延迟，高频率和 高容量的 场景，才是 websocket 的 最佳case。

记住，在internet上 ，不受你控制的 限制性代理 可能会阻止 WebSocket 交互，因为它们 没有配置为 传递Upgrade header，或者因为它们关闭了 看起来 空闲的长期连接。 这意味着将 websocket 应用于 防火墙 的内部app 是 一个 比 面向公众的app 更直接的 决定。



---

4.2 WebSocket

spring 提供了 WebSocket api，你可以用来 编写 处理websocket 消息的 客户端/服务器 的 app。



---

4.2.1 WebSocketHandler

创建一个 WebSocket server  非常简单，就是 实现 WebSocketHandler，或者 扩展 TextWebSocketHandler 或 BinaryWebSocketHandler。 下面是 使用了 TextWebSocketHandler

```java
import org.springframework.web.socket.WebSocketHandler;
import org.springframework.web.socket.WebSocketSession;
import org.springframework.web.socket.TextMessage;

public class MyHandler extends TextWebSocketHandler {

    @Override
    public void handleTextMessage(WebSocketSession session, TextMessage message) {
        // ...
    }

}
```

有专门用于 websocket java 配置 的 xml 命名空间 的支持，来将 前面的 websocket handler 映射到 特定url

```java
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;

@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(myHandler(), "/myHandler");
    }

    @Bean
    public WebSocketHandler myHandler() {
        return new MyHandler();
    }

}
```



```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        https://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:handlers>
        <websocket:mapping path="/myHandler" handler="myHandler"/>
    </websocket:handlers>

    <bean id="myHandler" class="org.springframework.samples.MyHandler"/>

</beans>
```

上面的例子 用在 mvc app中， 应该被包含在 DispatcherServlet 的配置中。 但是 spring 的 websocket支持并不依赖于 spring mvc。借助 WebSocketHttpRequestHandler 将 WebSocketHandler 集成到 其他HTTP 服务环境中 相对简单。

当 直接 或间接使用 WebSocketHandler API 时， 例如通过 STOMP 消息传递，app必须同步消息的发送，因为底层的 标准websocket session (JSR-356) 不允许 并发发送。 一种选择是使用 ConcurrentWebSocketSessionDecorator 包装的 WebSocketSession。



---

4.2.2 WebSocket Handshake

最简单的 自定义 初始http websocket 握手请求 的方法 是 通过 HandshakeInterceptor ， 这个 公开了 握手 之前 和 之后 的方法。你可以使用这样的 拦截器 来阻止 握手 或任何属性 对 WebSocketSession 可用。下面的例子使用 内置拦截器 来将 http session 属性  传递给 WebSocket session：

```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(new MyHandler(), "/myHandler")
            .addInterceptors(new HttpSessionHandshakeInterceptor());
    }

}
```

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        https://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:handlers>
        <websocket:mapping path="/myHandler" handler="myHandler"/>
        <websocket:handshake-interceptors>
            <bean class="org.springframework.web.socket.server.support.HttpSessionHandshakeInterceptor"/>
        </websocket:handshake-interceptors>
    </websocket:handlers>

    <bean id="myHandler" class="org.springframework.samples.MyHandler"/>

</beans>
```

一个更高级的选项是 扩展 DefaultHandshakeHandler ，它执行 websocket 握手的 步骤，包括 验证客户端来源，协商子协议，和其他细节。 如果app 需要配置 自定义的 RequestUpgradeStrategy 以适应 尚未支持 WEbsocket服务器 和 版本，则ap 可能还需要使用 此选项。 java配置和 xml 命名空间 使得配置 自定义 HandshakeHandler 成为可能。

spring提供了 WebSocketHandlerDecorator 基类，你可以用来 装饰具有附加行为 的 WebSocketHandler。 使用 websocket java配置 或 xml 命名空间 时，默认提供 日志记录 和 异常处理实现。 ExceptionWebSocketHandlerDecorator 捕获所有 从 任何 WebSocketHandler 方法中 抛出的异常，并以状态 1011( 这表示 服务器错误) 关闭 websocket session



---

4.2.3 Deployment

spring websocket api 易于 集成到 mvc 应用，此时，DisServ 服务 http websocket handshake，和 其他http 请求。通过调用 WebSocketHttpRequestHandler 也很容器 集成到 其他 http 处理场景中。这方便且容易理解。但要特别注意 JSR-356 的运行时。

java websocket api (jsr-356) 提供了2 种部署机制。 第一种 涉及 启动时的 servlet 容器 classpath 扫描 ( servlet 3 的功能)。另一个是在 servlet 容器 初始化时 使用的 注册 api。这些机制 都不能 使用 单个 "前端控制器" 来 处理 所有 http -- 包括 websocket 握手 和 所有 其他http 请求 -- 例如，spring mvc 的 DisServ。

这是 jsr-356 的一个重要限制，即 spring 的 websocket 支持使用 特定于服务器的 RequestUpgradeStrategy 实现来解决，即使在JSR-356 运行时 中运行也是如此。 目前此类策略适用于 tomcat,jetty,glassfish,weblogic,websphere,undertow,wildfly.

已经解决 java websocket api 中上述限制的 请求，可以在 eclipse-ee4j/websocket-api#211 中 进行后续处理tomcat，undertow，websphere 提供了 它们自己的 api 替代方案，可以做到这一点，jetty 也可以。

第二个考虑因素是， 支持 jsr-356 的 servlet 容器 预计会执行 ServletContainerInitializer (SCI ) 扫描，这可能会减慢app的 启动速度 -- 有些情况下，会显著降低。如果在升级到 支持 jsr-356 的servlet容器 版本后 观察到 显著影响，应该可以通过 在 web.xml 中使用 `<absolute-ordering />` 元素 选择性地 启用或禁用 web 片段 ( 和 SCI 扫描)：

```xml
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://java.sun.com/xml/ns/javaee
        https://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0">

    <absolute-ordering/>

</web-app>
```

然后，你可以按名字 选择性地启用 web 片段，如spring 自己的 SpringServletContainerInitializer，它提供了 对 Servlet 3 java 初始化 API 的支持 ：

```xml
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://java.sun.com/xml/ns/javaee
        https://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0">

    <absolute-ordering>
        <name>spring_web</name>
    </absolute-ordering>

</web-app>
```



---

4.2.4 Server Configuration

每个底层 websocket 引擎 都公开了 控制 运行时 特性的 配置属性，如 消息缓冲区大小，空闲超时 等。

对于 tomcat，wildfly,glassfish， 你可以增加一个 ServletServerContainerFactoryBean 到你的 websocket java 配置：

```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Bean
    public ServletServerContainerFactoryBean createWebSocketContainer() {
        ServletServerContainerFactoryBean container = new ServletServerContainerFactoryBean();
        container.setMaxTextMessageBufferSize(8192);
        container.setMaxBinaryMessageBufferSize(8192);
        return container;
    }

}
```

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        https://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <bean class="org.springframework...ServletServerContainerFactoryBean">
        <property name="maxTextMessageBufferSize" value="8192"/>
        <property name="maxBinaryMessageBufferSize" value="8192"/>
    </bean>

</beans>
```



对于客户端的 websocket 配置，你应该使用  WebSocketContainerFactoryBean (XML) 或 ContainerProvider.getWebSocketContainer() （java配置）



对于jetty，你需要 提供一个 预配置的 jetty  WebSocketServerFactory ，添加到 spring 的 DefaultHandshakeHandler 通过 你的 websocket java 配置：

```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(echoWebSocketHandler(),
            "/echo").setHandshakeHandler(handshakeHandler());
    }

    @Bean
    public DefaultHandshakeHandler handshakeHandler() {

        WebSocketPolicy policy = new WebSocketPolicy(WebSocketBehavior.SERVER);
        policy.setInputBufferSize(8192);
        policy.setIdleTimeout(600000);

        return new DefaultHandshakeHandler(
                new JettyRequestUpgradeStrategy(new WebSocketServerFactory(policy)));
    }

}
```

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        https://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:handlers>
        <websocket:mapping path="/echo" handler="echoHandler"/>
        <websocket:handshake-handler ref="handshakeHandler"/>
    </websocket:handlers>

    <bean id="handshakeHandler" class="org.springframework...DefaultHandshakeHandler">
        <constructor-arg ref="upgradeStrategy"/>
    </bean>

    <bean id="upgradeStrategy" class="org.springframework...JettyRequestUpgradeStrategy">
        <constructor-arg ref="serverFactory"/>
    </bean>

    <bean id="serverFactory" class="org.eclipse.jetty...WebSocketServerFactory">
        <constructor-arg>
            <bean class="org.eclipse.jetty...WebSocketPolicy">
                <constructor-arg value="SERVER"/>
                <property name="inputBufferSize" value="8092"/>
                <property name="idleTimeout" value="600000"/>
            </bean>
        </constructor-arg>
    </bean>

</beans>
```



---

4.2.5 Allowed Origins

spring 4.1.5 开始，websocket 和 sockjs 的默认行为 是 只接受 同源请求。也可以允许 所有 或 指定列表的 源。这个检查 主要是为了 浏览器客户端设计的。没有东西 可以阻止 其他类型的 客户端修改 Origin 头的值。 (更多信息，看 RFC 6454 : The Web Origin Concept)



3种可能的行为：

1. 只允许 同源请求 (默认)：这种模式下，当 sockjs 被启用，Iframe http 响应头 X-Frame-Options 设置为 SAMEORIGIN，并禁止 jsonp 传输，因为它不允许检查请求。因此，启用此模式时 不支持 IE6 和 IE7。
2. 允许指定列表的源：每个允许的源 必须以 http:// 或 https:// 开头。这种模式下，启用sockjs时， 会禁用 IFrame 传输，因此，启用这个模式时，不支持 IE6 到 IE9.
3. 允许所有源：要启用这个模式，你应该提供 * 作为 允许的 源。这种模式下， 所有传输都可用。



配置 websocket 和 sockjs 允许的 源：

```java
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;

@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(myHandler(), "/myHandler").setAllowedOrigins("https://mydomain.com");
    }

    @Bean
    public WebSocketHandler myHandler() {
        return new MyHandler();
    }

}
```

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        https://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:handlers allowed-origins="https://mydomain.com">
        <websocket:mapping path="/myHandler" handler="myHandler" />
    </websocket:handlers>

    <bean id="myHandler" class="org.springframework.samples.MyHandler"/>

</beans>
```



---

4.3 SockJS Fallback

在公共internet 上，你控制之外的 代理可能会组织 websocket 交互，因为 它们未配置为 传递Upgrade 头，或者 它们关闭 看似 空闲的长期连接。

解决方案是 websocket 仿真 -- 也就是，先尝试 使用websocket，然后 使用基于http的 技术 来模拟 websocket 交互 并公开 相同的 app级别 api。

在 servlet stack 上，spring 框架 为 SockJS 协议 提供了 服务器(和 客户端) 的支持。



---

4.3.1 Overview

sockjs 的目标是 让app 使用 websocket api，但运行时 ，在需要时 会回退到 非 websocket ， 而无需修改 app 代码。















。。。 从这里开始 到 本文件 结束 全是 随便抄点。。。



sockjs 被设计 用在 浏览器中。支持 非常广泛的 浏览器版本。

传输最终 是 3种： websocket，http streaming，http long polling



sockjs 客户端 首先 发送  GET /info  从服务器或的 基本信息。 然后 它决定使用哪种传输方式。如果可能，则使用 websocket，如果不行，在大多数 浏览器中，至少可以 http streaming。 如果不行，那么 http (long) polling。



传输请求的 结构：

```
https://host:port/myApp/myEndpoint/{server-id}/{session-id}/{transport}
```

server-id 用于 在 集群中 route 请求， 否则可以不用。

session-id 是 http 请求 属于 的 sockjs session

transport 表示 传输类型 (如， websocket，xhr-streaming 等)



需要一个 http请求 来 websocket 握手。然后 所有消息 在这个 socket 上 传输和交换。



启用 sockjs

```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(myHandler(), "/myHandler").withSockJS();
    }

    @Bean
    public WebSocketHandler myHandler() {
        return new MyHandler();
    }

}
```

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        https://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:handlers>
        <websocket:mapping path="/myHandler" handler="myHandler"/>
        <websocket:sockjs/>
    </websocket:handlers>

    <bean id="myHandler" class="org.springframework.samples.MyHandler"/>

</beans>
```



[`SockJsHttpRequestHandler`](https://docs.spring.io/spring-framework/docs/5.3.16/javadoc-api/org/springframework/web/socket/sockjs/support/SockJsHttpRequestHandler.html)



IE8 和 IE9

通过使用 微软的 XDomainRequest 来支持 IE8,9 的 Ajax/XHR streaming



spring 的 sockjs 有一个 sessionCookieNeeded 的属性， 默认启用，由于大多数 javaapp 都以来于 JSESSIONID cookie。



```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/portfolio").withSockJS()
                .setClientLibraryUrl("http://localhost:8080/myapp/js/sockjs-client.js");
    }

    // ...

}
```

```
<websocket:sockjs>
```



---

4.3.4 Heartbeats

spring sockJS 配置 有一个 属性 heartbeatTime 。默认是 25秒。



4.3.5 CLient Disconnects



---

4.3.6 SockJS and CORS



禁用CORS 头通过 spring 的 SockJsService 的 suppressCors 属性。



SockJS 期望下面的 头和 值

- `Access-Control-Allow-Origin`: Initialized from the value of the `Origin` request header.
- `Access-Control-Allow-Credentials`: Always set to `true`.
- `Access-Control-Request-Headers`: Initialized from values from the equivalent request header.
- `Access-Control-Allow-Methods`: The HTTP methods a transport supports (see `TransportType` enum).
- `Access-Control-Max-Age`: Set to 31536000 (1 year).

确切的实现，查看 TransportType 枚举 和 AbstractSockJsService 的 addCorsHeaders 。



---

4.3.7 SockJsClient

sockjs java 客户端支持 websocket,xhr-streaming,xhr-polling 传输。

你可以配置 WebSocketTransport with：

- `StandardWebSocketClient` in a JSR-356 runtime.
- `JettyWebSocketClient` by using the Jetty 9+ native WebSocket API.
- Any implementation of Spring’s `WebSocketClient`.



XhrTransport，支持 xhr-streaming, xhr-polling， 有2个实现

- `RestTemplateXhrTransport` uses Spring’s `RestTemplate` for HTTP requests.
- `JettyXhrTransport` uses Jetty’s `HttpClient` for HTTP requests.



```java
List<Transport> transports = new ArrayList<>(2);
transports.add(new WebSocketTransport(new StandardWebSocketClient()));
transports.add(new RestTemplateXhrTransport());

SockJsClient sockJsClient = new SockJsClient(transports);
sockJsClient.doHandshake(new MyWebSocketHandler(), "ws://example.com:8080/sockjs");
```



```java
HttpClient jettyHttpClient = new HttpClient();
jettyHttpClient.setMaxConnectionsPerDestination(1000);
jettyHttpClient.setExecutor(new QueuedThreadPool(1000));
```



```java
@Configuration
public class WebSocketConfig extends WebSocketMessageBrokerConfigurationSupport {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/sockjs").withSockJS()
            .setStreamBytesLimit(512 * 1024) 
            .setHttpMessageCacheSize(1000) 
            .setDisconnectDelay(30 * 1000); 
    }

    // ...
}
```





---

4.4 STOMP

Simple Text Oriented Messaging Protocol

用于 脚本语言 ( 如ruby，python，perl) 来连接 企业级消息代理。



优势：

1. 不需要 发明 自定义消息传输协议和消息格式
2. 有可用的 STOMP 客户端，包括 spring 框架的 java 客户端
3. 你可以 (可选的) 使用 消息代理 ( 如 rabbitmQ，activeMQ 等) 来管理 订阅 和 广播消息
4. app逻辑可以组织在 任意数量的 @Controller 实例中，并且可以根据 stomp destination header 将消息路由到它们，而不是 使用给定连接的 单个 WebSocketHandler 处理元是的 WebSocket 消息。
5. 可以使用 spring security 来保护消息 基于 stomp目的和 消息类型。



---

4.4.3 Enable STOMP

```java
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;

@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/portfolio").withSockJS();  
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.setApplicationDestinationPrefixes("/app"); 
        config.enableSimpleBroker("/topic", "/queue"); 
    }
}
```

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        https://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:message-broker application-destination-prefix="/app">
        <websocket:stomp-endpoint path="/portfolio">
            <websocket:sockjs/>
        </websocket:stomp-endpoint>
        <websocket:simple-broker prefix="/topic, /queue"/>
    </websocket:message-broker>

</beans>
```



```javascript
var socket = new SockJS("/spring-websocket-portfolio/portfolio");
var stompClient = webstomp.over(socket);

stompClient.connect({}, function(frame) {
}
```

```javascript
var socket = new WebSocket("/spring-websocket-portfolio/portfolio");
var stompClient = Stomp.over(socket);

stompClient.connect({}, function(frame) {
}
```





---

4.4.4 WebSocket Server

通过 StompEndpointRegistry 设置 HandshakeHandler 和 WebSocketPolicy

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/portfolio").setHandshakeHandler(handshakeHandler());
    }

    @Bean
    public DefaultHandshakeHandler handshakeHandler() {

        WebSocketPolicy policy = new WebSocketPolicy(WebSocketBehavior.SERVER);
        policy.setInputBufferSize(8192);
        policy.setIdleTimeout(600000);

        return new DefaultHandshakeHandler(
                new JettyRequestUpgradeStrategy(new WebSocketServerFactory(policy)));
    }
}
```



---

4.4.5 Flow of Messages



- [Message](https://docs.spring.io/spring-framework/docs/5.3.16/javadoc-api/org/springframework/messaging/Message.html): Simple representation for a message, including headers and payload.
- [MessageHandler](https://docs.spring.io/spring-framework/docs/5.3.16/javadoc-api/org/springframework/messaging/MessageHandler.html): Contract for handling a message.
- [MessageChannel](https://docs.spring.io/spring-framework/docs/5.3.16/javadoc-api/org/springframework/messaging/MessageChannel.html): Contract for sending a message that enables loose coupling between producers and consumers.
- [SubscribableChannel](https://docs.spring.io/spring-framework/docs/5.3.16/javadoc-api/org/springframework/messaging/SubscribableChannel.html): `MessageChannel` with `MessageHandler` subscribers.
- [ExecutorSubscribableChannel](https://docs.spring.io/spring-framework/docs/5.3.16/javadoc-api/org/springframework/messaging/support/ExecutorSubscribableChannel.html): `SubscribableChannel` that uses an `Executor` for delivering messages.



![message flow simple broker](https://docs.spring.io/spring-framework/docs/current/reference/html/images/message-flow-simple-broker.png)





![message flow broker relay](https://docs.spring.io/spring-framework/docs/current/reference/html/images/message-flow-broker-relay.png)







```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/portfolio");
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.setApplicationDestinationPrefixes("/app");
        registry.enableSimpleBroker("/topic");
    }
}

@Controller
public class GreetingController {

    @MessageMapping("/greeting")
    public String handle(String greeting) {
        return "[" + getTimestamp() + ": " + greeting;
    }
}
```





---

4.4.6 Annotated Controllers

- [`@MessageMapping`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket-stomp-message-mapping)
- [`@SubscribeMapping`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket-stomp-subscribe-mapping)
- [`@MessageExceptionHandler`](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket-stomp-exception-handler)





| Method argument                                              | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `Message`                                                    | For access to the complete message.                          |
| `MessageHeaders`                                             | For access to the headers within the `Message`.              |
| `MessageHeaderAccessor`, `SimpMessageHeaderAccessor`, and `StompHeaderAccessor` | For access to the headers through typed accessor methods.    |
| `@Payload`                                                   | For access to the payload of the message, converted (for example, from JSON) by a configured `MessageConverter`. The presence of this annotation is not required since it is, by default, assumed if no other argument is matched. You can annotate payload arguments with `@javax.validation.Valid` or Spring’s `@Validated`, to have the payload arguments be automatically validated. |
| `@Header`                                                    | For access to a specific header value — along with type conversion using an `org.springframework.core.convert.converter.Converter`, if necessary. |
| `@Headers`                                                   | For access to all headers in the message. This argument must be assignable to `java.util.Map`. |
| `@DestinationVariable`                                       | For access to template variables extracted from the message destination. Values are converted to the declared method argument type as necessary. |
| `java.security.Principal`                                    | Reflects the user logged in at the time of the WebSocket HTTP handshake. |



@SendTo,@SendToUser



Messages can be handled asynchronously and a `@MessageMapping` method can return `ListenableFuture`, `CompletableFuture`, or `CompletionStage`.



`@SubscribeMapping` is similar to `@MessageMapping` but narrows the mapping to subscription messages only. 

```java
@Autowired
private TaskScheduler messageBrokerTaskScheduler;

// During initialization..
stompClient.setTaskScheduler(this.messageBrokerTaskScheduler);

// When subscribing..
StompHeaders headers = new StompHeaders();
headers.setDestination("/topic/...");
headers.setReceipt("r1");
FrameHandler handler = ...;
stompSession.subscribe(headers, handler).addReceiptTask(() -> {
    // Subscription ready...
});
```





```java
@Controller
public class MyController {

    // ...

    @MessageExceptionHandler
    public ApplicationError handleException(MyException exception) {
        // ...
        return appError;
    }
}
```



---

4.4.7 Sending Messages

```java
@Controller
public class GreetingController {

    private SimpMessagingTemplate template;

    @Autowired
    public GreetingController(SimpMessagingTemplate template) {
        this.template = template;
    }

    @RequestMapping(path="/greetings", method=POST)
    public void greet(String greeting) {
        String text = "[" + getTimestamp() + "]:" + greeting;
        this.template.convertAndSend("/topic/greetings", text);
    }

}
```



---

4.4.8 Simple Broker



```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    private TaskScheduler messageBrokerTaskScheduler;

    @Autowired
    public void setMessageBrokerTaskScheduler(@Lazy TaskScheduler taskScheduler) {
        this.messageBrokerTaskScheduler = taskScheduler;
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableSimpleBroker("/queue/", "/topic/")
                .setHeartbeatValue(new long[] {10000, 20000})
                .setTaskScheduler(this.messageBrokerTaskScheduler);

        // ...
    }
}
```



---

4.4.9 External Broker

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/portfolio").withSockJS();
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableStompBrokerRelay("/topic", "/queue");
        registry.setApplicationDestinationPrefixes("/app");
    }

}
```



```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        https://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:message-broker application-destination-prefix="/app">
        <websocket:stomp-endpoint path="/portfolio" />
            <websocket:sockjs/>
        </websocket:stomp-endpoint>
        <websocket:stomp-broker-relay prefix="/topic,/queue" />
    </websocket:message-broker>

</beans>
```



---

4.4.10 Connecting to a Broker

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig extends AbstractWebSocketMessageBrokerConfigurer {

    // ...

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableStompBrokerRelay("/queue/", "/topic/").setTcpClient(createTcpClient());
        registry.setApplicationDestinationPrefixes("/app");
    }

    private ReactorNettyTcpClient<byte[]> createTcpClient() {
        return new ReactorNettyTcpClient<>(
                client -> client.addressSupplier(() -> ... ),
                new StompReactorNettyCodec());
    }
}
```



---

4.4.11 Dots as Separators

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    // ...

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.setPathMatcher(new AntPathMatcher("."));
        registry.enableStompBrokerRelay("/queue", "/topic");
        registry.setApplicationDestinationPrefixes("/app");
    }
}
```



```xml
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:websocket="http://www.springframework.org/schema/websocket"
        xsi:schemaLocation="
                http://www.springframework.org/schema/beans
                https://www.springframework.org/schema/beans/spring-beans.xsd
                http://www.springframework.org/schema/websocket
                https://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:message-broker application-destination-prefix="/app" path-matcher="pathMatcher">
        <websocket:stomp-endpoint path="/stomp"/>
        <websocket:stomp-broker-relay prefix="/topic,/queue" />
    </websocket:message-broker>

    <bean id="pathMatcher" class="org.springframework.util.AntPathMatcher">
        <constructor-arg index="0" value="."/>
    </bean>

</beans>
```





```java
@Controller
@MessageMapping("red")
public class RedController {

    @MessageMapping("blue.{green}")
    public void handleGreen(@DestinationVariable String green) {
        // ...
    }
}
```

The client can now send a message to `/app/red.blue.green123`.



---

4.4.12 Authentication



4.4.13 Token Authentication

```java
@Configuration
@EnableWebSocketMessageBroker
public class MyConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureClientInboundChannel(ChannelRegistration registration) {
        registration.interceptors(new ChannelInterceptor() {
            @Override
            public Message<?> preSend(Message<?> message, MessageChannel channel) {
                StompHeaderAccessor accessor =
                        MessageHeaderAccessor.getAccessor(message, StompHeaderAccessor.class);
                if (StompCommand.CONNECT.equals(accessor.getCommand())) {
                    Authentication user = ... ; // access authentication header(s)
                    accessor.setUser(user);
                }
                return message;
            }
        });
    }
}
```



---

4.4.14 Authorization



---

4.4.15 User Destinations

```java
@Controller
public class PortfolioController {

    @MessageMapping("/trade")
    @SendToUser("/queue/position-updates")
    public TradeResult executeTrade(Trade trade, Principal principal) {
        // ...
        return tradeResult;
    }
}
```



```java
@Controller
public class MyController {

    @MessageMapping("/action")
    public void handleAction() throws Exception{
        // raise MyBusinessException here
    }

    @MessageExceptionHandler
    @SendToUser(destinations="/queue/errors", broadcast=false)
    public ApplicationError handleException(MyBusinessException exception) {
        // ...
        return appError;
    }
}
```



```java
@Service
public class TradeServiceImpl implements TradeService {

    private final SimpMessagingTemplate messagingTemplate;

    @Autowired
    public TradeServiceImpl(SimpMessagingTemplate messagingTemplate) {
        this.messagingTemplate = messagingTemplate;
    }

    // ...

    public void afterTradeExecuted(Trade trade) {
        this.messagingTemplate.convertAndSendToUser(
                trade.getUserName(), "/queue/position-updates", trade.getResult());
    }
}
```





---

4.4.17 Order of Messages

```java
@Configuration
@EnableWebSocketMessageBroker
public class MyConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    protected void configureMessageBroker(MessageBrokerRegistry registry) {
        // ...
        registry.setPreservePublishOrder(true);
    }

}
```



```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        https://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:message-broker preserve-publish-order="true">
        <!-- ... -->
    </websocket:message-broker>

</beans>
```





---

4.4.17 Events

```
BrokerAvailabilityEvent
```

```
SessionConnectEvent
```

```
SessionConnectedEvent
```

```
SessionSubscribeEvent
```

```
SessionUnsubscribeEvent
```

```
SessionDisconnectEvent
```



---

4.4.18 Interception

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureClientInboundChannel(ChannelRegistration registration) {
        registration.interceptors(new MyChannelInterceptor());
    }
}
```

```java
public class MyChannelInterceptor implements ChannelInterceptor {

    @Override
    public Message<?> preSend(Message<?> message, MessageChannel channel) {
        StompHeaderAccessor accessor = StompHeaderAccessor.wrap(message);
        StompCommand command = accessor.getStompCommand();
        // ...
        return message;
    }
}
```



---

4.4.19 STOMP Client

```java
WebSocketClient webSocketClient = new StandardWebSocketClient();
WebSocketStompClient stompClient = new WebSocketStompClient(webSocketClient);
stompClient.setMessageConverter(new StringMessageConverter());
stompClient.setTaskScheduler(taskScheduler); // for heartbeats
```



```java
String url = "ws://127.0.0.1:8080/endpoint";
StompSessionHandler sessionHandler = new MyStompSessionHandler();
stompClient.connect(url, sessionHandler);
```



```java
public class MyStompSessionHandler extends StompSessionHandlerAdapter {

    @Override
    public void afterConnected(StompSession session, StompHeaders connectedHeaders) {
        // ...
    }
}
```



```java
session.send("/topic/something", "payload");
```

```java
session.subscribe("/topic/something", new StompFrameHandler() {

    @Override
    public Type getPayloadType(StompHeaders headers) {
        return String.class;
    }

    @Override
    public void handleFrame(StompHeaders headers, Object payload) {
        // ...
    }

});
```



---

4.4.20 WebSocket Scope

```java
@Controller
public class MyController {

    @MessageMapping("/action")
    public void handle(SimpMessageHeaderAccessor headerAccessor) {
        Map<String, Object> attrs = headerAccessor.getSessionAttributes();
        // ...
    }
}
```

```java
@Component
@Scope(scopeName = "websocket", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class MyBean {

    @PostConstruct
    public void init() {
        // Invoked after dependencies injected
    }

    // ...

    @PreDestroy
    public void destroy() {
        // Invoked when the WebSocket session ends
    }
}

@Controller
public class MyController {

    private final MyBean myBean;

    @Autowired
    public MyController(MyBean myBean) {
        this.myBean = myBean;
    }

    @MessageMapping("/action")
    public void handle() {
        // this.myBean from the current WebSocket session
    }
}
```



---

4.4.21 Performance



没有 银弹。许多因素，包括 消息的大小 和 数量， app方法的执行 是否需要阻塞，以及外部因素 ( 网络速度 和 其他问题)。



在消息传递的app 中，消息通过 线程池 来 异步执行 传递。

一个起点是 配置 支持 clientInboundChannel 和 clientOutboundChannel 的线程池。 默认情况下， 2者都可以 配置为 处理器数量的2倍。

如果注解的方法 中 消息的处理 主要受CPU 限制，则clientInboundChannel 的线程数量 应该保持接近 处理器 数量。 如果 受到IO限制，并且 需要在数据库 或 其他外部系统上阻塞 或等待，则可能需要增加 线程池 大小。



ThreadPoolExecutor 具有 3个重要属性： core thread pool size, max thread pool size, capacity for queue.

常见的 混淆点是 配置 核心池 (例如10) 和 最大池大小(例如20) 会导致 线程池 具有 10-20 个线程。事实上，如果将容量 保留为 其默认值 Integer.MAX_VALUE， 则线程池 的大小 永远不会超过 核心池大小，因为 所有额外的任务都会 排队。

在clientOutboundChannel 端，它是关于 向 WebScoket 客户端发送消息的。如果客户端位于 快速网络上，则线程数 应该保持 接近可用处理器的数量。如果它们速度慢 或带宽低，它们会花费 更长的时间来消费信息 并给线程池带来负担。因此，增加线程池大小变得 很有必要。

虽然可以预测 clientInboundChannel 的 工作负载 -- 毕竟，它基于app的功能，如何配置 clientOutboundChannel 更难，因为它基于app 无法控制的因素。所以 2个 附加属性 与消息的 发送有关：sendTImeLimit 和 sendBufferSizeLimit。你可以使用这些方法来配置 允许发送多长时间 以及在往客户端 发送消息时 可以缓冲多少数据。

一般的想法是，在任何给定时间，只有一个线程 可以用于发送客户端。同时，所有其他消息都会被缓冲，你可以使用这些属性 来观察/决定 发送消息需要多长时间，以及 同时可以 缓冲多少数据。

可能的配置：

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureWebSocketTransport(WebSocketTransportRegistration registration) {
        registration.setSendTimeLimit(15 * 1000).setSendBufferSizeLimit(512 * 1024);
    }

    // ...

}
```

```xml
    <websocket:message-broker>
        <websocket:transport send-timeout="15000" send-buffer-size="524288" />
        <!-- ... -->
    </websocket:message-broker>
```



```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureWebSocketTransport(WebSocketTransportRegistration registration) {
        registration.setMessageSizeLimit(128 * 1024);
    }

    // ...

}
```

```xml
    <websocket:message-broker>
        <websocket:transport message-size="131072" />
        <!-- ... -->
    </websocket:message-broker>
```



---

4.4.22 Monitoring

```
@EnableWebSocketMessageBroker` or `<websocket:message-broker>
```

当前有多少客户端session

建立了多少的 session

建立了，但是60秒内没有收到消息的 session

session 在 发送超时，或 发送缓冲区限制 后关闭，可能发生在 慢速客户端上。

session 在 传输错误后 关闭，比如 在websocket 或 http 请求/响应 上 读/写 失败。

CONNECT，CONNECTED，DISCONNECT 的总数。表示在stomp 级别上 连接了多少客户端。

客户端websocket session 建立到 代理的 tpc 连接数

代表客户端 转发到 代理或从 代理接受的 connect,connected,disconnect 的总数

用于支持 clientInboundChannel 的 线程池 的统计信息。

clientOutboundChannel 的 线程池的 统计信息。

用于发送心跳的 sockjs 的scheduler 的线程池 的 统计信息。



---

4.4.23 Testing

2种方法测试，第一种是 编写 服务器端 测试来验证控制器的功能及 注解了 message-handling 的controller。第二个是 编写 涉及运行客户端和服务器的 完整 端到端测试。

2者并不冲突。



---

5 Other Web Frameworks



5.1 Common Configuration

```xml
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/applicationContext*.xml</param-value>
</context-param>
```

```java
WebApplicationContext ctx = WebApplicationContextUtils.getWebApplicationContext(servletContext);
```

```
WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE
```



---

5.2 JSF

JavaServer Faces (JSF) is the JCP’s standard component-based, event-driven web user interface framework.

define `SpringBeanFacesELResolver` in your JSF `faces-context.xml` file

```xml
<faces-config>
    <application>
        <el-resolver>org.springframework.web.jsf.el.SpringBeanFacesELResolver</el-resolver>
        ...
    </application>
</faces-config>
```

```java
ApplicationContext ctx = FacesContextUtils.getWebApplicationContext(FacesContext.getCurrentInstance());
```



---

5.3 Apache Struts 2.x



---

5.4 Apache Tapestry 5.x



5.5 Futrher Resources







Version 5.3.16
 Last updated 2022-02-17 07:33:24 UTC

