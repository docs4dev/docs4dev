## 78.嵌入式Web服务器

每个Spring Boot Web应用程序都包含一个嵌入式Web服务器.此功能会导致许多操作方法问题，包括如何更改嵌入式服务器以及如何配置嵌入式服务器.本节回答了这些问题.

## 78.1使用其他Web服务器

许多Spring Boot启动器都包含默认的嵌入式容器.

- 对于servlet堆栈应用程序， `spring-boot-starter-web` 包含Tomcat包含 `spring-boot-starter-tomcat` ，但您可以使用 `spring-boot-starter-jetty` 或 `spring-boot-starter-undertow` 代替.

- 对于反应堆栈应用程序， `spring-boot-starter-webflux` 包含 `spring-boot-starter-reactor-netty` 包含Reactor Netty，但您可以使用 `spring-boot-starter-tomcat` ， `spring-boot-starter-jetty` 或 `spring-boot-starter-undertow` 代替.

切换到其他HTTP服务器时，除了包含所需的依赖项外，还需要排除默认依赖项. Spring Boot为HTTP服务器提供单独的启动程序，以帮助使此过程尽可能简单.

以下Maven示例显示如何排除Tomcat并为Spring MVC包含Jetty：

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

> 已经重写了Servlet API的版本，与Tomcat 9和Undertow 2.0不同，Jetty 9.4不支持Servlet 4.0.

以下Gradle示例显示如何排除Netty并包含Spring WebFlux的Undertow：

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

>  `spring-boot-starter-reactor-netty` 是使用 `WebClient` 类所必需的，因此即使您需要包含其他HTTP服务器，也可能需要依赖Netty.

## 78.2禁用Web服务器

如果您的类路径包含启动Web服务器所需的位，Spring Boot将自动启动它.要禁用此行为，请在 `application.properties` 中配置 `WebApplicationType` ，如以下示例所示：

```java
spring.main.web-application-type=none
```

## 78.3更改HTTP端口

在独立应用程序中，主HTTP端口默认为 `8080` ，但可以使用 `server.port` 进行设置（例如，在 `application.properties` 中或作为系统属性）.由于 `Environment` 值的轻松绑定，您还可以使用 `SERVER_PORT` （例如，作为OS环境变量）.

要完全关闭HTTPendpoints但仍然创建 `WebApplicationContext` ，请使用 `server.port=-1` . （这样做有时对测试很有用.）

有关更多详细信息，请参阅“Spring Boot功能”部分中的“[Section 28.4.4, “Customizing Embedded Servlet Containers”](boot-features-developing-web-applications.html#boot-features-customizing-embedded-containers)”或[ServerProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ServerProperties.java)源代码.

## 78.4使用随机未分配的HTTP端口

要扫描空闲端口（使用OS本机来防止冲突），请使用 `server.port=0` .

## 78.5在运行时发现HTTP端口

您可以从日志输出或 `ServletWebServerApplicationContext` 到 `WebServer` 访问服务器正在运行的端口.获得该功能并确保已初始化的最佳方法是添加 `@Bean` 类型 `ApplicationListener<ServletWebServerInitializedEvent>` ，并在发布时将容器拉出事件.

使用 `@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)` 的测试也可以使用 `@LocalServerPort` 注释将实际端口注入字段，如以下示例所示：

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

>  `@LocalServerPort` 是 `@Value("${local.server.port}")` 的元注释.不要尝试在常规应用程序中注入端口.正如我们刚刚看到的那样，只有在容器初始化之后才设置该值.与测试相反，应用程序代码回调会尽早处理（在值实际可用之前）.

## 78.6启用HTTP响应压缩

Jetty，Tomcat和Undertow支持HTTP响应压缩.它可以在 `application.properties` 中启用，如下所示：

```java
server.compression.enabled=true
```

默认情况下，响应必须至少为2048字节，才能执行压缩.您可以通过设置 `server.compression.min-response-size` 属性来配置此行为.

默认情况下，只有在内容类型为以下内容之一时才会压缩响应：

-  `text/html` 

-  `text/xml` 

-  `text/plain` 

-  `text/css` 

-  `text/javascript` 

-  `application/javascript` 

-  `application/json` 

-  `application/xml` 

您可以通过设置 `server.compression.mime-types` 属性来配置此行为.

## 78.7配置SSL

可以通过设置各种 `server.ssl.*` 属性以声明方式配置SSL，通常在 `application.properties` 或 `application.yml` 中.以下示例显示了在 `application.properties` 中设置SSL属性：

```java
server.port=8443
server.ssl.key-store=classpath:keystore.jks
server.ssl.key-store-password=secret
server.ssl.key-password=another-secret
```

有关所有受支持属性的详细信息，请参阅[Ssl](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/web/server/Ssl.java).

使用上述示例之类的配置意味着应用程序不再支持端口8080上的普通HTTP连接器.Spring Boot不支持通过 `application.properties` 配置HTTP连接器和HTTPS连接器.如果要同时使用这两者，则需要以编程方式配置其中一个.我们建议使用 `application.properties` 配置HTTPS，因为HTTP连接器更容易以编程方式配置.有关示例，请参见[spring-boot-sample-tomcat-multi-connectors](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-samples/spring-boot-sample-tomcat-multi-connectors)示例项目.

## 78.8配置HTTP / 2

您可以使用 `server.http2.enabled` 配置属性在Spring Boot应用程序中启用HTTP / 2支持.此支持取决于所选的Web服务器和应用程序环境，因为JDK8不支持该协议.

> Spring Boot不支持 `h2c` ，即HTTP / 2协议的明文版本.所以你必须[configure SSL first](howto-embedded-web-servers.html#howto-configure-ssl).

带有Undertow的
### 78.8.1 HTTP / 2

从Undertow 1.4.0开始，支持HTTP / 2，对JDK8没有任何额外要求.

带有Jetty的
### 78.8.2 HTTP / 2

从Jetty 9.4.8开始，[Conscrypt library](https://www.conscrypt.org/)也支持HTTP / 2.要启用该支持，您的应用程序需要有两个额外的依赖项： `org.eclipse.jetty:jetty-alpn-conscrypt-server` 和 `org.eclipse.jetty.http2:http2-server` .

与Tomcat一起使用
### 78.8.3 HTTP / 2

Spring Boot默认使用Tomcat 9.0.x，它在使用JDK 9或更高版本时支持HTTP / 2开箱即用.或者，如果 `libtcnative` 库及其依赖项安装在主机操作系统上，则可以在JDK 8上使用HTTP / 2.

必须使库文件夹（如果尚未可用）到JVM库路径.您可以使用JVM参数（例如 `-Djava.library.path=/usr/local/opt/tomcat-native/lib` ）执行此操作.更多关于[official Tomcat documentation](https://tomcat.apache.org/tomcat-9.0-doc/apr.html)的内容.

在没有该本机支持的情况下在JDK 8上启动Tomcat 9.0.x会记录以下错误：

```java
ERROR 8787 --- [           main] o.a.coyote.http11.Http11NioProtocol      : The upgrade handler [org.apache.coyote.http2.Http2Protocol] for [h2] only supports upgrade via ALPN but has been configured for the ["https-jsse-nio-8443"] connector that does not support ALPN.
```

此错误不是致命的，应用程序仍然以HTTP / 1.1 SSL支持启动.

### 78.8.4 HTTP / 2与Reactor Netty

`spring-boot-webflux-starter` 默认使用Reactor Netty作为服务器.可以使用JDK 9或更高版本的JDK支持为Reactor Netty配置HTTP / 2.对于JDK 8环境或最佳运行时性能，此服务器还支持具有本机库的HTTP / 2.要启用它，您的应用程序需要具有其他依赖项.

Spring Boot管理 `io.netty:netty-tcnative-boringssl-static`  "uber jar"的版本，包含所有平台的本机库.开发人员可以选择使用分类器仅导入所需的依赖项（请参阅[the Netty official documentation](http://netty.io/wiki/forked-tomcat-native.html)）.

## 78.9配置Web服务器

通常，您应首先考虑使用众多可用配置键中的一个，并通过在 `application.properties` （或 `application.yml` 或环境等）中添加新条目来自定义Web服务器，请参阅“[Section 77.8, “Discover Built-in Options for External Properties”](howto-properties-and-configuration.html#howto-discover-build-in-options-for-external-properties)”. `server.*` 名称空间在这里非常有用，它包括 `server.tomcat.*` ， `server.jetty.*` 等名称空间，用于特定于服务器的功能.请参阅[Appendix A, Common application properties](common-application-properties.html)列表.

前面的部分涵盖了许多常见用例，例如压缩，SSL或HTTP / 2.但是，如果您的用例不存在配置密钥，则应查看[WebServerFactoryCustomizer](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/api/org/springframework/boot/web/server/WebServerFactoryCustomizer.html).您可以声明这样的组件并获得与您选择的服务器工厂相关的访问权限：您应该为所选服务器（Tomcat，Jetty，Reactor Netty，Undertow）和所选Web堆栈（Servlet或Reactive）选择变体.

以下示例适用于具有 `spring-boot-starter-web` （Servlet堆栈）的Tomcat：

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

另外Spring Boot提供：

|服务器| Servlet堆栈|反应堆栈|
| ---- | ---- | ---- |
| Tomcat |  `TomcatServletWebServerFactory`  |  `TomcatReactiveWebServerFactory`  |
|码头|  `JettyServletWebServerFactory`  |  `JettyReactiveWebServerFactory`  |
| Undertow |  `UndertowServletWebServerFactory`  |  `UndertowReactiveWebServerFactory`  |
|反应器| N / A |  `NettyReactiveWebServerFactory`  |

一旦您有权访问 `WebServerFactory` ，您通常可以向其添加定制器以配置特定部件，例如连接器，服务器资源或服务器本身 - 所有这些都使用特定于服务器的API.

作为最后的手段，您还可以声明自己的 `WebServerFactory` 组件，它将覆盖Spring Boot提供的组件.在这种情况下，您不能再依赖 `server` 命名空间中的配置属性.

## 78.10向应用程序添加Servlet，过滤器或监听器

在servlet堆栈应用程序中，即使用 `spring-boot-starter-web` ，有两种方法可以将 `Servlet` ， `Filter` ， `ServletContextListener` 和Servlet API支持的其他侦听器添加到应用程序中：

- [Section 78.10.1, “Add a Servlet, Filter, or Listener by Using a Spring Bean”](howto-embedded-web-servers.html#howto-add-a-servlet-filter-or-listener-as-spring-bean)

- [Section 78.10.2, “Add Servlets, Filters, and Listeners by Using Classpath Scanning”](howto-embedded-web-servers.html#howto-add-a-servlet-filter-or-listener-using-scanning)

### 78.10.1使用Spring Bean添加Servlet，过滤器或监听器

要使用Spring bean添加 `Servlet` ， `Filter` 或Servlet  `*Listener` ，必须为其提供 `@Bean` 定义.当您想要注入配置或依赖项时，这样做非常有用.但是，您必须非常小心，它们不会导致太多其他bean的初始化，因为它们必须在应用程序生命周期的早期安装在容器中. （例如，让它们依赖于 `DataSource` 或JPA配置并不是一个好主意.）您可以通过在首次使用而不是初始化时懒惰地初始化bean来解决此类限制.

对于 `Filters` 和 `Servlets` ，您还可以通过添加 `FilterRegistrationBean` 或 `ServletRegistrationBean` 而不是底层组件来添加映射和初始化参数.

> 如果在过滤器注册中未指定 `dispatcherType` ，则使用 `REQUEST` .这与Servlet规范的默认调度程序类型一致.

像任何其他Spring bean一样，您可以定义Servlet过滤器bean的顺序;请务必查看“[the section called “Registering Servlets, Filters, and Listeners as Spring Beans”](boot-features-developing-web-applications.html#boot-features-embedded-container-servlets-filters-listeners-beans)”部分.

#### Disable注册Servlet或过滤器

作为[described earlier](howto-embedded-web-servers.html#howto-add-a-servlet-filter-or-listener-as-spring-bean)，任何 `Servlet` 或 `Filter`  bean都会自动注册到servlet容器.要禁用特定 `Filter` 或 `Servlet` bean的注册，请为其创建注册Bean并将其标记为已禁用，如以下示例所示：

```java
@Bean
public FilterRegistrationBean registration(MyFilter filter) {
	FilterRegistrationBean registration = new FilterRegistrationBean(filter);
	registration.setEnabled(false);
	return registration;
}
```

### 78.10.2使用类路径扫描添加Servlet，过滤器和监听器

`@WebServlet` ， `@WebFilter` 和 `@WebListener` 带注释的类可以通过使用 `@ServletComponentScan` 注释 `@Configuration` 类并指定包含该类的包来自动注册嵌入式servlet容器.要注册的组件.默认情况下， `@ServletComponentScan` 将从带注释的类的包中进行扫描.

## 78.11配置访问日志记录

可以通过各自的命名空间为Tomcat，Undertow和Jetty配置访问日志.

例如，以下设置使用[custom pattern](https://tomcat.apache.org/tomcat-8.5-doc/config/valve.html#Access_Logging)在Tomcat上记录访问权限.

```java
server.tomcat.basedir=my-tomcat
server.tomcat.accesslog.enabled=true
server.tomcat.accesslog.pattern=%t %a "%r" %s (%D ms)
```

> 日志的默认位置是相对于Tomcat基目录的 `logs` 目录.默认情况下， `logs` 目录是临时目录，因此您可能希望修复Tomcat的基目录或使用日志的绝对路径.在前面的示例中，日志在 `my-tomcat/logs` 中相对于应用程序的工作目录可用.

可以以类似的方式配置Undertow的访问日志记录，如以下示例所示：

```java
server.undertow.accesslog.enabled=true
server.undertow.accesslog.pattern=%t %a "%r" %s (%D ms)
```

日志存储在相对于应用程序工作目录的 `logs` 目录中.您可以通过设置 `server.undertow.accesslog.directory` 属性来自定义此位置.

最后，Jetty的访问日志记录也可以配置如下：

```java
server.jetty.accesslog.enabled=true
server.jetty.accesslog.filename=/var/log/jetty-access.log
```

默认情况下，日志会重定向到 `System.err` .有关更多详细信息，请参阅[the Jetty documentation](https://www.eclipse.org/jetty/documentation/9.4.x/configuring-jetty-request-logs.html).

## 78.12在前端代理服务器后面运行

您的应用程序可能需要发送 `302` 重定向或使用绝对链接将内容呈现给自身.在代理后面运行时，调用者需要指向代理的链接，而不是托管应用程序的计算机的物理地址.通常，这种情况是通过与代理的Contract来处理的，代理会添加Headers以告诉后端如何构建自身链接.

如果代理添加常规 `X-Forwarded-For` 和 `X-Forwarded-Proto` 标头（大多数代理服务器都这样做），则应该正确呈现绝对链接，前提是 `server.use-forward-headers` 在 `application.properties` 中设置为 `true` .

> 如果您的应用程序在Cloud Foundry或Heroku中运行，则 `server.use-forward-headers` 属性默认为 `true` .在所有其他实例中，它默认为 `false` .

### 78.12.1自定义Tomcat的代理配置

如果使用Tomcat，还可以配置用于携带“转发”信息的标头名称，如以下示例所示：

```java
server.tomcat.remote-ip-header=x-your-remote-ip-header
server.tomcat.protocol-header=x-your-protocol-header
```

Tomcat还配置了一个默认正则表达式，该表达式匹配要信任的内部代理.默认情况下， `10/8` ， `192.168/16` ， `169.254/16` 和 `127/8` 中的IP地址是可信的.您可以通过向 `application.properties` 添加条目来自定义阀门的配置，如以下示例所示：

```java
server.tomcat.internal-proxies=192\\.168\\.\\d{1,3}\\.\\d{1,3}
```

> 只有在使用属性文件进行配置时才需要双反斜杠.如果使用YAML，则单个反斜杠就足够了，并且与前面示例中显示的值相等的值为 `192\.168\.\d{1,3}\.\d{1,3}` .

> 您可以通过将 `internal-proxies` 设置为空来信任所有代理（但在生产环境中不这样做）.

您可以通过关闭自动关闭（为此，设置 `server.use-forward-headers=false` ）并在 `TomcatServletWebServerFactory`  bean中添加新的阀门实例来完全控制Tomcat的 `RemoteIpValve` 的配置.

## 78.13使用Tomcat启用多个连接器

您可以将 `org.apache.catalina.connector.Connector` 添加到 `TomcatServletWebServerFactory` ，这可以允许多个连接器，包括HTTP和HTTPS连接器，如以下示例所示：

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

## 78.14使用Tomcat的LegacyCookieProcessor

默认情况下，Spring Boot使用的嵌入式Tomcat不支持Cookie格式的“Version 0”，因此您可能会看到以下错误：

```java
java.lang.IllegalArgumentException: An invalid character [32] was present in the Cookie value
```

如果可能的话，您应该考虑将代码更新为仅存储符合以后Cookie规范的值.但是，如果您无法更改cookie的写入方式，则可以将Tomcat配置为使用 `LegacyCookieProcessor` .要切换到 `LegacyCookieProcessor` ，请使用添加 `TomcatContextCustomizer` 的 `WebServerFactoryCustomizer`  bean，如以下示例所示：

```java
@Bean
public WebServerFactoryCustomizer<TomcatServletWebServerFactory> cookieProcessorCustomizer() {
	return (factory) -> factory.addContextCustomizers(
			(context) -> context.setCookieProcessor(new LegacyCookieProcessor()));
}
```

## 78.15使用Undertow启用多个侦听器

将 `UndertowBuilderCustomizer` 添加到 `UndertowServletWebServerFactory` 并向 `Builder` 添加一个侦听器，如以下示例所示：

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

## 78.16使用@ServerEndpoint创建WebSocketendpoints

如果要在使用嵌入式容器的Spring Boot应用程序中使用 `@ServerEndpoint` ，则必须声明单个 `ServerEndpointExporter`   `@Bean` ，如以下示例所示：

```java
@Bean
public ServerEndpointExporter serverEndpointExporter() {
	return new ServerEndpointExporter();
}
```

前面示例中显示的bean使用基础WebSocket容器注册任何 `@ServerEndpoint` 带注释的bean.当部署到独立的servlet容器时，此角色由servlet容器初始化程序执行，并且不需要 `ServerEndpointExporter`  bean.

