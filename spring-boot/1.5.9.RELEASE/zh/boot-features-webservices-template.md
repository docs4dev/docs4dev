## 48.使用WebServiceTemplate调用Web服务

如果需要从应用程序调用远程Web服务，可以使用[WebServiceTemplate](https://docs.spring.io/spring-ws/docs/3.0.4.RELEASE/reference/#client-web-service-template)类.由于 `WebServiceTemplate` 实例在使用之前通常需要自定义，因此Spring Boot不提供任何单个自动配置的 `WebServiceTemplate` bean.但是，它会自动配置 `WebServiceTemplateBuilder` ，可在需要时用于创建 `WebServiceTemplate` 实例.

以下代码显示了一个典型示例：

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

默认情况下， `WebServiceTemplateBuilder` 使用类路径上的可用HTTP客户端库检测到合适的基于HTTP的 `WebServiceMessageSender` .您还可以按如下方式自定义读取和连接超时：

```java
@Bean
public WebServiceTemplate webServiceTemplate(WebServiceTemplateBuilder builder) {
	return builder.messageSenders(new HttpWebServiceMessageSenderBuilder()
			.setConnectTimeout(5000).setReadTimeout(2000).build()).build();
}
```
