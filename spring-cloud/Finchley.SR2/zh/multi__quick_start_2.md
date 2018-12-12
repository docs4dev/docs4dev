## 23.快速入门

您可以在不到5分钟的时间内尝试Spring Cloud Stream，甚至在您按照这个三步指南跳转到任何细节之前.

我们将向您展示如何创建一个Spring Cloud Stream应用程序，该应用程序接收来自您选择的消息传递中间件的消息（稍后将详细介绍）并将收到的消息记录到控制台.我们称之为 `LoggingConsumer` .虽然不太实用，但它提供了一些主要概念和抽象的良好介绍，使其更容易消化本用户指南的其余部分.

这三个步骤如下：

第23.1节“使用Spring Initializr创建示例应用程序”第23.2节“将项目导入IDE”第23.3节“添加消息处理程序，构建和运行”

## 23.1使用Spring Initializr创建示例应用程序

要开始使用，请访问[Spring Initializr](https://start.spring.io).从那里，您可以生成我们的 `LoggingConsumer` 应用程序.为此：

在“依赖关系”部分中，开始键入流.当出现“Cloud Stream”选项时，选择它.开始输入'kafka'或'rabbit'.选择“Kafka”或“RabbitMQ”.基本上，您选择应用程序绑定的消息传递中间件.我们建议您使用已安装的或安装和运行时感觉更舒适.此外，从Initilaizer屏幕中可以看到，您可以选择其他一些选项.例如，您可以选择Gradle作为构建工具而不是Maven（默认值）.在“工件”字段中，键入“logging-consumer”. Artifact字段的值成为应用程序名称.如果您选择RabbitMQ作为中间件，则Spring Initializr现在应该如下所示：单击Generate Project按钮.这样做会将生成的项目的压缩版本下载到硬盘驱动器.将文件解压缩到要用作项目目录的文件夹中.

> We鼓励您探索Spring Initializr中提供的众多可能性.它允许您创建许多不同类型的Spring应用程序.

## 23.2将项目导入IDE

现在，您可以将项目导入IDE.请记住，根据IDE，您可能需要遵循特定的导入过程.例如，根据项目的生成方式（Maven或Gradle），您可能需要遵循特定的导入过程（例如，在Eclipse或STS中，您需要使用File→Import→Maven→Existing Maven Project）.

导入后，项目必须没有任何错误.此外， `src/main/java` 应包含 `com.example.loggingconsumer.LoggingConsumerApplication` .

从技术上讲，此时，您可以运行应用程序的主类.它已经是一个有效的Spring Boot应用程序.但是，它没有做任何事情，所以我们想添加一些代码.

## 23.3添加消息处理程序，构建和运行

将 `com.example.loggingconsumer.LoggingConsumerApplication` 类修改为如下所示：

```java
@SpringBootApplication
@EnableBinding(Sink.class)
public class LoggingConsumerApplication {

	public static void main(String[] args) {
		SpringApplication.run(LoggingConsumerApplication.class, args);
	}

	@StreamListener(Sink.INPUT)
	public void handle(Person person) {
		System.out.println("Received: " + person);
	}

	public static class Person {
		private String name;
		public String getName() {
			return name;
		}
		public void setName(String name) {
			this.name = name;
		}
		public String toString() {
			return this.name;
		}
	}
}
```

从前面的清单中可以看出：

- 我们使用 `@EnableBinding(Sink.class)` 启用了 `Sink` 绑定（输入无输出）.这样做会向框架发出信号，以启动与消息传递中间件的绑定，从而自动创建绑定到 `Sink.INPUT` 通道的目标（即队列，主题和其他）.

- 我们添加了一个 `handler` 方法来接收 `Person` 类型的传入消息.这样做可以让您看到框架的核心功能之一：它尝试自动将传入的消息有效负载转换为 `Person` 类型.

您现在拥有一个功能齐全的Spring Cloud Stream应用程序，可以侦听消息.从这里开始，为简单起见，我们假设您在[step one](multi__quick_start_2.html#spring-cloud-stream-preface-creating-sample-application)中选择了RabbitMQ.假设您已安装并运行RabbitMQ，则可以通过在IDE中运行其 `main` 方法来启动应用程序.

你应该看到以下输出：

```java
--- [ main] c.s.b.r.p.RabbitExchangeQueueProvisioner : declaring queue for inbound: input.anonymous.CbMIwdkJSBO1ZoPDOtHtCg, bound to: input
	--- [ main] o.s.a.r.c.CachingConnectionFactory       : Attempting to connect to: [localhost:5672]
	--- [ main] o.s.a.r.c.CachingConnectionFactory       : Created new connection: rabbitConnectionFactory#2a3a299:0/[emailprotected] . .
	. . .
	--- [ main] o.s.i.a.i.AmqpInboundChannelAdapter      : started inbound.input.anonymous.CbMIwdkJSBO1ZoPDOtHtCg
	. . .
	--- [ main] c.e.l.LoggingConsumerApplication         : Started LoggingConsumerApplication in 2.531 seconds (JVM running for 2.897)
```

转到RabbitMQ管理控制台或任何其他RabbitMQ客户端并向 `input.anonymous.CbMIwdkJSBO1ZoPDOtHtCg` 发送消息.  `anonymous.CbMIwdkJSBO1ZoPDOtHtCg` 部分表示组名称并已生成，因此它在您的环境中必然会有所不同.对于更可预测的内容，您可以通过设置 `spring.cloud.stream.bindings.input.group=hello` （或您喜欢的任何名称）来使用显式组名.

消息的内容应该是 `Person` 类的JSON表示，如下所示：

```java
{"name":"Sam Spade"}
```

然后，在您的控制台中，您应该看到：

`Received: Sam Spade` 

您还可以将应用程序构建并打包到引导jar中（通过使用 `./mvnw clean install` ）并使用 `java -jar` 命令运行构建的JAR.

现在您有一个工作（尽管非常基本的）Spring Cloud Stream应用程序.

