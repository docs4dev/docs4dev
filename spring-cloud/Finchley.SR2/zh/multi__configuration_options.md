## 29.配置选项

Spring Cloud Stream支持常规配置选项以及绑定和Binders的配置.某些Binders允许其他绑定属性支持特定于中间件的功能.

可以通过Spring Boot支持的任何机制向Spring Cloud Stream应用程序提供配置选项.这包括应用程序参数，环境变量以及YAML或.properties文件.

## 29.1绑定服务属性

这些属性通过 `org.springframework.cloud.stream.config.BindingServiceProperties` 公开

spring.cloud.stream.instanceCount应用程序的已部署实例数.必须设置为在生产环境者端进行分区.必须在使用RabbitMQ时设置在消费者端，如果autoRebalanceEnabled = false，则必须设置为Kafka.默认值：1.spring.cloud.stream.instanceIndex应用程序的实例索引：从0到instanceCount的数字 -  1.用于使用RabbitMQ进行分区，如果autoRebalanceEnabled = false，则使用Kafka进行分区.在Cloud Foundry中自动设置以匹配应用程序的实例索引. spring.cloud.stream.dynamicDestinations可以动态绑定的目标列表（例如，在动态路由方案中）.如果设置，则只能绑定列出的目标.默认值：空（允许绑定任何目的地）. spring.cloud.stream.defaultBinder如果配置了多个Binders，则使用默认Binders.请参阅类路径上的多个Binders.默认值：空. spring.cloud.stream.overrideCloudConnectors此属性仅在Cloud配置文件处于活动状态且Spring Cloud Connectors随应用程序提供时才适用.如果属性为false（默认值），则绑定程序会检测到合适的绑定服务（例如，绑定在Cloud Foundry中的RabbitMQ服务，用于RabbitMQ绑定程序）并使用它来创建连接（通常通过Spring Cloud Connectors）.设置为true时，此属性指示Binders完全忽略绑定服务并依赖Spring Boot属性（例如，依赖于提供的spring.rabbitmq.*属性在RabbitMQBinders的环境中）.在连接到多个系统时，此属性的典型用法是嵌套在自定义环境中.默认值：false. spring.cloud.stream.bindingRetryInterval例如，当绑定程序不支持后期绑定且代理（例如，Apache Kafka）关闭时，重试绑定创建之间的间隔（以秒为单位）.将其设置为零以将此类条件视为致命，从而阻止应用程序启动.默认值：30

## 29.2绑定属性

使用 `spring.cloud.stream.bindings.<channelName>.<property>=<value>` 的格式提供绑定属性.  `<channelName>` 表示正在配置的通道的名称（例如， `output` 表示 `Source` ）.

为避免重复，Spring Cloud Stream支持所有通道的设置值，格式为 `spring.cloud.stream.default.<property>=<value>` .

在下文中，我们指出我们在哪里省略了 `spring.cloud.stream.bindings.<channelName>.` 前缀并仅关注属性名称，并理解运行时包含前缀ise.

### 29.2.1常用绑定属性

这些属性通过 `org.springframework.cloud.stream.config.BindingProperties` 公开

以下绑定属性可用于输入和输出绑定，并且必须以 `spring.cloud.stream.bindings.<channelName>.` 为前缀（例如， `spring.cloud.stream.bindings.input.destination=ticktock` ）.

可以使用 `spring.cloud.stream.default` 前缀设置默认值（例如`spring.cloud.stream.default.contentType = application / json`）.

destination绑定中间件上通道的目标目标（例如，RabbitMQ交换或Kafka主题）.如果通道绑定为使用者，则可以将其绑定到多个目标，并且可以将目标名称指定为逗号分隔的String值.如果未设置，则使用通道名称.无法覆盖此属性的默认值. group通道的使用者组.仅适用于入站绑定.见消费者组.默认值：null（表示匿名使用者）. contentType通道的内容类型.请参阅“第30章，内容类型协商”.默认值：null（不执行类型强制）. binder此绑定使用的Binders.有关详细信息，请参见“类路径”中的第28.4节“多个Binders”.默认值：null（使用默认Binders，如果存在）.

### 29.2.2消费者属性

这些属性通过 `org.springframework.cloud.stream.binder.ConsumerProperties` 公开

以下绑定属性仅可用于输入绑定，并且必须以 `spring.cloud.stream.bindings.<channelName>.consumer.` 为前缀（例如， `spring.cloud.stream.bindings.input.consumer.concurrency=3` ）.

可以使用 `spring.cloud.stream.default.consumer` 前缀（例如， `spring.cloud.stream.default.consumer.headerMode=none` ）设置默认值.

并发入站消费者的并发性.默认值：1.partitioned消费者是否从分区生产环境者接收数据.默认值：false. headerMode设置为none时，禁用输入上的标头解析.仅对本身不支持邮件头并且需要标头嵌入的邮件中间件有效.当不支持本机标头时，从非Spring Cloud Stream应用程序使用数据时，此选项很有用.设置为标头时，它使用中间件的本机标头机制.设置为embeddedHeaders时，它会将标头嵌入到消息有效内容中.默认值：取决于Binders实现. maxAttempts如果处理失败，则处理消息的尝试次数（包括第一次）.设置为1以禁用重试.默认值：3.backOffInitialInterval重试时的退避初始间隔.默认值：1000.backOffMaxInterval最大退避间隔.默认值：10000.backOffMultiplier退避乘数.默认值：2.0. instanceIndex当设置为大于等于零的值时，它允许自定义此使用者的实例索引（如果与spring.cloud.stream.instanceIndex不同）.设置为负值时，默认为spring.cloud.stream.instanceIndex.有关详细信息，请参见“第32.2节”，“实例索引和实例计数”.默认值：-1. instanceCount当设置为大于等于零的值时，它允许自定义此使用者的实例计数（如果与spring.cloud.stream.instanceCount不同）.设置为负值时，默认为spring.cloud.stream.instanceCount.有关详细信息，请参见“第32.2节”，“实例索引和实例计数”.默认值：-1. useNativeDecoding设置为true时，客户端库直接反序列化入站消息，必须相应地对其进行配置（例如，设置适当的Kafka生成器值反序列化器）.使用此配置时，入站消息解组不基于绑定的contentType.当使用本机解码时，生产环境者有责任使用适当的编码器（例如，Kafka生产环境者值序列化器）来序列化出站消息.此外，使用本机编码和解码时，将忽略headerMode = embeddedHeaders属性，并且不会在消息中嵌入标头.请参阅生产环境者属性useNativeEncoding.默认值：false.

### 29.2.3制片人属性

这些属性通过 `org.springframework.cloud.stream.binder.ProducerProperties` 公开

可以使用以下绑定属性仅输出绑定，必须以 `spring.cloud.stream.bindings.<channelName>.producer.` 为前缀（例如， `spring.cloud.stream.bindings.input.producer.partitionKeyExpression=payload.id` ）.

可以使用前缀 `spring.cloud.stream.default.producer` （例如， `spring.cloud.stream.default.producer.partitionKeyExpression=payload.id` ）设置默认值.

partitionKeyExpression一个SpEL表达式，用于确定如何对出站数据进行分区.如果设置，或者设置了partitionKeyExtractorClass，则此通道上的出站数据将被分区.必须将partitionCount设置为大于1的值才能生效.与partitionKeyExtractorClass互斥.请参见“第26.6节”，“分区支持”.默认值：null. partitionKeyExtractorClass一个PartitionKeyExtractorStrategy实现.如果设置，或者设置了partitionKeyExpression，则此通道上的出站数据将被分区.必须将partitionCount设置为大于1的值才能生效.与partitionKeyExpression互斥.请参见“第26.6节”，“分区支持”.默认值：null. partitionSelectorClass一个PartitionSelectorStrategy实现.与partitionSelectorExpression互斥.如果两者都未设置，则选择分区作为hashCode（key）％partitionCount，其中key通过partitionKeyExpression或partitionKeyExtractorClass计算.默认值：null. partitionSelectorExpression用于自定义分区选择的SpEL表达式.与partitionSelectorClass互斥.如果两者都未设置，则选择分区作为hashCode（key）％partitionCount，其中key通过partitionKeyExpression或partitionKeyExtractorClass计算.默认值：null. partitionCount如果启用了分区，则数据的目标分区数.如果生成器已分区，则必须设置为大于1的值.在Kafka，它被解释为暗示.使用较大的这个和目标主题的分区数来代替.默认值：1.requiredGroups生成器必须确保消息传递的组的逗号分隔列表，即使它们在创建之后启动（例如，通过在RabbitMQ中预先创建持久队列）. headerMode设置为none时，它会禁用输出中的标头嵌入.它仅对于本地不支持邮件头并且需要头嵌入的消息传递中间件有效.当不支持本机标头时，为非Spring Cloud Stream应用程序生成数据时，此选项很有用.设置为标头时，它使用中间件的本机标头机制.设置为embeddedHeaders时，它会将标头嵌入到消息有效内容中.默认值：取决于Binders实现. useNativeEncoding设置为true时，出站消息由客户端库直接序列化，必须相应地对其进行配置（例如，设置适当的Kafka生成器值序列化程序）.使用此配置时，出站邮件编组不基于绑定的contentType.当使用本机编码时，消费者有责任使用适当的解码器（例如，Kafka消费者值反序列化器）来反序列化入站消息.此外，使用本机编码和解码时，将忽略headerMode = embeddedHeaders属性，并且不会在消息中嵌入标头.请参阅使用者属性useNativeDecoding.默认值：false. errorChannelEnabled设置为true时，如果绑定程序支持异步发送结果，则会将发送失败发送到目标的错误通道.有关更多信息，请参见“???”.默认值：false.

## 29.3使用动态绑定目标

除了使用 `@EnableBinding` 定义的通道外，Spring Cloud Stream还允许应用程序将消息发送到动态绑定的目标.例如，当需要在运行时确定目标目标时，这很有用.应用程序可以使用由 `@EnableBinding` 注释自动注册的 `BinderAwareChannelResolver`  bean来实现.

'spring.cloud.stream.dynamicDestinations'属性可用于将动态目标名称限制为已知集（白名单）.如果未设置此属性，则可以动态绑定任何目标.

`BinderAwareChannelResolver` 可以直接使用，如以下REST控制器示例所示，使用路径变量来决定目标通道：

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

现在考虑当我们在默认端口（8080）上启动应用程序并使用CURL发出以下请求时会发生什么：

```java
curl -H "Content-Type: application/json" -X POST -d "customer-1" http://localhost:8080/customers

curl -H "Content-Type: application/json" -X POST -d "order-1" http://localhost:8080/orders
```

目的地，“客户”和“订单”在代理（在Rabbit的交换中或在Kafka的主题中）中创建，其名称为“客户”和“订单”，并且数据将发布到适当的目的地.

`BinderAwareChannelResolver` 是一个通用的Spring Integration  `DestinationResolver` ，可以注入其他组件 - 例如，在路由器中使用基于传入JSON消息的 `target` 字段的SpEL表达式.以下示例包含一个读取SpEL表达式的路由器：

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

[Router Sink Application](https://github.com/spring-cloud-stream-app-starters/router)使用此技术按需创建目标.

如果事先知道通道名称，则可以将生产环境者属性配置为与其他任何属性一样目的地.或者，如果您注册 `NewBindingCallback<>`  bean，则会在创建绑定之前调用它.回调采用Binders使用的扩展生产环境者属性的泛型类型.它有一个方法：

```java
void configure(String channelName, MessageChannel channel, ProducerProperties producerProperties,
T extendedProducerProperties);
```

以下示例显示如何使用RabbitMQBinders：

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

> 如果需要支持具有多种Binders类型的动态目标，请使用 `Object` 作为泛型类型，并根据需要强制转换 `extended` 参数.

