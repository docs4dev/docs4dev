## 82. HTTP Clients

Spring Boot offers a number of starters that work with HTTP clients. This section answers questions related to using them.

## 82.1 Configure RestTemplate to Use a Proxy

As described in [Section 34.1, “RestTemplate Customization”](boot-features-resttemplate.html#boot-features-resttemplate-customization), you can use a  `RestTemplateCustomizer`  with  `RestTemplateBuilder`  to build a customized  `RestTemplate` . This is the recommended approach for creating a  `RestTemplate`  configured to use a proxy.

The exact details of the proxy configuration depend on the underlying client request factory that is being used. The following example configures  `HttpComponentsClientRequestFactory`  with an  `HttpClient`  that uses a proxy for all hosts except  `192.168.0.5` :

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

