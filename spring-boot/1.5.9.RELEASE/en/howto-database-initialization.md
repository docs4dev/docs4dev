## 85. Database Initialization

An SQL database can be initialized in different ways depending on what your stack is. Of course, you can also do it manually, provided the database is a separate process.

## 85.1 Initialize a Database Using JPA

JPA has features for DDL generation, and these can be set up to run on startup against the database. This is controlled through two external properties:

-  `spring.jpa.generate-ddl`  (boolean) switches the feature on and off and is vendor independent.

-  `spring.jpa.hibernate.ddl-auto`  (enum) is a Hibernate feature that controls the behavior in a more fine-grained way. This feature is described in more detail later in this guide.

## 85.2 Initialize a Database Using Hibernate

You can set  `spring.jpa.hibernate.ddl-auto`  explicitly and the standard Hibernate property values are  `none` ,  `validate` ,  `update` ,  `create` , and  `create-drop` . Spring Boot chooses a default value for you based on whether it thinks your database is embedded. It defaults to  `create-drop`  if no schema manager has been detected or  `none`  in all other cases. An embedded database is detected by looking at the  `Connection`  type.  `hsqldb` ,  `h2` , and  `derby`  are embedded, and others are not. Be careful when switching from in-memory to a ‘real’ database that you do not make assumptions about the existence of the tables and data in the new platform. You either have to set  `ddl-auto`  explicitly or use one of the other mechanisms to initialize the database.

> You can output the schema creation by enabling the  `org.hibernate.SQL`  logger. This is done for you automatically if you enable the [debug mode](boot-features-logging.html#boot-features-logging-console-output).

In addition, a file named  `import.sql`  in the root of the classpath is executed on startup if Hibernate creates the schema from scratch (that is, if the  `ddl-auto`  property is set to  `create`  or  `create-drop` ). This can be useful for demos and for testing if you are careful but is probably not something you want to be on the classpath in production. It is a Hibernate feature (and has nothing to do with Spring).

## 85.3 Initialize a Database

Spring Boot can automatically create the schema (DDL scripts) of your  `DataSource`  and initialize it (DML scripts). It loads SQL from the standard root classpath locations:  `schema.sql`  and  `data.sql` , respectively. In addition, Spring Boot processes the  `schema-${platform}.sql`  and  `data-${platform}.sql`  files (if present), where  `platform`  is the value of  `spring.datasource.platform` . This allows you to switch to database-specific scripts if necessary. For example, you might choose to set it to the vendor name of the database ( `hsqldb` ,  `h2` ,  `oracle` ,  `mysql` ,  `postgresql` , and so on).

> Spring Boot automatically creates the schema of an embedded  `DataSource` . This behaviour can be customized by using the  `spring.datasource.initialization-mode`  property. For instance, if you want to always initialize the  `DataSource`  regardless of its type:

By default, Spring Boot enables the fail-fast feature of the Spring JDBC initializer. This means that, if the scripts cause exceptions, the application fails to start. You can tune that behavior by setting  `spring.datasource.continue-on-error` .

> In a JPA-based app, you can choose to let Hibernate create the schema or use  `schema.sql` , but you cannot do both. Make sure to disable  `spring.jpa.hibernate.ddl-auto`  if you use  `schema.sql` .

## 85.4 Initialize a Spring Batch Database

If you use Spring Batch, it comes pre-packaged with SQL initialization scripts for most popular database platforms. Spring Boot can detect your database type and execute those scripts on startup. If you use an embedded database, this happens by default. You can also enable it for any database type, as shown in the following example:

```java
spring.batch.initialize-schema=always
```

You can also switch off the initialization explicitly by setting  `spring.batch.initialize-schema=never` .

## 85.5 Use a Higher-level Database Migration Tool

Spring Boot supports two higher-level migration tools: [Flyway](https://flywaydb.org/) and [Liquibase](http://www.liquibase.org/).

### 85.5.1 Execute Flyway Database Migrations on Startup

To automatically run Flyway database migrations on startup, add the  `org.flywaydb:flyway-core`  to your classpath.

The migrations are scripts in the form  `V<VERSION>__<NAME>.sql`  (with  `<VERSION>`  an underscore-separated version, such as ‘1’ or ‘2_1’). By default, they are in a folder called  `classpath:db/migration` , but you can modify that location by setting  `spring.flyway.locations` . This is a comma-separated list of one or more  `classpath:`  or  `filesystem:`  locations. For example, the following configuration would search for scripts in both the default classpath location and the  `/opt/migration`  directory:

```java
spring.flyway.locations=classpath:db/migration,filesystem:/opt/migration
```

You can also add a special  `{vendor}`  placeholder to use vendor-specific scripts. Assume the following:

```java
spring.flyway.locations=classpath:db/migration/{vendor}
```

Rather than using  `db/migration` , the preceding configuration sets the folder to use according to the type of the database (such as  `db/migration/mysql`  for MySQL). The list of supported databases is available in [DatabaseDriver](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jdbc/DatabaseDriver.java).

[FlywayProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/flyway/FlywayProperties.java) provides most of Flyway’s settings and a small set of additional properties that can be used to disable the migrations or switch off the location checking. If you need more control over the configuration, consider registering a  `FlywayConfigurationCustomizer`  bean.

Spring Boot calls  `Flyway.migrate()`  to perform the database migration. If you would like more control, provide a  `@Bean`  that implements [FlywayMigrationStrategy](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/flyway/FlywayMigrationStrategy.java).

Flyway supports SQL and Java [callbacks](https://flywaydb.org/documentation/callbacks.html). To use SQL-based callbacks, place the callback scripts in the  `classpath:db/migration`  folder. To use Java-based callbacks, create one or more beans that implement  `Callback` . Any such beans are automatically registered with  `Flyway` . They can be ordered by using  `@Order`  or by implementing  `Ordered` . Beans that implement the deprecated  `FlywayCallback`  interface can also be detected, however they cannot be used alongside  `Callback`  beans.

By default, Flyway autowires the ( `@Primary` )  `DataSource`  in your context and uses that for migrations. If you like to use a different  `DataSource` , you can create one and mark its  `@Bean`  as  `@FlywayDataSource` . If you do so and want two data sources, remember to create another one and mark it as  `@Primary` . Alternatively, you can use Flyway’s native  `DataSource`  by setting  `spring.flyway.[url,user,password]`  in external properties. Setting either  `spring.flyway.url`  or  `spring.flyway.user`  is sufficient to cause Flyway to use its own  `DataSource` . If any of the three properties has not be set, the value of its equivalent  `spring.datasource`  property will be used.

There is a [Flyway sample](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-samples/spring-boot-sample-flyway) so that you can see how to set things up.

You can also use Flyway to provide data for specific scenarios. For example, you can place test-specific migrations in  `src/test/resources`  and they are run only when your application starts for testing. Also, you can use profile-specific configuration to customize  `spring.flyway.locations`  so that certain migrations run only when a particular profile is active. For example, in  `application-dev.properties` , you might specify the following setting:

```java
spring.flyway.locations=classpath:/db/migration,classpath:/dev/db/migration
```

With that setup, migrations in  `dev/db/migration`  run only when the  `dev`  profile is active.

### 85.5.2 Execute Liquibase Database Migrations on Startup

To automatically run Liquibase database migrations on startup, add the  `org.liquibase:liquibase-core`  to your classpath.

By default, the master change log is read from  `db/changelog/db.changelog-master.yaml` , but you can change the location by setting  `spring.liquibase.change-log` . In addition to YAML, Liquibase also supports JSON, XML, and SQL change log formats.

By default, Liquibase autowires the ( `@Primary` )  `DataSource`  in your context and uses that for migrations. If you need to use a different  `DataSource` , you can create one and mark its  `@Bean`  as  `@LiquibaseDataSource` . If you do so and you want two data sources, remember to create another one and mark it as  `@Primary` . Alternatively, you can use Liquibase’s native  `DataSource`  by setting  `spring.liquibase.[url,user,password]`  in external properties. Setting either  `spring.liquibase.url`  or  `spring.liquibase.user`  is sufficient to cause Liquibase to use its own  `DataSource` . If any of the three properties has not be set, the value of its equivalent  `spring.datasource`  property will be used.

See [LiquibaseProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/liquibase/LiquibaseProperties.java) for details about available settings such as contexts, the default schema, and others.

There is a [Liquibase sample](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-samples/spring-boot-sample-liquibase) so that you can see how to set things up.

