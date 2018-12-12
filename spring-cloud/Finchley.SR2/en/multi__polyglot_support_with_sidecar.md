## 19. Polyglot support with Sidecar

Do you have non-JVM languages with which you want to take advantage of Eureka, Ribbon, and Config Server? The Spring Cloud Netflix Sidecar was inspired by [Netflix Prana](https://github.com/Netflix/Prana). It includes an HTTP API to get all of the instances (by host and port) for a given service. You can also proxy service calls through an embedded Zuul proxy that gets its route entries from Eureka. The Spring Cloud Config Server can be accessed directly through host lookup or through the Zuul Proxy. The non-JVM application should implement a health check so the Sidecar can report to Eureka whether the app is up or down.

To include Sidecar in your project, use the dependency with a group ID of  `org.springframework.cloud`  and artifact ID or  `spring-cloud-netflix-sidecar` .

To enable the Sidecar, create a Spring Boot application with  `@EnableSidecar` . This annotation includes  `@EnableCircuitBreaker` ,  `@EnableDiscoveryClient` , and  `@EnableZuulProxy` . Run the resulting application on the same host as the non-JVM application.

To configure the side car, add  `sidecar.port`  and  `sidecar.health-uri`  to  `application.yml` . The  `sidecar.port`  property is the port on which the non-JVM application listens. This is so the Sidecar can properly register the application with Eureka. The  `sidecar.health-uri`  is a URI accessible on the non-JVM application that mimics a Spring Boot health indicator. It should return a JSON document that resembles the following:

**health-uri-document.**  

```java
{
"status":"UP"
}
```

The following application.yml example shows sample configuration for a Sidecar application:

**application.yml.**  

```java
server:
port: 5678
spring:
application:
name: sidecar

sidecar:
port: 8000
health-uri: http://localhost:8000/health.json
```

The API for the  `DiscoveryClient.getInstances()`  method is  `/hosts/{serviceId}` . The following example response for  `/hosts/customers`  returns two instances on different hosts:

**/hosts/customers.**  

```java
[
{
"host": "myhost",
"port": 9000,
"uri": "http://myhost:9000",
"serviceId": "CUSTOMERS",
"secure": false
},
{
"host": "myhost2",
"port": 9000,
"uri": "http://myhost2:9000",
"serviceId": "CUSTOMERS",
"secure": false
}
]
```

This API is accessible to the non-JVM application (if the sidecar is on port 5678) at  `http://localhost:5678/hosts/{serviceId}` .

The Zuul proxy automatically adds routes for each service known in Eureka to  `/<serviceId>` , so the customers service is available at  `/customers` . The non-JVM application can access the customer service at  `http://localhost:5678/customers`  (assuming the sidecar is listening on port 5678).

If the Config Server is registered with Eureka, the non-JVM application can access it through the Zuul proxy. If the  `serviceId`  of the ConfigServer is  `configserver`  and the Sidecar is on port 5678, then it can be accessed at [http://localhost:5678/configserver](http://localhost:5678/configserver).

Non-JVM applications can take advantage of the Config Server’s ability to return YAML documents. For example, a call to [http://sidecar.local.spring.io:5678/configserver/default-master.yml](http://sidecar.local.spring.io:5678/configserver/default-master.yml) might result in a YAML document resembling the following:

```java
eureka:
client:
serviceUrl:
defaultZone: http://localhost:8761/eureka/
password: password
info:
description: Spring Cloud Samples
url: https://github.com/spring-cloud-samples
```

To enable the health check request to accept all certificates when using HTTPs set  `sidecar.accept-all-ssl-certificates`  to `true.
