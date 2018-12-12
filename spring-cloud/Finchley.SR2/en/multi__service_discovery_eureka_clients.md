## 11. Service Discovery: Eureka Clients

Service Discovery is one of the key tenets of a microservice-based architecture. Trying to hand-configure each client or some form of convention can be difficult to do and can be brittle. Eureka is the Netflix Service Discovery Server and Client. The server can be configured and deployed to be highly available, with each server replicating state about the registered services to the others.

## 11.1 How to Include Eureka Client

To include the Eureka Client in your project, use the starter with a group ID of  `org.springframework.cloud`  and an artifact ID of  `spring-cloud-starter-netflix-eureka-client` . See the [Spring Cloud Project page](https://projects.spring.io/spring-cloud/) for details on setting up your build system with the current Spring Cloud Release Train.

## 11.2 Registering with Eureka

When a client registers with Eureka, it provides meta-data about itself — such as host, port, health indicator URL, home page, and other details. Eureka receives heartbeat messages from each instance belonging to a service. If the heartbeat fails over a configurable timetable, the instance is normally removed from the registry.

The following example shows a minimal Eureka client application:

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

Note that the preceding example shows a normal [Spring Boot](https://projects.spring.io/spring-boot/) application. By having  `spring-cloud-starter-netflix-eureka-client`  on the classpath, your application automatically registers with the Eureka Server. Configuration is required to locate the Eureka server, as shown in the following example:

**application.yml.**  

```java
eureka:
client:
serviceUrl:
defaultZone: http://localhost:8761/eureka/
```

In the preceding example, "defaultZone" is a magic string fallback value that provides the service URL for any client that does not express a preference (in other words, it is a useful default).

The default application name (that is, the service ID), virtual host, and non-secure port (taken from the  `Environment` ) are  `${spring.application.name}` ,  `${spring.application.name}`  and  `${server.port}` , respectively.

Having  `spring-cloud-starter-netflix-eureka-client`  on the classpath makes the app into both a Eureka “instance” (that is, it registers itself) and a “client” (it can query the registry to locate other services). The instance behaviour is driven by  `eureka.instance.*`  configuration keys, but the defaults are fine if you ensure that your application has a value for  `spring.application.name`  (this is the default for the Eureka service ID or VIP).

See [EurekaInstanceConfigBean](https://github.com/spring-cloud/spring-cloud-netflix/tree/master/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaInstanceConfigBean.java) and [EurekaClientConfigBean](https://github.com/spring-cloud/spring-cloud-netflix/tree/master/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaClientConfigBean.java) for more details on the configurable options.

To disable the Eureka Discovery Client, you can set  `eureka.client.enabled`  to  `false` .

## 11.3 Authenticating with the Eureka Server

HTTP basic authentication is automatically added to your eureka client if one of the  `eureka.client.serviceUrl.defaultZone`  URLs has credentials embedded in it (curl style, as follows:  `http://user:[email protected]:8761/eureka` ). For more complex needs, you can create a  `@Bean`  of type  `DiscoveryClientOptionalArgs`  and inject  `ClientFilter`  instances into it, all of which is applied to the calls from the client to the server.

> Because of a limitation in Eureka, it is not possible to support per-server basic auth credentials, so only the first set that are found is used.

## 11.4 Status Page and Health Indicator

The status page and health indicators for a Eureka instance default to  `/info`  and  `/health`  respectively, which are the default locations of useful endpoints in a Spring Boot Actuator application. You need to change these, even for an Actuator application if you use a non-default context path or servlet path (such as  `server.servletPath=/custom` ). The following example shows the default values for the two settings:

**application.yml.**  

```java
eureka:
instance:
statusPageUrlPath: ${server.servletPath}/info
healthCheckUrlPath: ${server.servletPath}/health
```

These links show up in the metadata that is consumed by clients and are used in some scenarios to decide whether to send requests to your application, so it is helpful if they are accurate.

> In Dalston it was also required to set the status and health check URLs when changing that management context path. This requirement was removed beginning in Edgware.

## 11.5 Registering a Secure Application

If your app wants to be contacted over HTTPS, you can set two flags in the  `EurekaInstanceConfig` :

-  `eureka.instance.[nonSecurePortEnabled]=[false]` 

-  `eureka.instance.[securePortEnabled]=[true]` 

Doing so makes Eureka publish instance information that shows an explicit preference for secure communication. The Spring Cloud  `DiscoveryClient`  always returns a URI starting with  `https`  for a service configured this way. Similarly, when a service is configured this way, the Eureka (native) instance information has a secure health check URL.

Because of the way Eureka works internally, it still publishes a non-secure URL for the status and home pages unless you also override those explicitly. You can use placeholders to configure the eureka instance URLs, as shown in the following example:

**application.yml.**  

```java
eureka:
instance:
statusPageUrl: https://${eureka.hostname}/info
healthCheckUrl: https://${eureka.hostname}/health
homePageUrl: https://${eureka.hostname}/
```

(Note that  `${eureka.hostname}`  is a native placeholder only available in later versions of Eureka. You could achieve the same thing with Spring placeholders as well — for example, by using  `${eureka.instance.hostName}` .)

> If your application runs behind a proxy, and the SSL termination is in the proxy (for example, if you run in Cloud Foundry or other platforms as a service), then you need to ensure that the proxy “forwarded” headers are intercepted and handled by the application. If the Tomcat container embedded in a Spring Boot application has explicit configuration for the 'X-Forwarded-\*` headers, this happens automatically. The links rendered by your app to itself being wrong (the wrong host, port, or protocol) is a sign that you got this configuration wrong.

## 11.6 Eureka’s Health Checks

By default, Eureka uses the client heartbeat to determine if a client is up. Unless specified otherwise, the Discovery Client does not propagate the current health check status of the application, per the Spring Boot Actuator. Consequently, after successful registration, Eureka always announces that the application is in 'UP' state. This behavior can be altered by enabling Eureka health checks, which results in propagating application status to Eureka. As a consequence, every other application does not send traffic to applications in states other then 'UP'. The following example shows how to enable health checks for the client:

**application.yml.**  

```java
eureka:
client:
healthcheck:
enabled: true
```

>  `eureka.client.healthcheck.enabled=true`  should only be set in  `application.yml` . Setting the value in  `bootstrap.yml`  causes undesirable side effects, such as registering in Eureka with an  `UNKNOWN`  status.

If you require more control over the health checks, consider implementing your own  `com.netflix.appinfo.HealthCheckHandler` .

## 11.7 Eureka Metadata for Instances and Clients

It is worth spending a bit of time understanding how the Eureka metadata works, so you can use it in a way that makes sense in your platform. There is standard metadata for information such as hostname, IP address, port numbers, the status page, and health check. These are published in the service registry and used by clients to contact the services in a straightforward way. Additional metadata can be added to the instance registration in the  `eureka.instance.metadataMap` , and this metadata is accessible in the remote clients. In general, additional metadata does not change the behavior of the client, unless the client is made aware of the meaning of the metadata. There are a couple of special cases, described later in this document, where Spring Cloud already assigns meaning to the metadata map.

### 11.7.1 Using Eureka on Cloud Foundry

Cloud Foundry has a global router so that all instances of the same app have the same hostname (other PaaS solutions with a similar architecture have the same arrangement). This is not necessarily a barrier to using Eureka. However, if you use the router (recommended or even mandatory, depending on the way your platform was set up), you need to explicitly set the hostname and port numbers (secure or non-secure) so that they use the router. You might also want to use instance metadata so that you can distinguish between the instances on the client (for example, in a custom load balancer). By default, the  `eureka.instance.instanceId`  is  `vcap.application.instance_id` , as shown in the following example:

**application.yml.**  

```java
eureka:
instance:
hostname: ${vcap.application.uris[0]}
nonSecurePort: 80
```

Depending on the way the security rules are set up in your Cloud Foundry instance, you might be able to register and use the IP address of the host VM for direct service-to-service calls. This feature is not yet available on Pivotal Web Services ([PWS](https://run.pivotal.io)).

### 11.7.2 Using Eureka on AWS

If the application is planned to be deployed to an AWS cloud, the Eureka instance must be configured to be AWS-aware. You can do so by customizing the [EurekaInstanceConfigBean](https://github.com/spring-cloud/spring-cloud-netflix/tree/master/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaInstanceConfigBean.java) as follows:

```java
@Bean
@Profile("!default")
public EurekaInstanceConfigBean eurekaInstanceConfig(InetUtils inetUtils) {
EurekaInstanceConfigBean b = new EurekaInstanceConfigBean(inetUtils);
AmazonInfo info = AmazonInfo.Builder.newBuilder().autoBuild("eureka");
b.setDataCenterInfo(info);
return b;
}
```

### 11.7.3 Changing the Eureka Instance ID

A vanilla Netflix Eureka instance is registered with an ID that is equal to its host name (that is, there is only one service per host). Spring Cloud Eureka provides a sensible default, which is defined as follows:

`${spring.cloud.client.hostname}:${spring.application.name}:${spring.application.instance_id:${server.port}}}` 

An example is  `myhost:myappname:8080` .

By using Spring Cloud, you can override this value by providing a unique identifier in  `eureka.instance.instanceId` , as shown in the following example:

**application.yml.**  

```java
eureka:
instance:
instanceId: ${spring.application.name}:${vcap.application.instance_id:${spring.application.instance_id:${random.value}}}
```

With the metadata shown in the preceding example and multiple service instances deployed on localhost, the random value is inserted there to make the instance unique. In Cloud Foundry, the  `vcap.application.instance_id`  is populated automatically in a Spring Boot application, so the random value is not needed.

## 11.8 Using the EurekaClient

Once you have an application that is a discovery client, you can use it to discover service instances from the [Eureka Server](multi_spring-cloud-eureka-server.html). One way to do so is to use the native  `com.netflix.discovery.EurekaClient`  (as opposed to the Spring Cloud  `DiscoveryClient` ), as shown in the following example:

```java
@Autowired
private EurekaClient discoveryClient;

public String serviceUrl() {
InstanceInfo instance = discoveryClient.getNextServerFromEureka("STORES", false);
return instance.getHomePageUrl();
}
```

> Do not use the  `EurekaClient`  in a  `@PostConstruct`  method or in a  `@Scheduled`  method (or anywhere where the  `ApplicationContext`  might not be started yet). It is initialized in a  `SmartLifecycle`  (with  `phase=0` ), so the earliest you can rely on it being available is in another  `SmartLifecycle`  with a higher phase.

### 11.8.1 EurekaClient without Jersey

By default, EurekaClient uses Jersey for HTTP communication. If you wish to avoid dependencies from Jersey, you can exclude it from your dependencies. Spring Cloud auto-configures a transport client based on Spring  `RestTemplate` . The following example shows Jersey being excluded:

```xml
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
<exclusions>
<exclusion>
<groupId>com.sun.jersey</groupId>
<artifactId>jersey-client</artifactId>
</exclusion>
<exclusion>
<groupId>com.sun.jersey</groupId>
<artifactId>jersey-core</artifactId>
</exclusion>
<exclusion>
<groupId>com.sun.jersey.contribs</groupId>
<artifactId>jersey-apache-client4</artifactId>
</exclusion>
</exclusions>
</dependency>
```

## 11.9 Alternatives to the Native Netflix EurekaClient

You need not use the raw Netflix  `EurekaClient` . Also, it is usually more convenient to use it behind a wrapper of some sort. Spring Cloud has support for [Feign](multi_spring-cloud-feign.html) (a REST client builder) and [Spring RestTemplate](multi_spring-cloud-ribbon.html) through the logical Eureka service identifiers (VIPs) instead of physical URLs. To configure Ribbon with a fixed list of physical servers, you can set  `<client>.ribbon.listOfServers`  to a comma-separated list of physical addresses (or hostnames), where  `<client>`  is the ID of the client.

You can also use the  `org.springframework.cloud.client.discovery.DiscoveryClient` , which provides a simple API (not specific to Netflix) for discovery clients, as shown in the following example:

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

## 11.10 Why Is It so Slow to Register a Service?

Being an instance also involves a periodic heartbeat to the registry (through the client’s  `serviceUrl` ) with a default duration of 30 seconds. A service is not available for discovery by clients until the instance, the server, and the client all have the same metadata in their local cache (so it could take 3 heartbeats). You can change the period by setting  `eureka.instance.leaseRenewalIntervalInSeconds` . Setting it to a value of less than 30 speeds up the process of getting clients connected to other services. In production, it is probably better to stick with the default, because of internal computations in the server that make assumptions about the lease renewal period.

## 11.11 Zones

If you have deployed Eureka clients to multiple zones, you may prefer that those clients use services within the same zone before trying services in another zone. To set that up, you need to configure your Eureka clients correctly.

First, you need to make sure you have Eureka servers deployed to each zone and that they are peers of each other. See the section on [zones and regions](multi_spring-cloud-eureka-server.html#spring-cloud-eureka-server-zones-and-regions) for more information.

Next, you need to tell Eureka which zone your service is in. You can do so by using the  `metadataMap`  property. For example, if  `service 1`  is deployed to both  `zone 1`  and  `zone 2` , you need to set the following Eureka properties in  `service 1` :

**Service 1 in Zone 1** 

```java
eureka.instance.metadataMap.zone = zone1
eureka.client.preferSameZoneEureka = true
```

**Service 1 in Zone 2** 

```java
eureka.instance.metadataMap.zone = zone2
eureka.client.preferSameZoneEureka = true
```

