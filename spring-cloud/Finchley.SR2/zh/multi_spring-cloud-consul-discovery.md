## 66.带Consul的服务发现

Service Discovery是基于微服务的体系结构的关键原则之一.尝试手动配置每个客户端或某种形式的约定可能非常困难，并且可能非常脆弱. Consul通过[HTTP API](https://www.consul.io/docs/agent/http.html)和[DNS](https://www.consul.io/docs/agent/dns.html)提供服务发现服务. Spring Cloud Consul利用HTTP API进行服务注册和发现.这并不妨碍非Spring Cloud应用程序利用DNS接口. Consul Agents服务器在[cluster](https://www.consul.io/docs/internals/architecture.html)中运行，通过[gossip protocol](https://www.consul.io/docs/internals/gossip.html)进行通信并使用[Raft consensus protocol](https://www.consul.io/docs/internals/consensus.html).

## 66.1如何激活

要激活Consul Service Discovery，请使用具有组 `org.springframework.cloud` 和工件ID  `spring-cloud-starter-consul-discovery` 的启动器.有关使用当前Spring Cloud Release Train设置构建系统的详细信息，请参阅[Spring Cloud Project page](https://projects.spring.io/spring-cloud/).

## 66.2向Consul登记

当客户向Consul注册时，它会提供有关自身的元数据，例如主机和端口，ID，名称和标签.默认情况下会创建HTTP [Check](https://www.consul.io/docs/agent/checks.html)，Consul每10秒触及一次 `/health` endpoints.如果运行状况检查失败，则将服务实例标记为严重.

示例Consul客户端：

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

（即完全正常的Spring Boot应用程序）.如果Consul客户端位于 `localhost:8500` 以外的其他位置，则需要配置以查找客户端.例：

**application.yml.** 

```java
spring:
cloud:
consul:
host: localhost
port: 8500
```

> 如果使用[Spring Cloud Consul Config](multi_spring-cloud-consul-config.html)，则需要将上述值放在 `bootstrap.yml` 而不是 `application.yml` 中.

从 `Environment` 获取的默认服务名称，实例ID和端口分别是 `${spring.application.name}` ，Spring Context ID和 `${server.port}` .

要禁用Consul Discovery Client，您可以将 `spring.cloud.consul.discovery.enabled` 设置为 `false` .

要禁用服务注册你可以设置 `spring.cloud.consul.discovery.register` 到 `false` .

## 66.3 HTTP运行状况检查

Consul实例的运行状况检查默认为"/health"，这是Spring Boot Actuator应用程序中有用endpoints的默认位置.如果使用非默认上下文路径或servlet路径（例如 `server.servletPath=/foo` ）或管理endpoints路径（例如 `management.server.servlet.context-path=/admin` ），则需要更改这些，即使对于Actuator应用程序也是如此.还可以配置Consul用于检查Healthendpoints的时间间隔. "10s"和"1m"分别代表10秒和1分钟.例：

**application.yml.** 

```java
spring:
cloud:
consul:
discovery:
healthCheckPath: ${management.server.servlet.context-path}/health
healthCheckInterval: 15s
```

您可以通过设置 `management.health.consul.enabled=false` 来禁用运行状况检查.

### 66.3.1元数据和Consul标签

Consul尚不支持服务元数据. Spring Cloud的 `ServiceInstance` 有一个 `Map<String, String> metadata` 字段. Spring Consul使用Consul标签来近似元数据，直到Consul正式支持元数据.形式为 `key=value` 的标签将被拆分并分别用作 `Map` 键和值.没有相同 `=` 符号的标签将用作键和值.

**application.yml.** 

```java
spring:
cloud:
consul:
discovery:
tags: foo=bar, baz
```

以上配置将生成带有 `foo→bar` 和 `baz→baz` 的Map.

### 66.3.2使Consul实例ID唯一

默认情况下，consul实例注册的ID等于其Spring Application Context ID.默认情况下，Spring Application Context ID为 `${spring.application.name}:comma,separated,profiles:${server.port}` .对于大多数情况，这将允许一台服务的多个实例在一台计算机上运行.如果需要进一步的唯一性，使用Spring Cloud可以通过在 `spring.cloud.consul.discovery.instanceId` 中提供唯一标识符来覆盖它.例如：

**application.yml.** 

```java
spring:
cloud:
consul:
discovery:
instanceId: ${spring.application.name}:${vcap.application.instance_id:${spring.application.instance_id:${random.value}}}
```

使用此元数据以及在localhost上部署的多个服务实例，随机值将在那里启动以使实例唯一.在Cloudfoundry中， `vcap.application.instance_id` 将在Spring Boot应用程序中自动填充，因此不需要随机值.

## 66.4查找服务

### 66.4.1使用功能区

Spring Cloud支持[Feign](https://github.com/spring-cloud/spring-cloud-netflix/blob/master/docs/src/main/asciidoc/spring-cloud-netflix.adoc#spring-cloud-feign)（一个REST客户端构建器）和[Spring RestTemplate](https://github.com/spring-cloud/spring-cloud-netflix/blob/master/docs/src/main/asciidoc/spring-cloud-netflix.adoc#spring-cloud-ribbon)，用于使用逻辑服务名称/ ID而不是物理URL查找服务. Feign和发现感知RestTemplate都使用[Ribbon](https://cloud.spring.io/spring-cloud-netflix/single/spring-cloud-netflix.html#spring-cloud-ribbon)进行客户端负载平衡.

如果您想使用RestTemplate访问服务STORES，只需声明：

```java
@LoadBalanced
@Bean
public RestTemplate loadbalancedRestTemplate() {
new RestTemplate();
}
```

并像这样使用它（注意我们如何使用Consul的STORES服务名称/ id而不是完全限定的域名）：

```java
@Autowired
RestTemplate restTemplate;

public String getFirstProduct() {
return this.restTemplate.getForObject("https://STORES/products/1", String.class);
}
```

如果您在多个数据中心中拥有Consul群集，并且您想要访问另一个数据中心中的服务，则仅服务名称/ ID是不够的.在这种情况下，您使用属性 `spring.cloud.consul.discovery.datacenters.STORES=dc-west` ，其中 `STORES` 是服务名称/ id， `dc-west` 是STORES服务所在的数据中心.

### 66.4.2使用DiscoveryClient

您还可以使用 `org.springframework.cloud.client.discovery.DiscoveryClient` ，它为不特定于Netflix的发现客户端提供简单的API，例如：

```java
@Autowired
private DiscoveryClient discoveryClient;

public String serviceUrl() {
List<ServiceInstance> list = discoveryClient.getInstances("STORES");
if (list != null && list.size() > 0 ) {
return list.get(0).getUri();
}
return null;
}
```

## 66.5Consul目录观察

Consul目录观察利用了Consul的能力[watch services](https://www.consul.io/docs/agent/watches.html#services). Catalog Watch会阻止Consul HTTP API调用，以确定是否有任何服务已更改.如果有新的服务数据，则发布心跳事件.

要更改调用Config Watch的频率，请更改 `spring.cloud.consul.config.discovery.catalog-services-watch-delay` .默认值为1000，以毫秒为单位.延迟是上一次调用结束和下一次调用开始之后的时间量.

禁用目录监视集 `spring.cloud.consul.discovery.catalogServicesWatch.enabled=false` .

Watch使用Spring  `TaskScheduler` 来安排对Consul的调用.默认情况下，它是 `ThreadPoolTaskScheduler` ， `poolSize` 为1.要更改 `TaskScheduler` ，请创建一个名为 `TaskScheduler` 的bean，并使用 `ConsulDiscoveryClientConfiguration.CATALOG_WATCH_TASK_SCHEDULER_NAME` 常量命名.

