## 70.与Hystrix的断路器

应用程序可以使用Spring Cloud Netflix项目提供的Hystrix断路器，将此启动器包含在项目pom.xml中： `spring-cloud-starter-hystrix` . Hystrix不依赖于Netflix Discovery Client.  `@EnableHystrix` 注释应放在配置类（通常是主类）上.然后可以用 `@HystrixCommand` 注释方法以受断路器保护.有关详细信息，请参阅[the documentation](https://projects.spring.io/spring-cloud/spring-cloud.html#_circuit_breaker_hystrix_clients).
