## 43. Spring Session

Spring Boot provides [Spring Session](https://projects.spring.io/spring-session/) auto-configuration for a wide range of data stores. When building a Servlet web application, the following stores can be auto-configured:

- JDBC

- Redis

- Hazelcast

- MongoDB

When building a reactive web application, the following stores can be auto-configured:

- Redis

- MongoDB

If a single Spring Session module is present on the classpath, Spring Boot uses that store implementation automatically. If you have more than one implementation, you must choose the [StoreType](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/session/StoreType.java) that you wish to use to store the sessions. For instance, to use JDBC as the back-end store, you can configure your application as follows:

```java
spring.session.store-type=jdbc
```

> You can disable Spring Session by setting the  `store-type`  to  `none` .

Each store has specific additional settings. For instance, it is possible to customize the name of the table for the JDBC store, as shown in the following example:

```java
spring.session.jdbc.table-name=SESSIONS
```

For setting the timeout of the session you can use the  `spring.session.timeout`  property. If that property is not set, the auto-configuration falls back to the value of  `server.servlet.session.timeout` .
