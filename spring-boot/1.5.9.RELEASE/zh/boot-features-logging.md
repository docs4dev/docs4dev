## 26.记录

Spring Boot使用[Commons Logging](https://commons.apache.org/logging)进行所有内部日志记录，但保留底层日志实现.为[Java Util Logging](https://docs.oracle.com/javase/8/docs/api//java/util/logging/package-summary.html)，[Log4J2](https://logging.apache.org/log4j/2.x/)和[Logback](http://logback.qos.ch/)提供了默认配置.在每种情况下，Logger都预先配置为使用带有可选文件的控制台输出输出也可用.

默认情况下，如果使用“Starters”，则使用Logback进行日志记录.还包括适当的Logback路由，以确保使用Java Util Logging，Commons Logging，Log4J或SLF4J的依赖库都能正常工作.

_0015`有许多可用于Java的日志框架.如果以上列表看起来令人困惑，请不要担心.通常，您不需要更改日志记录依赖项，并且Spring Boot默认值可以正常工作.

## 26.1日志格式

Spring Boot的默认日志输出类似于以下示例：

```java
2014-03-05 10:57:51.112  INFO 45469 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/7.0.52
2014-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2014-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1358 ms
2014-03-05 10:57:51.698  INFO 45469 --- [ost-startStop-1] o.s.b.c.e.ServletRegistrationBean        : Mapping servlet: 'dispatcherServlet' to [/]
2014-03-05 10:57:51.702  INFO 45469 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
```

输出以下项目：

- Date和Time：毫秒精度，易于排序.

- Log级别： `ERROR` ， `WARN` ， `INFO` ， `DEBUG` 或 `TRACE` .

- Process ID.

- A  `---` 分隔符，用于区分实际日志消息的开始.

- Thread名称：括在方括号中（可能会截断控制台输出）.

- Logger name：这通常是源类名（通常缩写）.

- 日志消息.

> Logback没有 `FATAL` 级别.它映射到 `ERROR` .

## 26.2控制台输出

默认日志配置会在写入时将消息回显到控制台.默认情况下，会记录 `ERROR` 级别， `WARN` 级别和 `INFO` 级别的消息.您还可以通过使用 `--debug` 标志启动应用程序来启用“调试”模式.

```java
$ java -jar myapp.jar --debug
```

> 您还可以在 `application.properties` 中指定 `debug=true` .

启用调试模式后，将选择核心Logger（嵌入式容器，Hibernate和Spring Boot）以输出更多信息.启用调试模式不会将应用程序配置为记录 `DEBUG` 级别的所有消息.

或者，您可以通过使用 `--trace` 标志（或 `application.properties` 中的 `trace=true` ）启动应用程序来启用“跟踪”模式.这样做可以为选择的核心Logger（嵌入式容器，Hibernate模式生成和整个Spring组合）启用跟踪日志记录.

### 26.2.1彩色编码输出

如果您的终端支持ANSI，则使用颜色输出来提高可读性.您可以将 `spring.output.ansi.enabled` 设置为[supported value](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/api/org/springframework/boot/ansi/AnsiOutput.Enabled.html)以覆盖自动检测.

使用 `%clr` 转换字配置颜色编码.在最简单的形式中，转换器根据日志级别为输出着色，如以下示例所示：

```java
%clr(%5p)
```

下表描述了日志级别到颜色的映射：

|等级|色彩|
| ---- | ---- |
|  `FATAL`  |红色|
|  `ERROR`  |红色|
|  `WARN`  |黄色|
|  `INFO`  |绿色|
|  `DEBUG`  |绿色|
|  `TRACE`  |绿色|

或者，您可以通过将其作为转换选项指定应使用的颜色或样式.例如，要使文本变为黄色，请使用以下设置：

```java
%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){yellow}
```

支持以下颜色和样式：

-  `blue` 

-  `cyan` 

-  `faint` 

-  `green` 

-  `magenta` 

-  `red` 

-  `yellow` 

## 26.3文件输出

默认情况下，Spring Boot仅记录到控制台，不会写入日志文件.如果除了控制台输出之外还要编写日志文件，则需要设置 `logging.file` 或 `logging.path` 属性（例如，在 `application.properties` 中）.

下表显示了 `logging.*` 属性如何一起使用：

**Table 26.1. Logging properties** 

|  `logging.file`  |  `logging.path`  |示例|说明|
| ---- | ---- | ---- | ---- |
|（无）|（无）| |仅控制台记录. |
|特定文件|（无）|  `my.log`  |写入指定的日志文件.名称可以是精确位置或相对于当前目录. |
|（无）|特定目录|  `/var/log`  |将 `spring.log` 写入指定目录.名称可以是精确位置或相对于当前目录. |

日志文件在达到10 MB时会轮换，与控制台输出一样，默认情况下会记录 `ERROR` 级别， `WARN` 级别和 `INFO` 级别的消息.可以使用 `logging.file.max-size` 属性更改大小限制.除非已设置 `logging.file.max-history` 属性，否则以前轮换的文件将无限期归档.

> 日志系统在应用程序生命周期的早期初始化.因此，在通过 `@PropertySource` 注释加载的属性文件中找不到日志记录属性.

> Logging属性独立于实际的日志记录基础结构.因此，spring Boot不管理特定的配置键（例如 `logback.configurationFile`  for Logback）.

## 26.4日志级别

所有受支持的日志记录系统都可以使用 `logging.level.<logger-name>=<level>` 在Spring  `Environment` 中设置Logger级别（例如，在 `application.properties` 中），其中 `level` 是TRACE，DEBUG，INFO，WARN，ERROR，FATAL或OFF之一.可以使用 `logging.level.root` 配置 `root` Logger.

以下示例显示 `application.properties` 中的潜在日志记录设置：

```java
logging.level.root=WARN
logging.level.org.springframework.web=DEBUG
logging.level.org.hibernate=ERROR
```

## 26.5日志组

能够将相关Logger组合在一起以便可以同时配置它们通常很有用.例如，您通常可以更改所有Tomcat相关Logger的日志记录级别，但您无法轻松记住顶级软件包.

为了解决这个问题，Spring Boot允许您在Spring  `Environment` 中定义日志记录组.例如，以下是如何通过将“Tomcat”组添加到您的“tomcat”组来定义它 `application.properties` ：

```java
logging.group.tomcat=org.apache.catalina, org.apache.coyote, org.apache.tomcat
```

定义后，您可以使用一行更改组中所有Logger的级别：

```java
logging.level.tomcat=TRACE
```

Spring Boot包含以下预定义的日志记录组，可以直接使用：

|名称|记录仪|
| ---- | ---- |
| web |  `org.springframework.core.codec` ， `org.springframework.http` ， `org.springframework.web`  |
| sql |  `org.springframework.jdbc.core` ， `org.hibernate.SQL`  |

## 26.6自定义日志配置

可以通过在类路径中包含适当的库来激活各种日志记录系统，并且可以通过在类路径的根目录中或在以下Spring  `Environment` 属性指定的位置提供合适的配置文件来进一步自定义： `logging.config` .

您可以使用 `org.springframework.boot.logging.LoggingSystem` 系统属性强制Spring Boot使用特定的日志记录系统.该值应该是 `LoggingSystem` 实现的完全限定类名.您还可以使用 `none` 值完全禁用Spring Boot的日志记录配置.

> Since日志记录已初始化 **before**   `ApplicationContext` 已创建，因此无法在Spring  `@Configuration` 文件中控制 `@PropertySources` 的日志记录.更改日志记录系统或完全禁用它的唯一方法是通过系统属性.

根据您的日志记录系统，将加载以下文件：

|记录系统|定制|
| ---- | ---- |
| Logback |  `logback-spring.xml` ， `logback-spring.groovy` ， `logback.xml` 或 `logback.groovy`  |
| Log4j2 |  `log4j2-spring.xml` 或 `log4j2.xml`  |
| JDK（Java Util Logging）|  `logging.properties`  |

> 如果可能，我们建议您使用 `-spring` 变量进行日志记录配置（例如， `logback-spring.xml` 而不是 `logback.xml` ）.如果使用标准配置位置，Spring无法完全控制日志初始化.

> 已知Java Developer Logging的类加载问题，从“可执行jar”运行时会导致问题.如果可能的话，我们建议您在从“可执行jar”运行时避免使用它.

为了帮助进行自定义，一些其他属性从Spring  `Environment` 传递到System属性，如下表所述：

| Spring Environment |系统属性|评论|
| ---- | ---- | ---- |
|  `logging.exception-conversion-word`  |  `LOG_EXCEPTION_CONVERSION_WORD`  |记录异常时使用的转换字. |
|  `logging.file`  |  `LOG_FILE`  |如果已定义，则在默认日志配置中使用. |
|  `logging.file.max-size`  |  `LOG_FILE_MAX_SIZE`  |最大日志文件大小（如果启用了LOG_FILE）. （仅支持默认的Logback设置.）|
|  `logging.file.max-history`  |  `LOG_FILE_MAX_HISTORY`  |要保留的最大归档日志文件数（如果启用了LOG_FILE）. （仅支持默认的Logback设置.）|
|  `logging.path`  |  `LOG_PATH`  |如果已定义，则在默认日志配置中使用. |
|  `logging.pattern.console`  |  `CONSOLE_LOG_PATTERN`  |在控制台上使用的日志模式（stdout）. （仅支持默认的Logback设置.）|
|  `logging.pattern.dateformat`  |  `LOG_DATEFORMAT_PATTERN`  |日志日期格式的Appender模式. （仅支持默认的Logback设置.）|
|  `logging.pattern.file`  |  `FILE_LOG_PATTERN`  |要在文件中使用的日志模式（如果启用了 `LOG_FILE` ）. （仅支持默认的Logback设置.）|
|  `logging.pattern.level`  |  `LOG_LEVEL_PATTERN`  |呈现日志级别时使用的格式（默认为 `%5p` ）. （仅支持默认的Logback设置.）|
|  `PID`  |  `PID`  |当前进程ID（如果可能，在尚未定义为OS环境变量时发现）. |

所有受支持的日志记录系统在解析其配置文件时都可以参考系统属性.有关示例，请参阅 `spring-boot.jar` 中的默认配置：

- [Logback](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/logback/defaults.xml)

- [Log4j 2](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/log4j2/log4j2.xml)

- [Java Util logging](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/java/logging-file.properties)

> 如果要在日志记录属性中使用占位符，则应使用[Spring Boot’s syntax](boot-features-external-config.html#boot-features-external-config-placeholders-in-properties)而不是底层框架的语法.值得注意的是，如果使用Logback，则应使用 `:` 作为属性名称与其默认值之间的分隔符，而不要使用 `:-` .

> 您可以通过仅覆盖 `LOG_LEVEL_PATTERN` （或带有Logback的 `logging.pattern.level` ）将MDC和其他临时内容添加到日志行.例如，如果使用 `logging.pattern.level=user:%X{user} %5p` ，则默认日志格式包含"user"的MDC条目（如果存在），如以下示例所示.

## 26.7 Logback Extensions

Spring Boot包含许多Logback扩展，可以帮助进行高级配置.您可以在 `logback-spring.xml` 配置文件中使用这些扩展名.

> B因为标准 `logback.xml` 配置文件加载太早，您无法在其中使用扩展名.您需要使用 `logback-spring.xml` 或定义 `logging.config` 属性.

> 扩展名不能与Logback的[configuration scanning](http://logback.qos.ch/manual/configuration.html#autoScan)一起使用.如果尝试这样做，则更改配置文件会导致类似于以下记录之一的错误：

```java
ERROR in [emailprotected]:71 - no applicable action for [springProperty], current ElementPath is [[configuration][springProperty]]
ERROR in [emailprotected]:71 - no applicable action for [springProfile], current ElementPath is [[configuration][springProfile]]
```

### 26.7.1特定于配置文件的配置

`<springProfile>` 标记允许您根据活动的Spring配置文件选择性地包含或排除配置部分.在 `<configuration>` 元素内的任何位置都支持配置文件节.使用 `name` 属性指定哪个配置文件接受配置.  `<springProfile>` 标记可以包含简单的配置文件名称（例如 `staging` ）或配置文件表达式.概要表达式允许表达更复杂的概要逻辑，例如 `production & (eu-central | eu-west)` .有关详细信息，请查看[reference guide](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/core.html#beans-definition-profiles-java).以下清单显示了三个示例配置文件：

```xml
<springProfile name="staging">
	<!-- configuration to be enabled when the "staging" profile is active -->
</springProfile>

<springProfile name="dev | staging">
	<!-- configuration to be enabled when the "dev" or "staging" profiles are active -->
</springProfile>

<springProfile name="!production">
	<!-- configuration to be enabled when the "production" profile is not active -->
</springProfile>
```

### 26.7.2环境属性

`<springProperty>` 标记允许您公开Spring  `Environment` 中的属性，以便在Logback中使用.如果要在Logback配置中访问 `application.properties` 文件中的值，则此操作非常有用.标签的工作方式与Logback的标准 `<property>` 标签类似.但是，不是指定直接 `value` ，而是指定属性的 `source` （来自 `Environment` ）.如果需要将属性存储在 `local` 范围以外的其他位置，则可以使用 `scope` 属性.如果需要回退值（如果未在 `Environment` 中设置该属性），则可以使用 `defaultValue` 属性.以下示例显示如何公开在Logback中使用的属性：

```xml
<springProperty scope="context" name="fluentHost" source="myapp.fluentd.host"
		defaultValue="localhost"/>
<appender name="FLUENT" class="ch.qos.logback.more.appenders.DataFluentAppender">
	<remoteHost>${fluentHost}</remoteHost>
	...
</appender>
```

> 必须在烤肉串案例中指定 `source` （例如 `my.property-name` ）.但是，可以使用宽松规则将属性添加到 `Environment` .

