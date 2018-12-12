## 47. Web Services

Spring Boot provides Web Services auto-configuration so that all you must do is define your  `Endpoints` .

The [Spring Web Services features](https://docs.spring.io/spring-ws/docs/3.0.4.RELEASE/reference/) can be easily accessed with the  `spring-boot-starter-webservices`  module.

`SimpleWsdl11Definition`  and  `SimpleXsdSchema`  beans can be automatically created for your WSDLs and XSDs respectively. To do so, configure their location, as shown in the following example:

```java
spring.webservices.wsdl-locations=classpath:/wsdl
```
