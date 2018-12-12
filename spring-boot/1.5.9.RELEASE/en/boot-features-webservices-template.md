## 48. Calling Web Services with WebServiceTemplate

If you need to call remote Web services from your application, you can use the [WebServiceTemplate](https://docs.spring.io/spring-ws/docs/3.0.4.RELEASE/reference/#client-web-service-template) class. Since  `WebServiceTemplate`  instances often need to be customized before being used, Spring Boot does not provide any single auto-configured  `WebServiceTemplate`  bean. It does, however, auto-configure a  `WebServiceTemplateBuilder` , which can be used to create  `WebServiceTemplate`  instances when needed.

The following code shows a typical example:

```java
@Service
public class MyService {

	private final WebServiceTemplate webServiceTemplate;

	public MyService(WebServiceTemplateBuilder webServiceTemplateBuilder) {
		this.webServiceTemplate = webServiceTemplateBuilder.build();
	}

	public DetailsResp someWsCall(DetailsReq detailsReq) {
		 return (DetailsResp) this.webServiceTemplate.marshalSendAndReceive(detailsReq, new SoapActionCallback(ACTION));

	}

}
```

By default,  `WebServiceTemplateBuilder`  detects a suitable HTTP-based  `WebServiceMessageSender`  using the available HTTP client libraries on the classpath. You can also customize read and connection timeouts as follows:

```java
@Bean
public WebServiceTemplate webServiceTemplate(WebServiceTemplateBuilder builder) {
	return builder.messageSenders(new HttpWebServiceMessageSenderBuilder()
			.setConnectTimeout(5000).setReadTimeout(2000).build()).build();
}
```
