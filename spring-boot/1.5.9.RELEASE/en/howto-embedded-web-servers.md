## 78. Embedded Web Servers

Each Spring Boot web application includes an embedded web server. This feature leads to a number of how-to questions, including how to change the embedded server and how to configure the embedded server. This section answers those questions.

## 78.1 Use Another Web Server

Many Spring Boot starters include default embedded containers.

- For servlet stack applications, the  `spring-boot-starter-web`  includes Tomcat by including  `spring-boot-starter-tomcat` , but you can use  `spring-boot-starter-jetty`  or  `spring-boot-starter-undertow`  instead.

- For reactive stack applications, the  `spring-boot-starter-webflux`  includes Reactor Netty by including  `spring-boot-starter-reactor-netty` , but you can use  `spring-boot-starter-tomcat` ,  `spring-boot-starter-jetty` , or  `spring-boot-starter-undertow`  instead.

When switching to a different HTTP server, you need to exclude the default dependencies in addition to including the one you need. Spring Boot provides separate starters for HTTP servers to help make this process as easy as possible.

The following Maven example shows how to exclude Tomcat and include Jetty for Spring MVC:

```xml
<properties>
	<servlet-api.version>3.1.0</servlet-api.version>
</properties>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	<exclusions>
		<!-- Exclude the Tomcat dependency -->
		<exclusion>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
		</exclusion>
	</exclusions>
</dependency>
<!-- Use Jetty instead -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

> The version of the Servlet API has been overridden as, unlike Tomcat 9 and Undertow 2.0, Jetty 9.4 does not support Servlet 4.0.

The following Gradle example shows how to exclude Netty and include Undertow for Spring WebFlux:

```java
configurations {
	// exclude Reactor Netty
	compile.exclude module: 'spring-boot-starter-reactor-netty'
}

dependencies {
	compile 'org.springframework.boot:spring-boot-starter-webflux'
	// Use Undertow instead
	compile 'org.springframework.boot:spring-boot-starter-undertow'
	// ...
}
```

>  `spring-boot-starter-reactor-netty`  is required to use the  `WebClient`  class, so you may need to keep a dependency on Netty even when you need to include a different HTTP server.

## 78.2 Disabling the Web Server

If your classpath contains the necessary bits to start a web server, Spring Boot will automatically start it. To disable this behaviour configure the  `WebApplicationType`  in your  `application.properties` , as shown in the following example:

```java
spring.main.web-application-type=none
```

## 78.3 Change the HTTP Port

In a standalone application, the main HTTP port defaults to  `8080`  but can be set with  `server.port`  (for example, in  `application.properties`  or as a System property). Thanks to relaxed binding of  `Environment`  values, you can also use  `SERVER_PORT`  (for example, as an OS environment variable).

To switch off the HTTP endpoints completely but still create a  `WebApplicationContext` , use  `server.port=-1` . (Doing so is sometimes useful for testing.)

For more details, see “[Section 28.4.4, “Customizing Embedded Servlet Containers”](boot-features-developing-web-applications.html#boot-features-customizing-embedded-containers)” in the ‘Spring Boot features’ section, or the [ServerProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ServerProperties.java) source code.

## 78.4 Use a Random Unassigned HTTP Port

To scan for a free port (using OS natives to prevent clashes) use  `server.port=0` .

## 78.5 Discover the HTTP Port at Runtime

You can access the port the server is running on from log output or from the  `ServletWebServerApplicationContext`  through its  `WebServer` . The best way to get that and be sure that it has been initialized is to add a  `@Bean`  of type  `ApplicationListener<ServletWebServerInitializedEvent>`  and pull the container out of the event when it is published.

Tests that use  `@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)`  can also inject the actual port into a field by using the  `@LocalServerPort`  annotation, as shown in the following example:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)
public class MyWebIntegrationTests {

	@Autowired
	ServletWebServerApplicationContext server;

	@LocalServerPort
	int port;

	// ...

}
```

>  `@LocalServerPort`  is a meta-annotation for  `@Value("${local.server.port}")` . Do not try to inject the port in a regular application. As we just saw, the value is set only after the container has been initialized. Contrary to a test, application code callbacks are processed early (before the value is actually available).

## 78.6 Enable HTTP Response Compression

HTTP response compression is supported by Jetty, Tomcat, and Undertow. It can be enabled in  `application.properties` , as follows:

```java
server.compression.enabled=true
```

By default, responses must be at least 2048 bytes in length for compression to be performed. You can configure this behavior by setting the  `server.compression.min-response-size`  property.

By default, responses are compressed only if their content type is one of the following:

-  `text/html` 

-  `text/xml` 

-  `text/plain` 

-  `text/css` 

-  `text/javascript` 

-  `application/javascript` 

-  `application/json` 

-  `application/xml` 

You can configure this behavior by setting the  `server.compression.mime-types`  property.

## 78.7 Configure SSL

SSL can be configured declaratively by setting the various  `server.ssl.*`  properties, typically in  `application.properties`  or  `application.yml` . The following example shows setting SSL properties in  `application.properties` :

```java
server.port=8443
server.ssl.key-store=classpath:keystore.jks
server.ssl.key-store-password=secret
server.ssl.key-password=another-secret
```

See [Ssl](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/web/server/Ssl.java) for details of all of the supported properties.

Using configuration such as the preceding example means the application no longer supports a plain HTTP connector at port 8080. Spring Boot does not support the configuration of both an HTTP connector and an HTTPS connector through  `application.properties` . If you want to have both, you need to configure one of them programmatically. We recommend using  `application.properties`  to configure HTTPS, as the HTTP connector is the easier of the two to configure programmatically. See the [spring-boot-sample-tomcat-multi-connectors](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-samples/spring-boot-sample-tomcat-multi-connectors) sample project for an example.

## 78.8 Configure HTTP/2

You can enable HTTP/2 support in your Spring Boot application with the  `server.http2.enabled`  configuration property. This support depends on the chosen web server and the application environment, since that protocol is not supported out-of-the-box by JDK8.

> Spring Boot does not support  `h2c` , the cleartext version of the HTTP/2 protocol. So you must [configure SSL first](howto-embedded-web-servers.html#howto-configure-ssl).

### 78.8.1 HTTP/2 with Undertow

As of Undertow 1.4.0+, HTTP/2 is supported without any additional requirement on JDK8.

### 78.8.2 HTTP/2 with Jetty

As of Jetty 9.4.8, HTTP/2 is also supported with the [Conscrypt library](https://www.conscrypt.org/). To enable that support, your application needs to have two additional dependencies:  `org.eclipse.jetty:jetty-alpn-conscrypt-server`  and  `org.eclipse.jetty.http2:http2-server` .

### 78.8.3 HTTP/2 with Tomcat

Spring Boot ships by default with Tomcat 9.0.x which supports HTTP/2 out of the box when using JDK 9 or later. Alternatively, HTTP/2 can be used on JDK 8 if the  `libtcnative`  library and its dependencies are installed on the host operating system.

The library folder must be made available, if not already, to the JVM library path. You can do so with a JVM argument such as  `-Djava.library.path=/usr/local/opt/tomcat-native/lib` . More on this in the [official Tomcat documentation](https://tomcat.apache.org/tomcat-9.0-doc/apr.html).

Starting Tomcat 9.0.x on JDK 8 without that native support logs the following error:

```java
ERROR 8787 --- [           main] o.a.coyote.http11.Http11NioProtocol      : The upgrade handler [org.apache.coyote.http2.Http2Protocol] for [h2] only supports upgrade via ALPN but has been configured for the ["https-jsse-nio-8443"] connector that does not support ALPN.
```

This error is not fatal, and the application still starts with HTTP/1.1 SSL support.

### 78.8.4 HTTP/2 with Reactor Netty

The  `spring-boot-webflux-starter`  is using by default Reactor Netty as a server. Reactor Netty can be configured for HTTP/2 using the JDK support with JDK 9 or later. For JDK 8 environments, or for optimal runtime performance, this server also supports HTTP/2 with native libraries. To enable that, your application needs to have an additional dependency.

Spring Boot manages the version for the  `io.netty:netty-tcnative-boringssl-static`  "uber jar", containing native libraries for all platforms. Developers can choose to import only the required dependencies using a classifier (see [the Netty official documentation](http://netty.io/wiki/forked-tomcat-native.html)).

## 78.9 Configure the Web Server

Generally, you should first consider using one of the many available configuration keys and customize your web server by adding new entries in your  `application.properties`  (or  `application.yml` , or environment, etc. see “[Section 77.8, “Discover Built-in Options for External Properties”](howto-properties-and-configuration.html#howto-discover-build-in-options-for-external-properties)”). The  `server.*`  namespace is quite useful here, and it includes namespaces like  `server.tomcat.*` ,  `server.jetty.*`  and others, for server-specific features. See the list of [Appendix A, Common application properties](common-application-properties.html).

The previous sections covered already many common use cases, such as compression, SSL or HTTP/2. However, if a configuration key doesn’t exist for your use case, you should then look at [WebServerFactoryCustomizer](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/api/org/springframework/boot/web/server/WebServerFactoryCustomizer.html). You can declare such a component and get access to the server factory relevant to your choice: you should select the variant for the chosen Server (Tomcat, Jetty, Reactor Netty, Undertow) and the chosen web stack (Servlet or Reactive).

The example below is for Tomcat with the  `spring-boot-starter-web`  (Servlet stack):

```java
@Component
public class MyTomcatWebServerCustomizer
		implements WebServerFactoryCustomizer<TomcatServletWebServerFactory> {

	@Override
	public void customize(TomcatServletWebServerFactory factory) {
		// customize the factory here
	}
}
```

In addition Spring Boot provides:

|Server|Servlet stack|Reactive stack|
|----|----|----|
|Tomcat | `TomcatServletWebServerFactory`  | `TomcatReactiveWebServerFactory`  |
|Jetty | `JettyServletWebServerFactory`  | `JettyReactiveWebServerFactory`  |
|Undertow | `UndertowServletWebServerFactory`  | `UndertowReactiveWebServerFactory`  |
|Reactor |N/A | `NettyReactiveWebServerFactory`  |

Once you’ve got access to a  `WebServerFactory` , you can often add customizers to it to configure specific parts, like connectors, server resources, or the server itself - all using server-specific APIs.

As a last resort, you can also declare your own  `WebServerFactory`  component, which will override the one provided by Spring Boot. In this case, you can’t rely on configuration properties in the  `server`  namespace anymore.

## 78.10 Add a Servlet, Filter, or Listener to an Application

In a servlet stack application, i.e. with the  `spring-boot-starter-web` , there are two ways to add  `Servlet` ,  `Filter` ,  `ServletContextListener` , and the other listeners supported by the Servlet API to your application:

- [Section 78.10.1, “Add a Servlet, Filter, or Listener by Using a Spring Bean”](howto-embedded-web-servers.html#howto-add-a-servlet-filter-or-listener-as-spring-bean)

- [Section 78.10.2, “Add Servlets, Filters, and Listeners by Using Classpath Scanning”](howto-embedded-web-servers.html#howto-add-a-servlet-filter-or-listener-using-scanning)

### 78.10.1 Add a Servlet, Filter, or Listener by Using a Spring Bean

To add a  `Servlet` ,  `Filter` , or Servlet  `*Listener`  by using a Spring bean, you must provide a  `@Bean`  definition for it. Doing so can be very useful when you want to inject configuration or dependencies. However, you must be very careful that they do not cause eager initialization of too many other beans, because they have to be installed in the container very early in the application lifecycle. (For example, it is not a good idea to have them depend on your  `DataSource`  or JPA configuration.) You can work around such restrictions by initializing the beans lazily when first used instead of on initialization.

In the case of  `Filters`  and  `Servlets` , you can also add mappings and init parameters by adding a  `FilterRegistrationBean`  or a  `ServletRegistrationBean`  instead of or in addition to the underlying component.

> If no  `dispatcherType`  is specified on a filter registration,  `REQUEST`  is used. This aligns with the Servlet Specification’s default dispatcher type.

Like any other Spring bean, you can define the order of Servlet filter beans; please make sure to check the “[the section called “Registering Servlets, Filters, and Listeners as Spring Beans”](boot-features-developing-web-applications.html#boot-features-embedded-container-servlets-filters-listeners-beans)” section.

#### Disable Registration of a Servlet or Filter

As [described earlier](howto-embedded-web-servers.html#howto-add-a-servlet-filter-or-listener-as-spring-bean), any  `Servlet`  or  `Filter`  beans are registered with the servlet container automatically. To disable registration of a particular  `Filter`  or  `Servlet`  bean, create a registration bean for it and mark it as disabled, as shown in the following example:

```java
@Bean
public FilterRegistrationBean registration(MyFilter filter) {
	FilterRegistrationBean registration = new FilterRegistrationBean(filter);
	registration.setEnabled(false);
	return registration;
}
```

### 78.10.2 Add Servlets, Filters, and Listeners by Using Classpath Scanning

`@WebServlet` ,  `@WebFilter` , and  `@WebListener`  annotated classes can be automatically registered with an embedded servlet container by annotating a  `@Configuration`  class with  `@ServletComponentScan`  and specifying the package(s) containing the components that you want to register. By default,  `@ServletComponentScan`  scans from the package of the annotated class.

## 78.11 Configure Access Logging

Access logs can be configured for Tomcat, Undertow, and Jetty through their respective namespaces.

For instance, the following settings log access on Tomcat with a [custom pattern](https://tomcat.apache.org/tomcat-8.5-doc/config/valve.html#Access_Logging).

```java
server.tomcat.basedir=my-tomcat
server.tomcat.accesslog.enabled=true
server.tomcat.accesslog.pattern=%t %a "%r" %s (%D ms)
```

> The default location for logs is a  `logs`  directory relative to the Tomcat base directory. By default, the  `logs`  directory is a temporary directory, so you may want to fix Tomcat’s base directory or use an absolute path for the logs. In the preceding example, the logs are available in  `my-tomcat/logs`  relative to the working directory of the application.

Access logging for Undertow can be configured in a similar fashion, as shown in the following example:

```java
server.undertow.accesslog.enabled=true
server.undertow.accesslog.pattern=%t %a "%r" %s (%D ms)
```

Logs are stored in a  `logs`  directory relative to the working directory of the application. You can customize this location by setting the  `server.undertow.accesslog.directory`  property.

Finally, access logging for Jetty can also be configured as follows:

```java
server.jetty.accesslog.enabled=true
server.jetty.accesslog.filename=/var/log/jetty-access.log
```

By default, logs are redirected to  `System.err` . For more details, see [the Jetty documentation](https://www.eclipse.org/jetty/documentation/9.4.x/configuring-jetty-request-logs.html).

## 78.12 Running Behind a Front-end Proxy Server

Your application might need to send  `302`  redirects or render content with absolute links back to itself. When running behind a proxy, the caller wants a link to the proxy and not to the physical address of the machine hosting your app. Typically, such situations are handled through a contract with the proxy, which adds headers to tell the back end how to construct links to itself.

If the proxy adds conventional  `X-Forwarded-For`  and  `X-Forwarded-Proto`  headers (most proxy servers do so), the absolute links should be rendered correctly, provided  `server.use-forward-headers`  is set to  `true`  in your  `application.properties` .

> If your application runs in Cloud Foundry or Heroku, the  `server.use-forward-headers`  property defaults to  `true` . In all other instances, it defaults to  `false` .

### 78.12.1 Customize Tomcat’s Proxy Configuration

If you use Tomcat, you can additionally configure the names of the headers used to carry “forwarded” information, as shown in the following example:

```java
server.tomcat.remote-ip-header=x-your-remote-ip-header
server.tomcat.protocol-header=x-your-protocol-header
```

Tomcat is also configured with a default regular expression that matches internal proxies that are to be trusted. By default, IP addresses in  `10/8` ,  `192.168/16` ,  `169.254/16`  and  `127/8`  are trusted. You can customize the valve’s configuration by adding an entry to  `application.properties` , as shown in the following example:

```java
server.tomcat.internal-proxies=192\\.168\\.\\d{1,3}\\.\\d{1,3}
```

> The double backslashes are required only when you use a properties file for configuration. If you use YAML, single backslashes are sufficient, and a value equivalent to that shown in the preceding example would be  `192\.168\.\d{1,3}\.\d{1,3}` .

> You can trust all proxies by setting the  `internal-proxies`  to empty (but do not do so in production).

You can take complete control of the configuration of Tomcat’s  `RemoteIpValve`  by switching the automatic one off (to do so, set  `server.use-forward-headers=false` ) and adding a new valve instance in a  `TomcatServletWebServerFactory`  bean.

## 78.13 Enable Multiple Connectors with Tomcat

You can add an  `org.apache.catalina.connector.Connector`  to the  `TomcatServletWebServerFactory` , which can allow multiple connectors, including HTTP and HTTPS connectors, as shown in the following example:

```java
@Bean
public ServletWebServerFactory servletContainer() {
	TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();
	tomcat.addAdditionalTomcatConnectors(createSslConnector());
	return tomcat;
}

private Connector createSslConnector() {
	Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
	Http11NioProtocol protocol = (Http11NioProtocol) connector.getProtocolHandler();
	try {
		File keystore = new ClassPathResource("keystore").getFile();
		File truststore = new ClassPathResource("keystore").getFile();
		connector.setScheme("https");
		connector.setSecure(true);
		connector.setPort(8443);
		protocol.setSSLEnabled(true);
		protocol.setKeystoreFile(keystore.getAbsolutePath());
		protocol.setKeystorePass("changeit");
		protocol.setTruststoreFile(truststore.getAbsolutePath());
		protocol.setTruststorePass("changeit");
		protocol.setKeyAlias("apitester");
		return connector;
	}
	catch (IOException ex) {
		throw new IllegalStateException("can't access keystore: [" + "keystore"
				+ "] or truststore: [" + "keystore" + "]", ex);
	}
}
```

## 78.14 Use Tomcat’s LegacyCookieProcessor

By default, the embedded Tomcat used by Spring Boot does not support "Version 0" of the Cookie format, so you may see the following error:

```java
java.lang.IllegalArgumentException: An invalid character [32] was present in the Cookie value
```

If at all possible, you should consider updating your code to only store values compliant with later Cookie specifications. If, however, you cannot change the way that cookies are written, you can instead configure Tomcat to use a  `LegacyCookieProcessor` . To switch to the  `LegacyCookieProcessor` , use an  `WebServerFactoryCustomizer`  bean that adds a  `TomcatContextCustomizer` , as shown in the following example:

```java
@Bean
public WebServerFactoryCustomizer<TomcatServletWebServerFactory> cookieProcessorCustomizer() {
	return (factory) -> factory.addContextCustomizers(
			(context) -> context.setCookieProcessor(new LegacyCookieProcessor()));
}
```

## 78.15 Enable Multiple Listeners with Undertow

Add an  `UndertowBuilderCustomizer`  to the  `UndertowServletWebServerFactory`  and add a listener to the  `Builder` , as shown in the following example:

```java
@Bean
public UndertowServletWebServerFactory servletWebServerFactory() {
	UndertowServletWebServerFactory factory = new UndertowServletWebServerFactory();
	factory.addBuilderCustomizers(new UndertowBuilderCustomizer() {

		@Override
		public void customize(Builder builder) {
			builder.addHttpListener(8080, "0.0.0.0");
		}

	});
	return factory;
}
```

## 78.16 Create WebSocket Endpoints Using @ServerEndpoint

If you want to use  `@ServerEndpoint`  in a Spring Boot application that used an embedded container, you must declare a single  `ServerEndpointExporter`   `@Bean` , as shown in the following example:

```java
@Bean
public ServerEndpointExporter serverEndpointExporter() {
	return new ServerEndpointExporter();
}
```

The bean shown in the preceding example registers any  `@ServerEndpoint`  annotated beans with the underlying WebSocket container. When deployed to a standalone servlet container, this role is performed by a servlet container initializer, and the  `ServerEndpointExporter`  bean is not required.

