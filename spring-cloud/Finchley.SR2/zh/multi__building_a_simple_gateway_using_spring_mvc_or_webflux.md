## 122.使用Spring MVC或Webflux构建简单网关

Spring Cloud Gateway提供了一个名为 `ProxyExchange` 的实用程序对象，您可以在常规Spring Web处理程序中将其用作方法参数.它通过镜像HTTP谓词的方法支持基本的下游HTTP交换.使用MVC，它还支持通过 `forward()` 方法转发到本地处理程序.要使用 `ProxyExchange` ，只需在类路径中包含正确的模块（ `spring-cloud-gateway-mvc` 或 `spring-cloud-gateway-webflux` ）.

MVC示例（代理向远程服务器下游“/ test”的请求）：

```xml
@RestController
@SpringBootApplication
public class GatewaySampleApplication {

	@Value("${remote.home}")
	private URI home;

	@GetMapping("/test")
	public ResponseEntity<?> proxy(ProxyExchange<byte[]> proxy) throws Exception {
		return proxy.uri(home.toString() + "/image/png").get();
	}

}
```

Webflux也是如此：

```xml
@RestController
@SpringBootApplication
public class GatewaySampleApplication {

	@Value("${remote.home}")
	private URI home;

	@GetMapping("/test")
	public Mono<ResponseEntity<?>> proxy(ProxyExchange<byte[]> proxy) throws Exception {
		return proxy.uri(home.toString() + "/image/png").get();
	}

}
```

`ProxyExchange` 上有便捷方法，使处理程序方法能够发现和增强传入请求的URI路径.例如，您可能希望提取路径的尾随元素以将其传递到下游：

```xml
@GetMapping("/proxy/path/**")
public ResponseEntity<?> proxyPath(ProxyExchange<byte[]> proxy) throws Exception {
String path = proxy.path("/proxy/path/");
return proxy.uri(home.toString() + "/foos/" + path).get();
}
```

Spring MVC或Webflux的所有功能都可用于Gateway处理程序方法.例如，您可以注入请求标头和查询参数，并且您可以使用映射注释中的声明来约束传入的请求.有关这些功能的更多详细信息，请参阅Spring MVC中 `@RequestMapping` 的文档.

可以使用 `ProxyExchange` 上的 `header()` 方法将标头添加到下游响应中.

您还可以通过向 `get()` 等方法添加映射器来操作响应标头（以及响应中您喜欢的任何其他内容）.映射器是 `Function` ，它接收传入的 `ResponseEntity` 并将其转换为传出的.

为"sensitive"标头（默认情况下为"cookie"和"authorization"）提供了第一类支持，这些标头未向下游传递，并为"proxy"标头（ `x-forwarded-*` ）提供.
