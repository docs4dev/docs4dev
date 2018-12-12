## 47.网络服务

Spring Boot提供Web服务自动配置，因此您必须做的就是定义 `Endpoints` .

`spring-boot-starter-webservices` 可以通过 `spring-boot-starter-webservices` 模块轻松访问.

可以分别为WSDL和XSD自动创建 `SimpleWsdl11Definition` 和 `SimpleXsdSchema`  beans.为此，请配置其位置，如以下示例所示：

```java
spring.webservices.wsdl-locations=classpath:/wsdl
```
