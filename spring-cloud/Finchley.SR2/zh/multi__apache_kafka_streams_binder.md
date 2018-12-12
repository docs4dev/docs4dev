## 38. Apache Kafka Streams Binder

## 38.1用法

要使用Kafka StreamsBinders，只需使用以下Maven坐标将其添加到Spring Cloud Stream应用程序：

```xml
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-stream-binder-kafka-streams</artifactId>
</dependency>
```

## 38.2 Kafka Streams Binder概述

Spring Cloud Stream的Apache Kafka支持还包括为Apache Kafka Streams绑定明确设计的Binders实现.通过这种本机集成，Spring Cloud Stream "processor"应用程序可以直接在核心业务逻辑中使用[Apache Kafka Streams](https://kafka.apache.org/documentation/streams/developer-guide) API.

Kafka StreamsBinders实现Build在[Kafka Streams in Spring Kafka](https://docs.spring.io/spring-kafka/reference/html/_reference.html#kafka-streams)项目提供的基础之上.

作为此本机集成的一部分，Kafka Streams API提供的高级[Streams DSL](https://docs.confluent.io/current/streams/developer-guide/dsl-api.html)也可用于业务逻辑.

还提供了早期版本的[Processor API](https://docs.confluent.io/current/streams/developer-guide/processor-api.html)支持.

如前所述，Kafka Streams在Spring Cloud Stream中的支持严格仅适用于处理器模型.可以应用从入站主题读取的消息，业务处理以及转换后的消息可以写入出站主题的模型.它也可以在没有出站目的地的处理器应用程序中使用.

### 38.2.1 Streams DSL

该应用程序使用来自Kafka主题的数据（例如， `words` ），在5秒时间窗口中计算每个唯一字的字数，并且将计算结果发送到下游主题（例如， `counts` ）以进行进一步处理.

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

一旦构建为超级jar（例如， `wordcount-processor.jar` ），您就可以运行上面的示例，如下所示.

```java
java -jar wordcount-processor.jar  --spring.cloud.stream.bindings.input.destination=words --spring.cloud.stream.bindings.output.destination=counts
```

此应用程序将使用来自Kafka主题 `words` 的消息，并将计算结果发布到输出主题 `counts` .

Spring Cloud Stream将确保来自传入和传出主题的消息自动绑定为KStream对象.作为开发人员，您可以专注于代码的业务方面，即编写处理器中所需的逻辑.设置Kafka Streams基础结构所需的Streams DSL特定配置由框架自动处理.

## 38.3配置选项

本节包含Kafka StreamsBinders使用的配置选项.

有关常用配置选项和与Binders有关的属性，请参阅[core documentation](multi__configuration_options.html#binding-properties).

### 38.3.1 Kafka Streams Properties

在Binders级别可以使用以下属性，并且必须以 `spring.cloud.stream.kafka.streams.binder.`  literal为前缀.

配置包含键/值对的映射与Apache Kafka Streams API相关的属性.此属性必须以spring.cloud.stream.kafka.streams.binder为前缀.以下是使用此属性的一些示例.

```java
spring.cloud.stream.kafka.streams.binder.configuration.default.key.serde=org.apache.kafka.common.serialization.Serdes$StringSerde
spring.cloud.stream.kafka.streams.binder.configuration.default.value.serde=org.apache.kafka.common.serialization.Serdes$StringSerde
spring.cloud.stream.kafka.streams.binder.configuration.commit.interval.ms=1000
```

有关可能进入流配置的所有属性的更多信息，请参阅Apache Kafka Streams文档中的StreamsConfig JavaDocs.

代理Broker URL默认值：localhost zkNodes Zookeeper URL默认值：localhost serdeError反序列化错误处理程序类型.可能的值为 -  logAndContinue，logAndFail或sendToDlq默认值：logAndFail applicationId当前应用程序上下文中所有流配置的应用程序ID.您可以使用绑定上的group属性覆盖单个StreamListener方法的应用程序ID.在相同方法的多个输入的情况下，您必须确保为所有输入绑定使用相同的组名.默认值：默认值

以下属性仅适用于Kafka Streams生成器，并且必须以 `spring.cloud.stream.kafka.streams.bindings.<binding name>.producer.`  literal为前缀.

keySerde键serde使用默认值：none. valueSerde值serde使用默认值：none. useNativeEncoding标志启用本机编码默认值：false.

以下属性仅适用于Kafka Streams使用者，并且必须以 `spring.cloud.stream.kafka.streams.bindings.<binding name>.consumer.`  literal为前缀.

keySerde键serde使用默认值：none. valueSerde值serde使用默认值：none. materializedAs state store在使用传入的KTable类型时实现Default：none. useNativeDecoding标志启用本机解码默认值：false. dlqName DLQ主题名称.默认值：无.

### 38.3.2 TimeWindow属性：

窗口化是流处理应用程序中的一个重要概念.以下属性可用于配置时间窗口计算.

spring.cloud.stream.kafka.streams.timeWindow.length当给出此属性时，您可以将TimeWindows bean自动装配到应用程序中.该值以毫秒表示.默认值：无. spring.cloud.stream.kafka.streams.timeWindow.advanceBy以毫秒为单位给出值.默认值：无.

## 38.4多输入绑定

对于需要多个传入KStream对象或KStream和KTable对象组合的用例，Kafka StreamsBinders提供多个绑定支持.

让我们看看它的实际效果.

### 38.4.1多个输入绑定作为接收器

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

在上面的例子中，应用程序被写为接收器，即没有输出绑定，应用程序必须决定下游处理.以此样式编写应用程序时，您可能希望将信息发送到下游或将其存储在状态存储中（请参阅下面的可查询状态存储）.

在传入KTable的情况下，如果要将计算实现到状态存储，则必须通过以下属性表达它.

```java
spring.cloud.stream.kafka.streams.bindings.inputTable.consumer.materializedAs: all-songs
```

### 38.4.2多个输入绑定作为处理器

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

## 38.5多输出绑定（又称分支）

Kafka Streams允许基于某些谓词将出站数据拆分为多个主题. Kafka StreamsBinders为此功能提供支持，而不会影响最终用户应用程序中通过 `StreamListener` 公开的编程模型.

您可以按照常规方式编写应用程序，如上面单词计数示例中所示.但是，在使用分支功能时，您需要执行一些操作.首先，您需要确保返回类型是 `KStream[]` 而不是常规 `KStream` .其次，您需要使用包含订单中输出绑定的 `SendTo` 注释（请参阅下面的示例）.对于每个输出绑定，您需要配置目标，内容类型等，符合标准的Spring Cloud Stream期望.

这是一个例子：

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

属性：

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

## 38.6消息转换

与基于消息通道的Binders应用程序类似，Kafka StreamsBinders可以适应开箱即用的内容类型转换，而不会有任何妥协.

Kafka Streams操作通常会知道用于正确转换键和值的SerDe的类型.因此，依赖于Apache Kafka Streams库本身在入站和出站转换中提供的SerDe工具而不是使用框架提供的内容类型转换可能更为自然.另一方面，您可能已经熟悉框架提供的内容类型转换模式，并且您希望继续使用入站和出站转换.

Kafka StreamsBinders实现中都支持这两个选项.

### 38.6.1出站序列化

如果禁用本机编码（这是默认设置），则框架将使用用户设置的contentType转换消息（否则，将应用默认的 `application/json` ）.在这种情况下，它将忽略出站序列化的出站上的任何SerDe设置.

以下是在出站上设置contentType的属性.

```java
spring.cloud.stream.bindings.output.contentType: application/json
```

这是属性启用本机编码.

```java
spring.cloud.stream.bindings.output.nativeEncoding: true
```

如果在输出绑定上启用了本机编码（用户必须如上所述显式启用它），那么框架将跳过出站的任何形式的自动消息转换.在这种情况下，它将切换到用户设置的Serde.将使用在实际输出绑定上设置的 `valueSerde` 属性.这是一个例子.

```java
spring.cloud.stream.kafka.streams.bindings.output.producer.valueSerde: org.apache.kafka.common.serialization.Serdes$StringSerde
```

如果未设置此属性，则它将使用"default" SerDe： `spring.cloud.stream.kafka.streams.binder.configuration.default.value.serde` .

值得一提的是，Kafka Streams binder不会在出站时序列化密钥 - 它只依赖于Kafka本身.因此，您必须在绑定上指定 `keySerde` 属性，否则它将默认为应用程序范围的公共 `keySerde` .

绑定级别密钥serde：

```java
spring.cloud.stream.kafka.streams.bindings.output.producer.keySerde
```

Common Key serde：

```java
spring.cloud.stream.kafka.streams.binder.configuration.default.key.serde
```

如果使用分支，则需要使用多个输出绑定.例如，

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

如果设置了 `nativeEncoding` ，则可以在单个输出绑定上设置不同的SerDe，如下所示.

```java
spring.cloud.stream.kafka.streams.bindings.output1.producer.valueSerde=IntegerSerde
spring.cloud.stream.kafka.streams.bindings.output2.producer.valueSerde=StringSerde
spring.cloud.stream.kafka.streams.bindings.output3.producer.valueSerde=JsonSerde
```

然后如果你有这样的 `SendTo` ，@ SendTo（{"output1"，"output2"，"output3"}），分支中的 `KStream[]` 应用了如上定义的适当的SerDe对象.如果未启用 `nativeEncoding` ，则可以在输出绑定上设置不同的contentType值，如下所示.在这种情况下，框架将使用适当的消息转换器在发送到Kafka之前转换消息.

```java
spring.cloud.stream.bindings.output1.contentType: application/json
spring.cloud.stream.bindings.output2.contentType: application/java-serialzied-object
spring.cloud.stream.bindings.output3.contentType: application/octet-stream
```

### 38.6.2入站反序列化

类似的规则适用于入站数据反序列化.

如果禁用本机解码（这是默认设置），则框架将使用用户设置的contentType转换消息（否则，将应用默认的 `application/json` ）.在这种情况下，它将忽略入站反序列化的入站上的任何SerDe集.

以下是在入站中设置contentType的属性.

```java
spring.cloud.stream.bindings.input.contentType: application/json
```

以下是启用本机解码的属性.

```java
spring.cloud.stream.bindings.input.nativeDecoding: true
```

如果在输入绑定上启用了本机解码（用户必须如上所述明确启用它），那么框架将跳过对入站进行任何消息转换.在这种情况下，它将切换到用户设置的SerDe.将使用在实际输出绑定上设置的 `valueSerde` 属性.这是一个例子.

```java
spring.cloud.stream.kafka.streams.bindings.input.consumer.valueSerde: org.apache.kafka.common.serialization.Serdes$StringSerde
```

如果未设置此属性，则将使用默认的SerDe： `spring.cloud.stream.kafka.streams.binder.configuration.default.value.serde` .

值得一提的是，Kafka StreamsBinders不会对入站密钥进行反序列化 - 它只依赖于Kafka本身.因此，您必须在绑定上指定 `keySerde` 属性，否则它将默认为应用程序范围的公共 `keySerde` .

绑定级别密钥serde：

```java
spring.cloud.stream.kafka.streams.bindings.input.consumer.keySerde
```

Common Key serde：

```java
spring.cloud.stream.kafka.streams.binder.configuration.default.key.serde
```

与出站时KStream分支的情况一样，每个绑定设置值SerDe的好处是，如果您有多个输入绑定（多个KStreams对象）并且它们都需要单独的值SerDe，那么您可以单独配置它们.如果使用通用配置方法，则此功能将不适用.

## 38.7错误处理

Apache Kafka Streams提供了本机处理反序列化错误的异常的功能.有关此支持的详细信息，请参阅[this](https://cwiki.apache.org/confluence/display/KAFKA/KIP-161%3A+streams+deserialization+exception+handlers)开箱即用，Apache Kafka Streams提供两种反序列化异常处理程序 -   `logAndContinue` 和 `logAndFail` .如名称所示，前者将记录错误并继续处理下一条记录，后者将记录错误并失败.  `LogAndFail` 是默认的反序列化异常处理程序.

### 38.7.1处理反序列化例外

Kafka Streams binder通过以下属性支持一系列异常处理程序.

```java
spring.cloud.stream.kafka.streams.binder.serdeError: logAndContinue
```

除了上述两个反序列化异常处理程序之外，绑定程序还提供了第三个用于将错误记录（毒丸）发送到DLQ主题的第三个.以下是启用此DLQ异常处理程序的方法.

```java
spring.cloud.stream.kafka.streams.binder.serdeError: sendToDlq
```

设置上述属性后，所有反序列化错误记录将自动发送到DLQ主题.

```java
spring.cloud.stream.kafka.streams.bindings.input.consumer.dlqName: foo-dlq
```

如果设置了此项，则会将错误记录发送到主题 `foo-dlq` .如果未设置，则会创建名为 `error.<input-topic-name>.<group-name>` 的DLQ主题.

在Kafka StreamsBinders中使用异常处理功能时要记住几件事.

- 属性 `spring.cloud.stream.kafka.streams.binder.serdeError` 适用于整个应用程序.这意味着如果同一个应用程序中有多个 `StreamListener` 方法，则此属性将应用于所有这些方法.

- 反序列化的异常处理与本机反序列化和框架提供的消息转换一致.

### 38.7.2处理非反序列化例外

对于Kafka Streams binder中的一般错误处理，最终用户应用程序可以处理应用程序级错误.作为为反序列化异常处理程序提供DLQ的副作用，Kafka StreamsBinders提供了一种直接从应用程序访问DLQ发送bean的方法.一旦访问该bean，就可以以编程方式将任何异常记录从应用程序发送到DLQ.

它仍然存在使用高级DSL难以进行强大的错误处理; Kafka Streams本身并不支持错误处理.

但是，在应用程序中使用低级Processor API时，可以选择控制此行为.见下文.

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

## 38.8交互式查询

作为公共Kafka Streams binder API的一部分，我们公开了一个名为 `QueryableStoreRegistry` 的类.您可以在应用程序中将其作为Spring bean进行访问.从应用程序访问此bean的一种简单方法是在应用程序中使用"autowire" bean.

```java
@Autowired
private QueryableStoreRegistry queryableStoreRegistry;
```

一旦获得对此bean的访问权限，就可以查询您感兴趣的特定状态存储.见下文.

```java
ReadOnlyKeyValueStore<Object, Object> keyValueStore =
						queryableStoreRegistry.getQueryableStoreType("my-store", QueryableStoreTypes.keyValueStore());
```

## 38.9访问基础KafkaStreams对象

来自spring-kafka的负责构造 `KafkaStreams` 对象的 `StreamBuilderFactoryBean` 可以通过编程方式访问.每个 `StreamBuilderFactoryBean` 都注册为 `stream-builder` 并附加 `StreamListener` 方法名称.例如，如果 `StreamListener` 方法的名称为 `process` ，则流构建器bean的名称为 `stream-builder-process` .由于这是一个工厂bean，因此在以编程方式访问它时，应通过添加＆符号（ `&` ）来访问它.以下是一个示例，它假设 `StreamListener` 方法被命名为 `process` 

```java
StreamsBuilderFactoryBean streamsBuilderFactoryBean = context.getBean("&stream-builder-process", StreamsBuilderFactoryBean.class);
			KafkaStreams kafkaStreams = streamsBuilderFactoryBean.getKafkaStreams();
```

