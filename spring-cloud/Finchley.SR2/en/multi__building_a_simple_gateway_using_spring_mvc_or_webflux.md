## 122. Building a Simple Gateway Using Spring MVC or Webflux

Spring Cloud Gateway provides a utility object called  `ProxyExchange`  which you can use inside a regular Spring web handler as a method parameter. It supports basic downstream HTTP exchanges via methods that mirror the HTTP verbs. With MVC it also supports forwarding to a local handler via the  `forward()`  method. To use the  `ProxyExchange`  just include the right module in your classpath (either  `spring-cloud-gateway-mvc`  or  `spring-cloud-gateway-webflux` ).

MVC example (proxying a request to "/test" downstream to a remote server):

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

The same thing with Webflux:

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

There are convenience methods on the  `ProxyExchange`  to enable the handler method to discover and enhance the URI path of the incoming request. For example you might want to extract the trailing elements of a path to pass them downstream:

```xml
@GetMapping("/proxy/path/**")
public ResponseEntity<?> proxyPath(ProxyExchange<byte[]> proxy) throws Exception {
String path = proxy.path("/proxy/path/");
return proxy.uri(home.toString() + "/foos/" + path).get();
}
```

All the features of Spring MVC or Webflux are available to Gateway handler methods. So you can inject request headers and query parameters, for instance, and you can constrain the incoming requests with declarations in the mapping annotation. See the documentation for  `@RequestMapping`  in Spring MVC for more details of those features.

Headers can be added to the downstream response using the  `header()`  methods on  `ProxyExchange` .

You can also manipulate response headers (and anything else you like in the response) by adding a mapper to the  `get()`  etc. method. The mapper is a  `Function`  that takes the incoming  `ResponseEntity`  and converts it to an outgoing one.

First class support is provided for "sensitive" headers ("cookie" and "authorization" by default) which are not passed downstream, and for "proxy" headers ( `x-forwarded-*` ).
