## 19.与Sidecar的Polyglot支持

您是否拥有可以利用Eureka，Ribbon和Config Server的非JVM语言？ Spring Cloud Netflix Sidecar的灵感来自[Netflix Prana](https://github.com/Netflix/Prana).它包含一个HTTP API，用于获取给定服务的所有实例（按主机和端口）.您还可以通过嵌入式Zuul代理代理服务调用，该代理从Eureka获取其路由条目.可以通过主机查找或Zuul代理直接访问Spring Cloud Config Server.非JVM应用程序应实施运行状况检查，以便Sidecar可以向Eureka报告应用程序是启动还是关闭.

要在项目中包含Sidecar，请使用组ID为 `org.springframework.cloud` 且工件ID或 `spring-cloud-netflix-sidecar` 的依赖项.

要启用Sidecar，请使用 `@EnableSidecar` 创建Spring Boot应用程序.此注释包括 `@EnableCircuitBreaker` ， `@EnableDiscoveryClient` 和 `@EnableZuulProxy` .在与非JVM应用程序相同的主机上运行生成的应用程序.

要配置侧车，请将 `sidecar.port` 和 `sidecar.health-uri` 添加到 `application.yml` .  `sidecar.port` 属性是非JVM应用程序侦听的端口.这样Sidecar可以正确地向Eureka注册应用程序.  `sidecar.health-uri` 是可在非JVM应用程序上访问的URI，它模仿Spring Boot运行状况指示器.它应该返回类似于以下内容的JSON文档：

**health-uri-document.** 

```java
{
"status":"UP"
}
```

以下application.yml示例显示了Sidecar应用程序的示例配置：

**application.yml.** 

```java
server:
port: 5678
spring:
application:
name: sidecar

sidecar:
port: 8000
health-uri: http://localhost:8000/health.json
```

`DiscoveryClient.getInstances()` 方法的API是 `/hosts/{serviceId}` .以下 `/hosts/customers` 的示例响应在不同主机上返回两个实例：

**/hosts/customers.** 

```java
[
{
"host": "myhost",
"port": 9000,
"uri": "http://myhost:9000",
"serviceId": "CUSTOMERS",
"secure": false
},
{
"host": "myhost2",
"port": 9000,
"uri": "http://myhost2:9000",
"serviceId": "CUSTOMERS",
"secure": false
}
]
```

非_JVM应用程序（如果边车位于端口5678上）可在 `http://localhost:5678/hosts/{serviceId}` 访问此API.

Zuul代理会自动将Eureka中已知的每项服务的路由添加到 `/<serviceId>` ，因此客户服务可在 `/customers` 处获得.非JVM应用程序可以在 `http://localhost:5678/customers` 访问客户服务（假设边车正在侦听端口5678）.

如果配置服务器已在Eureka中注册，则非JVM应用程序可以通过Zuul代理访问它.如果ConfigServer的 `serviceId` 是 `configserver` 且Sidecar在端口5678上，则可以在[http://localhost:5678/configserver](http://localhost:5678/configserver)访问它.

非JVM应用程序可以利用Config Server返回YAML文档的功能.例如，对[http://sidecar.local.spring.io:5678/configserver/default-master.yml](http://sidecar.local.spring.io:5678/configserver/default-master.yml)的调用可能会导致YAML文档类似于以下内容：

```java
eureka:
client:
serviceUrl:
defaultZone: http://localhost:8761/eureka/
password: password
info:
description: Spring Cloud Samples
url: https://github.com/spring-cloud-samples
```

要在使用HTTP时将运行状况检查请求接受所有证书，请将 `sidecar.accept-all-ssl-certificates` 设置为“true”.
