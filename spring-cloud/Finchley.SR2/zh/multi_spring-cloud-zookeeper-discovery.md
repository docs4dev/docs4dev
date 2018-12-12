## 73.使用Zookeeper进行服务发现

Service Discovery是基于微服务的体系结构的关键原则之一.尝试手动配置每个客户端或某种形式的约定可能很难做到并且可能很脆弱. [Curator](https://curator.apache.org)（Zookeeper的Java库）通过[Service Discovery Extension](https://curator.apache.org/curator-x-discovery/)提供服务发现. Spring Cloud Zookeeper使用此扩展进行服务注册和发现.

## 73.1激活

包含对 `org.springframework.cloud:spring-cloud-starter-zookeeper-discovery` 的依赖性可启用设置Spring Cloud Zookeeper Discovery的自动配置.

> 对于网络功能，您仍需要包含 `org.springframework.boot:spring-boot-starter-web` .

> 使用Zookeeper 3.4版时，您需要更改包含依赖关系的方式，如[here](multi_spring-cloud-zookeeper-install.html)所述.

## 73.2注册Zookeeper

当客户端向Zookeeper注册时，它会提供有关自身的元数据（例如主机和端口，ID和名称）.

以下示例显示了Zookeeper客户端：

```java
@SpringBootApplication
@RestController
public class Application {

@RequestMapping("/")
public String home() {
return "Hello world";
}

public static void main(String[] args) {
new SpringApplicationBuilder(Application.class).web(true).run(args);
}

}
```

> 前面的示例是一个普通的Spring Boot应用程序.

如果Zookeeper位于 `localhost:2181` 以外的其他位置，则配置必须提供服务器的位置，如以下示例所示：

**application.yml.** 

```java
spring:
cloud:
zookeeper:
connect-string: localhost:2181
```

> 如果使用[Spring Cloud Zookeeper Config](multi_spring-cloud-zookeeper-config.html)，则前面示例中显示的值必须是 `bootstrap.yml` 而不是 `application.yml` .

默认服务名称，实例ID和端口（取自 `Environment` ）分别是 `${spring.application.name}` ，Spring Context ID和 `${server.port}` .

在类路径上使用 `spring-cloud-starter-zookeeper-discovery` 使应用程序成为Zookeeper“服务”（即，它自己注册）和“客户端”（也就是说，它可以查询Zookeeper以查找其他服务）.

如果要禁用Zookeeper Discovery Client，可以将 `spring.cloud.zookeeper.discovery.enabled` 设置为 `false` .

## 73.3使用DiscoveryClient

Spring Cloud支持[Feign](https://github.com/spring-cloud/spring-cloud-netflix/blob/master/docs/src/main/asciidoc/spring-cloud-netflix.adoc#spring-cloud-feign)（REST客户端构建器）和[Spring RestTemplate](https://github.com/spring-cloud/spring-cloud-netflix/blob/master/docs/src/main/asciidoc/spring-cloud-netflix.adoc#spring-cloud-ribbon)，使用逻辑服务名称而不是物理URL.

您还可以使用 `org.springframework.cloud.client.discovery.DiscoveryClient` ，它为不特定于Netflix的发现客户端提供简单的API，如以下示例所示：

```java
@Autowired
private DiscoveryClient discoveryClient;

public String serviceUrl() {
List<ServiceInstance> list = discoveryClient.getInstances("STORES");
if (list != null && list.size() > 0 ) {
return list.get(0).getUri().toString();
}
return null;
}
```

