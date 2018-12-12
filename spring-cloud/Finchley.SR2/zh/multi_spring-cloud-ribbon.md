## 16.客户端负载均衡器：功能区

Ribbon是一个客户端负载均衡器，可以让您对HTTP和TCP客户端的行为进行大量控制. Feign已使用Ribbon，因此，如果您使用 `@FeignClient` ，则此部分也适用.

Ribbon中的一个核心概念是指定客户端的概念.每个负载均衡器都是一组组件的一部分，这些组件一起工作以按需联系远程服务器，并且该集合具有您作为应用程序开发人员提供的名称（例如，通过使用 `@FeignClient` 注释）.根据需要，Spring Cloud通过使用 `RibbonClientConfiguration` 为每个命名客户端创建一个新的集合 `ApplicationContext` .这包含（除其他外） `ILoadBalancer` ， `RestClient` 和 `ServerListFilter` .

## 16.1如何包含功能区

要在项目中包含功能区，请使用组ID为 `org.springframework.cloud` 且工件ID为 `spring-cloud-starter-netflix-ribbon` 的启动器.有关使用当前Spring Cloud Release Train设置构建系统的详细信息，请参阅[Spring Cloud Project page](https://projects.spring.io/spring-cloud/).

## 16.2自定义功能区客户端

您可以使用 `<client>.ribbon.*` 中的外部属性配置Ribbon客户端的某些位，这类似于本机使用Netflix API，但您可以使用Spring Boot配置文件.可以在[CommonClientConfigKey](https://github.com/Netflix/ribbon/blob/master/ribbon-core/src/main/java/com/netflix/client/config/CommonClientConfigKey.java)（带状内核的一部分）中将本机选项作为静态字段进行检查.

Spring Cloud还允许您通过使用 `@RibbonClient` 声明其他配置（在 `RibbonClientConfiguration` 之上）来完全控制客户端，如以下示例所示：

```java
@Configuration
@RibbonClient(name = "custom", configuration = CustomConfiguration.class)
public class TestConfiguration {
}
```

在这种情况下，客户端由 `RibbonClientConfiguration` 中已有的组件以及 `CustomConfiguration` 中的任何组件组成（后者通常会覆盖前者）.

>   `CustomConfiguration`  clas必须是 `@Configuration` 类，但请注意它不在主应用程序上下文的 `@ComponentScan` 中.否则，所有 `@RibbonClients` 共享它.如果您使用 `@ComponentScan` （或 `@SpringBootApplication` ），则需要使用避免包含它的步骤（例如，您可以将它放在一个单独的，不重叠的包中，或指定要在 `@ComponentScan` 中显式扫描的包）.

下表显示了Spring Cloud Netflix默认为Ribbon提供的bean：

| Bean类型| Bean名称|类名称|
| ---- | ---- | ---- |
|  `IClientConfig`  |  `ribbonClientConfig`  |  `DefaultClientConfigImpl`  |
|  `IRule`  |  `ribbonRule`  |  `ZoneAvoidanceRule`  |
|  `IPing`  |  `ribbonPing`  |  `DummyPing`  |
|  `ServerList<Server>`  |  `ribbonServerList`  |  `ConfigurationBasedServerList`  |
|  `ServerListFilter<Server>`  |  `ribbonServerListFilter`  |  `ZonePreferenceServerListFilter`  |
|  `ILoadBalancer`  |  `ribbonLoadBalancer`  |  `ZoneAwareLoadBalancer`  |
|  `ServerListUpdater`  |  `ribbonServerListUpdater`  |  `PollingServerListUpdater`  |

创建其中一种类型的bean并将其置于 `@RibbonClient` 配置（例如上面的 `FooConfiguration` ）中，可以覆盖所描述的每个bean，如以下示例所示：

```java
@Configuration
protected static class FooConfiguration {
	@Bean
	public ZonePreferenceServerListFilter serverListFilter() {
		ZonePreferenceServerListFilter filter = new ZonePreferenceServerListFilter();
		filter.setZone("myTestZone");
		return filter;
	}

	@Bean
	public IPing ribbonPing() {
		return new PingUrl();
	}
}
```

前面示例中的include语句将 `NoOpPing` 替换为 `PingUrl` 并提供自定义 `serverListFilter` .

## 16.3自定义所有功能区客户端的默认值

可以使用 `@RibbonClients` 注释并注册默认配置为所有功能区客户端提供默认配置，如以下示例所示：

```java
@RibbonClients(defaultConfiguration = DefaultRibbonConfig.class)
public class RibbonClientDefaultConfigurationTestsConfig {

	public static class BazServiceList extends ConfigurationBasedServerList {
		public BazServiceList(IClientConfig config) {
			super.initWithNiwsConfig(config);
		}
	}
}

@Configuration
class DefaultRibbonConfig {

	@Bean
	public IRule ribbonRule() {
		return new BestAvailableRule();
	}

	@Bean
	public IPing ribbonPing() {
		return new PingUrl();
	}

	@Bean
	public ServerList<Server> ribbonServerList(IClientConfig config) {
		return new RibbonClientDefaultConfigurationTestsConfig.BazServiceList(config);
	}

	@Bean
	public ServerListSubsetFilter serverListFilter() {
		ServerListSubsetFilter filter = new ServerListSubsetFilter();
		return filter;
	}

}
```

## 16.4通过设置属性自定义功能区客户端

从版本1.2.0开始，Spring Cloud Netflix现在支持通过将属性设置为与[Ribbon documentation](https://github.com/Netflix/ribbon/wiki/Working-with-load-balancers#components-of-load-balancer)兼容来自定义功能区客户端.

这使您可以在不同环境中的启动时更改行为.

以下列表显示了支持的属性>：

-  `<clientName>.ribbon.NFLoadBalancerClassName` ：应该实施 `ILoadBalancer` 

-  `<clientName>.ribbon.NFLoadBalancerRuleClassName` ：应该实施 `IRule` 

-  `<clientName>.ribbon.NFLoadBalancerPingClassName` ：应该实施 `IPing` 

-  `<clientName>.ribbon.NIWSServerListClassName` ：应该实施 `ServerList` 

-  `<clientName>.ribbon.NIWSServerListFilterClassName` ：应该实施 `ServerListFilter` 

> 这些属性中定义的类优先于使用 `@RibbonClient(configuration=MyRibbonConfig.class)` 定义的bean以及Spring Cloud Netflix提供的默认值.

要为名为 `users` 的服务名称设置 `IRule` ，可以设置以下属性：

**application.yml.** 

```java
users:
ribbon:
NIWSServerListClassName: com.netflix.loadbalancer.ConfigurationBasedServerList
NFLoadBalancerRuleClassName: com.netflix.loadbalancer.WeightedResponseTimeRule
```

有关Ribbon提供的实现，请参阅[Ribbon documentation](https://github.com/Netflix/ribbon/wiki/Working-with-load-balancers).

## 16.5使用Eureka功能区

当Eureka与Ribbon一起使用时（即两者都在类路径上）， `ribbonServerList` 被覆盖，扩展名为 `DiscoveryEnabledNIWSServerList` ，它填充了Eureka的服务器列表.它还将 `IPing` 接口替换为 `NIWSDiscoveryPing` ，后者委托Eureka确定服务器是否已启动.默认安装的 `ServerList` 是 `DomainExtractingServerList` .其目的是在不使用AWS AMI元数据的情况下使负载均衡器可以使用元数据（这是Netflix所依赖的）.默认情况下，服务器列表使用“区域”信息构建，如实例元数据中所提供的（因此，在远程客户端上，设置 `eureka.instance.metadataMap.zone` ）.如果缺少该标志并且设置了 `approximateZoneFromHostname` 标志，则可以使用服务器主机名中的域名作为区域的代理.区域信息可用后，可以在_11158中使用.默认情况下，它用于在与客户端相同的区域中查找服务器，因为默认值为 `ZonePreferenceServerListFilter` .默认情况下，客户端区域的确定方式与远程实例相同（即通过 `eureka.instance.metadataMap.zone` ）.

> 设置客户区的正统“archaius”方法是通过名为"@zone"的配置属性.如果可用，Spring Cloud优先于所有其他设置使用它（请注意，必须在YAML配置中引用密钥）.

> 如果没有其他区域数据源，则根据客户端配置（与实例配置相反）进行猜测.我们将 `eureka.client.availabilityZones` ，即从区域名称到区域列表的映射，并拉出实例自己区域的第一个区域（即 `eureka.client.region` ，默认为"us-east-1"，以便与本机Netflix兼容）.

## 16.6示例：如何在没有Eureka的情况下使用功能区

Eureka是一种抽象远程服务器发现的便捷方式，因此您无需在客户端中对其URL进行硬编码.但是，如果您不想使用Eureka，Ribbon和Feign也可以使用.假设您已为"stores"声明 `@RibbonClient` ，而Eureka未使用（甚至在类路径中也没有）.功能区客户端默认为已配置的服务器列表.您可以按如下方式提供配置：

**application.yml.** 

```java
stores:
ribbon:
listOfServers: example.com,google.com
```

## 16.7示例：禁用功能区中的Eureka使用

将 `ribbon.eureka.enabled` 属性设置为 `false` 会显式禁用在功能区中使用Eureka，如以下示例所示：

**application.yml.** 

```java
ribbon:
eureka:
enabled: false
```

## 16.8直接使用Ribbon API

您也可以直接使用 `LoadBalancerClient` ，如以下示例所示：

```java
public class MyClass {
@Autowired
private LoadBalancerClient loadBalancer;

public void doStuff() {
ServiceInstance instance = loadBalancer.choose("stores");
URI storesUri = URI.create(String.format("http://%s:%s", instance.getHost(), instance.getPort()));
// ... do something with the URI
}
}
```

## 16.9功能区配置缓存

每个名为client的功能区都有一个Spring Cloud维护的相应子应用程序Context.此应用程序上下文在第一次请求指向客户端时被延迟加载.通过指定Ribbon客户端的名称，可以将此延迟加载行为更改为在启动时急切地加载这些子应用程序上下文，如以下示例所示：

**application.yml.** 

```java
ribbon:
eager-load:
enabled: true
clients: client1, client2, client3
```

## 16.10如何配置Hystrix线程池

如果将 `zuul.ribbonIsolationStrategy` 更改为 `THREAD` ，则为线程隔离策略Hystrix用于所有路线.在这种情况下， `HystrixThreadPoolKey` 设置为 `RibbonCommand` 作为默认值.这意味着所有路由的HystrixCommands都在同一个Hystrix线程池中执行.可以使用以下配置更改此行为：

**application.yml.** 

```java
zuul:
threadPool:
useSeparateThreadPools: true
```

前面的示例导致HystrixCommands在Hystrix线程池中为每个路由执行.

在这种情况下，默认 `HystrixThreadPoolKey` 与每个路由的服务ID相同.要为 `HystrixThreadPoolKey` 添加前缀，请将 `zuul.threadPool.threadPoolKeyPrefix` 设置为要添加的值，如以下示例所示：

**application.yml.** 

```java
zuul:
threadPool:
useSeparateThreadPools: true
threadPoolKeyPrefix: zuulgw
```

## 16.11如何为Ribbon的IRule提供一个键

如果您需要提供自己的 `IRule` 实现来处理特殊的路由要求，如“canary”测试，请将一些信息传递给 `IRule` 的 `choose` 方法.

**com.netflix.loadbalancer.IRule.java.** 

```java
public interface IRule{
public Server choose(Object key);
:
```

您可以提供 `IRule` 实现用于选择目标服务器的一些信息，如以下示例所示：

```java
RequestContext.getCurrentContext()
.set(FilterConstants.LOAD_BALANCER_KEY, "canary-test");
```

如果使用 `FilterConstants.LOAD_BALANCER_KEY` 的键将任何对象放入 `RequestContext` ，则会将其传递给 `IRule` 实现的 `choose` 方法.必须在执行 `RibbonRoutingFilter` 之前执行前面示例中显示的代码. Zuul的预过滤器是最好的选择.您可以通过预过滤器中的 `RequestContext` 访问HTTP标头和查询参数，因此可以使用它来确定传递给Ribbon的 `LOAD_BALANCER_KEY` .如果在 `RequestContext` 中没有为 `LOAD_BALANCER_KEY` 添加任何值，则将null作为 `choose` 方法的参数传递.

