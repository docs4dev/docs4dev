## 11.服务发现：Eureka客户

Service Discovery是基于微服务的体系结构的关键原则之一.尝试手动配置每个客户端或某种形式的约定可能很难做到并且可能很脆弱. Eureka是Netflix服务发现服务器和客户端.可以配置和部署服务器以使其具有高可用性，每个服务器将关于已注册服务的状态复制到其他服务器.

## 11.1如何包含Eureka客户端

要在项目中包含Eureka Client，请使用组ID为 `org.springframework.cloud` 且工件ID为 `spring-cloud-starter-netflix-eureka-client` 的starter.有关使用当前Spring Cloud Release Train设置构建系统的详细信息，请参阅[Spring Cloud Project page](https://projects.spring.io/spring-cloud/).

## 11.2在Eureka注册

当客户向Eureka注册时，它会提供有关自身的元数据 - 例如主机，端口，运行状况指示器URL，主页和其他详细信息. Eureka从属于服务的每个实例接收心跳消息.如果心跳故障超过可配置的时间表，则通常会从注册表中删除该实例.

以下示例显示了最小的Eureka客户端应用程序：

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

请注意，前面的示例显示了一个普通的[Spring Boot](https://projects.spring.io/spring-boot/)应用程序.通过在类路径上使用 `spring-cloud-starter-netflix-eureka-client` ，您的应用程序会自动向Eureka Server注册.找到Eureka服务器需要进行配置，如以下示例所示：

**application.yml.** 

```java
eureka:
client:
serviceUrl:
defaultZone: http://localhost:8761/eureka/
```

在前面的示例中，“defaultZone”是一个魔术字符串后备值，它为任何不表示首选项的客户端提供服务URL（换句话说，它是一个有用的默认值）.

默认应用程序名称（即服务ID），虚拟主机和非安全端口（取自 `Environment` ）分别为 `${spring.application.name}` ， `${spring.application.name}` 和 `${server.port}` .

在类路径上使用 `spring-cloud-starter-netflix-eureka-client` 使应用程序成为Eureka“实例”（即，它自己注册）和“客户端”（它可以查询注册表以查找其他服务）.实例行为由 `eureka.instance.*` 配置键驱动，但如果您确保应用程序具有 `spring.application.name` 的值（这是Eureka服务ID或VIP的默认值），则默认值很好.

有关可配置选项的更多详细信息，请参见[EurekaInstanceConfigBean](https://github.com/spring-cloud/spring-cloud-netflix/tree/master/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaInstanceConfigBean.java)和[EurekaClientConfigBean](https://github.com/spring-cloud/spring-cloud-netflix/tree/master/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaClientConfigBean.java).

要禁用Eureka Discovery Client，可以将 `eureka.client.enabled` 设置为 `false` .

## 11.3使用Eureka Server进行身份验证

如果其中一个 `eureka.client.serviceUrl.defaultZone`  URL中嵌入了凭据，则会自动将HTTP基本身份验证添加到您的eureka客户端（卷曲样式，如下所示： `http://user:[email protected]:8761/eureka` ）.对于更复杂的需求，您可以创建 `@Bean` 类型 `DiscoveryClientOptionalArgs` 并将 `ClientFilter` 实例注入其中，所有实例都应用于从客户端到服务器的调用.

> B由于Eureka的限制，无法支持每服务器基本身份验证凭据，因此仅使用找到的第一个设置.

## 11.4状态页面和Health指标

Eureka实例的状态页面和运行状况指示器分别默认为 `/info` 和 `/health` ，它们是Spring Boot Actuator应用程序中有用endpoints的默认位置.如果使用非默认上下文路径或servlet路径（例如 `server.servletPath=/custom` ），则需要更改这些，即使对于Actuator应用程序也是如此.以下示例显示了两个设置的默认值：

**application.yml.** 

```java
eureka:
instance:
statusPageUrlPath: ${server.servletPath}/info
healthCheckUrlPath: ${server.servletPath}/health
```

这些链接显示在客户端使用的元数据中，并在某些情况下用于决定是否向您的应用程序发送请求，因此如果它们准确，则会很有帮助.

> 在Dalston中，还需要在更改管理上下文路径时设置状态和运行状况检查URL.从Edgware开始删除此要求.

## 11.5注册安全应用程序

如果您希望通过HTTPS联系您的应用，则可以在 `EurekaInstanceConfig` 中设置两个标志：

-  `eureka.instance.[nonSecurePortEnabled]=[false]` 

-  `eureka.instance.[securePortEnabled]=[true]` 

这样做会使Eureka发布实例信息，显示对安全通信的明确偏好.Spring天对于以这种方式配置的服务，Cloud  `DiscoveryClient` 始终返回以 `https` 开头的URI.同样，当以这种方式配置服务时，Eureka（本机）实例信息具有安全的运行状况检查URL.

由于Eureka在内部工作的方式，它仍然会发布状态和主页的非安全URL，除非您也明确地覆盖这些URL.您可以使用占位符来配置eureka实例URL，如以下示例所示：

**application.yml.** 

```java
eureka:
instance:
statusPageUrl: https://${eureka.hostname}/info
healthCheckUrl: https://${eureka.hostname}/health
homePageUrl: https://${eureka.hostname}/
```

（请注意， `${eureka.hostname}` 是仅在Eureka的更高版本中可用的本机占位符.您也可以使用Spring占位符实现相同的功能 - 例如，使用 `${eureka.instance.hostName}` .）

> 如果您的应用程序在代理后面运行，并且SSL终止在代理中（例如，如果您在Cloud Foundry或其他平台中作为服务运行），那么您需要确保代理“转发”标头被拦截和处理通过申请.如果嵌入在Spring Boot应用程序中的Tomcat容器具有“X-Forwarded  -  \ *”标头的显式配置，则会自动发生这种情况.您的应用程序呈现给自己错误的链接（错误的主机，端口或协议）表明您的配置错误.

## 11.6尤里卡的Health检查

默认情况下，Eureka使用客户端心跳来确定客户端是否已启动.除非另有说明，否则Discovery Client不会根据Spring Boot Actuator传播应用程序的当前运行状况检查状态.因此，在成功注册后，Eureka始终宣布应用程序处于“UP”状态.通过启用Eureka运行状况检查可以更改此行为，从而将应用程序状态传播到Eureka.因此，每个其他应用程序都不会向“UP”以外的状态下的应用程序发送流量.以下示例显示如何为客户端启用运行状况检查：

**application.yml.** 

```java
eureka:
client:
healthcheck:
enabled: true
```

>  `eureka.client.healthcheck.enabled=true` 只应在 `application.yml` 中设置.在 `bootstrap.yml` 中设置值会导致不良副作用，例如在Eureka中以 `UNKNOWN` 状态注册.

如果您需要更多控制运行状况检查，请考虑实现自己的 `com.netflix.appinfo.HealthCheckHandler` .

## 11.7实例和客户端的Eureka元数据

值得花些时间了解Eureka元数据的工作原理，因此您可以在平台中使用它.有标准元数据可用于提供主机名，IP地址，端口号，状态页和运行状况检查等信息.这些发布在服务注册表中，客户端使用它们以直接的方式联系服务.可以将其他元数据添加到 `eureka.instance.metadataMap` 中的实例注册中，并且可以在远程客户端中访问此元数据.通常，除非客户端了解元数据的含义，否则其他元数据不会更改客户端的行为.本文档后面将介绍几种特殊情况，其中Spring Cloud已经为元数据映射赋予了意义.

### 11.7.1在Cloud Foundry上使用Eureka

Cloud Foundry有一个全局路由器，因此同一个应用程序的所有实例都具有相同的主机名（具有类似架构的其他PaaS解决方案具有相同的安排）.这不一定是使用Eureka的障碍.但是，如果您使用路由器（建议甚至强制使用，具体取决于您的平台的设置方式），您需要明确设置主机名和端口号（安全或非安全），以便他们使用路由器.您可能还希望使用实例元数据，以便区分客户端上的实例（例如，在自定义负载平衡器中）.默认情况下， `eureka.instance.instanceId` 是 `vcap.application.instance_id` ，如以下示例所示：

**application.yml.** 

```java
eureka:
instance:
hostname: ${vcap.application.uris[0]}
nonSecurePort: 80
```

根据在Cloud Foundry实例中设置安全规则的方式，您可以注册并使用主机VM的IP地址进行直接服务到服务调用. Pivotal Web Services（[PWS](https://run.pivotal.io)）上尚未提供此功能.

### 11.7.2在AWS上使用Eureka

如果计划将应用程序部署到AWSCloud，则必须将Eureka实例配置为支持AWS.您可以通过自定义[EurekaInstanceConfigBean](https://github.com/spring-cloud/spring-cloud-netflix/tree/master/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaInstanceConfigBean.java)来执行此操作，如下所示：

```java
@Bean
@Profile("!default")
public EurekaInstanceConfigBean eurekaInstanceConfig(InetUtils inetUtils) {
EurekaInstanceConfigBean b = new EurekaInstanceConfigBean(inetUtils);
AmazonInfo info = AmazonInfo.Builder.newBuilder().autoBuild("eureka");
b.setDataCenterInfo(info);
return b;
}
```

### 11.7.3更改Eureka实例ID

一个vanilla Netflix Eureka实例注册的ID等于其主机名（即每个主机只有一个服务）. Spring Cloud Eureka提供合理的默认值，定义如下：

`${spring.cloud.client.hostname}:${spring.application.name}:${spring.application.instance_id:${server.port}}}` 

一个例子是 `myhost:myappname:8080` .

通过使用Spring Cloud，您可以通过在 `eureka.instance.instanceId` 中提供唯一标识符来覆盖此值，如以下示例所示：

**application.yml.** 

```java
eureka:
instance:
instanceId: ${spring.application.name}:${vcap.application.instance_id:${spring.application.instance_id:${random.value}}}
```

使用前面示例中显示的元数据和部署在localhost上的多个服务实例，将随机值插入其中以使实例唯一.在Cloud Foundry中， `vcap.application.instance_id` 会在Spring Boot应用程序中自动填充，因此不需要随机值.

## 11.8使用EurekaClient

一旦您拥有一个发现客户端的应用程序，您就可以使用它来发现[Eureka Server](multi_spring-cloud-eureka-server.html)中的服务实例.一种方法是使用本机 `com.netflix.discovery.EurekaClient` （而不是Spring Cloud  `DiscoveryClient` ），如以下示例所示：

```java
@Autowired
private EurekaClient discoveryClient;

public String serviceUrl() {
InstanceInfo instance = discoveryClient.getNextServerFromEureka("STORES", false);
return instance.getHomePageUrl();
}
```

> 不要在 `@PostConstruct` 方法或 `@Scheduled` 方法中使用 `EurekaClient` （或者 `ApplicationContext` 可能尚未启动的任何地方）.它在 `SmartLifecycle` （带 `phase=0` ）中初始化，因此最早可以依赖它的是另一个具有更高相位的 `SmartLifecycle` .

### 11.8.1没有Jersey的EurekaClient

默认情况下，EurekaClient使用Jersey进行HTTP通信.如果您希望避免来自Jersey的依赖项，则可以将其从依赖项中排除. Spring Cloud基于Spring  `RestTemplate` 自动配置传输客户端.以下示例显示Jersey被排除在外：

```xml
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
<exclusions>
<exclusion>
<groupId>com.sun.jersey</groupId>
<artifactId>jersey-client</artifactId>
</exclusion>
<exclusion>
<groupId>com.sun.jersey</groupId>
<artifactId>jersey-core</artifactId>
</exclusion>
<exclusion>
<groupId>com.sun.jersey.contribs</groupId>
<artifactId>jersey-apache-client4</artifactId>
</exclusion>
</exclusions>
</dependency>
```

## 11.9 Native Netflix EurekaClient的替代品

您无需使用原始Netflix  `EurekaClient` .此外，在某种包装后面使用它通常更方便. Spring Cloud通过逻辑Eureka服务标识符（VIP）而不是物理URL支持[Feign](multi_spring-cloud-feign.html)（REST客户端构建器）和[Spring RestTemplate](multi_spring-cloud-ribbon.html).要使用固定的物理服务器列表配置功能区，可以将 `<client>.ribbon.listOfServers` 设置为以逗号分隔的物理地址（或主机名）列表，其中 `<client>` 是客户端的ID.

您还可以使用 `org.springframework.cloud.client.discovery.DiscoveryClient` ，它为发现客户端提供简单的API（不特定于Netflix），如以下示例所示：

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

## 11.10为什么注册服务这么慢？

作为实例还涉及到注册表的定期心跳（通过客户端的 `serviceUrl` ），默认持续时间为30秒.在实例，服务器和客户端在其本地缓存中都具有相同的元数据之前，客户端无法发现服务（因此可能需要3次心跳）.您可以通过设置 `eureka.instance.leaseRenewalIntervalInSeconds` 来更改周期.将其设置为小于30的值可加快使客户端连接到其他服务的过程.在生产环境中，最好坚持使用默认值，因为服务器中的内部计算会对租赁续订期做出假设.

## 11.11区域

如果已将Eureka客户端部署到多个区域，则可能希望这些客户端在尝试其他区域中的服务之前使用同一区域内的服务.要进行此设置，您需要正确配置Eureka客户端.

首先，您需要确保将Eureka服务器部署到每个区域，并确保它们彼此对等.有关详细信息，请参阅[zones and regions](multi_spring-cloud-eureka-server.html#spring-cloud-eureka-server-zones-and-regions)部分.

接下来，您需要告诉Eureka您的服务所在的区域.您可以使用 `metadataMap` 属性来执行此操作.例如，如果将 `service 1` 部署到 `zone 1` 和 `zone 2` ，则需要在 `service 1` 中设置以下Eureka属性：

**Service 1 in Zone 1** 

```java
eureka.instance.metadataMap.zone = zone1
eureka.client.preferSameZoneEureka = true
```

**Service 1 in Zone 2** 

```java
eureka.instance.metadataMap.zone = zone2
eureka.client.preferSameZoneEureka = true
```

