## 23. SpringApplication

The  `SpringApplication`  class provides a convenient way to bootstrap a Spring application that is started from a  `main()`  method. In many situations, you can delegate to the static  `SpringApplication.run`  method, as shown in the following example:

```java
public static void main(String[] args) {
	SpringApplication.run(MySpringConfiguration.class, args);
}
```

When your application starts, you should see something similar to the following output:

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

By default,  `INFO`  logging messages are shown, including some relevant startup details, such as the user that launched the application. If you need a log level other than  `INFO` , you can set it, as described in [Section 26.4, “Log Levels”](boot-features-logging.html#boot-features-custom-log-levels),

## 23.1 Startup Failure

If your application fails to start, registered  `FailureAnalyzers`  get a chance to provide a dedicated error message and a concrete action to fix the problem. For instance, if you start a web application on port  `8080`  and that port is already in use, you should see something similar to the following message:

```java
***************************
APPLICATION FAILED TO START
***************************

Description:

Embedded servlet container failed to start. Port 8080 was already in use.

Action:

Identify and stop the process that's listening on port 8080 or configure this application to listen on another port.
```

> Spring Boot provides numerous  `FailureAnalyzer`  implementations, and you can [add your own](howto-spring-boot-application.html#howto-failure-analyzer).

If no failure analyzers are able to handle the exception, you can still display the full conditions report to better understand what went wrong. To do so, you need to [enable the debug property](boot-features-external-config.html) or [enable DEBUG logging](boot-features-logging.html#boot-features-custom-log-levels) for  `org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener` .

For instance, if you are running your application by using  `java -jar` , you can enable the  `debug`  property as follows:

```java
$ java -jar myproject-0.0.1-SNAPSHOT.jar --debug
```

## 23.2 Customizing the Banner

The banner that is printed on start up can be changed by adding a  `banner.txt`  file to your classpath or by setting the  `spring.banner.location`  property to the location of such a file. If the file has an encoding other than UTF-8, you can set  `spring.banner.charset` . In addition to a text file, you can also add a  `banner.gif` ,  `banner.jpg` , or  `banner.png`  image file to your classpath or set the  `spring.banner.image.location`  property. Images are converted into an ASCII art representation and printed above any text banner.

Inside your  `banner.txt`  file, you can use any of the following placeholders:

**Table 23.1. Banner variables** 

|Variable|Description|
|----|----|
| `${application.version}`  |The version number of your application, as declared in  `MANIFEST.MF` . For example,  `Implementation-Version: 1.0`  is printed as  `1.0` . |
| `${application.formatted-version}`  |The version number of your application, as declared in  `MANIFEST.MF`  and formatted for display (surrounded with brackets and prefixed with  `v` ). For example  `(v1.0)` . |
| `${spring-boot.version}`  |The Spring Boot version that you are using. For example  `2.1.0.RELEASE` . |
| `${spring-boot.formatted-version}`  |The Spring Boot version that you are using, formatted for display (surrounded with brackets and prefixed with  `v` ). For example  `(v2.1.0.RELEASE)` . |
| `${Ansi.NAME}`  (or  `${AnsiColor.NAME}` ,  `${AnsiBackground.NAME}` ,  `${AnsiStyle.NAME}` ) |Where  `NAME`  is the name of an ANSI escape code. See [AnsiPropertySource](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/ansi/AnsiPropertySource.java) for details. |
| `${application.title}`  |The title of your application, as declared in  `MANIFEST.MF` . For example  `Implementation-Title: MyApp`  is printed as  `MyApp` . |

> The  `SpringApplication.setBanner(…)`  method can be used if you want to generate a banner programmatically. Use the  `org.springframework.boot.Banner`  interface and implement your own  `printBanner()`  method.

You can also use the  `spring.main.banner-mode`  property to determine if the banner has to be printed on  `System.out`  ( `console` ), sent to the configured logger ( `log` ), or not produced at all ( `off` ).

The printed banner is registered as a singleton bean under the following name:  `springBootBanner` .

> YAML maps  `off`  to  `false` , so be sure to add quotes if you want to disable the banner in your application, as shown in the following example:

## 23.3 Customizing SpringApplication

If the  `SpringApplication`  defaults are not to your taste, you can instead create a local instance and customize it. For example, to turn off the banner, you could write:

```java
public static void main(String[] args) {
	SpringApplication app = new SpringApplication(MySpringConfiguration.class);
	app.setBannerMode(Banner.Mode.OFF);
	app.run(args);
}
```

> The constructor arguments passed to  `SpringApplication`  are configuration sources for Spring beans. In most cases, these are references to  `@Configuration`  classes, but they could also be references to XML configuration or to packages that should be scanned.

It is also possible to configure the  `SpringApplication`  by using an  `application.properties`  file. See [Chapter 24, Externalized Configuration](boot-features-external-config.html) for details.

For a complete list of the configuration options, see the [SpringApplication Javadoc](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/api/org/springframework/boot/SpringApplication.html).

## 23.4 Fluent Builder API

If you need to build an  `ApplicationContext`  hierarchy (multiple contexts with a parent/child relationship) or if you prefer using a “fluent” builder API, you can use the  `SpringApplicationBuilder` .

The  `SpringApplicationBuilder`  lets you chain together multiple method calls and includes  `parent`  and  `child`  methods that let you create a hierarchy, as shown in the following example:

```java
new SpringApplicationBuilder()
		.sources(Parent.class)
		.child(Application.class)
		.bannerMode(Banner.Mode.OFF)
		.run(args);
```

> There are some restrictions when creating an  `ApplicationContext`  hierarchy. For example, Web components  **must**  be contained within the child context, and the same  `Environment`  is used for both parent and child contexts. See the [SpringApplicationBuilder Javadoc](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/api/org/springframework/boot/builder/SpringApplicationBuilder.html) for full details.

## 23.5 Application Events and Listeners

In addition to the usual Spring Framework events, such as [ContextRefreshedEvent](https://docs.spring.io/spring/docs/5.1.2.RELEASE/javadoc-api/org/springframework/context/event/ContextRefreshedEvent.html), a  `SpringApplication`  sends some additional application events.

> Some events are actually triggered before the  `ApplicationContext`  is created, so you cannot register a listener on those as a  `@Bean` . You can register them with the  `SpringApplication.addListeners(…)`  method or the  `SpringApplicationBuilder.listeners(…)`  method.

Application events are sent in the following order, as your application runs:

An ApplicationStartingEvent is sent at the start of a run but before any processing, except for the registration of listeners and initializers. An ApplicationEnvironmentPreparedEvent is sent when the Environment to be used in the context is known but before the context is created. An ApplicationPreparedEvent is sent just before the refresh is started but after bean definitions have been loaded. An ApplicationStartedEvent is sent after the context has been refreshed but before any application and command-line runners have been called. An ApplicationReadyEvent is sent after any application and command-line runners have been called. It indicates that the application is ready to service requests. An ApplicationFailedEvent is sent if there is an exception on startup.

> You often need not use application events, but it can be handy to know that they exist. Internally, Spring Boot uses events to handle a variety of tasks.

Application events are sent by using Spring Framework’s event publishing mechanism. Part of this mechanism ensures that an event published to the listeners in a child context is also published to the listeners in any ancestor contexts. As a result of this, if your application uses a hierarchy of  `SpringApplication`  instances, a listener may receive multiple instances of the same type of application event.

To allow your listener to distinguish between an event for its context and an event for a descendant context, it should request that its application context is injected and then compare the injected context with the context of the event. The context can be injected by implementing  `ApplicationContextAware`  or, if the listener is a bean, by using  `@Autowired` .

## 23.6 Web Environment

A  `SpringApplication`  attempts to create the right type of  `ApplicationContext`  on your behalf. The algorithm used to determine a  `WebApplicationType`  is fairly simple:

- If Spring MVC is present, an  `AnnotationConfigServletWebServerApplicationContext`  is used

- If Spring MVC is not present and Spring WebFlux is present, an  `AnnotationConfigReactiveWebServerApplicationContext`  is used

- Otherwise,  `AnnotationConfigApplicationContext`  is used

This means that if you are using Spring MVC and the new  `WebClient`  from Spring WebFlux in the same application, Spring MVC will be used by default. You can override that easily by calling  `setWebApplicationType(WebApplicationType)` .

It is also possible to take complete control of the  `ApplicationContext`  type that is used by calling  `setApplicationContextClass(…)` .

> It is often desirable to call  `setWebApplicationType(WebApplicationType.NONE)`  when using  `SpringApplication`  within a JUnit test.

## 23.7 Accessing Application Arguments

If you need to access the application arguments that were passed to  `SpringApplication.run(…)` , you can inject a  `org.springframework.boot.ApplicationArguments`  bean. The  `ApplicationArguments`  interface provides access to both the raw  `String[]`  arguments as well as parsed  `option`  and  `non-option`  arguments, as shown in the following example:

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

> Spring Boot also registers a  `CommandLinePropertySource`  with the Spring  `Environment` . This lets you also inject single application arguments by using the  `@Value`  annotation.

## 23.8 Using the ApplicationRunner or CommandLineRunner

If you need to run some specific code once the  `SpringApplication`  has started, you can implement the  `ApplicationRunner`  or  `CommandLineRunner`  interfaces. Both interfaces work in the same way and offer a single  `run`  method, which is called just before  `SpringApplication.run(…)`  completes.

The  `CommandLineRunner`  interfaces provides access to application arguments as a simple string array, whereas the  `ApplicationRunner`  uses the  `ApplicationArguments`  interface discussed earlier. The following example shows a  `CommandLineRunner`  with a  `run`  method:

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

If several  `CommandLineRunner`  or  `ApplicationRunner`  beans are defined that must be called in a specific order, you can additionally implement the  `org.springframework.core.Ordered`  interface or use the  `org.springframework.core.annotation.Order`  annotation.

## 23.9 Application Exit

Each  `SpringApplication`  registers a shutdown hook with the JVM to ensure that the  `ApplicationContext`  closes gracefully on exit. All the standard Spring lifecycle callbacks (such as the  `DisposableBean`  interface or the  `@PreDestroy`  annotation) can be used.

In addition, beans may implement the  `org.springframework.boot.ExitCodeGenerator`  interface if they wish to return a specific exit code when  `SpringApplication.exit()`  is called. This exit code can then be passed to  `System.exit()`  to return it as a status code, as shown in the following example:

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

Also, the  `ExitCodeGenerator`  interface may be implemented by exceptions. When such an exception is encountered, Spring Boot returns the exit code provided by the implemented  `getExitCode()`  method.

## 23.10 Admin Features

It is possible to enable admin-related features for the application by specifying the  `spring.application.admin.enabled`  property. This exposes the [SpringApplicationAdminMXBean](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/admin/SpringApplicationAdminMXBean.java) on the platform  `MBeanServer` . You could use this feature to administer your Spring Boot application remotely. This feature could also be useful for any service wrapper implementation.

> If you want to know on which HTTP port the application is running, get the property with a key of  `local.server.port` .

> Take care when enabling this feature, as the MBean exposes a method to shutdown the application.

