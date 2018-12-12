## 13.断路器：Hystrix客户端

Netflix创建了一个名为[Hystrix](https://github.com/Netflix/Hystrix)的库，它实现了[circuit breaker pattern](http://martinfowler.com/bliki/CircuitBreaker.html).在微服务架构中，通常有多层服务调用，如以下示例所示：

**Figure 13.1. Microservice Graph** 

![Hystrix](https://www.docs4dev.com/images/b6ee86ca-c9b2-4a17-8265-d7481954a595.png)

较低级别的服务中的服务故障可能导致级联故障一直到用户.当对特定服务的调用超过 `circuitBreaker.requestVolumeThreshold` （默认值：20个请求）且失败百分比大于 `circuitBreaker.errorThresholdPercentage` （默认值：> 50％）在由 `metrics.rollingStats.timeInMilliseconds` （默认值：10秒）定义的滚动窗口中时，电路将打开并且呼叫为没有.在出现错误和开路的情况下，开发人员可以提供回退.

**Figure 13.2. Hystrix fallback prevents cascading failures** 

![HystrixFallback](https://www.docs4dev.com/images/f4971d8c-8fce-45ad-a951-4b09ccef2d8b.png)

开放式电路可以阻止级联故障，并且可以让不知所措或失败的服务有时间恢复.回退可以是另一个受Hystrix保护的调用，静态数据或合理的空值.可以链接回退，以便第一个回退进行一些其他业务调用，这反过来又回到静态数据.

## 13.1如何包含Hystrix

要在项目中包含Hystrix，请使用组ID为 `org.springframework.cloud` 且工件ID为 `spring-cloud-starter-netflix-hystrix` 的starter.有关使用当前Spring Cloud Release Train设置构建系统的详细信息，请参阅[Spring Cloud Project page](https://projects.spring.io/spring-cloud/).

以下示例显示了具有Hystrix断路器的最小Eureka服务器：

```java
@SpringBootApplication
@EnableCircuitBreaker
public class Application {

public static void main(String[] args) {
new SpringApplicationBuilder(Application.class).web(true).run(args);
}

}

@Component
public class StoreIntegration {

@HystrixCommand(fallbackMethod = "defaultStores")
public Object getStores(Map<String, Object> parameters) {
//do stuff that might fail
}

public Object defaultStores(Map<String, Object> parameters) {
return /* something useful */;
}
}
```

`@HystrixCommand` 由名为[“javanica”](https://github.com/Netflix/Hystrix/tree/master/hystrix-contrib/hystrix-javanica)的Netflix contrib库提供. Spring Cloud在连接到Hystrix断路器的代理中自动包装带有该注释的Spring bean.断路器计算何时打开和关闭电路以及在发生故障时应采取的措施.

要配置 `@HystrixCommand` ，可以将 `commandProperties` 属性与 `@HystrixProperty` 注释列表一起使用.有关详细信息，请参阅[here](https://github.com/Netflix/Hystrix/tree/master/hystrix-contrib/hystrix-javanica#configuration).有关可用属性的详细信息，请参阅[Hystrix wiki](https://github.com/Netflix/Hystrix/wiki/Configuration).

## 13.2传播安全上下文或使用Spring Scopes

如果您希望某些线程本地上下文传播到 `@HystrixCommand` ，则默认声明不起作用，因为它在线程池中执行该命令（如果超时）.您可以通过配置或直接在注释中切换Hystrix以使用与调用者相同的线程，方法是要求它使用不同的“隔离策略”.以下示例演示如何在注释中设置线程：

```java
@HystrixCommand(fallbackMethod = "stubMyService",
commandProperties = {
@HystrixProperty(name="execution.isolation.strategy", value="SEMAPHORE")
}
)
...
```

如果您使用 `@SessionScope` 同样适用或 `@RequestScope` .如果遇到运行时异常，表示无法找到作用域上下文，则需要使用相同的线程.

您还可以选择将 `hystrix.shareSecurityContext` 属性设置为 `true` .这样做会自动配置Hystrix并发策略插件挂钩，将 `SecurityContext` 从主线程传输到Hystrix命令使用的线程. Hystrix不会注册多个Hystrix并发策略，因此通过将自己的 `HystrixConcurrencyStrategy` 声明为Spring bean，可以使用扩展机制. Spring Cloud在Spring上下文中查找您的实现并将其包装在自己的插件中.

## 13.3Health指标

连接断路器的状态也在调用应用程序的 `/health` endpoints中公开，如以下示例所示：

```java
{
"hystrix": {
"openCircuitBreakers": [
"StoreIntegration::getStoresByLocationLink"
],
"status": "CIRCUIT_OPEN"
},
"status": "UP"
}
```

## 13.4 Hystrix度量标准流

要启用Hystrix度量标准流，请在 `spring-boot-starter-actuator` 上包含依赖项并设置 `management.endpoints.web.exposure.include: hystrix.stream` .这样做会将 `/actuator/hystrix.stream` 公开为管理endpoints，如以下示例所示：

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

