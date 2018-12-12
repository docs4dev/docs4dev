## 105.数据库后端

Vault支持多个数据库机密后端，以根据配置的角色动态生成数据库凭据.这意味着需要访问数据库的服务不再需要配置凭据：他们可以从Vault请求它们，并使用Vault的租赁机制更轻松地滚动密钥.

Spring Cloud Vault与这些后端集成：

- [Section 105.1, “Database”](multi_vault.config.backends.database-backends.html#vault.config.backends.database)

- [Section 105.2, “Apache Cassandra”](multi_vault.config.backends.database-backends.html#vault.config.backends.cassandra)

- [Section 105.3, “MongoDB”](multi_vault.config.backends.database-backends.html#vault.config.backends.mongodb)

- [Section 105.4, “MySQL”](multi_vault.config.backends.database-backends.html#vault.config.backends.mysql)

- [Section 105.5, “PostgreSQL”](multi_vault.config.backends.database-backends.html#vault.config.backends.postgresql)

使用数据库密钥后端需要在配置中启用后端和 `spring-cloud-vault-config-databases` 依赖项.

自0.7.1以来，Vault发布了一个专用的 `database` 秘密后端，允许通过插件进行数据库集成.您可以使用通用数据库后端来使用该特定后端.确保指定适当的后端路径，例如 `spring.cloud.vault.mysql.role.backend=database` .

**Example 105.1. pom.xml** 

```xml
<dependencies>
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-vault-config-databases</artifactId>
<version>Finchley.SR2</version>
</dependency>
</dependencies>
```

> 启用多个符合JDBC的数据库将生成凭据并默认将它们存储在相同的属性键中，因此需要单独配置JDBC机密的属性名称.

## 105.1数据库

Spring Cloud Vault可以获取[https://www.vaultproject.io/api/secret/databases/index.html](https://www.vaultproject.io/api/secret/databases/index.html)中列出的任何数据库的凭据.可以通过设置 `spring.cloud.vault.database.enabled=true` （默认 `false` ）并为角色名称 `spring.cloud.vault.database.role=…` 启用集成.

虽然数据库后端是通用的，但 `spring.cloud.vault.database` 专门针对JDBC数据库.用户名和密码存储在 `spring.datasource.username` 和 `spring.datasource.password` 中，因此使用Spring Boot将为 `DataSource` 获取生成的凭据，无需进一步配置.您可以通过设置 `spring.cloud.vault.database.username-property` 和 `spring.cloud.vault.database.password-property` 来配置属性名称.

```java
spring.cloud.vault:
database:
enabled: true
role: readonly
backend: database
username-property: spring.datasource.username
password-property: spring.datasource.username
```

-  `enabled` 将此值设置为 `true` 可启用数据库后端配置使用

-  `role` 设置数据库角色定义的角色名称

-  `backend` 设置要使用的数据库装载的路径

-  `username-property` 设置存储数据库用户名的属性名称

-  `password-property` 设置存储数据库密码的属性名称

另见：[Vault Documentation: Database Secrets backend](https://www.vaultproject.io/docs/secrets/databases/index.html)

## 105.2 Apache Cassandra

> 在Vault 0.7.1中已弃用 `cassandra` 后端，建议使用 `database` 后端并将其挂载为 `cassandra` .

Spring Cloud Vault可以获取Apache Cassandra的凭据.可以通过设置 `spring.cloud.vault.cassandra.enabled=true` （默认 `false` ）并为角色名称 `spring.cloud.vault.cassandra.role=…` 启用集成.

用户名和密码存储在 `spring.data.cassandra.username` 和 `spring.data.cassandra.password` 中，因此使用Spring Boot将获取生成的凭据，无需进一步配置.您可以通过设置 `spring.cloud.vault.cassandra.username-property` 和 `spring.cloud.vault.cassandra.password-property` 来配置属性名称.

```java
spring.cloud.vault:
cassandra:
enabled: true
role: readonly
backend: cassandra
username-property: spring.data.cassandra.username
password-property: spring.data.cassandra.username
```

-  `enabled` 将此值设置为 `true` 可启用Cassandra后端配置使用

-  `role` 设置Cassandra角色定义的角色名称

-  `backend` 设置要使用的Cassandra挂载的路径

-  `username-property` 设置存储Cassandra用户名的属性名称

-  `password-property` 设置存储Cassandra密码的属性名称

另见：[Vault Documentation: Setting up Apache Cassandra with Vault](https://www.vaultproject.io/docs/secrets/cassandra/index.html)

## 105.3 MongoDB

> 在Vault 0.7.1中已弃用 `mongodb` 后端，建议使用 `database` 后端并将其挂载为 `mongodb` .

Spring Cloud Vault可以获取MongoDB的凭据.可以通过设置 `spring.cloud.vault.mongodb.enabled=true` （默认 `false` ）并为角色名称提供 `spring.cloud.vault.mongodb.role=…` 来启用集成.

用户名和密码存储在 `spring.data.mongodb.username` 和 `spring.data.mongodb.password` 中，因此使用Spring Boot将获取生成的凭据而无需进一步配置.您可以通过设置 `spring.cloud.vault.mongodb.username-property` 和 `spring.cloud.vault.mongodb.password-property` 来配置属性名称.

```java
spring.cloud.vault:
mongodb:
enabled: true
role: readonly
backend: mongodb
username-property: spring.data.mongodb.username
password-property: spring.data.mongodb.password
```

-  `enabled` 将此值设置为 `true` 可启用MongodB后端配置使用

-  `role` 设置MongoDB角色定义的角色名称

-  `backend` 设置要使用的MongoDB挂载的路径

-  `username-property` 设置存储MongoDB用户名的属性名称

-  `password-property` 设置存储MongoDB密码的属性名称

另见：[Vault Documentation: Setting up MongoDB with Vault](https://www.vaultproject.io/docs/secrets/mongodb/index.html)

## 105.4 MySQL

> 在Vault 0.7.1中已弃用 `mysql` 后端，建议使用 `database` 后端并将其挂载为 `mysql` .  `spring.cloud.vault.mysql` 的配置将在以后的版本中删除.

Spring Cloud Vault可以获取MySQL的凭据.可以通过设置 `spring.cloud.vault.mysql.enabled=true` （默认 `false` ）并为角色名称提供 `spring.cloud.vault.mysql.role=…` 来启用集成.

用户名和密码存储在 `spring.datasource.username` 和 `spring.datasource.password` 中，因此使用Spring Boot将获取生成的凭据，无需进一步配置.您可以通过设置 `spring.cloud.vault.mysql.username-property` 和 `spring.cloud.vault.mysql.password-property` 来配置属性名称.

```java
spring.cloud.vault:
mysql:
enabled: true
role: readonly
backend: mysql
username-property: spring.datasource.username
password-property: spring.datasource.username
```

-  `enabled` 将此值设置为 `true` 可启用MySQL后端配置使用

-  `role` 设置MySQL角色定义的角色名称

-  `backend` 设置要使用的MySQL挂载的路径

-  `username-property` 设置存储MySQL用户名的属性名称

-  `password-property` 设置存储MySQL密码的属性名称

另见：[Vault Documentation: Setting up MySQL with Vault](https://www.vaultproject.io/docs/secrets/mysql/index.html)

## 105.5 PostgreSQL

> 在Vault 0.7.1中已弃用 `postgresql` 后端，建议使用 `database` 后端并将其挂载为 `postgresql` .  `spring.cloud.vault.postgresql` 的配置将在以后的版本中删除.

Spring Cloud Vault可以获取PostgreSQL的凭据.可以通过设置 `spring.cloud.vault.postgresql.enabled=true` （默认 `false` ）并为角色名称提供 `spring.cloud.vault.postgresql.role=…` 来启用集成.

用户名和密码存储在 `spring.datasource.username` 和 `spring.datasource.password` 中，因此使用Spring Boot将获取生成的凭据，无需进一步配置.您可以通过设置 `spring.cloud.vault.postgresql.username-property` 和 `spring.cloud.vault.postgresql.password-property` 来配置属性名称.

```java
spring.cloud.vault:
postgresql:
enabled: true
role: readonly
backend: postgresql
username-property: spring.datasource.username
password-property: spring.datasource.username
```

-  `enabled` 将此值设置为 `true` 可启用PostgreSQL后端配置使用

-  `role` 设置PostgreSQL角色定义的角色名称

-  `backend` 设置要使用的PostgreSQL装载的路径

-  `username-property` 设置存储PostgreSQL用户名的属性名称

-  `password-property` 设置存储PostgreSQL密码的属性名称

另见：[Vault Documentation: Setting up PostgreSQL with Vault](https://www.vaultproject.io/docs/secrets/postgresql/index.html)

