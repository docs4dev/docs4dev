## 37.发送电子邮件

Spring Framework提供了一个使用 `JavaMailSender` 接口发送电子邮件的简单抽象，Spring Boot为它提供了自动配置以及启动器模块.

> 有关如何使用 `JavaMailSender` 的详细说明，请参阅[reference documentation](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/integration.html#mail).

如果 `spring.mail.host` 和相关库（由 `spring-boot-starter-mail` 定义）可用，则创建默认 `JavaMailSender` （如果不存在）.可以通过 `spring.mail` 命名空间中的配置项进一步自定义发件人.有关详细信息，请参阅[MailProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/mail/MailProperties.java).

特别是，某些默认超时值是无限的，您可能希望更改它以避免线程被无响应的邮件服务器阻塞，如以下示例所示：

```java
spring.mail.properties.mail.smtp.connectiontimeout=5000
spring.mail.properties.mail.smtp.timeout=3000
spring.mail.properties.mail.smtp.writetimeout=5000
```

也可以使用JNDI中的现有 `Session` 配置 `JavaMailSender` ：

```java
spring.mail.jndi-name=mail/Session
```

设置 `jndi-name` 时，它优先于所有其他与会话相关的设置.
