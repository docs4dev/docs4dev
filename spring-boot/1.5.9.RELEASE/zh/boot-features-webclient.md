## 35.使用WebClient调用REST服务

如果在类路径上有Spring WebFlux，则还可以选择使用 `WebClient` 来调用远程REST服务.与 `RestTemplate` 相比，此客户端具有更多功能感并且完全被动.您可以在专用[section in the Spring Framework docs](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web-reactive.html#webflux-client)中了解有关 `WebClient` 的更多信息.

Spring Boot为您创建并预配置 `WebClient.Builder` ;强烈建议将其注入组件并使用它来创建 `WebClient` 实例. Spring Boot正在配置该构建器以共享HTTP资源，以与服务器相同的方式反映编解码器设置（请参阅[WebFlux HTTP codecs auto-configuration](boot-features-developing-web-applications.html#boot-features-webflux-httpcodecs)）等.

以下代码显示了一个典型示例：

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

## 35.1 WebClient运行时

Spring Boot将自动检测用于驱动 `WebClient` 的 `ClientHttpConnector` ，具体取决于应用程序类路径上可用的库.目前，支持Reactor Netty和Jetty RS客户端.

`spring-boot-starter-webflux` 启动程序默认依赖于 `io.projectreactor.netty:reactor-netty` ，这会带来服务器和客户端实现.如果您选择将Jetty用作反应式服务器，则应该在Jetty Reactive HTTP客户端库 `org.eclipse.jetty:jetty-reactive-httpclient` 上添加依赖项.对服务器和客户端使用相同的技术具有优势，因为它将自动在客户端和服务器之间共享HTTP资源.

开发人员可以通过提供自定义 `ReactorResourceFactory` 或 `JettyResourceFactory`  bean来覆盖Jetty和Reactor Netty的资源配置 - 这将应用于客户端和服务器.

如果您希望覆盖客户端的该选项，则可以定义自己的 `ClientHttpConnector`  bean并完全控制客户端配置.

您可以了解有关[WebClient configuration options in the Spring Framework reference documentation](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web-reactive.html#webflux-client-builder)的更多信息.

## 35.2 WebClient自定义

`WebClient` 自定义有三种主要方法，具体取决于您希望自定义应用的广泛程度.

要使任何自定义的范围尽可能窄，请注入自动配置的 `WebClient.Builder` ，然后根据需要调用其方法.  `WebClient.Builder` 实例是有状态的：构建器上的任何更改都会反映在随后使用它创建的所有客户端中.如果要使用同一构建器创建多个客户端，还可以考虑使用 `WebClient.Builder other = builder.clone();` 克隆构建器.

要对所有 `WebClient.Builder` 实例进行应用程序范围的附加自定义，可以声明 `WebClientCustomizer`  beans并在注入点本地更改 `WebClient.Builder` .

最后，您可以回退到原始API并使用 `WebClient.create()` .在这种情况下，不应用自动配置或 `WebClientCustomizer` .

