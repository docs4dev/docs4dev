## 28.开发Web应用程序

Spring Boot非常适合Web应用程序开发.您可以使用嵌入式Tomcat，Jetty，Undertow或Netty创建自包含的HTTP服务器.大多数Web应用程序使用 `spring-boot-starter-web` 模块快速启动和运行.您还可以选择使用 `spring-boot-starter-webflux` 模块构建响应式Web应用程序.

如果您尚未开发Spring Boot Web应用程序，则可以按照[Getting started](getting-started-first-application.html)部分中的"Hello World!"示例进行操作.

## 28.1“Spring Web MVC框架”

[Spring Web MVC framework](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#mvc)（通常简称为“Spring MVC”）是一个丰富的“模型视图控制器”Web框架. Spring MVC允许您创建特殊的 `@Controller` 或 `@RestController` bean来处理传入的HTTP请求.控制器中的方法通过 `@RequestMapping` 注释映射到HTTP.

以下代码显示了为JSON数据提供服务的典型 `@RestController` ：

```java
@RestController
@RequestMapping(value="/users")
public class MyRestController {

	@RequestMapping(value="/{user}", method=RequestMethod.GET)
	public User getUser(@PathVariable Long user) {
		// ...
	}

	@RequestMapping(value="/{user}/customers", method=RequestMethod.GET)
	List<Customer> getUserCustomers(@PathVariable Long user) {
		// ...
	}

	@RequestMapping(value="/{user}", method=RequestMethod.DELETE)
	public User deleteUser(@PathVariable Long user) {
		// ...
	}

}
```

Spring MVC是核心Spring Framework的一部分，详细信息可在[reference documentation](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#mvc)中找到.在[spring.io/guides](https://spring.io/guides)还有几个涵盖Spring MVC的指南.

### 28.1.1 Spring MVC自动配置

Spring Boot为Spring MVC提供自动配置，适用于大多数应用程序.

自动配置在Spring的默认值之上添加了以下功能：

- 包含 `ContentNegotiatingViewResolver` 和 `BeanNameViewResolver`  bean.

- Support用于提供静态资源，包括对WebJars的支持（涵盖[later in this document](boot-features-developing-web-applications.html#boot-features-spring-mvc-static-content)））.

-   `Converter` ， `GenericConverter` 和 `Formatter`  bean的自动注册.

- Support for  `HttpMessageConverters` （涵盖[later in this document](boot-features-developing-web-applications.html#boot-features-spring-mvc-message-converters)）.

- 自动注册 `MessageCodesResolver` （涵盖[later in this document](boot-features-developing-web-applications.html#boot-features-spring-message-codes)）.

- Static  `index.html` 支持.

- Custom  `Favicon` 支持（涵盖[later in this document](boot-features-developing-web-applications.html#boot-features-spring-mvc-favicon)）.

- 自动使用 `ConfigurableWebBindingInitializer`  bean（涵盖[later in this document](boot-features-developing-web-applications.html#boot-features-spring-mvc-web-binding-initializer)）.

如果要保留Spring Boot MVC功能并且想要添加其他[MVC configuration](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#mvc)（拦截器，格式化程序，视图控制器和其他功能），可以添加自己的 `@Configuration` 类 `WebMvcConfigurer` 但 **without**  `@EnableWebMvc` .如果要提供 `RequestMappingHandlerMapping` ， `RequestMappingHandlerAdapter` 或 `ExceptionHandlerExceptionResolver` 的自定义实例，可以声明 `WebMvcRegistrationsAdapter` 实例以提供此类组件.

如果您想完全控制Spring MVC，可以添加自己的 `@Configuration` 注释 `@EnableWebMvc` .

### 28.1.2 HttpMessageConverters

Spring MVC使用 `HttpMessageConverter` 接口来转换HTTP请求和响应.明智的默认值包含在开箱即用中.例如，对象可以自动转换为JSON（通过使用Jackson库）或XML（如果可用，则使用Jackson XML扩展，或者如果Jackson XML扩展不可用，则使用JAXB）.默认情况下，字符串在 `UTF-8` 中编码.

如果需要添加或自定义转换器，可以使用Spring Boot的 `HttpMessageConverters` 类，如下面的清单所示：

```java
import org.springframework.boot.autoconfigure.web.HttpMessageConverters;
import org.springframework.context.annotation.*;
import org.springframework.http.converter.*;

@Configuration
public class MyConfiguration {

	@Bean
	public HttpMessageConverters customConverters() {
		HttpMessageConverter<?> additional = ...
		HttpMessageConverter<?> another = ...
		return new HttpMessageConverters(additional, another);
	}

}
```

上下文中存在的任何 `HttpMessageConverter`  bean都将添加到转换器列表中.您也可以以相同的方式覆盖默认转换器.

### 28.1.3自定义JSON序列化程序和反序列化程序

如果您使用Jackson序列化和反序列化JSON数据，您可能需要编写自己的 `JsonSerializer` 和 `JsonDeserializer` 类.自定义序列化程序通常是[registered with Jackson through a module](https://github.com/FasterXML/jackson-docs/wiki/JacksonHowToCustomSerializers)，但Spring Boot提供了另一种 `@JsonComponent` 注释，可以更容易地直接注册Spring Beans.

你可以使用 `@JsonComponent` 直接在 `JsonSerializer` 或 `JsonDeserializer` 实现上进行注释.您还可以在包含序列化程序/反序列化程序作为内部类的类上使用它，如以下示例所示：

```java
import java.io.*;
import com.fasterxml.jackson.core.*;
import com.fasterxml.jackson.databind.*;
import org.springframework.boot.jackson.*;

@JsonComponent
public class Example {

	public static class Serializer extends JsonSerializer<SomeObject> {
		// ...
	}

	public static class Deserializer extends JsonDeserializer<SomeObject> {
		// ...
	}

}
```

`ApplicationContext` 中的所有 `@JsonComponent`  bean都会自动在Jackson注册.因为 `@JsonComponent` 是使用 `@Component` 元注释的，所以通常的组件扫描规则适用.

Spring Boot还提供[JsonObjectSerializer](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jackson/JsonObjectSerializer.java)和[JsonObjectDeserializer](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jackson/JsonObjectDeserializer.java)基类，在序列化对象时提供标准Jackson版本的有用替代方法.有关详细信息，请参阅Javadoc中的[JsonObjectSerializer](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/api/org/springframework/boot/jackson/JsonObjectSerializer.html)和[JsonObjectDeserializer](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/api/org/springframework/boot/jackson/JsonObjectDeserializer.html).

### 28.1.4 MessageCodesResolver

Spring MVC有一个生成错误代码的策略，用于从绑定错误中呈现错误消息： `MessageCodesResolver` .如果设置 `spring.mvc.message-codes-resolver.format` 属性 `PREFIX_ERROR_CODE` 或 `POSTFIX_ERROR_CODE` ，Spring Boot会为您创建一个（请参阅[DefaultMessageCodesResolver.Format](https://docs.spring.io/spring/docs/5.1.2.RELEASE/javadoc-api/org/springframework/validation/DefaultMessageCodesResolver.Format.html)中的枚举）.

### 28.1.5静态内容

默认情况下，Spring Boot从类路径中的 `/static` （或 `/public` 或 `/resources` 或 `/META-INF/resources` ）目录或 `ServletContext` 的根目录中提供静态内容.它使用Spring MVC中的 `ResourceHttpRequestHandler` ，以便您可以通过添加自己的 `WebMvcConfigurer` 并覆盖 `addResourceHandlers` 方法来修改该行为.

在独立的Web应用程序中，容器中的默认servlet也会启用并充当回退，如果Spring决定不处理它，则从 `ServletContext` 的根目录提供内容.大多数情况下，这不会发生（除非您修改默认的MVC配置），因为Spring总是可以通过 `DispatcherServlet` 处理请求.

默认情况下，资源映射到 `/**` ，但您可以使用 `spring.mvc.static-path-pattern` 属性对其进行调整.例如，将所有资源重新定位到 `/resources/**` 可以实现如下：

```java
spring.mvc.static-path-pattern=/resources/**
```

您还可以使用 `spring.resources.static-locations` 属性自定义静态资源位置（将默认值替换为目录位置列表）.根Servlet上下文路径 `"/"` 也会自动添加为位置.

除了前面提到的“标准”静态资源位置之外，还为[Webjars content](https://www.webjars.org/)制作了一个特例.如果_4102_中的路径以Webjars格式打包，那么它们将从jar文件中提供.

> 如果您的应用程序打包为jar，请不要使用 `src/main/webapp` 目录.虽然这个目录是一个通用的标准，但是它与war包装一起使用 **only** ，如果你生成一个jar，它会被大多数构建工具默默忽略.

Spring Boot还支持Spring MVC提供的高级资源处理功能，允许使用缓存破坏静态资源或使用与Webjars无关的URL.

要为Webjars使用版本无关的URL，请添加 `webjars-locator-core` 依赖项.然后声明你的Webjar.以jQuery为例，添加 `"/webjars/jquery/jquery.min.js"` 会导致 `"/webjars/jquery/x.y.z/jquery.min.js"` .其中 `x.y.z` 是Webjar版本.

> 如果你使用JBoss，你需要声明 `webjars-locator-jboss-vfs` 依赖而不是 `webjars-locator-core` .否则，所有Webjars都将解析为 `404` .

要使用缓存清除，以下配置会为所有静态资源配置缓存清除解决方案，从而在URL中有效添加内容哈希（例如 `<link href="/css/spring-2a2d595e6ed9a0b24f027f2b63b134d6.css"/>` ）：

```java
spring.resources.chain.strategy.content.enabled=true
spring.resources.chain.strategy.content.paths=/**
```

_0015由于为Thymeleaf和FreeMarker自动配置了 `ResourceUrlEncodingFilter` ，因此在运行时可以在模板中重写资源.您应该在使用JSP时手动声明此过滤器.目前不支持其他模板引擎，但可以使用自定义模板宏/帮助程序和[ResourceUrlProvider](https://docs.spring.io/spring/docs/5.1.2.RELEASE/javadoc-api/org/springframework/web/servlet/resource/ResourceUrlProvider.html).

使用（例如）JavaScript模块加载器动态加载资源时，不能重命名文件.这就是为什么其他策略也得到支持并可以合并的原因. “固定”策略在URL中添加静态版本字符串而不更改文件名，如以下示例所示：

```java
spring.resources.chain.strategy.content.enabled=true
spring.resources.chain.strategy.content.paths=/**
spring.resources.chain.strategy.fixed.enabled=true
spring.resources.chain.strategy.fixed.paths=/js/lib/
spring.resources.chain.strategy.fixed.version=v12
```

使用此配置，位于 `"/js/lib/"` 下的JavaScript模块使用固定版本控制策略（ `"/v12/js/lib/mymodule.js"` ），而其他资源仍使用内容1（ `<link href="/css/spring-2a2d595e6ed9a0b24f027f2b63b134d6.css"/>` ）.

有关更多支持的选项，请参阅[ResourceProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ResourceProperties.java).

> 这个功能已在专用[blog post](https://spring.io/blog/2014/07/24/spring-framework-4-1-handling-static-web-resources)和Spring Framework的[reference documentation](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#mvc-config-static-resources)中进行了详细描述.

### 28.1.6欢迎页面

Spring Boot支持静态和模板化欢迎页面.它首先在配置的静态内容位置中查找 `index.html` 文件.如果找不到，则查找 `index` 模板.如果找到任何一个，它将自动用作应用程序的欢迎页面.

### 28.1.7 Custom Favicon

Spring Boot在配置的静态内容位置和类路径的根（按此顺序）中查找 `favicon.ico` .如果存在这样的文件，它将自动用作应用程序的favicon.

### 28.1.8路径匹配和内容协商

Spring MVC可以通过查看请求路径并将其与应用程序中定义的映射（例如，Controller方法上的 `@GetMapping` 注释）相匹配，将传入的HTTP请求映射到处理程序.

Spring Boot默认选择禁用后缀模式匹配，这意味着像 `"GET /projects/spring-boot.json"` 这样的请求将不会与 `@GetMapping("/projects/spring-boot")` 映射匹配.这被认为是[best practice for Spring MVC applications](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#mvc-ann-requestmapping-suffix-pattern-match).这个功能是对于没有发送正确"Accept"请求标头的HTTP客户端，过去主要有用;我们需要确保将正确的内容类型发送给客户端.如今，内容协商更加可靠.

还有其他方法可以处理不一致发送正确的"Accept"请求标头的HTTP客户端.我们可以使用查询参数来确保像 `"GET /projects/spring-boot?format=json"` 这样的请求将映射到 `@GetMapping("/projects/spring-boot")` ，而不是使用后缀匹配：

```java
spring.mvc.contentnegotiation.favor-parameter=true

# We can change the parameter name, which is "format" by default:
# spring.mvc.contentnegotiation.parameter-name=myparam

# We can also register additional file extensions/media types with:
spring.mvc.contentnegotiation.media-types.markdown=text/markdown
```

如果您了解警告并仍希望您的应用程序使用后缀模式匹配，则需要以下配置：

```java
spring.mvc.contentnegotiation.favor-path-extension=true
spring.mvc.pathmatch.use-suffix-pattern=true
```

或者，不是打开所有后缀模式，而是仅支持已注册的后缀模式更安全：

```java
spring.mvc.contentnegotiation.favor-path-extension=true
spring.mvc.pathmatch.use-registered-suffix-pattern=true

# You can also register additional file extensions/media types with:
# spring.mvc.contentnegotiation.media-types.adoc=text/asciidoc
```

### 28.1.9 ConfigurableWebBindingInitializer

Spring MVC使用 `WebBindingInitializer` 为特定请求初始化 `WebDataBinder` .如果您创建自己的 `ConfigurableWebBindingInitializer`   `@Bean` ，Spring Boot会自动配置Spring MVC以使用它.

### 28.1.10模板引擎

除REST Web服务外，您还可以使用Spring MVC来提供动态HTML内容. Spring MVC支持各种模板技术，包括Thymeleaf，FreeMarker和JSP.此外，许多其他模板引擎包括他们自己的Spring MVC集成.

Spring Boot包含对以下模板引擎的自动配置支持：

- [FreeMarker](https://freemarker.apache.org/docs/)

- [Groovy](http://docs.groovy-lang.org/docs/next/html/documentation/template-engines.html#_the_markuptemplateengine)

- [Thymeleaf](http://www.thymeleaf.org)

- [Mustache](https://mustache.github.io/)

> 如果可能，应该避免使用JSP.将它们与嵌入式servlet容器一起使用时，有几个[known limitations](boot-features-developing-web-applications.html#boot-features-jsp-limitations).

当您使用其中一个模板引擎和默认配置时，您的模板将从 `src/main/resources/templates` 自动获取.

> 根据您运行应用程序的方式，IntelliJ IDEA以不同方式对类路径进行排序.从主方法在IDE中运行应用程序会产生与使用Maven或Gradle或其打包的jar运行应用程序时不同的顺序.这可能导致Spring Boot无法在类路径中找到模板.如果遇到此问题，可以在IDE中重新排序类路径，以便首先放置模块的类和资源.或者，您可以配置模板前缀以搜索类路径上的每个 `templates` 目录，如下所示： `classpath*:/templates/` .

### 28.1.11错误处理

默认情况下，Spring Boot提供 `/error` 映射，以合理的方式处理所有错误，并在servlet容器中注册为“全局”错误页面.对于计算机客户端，它会生成一个JSON响应，其中包含错误，HTTP状态和异常消息的详细信息.对于浏览器客户端，有一个“whitelabel”错误视图，以HTML格式呈现相同的数据（要自定义它，添加一个解析为 `error` 的 `View` ）.要完全替换默认行为，可以实现 `ErrorController` 并注册该类型的bean定义，或者添加 `ErrorAttributes` 类型的bean以使用现有机制但替换内容.

>   `BasicErrorController` 可以用作自定义 `ErrorController` 的基类.如果要为新内容类型添加处理程序，则此功能特别有用（默认情况下，具体处理 `text/html` 并为其他所有内容提供后备）.为此，请扩展 `BasicErrorController` ，添加具有 `produces` 属性的 `@RequestMapping` 的公共方法，并创建新类型的bean.

您还可以定义一个使用 `@ControllerAdvice` 注释的类，以自定义要为特定控制器和/或异常类型返回的JSON文档，如以下示例所示：

```java
@ControllerAdvice(basePackageClasses = AcmeController.class)
public class AcmeControllerAdvice extends ResponseEntityExceptionHandler {

	@ExceptionHandler(YourException.class)
	@ResponseBody
	ResponseEntity<?> handleControllerException(HttpServletRequest request, Throwable ex) {
		HttpStatus status = getStatus(request);
		return new ResponseEntity<>(new CustomErrorType(status.value(), ex.getMessage()), status);
	}

	private HttpStatus getStatus(HttpServletRequest request) {
		Integer statusCode = (Integer) request.getAttribute("javax.servlet.error.status_code");
		if (statusCode == null) {
			return HttpStatus.INTERNAL_SERVER_ERROR;
		}
		return HttpStatus.valueOf(statusCode);
	}

}
```

在前面的示例中，如果 `YourException` 由与 `AcmeController` 在同一个包中定义的控制器抛出，则使用 `CustomErrorType`  POJO的JSON表示而不是 `ErrorAttributes` 表示.

#### Custom错误页面

如果要显示给定状态代码的自定义HTML错误页面，可以将文件添加到 `/error` 文件夹.错误页面可以是静态HTML（即，添加到任何静态资源文件夹下），也可以使用模板构建.文件名应该是确切的状态代码或系列掩码.

例如，要将 `404` 映射到静态HTML文件，您的文件夹结构将如下所示：

```xml
src/
+- main/
+- java/
|   + <source code>
+- resources/
+- public/
+- error/
|   +- 404.html
+- <other public assets>
```

要使用FreeMarker模板映射所有 `5xx` 错误，您的文件夹结构如下：

```xml
src/
+- main/
+- java/
|   + <source code>
+- resources/
+- templates/
+- error/
|   +- 5xx.ftl
+- <other templates>
```

对于更复杂的映射，您还可以添加实现 `ErrorViewResolver` 接口的bean，如以下示例所示：

```java
public class MyErrorViewResolver implements ErrorViewResolver {

	@Override
	public ModelAndView resolveErrorView(HttpServletRequest request,
			HttpStatus status, Map<String, Object> model) {
		// Use the request or status to optionally return a ModelAndView
		return ...
	}

}
```

您还可以使用常规的Spring MVC功能，例如[@ExceptionHandler methods](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#mvc-exceptionhandlers)和[@ControllerAdvice](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#mvc-ann-controller-advice).然后 `ErrorController` 会接收任何未处理的异常.

在Spring MVC之外的
#### Mapping错误页面

对于不使用Spring MVC的应用程序，可以使用 `ErrorPageRegistrar` 接口直接注册 `ErrorPages` .这种抽象直接与底层嵌入式servlet容器一起工作，即使你没有Spring MVC  `DispatcherServlet` 也可以工作.

```java
@Bean
public ErrorPageRegistrar errorPageRegistrar(){
	return new MyErrorPageRegistrar();
}

// ...

private static class MyErrorPageRegistrar implements ErrorPageRegistrar {

	@Override
	public void registerErrorPages(ErrorPageRegistry registry) {
		registry.addErrorPages(new ErrorPage(HttpStatus.BAD_REQUEST, "/400"));
	}

}
```

> If您注册与最终由 `Filter` 正在处理的路径的 `ErrorPage` （如某些非Spring的web框架，如新泽西州和检票常见），那么 `Filter` 必须被明确地注册为 `ERROR` 调度，如图下列例：

```java
@Bean
public FilterRegistrationBean myFilter() {
	FilterRegistrationBean registration = new FilterRegistrationBean();
	registration.setFilter(new MyFilter());
	...
	registration.setDispatcherTypes(EnumSet.allOf(DispatcherType.class));
	return registration;
}
```

请注意，默认 `FilterRegistrationBean` 不包括 `ERROR` 调度程序类型.

小心：当部署到servlet容器时，Spring Boot使用其错误页面过滤器将具有错误状态的请求转发到相应的错误页面.如果尚未提交响应，则只能将请求转发到正确的错误页面.缺省情况下，WebSphere Application Server 8.0及更高版本在成功完成servlet的服务方法后提交响应.您应该通过将 `com.ibm.ws.webcontainer.invokeFlushAfterService` 设置为 `false` 来禁用此行为.

### 28.1.12Spring天的HATEOAS

如果您开发使用超媒体的RESTful API，Spring Boot为Spring HATEOAS提供自动配置，适用于大多数应用程序.自动配置取代了使用 `@EnableHypermediaSupport` 的需要，并注册了许多bean以简化构建基于超媒体的应用程序，包括 `LinkDiscoverers` （用于客户端支持）和 `ObjectMapper` 配置为正确地将响应编组到所需的表示中.  `ObjectMapper` 是通过设置各种 `spring.jackson.*` 属性自定义的，如果存在，则由 `Jackson2ObjectMapperBuilder`  bean设置.

您可以使用 `@EnableHypermediaSupport` 控制Spring HATEOAS的配置.请注意，这样做会禁用前面描述的 `ObjectMapper` 自定义.

### 28.1.13 CORS支持

[Cross-origin resource sharing](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)（CORS）是由[most browsers](https://caniuse.com/#feat=cors)实现的[W3C specification](https://www.w3.org/TR/cors/)，它允许您以灵活的方式指定授权何种类型的跨域请求，而不是使用一些不太安全且功能较弱的方法，如IFRAME或JSONP.

从版本4.2开始，Spring MVC [supports CORS](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#cors).在Spring Boot应用程序中使用[controller method CORS configuration](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#controller-method-cors-configuration)和[@CrossOrigin](https://docs.spring.io/spring/docs/5.1.2.RELEASE/javadoc-api/org/springframework/web/bind/annotation/CrossOrigin.html)注释不需要任何特定配置.可以通过使用自定义 `addCorsMappings(CorsRegistry)` 方法注册 `WebMvcConfigurer` bean来定义[Global CORS configuration](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#global-cors-configuration)，如以下示例所示：

```java
@Configuration
public class MyConfiguration {

	@Bean
	public WebMvcConfigurer corsConfigurer() {
		return new WebMvcConfigurer() {
			@Override
			public void addCorsMappings(CorsRegistry registry) {
				registry.addMapping("/api/**");
			}
		};
	}
}
```

## 28.2“Spring WebFlux框架”

Spring WebFlux是Spring Framework 5.0中引入的新的反应式Web框架.与Spring MVC不同，它不需要Servlet API，完全异步且无阻塞，并通过[the Reactor project](https://projectreactor.io/)实现[Reactive Streams](http://www.reactive-streams.org/)规范.

Spring WebFlux有两种版本：基于功能和注释.基于注释的注释非常接近Spring MVC模型，如以下示例所示：

```java
@RestController
@RequestMapping("/users")
public class MyRestController {

	@GetMapping("/{user}")
	public Mono<User> getUser(@PathVariable Long user) {
		// ...
	}

	@GetMapping("/{user}/customers")
	public Flux<Customer> getUserCustomers(@PathVariable Long user) {
		// ...
	}

	@DeleteMapping("/{user}")
	public Mono<User> deleteUser(@PathVariable Long user) {
		// ...
	}

}
```

“WebFlux.fn”是功能变体，它将路由配置与请求的实际处理分开，如以下示例所示：

```java
@Configuration
public class RoutingConfiguration {

	@Bean
	public RouterFunction<ServerResponse> monoRouterFunction(UserHandler userHandler) {
		return route(GET("/{user}").and(accept(APPLICATION_JSON)), userHandler::getUser)
				.andRoute(GET("/{user}/customers").and(accept(APPLICATION_JSON)), userHandler::getUserCustomers)
				.andRoute(DELETE("/{user}").and(accept(APPLICATION_JSON)), userHandler::deleteUser);
	}

}

@Component
public class UserHandler {

	public Mono<ServerResponse> getUser(ServerRequest request) {
		// ...
	}

	public Mono<ServerResponse> getUserCustomers(ServerRequest request) {
		// ...
	}

	public Mono<ServerResponse> deleteUser(ServerRequest request) {
		// ...
	}
}
```

WebFlux是Spring Framework的一部分，详细信息可在[reference documentation](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web-reactive.html#webflux-fn)中找到.

> 您可以根据需要定义尽可能多的 `RouterFunction`  bean来模块化路由器的定义.如果需要应用优先级，可以订购Bean.

首先，将 `spring-boot-starter-webflux` 模块添加到您的应用程序中.

> 在应用程序中添加 `spring-boot-starter-web` 和 `spring-boot-starter-webflux` 模块会导致Spring Boot自动配置Spring MVC，而不是WebFlux.选择此行为是因为许多Spring开发人员将 `spring-boot-starter-webflux` 添加到他们的Spring MVC应用程序中以使用reactive  `WebClient` .您仍然可以通过将所选应用程序类型设置为 `SpringApplication.setWebApplicationType(WebApplicationType.REACTIVE)` 来强制执行您的选择.

### 28.2.1 Spring WebFlux自动配置

Spring Boot为Spring WebFlux提供自动配置，适用于大多数应用程序.

自动配置在Spring的默认值之上添加了以下功能：

- 为 `HttpMessageReader` 和 `HttpMessageWriter` 实例配置编解码器（描述为[later in this document](boot-features-developing-web-applications.html#boot-features-webflux-httpcodecs)）.

- Support用于提供静态资源，包括对WebJars的支持（描述为[later in this document](boot-features-developing-web-applications.html#boot-features-spring-mvc-static-content)）.

如果你想保留Spring Boot WebFlux功能并且想要添加额外的[WebFlux configuration](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#web-reactive)，你可以添加自己的 `@Configuration` 类 `WebFluxConfigurer` 但是 **without**  `@EnableWebFlux` .

如果要完全控制Spring WebFlux，可以使用 `@EnableWebFlux` 添加自己的 `@Configuration` 注释.

### 28.2.2使用HttpMessageReaders和HttpMessageWriters的HTTP编解码器

Spring WebFlux使用 `HttpMessageReader` 和 `HttpMessageWriter` 接口来转换HTTP请求和响应.通过查看类路径中可用的库，它们配置为 `CodecConfigurer` 以具有合理的默认值.

Spring Boot通过使用 `CodecCustomizer` 实例进一步自定义.例如， `spring.jackson.*` 配置键应用于Jackson编解码器.

如果需要添加或自定义编解码器，可以创建自定义 `CodecCustomizer` 组件，如以下示例所示：

```java
import org.springframework.boot.web.codec.CodecCustomizer;

@Configuration
public class MyConfiguration {

	@Bean
	public CodecCustomizer myCodecCustomizer() {
		return codecConfigurer -> {
			// ...
		}
	}

}
```

你也可以利用[Boot’s custom JSON serializers and deserializers](boot-features-developing-web-applications.html#boot-features-json-components).

### 28.2.3静态内容

默认情况下，Spring Boot从类路径中名为 `/static` （或 `/public` 或 `/resources` 或 `/META-INF/resources` ）的目录中提供静态内容.它使用Spring WebFlux中的 `ResourceWebHandler` ，以便您可以通过添加自己的 `WebFluxConfigurer` 并覆盖 `addResourceHandlers` 方法来修改该行为.

默认情况下，资源映射到 `/**` ，但您可以通过设置 `spring.webflux.static-path-pattern` 属性来调整它.例如，将所有资源重新定位到 `/resources/**` 可以实现如下：

```java
spring.webflux.static-path-pattern=/resources/**
```

您还可以使用 `spring.resources.static-locations` 自定义静态资源位置.这样做会将默认值替换为目录位置列表.如果这样做，默认的欢迎页面检测将切换到您的自定义位置.因此，如果您在启动时的任何位置都有 `index.html` ，那么它就是应用程序的主页.

除了前面列出的“标准”静态资源位置之外，还为[Webjars content](https://www.webjars.org/)制作了一个特例.如果它们以Webjars格式打包，那么在jar文件中提供 `/webjars/**` 中具有路径的任何资源.

> Spring WebFlux应用程序并不严格依赖于Servlet API，因此它们不能作为war文件部署，也不能使用 `src/main/webapp` 目录.

### 28.2.4模板引擎

除REST Web服务外，您还可以使用Spring WebFlux来提供动态HTML内容. Spring WebFlux支持各种模板技术，包括Thymeleaf，FreeMarker和Mustache.

Spring Boot包含对以下模板引擎的自动配置支持：

- [FreeMarker](https://freemarker.apache.org/docs/)

- [Thymeleaf](http://www.thymeleaf.org)

- [Mustache](https://mustache.github.io/)

当您使用其中一个模板引擎和默认配置时，您的模板将从 `src/main/resources/templates` 自动获取.

### 28.2.5错误处理

Spring Boot提供 `WebExceptionHandler` ，以合理的方式处理所有错误.它在处理顺序中的位置紧接在WebFlux提供的处理程序之前，这被认为是最后的.对于计算机客户端，它会生成一个JSON响应，其中包含错误，HTTP状态和异常消息的详细信息.对于浏览器客户端，有一个“whitelabel”错误处理程序，它以HTML格式呈现相同的数据.您还可以提供自己的HTML模板来显示错误（请参阅[next section](boot-features-developing-web-applications.html#boot-features-webflux-error-handling-custom-error-pages)）.

自定义此功能的第一步通常涉及使用现有机制，但替换或扩充错误内容.为此，您可以添加 `ErrorAttributes` 类型的bean.

要更改错误处理行为，可以实现 `ErrorWebExceptionHandler` 并注册该类型的bean定义.因为 `WebExceptionHandler` 是非常低级的，所以Spring Boot还提供了一个方便的 `AbstractErrorWebExceptionHandler` 来让你以WebFlux功能方式处理错误，如下例所示：

```java
public class CustomErrorWebExceptionHandler extends AbstractErrorWebExceptionHandler {

	// Define constructor here

	@Override
	protected RouterFunction<ServerResponse> getRoutingFunction(ErrorAttributes errorAttributes) {

		return RouterFunctions
				.route(aPredicate, aHandler)
				.andRoute(anotherPredicate, anotherHandler);
	}

}
```

要获得更完整的图片，您还可以直接子类化 `DefaultErrorWebExceptionHandler` 并覆盖特定方法.

#### Custom错误页面

如果要显示给定状态代码的自定义HTML错误页面，可以将文件添加到 `/error` 文件夹.错误页面可以是静态HTML（即，添加到任何静态资源文件夹下）或使用模板构建.文件名应该是确切的状态代码或系列掩码.

例如，要将 `404` 映射到静态HTML文件，您的文件夹结构将如下所示：

```xml
src/
+- main/
+- java/
|   + <source code>
+- resources/
+- public/
+- error/
|   +- 404.html
+- <other public assets>
```

要使用Mustache模板映射所有 `5xx` 错误，您的文件夹结构如下：

```xml
src/
+- main/
+- java/
|   + <source code>
+- resources/
+- templates/
+- error/
|   +- 5xx.mustache
+- <other templates>
```

### 28.2.6网页过滤器

Spring WebFlux提供了一个 `WebFilter` 接口，可以实现过滤HTTP请求 - 响应交换.在应用程序上下文中找到的 `WebFilter`  bean将自动用于过滤每个交换.

如果过滤器的顺序很重要，则可以实现 `Ordered` 或使用 `@Order` 进行注释. Spring Boot自动配置可以为您配置Web过滤器.执行此操作时，将使用下表中显示的订单：

| Web过滤器|订购|
| ---- | ---- |
|  `MetricsWebFilter`  |  `Ordered.HIGHEST_PRECEDENCE + 1`  |
|  `WebFilterChainProxy` （Spring Security）|  `-100`  |
|  `HttpTraceWebFilter`  |  `Ordered.LOWEST_PRECEDENCE - 10`  |

## 28.3 JAX-RS和Jersey

如果您更喜欢RESTendpoints的JAX-RS编程模型，则可以使用其中一个可用的实现而不是Spring MVC. [Jersey](https://jersey.github.io/)和[Apache CXF](https://cxf.apache.org/)开箱即用. CXF要求您在应用程序上下文中将 `Servlet` 或 `Filter` 注册为 `@Bean` . Jersey有一些原生的Spring支持，所以我们还在Spring Boot中为它提供了自动配置支持以及一个启动器.

要开始使用Jersey，请将 `spring-boot-starter-jersey` 作为依赖项包含在内，然后您需要一个 `@Bean` 类型 `ResourceConfig` ，在其中注册所有endpoints，如以下示例所示：

```java
@Component
public class JerseyConfig extends ResourceConfig {

	public JerseyConfig() {
		register(Endpoint.class);
	}

}
```

> Jersey对扫描可执行档案的支持相当有限.例如，在运行可执行war文件时，它无法扫描 `WEB-INF/classes` 中找到的包中的endpoints.为避免此限制，不应使用 `packages` 方法，并且应使用 `register` 方法单独注册endpoints，如上例所示.

对于更高级的自定义，您还可以注册实现 `ResourceConfigCustomizer` 的任意数量的bean.

所有已注册的endpoints都应为 `@Components` ，并带有HTTP资源注释（ `@GET` 和其他），如以下示例所示：

```java
@Component
@Path("/hello")
public class Endpoint {

	@GET
	public String message() {
		return "Hello";
	}

}
```

由于 `Endpoint` 是Spring  `@Component` ，因此它的生命周期由Spring管理，您可以使用 `@Autowired` 注释注入依赖项并使用 `@Value` 注释注入外部配置.默认情况下，Jersey servlet已注册并映射到 `/*` .您可以通过将 `@ApplicationPath` 添加到 `ResourceConfig` 来更改映射.

默认情况下，Jersey被设置为 `@Bean` 类型为 `ServletRegistrationBean` 的名为 `jerseyServletRegistration` 的Servlet.默认情况下，servlet会被懒惰地初始化，但您可以通过设置 `spring.jersey.servlet.load-on-startup` 来自定义该行为.您可以通过创建一个来禁用或覆盖该bean你自己的名字相同.您也可以通过设置 `spring.jersey.type=filter` 来使用过滤器而不是servlet（在这种情况下， `@Bean` 要替换或覆盖是 `jerseyFilterRegistration` ）.过滤器有 `@Order` ，您可以使用 `spring.jersey.filter.order` 进行设置.通过使用 `spring.jersey.init.*` 指定属性映射，可以为servlet和过滤器注册提供init参数.

有一个[Jersey sample](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-samples/spring-boot-sample-jersey)，以便您可以看到如何设置.

## 28.4嵌入式Servlet容器支持

Spring Boot包括对嵌入式[Tomcat](https://tomcat.apache.org/)，[Jetty](https://www.eclipse.org/jetty/)和[Undertow](http://undertow.io/)服务器的支持.大多数开发人员使用适当的“Starter”来获取完全配置的实例.默认情况下，嵌入式服务器在端口 `8080` 上侦听HTTP请求.

> 如果选择在[CentOS](https://www.centos.org/)上使用Tomcat，请注意，默认情况下，临时目录用于存储已编译的JSP，文件上载等.应用程序运行时， `tmpwatch` 可能会删除此目录，从而导致失败.要避免此行为，您可能希望自定义 `tmpwatch` 配置，以便不删除 `tomcat.*` 目录或配置 `server.tomcat.basedir` ，以便嵌入式Tomcat使用不同的位置.

### 28.4.1 Servlet，过滤器和监听器

使用嵌入式servlet容器时，可以通过使用Spring bean或扫描Servlet组件，从Servlet规范中注册servlet，过滤器和所有侦听器（例如 `HttpSessionListener` ）.

#### 将Servlet，过滤器和监听器注册为Spring Bean

作为Spring bean的任何 `Servlet` ， `Filter` 或servlet  `*Listener` 实例都是在嵌入式容器中注册的.如果要在配置期间引用 `application.properties` 中的值，这可能特别方便.

默认情况下，如果上下文仅包含一个Servlet，则它将映射到 `/` .对于多个servlet bean，bean名称用作路径前缀.过滤器映射到 `/*` .

如果基于约定的映射不够灵活，则可以使用 `ServletRegistrationBean` ， `FilterRegistrationBean` 和 `ServletListenerRegistrationBean` 类进行完全控制.

Spring Boot附带了许多可以定义Filter bean的自动配置.以下是过滤器及其各自顺序的一些示例（较低的顺序值表示较高的优先级）：

| Servlet过滤器|订购|
| ---- | ---- |
|  `OrderedCharacterEncodingFilter`  |  `Ordered.HIGHEST_PRECEDENCE`  |
|  `WebMvcMetricsFilter`  |  `Ordered.HIGHEST_PRECEDENCE + 1`  |
|  `ErrorPageFilter`  |  `Ordered.HIGHEST_PRECEDENCE + 1`  |
|  `HttpTraceFilter`  |  `Ordered.LOWEST_PRECEDENCE - 10`  |

将过滤 beans无序放置通常是安全的.

如果需要特定的顺序，则应避免配置在 `Ordered.HIGHEST_PRECEDENCE` 处读取请求正文的过滤器，因为它可能违反应用程序的字符编码配置.如果Servlet过滤器包装请求，则应使用小于或等于 `OrderedFilter.REQUEST_WRAPPER_FILTER_MAX_ORDER` 的顺序进行配置.

### 28.4.2 Servlet上下文初始化

嵌入式servlet容器不直接执行Servlet 3.0  `javax.servlet.ServletContainerInitializer` 接口或Spring的 `org.springframework.web.WebApplicationInitializer` 接口.这是一项有意的设计决策，旨在降低设计在战争中运行的第三方库可能会破坏Spring Boot应用程序的风险.

如果需要在Spring Boot应用程序中执行servlet上下文初始化，则应注册实现 `org.springframework.boot.web.servlet.ServletContextInitializer` 接口的bean.单个 `onStartup` 方法提供对 `ServletContext` 的访问，如果需要，可以轻松地用作现有 `WebApplicationInitializer` 的适配器.

#### Scanning for Servlets，Filters和listeners

使用嵌入式容器时，可以使用 `@ServletComponentScan` 启用使用 `@WebServlet` ， `@WebFilter` 和 `@WebListener` 注释的类的自动注册.

>  `@ServletComponentScan` 在独立容器中没有任何效果，而是使用容器的内置发现机制.

### 28.4.3 ServletWebServerApplicationContext

在引擎盖下，Spring Boot使用不同类型的 `ApplicationContext` 来支持嵌入式servlet容器.  `ServletWebServerApplicationContext` 是一种特殊类型的 `WebApplicationContext` ，它通过搜索单个 `ServletWebServerFactory`  bean来引导自己.通常自动配置 `TomcatServletWebServerFactory` ， `JettyServletWebServerFactory` 或 `UndertowServletWebServerFactory` .

> 您通常不需要了解这些实现类.大多数应用程序都是自动配置的，并且代表您创建了相应的 `ApplicationContext` 和 `ServletWebServerFactory` .

### 28.4.4自定义嵌入式Servlet容器

可以使用Spring  `Environment` 属性配置公共servlet容器设置.通常，您将在 `application.properties` 文件中定义属性.

常用服务器设置包括：

- Network设置：侦听传入HTTP请求的端口（ `server.port` ），绑定到 `server.address` 的接口地址，依此类推.

- Session设置：会话是持久性的（ `server.servlet.session.persistence` ），会话超时（ `server.servlet.session.timeout` ），会话数据的位置（ `server.servlet.session.store-dir` ）和会话cookie配置（ `server.servlet.session.cookie.*` ）.

- Error管理：错误页面的位置（ `server.error.path` ）等.

- [SSL](howto-embedded-web-servers.html#howto-configure-ssl)

- [HTTP compression](howto-embedded-web-servers.html#how-to-enable-http-response-compression)

Spring Boot尽可能尝试公开常见设置，但这并不总是可行的.对于这些情况，专用命名空间提供特定于服务器的自定义（请参阅 `server.tomcat` 和 `server.undertow` ）.例如，[access logs](howto-embedded-web-servers.html#howto-configure-accesslogs)可以配置嵌入式servlet容器的特定功能.

> See[ServerProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ServerProperties.java)类为完整列表.

#### Programmatic自定义

如果需要以编程方式配置嵌入式servlet容器，可以注册实现 `WebServerFactoryCustomizer` 接口的Spring bean.  `WebServerFactoryCustomizer` 提供对 `ConfigurableServletWebServerFactory` 的访问，其中包括许多自定义setter方法.以下示例以编程方式设置端口：

```java
import org.springframework.boot.web.server.WebServerFactoryCustomizer;
import org.springframework.boot.web.servlet.server.ConfigurableServletWebServerFactory;
import org.springframework.stereotype.Component;

@Component
public class CustomizationBean implements WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> {

	@Override
	public void customize(ConfigurableServletWebServerFactory server) {
		server.setPort(9000);
	}

}
```

>  `TomcatServletWebServerFactory` ， `JettyServletWebServerFactory` 和 `UndertowServletWebServerFactory` 是 `ConfigurableServletWebServerFactory` 的专用变体，分别为Tomcat，Jetty和Undertow提供了其他自定义setter方法.

#### 直接定制ConfigurableServletWebServerFactory

如果前面的自定义技术太有限，您可以自己注册 `TomcatServletWebServerFactory` ， `JettyServletWebServerFactory` 或 `UndertowServletWebServerFactory`  bean.

```java
@Bean
public ConfigurableServletWebServerFactory webServerFactory() {
	TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();
	factory.setPort(9000);
	factory.setSessionTimeout(10, TimeUnit.MINUTES);
	factory.addErrorPages(new ErrorPage(HttpStatus.NOT_FOUND, "/notfound.html"));
	return factory;
}
```

为许多配置选项提供了Setter.如果您需要做一些更具异国情调的事情，还会提供一些受保护的方法“挂钩”.有关详细信息，请参阅[source code documentation](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/api/org/springframework/boot/web/servlet/server/ConfigurableServletWebServerFactory.html).

### 28.4.5 JSP限制

运行使用嵌入式servlet容器的Spring Boot应用程序（并打包为可执行存档）时，JSP支持存在一些限制.

- 如果使用war包装，它应该可以使用Jetty和Tomcat.使用 `java -jar` 启动时，可执行的war将起作用，并且还可以部署到任何标准容器.使用可执行jar时不支持JSP.

- Undertow不支持JSP.

- 创建自定义 `error.jsp` 页面不会覆盖[error handling](boot-features-developing-web-applications.html#boot-features-error-handling)的默认视图.应该使用[Custom error pages](boot-features-developing-web-applications.html#boot-features-error-handling-custom-error-pages).

有一个[JSP sample](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-samples/spring-boot-sample-web-jsp)，以便您可以看到如何设置.

## 28.5嵌入式Reactive Server支持

Spring Boot包括对以下嵌入式响应式Web服务器的支持：Reactor Netty，Tomcat，Jetty和Undertow.大多数开发人员使用适当的“Starter”来获取完全配置的实例.默认情况下，嵌入式服务器在端口8080上侦听HTTP请求.

## 28.6 Reactive Server资源配置

在自动配置Reactor Netty或Jetty服务器时，Spring Boot将创建特定的bean，为服务器实例提供HTTP资源： `ReactorResourceFactory` 或 `JettyResourceFactory` .

默认情况下，这些资源也将与Reactor Netty和Jetty客户端共享以获得最佳性能，具体如下：

- 相同的技术用于服务器和客户端

- 客户端实例是使用Spring Boot自动配置的 `WebClient.Builder`  bean构建的

开发人员可以通过提供自定义 `ReactorResourceFactory` 或 `JettyResourceFactory`  bean来覆盖Jetty和Reactor Netty的资源配置 - 这将应用于客户端和服务器.

您可以在[WebClient Runtime section](boot-features-webclient.html#boot-features-webclient-runtime)中了解有关客户端资源配置的更多信息.

