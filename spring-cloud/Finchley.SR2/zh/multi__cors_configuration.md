## 119. CORS配置

网关可以配置为控制CORS行为. "global" CORS配置是[Spring Framework CorsConfiguration](https://docs.spring.io/spring/docs/5.0.x/javadoc-api/org/springframework/web/cors/CorsConfiguration.html)的URL模式映射.

**application.yml.** 

```java
spring:
cloud:
gateway:
globalcors:
corsConfigurations:
'[/**]':
allowedOrigins: "docs.spring.io"
allowedMethods:
- GET
```

在上面的示例中，对于所有GET请求的路径，将允许来自docs.spring.io的请求的CORS请求.
