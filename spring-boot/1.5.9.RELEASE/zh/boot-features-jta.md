## 38.使用JTA进行分布式事务

Spring Boot通过使用[Atomikos](http://www.atomikos.com/)或[Bitronix](https://github.com/bitronix/btm)嵌入式事务支持跨多个XA资源的分布式JTA事务经理.部署到合适的Java EE Application Server时，也支持JTA事务.

检测到JTA环境时，Spring的 `JtaTransactionManager` 用于管理事务.自动配置的JMS，DataSource和JPA bean已升级为支持XA事务.您可以使用标准的Spring惯用语（例如 `@Transactional` ）来参与分布式事务.如果您在JTA环境中并仍想使用本地事务，则可以将 `spring.jta.enabled` 属性设置为 `false` 以禁用JTA自动配置.

## 38.1使用Atomikos事务管理器

[Atomikos](https://www.atomikos.com/)是一个流行的开源事务管理器，可以嵌入到Spring Boot应用程序中.您可以使用 `spring-boot-starter-jta-atomikos`  Starter引入相应的Atomikos库. Spring Boot自动配置Atomikos并确保将适当的 `depends-on` 设置应用于Spring bean，以便正确启动和关闭命令.

默认情况下，Atomikos事务日志将写入应用程序主目录（应用程序jar文件所在的目录）中的 `transaction-logs` 目录.您可以通过在 `application.properties` 文件中设置 `spring.jta.log-dir` 属性来自定义此目录的位置.以 `spring.jta.atomikos.properties` 开头的属性也可用于自定义Atomikos  `UserTransactionServiceImp` .有关完整的详细信息，请参阅[AtomikosProperties Javadoc](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/api/org/springframework/boot/jta/atomikos/AtomikosProperties.html).

> 确保多个事务管理器可以安全地协调相同的资源管理器，每个Atomikos实例必须配置唯一的ID.默认情况下，此ID是运行Atomikos的计算机的IP地址.要确保生产环境中的唯一性，应为应用程序的每个实例配置 `spring.jta.transaction-manager-id` 属性，并使用不同的值.

## 38.2使用Bitronix事务管理器

[Bitronix](https://github.com/bitronix/btm)是一种流行的开源JTA事务管理器实现.您可以使用 `spring-boot-starter-jta-bitronix`  starter将适当的Bitronix依赖项添加到项目中.与Atomikos一样，Spring Boot会自动配置Bitronix并对bean进行后处理，以确保启动和关闭顺序正确.

默认情况下，Bitronix事务日志文件（ `part1.btm` 和 `part2.btm` ）将写入应用程序主目录中的 `transaction-logs` 目录.您可以通过设置 `spring.jta.log-dir` 属性来自定义此目录的位置.以 `spring.jta.bitronix.properties` 开头的属性也绑定到 `bitronix.tm.Configuration`  bean，允许完全自定义.有关详细信息，请参阅[Bitronix documentation](https://github.com/bitronix/btm/wiki/Transaction-manager-configuration).

> 确保多个事务管理器可以安全地协调相同的资源管理器，每个Bitronix实例必须配置一个唯一的ID.默认情况下，此ID是运行Bitronix的计算机的IP地址.要确保生产环境中的唯一性，应为应用程序的每个实例配置 `spring.jta.transaction-manager-id` 属性，并使用不同的值.

## 38.3使用Java EE托管事务管理器

如果将Spring Boot应用程序打包为 `war` 或 `ear` 文件并将其部署到Java EE应用程序服务器，则可以使用应用程序服务器的内置事务管理器. Spring Boot尝试通过查看常见的JNDI位置（ `java:comp/UserTransaction` ， `java:comp/TransactionManager` 等）来自动配置事务管理器.如果使用应用程序服务器提供的事务服务，通常还需要确保所有资源都由服务器管理并通过JNDI公开. Spring Boot尝试通过在JNDI路径（ `java:/JmsXA` 或 `java:/XAConnectionFactory` ）上查找 `ConnectionFactory` 来自动配置JMS，并且可以使用[spring.datasource.jndi-name property](boot-features-sql.html#boot-features-connecting-to-a-jndi-datasource)配置 `DataSource` .

## 38.4混合XA和非XA JMS连接

使用JTA时，主JMS  `ConnectionFactory`  bean可识别XA并参与分布式事务.在某些情况下，您可能希望使用非XA  `ConnectionFactory` 处理某些JMS消息.例如，您的JMS处理逻辑可能需要比XA超时更长的时间.

如果要使用非XA  `ConnectionFactory` ，可以注入 `nonXaJmsConnectionFactory`  bean而不是 `@Primary`   `jmsConnectionFactory`  bean.为了保持一致性，还使用bean别名 `xaJmsConnectionFactory` 提供 `jmsConnectionFactory`  bean.

以下示例显示如何注入 `ConnectionFactory` 实例：

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

## 38.5支持替代嵌入式事务管理器

[XAConnectionFactoryWrapper](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jms/XAConnectionFactoryWrapper.java)和[XADataSourceWrapper](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jdbc/XADataSourceWrapper.java)接口可用于支持备用嵌入式事务管理器.接口负责包装 `XAConnectionFactory` 和 `XADataSource`  bean并将它们公开为常规的 `ConnectionFactory` 和 `DataSource`  bean，它们透明地注册分布式事务. DataSource和JMS自动配置使用JTA变体，前提是您在 `ApplicationContext` 中注册了 `JtaTransactionManager`  bean和相应的XA包装bean.

[BitronixXAConnectionFactoryWrapper](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jta/bitronix/BitronixXAConnectionFactoryWrapper.java)和[BitronixXADataSourceWrapper](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jta/bitronix/BitronixXADataSourceWrapper.java)提供了如何编写XA包装器的好例子.

