## 29. Configuration Options

Spring Cloud Stream supports general configuration options as well as configuration for bindings and binders. Some binders let additional binding properties support middleware-specific features.

Configuration options can be provided to Spring Cloud Stream applications through any mechanism supported by Spring Boot. This includes application arguments, environment variables, and YAML or .properties files.

## 29.1 Binding Service Properties

These properties are exposed via  `org.springframework.cloud.stream.config.BindingServiceProperties` 

spring.cloud.stream.instanceCount The number of deployed instances of an application. Must be set for partitioning on the producer side. Must be set on the consumer side when using RabbitMQ and with Kafka if autoRebalanceEnabled=false. Default: 1. spring.cloud.stream.instanceIndex The instance index of the application: A number from 0 to instanceCount - 1. Used for partitioning with RabbitMQ and with Kafka if autoRebalanceEnabled=false. Automatically set in Cloud Foundry to match the application’s instance index. spring.cloud.stream.dynamicDestinations A list of destinations that can be bound dynamically (for example, in a dynamic routing scenario). If set, only listed destinations can be bound. Default: empty (letting any destination be bound). spring.cloud.stream.defaultBinder The default binder to use, if multiple binders are configured. See Multiple Binders on the Classpath. Default: empty. spring.cloud.stream.overrideCloudConnectors This property is only applicable when the cloud profile is active and Spring Cloud Connectors are provided with the application. If the property is false (the default), the binder detects a suitable bound service (for example, a RabbitMQ service bound in Cloud Foundry for the RabbitMQ binder) and uses it for creating connections (usually through Spring Cloud Connectors). When set to true, this property instructs binders to completely ignore the bound services and rely on Spring Boot properties (for example, relying on the spring.rabbitmq.* properties provided in the environment for the RabbitMQ binder). The typical usage of this property is to be nested in a customized environment when connecting to multiple systems. Default: false. spring.cloud.stream.bindingRetryInterval The interval (in seconds) between retrying binding creation when, for example, the binder does not support late binding and the broker (for example, Apache Kafka) is down. Set it to zero to treat such conditions as fatal, preventing the application from starting. Default: 30

## 29.2 Binding Properties

Binding properties are supplied by using the format of  `spring.cloud.stream.bindings.<channelName>.<property>=<value>` . The  `<channelName>`  represents the name of the channel being configured (for example,  `output`  for a  `Source` ).

To avoid repetition, Spring Cloud Stream supports setting values for all channels, in the format of  `spring.cloud.stream.default.<property>=<value>` .

In what follows, we indicate where we have omitted the  `spring.cloud.stream.bindings.<channelName>.`  prefix and focus just on the property name, with the understanding that the prefix ise included at runtime.

### 29.2.1 Common Binding Properties

These properties are exposed via  `org.springframework.cloud.stream.config.BindingProperties` 

The following binding properties are available for both input and output bindings and must be prefixed with  `spring.cloud.stream.bindings.<channelName>.`  (for example,  `spring.cloud.stream.bindings.input.destination=ticktock` ).

Default values can be set by using the  `spring.cloud.stream.default`  prefix (for example`spring.cloud.stream.default.contentType=application/json`).

destination The target destination of a channel on the bound middleware (for example, the RabbitMQ exchange or Kafka topic). If the channel is bound as a consumer, it could be bound to multiple destinations, and the destination names can be specified as comma-separated String values. If not set, the channel name is used instead. The default value of this property cannot be overridden. group The consumer group of the channel. Applies only to inbound bindings. See Consumer Groups. Default: null (indicating an anonymous consumer). contentType The content type of the channel. See “Chapter 30, Content Type Negotiation”. Default: null (no type coercion is performed). binder The binder used by this binding. See “Section 28.4, “Multiple Binders on the Classpath”” for details. Default: null (the default binder is used, if it exists).

### 29.2.2 Consumer Properties

These properties are exposed via  `org.springframework.cloud.stream.binder.ConsumerProperties` 

The following binding properties are available for input bindings only and must be prefixed with  `spring.cloud.stream.bindings.<channelName>.consumer.`  (for example,  `spring.cloud.stream.bindings.input.consumer.concurrency=3` ).

Default values can be set by using the  `spring.cloud.stream.default.consumer`  prefix (for example,  `spring.cloud.stream.default.consumer.headerMode=none` ).

concurrency The concurrency of the inbound consumer. Default: 1. partitioned Whether the consumer receives data from a partitioned producer. Default: false. headerMode When set to none, disables header parsing on input. Effective only for messaging middleware that does not support message headers natively and requires header embedding. This option is useful when consuming data from non-Spring Cloud Stream applications when native headers are not supported. When set to headers, it uses the middleware’s native header mechanism. When set to embeddedHeaders, it embeds headers into the message payload. Default: depends on the binder implementation. maxAttempts If processing fails, the number of attempts to process the message (including the first). Set to 1 to disable retry. Default: 3. backOffInitialInterval The backoff initial interval on retry. Default: 1000. backOffMaxInterval The maximum backoff interval. Default: 10000. backOffMultiplier The backoff multiplier. Default: 2.0. instanceIndex When set to a value greater than equal to zero, it allows customizing the instance index of this consumer (if different from spring.cloud.stream.instanceIndex). When set to a negative value, it defaults to spring.cloud.stream.instanceIndex. See “Section 32.2, “Instance Index and Instance Count”” for more information. Default: -1. instanceCount When set to a value greater than equal to zero, it allows customizing the instance count of this consumer (if different from spring.cloud.stream.instanceCount). When set to a negative value, it defaults to spring.cloud.stream.instanceCount. See “Section 32.2, “Instance Index and Instance Count”” for more information. Default: -1. useNativeDecoding When set to true, the inbound message is deserialized directly by the client library, which must be configured correspondingly (for example, setting an appropriate Kafka producer value deserializer). When this configuration is being used, the inbound message unmarshalling is not based on the contentType of the binding. When native decoding is used, it is the responsibility of the producer to use an appropriate encoder (for example, the Kafka producer value serializer) to serialize the outbound message. Also, when native encoding and decoding is used, the headerMode=embeddedHeaders property is ignored and headers are not embedded in the message. See the producer property useNativeEncoding. Default: false.

### 29.2.3 Producer Properties

These properties are exposed via  `org.springframework.cloud.stream.binder.ProducerProperties` 

The following binding properties are available for output bindings only and must be prefixed with  `spring.cloud.stream.bindings.<channelName>.producer.`  (for example,  `spring.cloud.stream.bindings.input.producer.partitionKeyExpression=payload.id` ).

Default values can be set by using the prefix  `spring.cloud.stream.default.producer`  (for example,  `spring.cloud.stream.default.producer.partitionKeyExpression=payload.id` ).

partitionKeyExpression A SpEL expression that determines how to partition outbound data. If set, or if partitionKeyExtractorClass is set, outbound data on this channel is partitioned. partitionCount must be set to a value greater than 1 to be effective. Mutually exclusive with partitionKeyExtractorClass. See “Section 26.6, “Partitioning Support””. Default: null. partitionKeyExtractorClass A PartitionKeyExtractorStrategy implementation. If set, or if partitionKeyExpression is set, outbound data on this channel is partitioned. partitionCount must be set to a value greater than 1 to be effective. Mutually exclusive with partitionKeyExpression. See “Section 26.6, “Partitioning Support””. Default: null. partitionSelectorClass A PartitionSelectorStrategy implementation. Mutually exclusive with partitionSelectorExpression. If neither is set, the partition is selected as the hashCode(key) % partitionCount, where key is computed through either partitionKeyExpression or partitionKeyExtractorClass. Default: null. partitionSelectorExpression A SpEL expression for customizing partition selection. Mutually exclusive with partitionSelectorClass. If neither is set, the partition is selected as the hashCode(key) % partitionCount, where key is computed through either partitionKeyExpression or partitionKeyExtractorClass. Default: null. partitionCount The number of target partitions for the data, if partitioning is enabled. Must be set to a value greater than 1 if the producer is partitioned. On Kafka, it is interpreted as a hint. The larger of this and the partition count of the target topic is used instead. Default: 1. requiredGroups A comma-separated list of groups to which the producer must ensure message delivery even if they start after it has been created (for example, by pre-creating durable queues in RabbitMQ). headerMode When set to none, it disables header embedding on output. It is effective only for messaging middleware that does not support message headers natively and requires header embedding. This option is useful when producing data for non-Spring Cloud Stream applications when native headers are not supported. When set to headers, it uses the middleware’s native header mechanism. When set to embeddedHeaders, it embeds headers into the message payload. Default: Depends on the binder implementation. useNativeEncoding When set to true, the outbound message is serialized directly by the client library, which must be configured correspondingly (for example, setting an appropriate Kafka producer value serializer). When this configuration is being used, the outbound message marshalling is not based on the contentType of the binding. When native encoding is used, it is the responsibility of the consumer to use an appropriate decoder (for example, the Kafka consumer value de-serializer) to deserialize the inbound message. Also, when native encoding and decoding is used, the headerMode=embeddedHeaders property is ignored and headers are not embedded in the message. See the consumer property useNativeDecoding. Default: false. errorChannelEnabled When set to true, if the binder supports asynchroous send results, send failures are sent to an error channel for the destination. See “???” for more information. Default: false.

## 29.3 Using Dynamically Bound Destinations

Besides the channels defined by using  `@EnableBinding` , Spring Cloud Stream lets applications send messages to dynamically bound destinations. This is useful, for example, when the target destination needs to be determined at runtime. Applications can do so by using the  `BinderAwareChannelResolver`  bean, registered automatically by the  `@EnableBinding`  annotation.

The 'spring.cloud.stream.dynamicDestinations' property can be used for restricting the dynamic destination names to a known set (whitelisting). If this property is not set, any destination can be bound dynamically.

The  `BinderAwareChannelResolver`  can be used directly, as shown in the following example of a REST controller using a path variable to decide the target channel:

```java
@EnableBinding
@Controller
public class SourceWithDynamicDestination {

@Autowired
private BinderAwareChannelResolver resolver;

@RequestMapping(path = "/{target}", method = POST, consumes = "*/*")
@ResponseStatus(HttpStatus.ACCEPTED)
public void handleRequest(@RequestBody String body, @PathVariable("target") target,
@RequestHeader(HttpHeaders.CONTENT_TYPE) Object contentType) {
sendMessage(body, target, contentType);
}

private void sendMessage(String body, String target, Object contentType) {
resolver.resolveDestination(target).send(MessageBuilder.createMessage(body,
new MessageHeaders(Collections.singletonMap(MessageHeaders.CONTENT_TYPE, contentType))));
}
}
```

Now consider what happens when we start the application on the default port (8080) and make the following requests with CURL:

```java
curl -H "Content-Type: application/json" -X POST -d "customer-1" http://localhost:8080/customers

curl -H "Content-Type: application/json" -X POST -d "order-1" http://localhost:8080/orders
```

The destinations, 'customers' and 'orders', are created in the broker (in the exchange for Rabbit or in the topic for Kafka) with names of 'customers' and 'orders', and the data is published to the appropriate destinations.

The  `BinderAwareChannelResolver`  is a general-purpose Spring Integration  `DestinationResolver`  and can be injected in other components — for example, in a router using a SpEL expression based on the  `target`  field of an incoming JSON message. The following example includes a router that reads SpEL expressions:

```java
@EnableBinding
@Controller
public class SourceWithDynamicDestination {

@Autowired
private BinderAwareChannelResolver resolver;

@RequestMapping(path = "/", method = POST, consumes = "application/json")
@ResponseStatus(HttpStatus.ACCEPTED)
public void handleRequest(@RequestBody String body, @RequestHeader(HttpHeaders.CONTENT_TYPE) Object contentType) {
sendMessage(body, contentType);
}

private void sendMessage(Object body, Object contentType) {
routerChannel().send(MessageBuilder.createMessage(body,
new MessageHeaders(Collections.singletonMap(MessageHeaders.CONTENT_TYPE, contentType))));
}

@Bean(name = "routerChannel")
public MessageChannel routerChannel() {
return new DirectChannel();
}

@Bean
@ServiceActivator(inputChannel = "routerChannel")
public ExpressionEvaluatingRouter router() {
ExpressionEvaluatingRouter router =
new ExpressionEvaluatingRouter(new SpelExpressionParser().parseExpression("payload.target"));
router.setDefaultOutputChannelName("default-output");
router.setChannelResolver(resolver);
return router;
}
}
```

The [Router Sink Application](https://github.com/spring-cloud-stream-app-starters/router) uses this technique to create the destinations on-demand.

If the channel names are known in advance, you can configure the producer properties as with any other destination. Alternatively, if you register a  `NewBindingCallback<>`  bean, it is invoked just before the binding is created. The callback takes the generic type of the extended producer properties used by the binder. It has one method:

```java
void configure(String channelName, MessageChannel channel, ProducerProperties producerProperties,
T extendedProducerProperties);
```

The following example shows how to use the RabbitMQ binder:

```java
@Bean
public NewBindingCallback<RabbitProducerProperties> dynamicConfigurer() {
return (name, channel, props, extended) -> {
props.setRequiredGroups("bindThisQueue");
extended.setQueueNameGroupOnly(true);
extended.setAutoBindDlq(true);
extended.setDeadLetterQueueName("myDLQ");
};
}
```

> If you need to support dynamic destinations with multiple binder types, use  `Object`  for the generic type and cast the  `extended`  argument as needed.

