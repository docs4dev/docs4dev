## 30. Working with SQL Databases

The [Spring Framework](https://projects.spring.io/spring-framework/) provides extensive support for working with SQL databases, from direct JDBC access using  `JdbcTemplate`  to complete “object relational mapping” technologies such as Hibernate. [Spring Data](https://projects.spring.io/spring-data/) provides an additional level of functionality: creating  `Repository`  implementations directly from interfaces and using conventions to generate queries from your method names.

## 30.1 Configure a DataSource

Java’s  `javax.sql.DataSource`  interface provides a standard method of working with database connections. Traditionally, a 'DataSource' uses a  `URL`  along with some credentials to establish a database connection.

> See [the “How-to” section](howto-data-access.html#howto-configure-a-datasource) for more advanced examples, typically to take full control over the configuration of the DataSource.

### 30.1.1 Embedded Database Support

It is often convenient to develop applications by using an in-memory embedded database. Obviously, in-memory databases do not provide persistent storage. You need to populate your database when your application starts and be prepared to throw away data when your application ends.

> The “How-to” section includes a [section on how to initialize a database](howto-database-initialization.html).

Spring Boot can auto-configure embedded [H2](http://www.h2database.com), [HSQL](http://hsqldb.org/), and [Derby](https://db.apache.org/derby/) databases. You need not provide any connection URLs. You need only include a build dependency to the embedded database that you want to use.

> If you are using this feature in your tests, you may notice that the same database is reused by your whole test suite regardless of the number of application contexts that you use. If you want to make sure that each context has a separate embedded database, you should set  `spring.datasource.generate-unique-name`  to  `true` .

For example, the typical POM dependencies would be as follows:

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

> You need a dependency on  `spring-jdbc`  for an embedded database to be auto-configured. In this example, it is pulled in transitively through  `spring-boot-starter-data-jpa` .

> If, for whatever reason, you do configure the connection URL for an embedded database, take care to ensure that the database’s automatic shutdown is disabled. If you use H2, you should use  `DB_CLOSE_ON_EXIT=FALSE`  to do so. If you use HSQLDB, you should ensure that  `shutdown=true`  is not used. Disabling the database’s automatic shutdown lets Spring Boot control when the database is closed, thereby ensuring that it happens once access to the database is no longer needed.

### 30.1.2 Connection to a Production Database

Production database connections can also be auto-configured by using a pooling  `DataSource` . Spring Boot uses the following algorithm for choosing a specific implementation:

We prefer HikariCP for its performance and concurrency. If HikariCP is available, we always choose it. Otherwise, if the Tomcat pooling DataSource is available, we use it. If neither HikariCP nor the Tomcat pooling datasource are available and if Commons DBCP2 is available, we use it.

If you use the  `spring-boot-starter-jdbc`  or  `spring-boot-starter-data-jpa`  “starters”, you automatically get a dependency to  `HikariCP` .

> You can bypass that algorithm completely and specify the connection pool to use by setting the  `spring.datasource.type`  property. This is especially important if you run your application in a Tomcat container, as  `tomcat-jdbc`  is provided by default.

> Additional connection pools can always be configured manually. If you define your own  `DataSource`  bean, auto-configuration does not occur.

DataSource configuration is controlled by external configuration properties in  `spring.datasource.*` . For example, you might declare the following section in  `application.properties` :

```java
spring.datasource.url=jdbc:mysql://localhost/test
spring.datasource.username=dbuser
spring.datasource.password=dbpass
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

> You should at least specify the URL by setting the  `spring.datasource.url`  property. Otherwise, Spring Boot tries to auto-configure an embedded database.

> You often do not need to specify the  `driver-class-name` , since Spring Boot can deduce it for most databases from the  `url` .

> For a pooling  `DataSource`  to be created, we need to be able to verify that a valid  `Driver`  class is available, so we check for that before doing anything. In other words, if you set  `spring.datasource.driver-class-name=com.mysql.jdbc.Driver` , then that class has to be loadable.

See [DataSourceProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jdbc/DataSourceProperties.java) for more of the supported options. These are the standard options that work regardless of the actual implementation. It is also possible to fine-tune implementation-specific settings by using their respective prefix ( `spring.datasource.hikari.*` ,  `spring.datasource.tomcat.*` , and  `spring.datasource.dbcp2.*` ). Refer to the documentation of the connection pool implementation you are using for more details.

For instance, if you use the [Tomcat connection pool](https://tomcat.apache.org/tomcat-8.0-doc/jdbc-pool.html#Common_Attributes), you could customize many additional settings, as shown in the following example:

```java
# Number of ms to wait before throwing an exception if no connection is available.
spring.datasource.tomcat.max-wait=10000

# Maximum number of active connections that can be allocated from this pool at the same time.
spring.datasource.tomcat.max-active=50

# Validate the connection before borrowing it from the pool.
spring.datasource.tomcat.test-on-borrow=true
```

### 30.1.3 Connection to a JNDI DataSource

If you deploy your Spring Boot application to an Application Server, you might want to configure and manage your DataSource by using your Application Server’s built-in features and access it by using JNDI.

The  `spring.datasource.jndi-name`  property can be used as an alternative to the  `spring.datasource.url` ,  `spring.datasource.username` , and  `spring.datasource.password`  properties to access the  `DataSource`  from a specific JNDI location. For example, the following section in  `application.properties`  shows how you can access a JBoss AS defined  `DataSource` :

```java
spring.datasource.jndi-name=java:jboss/datasources/customers
```

## 30.2 Using JdbcTemplate

Spring’s  `JdbcTemplate`  and  `NamedParameterJdbcTemplate`  classes are auto-configured, and you can  `@Autowire`  them directly into your own beans, as shown in the following example:

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

You can customize some properties of the template by using the  `spring.jdbc.template.*`  properties, as shown in the following example:

```java
spring.jdbc.template.max-rows=500
```

> The  `NamedParameterJdbcTemplate`  reuses the same  `JdbcTemplate`  instance behind the scenes. If more than one  `JdbcTemplate`  is defined and no primary candidate exists, the  `NamedParameterJdbcTemplate`  is not auto-configured.

## 30.3 JPA and Spring Data JPA

The Java Persistence API is a standard technology that lets you “map” objects to relational databases. The  `spring-boot-starter-data-jpa`  POM provides a quick way to get started. It provides the following key dependencies:

- Hibernate: One of the most popular JPA implementations.

- Spring Data JPA: Makes it easy to implement JPA-based repositories.

- Spring ORMs: Core ORM support from the Spring Framework.

> We do not go into too many details of JPA or [Spring Data](https://projects.spring.io/spring-data/) here. You can follow the [“Accessing Data with JPA”](https://spring.io/guides/gs/accessing-data-jpa/) guide from [spring.io](https://spring.io) and read the [Spring Data JPA](https://projects.spring.io/spring-data-jpa/) and [Hibernate](https://hibernate.org/orm/documentation/) reference documentation.

### 30.3.1 Entity Classes

Traditionally, JPA “Entity” classes are specified in a  `persistence.xml`  file. With Spring Boot, this file is not necessary and “Entity Scanning” is used instead. By default, all packages below your main configuration class (the one annotated with  `@EnableAutoConfiguration`  or  `@SpringBootApplication` ) are searched.

Any classes annotated with  `@Entity` ,  `@Embeddable` , or  `@MappedSuperclass`  are considered. A typical entity class resembles the following example:

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

> You can customize entity scanning locations by using the  `@EntityScan`  annotation. See the “[Section 84.4, “Separate @Entity Definitions from Spring Configuration”](howto-data-access.html#howto-separate-entity-definitions-from-spring-configuration)” how-to.

### 30.3.2 Spring Data JPA Repositories

[Spring Data JPA](https://projects.spring.io/spring-data-jpa/) repositories are interfaces that you can define to access data. JPA queries are created automatically from your method names. For example, a  `CityRepository`  interface might declare a  `findAllByState(String state)`  method to find all the cities in a given state.

For more complex queries, you can annotate your method with Spring Data’s [Query](https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/Query.html) annotation.

Spring Data repositories usually extend from the [Repository](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/Repository.html) or [CrudRepository](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html) interfaces. If you use auto-configuration, repositories are searched from the package containing your main configuration class (the one annotated with  `@EnableAutoConfiguration`  or  `@SpringBootApplication` ) down.

The following example shows a typical Spring Data repository interface definition:

```java
package com.example.myapp.domain;

import org.springframework.data.domain.*;
import org.springframework.data.repository.*;

public interface CityRepository extends Repository<City, Long> {

	Page<City> findAll(Pageable pageable);

	City findByNameAndStateAllIgnoringCase(String name, String state);

}
```

Spring Data JPA repositories support three different modes of bootstrapping: default, deferred, and lazy. To enable deferred or lazy bootstrapping, set the  `spring.data.jpa.repositories.bootstrap-mode`  to  `deferred`  or  `lazy`  respectively. When using deferred or lazy bootstrapping, the auto-configured  `EntityManagerFactoryBuilder`  will use the context’s async task executor, if any, as the bootstrap executor.

> We have barely scratched the surface of Spring Data JPA. For complete details, see the [Spring Data JPA reference documentation](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/).

### 30.3.3 Creating and Dropping JPA Databases

By default, JPA databases are automatically created  **only**  if you use an embedded database (H2, HSQL, or Derby). You can explicitly configure JPA settings by using  `spring.jpa.*`  properties. For example, to create and drop tables you can add the following line to your  `application.properties` :

```java
spring.jpa.hibernate.ddl-auto=create-drop
```

> Hibernate’s own internal property name for this (if you happen to remember it better) is  `hibernate.hbm2ddl.auto` . You can set it, along with other Hibernate native properties, by using  `spring.jpa.properties.*`  (the prefix is stripped before adding them to the entity manager). The following line shows an example of setting JPA properties for Hibernate:

```java
spring.jpa.properties.hibernate.globally_quoted_identifiers=true
```

The line in the preceding example passes a value of  `true`  for the  `hibernate.globally_quoted_identifiers`  property to the Hibernate entity manager.

By default, the DDL execution (or validation) is deferred until the  `ApplicationContext`  has started. There is also a  `spring.jpa.generate-ddl`  flag, but it is not used if Hibernate auto-configuration is active, because the  `ddl-auto`  settings are more fine-grained.

### 30.3.4 Open EntityManager in View

If you are running a web application, Spring Boot by default registers [OpenEntityManagerInViewInterceptor](https://docs.spring.io/spring/docs/5.1.2.RELEASE/javadoc-api/org/springframework/orm/jpa/support/OpenEntityManagerInViewInterceptor.html) to apply the “Open EntityManager in View” pattern, to allow for lazy loading in web views. If you do not want this behavior, you should set  `spring.jpa.open-in-view`  to  `false`  in your  `application.properties` .

## 30.4 Spring Data JDBC

Spring Data includes repository support for JDBC and will automatically generate SQL for the methods on  `CrudRepository` . For more advanced queries, a  `@Query`  annotation is provided.

Spring Boot will auto-configure Spring Data’s JDBC repositories when the necessary dependencies are on the classpath. They can be added to your project with a single dependency on  `spring-boot-starter-data-jdbc` . If necessary, you can take control of Spring Data JDBC’s configuration by adding the  `@EnableJdbcRepositories`  annotation or a  `JdbcConfiguration`  subclass to your application.

> For complete details of Spring Data JDBC, please refer to the [reference documentation](https://projects.spring.io/spring-data-jdbc/).

## 30.5 Using H2’s Web Console

The [H2 database](http://www.h2database.com) provides a [browser-based console](http://www.h2database.com/html/quickstart.html#h2_console) that Spring Boot can auto-configure for you. The console is auto-configured when the following conditions are met:

- You are developing a servlet-based web application.

-  `com.h2database:h2`  is on the classpath.

- You are using [Spring Boot’s developer tools](using-boot-devtools.html).

> If you are not using Spring Boot’s developer tools but would still like to make use of H2’s console, you can configure the  `spring.h2.console.enabled`  property with a value of  `true` .

> The H2 console is only intended for use during development, so you should take care to ensure that  `spring.h2.console.enabled`  is not set to  `true`  in production.

### 30.5.1 Changing the H2 Console’s Path

By default, the console is available at  `/h2-console` . You can customize the console’s path by using the  `spring.h2.console.path`  property.

## 30.6 Using jOOQ

Java Object Oriented Querying ([jOOQ](http://www.jooq.org/)) is a popular product from [Data Geekery](http://www.datageekery.com/) which generates Java code from your database and lets you build type-safe SQL queries through its fluent API. Both the commercial and open source editions can be used with Spring Boot.

### 30.6.1 Code Generation

In order to use jOOQ type-safe queries, you need to generate Java classes from your database schema. You can follow the instructions in the [jOOQ user manual](https://www.jooq.org/doc/3.11.5/manual-single-page/#jooq-in-7-steps-step3). If you use the  `jooq-codegen-maven`  plugin and you also use the  `spring-boot-starter-parent`  “parent POM”, you can safely omit the plugin’s  `<version>`  tag. You can also use Spring Boot-defined version variables (such as  `h2.version` ) to declare the plugin’s database dependency. The following listing shows an example:

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

### 30.6.2 Using DSLContext

The fluent API offered by jOOQ is initiated through the  `org.jooq.DSLContext`  interface. Spring Boot auto-configures a  `DSLContext`  as a Spring Bean and connects it to your application  `DataSource` . To use the  `DSLContext` , you can  `@Autowire`  it, as shown in the following example:

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

> The jOOQ manual tends to use a variable named  `create`  to hold the  `DSLContext` .

You can then use the  `DSLContext`  to construct your queries, as shown in the following example:

```java
public List<GregorianCalendar> authorsBornAfter1980() {
	return this.create.selectFrom(AUTHOR)
		.where(AUTHOR.DATE_OF_BIRTH.greaterThan(new GregorianCalendar(1980, 0, 1)))
		.fetch(AUTHOR.DATE_OF_BIRTH);
}
```

### 30.6.3 jOOQ SQL Dialect

Unless the  `spring.jooq.sql-dialect`  property has been configured, Spring Boot determines the SQL dialect to use for your datasource. If Spring Boot could not detect the dialect, it uses  `DEFAULT` .

> Spring Boot can only auto-configure dialects supported by the open source version of jOOQ.

### 30.6.4 Customizing jOOQ

More advanced customizations can be achieved by defining your own  `@Bean`  definitions, which is used when the jOOQ  `Configuration`  is created. You can define beans for the following jOOQ Types:

-  `ConnectionProvider` 

-  `ExecutorProvider` 

-  `TransactionProvider` 

-  `RecordMapperProvider` 

-  `RecordUnmapperProvider` 

-  `RecordListenerProvider` 

-  `ExecuteListenerProvider` 

-  `VisitListenerProvider` 

-  `TransactionListenerProvider` 

You can also create your own  `org.jooq.Configuration`   `@Bean`  if you want to take complete control of the jOOQ configuration.

