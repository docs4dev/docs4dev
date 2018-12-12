## 83.记录

除了Commons Logging API之外，Spring Boot没有强制日志记录依赖性，Commons Logging API通常由Spring Framework的 `spring-jcl` 模块提供.要使用[Logback](http://logback.qos.ch)，您需要在类路径中包含它和 `spring-jcl` .最简单的方法是通过启动器，这些都依赖于 `spring-boot-starter-logging` .对于Web应用程序，您只需要 `spring-boot-starter-web` ，因为它依赖于日志记录启动器.如果您使用Maven，以下依赖项会为您添加日志记录：

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

Spring Boot有一个 `LoggingSystem` 抽象，它试图根据类路径的内容配置日志记录.如果Logback可用，则它是第一选择.

如果您需要对日志记录进行的唯一更改是设置各种Logger的级别，则可以使用"logging.level"前缀在 `application.properties` 中执行此操作，如以下示例所示：

```java
logging.level.org.springframework.web=DEBUG
logging.level.org.hibernate=ERROR
```

您还可以使用“logging.file”设置要写入日志的文件的位置（除控制台外）.

要配置日志记录系统的更细粒度设置，您需要使用 `LoggingSystem` 支持的本机配置格式.默认情况下，Spring Boot从系统的默认位置（例如 `classpath:logback.xml`  for Logback）中选择本机配置，但您可以使用"logging.config"属性设置配置文件的位置.

## 83.1配置日志记录的日志记录

如果将 `logback.xml` 放在类路径的根目录中，则从那里（或从 `logback-spring.xml` 中获取）以利用Boot提供的模板功能. Spring Boot提供了一个默认的基本配置，如果要设置级别，可以包括该配置，如以下示例所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
	<include resource="org/springframework/boot/logging/logback/base.xml"/>
	<logger name="org.springframework.web" level="DEBUG"/>
</configuration>
```

如果你看一下spring-boot jar中的 `base.xml` ，你可以看到它使用了 `LoggingSystem` 为你创建的一些有用的System属性：

-  `${PID}` ：当前进程ID.

-  `${LOG_FILE}` ：是否在Boot的外部配置中设置了 `logging.file` .

-  `${LOG_PATH}` ：是否在Boot的外部配置中设置了 `logging.path` （表示存放日志文件的目录）.

-  `${LOG_EXCEPTION_CONVERSION_WORD}` ：是否在Boot的外部配置中设置了 `logging.exception-conversion-word` .

Spring Boot还通过使用自定义Logback转换器在控制台上（但不在日志文件中）提供了一些漂亮的ANSI颜色终端输出.有关详细信息，请参阅默认的 `base.xml` 配置.

如果Groovy在类路径上，您应该能够使用 `logback.groovy` 配置Logback.如果存在，则优先考虑此设置.

### 83.1.1为仅文件输出配置回溯

如果要禁用控制台日志记录并仅将输出写入文件，则需要导入 `file-appender.xml` 但不导入 `console-appender.xml` 的自定义 `logback-spring.xml` ，如以下示例所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
	<include resource="org/springframework/boot/logging/logback/defaults.xml" />
	<property name="LOG_FILE" value="${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}/}spring.log}"/>
	<include resource="org/springframework/boot/logging/logback/file-appender.xml" />
	<root level="INFO">
		<appender-ref ref="FILE" />
	</root>
</configuration>
```

您还需要将 `logging.file` 添加到 `application.properties` ，如以下示例所示：

```java
logging.file=myapplication.log
```

## 83.2配置Log4j进行日志记录

如果Spring路径在类路径上，则Spring Boot支持[Log4j 2](https://logging.apache.org/log4j/2.x)用于记录配置.如果使用启动器来组装依赖项，则必须排除Logback，然后包含log4j 2.如果您不使用启动器，除了Log4j 2之外，还需要提供（至少） `spring-jcl` .

最简单的路径可能是通过初学者，即使它需要一些与排除的摇晃.以下示例显示如何在Maven中设置启动器：

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter</artifactId>
	<exclusions>
		<exclusion>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-logging</artifactId>
		</exclusion>
	</exclusions>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

和以下示例显示了在Gradle中设置启动器的一种方法：

```java
dependencies {
	compile 'org.springframework.boot:spring-boot-starter-web'
	compile 'org.springframework.boot:spring-boot-starter-log4j2'
}

configurations {
	all {
		exclude group: 'org.springframework.boot', module: 'spring-boot-starter-logging'
	}
}
```

>  Log4j启动器聚集了常见日志记录要求的依赖关系（例如让Tomcat使用 `java.util.logging` 但使用Log4j 2配置输出）.有关更多详细信息，请参阅[Actuator Log4j 2](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-samples/spring-boot-sample-actuator-log4j2)示例并查看其实际操作.

> 确保使用 `java.util.logging` 执行的调试日志记录路由到Log4j 2，通过将 `java.util.logging.manager` 系统属性设置为 `org.apache.logging.log4j.jul.LogManager` 来配置其[JDK logging adapter](https://logging.apache.org/log4j/2.0/log4j-jul/index.html).

### 83.2.1使用YAML或JSON配置Log4j 2

除了默认的XML配置格式外，Log4j 2还支持YAML和JSON配置文件.要将Log4j 2配置为使用备用配置文件格式，请将相应的依赖项添加到类路径，并将配置文件命名为与所选文件格式匹配，如以下示例所示：

|格式|依赖关系|文件名|
| ---- | ---- | ---- |
| YAML |  `com.fasterxml.jackson.core:jackson-databind`   `com.fasterxml.jackson.dataformat:jackson-dataformat-yaml`  |  `log4j2.yaml`   `log4j2.yml`  |
| JSON |  `com.fasterxml.jackson.core:jackson-databind`  |  `log4j2.json`   `log4j2.jsn`  |

