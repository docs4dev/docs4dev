## 32.应用程序间通信

Spring Cloud Stream支持应用程序之间的通信.跨应用程序通信是一个复杂的问题，涉及多个问题，如以下主题中所述：

_0009_“[Section 32.1, “Connecting Multiple Application Instances”](multi__inter_application_communication.html#spring-cloud-stream-overview-connecting-multiple-application-instances)”
_0009_“[Section 32.2, “Instance Index and Instance Count”](multi__inter_application_communication.html#spring-cloud-stream-overview-instance-index-instance-count)”
_0009_“[Section 32.3, “Partitioning”](multi__inter_application_communication.html#spring-cloud-stream-overview-partitioning)”

## 32.1连接多个应用程序实例

虽然Spring Cloud Stream使单个Spring Boot应用程序可以轻松连接到消息传递系统，但Spring Cloud Stream的典型场景是创建多应用程序管道，其中微服务应用程序相互发送数据.您可以通过关联“相邻”应用程序的输入和输出目标来实现此方案.

假设一个设计要求Time Source应用程序将数据发送到Log Sink应用程序.您可以使用名为 `ticktock` 的公共目标两个应用程序中的绑定.

时间源（具有通道名称 `output` ）将设置以下属性：

```java
spring.cloud.stream.bindings.output.destination=ticktock
```

Log Sink（具有通道名称 `input` ）将设置以下属性：

```java
spring.cloud.stream.bindings.input.destination=ticktock
```

## 32.2实例索引和实例计数

在扩展Spring Cloud Stream应用程序时，每个实例都可以接收有关同一应用程序存在多少其他实例以及它自己的实例索引的信息. Spring Cloud Stream通过 `spring.cloud.stream.instanceCount` 和 `spring.cloud.stream.instanceIndex` 属性执行此操作.例如，如果有三个HDFS接收器应用程序实例，则所有三个实例都将 `spring.cloud.stream.instanceCount` 设置为 `3` ，并且各个应用程序的 `spring.cloud.stream.instanceIndex` 分别设置为 `0` ， `1` 和 `2` .

当Spring Cloud Stream应用程序通过Spring Cloud Data Flow部署时，这些属性会自动配置;当Spring Cloud Stream应用程序独立启动时，必须正确设置这些属性.默认情况下， `spring.cloud.stream.instanceCount` 是 `1` ， `spring.cloud.stream.instanceIndex` 是 `0` .

在按比例放大的方案中，正确配置这两个属性对于解决分区行为（见下文）非常重要，并且某些绑定程序（例如，Kafka绑定程序）始终需要这两个属性，以确保数据在多个消费者实例之间正确分割.

## 32.3分区

Spring Cloud Stream中的分区包含两个任务：

_0009_“[Section 32.3.1, “Configuring Output Bindings for Partitioning”](multi__inter_application_communication.html#spring-cloud-stream-overview-configuring-output-bindings-partitioning)”
_0009_“[Section 32.3.2, “Configuring Input Bindings for Partitioning”](multi__inter_application_communication.html#spring-cloud-stream-overview-configuring-input-bindings-partitioning)”

### 32.3.1配置分区的输出绑定

您可以通过设置其中一个 `partitionKeyExpression` 或 `partitionKeyExtractorName` 属性及其 `partitionCount` 属性来配置输出绑定以发送分区数据.

例如，以下是有效且典型的配置：

```java
spring.cloud.stream.bindings.output.producer.partitionKeyExpression=payload.id
spring.cloud.stream.bindings.output.producer.partitionCount=5
```

基于该示例配置，使用以下逻辑将数据发送到目标分区.

根据 `partitionKeyExpression` 计算发送到分区输出通道的每条消息的分区键值.  `partitionKeyExpression` 是一个SpEL表达式，它根据出站消息进行计算以提取分区键.

如果SpEL表达式不足以满足您的需要，您可以通过提供 `org.springframework.cloud.stream.binder.PartitionKeyExtractorStrategy` 的实现并将其配置为bean（通过使用 `@Bean` 注释）来计算分区键值.如果在应用程序上下文中有多个类型为 `org.springframework.cloud.stream.binder.PartitionKeyExtractorStrategy` 的bean，则可以通过使用 `partitionKeyExtractorName` 属性指定其名称来进一步过滤它，如以下示例所示：

```java
--spring.cloud.stream.bindings.output.producer.partitionKeyExtractorName=customPartitionKeyExtractor
--spring.cloud.stream.bindings.output.producer.partitionCount=5
. . .
@Bean
public CustomPartitionKeyExtractorClass customPartitionKeyExtractor() {
return new CustomPartitionKeyExtractorClass();
}
```

> 在以前版本的Spring Cloud Stream中，您可以通过设置 `spring.cloud.stream.bindings.output.producer.partitionKeyExtractorClass` 属性来指定 `org.springframework.cloud.stream.binder.PartitionKeyExtractorStrategy` 的实现.从版本2.0开始，不推荐使用此属性，并且将在以后的版本中删除对该属性的支持.

一旦计算了消息密钥，分区选择过程就将目标分区确定为 `0` 和 `partitionCount - 1` 之间的值.适用于大多数情况的默认计算基于以下公式： `key.hashCode() % partitionCount` .这可以在绑定上自定义，方法是通过设置要根据'key'（通过 `partitionSelectorExpression` 属性）计算的SpEL表达式，或者将 `org.springframework.cloud.stream.binder.PartitionSelectorStrategy` 的实现配置为bean（通过使用@Bean注释）.与 `PartitionKeyExtractorStrategy` 类似，当应用程序上下文中有多个此类bean时，可以使用 `spring.cloud.stream.bindings.output.producer.partitionSelectorName` 属性进一步过滤它，如以下示例所示：

```java
--spring.cloud.stream.bindings.output.producer.partitionSelectorName=customPartitionSelector
. . .
@Bean
public CustomPartitionSelectorClass customPartitionSelector() {
return new CustomPartitionSelectorClass();
}
```

> 在以前版本的Spring Cloud Stream中，您可以通过设置 `spring.cloud.stream.bindings.output.producer.partitionSelectorClass` 属性来指定 `org.springframework.cloud.stream.binder.PartitionSelectorStrategy` 的实现.从版本2.0开始，不推荐使用此属性，并且将在以后的版本中删除对该属性的支持.

### 32.3.2配置分区的输入绑定

输入绑定（通道名称为 `input` ）配置为通过设置其 `partitioned` 属性以及应用程序本身的 `instanceIndex` 和 `instanceCount` 属性来接收分区数据，如以下示例所示：

```java
spring.cloud.stream.bindings.input.consumer.partitioned=true
spring.cloud.stream.instanceIndex=3
spring.cloud.stream.instanceCount=5
```

`instanceCount` 值表示应在其间分区数据的应用程序实例的总数.  `instanceIndex` 必须是多个实例中的唯一值，其值介于 `0` 和 `instanceCount - 1` 之间.实例索引可帮助每个应用程序实例识别从中接收数据的唯一分区.Binders需要使用不支持本机分区的技术.例如，使用RabbitMQ，每个分区都有一个队列，队列名称包含实例索引.对于Kafka，如果 `autoRebalanceEnabled` 是 `true` （默认），Kafka负责跨实例分发分区，并且不需要这些属性.如果 `autoRebalanceEnabled` 设置为false，则绑定程序使用 `instanceCount` 和 `instanceIndex` 来确定实例所订阅的分区（您必须至少具有与实例一样多的分区）.Binders分配分区而不是Kafka.如果您希望始终转到特定分区的消息，这可能很有用相同的实例.当Binders配置需要它们时，重要的是正确设置两个值以确保消耗所有数据并且应用程序实例接收互斥数据集.

虽然在独立的情况下使用多个实例进行分区数据处理的情况可能很复杂，但Spring Cloud Dataflow可以通过正确填充输入和输出值并让您依赖运行时基础结构来显着简化流程.提供有关实例索引和实例计数的信息.

