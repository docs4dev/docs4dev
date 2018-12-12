## 66. Service Discovery with Consul

Service Discovery is one of the key tenets of a microservice based architecture. Trying to hand configure each client or some form of convention can be very difficult to do and can be very brittle. Consul provides Service Discovery services via an [HTTP API](https://www.consul.io/docs/agent/http.html) and [DNS](https://www.consul.io/docs/agent/dns.html). Spring Cloud Consul leverages the HTTP API for service registration and discovery. This does not prevent non-Spring Cloud applications from leveraging the DNS interface. Consul Agents servers are run in a [cluster](https://www.consul.io/docs/internals/architecture.html) that communicates via a [gossip protocol](https://www.consul.io/docs/internals/gossip.html) and uses the [Raft consensus protocol](https://www.consul.io/docs/internals/consensus.html).

## 66.1 How to activate

To activate Consul Service Discovery use the starter with group  `org.springframework.cloud`  and artifact id  `spring-cloud-starter-consul-discovery` . See the [Spring Cloud Project page](https://projects.spring.io/spring-cloud/) for details on setting up your build system with the current Spring Cloud Release Train.

## 66.2 Registering with Consul

When a client registers with Consul, it provides meta-data about itself such as host and port, id, name and tags. An HTTP [Check](https://www.consul.io/docs/agent/checks.html) is created by default that Consul hits the  `/health`  endpoint every 10 seconds. If the health check fails, the service instance is marked as critical.

Example Consul client:

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

(i.e. utterly normal Spring Boot app). If the Consul client is located somewhere other than  `localhost:8500` , the configuration is required to locate the client. Example:

**application.yml.**  

```java
spring:
cloud:
consul:
host: localhost
port: 8500
```

> If you use [Spring Cloud Consul Config](multi_spring-cloud-consul-config.html), the above values will need to be placed in  `bootstrap.yml`  instead of  `application.yml` .

The default service name, instance id and port, taken from the  `Environment` , are  `${spring.application.name}` , the Spring Context ID and  `${server.port}`  respectively.

To disable the Consul Discovery Client you can set  `spring.cloud.consul.discovery.enabled`  to  `false` .

To disable the service registration you can set  `spring.cloud.consul.discovery.register`  to  `false` .

## 66.3 HTTP Health Check

The health check for a Consul instance defaults to "/health", which is the default locations of a useful endpoint in a Spring Boot Actuator application. You need to change these, even for an Actuator application if you use a non-default context path or servlet path (e.g.  `server.servletPath=/foo` ) or management endpoint path (e.g.  `management.server.servlet.context-path=/admin` ). The interval that Consul uses to check the health endpoint may also be configured. "10s" and "1m" represent 10 seconds and 1 minute respectively. Example:

**application.yml.**  

```java
spring:
cloud:
consul:
discovery:
healthCheckPath: ${management.server.servlet.context-path}/health
healthCheckInterval: 15s
```

You can disable the health check by setting  `management.health.consul.enabled=false` .

### 66.3.1 Metadata and Consul tags

Consul does not yet support metadata on services. Spring Cloud’s  `ServiceInstance`  has a  `Map<String, String> metadata`  field. Spring Cloud Consul uses Consul tags to approximate metadata until Consul officially supports metadata. Tags with the form  `key=value`  will be split and used as a  `Map`  key and value respectively. Tags without the equal  `=`  sign, will be used as both the key and value.

**application.yml.**  

```java
spring:
cloud:
consul:
discovery:
tags: foo=bar, baz
```

The above configuration will result in a map with  `foo→bar`  and  `baz→baz` .

### 66.3.2 Making the Consul Instance ID Unique

By default a consul instance is registered with an ID that is equal to its Spring Application Context ID. By default, the Spring Application Context ID is  `${spring.application.name}:comma,separated,profiles:${server.port}` . For most cases, this will allow multiple instances of one service to run on one machine. If further uniqueness is required, Using Spring Cloud you can override this by providing a unique identifier in  `spring.cloud.consul.discovery.instanceId` . For example:

**application.yml.**  

```java
spring:
cloud:
consul:
discovery:
instanceId: ${spring.application.name}:${vcap.application.instance_id:${spring.application.instance_id:${random.value}}}
```

With this metadata, and multiple service instances deployed on localhost, the random value will kick in there to make the instance unique. In Cloudfoundry the  `vcap.application.instance_id`  will be populated automatically in a Spring Boot application, so the random value will not be needed.

## 66.4 Looking up services

### 66.4.1 Using Ribbon

Spring Cloud has support for [Feign](https://github.com/spring-cloud/spring-cloud-netflix/blob/master/docs/src/main/asciidoc/spring-cloud-netflix.adoc#spring-cloud-feign) (a REST client builder) and also [Spring RestTemplate](https://github.com/spring-cloud/spring-cloud-netflix/blob/master/docs/src/main/asciidoc/spring-cloud-netflix.adoc#spring-cloud-ribbon) for looking up services using the logical service names/ids instead of physical URLs. Both Feign and the discovery-aware RestTemplate utilize [Ribbon](https://cloud.spring.io/spring-cloud-netflix/single/spring-cloud-netflix.html#spring-cloud-ribbon) for client-side load balancing.

If you want to access service STORES using the RestTemplate simply declare:

```java
@LoadBalanced
@Bean
public RestTemplate loadbalancedRestTemplate() {
new RestTemplate();
}
```

and use it like this (notice how we use the STORES service name/id from Consul instead of a fully qualified domainname):

```java
@Autowired
RestTemplate restTemplate;

public String getFirstProduct() {
return this.restTemplate.getForObject("https://STORES/products/1", String.class);
}
```

If you have Consul clusters in multiple datacenters and you want to access a service in another datacenter a service name/id alone is not enough. In that case you use property  `spring.cloud.consul.discovery.datacenters.STORES=dc-west`  where  `STORES`  is the service name/id and  `dc-west`  is the datacenter where the STORES service lives.

### 66.4.2 Using the DiscoveryClient

You can also use the  `org.springframework.cloud.client.discovery.DiscoveryClient`  which provides a simple API for discovery clients that is not specific to Netflix, e.g.

```java
@Autowired
private DiscoveryClient discoveryClient;

public String serviceUrl() {
List<ServiceInstance> list = discoveryClient.getInstances("STORES");
if (list != null && list.size() > 0 ) {
return list.get(0).getUri();
}
return null;
}
```

## 66.5 Consul Catalog Watch

The Consul Catalog Watch takes advantage of the ability of consul to [watch services](https://www.consul.io/docs/agent/watches.html#services). The Catalog Watch makes a blocking Consul HTTP API call to determine if any services have changed. If there is new service data a Heartbeat Event is published.

To change the frequency of when the Config Watch is called change  `spring.cloud.consul.config.discovery.catalog-services-watch-delay` . The default value is 1000, which is in milliseconds. The delay is the amount of time after the end of the previous invocation and the start of the next.

To disable the Catalog Watch set  `spring.cloud.consul.discovery.catalogServicesWatch.enabled=false` .

The watch uses a Spring  `TaskScheduler`  to schedule the call to consul. By default it is a  `ThreadPoolTaskScheduler`  with a  `poolSize`  of 1. To change the  `TaskScheduler` , create a bean of type  `TaskScheduler`  named with the  `ConsulDiscoveryClientConfiguration.CATALOG_WATCH_TASK_SCHEDULER_NAME`  constant.

