## 42. Spring Integration

Spring Boot offers several conveniences for working with [Spring Integration](https://projects.spring.io/spring-integration/), including the  `spring-boot-starter-integration`  “Starter”. Spring Integration provides abstractions over messaging and also other transports such as HTTP, TCP, and others. If Spring Integration is available on your classpath, it is initialized through the  `@EnableIntegration`  annotation.

Spring Boot also configures some features that are triggered by the presence of additional Spring Integration modules. If  `spring-integration-jmx`  is also on the classpath, message processing statistics are published over JMX . If  `spring-integration-jdbc`  is available, the default database schema can be created on startup, as shown in the following line:

```java
spring.integration.jdbc.initialize-schema=always
```

See the [IntegrationAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/integration/IntegrationAutoConfiguration.java) and [IntegrationProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/integration/IntegrationProperties.java) classes for more details.

By default, if a Micrometer  `meterRegistry`  bean is present, Spring Integration metrics will be managed by Micrometer. If you wish to use legacy Spring Integration metrics, add a  `DefaultMetricsFactory`  bean to the application context.
