## 28. Binders

Spring Cloud Stream provides a Binder abstraction for use in connecting to physical destinations at the external middleware. This section provides information about the main concepts behind the Binder SPI, its main components, and implementation-specific details.

## 28.1 Producers and Consumers

The following image shows the general relationship of producers and consumers:

**Figure 28.1. Producers and Consumers** 

images/producers-consumers.png

A producer is any component that sends messages to a channel. The channel can be bound to an external message broker with a  `Binder`  implementation for that broker. When invoking the  `bindProducer()`  method, the first parameter is the name of the destination within the broker, the second parameter is the local channel instance to which the producer sends messages, and the third parameter contains properties (such as a partition key expression) to be used within the adapter that is created for that channel.

A consumer is any component that receives messages from a channel. As with a producer, the consumer’s channel can be bound to an external message broker. When invoking the  `bindConsumer()`  method, the first parameter is the destination name, and a second parameter provides the name of a logical group of consumers. Each group that is represented by consumer bindings for a given destination receives a copy of each message that a producer sends to that destination (that is, it follows normal publish-subscribe semantics). If there are multiple consumer instances bound with the same group name, then messages are load-balanced across those consumer instances so that each message sent by a producer is consumed by only a single consumer instance within each group (that is, it follows normal queueing semantics).

## 28.2 Binder SPI

The Binder SPI consists of a number of interfaces, out-of-the box utility classes, and discovery strategies that provide a pluggable mechanism for connecting to external middleware.

The key point of the SPI is the  `Binder`  interface, which is a strategy for connecting inputs and outputs to external middleware. The following listing shows the definnition of the  `Binder`  interface:

```java
public interface Binder<T, C extends ConsumerProperties, P extends ProducerProperties> {
Binding<T> bindConsumer(String name, String group, T inboundBindTarget, C consumerProperties);

Binding<T> bindProducer(String name, T outboundBindTarget, P producerProperties);
}
```

The interface is parameterized, offering a number of extension points:

- Input and output bind targets. As of version 1.0, only  `MessageChannel`  is supported, but this is intended to be used as an extension point in the future.

- Extended consumer and producer properties, allowing specific Binder implementations to add supplemental properties that can be supported in a type-safe manner.

A typical binder implementation consists of the following:

- A class that implements the  `Binder`  interface;

- A Spring  `@Configuration`  class that creates a bean of type  `Binder`  along with the middleware connection infrastructure.

- A  `META-INF/spring.binders`  file found on the classpath containing one or more binder definitions, as shown in the following example:

```java
kafka:\
org.springframework.cloud.stream.binder.kafka.config.KafkaBinderConfiguration
```

## 28.3 Binder Detection

Spring Cloud Stream relies on implementations of the Binder SPI to perform the task of connecting channels to message brokers. Each Binder implementation typically connects to one type of messaging system.

### 28.3.1 Classpath Detection

By default, Spring Cloud Stream relies on Spring Boot’s auto-configuration to configure the binding process. If a single Binder implementation is found on the classpath, Spring Cloud Stream automatically uses it. For example, a Spring Cloud Stream project that aims to bind only to RabbitMQ can add the following dependency:

```xml
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-stream-binder-rabbit</artifactId>
</dependency>
```

For the specific Maven coordinates of other binder dependencies, see the documentation of that binder implementation.

## 28.4 Multiple Binders on the Classpath

When multiple binders are present on the classpath, the application must indicate which binder is to be used for each channel binding. Each binder configuration contains a  `META-INF/spring.binders`  file, which is a simple properties file, as shown in the following example:

```java
rabbit:\
org.springframework.cloud.stream.binder.rabbit.config.RabbitServiceAutoConfiguration
```

Similar files exist for the other provided binder implementations (such as Kafka), and custom binder implementations are expected to provide them as well. The key represents an identifying name for the binder implementation, whereas the value is a comma-separated list of configuration classes that each contain one and only one bean definition of type  `org.springframework.cloud.stream.binder.Binder` .

Binder selection can either be performed globally, using the  `spring.cloud.stream.defaultBinder`  property (for example,  `spring.cloud.stream.defaultBinder=rabbit` ) or individually, by configuring the binder on each channel binding. For instance, a processor application (that has channels named  `input`  and  `output`  for read and write respectively) that reads from Kafka and writes to RabbitMQ can specify the following configuration:

```java
spring.cloud.stream.bindings.input.binder=kafka
spring.cloud.stream.bindings.output.binder=rabbit
```

## 28.5 Connecting to Multiple Systems

By default, binders share the application’s Spring Boot auto-configuration, so that one instance of each binder found on the classpath is created. If your application should connect to more than one broker of the same type, you can specify multiple binder configurations, each with different environment settings.

> Turning on explicit binder configuration disables the default binder configuration process altogether. If you do so, all binders in use must be included in the configuration. Frameworks that intend to use Spring Cloud Stream transparently may create binder configurations that can be referenced by name, but they do not affect the default binder configuration. In order to do so, a binder configuration may have its  `defaultCandidate`  flag set to false (for example,  `spring.cloud.stream.binders.<configurationName>.defaultCandidate=false` ). This denotes a configuration that exists independently of the default binder configuration process.

The following example shows a typical configuration for a processor application that connects to two RabbitMQ broker instances:

```xml
spring:
cloud:
stream:
bindings:
input:
destination: thing1
binder: rabbit1
output:
destination: thing2
binder: rabbit2
binders:
rabbit1:
type: rabbit
environment:
spring:
rabbitmq:
host: <host1>
rabbit2:
type: rabbit
environment:
spring:
rabbitmq:
host: <host2>
```

## 28.6 Binding visualization and control

Since version 2.0, Spring Cloud Stream supports visualization and control of the Bindings through Actuator endpoints.

Starting with version 2.0 actuator and web are optional, you must first add one of the web dependencies as well as add the actuator dependency manually. The following example shows how to add the dependency for the Web framework:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

The following example shows how to add the dependency for the WebFlux framework:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

You can add the Actuator dependency as follows:

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

> To run Spring Cloud Stream 2.0 apps in Cloud Foundry, you must add  `spring-boot-starter-web`  and  `spring-boot-starter-actuator`  to the classpath. Otherwise, the application will not start due to health check failures.

You must also enable the  `bindings`  actuator endpoints by setting the following property:  `--management.endpoints.web.exposure.include=bindings` .

Once those prerequisites are satisfied. you should see the following in the logs when application start:

```java
: Mapped "{[/actuator/bindings/{name}],methods=[POST]. . .
: Mapped "{[/actuator/bindings],methods=[GET]. . .
: Mapped "{[/actuator/bindings/{name}],methods=[GET]. . .
```

To visualize the current bindings, access the following URL:  `http://<host>:<port>/actuator/bindings` 

Alternative, to see a single binding, access one of the URLs similar to the following:  `http://<host>:<port>/actuator/bindings/myBindingName` 

You can also stop, start, pause, and resume individual bindings by posting to the same URL while providing a  `state`  argument as JSON, as shown in the following examples:

curl -d '{"state":"STOPPED"}' -H "Content-Type: application/json" -X POST [http://<host>:<port>/actuator/bindings/myBindingName](http://<host>:<port>/actuator/bindings/myBindingName) curl -d '{"state":"STARTED"}' -H "Content-Type: application/json" -X POST [http://<host>:<port>/actuator/bindings/myBindingName](http://<host>:<port>/actuator/bindings/myBindingName) curl -d '{"state":"PAUSED"}' -H "Content-Type: application/json" -X POST [http://<host>:<port>/actuator/bindings/myBindingName](http://<host>:<port>/actuator/bindings/myBindingName) curl -d '{"state":"RESUMED"}' -H "Content-Type: application/json" -X POST [http://<host>:<port>/actuator/bindings/myBindingName](http://<host>:<port>/actuator/bindings/myBindingName)

>  `PAUSED`  and  `RESUMED`  work only when the corresponding binder and its underlying technology supports it. Otherwise, you see the warning message in the logs. Currently, only Kafka binder supports the  `PAUSED`  and  `RESUMED`  states.

## 28.7 Binder Configuration Properties

The following properties are available when customizing binder configurations. These properties exposed via  `org.springframework.cloud.stream.config.BinderProperties` 

They must be prefixed with  `spring.cloud.stream.binders.<configurationName>` .

type The binder type. It typically references one of the binders found on the classpath — in particular, a key in a META-INF/spring.binders file. By default, it has the same value as the configuration name. inheritEnvironment Whether the configuration inherits the environment of the application itself. Default: true. environment Root for a set of properties that can be used to customize the environment of the binder. When this property is set, the context in which the binder is being created is not a child of the application context. This setting allows for complete separation between the binder components and the application components. Default: empty. defaultCandidate Whether the binder configuration is a candidate for being considered a default binder or can be used only when explicitly referenced. This setting allows adding binder configurations without interfering with the default processing. Default: true.

