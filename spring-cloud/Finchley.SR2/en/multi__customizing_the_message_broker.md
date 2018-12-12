## 45. Customizing the Message Broker

Spring Cloud Bus uses [Spring Cloud Stream](https://cloud.spring.io/spring-cloud-stream) to broadcast the messages. So, to get messages to flow, you need only include the binder implementation of your choice in the classpath. There are convenient starters for the bus with AMQP (RabbitMQ) and Kafka ( `spring-cloud-starter-bus-[amqp|kafka]` ). Generally speaking, Spring Cloud Stream relies on Spring Boot autoconfiguration conventions for configuring middleware. For instance, the AMQP broker address can be changed with  `spring.rabbitmq.*`  configuration properties. Spring Cloud Bus has a handful of native configuration properties in  `spring.cloud.bus.*`  (for example,  `spring.cloud.bus.destination`  is the name of the topic to use as the external middleware). Normally, the defaults suffice.

To learn more about how to customize the message broker settings, consult the Spring Cloud Stream documentation.
