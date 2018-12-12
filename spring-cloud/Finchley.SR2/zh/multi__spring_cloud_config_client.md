## 10. Spring Cloud配置客户端

Spring Boot应用程序可以立即利用Spring Config Server（或应用程序开发人员提供的其他外部属性源）.它还提供了与 `Environment` 更改事件相关的一些其他有用功能.

## 10.1 Config First Bootstrap

在类路径上具有Spring Cloud Config Client的任何应用程序的默认行为如下所示：当配置客户端启动时，它将绑定到Config Server（通过 `spring.cloud.config.uri`  bootstrap配置属性）并使用远程属性源初始化Spring  `Environment` .

此行为的最终结果是，所有想要使用配置服务器的客户端应用程序都需要 `bootstrap.yml` （或环境变量），其服务器地址在 `spring.cloud.config.uri` 中设置（默认为"http://localhost:8888"）.

## 10.2 Discovery First Bootstrap

如果您使用“DiscoveryClient”实施，例如Spring Cloud Netflix和Eureka Service Discovery或Spring Cloud Consul，您可以将Config Server注册到Discovery Service.但是，在默认的“Config First”模式下，客户端无法利用注册.

如果您更喜欢使用 `DiscoveryClient` 来查找配置服务器，可以通过设置 `spring.cloud.config.discovery.enabled=true` （默认为 `false` ）来实现.这样做的最终结果是客户端应用程序都需要具有适当发现配置的 `bootstrap.yml` （或环境变量）.例如，使用Spring Cloud Netflix，您需要定义Eureka服务器地址（例如，在 `eureka.client.serviceUrl.defaultZone` 中）.使用此选项的价格是启动时额外的网络往返，以查找服务注册.好处是，只要发现服务是固定点，配置服务器就可以更改其坐标.默认服务ID是 `configserver` ，但您可以通过设置 `spring.cloud.config.discovery.serviceId` （在服务器上，以通常的方式为服务，例如通过设置 `spring.application.name` ）在客户端上更改它.

发现客户端实现都支持某种元数据映射（例如，我们为Eureka提供了 `eureka.instance.metadataMap` ）.可能需要在其服务注册元数据中配置Config Server的一些其他属性，以便客户端可以正确连接.如果使用HTTP Basic保护配置服务器，则可以将凭据配置为 `username` 和 `password` .此外，如果Config Server具有上下文路径，则可以设置 `configPath` .例如，以下YAML文件用于作为Eureka客户端的Config Server：

**bootstrap.yml.** 

```java
eureka:
instance:
...
metadataMap:
user: osufhalskjrtl
password: lviuhlszvaorhvlo5847
configPath: /config
```

## 10.3配置客户端快速失败

在某些情况下，如果服务无法连接到Config Server，您可能希望无法启动服务.如果这是所需的行为，请设置引导程序配置属性 `spring.cloud.config.fail-fast=true` 以使客户端停止并显示异常.

## 10.4配置客户端重试

如果您希望配置服务器在应用程序启动时偶尔可能不可用，您可以在失败后继续尝试.首先，您需要设置 `spring.cloud.config.fail-fast=true` .然后你需要将 `spring-retry` 和 `spring-boot-starter-aop` 添加到类路径中.默认行为是重试六次，初始退避间隔为1000毫秒，指数乘数为1.1，以便后续退避.您可以通过设置 `spring.cloud.config.retry.*` 配置属性来配置这些属性（和其他属性）.

> 要完全控制重试行为，请添加 `RetryOperationsInterceptor` 类型 `RetryOperationsInterceptor` ，ID为 `configServerRetryInterceptor` . Spring Retry有一个 `RetryInterceptorBuilder` ，支持创建一个.

## 10.5查找远程配置资源

Config Service提供来自 `/{name}/{profile}/{label}` 的属性源，其中客户端应用程序中的默认绑定如下：

- "name" =  `${spring.application.name}` 

- "profile" =  `${spring.profiles.active}` （实际 `Environment.getActiveProfiles()` ）

- "label" = "master"

> 当设置属性 `${spring.application.name}` 时，不要在应用程序名称前加上保留字 `application-` ，以防止解决正确属性源的问题.

您可以通过设置 `spring.cloud.config.*` （其中 `*` 是 `name` ， `profile` 或 `label` ）来覆盖所有这些.  `label` 对于回滚到以前版本的配置非常有用.使用默认的Config Server实现，它可以是git标签，分支名称或提交ID. Label也可以以逗号分隔列表的形式提供.在这种情况下，列表中的项目将逐个尝试，直到成功为止.在处理功能分支时，此行为非常有用.例如，您可能希望将配置标签与分支对齐，但使其成为可选（在这种情况下，请使用 `spring.cloud.config.label=myfeature,develop` ）.

## 10.6为Config Server指定多个URL

为了确保在部署了多个Config Server实例并期望一个或多个实例不时不可用时的高可用性，您可以指定多个URL（作为 `spring.cloud.config.uri` 属性下的逗号分隔列表）或拥有所有实例在Eureka等服务注册表中注册（如果使用Discovery-First Bootstrap模式）.请注意，只有在Config Server未运行时（即应用程序已退出时）或发生连接超时时，才能确保高可用性.例如，如果Config Server返回500（内部服务器错误）响应或Config Client从Config Server收到401（由于凭据错误或其他原因），则Config Client不会尝试从其他URL获取属性.这种错误表示用户问题而不是可用性问题.

如果在Config Server上使用HTTP基本安全性，则仅当您在 `spring.cloud.config.uri` 属性下指定的每个URL中嵌入凭据时，才能支持每个Config Server身份验证凭据.如果使用任何其他类型的安全机制，则无法（当前）支持每个Config Server身份验证和授权.

## 10.7配置读取超时

如果要配置读取超时，可以使用属性 `spring.cloud.config.request-read-timeout` 来完成.

## 10.8安全

如果在服务器上使用HTTP Basic安全性，则客户端需要知道密码（如果不是默认密码，则需要知道用户名）.您可以通过配置服务器URI或单独的用户名和密码属性指定用户名和密码，如以下示例所示：

**bootstrap.yml.** 

```java
spring:
cloud:
config:
uri: https://user:[emailprotected]
```

以下示例显示了传递相同信息的另一种方法：

**bootstrap.yml.** 

```java
spring:
cloud:
config:
uri: https://myconfig.mycompany.com
username: user
password: secret
```

`spring.cloud.config.password` 和 `spring.cloud.config.username` 值会覆盖URI中提供的任何内容.

如果您在Cloud Foundry上部署应用程序，提供密码的最佳方式是通过服务凭据（例如在URI中，因为它不需要在配置文件中）.以下示例适用于本地以及Cloud Foundry上名为 `configserver` 的用户提供的服务：

**bootstrap.yml.** 

```java
spring:
cloud:
config:
uri: ${vcap.services.configserver.credentials.uri:http://user:[emailprotected]:8888}
```

如果您使用其他形式的安全性，则可能需要[provide a RestTemplate](multi__spring_cloud_config_client.html#custom-rest-template)到 `ConfigServicePropertySourceLocator` （例如，通过在引导程序上下文中抓取它并注入它）.

### 10.8.1Health指标

Config Client提供Spring Boot Health Indicator，尝试从Config Server加载配置.可以通过设置 `health.config.enabled=false` 来禁用运行状况指示器.出于性能原因，还会缓存响应.生存的默认缓存时间为5分钟.要更改该值，请设置 `health.config.time-to-live` 属性（以毫秒为单位）.

### 10.8.2提供自定义RestTemplate

在某些情况下，您可能需要自定义从客户端向配置服务器发出的请求.通常，这样做涉及传递特殊的 `Authorization` 标头来验证对服务器的请求.要提供自定义 `RestTemplate` ：

使用PropertySourceLocator的实现创建一个新的配置bean，如以下示例所示：

**CustomConfigServiceBootstrapConfiguration.java.** 

```java
@Configuration
public class CustomConfigServiceBootstrapConfiguration {
@Bean
public ConfigServicePropertySourceLocator configServicePropertySourceLocator() {
ConfigClientProperties clientProperties = configClientProperties();
ConfigServicePropertySourceLocator configServicePropertySourceLocator =  new ConfigServicePropertySourceLocator(clientProperties);
configServicePropertySourceLocator.setRestTemplate(customRestTemplate(clientProperties));
return configServicePropertySourceLocator;
}
}
```

在resources / META-INF中，创建一个名为spring.factories的文件并指定自定义配置，如以下示例所示：

**spring.factories.** 

```java
org.springframework.cloud.bootstrap.BootstrapConfiguration = com.my.config.client.CustomConfigServiceBootstrapConfiguration
```

### 10.8.3保险柜

使用Vault作为配置服务器的后端时，客户端需要为服务器提供令牌以从Vault检索值.通过在 `bootstrap.yml` 中设置 `spring.cloud.config.token` ，可以在客户端内提供此令牌，如下所示以下示例：

**bootstrap.yml.** 

```java
spring:
cloud:
config:
token: YourVaultToken
```

## 10.9 Vault中的嵌套密钥

Vault支持将密钥嵌套在Vault中存储的值中，如以下示例所示：

`echo -n '{"appA": {"secret": "appAsecret"}, "bar": "baz"}' | vault write secret/myapp -` 

此命令将JSON对象写入Vault.要在Spring中访问这些值，您将使用传统的点（ `.` ）注释，如以下示例所示

```java
@Value("${appA.secret}")
String name = "World";
```

上面的代码会将 `name` 变量的值设置为 `appAsecret` .

