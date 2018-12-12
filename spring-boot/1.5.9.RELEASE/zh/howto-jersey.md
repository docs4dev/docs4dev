## 81.Jersey

## 81.1使用Spring Security保护Jerseyendpoints

Spring Security可用于保护基于Jersey的Web应用程序，其方式与用于保护基于Spring MVC的Web应用程序的方式非常相似.但是，如果要在Jersey中使用Spring Security的方法级安全性，则必须将Jersey配置为使用 `setStatus(int)` 而不是 `sendError(int)` .这可以防止Jersey在Spring Security有机会向客户端报告身份验证或授权失败之前提交响应.

必须在应用程序的 `ResourceConfig`  bean上将 `jersey.config.server.response.setStatusOverSendError` 属性设置为 `true` ，如以下示例所示：

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

