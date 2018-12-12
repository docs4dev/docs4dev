## 74.将Spring Cloud Zookeeper与Spring Cloud Netflix组件配合使用

Spring Cloud Netflix提供了有用的工具，无论您使用哪种 `DiscoveryClient` 实现，它都能正常工作. Feign，Turbine，Ribbon和Zuul都可以与Spring Cloud Zookeeper配合使用.

## 74.1带Zookeeper的功能区

Spring Cloud Zookeeper提供了Ribbon的 `ServerList` 的实现.使用 `spring-cloud-starter-zookeeper-discovery` 时，功能区自动配置为默认使用 `ZookeeperServerList` .

