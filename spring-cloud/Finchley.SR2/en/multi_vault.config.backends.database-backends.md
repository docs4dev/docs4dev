## 105. Database backends

Vault supports several database secret backends to generate database credentials dynamically based on configured roles. This means services that need to access a database no longer need to configure credentials: they can request them from Vault, and use Vault’s leasing mechanism to more easily roll keys.

Spring Cloud Vault integrates with these backends:

- [Section 105.1, “Database”](multi_vault.config.backends.database-backends.html#vault.config.backends.database)

- [Section 105.2, “Apache Cassandra”](multi_vault.config.backends.database-backends.html#vault.config.backends.cassandra)

- [Section 105.3, “MongoDB”](multi_vault.config.backends.database-backends.html#vault.config.backends.mongodb)

- [Section 105.4, “MySQL”](multi_vault.config.backends.database-backends.html#vault.config.backends.mysql)

- [Section 105.5, “PostgreSQL”](multi_vault.config.backends.database-backends.html#vault.config.backends.postgresql)

Using a database secret backend requires to enable the backend in the configuration and the  `spring-cloud-vault-config-databases`  dependency.

Vault ships since 0.7.1 with a dedicated  `database`  secret backend that allows database integration via plugins. You can use that specific backend by using the generic database backend. Make sure to specify the appropriate backend path, e.g.  `spring.cloud.vault.mysql.role.backend=database` .

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

> Enabling multiple JDBC-compliant databases will generate credentials and store them by default in the same property keys hence property names for JDBC secrets need to be configured separately.

## 105.1 Database

Spring Cloud Vault can obtain credentials for any database listed at [https://www.vaultproject.io/api/secret/databases/index.html](https://www.vaultproject.io/api/secret/databases/index.html). The integration can be enabled by setting  `spring.cloud.vault.database.enabled=true`  (default  `false` ) and providing the role name with  `spring.cloud.vault.database.role=…` .

While the database backend is a generic one,  `spring.cloud.vault.database`  specifically targets JDBC databases. Username and password are stored in  `spring.datasource.username`  and  `spring.datasource.password`  so using Spring Boot will pick up the generated credentials for your  `DataSource`  without further configuration. You can configure the property names by setting  `spring.cloud.vault.database.username-property`  and  `spring.cloud.vault.database.password-property` .

```java
spring.cloud.vault:
database:
enabled: true
role: readonly
backend: database
username-property: spring.datasource.username
password-property: spring.datasource.username
```

-  `enabled`  setting this value to  `true`  enables the Database backend config usage

-  `role`  sets the role name of the Database role definition

-  `backend`  sets the path of the Database mount to use

-  `username-property`  sets the property name in which the Database username is stored

-  `password-property`  sets the property name in which the Database password is stored

See also: [Vault Documentation: Database Secrets backend](https://www.vaultproject.io/docs/secrets/databases/index.html)

## 105.2 Apache Cassandra

> The  `cassandra`  backend has been deprecated in Vault 0.7.1 and it is recommended to use the  `database`  backend and mount it as  `cassandra` .

Spring Cloud Vault can obtain credentials for Apache Cassandra. The integration can be enabled by setting  `spring.cloud.vault.cassandra.enabled=true`  (default  `false` ) and providing the role name with  `spring.cloud.vault.cassandra.role=…` .

Username and password are stored in  `spring.data.cassandra.username`  and  `spring.data.cassandra.password`  so using Spring Boot will pick up the generated credentials without further configuration. You can configure the property names by setting  `spring.cloud.vault.cassandra.username-property`  and  `spring.cloud.vault.cassandra.password-property` .

```java
spring.cloud.vault:
cassandra:
enabled: true
role: readonly
backend: cassandra
username-property: spring.data.cassandra.username
password-property: spring.data.cassandra.username
```

-  `enabled`  setting this value to  `true`  enables the Cassandra backend config usage

-  `role`  sets the role name of the Cassandra role definition

-  `backend`  sets the path of the Cassandra mount to use

-  `username-property`  sets the property name in which the Cassandra username is stored

-  `password-property`  sets the property name in which the Cassandra password is stored

See also: [Vault Documentation: Setting up Apache Cassandra with Vault](https://www.vaultproject.io/docs/secrets/cassandra/index.html)

## 105.3 MongoDB

> The  `mongodb`  backend has been deprecated in Vault 0.7.1 and it is recommended to use the  `database`  backend and mount it as  `mongodb` .

Spring Cloud Vault can obtain credentials for MongoDB. The integration can be enabled by setting  `spring.cloud.vault.mongodb.enabled=true`  (default  `false` ) and providing the role name with  `spring.cloud.vault.mongodb.role=…` .

Username and password are stored in  `spring.data.mongodb.username`  and  `spring.data.mongodb.password`  so using Spring Boot will pick up the generated credentials without further configuration. You can configure the property names by setting  `spring.cloud.vault.mongodb.username-property`  and  `spring.cloud.vault.mongodb.password-property` .

```java
spring.cloud.vault:
mongodb:
enabled: true
role: readonly
backend: mongodb
username-property: spring.data.mongodb.username
password-property: spring.data.mongodb.password
```

-  `enabled`  setting this value to  `true`  enables the MongodB backend config usage

-  `role`  sets the role name of the MongoDB role definition

-  `backend`  sets the path of the MongoDB mount to use

-  `username-property`  sets the property name in which the MongoDB username is stored

-  `password-property`  sets the property name in which the MongoDB password is stored

See also: [Vault Documentation: Setting up MongoDB with Vault](https://www.vaultproject.io/docs/secrets/mongodb/index.html)

## 105.4 MySQL

> The  `mysql`  backend has been deprecated in Vault 0.7.1 and it is recommended to use the  `database`  backend and mount it as  `mysql` . Configuration for  `spring.cloud.vault.mysql`  will be removed in a future version.

Spring Cloud Vault can obtain credentials for MySQL. The integration can be enabled by setting  `spring.cloud.vault.mysql.enabled=true`  (default  `false` ) and providing the role name with  `spring.cloud.vault.mysql.role=…` .

Username and password are stored in  `spring.datasource.username`  and  `spring.datasource.password`  so using Spring Boot will pick up the generated credentials without further configuration. You can configure the property names by setting  `spring.cloud.vault.mysql.username-property`  and  `spring.cloud.vault.mysql.password-property` .

```java
spring.cloud.vault:
mysql:
enabled: true
role: readonly
backend: mysql
username-property: spring.datasource.username
password-property: spring.datasource.username
```

-  `enabled`  setting this value to  `true`  enables the MySQL backend config usage

-  `role`  sets the role name of the MySQL role definition

-  `backend`  sets the path of the MySQL mount to use

-  `username-property`  sets the property name in which the MySQL username is stored

-  `password-property`  sets the property name in which the MySQL password is stored

See also: [Vault Documentation: Setting up MySQL with Vault](https://www.vaultproject.io/docs/secrets/mysql/index.html)

## 105.5 PostgreSQL

> The  `postgresql`  backend has been deprecated in Vault 0.7.1 and it is recommended to use the  `database`  backend and mount it as  `postgresql` . Configuration for  `spring.cloud.vault.postgresql`  will be removed in a future version.

Spring Cloud Vault can obtain credentials for PostgreSQL. The integration can be enabled by setting  `spring.cloud.vault.postgresql.enabled=true`  (default  `false` ) and providing the role name with  `spring.cloud.vault.postgresql.role=…` .

Username and password are stored in  `spring.datasource.username`  and  `spring.datasource.password`  so using Spring Boot will pick up the generated credentials without further configuration. You can configure the property names by setting  `spring.cloud.vault.postgresql.username-property`  and  `spring.cloud.vault.postgresql.password-property` .

```java
spring.cloud.vault:
postgresql:
enabled: true
role: readonly
backend: postgresql
username-property: spring.datasource.username
password-property: spring.datasource.username
```

-  `enabled`  setting this value to  `true`  enables the PostgreSQL backend config usage

-  `role`  sets the role name of the PostgreSQL role definition

-  `backend`  sets the path of the PostgreSQL mount to use

-  `username-property`  sets the property name in which the PostgreSQL username is stored

-  `password-property`  sets the property name in which the PostgreSQL password is stored

See also: [Vault Documentation: Setting up PostgreSQL with Vault](https://www.vaultproject.io/docs/secrets/postgresql/index.html)

