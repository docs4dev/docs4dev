## 111. How to Include Spring Cloud Gateway

To include Spring Cloud Gateway in your project use the starter with group  `org.springframework.cloud`  and artifact id  `spring-cloud-starter-gateway` . See the [Spring Cloud Project page](https://projects.spring.io/spring-cloud/) for details on setting up your build system with the current Spring Cloud Release Train.

If you include the starter, but, for some reason, you do not want the gateway to be enabled, set  `spring.cloud.gateway.enabled=false` .

|images/important.png|Important|
|----|----|
|Spring Cloud Gateway requires the Netty runtime provided by Spring Boot and Spring Webflux. It does not work in a traditional Servlet Container or built as a WAR. |

