## 43.Spring季会议

Spring Boot为各种数据存储提供[Spring Session](https://projects.spring.io/spring-session/)自动配置.构建Servlet Web应用程序时，可以自动配置以下存储：

- JDBC

- Redis

- Hazelcast

- MongoDB

构建响应式Web应用程序时，可以自动配置以下存储：

- Redis

- MongoDB

如果类路径上存在单个Spring Session模块，则Spring Boot会自动使用该存储实现.如果您有多个实现，则必须选择要用于存储会话的[StoreType](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/session/StoreType.java).例如，要使用JDBC作为后端存储，您可以按如下方式配置应用程序：

```java
spring.session.store-type=jdbc
```

> 您可以通过将 `store-type` 设置为 `none` 来禁用Spring Session.

每个商店都有特定的附加设置.例如，可以为JDBC存储定制表的名称，如以下示例所示：

```java
spring.session.jdbc.table-name=SESSIONS
```

要设置会话超时，可以使用 `spring.session.timeout` 属性.如果未设置该属性，则自动配置将回退到 `server.servlet.session.timeout` 的值.
