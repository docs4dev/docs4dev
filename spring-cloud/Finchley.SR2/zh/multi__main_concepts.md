## 26.主要概念

Spring Cloud Stream提供了许多抽象和原语，简化了消息驱动的微服务应用程序的编写.本节概述了以下内容：

- [Spring Cloud Stream’s application model](multi__main_concepts.html#spring-cloud-stream-overview-application-model)

- [Section 26.2, “The Binder Abstraction”](multi__main_concepts.html#spring-cloud-stream-overview-binder-abstraction)

- [Persistent publish-subscribe support](multi__main_concepts.html#spring-cloud-stream-overview-persistent-publish-subscribe-support)

- [Consumer group support](multi__main_concepts.html#consumer-groups)

- [Partitioning support](multi__main_concepts.html#partitioning)

- [A pluggable Binder SPI](multi_spring-cloud-stream-overview-binders.html#spring-cloud-stream-overview-binder-api)

## 26.1应用程序模型

Spring Cloud Stream应用程序由中间件中立核心组成.该应用程序通过Spring Cloud Stream注入其中的输入和输出通道与外界通信.通过中间件特定的Binder实现，通道连接到外部代理.

**Figure 26.1. Spring Cloud Stream Application** 

图像/ SCST与 -  binder.png

### 26.1.1胖子JAR

Spring Cloud Stream应用程序可以在IDE中以独立模式运行以进行测试.要在生产环境中运行Spring Cloud Stream应用程序，可以使用为Maven或Gradle提供的标准Spring Boot工具创建可执行（或“胖”）JAR.有关详细信息，请参阅[Spring Boot Reference Guide](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-build.html#howto-create-an-executable-jar-with-maven).

## 26.2Binders抽象

Spring Cloud Stream为[Kafka](https://github.com/spring-cloud/spring-cloud-stream/tree/master/spring-cloud-stream-binders/spring-cloud-stream-binder-kafka)和[Rabbit MQ](https://github.com/spring-cloud/spring-cloud-stream/tree/master/spring-cloud-stream-binders/spring-cloud-stream-binder-rabbit)提供了Binder实现. Spring Cloud Stream还包含一个[TestSupportBinder](https://github.com/spring-cloud/spring-cloud-stream/blob/master/spring-cloud-stream-test-support/src/main/java/org/springframework/cloud/stream/test/binder/TestSupportBinder.java)，它保留了一个未修改的通道，以便测试可以直接与通道交互并可靠地断言接收到的内容.您还可以使用可扩展API编写自己的Binder.

Spring Cloud Stream使用Spring Boot进行配置，Binder抽象使Spring Cloud Stream应用程序可以灵活地连接到中间件.例如，部署者可以在运行时动态选择通道连接的目的地（例如Kafka主题或RabbitMQ交换）.这种配置可以通过外部配置属性以及Spring Boot支持的任何形式提供（包括应用程序参数，环境变量和 `application.yml` 或 `application.properties` 文件）.在[Chapter 25, Introducing Spring Cloud Stream](multi_spring-cloud-stream-overview-introducing.html)部分的接收器示例中，将 `spring.cloud.stream.bindings.input.destination`  application属性设置为 `raw-sensor-data` 会使其从 `raw-sensor-data`  Kafka主题或绑定到 `raw-sensor-data`  RabbitMQ交换的队列中读取.

Spring Cloud Stream自动检测并使用类路径中找到的Binders.您可以使用具有相同代码的不同类型的中间件.为此，请在构建时包含不同的Binders.对于更复杂的用例，您还可以在应用程序中打包多个Binders，并让它在运行时选择Binders（甚至是否为不同的通道使用不同的Binders）.

## 26.3持久发布 - 订阅支持

应用程序之间的通信遵循发布 - 订阅模型，其中数据通过共享主题广播.这可以在下图中看到，该图显示了一组交互式Spring Cloud Stream应用程序的典型部署.

**Figure 26.2. Spring Cloud Stream Publish-Subscribe** 

图像/ SCST-sensors.png

传感器向HTTPendpoints报告的数据将发送到名为 `raw-sensor-data` 的公共目标.从目的地开始，它由一个计算时间窗平均值的微服务应用程序和另一个将原始数据摄入HDFS（Hadoop分布式文件系统）的微服务应用程序独立处理.为了处理数据，两个应用程序都将主题声明为运行时的输入.

发布 - 订阅通信模型降低了生产环境者和消费者的复杂性，并允许将新应用程序添加到拓扑中，而不会中断现有流.例如，在平均值计算应用程序的下游，您可以添加计算显示和监视的最高温度值的应用程序.然后，您可以添加另一个应用程序来解释相同的平均流量以进行故障检测.通过共享主题而不是点对点队列进行所有通信可以减少微服务之间的耦合.

虽然发布 - 订阅消息传递的概念并不新鲜，但Spring Cloud Stream采取了额外的步骤，使其成为其应用程序模型的自觉选择.通过使用本机中间件支持，Spring Cloud Stream还简化了跨不同平台的发布 - 订阅模型的使用.

## 26.4消费者群体

虽然发布 - 订阅模型可以通过共享主题轻松连接应用程序，但可以通过创建来扩展给定应用程序的多个实例同样重要.执行此操作时，应用程序的不同实例将放置在竞争的使用者关系中，其中只有一个实例需要处理给定的消息.

Spring Cloud Stream通过消费者群体的概念对此行为进行建模. （Spring Cloud Stream消费者群体与Kafka消费者群体类似并受其启发.）每个消费者绑定都可以使用 `spring.cloud.stream.bindings.<channelName>.group` 属性来指定群组名称.对于下图中显示的使用者，此属性将设置为 `spring.cloud.stream.bindings.<channelName>.group=hdfsWrite` 或 `spring.cloud.stream.bindings.<channelName>.group=average` .

**Figure 26.3. Spring Cloud Stream Consumer Groups** 

图像/ SCST-groups.png

订阅给定目标的所有组都会收到已发布数据的副本，但每个组中只有一个成员从该目标接收给定的消息.默认情况下，当未指定组时，Spring Cloud Stream会将应用程序分配给与所有其他使用者组处于发布 - 订阅关系的匿名且独立的单成员使用者组.

## 26.5消费者类型

支持两种类型的消费者：

- Message驱动（有时称为异步）

- Polled（有时称为同步）

在2.0版之前，仅支持异步使用者.消息一旦可用就会传递，并且有一个线程可以处理它.

如果要控制处理消息的速率，可能需要使用同步使用者.

### 26.5.1耐久性

与Spring Cloud Stream的固定应用模型一致，消费者组订阅是持久的.也就是说，Binders实现确保组订阅是持久的，并且一旦创建了组的至少一个订阅，该组就接收消息，即使它们是在组中的所有应用程序都被停止时发送的.

> 匿名订阅本质上是非持久的.对于某些Binders实现（例如RabbitMQ），可以具有非持久的组订阅.

通常，在将应用程序绑定到给定目标时，最好始终指定使用者组.扩展Spring Cloud Stream应用程序时，必须为每个输入绑定指定一个使用者组.这样做可以防止应用程序的实例接收重复的消息（除非需要这种行为，这是不寻常的）.

## 26.6分区支持

Spring Cloud Stream支持在给定应用程序的多个实例之间对数据进行分区.在分区方案中，物理通信介质（例如代理主题）被视为结构化为多个分区.一个或多个生产环境者应用程序实例将数据发送到多个消费者应用程序实例，并确保由共同特征标识的数据由同一个消费者实例处理.

Spring Cloud Stream提供了一种通用抽象，用于以统一的方式实现分区处理用例.因此，无论代理本身是否自然分区（例如，Kafka）（例如，RabbitMQ），都可以使用分区.

**Figure 26.4. Spring Cloud Stream Partitioning** 

图像/ SCST-partitioning.png

分区是有状态处理中的一个关键概念，其中确保所有相关数据一起处理至关重要（出于性能或一致性原因）.例如，在时间窗口平均值计算示例中，重要的是来自任何给定传感器的所有测量值都由同一应用程序实例处理.

> 要设置分区处理方案，必须同时配置数据生成和数据消耗两端.

