## 20.重试失败的请求

Spring Cloud Netflix提供了多种方式来发出HTTP请求.您可以使用负载平衡 `RestTemplate` ，功能区或假设.无论您如何选择创建HTTP请求，总是有可能请求失败.请求失败时，您可能希望自动重试请求.要在使用Sping Cloud Netflix时执行此操作，您需要在应用程序的类路径中包含[Spring Retry](https://github.com/spring-projects/spring-retry).当Spring Retry存在时，负载均衡 `RestTemplates` ，Feign和Zuul会自动重试任何失败的请求（假设您的配置允许这样做）.

## 20.1 BackOff政策

默认情况下，重试请求时不使用退避策略.如果要配置退避策略，则需要创建 `LoadBalancedRetryFactory` 类型的bean并覆盖给定服务的 `createBackOffPolicy` 方法，如以下示例所示：

```java
@Configuration
public class MyConfiguration {
@Bean
LoadBalancedRetryFactory retryFactory() {
return new LoadBalancedRetryFactory() {
@Override
public BackOffPolicy createBackOffPolicy(String service) {
return new ExponentialBackOffPolicy();
}
};
}
}
```

## 20.2配置

将Ribbon与Spring Retry一起使用时，可以通过配置某些功能区属性来控制重试功能.为此，请设置 `client.ribbon.MaxAutoRetries` ， `client.ribbon.MaxAutoRetriesNextServer` 和 `client.ribbon.OkToRetryOnAllOperations` 属性.有关这些属性的说明，请参阅[Ribbon documentation](https://github.com/Netflix/ribbon/wiki/Getting-Started#the-properties-file-sample-clientproperties).

> Enabling  `client.ribbon.OkToRetryOnAllOperations` 包括重试POST请求，这会因请求正文的缓冲而对服务器资源产生影响.

此外，您可能希望在响应中返回某些状态代码时重试请求.您可以通过设置 `clientName.ribbon.retryableStatusCodes` 属性列出您希望Ribbon客户端重试的响应代码，如以下示例所示：

```java
clientName:
ribbon:
retryableStatusCodes: 404,502
```

您还可以创建 `LoadBalancedRetryPolicy` 类型的bean，并实现 `retryableStatusCode` 方法以在给定状态代码的情况下重试请求.

### 20.2.1 Zuul

您可以通过将 `zuul.retryable` 设置为 `false` 来关闭Zuul的重试功能.您也可以禁用重试通过将 `zuul.routes.routename.retryable` 设置为 `false` ，逐个路由的功能.

