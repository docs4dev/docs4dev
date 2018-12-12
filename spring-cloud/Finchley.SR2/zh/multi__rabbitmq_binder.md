## 39. RabbitMQBinders

## 39.1用法

要使用RabbitMQBinders，可以使用以下Maven坐标将其添加到Spring Cloud Stream应用程序中：

```xml
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-stream-binder-rabbit</artifactId>
</dependency>
```

或者，您可以使用Spring Cloud Stream RabbitMQ Starter，如下所示：

```xml
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-starter-stream-rabbit</artifactId>
</dependency>
```

## 39.2 RabbitMQBinders概述

以下简化图显示了RabbitMQBinders的运行方式：

**Figure 39.1. RabbitMQ Binder** 

图像/兔binder.png

默认情况下，RabbitMQ Binder实现将每个目标映射到 `TopicExchange` .对于每个使用者组， `Queue` 绑定到 `TopicExchange` .每个消费者实例都有一个对应的RabbitMQ  `Consumer` 实例，用于其组 `Queue` .对于分区生成器和使用者，队列以分区索引为后缀，并使用分区索引作为路由键.对于匿名使用者（没有 `group` 属性的用户），使用自动删除队列（具有随机的唯一名称）.

通过使用可选的 `autoBindDlq` 选项，您可以配置Binders以创建和配置死信队列（DLQ）（以及死信交换 `DLX` ，以及路由基础结构）.默认情况下，死信队列具有目标的名称，并附加 `.dlq` .如果启用了重试（ `maxAttempts > 1` ），则在重试耗尽后，失败的消息将传递到DLQ.如果禁用重试（ `maxAttempts = 1` ），则应将 `requeueRejected` 设置为 `false` （默认值），以便将失败的消息路由到DLQ，而不是重新排队.此外， `republishToDlq` 会导致绑定程序将失败的消息发布到DLQ（而不是拒绝它）.此功能允许将其他信息（例如 `x-exception-stacktrace` 标头中的堆栈跟踪）添加到标头中的消息中.此选项不需要重试.只需一次尝试即可重新发布失败的消息.从1.2版开始，您可以配置重新发布的消息的传递模式.见[republishDeliveryMode property](multi__rabbitmq_binder.html#spring-cloud-stream-rabbit-republish-delivery-mode).

|图片/ important.png |重要|
| ---- | ---- |
|将 `requeueRejected` 设置为 `true` （使用 `republishToDlq=false` ）会导致消息重新排队并连续重新传递，这可能不是您想要的，除非失败的原因是暂时的.通常，您应该通过将 `maxAttempts` 设置为大于1或通过将 `republishToDlq` 设置为 `true` 来在Binders中启用重试. |

有关这些属性的更多信息，请参见[Section 39.3.1, “RabbitMQ Binder Properties”](multi__rabbitmq_binder.html#rabbit-binder-properties).

该框架没有提供任何标准机制来使用死信消息（或将它们重新路由回主队列）. [Section 39.6, “Dead-Letter Queue Processing”](multi__rabbitmq_binder.html#rabbit-dlq-processing)中描述了一些选项.

> 当在Spring Cloud Stream应用程序中使用多个RabbitMQBinders时，禁用“RabbitAutoConfiguration”以避免将 `RabbitAutoConfiguration` 的相同配置应用于两个Binders非常重要.您可以使用 `@SpringBootApplication` 注释排除该类.

从版本2.0开始， `RabbitMessageChannelBinder` 将 `RabbitTemplate.userPublisherConnection` 属性设置为 `true` ，以便非事务生成器避免消费者死锁，如果缓存连接因代理上的[memory alarm](https://www.rabbitmq.com/memory.html)而被阻止，则可能发生死锁.

## 39.3配置选项

本节包含特定于RabbitMQ Binder和绑定通道的设置.

有关常规绑定配置选项和属性，请参阅[Spring Cloud Stream core documentation](https://github.com/spring-cloud/spring-cloud-stream/blob/master/spring-cloud-stream-core-docs/src/main/asciidoc/spring-cloud-stream-overview.adoc#configuration-options).

### 39.3.1 RabbitMQBinders属性

默认情况下，RabbitMQBinders使用Spring Boot的 `ConnectionFactory` .因此，它支持RabbitMQ的所有Spring Boot配置选项. （供参考，请参阅[Spring Boot documentation](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#common-application-properties)）. RabbitMQ配置选项使用 `spring.rabbitmq` 前缀.

除Spring Boot选项外，RabbitMQ binder还支持以下属性：

spring.cloud.stream.rabbit.binder.adminAddresses以逗号分隔的RabbitMQ管理插件URL列表.仅在节点包含多个条目时使用.此列表中的每个条目都必须具有spring.rabbitmq.addresses中的相应条目.仅在您使用RabbitMQ群集并希望从承载队列的节点使用时才需要.有关更多信息，请参阅Queue Affinity和LocalizedQueueConnectionFactory.默认值：空. spring.cloud.stream.rabbit.binder.nodes以逗号分隔的RabbitMQ节点名列表.当多个条目用于查找队列所在的服务器地址时.此列表中的每个条目都必须在spring.rabbitmq.addresses中具有相应的条目.仅在您使用RabbitMQ群集并希望从承载队列的节点使用时才需要.有关更多信息，请参阅Queue Affinity和LocalizedQueueConnectionFactory.默认值：空. spring.cloud.stream.rabbit.binder.compressionLevel压缩绑定的压缩级别.请参阅java.util.zip.Deflater.默认值：1（BEST_LEVEL）. spring.cloud.stream.binder.connection-name-prefix用于命名此Binders创建的连接的连接名称前缀.名称是此前缀后跟#n，其中每次打开新连接时n都会递增.默认值：none（默认为Spring AMQP）.

### 39.3.2 RabbitMQ消费者属性

以下属性仅适用于Rabbit使用者，必须以 `spring.cloud.stream.rabbit.bindings.<channelName>.consumer.` 为前缀.

acknowledgeMode确认模式.默认值：AUTO. autoBindDlq是否自动声明DLQ并将其绑定到BindersDLX.默认值：false. bindingRoutingKey用于将队列绑定到交换的路由密钥（如果bindQueue为true）.对于分区目标，将附加 -  <instanceIndex>.默认值：＃. bindQueue是否将队列绑定到目标交换.如果您已设置自己的基础结构并且之前已创建并绑定队列，请将其设置为false.默认值：true. deadLetterQueueName DLQ的名称默认值：prefix destination.dlq deadLetterExchange要分配给队列的DLX.仅当autoBindDlq为true时才相关.默认值：'prefix DLX'deadLetterRoutingKey一个用于分配给队列的死信路由密钥.仅当autoBindDlq为true时才相关.默认值：destination declareExchange是否声明目标的交换.默认值：true. delayedExchange是否将交换声明为延迟消息交换.需要代理上的延迟消息交换插件. x-delayed-type参数设置为exchangeType.默认值：false. dlqDeadLetterExchange如果声明了DLQ，则分配给该队列的DLX.默认值：none dlqDeadLetterRoutingKey如果声明了DLQ，则使用死信路由键分配给该队列.默认值：none dlqExpires删除未使用的死信队列的时间（以毫秒为单位）.默认值：no expiration dlqLazy使用x-queue-mode = lazy参数声明死信队列.请参阅“懒惰队列”.请考虑使用策略而不是此设置，因为使用策略允许更改设置而不删除队列.默认值：false. dlqMaxLength死信队列中的最大消息数.默认值：无限制dlqMaxLengthBytes所有消息的死信队列中的最大总字节数.默认值：无限制dlqMaxPriority死信队列中消息的最大优先级（0-255）.默认值：none dlqTtl声明时应用于死信队列的默认时间（以毫秒为单位）.默认值：无限制durableSubscription订阅是否应该是持久的.仅在设置了组时才有效.默认值：true. exchangeAutoDelete如果declareExchange为true，则是否应自动删除交换（即删除最后一个队列后删除）.默认值：true. exchangeDurable如果declareExchange为true，则交换是否应该是持久的（即，它在代理重启时仍然存在）.默认值：true. exchangeType交换类型：非分区目标的直接，扇出或主题，以及分区目标的直接或主题.默认值：主题.独家是否创造独家消费者.如果这是真的，并发应该是1.通常在需要严格排序时使用，但在发生故障后启用热备用实例.请参阅recoveryInterval，它控制备用实例尝试使用的频率.默认值：false. expires删除未使用的队列多长时间（以毫秒为单位）.默认值：no expiration failedDeclarationRetryInterval如果缺少队列，则尝试从队列中消耗的间隔（以毫秒为单位）.默认值：5000 headerPatterns要从入站消息映射的标头的模式.默认值：['*']（所有Headers）. lazy使用x-queue-mode = lazy参数声明队列.请参阅“懒惰队列”.请考虑使用策略而不是此设置，因为使用策略允许更改设置而不删除队列.默认值：false. maxConcurrency最大消费者数量.默认值：1.maxLength队列中的最大消息数.默认值：无限制maxLengthBytes队列中来自所有消息的最大总字节数.默认值：无限制maxPriority队列中消息的最大优先级（0-255）.默认值：none missingQueuesFatal当无法找到队列时，是否将条件视为致命并停止侦听器容器.默认为false，以便容器继续尝试从队列中使用 - 例如，在使用群集时，托管非HA队列的节点已关闭.默认值：false prefetch预取计数.默认值：1.prefix要添加到目标名称和队列的前缀.默认值：“”. queueDeclarationRetries如果缺少队列，则从队列重试消耗的次数.仅当missingQueuesFatal为真时才相关.否则，容器将无限期地重试.默认值：3 queueNameGroupOnly如果为true，则从名称等于该组的队列中使用.否则队列名称为destination.group.例如，当使用Spring Cloud Stream从现有RabbitMQ队列中使用时，这很有用.默认值：false. recoveryInterval连接恢复尝试之间的时间间隔（以毫秒为单位）.默认值：5000.requeueRejected禁用重试或republishToDlq为false时是否应重新排队传递失败.默认值：false.

republishDeliveryMode当republishToDlq为true时，指定重新发布的消息的传递模式.默认值：DeliveryMode.PERSISTENT republishToDlq默认情况下，拒绝重试后失败的消息.如果配置了死信队列（DLQ），RabbitMQ会将失败的消息（未更改）路由到DLQ.如果设置为true，则绑定程序会使用其他标头将失败的消息重新发布到DLQ，包括异常消息和最终失败原因的堆栈跟踪.默认值：false transacted是否使用事务处理通道.默认值：false. ttl声明时应用于队列的默认时间（以毫秒为单位）.默认值：无限制txSize ack之间的交付数量.默认值：1.

### 39.3.3兔子生产环境者属性

以下属性仅适用于Rabbit生成器，必须以 `spring.cloud.stream.rabbit.bindings.<channelName>.producer.` 为前缀.

autoBindDlq是否自动声明DLQ并将其绑定到BindersDLX.默认值：false. batchingEnabled是否启用生产环境者的消息批处理.根据以下属性（在此列表中的下三个条目中描述）将消息批处理为一条消息：'batchSize'，batchBufferLimit和batchTimeout.有关更多信息，请参阅批处理.默认值：false. batchSize启用批处理时要缓冲的消息数.默认值：100.batchBufferLimit启用批处理时的最大缓冲区大小.默认值：10000.batchTimeout启用批处理时的批处理超时.默认值：5000. bindingRoutingKey用于将队列绑定到交换的路由密钥（如果bindQueue为true）.仅适用于非分区目标.仅在提供requiredGroups时适用，然后仅适用于这些组.默认值：＃. bindQueue是否将队列绑定到目标交换.如果您已设置自己的基础结构并且之前已创建并绑定队列，请将其设置为false.仅在提供requiredGroups时适用，然后仅适用于这些组.默认值：true. compress是否应在发送时压缩数据.默认值：false. deadLetterQueueName DLQ的名称仅在提供requiredGroups时适用，然后仅适用于这些组.默认值：prefix destination.dlq deadLetterExchange要分配给队列的DLX.仅当autoBindDlq为true时才相关.仅在提供requiredGroups时应用，然后仅应用于这些组.默认值：'prefix DLX'deadLetterRoutingKey一个用于分配给队列的死信路由密钥.仅当autoBindDlq为true时才相关.仅在提供requiredGroups时应用，然后仅应用于这些组.默认值：destination declareExchange是否声明目标的交换.默认值：true. delayExpression一个SpEL表达式，用于评估应用于消息的延迟（x-delay标头）.如果交换不是延迟消息交换，则无效.默认值：未设置x延迟标头. delayedExchange是否将交换声明为延迟消息交换.需要代理上的延迟消息交换插件. x-delayed-type参数设置为exchangeType.默认值：false. deliveryMode交付模式.默认值：PERSISTENT. dlqDeadLetterExchange声明DLQ时，将分配给该队列的DLX.仅在提供requiredGroups时应用，然后仅应用于这些组.默认值：none dlqDeadLetterRoutingKey声明DLQ时，将使用死信路由键分配给该队列.仅在提供requiredGroups时应用，然后仅应用于这些组.默认值：none dlqExpires删除未使用的死信队列之前的时间（以毫秒为单位）.仅在提供requiredGroups时应用，然后仅应用于这些组.默认值：no expiration dlqLazy使用x-queue-mode = lazy参数声明死信队列.请参阅“懒惰队列”.考虑使用策略而不是此设置，因为使用策略允许更改设置而不删除队列.仅在提供requiredGroups时应用，然后仅应用于这些组. dlqMaxLength死信队列中的最大消息数.仅在提供requiredGroups时应用，然后仅应用于这些组.默认值：无限制dlqMaxLengthBytes所有消息的死信队列中的最大总字节数.仅在提供requiredGroups时应用，然后仅应用于这些组.默认值：无限制dlqMaxPriority死信队列中消息的最大优先级（0-255）仅在提供requiredGroups时应用，然后仅应用于这些组.默认值：none dlqTtl声明时应用于死信队列的默认时间（以毫秒为单位）.仅在提供requiredGroups时应用，然后仅应用于这些组.默认值：无限制exchangeAutoDelete如果declareExchange为true，则交换是否应该是自动删除（在删除最后一个队列后删除它）.默认值：true. exchangeDurable如果declareExchange为true，则交换是否应该是持久的（在代理重启时仍然存在）.默认值：true. exchangeType交换类型：非分区目标的直接，扇出或主题，以及分区目标的直接或主题.默认值：主题. expires删除未使用的队列之前的时间（以毫秒为单位）.仅在提供requiredGroups时应用，然后仅应用于这些组.默认值：no expiration headerPatterns要映射到出站消息的标头的模式.默认值：['*']（所有Headers）. lazy使用x-queue-mode = lazy参数声明队列.请参阅“懒惰队列”.请考虑使用策略而不是此设置，因为使用策略允许更改设置而不删除队列.仅在提供requiredGroups时应用，然后仅应用于这些组.默认值：false. maxLength队列中的最大消息数.仅在提供requiredGroups时应用，然后仅应用于这些组.默认值：无限制maxLengthBytes队列中所有消息的最大总字节数.仅在提供requiredGroups时适用，然后仅适用于这些组.默认值：无限制maxPriority队列中消息的最大优先级（0-255）.仅在提供requiredGroups时适用，然后仅适用于这些组.默认值：none prefix要添加到目标交换机名称的前缀.默认值：“”. queueNameGroupOnly如果为true，则从名称等于该组的队列中使用.否则队列名称为destination.group.例如，当使用Spring Cloud Stream从现有RabbitMQ队列中使用时，这很有用.仅在提供requiredGroups时应用，然后仅应用于这些组.默认值：false. routingKeyExpression一个SpEL表达式，用于确定发布消息时要使用的路由键.对于固定的路由键，请使用文字表达式，例如属性文件中的routingKeyExpression ='my.routingKey'或者YAML文件中的routingKeyExpression：'''my.routingKey'''.默认值：目标或目标 - 分区目标的<partition>.交易是否使用交易渠道.默认值：false. ttl声明时应用于队列的默认时间（以毫秒为单位）.仅在提供requiredGroups时应用，然后仅应用于这些组.默认值：无限制

> 对于RabbitMQ，内容类型Headers可以由外部应用程序设置. Spring Cloud Stream支持它们作为扩展内部协议的一部分，用于任何类型的传输 - 包括传输，如Kafka（0.11之前），本身不支持头.

## 39.4使用RabbitMQBinders重试

在Binders中启用重试时，将暂停侦听器容器线程以用于配置的任何后退时段.当单个消费者需要严格的订购时，这可能很重要.但是，对于其他用例，它会阻止在该线程上处理其他消息.使用Binders重试的另一种方法是设置死字母以及死信队列（DLQ）上的生存时间以及DLQ本身上的死信配置.有关此处讨论的属性的更多信息，请参阅“[Section 39.3.1, “RabbitMQ Binder Properties”](multi__rabbitmq_binder.html#rabbit-binder-properties)”.您可以使用以下示例配置来启用此功能：

- Set  `autoBindDlq` 至 `true` .Binders创建DLQ. （可选）您可以在 `deadLetterQueueName` 中指定名称.

- Set  `dlqTtl` 到您想要在重新开始之间等待的退避时间.

- Set  `dlqDeadLetterExchange` 到默认交换.来自DLQ的过期消息将路由到原始队列，因为缺省 `deadLetterRoutingKey` 是队列名称（ `destination.group` ）.通过将属性设置为无值来实现设置为默认交换，如下一个示例所示.

要强制消息为dead-lettered，请抛出 `AmqpRejectAndDontRequeueException` 或将 `requeueRejected` 设置为 `true` （默认值）并抛出任何异常.

循环继续没有结束，这对于瞬态问题很好，但是你可能想在经过一些尝试后放弃.幸运的是，RabbitMQ提供了 `x-death` 标头，可以让您确定已经发生了多少次循环.

要在放弃后确认消息，请抛出 `ImmediateAcknowledgeAmqpException` .

### 39.4.1全部放在一起

以下配置创建交换 `myDestination` ，其队列 `myDestination.consumerGroup` 绑定到具有通配符路由键 `#` 的主题交换：

```java
---
spring.cloud.stream.bindings.input.destination=myDestination
spring.cloud.stream.bindings.input.group=consumerGroup
#disable binder retries
spring.cloud.stream.bindings.input.consumer.max-attempts=1
#dlx/dlq setup
spring.cloud.stream.rabbit.bindings.input.consumer.auto-bind-dlq=true
spring.cloud.stream.rabbit.bindings.input.consumer.dlq-ttl=5000
spring.cloud.stream.rabbit.bindings.input.consumer.dlq-dead-letter-exchange=
---
```

此配置创建绑定到直接交换（ `DLX` ）的DLQ，路由密钥为 `myDestination.consumerGroup` .当邮件被拒绝时，它们将被路由到DLQ. 5秒后，消息将过期，并使用队列名称作为路由密钥路由到原始队列，如以下示例所示：

**Spring Boot application.** 

```java
@SpringBootApplication
@EnableBinding(Sink.class)
public class XDeathApplication {

public static void main(String[] args) {
SpringApplication.run(XDeathApplication.class, args);
}

@StreamListener(Sink.INPUT)
public void listen(String in, @Header(name = "x-death", required = false) Map<?,?> death) {
if (death != null && death.get("count").equals(3L)) {
// giving up - don't send to DLX
throw new ImmediateAcknowledgeAmqpException("Failed after 4 attempts");
}
throw new AmqpRejectAndDontRequeueException("failed");
}

}
```

请注意 `x-death` 标头中的count属性是 `Long` .

## 39.5错误Channels

从版本1.3开始，Binders无条件地为每个使用者目标向错误通道发送异常，并且还可以配置为将异步生成器发送失败发送到错误通道.有关详细信息，请参阅“[???]()”.

RabbitMQ有两种类型的发送失败：

- Returned消息，

- 否定承认[Publisher Confirms](https://www.rabbitmq.com/confirms.html).

后者很少见.根据RabbitMQ文档，“只有在负责队列的Erlang进程中发生内部错误时才会传递[A nack].”

除了启用生成器错误通道（如“[???]()”中所述）之外，如果连接工厂配置正确，RabbitMQBinders仅向通道发送消息，如下所示：

-  `ccf.setPublisherConfirms(true);` 

-  `ccf.setPublisherReturns(true);` 

将Spring Boot配置用于连接工厂时，请设置以下属性：

-  `spring.rabbitmq.publisher-confirms` 

-  `spring.rabbitmq.publisher-returns` 

返回消息的 `ErrorMessage` 的有效负载是 `ReturnedAmqpMessageException` ，具有以下属性：

-  `failedMessage` ：无法发送的Spring天消息 `Message<?>` .

-  `amqpMessage` ：原始的spring-amqp  `Message` .

-  `replyCode` ：表示失败原因的整数值（例如，312  - 无路径）.

-  `replyText` ：指示失败原因的文本值（例如， `NO_ROUTE` ）.

-  `exchange` ：邮件发布的交换.

-  `routingKey` ：发布消息时使用的路由密钥.

对于否定确认的确认，有效负载是 `NackedAmqpMessageException` ，具有以下属性：

-  `failedMessage` ：无法发送的spring-messaging  `Message<?>` .

-  `nackReason` ：原因（如果可用 - 您可能需要检查代理日志以获取更多信息）.

没有自动处理这些异常（例如发送到[dead-letter queue](multi__rabbitmq_binder.html#rabbit-dlq-processing)）.您可以使用自己的Spring Integration流程来使用这些异常.

## 39.6死信队列处理

因为您无法预测用户将如何处理死信消息，所以框架不提供任何标准机制来处理它们.如果死字法的原因是暂时的，您可能希望将消息路由回原始队列.但是，如果问题是一个永久性问题，那么可能会导致无限循环.以下Spring Boot应用程序显示了如何将这些消息路由回原始队列但在三次尝试后将它们移动到第三个“停车场”队列的示例.第二个示例使用[RabbitMQ Delayed Message Exchange](https://www.rabbitmq.com/blog/2015/04/16/scheduling-messages-with-rabbitmq/)为重新排队的消息引入延迟.在此示例中，每次尝试的延迟都会增加.这些示例使用 `@RabbitListener` 来接收来自DLQ的消息.您也可以在批处理中使用 `RabbitTemplate.receive()` .

这些示例假定原始目标是 `so8400in` ，而使用者组是 `so8400` .

### 39.6.1非分区目的地

前两个示例适用于目标为 **not** 分区的情况：

```java
@SpringBootApplication
public class ReRouteDlqApplication {

private static final String ORIGINAL_QUEUE = "so8400in.so8400";

private static final String DLQ = ORIGINAL_QUEUE + ".dlq";

private static final String PARKING_LOT = ORIGINAL_QUEUE + ".parkingLot";

private static final String X_RETRIES_HEADER = "x-retries";

public static void main(String[] args) throws Exception {
ConfigurableApplicationContext context = SpringApplication.run(ReRouteDlqApplication.class, args);
System.out.println("Hit enter to terminate");
System.in.read();
context.close();
}

@Autowired
private RabbitTemplate rabbitTemplate;

@RabbitListener(queues = DLQ)
public void rePublish(Message failedMessage) {
Integer retriesHeader = (Integer) failedMessage.getMessageProperties().getHeaders().get(X_RETRIES_HEADER);
if (retriesHeader == null) {
retriesHeader = Integer.valueOf(0);
}
if (retriesHeader < 3) {
failedMessage.getMessageProperties().getHeaders().put(X_RETRIES_HEADER, retriesHeader + 1);
this.rabbitTemplate.send(ORIGINAL_QUEUE, failedMessage);
}
else {
this.rabbitTemplate.send(PARKING_LOT, failedMessage);
}
}

@Bean
public Queue parkingLot() {
return new Queue(PARKING_LOT);
}

}
```

```java
@SpringBootApplication
public class ReRouteDlqApplication {

private static final String ORIGINAL_QUEUE = "so8400in.so8400";

private static final String DLQ = ORIGINAL_QUEUE + ".dlq";

private static final String PARKING_LOT = ORIGINAL_QUEUE + ".parkingLot";

private static final String X_RETRIES_HEADER = "x-retries";

private static final String DELAY_EXCHANGE = "dlqReRouter";

public static void main(String[] args) throws Exception {
ConfigurableApplicationContext context = SpringApplication.run(ReRouteDlqApplication.class, args);
System.out.println("Hit enter to terminate");
System.in.read();
context.close();
}

@Autowired
private RabbitTemplate rabbitTemplate;

@RabbitListener(queues = DLQ)
public void rePublish(Message failedMessage) {
Map<String, Object> headers = failedMessage.getMessageProperties().getHeaders();
Integer retriesHeader = (Integer) headers.get(X_RETRIES_HEADER);
if (retriesHeader == null) {
retriesHeader = Integer.valueOf(0);
}
if (retriesHeader < 3) {
headers.put(X_RETRIES_HEADER, retriesHeader + 1);
headers.put("x-delay", 5000 * retriesHeader);
this.rabbitTemplate.send(DELAY_EXCHANGE, ORIGINAL_QUEUE, failedMessage);
}
else {
this.rabbitTemplate.send(PARKING_LOT, failedMessage);
}
}

@Bean
public DirectExchange delayExchange() {
DirectExchange exchange = new DirectExchange(DELAY_EXCHANGE);
exchange.setDelayed(true);
return exchange;
}

@Bean
public Binding bindOriginalToDelay() {
return BindingBuilder.bind(new Queue(ORIGINAL_QUEUE)).to(delayExchange()).with(ORIGINAL_QUEUE);
}

@Bean
public Queue parkingLot() {
return new Queue(PARKING_LOT);
}

}
```

### 39.6.2分区目的地

对于分区目标，所有分区都有一个DLQ.我们从标头中确定原始队列.

#### republishToDlq = FALSE

当 `republishToDlq` 为 `false` 时，RabbitMQ将消息发布到DLX / DLQ，其中包含有关原始目标的信息的 `x-death` 标头，如以下示例所示：

```java
@SpringBootApplication
public class ReRouteDlqApplication {

	private static final String ORIGINAL_QUEUE = "so8400in.so8400";

	private static final String DLQ = ORIGINAL_QUEUE + ".dlq";

	private static final String PARKING_LOT = ORIGINAL_QUEUE + ".parkingLot";

	private static final String X_DEATH_HEADER = "x-death";

	private static final String X_RETRIES_HEADER = "x-retries";

	public static void main(String[] args) throws Exception {
		ConfigurableApplicationContext context = SpringApplication.run(ReRouteDlqApplication.class, args);
		System.out.println("Hit enter to terminate");
		System.in.read();
		context.close();
	}

	@Autowired
	private RabbitTemplate rabbitTemplate;

	@SuppressWarnings("unchecked")
	@RabbitListener(queues = DLQ)
	public void rePublish(Message failedMessage) {
		Map<String, Object> headers = failedMessage.getMessageProperties().getHeaders();
		Integer retriesHeader = (Integer) headers.get(X_RETRIES_HEADER);
		if (retriesHeader == null) {
			retriesHeader = Integer.valueOf(0);
		}
		if (retriesHeader < 3) {
			headers.put(X_RETRIES_HEADER, retriesHeader + 1);
			List<Map<String, ?>> xDeath = (List<Map<String, ?>>) headers.get(X_DEATH_HEADER);
			String exchange = (String) xDeath.get(0).get("exchange");
			List<String> routingKeys = (List<String>) xDeath.get(0).get("routing-keys");
			this.rabbitTemplate.send(exchange, routingKeys.get(0), failedMessage);
		}
		else {
			this.rabbitTemplate.send(PARKING_LOT, failedMessage);
		}
	}

	@Bean
	public Queue parkingLot() {
		return new Queue(PARKING_LOT);
	}

}
```

#### republishToDlq =真

当 `republishToDlq` 是 `true` 时，重新发布恢复器会将原始交换和路由密钥添加到标头，如以下示例所示：

```java
@SpringBootApplication
public class ReRouteDlqApplication {

	private static final String ORIGINAL_QUEUE = "so8400in.so8400";

	private static final String DLQ = ORIGINAL_QUEUE + ".dlq";

	private static final String PARKING_LOT = ORIGINAL_QUEUE + ".parkingLot";

	private static final String X_RETRIES_HEADER = "x-retries";

	private static final String X_ORIGINAL_EXCHANGE_HEADER = RepublishMessageRecoverer.X_ORIGINAL_EXCHANGE;

	private static final String X_ORIGINAL_ROUTING_KEY_HEADER = RepublishMessageRecoverer.X_ORIGINAL_ROUTING_KEY;

	public static void main(String[] args) throws Exception {
		ConfigurableApplicationContext context = SpringApplication.run(ReRouteDlqApplication.class, args);
		System.out.println("Hit enter to terminate");
		System.in.read();
		context.close();
	}

	@Autowired
	private RabbitTemplate rabbitTemplate;

	@RabbitListener(queues = DLQ)
	public void rePublish(Message failedMessage) {
		Map<String, Object> headers = failedMessage.getMessageProperties().getHeaders();
		Integer retriesHeader = (Integer) headers.get(X_RETRIES_HEADER);
		if (retriesHeader == null) {
			retriesHeader = Integer.valueOf(0);
		}
		if (retriesHeader < 3) {
			headers.put(X_RETRIES_HEADER, retriesHeader + 1);
			String exchange = (String) headers.get(X_ORIGINAL_EXCHANGE_HEADER);
			String originalRoutingKey = (String) headers.get(X_ORIGINAL_ROUTING_KEY_HEADER);
			this.rabbitTemplate.send(exchange, originalRoutingKey, failedMessage);
		}
		else {
			this.rabbitTemplate.send(PARKING_LOT, failedMessage);
		}
	}

	@Bean
	public Queue parkingLot() {
		return new Queue(PARKING_LOT);
	}

}
```

## 39.7使用RabbitMQ Binder进行分区

RabbitMQ本身不支持分区.

有时，将数据发送到特定分区是有利的 - 例如，当您要严格订购消息处理时，特定客户的所有消息都应该转到同一分区.

`RabbitMessageChannelBinder` 通过将每个分区的队列绑定到目标交换来提供分区.

以下Java和YAML示例显示如何配置生成器：

**Producer.** 

```java
@SpringBootApplication
@EnableBinding(Source.class)
public class RabbitPartitionProducerApplication {

private static final Random RANDOM = new Random(System.currentTimeMillis());

private static final String[] data = new String[] {
"abc1", "def1", "qux1",
"abc2", "def2", "qux2",
"abc3", "def3", "qux3",
"abc4", "def4", "qux4",
};

public static void main(String[] args) {
new SpringApplicationBuilder(RabbitPartitionProducerApplication.class)
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
destination: partitioned.destination
producer:
partitioned: true
partition-key-expression: headers['partitionKey']
partition-count: 2
required-groups:
- myGroup
```

> 前面示例中的配置使用默认分区（ `key.hashCode() % partitionCount` ）.根据键值，这可能会或可能不会提供适当平衡的算法.你可以覆盖这是默认使用 `partitionSelectorExpression` 或 `partitionSelectorClass` 属性.

以下配置提供了主题交换：

图像/部分exchange.png

以下队列绑定到该交换：

图像/部分queues.png

以下绑定将队列关联到交换：

图像/部分bindings.png

以下Java和YAML示例继续前面的示例，并说明如何配置使用者：

**Consumer.** 

```java
@SpringBootApplication
@EnableBinding(Sink.class)
public class RabbitPartitionConsumerApplication {

public static void main(String[] args) {
new SpringApplicationBuilder(RabbitPartitionConsumerApplication.class)
.web(false)
.run(args);
}

@StreamListener(Sink.INPUT)
public void listen(@Payload String in, @Header(AmqpHeaders.CONSUMER_QUEUE) String queue) {
System.out.println(in + " received from queue " + queue);
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
destination: partitioned.destination
group: myGroup
consumer:
partitioned: true
instance-index: 0
```

|图片/ important.png |重要|
| ---- | ---- |
|  `RabbitMessageChannelBinder` 不支持动态缩放.每个分区必须至少有一个消费者.消费者的 `instanceIndex` 用于指示消耗了哪个分区. Cloud Foundry等平台只能有一个带有 `instanceIndex` 的实例. |

