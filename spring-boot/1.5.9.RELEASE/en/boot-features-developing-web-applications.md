## 28. Developing Web Applications

Spring Boot is well suited for web application development. You can create a self-contained HTTP server by using embedded Tomcat, Jetty, Undertow, or Netty. Most web applications use the  `spring-boot-starter-web`  module to get up and running quickly. You can also choose to build reactive web applications by using the  `spring-boot-starter-webflux`  module.

If you have not yet developed a Spring Boot web application, you can follow the "Hello World!" example in the [Getting started](getting-started-first-application.html) section.

## 28.1 The “Spring Web MVC Framework”

The [Spring Web MVC framework](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#mvc) (often referred to as simply “Spring MVC”) is a rich “model view controller” web framework. Spring MVC lets you create special  `@Controller`  or  `@RestController`  beans to handle incoming HTTP requests. Methods in your controller are mapped to HTTP by using  `@RequestMapping`  annotations.

The following code shows a typical  `@RestController`  that serves JSON data:

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

Spring MVC is part of the core Spring Framework, and detailed information is available in the [reference documentation](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#mvc). There are also several guides that cover Spring MVC available at [spring.io/guides](https://spring.io/guides).

### 28.1.1 Spring MVC Auto-configuration

Spring Boot provides auto-configuration for Spring MVC that works well with most applications.

The auto-configuration adds the following features on top of Spring’s defaults:

- Inclusion of  `ContentNegotiatingViewResolver`  and  `BeanNameViewResolver`  beans.

- Support for serving static resources, including support for WebJars (covered [later in this document](boot-features-developing-web-applications.html#boot-features-spring-mvc-static-content))).

- Automatic registration of  `Converter` ,  `GenericConverter` , and  `Formatter`  beans.

- Support for  `HttpMessageConverters`  (covered [later in this document](boot-features-developing-web-applications.html#boot-features-spring-mvc-message-converters)).

- Automatic registration of  `MessageCodesResolver`  (covered [later in this document](boot-features-developing-web-applications.html#boot-features-spring-message-codes)).

- Static  `index.html`  support.

- Custom  `Favicon`  support (covered [later in this document](boot-features-developing-web-applications.html#boot-features-spring-mvc-favicon)).

- Automatic use of a  `ConfigurableWebBindingInitializer`  bean (covered [later in this document](boot-features-developing-web-applications.html#boot-features-spring-mvc-web-binding-initializer)).

If you want to keep Spring Boot MVC features and you want to add additional [MVC configuration](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#mvc) (interceptors, formatters, view controllers, and other features), you can add your own  `@Configuration`  class of type  `WebMvcConfigurer`  but  **without**   `@EnableWebMvc` . If you wish to provide custom instances of  `RequestMappingHandlerMapping` ,  `RequestMappingHandlerAdapter` , or  `ExceptionHandlerExceptionResolver` , you can declare a  `WebMvcRegistrationsAdapter`  instance to provide such components.

If you want to take complete control of Spring MVC, you can add your own  `@Configuration`  annotated with  `@EnableWebMvc` .

### 28.1.2 HttpMessageConverters

Spring MVC uses the  `HttpMessageConverter`  interface to convert HTTP requests and responses. Sensible defaults are included out of the box. For example, objects can be automatically converted to JSON (by using the Jackson library) or XML (by using the Jackson XML extension, if available, or by using JAXB if the Jackson XML extension is not available). By default, strings are encoded in  `UTF-8` .

If you need to add or customize converters, you can use Spring Boot’s  `HttpMessageConverters`  class, as shown in the following listing:

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

Any  `HttpMessageConverter`  bean that is present in the context is added to the list of converters. You can also override default converters in the same way.

### 28.1.3 Custom JSON Serializers and Deserializers

If you use Jackson to serialize and deserialize JSON data, you might want to write your own  `JsonSerializer`  and  `JsonDeserializer`  classes. Custom serializers are usually [registered with Jackson through a module](https://github.com/FasterXML/jackson-docs/wiki/JacksonHowToCustomSerializers), but Spring Boot provides an alternative  `@JsonComponent`  annotation that makes it easier to directly register Spring Beans.

You can use the  `@JsonComponent`  annotation directly on  `JsonSerializer`  or  `JsonDeserializer`  implementations. You can also use it on classes that contain serializers/deserializers as inner classes, as shown in the following example:

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

All  `@JsonComponent`  beans in the  `ApplicationContext`  are automatically registered with Jackson. Because  `@JsonComponent`  is meta-annotated with  `@Component` , the usual component-scanning rules apply.

Spring Boot also provides [JsonObjectSerializer](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jackson/JsonObjectSerializer.java) and [JsonObjectDeserializer](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jackson/JsonObjectDeserializer.java) base classes that provide useful alternatives to the standard Jackson versions when serializing objects. See [JsonObjectSerializer](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/api/org/springframework/boot/jackson/JsonObjectSerializer.html) and [JsonObjectDeserializer](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/api/org/springframework/boot/jackson/JsonObjectDeserializer.html) in the Javadoc for details.

### 28.1.4 MessageCodesResolver

Spring MVC has a strategy for generating error codes for rendering error messages from binding errors:  `MessageCodesResolver` . If you set the  `spring.mvc.message-codes-resolver.format`  property  `PREFIX_ERROR_CODE`  or  `POSTFIX_ERROR_CODE` , Spring Boot creates one for you (see the enumeration in [DefaultMessageCodesResolver.Format](https://docs.spring.io/spring/docs/5.1.2.RELEASE/javadoc-api/org/springframework/validation/DefaultMessageCodesResolver.Format.html)).

### 28.1.5 Static Content

By default, Spring Boot serves static content from a directory called  `/static`  (or  `/public`  or  `/resources`  or  `/META-INF/resources` ) in the classpath or from the root of the  `ServletContext` . It uses the  `ResourceHttpRequestHandler`  from Spring MVC so that you can modify that behavior by adding your own  `WebMvcConfigurer`  and overriding the  `addResourceHandlers`  method.

In a stand-alone web application, the default servlet from the container is also enabled and acts as a fallback, serving content from the root of the  `ServletContext`  if Spring decides not to handle it. Most of the time, this does not happen (unless you modify the default MVC configuration), because Spring can always handle requests through the  `DispatcherServlet` .

By default, resources are mapped on  `/**` , but you can tune that with the  `spring.mvc.static-path-pattern`  property. For instance, relocating all resources to  `/resources/**`  can be achieved as follows:

```java
spring.mvc.static-path-pattern=/resources/**
```

You can also customize the static resource locations by using the  `spring.resources.static-locations`  property (replacing the default values with a list of directory locations). The root Servlet context path,  `"/"` , is automatically added as a location as well.

In addition to the “standard” static resource locations mentioned earlier, a special case is made for [Webjars content](https://www.webjars.org/). Any resources with a path in  `/webjars/**`  are served from jar files if they are packaged in the Webjars format.

> Do not use the  `src/main/webapp`  directory if your application is packaged as a jar. Although this directory is a common standard, it works  **only**  with war packaging, and it is silently ignored by most build tools if you generate a jar.

Spring Boot also supports the advanced resource handling features provided by Spring MVC, allowing use cases such as cache-busting static resources or using version agnostic URLs for Webjars.

To use version agnostic URLs for Webjars, add the  `webjars-locator-core`  dependency. Then declare your Webjar. Using jQuery as an example, adding  `"/webjars/jquery/jquery.min.js"`  results in  `"/webjars/jquery/x.y.z/jquery.min.js"` . where  `x.y.z`  is the Webjar version.

> If you use JBoss, you need to declare the  `webjars-locator-jboss-vfs`  dependency instead of the  `webjars-locator-core` . Otherwise, all Webjars resolve as a  `404` .

To use cache busting, the following configuration configures a cache busting solution for all static resources, effectively adding a content hash, such as  `<link href="/css/spring-2a2d595e6ed9a0b24f027f2b63b134d6.css"/>` , in URLs:

```java
spring.resources.chain.strategy.content.enabled=true
spring.resources.chain.strategy.content.paths=/**
```

> Links to resources are rewritten in templates at runtime, thanks to a  `ResourceUrlEncodingFilter`  that is auto-configured for Thymeleaf and FreeMarker. You should manually declare this filter when using JSPs. Other template engines are currently not automatically supported but can be with custom template macros/helpers and the use of the [ResourceUrlProvider](https://docs.spring.io/spring/docs/5.1.2.RELEASE/javadoc-api/org/springframework/web/servlet/resource/ResourceUrlProvider.html).

When loading resources dynamically with, for example, a JavaScript module loader, renaming files is not an option. That is why other strategies are also supported and can be combined. A "fixed" strategy adds a static version string in the URL without changing the file name, as shown in the following example:

```java
spring.resources.chain.strategy.content.enabled=true
spring.resources.chain.strategy.content.paths=/**
spring.resources.chain.strategy.fixed.enabled=true
spring.resources.chain.strategy.fixed.paths=/js/lib/
spring.resources.chain.strategy.fixed.version=v12
```

With this configuration, JavaScript modules located under  `"/js/lib/"`  use a fixed versioning strategy ( `"/v12/js/lib/mymodule.js"` ), while other resources still use the content one ( `<link href="/css/spring-2a2d595e6ed9a0b24f027f2b63b134d6.css"/>` ).

See [ResourceProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ResourceProperties.java) for more supported options.

> This feature has been thoroughly described in a dedicated [blog post](https://spring.io/blog/2014/07/24/spring-framework-4-1-handling-static-web-resources) and in Spring Framework’s [reference documentation](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#mvc-config-static-resources).

### 28.1.6 Welcome Page

Spring Boot supports both static and templated welcome pages. It first looks for an  `index.html`  file in the configured static content locations. If one is not found, it then looks for an  `index`  template. If either is found, it is automatically used as the welcome page of the application.

### 28.1.7 Custom Favicon

Spring Boot looks for a  `favicon.ico`  in the configured static content locations and the root of the classpath (in that order). If such a file is present, it is automatically used as the favicon of the application.

### 28.1.8 Path Matching and Content Negotiation

Spring MVC can map incoming HTTP requests to handlers by looking at the request path and matching it to the mappings defined in your application (for example,  `@GetMapping`  annotations on Controller methods).

Spring Boot chooses to disable suffix pattern matching by default, which means that requests like  `"GET /projects/spring-boot.json"`  won’t be matched to  `@GetMapping("/projects/spring-boot")`  mappings. This is considered as a [best practice for Spring MVC applications](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#mvc-ann-requestmapping-suffix-pattern-match). This feature was mainly useful in the past for HTTP clients which did not send proper "Accept" request headers; we needed to make sure to send the correct Content Type to the client. Nowadays, Content Negotiation is much more reliable.

There are other ways to deal with HTTP clients that don’t consistently send proper "Accept" request headers. Instead of using suffix matching, we can use a query parameter to ensure that requests like  `"GET /projects/spring-boot?format=json"`  will be mapped to  `@GetMapping("/projects/spring-boot")` :

```java
spring.mvc.contentnegotiation.favor-parameter=true

# We can change the parameter name, which is "format" by default:
# spring.mvc.contentnegotiation.parameter-name=myparam

# We can also register additional file extensions/media types with:
spring.mvc.contentnegotiation.media-types.markdown=text/markdown
```

If you understand the caveats and would still like your application to use suffix pattern matching, the following configuration is required:

```java
spring.mvc.contentnegotiation.favor-path-extension=true
spring.mvc.pathmatch.use-suffix-pattern=true
```

Alternatively, rather than open all suffix patterns, it’s more secure to just support registered suffix patterns:

```java
spring.mvc.contentnegotiation.favor-path-extension=true
spring.mvc.pathmatch.use-registered-suffix-pattern=true

# You can also register additional file extensions/media types with:
# spring.mvc.contentnegotiation.media-types.adoc=text/asciidoc
```

### 28.1.9 ConfigurableWebBindingInitializer

Spring MVC uses a  `WebBindingInitializer`  to initialize a  `WebDataBinder`  for a particular request. If you create your own  `ConfigurableWebBindingInitializer`   `@Bean` , Spring Boot automatically configures Spring MVC to use it.

### 28.1.10 Template Engines

As well as REST web services, you can also use Spring MVC to serve dynamic HTML content. Spring MVC supports a variety of templating technologies, including Thymeleaf, FreeMarker, and JSPs. Also, many other templating engines include their own Spring MVC integrations.

Spring Boot includes auto-configuration support for the following templating engines:

- [FreeMarker](https://freemarker.apache.org/docs/)

- [Groovy](http://docs.groovy-lang.org/docs/next/html/documentation/template-engines.html#_the_markuptemplateengine)

- [Thymeleaf](http://www.thymeleaf.org)

- [Mustache](https://mustache.github.io/)

> If possible, JSPs should be avoided. There are several [known limitations](boot-features-developing-web-applications.html#boot-features-jsp-limitations) when using them with embedded servlet containers.

When you use one of these templating engines with the default configuration, your templates are picked up automatically from  `src/main/resources/templates` .

> Depending on how you run your application, IntelliJ IDEA orders the classpath differently. Running your application in the IDE from its main method results in a different ordering than when you run your application by using Maven or Gradle or from its packaged jar. This can cause Spring Boot to fail to find the templates on the classpath. If you have this problem, you can reorder the classpath in the IDE to place the module’s classes and resources first. Alternatively, you can configure the template prefix to search every  `templates`  directory on the classpath, as follows:  `classpath*:/templates/` .

### 28.1.11 Error Handling

By default, Spring Boot provides an  `/error`  mapping that handles all errors in a sensible way, and it is registered as a “global” error page in the servlet container. For machine clients, it produces a JSON response with details of the error, the HTTP status, and the exception message. For browser clients, there is a “whitelabel” error view that renders the same data in HTML format (to customize it, add a  `View`  that resolves to  `error` ). To replace the default behavior completely, you can implement  `ErrorController`  and register a bean definition of that type or add a bean of type  `ErrorAttributes`  to use the existing mechanism but replace the contents.

> The  `BasicErrorController`  can be used as a base class for a custom  `ErrorController` . This is particularly useful if you want to add a handler for a new content type (the default is to handle  `text/html`  specifically and provide a fallback for everything else). To do so, extend  `BasicErrorController` , add a public method with a  `@RequestMapping`  that has a  `produces`  attribute, and create a bean of your new type.

You can also define a class annotated with  `@ControllerAdvice`  to customize the JSON document to return for a particular controller and/or exception type, as shown in the following example:

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

In the preceding example, if  `YourException`  is thrown by a controller defined in the same package as  `AcmeController` , a JSON representation of the  `CustomErrorType`  POJO is used instead of the  `ErrorAttributes`  representation.

#### Custom Error Pages

If you want to display a custom HTML error page for a given status code, you can add a file to an  `/error`  folder. Error pages can either be static HTML (that is, added under any of the static resource folders) or be built by using templates. The name of the file should be the exact status code or a series mask.

For example, to map  `404`  to a static HTML file, your folder structure would be as follows:

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

To map all  `5xx`  errors by using a FreeMarker template, your folder structure would be as follows:

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

For more complex mappings, you can also add beans that implement the  `ErrorViewResolver`  interface, as shown in the following example:

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

You can also use regular Spring MVC features such as [@ExceptionHandler methods](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#mvc-exceptionhandlers) and [@ControllerAdvice](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#mvc-ann-controller-advice). The  `ErrorController`  then picks up any unhandled exceptions.

#### Mapping Error Pages outside of Spring MVC

For applications that do not use Spring MVC, you can use the  `ErrorPageRegistrar`  interface to directly register  `ErrorPages` . This abstraction works directly with the underlying embedded servlet container and works even if you do not have a Spring MVC  `DispatcherServlet` .

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

> If you register an  `ErrorPage`  with a path that ends up being handled by a  `Filter`  (as is common with some non-Spring web frameworks, like Jersey and Wicket), then the  `Filter`  has to be explicitly registered as an  `ERROR`  dispatcher, as shown in the following example:

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

Note that the default  `FilterRegistrationBean`  does not include the  `ERROR`  dispatcher type.

CAUTION:When deployed to a servlet container, Spring Boot uses its error page filter to forward a request with an error status to the appropriate error page. The request can only be forwarded to the correct error page if the response has not already been committed. By default, WebSphere Application Server 8.0 and later commits the response upon successful completion of a servlet’s service method. You should disable this behavior by setting  `com.ibm.ws.webcontainer.invokeFlushAfterService`  to  `false` .

### 28.1.12 Spring HATEOAS

If you develop a RESTful API that makes use of hypermedia, Spring Boot provides auto-configuration for Spring HATEOAS that works well with most applications. The auto-configuration replaces the need to use  `@EnableHypermediaSupport`  and registers a number of beans to ease building hypermedia-based applications, including a  `LinkDiscoverers`  (for client side support) and an  `ObjectMapper`  configured to correctly marshal responses into the desired representation. The  `ObjectMapper`  is customized by setting the various  `spring.jackson.*`  properties or, if one exists, by a  `Jackson2ObjectMapperBuilder`  bean.

You can take control of Spring HATEOAS’s configuration by using  `@EnableHypermediaSupport` . Note that doing so disables the  `ObjectMapper`  customization described earlier.

### 28.1.13 CORS Support

[Cross-origin resource sharing](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) (CORS) is a [W3C specification](https://www.w3.org/TR/cors/) implemented by [most browsers](https://caniuse.com/#feat=cors) that lets you specify in a flexible way what kind of cross-domain requests are authorized, instead of using some less secure and less powerful approaches such as IFRAME or JSONP.

As of version 4.2, Spring MVC [supports CORS](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#cors). Using [controller method CORS configuration](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#controller-method-cors-configuration) with [@CrossOrigin](https://docs.spring.io/spring/docs/5.1.2.RELEASE/javadoc-api/org/springframework/web/bind/annotation/CrossOrigin.html) annotations in your Spring Boot application does not require any specific configuration. [Global CORS configuration](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#global-cors-configuration) can be defined by registering a  `WebMvcConfigurer`  bean with a customized  `addCorsMappings(CorsRegistry)`  method, as shown in the following example:

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

## 28.2 The “Spring WebFlux Framework”

Spring WebFlux is the new reactive web framework introduced in Spring Framework 5.0. Unlike Spring MVC, it does not require the Servlet API, is fully asynchronous and non-blocking, and implements the [Reactive Streams](http://www.reactive-streams.org/) specification through [the Reactor project](https://projectreactor.io/).

Spring WebFlux comes in two flavors: functional and annotation-based. The annotation-based one is quite close to the Spring MVC model, as shown in the following example:

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

“WebFlux.fn”, the functional variant, separates the routing configuration from the actual handling of the requests, as shown in the following example:

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

WebFlux is part of the Spring Framework and detailed information is available in its [reference documentation](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web-reactive.html#webflux-fn).

> You can define as many  `RouterFunction`  beans as you like to modularize the definition of the router. Beans can be ordered if you need to apply a precedence.

To get started, add the  `spring-boot-starter-webflux`  module to your application.

> Adding both  `spring-boot-starter-web`  and  `spring-boot-starter-webflux`  modules in your application results in Spring Boot auto-configuring Spring MVC, not WebFlux. This behavior has been chosen because many Spring developers add  `spring-boot-starter-webflux`  to their Spring MVC application to use the reactive  `WebClient` . You can still enforce your choice by setting the chosen application type to  `SpringApplication.setWebApplicationType(WebApplicationType.REACTIVE)` .

### 28.2.1 Spring WebFlux Auto-configuration

Spring Boot provides auto-configuration for Spring WebFlux that works well with most applications.

The auto-configuration adds the following features on top of Spring’s defaults:

- Configuring codecs for  `HttpMessageReader`  and  `HttpMessageWriter`  instances (described [later in this document](boot-features-developing-web-applications.html#boot-features-webflux-httpcodecs)).

- Support for serving static resources, including support for WebJars (described [later in this document](boot-features-developing-web-applications.html#boot-features-spring-mvc-static-content)).

If you want to keep Spring Boot WebFlux features and you want to add additional [WebFlux configuration](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#web-reactive), you can add your own  `@Configuration`  class of type  `WebFluxConfigurer`  but  **without**   `@EnableWebFlux` .

If you want to take complete control of Spring WebFlux, you can add your own  `@Configuration`  annotated with  `@EnableWebFlux` .

### 28.2.2 HTTP Codecs with HttpMessageReaders and HttpMessageWriters

Spring WebFlux uses the  `HttpMessageReader`  and  `HttpMessageWriter`  interfaces to convert HTTP requests and responses. They are configured with  `CodecConfigurer`  to have sensible defaults by looking at the libraries available in your classpath.

Spring Boot applies further customization by using  `CodecCustomizer`  instances. For example,  `spring.jackson.*`  configuration keys are applied to the Jackson codec.

If you need to add or customize codecs, you can create a custom  `CodecCustomizer`  component, as shown in the following example:

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

You can also leverage [Boot’s custom JSON serializers and deserializers](boot-features-developing-web-applications.html#boot-features-json-components).

### 28.2.3 Static Content

By default, Spring Boot serves static content from a directory called  `/static`  (or  `/public`  or  `/resources`  or  `/META-INF/resources` ) in the classpath. It uses the  `ResourceWebHandler`  from Spring WebFlux so that you can modify that behavior by adding your own  `WebFluxConfigurer`  and overriding the  `addResourceHandlers`  method.

By default, resources are mapped on  `/**` , but you can tune that by setting the  `spring.webflux.static-path-pattern`  property. For instance, relocating all resources to  `/resources/**`  can be achieved as follows:

```java
spring.webflux.static-path-pattern=/resources/**
```

You can also customize the static resource locations by using  `spring.resources.static-locations` . Doing so replaces the default values with a list of directory locations. If you do so, the default welcome page detection switches to your custom locations. So, if there is an  `index.html`  in any of your locations on startup, it is the home page of the application.

In addition to the “standard” static resource locations listed earlier, a special case is made for [Webjars content](https://www.webjars.org/). Any resources with a path in  `/webjars/**`  are served from jar files if they are packaged in the Webjars format.

> Spring WebFlux applications do not strictly depend on the Servlet API, so they cannot be deployed as war files and do not use the  `src/main/webapp`  directory.

### 28.2.4 Template Engines

As well as REST web services, you can also use Spring WebFlux to serve dynamic HTML content. Spring WebFlux supports a variety of templating technologies, including Thymeleaf, FreeMarker, and Mustache.

Spring Boot includes auto-configuration support for the following templating engines:

- [FreeMarker](https://freemarker.apache.org/docs/)

- [Thymeleaf](http://www.thymeleaf.org)

- [Mustache](https://mustache.github.io/)

When you use one of these templating engines with the default configuration, your templates are picked up automatically from  `src/main/resources/templates` .

### 28.2.5 Error Handling

Spring Boot provides a  `WebExceptionHandler`  that handles all errors in a sensible way. Its position in the processing order is immediately before the handlers provided by WebFlux, which are considered last. For machine clients, it produces a JSON response with details of the error, the HTTP status, and the exception message. For browser clients, there is a “whitelabel” error handler that renders the same data in HTML format. You can also provide your own HTML templates to display errors (see the [next section](boot-features-developing-web-applications.html#boot-features-webflux-error-handling-custom-error-pages)).

The first step to customizing this feature often involves using the existing mechanism but replacing or augmenting the error contents. For that, you can add a bean of type  `ErrorAttributes` .

To change the error handling behavior, you can implement  `ErrorWebExceptionHandler`  and register a bean definition of that type. Because a  `WebExceptionHandler`  is quite low-level, Spring Boot also provides a convenient  `AbstractErrorWebExceptionHandler`  to let you handle errors in a WebFlux functional way, as shown in the following example:

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

For a more complete picture, you can also subclass  `DefaultErrorWebExceptionHandler`  directly and override specific methods.

#### Custom Error Pages

If you want to display a custom HTML error page for a given status code, you can add a file to an  `/error`  folder. Error pages can either be static HTML (that is, added under any of the static resource folders) or built with templates. The name of the file should be the exact status code or a series mask.

For example, to map  `404`  to a static HTML file, your folder structure would be as follows:

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

To map all  `5xx`  errors by using a Mustache template, your folder structure would be as follows:

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

### 28.2.6 Web Filters

Spring WebFlux provides a  `WebFilter`  interface that can be implemented to filter HTTP request-response exchanges.  `WebFilter`  beans found in the application context will be automatically used to filter each exchange.

Where the order of the filters is important they can implement  `Ordered`  or be annotated with  `@Order` . Spring Boot auto-configuration may configure web filters for you. When it does so, the orders shown in the following table will be used:

|Web Filter|Order|
|----|----|
| `MetricsWebFilter`  | `Ordered.HIGHEST_PRECEDENCE + 1`  |
| `WebFilterChainProxy`  (Spring Security) | `-100`  |
| `HttpTraceWebFilter`  | `Ordered.LOWEST_PRECEDENCE - 10`  |

## 28.3 JAX-RS and Jersey

If you prefer the JAX-RS programming model for REST endpoints, you can use one of the available implementations instead of Spring MVC. [Jersey](https://jersey.github.io/) and [Apache CXF](https://cxf.apache.org/) work quite well out of the box. CXF requires you to register its  `Servlet`  or  `Filter`  as a  `@Bean`  in your application context. Jersey has some native Spring support, so we also provide auto-configuration support for it in Spring Boot, together with a starter.

To get started with Jersey, include the  `spring-boot-starter-jersey`  as a dependency and then you need one  `@Bean`  of type  `ResourceConfig`  in which you register all the endpoints, as shown in the following example:

```java
@Component
public class JerseyConfig extends ResourceConfig {

	public JerseyConfig() {
		register(Endpoint.class);
	}

}
```

> Jersey’s support for scanning executable archives is rather limited. For example, it cannot scan for endpoints in a package found in  `WEB-INF/classes`  when running an executable war file. To avoid this limitation, the  `packages`  method should not be used, and endpoints should be registered individually by using the  `register`  method, as shown in the preceding example.

For more advanced customizations, you can also register an arbitrary number of beans that implement  `ResourceConfigCustomizer` .

All the registered endpoints should be  `@Components`  with HTTP resource annotations ( `@GET`  and others), as shown in the following example:

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

Since the  `Endpoint`  is a Spring  `@Component` , its lifecycle is managed by Spring and you can use the  `@Autowired`  annotation to inject dependencies and use the  `@Value`  annotation to inject external configuration. By default, the Jersey servlet is registered and mapped to  `/*` . You can change the mapping by adding  `@ApplicationPath`  to your  `ResourceConfig` .

By default, Jersey is set up as a Servlet in a  `@Bean`  of type  `ServletRegistrationBean`  named  `jerseyServletRegistration` . By default, the servlet is initialized lazily, but you can customize that behavior by setting  `spring.jersey.servlet.load-on-startup` . You can disable or override that bean by creating one of your own with the same name. You can also use a filter instead of a servlet by setting  `spring.jersey.type=filter`  (in which case, the  `@Bean`  to replace or override is  `jerseyFilterRegistration` ). The filter has an  `@Order` , which you can set with  `spring.jersey.filter.order` . Both the servlet and the filter registrations can be given init parameters by using  `spring.jersey.init.*`  to specify a map of properties.

There is a [Jersey sample](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-samples/spring-boot-sample-jersey) so that you can see how to set things up.

## 28.4 Embedded Servlet Container Support

Spring Boot includes support for embedded [Tomcat](https://tomcat.apache.org/), [Jetty](https://www.eclipse.org/jetty/), and [Undertow](http://undertow.io/) servers. Most developers use the appropriate “Starter” to obtain a fully configured instance. By default, the embedded server listens for HTTP requests on port  `8080` .

> If you choose to use Tomcat on [CentOS](https://www.centos.org/), be aware that, by default, a temporary directory is used to store compiled JSPs, file uploads, and so on. This directory may be deleted by  `tmpwatch`  while your application is running, leading to failures. To avoid this behavior, you may want to customize your  `tmpwatch`  configuration such that  `tomcat.*`  directories are not deleted or configure  `server.tomcat.basedir`  such that embedded Tomcat uses a different location.

### 28.4.1 Servlets, Filters, and listeners

When using an embedded servlet container, you can register servlets, filters, and all the listeners (such as  `HttpSessionListener` ) from the Servlet spec, either by using Spring beans or by scanning for Servlet components.

#### Registering Servlets, Filters, and Listeners as Spring Beans

Any  `Servlet` ,  `Filter` , or servlet  `*Listener`  instance that is a Spring bean is registered with the embedded container. This can be particularly convenient if you want to refer to a value from your  `application.properties`  during configuration.

By default, if the context contains only a single Servlet, it is mapped to  `/` . In the case of multiple servlet beans, the bean name is used as a path prefix. Filters map to  `/*` .

If convention-based mapping is not flexible enough, you can use the  `ServletRegistrationBean` ,  `FilterRegistrationBean` , and  `ServletListenerRegistrationBean`  classes for complete control.

Spring Boot ships with many auto-configurations that may define Filter beans. Here are a few examples of Filters and their respective order (lower order value means higher precedence):

|Servlet Filter|Order|
|----|----|
| `OrderedCharacterEncodingFilter`  | `Ordered.HIGHEST_PRECEDENCE`  |
| `WebMvcMetricsFilter`  | `Ordered.HIGHEST_PRECEDENCE + 1`  |
| `ErrorPageFilter`  | `Ordered.HIGHEST_PRECEDENCE + 1`  |
| `HttpTraceFilter`  | `Ordered.LOWEST_PRECEDENCE - 10`  |

It is usually safe to leave Filter beans unordered.

If a specific order is required, you should avoid configuring a Filter that reads the request body at  `Ordered.HIGHEST_PRECEDENCE` , since it might go against the character encoding configuration of your application. If a Servlet filter wraps the request, it should be configured with an order that is less than or equal to  `OrderedFilter.REQUEST_WRAPPER_FILTER_MAX_ORDER` .

### 28.4.2 Servlet Context Initialization

Embedded servlet containers do not directly execute the Servlet 3.0+  `javax.servlet.ServletContainerInitializer`  interface or Spring’s  `org.springframework.web.WebApplicationInitializer`  interface. This is an intentional design decision intended to reduce the risk that third party libraries designed to run inside a war may break Spring Boot applications.

If you need to perform servlet context initialization in a Spring Boot application, you should register a bean that implements the  `org.springframework.boot.web.servlet.ServletContextInitializer`  interface. The single  `onStartup`  method provides access to the  `ServletContext`  and, if necessary, can easily be used as an adapter to an existing  `WebApplicationInitializer` .

#### Scanning for Servlets, Filters, and listeners

When using an embedded container, automatic registration of classes annotated with  `@WebServlet` ,  `@WebFilter` , and  `@WebListener`  can be enabled by using  `@ServletComponentScan` .

>  `@ServletComponentScan`  has no effect in a standalone container, where the container’s built-in discovery mechanisms are used instead.

### 28.4.3 The ServletWebServerApplicationContext

Under the hood, Spring Boot uses a different type of  `ApplicationContext`  for embedded servlet container support. The  `ServletWebServerApplicationContext`  is a special type of  `WebApplicationContext`  that bootstraps itself by searching for a single  `ServletWebServerFactory`  bean. Usually a  `TomcatServletWebServerFactory` ,  `JettyServletWebServerFactory` , or  `UndertowServletWebServerFactory`  has been auto-configured.

> You usually do not need to be aware of these implementation classes. Most applications are auto-configured, and the appropriate  `ApplicationContext`  and  `ServletWebServerFactory`  are created on your behalf.

### 28.4.4 Customizing Embedded Servlet Containers

Common servlet container settings can be configured by using Spring  `Environment`  properties. Usually, you would define the properties in your  `application.properties`  file.

Common server settings include:

- Network settings: Listen port for incoming HTTP requests ( `server.port` ), interface address to bind to  `server.address` , and so on.

- Session settings: Whether the session is persistent ( `server.servlet.session.persistence` ), session timeout ( `server.servlet.session.timeout` ), location of session data ( `server.servlet.session.store-dir` ), and session-cookie configuration ( `server.servlet.session.cookie.*` ).

- Error management: Location of the error page ( `server.error.path` ) and so on.

- [SSL](howto-embedded-web-servers.html#howto-configure-ssl)

- [HTTP compression](howto-embedded-web-servers.html#how-to-enable-http-response-compression)

Spring Boot tries as much as possible to expose common settings, but this is not always possible. For those cases, dedicated namespaces offer server-specific customizations (see  `server.tomcat`  and  `server.undertow` ). For instance, [access logs](howto-embedded-web-servers.html#howto-configure-accesslogs) can be configured with specific features of the embedded servlet container.

> See the [ServerProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ServerProperties.java) class for a complete list.

#### Programmatic Customization

If you need to programmatically configure your embedded servlet container, you can register a Spring bean that implements the  `WebServerFactoryCustomizer`  interface.  `WebServerFactoryCustomizer`  provides access to the  `ConfigurableServletWebServerFactory` , which includes numerous customization setter methods. The following example shows programmatically setting the port:

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

>  `TomcatServletWebServerFactory` ,  `JettyServletWebServerFactory`  and  `UndertowServletWebServerFactory`  are dedicated variants of  `ConfigurableServletWebServerFactory`  that have additional customization setter methods for Tomcat, Jetty and Undertow respectively.

#### Customizing ConfigurableServletWebServerFactory Directly

If the preceding customization techniques are too limited, you can register the  `TomcatServletWebServerFactory` ,  `JettyServletWebServerFactory` , or  `UndertowServletWebServerFactory`  bean yourself.

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

Setters are provided for many configuration options. Several protected method “hooks” are also provided should you need to do something more exotic. See the [source code documentation](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/api/org/springframework/boot/web/servlet/server/ConfigurableServletWebServerFactory.html) for details.

### 28.4.5 JSP Limitations

When running a Spring Boot application that uses an embedded servlet container (and is packaged as an executable archive), there are some limitations in the JSP support.

- With Jetty and Tomcat, it should work if you use war packaging. An executable war will work when launched with  `java -jar` , and will also be deployable to any standard container. JSPs are not supported when using an executable jar.

- Undertow does not support JSPs.

- Creating a custom  `error.jsp`  page does not override the default view for [error handling](boot-features-developing-web-applications.html#boot-features-error-handling). [Custom error pages](boot-features-developing-web-applications.html#boot-features-error-handling-custom-error-pages) should be used instead.

There is a [JSP sample](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-samples/spring-boot-sample-web-jsp) so that you can see how to set things up.

## 28.5 Embedded Reactive Server Support

Spring Boot includes support for the following embedded reactive web servers: Reactor Netty, Tomcat, Jetty, and Undertow. Most developers use the appropriate “Starter” to obtain a fully configured instance. By default, the embedded server listens for HTTP requests on port 8080.

## 28.6 Reactive Server Resources Configuration

When auto-configuring a Reactor Netty or Jetty server, Spring Boot will create specific beans that will provide HTTP resources to the server instance:  `ReactorResourceFactory`  or  `JettyResourceFactory` .

By default, those resources will be also shared with the Reactor Netty and Jetty clients for optimal performances, given:

- the same technology is used for server and client

- the client instance is built using the  `WebClient.Builder`  bean auto-configured by Spring Boot

Developers can override the resource configuration for Jetty and Reactor Netty by providing a custom  `ReactorResourceFactory`  or  `JettyResourceFactory`  bean - this will be applied to both clients and servers.

You can learn more about the resource configuration on the client side in the [WebClient Runtime section](boot-features-webclient.html#boot-features-webclient-runtime).

