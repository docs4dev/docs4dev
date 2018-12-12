## 104.秘密后端

## 104.1通用后端

Spring Cloud Vault在基本级别支持通用秘密后端.通用秘密后端允许将任意值存储为键值存储.单个上下文可以存储一个或多个键值元组.可以按层次结构组织上下文. Spring Cloud Vault允许将应用程序名称和默认上下文名称（ `application` ）与活动配置文件结合使用.

```java
/secret/{application}/{profile}
/secret/{application}
/secret/{default-context}/{profile}
/secret/{default-context}
```

应用程序名称由属性确定：

-  `spring.cloud.vault.generic.application-name` 

-  `spring.cloud.vault.application-name` 

-  `spring.application.name` 

通过将其路径添加到应用程序名称（以逗号分隔），可以从通用后端中的其他上下文获取秘密.例如，给定应用程序名称 `usefulapp,mysql1,projectx/aws` ，将使用以下每个文件夹：

-  `/secret/usefulapp` 

-  `/secret/mysql1` 

-  `/secret/projectx/aws` 

Spring Cloud Vault会将所有活动的配置文件添加到可能的上下文路径列表中.没有活动的配置文件将跳过使用配置文件名称访问上下文.

属性被存储就像它们被存储一样（即没有额外的前缀）.

```java
spring.cloud.vault:
generic:
enabled: true
backend: secret
profile-separator: '/'
default-context: application
application-name: my-app
```

-  `enabled` 将此值设置为 `false` 会禁用秘密后端配置使用情况

-  `backend` 设置要使用的安装程序的路径

-  `default-context` 设置所有应用程序使用的上下文名称

-  `application-name` 会覆盖在通用后端中使用的应用程序名称

-  `profile-separator` 使用配置文件将配置文件名称与属性源中的上下文分开

> 键值秘密后端可以在版本化（v2）和非版本化（v1）模式下运行.根据操作模式，访问机密需要不同的API.确保为非版本化键值后端启用 `generic` 秘密后端使用，并为版本化键值后端启用 `kv` 秘密后端使用.

另见：[Vault Documentation: Using the KV Secrets Engine - Version 1 (generic secret backend)](https://www.vaultproject.io/docs/secrets/kv/kv-v1.html)

## 104.2版本化的键值后端

Spring Cloud Vault支持版本化的键值秘密后端.键值后端允许将任意值存储为键值存储.单个上下文可以存储一个或多个键值元组.可以按层次结构组织上下文. Spring Cloud Vault允许将应用程序名称和默认上下文名称（ `application` ）与活动配置文件结合使用.

```java
/secret/{application}/{profile}
/secret/{application}
/secret/{default-context}/{profile}
/secret/{default-context}
```

应用程序名称由属性确定：

-  `spring.cloud.vault.kv.application-name` 

-  `spring.cloud.vault.application-name` 

-  `spring.application.name` 

通过将其路径添加到应用程序名称（以逗号分隔），可以从键值后端内的其他上下文获取秘密.例如，给定应用程序名称 `usefulapp,mysql1,projectx/aws` ，这些文件夹中的每一个都将是用过的：

-  `/secret/usefulapp` 

-  `/secret/mysql1` 

-  `/secret/projectx/aws` 

Spring Cloud Vault会将所有活动的配置文件添加到可能的上下文路径列表中.没有活动的配置文件将跳过使用配置文件名称访问上下文.

属性被存储就像它们被存储一样（即没有额外的前缀）.

> Spring Cloud Vault在装载路径和实际上下文路径之间添加 `data/` 上下文.

```java
spring.cloud.vault:
kv:
enabled: true
backend: secret
profile-separator: '/'
default-context: application
application-name: my-app
```

-  `enabled` 将此值设置为 `false` 会禁用秘密后端配置使用情况

-  `backend` 设置要使用的机密挂载的路径

-  `default-context` 设置所有应用程序使用的上下文名称

-  `application-name` 会覆盖在通用后端中使用的应用程序名称

-  `profile-separator` 使用配置文件将配置文件名称与属性源中的上下文分开

> 键值秘密后端可以在版本化（v2）和非版本化（v1）模式下运行.根据操作模式，访问机密需要不同的API.确保为非版本化键值后端启用 `generic` 秘密后端使用，并为版本化键值后端启用 `kv` 秘密后端使用.

另见：[Vault Documentation: Using the KV Secrets Engine - Version 2 (versioned key-value backend)](https://www.vaultproject.io/docs/secrets/kv/kv-v2.html)

## 104.3Consul

Spring Cloud Vault可以获得HashiCorp Consul的凭据. Consul集成需要 `spring-cloud-vault-config-consul` 依赖项.

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

可以通过设置 `spring.cloud.vault.consul.enabled=true` （默认 `false` ）并为角色名称提供 `spring.cloud.vault.consul.role=…` 来启用集成.

获取的令牌存储在 `spring.cloud.consul.token` 中，因此使用Spring Cloud Consul可以获取生成的凭据而无需进一步配置.您可以通过设置 `spring.cloud.vault.consul.token-property` 来配置属性名称.

```java
spring.cloud.vault:
consul:
enabled: true
role: readonly
backend: consul
token-property: spring.cloud.consul.token
```

-  `enabled` 将此值设置为 `true` 可启用Consul后端配置使用

-  `role` 设置Consul角色定义的角色名称

-  `backend` 设置要使用的Consul挂载的路径

-  `token-property` 设置存储Consul ACL令牌的属性名称

另见：[Vault Documentation: Setting up Consul with Vault](https://www.vaultproject.io/docs/secrets/consul/index.html)

## 104.4 RabbitMQ

Spring Cloud Vault可以获取RabbitMQ的凭据.

RabbitMQ集成需要 `spring-cloud-vault-config-rabbitmq` 依赖项.

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

可以通过设置 `spring.cloud.vault.rabbitmq.enabled=true` （默认 `false` ）并为角色名称提供 `spring.cloud.vault.rabbitmq.role=…` 来启用集成.

用户名和密码存储在 `spring.rabbitmq.username` 和 `spring.rabbitmq.password` 中，因此使用Spring Boot将获取生成的凭据，无需进一步配置.您可以通过设置 `spring.cloud.vault.rabbitmq.username-property` 和 `spring.cloud.vault.rabbitmq.password-property` 来配置属性名称.

```java
spring.cloud.vault:
rabbitmq:
enabled: true
role: readonly
backend: rabbitmq
username-property: spring.rabbitmq.username
password-property: spring.rabbitmq.password
```

-  `enabled` 将此值设置为 `true` 可启用RabbitMQ后端配置使用

-  `role` 设置RabbitMQ角色定义的角色名称

-  `backend` 设置要使用的RabbitMQ挂载的路径

-  `username-property` 设置存储RabbitMQ用户名的属性名称

-  `password-property` 设置存储RabbitMQ密码的属性名称

另见：[Vault Documentation: Setting up RabbitMQ with Vault](https://www.vaultproject.io/docs/secrets/rabbitmq/index.html)

## 104.5 AWS

Spring Cloud Vault可以获取AWS的凭据.

AWS集成需要 `spring-cloud-vault-config-aws` 依赖项.

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

可以通过设置 `spring.cloud.vault.aws=true` （默认 `false` ）并为角色名称 `spring.cloud.vault.aws.role=…` 启用集成.

访问密钥和密钥存储在 `cloud.aws.credentials.accessKey` 和 `cloud.aws.credentials.secretKey` 中，因此使用Spring Cloud AWS将获取生成的凭据，无需进一步配置.您可以通过设置 `spring.cloud.vault.aws.access-key-property` 和 `spring.cloud.vault.aws.secret-key-property` 来配置属性名称.

```java
spring.cloud.vault:
aws:
enabled: true
role: readonly
backend: aws
access-key-property: cloud.aws.credentials.accessKey
secret-key-property: cloud.aws.credentials.secretKey
```

-  `enabled` 将此值设置为 `true` 可启用AWS后端配置使用

-  `role` 设置AWS角色定义的角色名称

-  `backend` 设置要使用的AWS挂载的路径

-  `access-key-property` 设置存储AWS访问密钥的属性名称

-  `secret-key-property` 设置存储AWS密钥的属性名称

另见：[Vault Documentation: Setting up AWS with Vault](https://www.vaultproject.io/docs/secrets/aws/index.html)

