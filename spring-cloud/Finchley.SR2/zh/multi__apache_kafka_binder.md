## 37. Apache Kafka Binder

## 37.1用法

要使用Apache KafkaBinders，您需要将 `spring-cloud-stream-binder-kafka` 添加为Spring Cloud Stream应用程序的依赖项，如以下Maven示例所示：

```xml
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-stream-binder-kafka</artifactId>
</dependency>
```

或者，您也可以使用Spring Cloud Stream Kafka Starter，如下面的Maven示例所示：

```xml
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-starter-stream-kafka</artifactId>
</dependency>
```

## 37.2 Apache Kafka Binder概述

下图显示了Apache KafkaBinders如何运行的简化图：

**Figure 37.1. Kafka Binder** 

图像/图片/Kafka-binder.png

Apache Kafka Binder实现将每个目标映射到Apache Kafka主题.使用者组直接映射到相同的Apache Kafka概念.分区也直接映射到Apache Kafka分区.

Binders当前使用Apache Kafka  `kafka-clients`  1.0.0 jar，旨在与至少该版本的代理一起使用.此客户端可以与较旧的代理进行通信（请参阅Kafka文档），但某些功能可能不可用.例如，对于早于0.11.x.x的版本，不支持本机标头.此外，0.11.x.x不支持 `autoAddPartitions` 属性.

## 37.3配置选项

本节包含Apache KafkaBinders使用的配置选项.

有关与Binders有关的常用配置选项和属性，请参阅[core documentation](multi__configuration_options.html#binding-properties).

### 37.3.1 Kafka Binder Properties

spring.cloud.stream.kafka.binder.brokers KafkaBinders连接的代理列表.默认值：localhost. spring.cloud.stream.kafka.binder.defaultBrokerPort代理允许使用或不使用端口信息指定的主机（例如，host1，host2：port2）.这在代理列表中未配置端口时设置默认端口.默认值：9092.spring.cloud.stream.kafka.binder.configuration传递给Binders创建的所有客户端的客户端属性（生产环境者和使用者）的键/值映射.由于生产环境者和消费者都使用这些属性，因此应将使用限制为常见属性 - 例如，安全设置.默认值：空Map. spring.cloud.stream.kafka.binder.headers由Binders传输的自定义标头列表.仅在与旧版应用程序（⇐1.3.x）通信且kafka-clients版本<0.11.0.0时才需要.较新版本本身支持标头.默认值：空. spring.cloud.stream.kafka.binder.healthTimeout等待获取分区信息的时间，以秒为单位.如果此计时器到期，Health状况将报告为关闭.默认值：10.spring.cloud.stream.kafka.binder.requiredAcks代理上所需的ack数.有关生产环境者acks属性，请参阅Kafka文档.默认值：1.spring.cloud.stream.kafka.binder.minPartitionCount仅在设置了autoCreateTopics或autoAddPartitions时有效.Binders在其生成或使用数据的主题上配置的全局最小分区数.它可以被生产环境者的partitionCount设置或生成器的instanceCount *并发设置的值取代（如果其中任何一个更大）.默认值：1.spring.cloud.stream.kafka.binder.replicationFactor如果autoCreateTopics处于活动状态，则自动创建主题的复制因子.可以在每个绑定上重写.默认值：1.spring.cloud.stream.kafka.binder.autoCreateTopics如果设置为true，则Binders会自动创建新主题.如果设置为false，则绑定程序依赖于已配置的主题.在后一种情况下，如果主题不存在，则绑定程序无法启动.注意此设置独立于代理的auto.topic.create.enable设置，不会影响它.如果服务器设置为自动创建主题，则可以使用默认代理设置将它们创建为元数据检索请求的一部分.默认值：true. spring.cloud.stream.kafka.binder.autoAddPartitions如果设置为true，则Binders会根据需要创建新分区.如果设置为false，则绑定程序将依赖于已配置主题的分区大小.如果是分区目标主题的计数小于预期值，绑定程序无法启动.默认值：false. spring.cloud.stream.kafka.binder.transaction.transactionIdPrefix在Binders中启用事务.请参阅kafka文档中的transaction.id和spring-kafka文档中的Transactions.启用事务时，将忽略各个生产环境者属性，并且所有生产环境者都使用spring.cloud.stream.kafka.binder.transaction.producer.*属性.默认null（无事务）spring.cloud.stream.kafka.binder.transaction.producer.*事务Binders中生产环境者的全局生产环境者属性.请参阅spring.cloud.stream.kafka.binder.transaction.transactionIdPrefix和第37.3.3节“Kafka Producer属性”以及所有Binders支持的常规生产环境者属性.默认值：查看各个生产环境者属性. spring.cloud.stream.kafka.binder.headerMapperBeanName KafkaHeaderMapper的bean名称，用于将弹出消息标头映射到Kafka标头和从Kafka标头映射.例如，如果您希望在对标头使用JSON反序列化的DefaultKafkaHeaderMapper中自定义受信任的包，请使用此选项.默认值：无.

### 37.3.2 Kafka Consumer Properties

以下属性仅适用于Kafka使用者，必须以 `spring.cloud.stream.kafka.bindings.<channelName>.consumer.` 为前缀.

admin.configuration配置主题时使用的Kafka主题属性的映射 - 例如，spring.cloud.stream.kafka.bindings.input.consumer.admin.configuration.message.format.version = 0.9.0.0默认值：无. admin.replicas-assignment一个Map <Integer，List <Integer >>的副本分配，其中键是分区，值是赋值.在配置新主题时使用.请参阅kafka-clients jar中的NewTopic Javadoc.默认值：无. admin.replication-factor配置主题时要使用的复制因子.覆盖Binders范围的设置.如果存在副本分配，则忽略.默认值：none（使用绑定程序范围的默认值1）. autoRebalanceEnabled如果为true，则主题分区会在使用者组的成员之间自动重新平衡.如果为false，则会根据spring.cloud.stream.instanceCount和spring.cloud.stream.instanceIndex为每个使用者分配一组固定的分区.这需要在每个已启动的实例上正确设置spring.cloud.stream.instanceCount和spring.cloud.stream.instanceIndex属性.在这种情况下，spring.cloud.stream.instanceCount属性的值通常必须大于1.默认值：true. ackEachRecord当autoCommitOffset为true时，此设置指示在处理每个记录后是否提交偏移量.默认情况下，在处理consumer.poll（）返回的记录批中的所有记录之后，将提交偏移.可以使用max.poll.records Kafka属性控制轮询返回的记录数，该属性通过使用者配置属性设置.将此设置为true可能会导致性能下降，但这样做会降低发生故障时重新传送记录的可能性.另外，请参阅binder requiredAcks属性，这也会影响提交偏移的性能.默认值：false. autoCommitOffset是否在处理消息时自动提交偏移量.如果设置为false，则入站消息中将出现带有org.springframework.kafka.support.Acknowledgment类型Headers的密钥kafka_acknowledgment的标头.应用程序可以使用此标头来确认消息.有关详细信息，请参阅示例部分当此属性设置为false时，Kafka binder将ack模式设置为org.springframework.kafka.listener.AbstractMessageListenerContainer.AckMode.MANUAL，并且应用程序负责确认记录.另见ackEachRecord.默认值：true. autoCommitOnError仅在autoCommitOffset设置为true时有效.如果设置为false，则禁止对导致错误的消息进行自动提交，并仅为成功的消息提交.它允许流在上次成功处理的消息中自动重放，以防出现持续故障.如果设置为true，则始终自动提交（如果启用了自动提交）.如果未设置（默认值），则它实际上具有与enableDlq相同的值，如果将错误消息发送到DLQ则自动提交错误消息，否则不提交.默认值：未设置. resetOffsets是否将使用者的偏移重置为startOffset提供的值.默认值：false. startOffset新组的起始偏移量.允许值：最早和最新.如果为消费者'绑定'明确设置了使用者组（通过spring.cloud.stream.bindings.<channelName> .group），则'startOffset'设置为最早.否则，对于匿名消费者组，它被设置为最新.另请参阅resetOffsets（此列表的前面部分）.默认值：null（相当于最早）. enableDlq设置为true时，它为使用者启用DLQ行为.默认情况下，导致错误的消息是转发到名为error.<destination>.<group>的主题.可以通过设置dlqName属性来配置DLQ主题名称.对于错误数量相对较小并且重放整个原始主题的情况可能过于繁琐的情况，这为更常见的Kafka重放场景提供了备选选项.有关更多信息，请参见第37.6节“死信主题处理”处理.从版本2.0开始，使用以下标头增强发送到DLQ主题的消息：x-original-topic，x-exception-message和x-exception-stacktrace as byte [].默认值：false.配置包含包含通用Kafka使用者属性的键/值对的映射.默认值：空Map. dlqName用于接收错误消息的DLQ主题的名称.默认值：null（如果未指定，则会将导致错误的消息转发到名为error.<destination>.<group>）的主题. dlqProducerProperties使用此属性，可以设置DLQ特定的生成器属性.可以通过此属性设置通过kafka生产环境者属性提供的所有属性.默认值：默认Kafka生产环境者属性. standardHeaders指示入站通道适配器填充的标准头.允许的值：none，id，timestamp或两者.如果使用本机反序列化并且接收消息的第一个组件需要id（例如配置为使用JDBC消息存储的聚合器），则非常有用.默认值：none converterBeanName实现RecordMessageConverter的bean的名称.在入站通道适配器中用于替换默认的MessagingMessageConverter.默认值：null idleEventInterval指示最近未收到消息的事件之间的间隔（以毫秒为单位）.使用ApplicationListener <ListenerContainerIdleEvent>来接收这些事件.有关用法示例，请参阅“示例：暂停和恢复使用者”一节.默认值：30000

### 37.3.3 Kafka Producer Properties

以下属性仅适用于Kafka生产环境者，必须以 `spring.cloud.stream.kafka.bindings.<channelName>.producer.` 为前缀.

admin.configuration配置新主题时使用的Kafka主题属性的映射 - 例如，spring.cloud.stream.kafka.bindings.input.consumer.admin.configuration.message.format.version = 0.9.0.0默认值：无. admin.replicas-assignment一个Map <Integer，List <Integer >>的副本分配，其中键是分区，值是赋值.在配置新主题时使用.请参阅kafka-clients jar中的NewTopic javadoc.默认值：无. admin.replication-factor配置新主题时要使用的复制因子.覆盖Binders范围的设置.如果存在副本分配，则忽略.默认值：none（使用绑定程序范围的默认值1）. bufferSize Kafka生产环境者在发送之前尝试批量处理的数据的上限（以字节为单位）.默认值：16384.sync生成器是否同步.默认值：false. batchTimeout生成器在发送消息之前等待允许更多消息在同一批次中累积的时间. （通常，生产环境者根本不会等待，只是发送在上一次发送过程中累积的所有消息.）非零值可能会以延迟为代价来增加吞吐量.默认值：0.messageKeyExpression针对用于填充生成的Kafka消息的密钥的传出消息计算SpEL表达式 - 例如，headers ['myKey'].无法使用有效负载，因为在评估此表达式时，有效负载已经是byte []的形式.默认值：无. headerPatterns以逗号分隔的简单模式列表，用于匹配要映射到ProducerRecord中的Kafka标头的Spring消息传递头.模式可以以通配符（星号）开头或结尾.前缀为！可以否定模式.匹配在第一场比赛后停止（正面或负面）.例如！问，因为*会通过灰但不会问.永远不会映射id和timestamp.默认值：*（所有标头 - 除了id和timestamp）配置带有包含通用Kafka生产环境者属性的键/值对的Map.默认值：空Map.

>  KafkaBinders使用生产环境者的 `partitionCount` 设置作为提示来创建具有给定分区计数的主题（与 `minPartitionCount` 一起使用，两者中的最大值是使用的值）.在为Binders配置 `minPartitionCount` 和为应用程序配置 `partitionCount` 时请务必谨慎，因为使用的值较大.如果已存在具有较小分区计数且禁用 `autoAddPartitions` 的主题（默认值），则绑定程序无法启动.如果已存在具有较小分区计数且已启用 `autoAddPartitions` 的主题，则会添加新分区.如果主题已存在且分区数大于（ `minPartitionCount` 或 `partitionCount` ）的最大分区数，则使用现有分区计数.

### 37.3.4用法示例

在本节中，我们将展示对特定方案使用前面的属性.

#### Example：设置autoCommitOffset虚假和依赖手工Acking

此示例说明了如何在消费者应用程序中手动确认偏移.

此示例要求 `spring.cloud.stream.kafka.bindings.input.consumer.autoCommitOffset` 设置为 `false` .使用相应的输入通道名称作为示例.

```java
@SpringBootApplication
@EnableBinding(Sink.class)
public class ManuallyAcknowdledgingConsumer {

public static void main(String[] args) {
SpringApplication.run(ManuallyAcknowdledgingConsumer.class, args);
}

@StreamListener(Sink.INPUT)
public void process(Message<?> message) {
Acknowledgment acknowledgment = message.getHeaders().get(KafkaHeaders.ACKNOWLEDGMENT, Acknowledgment.class);
if (acknowledgment != null) {
System.out.println("Acknowledgment provided");
acknowledgment.acknowledge();
}
}
}
```

#### Example：安全配置

Apache Kafka 0.9支持客户端和代理之间的安全连接.要利用此功能，请遵循[Apache Kafka Documentation](https://kafka.apache.org/090/documentation.html#security_configclients)中的指南以及Kafka 0.9 [security guidelines from the Confluent documentation](http://docs.confluent.io/2.0.0/kafka/security.html).使用 `spring.cloud.stream.kafka.binder.configuration` 选项为Binders创建的所有客户端设置安全属性.

例如，要将 `security.protocol` 设置为 `SASL_SSL` ，请设置以下属性：

```java
spring.cloud.stream.kafka.binder.configuration.security.protocol=SASL_SSL
```

可以以类似的方式设置所有其他安全属性.

使用Kerberos时，请按照[reference documentation](https://kafka.apache.org/090/documentation.html#security_sasl_clientconfig)中的说明创建和引用JAAS配置.

Spring Cloud Stream支持使用JAAS配置文件和Spring Boot属性将JAAS配置信息传递给应用程序.

##### 使用JAAS配置文件

可以使用系统属性为Spring Cloud Stream应用程序设置JAAS和（可选）krb5文件位置.以下示例显示如何使用JAAS配置文件使用SASL和Kerberos启动Spring Cloud Stream应用程序：

```java
java -Djava.security.auth.login.config=/path.to/kafka_client_jaas.conf -jar log.jar \
--spring.cloud.stream.kafka.binder.brokers=secure.server:9092 \
--spring.cloud.stream.bindings.input.destination=stream.ticktock \
--spring.cloud.stream.kafka.binder.configuration.security.protocol=SASL_PLAINTEXT
```

##### 使用Spring Boot属性

作为拥有JAAS配置文件的替代方法，Spring Cloud Stream提供了一种使用Spring Boot属性为Spring Cloud Stream应用程序设置JAAS配置的机制.

以下属性可用于配置Kafka客户端的登录上下文：

spring.cloud.stream.kafka.binder.jaas.loginModule登录模块名称.没有必要在正常情况下设置.默认值：com.sun.security.auth.module.Krb5LoginModule. spring.cloud.stream.kafka.binder.jaas.controlFlag登录模块的控制标志.默认值：必填. spring.cloud.stream.kafka.binder.jaas.options使用包含登录模块选项的键/值对映射.默认值：空Map.

以下示例说明如何使用Spring Boot配置属性启动带有SASL和Kerberos的Spring Cloud Stream应用程序：

```java
java --spring.cloud.stream.kafka.binder.brokers=secure.server:9092 \
--spring.cloud.stream.bindings.input.destination=stream.ticktock \
--spring.cloud.stream.kafka.binder.autoCreateTopics=false \
--spring.cloud.stream.kafka.binder.configuration.security.protocol=SASL_PLAINTEXT \
--spring.cloud.stream.kafka.binder.jaas.options.useKeyTab=true \
--spring.cloud.stream.kafka.binder.jaas.options.storeKey=true \
--spring.cloud.stream.kafka.binder.jaas.options.keyTab=/etc/security/keytabs/kafka_client.keytab \
--spring.cloud.stream.kafka.binder.jaas.options.principal=kafka-client-1@EXAMPLE.COM
```

上面的示例表示以下JAAS文件的等效项：

```java
KafkaClient {
com.sun.security.auth.module.Krb5LoginModule required
useKeyTab=true
storeKey=true
keyTab="/etc/security/keytabs/kafka_client.keytab"
principal="[emailprotected]";
};
```

如果所需主题已存在于代理上或将由管理员创建，则可以关闭自动创建，并且只需要发送客户端JAAS属性.

> 不要在同一个应用程序中混合使用JAAS配置文件和Spring Boot属性.如果 `-Djava.security.auth.login.config` 系统属性已存在，则Spring Cloud Stream将忽略Spring Boot属性.

> Be使用带有Kerberos的 `autoCreateTopics` 和 `autoAddPartitions` 时要小心.通常，应用程序可能使用在Kafka和Zookeeper中没有管理权限的主体.因此，依赖Spring Cloud Stream来创建/修改主题可能会失败.在安全环境中，我们强烈建议您使用Kafka工具以管理方式创建主题和管理ACL.

#### 示例：暂停和恢复消费者

如果您希望暂停使用但不会导致分区重新平衡，则可以暂停和恢复使用者.将 `Consumer` 作为参数添加到 `@StreamListener` 可以实现这一点.要恢复，您需要一个 `ApplicationListener`  for  `ListenerContainerIdleEvent` 实例.发布事件的频率由 `idleEventInterval` 属性控制.由于使用者不是线程安全的，因此必须在调用线程上调用这些方法.

以下简单的应用程序显示了如何暂停和恢复：

```java
@SpringBootApplication
@EnableBinding(Sink.class)
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

	@StreamListener(Sink.INPUT)
	public void in(String in, @Header(KafkaHeaders.CONSUMER) Consumer<?, ?> consumer) {
		System.out.println(in);
		consumer.pause(Collections.singleton(new TopicPartition("myTopic", 0)));
	}

	@Bean
	public ApplicationListener<ListenerContainerIdleEvent> idleListener() {
		return event -> {
			System.out.println(event);
			if (event.getConsumer().paused().size() > 0) {
				event.getConsumer().resume(event.getConsumer().paused());
			}
		};
	}

}
```

## 37.4错误Channels

从版本1.3开始，Binders无条件地为每个使用者目标向错误通道发送异常，并且还可以配置为将异步生成器发送失败发送到错误通道.有关更多信息，请参见[Section 27.4, “Error Handling”](multi__programming_model.html#spring-cloud-stream-overview-error-handling).

发送失败的 `ErrorMessage` 的有效负载是 `KafkaSendFailureException` ，其属性为：

-  `failedMessage` ：无法发送的Spring Messaging  `Message<?>` .

-  `record` ：从 `failedMessage` 创建的原始 `ProducerRecord` 

生产环境者异常没有自动处理（例如发送到[Dead-Letter queue](multi__apache_kafka_binder.html#kafka-dlq-processing)）.您可以使用自己的Spring Integration流程来使用这些异常.

## 37.5 Kafka Metrics

KafkaBinders模块公开以下指标：

`spring.cloud.stream.binder.kafka.someGroup.someTopic.lag` ：此度量标准指示给定的使用者组从给定的Binders主题尚未消耗的消息数量.例如，如果度量标准 `spring.cloud.stream.binder.kafka.myGroup.myTopic.lag` 的值为 `1000` ，则名为 `myGroup` 的使用者组等待从主题calle  `myTopic` 中消耗 `1000` 消息.此指标对于向PaaS平台提供自动缩放反馈特别有用.

## 37.6死信主题处理

因为您无法预测用户将如何处理死信消息，所以框架不提供任何标准机制来处理它们.如果死字体的原因是暂时的，您可能希望将消息路由回原始主题.但是，如果问题是一个永久性问题，可能导致无限循环.本主题中的示例Spring Boot应用程序是如何将这些消息路由回原始主题的示例，但是在三次尝试之后它将它们移动到“停车场”主题.该应用程序是另一个Spring-cloud-stream应用程序，它从死信主题中读取.它在5秒内没有收到任何消息时终止.

这些示例假设原始目标是 `so8400out` ，而使用者组是 `so8400` .

有几种策略需要考虑：

- Consider仅在主应用程序未运行时运行重新路由.否则，瞬态错误的重试会很快耗尽.

- 或者，使用两阶段方法：使用此应用程序路由到第三个主题，另一个用于从那里路由回主要主题.

以下代码清单显示了示例应用程序：

**application.properties.** 

```java
spring.cloud.stream.bindings.input.group=so8400replay
spring.cloud.stream.bindings.input.destination=error.so8400out.so8400

spring.cloud.stream.bindings.output.destination=so8400out
spring.cloud.stream.bindings.output.producer.partitioned=true

spring.cloud.stream.bindings.parkingLot.destination=so8400in.parkingLot
spring.cloud.stream.bindings.parkingLot.producer.partitioned=true

spring.cloud.stream.kafka.binder.configuration.auto.offset.reset=earliest

spring.cloud.stream.kafka.binder.headers=x-retries
```

**Application.** 

```java
@SpringBootApplication
@EnableBinding(TwoOutputProcessor.class)
public class ReRouteDlqKApplication implements CommandLineRunner {

private static final String X_RETRIES_HEADER = "x-retries";

public static void main(String[] args) {
SpringApplication.run(ReRouteDlqKApplication.class, args).close();
}

private final AtomicInteger processed = new AtomicInteger();

@Autowired
private MessageChannel parkingLot;

@StreamListener(Processor.INPUT)
@SendTo(Processor.OUTPUT)
public Message<?> reRoute(Message<?> failed) {
processed.incrementAndGet();
Integer retries = failed.getHeaders().get(X_RETRIES_HEADER, Integer.class);
if (retries == null) {
System.out.println("First retry for " + failed);
return MessageBuilder.fromMessage(failed)
.setHeader(X_RETRIES_HEADER, new Integer(1))
.setHeader(BinderHeaders.PARTITION_OVERRIDE,
failed.getHeaders().get(KafkaHeaders.RECEIVED_PARTITION_ID))
.build();
}
else if (retries.intValue() < 3) {
System.out.println("Another retry for " + failed);
return MessageBuilder.fromMessage(failed)
.setHeader(X_RETRIES_HEADER, new Integer(retries.intValue() + 1))
.setHeader(BinderHeaders.PARTITION_OVERRIDE,
failed.getHeaders().get(KafkaHeaders.RECEIVED_PARTITION_ID))
.build();
}
else {
System.out.println("Retries exhausted for " + failed);
parkingLot.send(MessageBuilder.fromMessage(failed)
.setHeader(BinderHeaders.PARTITION_OVERRIDE,
failed.getHeaders().get(KafkaHeaders.RECEIVED_PARTITION_ID))
.build());
}
return null;
}

@Override
public void run(String... args) throws Exception {
while (true) {
int count = this.processed.get();
Thread.sleep(5000);
if (count == this.processed.get()) {
System.out.println("Idle, terminating");
return;
}
}
}

public interface TwoOutputProcessor extends Processor {

@Output("parkingLot")
MessageChannel parkingLot();

}

}
```

## 37.7与KafkaBinders分区

Apache Kafka本机支持主题分区.

有时将数据发送到特定分区是有利的 - 例如，当您要严格订购消息处理时（特定客户的所有消息都应该转到同一分区）.

以下示例显示如何配置生产环境者和消费者方：

```java
@SpringBootApplication
@EnableBinding(Source.class)
public class KafkaPartitionProducerApplication {

private static final Random RANDOM = new Random(System.currentTimeMillis());

private static final String[] data = new String[] {
"foo1", "bar1", "qux1",
"foo2", "bar2", "qux2",
"foo3", "bar3", "qux3",
"foo4", "bar4", "qux4",
};

public static void main(String[] args) {
new SpringApplicationBuilder(KafkaPartitionProducerApplication.class)
.web(false)
.run(args);
}

@InboundChannelAdapter(channel = Source.OUTPUT, poller = @Poller(fixedRate = "5000"))
public Message<?> generate() {
String value = data[RANDOM.nextInt(data.length)];
System.out.println("Sending: " + value);
return MessageBuilder.withPayload(value)
.setHeader("partitionKey", value)
.build();
}

}
```

**application.yml.** 

```java
spring:
cloud:
stream:
bindings:
output:
destination: partitioned.topic
producer:
partitioned: true
partition-key-expression: headers['partitionKey']
partition-count: 12
```

|图片/ important.png |重要|
| ---- | ---- |
|必须配置主题以具有足够的分区以实现所有使用者组的所需并发性.上面的配置最多支持12个消费者实例（如果他们的 `concurrency` 是2，则为6，如果他们的并发性为3则为4，依此类推）.通常最好“过度配置”分区以允许将来增加消费者或并发性. |

> 上述配置使用默认分区（ `key.hashCode() % partitionCount` ）.根据键值，这可能会或可能不会提供适当平衡的算法.您可以使用 `partitionSelectorExpression` 或 `partitionSelectorClass` 属性覆盖此默认值.

由于分区由Kafka本地处理，因此在消费者方面不需要特殊配置. Kafka在实例之间分配分区.

以下Spring Boot应用程序侦听Kafka流并打印（到控制台）每条消息所针对的分区ID：

```java
@SpringBootApplication
@EnableBinding(Sink.class)
public class KafkaPartitionConsumerApplication {

public static void main(String[] args) {
new SpringApplicationBuilder(KafkaPartitionConsumerApplication.class)
.web(false)
.run(args);
}

@StreamListener(Sink.INPUT)
public void listen(@Payload String in, @Header(KafkaHeaders.RECEIVED_PARTITION_ID) int partition) {
System.out.println(in + " received from partition " + partition);
}

}
```

**application.yml.** 

```java
spring:
cloud:
stream:
bindings:
input:
destination: partitioned.topic
group: myGroup
```

您可以根据需要添加实例. Kafka重新平衡分区分配.如果实例计数（或 `instance count * concurrency` ）超过分区数，则某些消费者处于空闲状态.

