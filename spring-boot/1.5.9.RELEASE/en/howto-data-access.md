## 84. Data Access

Spring Boot includes a number of starters for working with data sources. This section answers questions related to doing so.

## 84.1 Configure a Custom DataSource

To configure your own  `DataSource` , define a  `@Bean`  of that type in your configuration. Spring Boot reuses your  `DataSource`  anywhere one is required, including database initialization. If you need to externalize some settings, you can bind your  `DataSource`  to the environment (see “[Section 24.8.1, “Third-party Configuration”](boot-features-external-config.html#boot-features-external-config-3rd-party-configuration)”).

The following example shows how to define a data source in a bean:

```java
@Bean
@ConfigurationProperties(prefix="app.datasource")
public DataSource dataSource() {
	return new FancyDataSource();
}
```

The following example shows how to define a data source by setting properties:

```java
app.datasource.url=jdbc:h2:mem:mydb
app.datasource.username=sa
app.datasource.pool-size=30
```

Assuming that your  `FancyDataSource`  has regular JavaBean properties for the URL, the username, and the pool size, these settings are bound automatically before the  `DataSource`  is made available to other components. The regular [database initialization](howto-database-initialization.html#howto-initialize-a-database-using-spring-jdbc) also happens (so the relevant sub-set of  `spring.datasource.*`  can still be used with your custom configuration).

Spring Boot also provides a utility builder class, called  `DataSourceBuilder` , that can be used to create one of the standard data sources (if it is on the classpath). The builder can detect the one to use based on what’s available on the classpath. It also auto-detects the driver based on the JDBC URL.

The following example shows how to create a data source by using a  `DataSourceBuilder` :

```java
@Bean
@ConfigurationProperties("app.datasource")
public DataSource dataSource() {
	return DataSourceBuilder.create().build();
}
```

To run an app with that  `DataSource` , all you need is the connection information. Pool-specific settings can also be provided. Check the implementation that is going to be used at runtime for more details.

The following example shows how to define a JDBC data source by setting properties:

```java
app.datasource.url=jdbc:mysql://localhost/test
app.datasource.username=dbuser
app.datasource.password=dbpass
app.datasource.pool-size=30
```

However, there is a catch. Because the actual type of the connection pool is not exposed, no keys are generated in the metadata for your custom  `DataSource`  and no completion is available in your IDE (because the  `DataSource`  interface exposes no properties). Also, if you happen to have Hikari on the classpath, this basic setup does not work, because Hikari has no  `url`  property (but does have a  `jdbcUrl`  property). In that case, you must rewrite your configuration as follows:

```java
app.datasource.jdbc-url=jdbc:mysql://localhost/test
app.datasource.username=dbuser
app.datasource.password=dbpass
app.datasource.maximum-pool-size=30
```

You can fix that by forcing the connection pool to use and return a dedicated implementation rather than  `DataSource` . You cannot change the implementation at runtime, but the list of options will be explicit.

The following example shows how create a  `HikariDataSource`  with  `DataSourceBuilder` :

```java
@Bean
@ConfigurationProperties("app.datasource")
public HikariDataSource dataSource() {
	return DataSourceBuilder.create().type(HikariDataSource.class).build();
}
```

You can even go further by leveraging what  `DataSourceProperties`  does for you — that is, by providing a default embedded database with a sensible username and password if no URL is provided. You can easily initialize a  `DataSourceBuilder`  from the state of any  `DataSourceProperties`  object, so you could also inject the DataSource that Spring Boot creates automatically. However, that would split your configuration into two namespaces:  `url` ,  `username` ,  `password` ,  `type` , and  `driver`  on  `spring.datasource`  and the rest on your custom namespace ( `app.datasource` ). To avoid that, you can redefine a custom  `DataSourceProperties`  on your custom namespace, as shown in the following example:

```java
@Bean
@Primary
@ConfigurationProperties("app.datasource")
public DataSourceProperties dataSourceProperties() {
	return new DataSourceProperties();
}

@Bean
@ConfigurationProperties("app.datasource.configuration")
public HikariDataSource dataSource(DataSourceProperties properties) {
	return properties.initializeDataSourceBuilder().type(HikariDataSource.class)
			.build();
}
```

This setup puts you in sync with what Spring Boot does for you by default, except that a dedicated connection pool is chosen (in code) and its settings are exposed in the  `app.datasource.configuration`  sub namespace. Because  `DataSourceProperties`  is taking care of the  `url` / `jdbcUrl`  translation for you, you can configure it as follows:

```java
app.datasource.url=jdbc:mysql://localhost/test
app.datasource.username=dbuser
app.datasource.password=dbpass
app.datasource.configuration.maximum-pool-size=30
```

> Spring Boot will expose Hikari-specific settings to  `spring.datasource.hikari` . This example uses a more generic  `configuration`  sub namespace as the example does not support multiple datasource implementations.

> Because your custom configuration chooses to go with Hikari,  `app.datasource.type`  has no effect. In practice, the builder is initialized with whatever value you might set there and then overridden by the call to  `.type()` .

See “[Section 30.1, “Configure a DataSource”](boot-features-sql.html#boot-features-configure-datasource)” in the “Spring Boot features” section and the [DataSourceAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jdbc/DataSourceAutoConfiguration.java) class for more details.

## 84.2 Configure Two DataSources

If you need to configure multiple data sources, you can apply the same tricks that are described in the previous section. You must, however, mark one of the  `DataSource`  instances as  `@Primary` , because various auto-configurations down the road expect to be able to get one by type.

If you create your own  `DataSource` , the auto-configuration backs off. In the following example, we provide the exact same feature set as the auto-configuration provides on the primary data source:

```java
@Bean
@Primary
@ConfigurationProperties("app.datasource.first")
public DataSourceProperties firstDataSourceProperties() {
	return new DataSourceProperties();
}

@Bean
@Primary
@ConfigurationProperties("app.datasource.first.configuration")
public HikariDataSource firstDataSource() {
	return firstDataSourceProperties().initializeDataSourceBuilder()
			.type(HikariDataSource.class).build();
}

@Bean
@ConfigurationProperties("app.datasource.second")
public BasicDataSource secondDataSource() {
	return DataSourceBuilder.create().type(BasicDataSource.class).build();
}
```

>  `firstDataSourceProperties`  has to be flagged as  `@Primary`  so that the database initializer feature uses your copy (if you use the initializer).

Both data sources are also bound for advanced customizations. For instance, you could configure them as follows:

```java
app.datasource.first.url=jdbc:mysql://localhost/first
app.datasource.first.username=dbuser
app.datasource.first.password=dbpass
app.datasource.first.configuration.maximum-pool-size=30

app.datasource.second.url=jdbc:mysql://localhost/second
app.datasource.second.username=dbuser
app.datasource.second.password=dbpass
app.datasource.second.max-total=30
```

You can apply the same concept to the secondary  `DataSource`  as well, as shown in the following example:

```java
@Bean
@Primary
@ConfigurationProperties("app.datasource.first")
public DataSourceProperties firstDataSourceProperties() {
	return new DataSourceProperties();
}

@Bean
@Primary
@ConfigurationProperties("app.datasource.first.configuration")
public HikariDataSource firstDataSource() {
	return firstDataSourceProperties().initializeDataSourceBuilder()
			.type(HikariDataSource.class).build();
}

@Bean
@ConfigurationProperties("app.datasource.second")
public DataSourceProperties secondDataSourceProperties() {
	return new DataSourceProperties();
}

@Bean
@ConfigurationProperties("app.datasource.second.configuration")
public BasicDataSource secondDataSource() {
	return secondDataSourceProperties().initializeDataSourceBuilder()
			.type(BasicDataSource.class).build();
}
```

The preceding example configures two data sources on custom namespaces with the same logic as Spring Boot would use in auto-configuration. Note that each  `configuration`  sub namespace provides advanced settings based on the chosen implementation.

## 84.3 Use Spring Data Repositories

Spring Data can create implementations of  `@Repository`  interfaces of various flavors. Spring Boot handles all of that for you, as long as those  `@Repositories`  are included in the same package (or a sub-package) of your  `@EnableAutoConfiguration`  class.

For many applications, all you need is to put the right Spring Data dependencies on your classpath (there is a  `spring-boot-starter-data-jpa`  for JPA and a  `spring-boot-starter-data-mongodb`  for Mongodb) and create some repository interfaces to handle your  `@Entity`  objects. Examples are in the [JPA sample](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-samples/spring-boot-sample-data-jpa) and the [Mongodb sample](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-samples/spring-boot-sample-data-mongodb).

Spring Boot tries to guess the location of your  `@Repository`  definitions, based on the  `@EnableAutoConfiguration`  it finds. To get more control, use the  `@EnableJpaRepositories`  annotation (from Spring Data JPA).

For more about Spring Data, see the [Spring Data project page](https://projects.spring.io/spring-data/).

## 84.4 Separate @Entity Definitions from Spring Configuration

Spring Boot tries to guess the location of your  `@Entity`  definitions, based on the  `@EnableAutoConfiguration`  it finds. To get more control, you can use the  `@EntityScan`  annotation, as shown in the following example:

```java
@Configuration
@EnableAutoConfiguration
@EntityScan(basePackageClasses=City.class)
public class Application {

	//...

}
```

## 84.5 Configure JPA Properties

Spring Data JPA already provides some vendor-independent configuration options (such as those for SQL logging), and Spring Boot exposes those options and a few more for Hibernate as external configuration properties. Some of them are automatically detected according to the context so you should not have to set them.

The  `spring.jpa.hibernate.ddl-auto`  is a special case, because, depending on runtime conditions, it has different defaults. If an embedded database is used and no schema manager (such as Liquibase or Flyway) is handling the  `DataSource` , it defaults to  `create-drop` . In all other cases, it defaults to  `none` .

The dialect to use is also automatically detected based on the current  `DataSource` , but you can set  `spring.jpa.database`  yourself if you want to be explicit and bypass that check on startup.

> Specifying a  `database`  leads to the configuration of a well-defined Hibernate dialect. Several databases have more than one  `Dialect` , and this may not suit your needs. In that case, you can either set  `spring.jpa.database`  to  `default`  to let Hibernate figure things out or set the dialect by setting the  `spring.jpa.database-platform`  property.

The most common options to set are shown in the following example:

```java
spring.jpa.hibernate.naming.physical-strategy=com.example.MyPhysicalNamingStrategy
spring.jpa.show-sql=true
```

In addition, all properties in  `spring.jpa.properties.*`  are passed through as normal JPA properties (with the prefix stripped) when the local  `EntityManagerFactory`  is created.

> If you need to apply advanced customization to Hibernate properties, consider registering a  `HibernatePropertiesCustomizer`  bean that will be invoked prior to creating the  `EntityManagerFactory` . This takes precedence to anything that is applied by the auto-configuration.

## 84.6 Configure Hibernate Naming Strategy

Hibernate uses [two different naming strategies](https://docs.jboss.org/hibernate/orm/5.3/userguide/html_single/Hibernate_User_Guide.html#naming) to map names from the object model to the corresponding database names. The fully qualified class name of the physical and the implicit strategy implementations can be configured by setting the  `spring.jpa.hibernate.naming.physical-strategy`  and  `spring.jpa.hibernate.naming.implicit-strategy`  properties, respectively. Alternatively, if  `ImplicitNamingStrategy`  or  `PhysicalNamingStrategy`  beans are available in the application context, Hibernate will be automatically configured to use them.

By default, Spring Boot configures the physical naming strategy with  `SpringPhysicalNamingStrategy` . This implementation provides the same table structure as Hibernate 4: all dots are replaced by underscores and camel casing is replaced by underscores as well. By default, all table names are generated in lower case, but it is possible to override that flag if your schema requires it.

For example, a  `TelephoneNumber`  entity is mapped to the  `telephone_number`  table.

If you prefer to use Hibernate 5’s default instead, set the following property:

```java
spring.jpa.hibernate.naming.physical-strategy=org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
```

Alternatively, you can configure the following bean:

```java
@Bean
public PhysicalNamingStrategy physicalNamingStrategy() {
	return new PhysicalNamingStrategyStandardImpl();
}
```

See [HibernateJpaAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/orm/jpa/HibernateJpaAutoConfiguration.java) and [JpaBaseConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/orm/jpa/JpaBaseConfiguration.java) for more details.

## 84.7 Configure Hibernate Second-Level Caching

Hibernate [second-level cache](https://docs.jboss.org/hibernate/orm/5.3/userguide/html_single/Hibernate_User_Guide.html#caching) can be configured for a range of cache providers. Rather than configuring Hibernate to lookup the cache provider again, it is better to provide the one that is available in the context whenever possible.

If you’re using JCache, this is pretty easy. First, make sure that  `org.hibernate:hibernate-jcache`  is available on the classpath. Then, add a  `HibernatePropertiesCustomizer`  bean as shown in the following example:

```java
@Configuration
public class HibernateSecondLevelCacheExample {

	@Bean
	public HibernatePropertiesCustomizer hibernateSecondLevelCacheCustomizer(
			JCacheCacheManager cacheManager) {
		return (properties) -> properties.put(ConfigSettings.CACHE_MANAGER,
				cacheManager.getCacheManager());

	}

}
```

This customizer will configure Hibernate to use the same  `CacheManager`  as the one that the application uses. It is also possible to use separate  `CacheManager`  instances, refer to [the Hibernate user guide](https://docs.jboss.org/hibernate/orm/5.3/userguide/html_single/Hibernate_User_Guide.html#caching-provider-jcache).

## 84.8 Use Dependency Injection in Hibernate Components

By default, Spring Boot registers a  `BeanContainer`  implementation that uses the  `BeanFactory`  so that converters and entity listeners can use regular dependency injection.

You can disable or tune this behaviour by registering a  `HibernatePropertiesCustomizer`  that removes or changes the  `hibernate.resource.beans.container`  property.

## 84.9 Use a Custom EntityManagerFactory

To take full control of the configuration of the  `EntityManagerFactory` , you need to add a  `@Bean`  named ‘entityManagerFactory’. Spring Boot auto-configuration switches off its entity manager in the presence of a bean of that type.

## 84.10 Use Two EntityManagers

Even if the default  `EntityManagerFactory`  works fine, you need to define a new one. Otherwise, the presence of the second bean of that type switches off the default. To make it easy to do, you can use the convenient  `EntityManagerBuilder`  provided by Spring Boot. Alternatively, you can just the  `LocalContainerEntityManagerFactoryBean`  directly from Spring ORM, as shown in the following example:

```java
// add two data sources configured as above

@Bean
public LocalContainerEntityManagerFactoryBean customerEntityManagerFactory(
		EntityManagerFactoryBuilder builder) {
	return builder
			.dataSource(customerDataSource())
			.packages(Customer.class)
			.persistenceUnit("customers")
			.build();
}

@Bean
public LocalContainerEntityManagerFactoryBean orderEntityManagerFactory(
		EntityManagerFactoryBuilder builder) {
	return builder
			.dataSource(orderDataSource())
			.packages(Order.class)
			.persistenceUnit("orders")
			.build();
}
```

The configuration above almost works on its own. To complete the picture, you need to configure  `TransactionManagers`  for the two  `EntityManagers`  as well. If you mark one of them as  `@Primary` , it could be picked up by the default  `JpaTransactionManager`  in Spring Boot. The other would have to be explicitly injected into a new instance. Alternatively, you might be able to use a JTA transaction manager that spans both.

If you use Spring Data, you need to configure  `@EnableJpaRepositories`  accordingly, as shown in the following example:

```java
@Configuration
@EnableJpaRepositories(basePackageClasses = Customer.class,
		entityManagerFactoryRef = "customerEntityManagerFactory")
public class CustomerConfiguration {
	...
}

@Configuration
@EnableJpaRepositories(basePackageClasses = Order.class,
		entityManagerFactoryRef = "orderEntityManagerFactory")
public class OrderConfiguration {
	...
}
```

## 84.11 Use a Traditional persistence.xml File

Spring Boot will not search for or use a  `META-INF/persistence.xml`  by default. If you prefer to use a traditional  `persistence.xml` , you need to define your own  `@Bean`  of type  `LocalEntityManagerFactoryBean`  (with an ID of ‘entityManagerFactory’) and set the persistence unit name there.

See [JpaBaseConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/orm/jpa/JpaBaseConfiguration.java) for the default settings.

## 84.12 Use Spring Data JPA and Mongo Repositories

Spring Data JPA and Spring Data Mongo can both automatically create  `Repository`  implementations for you. If they are both present on the classpath, you might have to do some extra configuration to tell Spring Boot which repositories to create. The most explicit way to do that is to use the standard Spring Data  `@EnableJpaRepositories`  and  `@EnableMongoRepositories`  annotations and provide the location of your  `Repository`  interfaces.

There are also flags ( `spring.data.*.repositories.enabled`  and  `spring.data.*.repositories.type` ) that you can use to switch the auto-configured repositories on and off in external configuration. Doing so is useful, for instance, in case you want to switch off the Mongo repositories and still use the auto-configured  `MongoTemplate` .

The same obstacle and the same features exist for other auto-configured Spring Data repository types (Elasticsearch, Solr, and others). To work with them, change the names of the annotations and flags accordingly.

## 84.13 Customize Spring Data’s Web Support

Spring Data provides web support that simplifies the use of Spring Data repositories in a web application. Spring Boot provides properties in the  `spring.data.web`  namespace for customizing its configuration. Note that if you are using Spring Data REST, you must use the properties in the  `spring.data.rest`  namespace instead.

## 84.14 Expose Spring Data Repositories as REST Endpoint

Spring Data REST can expose the  `Repository`  implementations as REST endpoints for you, provided Spring MVC has been enabled for the application.

Spring Boot exposes a set of useful properties (from the  `spring.data.rest`  namespace) that customize the [RepositoryRestConfiguration](https://docs.spring.io/spring-data/rest/docs/current/api/org/springframework/data/rest/core/config/RepositoryRestConfiguration.html). If you need to provide additional customization, you should use a [RepositoryRestConfigurer](https://docs.spring.io/spring-data/rest/docs/current/api/org/springframework/data/rest/webmvc/config/RepositoryRestConfigurer.html) bean.

> If you do not specify any order on your custom  `RepositoryRestConfigurer` , it runs after the one Spring Boot uses internally. If you need to specify an order, make sure it is higher than 0.

## 84.15 Configure a Component that is Used by JPA

If you want to configure a component that JPA uses, then you need to ensure that the component is initialized before JPA. When the component is auto-configured, Spring Boot takes care of this for you. For example, when Flyway is auto-configured, Hibernate is configured to depend upon Flyway so that Flyway has a chance to initialize the database before Hibernate tries to use it.

If you are configuring a component yourself, you can use an  `EntityManagerFactoryDependsOnPostProcessor`  subclass as a convenient way of setting up the necessary dependencies. For example, if you use Hibernate Search with Elasticsearch as its index manager, any  `EntityManagerFactory`  beans must be configured to depend on the  `elasticsearchClient`  bean, as shown in the following example:

```java
/**
* {@link EntityManagerFactoryDependsOnPostProcessor} that ensures that
* {@link EntityManagerFactory} beans depend on the {@code elasticsearchClient} bean.
*/
@Configuration
static class ElasticsearchJpaDependencyConfiguration
		extends EntityManagerFactoryDependsOnPostProcessor {

	ElasticsearchJpaDependencyConfiguration() {
		super("elasticsearchClient");
	}

}
```

## 84.16 Configure jOOQ with Two DataSources

If you need to use jOOQ with multiple data sources, you should create your own  `DSLContext`  for each one. Refer to [JooqAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jooq/JooqAutoConfiguration.java) for more details.

> In particular,  `JooqExceptionTranslator`  and  `SpringTransactionProvider`  can be reused to provide similar features to what the auto-configuration does with a single  `DataSource` .

