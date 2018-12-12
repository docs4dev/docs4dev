## 60. Sending Spans to Zipkin

By default, if you add  `spring-cloud-starter-zipkin`  as a dependency to your project, when the span is closed, it is sent to Zipkin over HTTP. The communication is asynchronous. You can configure the URL by setting the  `spring.zipkin.baseUrl`  property, as follows:

```java
spring.zipkin.baseUrl: http://192.168.99.100:9411/
```

If you want to find Zipkin through service discovery, you can pass the Zipkin’s service ID inside the URL, as shown in the following example for  `zipkinserver`  service ID:

```java
spring.zipkin.baseUrl: http://zipkinserver/
```

To disable this feature just set  `spring.zipkin.discoveryClientEnabled`  to `false.

When the Discovery Client feature is enabled, Sleuth uses  `LoadBalancerClient`  to find the URL of the Zipkin Server. It means that you can set up the load balancing configuration e.g. via Ribbon.

```java
zipkinserver:
ribbon:
ListOfServers: host1,host2
```

If you have web, rabbit, or kafka together on the classpath, you might need to pick the means by which you would like to send spans to zipkin. To do so, set  `web` ,  `rabbit` , or  `kafka`  to the  `spring.zipkin.sender.type`  property. The following example shows setting the sender type for  `web` :

```java
spring.zipkin.sender.type: web
```

To customize the  `RestTemplate`  that sends spans to Zipkin via HTTP, you can register the  `ZipkinRestTemplateCustomizer`  bean.

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

If, however, you would like to control the full process of creating the  `RestTemplate`  object, you will have to create a bean of  `zipkin2.reporter.Sender`  type.

```java
@Bean Sender myRestTemplateSender(ZipkinProperties zipkin,
			ZipkinRestTemplateCustomizer zipkinRestTemplateCustomizer) {
		RestTemplate restTemplate = mySuperCustomRestTemplate();
		zipkinRestTemplateCustomizer.customize(restTemplate);
		return myCustomSender(zipkin, restTemplate);
	}
```
