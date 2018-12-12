## 34. Calling REST Services with RestTemplate

If you need to call remote REST services from your application, you can use the Spring Framework’s [RestTemplate](https://docs.spring.io/spring/docs/5.1.2.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html) class. Since  `RestTemplate`  instances often need to be customized before being used, Spring Boot does not provide any single auto-configured  `RestTemplate`  bean. It does, however, auto-configure a  `RestTemplateBuilder` , which can be used to create  `RestTemplate`  instances when needed. The auto-configured  `RestTemplateBuilder`  ensures that sensible  `HttpMessageConverters`  are applied to  `RestTemplate`  instances.

The following code shows a typical example:

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

>  `RestTemplateBuilder`  includes a number of useful methods that can be used to quickly configure a  `RestTemplate` . For example, to add BASIC auth support, you can use  `builder.basicAuthentication("user", "password").build()` .

## 34.1 RestTemplate Customization

There are three main approaches to  `RestTemplate`  customization, depending on how broadly you want the customizations to apply.

To make the scope of any customizations as narrow as possible, inject the auto-configured  `RestTemplateBuilder`  and then call its methods as required. Each method call returns a new  `RestTemplateBuilder`  instance, so the customizations only affect this use of the builder.

To make an application-wide, additive customization, use a  `RestTemplateCustomizer`  bean. All such beans are automatically registered with the auto-configured  `RestTemplateBuilder`  and are applied to any templates that are built with it.

The following example shows a customizer that configures the use of a proxy for all hosts except  `192.168.0.5` :

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

Finally, the most extreme (and rarely used) option is to create your own  `RestTemplateBuilder`  bean. Doing so switches off the auto-configuration of a  `RestTemplateBuilder`  and prevents any  `RestTemplateCustomizer`  beans from being used.

