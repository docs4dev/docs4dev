## 92.传统部署

Spring Boot支持传统部署以及更现代的部署形式.本节回答有关传统部署的常见问题.

## 92.1创建可部署的战争文件

> B因为Spring WebFlux并不严格依赖于Servlet API，默认情况下会在嵌入式Reactor Netty服务器上部署应用程序，因此WebFlux应用程序不支持War部署.

生成可部署war文件的第一步是提供 `SpringBootServletInitializer` 子类并覆盖其 `configure` 方法.这样做可以利用Spring Framework的Servlet 3.0支持，并允许您在servlet容器启动时配置应用程序.通常，您应该更新应用程序的主类以扩展 `SpringBootServletInitializer` ，如以下示例所示：

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

下一步是更新构建配置，以便项目生成war文件而不是jar文件.如果您使用Maven和 `spring-boot-starter-parent` （为您配置Maven的war插件），您需要做的就是修改 `pom.xml` 以将包装更改为war，如下所示：

```xml
<packaging>war</packaging>
```

如果使用Gradle，则需要修改 `build.gradle` 以将war插件应用于项目，如下所示：

```java
apply plugin: 'war'
```

该过程的最后一步是确保嵌入式servlet容器不会干扰部署war文件的servlet容器.为此，您需要将嵌入式servlet容器依赖项标记为已提供.

如果您使用Maven，以下示例将servlet容器（在本例中为Tomcat）标记为提供：

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

如果使用Gradle，则以下示例将servlet容器（在本例中为Tomcat）标记为提供：

```java
dependencies {
	// …
	providedRuntime 'org.springframework.boot:spring-boot-starter-tomcat'
	// …
}
```

>  `providedRuntime` 优先于Gradle的 `compileOnly` 配置.除了其他限制之外， `compileOnly` 依赖项不在测试类路径中，因此任何基于Web的集成测试都会失败.

如果使用[Spring Boot build tools](build-tool-plugins.html)，则标记所提供的嵌入式servlet容器依赖项将生成一个可执行war文件，其中提供的依赖项打包在 `lib-provided` 目录中.这意味着，除了存在之外可部署到servlet容器，也可以在命令行上使用 `java -jar` 来运行应用程序.

_0015-查看Spring Boot的示例应用程序，了解上述配置的[Maven-based example](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-samples/spring-boot-sample-traditional/pom.xml).

## 92.2将现有应用程序转换为Spring Boot

对于非Web应用程序，应该很容易将现有的Spring应用程序转换为Spring Boot应用程序.为此，请丢弃创建 `ApplicationContext` 的代码，并将其替换为对 `SpringApplication` 或 `SpringApplicationBuilder` 的调用. Spring MVC Web应用程序通常可以首先创建可部署的war应用程序，然后再将其迁移到可执行的war或jar.见[Getting Started Guide on Converting a jar to a war](https://spring.io/guides/gs/convert-jar-to-war/).

要通过扩展 `SpringBootServletInitializer` （例如，在名为 `Application` 的类中）并添加Spring Boot  `@SpringBootApplication` 注释来创建可部署的war，请使用类似于以下示例中所示的代码：

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

请记住，无论你放入 `sources` 只是一个Spring天 `ApplicationContext` .通常，任何已经有效的东西都应该在这里工作.可能有一些bean可以在以后删除，让Spring Boot为它们提供自己的默认值，但是应该可以在需要之前获得一些工作.

静态资源可以移动到类路径根目录中的 `/public` （或 `/static` 或 `/resources` 或 `/META-INF/resources` ）.这同样适用于 `messages.properties` （Spring Boot会自动在类路径的根中检测到）.

使用Spring  `DispatcherServlet` 和Spring Security的Vanilla应该不需要进一步更改.如果应用程序中有其他功能（例如，使用其他servlet或过滤器），则可能需要通过从 `web.xml` 替换这些元素来向 `Application` 上下文添加一些配置，如下所示：

类型为 `Servlet` 或 `ServletRegistrationBean` 的
- A  `@Bean` 在容器中安装该bean，就好像它是 `<servlet/>` 中的 `<servlet/>` 和 `<servlet-mapping/>` 一样.
类型 `Filter` 或 `FilterRegistrationBean` 的
- A  `@Bean` 表现相似（作为 `<filter/>` 和 `<filter-mapping/>` ）.
可以通过 `Application` 中的 `@ImportResource` 添加XML文件中的
- An  `ApplicationContext` .或者，已经大量使用注释配置的简单情况可以在几行中重新创建为 `@Bean` 定义.

一旦war文件正常工作，您可以通过向 `Application` 添加 `main` 方法使其可执行，如以下示例所示：

```java
public static void main(String[] args) {
	SpringApplication.run(Application.class, args);
}
```

> 如果您打算将应用程序作为战争或可执行应用程序启动，则需要在 `SpringBootServletInitializer` 回调和类似于以下类的类中的 `main` 方法中使用的方法中共享构建器的自定义：

应用程序可以分为多个类别：

没有 `web.xml` 的
- Servlet 3.0应用程序.
带有 `web.xml` 的
- Applications.

- 具有上下文层次结构的应用程序.

- 没有上下文层次结构的应用程序.

所有这些都应该适合翻译，但每种都可能需要稍微不同的技术.

如果Servlet 3.0应用程序已经使用了Spring Servlet 3.0初始化程序支持类，那么它们可能很容易翻译.通常，现有 `WebApplicationInitializer` 中的所有代码都可以移动到 `SpringBootServletInitializer` 中.如果您现有的应用程序有多个 `ApplicationContext` （例如，如果它使用 `AbstractDispatcherServletInitializer` ），那么您可以将所有上下文源组合成一个 `SpringApplication` .您可能遇到的主要复杂问题是，如果组合不起作用，您需要维护上下文层次结构.有关示例，请参阅[entry on building a hierarchy](howto-spring-boot-application.html#howto-build-an-application-context-hierarchy).通常需要拆分包含特定于Web的功能的现有父上下文，以便所有 `ServletContextAware` 组件都位于子上下文中.

尚未安装Spring应用程序的应用程序可以转换为Spring Boot应用程序，前面提到的指南可能有所帮助.但是，您可能会遇到问题.在这种情况下，我们建议[asking questions on Stack Overflow with a tag of spring-boot](https://stackoverflow.com/questions/tagged/spring-boot).

## 92.3将WAR部署到WebLogic

要将Spring Boot应用程序部署到WebLogic，必须确保servlet初始化程序 **directly** 实现 `WebApplicationInitializer` （即使从已实现它的基类扩展）也是如此.

WebLogic的典型初始化程序应类似于以下示例：

```java
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;
import org.springframework.web.WebApplicationInitializer;

@SpringBootApplication
public class MyApplication extends SpringBootServletInitializer implements WebApplicationInitializer {

}
```

如果使用Logback，则还需要告知WebLogic更喜欢打包版本而不是服务器预安装的版本.您可以通过添加包含以下内容的 `WEB-INF/weblogic.xml` 文件来执行此操作：

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

## 92.4使用Jedis代替生菜

默认情况下，Spring Boot启动程序（ `spring-boot-starter-data-redis` ）使用[Lettuce](https://github.com/lettuce-io/lettuce-core/).您需要排除该依赖项并改为包含[Jedis](https://github.com/xetorthio/jedis/). Spring Boot管理这些依赖项，以帮助使这个过程尽可能简单.

以下示例显示了如何在Maven中执行此操作：

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

以下示例显示了如何在Gradle中执行此操作：

```java
configurations {
	compile.exclude module: "lettuce"
}

dependencies {
	compile("redis.clients:jedis")
	// ...
}
```

