## 118.配置

Spring Cloud Gateway的配置由`RouteDefinitionLocator`s的集合驱动.

**RouteDefinitionLocator.java.** 

```java
public interface RouteDefinitionLocator {
	Flux<RouteDefinition> getRouteDefinitions();
}
```

默认情况下， `PropertiesRouteDefinitionLocator` 使用Spring Boot的 `@ConfigurationProperties` 机制加载属性.

上面的配置示例都使用了一个使用位置参数而不是命名参数的快捷符号.以下两个例子是等效的：

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

对于网关的一些用法，属性是足够的，但是一些生产环境用例将受益于从外部源（例如数据库）加载配置.未来的里程碑版本将基于Spring Data Repositories实现 `RouteDefinitionLocator` 实现，例如：Redis，MongoDB和Cassandra.

## 118.1 Fluent Java Routes API

为了允许在Java中进行简单配置， `RouteLocatorBuilder`  bean中定义了一个流畅的API.

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

此样式还允许更多自定义谓词断言.  `RouteDefinitionLocator`  beans定义的谓词使用逻辑 `and` 进行组合.通过使用流畅的Java API，您可以在 `Predicate` 类上使用 `and()` ， `or()` 和 `negate()` 运算符.

## 118.2 DiscoveryClient路径定义定位器

可以将网关配置为基于使用 `DiscoveryClient` 兼容服务注册表注册的服务创建路由.

要启用此功能，请设置 `spring.cloud.gateway.discovery.locator.enabled=true` 并确保 `DiscoveryClient` 实现位于类路径上并已启用（例如Netflix Eureka，Consul或Zookeeper）.

