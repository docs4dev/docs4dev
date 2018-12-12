## 13. Circuit Breaker: Hystrix Clients

Netflix has created a library called [Hystrix](https://github.com/Netflix/Hystrix) that implements the [circuit breaker pattern](http://martinfowler.com/bliki/CircuitBreaker.html). In a microservice architecture, it is common to have multiple layers of service calls, as shown in the following example:

**Figure 13.1. Microservice Graph** 

![Hystrix](https://www.docs4dev.com/images/b6ee86ca-c9b2-4a17-8265-d7481954a595.png)

A service failure in the lower level of services can cause cascading failure all the way up to the user. When calls to a particular service exceed  `circuitBreaker.requestVolumeThreshold`  (default: 20 requests) and the failure percentage is greater than  `circuitBreaker.errorThresholdPercentage`  (default: >50%) in a rolling window defined by  `metrics.rollingStats.timeInMilliseconds`  (default: 10 seconds), the circuit opens and the call is not made. In cases of error and an open circuit, a fallback can be provided by the developer.

**Figure 13.2. Hystrix fallback prevents cascading failures** 

![HystrixFallback](https://www.docs4dev.com/images/f4971d8c-8fce-45ad-a951-4b09ccef2d8b.png)

Having an open circuit stops cascading failures and allows overwhelmed or failing services time to recover. The fallback can be another Hystrix protected call, static data, or a sensible empty value. Fallbacks may be chained so that the first fallback makes some other business call, which in turn falls back to static data.

## 13.1 How to Include Hystrix

To include Hystrix in your project, use the starter with a group ID of  `org.springframework.cloud`  and a artifact ID of  `spring-cloud-starter-netflix-hystrix` . See the [Spring Cloud Project page](https://projects.spring.io/spring-cloud/) for details on setting up your build system with the current Spring Cloud Release Train.

The following example shows a minimal Eureka server with a Hystrix circuit breaker:

```java
@SpringBootApplication
@EnableCircuitBreaker
public class Application {

public static void main(String[] args) {
new SpringApplicationBuilder(Application.class).web(true).run(args);
}

}

@Component
public class StoreIntegration {

@HystrixCommand(fallbackMethod = "defaultStores")
public Object getStores(Map<String, Object> parameters) {
//do stuff that might fail
}

public Object defaultStores(Map<String, Object> parameters) {
return /* something useful */;
}
}
```

The  `@HystrixCommand`  is provided by a Netflix contrib library called [“javanica”](https://github.com/Netflix/Hystrix/tree/master/hystrix-contrib/hystrix-javanica). Spring Cloud automatically wraps Spring beans with that annotation in a proxy that is connected to the Hystrix circuit breaker. The circuit breaker calculates when to open and close the circuit and what to do in case of a failure.

To configure the  `@HystrixCommand`  you can use the  `commandProperties`  attribute with a list of  `@HystrixProperty`  annotations. See [here](https://github.com/Netflix/Hystrix/tree/master/hystrix-contrib/hystrix-javanica#configuration) for more details. See the [Hystrix wiki](https://github.com/Netflix/Hystrix/wiki/Configuration) for details on the properties available.

## 13.2 Propagating the Security Context or Using Spring Scopes

If you want some thread local context to propagate into a  `@HystrixCommand` , the default declaration does not work, because it executes the command in a thread pool (in case of timeouts). You can switch Hystrix to use the same thread as the caller through configuration or directly in the annotation, by asking it to use a different “Isolation Strategy”. The following example demonstrates setting the thread in the annotation:

```java
@HystrixCommand(fallbackMethod = "stubMyService",
commandProperties = {
@HystrixProperty(name="execution.isolation.strategy", value="SEMAPHORE")
}
)
...
```

The same thing applies if you are using  `@SessionScope`  or  `@RequestScope` . If you encounter a runtime exception that says it cannot find the scoped context, you need to use the same thread.

You also have the option to set the  `hystrix.shareSecurityContext`  property to  `true` . Doing so auto-configures a Hystrix concurrency strategy plugin hook to transfer the  `SecurityContext`  from your main thread to the one used by the Hystrix command. Hystrix does not let multiple Hystrix concurrency strategy be registered so an extension mechanism is available by declaring your own  `HystrixConcurrencyStrategy`  as a Spring bean. Spring Cloud looks for your implementation within the Spring context and wrap it inside its own plugin.

## 13.3 Health Indicator

The state of the connected circuit breakers are also exposed in the  `/health`  endpoint of the calling application, as shown in the following example:

```java
{
"hystrix": {
"openCircuitBreakers": [
"StoreIntegration::getStoresByLocationLink"
],
"status": "CIRCUIT_OPEN"
},
"status": "UP"
}
```

## 13.4 Hystrix Metrics Stream

To enable the Hystrix metrics stream, include a dependency on  `spring-boot-starter-actuator`  and set  `management.endpoints.web.exposure.include: hystrix.stream` . Doing so exposes the  `/actuator/hystrix.stream`  as a management endpoint, as shown in the following example:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

