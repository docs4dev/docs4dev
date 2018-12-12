## 123.简介

Spring Cloud Function是一个具有以下高级目标的项目：

- Prom通过函数实现业务逻辑.

- 从任何特定的运行时目标中解耦业务逻辑的开发生命周期，以便相同的代码可以作为Webendpoints，流处理器或任务运行.

- 支持无服务器提供商之间的统一编程模型，以及独立运行（本地或PaaS）的能力.

- Enable Spring Boot功能（自动配置，依赖注入，指标）在无服务器提供商上.

它抽象出所有传输细节和基础架构，允许开发人员保留所有熟悉的工具和流程，并专注于业务逻辑.

这是一个完整的，可执行的，可测试的Spring Boot应用程序（实现简单的字符串操作）：

```java
@SpringBootApplication
public class Application {

@Bean
public Function<Flux<String>, Flux<String>> uppercase() {
return flux -> flux.map(value -> value.toUpperCase());
}

public static void main(String[] args) {
SpringApplication.run(Application.class, args);
}
}
```

它只是一个Spring Boot应用程序，因此它可以在本地和CI构建中构建，运行和测试，与任何其他Spring Boot应用程序相同.  `Function` 来自 `java.util` ， `Flux` 是来自[Project Reactor](https://projectreactor.io/)的[Reactive Streams](http://www.reactive-streams.org/)  `Publisher` .可以通过HTTP或消息传递访问该功能.

Spring Cloud Function有4个主要功能：

函数，消费者和供应商类型的@Beans的包装器，使用RabbitMQ，Kafka等将它们作为HTTPendpoints和/或消息流监听器/发布者公开给外部世界.将作为Java函数体的字符串编译为字节码，然后将它们转换为进入可以如上所述包装的@Beans.使用隔离的类加载器部署包含此类应用程序上下文的JAR文件，以便您可以将它们打包在一个JVM中. AWS Lambda，Azure，Apache OpenWhisk以及可能的其他“无服务器”服务提供商的适配器.

> Spring Cloud是在非限制性Apache 2.0许可下发布的.如果您想参与本文档的这一部分，或者如果发现错误，请在[github](https://github.com/spring-cloud/spring-cloud-function/tree/master/docs/src/main/asciidoc)找到项目中的源代码和问题跟踪器.

