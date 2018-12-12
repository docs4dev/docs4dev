## 12. Service Discovery: Eureka Server

This section describes how to set up a Eureka server.

## 12.1 How to Include Eureka Server

To include Eureka Server in your project, use the starter with a group ID of  `org.springframework.cloud`  and an artifact ID of  `spring-cloud-starter-netflix-eureka-server` . See the [Spring Cloud Project page](https://projects.spring.io/spring-cloud/) for details on setting up your build system with the current Spring Cloud Release Train.

## 12.2 How to Run a Eureka Server

The following example shows a minimal Eureka server:

```java
@SpringBootApplication
@EnableEurekaServer
public class Application {

public static void main(String[] args) {
new SpringApplicationBuilder(Application.class).web(true).run(args);
}

}
```

The server has a home page with a UI and HTTP API endpoints for the normal Eureka functionality under  `/eureka/*` .

The following links have some Eureka background reading: [flux capacitor](https://github.com/cfregly/fluxcapacitor/wiki/NetflixOSS-FAQ#eureka-service-discovery-load-balancer) and [google group discussion](https://groups.google.com/forum/?fromgroups#!topic/eureka_netflix/g3p2r7gHnN0).

> Due to Gradle’s dependency resolution rules and the lack of a parent bom feature, depending on  `spring-cloud-starter-netflix-eureka-server`  can cause failures on application startup. To remedy this issue, add the Spring Boot Gradle plugin and import the Spring cloud starter parent bom as follows:

## 12.3 High Availability, Zones and Regions

The Eureka server does not have a back end store, but the service instances in the registry all have to send heartbeats to keep their registrations up to date (so this can be done in memory). Clients also have an in-memory cache of Eureka registrations (so they do not have to go to the registry for every request to a service).

By default, every Eureka server is also a Eureka client and requires (at least one) service URL to locate a peer. If you do not provide it, the service runs and works, but it fills your logs with a lot of noise about not being able to register with the peer.

See also [below for details of Ribbon support](multi_spring-cloud-ribbon.html) on the client side for Zones and Regions.

## 12.4 Standalone Mode

The combination of the two caches (client and server) and the heartbeats make a standalone Eureka server fairly resilient to failure, as long as there is some sort of monitor or elastic runtime (such as Cloud Foundry) keeping it alive. In standalone mode, you might prefer to switch off the client side behavior so that it does not keep trying and failing to reach its peers. The following example shows how to switch off the client-side behavior:

**application.yml (Standalone Eureka Server).**  

```java
server:
port: 8761

eureka:
instance:
hostname: localhost
client:
registerWithEureka: false
fetchRegistry: false
serviceUrl:
defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

Notice that the  `serviceUrl`  is pointing to the same host as the local instance.

## 12.5 Peer Awareness

Eureka can be made even more resilient and available by running multiple instances and asking them to register with each other. In fact, this is the default behavior, so all you need to do to make it work is add a valid  `serviceUrl`  to a peer, as shown in the following example:

**application.yml (Two Peer Aware Eureka Servers).**  

```java
---
spring:
profiles: peer1
eureka:
instance:
hostname: peer1
client:
serviceUrl:
defaultZone: http://peer2/eureka/

---
spring:
profiles: peer2
eureka:
instance:
hostname: peer2
client:
serviceUrl:
defaultZone: http://peer1/eureka/
```

In the preceding example, we have a YAML file that can be used to run the same server on two hosts ( `peer1`  and  `peer2` ) by running it in different Spring profiles. You could use this configuration to test the peer awareness on a single host (there is not much value in doing that in production) by manipulating  `/etc/hosts`  to resolve the host names. In fact, the  `eureka.instance.hostname`  is not needed if you are running on a machine that knows its own hostname (by default, it is looked up by using  `java.net.InetAddress` ).

You can add multiple peers to a system, and, as long as they are all connected to each other by at least one edge, they synchronize the registrations amongst themselves. If the peers are physically separated (inside a data center or between multiple data centers), then the system can, in principle, survive “split-brain” type failures. You can add multiple peers to a system, and as long as they are all directly connected to each other, they will synchronize the registrations amongst themselves.

**application.yml (Three Peer Aware Eureka Servers).**  

```java
eureka:
client:
serviceUrl:
defaultZone: http://peer1/eureka/,http://peer2/eureka/,http://peer3/eureka/

---
spring:
profiles: peer1
eureka:
instance:
hostname: peer1

---
spring:
profiles: peer2
eureka:
instance:
hostname: peer2

---
spring:
profiles: peer3
eureka:
instance:
hostname: peer3
```

## 12.6 When to Prefer IP Address

In some cases, it is preferable for Eureka to advertise the IP addresses of services rather than the hostname. Set  `eureka.instance.preferIpAddress`  to  `true`  and, when the application registers with eureka, it uses its IP address rather than its hostname.

> If the hostname cannot be determined by Java, then the IP address is sent to Eureka. Only explict way of setting the hostname is by setting  `eureka.instance.hostname`  property. You can set your hostname at the run-time by using an environment variable — for example,  `eureka.instance.hostname=${HOST_NAME}` .

## 12.7 Securing The Eureka Server

You can secure your Eureka server simply by adding Spring Security to your server’s classpath via  `spring-boot-starter-security` . By default when Spring Security is on the classpath it will require that a valid CSRF token be sent with every request to the app. Eureka clients will not generally possess a valid cross site request forgery (CSRF) token you will need to disable this requirement for the  `/eureka/**`  endpoints. For example:

```java
@EnableWebSecurity
class WebSecurityConfig extends WebSecurityConfigurerAdapter {

@Override
protected void configure(HttpSecurity http) throws Exception {
http.csrf().ignoringAntMatchers("/eureka/**");
super.configure(http);
}
}
```

For more information on CSRF see the [Spring Security documentation](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#csrf).

A demo Eureka Server can be found in the Spring Cloud Samples [repo](https://github.com/spring-cloud-samples/eureka/tree/Eureka-With-Security).

