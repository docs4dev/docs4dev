## 37. Apache Kafka Binder

## 37.1 Usage

To use Apache Kafka binder, you need to add  `spring-cloud-stream-binder-kafka`  as a dependency to your Spring Cloud Stream application, as shown in the following example for Maven:

```xml
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-stream-binder-kafka</artifactId>
</dependency>
```

Alternatively, you can also use the Spring Cloud Stream Kafka Starter, as shown inn the following example for Maven:

```xml
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-starter-stream-kafka</artifactId>
</dependency>
```

## 37.2 Apache Kafka Binder Overview

The following image shows a simplified diagram of how the Apache Kafka binder operates:

**Figure 37.1. Kafka Binder** 

images/images/kafka-binder.png

The Apache Kafka Binder implementation maps each destination to an Apache Kafka topic. The consumer group maps directly to the same Apache Kafka concept. Partitioning also maps directly to Apache Kafka partitions as well.

The binder currently uses the Apache Kafka  `kafka-clients`  1.0.0 jar and is designed to be used with a broker of at least that version. This client can communicate with older brokers (see the Kafka documentation), but certain features may not be available. For example, with versions earlier than 0.11.x.x, native headers are not supported. Also, 0.11.x.x does not support the  `autoAddPartitions`  property.

## 37.3 Configuration Options

This section contains the configuration options used by the Apache Kafka binder.

For common configuration options and properties pertaining to binder, see the [core documentation](multi__configuration_options.html#binding-properties).

### 37.3.1 Kafka Binder Properties

spring.cloud.stream.kafka.binder.brokers A list of brokers to which the Kafka binder connects. Default: localhost. spring.cloud.stream.kafka.binder.defaultBrokerPort brokers allows hosts specified with or without port information (for example, host1,host2:port2). This sets the default port when no port is configured in the broker list. Default: 9092. spring.cloud.stream.kafka.binder.configuration Key/Value map of client properties (both producers and consumer) passed to all clients created by the binder. Due to the fact that these properties are used by both producers and consumers, usage should be restricted to common properties — for example, security settings. Default: Empty map. spring.cloud.stream.kafka.binder.headers The list of custom headers that are transported by the binder. Only required when communicating with older applications (⇐ 1.3.x) with a kafka-clients version < 0.11.0.0. Newer versions support headers natively. Default: empty. spring.cloud.stream.kafka.binder.healthTimeout The time to wait to get partition information, in seconds. Health reports as down if this timer expires. Default: 10. spring.cloud.stream.kafka.binder.requiredAcks The number of required acks on the broker. See the Kafka documentation for the producer acks property. Default: 1. spring.cloud.stream.kafka.binder.minPartitionCount Effective only if autoCreateTopics or autoAddPartitions is set. The global minimum number of partitions that the binder configures on topics on which it produces or consumes data. It can be superseded by the partitionCount setting of the producer or by the value of instanceCount * concurrency settings of the producer (if either is larger). Default: 1. spring.cloud.stream.kafka.binder.replicationFactor The replication factor of auto-created topics if autoCreateTopics is active. Can be overridden on each binding. Default: 1. spring.cloud.stream.kafka.binder.autoCreateTopics If set to true, the binder creates new topics automatically. If set to false, the binder relies on the topics being already configured. In the latter case, if the topics do not exist, the binder fails to start. Note This setting is independent of the auto.topic.create.enable setting of the broker and does not influence it. If the server is set to auto-create topics, they may be created as part of the metadata retrieval request, with default broker settings. Default: true. spring.cloud.stream.kafka.binder.autoAddPartitions If set to true, the binder creates new partitions if required. If set to false, the binder relies on the partition size of the topic being already configured. If the partition count of the target topic is smaller than the expected value, the binder fails to start. Default: false. spring.cloud.stream.kafka.binder.transaction.transactionIdPrefix Enables transactions in the binder. See transaction.id in the Kafka documentation and Transactions in the spring-kafka documentation. When transactions are enabled, individual producer properties are ignored and all producers use the spring.cloud.stream.kafka.binder.transaction.producer.* properties. Default null (no transactions) spring.cloud.stream.kafka.binder.transaction.producer.* Global producer properties for producers in a transactional binder. See spring.cloud.stream.kafka.binder.transaction.transactionIdPrefix and Section 37.3.3, “Kafka Producer Properties” and the general producer properties supported by all binders. Default: See individual producer properties. spring.cloud.stream.kafka.binder.headerMapperBeanName The bean name of a KafkaHeaderMapper used for mapping spring-messaging headers to and from Kafka headers. Use this, for example, if you wish to customize the trusted packages in a DefaultKafkaHeaderMapper that uses JSON deserialization for the headers. Default: none.

### 37.3.2 Kafka Consumer Properties

The following properties are available for Kafka consumers only and must be prefixed with  `spring.cloud.stream.kafka.bindings.<channelName>.consumer.` .

admin.configuration A Map of Kafka topic properties used when provisioning topics — for example, spring.cloud.stream.kafka.bindings.input.consumer.admin.configuration.message.format.version=0.9.0.0 Default: none. admin.replicas-assignment A Map<Integer, List<Integer>> of replica assignments, with the key being the partition and the value being the assignments. Used when provisioning new topics. See the NewTopic Javadocs in the kafka-clients jar. Default: none. admin.replication-factor The replication factor to use when provisioning topics. Overrides the binder-wide setting. Ignored if replicas-assignments is present. Default: none (the binder-wide default of 1 is used). autoRebalanceEnabled When true, topic partitions is automatically rebalanced between the members of a consumer group. When false, each consumer is assigned a fixed set of partitions based on spring.cloud.stream.instanceCount and spring.cloud.stream.instanceIndex. This requires both the spring.cloud.stream.instanceCount and spring.cloud.stream.instanceIndex properties to be set appropriately on each launched instance. The value of the spring.cloud.stream.instanceCount property must typically be greater than 1 in this case. Default: true. ackEachRecord When autoCommitOffset is true, this setting dictates whether to commit the offset after each record is processed. By default, offsets are committed after all records in the batch of records returned by consumer.poll() have been processed. The number of records returned by a poll can be controlled with the max.poll.records Kafka property, which is set through the consumer configuration property. Setting this to true may cause a degradation in performance, but doing so reduces the likelihood of redelivered records when a failure occurs. Also, see the binder requiredAcks property, which also affects the performance of committing offsets. Default: false. autoCommitOffset Whether to autocommit offsets when a message has been processed. If set to false, a header with the key kafka_acknowledgment of the type org.springframework.kafka.support.Acknowledgment header is present in the inbound message. Applications may use this header for acknowledging messages. See the examples section for details. When this property is set to false, Kafka binder sets the ack mode to org.springframework.kafka.listener.AbstractMessageListenerContainer.AckMode.MANUAL and the application is responsible for acknowledging records. Also see ackEachRecord. Default: true. autoCommitOnError Effective only if autoCommitOffset is set to true. If set to false, it suppresses auto-commits for messages that result in errors and commits only for successful messages. It allows a stream to automatically replay from the last successfully processed message, in case of persistent failures. If set to true, it always auto-commits (if auto-commit is enabled). If not set (the default), it effectively has the same value as enableDlq, auto-committing erroneous messages if they are sent to a DLQ and not committing them otherwise. Default: not set. resetOffsets Whether to reset offsets on the consumer to the value provided by startOffset. Default: false. startOffset The starting offset for new groups. Allowed values: earliest and latest. If the consumer group is set explicitly for the consumer 'binding' (through spring.cloud.stream.bindings.<channelName>.group), 'startOffset' is set to earliest. Otherwise, it is set to latest for the anonymous consumer group. Also see resetOffsets (earlier in this list). Default: null (equivalent to earliest). enableDlq When set to true, it enables DLQ behavior for the consumer. By default, messages that result in errors are forwarded to a topic named error.<destination>.<group>. The DLQ topic name can be configurable by setting the dlqName property. This provides an alternative option to the more common Kafka replay scenario for the case when the number of errors is relatively small and replaying the entire original topic may be too cumbersome. See Section 37.6, “Dead-Letter Topic Processing” processing for more information. Starting with version 2.0, messages sent to the DLQ topic are enhanced with the following headers: x-original-topic, x-exception-message, and x-exception-stacktrace as byte[]. Default: false. configuration Map with a key/value pair containing generic Kafka consumer properties. Default: Empty map. dlqName The name of the DLQ topic to receive the error messages. Default: null (If not specified, messages that result in errors are forwarded to a topic named error.<destination>.<group>). dlqProducerProperties Using this, DLQ-specific producer properties can be set. All the properties available through kafka producer properties can be set through this property. Default: Default Kafka producer properties. standardHeaders Indicates which standard headers are populated by the inbound channel adapter. Allowed values: none, id, timestamp, or both. Useful if using native deserialization and the first component to receive a message needs an id (such as an aggregator that is configured to use a JDBC message store). Default: none converterBeanName The name of a bean that implements RecordMessageConverter. Used in the inbound channel adapter to replace the default MessagingMessageConverter. Default: null idleEventInterval The interval, in milliseconds, between events indicating that no messages have recently been received. Use an ApplicationListener<ListenerContainerIdleEvent> to receive these events. See the section called “Example: Pausing and Resuming the Consumer” for a usage example. Default: 30000

### 37.3.3 Kafka Producer Properties

The following properties are available for Kafka producers only and must be prefixed with  `spring.cloud.stream.kafka.bindings.<channelName>.producer.` .

admin.configuration A Map of Kafka topic properties used when provisioning new topics — for example, spring.cloud.stream.kafka.bindings.input.consumer.admin.configuration.message.format.version=0.9.0.0 Default: none. admin.replicas-assignment A Map<Integer, List<Integer>> of replica assignments, with the key being the partition and the value being the assignments. Used when provisioning new topics. See NewTopic javadocs in the kafka-clients jar. Default: none. admin.replication-factor The replication factor to use when provisioning new topics. Overrides the binder-wide setting. Ignored if replicas-assignments is present. Default: none (the binder-wide default of 1 is used). bufferSize Upper limit, in bytes, of how much data the Kafka producer attempts to batch before sending. Default: 16384. sync Whether the producer is synchronous. Default: false. batchTimeout How long the producer waits to allow more messages to accumulate in the same batch before sending the messages. (Normally, the producer does not wait at all and simply sends all the messages that accumulated while the previous send was in progress.) A non-zero value may increase throughput at the expense of latency. Default: 0. messageKeyExpression A SpEL expression evaluated against the outgoing message used to populate the key of the produced Kafka message — for example, headers['myKey']. The payload cannot be used because, by the time this expression is evaluated, the payload is already in the form of a byte[]. Default: none. headerPatterns A comma-delimited list of simple patterns to match Spring messaging headers to be mapped to the Kafka Headers in the ProducerRecord. Patterns can begin or end with the wildcard character (asterisk). Patterns can be negated by prefixing with !. Matching stops after the first match (positive or negative). For example !ask,as* will pass ash but not ask. id and timestamp are never mapped. Default: * (all headers - except the id and timestamp) configuration Map with a key/value pair containing generic Kafka producer properties. Default: Empty map.

> The Kafka binder uses the  `partitionCount`  setting of the producer as a hint to create a topic with the given partition count (in conjunction with the  `minPartitionCount` , the maximum of the two being the value being used). Exercise caution when configuring both  `minPartitionCount`  for a binder and  `partitionCount`  for an application, as the larger value is used. If a topic already exists with a smaller partition count and  `autoAddPartitions`  is disabled (the default), the binder fails to start. If a topic already exists with a smaller partition count and  `autoAddPartitions`  is enabled, new partitions are added. If a topic already exists with a larger number of partitions than the maximum of ( `minPartitionCount`  or  `partitionCount` ), the existing partition count is used.

### 37.3.4 Usage examples

In this section, we show the use of the preceding properties for specific scenarios.

#### Example: Setting autoCommitOffset to false and Relying on Manual Acking

This example illustrates how one may manually acknowledge offsets in a consumer application.

This example requires that  `spring.cloud.stream.kafka.bindings.input.consumer.autoCommitOffset`  be set to  `false` . Use the corresponding input channel name for your example.

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

#### Example: Security Configuration

Apache Kafka 0.9 supports secure connections between client and brokers. To take advantage of this feature, follow the guidelines in the [Apache Kafka Documentation](https://kafka.apache.org/090/documentation.html#security_configclients) as well as the Kafka 0.9 [security guidelines from the Confluent documentation](http://docs.confluent.io/2.0.0/kafka/security.html). Use the  `spring.cloud.stream.kafka.binder.configuration`  option to set security properties for all clients created by the binder.

For example, to set  `security.protocol`  to  `SASL_SSL` , set the following property:

```java
spring.cloud.stream.kafka.binder.configuration.security.protocol=SASL_SSL
```

All the other security properties can be set in a similar manner.

When using Kerberos, follow the instructions in the [reference documentation](https://kafka.apache.org/090/documentation.html#security_sasl_clientconfig) for creating and referencing the JAAS configuration.

Spring Cloud Stream supports passing JAAS configuration information to the application by using a JAAS configuration file and using Spring Boot properties.

##### Using JAAS Configuration Files

The JAAS and (optionally) krb5 file locations can be set for Spring Cloud Stream applications by using system properties. The following example shows how to launch a Spring Cloud Stream application with SASL and Kerberos by using a JAAS configuration file:

```java
java -Djava.security.auth.login.config=/path.to/kafka_client_jaas.conf -jar log.jar \
--spring.cloud.stream.kafka.binder.brokers=secure.server:9092 \
--spring.cloud.stream.bindings.input.destination=stream.ticktock \
--spring.cloud.stream.kafka.binder.configuration.security.protocol=SASL_PLAINTEXT
```

##### Using Spring Boot Properties

As an alternative to having a JAAS configuration file, Spring Cloud Stream provides a mechanism for setting up the JAAS configuration for Spring Cloud Stream applications by using Spring Boot properties.

The following properties can be used to configure the login context of the Kafka client:

spring.cloud.stream.kafka.binder.jaas.loginModule The login module name. Not necessary to be set in normal cases. Default: com.sun.security.auth.module.Krb5LoginModule. spring.cloud.stream.kafka.binder.jaas.controlFlag The control flag of the login module. Default: required. spring.cloud.stream.kafka.binder.jaas.options Map with a key/value pair containing the login module options. Default: Empty map.

The following example shows how to launch a Spring Cloud Stream application with SASL and Kerberos by using Spring Boot configuration properties:

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

The preceding example represents the equivalent of the following JAAS file:

```java
KafkaClient {
com.sun.security.auth.module.Krb5LoginModule required
useKeyTab=true
storeKey=true
keyTab="/etc/security/keytabs/kafka_client.keytab"
principal="[emailprotected]";
};
```

If the topics required already exist on the broker or will be created by an administrator, autocreation can be turned off and only client JAAS properties need to be sent.

> Do not mix JAAS configuration files and Spring Boot properties in the same application. If the  `-Djava.security.auth.login.config`  system property is already present, Spring Cloud Stream ignores the Spring Boot properties.

> Be careful when using the  `autoCreateTopics`  and  `autoAddPartitions`  with Kerberos. Usually, applications may use principals that do not have administrative rights in Kafka and Zookeeper. Consequently, relying on Spring Cloud Stream to create/modify topics may fail. In secure environments, we strongly recommend creating topics and managing ACLs administratively by using Kafka tooling.

#### Example: Pausing and Resuming the Consumer

If you wish to suspend consumption but not cause a partition rebalance, you can pause and resume the consumer. This is facilitated by adding the  `Consumer`  as a parameter to your  `@StreamListener` . To resume, you need an  `ApplicationListener`  for  `ListenerContainerIdleEvent`  instances. The frequency at which events are published is controlled by the  `idleEventInterval`  property. Since the consumer is not thread-safe, you must call these methods on the calling thread.

The following simple application shows how to pause and resume:

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

## 37.4 Error Channels

Starting with version 1.3, the binder unconditionally sends exceptions to an error channel for each consumer destination and can also be configured to send async producer send failures to an error channel. See [Section 27.4, “Error Handling”](multi__programming_model.html#spring-cloud-stream-overview-error-handling) for more information.

The payload of the  `ErrorMessage`  for a send failure is a  `KafkaSendFailureException`  with properties:

-  `failedMessage` : The Spring Messaging  `Message<?>`  that failed to be sent.

-  `record` : The raw  `ProducerRecord`  that was created from the  `failedMessage` 

There is no automatic handling of producer exceptions (such as sending to a [Dead-Letter queue](multi__apache_kafka_binder.html#kafka-dlq-processing)). You can consume these exceptions with your own Spring Integration flow.

## 37.5 Kafka Metrics

Kafka binder module exposes the following metrics:

`spring.cloud.stream.binder.kafka.someGroup.someTopic.lag` : This metric indicates how many messages have not been yet consumed from a given binder’s topic by a given consumer group. For example, if the value of the metric  `spring.cloud.stream.binder.kafka.myGroup.myTopic.lag`  is  `1000` , the consumer group named  `myGroup`  has  `1000`  messages waiting to be consumed from the topic calle  `myTopic` . This metric is particularly useful for providing auto-scaling feedback to a PaaS platform.

## 37.6 Dead-Letter Topic Processing

Because you cannot anticipate how users would want to dispose of dead-lettered messages, the framework does not provide any standard mechanism to handle them. If the reason for the dead-lettering is transient, you may wish to route the messages back to the original topic. However, if the problem is a permanent issue, that could cause an infinite loop. The sample Spring Boot application within this topic is an example of how to route those messages back to the original topic, but it moves them to a “parking lot” topic after three attempts. The application is another spring-cloud-stream application that reads from the dead-letter topic. It terminates when no messages are received for 5 seconds.

The examples assume the original destination is  `so8400out`  and the consumer group is  `so8400` .

There are a couple of strategies to consider:

- Consider running the rerouting only when the main application is not running. Otherwise, the retries for transient errors are used up very quickly.

- Alternatively, use a two-stage approach: Use this application to route to a third topic and another to route from there back to the main topic.

The following code listings show the sample application:

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

## 37.7 Partitioning with the Kafka Binder

Apache Kafka supports topic partitioning natively.

Sometimes it is advantageous to send data to specific partitions — for example, when you want to strictly order message processing (all messages for a particular customer should go to the same partition).

The following example shows how to configure the producer and consumer side:

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

|images/important.png|Important|
|----|----|
|The topic must be provisioned to have enough partitions to achieve the desired concurrency for all consumer groups. The above configuration supports up to 12 consumer instances (6 if their  `concurrency`  is 2, 4 if their concurrency is 3, and so on). It is generally best to “over-provision” the partitions to allow for future increases in consumers or concurrency. |

> The preceding configuration uses the default partitioning ( `key.hashCode() % partitionCount` ). This may or may not provide a suitably balanced algorithm, depending on the key values. You can override this default by using the  `partitionSelectorExpression`  or  `partitionSelectorClass`  properties.

Since partitions are natively handled by Kafka, no special configuration is needed on the consumer side. Kafka allocates partitions across the instances.

The following Spring Boot application listens to a Kafka stream and prints (to the console) the partition ID to which each message goes:

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

You can add instances as needed. Kafka rebalances the partition allocations. If the instance count (or  `instance count * concurrency` ) exceeds the number of partitions, some consumers are idle.

