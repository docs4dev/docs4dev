## 40. Quick Start

Spring Cloud Bus works by adding Spring Boot autconfiguration if it detects itself on the classpath. To enable the bus, add  `spring-cloud-starter-bus-amqp`  or  `spring-cloud-starter-bus-kafka`  to your dependency management. Spring Cloud takes care of the rest. Make sure the broker (RabbitMQ or Kafka) is available and configured. When running on localhost, you need not do anything. If you run remotely, use Spring Cloud Connectors or Spring Boot conventions to define the broker credentials, as shown in the following example for Rabbit:

**application.yml.**  

```java
spring:
rabbitmq:
host: mybroker.com
port: 5672
username: user
password: secret
```

The bus currently supports sending messages to all nodes listening or all nodes for a particular service (as defined by Eureka). The  `/bus/*`  actuator namespace has some HTTP endpoints. Currently, two are implemented. The first,  `/bus/env` , sends key/value pairs to update each node’s Spring Environment. The second,  `/bus/refresh` , reloads each application’s configuration, as though they had all been pinged on their  `/refresh`  endpoint.

> The Spring Cloud Bus starters cover Rabbit and Kafka, because those are the two most common implementations. However, Spring Cloud Stream is quite flexible, and the binder works with  `spring-cloud-bus` .

