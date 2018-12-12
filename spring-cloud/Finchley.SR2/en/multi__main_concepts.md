## 26. Main Concepts

Spring Cloud Stream provides a number of abstractions and primitives that simplify the writing of message-driven microservice applications. This section gives an overview of the following:

- [Spring Cloud Stream’s application model](multi__main_concepts.html#spring-cloud-stream-overview-application-model)

- [Section 26.2, “The Binder Abstraction”](multi__main_concepts.html#spring-cloud-stream-overview-binder-abstraction)

- [Persistent publish-subscribe support](multi__main_concepts.html#spring-cloud-stream-overview-persistent-publish-subscribe-support)

- [Consumer group support](multi__main_concepts.html#consumer-groups)

- [Partitioning support](multi__main_concepts.html#partitioning)

- [A pluggable Binder SPI](multi_spring-cloud-stream-overview-binders.html#spring-cloud-stream-overview-binder-api)

## 26.1 Application Model

A Spring Cloud Stream application consists of a middleware-neutral core. The application communicates with the outside world through input and output channels injected into it by Spring Cloud Stream. Channels are connected to external brokers through middleware-specific Binder implementations.

**Figure 26.1. Spring Cloud Stream Application** 

images/SCSt-with-binder.png

### 26.1.1 Fat JAR

Spring Cloud Stream applications can be run in stand-alone mode from your IDE for testing. To run a Spring Cloud Stream application in production, you can create an executable (or “fat”) JAR by using the standard Spring Boot tooling provided for Maven or Gradle. See the [Spring Boot Reference Guide](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-build.html#howto-create-an-executable-jar-with-maven) for more details.

## 26.2 The Binder Abstraction

Spring Cloud Stream provides Binder implementations for [Kafka](https://github.com/spring-cloud/spring-cloud-stream/tree/master/spring-cloud-stream-binders/spring-cloud-stream-binder-kafka) and [Rabbit MQ](https://github.com/spring-cloud/spring-cloud-stream/tree/master/spring-cloud-stream-binders/spring-cloud-stream-binder-rabbit). Spring Cloud Stream also includes a [TestSupportBinder](https://github.com/spring-cloud/spring-cloud-stream/blob/master/spring-cloud-stream-test-support/src/main/java/org/springframework/cloud/stream/test/binder/TestSupportBinder.java), which leaves a channel unmodified so that tests can interact with channels directly and reliably assert on what is received. You can also use the extensible API to write your own Binder.

Spring Cloud Stream uses Spring Boot for configuration, and the Binder abstraction makes it possible for a Spring Cloud Stream application to be flexible in how it connects to middleware. For example, deployers can dynamically choose, at runtime, the destinations (such as the Kafka topics or RabbitMQ exchanges) to which channels connect. Such configuration can be provided through external configuration properties and in any form supported by Spring Boot (including application arguments, environment variables, and  `application.yml`  or  `application.properties`  files). In the sink example from the [Chapter 25, Introducing Spring Cloud Stream](multi_spring-cloud-stream-overview-introducing.html) section, setting the  `spring.cloud.stream.bindings.input.destination`  application property to  `raw-sensor-data`  causes it to read from the  `raw-sensor-data`  Kafka topic or from a queue bound to the  `raw-sensor-data`  RabbitMQ exchange.

Spring Cloud Stream automatically detects and uses a binder found on the classpath. You can use different types of middleware with the same code. To do so, include a different binder at build time. For more complex use cases, you can also package multiple binders with your application and have it choose the binder( and even whether to use different binders for different channels) at runtime.

## 26.3 Persistent Publish-Subscribe Support

Communication between applications follows a publish-subscribe model, where data is broadcast through shared topics. This can be seen in the following figure, which shows a typical deployment for a set of interacting Spring Cloud Stream applications.

**Figure 26.2. Spring Cloud Stream Publish-Subscribe** 

images/SCSt-sensors.png

Data reported by sensors to an HTTP endpoint is sent to a common destination named  `raw-sensor-data` . From the destination, it is independently processed by a microservice application that computes time-windowed averages and by another microservice application that ingests the raw data into HDFS (Hadoop Distributed File System). In order to process the data, both applications declare the topic as their input at runtime.

The publish-subscribe communication model reduces the complexity of both the producer and the consumer and lets new applications be added to the topology without disruption of the existing flow. For example, downstream from the average-calculating application, you can add an application that calculates the highest temperature values for display and monitoring. You can then add another application that interprets the same flow of averages for fault detection. Doing all communication through shared topics rather than point-to-point queues reduces coupling between microservices.

While the concept of publish-subscribe messaging is not new, Spring Cloud Stream takes the extra step of making it an opinionated choice for its application model. By using native middleware support, Spring Cloud Stream also simplifies use of the publish-subscribe model across different platforms.

## 26.4 Consumer Groups

While the publish-subscribe model makes it easy to connect applications through shared topics, the ability to scale up by creating multiple instances of a given application is equally important. When doing so, different instances of an application are placed in a competing consumer relationship, where only one of the instances is expected to handle a given message.

Spring Cloud Stream models this behavior through the concept of a consumer group. (Spring Cloud Stream consumer groups are similar to and inspired by Kafka consumer groups.) Each consumer binding can use the  `spring.cloud.stream.bindings.<channelName>.group`  property to specify a group name. For the consumers shown in the following figure, this property would be set as  `spring.cloud.stream.bindings.<channelName>.group=hdfsWrite`  or  `spring.cloud.stream.bindings.<channelName>.group=average` .

**Figure 26.3. Spring Cloud Stream Consumer Groups** 

images/SCSt-groups.png

All groups that subscribe to a given destination receive a copy of published data, but only one member of each group receives a given message from that destination. By default, when a group is not specified, Spring Cloud Stream assigns the application to an anonymous and independent single-member consumer group that is in a publish-subscribe relationship with all other consumer groups.

## 26.5 Consumer Types

Two types of consumer are supported:

- Message-driven (sometimes referred to as Asynchronous)

- Polled (sometimes referred to as Synchronous)

Prior to version 2.0, only asynchronous consumers were supported. A message is delivered as soon as it is available and a thread is available to process it.

When you wish to control the rate at which messages are processed, you might want to use a synchronous consumer.

### 26.5.1 Durability

Consistent with the opinionated application model of Spring Cloud Stream, consumer group subscriptions are durable. That is, a binder implementation ensures that group subscriptions are persistent and that, once at least one subscription for a group has been created, the group receives messages, even if they are sent while all applications in the group are stopped.

> Anonymous subscriptions are non-durable by nature. For some binder implementations (such as RabbitMQ), it is possible to have non-durable group subscriptions.

In general, it is preferable to always specify a consumer group when binding an application to a given destination. When scaling up a Spring Cloud Stream application, you must specify a consumer group for each of its input bindings. Doing so prevents the application’s instances from receiving duplicate messages (unless that behavior is desired, which is unusual).

## 26.6 Partitioning Support

Spring Cloud Stream provides support for partitioning data between multiple instances of a given application. In a partitioned scenario, the physical communication medium (such as the broker topic) is viewed as being structured into multiple partitions. One or more producer application instances send data to multiple consumer application instances and ensure that data identified by common characteristics are processed by the same consumer instance.

Spring Cloud Stream provides a common abstraction for implementing partitioned processing use cases in a uniform fashion. Partitioning can thus be used whether the broker itself is naturally partitioned (for example, Kafka) or not (for example, RabbitMQ).

**Figure 26.4. Spring Cloud Stream Partitioning** 

images/SCSt-partitioning.png

Partitioning is a critical concept in stateful processing, where it is critical (for either performance or consistency reasons) to ensure that all related data is processed together. For example, in the time-windowed average calculation example, it is important that all measurements from any given sensor are processed by the same application instance.

> To set up a partitioned processing scenario, you must configure both the data-producing and the data-consuming ends.

