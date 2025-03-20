https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#resources

5.3.15

这里从第二章开始，包含剩余的全部。

---

#### 2. Resources

本章覆盖 spring 如何处理资源，你如何使用spring中的资源，包括：

	1. 介绍
	1. Resource 接口
	1. 内置 Resource 实现
	1. ResourceLoader 接口
	1. ResourcePatternResolver 接口
	1. ResourceLoaderAware 接口
	1. 资源 as 依赖
	1. app ctx 和 资源路径



---

2.1 介绍

java标准的 URL类 和 各种URL 前缀的标准处理程序 不足以满足所有 对低级资源的访问。   
例如，没有标准化的URL实现 可以用于访问需要从类路径或相对于ServletContext 获取资源   
虽然可以为专门的URL前缀注册新的处理程序( 类似现有的前缀处理程序，如http:)，但这通常相当复杂，并且URL接口 仍缺乏一些理想的功能，例如检查 指向的资源 是否存在的方法。

---

2.2 Resource 接口

org.springframeword.core.io.Resource 是一个更强大的接口，用于抽象对低级资源的访问。下面的Resource 接口的方法：

```java
public interface Resource extends InputStreamSource {
    boolean exists();
    boolean isReadable();
    boolean isOpen();
    boolean isFile();
    URL getURL() throws IOException;
    URI getURI() throws IOException;
    File getFile() throws IOException;
    ReadableByteChannel readableChannel() throws IOException;
    long contentLength() throws IOException;
    long lastModified() throws IOException;
    Resource createRelative(String relativePath) throws IOException;
    String getFilename();
    String getDescription();
}
```

它扩展了 InputStreamSource 接口，InputStreamSource 只有一个方法：

```java
InputStream getInputStream() throws IOException;
```

Resource中一些重要的方法是：

1. getInputStream(): 定位和打开资源，返回一个 InputStream 为了从资源中读取。预计每次调用都返回一个新的InputStream。调用者 有责任 关闭stream。
2. exists(): 返回boolean，指示此资源是否以实际以物理形式存在
3. isOpen(): 返回boolean，指示 此资源是否 表现为具有 一个打开的流的句柄。如果true，InputStream不能被多次读取，必须之读取一次，然后关闭 以防止资源泄漏。对所有常用资源实现 返回false，InputStreamResource 除外。
4. getDescription(): 返回此资源的描述，用于在使用资源时 进行错误输出。这通常是文件的全名，或资源的实际URL。

其他方法让你获取表示资源的实际URL 或 File (如果底层实现兼容并支持 该功能)

一些Resource 的实现 也 实现了WritableResource接口，来支持写入。

spring本身 广泛使用Resource 抽象，当需要资源时，作为参数类型 在许多方法签名中。   
一些spring api 中的其他方法(如 各种AppCtx 实现的 构造器) 采用一个字符串，该字符串以 朴素 或简单的形式 用于创建适合该上下文实现的资源，或者通过字符串路径上的特殊前缀，让调用者指定必须创建和使用的特定的资源实现。

你在自己的代码中使用 Resource 也非常方便，虽然会导致 和spring的 耦合。它可以作为URL的更强大的替代品。



---

2.3 内置 Resource 实现

。。。这个Resource  好像 和 spring 没有太大关系啊。。。

spring提供了几个内置Resource 实现：

1. UrlResource
2. ClassPathResoruce
3. FileSystemResource
4. PathResource
5. ServletContextResource
6. InputStreamResource
7. ByteArrayResource

完整列表，请查看 Resource 的注释 中的 All Known Implementing Classes



---

2.3.1 UrlResource

UrlResource 封装了java.net.URL ，能用于通过URL访问资源，比如文件，https 目标，FTP 目标 等。   
所有的URL 有 标准化的 String 表示，比如一个合适的标准化的前缀，用来指示URL 类型。file:，https:，ftp: 等。

UrlResource 可以显式通过 它的构造器来创建， 但通常在调用 那些 使用string作为路径参数 API方法时 隐式创建。对于后一种，java.beans.PropertyEditor 最终决定创建哪种类型的资源。如果路径字符串包含一个众所周知的前缀，它会为该前缀创建一个适当的专有资源，但是，如果它无法识别前缀，它会假定该字符串是 标准URL 字符串，并创建一个UrlResource。



---

2.3.2 ClassPathResource

这个类 代表了一个资源，应该从 classpath 中获取。它使用 线程上下文类加载器，给定的类加载器，或给定的类 来加载资源。

如果classpath 资源 驻留在文件系统中，则此 Resource实现 支持解析为 java.io.File，但不支持驻留在jar中 且 尚未扩展(通过servlet引擎或任何环境) 的 classpath资源到 文件系统。为了解决这个问题，各种Resource实现 始终支持解析为 java.net.URL。

能显式通过 构造器来创建，但通常是 在 调用 以string作为参数来表达资源 的 API时 隐式创建。对于后一种情况 java.beans.PropertyEditor 识别字符串上的特殊前缀 classapth:， 并在这种情况下创建ClassPathResource.



---

2.3.3 FileSystemResource

为了处理 java.io.File 的 Resource 实现。也支持处理 java.nio.file.Path，应用spring的标准 基于string的 path 转换，但执行所有的操作 通过 java.nio.file.File api。对于 纯java.nio.path.Path 的支持，请使用PathResource。FileSystemResource 支持作为文件和 URL 的解析。



---

2.3.4 PathResource

为了处理 java.nio.file.Path 的 Resource 实现，通过 Path API 执行所有的 操作和 转换。它支持作为File 和 URL的 解析，并且还实现了 WritableResource 接口。PathResource 实际上是 基于纯java.nio.path.Path 的 FileSystemResource 替代方案，具有不同的 createRelative 行为。



---

2.3.5 ServletContextResource

为 ServletContext 资源的 Resource 实现，它解释 关于web 应用 根目录中的 相对路径。

它始终支持 stream 访问 和 URL 访问，但仅在扩展 web 应用存档 且资源物理位于文件系统上时才允许java.io.File 访问。它是否被扩展并在文件系统上 或直接从jar 或其他地址(如数据库 ) 访问 实际上取决于 Servlet 容器。

---

2.3.6 InputStreamResource

为给定的InputStream 的 Resource实现。只有没有特定的Resource 实现适用时 才应该使用它。特别是，尽可能首选ByteArrayResource 或 任何基于文件的 Resource 实现。

---

2.3.7 ByteArrayResource

为给定的byte 数组 的 Resource实现。它创建一个 ByteArrayInputStream 对 给定的byte 数组。



---

2.4 ResourceLoader 接口

接口的 实现 能返回 Resource 实例。

```java
public interface ResourceLoader {
    Resource getResource(String location);
    ClassLoader getClassLoader();
}
```

所有的 app ctx 实现了 ResourceLoader 接口。因此，所有 app ctx 能用来 获得 Resource 实例。

当你调用 getResource() 在一个特定的app ctx，并且 指定的路径没有特定前缀时，你将返回 适合 该特定app ctx 的资源类型。例如，假设在 ClassPathXmlApplciationContext 实例上运行下面的代码：

```java
Resource template = ctx.getResource("some/resource/path/myTemplate.txt");
```

对于ClassPathXmlAppCtx，返回 ClassPathResource。对于FileSystemXmlApplicationContext，返回 FileSystemResource。对于WebApplicationContext，返回 ServletContextResource。它对每个context返回 适当的对象。

因此，你可以以 适合特定应用程序 上下文的方式 加载资源。

另一方面，你也可以通过指定特殊的classpath: 前缀来强制使用 ClassPathResource，而不管 app ctx 类型：

```java
Resource template = ctx.getResource("classpath:some/resource/path/myTemplate.txt");
```

类似，你可以强制 UrlResource 通过使用 标准 java.net.URL 前缀，下面是 file 和 https 前缀：

```java
Resource template = ctx.getResource("file:///some/resource/path/myTemplate.txt");
```

```java
Resource template = ctx.getResource("https://myhost.com/resource/path/myTemplate.txt");
```

下面总结了 string 到 Resource 的转换：

| Prefix     | Example                        | Explanation            |
| ---------- | ------------------------------ | ---------------------- |
| classpath: | classpath:com/myapp/config.xml | 从classpath加载        |
| file:      | file:///data/config.xml        | 从文件系统 作为URL加载 |
| https:     | https://myserver/logo.png      | 加载作为URL            |
| (none)     | /data/config.xml               | 依赖AppCtx             |



---

2.5 ResourcePatternResolver 接口

是ResourceLoader 的扩展，它定义了一个策略 来 解析一个 地址pattern (比如，Ant 风格的 path pattern) 到 Resource 对象：

```java
public interface ResourcePatternResolver extends ResourceLoader {
    String CLASSPATH_ALL_URL_PREFIX = "classpath*:";
    Resource[] getResources(String locationPattern) throws IOException;
}
```

这个接口还为classpath中所有匹配的资源定义了一个特殊的 classpath*: 前缀。注意，这种情况下，资源位置应该是 没有占位符的 路径，例如，classpath\*:/config/beans.xml。   
jar文件或 不同目录 in class path 可以包含多个具有相同路径和相同名称的文件。更多classpaht\*: 资源前缀的通配符信息 查看 其他章节。

可以检查传入的ResourceLoader( 例如，通过ResourceLoaderAware 提供) 是否也实现了此接口。

PathMatchingResourcePatternResolver 是一个独立的实现，可以在AppCtx 之外使用，也可以用于 ResourceArrayPropertyEditor 中 来填充 Resource[] bean属性。   
PathMatchingResourcePatternResolver 能够将指定的资源位置路径解析为 一个或多个匹配的Resource 对象。源路径可以是 和 目标Resource 1-1映射的 简单路径，也可以是包含特殊classpath\*: 前缀 和/或 ant-格式 的 正则表达式 (使用AntPathMatcher 来进行匹配)。后者都是有效的通配符。

标准AppCtx 中默认的 ResourceLoader 实际上是 实现ResourcePatternResolver 接口的 PathMatchingResourcePatternResolver 的一个实例。AppCtx 实例 也实现了 ResourcePatternResolver 接口，并委托给默认的 PathMatchingResourcePatternResolver。



---

2.6 ResourceLoaderAware 接口

是一个特殊回调接口，它标识 希望被提供 ResourceLoader ref的组件。下面的是接口定义：

```java
public interface ResourceLoaderAware {
    void setResourceLoader(ResourceLoader resourceLoader);
}
```

由于AppCtx 就是一个 ResourceLoader，所以 bean 也可以实现 ApplicationContextAware 接口，使用AppCtx 来加载资源。当然如果你只需要加载资源，那么最好使用专门的 ResourceLoader 接口，这样，你的代码 只和 加载资源的 spring 代码 耦合，而不是 整个AppCtx 耦合。

在应用程序组件中，你还可以依赖 ResourceLoader 的自动装配 来代替 ResourceLoaderAware 接口。    
传统的 构造器 和 by-type 自动装配模式 能分别为构造器 和 setter 提供ResourceLoader。   
要更大的灵活性(包括自动装配字段和多个参数方法的能力)，请考虑使用基于注解的自动装配功能。在这种情况下，只要相关属性，构造器，或方法带有@Autowired注解，ResourceLoader就会自动装配进去。 

要为 包含通配符 或 使用特殊的classpath\*: 资源前缀的资源路径加载一个或多个 Resource对象，请考虑将ResourcePatternResolver 的实例自动装配到 你的 app中，而不是 ResourceLoader。



---

2.7 Resources as Dependencies

如果bean 本身要通过某种动态过程来确定和提供资源路径，那么bean使用ResourceLoader 或 ResourcePatternResolver 接口来加载资源 可能是有意义的。   
如果资源是静态的，那么消除对ResourceLoader 或 ResourcePatternResolver 的使用是有意义的，让bean公开它需要的Resource 属性，并期望它们被注入其中。

注入这些属性变得微不足道的是，所有的 app ctx 都 注册并使用 同一个 PropertyEditor, 它可以将string 路径转为 Resource对象。例如，下面MyBean 类具有 Resource 类型的 template 属性：

```java
package example;

public class MyBean {

    private Resource template;

    public setTemplate(Resource template) {
        this.template = template;
    }

    // ...
}
```

xml中，template 属性能 直接用 string 来配置：

```xml
<bean id="myBean" class="example.MyBean">
    <property name="template" value="some/resource/path/myTemplate.txt"/>
</bean>
```

注意，资源路径 没有前缀，后续会由于 app ctx 被当作 ResourceLoader 来加载资源，所以这个资源 是通过 ClassPathResource, FileSystemResource,ServletContextResource 来加载，取决于 app ctx 的类型。

如果你需要强制 指定使用的 Resource 类型，你可以使用前缀。下面的2个例子展示了 如何强制ClassPathResource 和 UrlResource:

```xml
<property name="template" value="classpath:some/resource/path/myTemplate.txt">
```

```xml
<property name="template" value="file:///some/resource/path/myTemplate.txt"/>
```

如果 MyBean  类被重构成 注解驱动的配置，指向myTemplate.txt 的路径 可以被保存到 template.path 的key中 -- 例如，spring的 Environment 可用的 properties 文件。 路径可以通过 @Value 注解 + 占位符来 ref。

```java
@Component
public class MyBean {

    private final Resource template;

    public MyBean(@Value("${template.path}") Resource template) {
        this.template = template;
    }

    // ...
}
```

如果我们想支持 在类路径中的 多个位置的同一路径下发现的 多个模板( 例如，在类路径中的多个jar中)，我们可以使用特殊的 classpath\*: 前缀 和 通配符，将 templates.key 定义为 classpath\*:/config/templates/*.txt。下面的代码，spring 会转换template path pattern 为 Resource 数组，注入到 MyBean 构造器中。

```java
@Component
public class MyBean {

    private final Resource[] templates;

    public MyBean(@Value("${templates.path}") Resource[] templates) {
        this.templates = templates;
    }

    // ...
}
```



---

2.8 Application Contexts and Resource Paths

如何创建 带资源的 app ctx，包括 使用xml的快捷方式，如何使用通配符 及其他。

---

2.8.1 构造app ctx

app ctx 的 构造器 通常 接受一个 string 或 string数组 作为 资源的地址，比如构成context定义的 xml 文件。

如果位置路径没有前缀，那么根据app ctx 的类型 会生产 Resource 子类的 类型，并且加载 bean定义。

```java
ApplicationContext ctx = new ClassPathXmlApplicationContext("conf/appContext.xml");
```

bean 定义 从classpath 加载，因为使用了 ClassPathResource。   
。。。看源码，构造器里 直接使用 ClassPathResource的：：：：

```java
	public ClassPathXmlApplicationContext(String[] paths, Class<?> clazz, @Nullable ApplicationContext parent)
			throws BeansException {

		super(parent);
		Assert.notNull(paths, "Path array must not be null");
		Assert.notNull(clazz, "Class argument must not be null");
		this.configResources = new Resource[paths.length];
		for (int i = 0; i < paths.length; i++) {
			this.configResources[i] = new ClassPathResource(paths[i], clazz);
		}
		refresh();
	}
```



```java
ApplicationContext ctx = new FileSystemXmlApplicationContext("conf/appContext.xml");
```

这个从 文件系统 加载bean定义(本例中，是想对于当前工作目录)。   

使用classpath前缀 或 标准URL前缀，会覆盖 默认的 Resource 类型。

```java
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("classpath:conf/appContext.xml");
```

使用 FileSystemXmlAppCtx 来加载 bean 定义 从classpath 中。但是，它仍然是一个 FileSystemXmlApplicationContext。如果这个AppCtx 继续被当作 ResourceLoader，那么 没有前缀的路径 被认为是 文件系统 路径。



---

构造 ClassPathXmlApplicationContext 实例 -- 快捷方式

ClassPathXmlAppCtx 公开了很多构造器 以便方便地 实例化。基本思想是你可以 仅提供一个 包含xml文件名( 没有前导路径信息)的 字符串数组，还可以提供一个 Class。ClassPathXmlAppCtx 从提供的 Class 提取出路径信息。

考虑下面的目录结构：

```
com/
  example/
    services.xml
    repositories.xml
    MessengerService.class
```

下面是实例化：

```java
ApplicationContext ctx = new ClassPathXmlApplicationContext(
    new String[] {"services.xml", "repositories.xml"}, MessengerService.class);
```



---

2.8.2 AppCtx 构造器中 Resource 文件的 通配符

app ctx 构造器的 资源路径可以是 1对1的简单路径，也可以包含 classpath:\*: 前缀 或 Ant风格。这2个可以使用通配符。

这种机制的用处之一是 当你需要进行组件式应用程序组装。所有的组件都可以将 context 定义片段 发布到 众所周知的 位置路径，并且，当最终 app ctx 被通过 以 classpaht\*: 为前缀的 相同路径 创建时，所有组件 片段会被自动获取。

请注意，通配符 只用在 app ctx 构造器 或 PathMatcher 的 资源路径中，在构造时被解析。它不会影响Resource 类型。你不能使用 classpath\*: 前缀 来 构造 Resource，因为一个 Resource 只能指向一个 资源。



---

Ant风格 Patterns

路径地址可以包含 Ant风格 pattern，就像下面：

```
/WEB-INF/*-context.xml
com/mycompany/**/applicationContext.xml
file:C:/some/path/*-context.xml
classpath:com/mycompany/**/applicationContext.xml
```

当路径地址 包含 Ant风格 pattern，解析器 使用更复杂的过程来 尝试解析通配符。它生成一个 Resource for 最后一个 非通配符的路径段 并从中获得一个 URL。     
。。应该是指 这个路径的 最后 一段 不包含通配符的 那段。   
如果这个URL 不是 jar: 或 特定于容器的( 如 zip: for WebLogic， wsjar for WebSphere )，从中获得 java.io.File，并用于 通过遍历来解析通配符文件系统。   
对于jar URL，解析器 要么 从中获取 java.net.JarURLConnection ，要么手动解析jar URL，然后遍历jar 文件的内容以解析通配符。



---

对可移植性的影响

如果制定的path 已经是一个 file URL ( 显式 或 隐式地因为 ResourceLoader 是一个 文件系统)，通配符保证 以完全可移植的方式 工作。

如果指定的路径是 classpath 地址，则解析器必须通过 调用 Classloader.getResource() 来获取 最后一个 非通配符路径段URL。由于这只是路径的一个节点，因此实际上未定义 在这种情况下 返回的 URL类型。   
在实践中，总是一个 java.io.File 代表 目录( classpath resource 被解析为文件系统地址) 或 某种jar URL (classpath resource 被解析为 jar 地址)。这个操作 仍然存在可移植性问题。

如果从最后一个非通配符段获取 jar URL，解析器必须能够从中获取 java.net.JarURLConnection 或手动解析 jar URL， 以便能够 遍历jar的 内容并解析通配符。这在大多数环境中 都有效，但在其他环境中失败，我们强烈建议你在依赖它之前，在你的特定环境中彻底 测试来自jar的 资源的通配符解析。



---

classpath\*: 前缀

构造基于xml的 app ctx 时，一个地址可能使用了 这个前缀：

```java
ApplicationContext ctx =
    new ClassPathXmlApplicationContext("classpath*:conf/appContext.xml");
```

这个特殊前缀 表示 所有匹配这个名字的 classpath 资源 必须被获得 ( 在内部，基本上是通过 调用 ClassLoader.getResource() )，然后合并形成 最终app ctx 定义。

通配符classpath 依赖于 底层的ClassLoader 的 getResources() 方法。   
由于现在大多数应用服务器都提供它们自己的 ClassLoader实现，所以行为可能不同，尤其在处理jar 文件时。   
检查classpath\* 是否生效的一个简单测试是使用 ClassLoader 来 从 classpath 的 jar中 加载文件 (getClass().getClassLoader().getResources("一些在jar 中的文件"))。尝试使用具有相同名称但位于2个不同位置的文件进行此测试 -- 例如，具有相同名称和相同包 但是在不同jar 中的文件。如果返回不适当的结果，请检查app server 文档 以获得可能影响ClassLoader 行为的设置。

你还可以在位置路径的其余部分( 如 classpath\*:META-INF/\*-beans.xml ) 中将 classpath\*: 前缀与PathMatcher 结合起来。这种情况下，解析策略相当简单：在最后一个非通配符路径段上使用 ClassLoader.getResources() 来获取类加载器层次结构中的所有匹配资源，然后对每个资源进行相同的PathMatcher解析 前面描述的策略用于通配符子路径。

。。。感觉 最后一个非同佩服路径段，应该是指 从第一个char开始，到 第一个通配符的前面 这段。



---

通配符的其他说明

注意，当classpath\*: 和Ant模式 结合使用时，能可靠工作，只有当 模式开始前 至少有一个 根目录，除非实际的目标文件驻留在文件系统中。这意味着，classpath\*:\*.xml 之类的可能不会从 jar 的根目录中检索文件，而是从扩展目录的 根目录中检索文件。

spring的检索类路径条目的能力源自jkd的 ClassLoader.getResources() 方法，该方法仅返回 空字符串的文件系统位置 ( 指示要搜索的潜在根)。spring也会评估URLClassLoader 运行时配置 和 jar文件中 java.class.path 清单，但这并不保证会导致 可移植行为。

classpath 包的扫描 需要类路径中存在相对的目录条目。使用Ant构建 jar时，不要激活 files-only 开关。此外，根据某些环境中的安全策略，类路径目录可能不会暴露 -- 例如，Jdk 1.7.0_45 及之上的 独立应用程序 ( 这需要在你的清单中设置 Trusted-Library)

在jdk9 的 module path ( Jigsaw) 上，spring的类路径扫描通常按预期工作。在这里也强烈建议 将资源放入专用目录，避免上述搜索jar文件跟级别的可移植性问题。

使用classpath: 的 Ant风格 的资源 不能保证 找到匹配的资源，如果要搜索的根包 在多个类路径位置中可用。考虑下面的资源地址：

```
com/mycompany/package1/service-context.xml
```

考虑Ant风格 path，用来找上面的文件：

```
classpath:com/mycompany/**/service-context.xml
```

这样的资源可能只存在于类路径中的一个位置，但是当使用前面的例子中的 路径来解析时，解析器会处理由 getResource("com/mycompany") 返回的 (第一个)URL。如果这个基础包结点 存在于 多个ClassLoader 位置，则所需资源可能不存在于 找到的 第一个位置。因此，在这种情况下，你应该更喜欢 具有相同Ant 模式的 classpath\*: ，它搜索包含 com.mycompany 基本包 的所有 类路径位置：classpath\*:com/mycompany/**/service-context.xml 



---

2.8.3 FileSystemResource 注意事项

没有附加到 FileSystemApplicationContext 的 FileSystemResource ( 即，当FileSystemApplicationContext 不是实际的ResourceLoader 时 ) 按照你的预期 处理绝对路径 和相对路径。相对路径是相对于 当前工作目录的，而绝对路径是相对于 文件系统的根目录的。

然后，出于向后兼容性( 历史) 的原因，但FileSystemApplicationContext 是 ResourceLoader 时，这会发生变化。FileSystemAppCtx 强制所有附加的 FileSystemResource 实例 将所有位置路径视为 相对路径，无论是否是 斜杠 开头。 

这意味着下面是等价的：

```java
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("conf/context.xml");
```

```java
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("/conf/context.xml");
```

下面也是等价的：

```java
FileSystemXmlApplicationContext ctx = ...;
ctx.getResource("some/resource/path/myTemplate.txt");
```

```java
FileSystemXmlApplicationContext ctx = ...;
ctx.getResource("/some/resource/path/myTemplate.txt");
```



在实践中，如果你想要 真正的 绝对文件系统路径，则应该避免 绝对路径 与 FileSystemResource 或 FileSystemXmlAppCtx 一起使用，并通过file: 前缀 来强制使用 UrlResource:

```java
// actual context type doesn't matter, the Resource will always be UrlResource
ctx.getResource("file:///some/resource/path/myTemplate.txt");
```

```java
// force this FileSystemXmlApplicationContext to load its definition via a UrlResource
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("file:///conf/context.xml");
```

















---

---

#### 3. Validation, Data Binding, and Type Conversion

将验证视为业务逻辑有利有弊。









Setting and getting properties is done through the `setPropertyValue` and `getPropertyValue` overloaded method variants of `BeanWrapper`



Formatter

Convertor

































































































