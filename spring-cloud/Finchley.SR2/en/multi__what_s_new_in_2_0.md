## 24. What’s New in 2.0?

Spring Cloud Stream introduces a number of new features, enhancements, and changes. The following sections outline the most notable ones:

- [Section 24.1, “New Features and Components”](multi__what_s_new_in_2_0.html#spring-cloud-stream-preface-new-features)

- [Section 24.2, “Notable Enhancements”](multi__what_s_new_in_2_0.html#spring-cloud-stream-preface-notable-enhancements)

## 24.1 New Features and Components

-  **Polling Consumers** : Introduction of polled consumers, which lets the application control message processing rates. See “[Section 27.3.4, “Using Polled Consumers”](multi__programming_model.html#spring-cloud-streams-overview-using-polled-consumers)” for more details. You can also read [this blog post](https://spring.io/blog/2018/02/27/spring-cloud-stream-2-0-polled-consumers) for more details.

-  **Micrometer Support** : Metrics has been switched to use [Micrometer](https://micrometer.io/).  `MeterRegistry`  is also provided as a bean so that custom applications can autowire it to capture custom metrics. See “[Chapter 35, Metrics Emitter](multi_spring-cloud-stream-overview-metrics-emitter.html)” for more details.

-  **New Actuator Binding Controls** : New actuator binding controls let you both visualize and control the Bindings lifecycle. For more details, see [Section 28.6, “Binding visualization and control”](multi_spring-cloud-stream-overview-binders.html#_binding_visualization_and_control).

-  **Configurable RetryTemplate** : Aside from providing properties to configure  `RetryTemplate` , we now let you provide your own template, effectively overriding the one provided by the framework. To use it, configure it as a  `@Bean`  in your application.

## 24.2 Notable Enhancements

This version includes the following notable enhancements:

- [Section 24.2.1, “Both Actuator and Web Dependencies Are Now Optional”](multi__what_s_new_in_2_0.html#spring-cloud-stream-preface-actuator-web-dependencies)

- [Section 24.2.2, “Content-type Negotiation Improvements”](multi__what_s_new_in_2_0.html#spring-cloud-stream-preface-content-type-negotiation-improvements)

- [Section 24.3, “Notable Deprecations”](multi__what_s_new_in_2_0.html#spring-cloud-stream-preface-notable-deprecations)

### 24.2.1 Both Actuator and Web Dependencies Are Now Optional

This change slims down the footprint of the deployed application in the event neither actuator nor web dependencies required. It also lets you switch between the reactive and conventional web paradigms by manually adding one of the following dependencies.

The following listing shows how to add the conventional web framework:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

The following listing shows how to add the reactive web framework:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

The following list shows how to add the actuator dependency:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### 24.2.2 Content-type Negotiation Improvements

One of the core themes for verion 2.0 is improvements (in both consistency and performance) around content-type negotiation and message conversion. The following summary outlines the notable changes and improvements in this area. See the “[Chapter 30, Content Type Negotiation](multi_content-type-management.html)” section for more details. Also [this blog post](https://spring.io/blog/2018/02/26/spring-cloud-stream-2-0-content-type-negotiation-and-transformation) contains more detail.

- All message conversion is now handled  **only**  by  `MessageConverter`  objects.

- We introduced the  `@StreamMessageConverter`  annotation to provide custom  `MessageConverter`  objects.

- We introduced the default  `Content Type`  as  `application/json` , which needs to be taken into consideration when migrating 1.3 application or operating in the mixed mode (that is, 1.3 producer → 2.0 consumer).

- Messages with textual payloads and a  `contentType`  of  `text/…`  or  `…/json`  are no longer converted to  `Message<String>`  for cases where the argument type of the provided  `MessageHandler`  can not be determined (that is,  `public void handle(Message<?> message)`  or  `public void handle(Object payload)` ). Furthermore, a strong argument type may not be enough to properly convert messages, so the  `contentType`  header may be used as a supplement by some  `MessageConverters` .

## 24.3 Notable Deprecations

As of version 2.0, the following items have been deprecated:

- [Section 24.3.1, “Java Serialization (Java Native and Kryo)”](multi__what_s_new_in_2_0.html#spring-cloud-stream-preface-deprecation-java-serialization)

- [Section 24.3.2, “Deprecated Classes and Methods”](multi__what_s_new_in_2_0.html#spring-cloud-stream-preface-deprecation-classes-methods)

### 24.3.1 Java Serialization (Java Native and Kryo)

`JavaSerializationMessageConverter`  and  `KryoMessageConverter`  remain for now. However, we plan to move them out of the core packages and support in the future. The main reason for this deprecation is to flag the issue that type-based, language-specific serialization could cause in distributed environments, where Producers and Consumers may depend on different JVM versions or have different versions of supporting libraries (that is, Kryo). We also wanted to draw the attention to the fact that Consumers and Producers may not even be Java-based, so polyglot style serialization (i.e., JSON) is better suited.

### 24.3.2 Deprecated Classes and Methods

The following is a quick summary of notable deprecations. See the corresponding {spring-cloud-stream-javadoc-current}[javadoc] for more details.

-  `SharedChannelRegistry` . Use  `SharedBindingTargetRegistry` .

-  `Bindings` . Beans qualified by it are already uniquely identified by their type — for example, provided  `Source` ,  `Processor` , or custom bindings:

```java
public interface Sample {
	String OUTPUT = "sampleOutput";

	@Output(Sample.OUTPUT)
	MessageChannel output();
}
```

-  `HeaderMode.raw` . Use  `none` ,  `headers`  or  `embeddedHeaders` 

-  `ProducerProperties.partitionKeyExtractorClass`  in favor of  `partitionKeyExtractorName`  and  `ProducerProperties.partitionSelectorClass`  in favor of  `partitionSelectorName` . This change ensures that both components are Spring configured and managed and are referenced in a Spring-friendly way.

-  `BinderAwareRouterBeanPostProcessor` . While the component remains, it is no longer a  `BeanPostProcessor`  and will be renamed in the future.

-  `BinderProperties.setEnvironment(Properties environment)` . Use  `BinderProperties.setEnvironment(Map<String, Object> environment)` .

This section goes into more detail about how you can work with Spring Cloud Stream. It covers topics such as creating and running stream applications.

