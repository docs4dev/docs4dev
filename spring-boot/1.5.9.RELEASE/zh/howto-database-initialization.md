## 85.数据库初始化

可以使用不同的方式初始化SQL数据库，具体取决于堆栈的内容.当然，如果数据库是一个单独的进程，您也可以手动执行此操作.

## 85.1使用JPA初始化数据库

JPA具有DDL生成功能，可以将这些功能设置为在数据库启动时运行.这是通过两个外部属性控制的：

-  `spring.jpa.generate-ddl` （布尔值）打开和关闭该功能，与供应商无关.

-  `spring.jpa.hibernate.ddl-auto` （枚举）是一种Hibernate功能，它以更细粒度的方式控制行为.本指南后面将详细介绍此功能.

## 85.2使用Hibernate初始化数据库

您可以显式设置 `spring.jpa.hibernate.ddl-auto` ，标准Hibernate属性值为 `none` ， `validate` ， `update` ， `create` 和 `create-drop` . Spring Boot会根据是否认为您的数据库是嵌入式的，为您选择一个默认值.如果未检测到架构管理器，则默认为 `create-drop` ;在所有其他情况下，默认为 `none` .通过查看 `Connection` 类型来检测嵌入式数据库.嵌入了 `hsqldb` ， `h2` 和 `derby` ，而其他则不是.从内存切换到“真实”数据库时要小心，不要假设新平台中存在表和数据.您必须显式设置 `ddl-auto` 或使用其他机制之一来初始化数据库.

> 您可以通过启用 `org.hibernate.SQL` Logger来输出架构创建.如果启用[debug mode](boot-features-logging.html#boot-features-logging-console-output)，则会自动完成此操作.

此外，如果Hibernate从头开始创建模式（即，如果 `ddl-auto` 属性设置为 `create` 或 `create-drop` ），则会在启动时执行类路径根目录中名为 `import.sql` 的文件.这对于演示和测试很有用，如果你小心，但可能不是你想要在生产环境中的类路径上.它是一个Hibernate功能（与Spring无关）.

## 85.3初始化数据库

Spring Boot可以自动创建 `DataSource` 的模式（DDL脚本）并初始化它（DML脚本）.它从标准根类路径位置加载SQL： `schema.sql` 和 `data.sql` .此外，Spring Boot处理 `schema-${platform}.sql` 和 `data-${platform}.sql` 文件（如果存在），其中 `platform` 是 `spring.datasource.platform` 的值.这允许您在必要时切换到特定于数据库的脚本.例如，您可以选择将其设置为数据库的供应商名称（ `hsqldb` ， `h2` ， `oracle` ， `mysql` ， `postgresql` 等）.

> Spring Boot会自动创建嵌入式 `DataSource` 的架构.可以使用 `spring.datasource.initialization-mode` 属性自定义此行为.例如，如果要始终初始化 `DataSource` 而不管其类型如何：

默认情况下，Spring Boot启用Spring JDBC初始化程序的快速故障功能.这意味着，如果脚本导致异常，则应用程序无法启动.您可以通过设置 `spring.datasource.continue-on-error` 来调整该行为.

> 在基于JPA的应用程序中，您可以选择让Hibernate创建架构或使用 `schema.sql` ，但您不能同时执行这两种操作.如果使用 `schema.sql` ，请务必禁用 `spring.jpa.hibernate.ddl-auto` .

## 85.4初始化Spring Batch数据库

如果您使用Spring Batch，它会预先打包用于大多数流行数据库平台的SQL初始化脚本. Spring Boot可以检测您的数据库类型并在启动时执行这些脚本.如果使用嵌入式数据库，默认情况下会发生这种情况.您还可以为任何数据库类型启用它，如以下示例所示：

```java
spring.batch.initialize-schema=always
```

您也可以通过设置 `spring.batch.initialize-schema=never` 显式关闭初始化.

## 85.5使用更高级别的数据库迁移工具

Spring Boot支持两种更高级别的迁移工具：[Flyway](https://flywaydb.org/)和[Liquibase](http://www.liquibase.org/).

### 85.5.1在启动时执行Flyway数据库迁移

要在启动时自动运行Flyway数据库迁移，请将 `org.flywaydb:flyway-core` 添加到类路径中.

迁移是 `V<VERSION>__<NAME>.sql` 形式的脚本（ `<VERSION>` 是下划线分隔的版本，例如'1'或'2_1'）.默认情况下，它们位于名为 `classpath:db/migration` 的文件夹中，但您可以通过设置 `spring.flyway.locations` 来修改该位置.这是一个以逗号分隔的一个或多个 `classpath:` 或 `filesystem:` 位置的列表.例如，以下配置将在默认类路径位置和 `/opt/migration` 目录中搜索脚本：

```java
spring.flyway.locations=classpath:db/migration,filesystem:/opt/migration
```

您还可以添加特殊的 `{vendor}` 占位符以使用特定于供应商的脚本.假设如下：

```java
spring.flyway.locations=classpath:db/migration/{vendor}
```

前面的配置不是使用 `db/migration` ，而是根据数据库的类型设置要使用的文件夹（例如MySQL的 `db/migration/mysql` ）.支持的数据库列表在[DatabaseDriver](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jdbc/DatabaseDriver.java)中可用.

[FlywayProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/flyway/FlywayProperties.java)提供了大部分Flyway的设置和一小组其他属性，可用于禁用迁移或关闭位置检查.如果您需要更多控制配置，请考虑注册 `FlywayConfigurationCustomizer`  bean.

Spring Boot调用 `Flyway.migrate()` 来执行数据库迁移.如果您想要更多控制，请提供实现[FlywayMigrationStrategy](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/flyway/FlywayMigrationStrategy.java)的 `@Bean` .

Flyway支持SQL和Java [callbacks](https://flywaydb.org/documentation/callbacks.html).要使用基于SQL的回调，请将回调脚本放在 `classpath:db/migration` 文件夹中.要使用基于Java的回调，请创建一个或多个实现的bean `Callback` .任何此类bean都会自动注册 `Flyway` .可以使用 `@Order` 或通过实现 `Ordered` 来订购它们.也可以检测实现已弃用的 `FlywayCallback` 接口的Bean，但它们不能与 `Callback`  bean一起使用.

默认情况下，Flyway会在您的上下文中自动装配（ `@Primary` ） `DataSource` 并将其用于迁移.如果你想使用不同的 `DataSource` ，你可以创建一个并将其 `@Bean` 标记为 `@FlywayDataSource` .如果您这样做并想要两个数据源，请记住创建另一个数据源并将其标记为 `@Primary` .或者，您可以通过在外部属性中设置 `spring.flyway.[url,user,password]` 来使用Flyway的原生 `DataSource` .设置 `spring.flyway.url` 或 `spring.flyway.user` 足以使Flyway使用自己的 `DataSource` .如果未设置三个属性中的任何一个，将使用其等效 `spring.datasource` 属性的值.

有一个[Flyway sample](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-samples/spring-boot-sample-flyway)，以便您可以看到如何设置.

您还可以使用Flyway为特定方案提供数据.例如，您可以在 `src/test/resources` 中放置特定于测试的迁移，并且只有在应用程序启动进行测试时才会运行它们.此外，您可以使用特定于配置文件的配置来自定义 `spring.flyway.locations` ，以便某些迁移仅在特定配置文件处于活动状态时运行.例如，在 `application-dev.properties` 中，您可以指定以下设置：

```java
spring.flyway.locations=classpath:/db/migration,classpath:/dev/db/migration
```

使用该设置， `dev/db/migration` 中的迁移仅在 `dev` 配置文件处于活动状态时运行.

### 85.5.2在启动时执行Liquibase数据库迁移

要在启动时自动运行Liquibase数据库迁移，请将 `org.liquibase:liquibase-core` 添加到类路径中.

默认情况下，从 `db/changelog/db.changelog-master.yaml` 读取主更改日志，但您可以通过设置 `spring.liquibase.change-log` 来更改位置.除了YAML，Liquibase还支持JSON，XML和SQL更改日志格式.

默认情况下，Liquibase会在您的上下文中自动装配（ `@Primary` ） `DataSource` 并将其用于迁移.如果需要使用不同的 `DataSource` ，可以创建一个并将其 `@Bean` 标记为 `@LiquibaseDataSource` .如果您这样做并且想要两个数据源，请记住创建另一个数据源并将其标记为 `@Primary` .或者，您可以通过在外部属性中设置 `spring.liquibase.[url,user,password]` 来使用Liquibase的本机 `DataSource` .设置 `spring.liquibase.url` 或 `spring.liquibase.user` 足以使Liquibase使用自己的 `DataSource` .如果未设置三个属性中的任何一个，将使用其等效 `spring.datasource` 属性的值.

有关可用设置（如上下文，默认架构等）的详细信息，请参阅[LiquibaseProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/liquibase/LiquibaseProperties.java).

有一个[Liquibase sample](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-samples/spring-boot-sample-liquibase)，以便您可以看到如何设置.

