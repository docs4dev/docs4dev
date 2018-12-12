## 83. Logging

Spring Boot has no mandatory logging dependency, except for the Commons Logging API, which is typically provided by Spring Framework’s  `spring-jcl`  module. To use [Logback](http://logback.qos.ch), you need to include it and  `spring-jcl`  on the classpath. The simplest way to do that is through the starters, which all depend on  `spring-boot-starter-logging` . For a web application, you need only  `spring-boot-starter-web` , since it depends transitively on the logging starter. If you use Maven, the following dependency adds logging for you:

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

Spring Boot has a  `LoggingSystem`  abstraction that attempts to configure logging based on the content of the classpath. If Logback is available, it is the first choice.

If the only change you need to make to logging is to set the levels of various loggers, you can do so in  `application.properties`  by using the "logging.level" prefix, as shown in the following example:

```java
logging.level.org.springframework.web=DEBUG
logging.level.org.hibernate=ERROR
```

You can also set the location of a file to which to write the log (in addition to the console) by using "logging.file".

To configure the more fine-grained settings of a logging system, you need to use the native configuration format supported by the  `LoggingSystem`  in question. By default, Spring Boot picks up the native configuration from its default location for the system (such as  `classpath:logback.xml`  for Logback), but you can set the location of the config file by using the "logging.config" property.

## 83.1 Configure Logback for Logging

If you put a  `logback.xml`  in the root of your classpath, it is picked up from there (or from  `logback-spring.xml` , to take advantage of the templating features provided by Boot). Spring Boot provides a default base configuration that you can include if you want to set levels, as shown in the following example:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
	<include resource="org/springframework/boot/logging/logback/base.xml"/>
	<logger name="org.springframework.web" level="DEBUG"/>
</configuration>
```

If you look at  `base.xml`  in the spring-boot jar, you can see that it uses some useful System properties that the  `LoggingSystem`  takes care of creating for you:

-  `${PID}` : The current process ID.

-  `${LOG_FILE}` : Whether  `logging.file`  was set in Boot’s external configuration.

-  `${LOG_PATH}` : Whether  `logging.path`  (representing a directory for log files to live in) was set in Boot’s external configuration.

-  `${LOG_EXCEPTION_CONVERSION_WORD}` : Whether  `logging.exception-conversion-word`  was set in Boot’s external configuration.

Spring Boot also provides some nice ANSI color terminal output on a console (but not in a log file) by using a custom Logback converter. See the default  `base.xml`  configuration for details.

If Groovy is on the classpath, you should be able to configure Logback with  `logback.groovy`  as well. If present, this setting is given preference.

### 83.1.1 Configure Logback for File-only Output

If you want to disable console logging and write output only to a file, you need a custom  `logback-spring.xml`  that imports  `file-appender.xml`  but not  `console-appender.xml` , as shown in the following example:

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

You also need to add  `logging.file`  to your  `application.properties` , as shown in the following example:

```java
logging.file=myapplication.log
```

## 83.2 Configure Log4j for Logging

Spring Boot supports [Log4j 2](https://logging.apache.org/log4j/2.x) for logging configuration if it is on the classpath. If you use the starters for assembling dependencies, you have to exclude Logback and then include log4j 2 instead. If you do not use the starters, you need to provide (at least)  `spring-jcl`  in addition to Log4j 2.

The simplest path is probably through the starters, even though it requires some jiggling with excludes. The following example shows how to set up the starters in Maven:

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

And the following example shows one way to set up the starters in Gradle:

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

> The Log4j starters gather together the dependencies for common logging requirements (such as having Tomcat use  `java.util.logging`  but configuring the output using Log4j 2). See the [Actuator Log4j 2](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-samples/spring-boot-sample-actuator-log4j2) samples for more detail and to see it in action.

> To ensure that debug logging performed using  `java.util.logging`  is routed into Log4j 2, configure its [JDK logging adapter](https://logging.apache.org/log4j/2.0/log4j-jul/index.html) by setting the  `java.util.logging.manager`  system property to  `org.apache.logging.log4j.jul.LogManager` .

### 83.2.1 Use YAML or JSON to Configure Log4j 2

In addition to its default XML configuration format, Log4j 2 also supports YAML and JSON configuration files. To configure Log4j 2 to use an alternative configuration file format, add the appropriate dependencies to the classpath and name your configuration files to match your chosen file format, as shown in the following example:

|Format|Dependencies|File names|
|----|----|----|
|YAML | `com.fasterxml.jackson.core:jackson-databind`   `com.fasterxml.jackson.dataformat:jackson-dataformat-yaml`  | `log4j2.yaml`   `log4j2.yml`  |
|JSON | `com.fasterxml.jackson.core:jackson-databind`  | `log4j2.json`   `log4j2.jsn`  |

