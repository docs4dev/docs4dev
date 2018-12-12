## 84.数据访问

Spring Boot包含许多用于处理数据源的启动器.本节回答与此相关的问题.

## 84.1配置自定义数据源

要配置自己的 `DataSource` ，请在配置中定义该类型的 `@Bean` . Spring Boot会在需要的任何地方重用 `DataSource` ，包括数据库初始化.如果需要外部化某些设置，可以将 `DataSource` 绑定到环境中（请参阅“[Section 24.8.1, “Third-party Configuration”](boot-features-external-config.html#boot-features-external-config-3rd-party-configuration)”）.

以下示例显示如何在bean中定义数据源：

```java
@Bean
@ConfigurationProperties(prefix="app.datasource")
public DataSource dataSource() {
	return new FancyDataSource();
}
```

以下示例显示如何通过设置属性来定义数据源：

```java
app.datasource.url=jdbc:h2:mem:mydb
app.datasource.username=sa
app.datasource.pool-size=30
```

假设您的 `FancyDataSource` 具有URL的常规JavaBean属性，用户名和池大小，这些设置会在 `DataSource` 可用于其他组件之前自动绑定.常规[database initialization](howto-database-initialization.html#howto-initialize-a-database-using-spring-jdbc)也会发生（因此 `spring.datasource.*` 的相关子集仍可用于您的自定义配置）.

Spring Boot还提供了一个名为 `DataSourceBuilder` 的实用程序构建器类，可用于创建其中一个标准数据源（如果它位于类路径中）.构建器可以根据类路径上的可用内容检测要使用的那个.它还会根据JDBC URL自动检测驱动程序.

以下示例显示如何使用 `DataSourceBuilder` 创建数据源：

```java
@Bean
@ConfigurationProperties("app.datasource")
public DataSource dataSource() {
	return DataSourceBuilder.create().build();
}
```

要使用 `DataSource` 运行应用程序，您只需要连接信息.还可以提供特定于池的设置.检查将在运行时使用的实现以获取更多详细信息.

以下示例说明如何通过设置属性来定义JDBC数据源：

```java
app.datasource.url=jdbc:mysql://localhost/test
app.datasource.username=dbuser
app.datasource.password=dbpass
app.datasource.pool-size=30
```

然而，有一个问题.由于未公开连接池的实际类型，因此您的自定义 `DataSource` 的元数据中不会生成任何键，并且IDE中没有可用的完成（因为 `DataSource` 接口不显示任何属性）.此外，如果你碰巧在类路径上有Hikari，这个基本设置不起作用，因为Hikari没有 `url` 属性（但确实有 `jdbcUrl` 属性）.在这种情况下，您必须按如下方式重写配置：

```java
app.datasource.jdbc-url=jdbc:mysql://localhost/test
app.datasource.username=dbuser
app.datasource.password=dbpass
app.datasource.maximum-pool-size=30
```

您可以通过强制连接池使用并返回专用实现而不是 `DataSource` 来解决此问题.您无法在运行时更改实现，但选项列表将是显式的.

以下示例显示如何使用 `DataSourceBuilder` 创建 `HikariDataSource` ：

```java
@Bean
@ConfigurationProperties("app.datasource")
public HikariDataSource dataSource() {
	return DataSourceBuilder.create().type(HikariDataSource.class).build();
}
```

您甚至可以通过利用 `DataSourceProperties` 为您做的事情来进一步发展 - 也就是说，如果没有提供URL，则通过提供具有合理用户名和密码的默认嵌入式数据库.您可以从任何 `DataSourceProperties` 对象的状态轻松初始化 `DataSourceBuilder` ，因此您也可以注入Spring Boot自动创建的DataSource.但是，这会将您的配置拆分为两个名称空间： `url` ， `username` ， `password` ， `type` 和 `driver` 在 `spring.datasource` 上，其余名称空间在您自定义名称空间（ `app.datasource` ）上.为避免这种情况，您可以在自定义命名空间上重新定义自定义 `DataSourceProperties` ，如以下示例所示：

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

默认情况下，此设置使您与Spring Boot为您执行的操作保持同步，除了选择了专用连接池（在代码中）并且其设置在 `app.datasource.configuration` 子命名空间中公开.因为 `DataSourceProperties` 正在为您处理 `url`  /  `jdbcUrl` 翻译，您可以按如下方式对其进行配置：

```java
app.datasource.url=jdbc:mysql://localhost/test
app.datasource.username=dbuser
app.datasource.password=dbpass
app.datasource.configuration.maximum-pool-size=30
```

> Spring Boot会将Hikari特定的设置暴露给 `spring.datasource.hikari` .此示例使用更通用的 `configuration` 子命名空间，因为该示例不支持多个数据源实现.

> B因为您的自定义配置选择使用Hikari， `app.datasource.type` 无效.在实践中，构建器初始化为您可能在那里设置的任何值，然后通过调用 `.type()` 来覆盖.

有关详细信息，请参阅“Spring Boot功能”部分中的“[Section 30.1, “Configure a DataSource”](boot-features-sql.html#boot-features-configure-datasource)”和[DataSourceAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jdbc/DataSourceAutoConfiguration.java)类.

## 84.2配置两个数据源

如果需要配置多个数据源，可以应用上一节中描述的相同技巧.但是，您必须将 `DataSource` 实例之一标记为 `@Primary` ，因为各种各样未来的自动配置期望能够按类型获得.

如果您创建自己的 `DataSource` ，则自动配置会退回.在以下示例中，我们提供与主数据源上提供的自动配置完全相同的功能集：

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

必须将
>  `firstDataSourceProperties` 标记为 `@Primary` ，以便数据库初始化程序功能使用您的副本（如果使用初始化程序）.

这两个数据源也绑定了高级自定义.例如，您可以按如下方式配置它们：

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

您也可以将相同的概念应用于辅助 `DataSource` ，如以下示例所示：

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

上面的示例使用与Spring Boot在自动配置中使用的逻辑相同的逻辑在自定义命名空间上配置两个数据源.请注意，每个 `configuration`  sub命名空间都根据所选实现提供高级设置.

## 84.3使用Spring Data Repositories

Spring Data可以创建各种风格的 `@Repository` 接口的实现. Spring Boot会为您处理所有这些，只要 `@Repositories` 包含在 `@EnableAutoConfiguration` 类的同一个包（或子包）中.

对于许多应用程序，您只需要在类路径上放置正确的Spring Data依赖项（JPA为 `spring-boot-starter-data-jpa` ，Mongodb为 `spring-boot-starter-data-mongodb` ）并创建一些存储库接口来处理 `@Entity` 对象.示例位于[JPA sample](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-samples/spring-boot-sample-data-jpa)和[Mongodb sample](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-samples/spring-boot-sample-data-mongodb)中.

Spring Boot会根据它找到的 `@EnableAutoConfiguration` 来猜测 `@Repository` 定义的位置.要获得更多控制，请使用 `@EnableJpaRepositories` 注释（来自Spring Data JPA）.

有关Spring Data的更多信息，请参阅[Spring Data project page](https://projects.spring.io/spring-data/).

## 84.4从Spring配置中分离@Entity定义

Spring Boot会根据它找到的 `@EnableAutoConfiguration` 来猜测 `@Entity` 定义的位置.要获得更多控制，可以使用 `@EntityScan` 注释，如以下示例所示：

```java
@Configuration
@EnableAutoConfiguration
@EntityScan(basePackageClasses=City.class)
public class Application {

	//...

}
```

## 84.5配置JPA属性

Spring Data JPA已经提供了一些独立于供应商的配置选项（例如用于SQL日志记录的选项），Spring Boot公开了这些选项以及Hibernate的一些选项作为外部配置属性.根据上下文自动检测其中一些，因此您不必设置它们.

`spring.jpa.hibernate.ddl-auto` 是一种特殊情况，因为根据运行时条件，它具有不同的默认值.如果使用嵌入式数据库且没有架构管理器（例如Liquibase或Flyway）正在处理 `DataSource` ，则默认为 `create-drop` .在所有其他情况下，它默认为 `none` .

使用的方言也会根据当前的 `DataSource` 自动检测，但如果您想要明确并且在启动时绕过该检查，则可以自己设置 `spring.jpa.database` .

> S指定 `database` 会导致配置明确定义的Hibernate方言.有几个数据库有多个 `Dialect` ，这可能不适合您的需求.在这种情况下，您可以将 `spring.jpa.database` 设置为 `default` 以让Hibernate解决问题或通过设置 `spring.jpa.database-platform` 属性来设置方言.

以下示例中显示了最常用的设置选项：

```java
spring.jpa.hibernate.naming.physical-strategy=com.example.MyPhysicalNamingStrategy
spring.jpa.show-sql=true
```

此外，当创建本地 `EntityManagerFactory` 时， `spring.jpa.properties.*` 中的所有属性都将作为普通JPA属性（带有前缀剥离）传递.

> 如果需要对Hibernate属性应用高级自定义，请考虑注册将在创建 `EntityManagerFactory` 之前调用的 `HibernatePropertiesCustomizer`  bean.这优先于自动配置应用的任何内容.

## 84.6配置Hibernate命名策略

Hibernate使用[two different naming strategies](https://docs.jboss.org/hibernate/orm/5.3/userguide/html_single/Hibernate_User_Guide.html#naming)将对象模型中的名称映射到相应的数据库名称.可以通过分别设置 `spring.jpa.hibernate.naming.physical-strategy` 和 `spring.jpa.hibernate.naming.implicit-strategy` 属性来配置物理策略实现和隐式策略实现的完全限定类名.或者，如果应用程序上下文中有 `ImplicitNamingStrategy` 或 `PhysicalNamingStrategy` beans，则Hibernate将自动配置为使用它们.

默认情况下，Spring Boot使用 `SpringPhysicalNamingStrategy` 配置物理命名策略.此实现提供了与Hibernate 4相同的表结构：所有点都被下划线替换，并且驼峰外壳也被下划线替换.默认情况下，所有表名都以小写形式生成，但如果您的架构需要，则可以覆盖该标志.

例如， `TelephoneNumber` 实体映射到 `telephone_number` 表.

如果您更喜欢使用Hibernate 5的默认设置，请设置以下属性：

```java
spring.jpa.hibernate.naming.physical-strategy=org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
```

或者，您可以配置以下bean：

```java
@Bean
public PhysicalNamingStrategy physicalNamingStrategy() {
	return new PhysicalNamingStrategyStandardImpl();
}
```

有关详细信息，请参阅[HibernateJpaAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/orm/jpa/HibernateJpaAutoConfiguration.java)和[JpaBaseConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/orm/jpa/JpaBaseConfiguration.java).

## 84.7配置Hibernate二级缓存

可以为一系列缓存提供程序配置Hibernate [second-level cache](https://docs.jboss.org/hibernate/orm/5.3/userguide/html_single/Hibernate_User_Guide.html#caching).不是将Hibernate配置为再次查找缓存提供程序，最好尽可能提供上下文中可用的那个.

如果你正在使用JCache，这很容易.首先，确保类路径上有 `org.hibernate:hibernate-jcache` 可用.然后，添加 `HibernatePropertiesCustomizer`  bean，如以下示例所示：

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

此自定义程序将配置Hibernate使用与应用程序使用的 `CacheManager` 相同的 `CacheManager` .也可以使用单独的 `CacheManager` 实例，请参阅[the Hibernate user guide](https://docs.jboss.org/hibernate/orm/5.3/userguide/html_single/Hibernate_User_Guide.html#caching-provider-jcache).

## 84.8在Hibernate组件中使用依赖注入

默认情况下，Spring Boot注册一个使用 `BeanFactory` 的 `BeanContainer` 实现，以便转换器和实体侦听器可以使用常规依赖注入.

您可以通过注册删除或更改 `hibernate.resource.beans.container` 属性的 `HibernatePropertiesCustomizer` 来禁用或调整此行为.

## 84.9使用自定义EntityManagerFactory

要完全控制 `EntityManagerFactory` 的配置，您需要添加名为'entityManagerFactory'的 `@Bean` . Spring Boot自动配置在存在该类型的bean的情况下关闭其实体管理器.

## 84.10使用两个EntityManagers

即使默认 `EntityManagerFactory` 工作正常，您也需要定义一个新的.否则，该类型的第二个bean的存在将关闭默认值.为了方便起见，您可以使用Spring Boot提供的方便的 `EntityManagerBuilder` .或者，您可以直接从Spring ORM中选择 `LocalContainerEntityManagerFactoryBean` ，如以下示例所示：

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

上面的配置几乎可以单独使用.要完成图片，您还需要为两个 `EntityManagers` 配置 `TransactionManagers` .如果您将其中一个标记为 `@Primary` ，则可以通过Spring Boot中的默认 `JpaTransactionManager` 获取它.另一个必须明确地注入新实例.或者，您可以使用跨越两者的JTA事务管理器.

如果使用Spring Data，则需要相应地配置 `@EnableJpaRepositories` ，如以下示例所示：

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

## 84.11使用传统的persistence.xml文件

Spring Boot默认不会搜索或使用 `META-INF/persistence.xml` .如果您更喜欢使用传统的 `persistence.xml` ，则需要定义自己的 `@Bean` 类型 `LocalEntityManagerFactoryBean` （ID为'entityManagerFactory'）并在其中设置持久性单元名称.

有关默认设置，请参阅[JpaBaseConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/orm/jpa/JpaBaseConfiguration.java).

## 84.12使用Spring Data JPA和Mongo存储库

Spring Data JPA和Spring Data Mongo都可以自动为您创建 `Repository` 实现.如果它们都存在于类路径中，则可能需要执行一些额外的配置来告诉Spring Boot要创建哪些存储库.最明确的方法是使用标准的Spring Data  `@EnableJpaRepositories` 和 `@EnableMongoRepositories` 注释，并提供 `Repository` 接口的位置.

还有一些标志（ `spring.data.*.repositories.enabled` 和 `spring.data.*.repositories.type` ）可用于在外部配置中打开和关闭自动配置的存储库.这样做很有用，例如，如果您想关闭Mongo存储库并仍然使用自动配置的 `MongoTemplate` .

其他自动配置的Spring Data存储库类型（Elasticsearch，Solr等）也存在相同的障碍和相同的功能.要使用它们，请相应地更改注释和标志的名称.

## 84.13自定义Spring Data的Web支持

Spring Data提供Web支持，简化了Web应用程序中Spring Data存储库的使用. Spring Boot在 `spring.data.web` 名称空间中提供属性以自定义其配置.请注意，如果您使用的是Spring Data REST，则必须使用 `spring.data.rest` 命名空间中的属性.

## 84.14将Spring数据存储库公开为RESTendpoints

如果为应用程序启用了Spring MVC，Spring Data REST可以将 `Repository` 实现公开为RESTendpoints.

Spring Boot公开了一组自定义[RepositoryRestConfiguration](https://docs.spring.io/spring-data/rest/docs/current/api/org/springframework/data/rest/core/config/RepositoryRestConfiguration.html)的有用属性（来自 `spring.data.rest` 命名空间）.如果需要提供其他自定义，则应使用[RepositoryRestConfigurer](https://docs.spring.io/spring-data/rest/docs/current/api/org/springframework/data/rest/webmvc/config/RepositoryRestConfigurer.html) bean.

> 如果您没有在自定义 `RepositoryRestConfigurer` 上指定任何顺序，它将在内部使用一个Spring Boot后运行.如果需要指定订单，请确保订单高于0.

## 84.15配置JPA使用的组件

如果要配置JPA使用的组件，则需要确保在JPA之前初始化组件.组件自动配置后，Spring Boot会为您解决此问题.例如，当自动配置Flyway时，Hibernate配置为依赖Flyway，以便Flyway有机会在Hibernate尝试使用它之前初始化数据库.

如果您自己配置组件，则可以使用 `EntityManagerFactoryDependsOnPostProcessor` 子类作为设置必要依赖项的便捷方法.例如，如果将Hibernate Search与Elasticsearch一起用作其索引管理器，则必须将任何 `EntityManagerFactory`  bean配置为依赖于 `elasticsearchClient`  bean，如以下示例所示：

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

## 84.16使用两个DataSource配置jOOQ

如果您需要将jOOQ与多个数据源一起使用，则应为每个数据源创建自己的 `DSLContext` .有关详细信息，请参阅[JooqAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jooq/JooqAutoConfiguration.java).

> 特别是， `JooqExceptionTranslator` 和 `SpringTransactionProvider` 可以重复使用，以提供与单个 `DataSource` 的自动配置相似的功能.

