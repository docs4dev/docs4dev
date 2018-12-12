## 37. Sending Email

The Spring Framework provides an easy abstraction for sending email by using the  `JavaMailSender`  interface, and Spring Boot provides auto-configuration for it as well as a starter module.

> See the [reference documentation](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/integration.html#mail) for a detailed explanation of how you can use  `JavaMailSender` .

If  `spring.mail.host`  and the relevant libraries (as defined by  `spring-boot-starter-mail` ) are available, a default  `JavaMailSender`  is created if none exists. The sender can be further customized by configuration items from the  `spring.mail`  namespace. See [MailProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/mail/MailProperties.java) for more details.

In particular, certain default timeout values are infinite, and you may want to change that to avoid having a thread blocked by an unresponsive mail server, as shown in the following example:

```java
spring.mail.properties.mail.smtp.connectiontimeout=5000
spring.mail.properties.mail.smtp.timeout=3000
spring.mail.properties.mail.smtp.writetimeout=5000
```

It is also possible to configure a  `JavaMailSender`  with an existing  `Session`  from JNDI:

```java
spring.mail.jndi-name=mail/Session
```

When a  `jndi-name`  is set, it takes precedence over all other Session-related settings.
