## 24. 2.0中有什么新功能？

Spring Cloud Stream引入了许多新功能，增强功能和更改.以下部分概述了最值得注意的部分：

- [Section 24.1, “New Features and Components”](multi__what_s_new_in_2_0.html#spring-cloud-stream-preface-new-features)

- [Section 24.2, “Notable Enhancements”](multi__what_s_new_in_2_0.html#spring-cloud-stream-preface-notable-enhancements)

## 24.1新功能和组件

-  **Polling Consumers** ：引入轮询的消费者，它允许应用程序控制消息处理速率.有关详细信息，请参阅“[Section 27.3.4, “Using Polled Consumers”](multi__programming_model.html#spring-cloud-streams-overview-using-polled-consumers)”.你也可以阅读[this blog post](https://spring.io/blog/2018/02/27/spring-cloud-stream-2-0-polled-consumers)了解更多详情.

-  **Micrometer Support** ：度量标准已切换为使用[Micrometer](https://micrometer.io/).  `MeterRegistry` 也作为bean提供，以便自定义应用程序可以自动装配它以捕获自定义指标.有关详细信息，请参阅“[Chapter 35, Metrics Emitter](multi_spring-cloud-stream-overview-metrics-emitter.html)”.

-  **New Actuator Binding Controls** ：新的Actuator绑定控件可让您可视化和控制Bindings生命周期.有关更多详细信息，请参阅[Section 28.6, “Binding visualization and control”](multi_spring-cloud-stream-overview-binders.html#_binding_visualization_and_control).

-  **Configurable RetryTemplate** ：除了提供配置 `RetryTemplate` 的属性之外，我们现在允许您提供自己的模板，有效地覆盖框架提供的模板.要使用它，请在应用程序中将其配置为 `@Bean` .

## 24.2值得注意的增强功能

此版本包括以下显着增强功能：

- [Section 24.2.1, “Both Actuator and Web Dependencies Are Now Optional”](multi__what_s_new_in_2_0.html#spring-cloud-stream-preface-actuator-web-dependencies)

- [Section 24.2.2, “Content-type Negotiation Improvements”](multi__what_s_new_in_2_0.html#spring-cloud-stream-preface-content-type-negotiation-improvements)

- [Section 24.3, “Notable Deprecations”](multi__what_s_new_in_2_0.html#spring-cloud-stream-preface-notable-deprecations)

### 24.2.1Actuator和Web依赖关系现在都是可选的

如果不需要Actuator或Web依赖性，则此更改会减少已部署应用程序的占用空间.它还允许您通过手动添加以下依赖项之一在响应和传统Web范例之间切换.

以下清单显示了如何添加传统的Web框架：

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

以下清单显示了如何添加响应式Web框架：

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

以下列表显示了如何添加Actuator依赖项：

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### 24.2.2内容类型谈判改进

verion 2.0的核心主题之一是围绕内容类型协商和消息转换的改进（在一致性和性能方面）.以下摘要概述了该领域的显着变化和改进.有关详细信息，请参阅“[Chapter 30, Content Type Negotiation](multi_content-type-management.html)”部分.另外[this blog post](https://spring.io/blog/2018/02/26/spring-cloud-stream-2-0-content-type-negotiation-and-transformation)包含更多细节.

- 所有消息转换现在由 `MessageConverter` 对象处理 **only** .

- We引入了 `@StreamMessageConverter` 注释来提供自定义 `MessageConverter` 对象.

- We引入了默认 `Content Type` 为 `application/json` ，在迁移1.3应用程序或在混合模式下运行时需要考虑这一点（即1.3生产环境者→2.0使用者）.
如果无法确定提供的 `MessageHandler` 的参数类型（即 `public void handle(Message<?> message)` 或 `public void handle(Object payload)` ），则
- 具有文本有效负载的消息和 `contentType` 的 `text/…` 或 `…/json` 不再转换为 `Message<String>` .此外，强大的参数类型可能不足以正确转换消息，因此 `contentType` Headers可能被某些 `MessageConverters` 用作补充.

## 24.3值得注意的弃用

从2.0版开始，不推荐使用以下项目：

- [Section 24.3.1, “Java Serialization (Java Native and Kryo)”](multi__what_s_new_in_2_0.html#spring-cloud-stream-preface-deprecation-java-serialization)

- [Section 24.3.2, “Deprecated Classes and Methods”](multi__what_s_new_in_2_0.html#spring-cloud-stream-preface-deprecation-classes-methods)

### 24.3.1 Java序列化（Java Native和Kryo）

`JavaSerializationMessageConverter` 和 `KryoMessageConverter` 暂时保留.但是，我们计划在未来将它们从核心软件包和支持中移除.这种弃用的主要原因是标记基于类型的，特定于语言的序列化可能在分布式环境中引起的问题，其中生产环境者和消费者可能依赖于不同的JVM版本或具有不同版本的支持库（即Kryo）.我们还想提请注意消费者和生产环境者甚至可能不是基于Java的事实，因此多语言样式序列化（即JSON）更适合.

### 24.3.2不推荐使用的类和方法

以下是显着弃用的快速摘要.有关更多详细信息，请参阅相应的{spring-cloud-stream-javadoc-current} [javadoc].

-  `SharedChannelRegistry` .使用 `SharedBindingTargetRegistry` .

-  `Bindings` .由它限定的 beans已经由它们的类型唯一标识 - 例如，提供 `Source` ，_  `Processor` 或自定义绑定：

```java
public interface Sample {
	String OUTPUT = "sampleOutput";

	@Output(Sample.OUTPUT)
	MessageChannel output();
}
```

-  `HeaderMode.raw` .使用 `none` ， `headers` 或 `embeddedHeaders` 

-  `ProducerProperties.partitionKeyExtractorClass` 支持 `partitionKeyExtractorName` 和 `ProducerProperties.partitionSelectorClass` 支持 `partitionSelectorName` .此更改确保两个组件都是Spring配置和管理的，并以Spring友好的方式引用.

-  `BinderAwareRouterBeanPostProcessor` .虽然该组件仍然存在，但它不再是 `BeanPostProcessor` ，将来会重命名.

-  `BinderProperties.setEnvironment(Properties environment)` .使用 `BinderProperties.setEnvironment(Map<String, Object> environment)` .

本节详细介绍了如何使用Spring Cloud Stream.它涵盖了创建和运行流应用程序等主题.

