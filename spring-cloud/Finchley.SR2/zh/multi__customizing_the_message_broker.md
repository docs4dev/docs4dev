## 45.自定义Message Broker

Spring Cloud Bus使用[Spring Cloud Stream](https://cloud.spring.io/spring-cloud-stream)来广播消息.因此，要使消息流动，您只需要在类路径中包含您选择的Binders实现.有AMQP（RabbitMQ）和Kafka（ `spring-cloud-starter-bus-[amqp|kafka]` ）的公共汽车有方便的起动器.一般来说，Spring Cloud Stream依赖于Spring Boot自动配置约定来配置中间件.例如，可以使用 `spring.rabbitmq.*` 配置属性更改AMQP代理地址. Spring Cloud Bus在 `spring.cloud.bus.*` 中有一些本机配置属性（例如， `spring.cloud.bus.destination` 是用作外部中间件的主题的名称）.通常，默认值就足够了.

要了解有关如何自定义消息代理设置的更多信息，请参阅Spring Cloud Stream文档.
