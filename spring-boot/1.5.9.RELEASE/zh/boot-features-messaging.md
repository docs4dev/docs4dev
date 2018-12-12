## 33.消息

Spring Framework为与消息传递系统的集成提供了广泛的支持，从使用 `JmsTemplate` 的JMS API的简化使用到异步接收消息的完整基础结构. Spring AMQP为高级消息队列协议提供了类似的功能集. Spring Boot还为 `RabbitTemplate` 和RabbitMQ提供自动配置选项. Spring WebSocket本身包含对STOMP消息传递的支持，Spring Boot通过启动器和少量自动配置支持它. Spring Boot也支持Apache Kafka.

## 33.1 JMS

`javax.jms.ConnectionFactory` 接口提供了一种创建 `javax.jms.Connection` 以与JMS代理进行交互的标准方法.虽然Spring需要一个 `ConnectionFactory` 来与JMS一起工作，但是你通常不需要自己直接使用它，而是可以依赖更高级别的消息传递抽象. （有关详细信息，请参阅Spring Framework参考文档的[relevant section](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/integration.html#jms).）Spring Boot还会自动配置发送和接收消息所需的基础结构.

### 33.1.1 ActiveMQ支持

当[ActiveMQ](http://activemq.apache.org/)在类路径上可用时，Spring Boot也可以配置 `ConnectionFactory` .如果代理存在，则会自动启动并配置嵌入式代理（前提是未通过配置指定代理URL）.

> 如果使用 `spring-boot-starter-activemq` ，则提供连接或嵌入ActiveMQ实例的必要依赖项，以及与JMS集成的Spring基础结构.

ActiveMQ配置由 `spring.activemq.*` 中的外部配置属性控制.例如，您可以在 `application.properties` 中声明以下部分：

```java
spring.activemq.broker-url=tcp://192.168.1.210:9876
spring.activemq.user=admin
spring.activemq.password=secret
```

默认情况下， `CachingConnectionFactory` 使用 `spring.jms.*` 中的外部配置属性可以控制的合理设置包装本机 `ConnectionFactory` ：

```java
spring.jms.cache.session-cache-size=5
```

如果您更愿意使用本机池，则可以通过向 `org.messaginghub:pooled-jms` 添加依赖项并相应地配置 `JmsPoolConnectionFactory` 来实现，如以下示例所示：

```java
spring.activemq.pool.enabled=true
spring.activemq.pool.max-connections=50
```

> 查看[ActiveMQProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jms/activemq/ActiveMQProperties.java)了解更多受支持的选项.您还可以注册实现 `ActiveMQConnectionFactoryCustomizer` 的任意数量的bean以进行更高级的自定义.

默认情况下，ActiveMQ会创建一个目标（如果它尚不存在），以便根据提供的名称解析目标.

### 33.1.2阿耳特弥斯支持

当Spring Boot检测到[Artemis](http://activemq.apache.org/artemis/)在类路径上可用时，它可以自动配置 `ConnectionFactory` .如果存在代理，则会自动启动并配置嵌入式代理（除非已明确设置mode属性）.支持的模式是 `embedded` （以明确表示需要嵌入式代理，如果代理路径在类路径上不可用则应发生错误）和 `native` （使用 `netty` 传输协议连接到代理）.配置后者后，Spring Boot会使用默认设置配置连接到本地计算机上运行的代理的 `ConnectionFactory` .

> 如果使用 `spring-boot-starter-artemis` ，则会提供连接到现有Artemis实例的必要依赖项，以及与JMS集成的Spring基础结构.将 `org.apache.activemq:artemis-jms-server` 添加到您的应用程序可让您使用嵌入模式.

Artemis配置由 `spring.artemis.*` 中的外部配置属性控制.例如，您可以在 `application.properties` 中声明以下部分：

```java
spring.artemis.mode=native
spring.artemis.host=192.168.1.210
spring.artemis.port=9876
spring.artemis.user=admin
spring.artemis.password=secret
```

嵌入代理时，您可以选择是否要启用持久性并列出应该可用的目标.可以将这些指定为以逗号分隔的列表，以使用默认选项创建它们，也可以分别为高级队列和主题配置定义 `org.apache.activemq.artemis.jms.server.config.JMSQueueConfiguration` 或 `org.apache.activemq.artemis.jms.server.config.TopicConfiguration` 类型的bean.

默认情况下， `CachingConnectionFactory` 使用 `spring.jms.*` 中的外部配置属性可以控制的合理设置包装本机 `ConnectionFactory` ：

```java
spring.jms.cache.session-cache-size=5
```

如果您更愿意使用本机池，则可以通过向 `org.messaginghub:pooled-jms` 添加依赖项并相应地配置 `JmsPoolConnectionFactory` 来实现，如以下示例所示：

```java
spring.artemis.pool.enabled=true
spring.artemis.pool.max-connections=50
```

有关更多支持的选项，请参阅[ArtemisProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jms/artemis/ArtemisProperties.java).

不涉及JNDI查找，并使用Artemis配置中的 `name` 属性或通过配置提供的名称来解析目标名称.

### 33.1.3使用JNDI ConnectionFactory

如果您在应用程序服务器中运行应用程序，Spring Boot会尝试使用JNDI找到JMS  `ConnectionFactory` .默认情况下，会检查 `java:/JmsXA` 和 `java:/XAConnectionFactory` 位置.如果需要指定备用位置，可以使用 `spring.jms.jndi-name` 属性，如下所示以下示例：

```java
spring.jms.jndi-name=java:/MyConnectionFactory
```

### 33.1.4发送消息

Spring的 `JmsTemplate` 是自动配置的，你可以直接将它自动装入你自己的bean中，如下例所示：

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

> [JmsMessagingTemplate](https://docs.spring.io/spring/docs/5.1.2.RELEASE/javadoc-api/org/springframework/jms/core/JmsMessagingTemplate.html)可以以类似的方式注入.如果定义了 `DestinationResolver` 或 `MessageConverter`  bean，它将自动关联到自动配置的 `JmsTemplate` .

### 33.1.5收到留言

当存在JMS基础结构时，可以使用 `@JmsListener` 注释任何bean以创建侦听器endpoints.如果未定义 `JmsListenerContainerFactory` ，则会自动配置默认值.如果定义了 `DestinationResolver` 或 `MessageConverter`  beans，则它会自动关联到默认工厂.

默认情况下，默认工厂是事务性的.如果您在存在 `JtaTransactionManager` 的基础结构中运行，则默认情况下它与侦听器容器关联.如果不是，则启用 `sessionTransacted` 标志.在后一种情况下，您可以通过在侦听器方法（或其委托）上添加 `@Transactional` ，将本地数据存储事务与传入消息的处理相关联.这确保了在本地事务完成后确认传入消息.这还包括发送已在同一JMS会话上执行的响应消息.

以下组件在 `someQueue` 目标上创建侦听器endpoints：

```java
@Component
public class MyBean {

	@JmsListener(destination = "someQueue")
	public void processMessage(String content) {
		// ...
	}

}
```

> 查看[the Javadoc of @EnableJms](https://docs.spring.io/spring/docs/5.1.2.RELEASE/javadoc-api/org/springframework/jms/annotation/EnableJms.html)了解更多详情.

如果您需要创建更多 `JmsListenerContainerFactory` 实例，或者如果要覆盖默认值，则Spring Boot提供 `DefaultJmsListenerContainerFactoryConfigurer` ，您可以使用 `DefaultJmsListenerContainerFactoryConfigurer` 初始化 `DefaultJmsListenerContainerFactory` ，其设置与自动配置的设置相同.

例如，以下示例公开了另一个使用特定 `MessageConverter` 的工厂：

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

然后您可以在任何 `@JmsListener` 注释方法中使用工厂，如下所示：

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

高级消息队列协议（AMQP）是面向消息的中间件的平台中立的线级协议. Spring AMQP项目将核心Spring概念应用于基于AMQP的消息传递解决方案的开发. Spring Boot提供了几种通过RabbitMQ使用AMQP的便利，包括 `spring-boot-starter-amqp` “Starter”.

### 33.2.1 RabbitMQ支持

[RabbitMQ](https://www.rabbitmq.com/)是基于AMQP协议的轻量级，可靠，可扩展且可移植的消息代理. Spring使用 `RabbitMQ` 通过AMQP协议进行通信.

RabbitMQ配置由 `spring.rabbitmq.*` 中的外部配置属性控制.例如，您可以在 `application.properties` 中声明以下部分：

```java
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=admin
spring.rabbitmq.password=secret
```

如果上下文中存在 `ConnectionNameStrategy`  bean，它将自动用于命名由自动配置的 `ConnectionFactory` 创建的连接.有关更多支持的选项，请参阅[RabbitProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/amqp/RabbitProperties.java).

> 查看[Understanding AMQP, the protocol used by RabbitMQ](https://spring.io/blog/2010/06/14/understanding-amqp-the-protocol-used-by-rabbitmq/)了解更多详情.

### 33.2.2发送消息

Spring的 `AmqpTemplate` 和 `AmqpAdmin` 是自动配置的，您可以将它们直接自动装入自己的bean中，如下例所示：

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

> [RabbitMessagingTemplate](https://docs.spring.io/spring-amqp/docs/current/api/org/springframework/amqp/rabbit/core/RabbitMessagingTemplate.html)可以以类似的方式注入.如果定义了 `MessageConverter`  bean，它将自动关联到自动配置的 `AmqpTemplate` .

如有必要，任何定义为bean的_3195都会自动用于在RabbitMQ实例上声明相应的队列.

要重试操作，可以在 `AmqpTemplate` 上启用重试（例如，在代理连接丢失的情况下）：

```java
spring.rabbitmq.template.retry.enabled=true
spring.rabbitmq.template.retry.initial-interval=2s
```

默认情况下禁用重试.您还可以通过声明 `RabbitRetryTemplateCustomizer`  bean以编程方式自定义 `RetryTemplate` .

### 33.2.3收到留言

当Rabbit基础结构存在时，可以使用 `@RabbitListener` 注释任何bean以创建侦听器endpoints.如果未定义 `RabbitListenerContainerFactory` ，则会自动配置默认 `SimpleRabbitListenerContainerFactory` ，您可以使用 `spring.rabbitmq.listener.type` 属性切换到直接容器.如果定义了 `MessageConverter` 或 `MessageRecoverer`  bean，它将自动与默认工厂关联.

以下示例组件在 `someQueue` 队列上创建侦听器endpoints：

```java
@Component
public class MyBean {

	@RabbitListener(queues = "someQueue")
	public void processMessage(String content) {
		// ...
	}

}
```

> 参见[the Javadoc of @EnableRabbit](https://docs.spring.io/spring-amqp/docs/current/api/org/springframework/amqp/rabbit/annotation/EnableRabbit.html)了解更多详情.

如果您需要创建更多 `RabbitListenerContainerFactory` 实例或者如果要覆盖默认值，Spring Boot会提供 `SimpleRabbitListenerContainerFactoryConfigurer` 和 `DirectRabbitListenerContainerFactoryConfigurer` ，您可以使用它来初始化 `SimpleRabbitListenerContainerFactory` 和 `DirectRabbitListenerContainerFactory` ，其设置与自动配置使用的工厂相同.

> 您选择的容器类型无关紧要.这两个bean通过自动配置公开.

例如，以下配置类公开了另一个使用特定 `MessageConverter` 的工厂：

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

然后您可以在任何 `@RabbitListener` 注释方法中使用工厂，如下所示：

```java
@Component
public class MyBean {

	@RabbitListener(queues = "someQueue", containerFactory="myFactory")
	public void processMessage(String content) {
		// ...
	}

}
```

您可以启用重试来处理侦听器抛出异常的情况.默认情况下，使用 `RejectAndDontRequeueRecoverer` ，但您可以定义自己的 `MessageRecoverer` .当重试耗尽时，如果代理配置了这样做，则拒绝该消息并将其丢弃或路由到死信交换.默认情况下，禁用重试.您还可以通过声明 `RabbitRetryTemplateCustomizer`  bean以编程方式自定义 `RetryTemplate` .

|图片/ important.png |重要|
| ---- | ---- |
|默认情况下，如果禁用重试并且侦听器抛出异常，则会重试传递无限期.您可以通过两种方式修改此行为：将 `defaultRequeueRejected` 属性设置为 `false` ，以便尝试零重新传递或抛出 `AmqpRejectAndDontRequeueException` 以表示应拒绝该消息.后者是启用重试并且达到最大传递尝试次数时使用的机制. |

## 33.3 Apache Kafka支持

通过提供 `spring-kafka` 项目的自动配置来支持[Apache Kafka](https://kafka.apache.org/).

Kafka配置由 `spring.kafka.*` 中的外部配置属性控制.例如，您可以在 `application.properties` 中声明以下部分：

```java
spring.kafka.bootstrap-servers=localhost:9092
spring.kafka.consumer.group-id=myGroup
```

> 在启动时创建主题，添加 `NewTopic` 类型的bean.如果主题已存在，则忽略该bean.

有关更多支持的选项，请参阅[KafkaProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/kafka/KafkaProperties.java).

### 33.3.1发送消息

Spring的 `KafkaTemplate` 是自动配置的，您可以直接在自己的bean中自动装配它，如下例所示：

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

> 如果定义了属性 `spring.kafka.producer.transaction-id-prefix` ，则会自动配置 `KafkaTransactionManager` .此外，如果定义了 `RecordMessageConverter`  bean，它将自动关联到自动配置的 `KafkaTemplate` .

### 33.3.2收到消息

当存在Apache Kafka基础结构时，可以使用 `@KafkaListener` 注释任何bean以创建侦听器endpoints.如果未定义 `KafkaListenerContainerFactory` ，则会使用 `spring.kafka.listener.*` 中定义的键自动配置默认值.

以下组件在 `someTopic` 主题上创建侦听器endpoints：

```java
@Component
public class MyBean {

	@KafkaListener(topics = "someTopic")
	public void processMessage(String content) {
		// ...
	}

}
```

如果定义了 `KafkaTransactionManager`  bean，它将自动关联到容器工厂.同样，如果定义了 `RecordMessageConverter` ， `ErrorHandler` 或 `AfterRollbackProcessor`  bean，它将自动关联到默认工厂.

> A自定义 `ChainedKafkaTransactionManager` 必须标记为 `@Primary` ，因为它通常引用自动配置的 `KafkaTransactionManager`  bean.

### 33.3.3 Kafka Streams

Spring for Apache Kafka提供了一个工厂bean来创建 `StreamsBuilder` 对象并管理其流的生命周期.只要 `kafka-streams` 在类路径上并且通过 `@EnableKafkaStreams` 注释启用Kafka Streams，Spring Boot就会自动配置所需的 `KafkaStreamsConfiguration`  bean.

启用Kafka Streams意味着必须设置应用程序ID和引导程序服务器.前者可以使用 `spring.kafka.streams.application-id` 进行配置，如果未设置则默认为 `spring.application.name` .后者可以全局设置或专门为流而重写.

使用专用属性可以获得其他几个属性;可以使用 `spring.kafka.streams.properties` 命名空间设置其他任意Kafka属性.有关更多信息，另请参见[Section 33.3.4, “Additional Kafka Properties”](boot-features-messaging.html#boot-features-kafka-extra-props).

要使用工厂bean，只需将 `StreamsBuilder` 连接到 `@Bean` ，如以下示例所示：

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

默认情况下，由其创建的 `StreamBuilder` 对象管理的流将自动启动.您可以使用 `spring.kafka.streams.auto-startup` 属性自定义此行为.

### 33.3.4其他Kafkaproperties

自动配置支持的属性显示在[Appendix A, Common application properties](common-application-properties.html)中.请注意，在大多数情况下，这些属性（连字符或camelCase）直接映射到Apache Kafka点状属性.有关详细信息，请参阅Apache Kafka文档.

这些属性中的前几个适用于所有组件（生产环境者，使用者，管理员和流），但如果您希望使用不同的值，则可以在组件级别指定. Apache Kafka指定重要性为HIGH，MEDIUM或LOW的属性. Spring Boot自动配置支持所有HIGH重要性属性，一些选择的MEDIUM和LOW属性，以及任何没有默认值的属性.

只有Kafka支持的属性的子集可以直接通过 `KafkaProperties` 类获得.如果您希望使用不直接支持的其他属性配置生产环境者或使用者，请使用以下属性：

```java
spring.kafka.properties.prop.one=first
spring.kafka.admin.properties.prop.two=second
spring.kafka.consumer.properties.prop.three=third
spring.kafka.producer.properties.prop.four=fourth
spring.kafka.streams.properties.prop.five=fifth
```

这将常见的 `prop.one`  Kafka属性设置为 `first` （适用于生产环境者，使用者和管理员），将 `prop.two`  admin属性设置为 `second` ，将 `prop.three` 使用者属性设置为 `third` ，将 `prop.four` 生产环境者属性设置为 `fourth` ，将 `prop.five`  streams属性设置为 `fifth` .

您还可以按如下方式配置Spring Kafka  `JsonDeserializer` ：

```java
spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer
spring.kafka.consumer.properties.spring.json.value.default.type=com.example.Invoice
spring.kafka.consumer.properties.spring.json.trusted.packages=com.example,org.acme
```

同样，您可以禁用在标头中发送类型信息的 `JsonSerializer` 默认行为：

```java
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer
spring.kafka.producer.properties.spring.json.add.type.headers=false
```

|图片/ important.png |重要|
| ---- | ---- |
|以这种方式设置的属性将覆盖Spring Boot明确支持的任何配置项. |

