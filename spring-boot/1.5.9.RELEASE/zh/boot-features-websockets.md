## 46. WebSockets

Spring Boot为嵌入式Tomcat，Jetty和Undertow提供WebSockets自动配置.如果将war文件部署到独立容器，则Spring Boot会假定容器负责其WebSocket支持的配置.

Spring Framework为MVC Web应用程序提供[rich WebSocket support](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#websocket)，可以通过 `spring-boot-starter-websocket` 模块轻松访问.

[reactive web applications](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web-reactive.html#webflux-websocket)也可以使用WebSocket支持，并且需要在 `spring-boot-starter-webflux` 旁边包含WebSocket API：

```xml
<dependency>
	<groupId>javax.websocket</groupId>
	<artifactId>javax.websocket-api</artifactId>
</dependency>
```
