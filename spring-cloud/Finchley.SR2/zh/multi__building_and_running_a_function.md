## 125.构建并运行一个函数

上面的示例 `@SpringBootApplication` 有一个函数可以在运行时由Spring Cloud Function作为HTTPendpoints或Stream处理器进行修饰，例如RabbitMQ，Apache Kafka或JMS.

`@Beans` 可以是 `Function` ， `Consumer` 或 `Supplier` （全部来自 `java.util` ），它们的参数类型可以是String或POJO.如果 `spring-cloud-function-stream` 在类路径上， `Function` 将作为Spring Cloud Stream  `Processor` 公开.  `Consumer` 也暴露了作为流 `Sink` 和 `Supplier` 转换为流 `Source` .如果StreamBinders是 `spring-cloud-stream-binder-servlet` ，则公开HTTPendpoints.

函数可以是 `Flux<String>` 或 `Flux<Pojo>` ，Spring Cloud Function负责将数据转换为所需类型的数据，只要它以纯文本或（在POJO的情况下）JSON的形式出现. TBD：支持 `Flux<Message<Pojo>>` ，也许是普通的 `Pojo` 类型（框架隐含和实施的Fluxes）.

函数可以在一个应用程序中组合在一起，也可以在每个jar中部署一次.这取决于开发人员的选择.具有多种功能的应用程序可以在不同的“个性”中多次部署，从而在不同的物理传输上暴露不同的功能.
