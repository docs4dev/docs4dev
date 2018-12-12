## 20. Retrying Failed Requests

Spring Cloud Netflix offers a variety of ways to make HTTP requests. You can use a load balanced  `RestTemplate` , Ribbon, or Feign. No matter how you choose to create your HTTP requests, there is always a chance that a request may fail. When a request fails, you may want to have the request be retried automatically. To do so when using Sping Cloud Netflix, you need to include [Spring Retry](https://github.com/spring-projects/spring-retry) on your application’s classpath. When Spring Retry is present, load-balanced  `RestTemplates` , Feign, and Zuul automatically retry any failed requests (assuming your configuration allows doing so).

## 20.1 BackOff Policies

By default, no backoff policy is used when retrying requests. If you would like to configure a backoff policy, you need to create a bean of type  `LoadBalancedRetryFactory`  and override the  `createBackOffPolicy`  method for a given service, as shown in the following example:

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

## 20.2 Configuration

When you use Ribbon with Spring Retry, you can control the retry functionality by configuring certain Ribbon properties. To do so, set the  `client.ribbon.MaxAutoRetries` ,  `client.ribbon.MaxAutoRetriesNextServer` , and  `client.ribbon.OkToRetryOnAllOperations`  properties. See the [Ribbon documentation](https://github.com/Netflix/ribbon/wiki/Getting-Started#the-properties-file-sample-clientproperties) for a description of what these properties do.

> Enabling  `client.ribbon.OkToRetryOnAllOperations`  includes retrying POST requests, which can have an impact on the server’s resources, due to the buffering of the request body.

In addition, you may want to retry requests when certain status codes are returned in the response. You can list the response codes you would like the Ribbon client to retry by setting the  `clientName.ribbon.retryableStatusCodes`  property, as shown in the following example:

```java
clientName:
ribbon:
retryableStatusCodes: 404,502
```

You can also create a bean of type  `LoadBalancedRetryPolicy`  and implement the  `retryableStatusCode`  method to retry a request given the status code.

### 20.2.1 Zuul

You can turn off Zuul’s retry functionality by setting  `zuul.retryable`  to  `false` . You can also disable retry functionality on a route-by-route basis by setting  `zuul.routes.routename.retryable`  to  `false` .

