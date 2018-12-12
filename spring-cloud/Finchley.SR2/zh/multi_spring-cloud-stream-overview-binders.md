## 28.Binders

Spring Cloud Stream提供了一个Binder抽象，用于连接外部中间件的物理目标.本节提供有关Binder SPI背后的主要概念，其主要组件以及特定于实现的详细信息的信息.

## 28.1生产环境者和消费者

下图显示了生产环境者和消费者的一般关系：

**Figure 28.1. Producers and Consumers** 

图片/生产环境商，consumers.png

生产环境者是向消息发送消息的任何组件.可以将通道绑定到具有该代理的 `Binder` 实现的外部消息代理.调用 `bindProducer()` 方法时，第一个参数是其中的目标名称代理，第二个参数是生产环境者向其发送消息的本地通道实例，第三个参数包含要在为该通道创建的适配器中使用的属性（例如分区键表达式）.

使用者是从通道接收消息的任何组件.与生产环境者一样，消费者的渠道可以绑定到外部消息代理.调用 `bindConsumer()` 方法时，第一个参数是目标名称，第二个参数提供逻辑消费者组的名称.由给定目标的使用者绑定表示的每个组接收生成器发送到该目标的每个消息的副本（即，它遵循正常的发布 - 订阅语义）.如果有多个使用相同组名绑定的使用者实例，则会在这些使用者实例之间对消息进行负载平衡，以便生成者发送的每条消息仅由每个组中的单个使用者实例使用（即，它遵循正常排队）语义）.

## 28.2 Binder SPI

Binder SPI由许多接口，开箱即用的实用程序类和发现策略组成，这些策略提供了可连接到外部中间件的可插拔机制.

SPI的关键点是 `Binder` 接口，这是一种将输入和输出连接到外部中间件的策略.以下清单显示了 `Binder` 界面的定义：

```java
public interface Binder<T, C extends ConsumerProperties, P extends ProducerProperties> {
Binding<T> bindConsumer(String name, String group, T inboundBindTarget, C consumerProperties);

Binding<T> bindProducer(String name, T outboundBindTarget, P producerProperties);
}
```

接口已参数化，提供了许多扩展点：

- Input和输出绑定目标.从版本1.0开始，仅支持 `MessageChannel` ，但这将在以后用作扩展点.

- Extended Consumer和producer属性，允许特定的Binder实现添加可以类型安全方式支持的补充属性.

典型的Binders实现包括以下内容：

- 实现 `Binder` 接口的类;

- A Spring  `@Configuration` 类，它创建类型为 `Binder` 的bean以及中间件连接基础结构.
在包含一个或多个Binders定义的类路径中找到
- A  `META-INF/spring.binders` 文件，如以下示例所示：

```java
kafka:\
org.springframework.cloud.stream.binder.kafka.config.KafkaBinderConfiguration
```

## 28.3Binders检测

Spring Cloud Stream依赖于Binder SPI的实现来执行将通道连接到消息代理的任务.每个Binder实现通常连接到一种类型的消息传递系统.

### 28.3.1类路径检测

默认情况下，Spring Cloud Stream依靠Spring Boot的自动配置来配置绑定过程.如果在类路径上找到单个Binder实现，则Spring Cloud Stream会自动使用它.例如，旨在仅绑定到RabbitMQ的Spring Cloud Stream项目可以添加以下依赖项：

```xml
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-stream-binder-rabbit</artifactId>
</dependency>
```

有关其他绑定程序依赖项的特定Maven坐标，请参阅该绑定程序实现的文档.

## 28.4类路径上的多个Binders

当类路径上存在多个Binders时，应用程序必须指示每个通道绑定使用哪个Binders.每个Binders配置都包含一个 `META-INF/spring.binders` 文件，该文件是一个简单的属性文件，如以下示例所示：

```java
rabbit:\
org.springframework.cloud.stream.binder.rabbit.config.RabbitServiceAutoConfiguration
```

其他提供的Binders实现（例如Kafka）存在类似的文件，并且预期自定义Binders实现也提供它们.键表示Binders实现的标识名称，而值是以逗号分隔的配置类列表，每个配置类包含一个且仅包含一个类型为 `org.springframework.cloud.stream.binder.Binder` 的bean定义.

通过在每个通道绑定上配置Binders，可以使用 `spring.cloud.stream.defaultBinder` 属性（例如， `spring.cloud.stream.defaultBinder=rabbit` ）全局执行Binders选择，也可以单独执行Binders选择.例如，从Kafka读取并写入RabbitMQ的处理器应用程序（具有分别为 `input` 和 `output` 用于读取和写入的通道）可以指定以下配置：

```java
spring.cloud.stream.bindings.input.binder=kafka
spring.cloud.stream.bindings.output.binder=rabbit
```

## 28.5连接到多个系统

默认情况下，Binders共享应用程序的Spring Boot自动配置，以便创建在类路径中找到的每个Binders的一个实例.如果您的应用程序应连接到多个相同类型的代理，则可以指定多个Binders配置，每个配置具有不同的环境设置.

_0015在显式Binders配置上，完全禁用默认Binders配置过程.如果这样做，则所有正在使用的Binders必须包含在配置中.打算透明地使用Spring Cloud Stream的框架可以创建可以通过名称引用的Binders配置，但它们不会影响默认的Binders配置.为此，Binders配置可能将其 `defaultCandidate` 标志设置为false（例如， `spring.cloud.stream.binders.<configurationName>.defaultCandidate=false` ）.这表示独立于默认Binders配置过程而存在的配置.

以下示例显示了a的典型配置连接到两个RabbitMQ代理实例的处理器应用程序：

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

## 28.6绑定可视化和控制

从2.0版开始，Spring Cloud Stream通过Actuatorendpoints支持Bindings的可视化和控制.

从版本2.0开始，Actuator和Web是可选的，您必须首先添加一个Web依赖项，并手动添加Actuator依赖项.以下示例显示如何添加Web框架的依赖项：

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

以下示例显示如何为WebFlux框架添加依赖项：

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

您可以按如下方式添加Actuator依赖关系：

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

> 要在Cloud Foundry中运行Spring Cloud Stream 2.0应用程序，必须将 `spring-boot-starter-web` 和 `spring-boot-starter-actuator` 添加到类路径中.否则，由于运行状况检查失败，应用程序将无法启动.

您还必须通过设置以下属性来启用 `bindings` Actuatorendpoints： `--management.endpoints.web.exposure.include=bindings` .

一旦满足这些先决条件.应用程序启动时，您应该在日志中看到以下内容：

```java
: Mapped "{[/actuator/bindings/{name}],methods=[POST]. . .
: Mapped "{[/actuator/bindings],methods=[GET]. . .
: Mapped "{[/actuator/bindings/{name}],methods=[GET]. . .
```

要显示当前绑定，请访问以下URL： `http://<host>:<port>/actuator/bindings` 

另外，要查看单个绑定，请访问其中一个类似于以下内容的URL： `http://<host>:<port>/actuator/bindings/myBindingName` 

您还可以通过发布到同一URL来停止，启动，暂停和恢复单个绑定，同时提供 `state` 参数作为JSON，如以下示例所示：

curl -d'{"state"："STOPPED"}'-H "Content-Type: application/json" -X POST [http://<host>:<port>/actuator/bindings/myBindingName](http://<host>:<port>/actuator/bindings/myBindingName) curl -d'{"state"："STARTED"}'-H "Content-Type: application/json" -X POST [http://<host>:<port>/actuator/bindings/myBindingName](http://<host>:<port>/actuator/bindings/myBindingName) curl -d'{"state"："PAUSED"}'-H "Content-Type: application/json" -X POST [http://<host>:<port>/actuator/bindings/myBindingName](http://<host>:<port>/actuator/bindings/myBindingName) curl -d'{"state"："RESUMED"}'-H "Content-Type: application/json" -X POST [http://<host>:<port>/actuator/bindings/myBindingName](http://<host>:<port>/actuator/bindings/myBindingName)

>  `PAUSED` 和 `RESUMED` 仅在相应的Binders及其基础技术支持时才起作用.否则，您会在日志中看到警告消息.目前，只有KafkaBinders支持 `PAUSED` 和 `RESUMED` 状态.

## 28.7Binders配置属性

自定义Binders配置时，可以使用以下属性.这些属性通过 `org.springframework.cloud.stream.config.BinderProperties` 暴露

它们必须以 `spring.cloud.stream.binders.<configurationName>` 为前缀.

type类型Binders.它通常引用类路径中找到的一个Binders - 特别是META-INF / spring.binders文件中的一个键.默认情况下，它具有与配置名称相同的值. inheritEnvironment配置是否继承了应用程序本身的环境.默认值：true. environment一组属性的根，可用于自定义Binders的环境.设置此属性后，创建Binders的上下文不是应用程序上下文的子项.此设置允许Binders组件和应用组件之间的完全分离.默认值：空. defaultCandidateBinders配置是否可以被视为默认Binders，或者只能在显式引用时使用.此设置允许添加Binders配置，而不会干扰默认处理.默认值：true.

