## 32. Inter-Application Communication

Spring Cloud Stream enables communication between applications. Inter-application communication is a complex issue spanning several concerns, as described in the following topics:

- “[Section 32.1, “Connecting Multiple Application Instances”](multi__inter_application_communication.html#spring-cloud-stream-overview-connecting-multiple-application-instances)”

- “[Section 32.2, “Instance Index and Instance Count”](multi__inter_application_communication.html#spring-cloud-stream-overview-instance-index-instance-count)”

- “[Section 32.3, “Partitioning”](multi__inter_application_communication.html#spring-cloud-stream-overview-partitioning)”

## 32.1 Connecting Multiple Application Instances

While Spring Cloud Stream makes it easy for individual Spring Boot applications to connect to messaging systems, the typical scenario for Spring Cloud Stream is the creation of multi-application pipelines, where microservice applications send data to each other. You can achieve this scenario by correlating the input and output destinations of “adjacent” applications.

Suppose a design calls for the Time Source application to send data to the Log Sink application. You could use a common destination named  `ticktock`  for bindings within both applications.

Time Source (that has the channel name  `output` ) would set the following property:

```java
spring.cloud.stream.bindings.output.destination=ticktock
```

Log Sink (that has the channel name  `input` ) would set the following property:

```java
spring.cloud.stream.bindings.input.destination=ticktock
```

## 32.2 Instance Index and Instance Count

When scaling up Spring Cloud Stream applications, each instance can receive information about how many other instances of the same application exist and what its own instance index is. Spring Cloud Stream does this through the  `spring.cloud.stream.instanceCount`  and  `spring.cloud.stream.instanceIndex`  properties. For example, if there are three instances of a HDFS sink application, all three instances have  `spring.cloud.stream.instanceCount`  set to  `3` , and the individual applications have  `spring.cloud.stream.instanceIndex`  set to  `0` ,  `1` , and  `2` , respectively.

When Spring Cloud Stream applications are deployed through Spring Cloud Data Flow, these properties are configured automatically; when Spring Cloud Stream applications are launched independently, these properties must be set correctly. By default,  `spring.cloud.stream.instanceCount`  is  `1` , and  `spring.cloud.stream.instanceIndex`  is  `0` .

In a scaled-up scenario, correct configuration of these two properties is important for addressing partitioning behavior (see below) in general, and the two properties are always required by certain binders (for example, the Kafka binder) in order to ensure that data are split correctly across multiple consumer instances.

## 32.3 Partitioning

Partitioning in Spring Cloud Stream consists of two tasks:

- “[Section 32.3.1, “Configuring Output Bindings for Partitioning”](multi__inter_application_communication.html#spring-cloud-stream-overview-configuring-output-bindings-partitioning)”

- “[Section 32.3.2, “Configuring Input Bindings for Partitioning”](multi__inter_application_communication.html#spring-cloud-stream-overview-configuring-input-bindings-partitioning)”

### 32.3.1 Configuring Output Bindings for Partitioning

You can configure an output binding to send partitioned data by setting one and only one of its  `partitionKeyExpression`  or  `partitionKeyExtractorName`  properties, as well as its  `partitionCount`  property.

For example, the following is a valid and typical configuration:

```java
spring.cloud.stream.bindings.output.producer.partitionKeyExpression=payload.id
spring.cloud.stream.bindings.output.producer.partitionCount=5
```

Based on that example configuration, data is sent to the target partition by using the following logic.

A partition key’s value is calculated for each message sent to a partitioned output channel based on the  `partitionKeyExpression` . The  `partitionKeyExpression`  is a SpEL expression that is evaluated against the outbound message for extracting the partitioning key.

If a SpEL expression is not sufficient for your needs, you can instead calculate the partition key value by providing an implementation of  `org.springframework.cloud.stream.binder.PartitionKeyExtractorStrategy`  and configuring it as a bean (by using the  `@Bean`  annotation). If you have more then one bean of type  `org.springframework.cloud.stream.binder.PartitionKeyExtractorStrategy`  available in the Application Context, you can further filter it by specifying its name with the  `partitionKeyExtractorName`  property, as shown in the following example:

```java
--spring.cloud.stream.bindings.output.producer.partitionKeyExtractorName=customPartitionKeyExtractor
--spring.cloud.stream.bindings.output.producer.partitionCount=5
. . .
@Bean
public CustomPartitionKeyExtractorClass customPartitionKeyExtractor() {
return new CustomPartitionKeyExtractorClass();
}
```

> In previous versions of Spring Cloud Stream, you could specify the implementation of  `org.springframework.cloud.stream.binder.PartitionKeyExtractorStrategy`  by setting the  `spring.cloud.stream.bindings.output.producer.partitionKeyExtractorClass`  property. Since version 2.0, this property is deprecated, and support for it will be removed in a future version.

Once the message key is calculated, the partition selection process determines the target partition as a value between  `0`  and  `partitionCount - 1` . The default calculation, applicable in most scenarios, is based on the following formula:  `key.hashCode() % partitionCount` . This can be customized on the binding, either by setting a SpEL expression to be evaluated against the 'key' (through the  `partitionSelectorExpression`  property) or by configuring an implementation of  `org.springframework.cloud.stream.binder.PartitionSelectorStrategy`  as a bean (by using the @Bean annotation). Similar to the  `PartitionKeyExtractorStrategy` , you can further filter it by using the  `spring.cloud.stream.bindings.output.producer.partitionSelectorName`  property when more than one bean of this type is available in the Application Context, as shown in the following example:

```java
--spring.cloud.stream.bindings.output.producer.partitionSelectorName=customPartitionSelector
. . .
@Bean
public CustomPartitionSelectorClass customPartitionSelector() {
return new CustomPartitionSelectorClass();
}
```

> In previous versions of Spring Cloud Stream you could specify the implementation of  `org.springframework.cloud.stream.binder.PartitionSelectorStrategy`  by setting the  `spring.cloud.stream.bindings.output.producer.partitionSelectorClass`  property. Since version 2.0, this property is deprecated and support for it will be removed in a future version.

### 32.3.2 Configuring Input Bindings for Partitioning

An input binding (with the channel name  `input` ) is configured to receive partitioned data by setting its  `partitioned`  property, as well as the  `instanceIndex`  and  `instanceCount`  properties on the application itself, as shown in the following example:

```java
spring.cloud.stream.bindings.input.consumer.partitioned=true
spring.cloud.stream.instanceIndex=3
spring.cloud.stream.instanceCount=5
```

The  `instanceCount`  value represents the total number of application instances between which the data should be partitioned. The  `instanceIndex`  must be a unique value across the multiple instances, with a value between  `0`  and  `instanceCount - 1` . The instance index helps each application instance to identify the unique partition(s) from which it receives data. It is required by binders using technology that does not support partitioning natively. For example, with RabbitMQ, there is a queue for each partition, with the queue name containing the instance index. With Kafka, if  `autoRebalanceEnabled`  is  `true`  (default), Kafka takes care of distributing partitions across instances, and these properties are not required. If  `autoRebalanceEnabled`  is set to false, the  `instanceCount`  and  `instanceIndex`  are used by the binder to determine which partition(s) the instance subscribes to (you must have at least as many partitions as there are instances). The binder allocates the partitions instead of Kafka. This might be useful if you want messages for a particular partition to always go to the same instance. When a binder configuration requires them, it is important to set both values correctly in order to ensure that all of the data is consumed and that the application instances receive mutually exclusive datasets.

While a scenario in which using multiple instances for partitioned data processing may be complex to set up in a standalone case, Spring Cloud Dataflow can simplify the process significantly by populating both the input and output values correctly and by letting you rely on the runtime infrastructure to provide information about the instance index and instance count.

