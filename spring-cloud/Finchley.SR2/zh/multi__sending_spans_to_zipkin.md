## 60.向Zipkin发送Span

默认情况下，如果将 `spring-cloud-starter-zipkin` 作为依赖项添加到项目中，则当Span关闭时，它将通过HTTP发送到Zipkin.通信是异步的.您可以通过设置 `spring.zipkin.baseUrl` 属性来配置URL，如下所示：

```java
spring.zipkin.baseUrl: http://192.168.99.100:9411/
```

如果您想通过服务发现找到Zipkin，可以在URL中传递Zipkin的服务ID，如以下示例所示 `zipkinserver` 服务ID：

```java
spring.zipkin.baseUrl: http://zipkinserver/
```

要禁用此功能，只需将 `spring.zipkin.discoveryClientEnabled` 设置为“false”.

启用Discovery Client功能后，Sleuth使用 `LoadBalancerClient` 查找Zipkin服务器的URL.这意味着您可以设置负载平衡配置，例如通过功能区.

```java
zipkinserver:
ribbon:
ListOfServers: host1,host2
```

如果您在类路径上一起使用web，rabbit或kafka，则可能需要选择要将spans发送到zipkin的方法.为此，请将 `web` ， `rabbit` 或 `kafka` 设置为 `spring.zipkin.sender.type` 属性.以下示例显示为 `web` 设置发件人类型：

```java
spring.zipkin.sender.type: web
```

要自定义通过HTTP向Zipkin发送Span的_13386，您可以注册 `ZipkinRestTemplateCustomizer`  bean.

```java
@Configuration
class MyConfig {
	@Bean ZipkinRestTemplateCustomizer myCustomizer() {
		return new ZipkinRestTemplateCustomizer() {
			@Override
			void customize(RestTemplate restTemplate) {
				// customize the RestTemplate
			}
		};
	}
}
```

但是，如果您想控制创建 `RestTemplate` 对象的完整过程，则必须创建一个 `zipkin2.reporter.Sender` 的bean类型.

```java
@Bean Sender myRestTemplateSender(ZipkinProperties zipkin,
			ZipkinRestTemplateCustomizer zipkinRestTemplateCustomizer) {
		RestTemplate restTemplate = mySuperCustomRestTemplate();
		zipkinRestTemplateCustomizer.customize(restTemplate);
		return myCustomSender(zipkin, restTemplate);
	}
```
