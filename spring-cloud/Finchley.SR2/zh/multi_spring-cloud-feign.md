## 22.声明性REST客户端：Feign

[Feign](https://github.com/Netflix/feign)是声明性Web服务客户端.它使编写Web服务客户端变得更容易.要使用Feign，请创建一个界面并对其进行注释.它具有可插入的注释支持，包括Feign注释和JAX-RS注释. Feign还支持可插拔编码器和解码器. Spring Cloud增加了对Spring MVC注释的支持，并使用了Spring Web中默认使用的相同 `HttpMessageConverters` . Spring Cloud集成了Ribbon和Eureka，可在使用Feign时提供负载均衡的http客户端.

## 22.1如何包含假设

要在项目中包含Feign，请使用包含组 `org.springframework.cloud` 和工件ID  `spring-cloud-starter-openfeign` 的启动器.有关使用当前Spring Cloud Release Train设置构建系统的详细信息，请参阅[Spring Cloud Project page](https://projects.spring.io/spring-cloud/).

spring启动应用示例

```java
@SpringBootApplication
@EnableFeignClients
public class Application {

public static void main(String[] args) {
SpringApplication.run(Application.class, args);
}

}
```

**StoreClient.java.** 

```java
@FeignClient("stores")
public interface StoreClient {
@RequestMapping(method = RequestMethod.GET, value = "/stores")
List<Store> getStores();

@RequestMapping(method = RequestMethod.POST, value = "/stores/{storeId}", consumes = "application/json")
Store update(@PathVariable("storeId") Long storeId, Store store);
}
```

在 `@FeignClient` 注释中，String值（上面的"stores"）是一个任意客户端名称，用于创建功能区负载均衡器（请参阅[below for details of Ribbon support](multi_spring-cloud-ribbon.html)）.您还可以使用 `url` 属性指定URL（绝对值或仅指定主机名）.应用程序上下文中bean的名称是接口的完全限定名称.要指定自己的别名值，可以使用 `@FeignClient` 注释的 `qualifier` 值.

上面的Ribbon客户端将要发现"stores"服务的物理地址.如果您的应用程序是Eureka客户端，那么它将解析Eureka服务注册表中的服务.如果您不想使用Eureka，只需在外部配置中配置服务器列表（请参阅[above for example](multi_spring-cloud-ribbon.html#spring-cloud-ribbon-without-eureka)）.

## 22.2覆盖假设默认值

Spring Cloud的Feign支持的核心概念是指定客户端的概念.每个假装客户端都是一组组件的一部分，这些组件一起工作以按需联系远程服务器，并且整体具有您使用 `@FeignClient` 注释作为应用程序开发人员提供的名称. Spring Cloud使用 `FeignClientsConfiguration` 为每个命名客户端创建一个新的集合 `ApplicationContext` .这包含（除其他外） `feign.Decoder` ， `feign.Encoder` 和 `feign.Contract` .

Spring Cloud允许您通过使用 `@FeignClient` 声明其他配置（在 `FeignClientsConfiguration` 之上）来完全控制假装客户端.例：

```java
@FeignClient(name = "stores", configuration = FooConfiguration.class)
public interface StoreClient {
//..
}
```

在这种情况下，客户端由 `FeignClientsConfiguration` 中的组件和 `FooConfiguration` 中的任何组件组成（后者将覆盖前者）.

>  `FooConfiguration` 不需要使用 `@Configuration` 进行注释.但是，如果是，则注意将其从任何包含此配置的_11611中排除，因为它将在指定时成为 `feign.Decoder` ，_  `feign.Encoder` ， `feign.Contract` 等的默认源.这可以通过将其放在任何 `@ComponentScan` 或 `@SpringBootApplication` 的单独的非重叠包中来避免，或者可以在 `@ComponentScan` 中明确排除.

> 现在不推荐使用 `serviceId` 属性，而使用 `name` 属性.

> 以前，使用 `url` 属性，不需要 `name` 属性.现在需要使用 `name` .

`name` 和 `url` 属性支持占位符.

```java
@FeignClient(name = "${feign.name}", url = "${feign.url}")
public interface StoreClient {
//..
}
```

Spring Cloud Netflix默认为feign提供以下bean（ `BeanType`  beanName： `ClassName` ）：

-  `Decoder`  feignDecoder： `ResponseEntityDecoder` （包装 `SpringDecoder` ）

-  `Encoder`  feignEncoder： `SpringEncoder` 

-  `Logger`  feignLogger： `Slf4jLogger` 

-  `Contract`  feignContract： `SpringMvcContract` 

-  `Feign.Builder`  feignBuilder： `HystrixFeign.Builder` 

-  `Client`  feignClient：如果启用了功能区，则为 `LoadBalancerFeignClient` ，否则使用默认的假设客户端.

可以通过分别将 `feign.okhttp.enabled` 或 `feign.httpclient.enabled` 设置为 `true` 并将它们放在类路径上来使用OkHttpClient和ApacheHttpClient假装客户端.您可以通过在使用Apache时提供 `ClosableHttpClient` 的bean或使用OK HTTP时 `OkHttpClient` 来自定义所使用的HTTP客户端.

对于feign，Spring Cloud Netflix默认不提供以下bean，但仍然从应用程序上下文中查找这些类型的bean以创建feign客户端：

-  `Logger.Level` 

-  `Retryer` 

-  `ErrorDecoder` 

-  `Request.Options` 

-  `Collection<RequestInterceptor>` 

-  `SetterFactory` 

创建其中一种类型的bean并将其置于 `@FeignClient` 配置中（例如 `FooConfiguration` 上面）允许你覆盖所描述的每一个bean.例：

```java
@Configuration
public class FooConfiguration {
@Bean
public Contract feignContract() {
return new feign.Contract.Default();
}

@Bean
public BasicAuthRequestInterceptor basicAuthRequestInterceptor() {
return new BasicAuthRequestInterceptor("user", "password");
}
}
```

这将 `SpringMvcContract` 替换为 `feign.Contract.Default` ，并将 `RequestInterceptor` 添加到 `RequestInterceptor` 的集合中.

`@FeignClient` 也可以使用配置属性进行配置.

application.yml

```java
feign:
client:
config:
feignName:
connectTimeout: 5000
readTimeout: 5000
loggerLevel: full
errorDecoder: com.example.SimpleErrorDecoder
retryer: com.example.SimpleRetryer
requestInterceptors:
- com.example.FooRequestInterceptor
- com.example.BarRequestInterceptor
decode404: false
encoder: com.example.SimpleEncoder
decoder: com.example.SimpleDecoder
contract: com.example.SimpleContract
```

可以以与上述类似的方式在 `@EnableFeignClients` 属性 `defaultConfiguration` 中指定默认配置.不同之处在于此配置将适用于所有假装客户端.

如果您更喜欢使用配置属性来配置所有 `@FeignClient` ，则可以使用 `default`  feign name创建配置属性.

application.yml

```java
feign:
client:
config:
default:
connectTimeout: 5000
readTimeout: 5000
loggerLevel: basic
```

如果我们同时创建 `@Configuration`  bean和配置属性，配置属性将获胜.它将覆盖 `@Configuration` 值.但是，如果要将优先级更改为 `@Configuration` ，可以将 `feign.client.default-to-properties` 更改为 `false` .

> 如果需要在 `RequestInterceptor`s you will need to either set the thread isolation strategy for Hystrix to `SEMAPHORE` 中使用 `ThreadLocal` 绑定变量或在Feign中禁用Hystrix.

application.yml

```java
# To disable Hystrix in Feign
feign:
hystrix:
enabled: false

# To set thread isolation to SEMAPHORE
hystrix:
command:
default:
execution:
isolation:
strategy: SEMAPHORE
```

## 22.3手动创建Feign客户端

在某些情况下，可能需要以使用上述方法无法实现的方式自定义Feign客户端.在这种情况下，您可以使用[Feign Builder API](https://github.com/OpenFeign/feign/#basics)创建客户端.下面是一个示例，它创建两个具有相同接口的Feign客户端，但使用单独的请求拦截器配置每个客户端.

```java
@Import(FeignClientsConfiguration.class)
class FooController {

	private FooClient fooClient;

	private FooClient adminClient;

	@Autowired
	public FooController(Decoder decoder, Encoder encoder, Client client, Contract contract) {
		this.fooClient = Feign.builder().client(client)
				.encoder(encoder)
				.decoder(decoder)
				.contract(contract)
				.requestInterceptor(new BasicAuthRequestInterceptor("user", "user"))
				.target(FooClient.class, "http://PROD-SVC");

		this.adminClient = Feign.builder().client(client)
				.encoder(encoder)
				.decoder(decoder)
				.contract(contract)
				.requestInterceptor(new BasicAuthRequestInterceptor("admin", "admin"))
				.target(FooClient.class, "http://PROD-SVC");
}
}
```

> 在上面的示例中， `FeignClientsConfiguration.class` 是Spring Cloud Netflix提供的默认配置.

>  `PROD-SVC` 是客户端将向其发出请求的服务的名称.

> Feign  `Contract` 对象定义了哪些注释和值在接口上有效.自动装配的 `Contract`  bean提供对SpringMVC注释的支持，而不是默认的Feign本机注释.

## 22.4 Feign Hystrix支持

如果Hystrix在类路径上并且 `feign.hystrix.enabled=true` ，则Feign将使用断路器包装所有方法.还可以返回 `com.netflix.hystrix.HystrixCommand` .这允许您使用反应模式（调用 `.toObservable()` 或 `.observe()` 或异步使用（调用 `.queue()` ）.

要在每个客户端的基础上禁用Hystrix支持，请创建一个带有"prototype"范围的vanilla  `Feign.Builder` ，例如：

```java
@Configuration
public class FooConfiguration {
	@Bean
	@Scope("prototype")
	public Feign.Builder feignBuilder() {
		return Feign.builder();
	}
}
```

_0015.在Spring Cloud Dalston发布之前，如果Hystrix在类路径上，Feign会默认将所有方法包装在断路器中.在Spring Cloud Dalston中更改了此默认行为，转而采用了选择加入方法.

## 22.5 Feign Hystrix后退

Hystrix支持回退的概念：在电路打开或出现错误时执行的默认代码路径.要为给定的 `@FeignClient` 启用回退，请将 `fallback` 属性设置为实现回退的类名.您还需要将实现声明为Spring bean.

```java
@FeignClient(name = "hello", fallback = HystrixClientFallback.class)
protected interface HystrixClient {
@RequestMapping(method = RequestMethod.GET, value = "/hello")
Hello iFailSometimes();
}

static class HystrixClientFallback implements HystrixClient {
@Override
public Hello iFailSometimes() {
return new Hello("fallback");
}
}
```

如果需要访问导致回退触发器的原因，可以使用 `@FeignClient` 中的 `fallbackFactory` 属性.

```java
@FeignClient(name = "hello", fallbackFactory = HystrixClientFallbackFactory.class)
protected interface HystrixClient {
	@RequestMapping(method = RequestMethod.GET, value = "/hello")
	Hello iFailSometimes();
}

@Component
static class HystrixClientFallbackFactory implements FallbackFactory<HystrixClient> {
	@Override
	public HystrixClient create(Throwable cause) {
		return new HystrixClient() {
			@Override
			public Hello iFailSometimes() {
				return new Hello("fallback; reason was: " + cause.getMessage());
			}
		};
	}
}
```

> 在Feign中实施回退以及Hystrix回退的工作方式存在限制.返回 `com.netflix.hystrix.HystrixCommand` 和 `rx.Observable` 的方法目前不支持回退.

## 22.6 Feign和@Primary

使用Feign with Hystrix后备时， `ApplicationContext` 中有多个相同类型的bean.这将导致 `@Autowired` 无法正常工作，因为没有一个bean或一个标记为primary的bean.为了解决这个问题，Spring Cloud Netflix将所有Feign实例标记为 `@Primary` ，因此Spring Framework将知道要注入哪个bean.在某些情况下，这可能并不理想.要关闭此行为，请将 `@FeignClient` 的 `primary` 属性设置为false.

```java
@FeignClient(name = "hello", primary = false)
public interface HelloClient {
	// methods here
}
```

## 22.7 Feign继承支持

Feign通过单继承接口支持样板apis.这允许将常见操作分组为方便的基本接口.

**UserService.java.** 

```java
public interface UserService {

@RequestMapping(method = RequestMethod.GET, value ="/users/{id}")
User getUser(@PathVariable("id") long id);
}
```

**UserResource.java.** 

```java
@RestController
public class UserResource implements UserService {

}
```

**UserClient.java.** 

```java
package project.user;

@FeignClient("users")
public interface UserClient extends UserService {

}
```

> 通常不建议在服务器和客户端之间共享接口.它引入了紧耦合，并且实际上也不能以当前形式使用Spring MVC（方法参数映射不是继承的）.

## 22.8假设请求/响应压缩

您可以考虑为您的Feign请求启用请求或响应GZIP压缩.您可以通过启用以下属性之一来执行此操作：

```java
feign.compression.request.enabled=true
feign.compression.response.enabled=true
```

假设请求压缩为您提供类似于您为Web服务器设置的设置：

```java
feign.compression.request.enabled=true
feign.compression.request.mime-types=text/xml,application/xml,application/json
feign.compression.request.min-request-size=2048
```

通过这些属性，您可以选择压缩介质类型和最小请求阈值长度.

## 22.9记录日志

为每个创建的Feign客户端创建一个Logger.默认情况下，Logger的名称是用于创建Feign客户端的接口的完整类名. Feign日志记录仅响应 `DEBUG` 级别.

**application.yml.** 

```java
logging.level.project.user.UserClient: DEBUG
```

您可以为每个客户端配置的 `Logger.Level` 对象告诉Feign要记录多少.选择是：

-  `NONE` ，无记录（ **DEFAULT** ）.

-  `BASIC` ，仅记录请求方法和URL以及响应状态代码和执行时间.

-  `HEADERS` ，记录基本信息以及请求和响应标头.

-  `FULL` ，记录Headers，请求和响应的正文和元数据.

例如，以下将 `Logger.Level` 设置为 `FULL` ：

```java
@Configuration
public class FooConfiguration {
@Bean
Logger.Level feignLoggerLevel() {
return Logger.Level.FULL;
}
}
```

