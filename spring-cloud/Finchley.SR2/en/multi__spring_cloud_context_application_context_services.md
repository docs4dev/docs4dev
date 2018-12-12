## 2. Spring Cloud Context: Application Context Services

Spring Boot has an opinionated view of how to build an application with Spring. For instance, it has conventional locations for common configuration files and has endpoints for common management and monitoring tasks. Spring Cloud builds on top of that and adds a few features that probably all components in a system would use or occasionally need.

## 2.1 The Bootstrap Application Context

A Spring Cloud application operates by creating a “bootstrap” context, which is a parent context for the main application. It is responsible for loading configuration properties from the external sources and for decrypting properties in the local external configuration files. The two contexts share an  `Environment` , which is the source of external properties for any Spring application. By default, bootstrap properties (not  `bootstrap.properties`  but properties that are loaded during the bootstrap phase) are added with high precedence, so they cannot be overridden by local configuration.

The bootstrap context uses a different convention for locating external configuration than the main application context. Instead of  `application.yml`  (or  `.properties` ), you can use  `bootstrap.yml` , keeping the external configuration for bootstrap and main context nicely separate. The following listing shows an example:

**bootstrap.yml.**  

```java
spring:
application:
name: foo
cloud:
config:
uri: ${SPRING_CONFIG_URI:http://localhost:8888}
```

If your application needs any application-specific configuration from the server, it is a good idea to set the  `spring.application.name`  (in  `bootstrap.yml`  or  `application.yml` ).

You can disable the bootstrap process completely by setting  `spring.cloud.bootstrap.enabled=false`  (for example, in system properties).

## 2.2 Application Context Hierarchies

If you build an application context from  `SpringApplication`  or  `SpringApplicationBuilder` , then the Bootstrap context is added as a parent to that context. It is a feature of Spring that child contexts inherit property sources and profiles from their parent, so the “main” application context contains additional property sources, compared to building the same context without Spring Cloud Config. The additional property sources are:

- “bootstrap”: If any  `PropertySourceLocators`  are found in the Bootstrap context and if they have non-empty properties, an optional  `CompositePropertySource`  appears with high priority. An example would be properties from the Spring Cloud Config Server. See “[Section 2.6, “Customizing the Bootstrap Property Sources”](multi__spring_cloud_context_application_context_services.html#customizing-bootstrap-property-sources)” for instructions on how to customize the contents of this property source.

- “applicationConfig: [classpath:bootstrap.yml]” (and related files if Spring profiles are active): If you have a  `bootstrap.yml`  (or  `.properties` ), those properties are used to configure the Bootstrap context. Then they get added to the child context when its parent is set. They have lower precedence than the  `application.yml`  (or  `.properties` ) and any other property sources that are added to the child as a normal part of the process of creating a Spring Boot application. See “[Section 2.3, “Changing the Location of Bootstrap Properties”](multi__spring_cloud_context_application_context_services.html#customizing-bootstrap-properties)” for instructions on how to customize the contents of these property sources.

Because of the ordering rules of property sources, the “bootstrap” entries take precedence. However, note that these do not contain any data from  `bootstrap.yml` , which has very low precedence but can be used to set defaults.

You can extend the context hierarchy by setting the parent context of any  `ApplicationContext`  you create — for example, by using its own interface or with the  `SpringApplicationBuilder`  convenience methods ( `parent()` ,  `child()`  and  `sibling()` ). The bootstrap context is the parent of the most senior ancestor that you create yourself. Every context in the hierarchy has its own “bootstrap” (possibly empty) property source to avoid promoting values inadvertently from parents down to their descendants. If there is a Config Server, every context in the hierarchy can also (in principle) have a different  `spring.application.name`  and, hence, a different remote property source. Normal Spring application context behavior rules apply to property resolution: properties from a child context override those in the parent, by name and also by property source name. (If the child has a property source with the same name as the parent, the value from the parent is not included in the child).

Note that the  `SpringApplicationBuilder`  lets you share an  `Environment`  amongst the whole hierarchy, but that is not the default. Thus, sibling contexts, in particular, do not need to have the same profiles or property sources, even though they may share common values with their parent.

## 2.3 Changing the Location of Bootstrap Properties

The  `bootstrap.yml`  (or  `.properties` ) location can be specified by setting  `spring.cloud.bootstrap.name`  (default:  `bootstrap` ) or  `spring.cloud.bootstrap.location`  (default: empty) — for example, in System properties. Those properties behave like the  `spring.config.*`  variants with the same name. In fact, they are used to set up the bootstrap  `ApplicationContext`  by setting those properties in its  `Environment` . If there is an active profile (from  `spring.profiles.active`  or through the  `Environment`  API in the context you are building), properties in that profile get loaded as well, the same as in a regular Spring Boot app — for example, from  `bootstrap-development.properties`  for a  `development`  profile.

## 2.4 Overriding the Values of Remote Properties

The property sources that are added to your application by the bootstrap context are often “remote” (from example, from Spring Cloud Config Server). By default, they cannot be overridden locally. If you want to let your applications override the remote properties with their own System properties or config files, the remote property source has to grant it permission by setting  `spring.cloud.config.allowOverride=true`  (it does not work to set this locally). Once that flag is set, two finer-grained settings control the location of the remote properties in relation to system properties and the application’s local configuration:

-  `spring.cloud.config.overrideNone=true` : Override from any local property source.

-  `spring.cloud.config.overrideSystemProperties=false` : Only system properties, command line arguments, and environment variables (but not the local config files) should override the remote settings.

## 2.5 Customizing the Bootstrap Configuration

The bootstrap context can be set to do anything you like by adding entries to  `/META-INF/spring.factories`  under a key named  `org.springframework.cloud.bootstrap.BootstrapConfiguration` . This holds a comma-separated list of Spring  `@Configuration`  classes that are used to create the context. Any beans that you want to be available to the main application context for autowiring can be created here. There is a special contract for  `@Beans`  of type  `ApplicationContextInitializer` . If you want to control the startup sequence, classes can be marked with an  `@Order`  annotation (the default order is  `last` ).

> When adding custom  `BootstrapConfiguration` , be careful that the classes you add are not  `@ComponentScanned`  by mistake into your “main” application context, where they might not be needed. Use a separate package name for boot configuration classes and make sure that name is not already covered by your  `@ComponentScan`  or  `@SpringBootApplication`  annotated configuration classes.

The bootstrap process ends by injecting initializers into the main  `SpringApplication`  instance (which is the normal Spring Boot startup sequence, whether it is running as a standalone application or deployed in an application server). First, a bootstrap context is created from the classes found in  `spring.factories` . Then, all  `@Beans`  of type  `ApplicationContextInitializer`  are added to the main  `SpringApplication`  before it is started.

## 2.6 Customizing the Bootstrap Property Sources

The default property source for external configuration added by the bootstrap process is the Spring Cloud Config Server, but you can add additional sources by adding beans of type  `PropertySourceLocator`  to the bootstrap context (through  `spring.factories` ). For instance, you can insert additional properties from a different server or from a database.

As an example, consider the following custom locator:

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

The  `Environment`  that is passed in is the one for the  `ApplicationContext`  about to be created — in other words, the one for which we supply additional property sources for. It already has its normal Spring Boot-provided property sources, so you can use those to locate a property source specific to this  `Environment`  (for example, by keying it on  `spring.application.name` , as is done in the default Spring Cloud Config Server property source locator).

If you create a jar with this class in it and then add a  `META-INF/spring.factories`  containing the following, the  `customProperty`   `PropertySource`  appears in any application that includes that jar on its classpath:

```java
org.springframework.cloud.bootstrap.BootstrapConfiguration=sample.custom.CustomPropertySourceLocator
```

## 2.7 Logging Configuration

If you are going to use Spring Boot to configure log settings than you should place this configuration in `bootstrap.[yml | properties] if you would like it to apply to all events.

> For Spring Cloud to initialize logging configuration properly you cannot use a custom prefix. For example, using  `custom.loggin.logpath`  will not be recognized by Spring Cloud when initializing the logging system.

## 2.8 Environment Changes

The application listens for an  `EnvironmentChangeEvent`  and reacts to the change in a couple of standard ways (additional  `ApplicationListeners`  can be added as  `@Beans`  by the user in the normal way). When an  `EnvironmentChangeEvent`  is observed, it has a list of key values that have changed, and the application uses those to:

- Re-bind any  `@ConfigurationProperties`  beans in the context

- Set the logger levels for any properties in  `logging.level.*` 

Note that the Config Client does not, by default, poll for changes in the  `Environment` . Generally, we would not recommend that approach for detecting changes (although you could set it up with a  `@Scheduled`  annotation). If you have a scaled-out client application, it is better to broadcast the  `EnvironmentChangeEvent`  to all the instances instead of having them polling for changes (for example, by using the [Spring Cloud Bus](https://github.com/spring-cloud/spring-cloud-bus)).

The  `EnvironmentChangeEvent`  covers a large class of refresh use cases, as long as you can actually make a change to the  `Environment`  and publish the event. Note that those APIs are public and part of core Spring). You can verify that the changes are bound to  `@ConfigurationProperties`  beans by visiting the  `/configprops`  endpoint (a normal Spring Boot Actuator feature). For instance, a  `DataSource`  can have its  `maxPoolSize`  changed at runtime (the default  `DataSource`  created by Spring Boot is an  `@ConfigurationProperties`  bean) and grow capacity dynamically. Re-binding  `@ConfigurationProperties`  does not cover another large class of use cases, where you need more control over the refresh and where you need a change to be atomic over the whole  `ApplicationContext` . To address those concerns, we have  `@RefreshScope` .

## 2.9 Refresh Scope

When there is a configuration change, a Spring  `@Bean`  that is marked as  `@RefreshScope`  gets special treatment. This feature addresses the problem of stateful beans that only get their configuration injected when they are initialized. For instance, if a  `DataSource`  has open connections when the database URL is changed via the  `Environment` , you probably want the holders of those connections to be able to complete what they are doing. Then, the next time something borrows a connection from the pool, it gets one with the new URL.

Sometimes, it might even be mandatory to apply the  `@RefreshScope`  annotation on some beans which can be only initialized once. If a bean is "immutable", you will have to either annotate the bean with  `@RefreshScope`  or specify the classname under the property key  `spring.cloud.refresh.extra-refreshable` .

Refresh scope beans are lazy proxies that initialize when they are used (that is, when a method is called), and the scope acts as a cache of initialized values. To force a bean to re-initialize on the next method call, you must invalidate its cache entry.

The  `RefreshScope`  is a bean in the context and has a public  `refreshAll()`  method to refresh all beans in the scope by clearing the target cache. The  `/refresh`  endpoint exposes this functionality (over HTTP or JMX). To refresh an individual bean by name, there is also a  `refresh(String)`  method.

To expose the  `/refresh`  endpoint, you need to add following configuration to your application:

```java
management:
endpoints:
web:
exposure:
include: refresh
```

>  `@RefreshScope`  works (technically) on an  `@Configuration`  class, but it might lead to surprising behavior. For example, it does not mean that all the  `@Beans`  defined in that class are themselves in  `@RefreshScope` . Specifically, anything that depends on those beans cannot rely on them being updated when a refresh is initiated, unless it is itself in  `@RefreshScope` . In that case, it is rebuilt on a refresh and its dependencies are re-injected. At that point, they are re-initialized from the refreshed  `@Configuration` ).

## 2.10 Encryption and Decryption

Spring Cloud has an  `Environment`  pre-processor for decrypting property values locally. It follows the same rules as the Config Server and has the same external configuration through  `encrypt.*` . Thus, you can use encrypted values in the form of  `{cipher}*`  and, as long as there is a valid key, they are decrypted before the main application context gets the  `Environment`  settings. To use the encryption features in an application, you need to include Spring Security RSA in your classpath (Maven co-ordinates: "org.springframework.security:spring-security-rsa"), and you also need the full strength JCE extensions in your JVM.

If you get an exception due to "Illegal key size" and you use Sun’s JDK, you need to install the Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files. See the following links for more information:

- [Java 6 JCE](http://www.oracle.com/technetwork/java/javase/downloads/jce-6-download-429243.html)

- [Java 7 JCE](http://www.oracle.com/technetwork/java/javase/downloads/jce-7-download-432124.html)

- [Java 8 JCE](http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html)

Extract the files into the JDK/jre/lib/security folder for whichever version of JRE/JDK x64/x86 you use.

## 2.11 Endpoints

For a Spring Boot Actuator application, some additional management endpoints are available. You can use:

-  `POST`  to  `/actuator/env`  to update the  `Environment`  and rebind  `@ConfigurationProperties`  and log levels.

-  `/actuator/refresh`  to re-load the boot strap context and refresh the  `@RefreshScope`  beans.

-  `/actuator/restart`  to close the  `ApplicationContext`  and restart it (disabled by default).

-  `/actuator/pause`  and  `/actuator/resume`  for calling the  `Lifecycle`  methods ( `stop()`  and  `start()`  on the  `ApplicationContext` ).

> If you disable the  `/actuator/restart`  endpoint then the  `/actuator/pause`  and  `/actuator/resume`  endpoints will also be disabled since they are just a special case of  `/actuator/restart` .

