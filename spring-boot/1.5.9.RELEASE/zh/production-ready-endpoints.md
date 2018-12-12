## 53.endpoints

通过Actuatorendpoints，您可以监控应用程序并与之交互. Spring Boot包含许多内置endpoints，允许您添加自己的endpoints.例如， `health` endpoints提供基本的应用程序运行状况信息.

每个endpoints都可以[enabled or disabled](production-ready-endpoints.html#production-ready-endpoints-enabling-endpoints).它控制是否创建endpoints并且其bean存在于应用程序上下文中.要远程访问，endpoints也必须是[exposed via JMX or HTTP](production-ready-endpoints.html#production-ready-endpoints-exposing-endpoints).大多数应用程序选择HTTP，其中endpoints的ID以及 `/actuator` 的前缀映射到URL.例如，默认情况下， `health` endpoints映射到 `/actuator/health` .

可以使用以下与技术无关的endpoints：

| ID |说明|默认情况下启用|
| ---- | ---- | ---- |
|  `auditevents`  |公开当前应用程序的审核事件信息. |是的|
|  `beans`  |显示应用程序中所有Spring bean的完整列表. |是的|
|  `caches`  |公开可用的缓存. |是的|
|  `conditions`  |显示在配置和自动配置类上评估的条件以及它们匹配或不匹配的原因. |是的|
|  `configprops`  |显示所有 `@ConfigurationProperties` 的整理列表. |是的|
|  `env`  |公开Spring的 `ConfigurableEnvironment` 中的属性. |是的|
|  `flyway`  |显示已应用的所有Flyway数据库迁移. |是的|
|  `health`  |显示应用程序运行状况信息. |是的|
|  `httptrace`  |显示HTTP跟踪信息（默认情况下，最后100个HTTP请求 - 响应交换）. |是的|
|  `info`  |显示任意应用程序信息. |是的|
|  `integrationgraph`  |显示Spring Integration图. |是的|
|  `loggers`  |显示和修改应用程序中Logger的配置. |是的|
|  `liquibase`  |显示已应用的所有Liquibase数据库迁移. |是的|
|  `metrics`  |显示当前应用程序的“指标”信息. |是的|
|  `mappings`  |显示所有 `@RequestMapping` 路径的整理列表. |是的|
|  `scheduledtasks`  |显示应用程序中的计划任务. |是的|
|  `sessions`  |允许从Spring Session支持的会话存储中检索和删除用户会话.使用Spring Session对响应式Web应用程序的支持时不可用. |是的|
|  `shutdown`  |允许应用程序正常关闭. |不|
|  `threaddump`  |执行线程转储. |是的|

如果您的应用程序是Web应用程序（Spring MVC，Spring WebFlux或Jersey），则可以使用以下附加endpoints：

| ID |说明|默认情况下启用|
| ---- | ---- | ---- |
|  `heapdump`  |返回GZip压缩的 `hprof` 堆转储文件. |是的|
|  `jolokia`  |通过HTTP公开JMX bean（当Jolokia在类路径上时，不适用于WebFlux）. |是的|
|  `logfile`  |返回日志文件的内容（如果已设置 `logging.file` 或 `logging.path` 属性）.支持使用HTTP  `Range` 标头来检索部分日志文件的内容. |是的|
|  `prometheus`  |以Prometheus服务器可以抓取的格式公开指标. |是的|

要了解有关Actuatorendpoints及其请求和响应格式的更多信息，请参阅单独的API文档（[HTML](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/actuator-api//html)或[PDF](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/actuator-api//pdf/spring-boot-actuator-web-api.pdf)）.

## 53.1启用endpoints

默认情况下，启用除 `shutdown` 之外的所有endpoints.要配置endpoints的启用，请使用其 `management.endpoint.<id>.enabled` 属性.以下示例启用 `shutdown` endpoints：

```java
management.endpoint.shutdown.enabled=true
```

如果您希望endpoints启用是选择加入而不是选择退出，请将 `management.endpoints.enabled-by-default` 属性设置为 `false` 并使用单个endpoints `enabled` 属性重新加入.以下示例启用 `info` endpoints并禁用所有其他endpoints：

```java
management.endpoints.enabled-by-default=false
management.endpoint.info.enabled=true
```

> Disabledendpoints完全从应用程序上下文中删除.如果您只想更改endpoints所暴露的技术，请改用[include and exclude properties](production-ready-endpoints.html#production-ready-endpoints-exposing-endpoints).

## 53.2公开endpoints

由于endpoints可能包含敏感信息，因此应仔细考虑何时公开它们.下表显示了默认曝光内置endpoints：

| ID | JMX |网络|
| ---- | ---- | ---- |
|  `auditevents`  |是|否|
|  `beans`  |是|否|
|  `caches`  |是|否|
|  `conditions`  |是|否|
|  `configprops`  |是|否|
|  `env`  |是|否|
|  `flyway`  |是|否|
|  `health`  |是|是|
|  `heapdump`  | N / A |否|
|  `httptrace`  |是|否|
|  `info`  |是|是|
|  `integrationgraph`  |是|否|
|  `jolokia`  | N / A |否|
|  `logfile`  | N / A |否|
|  `loggers`  |是|否|
|  `liquibase`  |是|否|
|  `metrics`  |是|否|
|  `mappings`  |是|否|
|  `prometheus`  | N / A |否|
|  `scheduledtasks`  |是|否|
|  `sessions`  |是|否|
|  `shutdown`  |是|否|
|  `threaddump`  |是|否|

要更改公开的endpoints，请使用以下特定于技术的 `include` 和 `exclude` 属性：

|房产|设为首页|
| ---- | ---- |
|  `management.endpoints.jmx.exposure.exclude`  | |
|  `management.endpoints.jmx.exposure.include`  |  `*`  |
|  `management.endpoints.web.exposure.exclude`  | |
|  `management.endpoints.web.exposure.include`  |  `info, health`  |

`include` 属性列出了公开的endpoints的ID.  `exclude` 属性列出了不应公开的endpoints的ID.  `exclude` 属性优先于 `include` 属性.  `include` 和 `exclude` 属性都可以配置endpointsID列表.

例如，要停止通过JMX公开所有endpoints并仅显示 `health` 和 `info` endpoints，请使用以下属性：

```java
management.endpoints.jmx.exposure.include=health,info
```

`*` 可用于选择所有endpoints.例如，要通过HTTP公开除 `env` 和 `beans` endpoints之外的所有内容，请使用以下属性：

```java
management.endpoints.web.exposure.include=*
management.endpoints.web.exposure.exclude=env,beans
```

>  `*` 在YAML中具有特殊含义，因此如果要包括（或排除）所有endpoints，请务必添加引号，如以下示例所示：

> 如果您的申请被公开曝光，我们强烈建议您也[secure your endpoints](production-ready-endpoints.html#production-ready-endpoints-security).

> 如果要实现自己的策略以显示endpoints，可以注册 `EndpointFilter`  bean.

## 53.3保护HTTPendpoints

您应该像使用任何其他敏感URL一样注意保护HTTPendpoints.如果存在Spring Security，则默认使用Spring Security的内容协商策略来保护endpoints.例如，如果您希望为HTTPendpoints配置自定义安全性，只允许具有特定角色的用户访问它们，Spring Boot提供了一些方便的 `RequestMatcher` 对象，可以与Spring Security结合使用.

典型的Spring Security配置可能类似于以下示例：

```java
@Configuration
public class ActuatorSecurity extends WebSecurityConfigurerAdapter {

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http.requestMatcher(EndpointRequest.toAnyEndpoint()).authorizeRequests()
				.anyRequest().hasRole("ENDPOINT_ADMIN")
				.and()
			.httpBasic();
	}

}
```

上面的示例使用 `EndpointRequest.toAnyEndpoint()` 将请求与任何endpoints进行匹配，然后确保所有endpoints都具有 `ENDPOINT_ADMIN` 角色.  `EndpointRequest` 上还提供了其他几种匹配方法.有关详细信息，请参阅API文档（[HTML](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/actuator-api//html)或[PDF](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/actuator-api//pdf/spring-boot-actuator-web-api.pdf)）.

如果在防火墙后部署应用程序，您可能希望无需身份验证即可访问所有Actuatorendpoints.您可以通过更改 `management.endpoints.web.exposure.include` 属性来执行此操作，如下所示：

**application.properties.** 

```java
management.endpoints.web.exposure.include=*
```

此外，如果存在Spring Security，则需要添加自定义安全配置，以允许对endpoints进行未经身份验证的访问，如以下示例所示：

```java
@Configuration
public class ActuatorSecurity extends WebSecurityConfigurerAdapter {

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http.requestMatcher(EndpointRequest.toAnyEndpoint()).authorizeRequests()
			.anyRequest().permitAll();
	}

}
```

## 53.4配置endpoints

endpoints自动缓存对不带任何参数的读取操作的响应.要配置endpoints缓存响应的时间量，请使用其 `cache.time-to-live` 属性.以下示例将 `beans` endpoints缓存的生存时间设置为10秒：

**application.properties.** 

```java
management.endpoint.beans.cache.time-to-live=10s
```

> 前缀 `management.endpoint.<name>` 用于唯一标识正在配置的endpoints.

> 当进行经过身份验证的HTTP请求时， `Principal` 被视为endpoints的输入，因此不会缓存响应.

## 53.5用于ActuatorWebendpoints的超媒体

添加了“发现页面”，其中包含指向所有endpoints的链接.默认情况下，“发现页面”在 `/actuator` 上可用.

配置自定义管理上下文路径后，“发现页面”会自动从 `/actuator` 移动到管理上下文的根目录.例如，如果管理上下文路径为 `/management` ，则发现页面可从 `/management` 获得.当管理上下文路径设置为 `/` 时，将禁用发现页面以防止与其他映射冲突的可能性.

## 53.6 CORS支持

[Cross-origin resource sharing](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)（CORS）是一个[W3C specification](https://www.w3.org/TR/cors/)，它允许您以灵活的方式指定授权的跨域请求类型.如果您使用Spring MVC或Spring WebFlux，则可以配置Actuator的Webendpoints以支持此类方案.

默认情况下禁用CORS支持，仅在设置了 `management.endpoints.web.cors.allowed-origins` 属性后才启用CORS支持.以下配置允许 `example.com` 域中的 `GET` 和 `POST` 调用：

```java
management.endpoints.web.cors.allowed-origins=http://example.com
management.endpoints.web.cors.allowed-methods=GET,POST
```

> 查看[CorsEndpointProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator-autoconfigure/src/main/java/org/springframework/boot/actuate/autoconfigure/endpoint/web/CorsEndpointProperties.java)以获取完整的选项列表.

## 53.7实施自定义endpoints

如果添加带有 `@Endpoint` 的 `@Bean` 注释，则使用 `@ReadOperation` ， `@WriteOperation` 或 `@DeleteOperation` 注释的任何方法都将通过JMX自动公开，并且在Web应用程序中也会通过HTTP自动公开.可以使用Jersey，Spring MVC或Spring WebFlux通过HTTP公开endpoints.

您还可以使用 `@JmxEndpoint` 或 `@WebEndpoint` 编写特定于技术的endpoints.这些endpoints仅限于各自的技术.例如， `@WebEndpoint` 仅被公开通过HTTP而不是通过JMX.

您可以使用 `@EndpointWebExtension` 和 `@EndpointJmxExtension` 编写特定于技术的扩展.通过这些注释，您可以提供特定于技术的操作来扩充现有endpoints.

最后，如果您需要访问特定于Web框架的功能，则可以实现Servlet或Spring  `@Controller` 和 `@RestController` endpoints，但代价是它们无法通过JMX或使用其他Web框架.

### 53.7.1接收输入

endpoints上的操作通过其参数接收输入.通过Web公开时，这些参数的值取自URL的查询参数和JSON请求体.通过JMX公开时，参数将映射到MBean操作的参数.默认情况下需要参数.可以通过使用 `@org.springframework.lang.Nullable` 注释它们来使它们成为可选项.

JSON请求正文中的每个根属性都可以映射到endpoints的参数.请考虑以下JSON请求正文：

```java
{
	"name": "test",
	"counter": 42
}
```

这可用于调用带有 `String name` 和 `int counter` 参数的写操作.

> B因为endpoints是技术不可知的，所以只能在方法签名中指定简单类型.特别是不支持使用定义 `name` 和 `counter` 属性的自定义类型声明单个参数.

> 允许输入映射到操作方法的参数，实现endpoints的Java代码应该用 `-parameters` 编译，实现endpoints的Kotlin代码应该用 `-java-parameters` 编译.如果您使用的是Spring Boot的Gradle插件，或者您使用的是Maven和 `spring-boot-starter-parent` ，则会自动执行此操作.

#### Input类型转换

如有必要，传递给endpoints操作方法的参数将自动转换为所需类型.在调用操作方法之前，使用 `ApplicationConversionService` 实例将通过JMX或HTTP请求接收的输入转换为所需类型.

### 53.7.2自定义Webendpoints

使用Jersey，Spring MVC或Spring WebFlux通过HTTP自动公开 `@Endpoint` ， `@WebEndpoint` 或 `@EndpointWebExtension` 上的操作.

#### Webendpoints请求谓词

为Web暴露的endpoints上的每个操作自动生成请求谓词.

#### Path

谓词的路径由endpoints的ID和Web暴露的endpoints的基本路径确定.默认基本路径为 `/actuator` .例如，ID为 `sessions` 的endpoints将使用 `/actuator/sessions` 作为谓词中的路径.

可以通过使用 `@Selector` 注释操作方法的一个或多个参数来进一步定制路径.这样的参数作为路径变量添加到路径谓词中.调用endpoints操作时，将变量的值传递给操作方法.

#### HTTP方法

谓词的HTTP方法由操作类型决定，如下表所示：

|操作| HTTP方法|
| ---- | ---- |
|  `@ReadOperation`  |  `GET`  |
|  `@WriteOperation`  |  `POST`  |
|  `@DeleteOperation`  |  `DELETE`  |

#### Consumes

对于使用请求主体的 `@WriteOperation` （HTTP  `POST` ），谓词的consumemes子句是 `application/vnd.spring-boot.actuator.v2+json, application/json` .对于所有其他操作，consumemes子句为空.

#### Produces

谓词的produce子句可以由 `@DeleteOperation` ， `@ReadOperation` 和 `@WriteOperation` 注释的 `produces` 属性确定.该属性是可选的.如果未使用，则自动确定produce子句.

如果操作方法返回 `void` 或 `Void` ，则produce子句为空.如果操作方法返回 `org.springframework.core.io.Resource` ，则produce子句为 `application/octet-stream` .对于所有其他操作，produce子句是 `application/vnd.spring-boot.actuator.v2+json, application/json` .

#### Webendpoints响应状态

endpoints操作的默认响应状态取决于操作类型（读取，写入或删除）以及操作返回的内容（如果有）.

`@ReadOperation` 返回一个值，响应状态为200（OK）.如果它未返回值，则响应状态将为404（未找到）.

如果 `@WriteOperation` 或 `@DeleteOperation` 返回一个值，则响应状态将为200（OK）.如果它没有返回值，则响应状态将为204（无内容）.

如果在没有必需参数的情况下调用操作，或者使用无法转换为所需类型的参数，则不会调用操作方法，并且响应状态将为400（错误请求）.

#### Webendpoints范围请求

HTTP范围请求可用于请求HTTP资源的一部分.使用Spring MVC或Spring Web Flux时，返回 `org.springframework.core.io.Resource` 的操作会自动支持范围请求.

使用Jersey时不支持
> Range请求.

#### Webendpoints安全

Webendpoints或特定于Web的endpoints扩展上的操作可以接收当前 `java.security.Principal` 或 `org.springframework.boot.actuate.endpoint.SecurityContext` 作为方法参数.前者通常与 `@Nullable` 结合使用，为经过身份验证和未经身份验证的用户提供不同的行为.后者通常用于使用 `isUserInRole(String)` 方法执行授权检查.

### 53.7.3 Servletendpoints

`Servlet` 可以作为endpoints公开通过实现一个用 `@ServletEndpoint` 注释的类来实现 `Supplier<EndpointServlet>` . Servletendpoints提供与Servlet容器更深层次的集成，但代价是可移植性.它们旨在用于将现有 `Servlet` 作为endpoints公开.对于新endpoints，应尽可能优先选择 `@Endpoint` 和 `@WebEndpoint` 注释.

### 53.7.4控制器endpoints

`@ControllerEndpoint` 和 `@RestControllerEndpoint` 可用于实现仅由Spring MVC或Spring WebFlux公开的endpoints.使用Spring MVC和Spring WebFlux的标准注释（例如 `@RequestMapping` 和 `@GetMapping` ）映射方法，并将endpoints的ID用作路径的前缀.控制器endpoints提供与Spring的Web框架更深层次的集成，但代价是可移植性.应尽可能优先选择 `@Endpoint` 和 `@WebEndpoint` 注释.

## 53.8Health信息

您可以使用运行状况信息来检查正在运行的应用程序的状态.监视软件经常使用它来在生产环境系统出现故障时向某人发出警报.  `health` endpoints公开的信息取决于 `management.endpoint.health.show-details` 属性，该属性可以使用以下值之一进行配置：

|名称|说明|
| ---- | ---- |
|  `never`  |详细信息永远不会显示. |
|  `when-authorized`  |详细信息仅向授权用户显示.可以使用 `management.endpoint.health.roles` 配置授权角色. |
|  `always`  |向所有用户显示详细信息. |

默认值为 `never` .当用户处于一个或多个endpoints的角色时，将被视为已获得授权.如果endpoints没有配置角色（默认值），则认为所有经过身份验证的用户都已获得授权.可以使用 `management.endpoint.health.roles` 属性配置角色.

> 如果您已保护应用程序并希望使用 `always` ，则您的安全配置必须允许对经过身份验证和未经身份验证的用户访问运行状况终结点.

Health信息是从[HealthIndicatorRegistry](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthIndicatorRegistry.java)的内容中收集的（默认情况下， `ApplicationContext` 中定义的所有[HealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthIndicator.java)实例.Spring Boot包含一些自动配置的 `HealthIndicators` ，您也可以编写自己的.默认情况下，最终系统状态由 `HealthAggregator` 根据有序的状态列表对每个 `HealthIndicator` 的状态进行排序.排序列表中的第一个状态用作整体Health状态.如果没有 `HealthIndicator` 返回 `HealthAggregator` 已知的状态，则使用 `UNKNOWN` 状态.

>   `HealthIndicatorRegistry` 可用于在运行时注册和取消注册运行状况指示器.

### 53.8.1自动配置的HealthIndicators

适当时，Spring Boot会自动配置以下 `HealthIndicators` ：

|名称|说明|
| ---- | ---- |
| [CassandraHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/cassandra/CassandraHealthIndicator.java) |检查Cassandra数据库是否已启动. |
| [CouchbaseHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/couchbase/CouchbaseHealthIndicator.java) |检查Couchbase群集是否已启动. |
| [DiskSpaceHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/system/DiskSpaceHealthIndicator.java) |检查磁盘空间不足. |
| [DataSourceHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/jdbc/DataSourceHealthIndicator.java) |检查是否可以获得与 `DataSource` 的连接. |
| [ElasticsearchHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/elasticsearch/ElasticsearchHealthIndicator.java) |检查Elasticsearch集群是否已启动. |
| [InfluxDbHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/influx/InfluxDbHealthIndicator.java) |检查InfluxDB服务器是否已启动. |
| [JmsHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/jms/JmsHealthIndicator.java) |检查JMS代理是否已启动. |
| [MailHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/mail/MailHealthIndicator.java) |检查邮件服务器是否已启动. |
| [MongoHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/mongo/MongoHealthIndicator.java) |检查Mongo数据库是否已启动. |
| [Neo4jHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/neo4j/Neo4jHealthIndicator.java) |检查Neo4j服务器是否已启动. |
| [RabbitHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/amqp/RabbitHealthIndicator.java) |检查Rabbit服务器是否已启动. |
| [RedisHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/redis/RedisHealthIndicator.java) |检查Redis服务器是否已启动. |
| [SolrHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/solr/SolrHealthIndicator.java) |检查Solr服务器是否已启动. |

> 您可以通过设置 `management.health.defaults.enabled` 属性来禁用它们.

### 53.8.2编写自定义HealthIndicators

要提供自定义运行状况信息，可以注册实现[HealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthIndicator.java)接口的Spring bean.您需要提供 `health()` 方法的实现并返回 `Health` 响应.  `Health` 响应应包含状态，并且可以选择包含要显示的其他详细信息.以下代码显示了一个示例 `HealthIndicator` 实现：

```java
import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

@Component
public class MyHealthIndicator implements HealthIndicator {

	@Override
	public Health health() {
		int errorCode = check(); // perform some specific health check
		if (errorCode != 0) {
			return Health.down().withDetail("Error Code", errorCode).build();
		}
		return Health.up().build();
	}

}
```

> 给定 `HealthIndicator` 的标识符是没有 `HealthIndicator` 后缀的bean的名称（如果存在）.在前面的示例中，Health信息在名为 `my` 的条目中可用.

除了Spring Boot的预定义[Status](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/Status.java)类型之外， `Health` 还可以返回表示新系统状态的自定义 `Status` .在这种情况下，还需要提供[HealthAggregator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthAggregator.java)接口的自定义实现，或者必须使用 `management.health.status.order` 配置属性配置默认实现.

例如，假设在 `HealthIndicator` 实现之一中使用了带有代码 `FATAL` 的新 `Status` .要配置严重性顺序，请将以下属性添加到应用程序属性：

```java
management.health.status.order=FATAL, DOWN, OUT_OF_SERVICE, UNKNOWN, UP
```

响应中的HTTP状态代码反映了整体运行状况（例如， `UP` 映射到200，而 `OUT_OF_SERVICE` 和 `DOWN` 映射到503）.如果通过HTTP访问运行状况endpoints，则可能还需要注册自定义状态映射.例如，以下属性将 `FATAL` 映射到503（服务不可用）：

```java
management.health.status.http-mapping.FATAL=503
```

> 如果需要更多控制，可以定义自己的 `HealthStatusHttpMapper`  bean.

下表显示了内置状态的默认状态映射：

|状态|贴图|
| ---- | ---- |
| DOWN | SERVICE_UNAVAILABLE（503）|
| OUT_OF_SERVICE | SERVICE_UNAVAILABLE（503）|
| UP |默认情况下没有映射，因此http状态为200 |
| UNKNOWN |默认情况下没有映射，因此http状态为200 |

### 53.8.3反应Health指标

对于反应式应用程序，例如使用Spring WebFlux的应用程序， `ReactiveHealthIndicator` 提供了一个非阻塞的Contract来获取应用程序运行状况.与传统的 `HealthIndicator` 类似，Health信息是从[ReactiveHealthIndicatorRegistry](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/ReactiveHealthIndicatorRegistry.java)的内容中收集的（默认情况下， `ApplicationContext` 中定义的所有[HealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthIndicator.java)和[ReactiveHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/ReactiveHealthIndicator.java)个实例.在弹性调度程序上执行不检查反应API的常规 `HealthIndicator` .

> 在响应式应用程序中， `ReactiveHealthIndicatorRegistry` 可用于在运行时注册和取消注册运行状况指示器.

要从反应式API提供自定义运行状况信息，可以注册实现[ReactiveHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/ReactiveHealthIndicator.java)接口的Spring bean.以下代码显示了一个示例 `ReactiveHealthIndicator` 实现：

```java
@Component
public class MyReactiveHealthIndicator implements ReactiveHealthIndicator {

	@Override
	public Mono<Health> health() {
		return doHealthCheck() //perform some specific health check that returns a Mono<Health>
			.onErrorResume(ex -> Mono.just(new Health.Builder().down(ex).build())));
	}

}
```

> 自动处理错误，考虑从 `AbstractReactiveHealthIndicator` 扩展.

### 53.8.4自动配置的ReactiveHealthIndicators

适当时，Spring Boot会自动配置以下 `ReactiveHealthIndicators` ：

|名称|说明|
| ---- | ---- |
| [CassandraReactiveHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/cassandra/CassandraReactiveHealthIndicator.java) |检查Cassandra数据库是否已启动. |
| [CouchbaseReactiveHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/couchbase/CouchbaseReactiveHealthIndicator.java) |检查Couchbase群集是否已启动. |
| [MongoReactiveHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/mongo/MongoReactiveHealthIndicator.java) |检查Mongo数据库是否已启动. |
| [RedisReactiveHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/redis/RedisReactiveHealthIndicator.java) |检查Redis服务器是否已启动. |

> 如果必要，反应指标取代常规指标.此外，任何未明确处理的 `HealthIndicator` 都会自动换行.

## 53.9申请信息

应用程序信息公开从 `ApplicationContext` 中定义的所有[InfoContributor](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/info/InfoContributor.java) bean收集的各种信息. Spring Boot包含许多自动配置的 `InfoContributor`  bean，你可以编写自己的bean.

### 53.9.1自动配置的InfoContributors

适当时，Spring Boot会自动配置以下 `InfoContributor`  bean：

|名称|说明|
| ---- | ---- |
| [EnvironmentInfoContributor](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/info/EnvironmentInfoContributor.java) |在 `info` 键下显示 `Environment` 中的任何键. |
| [GitInfoContributor](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/info/GitInfoContributor.java) |如果 `git.properties` 文件可用，则公开git信息. |
| [BuildInfoContributor](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/info/BuildInfoContributor.java) |如果 `META-INF/build-info.properties` 文件可用，则公开构建信息. |

> 可以通过设置 `management.info.defaults.enabled` 属性来禁用它们.

### 53.9.2自定义应用程序信息

您可以通过设置 `info.*`  Spring属性来自定义 `info` endpoints公开的数据.  `info` 键下的所有 `Environment` 属性都会自动公开.例如，您可以将以下设置添加到 `application.properties` 文件中：

```java
info.app.encoding=UTF-8
info.app.java.source=1.8
info.app.java.target=1.8
```

> Rather比硬编码这些值，你也可以[expand info properties at build time](howto-properties-and-configuration.html#howto-automatic-expansion).

### 53.9.3 Git提交信息

`info` endpoints的另一个有用功能是它能够在构建项目时发布有关 `git` 源代码存储库状态的信息.如果 `GitProperties`  bean可用，则会公开 `git.branch` ， `git.commit.id` 和 `git.commit.time` 属性.

如果类路径的根目录中有 `git.properties` 文件，则自动配置
> A  `GitProperties`  bean.有关详细信息，请参阅“[Generate git information](howto-build.html#howto-git-info)”.

如果要显示完整的git信息（即 `git.properties` 的完整内容），请使用 `management.info.git.mode` 属性，如下所示：

```java
management.info.git.mode=full
```

### 53.9.4构建信息

如果 `BuildProperties`  bean可用， `info` endpoints也可以发布有关构建的信息.如果类路径中有 `META-INF/build-info.properties` 文件，则会发生这种情况.

>  Maven和Gradle插件都可以生成该文件.有关详细信息，请参阅“[Generate build information](howto-build.html#howto-build-info)”.

### 53.9.5编写自定义InfoContributors

要提供自定义应用程序信息，可以注册实现[InfoContributor](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/info/InfoContributor.java)接口的Spring bean.

以下示例使用单个值提供 `example` 条目：

```java
import java.util.Collections;

import org.springframework.boot.actuate.info.Info;
import org.springframework.boot.actuate.info.InfoContributor;
import org.springframework.stereotype.Component;

@Component
public class ExampleInfoContributor implements InfoContributor {

	@Override
	public void contribute(Info.Builder builder) {
		builder.withDetail("example",
				Collections.singletonMap("key", "value"));
	}

}
```

如果到达 `info` endpoints，您应该看到包含以下附加条目的响应：

```java
{
	"example": {
		"key" : "value"
	}
}
```

