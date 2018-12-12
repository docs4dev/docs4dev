## 15. Hystrix超时和功能区客户端

使用包装功能区客户端的Hystrix命令时，要确保将Hystrix超时配置为长于配置的功能区超时，包括可能进行的任何可能的重试.例如，如果您的功能区连接超时为一秒，并且功能区客户端可能会重试该请求三次，那么您的Hystrix超时应该稍微超过三秒.

## 15.1如何包含Hystrix仪表板

要在项目中包含Hystrix仪表板，请使用组ID为 `org.springframework.cloud` 且工件ID为 `spring-cloud-starter-netflix-hystrix-dashboard` 的starter.有关使用当前Spring Cloud Release Train设置构建系统的详细信息，请参阅[Spring Cloud Project page](https://projects.spring.io/spring-cloud/).

要运行Hystrix仪表板，请使用 `@EnableHystrixDashboard` 注释Spring Boot主类.然后访问 `/hystrix` 并将仪表板指向Hystrix客户端应用程序中的单个实例的 `/hystrix.stream` endpoints.

> 当连接到使用HTTPS的 `/hystrix.stream` endpoints时，JVM必须信任服务器使用的证书.如果证书不受信任，则必须将证书导入JVM，以便Hystrix仪表板成功连接到流endpoints.

## 15.2涡轮机

查看单个实例的Hystrix数据在系统的整体运行状况方面不是很有用. [Turbine](https://github.com/Netflix/Turbine)是一个应用程序，它将所有相关的 `/hystrix.stream` endpoints聚合为组合 `/turbine.stream` ，以便在Hystrix仪表板中使用.个别实例位于尤里卡.运行Turbine需要使用 `@EnableTurbine` 注释来注释主类（例如，通过使用spring-cloud-starter-netflix-turbine来设置类路径）. [the Turbine 1 wiki](https://github.com/Netflix/Turbine/wiki/Configuration-(1.x))中的所有记录的配置属性均适用.唯一的区别是 `turbine.instanceUrlSuffix` 不需要前置端口，因为除非 `turbine.instanceInsertPort=false` ，否则会自动处理.

> By默认情况下，Turbine通过在Eureka中查找其 `hostName` 和 `port` 条目然后将 `/hystrix.stream` 附加到其上来查找已注册实例上的 `/hystrix.stream` endpoints.如果实例的元数据包含 `management.port` ，则使用它来代替 `/hystrix.stream` endpoints的 `port` 值.默认情况下，名为 `management.port` 的元数据条目等于 `management.port` 配置属性.它可以通过以下配置覆盖：

```java
eureka:
instance:
metadata-map:
management.port: ${management.port:8081}
```

`turbine.appConfig` 配置密钥是Turb用于查找实例的Eureka serviceId列表.然后，在Hystrix仪表板中使用涡轮流，其URL类似于以下内容：

`http://my.turbine.server:8080/turbine.stream?cluster=CLUSTERNAME` 

如果名称为 `default` ，则可以省略cluster参数.  `cluster` 参数必须与 `turbine.aggregator.clusterConfig` 中的条目匹配.从尤里卡返回的值是大写的.因此，如果在Eureka中注册了名为 `customers` 的应用程序，则以下示例有效：

```java
turbine:
aggregator:
clusterConfig: CUSTOMERS
appConfig: customers
```

如果需要自定义Turbine应使用的集群名称（因为您不希望在 `turbine.aggregator.clusterConfig` 配置中存储集群名称），请提供类型为 `TurbineClustersProvider` 的bean.

`clusterName` 可以通过 `turbine.clusterNameExpression` 中的SPEL表达式进行自定义，其中root用作 `InstanceInfo` 的实例.默认值为 `appName` ，这意味着Eureka  `serviceId` 成为群集密钥（即， `InstanceInfo` 对于客户的 `appName` 为 `CUSTOMERS` ）.另一个示例是 `turbine.clusterNameExpression=aSGName` ，它从AWS ASG名称中获取群集名称.以下清单显示了另一个示例：

```java
turbine:
aggregator:
clusterConfig: SYSTEM,USER
appConfig: customers,stores,ui,admin
clusterNameExpression: metadata['cluster']
```

在前面的示例中，来自四个服务的集群名称是从其元数据映射中提取的，并且预计具有包含 `SYSTEM` 和 `USER` 的值.

要为所有应用程序使用“默认”群集，您需要一个字符串文字表达式（使用单引号并使用双引号进行转义，如果它在YAML也是）：

```java
turbine:
appConfig: customers,stores
clusterNameExpression: "'default'"
```

Spring Cloud提供 `spring-cloud-starter-netflix-turbine` ，它具有运行Turbine服务器所需的所有依赖项.要广告Turnbine，请创建一个Spring Boot应用程序并使用 `@EnableTurbine` 进行注释.

> By默认情况下，Spring Cloud允许Turbine使用主机和端口为每个群集允许每个主机进行多个进程.如果您希望Turbine内置的本机Netflix行为不允许每个主机有多个进程，每个群集（实例ID的键是主机名），请设置 `turbine.combineHostPort=false` .

### 15.2.1集群endpoints

在某些情况下，其他应用程序可能会知道在Turbine中配置了哪些custers.为了支持这一点，您可以使用 `/clusters` endpoints，它将返回所有已配置集群的JSON数组.

**GET /clusters.** 

```java
[
{
"name": "RACES",
"link": "http://localhost:8383/turbine.stream?cluster=RACES"
},
{
"name": "WEB",
"link": "http://localhost:8383/turbine.stream?cluster=WEB"
}
]
```

可以通过将 `turbine.endpoints.clusters.enabled` 设置为 `false` 来禁用此endpoints.

## 15.3涡轮流

在某些环境中（例如在PaaS设置中），从所有分布式Hystrix命令中提取指标的经典Turbine模型不起作用.在这种情况下，您可能希望让Hystrix命令将指标推送到Turbine. Spring Cloud通过消息传递实现.要在客户端上执行此操作，请将依赖项添加到 `spring-cloud-netflix-hystrix-stream` 和您选择的 `spring-cloud-starter-stream-*` .有关代理以及如何配置客户端凭据的详细信息，请参阅[Spring Cloud Stream documentation](https://docs.spring.io/spring-cloud-stream/docs/current/reference/htmlsingle/).它应该为本地经纪人开箱即用.

在服务器端，创建一个Spring Boot应用程序并使用 `@EnableTurbineStream` 进行注释. Turbine Stream服务器需要使用Spring Webflux，因此 `spring-boot-starter-webflux` 需要包含在您的项目中.默认情况下，在将 `spring-cloud-starter-netflix-turbine-stream` 添加到应用程序时包含 `spring-boot-starter-webflux` .

然后，您可以将Hystrix仪表板指向Turbine Stream Server而不是单独的Hystrix流.如果Turbine Stream在myhost上的端口8989上运行，则将 `http://myhost:8989` 放在Hystrix仪表板的流输入字段中.电路的前缀为 `serviceId` ，后跟一个点（ `.` ），然后是电路名称.

Spring Cloud提供 `spring-cloud-starter-netflix-turbine-stream` ，它具有运行Turbine Stream服务器所需的所有依赖项.然后，您可以添加所选的StreamBinders - 例如 `spring-cloud-starter-stream-rabbit` .

Turbine Stream服务器还支持 `cluster` 参数.与Turbine服务器不同，Turbine Stream使用eureka serviceId作为集群名称，这些不可配置.

如果Turbine Stream服务器在 `my.turbine.server` 上的端口8989上运行，并且您的环境中有两个eureka serviceIds  `customers` 和 `products` ，则您的Turbine Stream服务器上将提供以下URL.  `default` 和空集群名称将提供Turbine Stream服务器接收的所有指标.

```java
http://my.turbine.sever:8989/turbine.stream?cluster=customers
http://my.turbine.sever:8989/turbine.stream?cluster=products
http://my.turbine.sever:8989/turbine.stream?cluster=default
http://my.turbine.sever:8989/turbine.stream
```

因此，您可以将eureka serviceId用作Turbine仪表板（或任何兼容的仪表板）的群集名称.您无需为Turbine Stream服务器配置任何属性，如 `turbine.appConfig` ， `turbine.clusterNameExpression` 和 `turbine.aggregator.clusterConfig` .

> Turbine Stream服务器使用Spring Cloud Stream从配置的输入通道收集所有指标.这意味着它不会从每个实例主动收集Hystrix指标.它只能提供每个实例已经收集到输入通道中的指标.

