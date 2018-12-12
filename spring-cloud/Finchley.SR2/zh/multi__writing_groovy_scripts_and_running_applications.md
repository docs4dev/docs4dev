## 81.编写Groovy脚本并运行应用程序

Spring Cloud CLI支持大多数Spring Cloud声明性功能，例如 `@Enable*` 注释类.例如，这是一个功能齐全的Eureka服务器

**app.groovy.** 

```java
@EnableEurekaServer
class Eureka {}
```

您可以从命令行运行，如下所示

```java
$ spring run app.groovy
```

要包含其他依赖项，通常只需添加适当的依赖项即可启用特征的注释，例如，  `@EnableConfigServer` ， `@EnableOAuth2Sso` 或 `@EnableEurekaClient` .要手动包含依赖项，您可以使用带有特殊"Spring Boot"短样式工件坐标的 `@Grab` ，即仅使用工件ID（不需要组或版本信息），例如，设置客户端应用程序以在AMQP上收听来自Spring CLoud Bus的管理事件：

**app.groovy.** 

```java
@Grab('spring-cloud-starter-bus-amqp')
@RestController
class Service {
@RequestMapping('/')
def home() { [message: 'Hello'] }
}
```

