## 111.如何包含Spring Cloud Gateway

要在项目中包含Spring Cloud Gateway，请使用包含组 `org.springframework.cloud` 和工件ID  `spring-cloud-starter-gateway` 的启动器.有关使用当前Spring Cloud Release Train设置构建系统的详细信息，请参阅[Spring Cloud Project page](https://projects.spring.io/spring-cloud/).

如果包含启动器，但由于某种原因，您不希望启用网关，请设置 `spring.cloud.gateway.enabled=false` .

|图片/ important.png |重要|
| ---- | ---- |
| Spring Cloud Gateway需要Spring Boot和Spring Webflux提供的Netty运行时.它不能在传统的Servlet容器中工作或构建为WAR. |

