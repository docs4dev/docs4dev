## 81. Jersey

## 81.1 Secure Jersey endpoints with Spring Security

Spring Security can be used to secure a Jersey-based web application in much the same way as it can be used to secure a Spring MVC-based web application. However, if you want to use Spring Security’s method-level security with Jersey, you must configure Jersey to use  `setStatus(int)`  rather  `sendError(int)` . This prevents Jersey from committing the response before Spring Security has had an opportunity to report an authentication or authorization failure to the client.

The  `jersey.config.server.response.setStatusOverSendError`  property must be set to  `true`  on the application’s  `ResourceConfig`  bean, as shown in the following example:

```java
@Component
public class JerseyConfig extends ResourceConfig {

	public JerseyConfig() {
		register(Endpoint.class);
		setProperties(Collections.singletonMap(
				"jersey.config.server.response.setStatusOverSendError", true));
	}

}
```

