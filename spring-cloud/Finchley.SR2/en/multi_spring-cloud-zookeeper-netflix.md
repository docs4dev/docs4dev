## 74. Using Spring Cloud Zookeeper with Spring Cloud Netflix Components

Spring Cloud Netflix supplies useful tools that work regardless of which  `DiscoveryClient`  implementation you use. Feign, Turbine, Ribbon, and Zuul all work with Spring Cloud Zookeeper.

## 74.1 Ribbon with Zookeeper

Spring Cloud Zookeeper provides an implementation of Ribbon’s  `ServerList` . When you use the  `spring-cloud-starter-zookeeper-discovery` , Ribbon is autoconfigured to use the  `ZookeeperServerList`  by default.

