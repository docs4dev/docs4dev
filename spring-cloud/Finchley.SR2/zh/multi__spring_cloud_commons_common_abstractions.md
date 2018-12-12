## 3. Spring Cloud Commons：Common Abstractions

诸如服务发现，负载平衡和断路器之类的模式适用于所有Spring Cloud客户端可以使用的公共抽象层，与实现无关（例如，使用Eureka或Consul进行发现）.

## 3.1 @EnableDiscoveryClient

Spring Cloud Commons提供 `@EnableDiscoveryClient` 注释.这将查找带有 `META-INF/spring.factories` 的 `DiscoveryClient` 接口的实现. Discovery Client的实现将配置类添加到 `org.springframework.cloud.client.discovery.EnableDiscoveryClient` 键下的 `spring.factories` .  `DiscoveryClient` 实现的示例包括[Spring Cloud Netflix Eureka](https://cloud.spring.io/spring-cloud-netflix/)，[Spring Cloud Consul Discovery](https://cloud.spring.io/spring-cloud-consul/)和[Spring Cloud Zookeeper Discovery](https://cloud.spring.io/spring-cloud-zookeeper/).

默认情况下， `DiscoveryClient` 的实现会使用远程发现服务器自动注册本地Spring Boot服务器.通过在 `@EnableDiscoveryClient` 中设置 `autoRegister=false` 可以禁用此行为.

不再需要
>  `@EnableDiscoveryClient` .您可以在类路径上放置 `DiscoveryClient` 实现，以使Spring Boot应用程序向服务发现服务器注册.

### 3.1.1Health指标

Commons创建了一个Spring Boot  `HealthIndicator` ， `DiscoveryClient` 实现可以通过实现 `DiscoveryHealthIndicator` 来参与.要禁用复合 `HealthIndicator` ，请设置 `spring.cloud.discovery.client.composite-indicator.enabled=false` .基于 `DiscoveryClient` 的通用 `HealthIndicator` 是自动配置的（ `DiscoveryClientHealthIndicator` ）.要禁用它，请设置 `spring.cloud.discovery.client.health-indicator.enabled=false` .要禁用 `DiscoveryClientHealthIndicator` 的描述字段，请设置 `spring.cloud.discovery.client.health-indicator.include-description=false` .否则，它可能会随着 `HealthIndicator` 的 `description` 而冒泡.

## 3.2 ServiceRegistry

Commons现在提供了一个 `ServiceRegistry` 接口，提供 `register(Registration)` 和 `deregister(Registration)` 等方法，可以提供自定义注册服务.  `Registration` 是标记界面.

以下示例显示正在使用的 `ServiceRegistry` ：

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

每个 `ServiceRegistry` 实现都有自己的 `Registry` 实现.

-  `ZookeeperRegistration` 与 `ZookeeperServiceRegistry` 一起使用

-  `EurekaRegistration` 与 `EurekaServiceRegistry` 一起使用

-  `ConsulRegistration` 与 `ConsulServiceRegistry` 一起使用

如果您使用 `ServiceRegistry` 接口，则需要为正在使用的 `ServiceRegistry` 实现传递正确的 `Registry` 实现.

### 3.2.1 ServiceRegistry自动注册

默认情况下， `ServiceRegistry` 实现会自动注册正在运行的服务.要禁用该行为，您可以设置：*  `@EnableDiscoveryClient(autoRegister=false)` 永久禁用自动注册. *  `spring.cloud.service-registry.auto-registration.enabled=false` 通过配置禁用行为.

#### ServiceRegistry自动注册事件

当服务自动注册时，将触发两个事件.第一个事件名为 `InstancePreRegisteredEvent` ，在注册服务之前被触发.第二个事件名为 `InstanceRegisteredEvent` ，在注册服务后触发.您可以注册 `ApplicationListener` （s）来收听并回应这些事件.

> 如果 `spring.cloud.service-registry.auto-registration.enabled` 设置为 `false` ，则不会触发这些事件.

### 3.2.2 Service Registry Actuatorendpoints

Spring Cloud Commons提供 `/service-registry` Actuatorendpoints.此endpoints依赖于Spring Application Context中的 `Registration`  bean.使用GET调用 `/service-registry` 将返回 `Registration` 的状态.将POST用于具有JSON主体的同一endpoints会将当前 `Registration` 的状态更改为新值. JSON主体必须包含具有首选值的 `status` 字段.在更新状态和为状态返回的值时，请参阅用于允许值的 `ServiceRegistry` 实现的文档.例如，Eureka支持的状态是 `UP` ， `DOWN` ， `OUT_OF_SERVICE` 和 `UNKNOWN` .

## 3.3 Spring RestTemplate作为负载均衡器客户端

`RestTemplate` 可以自动配置为使用功能区.要创建负载平衡 `RestTemplate` ，请创建 `RestTemplate`   `@Bean` 并使用 `@LoadBalanced` 限定符，如以下示例所示：

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

> A  `RestTemplate`  bean不再通过自动配置创建.个人应用程序必须创建它.

URI需要使用虚拟主机名（即服务名称，而不是主机名）.功能区客户端用于创建完整的物理地址.有关如何设置 `RestTemplate` 的详细信息，请参见[RibbonAutoConfiguration](https://github.com/spring-cloud/spring-cloud-netflix/blob/master/spring-cloud-netflix-core/src/main/java/org/springframework/cloud/netflix/ribbon/RibbonAutoConfiguration.java).

## 3.4 Spring WebClient作为负载均衡器客户端

`WebClient` 可以自动配置为使用 `LoadBalancerClient` .要创建负载平衡 `WebClient` ，请创建 `WebClient.Builder`   `@Bean` 并使用 `@LoadBalanced` 限定符，如以下示例所示：

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

URI需要使用虚拟主机名（即服务名称，而不是主机名）.功能区客户端用于创建完整的物理地址.

### 3.4.1重试失败的请求

可以将负载平衡 `RestTemplate` 配置为重试失败的请求.默认情况下，禁用此逻辑.您可以通过将[Spring Retry](https://github.com/spring-projects/spring-retry)添加到应用程序的类路径来启用它.负载平衡 `RestTemplate` 表示与重试失败请求相关的一些功能区配置值.您可以使用 `client.ribbon.MaxAutoRetries` ， `client.ribbon.MaxAutoRetriesNextServer` 和 `client.ribbon.OkToRetryOnAllOperations` 属性.如果要在类路径上使用Spring Retry禁用重试逻辑，可以设置 `spring.cloud.loadbalancer.retry.enabled=false` .有关这些属性的说明，请参阅[Ribbon documentation](https://github.com/Netflix/ribbon/wiki/Getting-Started#the-properties-file-sample-clientproperties).

如果要在重试中实现 `BackOffPolicy` ，则需要创建 `LoadBalancedRetryFactory` 类型的bean并覆盖 `createBackOffPolicy` 方法：

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

上述示例中的
>  `client` 应替换为Ribbon客户端的名称.

如果要在重试功能中添加一个或多个 `RetryListener` 实现，则需要创建 `LoadBalancedRetryListenerFactory` 类型的bean，并返回要用于给定服务的 `RetryListener` 数组，如以下示例所示：

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

## 3.5多个RestTemplate对象

如果你想要一个没有负载平衡的 `RestTemplate` ，创建一个 `RestTemplate`  bean并注入它.要访问负载平衡 `RestTemplate` ，请在创建 `@Bean` 时使用 `@LoadBalanced` 限定符，如以下示例所示：\

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

|图片/ important.png |重要|
| ---- | ---- |
|请注意在前面示例中的plain  `RestTemplate` 声明中使用 `@Primary` 注释来消除不合格 `@Autowired` 注入的歧义. |

> 如果您看到 `java.lang.IllegalArgumentException: Can not set org.springframework.web.client.RestTemplate field com.my.app.Foo.restTemplate to com.sun.proxy.$Proxy89` 等错误，请尝试注入 `RestOperations` 或设置 `spring.aop.proxyTargetClass=true` .

## 3.6 Spring WebFlux WebClient作为负载均衡器客户端

`WebClient` 可以配置为使用 `LoadBalancerClient` .如果 `spring-webflux` 在类路径上，则自动配置 `LoadBalancerExchangeFilterFunction` .以下示例显示如何配置 `WebClient` 以使用负载均衡器：

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

URI需要使用虚拟主机名（即服务名称，而不是主机名）.  `LoadBalancerClient` 用于创建完整的物理地址.

## 3.7忽略网络接口

有时，忽略某些命名的网络接口以便可以从Service Discovery注册中排除它们（例如，在Docker容器中运行时）是很有用的.可以设置正则表达式列表以使所需的网络接口被忽略.以下配置忽略 `docker0` 接口以及以 `veth` 开头的所有接口：

**application.yml.** 

```java
spring:
cloud:
inetutils:
ignoredInterfaces:
- docker0
- veth.*
```

您还可以使用正则表达式列表强制仅使用指定的网络地址，如以下示例所示：

**bootstrap.yml.** 

```java
spring:
cloud:
inetutils:
preferredNetworks:
- 192.168
- 10.0
```

您还可以强制仅使用站点本地地址，如以下示例所示：.application.yml

```java
spring:
cloud:
inetutils:
useOnlySiteLocalInterfaces: true
```

有关构成站点本地地址的更多详细信息，请参阅[Inet4Address.html.isSiteLocalAddress()](https://docs.oracle.com/javase/8/docs/api/java/net/Inet4Address.html#isSiteLocalAddress--).

## 3.8 HTTP客户端工厂

Spring Cloud Commons提供用于创建Apache HTTP客户端（ `ApacheHttpClientFactory` ）和OK HTTP客户端（ `OkHttpClientFactory` ）的bean.仅当OK HTTP jar在类路径上时才会创建 `OkHttpClientFactory`  bean.此外，Spring Cloud Commons提供了用于创建两个客户端使用的连接管理器的bean： `ApacheHttpClientConnectionManagerFactory` 用于Apache HTTP客户端， `OkHttpClientConnectionPoolFactory` 用于OK HTTP客户端.如果要自定义在下游项目中创建HTTP客户端的方式，可以提供自己的这些bean实现.此外，如果提供 `HttpClientBuilder` 或 `OkHttpClient.Builder` 类型的bean，则默认工厂将使用这些构建器作为返回到下游项目的构建器的基础.您还可以通过将 `spring.cloud.httpclientfactories.apache.enabled` 或 `spring.cloud.httpclientfactories.ok.enabled` 设置为 `false` 来禁用这些bean的创建.

## 3.9已启用的功能

Spring Cloud Commons提供 `/features` Actuatorendpoints.此endpoints返回类路径上可用的功能以及它们是否已启用.返回的信息包括功能类型，名称，版本和供应商.

### 3.9.1要素类型

有两种类型的“特征”：抽象和命名.

抽象功能是定义接口或抽象类以及创建的实现的功能，例如 `DiscoveryClient` ， `LoadBalancerClient` 或 `LockService` .抽象类或接口用于在上下文中查找该类型的bean.显示的版本是 `bean.getClass().getPackage().getImplementationVersion()` .

命名功能是没有他们实现的特定类的功能，例如“断路器”，“API网关”，“Spring Cloud Bus”等.这些功能需要一个名称和 beans类型.

### 3.9.2声明功能

任何模块都可以声明任意数量的 `HasFeature`  bean，如以下示例所示：

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

这些 beans子中的每一个都应该进行适当的保护 `@Configuration` .

