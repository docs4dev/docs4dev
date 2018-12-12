## 23. SpringApplication

`SpringApplication` 类提供了一种方便的方法来引导从 `main()` 方法启动的Spring应用程序.在许多情况下，您可以委派静态 `SpringApplication.run` 方法，如以下示例所示：

```java
public static void main(String[] args) {
	SpringApplication.run(MySpringConfiguration.class, args);
}
```

当您的应用程序启动时，您应该看到类似于以下输出的内容：

```java
.   ____          _            __ _ _
/\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
\\/  ___)| |_)| | | | | || (_| |  ) ) ) )
'  |____| .__|_| |_|_| |_\__, | / / / /
=========|_|==============|___/=/_/_/_/
:: Spring Boot ::   v2.1.0.RELEASE

2013-07-31 00:08:16.117  INFO 56603 --- [           main] o.s.b.s.app.SampleApplication            : Starting SampleApplication v0.1.0 on mycomputer with PID 56603 (/apps/myapp.jar started by pwebb)
2013-07-31 00:08:16.166  INFO 56603 --- [           main] ationConfigServletWebServerApplicationContext : Refreshing org.springframework.boot.web.ser[emailprotected]6e5a8246: startup date [Wed Jul 31 00:08:16 PDT 2013]; root of context hierarchy
2014-03-04 13:09:54.912  INFO 41370 --- [           main] .t.TomcatServletWebServerFactory : Server initialized with port: 8080
2014-03-04 13:09:56.501  INFO 41370 --- [           main] o.s.b.s.app.SampleApplication            : Started SampleApplication in 2.992 seconds (JVM running for 3.658)
```

默认情况下，会显示 `INFO` 日志消息，包括一些相关的启动详细信息，例如启动应用程序的用户.如果您需要 `INFO` 以外的日志级别，则可以设置它，如[Section 26.4, “Log Levels”](boot-features-logging.html#boot-features-custom-log-levels)中所述，

## 23.1启动失败

如果您的应用程序无法启动，则已注册 `FailureAnalyzers` 有机会提供专用错误消息和具体操作来解决问题.例如，如果您启动网络在端口 `8080` 上的应用程序以及该端口已在使用中，您应该看到类似于以下消息的内容：

```java
***************************
APPLICATION FAILED TO START
***************************

Description:

Embedded servlet container failed to start. Port 8080 was already in use.

Action:

Identify and stop the process that's listening on port 8080 or configure this application to listen on another port.
```

> Spring Boot提供了许多 `FailureAnalyzer` 实现，你可以[add your own](howto-spring-boot-application.html#howto-failure-analyzer).

如果没有故障分析器能够处理异常，您仍然可以显示完整的条件报告，以便更好地了解出现了什么问题.为此，您需要[enable the debug property](boot-features-external-config.html)或[enable DEBUG logging](boot-features-logging.html#boot-features-custom-log-levels)表示 `org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener` .

例如，如果使用 `java -jar` 运行应用程序，则可以按如下方式启用 `debug` 属性：

```java
$ java -jar myproject-0.0.1-SNAPSHOT.jar --debug
```

## 23.2自定义Banner

可以通过将 `banner.txt` 文件添加到类路径或将 `spring.banner.location` 属性设置为此类文件的位置来更改启动时打印的Banner.如果文件的编码不是UTF-8，则可以设置 `spring.banner.charset` .除了文本文件，您还可以将 `banner.gif` ， `banner.jpg` 或 `banner.png` 图像文件添加到类路径或设置 `spring.banner.image.location` 属性.图像将转换为ASCII艺术表示，并打印在任何文本Banner上方.

在 `banner.txt` 文件中，您可以使用以下任何占位符：

**Table 23.1. Banner variables** 

|变量|说明|
| ---- | ---- |
|  `${application.version}`  |应用程序的版本号，如 `MANIFEST.MF` 中所声明.例如， `Implementation-Version: 1.0` 打印为 `1.0` . |
|  `${application.formatted-version}`  |应用程序的版本号，在 `MANIFEST.MF` 中声明并格式化以显示（用括号括起来并以 `v` 为前缀）.例如 `(v1.0)` . |
|  `${spring-boot.version}`  |您正在使用的Spring Boot版本.例如 `2.1.0.RELEASE` . |
|  `${spring-boot.formatted-version}`  |您正在使用的Spring Boot版本，格式化显示（用括号括起来并以 `v` 为前缀）.例如 `(v2.1.0.RELEASE)` . |
|  `${Ansi.NAME}` （或 `${AnsiColor.NAME}` ， `${AnsiBackground.NAME}` ， `${AnsiStyle.NAME}` ）|其中 `NAME` 是ANSI转义码的名称.有关详细信息，请参阅[AnsiPropertySource](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/ansi/AnsiPropertySource.java). |
|  `${application.title}`  |应用程序的Headers，在 `MANIFEST.MF` 中声明.例如 `Implementation-Title: MyApp` 打印为 `MyApp` . |

> 如果要以编程方式生成Banner，可以使用 `SpringApplication.setBanner(…)` 方法.使用 `org.springframework.boot.Banner` 接口并实现自己的 `printBanner()` 方法.

您还可以使用 `spring.main.banner-mode` 属性来确定是否必须在 `System.out` （ `console` ）上打印Banner，发送到配置的Logger（ `log` ），或者根本不产生Banner（ `off` ）.

打印的Banner在以下名称下注册为单例bean： `springBootBanner` .

> YAML将 `off` 映射到 `false` ，因此如果要在应用程序中禁用Banner，请务必添加引号，如以下示例所示：

## 23.3自定义SpringApplication

如果 `SpringApplication` 默认值不符合您的口味，您可以改为创建本地实例并对其进行自定义.例如，要关闭Banner，您可以写：

```java
public static void main(String[] args) {
	SpringApplication app = new SpringApplication(MySpringConfiguration.class);
	app.setBannerMode(Banner.Mode.OFF);
	app.run(args);
}
```

> 传递给 `SpringApplication` 的构造函数参数是Spring bean的配置源.在大多数情况下，这些是对 `@Configuration` 类的引用，但它们也可以是对XML配置或应扫描的包的引用.

也可以使用 `application.properties` 文件配置 `SpringApplication` .有关详细信息，请参阅[Chapter 24, Externalized Configuration](boot-features-external-config.html).

有关配置选项的完整列表，请参阅[SpringApplication Javadoc](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/api/org/springframework/boot/SpringApplication.html).

## 23.4 Fluent Builder API

如果需要构建 `ApplicationContext` 层次结构（具有父/子关系的多个上下文）或者如果您更喜欢使用“流畅”构建器API，则可以使用 `SpringApplicationBuilder` .

`SpringApplicationBuilder` 允许您将多个方法调用链接在一起，并包含允许您创建层次结构的 `parent` 和 `child` 方法，如以下示例所示：

```java
new SpringApplicationBuilder()
		.sources(Parent.class)
		.child(Application.class)
		.bannerMode(Banner.Mode.OFF)
		.run(args);
```

> 创建 `ApplicationContext` 层次结构时存在一些限制.例如，Web组件 **must** 包含在子上下文中，并且相同的 `Environment` 用于父上下文和子上下文.有关详细信息，请参阅[SpringApplicationBuilder Javadoc](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/api/org/springframework/boot/builder/SpringApplicationBuilder.html).

## 23.5应用程序事件和监听器

除了通常的Spring Framework事件（例如[ContextRefreshedEvent](https://docs.spring.io/spring/docs/5.1.2.RELEASE/javadoc-api/org/springframework/context/event/ContextRefreshedEvent.html)）之外， `SpringApplication` 还会发送一些其他应用程序事件.

> Some事件实际上是在创建 `ApplicationContext` 之前触发的，因此您无法将_1188_上的侦听器注册为 `@Bean` .您可以使用 `SpringApplication.addListeners(…)` 方法或 `SpringApplicationBuilder.listeners(…)` 方法注册它们.

应用程序运行时，应按以下顺序发送应用程序事件：

ApplicationStartingEvent在运行开始时但在任何处理之前发送，但监听器和初始化程序的注册除外.当要在上下文中使用的环境已知但在创建上下文之前，将发送ApplicationEnvironmentPreparedEvent. ApplicationPreparedEvent在刷新开始之前但在加载bean定义之后发送.在刷新上下文之后但在调用任何应用程序和命令行运行程序之前发送ApplicationStartedEvent.在调用任何应用程序和命令行运行程序之后发送ApplicationReadyEvent.它表示应用程序已准备好为请求提供服务.如果启动时发生异常，则发送ApplicationFailedEvent.

> 您经常不需要使用应用程序事件，但知道它们存在可能很方便.在内部，Spring Boot使用事件处理各种任务.

使用Spring Framework的事件发布机制发送应用程序事件.此机制的一部分确保在子上下文中发布给侦听器的事件也会在任何祖先上下文中发布给侦听器.因此，如果您的应用程序使用 `SpringApplication` 实例的层次结构，则侦听器可能会收到相同类型的应用程序事件的多个实例.

为了允许侦听器区分其上下文的事件和后代上下文的事件，它应该请求注入其应用程序上下文，然后将注入的上下文与事件的上下文进行比较.可以通过实现 `ApplicationContextAware` 来注入上下文，或者，如果侦听器是bean，则使用 `@Autowired` .

## 23.6网络环境

`SpringApplication` 试图代表您创建正确类型的 `ApplicationContext` .用于确定 `WebApplicationType` 的算法非常简单：

- 如果存在Spring MVC，则使用 `AnnotationConfigServletWebServerApplicationContext` 

- 如果Spring MVC不存在且存在Spring WebFlux，则使用 `AnnotationConfigReactiveWebServerApplicationContext` 

- 否则，使用 `AnnotationConfigApplicationContext` 

这意味着如果您在同一个应用程序中使用Spring MVC和Spring WebFlux中的新 `WebClient` ，默认情况下将使用Spring MVC.您可以通过调用 `setWebApplicationType(WebApplicationType)` 轻松覆盖它.

也可以通过调用 `setApplicationContextClass(…)` 来完全控制 `ApplicationContext` 类型.

> 在JUnit测试中使用 `SpringApplication` 时，通常需要调用 `setWebApplicationType(WebApplicationType.NONE)` .

## 23.7访问应用程序参数

如果需要访问传递给 `SpringApplication.run(…)` 的应用程序参数，则可以注入 `org.springframework.boot.ApplicationArguments`  bean.  `ApplicationArguments` 接口提供对原始 `String[]` 参数以及解析的 `option` 和 `non-option` 参数的访问，如以下示例所示：

```java
import org.springframework.boot.*;
import org.springframework.beans.factory.annotation.*;
import org.springframework.stereotype.*;

@Component
public class MyBean {

	@Autowired
	public MyBean(ApplicationArguments args) {
		boolean debug = args.containsOption("debug");
		List<String> files = args.getNonOptionArgs();
		// if run with "--debug logfile.txt" debug=true, files=["logfile.txt"]
	}

}
```

> Spring Boot还在Spring  `Environment` 中注册 `CommandLinePropertySource` .这使您还可以使用 `@Value` 注释注入单个应用程序参数.

## 23.8使用ApplicationRunner或CommandLineRunner

如果在 `SpringApplication` 启动后需要运行某些特定代码，则可以实现 `ApplicationRunner` 或 `CommandLineRunner` 接口.两个接口以相同的方式工作，并提供单个 `run` 方法，该方法在 `SpringApplication.run(…)` 完成之前调用.

`CommandLineRunner` 接口提供对应用程序参数的访问，作为简单的字符串数组，而 `ApplicationRunner` 使用前面讨论的 `ApplicationArguments` 接口.以下示例显示带有 `run` 方法的 `CommandLineRunner` ：

```java
import org.springframework.boot.*;
import org.springframework.stereotype.*;

@Component
public class MyBean implements CommandLineRunner {

	public void run(String... args) {
		// Do something...
	}

}
```

如果定义了必须以特定顺序调用的多个 `CommandLineRunner` 或 `ApplicationRunner`  bean，则可以另外实现 `org.springframework.core.Ordered` 接口或使用 `org.springframework.core.annotation.Order` 批注.

## 23.9申请退出

每个 `SpringApplication` 都会向JVM注册一个关闭钩子，以确保 `ApplicationContext` 在退出时正常关闭.可以使用所有标准的Spring生命周期回调（例如 `DisposableBean` 接口或 `@PreDestroy` 注释）.

此外，如果bean在调用 `SpringApplication.exit()` 时希望返回特定的退出代码，则bean可以实现 `org.springframework.boot.ExitCodeGenerator` 接口.然后可以将此退出代码传递给 `System.exit()` 以将其作为状态代码返回，如以下示例所示：

```java
@SpringBootApplication
public class ExitCodeApplication {

	@Bean
	public ExitCodeGenerator exitCodeGenerator() {
		return () -> 42;
	}

	public static void main(String[] args) {
		System.exit(SpringApplication
				.exit(SpringApplication.run(ExitCodeApplication.class, args)));
	}

}
```

此外， `ExitCodeGenerator` 接口可以通过例外来实现.遇到这样的异常时，Spring Boot返回实现的 `getExitCode()` 方法提供的退出代码.

## 23.10管理员功能

通过指定 `spring.application.admin.enabled` 属性，可以为应用程序启用与管理相关的功能.这暴露了平台 `MBeanServer` 上的[SpringApplicationAdminMXBean](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/admin/SpringApplicationAdminMXBean.java).您可以使用此功能远程管理Spring Boot应用程序.此功能对于任何服务包装器实现也很有用.

> 如果您想知道应用程序正在运行的HTTP端口，请使用 `local.server.port` 键获取该属性.

> 启用此功能时要小心，因为MBean公开了一种关闭应用程序的方法.

