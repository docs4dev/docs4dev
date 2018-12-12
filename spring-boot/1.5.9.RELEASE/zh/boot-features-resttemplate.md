## 34.使用RestTemplate调用REST服务

如果需要从应用程序调用远程REST服务，可以使用Spring Framework的[RestTemplate](https://docs.spring.io/spring/docs/5.1.2.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html)类.由于 `RestTemplate` 实例在使用之前通常需要进行自定义，因此Spring Boot不提供任何单个自动配置的 `RestTemplate`  bean.但是，它会自动配置 `RestTemplateBuilder` ，可在需要时用于创建 `RestTemplate` 实例.自动配置的 `RestTemplateBuilder` 确保将合理的 `HttpMessageConverters` 应用于 `RestTemplate` 实例.

以下代码显示了一个典型示例：

```java
@Service
public class MyService {

	private final RestTemplate restTemplate;

	public MyService(RestTemplateBuilder restTemplateBuilder) {
		this.restTemplate = restTemplateBuilder.build();
	}

	public Details someRestCall(String name) {
		return this.restTemplate.getForObject("/{name}/details", Details.class, name);
	}

}
```

>  `RestTemplateBuilder` 包含许多可用于快速配置 `RestTemplate` 的有用方法.例如，要添加BASIC auth支持，可以使用 `builder.basicAuthentication("user", "password").build()` .

## 34.1 RestTemplate自定义

主要有三种方法到 `RestTemplate` 自定义，具体取决于您希望自定义应用的广泛程度.

要使任何自定义的范围尽可能窄，请注入自动配置的 `RestTemplateBuilder` ，然后根据需要调用其方法.每个方法调用都返回一个新的 `RestTemplateBuilder` 实例，因此自定义只会影响构建器的这种使用.

要进行应用程序范围的附加定制，请使用 `RestTemplateCustomizer`  bean.所有这些bean都会自动注册自动配置的 `RestTemplateBuilder` ，并应用于使用它构建的任何模板.

以下示例显示了一个自定义程序，它为除 `192.168.0.5` 之外的所有主机配置代理的使用：

```java
static class ProxyCustomizer implements RestTemplateCustomizer {

	@Override
	public void customize(RestTemplate restTemplate) {
		HttpHost proxy = new HttpHost("proxy.example.com");
		HttpClient httpClient = HttpClientBuilder.create()
				.setRoutePlanner(new DefaultProxyRoutePlanner(proxy) {

					@Override
					public HttpHost determineProxy(HttpHost target,
							HttpRequest request, HttpContext context)
							throws HttpException {
						if (target.getHostName().equals("192.168.0.5")) {
							return null;
						}
						return super.determineProxy(target, request, context);
					}

				}).build();
		restTemplate.setRequestFactory(
				new HttpComponentsClientHttpRequestFactory(httpClient));
	}

}
```

最后，最极端（也很少使用）的选项是创建自己的 `RestTemplateBuilder`  bean.这样做会关闭 `RestTemplateBuilder` 的自动配置，并防止使用任何 `RestTemplateCustomizer`  bean.

