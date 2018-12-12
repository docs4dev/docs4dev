## 92. Traditional Deployment

Spring Boot supports traditional deployment as well as more modern forms of deployment. This section answers common questions about traditional deployment.

## 92.1 Create a Deployable War File

> Because Spring WebFlux does not strictly depend on the Servlet API and applications are deployed by default on an embedded Reactor Netty server, War deployment is not supported for WebFlux applications.

The first step in producing a deployable war file is to provide a  `SpringBootServletInitializer`  subclass and override its  `configure`  method. Doing so makes use of Spring Framework’s Servlet 3.0 support and lets you configure your application when it is launched by the servlet container. Typically, you should update your application’s main class to extend  `SpringBootServletInitializer` , as shown in the following example:

```java
@SpringBootApplication
public class Application extends SpringBootServletInitializer {

	@Override
	protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
		return application.sources(Application.class);
	}

	public static void main(String[] args) throws Exception {
		SpringApplication.run(Application.class, args);
	}

}
```

The next step is to update your build configuration such that your project produces a war file rather than a jar file. If you use Maven and  `spring-boot-starter-parent`  (which configures Maven’s war plugin for you), all you need to do is to modify  `pom.xml`  to change the packaging to war, as follows:

```xml
<packaging>war</packaging>
```

If you use Gradle, you need to modify  `build.gradle`  to apply the war plugin to the project, as follows:

```java
apply plugin: 'war'
```

The final step in the process is to ensure that the embedded servlet container does not interfere with the servlet container to which the war file is deployed. To do so, you need to mark the embedded servlet container dependency as being provided.

If you use Maven, the following example marks the servlet container (Tomcat, in this case) as being provided:

```xml
<dependencies>
	<!-- … -->
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-tomcat</artifactId>
		<scope>provided</scope>
	</dependency>
	<!-- … -->
</dependencies>
```

If you use Gradle, the following example marks the servlet container (Tomcat, in this case) as being provided:

```java
dependencies {
	// …
	providedRuntime 'org.springframework.boot:spring-boot-starter-tomcat'
	// …
}
```

>  `providedRuntime`  is preferred to Gradle’s  `compileOnly`  configuration. Among other limitations,  `compileOnly`  dependencies are not on the test classpath, so any web-based integration tests fail.

If you use the [Spring Boot build tools](build-tool-plugins.html), marking the embedded servlet container dependency as provided produces an executable war file with the provided dependencies packaged in a  `lib-provided`  directory. This means that, in addition to being deployable to a servlet container, you can also run your application by using  `java -jar`  on the command line.

> Take a look at Spring Boot’s sample applications for a [Maven-based example](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-samples/spring-boot-sample-traditional/pom.xml) of the previously described configuration.

## 92.2 Convert an Existing Application to Spring Boot

For a non-web application, it should be easy to convert an existing Spring application to a Spring Boot application. To do so, throw away the code that creates your  `ApplicationContext`  and replace it with calls to  `SpringApplication`  or  `SpringApplicationBuilder` . Spring MVC web applications are generally amenable to first creating a deployable war application and then migrating it later to an executable war or jar. See the [Getting Started Guide on Converting a jar to a war](https://spring.io/guides/gs/convert-jar-to-war/).

To create a deployable war by extending  `SpringBootServletInitializer`  (for example, in a class called  `Application` ) and adding the Spring Boot  `@SpringBootApplication`  annotation, use code similar to that shown in the following example:

```java
@SpringBootApplication
public class Application extends SpringBootServletInitializer {

	@Override
	protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
		// Customize the application or call application.sources(...) to add sources
		// Since our example is itself a @Configuration class (via @SpringBootApplication)
		// we actually don't need to override this method.
		return application;
	}

}
```

Remember that, whatever you put in the  `sources`  is merely a Spring  `ApplicationContext` . Normally, anything that already works should work here. There might be some beans you can remove later and let Spring Boot provide its own defaults for them, but it should be possible to get something working before you need to do that.

Static resources can be moved to  `/public`  (or  `/static`  or  `/resources`  or  `/META-INF/resources` ) in the classpath root. The same applies to  `messages.properties`  (which Spring Boot automatically detects in the root of the classpath).

Vanilla usage of Spring  `DispatcherServlet`  and Spring Security should require no further changes. If you have other features in your application (for instance, using other servlets or filters), you may need to add some configuration to your  `Application`  context, by replacing those elements from the  `web.xml` , as follows:

- A  `@Bean`  of type  `Servlet`  or  `ServletRegistrationBean`  installs that bean in the container as if it were a  `<servlet/>`  and  `<servlet-mapping/>`  in  `web.xml` .

- A  `@Bean`  of type  `Filter`  or  `FilterRegistrationBean`  behaves similarly (as a  `<filter/>`  and  `<filter-mapping/>` ).

- An  `ApplicationContext`  in an XML file can be added through an  `@ImportResource`  in your  `Application` . Alternatively, simple cases where annotation configuration is heavily used already can be recreated in a few lines as  `@Bean`  definitions.

Once the war file is working, you can make it executable by adding a  `main`  method to your  `Application` , as shown in the following example:

```java
public static void main(String[] args) {
	SpringApplication.run(Application.class, args);
}
```

> If you intend to start your application as a war or as an executable application, you need to share the customizations of the builder in a method that is both available to the  `SpringBootServletInitializer`  callback and in the  `main`  method in a class similar to the following:

Applications can fall into more than one category:

- Servlet 3.0+ applications with no  `web.xml` .

- Applications with a  `web.xml` .

- Applications with a context hierarchy.

- Applications without a context hierarchy.

All of these should be amenable to translation, but each might require slightly different techniques.

Servlet 3.0+ applications might translate pretty easily if they already use the Spring Servlet 3.0+ initializer support classes. Normally, all the code from an existing  `WebApplicationInitializer`  can be moved into a  `SpringBootServletInitializer` . If your existing application has more than one  `ApplicationContext`  (for example, if it uses  `AbstractDispatcherServletInitializer` ) then you might be able to combine all your context sources into a single  `SpringApplication` . The main complication you might encounter is if combining does not work and you need to maintain the context hierarchy. See the [entry on building a hierarchy](howto-spring-boot-application.html#howto-build-an-application-context-hierarchy) for examples. An existing parent context that contains web-specific features usually needs to be broken up so that all the  `ServletContextAware`  components are in the child context.

Applications that are not already Spring applications might be convertible to Spring Boot applications, and the previously mentioned guidance may help. However, you may yet encounter problems. In that case, we suggest [asking questions on Stack Overflow with a tag of spring-boot](https://stackoverflow.com/questions/tagged/spring-boot).

## 92.3 Deploying a WAR to WebLogic

To deploy a Spring Boot application to WebLogic, you must ensure that your servlet initializer  **directly**  implements  `WebApplicationInitializer`  (even if you extend from a base class that already implements it).

A typical initializer for WebLogic should resemble the following example:

```java
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;
import org.springframework.web.WebApplicationInitializer;

@SpringBootApplication
public class MyApplication extends SpringBootServletInitializer implements WebApplicationInitializer {

}
```

If you use Logback, you also need to tell WebLogic to prefer the packaged version rather than the version that was pre-installed with the server. You can do so by adding a  `WEB-INF/weblogic.xml`  file with the following contents:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<wls:weblogic-web-app
	xmlns:wls="http://xmlns.oracle.com/weblogic/weblogic-web-app"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
		http://java.sun.com/xml/ns/javaee/ejb-jar_3_0.xsd
		http://xmlns.oracle.com/weblogic/weblogic-web-app
		http://xmlns.oracle.com/weblogic/weblogic-web-app/1.4/weblogic-web-app.xsd">
	<wls:container-descriptor>
		<wls:prefer-application-packages>
			<wls:package-name>org.slf4j</wls:package-name>
		</wls:prefer-application-packages>
	</wls:container-descriptor>
</wls:weblogic-web-app>
```

## 92.4 Use Jedis Instead of Lettuce

By default, the Spring Boot starter ( `spring-boot-starter-data-redis` ) uses [Lettuce](https://github.com/lettuce-io/lettuce-core/). You need to exclude that dependency and include the [Jedis](https://github.com/xetorthio/jedis/) one instead. Spring Boot manages these dependencies to help make this process as easy as possible.

The following example shows how to do so in Maven:

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-redis</artifactId>
	<exclusions>
		<exclusion>
			<groupId>io.lettuce</groupId>
			<artifactId>lettuce-core</artifactId>
		</exclusion>
	</exclusions>
</dependency>
<dependency>
	<groupId>redis.clients</groupId>
	<artifactId>jedis</artifactId>
</dependency>
```

The following example shows how to do so in Gradle:

```java
configurations {
	compile.exclude module: "lettuce"
}

dependencies {
	compile("redis.clients:jedis")
	// ...
}
```

