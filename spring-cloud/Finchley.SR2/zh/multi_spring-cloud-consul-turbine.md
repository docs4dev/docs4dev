## 71. Hystrix指标与Turbine和Consul汇总

Turbine（由Spring Cloud Netflix项目提供）聚合多个Hystrix指标流实例，因此仪表板可以显示聚合视图. Turbine使用 `DiscoveryClient` 接口查找相关实例.要将Turbine与Spring Cloud Consul一起使用，请以类似于以下示例的方式配置Turbine应用程序：

**pom.xml.** 

```xml
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-netflix-turbine</artifactId>
</dependency>
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-starter-consul-discovery</artifactId>
</dependency>
```

请注意，Turbine依赖性不是启动器.涡轮启动器包括对Netflix Eureka的支持.

**application.yml.** 

```java
spring.application.name: turbine
applications: consulhystrixclient
turbine:
aggregator:
clusterConfig: ${applications}
appConfig: ${applications}
```

`clusterConfig` 和 `appConfig` 部分必须匹配，因此将逗号分隔的服务列表放在一起很有用ID是一个单独的配置属性.

**Turbine.java.** 

```java
@EnableTurbine
@SpringBootApplication
public class Turbine {
public static void main(String[] args) {
SpringApplication.run(DemoturbinecommonsApplication.class, args);
}
}
```

