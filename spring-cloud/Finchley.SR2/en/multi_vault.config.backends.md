## 104. Secret Backends

## 104.1 Generic Backend

Spring Cloud Vault supports at the basic level the generic secret backend. The generic secret backend allows storage of arbitrary values as key-value store. A single context can store one or many key-value tuples. Contexts can be organized hierarchically. Spring Cloud Vault allows using the Application name and a default context name ( `application` ) in combination with active profiles.

```java
/secret/{application}/{profile}
/secret/{application}
/secret/{default-context}/{profile}
/secret/{default-context}
```

The application name is determined by the properties:

-  `spring.cloud.vault.generic.application-name` 

-  `spring.cloud.vault.application-name` 

-  `spring.application.name` 

Secrets can be obtained from other contexts within the generic backend by adding their paths to the application name, separated by commas. For example, given the application name  `usefulapp,mysql1,projectx/aws` , each of these folders will be used:

-  `/secret/usefulapp` 

-  `/secret/mysql1` 

-  `/secret/projectx/aws` 

Spring Cloud Vault adds all active profiles to the list of possible context paths. No active profiles will skip accessing contexts with a profile name.

Properties are exposed like they are stored (i.e. without additional prefixes).

```java
spring.cloud.vault:
generic:
enabled: true
backend: secret
profile-separator: '/'
default-context: application
application-name: my-app
```

-  `enabled`  setting this value to  `false`  disables the secret backend config usage

-  `backend`  sets the path of the secret mount to use

-  `default-context`  sets the context name used by all applications

-  `application-name`  overrides the application name for use in the generic backend

-  `profile-separator`  separates the profile name from the context in property sources with profiles

> The key-value secret backend can be operated in versioned (v2) and non-versioned (v1) modes. Depending on the mode of operation, a different API is required to access secrets. Make sure to enable  `generic`  secret backend usage for non-versioned key-value backends and  `kv`  secret backend usage for versioned key-value backends.

See also: [Vault Documentation: Using the KV Secrets Engine - Version 1 (generic secret backend)](https://www.vaultproject.io/docs/secrets/kv/kv-v1.html)

## 104.2 Versioned Key-Value Backend

Spring Cloud Vault supports the versioned Key-Value secret backend. The key-value backend allows storage of arbitrary values as key-value store. A single context can store one or many key-value tuples. Contexts can be organized hierarchically. Spring Cloud Vault allows using the Application name and a default context name ( `application` ) in combination with active profiles.

```java
/secret/{application}/{profile}
/secret/{application}
/secret/{default-context}/{profile}
/secret/{default-context}
```

The application name is determined by the properties:

-  `spring.cloud.vault.kv.application-name` 

-  `spring.cloud.vault.application-name` 

-  `spring.application.name` 

Secrets can be obtained from other contexts within the key-value backend by adding their paths to the application name, separated by commas. For example, given the application name  `usefulapp,mysql1,projectx/aws` , each of these folders will be used:

-  `/secret/usefulapp` 

-  `/secret/mysql1` 

-  `/secret/projectx/aws` 

Spring Cloud Vault adds all active profiles to the list of possible context paths. No active profiles will skip accessing contexts with a profile name.

Properties are exposed like they are stored (i.e. without additional prefixes).

> Spring Cloud Vault adds the  `data/`  context between the mount path and the actual context path.

```java
spring.cloud.vault:
kv:
enabled: true
backend: secret
profile-separator: '/'
default-context: application
application-name: my-app
```

-  `enabled`  setting this value to  `false`  disables the secret backend config usage

-  `backend`  sets the path of the secret mount to use

-  `default-context`  sets the context name used by all applications

-  `application-name`  overrides the application name for use in the generic backend

-  `profile-separator`  separates the profile name from the context in property sources with profiles

> The key-value secret backend can be operated in versioned (v2) and non-versioned (v1) modes. Depending on the mode of operation, a different API is required to access secrets. Make sure to enable  `generic`  secret backend usage for non-versioned key-value backends and  `kv`  secret backend usage for versioned key-value backends.

See also: [Vault Documentation: Using the KV Secrets Engine - Version 2 (versioned key-value backend)](https://www.vaultproject.io/docs/secrets/kv/kv-v2.html)

## 104.3 Consul

Spring Cloud Vault can obtain credentials for HashiCorp Consul. The Consul integration requires the  `spring-cloud-vault-config-consul`  dependency.

**Example 104.1. pom.xml** 

```xml
<dependencies>
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-vault-config-consul</artifactId>
<version>Finchley.SR2</version>
</dependency>
</dependencies>
```

The integration can be enabled by setting  `spring.cloud.vault.consul.enabled=true`  (default  `false` ) and providing the role name with  `spring.cloud.vault.consul.role=…` .

The obtained token is stored in  `spring.cloud.consul.token`  so using Spring Cloud Consul can pick up the generated credentials without further configuration. You can configure the property name by setting  `spring.cloud.vault.consul.token-property` .

```java
spring.cloud.vault:
consul:
enabled: true
role: readonly
backend: consul
token-property: spring.cloud.consul.token
```

-  `enabled`  setting this value to  `true`  enables the Consul backend config usage

-  `role`  sets the role name of the Consul role definition

-  `backend`  sets the path of the Consul mount to use

-  `token-property`  sets the property name in which the Consul ACL token is stored

See also: [Vault Documentation: Setting up Consul with Vault](https://www.vaultproject.io/docs/secrets/consul/index.html)

## 104.4 RabbitMQ

Spring Cloud Vault can obtain credentials for RabbitMQ.

The RabbitMQ integration requires the  `spring-cloud-vault-config-rabbitmq`  dependency.

**Example 104.2. pom.xml** 

```xml
<dependencies>
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-vault-config-rabbitmq</artifactId>
<version>Finchley.SR2</version>
</dependency>
</dependencies>
```

The integration can be enabled by setting  `spring.cloud.vault.rabbitmq.enabled=true`  (default  `false` ) and providing the role name with  `spring.cloud.vault.rabbitmq.role=…` .

Username and password are stored in  `spring.rabbitmq.username`  and  `spring.rabbitmq.password`  so using Spring Boot will pick up the generated credentials without further configuration. You can configure the property names by setting  `spring.cloud.vault.rabbitmq.username-property`  and  `spring.cloud.vault.rabbitmq.password-property` .

```java
spring.cloud.vault:
rabbitmq:
enabled: true
role: readonly
backend: rabbitmq
username-property: spring.rabbitmq.username
password-property: spring.rabbitmq.password
```

-  `enabled`  setting this value to  `true`  enables the RabbitMQ backend config usage

-  `role`  sets the role name of the RabbitMQ role definition

-  `backend`  sets the path of the RabbitMQ mount to use

-  `username-property`  sets the property name in which the RabbitMQ username is stored

-  `password-property`  sets the property name in which the RabbitMQ password is stored

See also: [Vault Documentation: Setting up RabbitMQ with Vault](https://www.vaultproject.io/docs/secrets/rabbitmq/index.html)

## 104.5 AWS

Spring Cloud Vault can obtain credentials for AWS.

The AWS integration requires the  `spring-cloud-vault-config-aws`  dependency.

**Example 104.3. pom.xml** 

```xml
<dependencies>
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-vault-config-aws</artifactId>
<version>Finchley.SR2</version>
</dependency>
</dependencies>
```

The integration can be enabled by setting  `spring.cloud.vault.aws=true`  (default  `false` ) and providing the role name with  `spring.cloud.vault.aws.role=…` .

The access key and secret key are stored in  `cloud.aws.credentials.accessKey`  and  `cloud.aws.credentials.secretKey`  so using Spring Cloud AWS will pick up the generated credentials without further configuration. You can configure the property names by setting  `spring.cloud.vault.aws.access-key-property`  and  `spring.cloud.vault.aws.secret-key-property` .

```java
spring.cloud.vault:
aws:
enabled: true
role: readonly
backend: aws
access-key-property: cloud.aws.credentials.accessKey
secret-key-property: cloud.aws.credentials.secretKey
```

-  `enabled`  setting this value to  `true`  enables the AWS backend config usage

-  `role`  sets the role name of the AWS role definition

-  `backend`  sets the path of the AWS mount to use

-  `access-key-property`  sets the property name in which the AWS access key is stored

-  `secret-key-property`  sets the property name in which the AWS secret key is stored

See also: [Vault Documentation: Setting up AWS with Vault](https://www.vaultproject.io/docs/secrets/aws/index.html)

