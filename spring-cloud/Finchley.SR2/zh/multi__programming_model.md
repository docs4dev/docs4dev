## 27.编程模型

要了解编程模型，您应该熟悉以下核心概念：

-  **Destination Binders:** 负责提供与外部邮件系统集成的组件.

-  **Destination Bindings:** 外部消息传递系统和应用程序之间的桥接提供消息的生产环境者和消费者（由目标Binders创建）.

-  **Message:** 生产环境者和消费者用于与目标Binders（以及通过外部消息系统的其他应用程序）通信的规范数据结构.

图像/ SCST-overview.png

## 27.1目的地Binders

目标Binders是Spring Cloud Stream的扩展组件，负责提供必要的配置和实现，以促进与外部邮件系统的集成.此集成负责与生产环境者和使用者之间的消息的连接，委派和路由，数据类型转换，用户代码的调用等.

Binders处理许多锅炉板的责任，否则它们会落在你的肩膀上.然而，为了实现这一点，Binders仍然需要来自用户的简约但需要的指令集的形式的一些帮助，其通常以某种类型的配置的形式出现.

虽然它已经过时了本节的范围将讨论所有可用的Binders和绑定配置选项（本手册的其余部分将对其进行全面介绍），目标绑定需要特别注意.下一节将详细讨论它.

## 27.2目的地绑定

如前所述，Destination Bindings在外部消息传递系统和应用程序提供的生产环境者和消费者之间提供了一个桥梁.

将@EnableBinding批注应用于其中一个应用程序的配置类可定义目标绑定.  `@EnableBinding` 注释本身使用 `@Configuration` 进行元注释，并触发Spring Cloud Stream基础结构的配置.

以下示例显示了一个完全配置且正常运行的Spring Cloud Stream应用程序，它将 `INPUT` 目标中的消息的有效负载作为 `String` 类型接收（请参阅[Chapter 30, Content Type Negotiation](multi_content-type-management.html)部分），将其记录到控制台并在将其转换为 `OUTPUT` 目标后将其发送到 `OUTPUT` 目标大写.

```java
@SpringBootApplication
@EnableBinding(Processor.class)
public class MyApplication {

	public static void main(String[] args) {
		SpringApplication.run(MyApplication.class, args);
	}

	@StreamListener(Processor.INPUT)
	@SendTo(Processor.OUTPUT)
	public String handle(String value) {
		System.out.println("Received: " + value);
		return value.toUpperCase();
	}
}
```

如您所见， `@EnableBinding` 注释可以将一个或多个接口类作为参数.这些参数称为绑定，它们包含表示可绑定组件的方法.这些组件通常是基于通道的Binders（例如Rabbit，Kafka等）的消息通道（请参阅[Spring Messaging](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-messaging.html)）.然而，其他类型的绑定可以为相应技术的本机特征提供支持.例如，Kafka Streams binder（以前称为KStream）允许直接绑定到Kafka Streams的本机绑定（有关详细信息，请参阅[Kafka Streams](https://docs.spring.io/autorepo/docs/spring-cloud-stream-binder-kafka-docs/1.1.0.M1/reference/htmlsingle/)）.

Spring Cloud Stream已经为典型的消息交换Contract提供了绑定接口，其中包括：

-  **Sink:** 通过提供消息所用的目标来标识消息使用者的Contract.

-  **Source:** 通过提供发送生成的消息的目标来标识消息生产环境者的Contract.

-  **Processor:** 通过公开允许消费和生成消息的两个目的地来封装接收器和源Contract.

```java
public interface Sink {

String INPUT = "input";

@Input(Sink.INPUT)
SubscribableChannel input();
}
```

```java
public interface Source {

String OUTPUT = "output";

@Output(Source.OUTPUT)
MessageChannel output();
}
```

```java
public interface Processor extends Source, Sink {}
```

虽然前面的示例满足大多数情况，但您也可以通过定义自己的绑定接口来定义自己的Contract，并使用 `@Input` 和 `@Output` 注释来标识实际的可绑定组件.

例如：

```java
public interface Barista {

@Input
SubscribableChannel orders();

@Output
MessageChannel hotDrinks();

@Output
MessageChannel coldDrinks();
}
```

使用前面示例中显示的接口作为 `@EnableBinding` 的参数，分别触发创建名为 `orders` ， `hotDrinks` 和 `coldDrinks` 的三个绑定通道.

您可以根据需要提供尽可能多的绑定接口，作为 `@EnableBinding` 注释的参数，如以下示例所示：

```java
@EnableBinding(value = { Orders.class, Payment.class })
```

在Spring Cloud Stream中，可绑定的 `MessageChannel` 组件是Spring Messaging  `MessageChannel` （用于出站）及其扩展名 `SubscribableChannel` （用于入站）.

**Pollable Destination Binding** 

虽然之前描述的绑定支持基于事件的消息消耗，但有时您需要更多控制，例如消耗率.

从2.0版开始，您现在可以绑定可轮询的使用者：

以下示例显示如何绑定可轮询使用者：

```java
public interface PolledBarista {

@Input
PollableMessageSource orders();
	. . .
}
```

在这种情况下， `PollableMessageSource` 的实现绑定到 `orders` “通道”.有关详细信息，请参阅[Section 27.3.4, “Using Polled Consumers”](multi__programming_model.html#spring-cloud-streams-overview-using-polled-consumers).

**Customizing Channel Names** 

通过使用 `@Input` 和 `@Output` 注释，您可以为通道指定自定义通道名称，如以下示例所示：

```java
public interface Barista {
@Input("inboundOrders")
SubscribableChannel orders();
}
```

在前面的示例中，创建的绑定通道名为 `inboundOrders` .

通常，您无需直接访问单个通道或绑定（除了通过 `@EnableBinding` 注释配置它们之外）.但是，有时可能会出现测试或其他角落情况.

除了为每个绑定生成通道并将它们注册为Spring bean之外，对于每个绑定的接口，Spring Cloud Stream都会生成一个实现该接口的bean.这意味着您可以通过在应用程序中自动连接来访问表示绑定或单个通道的接口，如以下两个示例所示：

Autowire绑定界面

```java
@Autowire
private Source source

public void sayHello(String name) {
source.output().send(MessageBuilder.withPayload(name).build());
}
```

自动装配个人Channels

```java
@Autowire
private MessageChannel output;

public void sayHello(String name) {
output.send(MessageBuilder.withPayload(name).build());
}
```

对于自定义通道名称的情况或需要特定命名通道的多通道方案，您还可以使用标准Spring的 `@Qualifier` 注释.

以下示例显示如何以这种方式使用@Qualifier注释：

```java
@Autowire
@Qualifier("myChannel")
private MessageChannel output;
```

## 27.3制作和使用消息

您可以使用Spring Integration注释或Spring Cloud Stream本机注释编写Spring Cloud Stream应用程序.

### 27.3.1 Spring集成支持

Spring Cloud StreamBuild在[Enterprise Integration Patterns](http://www.enterpriseintegrationpatterns.com/)定义的概念和模式之上，并依赖于其内部实现，在Spring项目组合中已经Build和流行的企业集成模式实现：[Spring Integration](https://projects.spring.io/spring-integration/)框架.

所以它唯一的理由就是支持Spring Integration已经Build的基础，语义和配置选项

例如，你可以将 `Source` 的输出通道附加到 `MessageSource` 并使用熟悉的 `@InboundChannelAdapter` 注释，如下所示：

```java
@EnableBinding(Source.class)
public class TimerSource {

@Bean
@InboundChannelAdapter(value = Source.OUTPUT, poller = @Poller(fixedDelay = "10", maxMessagesPerPoll = "1"))
public MessageSource<String> timerMessageSource() {
return () -> new GenericMessage<>("Hello Spring Cloud Stream");
}
}
```

同样，您可以使用@Transformer或@ServiceActivator，同时为处理器绑定Contract提供消息处理程序方法的实现，如以下示例所示：

```java
@EnableBinding(Processor.class)
public class TransformProcessor {
@Transformer(inputChannel = Processor.INPUT, outputChannel = Processor.OUTPUT)
public Object transform(String message) {
return message.toUpperCase();
}
}
```

> 虽然这可能会稍微跳过一点，但重要的是要理解，当您使用 `@StreamListener` 注释从同一个绑定中使用时，会使用pub-sub模型.每个使用 `@StreamListener` 注释的方法都会收到自己的消息副本，每个消息都有自己的使用者组.但是，如果使用其中一个Spring Integration注释（例如 `@Aggregator` ， `@Transformer` 或 `@ServiceActivator` ）从相同的绑定中使用它们，则会在竞争模型中使用这些注释.没有为每个订阅创建单个消费者组.

### 27.3.2使用@StreamListener注释

作为Spring Integration支持的补充，Spring Cloud Stream提供了自己的 `@StreamListener` 注释，模仿其他Spring Messaging注释（ `@MessageMapping` ， `@JmsListener` ， `@RabbitListener` 等），并提供了诸如基于内容的路由等方便性.

```java
@EnableBinding(Sink.class)
public class VoteHandler {

@Autowired
VotingService votingService;

@StreamListener(Sink.INPUT)
public void handle(Vote vote) {
votingService.record(vote);
}
}
```

与其他Spring Messaging方法一样，方法参数可以使用 `@Payload` ， `@Headers` 和 `@Header` 进行批注.

对于返回数据的方法，必须使用 `@SendTo` 批注指定方法返回的数据的输出绑定目标，如以下示例所示：

```java
@EnableBinding(Processor.class)
public class TransformProcessor {

@Autowired
VotingService votingService;

@StreamListener(Processor.INPUT)
@SendTo(Processor.OUTPUT)
public VoteResult handle(Vote vote) {
return votingService.record(vote);
}
}
```

### 27.3.3使用@StreamListener进行基于内容的路由

Spring Cloud Stream支持根据条件将消息分派给使用 `@StreamListener` 注释的多个处理程序方法.

为了有资格支持条件分派，方法必须满足以下条件：

- 不得返回值.

- 它必须是单独的消息处理方法（不支持反应式API方法）.

条件由注释的 `condition` 参数中的SpEL表达式指定，并针对每条消息进行评估.匹配条件的所有处理程序都在同一个线程中调用，并且不必假设调用发生的顺序.

在具有调度条件的 `@StreamListener` 的以下示例中，带有值为 `bogey` 的头 `type` 的所有消息都将调度到 `receiveBogey` 方法，并且带有值 `type` 的头 `type` 的所有消息都将调度到 `receiveBacall` 方法.

```java
@EnableBinding(Sink.class)
@EnableAutoConfiguration
public static class TestPojoWithAnnotatedArguments {

@StreamListener(target = Sink.INPUT, condition = "headers['type']=='bogey'")
public void receiveBogey(@Payload BogeyPojo bogeyPojo) {
// handle the message
}

@StreamListener(target = Sink.INPUT, condition = "headers['type']=='bacall'")
public void receiveBacall(@Payload BacallPojo bacallPojo) {
// handle the message
}
}
```

**Content Type Negotiation in the Context of condition** 

使用 `@StreamListener` 的 `condition` 参数了解基于内容的路由背后的一些机制非常重要，尤其是在整个消息类型的上下文中.如果您在继续操作之前熟悉[Chapter 30, Content Type Negotiation](multi_content-type-management.html)，这也可能有所帮助.

请考虑以下情形：

```java
@EnableBinding(Sink.class)
@EnableAutoConfiguration
public static class CatsAndDogs {

@StreamListener(target = Sink.INPUT, condition = "payload.class.simpleName=='Dog'")
public void bark(Dog dog) {
// handle the message
}

@StreamListener(target = Sink.INPUT, condition = "payload.class.simpleName=='Cat'")
public void purr(Cat cat) {
// handle the message
}
}
```

上述代码完全有效.它编译和部署没有任何问题，但它永远不会产生您期望的结果.

那是因为你正在测试一些在你期望的状态下尚不存在的东西.这是因为消息的有效负载尚未从有线格式（ `byte[]` ）转换为所需类型.换句话说，它还没有经过[Chapter 30, Content Type Negotiation](multi_content-type-management.html)中描述的类型转换过程.

因此，除非使用评估原始数据的SPeL表达式（例如，字节数组中第一个字节的值），否则请使用基于消息头的表达式（例如 `condition = "headers['type']=='dog'"` ）.

> 目前，仅支持基于通道的Binders（不支持响应式编程）支持，通过 `@StreamListener` 条件进行调度.

### 27.3.4使用受轮询的消费者

使用轮询的使用者时，您可以根据需要轮询 `PollableMessageSource` .考虑以下轮询消费者的示例：

```java
public interface PolledConsumer {

@Input
PollableMessageSource destIn();

@Output
MessageChannel destOut();

}
```

鉴于前面示例中的轮询消费者，您可以按如下方式使用它：

```java
@Bean
public ApplicationRunner poller(PollableMessageSource destIn, MessageChannel destOut) {
return args -> {
while (someCondition()) {
try {
if (!destIn.poll(m -> {
String newPayload = ((String) m.getPayload()).toUpperCase();
destOut.send(new GenericMessage<>(newPayload));
})) {
Thread.sleep(1000);
}
}
catch (Exception e) {
// handle failure (throw an exception to reject the message);
}
}
};
}
```

`PollableMessageSource.poll()` 方法采用 `MessageHandler` 参数（通常是lambda表达式，如此处所示）.如果收到并成功处理了消息，则返回 `true` .

与消息驱动的使用者一样，如果 `MessageHandler` 抛出异常，则会将消息发布到错误通道，如“[???]()”中所述.

通常， `poll()` 方法在 `MessageHandler` 退出时确认消息.如果方法异常退出，则拒绝该消息（不重新排队）.您可以通过承担确认责任来覆盖该行为，如以下示例所示：

```java
@Bean
public ApplicationRunner poller(PollableMessageSource dest1In, MessageChannel dest2Out) {
return args -> {
while (someCondition()) {
if (!dest1In.poll(m -> {
StaticMessageHeaderAccessor.getAcknowledgmentCallback(m).noAutoAck();
// e.g. hand off to another thread which can perform the ack
// or acknowledge(Status.REQUEUE)

})) {
Thread.sleep(1000);
}
}
};
}
```

|图片/ important.png |重要|
| ---- | ---- |
|您必须在某个时刻 `ack` （或 `nack` ）消息，以避免资源泄漏. |

|图片/ important.png |重要|
| ---- | ---- |
|某些消息传递系统（如Apache Kafka）在日志中维护一个简单的偏移量.如果传递失败并使用 `StaticMessageHeaderAccessor.getAcknowledgmentCallback(m).acknowledge(Status.REQUEUE);` 重新排队，则会重新传递任何以后成功获得的消息. |

还有一个重载的 `poll` 方法，其定义如下：

```java
poll(MessageHandler handler, ParameterizedTypeReference<?> type)
```

`type` 是转换提示，允许转换传入的消息有效内容，如以下示例所示：

```java
boolean result = pollableSource.poll(received -> {
			Map<String, Foo> payload = (Map<String, Foo>) received.getPayload();
...

		}, new ParameterizedTypeReference<Map<String, Foo>>() {});
```

## 27.4错误处理

错误发生，Spring CloudStream提供了几种灵活的机制来处理它们.错误处理有两种形式：

-  **application:** 错误处理在应用程序内完成（自定义错误处理程序）.

-  **system:** 错误处理被委托给绑定程序（重新排队，DL和其他）.请注意，这些技术取决于Binders实现和底层消息传递中间件的功能.

Spring Cloud Stream使用[Spring Retry](https://github.com/spring-projects/spring-retry)库来促进成功的消息处理.有关详细信息，请参阅[Section 27.4.3, “Retry Template”](multi__programming_model.html#_retry_template).但是，当全部失败时，消息处理程序抛出的异常将传播回Binders.此时，binder调用自定义错误处理程序或将错误传回消息传递系统（重新排队，DLQ和其他）.

### 27.4.1应用程序错误处理

有两种类型的应用程序级错误处理.可以在每个绑定订阅处理错误，或者全局处理程序可以处理所有绑定订阅错误.我们来看看细节.

**Figure 27.1. A Spring Cloud Stream Sink Application with Custom and Global Error Handlers** 

图片/ custom_vs_global_error_channels.png

对于每个输入绑定，Spring Cloud Stream使用以下语义创建专用错误通道 `<destinationName>.errors` .

>   `<destinationName>` 由绑定的名称（例如 `input` ）和组的名称（例如 `myGroup` ）组成.

考虑以下：

```java
spring.cloud.stream.bindings.input.group=myGroup
```

```java
@StreamListener(Sink.INPUT) // destination name 'input.myGroup'
public void handle(Person value) {
	throw new RuntimeException("BOOM!");
}

@ServiceActivator(inputChannel = Processor.INPUT + ".myGroup.errors") //channel name 'input.myGroup.errors'
public void error(Message<?> message) {
	System.out.println("Handling ERROR: " + message);
}
```

在前面的示例中，目标名称为 `input.myGroup` ，专用错误通道名称为 `input.myGroup.errors` .

>  @StreamListener注释的使用专门用于定义桥接内部通道和外部目标的绑定.鉴于目标特定错误通道没有关联的外部目标，此类通道是Spring Integration（SI）的特权.这意味着必须使用SI处理程序注释之一来定义此类目标的处理程序（即@ServiceActivator，@ Transformer等）.

> 如果未指定 `group` ，则使用匿名组（类似于 `input.anonymous.2K37rb06Q6m2r51-SPIDDQ` ），这不适合错误处理，因为在创建目标之前您不知道它会是什么.

此外，如果您绑定到现有目标，例如：

```java
spring.cloud.stream.bindings.input.destination=myFooDestination
spring.cloud.stream.bindings.input.group=myGroup
```

完整的目标名称是 `myFooDestination.myGroup` ，然后专用的错误通道名称是 `myFooDestination.myGroup.errors` .

回到例子......

订阅名为 `input` 的通道的 `handle(..)` 方法会抛出异常.鉴于还存在错误信道 `input.myGroup.errors` 的订户，所有错误消息都由该订户处理.

如果您有多个绑定，则可能需要一个错误处理程序. Spring Cloud Stream通过将每个错误通道桥接到名为 `errorChannel` 的通道，自动为全局错误通道提供支持，允许单个订户处理所有错误，如以下示例所示：

```java
@StreamListener("errorChannel")
public void error(Message<?> message) {
	System.out.println("Handling ERROR: " + message);
}
```

如果错误处理逻辑相同，无论哪个处理程序产生错误，这可能是一个方便的选项.

此外，通过为出站目标配置名为 `error` 的绑定，可以将发送到 `errorChannel` 的错误消息发布到代理的特定目标.此选项提供了一种机制，可以将错误消息自动发送到绑定到该目标的另一个应用程序，或供以后检索（例如，审计）.例如，要将错误消息发布到名为 `myErrors` 的代理目标，请设置以下属性：

```java
spring.cloud.stream.bindings.error.destination=myErrors.
```

> 将全局错误通道桥接到代理目标的能力实质上提供了一种机制，它将应用程序级错误处理与系统级错误处理相连接.

### 27.4.2系统错误处理

系统级错误处理意味着将错误传递回消息传递系统，并且假设并非每个消息传递系统都相同，则功能可能因绑定程序而异.

也就是说，在本节中，我们将解释系统级错误处理背后的一般概念，并以Rabbit binder为例.注意：虽然某些配置属性有所不同，但Kafka binder提供了类似的支持.另外，有关更多详细信息和配置选项，请参阅各个Binders的文档.

如果未配置内部错误处理程序，则错误会传播到绑定程序，然后绑定程序会将这些错误传播回消息传递系统.根据消息传递系统的功能，这样的系统可以丢弃消息，重新排队消息以进行重新处理或将失败的消息发送到DLQ. Rabbit和Kafka都支持这些概念.但是，其他绑定程序可能不会，因此请参阅各个绑定程序的文档，以获取有关受支持的系统级错误处理选项的详细信息.

#### Drop失败的消息

默认情况下，如果未提供其他系统级配置，则消息传递系统将丢弃失败的消息.虽然在某些情况下可接受，但在大多数情况下，它不是，我们需要一些恢复机制来避免消息丢失.

#### DLQ - 死信队列

DLQ允许发送失败的消息一个特殊的目的地： - 死信队列.

配置后，失败的消息将发送到此目标，以便后续重新处理或审核和协调.

例如，继续上一个示例并使用Rabbit binder设置DLQ，您需要设置以下属性：

```java
spring.cloud.stream.rabbit.bindings.input.consumer.auto-bind-dlq=true
```

请记住，在上面的属性中， `input` 对应于输入目标绑定的名称.  `consumer` 表示它是一个使用者属性， `auto-bind-dlq` 指示Binders为 `input` 目标配置DLQ，这会导致另一个名为 `input.myGroup.dlq` 的Rabbit队列.

配置完成后，所有失败的消息都将路由到此队列，并显示类似于以下内容的错误消息：

```java
delivery_mode:	1
headers:
x-death:
count:	1
reason:	rejected
queue:	input.hello
time:	1522328151
exchange:
routing-keys:	input.myGroup
Payload {"name”:"Bob"}
```

从上面的内容可以看出，您的原始邮件会被保留以供进一步操作.

但是，您可能注意到的一件事是，有关消息处理的原始问题的信息有限.例如，您没有看到与原始错误对应的堆栈跟踪.要获取有关原始错误的更多相关信息，您必须设置其他属性：

```java
spring.cloud.stream.rabbit.bindings.input.consumer.republish-to-dlq=true
```

这样做会强制内部错误处理程序拦截错误消息，并在将其发布到DLQ之前向其添加其他信息.配置完成后，您可以看到错误消息包含与原始错误相关的更多信息，如下所示：

```java
delivery_mode:	2
headers:
x-original-exchange:
x-exception-message:	has an error
x-original-routingKey:	input.myGroup
x-exception-stacktrace:	org.springframework.messaging.MessageHandlingException: nested exception is
org.springframework.messaging.MessagingException: has an error, failedMessage=GenericMessage [payload=byte[15],
headers={amqp_receivedDeliveryMode=NON_PERSISTENT, amqp_receivedRoutingKey=input.hello, amqp_deliveryTag=1,
deliveryAttempt=3, amqp_consumerQueue=input.hello, amqp_redelivered=false, id=a15231e6-3f80-677b-5ad7-d4b1e61e486e,
amqp_consumerTag=amq.ctag-skBFapilvtZhDsn0k3ZmQg, contentType=application/json, timestamp=1522327846136}]
at org.spring...integ...han...MethodInvokingMessageProcessor.processMessage(MethodInvokingMessageProcessor.java:107)
at. . . . .
Payload {"name”:"Bob"}
```

这有效地结合了应用程序级和系统级错误处理，以进一步帮助下游故障排除机制.

#### Re-queue失败的消息

如前所述，当前支持的Binders（Rabbit和Kafka）依赖 `RetryTemplate` 来促进成功的消息处理.有关详细信息，请参阅[Section 27.4.3, “Retry Template”](multi__programming_model.html#_retry_template).但是，对于 `max-attempts` 属性设置为1的情况，将禁用消息的内部重新处理.此时，您可以通过指示消息传递系统重新排队失败的消息来促进消息重新处理（重新尝试）.重新排队后，失败的消息将被发送回原始处理程序，实质上是创建重试循环.

对于错误的性质与某些资源的某些零星但短期不可用相关的情况，此选项可能是可行的.

要实现此目的，您必须设置以下属性：

```java
spring.cloud.stream.bindings.input.consumer.max-attempts=1
spring.cloud.stream.rabbit.bindings.input.consumer.requeue-rejected=true
```

在前面的示例中， `max-attempts` 设置为1基本上禁用内部重试， `requeue-rejected` （重新排队拒绝消息的简称）设置为 `true` .一旦设置，失败的消息将重新提交到同一个处理程序并连续循环或直到处理程序抛出 `AmqpRejectAndDontRequeueException` 本质上允许您在处理程序本身内构建自己的重试逻辑.

### 27.4.3重试模板

`RetryTemplate` 是[Spring Retry](https://github.com/spring-projects/spring-retry)库的一部分.虽然涵盖 `RetryTemplate` 的所有功能超出了本文档的范围，但我们将提及以下与 `RetryTemplate` 具体相关的消费者属性：

maxAttempts处理消息的尝试次数.默认值：3.backOffInitialInterval重试时的退避初始间隔.默认为1000毫秒. backOffMaxInterval最大退避间隔.默认10000毫秒. backOffMultiplier退避乘数.默认2.0.

虽然前面的设置足以满足大多数自定义要求，但它们可能无法满足某些复杂要求，您可能希望提供自己的 `RetryTemplate` 实例.为此，请将其配置为应用程序配置中的bean.应用程序提供的实例将覆盖框架提供的实例.另外，为了避免冲突，您必须将要将Binders使用的 `RetryTemplate` 实例限定为 `@StreamRetryTemplate` .例如，

```java
@StreamRetryTemplate
public RetryTemplate myRetryTemplate() {
return new RetryTemplate();
}
```

从上面的例子中可以看出，你不需要用 `@Bean` 注释它，因为 `@StreamRetryTemplate` 是一个合格的 `@Bean` .

## 27.5反应式编程支持

Spring Cloud Stream还支持使用响应式API，其中传入和传出数据作为连续数据流进行处理.通过 `spring-cloud-stream-reactive` 可以获得对响应API的支持，需要将其明确添加到您的项目中.

具有反应API的编程模型是声明性的.您可以使用描述从入站数据流到出站数据流的功能转换的运算符，而不是指定应如何处理每条消息.

目前Spring Cloud Stream仅支持[Reactor API](https://projectreactor.io/).将来，我们打算支持基于Reactive Streams的更通用的模型.

反应式编程模型还使用 `@StreamListener` 注释来设置反应式处理程序.不同之处在于：

-   `@StreamListener` 注释不能指定输入或输出，因为它们作为参数提供并从方法返回值.

- 方法的参数必须使用 `@Input` 和 `@Output` 进行注释，分别指示传入和传出数据流连接到哪个输入或输出.

- 方法的返回值（如果有）用 `@Output` 注释，表示输入应该发送数据的位置.

> Reactive编程支持需要Java 1.8.

> 作为Spring Cloud Stream 1.1.1及更高版本（从版本系列Brooklyn.SR2开始），反应式编程支持需要使用Reactor 3.0.4.RELEASE及更高版本.不支持早期的Reactor版本（包括3.0.1.RELEASE，3.0.2.RELEASE和3.0.3.RELEASE）.  `spring-cloud-stream-reactive` 传递性地检索正确的版本，但项目结构可以将 `io.projectreactor:reactor-core` 的版本管理到早期版本，尤其是在使用Maven时.对于使用Spring Initializr和Spring Boot 1.x生成的项目就是这种情况，它将Reactor版本覆盖到 `2.0.8.RELEASE` .在这种情况下，您必须确保释放正确版本的工件.您可以通过将 `io.projectreactor:reactor-core` 的直接依赖项添加到项目的 `3.0.4.RELEASE` 或更高版本中来实现.

> 术语“被动”的使用，当前是指所使用的被动API而不是被动的执行模型（即，绑定endpoints仍然使用'推'而不是'拉'模型）.虽然使用Reactor提供了一些背压支持，但我们打算在未来的版本中通过使用连接中间件的本机反应客户端来支持完全反应的管道.

### 27.5.1基于反应堆的处理程序

基于Reactor的处理程序可以具有以下参数类型：

- For用 `@Input` 注释的参数，它支持Reactor  `Flux` 类型.入站Flux的参数化遵循与单个消息处理相同的规则：它可以是整个 `Message` ，可以是 `Message` 有效负载的POJO，或者是基于 `Message` 内容的转换结果的POJO -type标头.提供多个输入.

- For用 `Output` 注释的参数，它支持 `FluxSender` 类型，它将方法生成的 `Flux` 与输出连接起来.一般而言，仅当方法可以具有多个输出时，才建议将输出指定为参数.

基于Reactor的处理程序支持 `Flux` 的返回类型.在这种情况下，必须使用 `@Output` 进行注释.当单个输出 `Flux` 可用时，我们建议使用该方法的返回值.

以下示例显示了基于Reactor的 `Processor` ：

```java
@EnableBinding(Processor.class)
@EnableAutoConfiguration
public static class UppercaseTransformer {

@StreamListener
@Output(Processor.OUTPUT)
public Flux<String> receive(@Input(Processor.INPUT) Flux<String> input) {
return input.map(s -> s.toUpperCase());
}
}
```

使用输出参数的同一处理器类似于以下示例：

```java
@EnableBinding(Processor.class)
@EnableAutoConfiguration
public static class UppercaseTransformer {

@StreamListener
public void receive(@Input(Processor.INPUT) Flux<String> input,
@Output(Processor.OUTPUT) FluxSender output) {
output.send(input.map(s -> s.toUpperCase()));
}
}
```

### 27.5.2无功来源

Spring Cloud Stream反应支持还提供了通过 `@StreamEmitter` 注释创建反应源的功能.通过使用 `@StreamEmitter` 注释，常规源可以转换为反应源.  `@StreamEmitter` 是一个方法级别注释，它将方法标记为使用 `@EnableBinding` 声明的输出的Launcher.您不能将 `@Input` 注释与 `@StreamEmitter` 一起使用，因为使用此注释标记的方法不会侦听任何输入.相反，标有 `@StreamEmitter` 的方法会生成输出.遵循 `@StreamListener` 中使用的相同编程模型， `@StreamEmitter` 还允许使用 `@Output` 注释的灵活方式，具体取决于方法是否具有任何参数，返回类型和其他注意事项.

本节的其余部分包含使用各种样式的 `@StreamEmitter` 注释的示例.

以下示例每毫秒发出 `Hello, World` 消息并发布到Reactor  `Flux` ：

```java
@EnableBinding(Source.class)
@EnableAutoConfiguration
public static class HelloWorldEmitter {

@StreamEmitter
@Output(Source.OUTPUT)
public Flux<String> emit() {
return Flux.intervalMillis(1)
.map(l -> "Hello World");
}
}
```

在前面的示例中， `Flux` 中的结果消息被发送到 `Source` 的输出通道.

下一个例子是发送Reactor  `Flux` 的 `@StreamEmmitter` 的另一种味道.以下方法使用 `FluxSender` 以编程方式从源发送 `Flux` ，而不是返回 `Flux` ：

```java
@EnableBinding(Source.class)
@EnableAutoConfiguration
public static class HelloWorldEmitter {

@StreamEmitter
@Output(Source.OUTPUT)
public void emit(FluxSender output) {
output.send(Flux.intervalMillis(1)
.map(l -> "Hello World"));
}
}
```

下一个示例与功能和样式中的上述代码段完全相同.但是，它不使用方法上的显式 `@Output` 注释，而是使用方法参数上的注释.

```java
@EnableBinding(Source.class)
@EnableAutoConfiguration
public static class HelloWorldEmitter {

@StreamEmitter
public void emit(@Output(Source.OUTPUT) FluxSender output) {
output.send(Flux.intervalMillis(1)
.map(l -> "Hello World"));
}
}
```

本节中的最后一个示例是使用Reactive Streams Publisher API编写反应源的另一种方法，并利用[Spring Integration Java DSL](https://github.com/spring-projects/spring-integration-java-dsl/wiki/Spring-Integration-Java-DSL-Reference)中对它的支持.以下示例中的 `Publisher` 仍然使用Reactor  `Flux` ，但从应用程序的角度来看，这对用户是透明的，并且只需要Reactive Streams和Java DSL for Spring Integration：

```java
@EnableBinding(Source.class)
@EnableAutoConfiguration
public static class HelloWorldEmitter {

@StreamEmitter
@Output(Source.OUTPUT)
@Bean
public Publisher<Message<String>> emit() {
return IntegrationFlows.from(() ->
new GenericMessage<>("Hello World"),
e -> e.poller(p -> p.fixedDelay(1)))
.toReactivePublisher();
}
}
```

