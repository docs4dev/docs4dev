## 30.使用SQL数据库

[Spring Framework](https://projects.spring.io/spring-framework/)为使用提供了广泛的支持SQL数据库，从直接JDBC访问使用 `JdbcTemplate` 来完成“对象关系映射”技术，如Hibernate. [Spring Data](https://projects.spring.io/spring-data/)提供了更多级别的功能：直接从接口创建 `Repository` 实现，并使用约定从方法名称生成查询.

## 30.1配置数据源

Java的 `javax.sql.DataSource` 接口提供了一种使用数据库连接的标准方法.传统上，'DataSource'使用 `URL` 以及一些凭据来Build数据库连接.

> 参见[the “How-to” section](howto-data-access.html#howto-configure-a-datasource)了解更多高级示例，通常是为了完全控制DataSource的配置.

### 30.1.1嵌入式数据库支持

通过使用内存中嵌入式数据库来开发应用程序通常很方便.显然，内存数据库不提供持久存储.您需要在应用程序启动时填充数据库，并准备在应用程序结束时丢弃数据.

> “操作方法”部分包含[section on how to initialize a database](howto-database-initialization.html).

Spring Boot可以自动配置嵌入的[H2](http://www.h2database.com)，[HSQL](http://hsqldb.org/)和[Derby](https://db.apache.org/derby/)数据库.您无需提供任何连接URL.您只需要包含要使用的嵌入式数据库的构建依赖项.

> 如果您在测试中使用此功能，您可能会注意到整个测试套件都会重复使用相同的数据库，无论您使用多少个应用程序上下文.如果要确保每个上下文都有一个单独的嵌入式数据库，则应将 `spring.datasource.generate-unique-name` 设置为 `true` .

例如，典型的POM依赖关系如下：

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
	<groupId>org.hsqldb</groupId>
	<artifactId>hsqldb</artifactId>
	<scope>runtime</scope>
</dependency>
```

> 您需要依赖 `spring-jdbc` 才能自动配置嵌入式数据库.在这个例子中，它通过 `spring-boot-starter-data-jpa` 传递.

> 如果出于某种原因，您确实为嵌入式数据库配置了连接URL，请注意确保禁用数据库的自动关闭.如果使用H2，则应使用 `DB_CLOSE_ON_EXIT=FALSE` 来执行此操作.如果使用HSQLDB，则应确保未使用 `shutdown=true` .禁用数据库的自动关闭可以在数据库关闭时进行Spring Boot控制，从而确保在不再需要访问数据库时发生.

### 30.1.2与生产环境数据库的连接

也可以使用池 `DataSource` 自动配置生产环境数据库连接. Spring Boot使用以下算法来选择特定的实现：

我们更喜欢HikariCP的性能和并发性.如果HikariCP可用，我们总是选择它.否则，如果Tomcat池化DataSource可用，我们将使用它.如果HikariCP和Tomcat池化数据源都不可用，并且Commons DBCP2可用，我们就会使用它.

如果使用 `spring-boot-starter-jdbc` 或 `spring-boot-starter-data-jpa` “starters”，则会自动获得 `HikariCP` 的依赖关系.

> 您可以完全绕过该算法，并通过设置 `spring.datasource.type` 属性指定要使用的连接池.如果您在Tomcat容器中运行应用程序，这一点尤为重要，因为默认情况下会提供 `tomcat-jdbc` .

> 始终可以手动配置其他连接池.如果您定义自己的 `DataSource`  bean，则不会进行自动配置.

DataSource配置由 `spring.datasource.*` 中的外部配置属性控制.例如，您可以在 `application.properties` 中声明以下部分：

```java
spring.datasource.url=jdbc:mysql://localhost/test
spring.datasource.username=dbuser
spring.datasource.password=dbpass
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

> 您至少应该通过设置 `spring.datasource.url` 属性来指定URL.否则，Spring Boot会尝试自动配置嵌入式数据库.

> 您通常不需要指定 `driver-class-name` ，因为Spring Boot可以从 `url` 中为大多数数据库推断出它.

> 对于要创建的池 `DataSource` ，我们需要能够验证有效的 `Driver` 类是否可用，因此我们在执行任何操作之前检查它.换句话说，如果设置 `spring.datasource.driver-class-name=com.mysql.jdbc.Driver` ，那么该类必须是可加载的.

有关更多支持的选项，请参阅[DataSourceProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jdbc/DataSourceProperties.java).无论实际实施如何，这些都是标准选项.也可以通过使用各自的前缀（ `spring.datasource.hikari.*` ， `spring.datasource.tomcat.*` 和 `spring.datasource.dbcp2.*` ）来微调特定于实现的设置.有关更多详细信息，请参阅您正在使用的连接池实现的文档.

例如，如果您使用[Tomcat connection pool](https://tomcat.apache.org/tomcat-8.0-doc/jdbc-pool.html#Common_Attributes)，则可以自定义许多其他设置，如以下示例所示：

```java
# Number of ms to wait before throwing an exception if no connection is available.
spring.datasource.tomcat.max-wait=10000

# Maximum number of active connections that can be allocated from this pool at the same time.
spring.datasource.tomcat.max-active=50

# Validate the connection before borrowing it from the pool.
spring.datasource.tomcat.test-on-borrow=true
```

### 30.1.3连接到JNDI数据源

如果将Spring Boot应用程序部署到Application Server，则可能需要使用Application Server的内置功能配置和管理DataSource，并使用JNDI访问它.

`spring.datasource.jndi-name` 属性可用作 `spring.datasource.url` ， `spring.datasource.username` 和 `spring.datasource.password` 属性的替代，以从特定JNDI位置访问 `DataSource` .例如， `application.properties` 中的以下部分显示了如何访问定义的_B84_ JBoss AS：

```java
spring.datasource.jndi-name=java:jboss/datasources/customers
```

## 30.2使用JdbcTemplate

Spring的 `JdbcTemplate` 和 `NamedParameterJdbcTemplate` 类是自动配置的，您可以将它们 `@Autowire` 直接放入您自己的bean中，如下所示例：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

	private final JdbcTemplate jdbcTemplate;

	@Autowired
	public MyBean(JdbcTemplate jdbcTemplate) {
		this.jdbcTemplate = jdbcTemplate;
	}

	// ...

}
```

您可以使用 `spring.jdbc.template.*` 属性自定义模板的某些属性，如以下示例所示：

```java
spring.jdbc.template.max-rows=500
```

>   `NamedParameterJdbcTemplate` 在幕后重复使用相同的 `JdbcTemplate` 实例.如果定义了多个 `JdbcTemplate` 且没有主要候选项，则不会自动配置 `NamedParameterJdbcTemplate` .

## 30.3 JPA和Spring Data JPA

Java Persistence API是一种标准技术，可让您将对象“映射”到关系数据库.  `spring-boot-starter-data-jpa`  POM提供了一种快速入门方式.它提供以下关键依赖项：

- Hibernate：最受欢迎的JPA实现之一.

- Spring Data JPA：使实现基于JPA的存储库变得容易.

- Spring ORMs：Spring Framework的核心ORM支持.

> 我们不会在这里详细介绍JPA或[Spring Data](https://projects.spring.io/spring-data/).您可以按照[spring.io](https://spring.io)的[“Accessing Data with JPA”](https://spring.io/guides/gs/accessing-data-jpa/)指南阅读[Spring Data JPA](https://projects.spring.io/spring-data-jpa/)和[Hibernate](https://hibernate.org/orm/documentation/)参考文档.

### 30.3.1实体类

传统上，JPA“实体”类在 `persistence.xml` 文件中指定.使用Spring Boot，此文件不是必需的，而是使用“实体扫描”.默认情况下，将搜索主配置类（注释为 `@EnableAutoConfiguration` 或 `@SpringBootApplication` ）下的所有包.

任何使用 `@Entity` ， `@Embeddable` 或 `@MappedSuperclass` 注释的类都会被考虑.典型的实体类类似于以下示例：

```java
package com.example.myapp.domain;

import java.io.Serializable;
import javax.persistence.*;

@Entity
public class City implements Serializable {

	@Id
	@GeneratedValue
	private Long id;

	@Column(nullable = false)
	private String name;

	@Column(nullable = false)
	private String state;

	// ... additional members, often include @OneToMany mappings

	protected City() {
		// no-args constructor required by JPA spec
		// this one is protected since it shouldn't be used directly
	}

	public City(String name, String state) {
		this.name = name;
		this.state = state;
	}

	public String getName() {
		return this.name;
	}

	public String getState() {
		return this.state;
	}

	// ... etc

}
```

> 您可以使用 `@EntityScan` 注释自定义实体扫描位置.请参阅“[Section 84.4, “Separate @Entity Definitions from Spring Configuration”](howto-data-access.html#howto-separate-entity-definitions-from-spring-configuration)”操作方法.

### 30.3.2 Spring Data JPA存储库

[Spring Data JPA](https://projects.spring.io/spring-data-jpa/)存储库是您可以定义以访问数据的接口. JPA查询是从您的方法名称自动创建的.例如， `CityRepository` 接口可能会声明一个 `findAllByState(String state)` 方法来查找给定状态中的所有城市.

对于更复杂的查询，您可以使用Spring Data的[Query](https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/Query.html)注释来注释您的方法.

Spring Data存储库通常从[Repository](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/Repository.html)或[CrudRepository](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html)接口扩展.如果使用自动配置，则会从包含主配置类（使用 `@EnableAutoConfiguration` 或 `@SpringBootApplication` 注释的包）的包中搜索存储库.

以下示例显示了典型的Spring Data存储库接口定义：

```java
package com.example.myapp.domain;

import org.springframework.data.domain.*;
import org.springframework.data.repository.*;

public interface CityRepository extends Repository<City, Long> {

	Page<City> findAll(Pageable pageable);

	City findByNameAndStateAllIgnoringCase(String name, String state);

}
```

Spring Data JPA存储库支持三种不同的引导模式：default，deferred和lazy.要启用延迟或延迟引导，请分别将 `spring.data.jpa.repositories.bootstrap-mode` 设置为 `deferred` 或 `lazy` .使用延迟或延迟引导时，自动配置的 `EntityManagerFactoryBuilder` 将使用上下文的异步任务执行程序（如果有）作为引导程序执行程序.

> 我们几乎没有涉及Spring Data JPA的表面.有关完整的详细信息，请参阅[Spring Data JPA reference documentation](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/).

### 30.3.3创建和删除JPA数据库

默认情况下，如果使用嵌入式数据库（H2，HSQL或Derby），则会自动创建JPA数据库 **only** .您可以使用 `spring.jpa.*` 属性显式配置JPA设置.例如，要创建和删除表，可以将以下行添加到 `application.properties` ：

```java
spring.jpa.hibernate.ddl-auto=create-drop
```

> Hibernate自己的内部属性名称（如果你碰巧记得更好）是 `hibernate.hbm2ddl.auto` .您可以使用 `spring.jpa.properties.*` （在将它们添加到实体管理器之前删除前缀）来设置它以及其他Hibernate本机属性.以下行显示了为Hibernate设置JPA属性的示例：

```java
spring.jpa.properties.hibernate.globally_quoted_identifiers=true
```

前面示例中的行将 `hibernate.globally_quoted_identifiers` 属性的值 `true` 传递给Hibernate实体管理器.

默认情况下，DDL执行（或验证）将延迟到 `ApplicationContext` 启动.还有一个 `spring.jpa.generate-ddl` 标志，但如果Hibernate自动配置处于活动状态，则不会使用它，因为 `ddl-auto` 设置更精细.

### 30.3.4在View中打开EntityManager

如果您正在运行Web应用程序，Spring Boot默认情况下会注册[OpenEntityManagerInViewInterceptor](https://docs.spring.io/spring/docs/5.1.2.RELEASE/javadoc-api/org/springframework/orm/jpa/support/OpenEntityManagerInViewInterceptor.html)以应用“在视图中打开EntityManager”模式，以允许在Web视图中延迟加载.如果您不想要此行为，则应在 `application.properties` 中将 `spring.jpa.open-in-view` 设置为 `false` .

## 30.4 Spring Data JDBC

Spring Data包含对JDBC的存储库支持，并将自动为 `CrudRepository` 上的方法生成SQL.对于更高级的查询，提供了 `@Query` 注释.

当必要的依赖项在类路径上时，Spring Boot将自动配置Spring Data的JDBC存储库.可以使用 `spring-boot-starter-data-jdbc` 上的单个依赖项将它们添加到项目中.如有必要，可以通过将 `@EnableJdbcRepositories` 注释或 `JdbcConfiguration` 子类添加到应用程序来控制Spring Data JDBC的配置.

> 有关Spring Data JDBC的完整详细信息，请参阅[reference documentation](https://projects.spring.io/spring-data-jdbc/).

## 30.5使用H2的Web控制台

[H2 database](http://www.h2database.com)提供[browser-based console](http://www.h2database.com/html/quickstart.html#h2_console)，Spring Boot可以为您自动配置.满足以下条件时，将自动配置控制台：

- 您正在开发基于servlet的Web应用程序.

-  `com.h2database:h2` 在类路径上.

- 您正在使用[Spring Boot’s developer tools](using-boot-devtools.html).

> 如果您没有使用Spring Boot的开发人员工具但仍想使用H2的控制台，则可以使用值为 `spring.h2.console.enabled` 属性配置 `true` .

>  H2控制台仅用于开发期间，因此您应该注意确保 `spring.h2.console.enabled` 未在生产环境中设置为 `true` .

### 30.5.1更改H2控制台的路径

默认情况下，控制台位于 `/h2-console` .您可以使用 `spring.h2.console.path` 属性自定义控制台的路径.

## 30.6使用jOOQ

Java面向对象查询（[jOOQ](http://www.jooq.org/)）是来自[Data Geekery](http://www.datageekery.com/)的流行产品，它从您的数据库生成Java代码，并允许您通过其流畅的API构建类型安全的SQL查询.商业版和开源版都可以与Spring Boot一起使用.

### 30.6.1代码生成

要使用jOOQ类型安全查询，您需要从数据库模式生成Java类.您可以按照[jOOQ user manual](https://www.jooq.org/doc/3.11.5/manual-single-page/#jooq-in-7-steps-step3)中的说明进行操作.如果您使用 `jooq-codegen-maven` 插件并且还使用 `spring-boot-starter-parent` “父POM”，则可以安全地省略插件的 `<version>` 标记.您还可以使用Spring Boot定义的版本变量（例如 `h2.version` ）来声明插件的数据库依赖性.以下清单显示了一个示例：

```xml
<plugin>
	<groupId>org.jooq</groupId>
	<artifactId>jooq-codegen-maven</artifactId>
	<executions>
		...
	</executions>
	<dependencies>
		<dependency>
			<groupId>com.h2database</groupId>
			<artifactId>h2</artifactId>
			<version>${h2.version}</version>
		</dependency>
	</dependencies>
	<configuration>
		<jdbc>
			<driver>org.h2.Driver</driver>
			<url>jdbc:h2:~/yourdatabase</url>
		</jdbc>
		<generator>
			...
		</generator>
	</configuration>
</plugin>
```

### 30.6.2使用DSLContext

jOOQ提供的流畅API通过 `org.jooq.DSLContext` 接口启动. Spring Boot自动将 `DSLContext` 配置为Spring Bean并将其连接到您的应用程序 `DataSource` .要使用 `DSLContext` ，您可以 `@Autowire` ，如下例所示：

```java
@Component
public class JooqExample implements CommandLineRunner {

	private final DSLContext create;

	@Autowired
	public JooqExample(DSLContext dslContext) {
		this.create = dslContext;
	}

}
```

>  jOOQ手册倾向于使用名为 `create` 的变量来保存 `DSLContext` .

然后，您可以使用 `DSLContext` 构建查询，如以下示例所示：

```java
public List<GregorianCalendar> authorsBornAfter1980() {
	return this.create.selectFrom(AUTHOR)
		.where(AUTHOR.DATE_OF_BIRTH.greaterThan(new GregorianCalendar(1980, 0, 1)))
		.fetch(AUTHOR.DATE_OF_BIRTH);
}
```

### 30.6.3 jOOQ SQL Dialect

除非已配置 `spring.jooq.sql-dialect` 属性，否则Spring Boot会确定用于数据源的SQL方言.如果Spring Boot无法检测到方言，则使用 `DEFAULT` .

> Spring Boot只能自动配置开源版本jOOQ支持的方言.

### 30.6.4自定义jOOQ

通过定义自己的 `@Bean` 定义可以实现更高级的自定义，这些定义在创建jOOQ  `Configuration` 时使用.您可以为以下jOOQ类型定义bean：

-  `ConnectionProvider` 

-  `ExecutorProvider` 

-  `TransactionProvider` 

-  `RecordMapperProvider` 

-  `RecordUnmapperProvider` 

-  `RecordListenerProvider` 

-  `ExecuteListenerProvider` 

-  `VisitListenerProvider` 

-  `TransactionListenerProvider` 

如果要完全控制jOOQ配置，也可以创建自己的 `org.jooq.Configuration`   `@Bean` .

