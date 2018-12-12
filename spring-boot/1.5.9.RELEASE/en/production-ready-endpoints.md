## 53. Endpoints

Actuator endpoints let you monitor and interact with your application. Spring Boot includes a number of built-in endpoints and lets you add your own. For example, the  `health`  endpoint provides basic application health information.

Each individual endpoint can be [enabled or disabled](production-ready-endpoints.html#production-ready-endpoints-enabling-endpoints). This controls whether or not the endpoint is created and its bean exists in the application context. To be remotely accessible an endpoint also has to be [exposed via JMX or HTTP](production-ready-endpoints.html#production-ready-endpoints-exposing-endpoints). Most applications choose HTTP, where the ID of the endpoint along with a prefix of  `/actuator`  is mapped to a URL. For example, by default, the  `health`  endpoint is mapped to  `/actuator/health` .

The following technology-agnostic endpoints are available:

|ID|Description|Enabled by default|
|----|----|----|
| `auditevents`  |Exposes audit events information for the current application. |Yes |
| `beans`  |Displays a complete list of all the Spring beans in your application. |Yes |
| `caches`  |Exposes available caches. |Yes |
| `conditions`  |Shows the conditions that were evaluated on configuration and auto-configuration classes and the reasons why they did or did not match. |Yes |
| `configprops`  |Displays a collated list of all  `@ConfigurationProperties` . |Yes |
| `env`  |Exposes properties from Spring’s  `ConfigurableEnvironment` . |Yes |
| `flyway`  |Shows any Flyway database migrations that have been applied. |Yes |
| `health`  |Shows application health information. |Yes |
| `httptrace`  |Displays HTTP trace information (by default, the last 100 HTTP request-response exchanges). |Yes |
| `info`  |Displays arbitrary application info. |Yes |
| `integrationgraph`  |Shows the Spring Integration graph. |Yes |
| `loggers`  |Shows and modifies the configuration of loggers in the application. |Yes |
| `liquibase`  |Shows any Liquibase database migrations that have been applied. |Yes |
| `metrics`  |Shows ‘metrics’ information for the current application. |Yes |
| `mappings`  |Displays a collated list of all  `@RequestMapping`  paths. |Yes |
| `scheduledtasks`  |Displays the scheduled tasks in your application. |Yes |
| `sessions`  |Allows retrieval and deletion of user sessions from a Spring Session-backed session store. Not available when using Spring Session’s support for reactive web applications. |Yes |
| `shutdown`  |Lets the application be gracefully shutdown. |No |
| `threaddump`  |Performs a thread dump. |Yes |

If your application is a web application (Spring MVC, Spring WebFlux, or Jersey), you can use the following additional endpoints:

|ID|Description|Enabled by default|
|----|----|----|
| `heapdump`  |Returns a GZip compressed  `hprof`  heap dump file. |Yes |
| `jolokia`  |Exposes JMX beans over HTTP (when Jolokia is on the classpath, not available for WebFlux). |Yes |
| `logfile`  |Returns the contents of the logfile (if  `logging.file`  or  `logging.path`  properties have been set). Supports the use of the HTTP  `Range`  header to retrieve part of the log file’s content. |Yes |
| `prometheus`  |Exposes metrics in a format that can be scraped by a Prometheus server. |Yes |

To learn more about the Actuator’s endpoints and their request and response formats, please refer to the separate API documentation ([HTML](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/actuator-api//html) or [PDF](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/actuator-api//pdf/spring-boot-actuator-web-api.pdf)).

## 53.1 Enabling Endpoints

By default, all endpoints except for  `shutdown`  are enabled. To configure the enablement of an endpoint, use its  `management.endpoint.<id>.enabled`  property. The following example enables the  `shutdown`  endpoint:

```java
management.endpoint.shutdown.enabled=true
```

If you prefer endpoint enablement to be opt-in rather than opt-out, set the  `management.endpoints.enabled-by-default`  property to  `false`  and use individual endpoint  `enabled`  properties to opt back in. The following example enables the  `info`  endpoint and disables all other endpoints:

```java
management.endpoints.enabled-by-default=false
management.endpoint.info.enabled=true
```

> Disabled endpoints are removed entirely from the application context. If you want to change only the technologies over which an endpoint is exposed, use the [include and exclude properties](production-ready-endpoints.html#production-ready-endpoints-exposing-endpoints) instead.

## 53.2 Exposing Endpoints

Since Endpoints may contain sensitive information, careful consideration should be given about when to expose them. The following table shows the default exposure for the built-in endpoints:

|ID|JMX|Web|
|----|----|----|
| `auditevents`  |Yes |No |
| `beans`  |Yes |No |
| `caches`  |Yes |No |
| `conditions`  |Yes |No |
| `configprops`  |Yes |No |
| `env`  |Yes |No |
| `flyway`  |Yes |No |
| `health`  |Yes |Yes |
| `heapdump`  |N/A |No |
| `httptrace`  |Yes |No |
| `info`  |Yes |Yes |
| `integrationgraph`  |Yes |No |
| `jolokia`  |N/A |No |
| `logfile`  |N/A |No |
| `loggers`  |Yes |No |
| `liquibase`  |Yes |No |
| `metrics`  |Yes |No |
| `mappings`  |Yes |No |
| `prometheus`  |N/A |No |
| `scheduledtasks`  |Yes |No |
| `sessions`  |Yes |No |
| `shutdown`  |Yes |No |
| `threaddump`  |Yes |No |

To change which endpoints are exposed, use the following technology-specific  `include`  and  `exclude`  properties:

|Property|Default|
|----|----|
| `management.endpoints.jmx.exposure.exclude`  | |
| `management.endpoints.jmx.exposure.include`  | `*`  |
| `management.endpoints.web.exposure.exclude`  | |
| `management.endpoints.web.exposure.include`  | `info, health`  |

The  `include`  property lists the IDs of the endpoints that are exposed. The  `exclude`  property lists the IDs of the endpoints that should not be exposed. The  `exclude`  property takes precedence over the  `include`  property. Both  `include`  and  `exclude`  properties can be configured with a list of endpoint IDs.

For example, to stop exposing all endpoints over JMX and only expose the  `health`  and  `info`  endpoints, use the following property:

```java
management.endpoints.jmx.exposure.include=health,info
```

`*`  can be used to select all endpoints. For example, to expose everything over HTTP except the  `env`  and  `beans`  endpoints, use the following properties:

```java
management.endpoints.web.exposure.include=*
management.endpoints.web.exposure.exclude=env,beans
```

>  `*`  has a special meaning in YAML, so be sure to add quotes if you want to include (or exclude) all endpoints, as shown in the following example:

> If your application is exposed publicly, we strongly recommend that you also [secure your endpoints](production-ready-endpoints.html#production-ready-endpoints-security).

> If you want to implement your own strategy for when endpoints are exposed, you can register an  `EndpointFilter`  bean.

## 53.3 Securing HTTP Endpoints

You should take care to secure HTTP endpoints in the same way that you would any other sensitive URL. If Spring Security is present, endpoints are secured by default using Spring Security’s content-negotiation strategy. If you wish to configure custom security for HTTP endpoints, for example, only allow users with a certain role to access them, Spring Boot provides some convenient  `RequestMatcher`  objects that can be used in combination with Spring Security.

A typical Spring Security configuration might look something like the following example:

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

The preceding example uses  `EndpointRequest.toAnyEndpoint()`  to match a request to any endpoint and then ensures that all have the  `ENDPOINT_ADMIN`  role. Several other matcher methods are also available on  `EndpointRequest` . See the API documentation ([HTML](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/actuator-api//html) or [PDF](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/actuator-api//pdf/spring-boot-actuator-web-api.pdf)) for details.

If you deploy applications behind a firewall, you may prefer that all your actuator endpoints can be accessed without requiring authentication. You can do so by changing the  `management.endpoints.web.exposure.include`  property, as follows:

**application.properties.**  

```java
management.endpoints.web.exposure.include=*
```

Additionally, if Spring Security is present, you would need to add custom security configuration that allows unauthenticated access to the endpoints as shown in the following example:

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

## 53.4 Configuring Endpoints

Endpoints automatically cache responses to read operations that do not take any parameters. To configure the amount of time for which an endpoint will cache a response, use its  `cache.time-to-live`  property. The following example sets the time-to-live of the  `beans`  endpoint’s cache to 10 seconds:

**application.properties.**  

```java
management.endpoint.beans.cache.time-to-live=10s
```

> The prefix  `management.endpoint.<name>`  is used to uniquely identify the endpoint that is being configured.

> When making an authenticated HTTP request, the  `Principal`  is considered as input to the endpoint and, therefore, the response will not be cached.

## 53.5 Hypermedia for Actuator Web Endpoints

A “discovery page” is added with links to all the endpoints. The “discovery page” is available on  `/actuator`  by default.

When a custom management context path is configured, the “discovery page” automatically moves from  `/actuator`  to the root of the management context. For example, if the management context path is  `/management` , then the discovery page is available from  `/management` . When the management context path is set to  `/` , the discovery page is disabled to prevent the possibility of a clash with other mappings.

## 53.6 CORS Support

[Cross-origin resource sharing](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) (CORS) is a [W3C specification](https://www.w3.org/TR/cors/) that lets you specify in a flexible way what kind of cross-domain requests are authorized. If you use Spring MVC or Spring WebFlux, Actuator’s web endpoints can be configured to support such scenarios.

CORS support is disabled by default and is only enabled once the  `management.endpoints.web.cors.allowed-origins`  property has been set. The following configuration permits  `GET`  and  `POST`  calls from the  `example.com`  domain:

```java
management.endpoints.web.cors.allowed-origins=http://example.com
management.endpoints.web.cors.allowed-methods=GET,POST
```

> See [CorsEndpointProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator-autoconfigure/src/main/java/org/springframework/boot/actuate/autoconfigure/endpoint/web/CorsEndpointProperties.java) for a complete list of options.

## 53.7 Implementing Custom Endpoints

If you add a  `@Bean`  annotated with  `@Endpoint` , any methods annotated with  `@ReadOperation` ,  `@WriteOperation` , or  `@DeleteOperation`  are automatically exposed over JMX and, in a web application, over HTTP as well. Endpoints can be exposed over HTTP using Jersey, Spring MVC, or Spring WebFlux.

You can also write technology-specific endpoints by using  `@JmxEndpoint`  or  `@WebEndpoint` . These endpoints are restricted to their respective technologies. For example,  `@WebEndpoint`  is exposed only over HTTP and not over JMX.

You can write technology-specific extensions by using  `@EndpointWebExtension`  and  `@EndpointJmxExtension` . These annotations let you provide technology-specific operations to augment an existing endpoint.

Finally, if you need access to web-framework-specific functionality, you can implement Servlet or Spring  `@Controller`  and  `@RestController`  endpoints at the cost of them not being available over JMX or when using a different web framework.

### 53.7.1 Receiving Input

Operations on an endpoint receive input via their parameters. When exposed via the web, the values for these parameters are taken from the URL’s query parameters and from the JSON request body. When exposed via JMX, the parameters are mapped to the parameters of the MBean’s operations. Parameters are required by default. They can be made optional by annotating them with  `@org.springframework.lang.Nullable` .

Each root property in the JSON request body can be mapped to a parameter of the endpoint. Consider the following JSON request body:

```java
{
	"name": "test",
	"counter": 42
}
```

This can be used to invoke a write operation that takes  `String name`  and  `int counter`  parameters.

> Because endpoints are technology agnostic, only simple types can be specified in the method signature. In particular declaring a single parameter with a custom type defining a  `name`  and  `counter`  properties is not supported.

> To allow the input to be mapped to the operation method’s parameters, Java code implementing an endpoint should be compiled with  `-parameters` , and Kotlin code implementing an endpoint should be compiled with  `-java-parameters` . This will happen automatically if you are using Spring Boot’s Gradle plugin or if you are using Maven and  `spring-boot-starter-parent` .

#### Input type conversion

The parameters passed to endpoint operation methods are, if necessary, automatically converted to the required type. Before calling an operation method, the input received via JMX or an HTTP request is converted to the required types using an instance of  `ApplicationConversionService` .

### 53.7.2 Custom Web Endpoints

Operations on an  `@Endpoint` ,  `@WebEndpoint` , or  `@EndpointWebExtension`  are automatically exposed over HTTP using Jersey, Spring MVC, or Spring WebFlux.

#### Web Endpoint Request Predicates

A request predicate is automatically generated for each operation on a web-exposed endpoint.

#### Path

The path of the predicate is determined by the ID of the endpoint and the base path of web-exposed endpoints. The default base path is  `/actuator` . For example, an endpoint with the ID  `sessions`  will use  `/actuator/sessions`  as its path in the predicate.

The path can be further customized by annotating one or more parameters of the operation method with  `@Selector` . Such a parameter is added to the path predicate as a path variable. The variable’s value is passed into the operation method when the endpoint operation is invoked.

#### HTTP method

The HTTP method of the predicate is determined by the operation type, as shown in the following table:

|Operation|HTTP method|
|----|----|
| `@ReadOperation`  | `GET`  |
| `@WriteOperation`  | `POST`  |
| `@DeleteOperation`  | `DELETE`  |

#### Consumes

For a  `@WriteOperation`  (HTTP  `POST` ) that uses the request body, the consumes clause of the predicate is  `application/vnd.spring-boot.actuator.v2+json, application/json` . For all other operations the consumes clause is empty.

#### Produces

The produces clause of the predicate can be determined by the  `produces`  attribute of the  `@DeleteOperation` ,  `@ReadOperation` , and  `@WriteOperation`  annotations. The attribute is optional. If it is not used, the produces clause is determined automatically.

If the operation method returns  `void`  or  `Void`  the produces clause is empty. If the operation method returns a  `org.springframework.core.io.Resource` , the produces clause is  `application/octet-stream` . For all other operations the produces clause is  `application/vnd.spring-boot.actuator.v2+json, application/json` .

#### Web Endpoint Response Status

The default response status for an endpoint operation depends on the operation type (read, write, or delete) and what, if anything, the operation returns.

A  `@ReadOperation`  returns a value, the response status will be 200 (OK). If it does not return a value, the response status will be 404 (Not Found).

If a  `@WriteOperation`  or  `@DeleteOperation`  returns a value, the response status will be 200 (OK). If it does not return a value the response status will be 204 (No Content).

If an operation is invoked without a required parameter, or with a parameter that cannot be converted to the required type, the operation method will not be called and the response status will be 400 (Bad Request).

#### Web Endpoint Range Requests

An HTTP range request can be used to request part of an HTTP resource. When using Spring MVC or Spring Web Flux, operations that return a  `org.springframework.core.io.Resource`  automatically support range requests.

> Range requests are not supported when using Jersey.

#### Web Endpoint Security

An operation on a web endpoint or a web-specific endpoint extension can receive the current  `java.security.Principal`  or  `org.springframework.boot.actuate.endpoint.SecurityContext`  as a method parameter. The former is typically used in conjunction with  `@Nullable`  to provide different behaviour for authenticated and unauthenticated users. The latter is typically used to perform authorization checks using its  `isUserInRole(String)`  method.

### 53.7.3 Servlet endpoints

A  `Servlet`  can be exposed as an endpoint by implementing a class annotated with  `@ServletEndpoint`  that also implements  `Supplier<EndpointServlet>` . Servlet endpoints provide deeper integration with the Servlet container but at the expense of portability. They are intended to be used to expose an existing  `Servlet`  as an endpoint. For new endpoints, the  `@Endpoint`  and  `@WebEndpoint`  annotations should be preferred whenever possible.

### 53.7.4 Controller endpoints

`@ControllerEndpoint`  and  `@RestControllerEndpoint`  can be used to implement an endpoint that is only exposed by Spring MVC or Spring WebFlux. Methods are mapped using the standard annotations for Spring MVC and Spring WebFlux such as  `@RequestMapping`  and  `@GetMapping` , with the endpoint’s ID being used as a prefix for the path. Controller endpoints provide deeper integration with Spring’s web frameworks but at the expense of portability. The  `@Endpoint`  and  `@WebEndpoint`  annotations should be preferred whenever possible.

## 53.8 Health Information

You can use health information to check the status of your running application. It is often used by monitoring software to alert someone when a production system goes down. The information exposed by the  `health`  endpoint depends on the  `management.endpoint.health.show-details`  property which can be configured with one of the following values:

|Name|Description|
|----|----|
| `never`  |Details are never shown. |
| `when-authorized`  |Details are only shown to authorized users. Authorized roles can be configured using  `management.endpoint.health.roles` . |
| `always`  |Details are shown to all users. |

The default value is  `never` . A user is considered to be authorized when they are in one or more of the endpoint’s roles. If the endpoint has no configured roles (the default) all authenticated users are considered to be authorized. The roles can be configured using the  `management.endpoint.health.roles`  property.

> If you have secured your application and wish to use  `always` , your security configuration must permit access to the health endpoint for both authenticated and unauthenticated users.

Health information is collected from the content of a [HealthIndicatorRegistry](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthIndicatorRegistry.java) (by default all [HealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthIndicator.java) instances defined in your  `ApplicationContext` . Spring Boot includes a number of auto-configured  `HealthIndicators`  and you can also write your own. By default, the final system state is derived by the  `HealthAggregator`  which sorts the statuses from each  `HealthIndicator`  based on an ordered list of statuses. The first status in the sorted list is used as the overall health status. If no  `HealthIndicator`  returns a status that is known to the  `HealthAggregator` , an  `UNKNOWN`  status is used.

> The  `HealthIndicatorRegistry`  can be used to register and unregister health indicators at runtime.

### 53.8.1 Auto-configured HealthIndicators

The following  `HealthIndicators`  are auto-configured by Spring Boot when appropriate:

|Name|Description|
|----|----|
|[CassandraHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/cassandra/CassandraHealthIndicator.java) |Checks that a Cassandra database is up. |
|[CouchbaseHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/couchbase/CouchbaseHealthIndicator.java) |Checks that a Couchbase cluster is up. |
|[DiskSpaceHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/system/DiskSpaceHealthIndicator.java) |Checks for low disk space. |
|[DataSourceHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/jdbc/DataSourceHealthIndicator.java) |Checks that a connection to  `DataSource`  can be obtained. |
|[ElasticsearchHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/elasticsearch/ElasticsearchHealthIndicator.java) |Checks that an Elasticsearch cluster is up. |
|[InfluxDbHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/influx/InfluxDbHealthIndicator.java) |Checks that an InfluxDB server is up. |
|[JmsHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/jms/JmsHealthIndicator.java) |Checks that a JMS broker is up. |
|[MailHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/mail/MailHealthIndicator.java) |Checks that a mail server is up. |
|[MongoHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/mongo/MongoHealthIndicator.java) |Checks that a Mongo database is up. |
|[Neo4jHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/neo4j/Neo4jHealthIndicator.java) |Checks that a Neo4j server is up. |
|[RabbitHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/amqp/RabbitHealthIndicator.java) |Checks that a Rabbit server is up. |
|[RedisHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/redis/RedisHealthIndicator.java) |Checks that a Redis server is up. |
|[SolrHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/solr/SolrHealthIndicator.java) |Checks that a Solr server is up. |

> You can disable them all by setting the  `management.health.defaults.enabled`  property.

### 53.8.2 Writing Custom HealthIndicators

To provide custom health information, you can register Spring beans that implement the [HealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthIndicator.java) interface. You need to provide an implementation of the  `health()`  method and return a  `Health`  response. The  `Health`  response should include a status and can optionally include additional details to be displayed. The following code shows a sample  `HealthIndicator`  implementation:

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

> The identifier for a given  `HealthIndicator`  is the name of the bean without the  `HealthIndicator`  suffix, if it exists. In the preceding example, the health information is available in an entry named  `my` .

In addition to Spring Boot’s predefined [Status](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/Status.java) types, it is also possible for  `Health`  to return a custom  `Status`  that represents a new system state. In such cases, a custom implementation of the [HealthAggregator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthAggregator.java) interface also needs to be provided, or the default implementation has to be configured by using the  `management.health.status.order`  configuration property.

For example, assume a new  `Status`  with code  `FATAL`  is being used in one of your  `HealthIndicator`  implementations. To configure the severity order, add the following property to your application properties:

```java
management.health.status.order=FATAL, DOWN, OUT_OF_SERVICE, UNKNOWN, UP
```

The HTTP status code in the response reflects the overall health status (for example,  `UP`  maps to 200, while  `OUT_OF_SERVICE`  and  `DOWN`  map to 503). You might also want to register custom status mappings if you access the health endpoint over HTTP. For example, the following property maps  `FATAL`  to 503 (service unavailable):

```java
management.health.status.http-mapping.FATAL=503
```

> If you need more control, you can define your own  `HealthStatusHttpMapper`  bean.

The following table shows the default status mappings for the built-in statuses:

|Status|Mapping|
|----|----|
|DOWN |SERVICE_UNAVAILABLE (503) |
|OUT_OF_SERVICE |SERVICE_UNAVAILABLE (503) |
|UP |No mapping by default, so http status is 200 |
|UNKNOWN |No mapping by default, so http status is 200 |

### 53.8.3 Reactive Health Indicators

For reactive applications, such as those using Spring WebFlux,  `ReactiveHealthIndicator`  provides a non-blocking contract for getting application health. Similar to a traditional  `HealthIndicator` , health information is collected from the content of a [ReactiveHealthIndicatorRegistry](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/ReactiveHealthIndicatorRegistry.java) (by default all [HealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthIndicator.java) and [ReactiveHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/ReactiveHealthIndicator.java) instances defined in your  `ApplicationContext` . Regular  `HealthIndicator`  that do not check against a reactive API are executed on the elastic scheduler.

> In a reactive application, The  `ReactiveHealthIndicatorRegistry`  can be used to register and unregister health indicators at runtime.

To provide custom health information from a reactive API, you can register Spring beans that implement the [ReactiveHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/ReactiveHealthIndicator.java) interface. The following code shows a sample  `ReactiveHealthIndicator`  implementation:

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

> To handle the error automatically, consider extending from  `AbstractReactiveHealthIndicator` .

### 53.8.4 Auto-configured ReactiveHealthIndicators

The following  `ReactiveHealthIndicators`  are auto-configured by Spring Boot when appropriate:

|Name|Description|
|----|----|
|[CassandraReactiveHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/cassandra/CassandraReactiveHealthIndicator.java) |Checks that a Cassandra database is up. |
|[CouchbaseReactiveHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/couchbase/CouchbaseReactiveHealthIndicator.java) |Checks that a Couchbase cluster is up. |
|[MongoReactiveHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/mongo/MongoReactiveHealthIndicator.java) |Checks that a Mongo database is up. |
|[RedisReactiveHealthIndicator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/redis/RedisReactiveHealthIndicator.java) |Checks that a Redis server is up. |

> If necessary, reactive indicators replace the regular ones. Also, any  `HealthIndicator`  that is not handled explicitly is wrapped automatically.

## 53.9 Application Information

Application information exposes various information collected from all [InfoContributor](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/info/InfoContributor.java) beans defined in your  `ApplicationContext` . Spring Boot includes a number of auto-configured  `InfoContributor`  beans, and you can write your own.

### 53.9.1 Auto-configured InfoContributors

The following  `InfoContributor`  beans are auto-configured by Spring Boot, when appropriate:

|Name|Description|
|----|----|
|[EnvironmentInfoContributor](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/info/EnvironmentInfoContributor.java) |Exposes any key from the  `Environment`  under the  `info`  key. |
|[GitInfoContributor](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/info/GitInfoContributor.java) |Exposes git information if a  `git.properties`  file is available. |
|[BuildInfoContributor](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/info/BuildInfoContributor.java) |Exposes build information if a  `META-INF/build-info.properties`  file is available. |

> It is possible to disable them all by setting the  `management.info.defaults.enabled`  property.

### 53.9.2 Custom Application Information

You can customize the data exposed by the  `info`  endpoint by setting  `info.*`  Spring properties. All  `Environment`  properties under the  `info`  key are automatically exposed. For example, you could add the following settings to your  `application.properties`  file:

```java
info.app.encoding=UTF-8
info.app.java.source=1.8
info.app.java.target=1.8
```

> Rather than hardcoding those values, you could also [expand info properties at build time](howto-properties-and-configuration.html#howto-automatic-expansion).

### 53.9.3 Git Commit Information

Another useful feature of the  `info`  endpoint is its ability to publish information about the state of your  `git`  source code repository when the project was built. If a  `GitProperties`  bean is available, the  `git.branch` ,  `git.commit.id` , and  `git.commit.time`  properties are exposed.

> A  `GitProperties`  bean is auto-configured if a  `git.properties`  file is available at the root of the classpath. See "[Generate git information](howto-build.html#howto-git-info)" for more details.

If you want to display the full git information (that is, the full content of  `git.properties` ), use the  `management.info.git.mode`  property, as follows:

```java
management.info.git.mode=full
```

### 53.9.4 Build Information

If a  `BuildProperties`  bean is available, the  `info`  endpoint can also publish information about your build. This happens if a  `META-INF/build-info.properties`  file is available in the classpath.

> The Maven and Gradle plugins can both generate that file. See "[Generate build information](howto-build.html#howto-build-info)" for more details.

### 53.9.5 Writing Custom InfoContributors

To provide custom application information, you can register Spring beans that implement the [InfoContributor](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/info/InfoContributor.java) interface.

The following example contributes an  `example`  entry with a single value:

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

If you reach the  `info`  endpoint, you should see a response that contains the following additional entry:

```java
{
	"example": {
		"key" : "value"
	}
}
```

