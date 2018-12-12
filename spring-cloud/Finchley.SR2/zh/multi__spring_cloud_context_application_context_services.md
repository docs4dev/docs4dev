## 2. Spring Cloud Context：应用程序上下文服务

Spring Boot有一个关于如何使用Spring构建应用程序的观点.例如，它具有常见配置文件的常规位置，并具有用于常见管理和监视任务的endpoints. Spring Cloud构建于此之上，并添加了一些功能，可能是系统中的所有组件都可能使用或偶尔需要的功能.

## 2.1 Bootstrap应用程序上下文

Spring Cloud应用程序通过创建“引导程序”上下文来运行，该上下文是主应用程序的父上下文.它负责从外部源加载配置属性以及解密本地外部配置文件中的属性.这两个上下文共享一个 `Environment` ，它是任何Spring应用程序的外部属性的来源.默认情况下，引导属性（不是 `bootstrap.properties` ，而是在引导阶段加载的属性）以高优先级添加，因此它们不能被本地配置覆盖.

引导上下文使用不同的约定来定位外部配置而不是主应用程序上下文.而不是 `application.yml` （或 `.properties` ），您可以使用 `bootstrap.yml` ，保持bootstrap和主上下文的外部配置很好地分开.以下清单显示了一个示例：

**bootstrap.yml.** 

```java
spring:
application:
name: foo
cloud:
config:
uri: ${SPRING_CONFIG_URI:http://localhost:8888}
```

如果您的应用程序需要来自服务器的任何特定于应用程序的配置，则最好设置 `spring.application.name` （在 `bootstrap.yml` 或 `application.yml` 中）.

您可以通过设置 `spring.cloud.bootstrap.enabled=false` （例如，在系统属性中）完全禁用引导过程.

## 2.2应用程序上下文层次结构

如果从 `SpringApplication` 或 `SpringApplicationBuilder` 构建应用程序上下文，则会将Bootstrap上下文添加为该上下文的父级. Spring的一个特性是子上下文从其父级继承属性源和配置文件，因此与构建没有Spring Cloud Config的相同上下文相比，“主”应用程序上下文包含其他属性源.其他property来源是：

- “bootstrap”：如果在Bootstrap上下文中找到任何 `PropertySourceLocators` ，并且它们具有非空属性，则会出现具有高优先级的可选 `CompositePropertySource` .一个例子是Spring Cloud Config Server的属性.看到“[Section 2.6, “Customizing the Bootstrap Property Sources”](multi__spring_cloud_context_application_context_services.html#customizing-bootstrap-property-sources)”有关如何自定义此属性源内容的说明.

- “applicationConfig：[classpath：bootstrap.yml]”（以及相关文件，如果Spring配置文件处于活动状态）：如果您有 `bootstrap.yml` （或 `.properties` ），则这些属性用于配置Bootstrap上下文.然后，在设置其父级时，它们将添加到子上下文中.它们的优先级低于 `application.yml` （或 `.properties` ）以及添加到子级的任何其他属性源，作为创建Spring Boot应用程序的正常部分.有关如何自定义这些属性源的内容的说明，请参阅“[Section 2.3, “Changing the Location of Bootstrap Properties”](multi__spring_cloud_context_application_context_services.html#customizing-bootstrap-properties)”.

由于属性源的排序规则，“bootstrap”条目优先.但请注意，这些数据不包含来自 `bootstrap.yml` 的任何数据，它具有非常低的优先级，但可用于设置默认值.

您可以通过设置您创建的任何 `ApplicationContext` 的父上下文来扩展上下文层次结构 - 例如，通过使用自己的接口或 `SpringApplicationBuilder` 便捷方法（ `parent()` ， `child()` 和 `sibling()` ）.引导上下文是您自己创建的最高级祖先的父级.层次结构中的每个上下文都有自己的“引导程序”（可能是空的）属性源，以避免无意中将父级的值提升到其后代.如果存在Config Server，则层次结构中的每个上下文（原则上）也可以具有不同的 `spring.application.name` ，因此具有不同的远程属性源.普通的Spring应用程序上下文行为规则适用于属性解析：来自子上下文的属性按名称和属性源名称覆盖父级中的属性. （如果子项具有与父项具有相同名称的属性源，则父项中的值不包含在子项中）.

请注意， `SpringApplicationBuilder` 允许您在整个层次结构中共享 `Environment` ，但这不是默认值.因此，兄弟情境尤其不需要具有相同的Profiles或property来源，即使他们可能与父母共享共同的Value观.

## 2.3更改Bootstrap属性的位置

可以通过设置 `spring.cloud.bootstrap.name` （默认值： `bootstrap` ）或 `spring.cloud.bootstrap.location` （默认值：空）来指定 `bootstrap.yml` （或 `.properties` ）位置 - 例如，在系统属性中.这些属性的行为类似于具有相同名称的 `spring.config.*` 变体.实际上，它们用于通过在 `Environment` 中设置这些属性来设置引导程序 `ApplicationContext` .如果存在活动配置文件（来自 `spring.profiles.active` 或正在构建的上下文中的 `Environment`  API），则该配置文件中的属性也会加载，与常规Spring Boot应用程序中的属性相同 - 例如，对于 `development` 配置文件，来自 `bootstrap-development.properties`  .

## 2.4覆盖远程属性的值

引导上下文添加到应用程序的属性源通常是“远程”（例如，来自Spring Cloud Config Server）.默认情况下，它们无法在本地重写.如果要让应用程序使用自己的系统属性或配置文件覆盖远程属性，则远程属性源必须通过设置 `spring.cloud.config.allowOverride=true` 来授予它权限（它不能在本地设置它）.设置该标志后，两个更细粒度的设置将控制远程属性相对于系统属性和应用程序本地配置的位置：

-  `spring.cloud.config.overrideNone=true` ：从任何本地属性源覆盖.

-  `spring.cloud.config.overrideSystemProperties=false` ：只有系统属性，命令行参数和环境变量（但不包括本地配置文件）才应覆盖远程设置.

## 2.5自定义Bootstrap配置

通过在名为 `org.springframework.cloud.bootstrap.BootstrapConfiguration` 的键下向 `/META-INF/spring.factories` 添加条目，可以将引导上下文设置为执行任何操作.它包含一个以逗号分隔的Spring  `@Configuration` 类列表，用于创建上下文.您可以在此处创建您希望可用于主应用程序上下文以进行自动装配的任何Bean.  `@Beans` 类型 `ApplicationContextInitializer` 的特殊Contract.如果要控制启动顺序，可以使用 `@Order` 注释标记类（默认顺序为 `last` ）.

> 当添加自定义 `BootstrapConfiguration` 时，请注意您添加的类错误地不是 `@ComponentScanned` 到您的“主”应用程序上下文中，可能不需要它们.对引导配置类使用单独的包名称，并确保 `@ComponentScan` 或 `@SpringBootApplication` 带注释的配置类尚未涵盖该名称.

引导过程通过将初始化程序注入主 `SpringApplication` 实例（这是正常的Spring Boot启动顺序，无论是作为独立应用程序运行还是部署在应用程序服务器中）来结束.首先，根据 `spring.factories` 中的类创建引导上下文.然后，所有 `@Beans` 类型的 `ApplicationContextInitializer` 都会在启动之前添加到主 `SpringApplication` 中.

## 2.6自定义Bootstrap属性源

引导过程添加的外部配置的默认属性源是Spring Cloud Config Server，但您可以添加其他内容通过将 `PropertySourceLocator` 类型的bean添加到引导上下文（通过 `spring.factories` ）来获取源代码.例如，您可以从其他服务器或数据库插入其他属性.

例如，请考虑以下自定义定位器：

```java
@Configuration
public class CustomPropertySourceLocator implements PropertySourceLocator {

@Override
public PropertySource<?> locate(Environment environment) {
return new MapPropertySource("customProperty",
Collections.<String, Object>singletonMap("property.from.sample.custom.source", "worked as intended"));
}

}
```

传入的 `Environment` 是即将创建的 `ApplicationContext` 的一个 - 换句话说，我们为其提供其他属性源的那个.它已经具有正常的Spring Boot提供的属性源，因此您可以使用它们来定位特定于此 `Environment` 的属性源（例如，通过在 `spring.application.name` 上键入它，就像在默认的Spring Cloud Config Server属性源定位器中所做的那样） .

如果您在其中创建一个包含此类的jar，然后添加包含以下内容的 `META-INF/spring.factories` ，则 `customProperty`   `PropertySource` 将出现在其类路径中包含该jar的任何应用程序中：

```java
org.springframework.cloud.bootstrap.BootstrapConfiguration=sample.custom.CustomPropertySourceLocator
```

## 2.7记录配置

如果要使用Spring Boot配置日志设置，则应将此配置放在`bootstrap.[yml |如果您希望它适用于所有事件.

> 要使Spring Cloud正确初始化日志记录配置，您无法使用自定义前缀.例如，初始化日志记录系统时，Spring Cloud将无法识别使用 `custom.loggin.logpath` .

## 2.8环境变化

应用程序侦听 `EnvironmentChangeEvent` 并以几种标准方式对更改做出反应（用户可以正常方式添加 `ApplicationListeners` 作为 `@Beans` ）.当观察到 `EnvironmentChangeEvent` 时，它有一个已更改的键值列表，应用程序使用它们：

- 在上下文中重新绑定任何 `@ConfigurationProperties`  bean

- Set  `logging.level.*` 中任何属性的Logger级别

请注意，默认情况下，Config Client不会轮询 `Environment` 中的更改.通常，我们不建议使用这种方法来检测更改（尽管您可以使用 `@Scheduled` 注释进行设置）.如果您有一个扩展的客户端应用程序，最好将 `EnvironmentChangeEvent` 广播到所有实例，而不是让它们轮询更改（例如，使用[Spring Cloud Bus](https://github.com/spring-cloud/spring-cloud-bus)）.

`EnvironmentChangeEvent` 涵盖了一大类刷新用例，只要您可以实际更改 `Environment` 并发布事件即可.请注意，这些API是公共的，是Spring的核心部分.您可以通过访问 `/configprops` endpoints（一个正常的Spring Boot Actuator功能）来验证更改是否绑定到 `@ConfigurationProperties`  beans.例如， `DataSource` 可以在运行时更改 `maxPoolSize` （Spring Boot创建的默认 `DataSource` 是 `@ConfigurationProperties`  bean）并动态增加容量.重新绑定 `@ConfigurationProperties` 不包括另一大类用例，您需要更多控制刷新以及需要在整个 `ApplicationContext` 上进行原子更改的位置.为了解决这些问题，我们有 `@RefreshScope` .

## 2.9刷新范围

当配置发生变化时，标记为 `@RefreshScope` 的Spring  `@Bean` 会得到特殊处理.此功能解决了有状态bean的问题，只有在初始化时才会注入其配置.例如，如果 `DataSource` 在通过 `Environment` 更改数据库URL时具有打开的连接，您可能希望这些连接的持有者能够完成他们正在做的事情.然后，下次从池中借用某个连接时，它会获得一个带有新URL的连接.

有时，甚至可能必须在一些只能初始化一次的bean上应用 `@RefreshScope` 注释.如果bean是"immutable"，则必须使用 `@RefreshScope` 注释bean或在属性键 `spring.cloud.refresh.extra-refreshable` 下指定classname.

刷新范围bean是在使用它们时初始化的惰性代理（即，在调用方法时），并且范围充当初始化值的缓存.要强制bean在下一个方法调用上重新初始化，必须使其缓存条目无效.

`RefreshScope` 是上下文中的一个bean，它有一个公共 `refreshAll()` 方法，通过清除目标缓存来刷新作用域中的所有bean.  `/refresh` endpoints公开此功能（通过HTTP或JMX）.要按名称刷新单个bean，还有一个 `refresh(String)` 方法.

要公开 `/refresh` endpoints，您需要将以下配置添加到您的应用程序：

```java
management:
endpoints:
web:
exposure:
include: refresh
```

>  `@RefreshScope` （在技术上）在 `@Configuration` 类上工作，但它可能会导致令人惊讶的行为.例如，它并不意味着该类中定义的所有 `@Beans` 本身都在 `@RefreshScope` 中.具体来说，依赖于那些bean的任何东西都不能依赖于在启动刷新时更新它们，除非它本身在 `@RefreshScope` 中.在这种情况下，它会在刷新时重建，并重新注入其依赖项.此时，它们将从刷新的 `@Configuration` 重新初始化.

## 2.10加密和解密

Spring Cloud有一个 `Environment` 预处理器，用于在本地解密属性值.它遵循与Config Server相同的规则，并通过 `encrypt.*` 具有相同的外部配置.因此，您可以使用加密值的形式 `{cipher}*` 并且，只要有有效密钥，它们就会在主应用程序上下文获得 `Environment` 设置之前被解密.要在应用程序中使用加密功能，您需要在类路径中包含Spring Security RSA（Maven坐标："org.springframework.security:spring-security-rsa"），并且还需要JVM中的全功能JCE扩展.

如果由于“非法密钥大小”而导致异常并且您使用Sun的JDK，则需要安装Java Cryptography Extension（JCE）Unlimited Strength Jurisdiction Policy Files.有关更多信息，请参阅以下链接：

- [Java 6 JCE](http://www.oracle.com/technetwork/java/javase/downloads/jce-6-download-429243.html)

- [Java 7 JCE](http://www.oracle.com/technetwork/java/javase/downloads/jce-7-download-432124.html)

- [Java 8 JCE](http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html)

无论您使用哪种版本的JRE / JDK x64 / x86，都要将文件解压缩到JDK / jre / lib / security文件夹中.

## 2.11endpoints

对于Spring Boot Actuator应用程序，可以使用一些其他管理endpoints.您可以使用：

-  `POST` 至 `/actuator/env` 更新 `Environment` 并重新绑定 `@ConfigurationProperties` 和日志级别.

-  `/actuator/refresh` 重新加载启动带上下文并刷新 `@RefreshScope`  bean.

-  `/actuator/restart` 关闭 `ApplicationContext` 并重新启动它（默认情况下禁用）.

-  `/actuator/pause` 和 `/actuator/resume` 用于调用 `Lifecycle` 方法（ `ApplicationContext` 上的 `stop()` 和 `start()` ）.

> 如果禁用 `/actuator/restart` endpoints，则 `/actuator/pause` 和 `/actuator/resume` endpoints也将被禁用，因为它们只是 `/actuator/restart` 的特例.

