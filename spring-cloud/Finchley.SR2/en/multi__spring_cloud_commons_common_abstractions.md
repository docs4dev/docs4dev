## 3. Spring Cloud Commons: Common Abstractions

Patterns such as service discovery, load balancing, and circuit breakers lend themselves to a common abstraction layer that can be consumed by all Spring Cloud clients, independent of the implementation (for example, discovery with Eureka or Consul).

## 3.1 @EnableDiscoveryClient

Spring Cloud Commons provides the  `@EnableDiscoveryClient`  annotation. This looks for implementations of the  `DiscoveryClient`  interface with  `META-INF/spring.factories` . Implementations of the Discovery Client add a configuration class to  `spring.factories`  under the  `org.springframework.cloud.client.discovery.EnableDiscoveryClient`  key. Examples of  `DiscoveryClient`  implementations include [Spring Cloud Netflix Eureka](https://cloud.spring.io/spring-cloud-netflix/), [Spring Cloud Consul Discovery](https://cloud.spring.io/spring-cloud-consul/), and [Spring Cloud Zookeeper Discovery](https://cloud.spring.io/spring-cloud-zookeeper/).

By default, implementations of  `DiscoveryClient`  auto-register the local Spring Boot server with the remote discovery server. This behavior can be disabled by setting  `autoRegister=false`  in  `@EnableDiscoveryClient` .

>  `@EnableDiscoveryClient`  is no longer required. You can put a  `DiscoveryClient`  implementation on the classpath to cause the Spring Boot application to register with the service discovery server.

### 3.1.1 Health Indicator

Commons creates a Spring Boot  `HealthIndicator`  that  `DiscoveryClient`  implementations can participate in by implementing  `DiscoveryHealthIndicator` . To disable the composite  `HealthIndicator` , set  `spring.cloud.discovery.client.composite-indicator.enabled=false` . A generic  `HealthIndicator`  based on  `DiscoveryClient`  is auto-configured ( `DiscoveryClientHealthIndicator` ). To disable it, set  `spring.cloud.discovery.client.health-indicator.enabled=false` . To disable the description field of the  `DiscoveryClientHealthIndicator` , set  `spring.cloud.discovery.client.health-indicator.include-description=false` . Otherwise, it can bubble up as the  `description`  of the rolled up  `HealthIndicator` .

## 3.2 ServiceRegistry

Commons now provides a  `ServiceRegistry`  interface that provides methods such as  `register(Registration)`  and  `deregister(Registration)` , which let you provide custom registered services.  `Registration`  is a marker interface.

The following example shows the  `ServiceRegistry`  in use:

```java
@Configuration
@EnableDiscoveryClient(autoRegister=false)
public class MyConfiguration {
private ServiceRegistry registry;

public MyConfiguration(ServiceRegistry registry) {
this.registry = registry;
}

// called through some external process, such as an event or a custom actuator endpoint
public void register() {
Registration registration = constructRegistration();
this.registry.register(registration);
}
}
```

Each  `ServiceRegistry`  implementation has its own  `Registry`  implementation.

-  `ZookeeperRegistration`  used with  `ZookeeperServiceRegistry` 

-  `EurekaRegistration`  used with  `EurekaServiceRegistry` 

-  `ConsulRegistration`  used with  `ConsulServiceRegistry` 

If you are using the  `ServiceRegistry`  interface, you are going to need to pass the correct  `Registry`  implementation for the  `ServiceRegistry`  implementation you are using.

### 3.2.1 ServiceRegistry Auto-Registration

By default, the  `ServiceRegistry`  implementation auto-registers the running service. To disable that behavior, you can set: *  `@EnableDiscoveryClient(autoRegister=false)`  to permanently disable auto-registration. *  `spring.cloud.service-registry.auto-registration.enabled=false`  to disable the behavior through configuration.

#### ServiceRegistry Auto-Registration Events

There are two events that will be fired when a service auto-registers. The first event, called  `InstancePreRegisteredEvent` , is fired before the service is registered. The second event, called  `InstanceRegisteredEvent` , is fired after the service is registered. You can register an  `ApplicationListener` (s) to listen to and react to these events.

> These events will not be fired if  `spring.cloud.service-registry.auto-registration.enabled`  is set to  `false` .

### 3.2.2 Service Registry Actuator Endpoint

Spring Cloud Commons provides a  `/service-registry`  actuator endpoint. This endpoint relies on a  `Registration`  bean in the Spring Application Context. Calling  `/service-registry`  with GET returns the status of the  `Registration` . Using POST to the same endpoint with a JSON body changes the status of the current  `Registration`  to the new value. The JSON body has to include the  `status`  field with the preferred value. Please see the documentation of the  `ServiceRegistry`  implementation you use for the allowed values when updating the status and the values returned for the status. For instance, Eureka’s supported statuses are  `UP` ,  `DOWN` ,  `OUT_OF_SERVICE` , and  `UNKNOWN` .

## 3.3 Spring RestTemplate as a Load Balancer Client

`RestTemplate`  can be automatically configured to use ribbon. To create a load-balanced  `RestTemplate` , create a  `RestTemplate`   `@Bean`  and use the  `@LoadBalanced`  qualifier, as shown in the following example:

```java
@Configuration
public class MyConfiguration {

@LoadBalanced
@Bean
RestTemplate restTemplate() {
return new RestTemplate();
}
}

public class MyClass {
@Autowired
private RestTemplate restTemplate;

public String doOtherStuff() {
String results = restTemplate.getForObject("http://stores/stores", String.class);
return results;
}
}
```

> A  `RestTemplate`  bean is no longer created through auto-configuration. Individual applications must create it.

The URI needs to use a virtual host name (that is, a service name, not a host name). The Ribbon client is used to create a full physical address. See [RibbonAutoConfiguration](https://github.com/spring-cloud/spring-cloud-netflix/blob/master/spring-cloud-netflix-core/src/main/java/org/springframework/cloud/netflix/ribbon/RibbonAutoConfiguration.java) for details of how the  `RestTemplate`  is set up.

## 3.4 Spring WebClient as a Load Balancer Client

`WebClient`  can be automatically configured to use the  `LoadBalancerClient` . To create a load-balanced  `WebClient` , create a  `WebClient.Builder`   `@Bean`  and use the  `@LoadBalanced`  qualifier, as shown in the following example:

```java
@Configuration
public class MyConfiguration {

	@Bean
	@LoadBalanced
	public WebClient.Builder loadBalancedWebClientBuilder() {
		return WebClient.builder();
	}
}

public class MyClass {
@Autowired
private WebClient.Builder webClientBuilder;

public Mono<String> doOtherStuff() {
return webClientBuilder.build().get().uri("http://stores/stores")
				.retrieve().bodyToMono(String.class);
}
}
```

The URI needs to use a virtual host name (that is, a service name, not a host name). The Ribbon client is used to create a full physical address.

### 3.4.1 Retrying Failed Requests

A load-balanced  `RestTemplate`  can be configured to retry failed requests. By default, this logic is disabled. You can enable it by adding [Spring Retry](https://github.com/spring-projects/spring-retry) to your application’s classpath. The load-balanced  `RestTemplate`  honors some of the Ribbon configuration values related to retrying failed requests. You can use  `client.ribbon.MaxAutoRetries` ,  `client.ribbon.MaxAutoRetriesNextServer` , and  `client.ribbon.OkToRetryOnAllOperations`  properties. If you would like to disable the retry logic with Spring Retry on the classpath, you can set  `spring.cloud.loadbalancer.retry.enabled=false` . See the [Ribbon documentation](https://github.com/Netflix/ribbon/wiki/Getting-Started#the-properties-file-sample-clientproperties) for a description of what these properties do.

If you would like to implement a  `BackOffPolicy`  in your retries, you need to create a bean of type  `LoadBalancedRetryFactory`  and override the  `createBackOffPolicy`  method:

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

>  `client`  in the preceding examples should be replaced with your Ribbon client’s name.

If you want to add one or more  `RetryListener`  implementations to your retry functionality, you need to create a bean of type  `LoadBalancedRetryListenerFactory`  and return the  `RetryListener`  array you would like to use for a given service, as shown in the following example:

```java
@Configuration
public class MyConfiguration {
@Bean
LoadBalancedRetryListenerFactory retryListenerFactory() {
return new LoadBalancedRetryListenerFactory() {
@Override
public RetryListener[] createRetryListeners(String service) {
return new RetryListener[]{new RetryListener() {
@Override
public <T, E extends Throwable> boolean open(RetryContext context, RetryCallback<T, E> callback) {
//TODO Do you business...
return true;
}

@Override
public <T, E extends Throwable> void close(RetryContext context, RetryCallback<T, E> callback, Throwable throwable) {
//TODO Do you business...
}

@Override
public <T, E extends Throwable> void onError(RetryContext context, RetryCallback<T, E> callback, Throwable throwable) {
//TODO Do you business...
}
}};
}
};
}
}
```

## 3.5 Multiple RestTemplate objects

If you want a  `RestTemplate`  that is not load-balanced, create a  `RestTemplate`  bean and inject it. To access the load-balanced  `RestTemplate` , use the  `@LoadBalanced`  qualifier when you create your  `@Bean` , as shown in the following example:\

```java
@Configuration
public class MyConfiguration {

@LoadBalanced
@Bean
RestTemplate loadBalanced() {
return new RestTemplate();
}

@Primary
@Bean
RestTemplate restTemplate() {
return new RestTemplate();
}
}

public class MyClass {
@Autowired
private RestTemplate restTemplate;

@Autowired
@LoadBalanced
private RestTemplate loadBalanced;

public String doOtherStuff() {
return loadBalanced.getForObject("http://stores/stores", String.class);
}

public String doStuff() {
return restTemplate.getForObject("http://example.com", String.class);
}
}
```

|images/important.png|Important|
|----|----|
|Notice the use of the  `@Primary`  annotation on the plain  `RestTemplate`  declaration in the preceding example to disambiguate the unqualified  `@Autowired`  injection. |

> If you see errors such as  `java.lang.IllegalArgumentException: Can not set org.springframework.web.client.RestTemplate field com.my.app.Foo.restTemplate to com.sun.proxy.$Proxy89` , try injecting  `RestOperations`  or setting  `spring.aop.proxyTargetClass=true` .

## 3.6 Spring WebFlux WebClient as a Load Balancer Client

`WebClient`  can be configured to use the  `LoadBalancerClient` .  `LoadBalancerExchangeFilterFunction`  is auto-configured if  `spring-webflux`  is on the classpath. The following example shows how to configure a  `WebClient`  to use load balancer:

```java
public class MyClass {
@Autowired
private LoadBalancerExchangeFilterFunction lbFunction;

public Mono<String> doOtherStuff() {
return WebClient.builder().baseUrl("http://stores")
.filter(lbFunction)
.build()
.get()
.uri("/stores")
.retrieve()
.bodyToMono(String.class);
}
}
```

The URI needs to use a virtual host name (that is, a service name, not a host name). The  `LoadBalancerClient`  is used to create a full physical address.

## 3.7 Ignore Network Interfaces

Sometimes, it is useful to ignore certain named network interfaces so that they can be excluded from Service Discovery registration (for example, when running in a Docker container). A list of regular expressions can be set to cause the desired network interfaces to be ignored. The following configuration ignores the  `docker0`  interface and all interfaces that start with  `veth` :

**application.yml.**  

```java
spring:
cloud:
inetutils:
ignoredInterfaces:
- docker0
- veth.*
```

You can also force the use of only specified network addresses by using a list of regular expressions, as shown in the following example:

**bootstrap.yml.**  

```java
spring:
cloud:
inetutils:
preferredNetworks:
- 192.168
- 10.0
```

You can also force the use of only site-local addresses, as shown in the following example: .application.yml

```java
spring:
cloud:
inetutils:
useOnlySiteLocalInterfaces: true
```

See [Inet4Address.html.isSiteLocalAddress()](https://docs.oracle.com/javase/8/docs/api/java/net/Inet4Address.html#isSiteLocalAddress--) for more details about what constitutes a site-local address.

## 3.8 HTTP Client Factories

Spring Cloud Commons provides beans for creating both Apache HTTP clients ( `ApacheHttpClientFactory` ) and OK HTTP clients ( `OkHttpClientFactory` ). The  `OkHttpClientFactory`  bean is created only if the OK HTTP jar is on the classpath. In addition, Spring Cloud Commons provides beans for creating the connection managers used by both clients:  `ApacheHttpClientConnectionManagerFactory`  for the Apache HTTP client and  `OkHttpClientConnectionPoolFactory`  for the OK HTTP client. If you would like to customize how the HTTP clients are created in downstream projects, you can provide your own implementation of these beans. In addition, if you provide a bean of type  `HttpClientBuilder`  or  `OkHttpClient.Builder` , the default factories use these builders as the basis for the builders returned to downstream projects. You can also disable the creation of these beans by setting  `spring.cloud.httpclientfactories.apache.enabled`  or  `spring.cloud.httpclientfactories.ok.enabled`  to  `false` .

## 3.9 Enabled Features

Spring Cloud Commons provides a  `/features`  actuator endpoint. This endpoint returns features available on the classpath and whether they are enabled. The information returned includes the feature type, name, version, and vendor.

### 3.9.1 Feature types

There are two types of 'features': abstract and named.

Abstract features are features where an interface or abstract class is defined and that an implementation the creates, such as  `DiscoveryClient` ,  `LoadBalancerClient` , or  `LockService` . The abstract class or interface is used to find a bean of that type in the context. The version displayed is  `bean.getClass().getPackage().getImplementationVersion()` .

Named features are features that do not have a particular class they implement, such as "Circuit Breaker", "API Gateway", "Spring Cloud Bus", and others. These features require a name and a bean type.

### 3.9.2 Declaring features

Any module can declare any number of  `HasFeature`  beans, as shown in the following examples:

```java
@Bean
public HasFeatures commonsFeatures() {
return HasFeatures.abstractFeatures(DiscoveryClient.class, LoadBalancerClient.class);
}

@Bean
public HasFeatures consulFeatures() {
return HasFeatures.namedFeatures(
new NamedFeature("Spring Cloud Bus", ConsulBusAutoConfiguration.class),
new NamedFeature("Circuit Breaker", HystrixCommandAspect.class));
}

@Bean
HasFeatures localFeatures() {
return HasFeatures.builder()
.abstractFeature(Foo.class)
.namedFeature(new NamedFeature("Bar Feature", Bar.class))
.abstractFeature(Baz.class)
.build();
}
```

Each of these beans should go in an appropriately guarded  `@Configuration` .

