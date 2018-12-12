## 33. Messaging

The Spring Framework provides extensive support for integrating with messaging systems, from simplified use of the JMS API using  `JmsTemplate`  to a complete infrastructure to receive messages asynchronously. Spring AMQP provides a similar feature set for the Advanced Message Queuing Protocol. Spring Boot also provides auto-configuration options for  `RabbitTemplate`  and RabbitMQ. Spring WebSocket natively includes support for STOMP messaging, and Spring Boot has support for that through starters and a small amount of auto-configuration. Spring Boot also has support for Apache Kafka.

## 33.1 JMS

The  `javax.jms.ConnectionFactory`  interface provides a standard method of creating a  `javax.jms.Connection`  for interacting with a JMS broker. Although Spring needs a  `ConnectionFactory`  to work with JMS, you generally need not use it directly yourself and can instead rely on higher level messaging abstractions. (See the [relevant section](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/integration.html#jms) of the Spring Framework reference documentation for details.) Spring Boot also auto-configures the necessary infrastructure to send and receive messages.

### 33.1.1 ActiveMQ Support

When [ActiveMQ](http://activemq.apache.org/) is available on the classpath, Spring Boot can also configure a  `ConnectionFactory` . If the broker is present, an embedded broker is automatically started and configured (provided no broker URL is specified through configuration).

> If you use  `spring-boot-starter-activemq` , the necessary dependencies to connect or embed an ActiveMQ instance are provided, as is the Spring infrastructure to integrate with JMS.

ActiveMQ configuration is controlled by external configuration properties in  `spring.activemq.*` . For example, you might declare the following section in  `application.properties` :

```java
spring.activemq.broker-url=tcp://192.168.1.210:9876
spring.activemq.user=admin
spring.activemq.password=secret
```

By default, a  `CachingConnectionFactory`  wraps the native  `ConnectionFactory`  with sensible settings that you can control by external configuration properties in  `spring.jms.*` :

```java
spring.jms.cache.session-cache-size=5
```

If you’d rather use native pooling, you can do so by adding a dependency to  `org.messaginghub:pooled-jms`  and configuring the  `JmsPoolConnectionFactory`  accordingly, as shown in the following example:

```java
spring.activemq.pool.enabled=true
spring.activemq.pool.max-connections=50
```

> See [ActiveMQProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jms/activemq/ActiveMQProperties.java) for more of the supported options. You can also register an arbitrary number of beans that implement  `ActiveMQConnectionFactoryCustomizer`  for more advanced customizations.

By default, ActiveMQ creates a destination if it does not yet exist so that destinations are resolved against their provided names.

### 33.1.2 Artemis Support

Spring Boot can auto-configure a  `ConnectionFactory`  when it detects that [Artemis](http://activemq.apache.org/artemis/) is available on the classpath. If the broker is present, an embedded broker is automatically started and configured (unless the mode property has been explicitly set). The supported modes are  `embedded`  (to make explicit that an embedded broker is required and that an error should occur if the broker is not available on the classpath) and  `native`  (to connect to a broker using the  `netty`  transport protocol). When the latter is configured, Spring Boot configures a  `ConnectionFactory`  that connects to a broker running on the local machine with the default settings.

> If you use  `spring-boot-starter-artemis` , the necessary dependencies to connect to an existing Artemis instance are provided, as well as the Spring infrastructure to integrate with JMS. Adding  `org.apache.activemq:artemis-jms-server`  to your application lets you use embedded mode.

Artemis configuration is controlled by external configuration properties in  `spring.artemis.*` . For example, you might declare the following section in  `application.properties` :

```java
spring.artemis.mode=native
spring.artemis.host=192.168.1.210
spring.artemis.port=9876
spring.artemis.user=admin
spring.artemis.password=secret
```

When embedding the broker, you can choose if you want to enable persistence and list the destinations that should be made available. These can be specified as a comma-separated list to create them with the default options, or you can define bean(s) of type  `org.apache.activemq.artemis.jms.server.config.JMSQueueConfiguration`  or  `org.apache.activemq.artemis.jms.server.config.TopicConfiguration` , for advanced queue and topic configurations, respectively.

By default, a  `CachingConnectionFactory`  wraps the native  `ConnectionFactory`  with sensible settings that you can control by external configuration properties in  `spring.jms.*` :

```java
spring.jms.cache.session-cache-size=5
```

If you’d rather use native pooling, you can do so by adding a dependency to  `org.messaginghub:pooled-jms`  and configuring the  `JmsPoolConnectionFactory`  accordingly, as shown in the following example:

```java
spring.artemis.pool.enabled=true
spring.artemis.pool.max-connections=50
```

See [ArtemisProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jms/artemis/ArtemisProperties.java) for more supported options.

No JNDI lookup is involved, and destinations are resolved against their names, using either the  `name`  attribute in the Artemis configuration or the names provided through configuration.

### 33.1.3 Using a JNDI ConnectionFactory

If you are running your application in an application server, Spring Boot tries to locate a JMS  `ConnectionFactory`  by using JNDI. By default, the  `java:/JmsXA`  and  `java:/XAConnectionFactory`  location are checked. You can use the  `spring.jms.jndi-name`  property if you need to specify an alternative location, as shown in the following example:

```java
spring.jms.jndi-name=java:/MyConnectionFactory
```

### 33.1.4 Sending a Message

Spring’s  `JmsTemplate`  is auto-configured, and you can autowire it directly into your own beans, as shown in the following example:

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

	private final JmsTemplate jmsTemplate;

	@Autowired
	public MyBean(JmsTemplate jmsTemplate) {
		this.jmsTemplate = jmsTemplate;
	}

	// ...

}
```

> [JmsMessagingTemplate](https://docs.spring.io/spring/docs/5.1.2.RELEASE/javadoc-api/org/springframework/jms/core/JmsMessagingTemplate.html) can be injected in a similar manner. If a  `DestinationResolver`  or a  `MessageConverter`  bean is defined, it is associated automatically to the auto-configured  `JmsTemplate` .

### 33.1.5 Receiving a Message

When the JMS infrastructure is present, any bean can be annotated with  `@JmsListener`  to create a listener endpoint. If no  `JmsListenerContainerFactory`  has been defined, a default one is configured automatically. If a  `DestinationResolver`  or a  `MessageConverter`  beans is defined, it is associated automatically to the default factory.

By default, the default factory is transactional. If you run in an infrastructure where a  `JtaTransactionManager`  is present, it is associated to the listener container by default. If not, the  `sessionTransacted`  flag is enabled. In that latter scenario, you can associate your local data store transaction to the processing of an incoming message by adding  `@Transactional`  on your listener method (or a delegate thereof). This ensures that the incoming message is acknowledged, once the local transaction has completed. This also includes sending response messages that have been performed on the same JMS session.

The following component creates a listener endpoint on the  `someQueue`  destination:

```java
@Component
public class MyBean {

	@JmsListener(destination = "someQueue")
	public void processMessage(String content) {
		// ...
	}

}
```

> See [the Javadoc of @EnableJms](https://docs.spring.io/spring/docs/5.1.2.RELEASE/javadoc-api/org/springframework/jms/annotation/EnableJms.html) for more details.

If you need to create more  `JmsListenerContainerFactory`  instances or if you want to override the default, Spring Boot provides a  `DefaultJmsListenerContainerFactoryConfigurer`  that you can use to initialize a  `DefaultJmsListenerContainerFactory`  with the same settings as the one that is auto-configured.

For instance, the following example exposes another factory that uses a specific  `MessageConverter` :

```java
@Configuration
static class JmsConfiguration {

	@Bean
	public DefaultJmsListenerContainerFactory myFactory(
			DefaultJmsListenerContainerFactoryConfigurer configurer) {
		DefaultJmsListenerContainerFactory factory =
				new DefaultJmsListenerContainerFactory();
		configurer.configure(factory, connectionFactory());
		factory.setMessageConverter(myMessageConverter());
		return factory;
	}

}
```

Then you can use the factory in any  `@JmsListener` -annotated method as follows:

```java
@Component
public class MyBean {

	@JmsListener(destination = "someQueue", containerFactory="myFactory")
	public void processMessage(String content) {
		// ...
	}

}
```

## 33.2 AMQP

The Advanced Message Queuing Protocol (AMQP) is a platform-neutral, wire-level protocol for message-oriented middleware. The Spring AMQP project applies core Spring concepts to the development of AMQP-based messaging solutions. Spring Boot offers several conveniences for working with AMQP through RabbitMQ, including the  `spring-boot-starter-amqp`  “Starter”.

### 33.2.1 RabbitMQ support

[RabbitMQ](https://www.rabbitmq.com/) is a lightweight, reliable, scalable, and portable message broker based on the AMQP protocol. Spring uses  `RabbitMQ`  to communicate through the AMQP protocol.

RabbitMQ configuration is controlled by external configuration properties in  `spring.rabbitmq.*` . For example, you might declare the following section in  `application.properties` :

```java
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=admin
spring.rabbitmq.password=secret
```

If a  `ConnectionNameStrategy`  bean exists in the context, it will be automatically used to name connections created by the auto-configured  `ConnectionFactory` . See [RabbitProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/amqp/RabbitProperties.java) for more of the supported options.

> See [Understanding AMQP, the protocol used by RabbitMQ](https://spring.io/blog/2010/06/14/understanding-amqp-the-protocol-used-by-rabbitmq/) for more details.

### 33.2.2 Sending a Message

Spring’s  `AmqpTemplate`  and  `AmqpAdmin`  are auto-configured, and you can autowire them directly into your own beans, as shown in the following example:

```java
import org.springframework.amqp.core.AmqpAdmin;
import org.springframework.amqp.core.AmqpTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

	private final AmqpAdmin amqpAdmin;
	private final AmqpTemplate amqpTemplate;

	@Autowired
	public MyBean(AmqpAdmin amqpAdmin, AmqpTemplate amqpTemplate) {
		this.amqpAdmin = amqpAdmin;
		this.amqpTemplate = amqpTemplate;
	}

	// ...

}
```

> [RabbitMessagingTemplate](https://docs.spring.io/spring-amqp/docs/current/api/org/springframework/amqp/rabbit/core/RabbitMessagingTemplate.html) can be injected in a similar manner. If a  `MessageConverter`  bean is defined, it is associated automatically to the auto-configured  `AmqpTemplate` .

If necessary, any  `org.springframework.amqp.core.Queue`  that is defined as a bean is automatically used to declare a corresponding queue on the RabbitMQ instance.

To retry operations, you can enable retries on the  `AmqpTemplate`  (for example, in the event that the broker connection is lost):

```java
spring.rabbitmq.template.retry.enabled=true
spring.rabbitmq.template.retry.initial-interval=2s
```

Retries are disabled by default. You can also customize the  `RetryTemplate`  programmatically by declaring a  `RabbitRetryTemplateCustomizer`  bean.

### 33.2.3 Receiving a Message

When the Rabbit infrastructure is present, any bean can be annotated with  `@RabbitListener`  to create a listener endpoint. If no  `RabbitListenerContainerFactory`  has been defined, a default  `SimpleRabbitListenerContainerFactory`  is automatically configured and you can switch to a direct container using the  `spring.rabbitmq.listener.type`  property. If a  `MessageConverter`  or a  `MessageRecoverer`  bean is defined, it is automatically associated with the default factory.

The following sample component creates a listener endpoint on the  `someQueue`  queue:

```java
@Component
public class MyBean {

	@RabbitListener(queues = "someQueue")
	public void processMessage(String content) {
		// ...
	}

}
```

> See [the Javadoc of @EnableRabbit](https://docs.spring.io/spring-amqp/docs/current/api/org/springframework/amqp/rabbit/annotation/EnableRabbit.html) for more details.

If you need to create more  `RabbitListenerContainerFactory`  instances or if you want to override the default, Spring Boot provides a  `SimpleRabbitListenerContainerFactoryConfigurer`  and a  `DirectRabbitListenerContainerFactoryConfigurer`  that you can use to initialize a  `SimpleRabbitListenerContainerFactory`  and a  `DirectRabbitListenerContainerFactory`  with the same settings as the factories used by the auto-configuration.

> It does not matter which container type you chose. Those two beans are exposed by the auto-configuration.

For instance, the following configuration class exposes another factory that uses a specific  `MessageConverter` :

```java
@Configuration
static class RabbitConfiguration {

	@Bean
	public SimpleRabbitListenerContainerFactory myFactory(
			SimpleRabbitListenerContainerFactoryConfigurer configurer) {
		SimpleRabbitListenerContainerFactory factory =
				new SimpleRabbitListenerContainerFactory();
		configurer.configure(factory, connectionFactory);
		factory.setMessageConverter(myMessageConverter());
		return factory;
	}

}
```

Then you can use the factory in any  `@RabbitListener` -annotated method, as follows:

```java
@Component
public class MyBean {

	@RabbitListener(queues = "someQueue", containerFactory="myFactory")
	public void processMessage(String content) {
		// ...
	}

}
```

You can enable retries to handle situations where your listener throws an exception. By default,  `RejectAndDontRequeueRecoverer`  is used, but you can define a  `MessageRecoverer`  of your own. When retries are exhausted, the message is rejected and either dropped or routed to a dead-letter exchange if the broker is configured to do so. By default, retries are disabled. You can also customize the  `RetryTemplate`  programmatically by declaring a  `RabbitRetryTemplateCustomizer`  bean.

|images/important.png|Important|
|----|----|
|By default, if retries are disabled and the listener throws an exception, the delivery is retried indefinitely. You can modify this behavior in two ways: Set the  `defaultRequeueRejected`  property to  `false`  so that zero re-deliveries are attempted or throw an  `AmqpRejectAndDontRequeueException`  to signal the message should be rejected. The latter is the mechanism used when retries are enabled and the maximum number of delivery attempts is reached. |

## 33.3 Apache Kafka Support

[Apache Kafka](https://kafka.apache.org/) is supported by providing auto-configuration of the  `spring-kafka`  project.

Kafka configuration is controlled by external configuration properties in  `spring.kafka.*` . For example, you might declare the following section in  `application.properties` :

```java
spring.kafka.bootstrap-servers=localhost:9092
spring.kafka.consumer.group-id=myGroup
```

> To create a topic on startup, add a bean of type  `NewTopic` . If the topic already exists, the bean is ignored.

See [KafkaProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/kafka/KafkaProperties.java) for more supported options.

### 33.3.1 Sending a Message

Spring’s  `KafkaTemplate`  is auto-configured, and you can autowire it directly in your own beans, as shown in the following example:

```java
@Component
public class MyBean {

	private final KafkaTemplate kafkaTemplate;

	@Autowired
	public MyBean(KafkaTemplate kafkaTemplate) {
		this.kafkaTemplate = kafkaTemplate;
	}

	// ...

}
```

> If the property  `spring.kafka.producer.transaction-id-prefix`  is defined, a  `KafkaTransactionManager`  is automatically configured. Also, if a  `RecordMessageConverter`  bean is defined, it is automatically associated to the auto-configured  `KafkaTemplate` .

### 33.3.2 Receiving a Message

When the Apache Kafka infrastructure is present, any bean can be annotated with  `@KafkaListener`  to create a listener endpoint. If no  `KafkaListenerContainerFactory`  has been defined, a default one is automatically configured with keys defined in  `spring.kafka.listener.*` .

The following component creates a listener endpoint on the  `someTopic`  topic:

```java
@Component
public class MyBean {

	@KafkaListener(topics = "someTopic")
	public void processMessage(String content) {
		// ...
	}

}
```

If a  `KafkaTransactionManager`  bean is defined, it is automatically associated to the container factory. Similarly, if a  `RecordMessageConverter` ,  `ErrorHandler`  or  `AfterRollbackProcessor`  bean is defined, it is automatically associated to the default factory.

> A custom  `ChainedKafkaTransactionManager`  must be marked  `@Primary`  as it usually references the auto-configured  `KafkaTransactionManager`  bean.

### 33.3.3 Kafka Streams

Spring for Apache Kafka provides a factory bean to create a  `StreamsBuilder`  object and manage the lifecycle of its streams. Spring Boot auto-configures the required  `KafkaStreamsConfiguration`  bean as long as  `kafka-streams`  is on the classpath and Kafka Streams is enabled via the  `@EnableKafkaStreams`  annotation.

Enabling Kafka Streams means that the application id and bootstrap servers must be set. The former can be configured using  `spring.kafka.streams.application-id` , defaulting to  `spring.application.name`  if not set. The latter can be set globally or specifically overridden just for streams.

Several additional properties are available using dedicated properties; other arbitrary Kafka properties can be set using the  `spring.kafka.streams.properties`  namespace. See also [Section 33.3.4, “Additional Kafka Properties”](boot-features-messaging.html#boot-features-kafka-extra-props) for more information.

To use the factory bean, simply wire  `StreamsBuilder`  into your  `@Bean`  as shown in the following example:

```java
@Configuration
@EnableKafkaStreams
static class KafkaStreamsExampleConfiguration {

	@Bean
	public KStream<Integer, String> kStream(StreamsBuilder streamsBuilder) {
		KStream<Integer, String> stream = streamsBuilder.stream("ks1In");
		stream.map((k, v) -> new KeyValue<>(k, v.toUpperCase())).to("ks1Out",
				Produced.with(Serdes.Integer(), new JsonSerde<>()));
		return stream;
	}

}
```

By default, the streams managed by the  `StreamBuilder`  object it creates are started automatically. You can customize this behaviour using the  `spring.kafka.streams.auto-startup`  property.

### 33.3.4 Additional Kafka Properties

The properties supported by auto configuration are shown in [Appendix A, Common application properties](common-application-properties.html). Note that, for the most part, these properties (hyphenated or camelCase) map directly to the Apache Kafka dotted properties. Refer to the Apache Kafka documentation for details.

The first few of these properties apply to all components (producers, consumers, admins, and streams) but can be specified at the component level if you wish to use different values. Apache Kafka designates properties with an importance of HIGH, MEDIUM, or LOW. Spring Boot auto-configuration supports all HIGH importance properties, some selected MEDIUM and LOW properties, and any properties that do not have a default value.

Only a subset of the properties supported by Kafka are available directly through the  `KafkaProperties`  class. If you wish to configure the producer or consumer with additional properties that are not directly supported, use the following properties:

```java
spring.kafka.properties.prop.one=first
spring.kafka.admin.properties.prop.two=second
spring.kafka.consumer.properties.prop.three=third
spring.kafka.producer.properties.prop.four=fourth
spring.kafka.streams.properties.prop.five=fifth
```

This sets the common  `prop.one`  Kafka property to  `first`  (applies to producers, consumers and admins), the  `prop.two`  admin property to  `second` , the  `prop.three`  consumer property to  `third` , the  `prop.four`  producer property to  `fourth`  and the  `prop.five`  streams property to  `fifth` .

You can also configure the Spring Kafka  `JsonDeserializer`  as follows:

```java
spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer
spring.kafka.consumer.properties.spring.json.value.default.type=com.example.Invoice
spring.kafka.consumer.properties.spring.json.trusted.packages=com.example,org.acme
```

Similarly, you can disable the  `JsonSerializer`  default behavior of sending type information in headers:

```java
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer
spring.kafka.producer.properties.spring.json.add.type.headers=false
```

|images/important.png|Important|
|----|----|
|Properties set in this way override any configuration item that Spring Boot explicitly supports. |

