## 16. Client Side Load Balancer: Ribbon

Ribbon is a client-side load balancer that gives you a lot of control over the behavior of HTTP and TCP clients. Feign already uses Ribbon, so, if you use  `@FeignClient` , this section also applies.

A central concept in Ribbon is that of the named client. Each load balancer is part of an ensemble of components that work together to contact a remote server on demand, and the ensemble has a name that you give it as an application developer (for example, by using the  `@FeignClient`  annotation). On demand, Spring Cloud creates a new ensemble as an  `ApplicationContext`  for each named client by using  `RibbonClientConfiguration` . This contains (amongst other things) an  `ILoadBalancer` , a  `RestClient` , and a  `ServerListFilter` .

## 16.1 How to Include Ribbon

To include Ribbon in your project, use the starter with a group ID of  `org.springframework.cloud`  and an artifact ID of  `spring-cloud-starter-netflix-ribbon` . See the [Spring Cloud Project page](https://projects.spring.io/spring-cloud/) for details on setting up your build system with the current Spring Cloud Release Train.

## 16.2 Customizing the Ribbon Client

You can configure some bits of a Ribbon client by using external properties in  `<client>.ribbon.*` , which is similar to using the Netflix APIs natively, except that you can use Spring Boot configuration files. The native options can be inspected as static fields in [CommonClientConfigKey](https://github.com/Netflix/ribbon/blob/master/ribbon-core/src/main/java/com/netflix/client/config/CommonClientConfigKey.java) (part of ribbon-core).

Spring Cloud also lets you take full control of the client by declaring additional configuration (on top of the  `RibbonClientConfiguration` ) using  `@RibbonClient` , as shown in the following example:

```java
@Configuration
@RibbonClient(name = "custom", configuration = CustomConfiguration.class)
public class TestConfiguration {
}
```

In this case, the client is composed from the components already in  `RibbonClientConfiguration` , together with any in  `CustomConfiguration`  (where the latter generally overrides the former).

> The  `CustomConfiguration`  clas must be a  `@Configuration`  class, but take care that it is not in a  `@ComponentScan`  for the main application context. Otherwise, it is shared by all the  `@RibbonClients` . If you use  `@ComponentScan`  (or  `@SpringBootApplication` ), you need to take steps to avoid it being included (for instance, you can put it in a separate, non-overlapping package or specify the packages to scan explicitly in the  `@ComponentScan` ).

The following table shows the beans that Spring Cloud Netflix provides by default for Ribbon:

|Bean Type|Bean Name|Class Name|
|----|----|----|
| `IClientConfig`  | `ribbonClientConfig`  | `DefaultClientConfigImpl`  |
| `IRule`  | `ribbonRule`  | `ZoneAvoidanceRule`  |
| `IPing`  | `ribbonPing`  | `DummyPing`  |
| `ServerList<Server>`  | `ribbonServerList`  | `ConfigurationBasedServerList`  |
| `ServerListFilter<Server>`  | `ribbonServerListFilter`  | `ZonePreferenceServerListFilter`  |
| `ILoadBalancer`  | `ribbonLoadBalancer`  | `ZoneAwareLoadBalancer`  |
| `ServerListUpdater`  | `ribbonServerListUpdater`  | `PollingServerListUpdater`  |

Creating a bean of one of those type and placing it in a  `@RibbonClient`  configuration (such as  `FooConfiguration`  above) lets you override each one of the beans described, as shown in the following example:

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

The include statement in the preceding example replaces  `NoOpPing`  with  `PingUrl`  and provides a custom  `serverListFilter` .

## 16.3 Customizing the Default for All Ribbon Clients

A default configuration can be provided for all Ribbon Clients by using the  `@RibbonClients`  annotation and registering a default configuration, as shown in the following example:

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

## 16.4 Customizing the Ribbon Client by Setting Properties

Starting with version 1.2.0, Spring Cloud Netflix now supports customizing Ribbon clients by setting properties to be compatible with the [Ribbon documentation](https://github.com/Netflix/ribbon/wiki/Working-with-load-balancers#components-of-load-balancer).

This lets you change behavior at start up time in different environments.

The following list shows the supported properties>:

-  `<clientName>.ribbon.NFLoadBalancerClassName` : Should implement  `ILoadBalancer` 

-  `<clientName>.ribbon.NFLoadBalancerRuleClassName` : Should implement  `IRule` 

-  `<clientName>.ribbon.NFLoadBalancerPingClassName` : Should implement  `IPing` 

-  `<clientName>.ribbon.NIWSServerListClassName` : Should implement  `ServerList` 

-  `<clientName>.ribbon.NIWSServerListFilterClassName` : Should implement  `ServerListFilter` 

> Classes defined in these properties have precedence over beans defined by using  `@RibbonClient(configuration=MyRibbonConfig.class)`  and the defaults provided by Spring Cloud Netflix.

To set the  `IRule`  for a service name called  `users` , you could set the following properties:

**application.yml.**  

```java
users:
ribbon:
NIWSServerListClassName: com.netflix.loadbalancer.ConfigurationBasedServerList
NFLoadBalancerRuleClassName: com.netflix.loadbalancer.WeightedResponseTimeRule
```

See the [Ribbon documentation](https://github.com/Netflix/ribbon/wiki/Working-with-load-balancers) for implementations provided by Ribbon.

## 16.5 Using Ribbon with Eureka

When Eureka is used in conjunction with Ribbon (that is, both are on the classpath), the  `ribbonServerList`  is overridden with an extension of  `DiscoveryEnabledNIWSServerList` , which populates the list of servers from Eureka. It also replaces the  `IPing`  interface with  `NIWSDiscoveryPing` , which delegates to Eureka to determine if a server is up. The  `ServerList`  that is installed by default is a  `DomainExtractingServerList` . Its purpose is to make metadata available to the load balancer without using AWS AMI metadata (which is what Netflix relies on). By default, the server list is constructed with “zone” information, as provided in the instance metadata (so, on the remote clients, set  `eureka.instance.metadataMap.zone` ). If that is missing and if the  `approximateZoneFromHostname`  flag is set, it can use the domain name from the server hostname as a proxy for the zone. Once the zone information is available, it can be used in a  `ServerListFilter` . By default, it is used to locate a server in the same zone as the client, because the default is a  `ZonePreferenceServerListFilter` . By default, the zone of the client is determined in the same way as the remote instances (that is, through  `eureka.instance.metadataMap.zone` ).

> The orthodox “archaius” way to set the client zone is through a configuration property called "@zone". If it is available, Spring Cloud uses that in preference to all other settings (note that the key must be quoted in YAML configuration).

> If there is no other source of zone data, then a guess is made, based on the client configuration (as opposed to the instance configuration). We take  `eureka.client.availabilityZones` , which is a map from region name to a list of zones, and pull out the first zone for the instance’s own region (that is, the  `eureka.client.region` , which defaults to "us-east-1", for compatibility with native Netflix).

## 16.6 Example: How to Use Ribbon Without Eureka

Eureka is a convenient way to abstract the discovery of remote servers so that you do not have to hard code their URLs in clients. However, if you prefer not to use Eureka, Ribbon and Feign also work. Suppose you have declared a  `@RibbonClient`  for "stores", and Eureka is not in use (and not even on the classpath). The Ribbon client defaults to a configured server list. You can supply the configuration as follows:

**application.yml.**  

```java
stores:
ribbon:
listOfServers: example.com,google.com
```

## 16.7 Example: Disable Eureka Use in Ribbon

Setting the  `ribbon.eureka.enabled`  property to  `false`  explicitly disables the use of Eureka in Ribbon, as shown in the following example:

**application.yml.**  

```java
ribbon:
eureka:
enabled: false
```

## 16.8 Using the Ribbon API Directly

You can also use the  `LoadBalancerClient`  directly, as shown in the following example:

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

## 16.9 Caching of Ribbon Configuration

Each Ribbon named client has a corresponding child application Context that Spring Cloud maintains. This application context is lazily loaded on the first request to the named client. This lazy loading behavior can be changed to instead eagerly load these child application contexts at startup, by specifying the names of the Ribbon clients, as shown in the following example:

**application.yml.**  

```java
ribbon:
eager-load:
enabled: true
clients: client1, client2, client3
```

## 16.10 How to Configure Hystrix Thread Pools

If you change  `zuul.ribbonIsolationStrategy`  to  `THREAD` , the thread isolation strategy for Hystrix is used for all routes. In that case, the  `HystrixThreadPoolKey`  is set to  `RibbonCommand`  as the default. It means that HystrixCommands for all routes are executed in the same Hystrix thread pool. This behavior can be changed with the following configuration:

**application.yml.**  

```java
zuul:
threadPool:
useSeparateThreadPools: true
```

The preceding example results in HystrixCommands being executed in the Hystrix thread pool for each route.

In this case, the default  `HystrixThreadPoolKey`  is the same as the service ID for each route. To add a prefix to  `HystrixThreadPoolKey` , set  `zuul.threadPool.threadPoolKeyPrefix`  to the value that you want to add, as shown in the following example:

**application.yml.**  

```java
zuul:
threadPool:
useSeparateThreadPools: true
threadPoolKeyPrefix: zuulgw
```

## 16.11 How to Provide a Key to Ribbon’s IRule

If you need to provide your own  `IRule`  implementation to handle a special routing requirement like a “canary” test, pass some information to the  `choose`  method of  `IRule` .

**com.netflix.loadbalancer.IRule.java.**  

```java
public interface IRule{
public Server choose(Object key);
:
```

You can provide some information that is used by your  `IRule`  implementation to choose a target server, as shown in the following example:

```java
RequestContext.getCurrentContext()
.set(FilterConstants.LOAD_BALANCER_KEY, "canary-test");
```

If you put any object into the  `RequestContext`  with a key of  `FilterConstants.LOAD_BALANCER_KEY` , it is passed to the  `choose`  method of the  `IRule`  implementation. The code shown in the preceding example must be executed before  `RibbonRoutingFilter`  is executed. Zuul’s pre filter is the best place to do that. You can access HTTP headers and query parameters through the  `RequestContext`  in pre filter, so it can be used to determine the  `LOAD_BALANCER_KEY`  that is passed to Ribbon. If you do not put any value with  `LOAD_BALANCER_KEY`  in  `RequestContext` , null is passed as a parameter of the  `choose`  method.

