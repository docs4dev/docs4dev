## 119. CORS Configuration

The gateway can be configured to control CORS behavior. The "global" CORS configuration is a map of URL patterns to [Spring Framework CorsConfiguration](https://docs.spring.io/spring/docs/5.0.x/javadoc-api/org/springframework/web/cors/CorsConfiguration.html).

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

In the example above, CORS requests will be allowed from requests that originate from docs.spring.io for all GET requested paths.
