## 82. HTTP客户端

Spring Boot提供了许多可与HTTP客户端配合使用的启动器.本节回答与使用它们相关的问题.

## 82.1配置RestTemplate以使用代理

如[Section 34.1, “RestTemplate Customization”](boot-features-resttemplate.html#boot-features-resttemplate-customization)中所述，您可以使用 `RestTemplateCustomizer` 和 `RestTemplateBuilder` 来构建自定义的 `RestTemplate` .这是创建配置为使用代理的 `RestTemplate` 的推荐方法.

代理配置的确切详细信息取决于正在使用的基础客户端请求工厂.以下示例使用 `HttpClient` 配置 `HttpComponentsClientRequestFactory` ，该使用代理用于除 `192.168.0.5` 之外的所有主机：

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

