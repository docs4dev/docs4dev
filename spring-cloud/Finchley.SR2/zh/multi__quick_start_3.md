## 40.快速入门

Spring Cloud Bus的工作原理是，如果它在类路径上检测到自身，则添加Spring Boot自动配置.要启用总线，请将 `spring-cloud-starter-bus-amqp` 或 `spring-cloud-starter-bus-kafka` 添加到依赖关系管理中. Spring Cloud负责其余部分.确保代理（RabbitMQ或Kafka）可用且已配置.在localhost上运行时，您无需执行任何操作.如果您远程运行，请使用Spring Cloud Connectors或Spring Boot约定来定义代理凭据，如以下Rabbit示例所示：

**application.yml.** 

```java
spring:
rabbitmq:
host: mybroker.com
port: 5672
username: user
password: secret
```

总线当前支持向所有侦听节点或特定服务的所有节点（由Eureka定义）发送消息.  `/bus/*` Actuator命名空间有一些HTTPendpoints.目前，有两个已实施.第一个 `/bus/env` 发送键/值对来更新每个节点的Spring环境.第二个， `/bus/refresh` ，重新加载每个应用程序的配置，就好像它们都已经在它们的 `/refresh` endpoints上被ping了一样.

>  Spring Cloud Bus的初学者包括Rabbit和Kafka，因为这是两种最常见的实现.但是，Spring Cloud Stream非常灵活，并且Binders与 `spring-cloud-bus` 一起使用.

