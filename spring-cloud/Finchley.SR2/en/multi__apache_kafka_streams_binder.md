## 38. Apache Kafka Streams Binder

## 38.1 Usage

For using the Kafka Streams binder, you just need to add it to your Spring Cloud Stream application, using the following Maven coordinates:

```xml
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-stream-binder-kafka-streams</artifactId>
</dependency>
```

## 38.2 Kafka Streams Binder Overview

Spring Cloud Stream’s Apache Kafka support also includes a binder implementation designed explicitly for Apache Kafka Streams binding. With this native integration, a Spring Cloud Stream "processor" application can directly use the [Apache Kafka Streams](https://kafka.apache.org/documentation/streams/developer-guide) APIs in the core business logic.

Kafka Streams binder implementation builds on the foundation provided by the [Kafka Streams in Spring Kafka](https://docs.spring.io/spring-kafka/reference/html/_reference.html#kafka-streams) project.

As part of this native integration, the high-level [Streams DSL](https://docs.confluent.io/current/streams/developer-guide/dsl-api.html) provided by the Kafka Streams API is available for use in the business logic, too.

An early version of the [Processor API](https://docs.confluent.io/current/streams/developer-guide/processor-api.html) support is available as well.

As noted early-on, Kafka Streams support in Spring Cloud Stream strictly only available for use in the Processor model. A model in which the messages read from an inbound topic, business processing can be applied, and the transformed messages can be written to an outbound topic. It can also be used in Processor applications with a no-outbound destination.

### 38.2.1 Streams DSL

This application consumes data from a Kafka topic (e.g.,  `words` ), computes word count for each unique word in a 5 seconds time window, and the computed results are sent to a downstream topic (e.g.,  `counts` ) for further processing.

```java
@SpringBootApplication
@EnableBinding(KStreamProcessor.class)
public class WordCountProcessorApplication {

	@StreamListener("input")
	@SendTo("output")
	public KStream<?, WordCount> process(KStream<?, String> input) {
		return input
.flatMapValues(value -> Arrays.asList(value.toLowerCase().split("\\W+")))
.groupBy((key, value) -> value)
.windowedBy(TimeWindows.of(5000))
.count(Materialized.as("WordCounts-multi"))
.toStream()
.map((key, value) -> new KeyValue<>(null, new WordCount(key.key(), value, new Date(key.window().start()), new Date(key.window().end()))));
}

	public static void main(String[] args) {
		SpringApplication.run(WordCountProcessorApplication.class, args);
	}
```

Once built as a uber-jar (e.g.,  `wordcount-processor.jar` ), you can run the above example like the following.

```java
java -jar wordcount-processor.jar  --spring.cloud.stream.bindings.input.destination=words --spring.cloud.stream.bindings.output.destination=counts
```

This application will consume messages from the Kafka topic  `words`  and the computed results are published to an output topic  `counts` .

Spring Cloud Stream will ensure that the messages from both the incoming and outgoing topics are automatically bound as KStream objects. As a developer, you can exclusively focus on the business aspects of the code, i.e. writing the logic required in the processor. Setting up the Streams DSL specific configuration required by the Kafka Streams infrastructure is automatically handled by the framework.

## 38.3 Configuration Options

This section contains the configuration options used by the Kafka Streams binder.

For common configuration options and properties pertaining to binder, refer to the [core documentation](multi__configuration_options.html#binding-properties).

### 38.3.1 Kafka Streams Properties

The following properties are available at the binder level and must be prefixed with  `spring.cloud.stream.kafka.streams.binder.`  literal.

configuration Map with a key/value pair containing properties pertaining to Apache Kafka Streams API. This property must be prefixed with spring.cloud.stream.kafka.streams.binder.. Following are some examples of using this property.

```java
spring.cloud.stream.kafka.streams.binder.configuration.default.key.serde=org.apache.kafka.common.serialization.Serdes$StringSerde
spring.cloud.stream.kafka.streams.binder.configuration.default.value.serde=org.apache.kafka.common.serialization.Serdes$StringSerde
spring.cloud.stream.kafka.streams.binder.configuration.commit.interval.ms=1000
```

For more information about all the properties that may go into streams configuration, see StreamsConfig JavaDocs in Apache Kafka Streams docs.

brokers Broker URL Default: localhost zkNodes Zookeeper URL Default: localhost serdeError Deserialization error handler type. Possible values are - logAndContinue, logAndFail or sendToDlq Default: logAndFail applicationId Application ID for all the stream configurations in the current application context. You can override the application id for an individual StreamListener method using the group property on the binding. You have to ensure that you are using the same group name for all input bindings in the case of multiple inputs on the same methods. Default: default

The following properties are only available for Kafka Streams producers and must be prefixed with  `spring.cloud.stream.kafka.streams.bindings.<binding name>.producer.`  literal.

keySerde key serde to use Default: none. valueSerde value serde to use Default: none. useNativeEncoding flag to enable native encoding Default: false.

The following properties are only available for Kafka Streams consumers and must be prefixed with  `spring.cloud.stream.kafka.streams.bindings.<binding name>.consumer.`  literal.

keySerde key serde to use Default: none. valueSerde value serde to use Default: none. materializedAs state store to materialize when using incoming KTable types Default: none. useNativeDecoding flag to enable native decoding Default: false. dlqName DLQ topic name. Default: none.

### 38.3.2 TimeWindow properties:

Windowing is an important concept in stream processing applications. Following properties are available to configure time-window computations.

spring.cloud.stream.kafka.streams.timeWindow.length When this property is given, you can autowire a TimeWindows bean into the application. The value is expressed in milliseconds. Default: none. spring.cloud.stream.kafka.streams.timeWindow.advanceBy Value is given in milliseconds. Default: none.

## 38.4 Multiple Input Bindings

For use cases that requires multiple incoming KStream objects or a combination of KStream and KTable objects, the Kafka Streams binder provides multiple bindings support.

Let’s see it in action.

### 38.4.1 Multiple Input Bindings as a Sink

```java
@EnableBinding(KStreamKTableBinding.class)
.....
.....
@StreamListener
public void process(@Input("inputStream") KStream<String, PlayEvent> playEvents,
@Input("inputTable") KTable<Long, Song> songTable) {
....
....
}

interface KStreamKTableBinding {

@Input("inputStream")
KStream<?, ?> inputStream();

@Input("inputTable")
KTable<?, ?> inputTable();
}
```

In the above example, the application is written as a sink, i.e. there are no output bindings and the application has to decide concerning downstream processing. When you write applications in this style, you might want to send the information downstream or store them in a state store (See below for Queryable State Stores).

In the case of incoming KTable, if you want to materialize the computations to a state store, you have to express it through the following property.

```java
spring.cloud.stream.kafka.streams.bindings.inputTable.consumer.materializedAs: all-songs
```

### 38.4.2 Multiple Input Bindings as a Processor

```java
@EnableBinding(KStreamKTableBinding.class)
....
....

@StreamListener
@SendTo("output")
public KStream<String, Long> process(@Input("input") KStream<String, Long> userClicksStream,
@Input("inputTable") KTable<String, String> userRegionsTable) {
....
....
}

interface KStreamKTableBinding extends KafkaStreamsProcessor {

@Input("inputX")
KTable<?, ?> inputTable();
}
```

## 38.5 Multiple Output Bindings (aka Branching)

Kafka Streams allow outbound data to be split into multiple topics based on some predicates. The Kafka Streams binder provides support for this feature without compromising the programming model exposed through  `StreamListener`  in the end user application.

You can write the application in the usual way as demonstrated above in the word count example. However, when using the branching feature, you are required to do a few things. First, you need to make sure that your return type is  `KStream[]`  instead of a regular  `KStream` . Second, you need to use the  `SendTo`  annotation containing the output bindings in the order (see example below). For each of these output bindings, you need to configure destination, content-type etc., complying with the standard Spring Cloud Stream expectations.

Here is an example:

```java
@EnableBinding(KStreamProcessorWithBranches.class)
@EnableAutoConfiguration
public static class WordCountProcessorApplication {

@Autowired
private TimeWindows timeWindows;

@StreamListener("input")
@SendTo({"output1","output2","output3})
public KStream<?, WordCount>[] process(KStream<Object, String> input) {

			Predicate<Object, WordCount> isEnglish = (k, v) -> v.word.equals("english");
			Predicate<Object, WordCount> isFrench =  (k, v) -> v.word.equals("french");
			Predicate<Object, WordCount> isSpanish = (k, v) -> v.word.equals("spanish");

			return input
					.flatMapValues(value -> Arrays.asList(value.toLowerCase().split("\\W+")))
					.groupBy((key, value) -> value)
					.windowedBy(timeWindows)
					.count(Materialized.as("WordCounts-1"))
					.toStream()
					.map((key, value) -> new KeyValue<>(null, new WordCount(key.key(), value, new Date(key.window().start()), new Date(key.window().end()))))
					.branch(isEnglish, isFrench, isSpanish);
}

interface KStreamProcessorWithBranches {

		@Input("input")
		KStream<?, ?> input();

		@Output("output1")
		KStream<?, ?> output1();

		@Output("output2")
		KStream<?, ?> output2();

		@Output("output3")
		KStream<?, ?> output3();
	}
}
```

Properties:

```java
spring.cloud.stream.bindings.output1.contentType: application/json
spring.cloud.stream.bindings.output2.contentType: application/json
spring.cloud.stream.bindings.output3.contentType: application/json
spring.cloud.stream.kafka.streams.binder.configuration.commit.interval.ms: 1000
spring.cloud.stream.kafka.streams.binder.configuration:
default.key.serde: org.apache.kafka.common.serialization.Serdes$StringSerde
default.value.serde: org.apache.kafka.common.serialization.Serdes$StringSerde
spring.cloud.stream.bindings.output1:
destination: foo
producer:
headerMode: raw
spring.cloud.stream.bindings.output2:
destination: bar
producer:
headerMode: raw
spring.cloud.stream.bindings.output3:
destination: fox
producer:
headerMode: raw
spring.cloud.stream.bindings.input:
destination: words
consumer:
headerMode: raw
```

## 38.6 Message Conversion

Similar to message-channel based binder applications, the Kafka Streams binder adapts to the out-of-the-box content-type conversions without any compromise.

It is typical for Kafka Streams operations to know the type of SerDe’s used to transform the key and value correctly. Therefore, it may be more natural to rely on the SerDe facilities provided by the Apache Kafka Streams library itself at the inbound and outbound conversions rather than using the content-type conversions offered by the framework. On the other hand, you might be already familiar with the content-type conversion patterns provided by the framework, and that, you’d like to continue using for inbound and outbound conversions.

Both the options are supported in the Kafka Streams binder implementation.

### 38.6.1 Outbound serialization

If native encoding is disabled (which is the default), then the framework will convert the message using the contentType set by the user (otherwise, the default  `application/json`  will be applied). It will ignore any SerDe set on the outbound in this case for outbound serialization.

Here is the property to set the contentType on the outbound.

```java
spring.cloud.stream.bindings.output.contentType: application/json
```

Here is the property to enable native encoding.

```java
spring.cloud.stream.bindings.output.nativeEncoding: true
```

If native encoding is enabled on the output binding (user has to enable it as above explicitly), then the framework will skip any form of automatic message conversion on the outbound. In that case, it will switch to the Serde set by the user. The  `valueSerde`  property set on the actual output binding will be used. Here is an example.

```java
spring.cloud.stream.kafka.streams.bindings.output.producer.valueSerde: org.apache.kafka.common.serialization.Serdes$StringSerde
```

If this property is not set, then it will use the "default" SerDe:  `spring.cloud.stream.kafka.streams.binder.configuration.default.value.serde` .

It is worth to mention that Kafka Streams binder does not serialize the keys on outbound - it simply relies on Kafka itself. Therefore, you either have to specify the  `keySerde`  property on the binding or it will default to the application-wide common  `keySerde` .

Binding level key serde:

```java
spring.cloud.stream.kafka.streams.bindings.output.producer.keySerde
```

Common Key serde:

```java
spring.cloud.stream.kafka.streams.binder.configuration.default.key.serde
```

If branching is used, then you need to use multiple output bindings. For example,

```java
interface KStreamProcessorWithBranches {

		@Input("input")
		KStream<?, ?> input();

		@Output("output1")
		KStream<?, ?> output1();

		@Output("output2")
		KStream<?, ?> output2();

		@Output("output3")
		KStream<?, ?> output3();
	}
```

If  `nativeEncoding`  is set, then you can set different SerDe’s on individual output bindings as below.

```java
spring.cloud.stream.kafka.streams.bindings.output1.producer.valueSerde=IntegerSerde
spring.cloud.stream.kafka.streams.bindings.output2.producer.valueSerde=StringSerde
spring.cloud.stream.kafka.streams.bindings.output3.producer.valueSerde=JsonSerde
```

Then if you have  `SendTo`  like this, @SendTo({"output1", "output2", "output3"}), the  `KStream[]`  from the branches are applied with proper SerDe objects as defined above. If you are not enabling  `nativeEncoding` , you can then set different contentType values on the output bindings as below. In that case, the framework will use the appropriate message converter to convert the messages before sending to Kafka.

```java
spring.cloud.stream.bindings.output1.contentType: application/json
spring.cloud.stream.bindings.output2.contentType: application/java-serialzied-object
spring.cloud.stream.bindings.output3.contentType: application/octet-stream
```

### 38.6.2 Inbound Deserialization

Similar rules apply to data deserialization on the inbound.

If native decoding is disabled (which is the default), then the framework will convert the message using the contentType set by the user (otherwise, the default  `application/json`  will be applied). It will ignore any SerDe set on the inbound in this case for inbound deserialization.

Here is the property to set the contentType on the inbound.

```java
spring.cloud.stream.bindings.input.contentType: application/json
```

Here is the property to enable native decoding.

```java
spring.cloud.stream.bindings.input.nativeDecoding: true
```

If native decoding is enabled on the input binding (user has to enable it as above explicitly), then the framework will skip doing any message conversion on the inbound. In that case, it will switch to the SerDe set by the user. The  `valueSerde`  property set on the actual output binding will be used. Here is an example.

```java
spring.cloud.stream.kafka.streams.bindings.input.consumer.valueSerde: org.apache.kafka.common.serialization.Serdes$StringSerde
```

If this property is not set, it will use the default SerDe:  `spring.cloud.stream.kafka.streams.binder.configuration.default.value.serde` .

It is worth to mention that Kafka Streams binder does not deserialize the keys on inbound - it simply relies on Kafka itself. Therefore, you either have to specify the  `keySerde`  property on the binding or it will default to the application-wide common  `keySerde` .

Binding level key serde:

```java
spring.cloud.stream.kafka.streams.bindings.input.consumer.keySerde
```

Common Key serde:

```java
spring.cloud.stream.kafka.streams.binder.configuration.default.key.serde
```

As in the case of KStream branching on the outbound, the benefit of setting value SerDe per binding is that if you have multiple input bindings (multiple KStreams object) and they all require separate value SerDe’s, then you can configure them individually. If you use the common configuration approach, then this feature won’t be applicable.

## 38.7 Error Handling

Apache Kafka Streams provide the capability for natively handling exceptions from deserialization errors. For details on this support, please see [this](https://cwiki.apache.org/confluence/display/KAFKA/KIP-161%3A+streams+deserialization+exception+handlers) Out of the box, Apache Kafka Streams provide two kinds of deserialization exception handlers -  `logAndContinue`  and  `logAndFail` . As the name indicates, the former will log the error and continue processing the next records and the latter will log the error and fail.  `LogAndFail`  is the default deserialization exception handler.

### 38.7.1 Handling Deserialization Exceptions

Kafka Streams binder supports a selection of exception handlers through the following properties.

```java
spring.cloud.stream.kafka.streams.binder.serdeError: logAndContinue
```

In addition to the above two deserialization exception handlers, the binder also provides a third one for sending the erroneous records (poison pills) to a DLQ topic. Here is how you enable this DLQ exception handler.

```java
spring.cloud.stream.kafka.streams.binder.serdeError: sendToDlq
```

When the above property is set, all the deserialization error records are automatically sent to the DLQ topic.

```java
spring.cloud.stream.kafka.streams.bindings.input.consumer.dlqName: foo-dlq
```

If this is set, then the error records are sent to the topic  `foo-dlq` . If this is not set, then it will create a DLQ topic with the name  `error.<input-topic-name>.<group-name>` .

A couple of things to keep in mind when using the exception handling feature in Kafka Streams binder.

- The property  `spring.cloud.stream.kafka.streams.binder.serdeError`  is applicable for the entire application. This implies that if there are multiple  `StreamListener`  methods in the same application, this property is applied to all of them.

- The exception handling for deserialization works consistently with native deserialization and framework provided message conversion.

### 38.7.2 Handling Non-Deserialization Exceptions

For general error handling in Kafka Streams binder, it is up to the end user applications to handle application level errors. As a side effect of providing a DLQ for deserialization exception handlers, Kafka Streams binder provides a way to get access to the DLQ sending bean directly from your application. Once you get access to that bean, you can programmatically send any exception records from your application to the DLQ.

It continues to remain hard to robust error handling using the high-level DSL; Kafka Streams doesn’t natively support error handling yet.

However, when you use the low-level Processor API in your application, there are options to control this behavior. See below.

```java
@Autowired
private SendToDlqAndContinue dlqHandler;

@StreamListener("input")
@SendTo("output")
public KStream<?, WordCount> process(KStream<Object, String> input) {

input.process(() -> new Processor() {
			ProcessorContext context;

			@Override
			public void init(ProcessorContext context) {
				this.context = context;
			}

			@Override
			public void process(Object o, Object o2) {

			    try {
			        .....
			        .....
			    }
			    catch(Exception e) {
			        //explicitly provide the kafka topic corresponding to the input binding as the first argument.
//DLQ handler will correctly map to the dlq topic from the actual incoming destination.
dlqHandler.sendToDlq("topic-name", (byte[]) o1, (byte[]) o2, context.partition());
			    }
			}

			.....
			.....
});
}
```

## 38.8 Interactive Queries

As part of the public Kafka Streams binder API, we expose a class called  `QueryableStoreRegistry` . You can access this as a Spring bean in your application. An easy way to get access to this bean from your application is to "autowire" the bean in your application.

```java
@Autowired
private QueryableStoreRegistry queryableStoreRegistry;
```

Once you gain access to this bean, then you can query for the particular state-store that you are interested. See below.

```java
ReadOnlyKeyValueStore<Object, Object> keyValueStore =
						queryableStoreRegistry.getQueryableStoreType("my-store", QueryableStoreTypes.keyValueStore());
```

## 38.9 Accessing the underlying KafkaStreams object

`StreamBuilderFactoryBean`  from spring-kafka that is responsible for constructing the  `KafkaStreams`  object can be accessed programmatically. Each  `StreamBuilderFactoryBean`  is registered as  `stream-builder`  and appended with the  `StreamListener`  method name. If your  `StreamListener`  method is named as  `process`  for example, the stream builder bean is named as  `stream-builder-process` . Since this is a factory bean, it should be accessed by prepending an ampersand ( `&` ) when accessing it programmatically. Following is an example and it assumes the  `StreamListener`  method is named as  `process` 

```java
StreamsBuilderFactoryBean streamsBuilderFactoryBean = context.getBean("&stream-builder-process", StreamsBuilderFactoryBean.class);
			KafkaStreams kafkaStreams = streamsBuilderFactoryBean.getKafkaStreams();
```

