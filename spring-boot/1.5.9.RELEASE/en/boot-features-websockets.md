## 46. WebSockets

Spring Boot provides WebSockets auto-configuration for embedded Tomcat, Jetty, and Undertow. If you deploy a war file to a standalone container, Spring Boot assumes that the container is responsible for the configuration of its WebSocket support.

Spring Framework provides [rich WebSocket support](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#websocket) for MVC web applications that can be easily accessed through the  `spring-boot-starter-websocket`  module.

WebSocket support is also available for [reactive web applications](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web-reactive.html#webflux-websocket) and requires to include the WebSocket API alongside  `spring-boot-starter-webflux` :

```xml
<dependency>
	<groupId>javax.websocket</groupId>
	<artifactId>javax.websocket-api</artifactId>
</dependency>
```
