## 73. Service Discovery with Zookeeper

Service Discovery is one of the key tenets of a microservice based architecture. Trying to hand-configure each client or some form of convention can be difficult to do and can be brittle. [Curator](https://curator.apache.org)(A Java library for Zookeeper) provides Service Discovery through a [Service Discovery Extension](https://curator.apache.org/curator-x-discovery/). Spring Cloud Zookeeper uses this extension for service registration and discovery.

## 73.1 Activating

Including a dependency on  `org.springframework.cloud:spring-cloud-starter-zookeeper-discovery`  enables autoconfiguration that sets up Spring Cloud Zookeeper Discovery.

> For web functionality, you still need to include  `org.springframework.boot:spring-boot-starter-web` .

> When working with version 3.4 of Zookeeper you need to change the way you include the dependency as described [here](multi_spring-cloud-zookeeper-install.html).

## 73.2 Registering with Zookeeper

When a client registers with Zookeeper, it provides metadata (such as host and port, ID, and name) about itself.

The following example shows a Zookeeper client:

```java
@SpringBootApplication
@RestController
public class Application {

@RequestMapping("/")
public String home() {
return "Hello world";
}

public static void main(String[] args) {
new SpringApplicationBuilder(Application.class).web(true).run(args);
}

}
```

> The preceding example is a normal Spring Boot application.

If Zookeeper is located somewhere other than  `localhost:2181` , the configuration must provide the location of the server, as shown in the following example:

**application.yml.**  

```java
spring:
cloud:
zookeeper:
connect-string: localhost:2181
```

> If you use [Spring Cloud Zookeeper Config](multi_spring-cloud-zookeeper-config.html), the values shown in the preceding example need to be in  `bootstrap.yml`  instead of  `application.yml` .

The default service name, instance ID, and port (taken from the  `Environment` ) are  `${spring.application.name}` , the Spring Context ID, and  `${server.port}` , respectively.

Having  `spring-cloud-starter-zookeeper-discovery`  on the classpath makes the app into both a Zookeeper “service” (that is, it registers itself) and a “client” (that is, it can query Zookeeper to locate other services).

If you would like to disable the Zookeeper Discovery Client, you can set  `spring.cloud.zookeeper.discovery.enabled`  to  `false` .

## 73.3 Using the DiscoveryClient

Spring Cloud has support for [Feign](https://github.com/spring-cloud/spring-cloud-netflix/blob/master/docs/src/main/asciidoc/spring-cloud-netflix.adoc#spring-cloud-feign) (a REST client builder) and [Spring RestTemplate](https://github.com/spring-cloud/spring-cloud-netflix/blob/master/docs/src/main/asciidoc/spring-cloud-netflix.adoc#spring-cloud-ribbon), using logical service names instead of physical URLs.

You can also use the  `org.springframework.cloud.client.discovery.DiscoveryClient` , which provides a simple API for discovery clients that is not specific to Netflix, as shown in the following example:

```java
@Autowired
private DiscoveryClient discoveryClient;

public String serviceUrl() {
List<ServiceInstance> list = discoveryClient.getInstances("STORES");
if (list != null && list.size() > 0 ) {
return list.get(0).getUri().toString();
}
return null;
}
```

