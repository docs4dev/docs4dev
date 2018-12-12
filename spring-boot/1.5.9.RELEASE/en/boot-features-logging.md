## 26. Logging

Spring Boot uses [Commons Logging](https://commons.apache.org/logging) for all internal logging but leaves the underlying log implementation open. Default configurations are provided for [Java Util Logging](https://docs.oracle.com/javase/8/docs/api//java/util/logging/package-summary.html), [Log4J2](https://logging.apache.org/log4j/2.x/), and [Logback](http://logback.qos.ch/). In each case, loggers are pre-configured to use console output with optional file output also available.

By default, if you use the “Starters”, Logback is used for logging. Appropriate Logback routing is also included to ensure that dependent libraries that use Java Util Logging, Commons Logging, Log4J, or SLF4J all work correctly.

> There are a lot of logging frameworks available for Java. Do not worry if the above list seems confusing. Generally, you do not need to change your logging dependencies and the Spring Boot defaults work just fine.

## 26.1 Log Format

The default log output from Spring Boot resembles the following example:

```java
2014-03-05 10:57:51.112  INFO 45469 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/7.0.52
2014-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2014-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1358 ms
2014-03-05 10:57:51.698  INFO 45469 --- [ost-startStop-1] o.s.b.c.e.ServletRegistrationBean        : Mapping servlet: 'dispatcherServlet' to [/]
2014-03-05 10:57:51.702  INFO 45469 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
```

The following items are output:

- Date and Time: Millisecond precision and easily sortable.

- Log Level:  `ERROR` ,  `WARN` ,  `INFO` ,  `DEBUG` , or  `TRACE` .

- Process ID.

- A  `---`  separator to distinguish the start of actual log messages.

- Thread name: Enclosed in square brackets (may be truncated for console output).

- Logger name: This is usually the source class name (often abbreviated).

- The log message.

> Logback does not have a  `FATAL`  level. It is mapped to  `ERROR` .

## 26.2 Console Output

The default log configuration echoes messages to the console as they are written. By default,  `ERROR` -level,  `WARN` -level, and  `INFO` -level messages are logged. You can also enable a “debug” mode by starting your application with a  `--debug`  flag.

```java
$ java -jar myapp.jar --debug
```

> You can also specify  `debug=true`  in your  `application.properties` .

When the debug mode is enabled, a selection of core loggers (embedded container, Hibernate, and Spring Boot) are configured to output more information. Enabling the debug mode does not configure your application to log all messages with  `DEBUG`  level.

Alternatively, you can enable a “trace” mode by starting your application with a  `--trace`  flag (or  `trace=true`  in your  `application.properties` ). Doing so enables trace logging for a selection of core loggers (embedded container, Hibernate schema generation, and the whole Spring portfolio).

### 26.2.1 Color-coded Output

If your terminal supports ANSI, color output is used to aid readability. You can set  `spring.output.ansi.enabled`  to a [supported value](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/api/org/springframework/boot/ansi/AnsiOutput.Enabled.html) to override the auto detection.

Color coding is configured by using the  `%clr`  conversion word. In its simplest form, the converter colors the output according to the log level, as shown in the following example:

```java
%clr(%5p)
```

The following table describes the mapping of log levels to colors:

|Level|Color|
|----|----|
| `FATAL`  |Red |
| `ERROR`  |Red |
| `WARN`  |Yellow |
| `INFO`  |Green |
| `DEBUG`  |Green |
| `TRACE`  |Green |

Alternatively, you can specify the color or style that should be used by providing it as an option to the conversion. For example, to make the text yellow, use the following setting:

```java
%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){yellow}
```

The following colors and styles are supported:

-  `blue` 

-  `cyan` 

-  `faint` 

-  `green` 

-  `magenta` 

-  `red` 

-  `yellow` 

## 26.3 File Output

By default, Spring Boot logs only to the console and does not write log files. If you want to write log files in addition to the console output, you need to set a  `logging.file`  or  `logging.path`  property (for example, in your  `application.properties` ).

The following table shows how the  `logging.*`  properties can be used together:

**Table 26.1. Logging properties** 

| `logging.file` | `logging.path` |Example|Description|
|----|----|----|----|
|(none) |(none) | |Console only logging. |
|Specific file |(none) | `my.log`  |Writes to the specified log file. Names can be an exact location or relative to the current directory. |
|(none) |Specific directory | `/var/log`  |Writes  `spring.log`  to the specified directory. Names can be an exact location or relative to the current directory. |

Log files rotate when they reach 10 MB and, as with console output,  `ERROR` -level,  `WARN` -level, and  `INFO` -level messages are logged by default. Size limits can be changed using the  `logging.file.max-size`  property. Previously rotated files are archived indefinitely unless the  `logging.file.max-history`  property has been set.

> The logging system is initialized early in the application lifecycle. Consequently, logging properties are not found in property files loaded through  `@PropertySource`  annotations.

> Logging properties are independent of the actual logging infrastructure. As a result, specific configuration keys (such as  `logback.configurationFile`  for Logback) are not managed by spring Boot.

## 26.4 Log Levels

All the supported logging systems can have the logger levels set in the Spring  `Environment`  (for example, in  `application.properties` ) by using  `logging.level.<logger-name>=<level>`  where  `level`  is one of TRACE, DEBUG, INFO, WARN, ERROR, FATAL, or OFF. The  `root`  logger can be configured by using  `logging.level.root` .

The following example shows potential logging settings in  `application.properties` :

```java
logging.level.root=WARN
logging.level.org.springframework.web=DEBUG
logging.level.org.hibernate=ERROR
```

## 26.5 Log Groups

It’s often useful to be able to group related loggers together so that they can all be configured at the same time. For example, you might commonly change the logging levels for all Tomcat related loggers, but you can’t easily remember top level packages.

To help with this, Spring Boot allows you to define logging groups in your Spring  `Environment` . For example, here’s how you could define a “tomcat” group by adding it to your  `application.properties` :

```java
logging.group.tomcat=org.apache.catalina, org.apache.coyote, org.apache.tomcat
```

Once defined, you can change the level for all the loggers in the group with a single line:

```java
logging.level.tomcat=TRACE
```

Spring Boot includes the following pre-defined logging groups that can be used out-of-the-box:

|Name|Loggers|
|----|----|
|web | `org.springframework.core.codec` ,  `org.springframework.http` ,  `org.springframework.web`  |
|sql | `org.springframework.jdbc.core` ,  `org.hibernate.SQL`  |

## 26.6 Custom Log Configuration

The various logging systems can be activated by including the appropriate libraries on the classpath and can be further customized by providing a suitable configuration file in the root of the classpath or in a location specified by the following Spring  `Environment`  property:  `logging.config` .

You can force Spring Boot to use a particular logging system by using the  `org.springframework.boot.logging.LoggingSystem`  system property. The value should be the fully qualified class name of a  `LoggingSystem`  implementation. You can also disable Spring Boot’s logging configuration entirely by using a value of  `none` .

> Since logging is initialized  **before**  the  `ApplicationContext`  is created, it is not possible to control logging from  `@PropertySources`  in Spring  `@Configuration`  files. The only way to change the logging system or disable it entirely is via System properties.

Depending on your logging system, the following files are loaded:

|Logging System|Customization|
|----|----|
|Logback | `logback-spring.xml` ,  `logback-spring.groovy` ,  `logback.xml` , or  `logback.groovy`  |
|Log4j2 | `log4j2-spring.xml`  or  `log4j2.xml`  |
|JDK (Java Util Logging) | `logging.properties`  |

> When possible, we recommend that you use the  `-spring`  variants for your logging configuration (for example,  `logback-spring.xml`  rather than  `logback.xml` ). If you use standard configuration locations, Spring cannot completely control log initialization.

> There are known classloading issues with Java Util Logging that cause problems when running from an 'executable jar'. We recommend that you avoid it when running from an 'executable jar' if at all possible.

To help with the customization, some other properties are transferred from the Spring  `Environment`  to System properties, as described in the following table:

|Spring Environment|System Property|Comments|
|----|----|----|
| `logging.exception-conversion-word`  | `LOG_EXCEPTION_CONVERSION_WORD`  |The conversion word used when logging exceptions. |
| `logging.file`  | `LOG_FILE`  |If defined, it is used in the default log configuration. |
| `logging.file.max-size`  | `LOG_FILE_MAX_SIZE`  |Maximum log file size (if LOG_FILE enabled). (Only supported with the default Logback setup.) |
| `logging.file.max-history`  | `LOG_FILE_MAX_HISTORY`  |Maximum number of archive log files to keep (if LOG_FILE enabled). (Only supported with the default Logback setup.) |
| `logging.path`  | `LOG_PATH`  |If defined, it is used in the default log configuration. |
| `logging.pattern.console`  | `CONSOLE_LOG_PATTERN`  |The log pattern to use on the console (stdout). (Only supported with the default Logback setup.) |
| `logging.pattern.dateformat`  | `LOG_DATEFORMAT_PATTERN`  |Appender pattern for log date format. (Only supported with the default Logback setup.) |
| `logging.pattern.file`  | `FILE_LOG_PATTERN`  |The log pattern to use in a file (if  `LOG_FILE`  is enabled). (Only supported with the default Logback setup.) |
| `logging.pattern.level`  | `LOG_LEVEL_PATTERN`  |The format to use when rendering the log level (default  `%5p` ). (Only supported with the default Logback setup.) |
| `PID`  | `PID`  |The current process ID (discovered if possible and when not already defined as an OS environment variable). |

All the supported logging systems can consult System properties when parsing their configuration files. See the default configurations in  `spring-boot.jar`  for examples:

- [Logback](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/logback/defaults.xml)

- [Log4j 2](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/log4j2/log4j2.xml)

- [Java Util logging](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/java/logging-file.properties)

> If you want to use a placeholder in a logging property, you should use [Spring Boot’s syntax](boot-features-external-config.html#boot-features-external-config-placeholders-in-properties) and not the syntax of the underlying framework. Notably, if you use Logback, you should use  `:`  as the delimiter between a property name and its default value and not use  `:-` .

> You can add MDC and other ad-hoc content to log lines by overriding only the  `LOG_LEVEL_PATTERN`  (or  `logging.pattern.level`  with Logback). For example, if you use  `logging.pattern.level=user:%X{user} %5p` , then the default log format contains an MDC entry for "user", if it exists, as shown in the following example.

## 26.7 Logback Extensions

Spring Boot includes a number of extensions to Logback that can help with advanced configuration. You can use these extensions in your  `logback-spring.xml`  configuration file.

> Because the standard  `logback.xml`  configuration file is loaded too early, you cannot use extensions in it. You need to either use  `logback-spring.xml`  or define a  `logging.config`  property.

> The extensions cannot be used with Logback’s [configuration scanning](http://logback.qos.ch/manual/configuration.html#autoScan). If you attempt to do so, making changes to the configuration file results in an error similar to one of the following being logged:

```java
ERROR in [emailprotected]:71 - no applicable action for [springProperty], current ElementPath is [[configuration][springProperty]]
ERROR in [emailprotected]:71 - no applicable action for [springProfile], current ElementPath is [[configuration][springProfile]]
```

### 26.7.1 Profile-specific Configuration

The  `<springProfile>`  tag lets you optionally include or exclude sections of configuration based on the active Spring profiles. Profile sections are supported anywhere within the  `<configuration>`  element. Use the  `name`  attribute to specify which profile accepts the configuration. The  `<springProfile>`  tag can contain a simple profile name (for example  `staging` ) or a profile expression. A profile expression allows for more complicated profile logic to be expressed, for example  `production & (eu-central | eu-west)` . Check the [reference guide](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/core.html#beans-definition-profiles-java) for more details. The following listing shows three sample profiles:

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

### 26.7.2 Environment Properties

The  `<springProperty>`  tag lets you expose properties from the Spring  `Environment`  for use within Logback. Doing so can be useful if you want to access values from your  `application.properties`  file in your Logback configuration. The tag works in a similar way to Logback’s standard  `<property>`  tag. However, rather than specifying a direct  `value` , you specify the  `source`  of the property (from the  `Environment` ). If you need to store the property somewhere other than in  `local`  scope, you can use the  `scope`  attribute. If you need a fallback value (in case the property is not set in the  `Environment` ), you can use the  `defaultValue`  attribute. The following example shows how to expose properties for use within Logback:

```xml
<springProperty scope="context" name="fluentHost" source="myapp.fluentd.host"
		defaultValue="localhost"/>
<appender name="FLUENT" class="ch.qos.logback.more.appenders.DataFluentAppender">
	<remoteHost>${fluentHost}</remoteHost>
	...
</appender>
```

> The  `source`  must be specified in kebab case (such as  `my.property-name` ). However, properties can be added to the  `Environment`  by using the relaxed rules.

