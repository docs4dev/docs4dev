## 42. Spring Integration

Spring Boot提供了一些使用[Spring Integration](https://projects.spring.io/spring-integration/)的便利，包括 `spring-boot-starter-integration` “Starter”. Spring Integration提供了有关消息传递以及其他传输（如HTTP，TCP等）的抽象.如果类路径上有Spring Integration，则会通过 `@EnableIntegration` 注释对其进行初始化.

Spring Boot还配置了一些由于存在其他Spring Integration模块而触发的功能.如果 `spring-integration-jmx` 也在类路径上，则通过JMX发布消息处理统计信息.如果 `spring-integration-jdbc` 可用，则可以在启动时创建默认数据库模式，如以下行所示：

```java
spring.integration.jdbc.initialize-schema=always
```

有关更多详细信息，请参阅[IntegrationAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/integration/IntegrationAutoConfiguration.java)和[IntegrationProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/integration/IntegrationProperties.java)类.

默认情况下，如果存在Micrometer  `meterRegistry`  bean，则Micro Integration将管理Spring Integration指标.如果您希望使用旧版Spring Integration度量标准，请将 `DefaultMetricsFactory`  bean添加到应用程序上下文中.
