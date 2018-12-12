## 77. Properties and Configuration

This section includes topics about setting and reading properties and configuration settings and their interaction with Spring Boot applications.

## 77.1 Automatically Expand Properties at Build Time

Rather than hardcoding some properties that are also specified in your project’s build configuration, you can automatically expand them by instead using the existing build configuration. This is possible in both Maven and Gradle.

### 77.1.1 Automatic Property Expansion Using Maven

You can automatically expand properties from the Maven project by using resource filtering. If you use the  `spring-boot-starter-parent` , you can then refer to your Maven ‘project properties’ with  `@[email protected]`  placeholders, as shown in the following example:

```java
app.encoding[emailprotected]@
app.java.version[emailprotected]@
```

> Only production configuration is filtered that way (in other words, no filtering is applied on  `src/test/resources` ).

> If you enable the  `addResources`  flag, the  `spring-boot:run`  goal can add  `src/main/resources`  directly to the classpath (for hot reloading purposes). Doing so circumvents the resource filtering and this feature. Instead, you can use the  `exec:java`  goal or customize the plugin’s configuration. See the [plugin usage page](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/maven-plugin/usage.html) for more details.

If you do not use the starter parent, you need to include the following element inside the  `<build/>`  element of your  `pom.xml` :

```xml
<resources>
	<resource>
		<directory>src/main/resources</directory>
		<filtering>true</filtering>
	</resource>
</resources>
```

You also need to include the following element inside  `<plugins/>` :

```xml
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-resources-plugin</artifactId>
	<version>2.7</version>
	<configuration>
		<delimiters>
			<delimiter>@</delimiter>
		</delimiters>
		<useDefaultDelimiters>false</useDefaultDelimiters>
	</configuration>
</plugin>
```

> The  `useDefaultDelimiters`  property is important if you use standard Spring placeholders (such as  `${placeholder}` ) in your configuration. If that property is not set to  `false` , these may be expanded by the build.

### 77.1.2 Automatic Property Expansion Using Gradle

You can automatically expand properties from the Gradle project by configuring the Java plugin’s  `processResources`  task to do so, as shown in the following example:

```java
processResources {
	expand(project.properties)
}
```

You can then refer to your Gradle project’s properties by using placeholders, as shown in the following example:

```java
app.name=${name}
app.description=${description}
```

> Gradle’s  `expand`  method uses Groovy’s  `SimpleTemplateEngine` , which transforms  `${..}`  tokens. The  `${..}`  style conflicts with Spring’s own property placeholder mechanism. To use Spring property placeholders together with automatic expansion, escape the Spring property placeholders as follows:  `\${..}` .

## 77.2 Externalize the Configuration of SpringApplication

A  `SpringApplication`  has bean properties (mainly setters), so you can use its Java API as you create the application to modify its behavior. Alternatively, you can externalize the configuration by setting properties in  `spring.main.*` . For example, in  `application.properties` , you might have the following settings:

```java
spring.main.web-application-type=none
spring.main.banner-mode=off
```

Then the Spring Boot banner is not printed on startup, and the application is not starting an embedded web server.

Properties defined in external configuration override the values specified with the Java API, with the notable exception of the sources used to create the  `ApplicationContext` . Consider the following application:

```java
new SpringApplicationBuilder()
	.bannerMode(Banner.Mode.OFF)
	.sources(demo.MyApp.class)
	.run(args);
```

Now consider the following configuration:

```java
spring.main.sources=com.acme.Config,com.acme.ExtraConfig
spring.main.banner-mode=console
```

The actual application now shows the banner (as overridden by configuration) and uses three sources for the  `ApplicationContext`  (in the following order):  `demo.MyApp` ,  `com.acme.Config` , and  `com.acme.ExtraConfig` .

## 77.3 Change the Location of External Properties of an Application

By default, properties from different sources are added to the Spring  `Environment`  in a defined order (see “[Chapter 24, Externalized Configuration](boot-features-external-config.html)” in the ‘Spring Boot features’ section for the exact order).

A nice way to augment and modify this ordering is to add  `@PropertySource`  annotations to your application sources. Classes passed to the  `SpringApplication`  static convenience methods and those added using  `setSources()`  are inspected to see if they have  `@PropertySources` . If they do, those properties are added to the  `Environment`  early enough to be used in all phases of the  `ApplicationContext`  lifecycle. Properties added in this way have lower priority than any added by using the default locations (such as  `application.properties` ), system properties, environment variables, or the command line.

You can also provide the following System properties (or environment variables) to change the behavior:

-  `spring.config.name`  ( `SPRING_CONFIG_NAME` ): Defaults to  `application`  as the root of the file name.

-  `spring.config.location`  ( `SPRING_CONFIG_LOCATION` ): The file to load (such as a classpath resource or a URL). A separate  `Environment`  property source is set up for this document and it can be overridden by system properties, environment variables, or the command line.

No matter what you set in the environment, Spring Boot always loads  `application.properties`  as described above. By default, if YAML is used, then files with the ‘.yml’ extension are also added to the list.

Spring Boot logs the configuration files that are loaded at the  `DEBUG`  level and the candidates it has not found at  `TRACE`  level.

See [ConfigFileApplicationListener](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/context/config/ConfigFileApplicationListener.java) for more detail.

## 77.4 Use ‘Short’ Command Line Arguments

Some people like to use (for example)  `--port=9000`  instead of  `--server.port=9000`  to set configuration properties on the command line. You can enable this behavior by using placeholders in  `application.properties` , as shown in the following example:

```java
server.port=${port:8080}
```

> If you inherit from the  `spring-boot-starter-parent`  POM, the default filter token of the  `maven-resources-plugins`  has been changed from  `${*}`  to  `@`  (that is,  `@[email protected]`  instead of  `${maven.token}` ) to prevent conflicts with Spring-style placeholders. If you have enabled Maven filtering for the  `application.properties`  directly, you may want to also change the default filter token to use [other delimiters](https://maven.apache.org/plugins/maven-resources-plugin/resources-mojo.html#delimiters).

> In this specific case, the port binding works in a PaaS environment such as Heroku or Cloud Foundry. In those two platforms, the  `PORT`  environment variable is set automatically and Spring can bind to capitalized synonyms for  `Environment`  properties.

## 77.5 Use YAML for External Properties

YAML is a superset of JSON and, as such, is a convenient syntax for storing external properties in a hierarchical format, as shown in the following example:

```java
spring:
	application:
		name: cruncher
	datasource:
		driverClassName: com.mysql.jdbc.Driver
		url: jdbc:mysql://localhost/test
server:
	port: 9000
```

Create a file called  `application.yml`  and put it in the root of your classpath. Then add  `snakeyaml`  to your dependencies (Maven coordinates  `org.yaml:snakeyaml` , already included if you use the  `spring-boot-starter` ). A YAML file is parsed to a Java  `Map<String,Object>`  (like a JSON object), and Spring Boot flattens the map so that it is one level deep and has period-separated keys, as many people are used to with  `Properties`  files in Java.

The preceding example YAML corresponds to the following  `application.properties`  file:

```java
spring.application.name=cruncher
spring.datasource.driverClassName=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost/test
server.port=9000
```

See “[Section 24.7, “Using YAML Instead of Properties”](boot-features-external-config.html#boot-features-external-config-yaml)” in the ‘Spring Boot features’ section for more information about YAML.

## 77.6 Set the Active Spring Profiles

The Spring  `Environment`  has an API for this, but you would normally set a System property ( `spring.profiles.active` ) or an OS environment variable ( `SPRING_PROFILES_ACTIVE` ). Also, you can launch your application with a  `-D`  argument (remember to put it before the main class or jar archive), as follows:

```java
$ java -jar -Dspring.profiles.active=production demo-0.0.1-SNAPSHOT.jar
```

In Spring Boot, you can also set the active profile in  `application.properties` , as shown in the following example:

```java
spring.profiles.active=production
```

A value set this way is replaced by the System property or environment variable setting but not by the  `SpringApplicationBuilder.profiles()`  method. Thus, the latter Java API can be used to augment the profiles without changing the defaults.

See “[Chapter 25, Profiles](boot-features-profiles.html)” in the “Spring Boot features” section for more information.

## 77.7 Change Configuration Depending on the Environment

A YAML file is actually a sequence of documents separated by  `---`  lines, and each document is parsed separately to a flattened map.

If a YAML document contains a  `spring.profiles`  key, then the profiles value (a comma-separated list of profiles) is fed into the Spring  `Environment.acceptsProfiles()`  method. If any of those profiles is active, that document is included in the final merge (otherwise, it is not), as shown in the following example:

```java
server:
	port: 9000
---

spring:
	profiles: development
server:
	port: 9001

---

spring:
	profiles: production
server:
	port: 0
```

In the preceding example, the default port is 9000. However, if the Spring profile called ‘development’ is active, then the port is 9001. If ‘production’ is active, then the port is 0.

> The YAML documents are merged in the order in which they are encountered. Later values override earlier values.

To do the same thing with properties files, you can use  `application-${profile}.properties`  to specify profile-specific values.

## 77.8 Discover Built-in Options for External Properties

Spring Boot binds external properties from  `application.properties`  (or  `.yml`  files and other places) into an application at runtime. There is not (and technically cannot be) an exhaustive list of all supported properties in a single location, because contributions can come from additional jar files on your classpath.

A running application with the Actuator features has a  `configprops`  endpoint that shows all the bound and bindable properties available through  `@ConfigurationProperties` .

The appendix includes an [application.properties](common-application-properties.html) example with a list of the most common properties supported by Spring Boot. The definitive list comes from searching the source code for  `@ConfigurationProperties`  and  `@Value`  annotations as well as the occasional use of  `Binder` . For more about the exact ordering of loading properties, see "[Chapter 24, Externalized Configuration](boot-features-external-config.html)".

