## 38. Distributed Transactions with JTA

Spring Boot supports distributed JTA transactions across multiple XA resources by using either an [Atomikos](http://www.atomikos.com/) or [Bitronix](https://github.com/bitronix/btm) embedded transaction manager. JTA transactions are also supported when deploying to a suitable Java EE Application Server.

When a JTA environment is detected, Spring’s  `JtaTransactionManager`  is used to manage transactions. Auto-configured JMS, DataSource, and JPA beans are upgraded to support XA transactions. You can use standard Spring idioms, such as  `@Transactional` , to participate in a distributed transaction. If you are within a JTA environment and still want to use local transactions, you can set the  `spring.jta.enabled`  property to  `false`  to disable the JTA auto-configuration.

## 38.1 Using an Atomikos Transaction Manager

[Atomikos](https://www.atomikos.com/) is a popular open source transaction manager which can be embedded into your Spring Boot application. You can use the  `spring-boot-starter-jta-atomikos`  Starter to pull in the appropriate Atomikos libraries. Spring Boot auto-configures Atomikos and ensures that appropriate  `depends-on`  settings are applied to your Spring beans for correct startup and shutdown ordering.

By default, Atomikos transaction logs are written to a  `transaction-logs`  directory in your application’s home directory (the directory in which your application jar file resides). You can customize the location of this directory by setting a  `spring.jta.log-dir`  property in your  `application.properties`  file. Properties starting with  `spring.jta.atomikos.properties`  can also be used to customize the Atomikos  `UserTransactionServiceImp` . See the [AtomikosProperties Javadoc](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/api/org/springframework/boot/jta/atomikos/AtomikosProperties.html) for complete details.

> To ensure that multiple transaction managers can safely coordinate the same resource managers, each Atomikos instance must be configured with a unique ID. By default, this ID is the IP address of the machine on which Atomikos is running. To ensure uniqueness in production, you should configure the  `spring.jta.transaction-manager-id`  property with a different value for each instance of your application.

## 38.2 Using a Bitronix Transaction Manager

[Bitronix](https://github.com/bitronix/btm) is a popular open-source JTA transaction manager implementation. You can use the  `spring-boot-starter-jta-bitronix`  starter to add the appropriate Bitronix dependencies to your project. As with Atomikos, Spring Boot automatically configures Bitronix and post-processes your beans to ensure that startup and shutdown ordering is correct.

By default, Bitronix transaction log files ( `part1.btm`  and  `part2.btm` ) are written to a  `transaction-logs`  directory in your application home directory. You can customize the location of this directory by setting the  `spring.jta.log-dir`  property. Properties starting with  `spring.jta.bitronix.properties`  are also bound to the  `bitronix.tm.Configuration`  bean, allowing for complete customization. See the [Bitronix documentation](https://github.com/bitronix/btm/wiki/Transaction-manager-configuration) for details.

> To ensure that multiple transaction managers can safely coordinate the same resource managers, each Bitronix instance must be configured with a unique ID. By default, this ID is the IP address of the machine on which Bitronix is running. To ensure uniqueness in production, you should configure the  `spring.jta.transaction-manager-id`  property with a different value for each instance of your application.

## 38.3 Using a Java EE Managed Transaction Manager

If you package your Spring Boot application as a  `war`  or  `ear`  file and deploy it to a Java EE application server, you can use your application server’s built-in transaction manager. Spring Boot tries to auto-configure a transaction manager by looking at common JNDI locations ( `java:comp/UserTransaction` ,  `java:comp/TransactionManager` , and so on). If you use a transaction service provided by your application server, you generally also want to ensure that all resources are managed by the server and exposed over JNDI. Spring Boot tries to auto-configure JMS by looking for a  `ConnectionFactory`  at the JNDI path ( `java:/JmsXA`  or  `java:/XAConnectionFactory` ), and you can use the [spring.datasource.jndi-name property](boot-features-sql.html#boot-features-connecting-to-a-jndi-datasource) to configure your  `DataSource` .

## 38.4 Mixing XA and Non-XA JMS Connections

When using JTA, the primary JMS  `ConnectionFactory`  bean is XA-aware and participates in distributed transactions. In some situations, you might want to process certain JMS messages by using a non-XA  `ConnectionFactory` . For example, your JMS processing logic might take longer than the XA timeout.

If you want to use a non-XA  `ConnectionFactory` , you can inject the  `nonXaJmsConnectionFactory`  bean rather than the  `@Primary`   `jmsConnectionFactory`  bean. For consistency, the  `jmsConnectionFactory`  bean is also provided by using the bean alias  `xaJmsConnectionFactory` .

The following example shows how to inject  `ConnectionFactory`  instances:

```java
// Inject the primary (XA aware) ConnectionFactory
@Autowired
private ConnectionFactory defaultConnectionFactory;

// Inject the XA aware ConnectionFactory (uses the alias and injects the same as above)
@Autowired
@Qualifier("xaJmsConnectionFactory")
private ConnectionFactory xaConnectionFactory;

// Inject the non-XA aware ConnectionFactory
@Autowired
@Qualifier("nonXaJmsConnectionFactory")
private ConnectionFactory nonXaConnectionFactory;
```

## 38.5 Supporting an Alternative Embedded Transaction Manager

The [XAConnectionFactoryWrapper](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jms/XAConnectionFactoryWrapper.java) and [XADataSourceWrapper](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jdbc/XADataSourceWrapper.java) interfaces can be used to support alternative embedded transaction managers. The interfaces are responsible for wrapping  `XAConnectionFactory`  and  `XADataSource`  beans and exposing them as regular  `ConnectionFactory`  and  `DataSource`  beans, which transparently enroll in the distributed transaction. DataSource and JMS auto-configuration use JTA variants, provided you have a  `JtaTransactionManager`  bean and appropriate XA wrapper beans registered within your  `ApplicationContext` .

The [BitronixXAConnectionFactoryWrapper](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jta/bitronix/BitronixXAConnectionFactoryWrapper.java) and [BitronixXADataSourceWrapper](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jta/bitronix/BitronixXADataSourceWrapper.java) provide good examples of how to write XA wrappers.

