

https://docs.spring.io/spring-cloud/docs/2020.0.2/reference/htmlsingle/#spring-cloud-config

spring cloud config

提供了 服务端 和 客户端的 支持，为了 分布式系统中的 外部配置。   
通过Config Server， 你有一个 中央地址 来管理 所有环境的应用的 外部配置。   

#### Quick Start

启动server
```
$ cd spring-cloud-config-server
$ ../mvnw spring-boot:run
```

客户端使用
```
$ curl localhost:8888/foo/development
{
  "name": "foo",
  "profiles": [
    "development"
  ]
  ....
  "propertySources": [
    {
      "name": "https://github.com/spring-cloud-samples/config-repo/foo-development.properties",
      "source": {
        "bar": "spam",
        "foo": "from foo development"
      }
    },
    {
      "name": "https://github.com/spring-cloud-samples/config-repo/foo.properties",
      "source": {
        "foo": "from foo props",
        "democonfigclient.message": "hello spring io"
      }
    },
    ....
```

默认的定位property source的策略是 克隆git repository(在 spring.cloud.config.server.git.uri) 然后用它来初始化一个 迷你的 Spring应用。  
迷你应用的 Environment 用于 枚举 property source，且 发布它们到一个JSON端点。

提供的HTTP 服务：
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties

比如：
curl localhost:8888/foo/development
curl localhost:8888/foo/development/master
curl localhost:8888/foo/development,db/master
curl localhost:8888/foo-development.yml
curl localhost:8888/foo-db.properties
curl localhost:8888/master/foo-db.properties

{application} 是 spring应用中的 spring.config.name.
{profile} 是一个激活的 profile (或逗号分隔的list)
{label}是一个可选的 git label (默认是 master)

spring cloud config server 从不同来源 拉取远程客户端的配置。   
下面的例子 会从 git仓库 获得 配置(git仓库必须提供)：
```
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
```
其他的来源可以是 JDBC兼容的数据库，svn，Hashicorp Vault, Credhub 和 本地文件系统。

---
#### Client Side Usage

为了在应用中使用这些功能，你可以创建它 当作一个SpringBoot应用，应用依赖 spring-cloud-config-client.   
最方便的增加依赖的方法是 SpringBootStarter： org.springframework.cloud:spring-cloud-starter-config. 对于maven用户，也有一个父pom和BOM(spring-cloud-starter-parent), 对于Gradle和SpringCLI用户有一个 SpringIO 版本管理属性文件。   
下面的例子是一个典型的Maven配置：   
pom.xml
```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>{spring-boot-docs-version}</version>
    <relativePath /> <!-- lookup parent from repository -->
</parent>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>{spring-cloud-version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

现在你可以创建一个标准 SpringBoot 应用，下面是一个HTTP server ：
```
@SpringBootApplication
@RestController
public class Application {

    @RequestMapping("/")
    public String home() {
        return "Hello World!";
    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```
当这个 HTTP server 运行，它从 默认 的本地配置服务器的8888端口 (如果有的话) 获取 外部配置。   
要修改这个行为，你可以修改 config server的地址，通过使用 application.properties，就像下面的例子：
spring.config.import=optional:configserver:http://myconfigserver.com

默认情况下，如果没有设置 应用名字， application 会被使用。 修改名字，可以在application.properties中修改：
spring.application.name: myapp

当设置了 ${spring.application.name}, 不要在你的app name前面加 application- 前缀， 防止 在 解析属性源是发生问题。   

Config Server 属性 在 /env 端点展现，作为最高优先级属性来源：
```
$ curl localhost:8080/env
{
  "activeProfiles": [],
  {
    "name": "servletContextInitParams",
    "properties": {}
  },
  {
    "name": "configserver:https://github.com/spring-cloud-samples/config-repo/foo.properties",
    "properties": {
      "foo": {
        "value": "bar",
        "origin": "Config Server https://github.com/spring-cloud-samples/config-repo/foo.properties:2:12"
      }
    }
  },
  ...
}
```

一个名字为 ```configserver:<url of remote repository>/<file name>``` 的 属性来源 包含了 一个 值为 bar，名字为foo 的属性

如果你使用 Spring Cloud Config Client, 你需要设置 spring.config.import 属性 来绑定 Config Server。   

---

#### Spring Cloud Config Server

提供了一个 为外部配置(k-v对 或 yaml) 基于HTTP的 API。   
通过 ***@EnableConfigServer*** ,服务可以嵌入到SpringBoot应用中。   

ConfigServer.java
```
@SpringBootApplication
@EnableConfigServer
public class ConfigServer {
  public static void main(String[] args) {
    SpringApplication.run(ConfigServer.class, args);
  }
}
```
和其他SpringBoot应用一样，默认监听8080端口。 有多种方法把端口改成8888.   
最简单的是，再设置一个默认配置仓库，通过 spring.config.name=configserver(在ConfigServer jar中有一个 configserver.yml).   
另一种是使用你自己的application.properties,就像下面：
```
server.port: 8888
spring.cloud.config.server.git.uri: file://${user.home}/config-repo
```
。。。= 和 : 都可以？   :后需要空格吗？

${user.home}/config-repo 是一个git仓库，包含了yaml 和 properties 文件。

window操作系统，你需要一个额外的 "/" 在 file url， 如果是 带有磁盘前缀的 绝对路径。(如 /${user.home}/config-repo )   

下面展示了 创建git仓库的 命令：
```
$ cd $HOME
$ mkdir config-repo
$ cd config-repo
$ git init .
$ echo info.foo: bar > application.properties
$ git add -A .
$ git commit -m "Add application.properties"
```

使用本地文件作为git仓库，只适合于测试。   
在生产环境，你需要一个服务器来 存储 你的 配置仓库   

如果git仓库只有文本文件，初始的clone是很快的。   
如果你存储了二进制文件，特别是很大的，在第一次请求配置时，你可能有延迟，或者 oom。

---
#### Environment Repository

你应该在哪里存储 ConfigServer 的配置数据？   
管理这个行为的策略是 EnvironmentRepository，服务 Environment对象。这个Environment 是 Spring Environment的 浅拷贝(shallow copy)。   
Environment资源被3个变量参数化：   
{application}: 映射到 客户端的 spring.application.name
{profile}: 映射到 客户端的 spring.profiles.active (是一个 逗号分隔的list)
{label}: 服务端的功能，标记了 已版本控制的 配置文件集。  

一般来说，仓库实现的行为就像一个 SpringBoot应用， 加载配置文件 从一个 spring.config.name 等于{application} 参数，和 spring.profiles.active 等于{profiles}参数。   
> ***profiles的优先级规则 和 普通的SpringBoot应用的规则一样： 激活的profile 优先于默认的，如果有多个profile，最后一个生效。***

下面是简单客户端应用 的 bootstrap配置：
```
pring:
  application:
    name: foo
  profiles:
    active: dev,mysql
```
作为SpringBoot 应用，这些属性也可以被设置到 环境变量 或 命令行参数。  

如果仓库是基于文件的，server创建一个Environment 从 application.yml (在所有客户端间共享) 和 foo.yml (foo.yml优先级高)。   
如果yaml文件 有 document，指向了 Spring profile. 这些 的优先级更高(按照 profile展现的顺序)。   
如果有 特定于profile 的 yaml或properties 文件，这些也比 default 更优先。   
更高的优先级 转换为 Environment中 较早列出的 PropertySource。 (这些规格和 单体SpringBoot应用一致)

你可以设置 spring.cloud.config.server.accept-empty 为 false， 这样 当找不到 application时，返回 HTTP 404。 默认是true。

---

#### Git Backend

EnvironmentRepository 的默认实现 使用 Git。   
要改变仓库的位置，你可以设置 spring.cloud.config.server.git.uri 配置属性 在 ConfigServer中 (比如在 application.yml)   
如果你设置上面这个配置属性的值 以file: 前缀，它会和 本地仓库一起工作，你可以 快速启动 而不需要一个服务器。   
当然，在上面的例子中，server 直接在 本地仓库操作，而不会 clone。(如果ConfigServer 不会修改"远程"仓库，那么不会有问题)。   
为了按比例增加(伸缩) Config Server, 并且使得高可用， 你需要让所有server的实例指向相同的仓库，所以只有一个 共享的文件系统 有用。   
即使在那种情况下，对于一个 共享的文件系统仓库 使用ssh协议 更好，这样，server能clone它，使用一个 本地复制 作为缓存。   

这种仓库实现，映射http资源中的 {label} 参数到一个  git label (commit id, branch name, or tag)。   
如果git branch 或 tag 名字包含/ , HTTP url中的 label 需要用 ```(_)``` 替换。例如，过 label是 foo/bar, 替换/后变成 ```foo(_)bar```   
```(_)``` 的规格也可以应用到 {application} 参数。  
如果你使用 命令行客户端 比如 curl， 小心 URL中的 括号， 你需要在shell中转义它们通过 单引号。   

---

#### Skipping SSL Certificate Validation

配置服务器 对于 Git server的SSL验证 可以 禁用，通过 设置  git.skipSslValidation 为 true (默认是false) 
```
spring:
  cloud:
    config:
      server:
        git:
          uri: https://example.com/my/repo
          skipSslValidation: true
```

---

#### Setting HTTP Connection Timeout
以 秒 为单位，设置 配置服务器 等待 HTTP连接的 时间， 使用 git.timeout
```
spring:
  cloud:
    config:
      server:
        git:
          uri: https://example.com/my/repo
          skipSslValidation: true
```

---

#### Placeholders(占位符) in Git URI

spring cloud config server 支持 git仓库url 的 占位符： {application} {profile} {label}   
你可以支持 "每个应用一个仓库" 通过下面：
```
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/myorg/{application}
```

你也可以 "一个profile 一个仓库" 通过使用 {profile}

另外，使用 特殊string "(_)" 在你的 {application} 参数，能 支持 多个组织 ： 
```
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/{application}
```
这里的{application} 在访问的时候的 格式：  organization(_)application
。。。这个配置里  没有指明 github的 用户。

---
#### Pattern Matching and Multiple Repositories

Spring Cloud Config 也包括 支持 更复杂的需求 通过 pattern match 在 application 和 profile 名字。   
pattern format 是一个 逗号分隔的 由 带通配符的{application}/{profile} 组成 的列表。 (带通配符的pattern 可能需要被 引号包围)：
```
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          repos:
            simple: https://github.com/simple/config-repo
            special:
              pattern: special*/dev*,*special*/dev*
              uri: https://github.com/special/config-repo
            local:
              pattern: local*
              uri: file:/home/configsvc/config-repo
```

如果 {application}/{profile} 不匹配任何 pattern，使用默认的url( 通过 spring.cloud.config.server.git.uri 指定)。   
在上面的例子中，对于 "simple" 仓库， pattern是 simple/* (在所有profile中，只匹配 名为simple的应用)   
"local" 仓库 匹配所有 名字以local开头的 所有的profile的 应用 ( /* 后缀 自动增加到 任意pattern 只要这个pattern没有 profile matcher )   

"simple" 这个例子中 使用了 一行的简写， 只有当 需要设置的属性只有一个，且是URI时，才可以使用。   
如果你还有其他属性需要设置，你必须使用 全格式  

pattern属性 可以是一个数组，所以你能使用 yaml数组 (或 在properties文件中 使用[0],[1]等后缀) 来绑定多个pattern。   
```
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          repos:
            development:
              pattern:
                - '*/development'
                - '*/staging'
              uri: https://github.com/development/config-repo
            staging:
              pattern:
                - '*/qa'
                - '*/production'
              uri: https://github.com/staging/config-repo
```

SpringCloud 猜测 pattern 包含profile，并且这个profile不以* 结尾 意味着你通过这个pattern 匹配 一系列的profile ( 所以 */staging 是 ```["*/staging", "*/staging,*"]``` 的简写)   
这很常见，例如，你需要在本地执行 dev profile，但是在 服务器上执行 cloud profile。
。。。。？

每个仓库也可以 可选地 在子目录 中保存 配置文件， 搜索这些 目录的 pattern 在 searchPaths 中配置：
```
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          searchPaths: foo,bar*
```
在上面的例子中，server 搜索配置文件在 最顶层，和 foo/ 子目录 和任何 以bar开头的子目录

默认下，server clone 远程仓库 当 配置被第一次请求的时候， server 能被配置为 在启动时clone：
```
spring:
  cloud:
    config:
      server:
        git:
          uri: https://git/common/config-repo.git
          repos:
            team-a:
                pattern: team-a-*
                cloneOnStart: true
                uri: https://git/team-a/config-repo.git
            team-b:
                pattern: team-b-*
                cloneOnStart: false
                uri: https://git/team-b/config-repo.git
            team-c:
                pattern: team-c-*
                uri: https://git/team-a/config-repo.git
```
team-a 的配置 在启动时 clone， 其他2个team的 在 仓库被请求时 clone

在启动时clone，可以有助于 分辨 配置错误的 配置源 (如 无效仓库uri)。

---

#### Authentication
为了在 远程仓库 使用 基于HTTP的 验证，增加 username password 属性：
```
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          username: trolley
          password: strongpassword
```

如果你使用HTTPS 和 user 证书，SSH应该开箱即用(work out of the box) 当你的key保存在 默认目录(~/.ssh) 且URI指向SSH地址，比如git@github.com:configuration/cloud-configuration   
～/.ssh/know_hosts 文件中 要有 git server 并且是 ssh-rsa格式的。 其他格式不支持。   
你应该确保只有一个 entry 在 know_hosts 中 for git server,并且url 和你在 config server中的 匹配。   
HTTPS 代理配置 能在 ~/.git/config 或 系统属性(-Dhttps.proxyHost 和 -Dhttps.proxyPort) 中配置   

如果你不知道你的 ～/.git 目录， 使用 git config --global 直接配置 (如 git config --global http.sslVerify false)   

JGit 需要 RSA key 在 PEM 格式中，下面是例子： ssh-keygen (from openssh) 命令来创建 key 在正确的格式：
```
ssh-keygen -m PEM -t rsa -b 4096 -f ~/config_server_deploy_key.rsa
```

警告：使用SSH key，期望的ssh private-key 必须以 ```-----BEGIN RSA PRIVATE KEY-----``` 开始，如果以```-----BEGIN OPENSSH PRIVATE KEY-----``` 开始，则在spring-cloud-config server启动时，不会加载 RSA key，报错类似：
```
- Error in object 'spring.cloud.config.server.git': codes 
[PrivateKeyIsValid.spring.cloud.config.server.git,PrivateKeyIsValid]; 
arguments [org.springframework.context.support.DefaultMessageSourceResolvable: 
codes [spring.cloud.config.server.git.,]; arguments []; default message []]; 
default message [Property 'spring.cloud.config.server.git.privateKey' is not a valid private key]
```
为了纠正上述错误，RSA key必须是 PEM 格式。   

---
#### Authentication with AWS CodeCommit
spring cloud config server 也支持 AWS CodeCommit 验证。   
AWS CodeCommit 使用一个 验证帮助器，当在 命令行使用git时。   
这个帮助器不能和 JGit lib 一起使用，所以 JGit CredentialProvider for AWS CodeCommit 被创建 如果URI 匹配 AWS CodeCommit的模式。   
AWS CodeCommit URI的模式：
```
https//git-codecommit.${AWS_REGION}.amazonaws.com/v1/repos/${repo}.
```

如果你提供了 username password 给 AWS CC URI，它们必须是 AWS accessKeyId 和 secretAccessKey.   
如果没有指定 username password， accessKeyId 和 secretAccessKey 会使用 AWS Default Credential Provider Chain 来检索   

如果Git URI 匹配 CodeCommit URI， 你必须提供有效的 AWS 凭证 在 username password 或 是default credential provider chain 支持的某个位置。   
AWS EC2 实例 应该使用 IAM Roles for EC2 Instances (https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)

aws-java-sdk-core jar是一个可选依赖， 如果不在classpath中，AWS Code Commit credential provider 不会被创建， 不论 git server 的URI 是否匹配。

---

#### Authentication with Google Cloud Source

支持 对 Google Cloud Source(https://cloud.google.com/source-repositories/) 仓库的 验证   

如果Git URI 使用 http 或 https，域名是 source.developers.google.com，Google Cloud Source credentials provider 会被创建。   
Google Cloud Source URI 仓库的格式 ```source.developers.google.com/p/${GCP_PROJECT}/r/${REPO}```   
为你的仓库 获得这个URI，需要点击 "Clone", 选择 "Manually generated credentials". 不要创建任何凭证，只需要复制展现的URI。

com.google.auth:google-auth-library-oauth2-http 可选jar，如果没有，不会创建 Google Cloud Source credential provider，不管git server URI。

---
#### Git SSH 配置，通过 properties

默认情况下， spring cloud config server 使用的 JGit 使用 SSH配置文件 比如 ～/.ssh/know_hosts 和 /etc/ssh/ssh_config。   
在云环境下，本地文件系统 是短暂的(ephemeral)，或 不容易访问，对于这些情况，SSH配置 可以设置到 java properties中。   
为了使用 基于properties的 SSH 配置， ```spring.cloud.config.server.git.ignoreLocalSshSettings``` 必须设置为 true。
```
  spring:
    cloud:
      config:
        server:
          git:
            uri: git@gitserver.com:team/repo1.git
            ignoreLocalSshSettings: true
            hostKey: someHostKey
            hostKeyAlgorithm: ssh-rsa
            privateKey: |
                         -----BEGIN RSA PRIVATE KEY-----
                         MIIEpgIBAAKCAQEAx4UbaDzY5xjW6hc9jwN0mX33XpTDVW9WqHp5AKaRbtAC3DqX
                         IXFMPgw3K45jxRb93f8tv9vL3rD9CUG1Gv4FM+o7ds7FRES5RTjv2RT/JVNJCoqF
                         ol8+ngLqRZCyBtQN7zYByWMRirPGoDUqdPYrj2yq+ObBBNhg5N+hOwKjjpzdj2Ud
                         1l7R+wxIqmJo1IYyy16xS8WsjyQuyC0lL456qkd5BDZ0Ag8j2X9H9D5220Ln7s9i
                         oezTipXipS7p7Jekf3Ywx6abJwOmB0rX79dV4qiNcGgzATnG1PkXxqt76VhcGa0W
                         DDVHEEYGbSQ6hIGSh0I7BQun0aLRZojfE3gqHQIDAQABAoIBAQCZmGrk8BK6tXCd
                         fY6yTiKxFzwb38IQP0ojIUWNrq0+9Xt+NsypviLHkXfXXCKKU4zUHeIGVRq5MN9b
                         BO56/RrcQHHOoJdUWuOV2qMqJvPUtC0CpGkD+valhfD75MxoXU7s3FK7yjxy3rsG
                         EmfA6tHV8/4a5umo5TqSd2YTm5B19AhRqiuUVI1wTB41DjULUGiMYrnYrhzQlVvj
                         5MjnKTlYu3V8PoYDfv1GmxPPh6vlpafXEeEYN8VB97e5x3DGHjZ5UrurAmTLTdO8
                         +AahyoKsIY612TkkQthJlt7FJAwnCGMgY6podzzvzICLFmmTXYiZ/28I4BX/mOSe
                         pZVnfRixAoGBAO6Uiwt40/PKs53mCEWngslSCsh9oGAaLTf/XdvMns5VmuyyAyKG
                         ti8Ol5wqBMi4GIUzjbgUvSUt+IowIrG3f5tN85wpjQ1UGVcpTnl5Qo9xaS1PFScQ
                         xrtWZ9eNj2TsIAMp/svJsyGG3OibxfnuAIpSXNQiJPwRlW3irzpGgVx/AoGBANYW
                         dnhshUcEHMJi3aXwR12OTDnaLoanVGLwLnkqLSYUZA7ZegpKq90UAuBdcEfgdpyi
                         PhKpeaeIiAaNnFo8m9aoTKr+7I6/uMTlwrVnfrsVTZv3orxjwQV20YIBCVRKD1uX
                         VhE0ozPZxwwKSPAFocpyWpGHGreGF1AIYBE9UBtjAoGBAI8bfPgJpyFyMiGBjO6z
                         FwlJc/xlFqDusrcHL7abW5qq0L4v3R+FrJw3ZYufzLTVcKfdj6GelwJJO+8wBm+R
                         gTKYJItEhT48duLIfTDyIpHGVm9+I1MGhh5zKuCqIhxIYr9jHloBB7kRm0rPvYY4
                         VAykcNgyDvtAVODP+4m6JvhjAoGBALbtTqErKN47V0+JJpapLnF0KxGrqeGIjIRV
                         cYA6V4WYGr7NeIfesecfOC356PyhgPfpcVyEztwlvwTKb3RzIT1TZN8fH4YBr6Ee
                         KTbTjefRFhVUjQqnucAvfGi29f+9oE3Ei9f7wA+H35ocF6JvTYUsHNMIO/3gZ38N
                         CPjyCMa9AoGBAMhsITNe3QcbsXAbdUR00dDsIFVROzyFJ2m40i4KCRM35bC/BIBs
                         q0TY3we+ERB40U8Z2BvU61QuwaunJ2+uGadHo58VSVdggqAo0BSkH58innKKt96J
                         69pcVH/4rmLbXdcmNYGm6iu+MlPQk4BUZknHSmVHIFdJ0EPupVaQ8RHT
                         -----END RSA PRIVATE KEY-----
```
下面是 SSH 配置的属性   
```
ignoreLocalSshSettings   ：spring.cloud.config.server.git.ignoreLocalSshSettings   
privateKey   
hostKey             ：ssh-dss, ssh-rsa, ecdsa-sha2-nistp256, ecdsa-sha2-nistp384, or ecdsa-sha2-nistp521   
hostKeyAlgorithm      
strictHostKeyChecking   
knownHostsFile    
preferredAuthentications    
```

#### PlaceHolders in Git Search Paths
支持 {application} {profile} {label} 占位符 ：
```
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          searchPaths: '{application}'
```

#### Force pull in Git Repositories

spring cloud config server 制作 远程git仓库的 clone， 如果本地副本被 污染了(如 文件夹内容被 OS修改)，以至于 spring cloud config server 不能从 远程仓库 更新本地副本。   

为了解决这个问题， force-pull 属性， 让 config server 强制拉取 从远程仓库，如果本地副本被污染：
```
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          force-pull: true
```

如果你有多个仓库配置，你可以配置每个仓库的 force-pull:
```
spring:
  cloud:
    config:
      server:
        git:
          uri: https://git/common/config-repo.git
          force-pull: true
          repos:
            team-a:
                pattern: team-a-*
                uri: https://git/team-a/config-repo.git
                force-pull: true
            team-b:
                pattern: team-b-*
                uri: https://git/team-b/config-repo.git
                force-pull: true
            team-c:
                pattern: team-c-*
                uri: https://git/team-a/config-repo.git
```

force-pull 默认 false

---
#### Deleting untracked branches in Git Repositories

由于 config server 有一个clone ， 在 checkout分支到本地仓库后， config server会一直保持这个branch，直到重启(重启会创建新的本地仓库)。   
所以可能 远程分支被删除了，但是本地依然有效。   
如果config server 客户端服务 以 ```--spring.cloud.config.label=deletedRemoteBranch,master``` 启动， 它会从 deletedRemoteBranch 本地分支获得属性，而不是 master。

为了保持 本地仓库 和 remote 一致， deleteUntrackedBranches 属性被设置。 它会让 config server 强制 删除 本地仓库中的 untracked 分支：
```
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          deleteUntrackedBranches: true
```
deleteUntrackedBranches 默认 false

---
#### Git Refresh Rate

使用 spring.cloud.config.server.git.refreshRate 来控制 config server 更新配置的频率。 单位是 秒。   
默认是0,意味着 每次当config server 被请求时， config server 都会去 fetch 配置。

---
#### Version Control Backend FileSystem Use

使用 基于VSC 的backend (git,svn), 文件被 checkout 或 clone 到本地文件系统。默认情况下，他们被放在 系统的临时文件目录，以 config-repo- 开头。   
Linux，可能是 /tmp/config-repo-<randomid>    
有些OS，会 常规地清理 临时目录。 这会导致 无法预期的行为，必须 缺少配置。
避免这个问题，通过 spring.cloud.config.server.git.basedir 或 spring.cloud.config.server.svn.basedir 来修改 目录。

---

#### File System Backend
config server 有一个 native profile， 从local classpath 或 文件系统 (spring.cloud.config.server.native.searchLocations 指定的 静态URL) 来加载 配置文件。   
要使用这个 native profile。 配置 spring.profiles.active=native  

记得使用 file: 前缀。没有这个前缀 被认为是 classpath。   
和任何spring boot配置一样，你能 嵌入一个 ${} 格式的环境占位符，记住 windows中的 绝对路径需要一个 额外的 / (如 /${user.home}/config-repo)   

searchLocations 的默认值是 和 boot应用完全一样 (是 ```[classpath:/, classpath:/config, file:./, file:./config]``` )   
这不会暴露 application.properties 从服务端到 客户端，因为 任何 server中的 property source 会被移除， 在 发送到客户端之前。   
。。。应该是指， 客户端不会知道 属性来源。

基于文件系统的 config server 启动很快，适合test。 但是在生产，你需要确保 文件系统是 可靠的，并且能被 所有 config server 的实例 访问。

search location 能包含 占位符 {application} {profile} {label}

如果在 search location 中没有使用 占位符，这个仓库也 增加{label} 参数 在 HTTP中 来 作为 search path 的后缀。   
没有占位符的默认行为，就像加了一个/{label}/ 。 例如， file:/tmp/config 等同于 file:/tmp/config, file:/tmp/config/{label}。
这种行为能通过 spring.cloud.config.server.native.addLabelLocations=false 来禁用。

---

#### Vault Backend

config server 支持 Vault

为了让 config server 使用 Vault，可以让 config server 跑在 vault profile。 比如，在你的config server 的 application.properties 中 可以增加 spring.profiles.active=vault   

默认下，config server 假设你的 Vault server 运行在 127.0.0.1：8200 ， 名字是 secret， key是 application。 这些都可以配置：
```
Name 	            Default Value
host                127.0.0.1
port                8200
scheme              http
backend             secret
defaultKey          application
profileSeparator    ,
kvVersion           1
skipSslValidation   false
timeout             5
namespace           null
```

上面属性的前缀 都是 spring.cloud.config.server.vault

所有可配置属性能在 org.springframework.cloud.config.server.environment.VaultEnvironmentProperties 中找到。

Vault 0.10.0 有一个版本控制的 k-v 后端。 需要一个 data/ 在 mount 路径 和 实际路径 在 data 对象中。 设置 spring.cloud.config.server.vault.kv-version=2 会加载这个。

```
。。。跳过了，简单复制一些
可选的，支持  X-Vault-Namespace 头， 发送这个到Vault 设置 namespace 属性。
spring.cloud.config.server.vault.authentication

$ vault kv put secret/application foo=bar baz=bam
$ vault kv put secret/myapp foo=myappsbar

$ curl -X "GET" "http://localhost:8888/myapp/default" -H "X-Config-Token: yourtoken"

spring.cloud.vault

```

---

#### Multiple Properties Sources
使用Vault，你可以提供给你的app 多个 属性源。 例如，假设你在Vault中 写入数据到如下的路径：
```
secret/myApp,dev
secret/myApp
secret/application,dev
secret/application
```
secret/application 的属性 可以被所有 使用 config server的 应用获得。     
一个名叫 myApp的应用可以获得 secret/myApp  和 secret/application 的 属性   
当 myApp 有 dev profile时， 可以获得 所有上面4个路径的 属性，第一个路径中的属性 优先于 其他的。

---
#### Accessing Backends Through a Proxy
能通过 Http或 Https 代理 访问 Git 或Vault。   
通过 proxy.http, proxy.https 来控制这种行为。   
这些设置 是每个 仓库的。   
可以同时设置Http 和 Https 代理 为一个backend。   

下面是 代理的配置属性 for http 和 https 代理。   
都需要加前缀 proxy.http 或 proxy.https  
```
host
port
nonProxyHosts
username
password
```

下面使用Https 代理来访问 Git 仓库
```
spring:
  profiles:
    active: git
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          proxy:
            https:
              host: my-proxy.host.io
              password: myproxypassword
              port: '3128'
              username: myproxyusername
              nonProxyHosts: example.com
```

---
#### Sharing Configuration with All Applications

在所有应用间共享配置 根据你采用的方法 不同 而不同：   
File Based Repositories
Vault Server

---
#### File Based Repositories
使用基于文件的仓库(git,svn,native)， 文件名为 application* (application.properties,application.yml,application-*.properties等) 的资源 在所有客户端app间共享。   
你可以使用 这种命名的资源来配置 全局默认值，然后在 应用独享 的文件中 覆盖它们。

属性覆盖 (https://docs.spring.io/spring-cloud/docs/2020.0.2/reference/htmlsingle/#property-overrides)
也允许用来设置 全局默认，with 占位符应用可以在本地覆盖它们。

使用"native" profile( 后端是 本地文件系统)，你应该使用一个明确的搜索地址，这个地址不是server自己的配置的一部分。
否则，在默认搜索地址的application*资源 会被移除，因为他们是server的一部分。

---
#### Vault Server
当你使用 Vault 作为后端，你可以分享配置 在所有应用间， 通过 把配置放在 secret/application。   
例如，如果你执行 下面的 vault 命令， 所有的使用config server的应用 可以获得 foo 和 baz 属性
```
$ vault write secret/application foo=bar baz=bam
```

#### CredHub Server
把需要共享的配置放到 /application/，或者放到 default profile。   
若你执行下面的 CredHub 命令，所有使用 config server的应用 能获得 shared.color1 和 shared.color2
```
credhub set --name "/application/profile/master/shared" --type=json
value: {"shared.color1": "blue", "shared.color": "red"}
```
```
credhub set --name "/my-app/default/master/more-shared" --type=json
value: {"shared.word1": "hello", "shared.word2": "world"}
```

---
#### JDBC Backend
spring cloud config server 支持 JDBC (关系型数据库)。   
激活这个功能 通过增加 spring-jdbc 到 classpath， 使用 jdbc profile 或增加一个 JdbcEnvironmentRepository类型的bean   
如果你在 classpath中 包含了正确的 依赖， spring-boot会配置数据源。   

你可以禁用 JdbcEnvironmentRepository 的自动配置， 通过 spring.cloud.config.server.jdbc.enabled=false   

数据库需要有一张表 叫做 properties， 有列：application,profile,label, 加上 key，value作为properties的k-v对。   
所有的 列都是 java的String， 所以可以用 varchar，任意长度。   
属性值的行为就像 它们在 SpringBoot 属性文件 {application}-{profile}.properties 中 一样，包括 所有的 加密 解密。

---
#### Redis Backend
支持Redis 作为后端 用于 保存 配置属性。   
增加 依赖 Spring Data Redis 来激活这个功能。   

pom.xml
```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
</dependencies>
```

下面的配置 使用SpringData RedisTemplate 来访问 Redis，可以使用 spring.redis.* 属性来 覆盖默认连接配置：
```
spring:
  profiles:
    active: redis
  redis:
    host: redis
    port: 16379
```

属性应该被保存在 hash 的 field中。 hash的名字应该和 spring.application.name 一样，或者是 spring.application.name 和 spring.profiles.active[n] 的连接   

> HMSET sample-app server.port "8100" sample.topic.name "test" test.property1 "property1"

上面执行完后，能查到下面的：
```
HGETALL sample-app
{
  "server.port": "8100",
  "sample.topic.name": "test",
  "test.property1": "property1"
}
```

没有profile，就用 default

---

#### AWS S3 Backend
支持AWS S3。   
激活这个功能通过增加 AWS java SDK For Amazon S3 的依赖

pom.xml
```
<dependencies>
    <dependency>
        <groupId>com.amazonaws</groupId>
        <artifactId>aws-java-sdk-s3</artifactId>
    </dependency>
</dependencies>
```

下面的配置 使用 AWS S3 客户端来访问 配置文件， 可以使用 spring.awss3.* 选择 存储你的配置的 bucket
```
spring:
  profiles:
    active: awss3
  cloud:
    config:
      server:
        awss3:
          region: us-east-1
          bucket: bucket1
```

可以通过 spring.awss3.endpoint 来覆盖 你的S3 服务的 标准端点。

访问凭证 被 Default AWS Credential Provider Chain 找到。

你的bucket中的 配置文件就像： {application}-{profile}.properties, {application}-{profile}.yml, {application}-{profile}.json   
可选的label，用来确定 文件目录的路径   

profile 默认 default

---
#### CredHub Backend

pom.xml
```
<dependencies>
    <dependency>
        <groupId>org.springframework.credhub</groupId>
        <artifactId>spring-credhub-starter</artifactId>
    </dependency>
</dependencies>
```

使用 mutual(相互的) TLS 访问
```
spring:
  profiles:
    active: credhub
  cloud:
    config:
      server:
        credhub:
          url: https://credhub:8844
```

属性应该被保存为JSON，就像：
```
credhub set --name "/demo-app/default/master/toggles" --type=json
value: {"toggle.button": "blue", "toggle.link": "red"}
```
```
credhub set --name "/demo-app/default/master/abs" --type=json
value: {"marketing.enabled": true, "external.enabled": false}
```

所有客户端应用 with spring.cloud.config.name=demo-app 的 可以获得下列属性
```
{
    toggle.button: "blue",
    toggle.link: "red",
    marketing.enabled: true,
    external.enabled: false
}
```

没有profile，使用default， 没有label，使用master。
在application中的数据 会被 所有app 共享。

#### OAuth 2.0

身份验证 with OAuth 2.0 使用 UUA 作为provider

pom.xml
```
<dependencies>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-config</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-oauth2-client</artifactId>
    </dependency>
</dependencies>
```

下面的配置使用 OAuth 2.0 和 UAA 来访问 CredHub
```
spring:
  profiles:
    active: credhub
  cloud:
    config:
      server:
        credhub:
          url: https://credhub:8844
          oauth2:
            registration-id: credhub-client
  security:
    oauth2:
      client:
        registration:
          credhub-client:
            provider: uaa
            client-id: credhub_config_server
            client-secret: asecret
            authorization-grant-type: client_credentials
        provider:
          uaa:
            token-uri: https://uaa:8443/oauth/token
```

UUA client-id 应该有 credhub.read as scope

---
#### Composite Environment Repositories

一些场景中，你可能希望拉取配置数据 从多个 Environment 仓库。   
你可以激活 composite profile 在你的 配置服务器 的application properies或yaml文件。   
例如，你想从 1台svn，2台git 中拉取 配置数据：
```yaml
spring:
  profiles:
    active: composite
  cloud:
    config:
      server:
        composite:
        -
          type: svn
          uri: file:///path/to/svn/repo
        -
          type: git
          uri: file:///path/to/rex/git/repo
        -
          type: git
          uri: file:///path/to/walter/git/repo
```

优先级通过 仓库的 顺序来决定。   
在上面的例子中， svn 写在最前面，所以 如果 从svn中 发现的值 会覆盖 从git中发现的值。

如果你想拉取配置文件，从不同的类型的存储器。   
你可以 激活相应的 profile， 而不是 composite， 在你的 配置服务器的 application properties 或yaml文件中。   
例如，你想pull配置数据 从 一台git 和一台 HashCorp Vault。你可以配置：
```yaml
spring:
  profiles:
    active: git, vault
  cloud:
    config:
      server:
        git:
          uri: file:///path/to/git/repo
          order: 2
        vault:
          host: 127.0.0.1
          port: 8200
          order: 1
```

使用这个配置， 优先级被 order 属性决定。 数值越小，越有限。

当使用 composite时， 确保 所有 仓库都有 相同的 label。 因为一个仓库失败，全部失败。


#### Custom Composite Environment Repositories
除了使用 SpringCloud 提供的 environment repositories，你也可以提供自己的 EnvironmentRepository bean 来被作为 composite 环境的一部分。   
你的bean必须实现 EnvironmentRepository接口。 如果你想控制 优先级，那么 实现 Ordered 接口 并覆盖getOrdered()方法，如果没有实现Ordered接口，你的EnvironmentRepository 是最低优先级。   

---
---
---
### Property Overrides
https://docs.spring.io/spring-cloud/docs/2020.0.2/reference/htmlsingle/#property-overrides

Config Server 有 覆盖这种特性， 使得 operator 提供 配置属性 给所有应用。   
被覆盖的属性不能被 应用通过普通的SpringBoot hook 来修改。   
为了声明 覆盖，增加一个 name-value 的map 到 spring.cloud.config.server.overrides :
```yaml
spring:
  cloud:
    config:
      server:
        overrides:
          foo: bar
```
上面的配置会 使得 所有 config client应用 读到 foo=bar， 在它们自己的配置之外。   

spring environment placeholders with ${} 能被转义 ( 在client中resolve) 通过使用 \ 来转义$ 或 {。  如 \${app.foo:bar}  获得bar 除非应用提供它自己的 app.foo   

在yaml中，你不需要转义 \ 它自己， 在properties中，你需要转义\ 。

你可以将客户端中所有 覆盖的优先级 更改为更新 默认值，让应用提供它们自己的值 在 环境变量 或 系统属性，通过设置 spring.cloud.config.overrideNone=true (默认false) 在远程仓库。


---
#### Health Indicator

Config Server 提供了 健康指示器，来检查 配置的 EnvironmentRepository 是否在工作。   
默认，它 询问 EnvironmentRepository for 一个 名叫app的应用。 default profile，默认的label ，由 EnvironmentRepository实现提供。

你可以配置 Health Indicator 检查更多 应用 用自定义profile和label：
```yaml
spring:
  cloud:
    config:
      server:
        health:
          repositories:
            myservice:
              label: mylabel
            myservice-dev:
              name: myservice
              profiles: development
```

禁用健康指示器： management.health.config.enabled=false

---
#### Security

你可以保护你的Config Server 通过任何手段(从物理隔离到OAuth2令牌)，因为SpringSecure 和SpringBoot支持许多安全措施   

要使用 默认的 SpringBoot 配置的 HTTP 基础安全保护，把Spring Security放在classpath中 (如，通过spring-boot-starter-security).   
默认 user 这个用户名 和一个 随机生成的密码。   
随机密码 实践中没有什么作用，可以配置密码 spring.security.user.password 且 加密它   

---
#### Encryption and Decryption

要使用加密和解密，需要 完整的JCE 安装在你的JVM (默认不包含JCE的 )中。   
你可以下载 Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files 从Oracle。   
然后安装。( 你需要 用下载的文件 替换2个policy文件 在 JRE lib/security目录)

如果远程属性源 包含了加密内容 (以{cipher}开头的值)，它们会被解密 在发送给客户端之前。   
如果值无法被解密，它会被移除 从property source，会增加一个属性， key是无法解密的value的key，值是 invalid 和 表达不可用的标识(通常是N/A)。防止 密文泄露。   

如果你设置了 远程配置仓库 for 配置客户端应用。应该包含application.yml ，类似：
application.yml
```yaml
spring:
  datasource:
    username: dbuser
    password: '{cipher}FKSAJDFGYOS8F7GLHAKERGFHLSAJ'
```

如果是application.properties，那么 密文不需要 引号 。   

application.properties
```properties
spring.datasource.username: dbuser
spring.datasource.password: {cipher}FKSAJDFGYOS8F7GLHAKERGFHLSAJ
```

server 暴露了 /encrypt 和 /decrypt 端点 ，用来 加密 原文。
```
$ curl localhost:8888/encrypt -s -d mysecret
682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda
```

如果你用 curl 测试，那么使用 --data-urlencode (代替-d)， 在原文前面加= 或者明确设置 Content-Type: test/plain 来确保curl 转码正确。

确保不要包含 任何 curl命令statistics 在 加密的值中，这就是为什么 上面的例子中使用 -s 来静默它们。 输出值到一个文件 有助于避免这个问题。   

/decrypt 用于解密 (服务器需要配置 对称密钥 或 完整密钥对)   
```
$ curl localhost:8888/decrypt -s -d 682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda
mysecret
```

把加密后的值，加一个{cipher} 前缀，然后写到yaml或properties文件。

/encrypt 和 /decrypt，都接受/*/{application}/{profiles} 的路径，能被用来控制 每个app每个profile的 密码使用方式   

为了控制 密码使用方式 在/*/{application}/{profiles} 这个粒度，你需要提供一个 TextEncryptorController 类型的 Bean， 来对 每个app每个profile创建不同的 加密器。   

spring 命令行客户端(需要安装SpringCloud CLI) 也能用来 加密解密：
```
$ spring encrypt mysecret --key foo
682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda
$ spring decrypt --key foo 682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda
mysecret
```

要使用 文件中的key (比如一个 RSA公钥 用于加密)， 在密钥值前面加一个 @，和提供的文件路径：
```
$ spring encrypt mysecret --key @${HOME}/.ssh/id_rsa.pub
AQAjPgt3eFZQXwt8tsHAVv/QHiY5sI2dRcR+...
```

--key 参数是必须的。

---
#### Key Management
ConfigServer 能使用 对称密钥 或 非对称密钥(RSA密钥对)。   
非对称 安全性更好。   
对称 更方便，因为它只需在 bootstrap.properties 中配置的 一个property 就可以   

为了配置 对称key，你需要设置 encrypt.key 为一个 密钥。 (或者使用 ENCRYPT_KEY 环境变量 来让密钥 和 配置分离)

encrypt.key 不能用于 非对称密钥。   

为了配置非对称密钥 使用一个 keystore (比如 被JDK的keytool创建)：
```
encrypt.keyStore.location   Contains a Resource location
encrypt.keyStore.password   Holds the password that unlocks the keystore
encrypt.keyStore.alias      Identifies which key in the store to use
encrypt.keyStore.type       The type of KeyStore to create. Defaults to jks.
```

加密用公钥，解密用私钥。

你可以只配公钥，如果这个server 只需要加密。

---
#### Creating a Key Store for Testing

为了创建keyStore,你可以使用：
```
$ keytool -genkeypair -alias mytestkey -keyalg RSA \
  -dname "CN=Web Server,OU=Unit,O=Organization,L=City,S=State,C=US" \
  -keypass changeme -keystore server.jks -storepass letmein
```

jdk11 及之后，你可以收到下面的警告，这种情况下，你需要确保 keypass 和 storepass 值匹配
```
Warning:  Different store and key passwords not supported for PKCS12 KeyStores. Ignoring user-specified -keypass value.
```

把 server.jks 文件 放到classpath，在ConfigServer的 bootstrap.yml 中 配置：
```
encrypt:
  keyStore:
    location: classpath:/server.jks
    password: letmein
    alias: mytestkey
    secret: changeme
```

---
#### Using Multiple Keys and Key Rotation

在密文前面加 {cipher} 前缀后，ConfigServer 查询 0或多个 {name:value} 前缀，在 启动cipher 文本前。   
keys被传递到 TextEncryptLocator, 这个类会 找到 cipher的 TextEncryptor。   
如果你配置了keystore (encrypt.keystore.location)， 默认locator会 搜索密钥 通过key前缀提供的别名：
```
foo:
  bar: `{cipher}{key:testkey}...`
```

定位器查找一个 称为 testkey 的key， secret 也能通过 在前缀中以 {secret:...} 的方式提供。 如果没有提供，默认就使用keystore密码，如果你提供了secret，你也应该加密secret 通过自定义的 SecretLocator。

当密钥只被用来 加密 少量byte的配置数据，key rotation 几乎不需要。  
你可能 偶尔需要修改key，此时，所有的客户端 需要修改它们的源配置文件(比如，在git中)，使用 {key:...}  的前缀在所有的 cipher。   
记住：客户端需要首先检查 key alias 是否在ConfigServer keystore 中存在。   

如果你希望让 ConfigServer 处理所有的 加解密， {name:value} 前缀 也能被增加到 发送给/encrypt 的 文本中。

---
#### Serving Encrypted Properties

有时，你想客户端 本地加密配置，而不是在服务端。   
如果你提供了 encrypt.* 配置 来定位一个 key，你仍然有 /encrypt /decrypt， 但你需要 明确切换 输出属性的解密， 通过 spring.cloud.config.server.encrypt.enabled=false 在 bootstrap.yml/properties。   
如果你不关注端点，如果你不配置密钥或启动标记，它应该可以工作。

---
#### Serving Alternative Formats

环境端点的 默认 JSON格式 是 非常 适合 Spring应用 消费的，因为它直接 映射 Environment 抽象。   
你也可以以 YAML 或 properties 的格式 消费， 通过增加一个前缀 (.yml .yaml .properties) 到 资源路径。   

yaml 和 properties 的格式 有一个额外的标志 (resolvePlaceholders 的bool) 来表示 源文档中的 占位符 (${...}) 应该在 渲染之前在 输出中解析。   

yaml 和 properties 格式 有限制，主要是和 元数据的丢失有关。   
比如，JSON能构建 属性源的 有序列表， yaml 和properties中只是一个 map。 而且 原始的source files 的名字也会丢失。   

---
#### Serving Plain Text

不使用 Environment 抽象 (或 yaml 或 properties 格式)，你的app需要 文本配置文件，这些文件是为环境定制的。   
ConfigServer 提供它们 通过 额外的端点 在 /{application}/{profile}/{label}/{path}, 这里的application,profile,label 和 普通环境端点有相同的意义，path是一个到文件名的路径。   
这个端点的源文件 的定位方式 和 环境端点的 相同。   
只有第一个匹配的 返回。   

在资源被定位后，占位符 被解析，通过有效的 Environment for 支持的app名字，profile，label。

profile 用于解析文件名字， 所以如果你需要一个 profile-specific 的文件 /*/development/*/logback.xml， 能被 名为 logback-development.xml 的文件解析 ( 优先于 logback.xml )

如果你不想支持 label， 并 让server 使用默认label，你可以提供一个 useDefaultLabel 的请求参数。  /sample/default/nginx.conf?useDefaultLabel

---
目前，SpringCloudConfig 能为 git,svn,本地，AWS S3 提供 纯文本服务。AWS S3的 和其他有一点不同。

#### Git, SVN, Native Backends

考虑下面的 git svn native 的例子：
```
application.yml
nginx.conf
```

nginx.conf 内容类似：
```
server {
    listen              80;
    server_name         ${nginx.server.name};
}
```

application.yml 类似：
```
nginx:
  server:
    name: example.com
---
spring:
  profiles: development
nginx:
  server:
    name: develop.com
```

/sample/default/master/nginx.conf 类似：
```
server {
    listen              80;
    server_name         example.com;
}
```

/sample/development/master/nginx.conf :
```
server {
    listen              80;
    server_name         develop.com;
}
```

---
#### AWS S3

需要包含 spring cloud aws依赖，还需要配置。

---
#### Decrypting Plain Text

默认下，文本文件中 加密的值 是不会被解密的   
为了解密，配置 spring.cloud.config.server.encrypt.enabled=true 和 spring.cloud.config.server.encrypt.plainTextEncrypt=true 到 bootstrap.yml/properties   

解密文本文件 只支持 yaml json properties 文件扩展名。

如果激活了这个功能，但是 需要对 不支持的 文件扩展名 解密， 这个文件里的 所有加密的值 不会被 解密。   

---
#### Embedding the Config Server
config server 最好是作为 单体应用运行。   
不过，它也可以嵌入到 另一个应用中。   
> @EnableConfigServer

一个可选的属性 spring.cloud.config.server.bootstrap ，这是一个标记，来 表明 server 是否应该配置它自己 按照它自己的远程仓库。 默认是关闭(off)，因为它可以延迟启动。   
当嵌入到其他一样，与其他应用 按相同的方式 初始化是有意义的。 当设置 spring.cloud.config.server.bootstrap为true， 你必须使用 **_composite_** environment repository configuration， 如：
```
spring:
  application:
    name: configserver
  profiles:
    active: composite
  cloud:
    config:
      server:
        composite:
          - type: native
            search-locations: ${HOME}/Desktop/config
        bootstrap: true
```

如果你使用bootstrap 标记，config server 需要有它自己的名字 和 仓库URI，配置在bootstrap.yml中。   

修改server端点的位置， 你可以设置 spring.cloud.config.server.prefix (比如设置为 /config) 来 server 资源 在prefix下。prefix以/开始，不能以/结束。它应用到 config server 的 @RequestMappings (也就是说，在SpringBoot的 server.servletPath 和 server.contextPath 前缀下) 。

如果你想直接从 仓库读取 应用的配置 (而不是从config server)， 你需要一个 没有endpoint的嵌入式的config server， 你可以关闭所有端点，通过 不使用 @EnableConfigServer (设置 spring.cloud.config.server.bootstrap=true)   

---
#### Push Notifications and Spring Cloud Bus

许多 源代码仓库 提供者(github,gitlab,gitea,gitee,gogs,bitbucket) 通过webhook 提醒你 仓库中的修改。   
你可以配置webhook 通过 提供者的 用户接口。   
比如，github 使用 POST 发送JSON(包含了commit列表和 header(X-Github-Event 设置为push)) 到 webhook。   
如果你增加了 spring-cloud-config-monitor 的依赖，且激活 config server 中的 SpringCloudBus。 /monitor 端点可用。   

当webhook 激活，config server 发送 RefreshRemoteApplicationEvent 对于它认为已修改的app。   
修改探测策略是可配置的。默认，它 搜索 根据匹配应用名字的文件 的修改 (如，foo.properties 被foo应用， application.properties 被所有应用关注)   
策略可以通过 重写 PropertyPathNotificationExtractor 中的方法来修改，这些接受 request header 和body 作为参数，返回 被修改的文件路径列表。

默认的配置对于 github,gitlab,gitea,gitee,gogs,bitbucket 开箱即用。   
除了来自上述 的JSON 通知外， 你还可以通过 path={application} 格式的 表单格式的body 通过POST 发送到 /monitor 来出发一个修改。 这样做会广播到与 {application} 匹配的应用   

只有当 config-server 和 客户端应用 都有 spring-cloud-bus 时， RefreshRemoteApplicationEvent 才会发送  

默认配置还会 检测 本地git仓库的 文件的修改。这种情况下，webhook不会被使用。 当然，只要你修改了配置文件，一个refresh 被广播

---
#### Spring Cloud Config Client

SpringBoot 应用能直接使用spring config server, 还有一些关于 Environment修改事件的有用的功能。

#### Spring Boot Config Data Import
Spring Boot 2.4 介绍了一种全新的方式 来 导入配置数据，通过 ***spring.config.import*** 属性, 这是现在 绑定到 config server 的默认方式。   

要选择性地连接 config server，设置下面 到 application.properties:   
application.properties
```
spring.config.import=optional:configserver:
```
这个会连接 config server ，在 http://localhost:8888 这个url。   
移除optional: 前缀，会导致 config client 失败，如果它连不到 config server。
修改 config server 的地址： ***spring.cloud.config.uri***, 或增加 url 到 ***spring.config.import*** 中，如 ***spring.cloud.import=optional:configserver:http://myhost:8888***    
import中的地址 优先于 uri 属性

bootstrap 文件 不是必须的， 如果 Spring Boot Config Data 通过 spring.config.import 导入。

---
#### Config First Bootstrap

为了使用 legacy(遗产) bootstrap way 来连接 config server，bootstrap必须通过 property 或 spring-cloud-starter-bootstrap 来enable。   
property 是 spring.cloud.bootstrap.enabled=true,必须设置为 系统属性 或 环境变量。   
一旦bootstrap启用，那些把 spring cloud config client 放在classpath 中的应用 会连接config server，按如下： 当一个config client启动，它绑定到config server (通过 spring.cloud.config.uri 这个bootstrap配置属性)，且通过远程属性源来初始化spring Environment。

这个行为的结果是，所有 需要消费config server的客户端app 都需要一个bootstrap.yml (或一个环境变量)，来指明 spring.cloud.config.uri (默认是http://localhost:8888)

---

#### Discovery First Bootstrap

如果你使用 DiscoveryClient 实现，比如 spring cloud netflix 和 Eureka Service Discovery 或 Spring Cloud Consul, 你可以让 config server 向discovery service注册。   
当然，在默认的"Config First"模式，客户端不能利用注册

如果你希望使用 DiscoveryClient 来定位 Config Server，那么你需要设置 spring.cloud.config.discovery.enabled=true (默认是false)。 这样做的结果是 client应用需要 bootstrap.yml (或一个环境变量)，并且配置合适的 discovery配置。   
比如，要使用 Spring Cloud Netflix, 你需要定义 Eureka server address(比如，在eureka.client.serviceUrl.defaultZone ).   
使用这个选项的代价是 启动时 需要网络往返，来定位服务注册中心。   
好处是，只要Discovery Service 是 一个固定的 点， config server 就可以改变它的坐标。   
默认的 serviceID 是 *configserver*，你可以在客户端修改serviceID,通过设置 spring.cloud.config.discovery.serviceId ( 在server端，一般的方法是 设置 spring.application.name)   

discovery client 实现 都支持 一些 metadata map (比如，对于eureka 我们有 eureka.instance.metadatamap)。   
config server 的一些附加属性 可能需要配置到它的服务注册元信息中 后 才能正确连接。   
如果config server 通过HTTP Basic 来保证安全，你需要配置凭证信息：user，password。  
如果config server 有一个context path，你需要设置 configPath。   
下面是一个用于Eureka客户端的 config server：
```yaml
eureka:
  instance:
    ...
    metadataMap:
      user: osufhalskjrtl
      password: lviuhlszvaorhvlo5847
      configPath: /config
```

---
#### Config Client Fail Fast

spring.cloud.config.fail-fast=true， 启动时，当客户端无法连接到config server时，抛出异常

如果你使用 spring.config.import, 那么忽略 optional: 前缀， 也可以快速失败。

---

#### Config Client Retry

如果你认为，当你的app启动时，config server 可能偶尔不可用，你可以让你的app在失败后重试。   
首先，你需要设置 spring.cloud.config.fail-fast=true,    
然后你需要增加 spring-retry 和 spring-boot-starter-aop 到你的classpath。   
> 默认下，重试 6 次， 第一次重试间隔1000ms， 后面的重试间隔 在上一次的基础上 *1.1   

你可以配置这些属性(及其他的) 在 spring.cloud.config.retry.*   

获得完整的 retry行为的控制， 增加一个 RetryOperationsInterceptor 的Bean，这个Bean的ID 是 configServerRetryInterceptor   
Spring Retry 有一个 RetryInterceptorBuilder 来支持 创建它。

---
#### Locating Remote Configuration Resources

config service 提供资源 从 /{application}/{profile}/{label}, 这些默认的绑定是 客户端app中的：   
name = ${spring.application.name}
profile = ${spring.profiles.active}  (实际上是 Environment.getActiveProfiles())
label = "master"

设置${spring.application.name} 时，不要以 application- 为前缀。 否则无法解析到正确的 property source   

可以覆盖上面的全部，通过设置  spring.cloud.config.* (*是 name，profile，label)。   
label 在 回滚到前一个配置版本 是有用的。在默认的config server实现中，这个可以是 git label, branch name, 或 commit id.   
label 可以是 逗号分隔的list。 这种情况下， 会按顺序尝试，指导有一个成功。这种行为是有用的 在feature branch，比如，你可以希望 align the config label(对齐配置标签) with 你的branch，但是使得它可选，此时，使用 spring.cloud.config.label=myfeature,develop   

---
#### Specifying Multiple Urls for the Config Server
为了高可用，你有多个config server实例。   
你可以 指明多个URLs (spring.cloud.config.uri 的逗号分隔的list) 或 让你的所有实例 注册到 服务注册中心 比如Eureka (如果使用了Discovery-First 启动模式)   
记住，这样做，只有在 config server 不在运行 或连接超时 时才能保证 高可用。 如果config server返回500,或者config client收到401, config client 不会尝试去从其他URL fetch属性。因为这些错误是用户的问题，而不是可用性的问题。   

如果你使用HTTP basic security 在你的config server。目前可以支持每个config server 身份认证，只有当你 嵌入认证 到每个URL (在spring.cloud.config.uri 中配置的) 中.   
如果你使用其他安全机制，目前不支持 每个 config server 身份认证和授权。   
。。就是 用户和密码 作为参数放到url中，这样就每个 config server 一个帐号密码， 不然的话，只能用通用的 帐号密码   
。。 不知道 嵌入用户名密码 的时候 配置的通用用户名密码 会不会生效，是依然追加在uri上？还是会过滤下？
。。不，写法是 https://user:secret@myconfig.mycompany.com ，并不是 作为 参数添加的。。   
。。不能混用，混用的时候属性优先。。url中的无效   

---
#### Configuring Timeouts
如果你想要配置 超时：   
spring.cloud.config.request-read-timeout, 读超时   
spring.cloud.config.request-connect-timeout, 连接超时   

---
#### Security
如果你使用HTTP Basic security 在服务器，客户端需要知道密码( 如果用户名不是默认的，那么还需要知道用户名)。你可以指明 用户名和密码 通过 config server URI 或 通过 配置的用户名和密码属性：   
```yaml
spring:
  cloud:
    config:
     uri: https://user:secret@myconfig.mycompany.com
```
下面和上面一样。
```yaml
spring:
  cloud:
    config:
     uri: https://myconfig.mycompany.com
     username: user
     password: secret
```

spring.cloud.config.password/username 会覆盖 URI中的信息

如果你发布你的应用在Cloud Foundry, 提供密码的最好方法是通过service credentials (在URI中，它不必是一个配置文件)   
下面是本地运行，并适用于Cloud Foundry上，名为configserver 的用户提供的服务：
```
spring:
  cloud:
    config:
     uri: ${vcap.services.configserver.credentials.uri:http://user:password@localhost:8888}
```

如果config server 要求 客户端 TLS证书，你可以配置 客户端TLS 证书和trust store，通过属性：
```yaml
spring:
  cloud:
    config:
      uri: https://myconfig.myconfig.com
      tls:
        enabled: true
        key-store: <path-of-key-store>
        key-store-type: PKCS12
        key-store-password: <key-store-password>
        key-password: <key-password>
        trust-store: <path-of-trust-store>
        trust-store-type: PKCS12
        trust-store-password: <trust-store-password>
```

spring.cloud.config.tls.enabled 需要设置为true，来激活config client 的TLS。   
如果省略xx.trust-store, 那么JVM 默认的trust store 被使用。
xx.key-store-type 和 xx.trust-store-type 的默认值是 PKCS12。   
当 password 被省略时， 假设密码为空。

如果你使用其他方式的security，你需要提供一个 RestTemplate 到 ConfigServicePropertySourceLocator。

---
#### Health Indicator

Config Client 支持 SpringBootHealthIndicator，它尝试从config server 加载配置。   
health.config.enabled=false 来禁止 health indicator    
响应会被cache for 性能，默认cache时间5分钟。 health.config.time-to-live 修改它，单位毫秒   

---
#### Providing A Custom RestTemplate
有时，你需要自定义请求来从客户端访问config server。   
通常，这么做会涉及：传递 特定的 Authorization header 来进行身份验证。

提供一个自定义的RestTemplate:
1 创建新配置的bean 通过 PropertySourceLocator 的实现：
```java
@Configuration
public class CustomConfigServiceBootstrapConfiguration {
    @Bean
    public ConfigServicePropertySourceLocator configServicePropertySourceLocator() {
        ConfigClientProperties clientProperties = configClientProperties();
        ConfigServicePropertySourceLocator configServicePropertySourceLocator =  new ConfigServicePropertySourceLocator(clientProperties);
        configServicePropertySourceLocator.setRestTemplate(customRestTemplate(clientProperties));
        return configServicePropertySourceLocator;
    }
}
```

为了增加Authorization 头，可以使用 spring.cloud.config.headers.*   

2 在resources/META-INF, 创建一个文件名叫 spring.factories. 指明你自定义的配置：
```
org.springframework.cloud.bootstrap.BootstrapConfiguration = com.my.config.client.CustomConfigServiceBootstrapConfiguration
```

---
#### Vault
使用Vault作为你的config server，客户端需要 提供令牌 给server 以检索值。   
在客户端中，能通过bootstrap.yml 中的 spring.cloud.config.token 来提供 令牌   
```yaml
spring:
  cloud:
    config:
      token: YourVaultToken
```

---
#### Nested Keys In Vault
Vault 支持 值中嵌套key。   
> echo -n '{"appA": {"secret": "appAsecret"}, "bar": "baz"}' | vault write secret/myapp -   

这个命令写入一个 JSON 到你的Vault。 在Spring中访问值，需要使用 . 
```
@Value("${appA.secret}")
String name = "World";
```
上面将 name变量 设置为 appAsecret










