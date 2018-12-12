## 35. Calling REST Services with WebClient

If you have Spring WebFlux on your classpath, you can also choose to use  `WebClient`  to call remote REST services. Compared to  `RestTemplate` , this client has a more functional feel and is fully reactive. You can learn more about the  `WebClient`  in the dedicated [section in the Spring Framework docs](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web-reactive.html#webflux-client).

Spring Boot creates and pre-configures a  `WebClient.Builder`  for you; it is strongly advised to inject it in your components and use it to create  `WebClient`  instances. Spring Boot is configuring that builder to share HTTP resources, reflect codecs setup in the same fashion as the server ones (see [WebFlux HTTP codecs auto-configuration](boot-features-developing-web-applications.html#boot-features-webflux-httpcodecs)), and more.

The following code shows a typical example:

```java
@Service
public class MyService {

	private final WebClient webClient;

	public MyService(WebClient.Builder webClientBuilder) {
		this.webClient = webClientBuilder.baseUrl("http://example.org").build();
	}

	public Mono<Details> someRestCall(String name) {
		return this.webClient.get().uri("/{name}/details", name)
						.retrieve().bodyToMono(Details.class);
	}

}
```

## 35.1 WebClient Runtime

Spring Boot will auto-detect which  `ClientHttpConnector`  to use to drive  `WebClient` , depending on the libraries available on the application classpath. For now, Reactor Netty and Jetty RS client are supported.

The  `spring-boot-starter-webflux`  starter depends on  `io.projectreactor.netty:reactor-netty`  by default, which brings both server and client implementations. If you choose to use Jetty as a reactive server instead, you should add a dependency on the Jetty Reactive HTTP client library,  `org.eclipse.jetty:jetty-reactive-httpclient` . Using the same technology for server and client has it advantages, as it will automatically share HTTP resources between client and server.

Developers can override the resource configuration for Jetty and Reactor Netty by providing a custom  `ReactorResourceFactory`  or  `JettyResourceFactory`  bean - this will be applied to both clients and servers.

If you wish to override that choice for the client, you can define your own  `ClientHttpConnector`  bean and have full control over the client configuration.

You can learn more about the [WebClient configuration options in the Spring Framework reference documentation](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web-reactive.html#webflux-client-builder).

## 35.2 WebClient Customization

There are three main approaches to  `WebClient`  customization, depending on how broadly you want the customizations to apply.

To make the scope of any customizations as narrow as possible, inject the auto-configured  `WebClient.Builder`  and then call its methods as required.  `WebClient.Builder`  instances are stateful: Any change on the builder is reflected in all clients subsequently created with it. If you want to create several clients with the same builder, you can also consider cloning the builder with  `WebClient.Builder other = builder.clone();` .

To make an application-wide, additive customization to all  `WebClient.Builder`  instances, you can declare  `WebClientCustomizer`  beans and change the  `WebClient.Builder`  locally at the point of injection.

Finally, you can fall back to the original API and use  `WebClient.create()` . In that case, no auto-configuration or  `WebClientCustomizer`  is applied.

