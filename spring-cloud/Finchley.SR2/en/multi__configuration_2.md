## 118. Configuration

Configuration for Spring Cloud Gateway is driven by a collection of `RouteDefinitionLocator`s.

**RouteDefinitionLocator.java.**  

```java
public interface RouteDefinitionLocator {
	Flux<RouteDefinition> getRouteDefinitions();
}
```

By default, a  `PropertiesRouteDefinitionLocator`  loads properties using Spring Boot’s  `@ConfigurationProperties`  mechanism.

The configuration examples above all use a shortcut notation that uses positional arguments rather than named ones. The two examples below are equivalent:

**application.yml.**  

```java
spring:
cloud:
gateway:
routes:
- id: setstatus_route
uri: http://example.org
filters:
- name: SetStatus
args:
status: 401
- id: setstatusshortcut_route
uri: http://example.org
filters:
- SetStatus=401
```

For some usages of the gateway, properties will be adequate, but some production use cases will benefit from loading configuration from an external source, such as a database. Future milestone versions will have  `RouteDefinitionLocator`  implementations based off of Spring Data Repositories such as: Redis, MongoDB and Cassandra.

## 118.1 Fluent Java Routes API

To allow for simple configuration in Java, there is a fluent API defined in the  `RouteLocatorBuilder`  bean.

**GatewaySampleApplication.java.**  

```java
// static imports from GatewayFilters and RoutePredicates
@Bean
public RouteLocator customRouteLocator(RouteLocatorBuilder builder, ThrottleGatewayFilterFactory throttle) {
return builder.routes()
.route(r -> r.host("**.abc.org").and().path("/image/png")
.filters(f ->
f.addResponseHeader("X-TestHeader", "foobar"))
.uri("http://httpbin.org:80")
)
.route(r -> r.path("/image/webp")
.filters(f ->
f.addResponseHeader("X-AnotherHeader", "baz"))
.uri("http://httpbin.org:80")
)
.route(r -> r.order(-1)
.host("**.throttle.org").and().path("/get")
.filters(f -> f.filter(throttle.apply(1,
1,
10,
TimeUnit.SECONDS)))
.uri("http://httpbin.org:80")
)
.build();
}
```

This style also allows for more custom predicate assertions. The predicates defined by  `RouteDefinitionLocator`  beans are combined using logical  `and` . By using the fluent Java API, you can use the  `and()` ,  `or()`  and  `negate()`  operators on the  `Predicate`  class.

## 118.2 DiscoveryClient Route Definition Locator

The Gateway can be configured to create routes based on services registered with a  `DiscoveryClient`  compatible service registry.

To enable this, set  `spring.cloud.gateway.discovery.locator.enabled=true`  and make sure a  `DiscoveryClient`  implementation is on the classpath and enabled (such as Netflix Eureka, Consul or Zookeeper).

