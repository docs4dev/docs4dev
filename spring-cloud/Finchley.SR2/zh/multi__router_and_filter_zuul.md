## 18.路由器和过滤器：Zuul

路由是微服务架构不可或缺的一部分.例如， `/` 可以映射到您的Web应用程序， `/api/users` 映射到用户服务， `/api/shop` 映射到商店服务. [Zuul](https://github.com/Netflix/zuul)是Netflix的基于JVM的路由器和服务器端负载均衡器.

[Netflix uses Zuul](http://www.slideshare.net/MikeyCohen1/edge-architecture-ieee-international-conference-on-cloud-engineering-32240146/27)对于以下内容：

- Authentication

- Insights

- 应力测试

- Canary Testing

- Dynamic路由

- Service迁移

- Load脱落

- Security

- Static响应处理

- Active /主动流量管理

Zuul的规则引擎允许规则和过滤器基本上以任何JVM语言编写，内置支持Java和Groovy.

> 配置属性 `zuul.max.host.connections` 已被两个新属性 `zuul.host.maxTotalConnections` 和 `zuul.host.maxPerRouteConnections` 替换，默认分别为200和20.

> 所有路径的默认Hystrix隔离模式（ `ExecutionIsolationStrategy` ）为 `SEMAPHORE` .如果首选隔离模式， `zuul.ribbonIsolationStrategy` 可以更改为 `THREAD` .

## 18.1如何包括Zuul

要在项目中包含Zuul，请使用组ID为 `org.springframework.cloud` 且工件ID为 `spring-cloud-starter-netflix-zuul` 的starter.有关使用当前Spring Cloud Release Train设置构建系统的详细信息，请参阅[Spring Cloud Project page](https://projects.spring.io/spring-cloud/).

## 18.2嵌入式Zuul反向代理

Spring Cloud创建了一个嵌入式Zuul代理，以简化UI应用程序想要对一个或多个后端服务进行代理调用的常见用例的开发.此功能对于用户界面代理其所需的后端服务非常有用，从而无需为所有后端独立管理CORS和身份验证问题.

要启用它，请使用 `@EnableZuulProxy` 注释Spring Boot主类.这样做会导致本地呼叫转发到适当的服务.按照惯例，ID为 `users` 的服务从位于 `/users` 的代理接收请求（前缀已剥离）.代理使用功能区来定位要通过发现转发的实例.所有请求都在[hystrix command](multi__router_and_filter_zuul.html#hystrix-fallbacks-for-routes)中执行，因此失败出现在Hystrix指标中.电路打开后，代理不会尝试联系该服务.

>  Zuul启动器不包含发现客户端，因此，对于基于服务ID的路由，您还需要在类路径中提供其中一个（Eureka是一种选择）.

要跳过自动添加的服务，请将 `zuul.ignored-services` 设置为服务ID模式列表.如果服务与忽略的模式匹配，但也包含在显式配置中路由映射，它是不带号的，如下例所示：

**application.yml.** 

```java
zuul:
ignoredServices: '*'
routes:
users: /myusers/**
```

在前面的示例中，将忽略所有服务， **except** 表示 `users` .

要扩充或更改代理路由，可以添加外部配置，如下所示：

**application.yml.** 

```java
zuul:
routes:
users: /myusers/**
```

前面的示例表示对 `/myusers` 的HTTP调用转发到 `users` 服务（例如 `/myusers/101` 被转发到 `/101` ）.

要对路由进行更细粒度的控制，可以单独指定路径和serviceId，如下所示：

**application.yml.** 

```java
zuul:
routes:
users:
path: /myusers/**
serviceId: users_service
```

前面的示例意味着对 `/myusers` 的HTTP调用将转发到 `users_service` 服务.路径必须具有可以指定为Ant风格模式的 `path` ，因此 `/myusers/*` 仅匹配一个级别，但 `/myusers/**` 分层匹配.

后端的位置可以指定为 `serviceId` （对于发现中的服务）或 `url` （对于物理位置），如以下示例所示：

**application.yml.** 

```java
zuul:
routes:
users:
path: /myusers/**
url: http://example.com/users_service
```

这些简单的url-routes不会作为 `HystrixCommand` 执行，也不会使用Ribbon对多个URL进行负载平衡.要实现这些目标，您可以使用静态服务器列表指定 `serviceId` ，如下所示：

**application.yml.** 

```java
zuul:
routes:
echo:
path: /myusers/**
serviceId: myusers-service
stripPrefix: true

hystrix:
command:
myusers-service:
execution:
isolation:
thread:
timeoutInMilliseconds: ...

myusers-service:
ribbon:
NIWSServerListClassName: com.netflix.loadbalancer.ConfigurationBasedServerList
listOfServers: http://example1.com,http://example2.com
ConnectTimeout: 1000
ReadTimeout: 3000
MaxTotalHttpConnections: 500
MaxConnectionsPerHost: 100
```

另一种方法是指定服务路由并为 `serviceId` 配置功能区客户端（这样做需要在功能区中禁用Eureka支持 - 请参阅[above for more information](multi_spring-cloud-ribbon.html#spring-cloud-ribbon-without-eureka)），如以下示例所示：

**application.yml.** 

```java
zuul:
routes:
users:
path: /myusers/**
serviceId: users

ribbon:
eureka:
enabled: false

users:
ribbon:
listOfServers: example.com,google.com
```

您可以使用 `regexmapper` 在 `serviceId` 和路由之间提供约定.它使用正则表达式命名组从 `serviceId` 中提取变量并将它们注入路由模式，如以下示例所示：

**ApplicationConfiguration.java.** 

```xml
@Bean
public PatternServiceRouteMapper serviceRouteMapper() {
return new PatternServiceRouteMapper(
"(?<name>^.+)-(?<version>v.+$)",
"${version}/${name}");
}
```

前面的示例意味着 `serviceId` 的 `myusers-v1` 被映射到路由 `/v1/myusers/**` .接受任何正则表达式，但所有命名组必须同时出现在 `servicePattern` 和 `routePattern` 中.如果 `servicePattern` 与 `serviceId` 不匹配，则使用默认行为.在前面的示例中， `serviceId` 的 `myusers` 映射到"/myusers/**"路由（未检测到版本）.默认情况下禁用此功能，仅适用于已发现的服务.

要为所有映射添加前缀，请将 `zuul.prefix` 设置为值，例如 `/api` .默认情况下，在转发请求之前，会从请求中删除代理前缀（您可以使用 `zuul.stripPrefix=false` 关闭此行为）.您还可以关闭从各个路由中剥离特定于服务的前缀，如以下示例所示：

**application.yml.** 

```java
zuul:
routes:
users:
path: /myusers/**
stripPrefix: false
```

>  `zuul.stripPrefix` 仅适用于 `zuul.prefix` 中设置的前缀.它对给定路径 `path` 中定义的前缀没有任何影响.

在前面的示例中，对 `/myusers/101` 的请求将转发到 `users` 服务上的 `/myusers/101` .

`zuul.routes` 条目实际上绑定到 `ZuulProperties` 类型的对象.如果查看该对象的属性，可以看到它还有一个 `retryable` 标志.将该标志设置为 `true` ，以使Ribbon客户端自动重试失败的请求.当您需要修改使用功能区客户端配置的重试操作的参数时，也可以将该标志设置为 `true` .

默认情况下， `X-Forwarded-Host` 标头会添加到转发的请求中.要将其关闭，请设置 `zuul.addProxyHeaders = false` .默认情况下，前缀路径被剥离，对后端的请求会选择 `X-Forwarded-Prefix` 标头（前面显示的示例中为 `/myusers` ）.

如果设置了默认路由（ `/` ），则具有 `@EnableZuulProxy` 的应用程序可以充当独立服务器.例如， `zuul.route.home: /` 会将所有流量（"/**"）路由到"home"服务.

如果需要更细粒度的忽略，则可以指定要忽略的特定模式.这些模式在路径定位过程开始时进行评估，这意味着前缀应包含在模式中以保证匹配.忽略的模式跨越所有服务并取代任何其他路由规范.以下示例显示如何创建忽略的模式：

**application.yml.** 

```java
zuul:
ignoredPatterns: /**/admin/**
routes:
users: /myusers/**
```

上面的示例表示所有调用（例如 `/myusers/101` ）都转发到 `users` 服务上的 `/101` .但是，包括 `/admin/` 在内的呼叫无法解决.

> 如果您需要路由保留其订单，则需要使用YAML文件，因为使用属性文件时排序会丢失.以下示例显示了这样的YAML文件：

**application.yml.** 

```java
zuul:
routes:
users:
path: /myusers/**
legacy:
path: /**
```

如果您要使用属性文件，则 `legacy` 路径可能最终位于 `users` 路径的前面，从而导致 `users` 路径无法访问.

## 18.3 Zuul Http客户端

Zuul使用的默认HTTP客户端现在由Apache HTTP Client支持，而不是不推荐使用的Ribbon  `RestClient` .要使用 `RestClient` 或 `okhttp3.OkHttpClient` ，请分别设置 `ribbon.restclient.enabled=true` 或 `ribbon.okhttp.enabled=true` .如果要自定义Apache HTTP客户端或OK HTTP客户端，请提供 `ClosableHttpClient` 或 `OkHttpClient` 类型的bean.

## 18.4Cookies和敏感Headers

您可以在同一系统中的服务之间共享标头，但您可能不希望敏感标头向下游泄漏到外部服务器.您可以在路由配置中指定忽略的标头列表.Cookie起着特殊的作用，因为它们在浏览器中具有良好定义的语义，并且它们始终被视为敏感.如果您的代理的消费者是浏览器，那么下游服务的cookie也会给用户带来问题，因为它们都混杂起来（所有下游服务看起来都来自同一个地方）.

如果您对服务的设计非常小心（例如，如果只有一个下游服务设置了cookie），您可以让它们从后端一直流到调用者.此外，如果您的代理设置了cookie并且所有后端服务都是同一系统的一部分，那么简单地共享它们就很自然（例如，使用Spring Session将它们链接到某个共享状态）.除此之外，由下游服务设置的任何cookie都可能对调用者没用，因此建议您（至少） `Set-Cookie` 和 `Cookie` 进入不属于您的域的路由的敏感标头.即使是属于您域名的路由，也要在让cookie和代理之间流动之前仔细考虑它的含义.

可以将敏感标头配置为每个路由的逗号分隔列表，如以下示例所示：

**application.yml.** 

```java
zuul:
routes:
users:
path: /myusers/**
sensitiveHeaders: Cookie,Set-Cookie,Authorization
url: https://downstream
```

> T这是 `sensitiveHeaders` 的默认值，因此除非您希望它不同，否则无需进行设置.这是Spring Cloud Netflix 1.1中的新功能（在1.0中，用户无法控制Headers，并且所有Cookie都在两个方向上流动）.

`sensitiveHeaders` 是黑名单，默认值不为空.因此，要使Zuul发送所有标头（ `ignored` 除外），必须将其明确设置为空列表.如果要将cookie或授权标头传递到后端，则必须这样做.以下示例显示如何使用 `sensitiveHeaders` ：

**application.yml.** 

```java
zuul:
routes:
users:
path: /myusers/**
sensitiveHeaders:
url: https://downstream
```

您还可以通过设置 `zuul.sensitiveHeaders` 来设置敏感标头.如果在路由上设置了 `sensitiveHeaders` ，它将覆盖全局 `sensitiveHeaders` 设置.

## 18.5忽略Headers

除路由敏感标头外，您还可以为与下游服务交互期间应丢弃的值（请求和响应）设置名为 `zuul.ignoredHeaders` 的全局值.默认情况下，如果Spring Security不在类路径中，则它们为空.否则，它们被初始化为一组众所周知的“安全”头文件（例如，涉及缓存），如Spring Security所指定的那样.在这种情况下的假设是下游服务也可能添加这些头，但我们想要代理的值.要在Spring Security位于类路径上时不丢弃这些众所周知的安全标头，可以将 `zuul.ignoreSecurityHeaders` 设置为 `false` .如果您在Spring Security中禁用了HTTP安全响应标头并希望下游服务提供的值，那么这样做会非常有用.

## 18.6管理endpoints

默认情况下，如果将 `@EnableZuulProxy` 与Spring Boot Actuator一起使用，则启用另外两个endpoints：

- Routes

- Filters

### 18.6.1路由endpoints

到 `/routes` 的路由endpoints的GET返回映射路由的列表：

**GET /routes.** 

```java
{
/stores/**: "http://localhost:8081"
}
```

可以通过将 `?format=details` 查询字符串添加到 `/routes` 来请求其他路径详细信息.这样做会产生以下输出：

**GET /routes/details.** 

```java
{
"/stores/**": {
"id": "stores",
"fullPath": "/stores/**",
"location": "http://localhost:8081",
"path": "/**",
"prefix": "/stores",
"retryable": false,
"customSensitiveHeaders": false,
"prefixStripped": true
}
}
```

`POST` 到 `/routes` 强制刷新现有路径（例如，当服务目录中有更改时）.您可以通过将 `endpoints.routes.enabled` 设置为 `false` 来禁用此endpoints.

> 路由应自动响应服务目录中的更改，但 `POST` 至 `/routes` 是强制更改立即发生的一种方法.

### 18.6.2过滤器endpoints

`/filters` 处的过滤器endpoints的 `GET` 按类型返回Zuul过滤器的映射.对于Map中的每种过滤器类型，您将获得该类型的所有过滤器及其详细信息的列表.

## 18.7扼杀模式和本地前锋

迁移现有应用程序或API时的一种常见模式是“扼杀”旧endpoints，慢慢用不同的实现替换它们. Zuul代理是一个有用的工具，因为您可以使用它来处理来自旧endpoints的客户端的所有流量，但将一些请求重定向到新的endpoints.

以下示例显示“strangle”方案的配置详细信息：

**application.yml.** 

```java
zuul:
routes:
first:
path: /first/**
url: http://first.example.com
second:
path: /second/**
url: forward:/second
third:
path: /third/**
url: forward:/3rd
legacy:
path: /**
url: http://legacy.example.com
```

在前面的示例中，我们扼杀了“遗留”应用程序，该应用程序映射到与其他模式之一不匹配的所有请求.  `/first/**` 中的路径已被提取到具有外部URL的新服务中.转发 `/second/**` 中的路径，以便可以在本地处理它们（例如，使用正常的Spring  `@RequestMapping` ）.  `/third/**` 中的路径也被转发但具有不同的前缀（ `/third/foo` 被转发到 `/3rd/foo` ）.

> 忽略的模式不会被完全忽略，它们只是不由代理处理（所以它们也在本地有效转发）.

## 18.8通过Zuul上传文件

如果使用 `@EnableZuulProxy` ，则可以使用代理路径进行上载文件，它应该工作，只要文件很小.对于大型文件，有一个替代路径绕过"/zuul/*"中的Spring  `DispatcherServlet` （以避免多部分处理）.换句话说，如果你有 `zuul.routes.customers=/customers/**` ，那么你可以 `POST` 大文件到 `/zuul/customers/*` . servlet路径通过 `zuul.servletPath` 外部化.如果代理路由引导您完成功能区负载平衡器，则极大文件也需要提升超时设置，如以下示例所示：

**application.yml.** 

```java
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds: 60000
ribbon:
ConnectTimeout: 3000
ReadTimeout: 60000
```

请注意，要使用大型文件进行流式处理，您需要在请求中使用分块编码（默认情况下某些浏览器不会这样做），如以下示例所示：

```java
$ curl -v -H "Transfer-Encoding: chunked" \
-F "[emailprotected]" localhost:9999/zuul/simple/file
```

## 18.9查询字符串编码

处理传入请求时，将对查询参数进行解码，以便它们可用于Zuul过滤器中的可能修改.然后对它们进行重新编码，在路由过滤器中重建后端请求.如果（例如）它使用Javascript的 `encodeURIComponent()` 方法编码，则结果可能与原始输入不同.虽然这在大多数情况下不会引起任何问题，但某些Web服务器可能会因复杂查询字符串的编码而变得挑剔.

要强制查询字符串的原始编码，可以将特殊标志传递给 `ZuulProperties` ，以便使用 `HttpServletRequest::getQueryString` 方法按原样获取查询字符串，如以下示例所示：

**application.yml.** 

```java
zuul:
forceOriginalQueryStringEncoding: true
```

> 此特殊标志仅适用于 `SimpleHostRoutingFilter` .此外，您无法使用 `RequestContext.getCurrentContext().setRequestQueryParams(someOverriddenParameters)` 轻松覆盖查询参数，因为查询字符串现在直接在原始 `HttpServletRequest` 上提取.

## 18.10 Plain Embedded Zuul

如果您使用 `@EnableZuulServer` （而不是 `@EnableZuulProxy` ），您还可以运行Zuul服务器而无需代理或有选择地切换代理平台的某些部分.您添加到 `ZuulFilter` 类型的应用程序的任何bean都会自动安装（与 `@EnableZuulProxy` 一样），但不会自动添加任何代理过滤器.

在这种情况下，仍然通过配置“zuul.routes.*”来指定进入Zuul服务器的路由，但是没有服务发现和代理.因此，忽略“serviceId”和“url”设置.以下示例将“/ api / **”中的所有路径映射到Zuul过滤器链：

**application.yml.** 

```java
zuul:
routes:
api: /api/**
```

## 18.11禁用Zuul过滤器

Zuul for Spring Cloud在代理和服务器模式下都默认启用了许多 `ZuulFilter`  beans.有关可以启用的过滤器列表，请参阅[the Zuul filters package](https://github.com/spring-cloud/spring-cloud-netflix/tree/master/spring-cloud-netflix-zuul/src/main/java/org/springframework/cloud/netflix/zuul/filters).如果要禁用其中一个，请设置 `zuul.<SimpleClassName>.<filterType>.disable=true` .按照惯例， `filters` 之后的包是Zuul过滤器类型.例如，要禁用 `org.springframework.cloud.netflix.zuul.filters.post.SendResponseFilter` ，请设置 `zuul.SendResponseFilter.post.disable=true` .

## 18.12为路线提供Hystrix后备

当Zuul中给定路径的电路跳闸时，您可以通过创建 `FallbackProvider` 类型的bean来提供回退响应.在此bean中，您需要指定回退所针对的路由ID，并提供 `ClientHttpResponse` 作为回退返回.以下示例显示了一个相对简单的 `FallbackProvider` 实现：

```java
class MyFallbackProvider implements FallbackProvider {

@Override
public String getRoute() {
return "customers";
}

@Override
public ClientHttpResponse fallbackResponse(String route, final Throwable cause) {
if (cause instanceof HystrixTimeoutException) {
return response(HttpStatus.GATEWAY_TIMEOUT);
} else {
return response(HttpStatus.INTERNAL_SERVER_ERROR);
}
}

private ClientHttpResponse response(final HttpStatus status) {
return new ClientHttpResponse() {
@Override
public HttpStatus getStatusCode() throws IOException {
return status;
}

@Override
public int getRawStatusCode() throws IOException {
return status.value();
}

@Override
public String getStatusText() throws IOException {
return status.getReasonPhrase();
}

@Override
public void close() {
}

@Override
public InputStream getBody() throws IOException {
return new ByteArrayInputStream("fallback".getBytes());
}

@Override
public HttpHeaders getHeaders() {
HttpHeaders headers = new HttpHeaders();
headers.setContentType(MediaType.APPLICATION_JSON);
return headers;
}
};
}
}
```

以下示例显示了上一个示例的路由配置可能如何显示：

```java
zuul:
routes:
customers: /customers/**
```

如果要为所有路由提供默认回退，可以创建 `FallbackProvider` 类型的bean，并使 `getRoute` 方法返回 `*` 或 `null` ，如以下示例所示：

```java
class MyFallbackProvider implements FallbackProvider {
@Override
public String getRoute() {
return "*";
}

@Override
public ClientHttpResponse fallbackResponse(String route, Throwable throwable) {
return new ClientHttpResponse() {
@Override
public HttpStatus getStatusCode() throws IOException {
return HttpStatus.OK;
}

@Override
public int getRawStatusCode() throws IOException {
return 200;
}

@Override
public String getStatusText() throws IOException {
return "OK";
}

@Override
public void close() {

}

@Override
public InputStream getBody() throws IOException {
return new ByteArrayInputStream("fallback".getBytes());
}

@Override
public HttpHeaders getHeaders() {
HttpHeaders headers = new HttpHeaders();
headers.setContentType(MediaType.APPLICATION_JSON);
return headers;
}
};
}
}
```

## 18.13 Zuul超时

如果要为通过Zuul代理的请求配置套接字超时和读取超时，则有两种选择，具体取决于您的配置：

- 如果Zuul使用服务发现，则需要使用 `ribbon.ReadTimeout` 和 `ribbon.SocketTimeout` 功能区属性配置这些超时.

如果通过指定URL配置了Zuul路由，则需要使用 `zuul.host.connect-timeout-millis` 和 `zuul.host.socket-timeout-millis` .

## 18.14重写Location标头

如果Zuul面向Web应用程序，则当Web应用程序重定向HTTP状态代码 `3XX` 时，您可能需要重新编写 `Location` 标头.否则，浏览器会重定向到Web应用程序的URL而不是Zuul URL.您可以配置 `LocationRewriteFilter`  Zuul过滤器以将 `Location` 标头重写为Zuul的URL.它还会添加剥离的全局和路由特定前缀.以下示例使用Spring配置文件添加过滤器：

```java
import org.springframework.cloud.netflix.zuul.filters.post.LocationRewriteFilter;
...

@Configuration
@EnableZuulProxy
public class ZuulConfig {
@Bean
public LocationRewriteFilter locationRewriteFilter() {
return new LocationRewriteFilter();
}
}
```

> 请仔细使用此过滤器.过滤器作用于ALL  `3XX` 响应代码的 `Location` 标头，这可能不适用于所有情况，例如将用户重定向到外部URL时.

## 18.15指标

对于路由请求时可能发生的任何故障，Zuul将在Actuator指标endpoints下提供指标.点击 `/actuator/metrics` 即可查看这些指标.度量标准的名称格式为 `ZUUL::EXCEPTION:errorCause:statusCode` .

## 18.16 Zuul开发人员指南

有关Zuul如何工作的一般概述，请参阅[the Zuul Wiki](https://github.com/Netflix/zuul/wiki/How-it-Works).

### 18.16.1 Zuul Servlet

Zuul是作为Servlet实现的.对于一般情况，Zuul嵌入到Spring Dispatch机制中.这让Spring MVC可以控制路由.在这种情况下，Zuul缓冲请求.如果有的话需要在没有缓冲请求的情况下通过Zuul（例如，对于大型文件上传），Servlet也安装在Spring Dispatcher之外.默认情况下，servlet的地址为 `/zuul` .可以使用 `zuul.servlet-path` 属性更改此路径.

### 18.16.2 Zuul RequestContext

要在过滤器之间传递信息，Zuul使用[RequestContext](https://github.com/Netflix/zuul/blob/1.x/zuul-core/src/main/java/com/netflix/zuul/context/RequestContext.java).其数据保存在特定于每个请求的 `ThreadLocal` 中.有关在何处路由请求，错误以及实际 `HttpServletRequest` 和 `HttpServletResponse` 的信息存储在那里.  `RequestContext` 扩展 `ConcurrentHashMap` ，因此任何内容都可以存储在上下文中. [FilterConstants](https://github.com/spring-cloud/spring-cloud-netflix/blob/master/spring-cloud-netflix-core/src/main/java/org/springframework/cloud/netflix/zuul/filters/support/FilterConstants.java)包含Spring Cloud Netflix安装的过滤器使用的密钥（更多关于这些[later](multi__router_and_filter_zuul.html#zuul-developer-guide-enable-filters)）.

### 18.16.3 @EnableZuulProxy与@EnableZuulServer

Spring Cloud Netflix安装了许多过滤器，具体取决于使用哪个注释来启用Zuul.  `@EnableZuulProxy` 是 `@EnableZuulServer` 的超集.换句话说， `@EnableZuulProxy` 包含 `@EnableZuulServer` 安装的所有过滤器. “代理”中的其他过滤器启用路由功能.如果你想要一个“空白”Zuul，你应该使用 `@EnableZuulServer` .

### 18.16.4 @EnableZuulServer过滤器

`@EnableZuulServer` 创建 `SimpleRouteLocator` ，从Spring Boot配置文件加载路由定义.

安装了以下过滤器（与普通的Spring Bean一样）：

- Pre过滤器：

-  `ServletDetectionFilter` ：检测请求是否通过Spring Dispatcher.设置一个键为 `FilterConstants.IS_DISPATCHER_SERVLET_REQUEST_KEY` 的布尔值.

-  `FormBodyWrapperFilter` ：解析表单数据并为下游请求重新编码.

-  `DebugFilter` ：如果设置了 `debug` 请求参数，则将 `RequestContext.setDebugRouting()` 和 `RequestContext.setDebugRequest()` 设置为 `true` . *路线过滤器：

-  `SendForwardFilter` ：使用Servlet  `RequestDispatcher` 转发请求.转发位置存储在 `RequestContext` 属性 `FilterConstants.FORWARD_TO_KEY` 中.这对于转发到当前应用程序中的endpoints非常有用.

- Post过滤器：

-  `SendResponseFilter` ：将代理请求的响应写入当前响应.

- Error过滤器：

-  `SendErrorFilter` ：如果 `RequestContext.getThrowable()` 不为空，则转发到 `/error` （默认情况下）.您可以通过设置 `error.path` 属性来更改默认转发路径（ `/error` ）.

### 18.16.5 @EnableZuulProxy过滤器

创建 `DiscoveryClientRouteLocator` ，从 `DiscoveryClient` （例如Eureka）以及属性加载路径定义.为 `DiscoveryClient` 中的每个 `serviceId` 创建一个路径.添加新服务后，将刷新路由.

除了前面描述的过滤器之外，还安装了以下过滤器（与普通的Spring Bean一样）：

- Pre过滤器：

-  `PreDecorationFilter` ：根据提供的 `RouteLocator` 确定路由的位置和方式.它还为下游请求设置各种与代理相关的标头.

- Route过滤器：

-  `RibbonRoutingFilter` ：使用Ribbon，Hystrix和可插入的HTTP客户端发送请求.服务ID位于 `RequestContext` 属性 `FilterConstants.SERVICE_ID_KEY` 中.此过滤器可以使用不同的HTTP客户端：

- Apache  `HttpClient` ：默认客户端.

- Squareup  `OkHttpClient`  v3：通过在类路径上设置 `com.squareup.okhttp3:okhttp` 库并设置 `ribbon.okhttp.enabled=true` 来启用.

- Netflix功能区HTTP客户端：通过设置 `ribbon.restclient.enabled=true` 启用.此客户端具有限制，包括它不支持PATCH方法，但它也具有内置重试.

-  `SimpleHostRoutingFilter` ：通过Apache HttpClient向预定URL发送请求. URL位于 `RequestContext.getRouteHost()` 中.

### 18.16.6自定义Zuul过滤器示例

下面的大多数"How to Write"示例都包含在[Sample Zuul Filters](https://github.com/spring-cloud-samples/sample-zuul-filters)项目中.还有一些操作该存储库中的请求或响应主体的示例.

本节包括以下示例：

- [the section called “How to Write a Pre Filter”](multi__router_and_filter_zuul.html#zuul-developer-guide-sample-pre-filter)

- [the section called “How to Write a Route Filter”](multi__router_and_filter_zuul.html#zuul-developer-guide-sample-route-filter)

- [the section called “How to Write a Post Filter”](multi__router_and_filter_zuul.html#zuul-developer-guide-sample-post-filter)

#### 如何编写预过滤器

预过滤器在 `RequestContext` 中设置数据，以用于下游过滤器.主要用例是设置路由过滤器所需的信息.以下示例显示了Zuul预过滤器：

```java
public class QueryParamPreFilter extends ZuulFilter {
	@Override
	public int filterOrder() {
		return PRE_DECORATION_FILTER_ORDER - 1; // run before PreDecoration
	}

	@Override
	public String filterType() {
		return PRE_TYPE;
	}

	@Override
	public boolean shouldFilter() {
		RequestContext ctx = RequestContext.getCurrentContext();
		return !ctx.containsKey(FORWARD_TO_KEY) // a filter has already forwarded
				&& !ctx.containsKey(SERVICE_ID_KEY); // a filter has already determined serviceId
	}
@Override
public Object run() {
RequestContext ctx = RequestContext.getCurrentContext();
		HttpServletRequest request = ctx.getRequest();
		if (request.getParameter("sample") != null) {
		    // put the serviceId in `RequestContext`
		ctx.put(SERVICE_ID_KEY, request.getParameter("foo"));
	}
return null;
}
}
```

前面的过滤器从 `sample`  request参数填充 `SERVICE_ID_KEY` .在实践中，您不应该进行这种直接映射.相反，应该从 `sample` 的值中查找服务ID.

现在 `SERVICE_ID_KEY` 已填充， `PreDecorationFilter` 不运行且 `RibbonRoutingFilter` 运行.

> 如果要路由到完整URL，请改为调用 `ctx.setRouteHost(url)` .

要修改路由过滤器转发的路径，请设置 `REQUEST_URI_KEY` .

#### 如何编写路由过滤器

路由过滤器在预过滤器之后运行并向其他服务发出请求.这里的大部分工作是将请求和响应数据转换为客户端所需的模型.以下示例显示了Zuul路由过滤器：

```java
public class OkHttpRoutingFilter extends ZuulFilter {
	@Autowired
	private ProxyRequestHelper helper;

	@Override
	public String filterType() {
		return ROUTE_TYPE;
	}

	@Override
	public int filterOrder() {
		return SIMPLE_HOST_ROUTING_FILTER_ORDER - 1;
	}

	@Override
	public boolean shouldFilter() {
		return RequestContext.getCurrentContext().getRouteHost() != null
				&& RequestContext.getCurrentContext().sendZuulResponse();
	}

@Override
public Object run() {
		OkHttpClient httpClient = new OkHttpClient.Builder()
				// customize
				.build();

		RequestContext context = RequestContext.getCurrentContext();
		HttpServletRequest request = context.getRequest();

		String method = request.getMethod();

		String uri = this.helper.buildZuulRequestURI(request);

		Headers.Builder headers = new Headers.Builder();
		Enumeration<String> headerNames = request.getHeaderNames();
		while (headerNames.hasMoreElements()) {
			String name = headerNames.nextElement();
			Enumeration<String> values = request.getHeaders(name);

			while (values.hasMoreElements()) {
				String value = values.nextElement();
				headers.add(name, value);
			}
		}

		InputStream inputStream = request.getInputStream();

		RequestBody requestBody = null;
		if (inputStream != null && HttpMethod.permitsRequestBody(method)) {
			MediaType mediaType = null;
			if (headers.get("Content-Type") != null) {
				mediaType = MediaType.parse(headers.get("Content-Type"));
			}
			requestBody = RequestBody.create(mediaType, StreamUtils.copyToByteArray(inputStream));
		}

		Request.Builder builder = new Request.Builder()
				.headers(headers.build())
				.url(uri)
				.method(method, requestBody);

		Response response = httpClient.newCall(builder.build()).execute();

		LinkedMultiValueMap<String, String> responseHeaders = new LinkedMultiValueMap<>();

		for (Map.Entry<String, List<String>> entry : response.headers().toMultimap().entrySet()) {
			responseHeaders.put(entry.getKey(), entry.getValue());
		}

		this.helper.setResponse(response.code(), response.body().byteStream(),
				responseHeaders);
		context.setRouteHost(null); // prevent SimpleHostRoutingFilter from running
		return null;
}
}
```

前面的过滤器将Servlet请求信息转换为OkHttp3请求信息，执行HTTP请求，并将OkHttp3响应信息转换为Servlet响应.

#### 如何编写后置过滤器

后置过滤器通常会操纵响应.以下过滤器添加随机 `UUID` 作为 `X-Sample` 标头：

```java
public class AddResponseHeaderFilter extends ZuulFilter {
	@Override
	public String filterType() {
		return POST_TYPE;
	}

	@Override
	public int filterOrder() {
		return SEND_RESPONSE_FILTER_ORDER - 1;
	}

	@Override
	public boolean shouldFilter() {
		return true;
	}

	@Override
	public Object run() {
		RequestContext context = RequestContext.getCurrentContext();
	HttpServletResponse servletResponse = context.getResponse();
		servletResponse.addHeader("X-Sample", UUID.randomUUID().toString());
		return null;
	}
}
```

> 其他操作，例如转换响应体，更加复杂和计算密集.

### 18.16.7 Zuul错误如何工作

如果是例外在Zuul过滤器生命周期的任何部分抛出，执行错误过滤器.仅当 `RequestContext.getThrowable()` 不是 `null` 时，才会运行 `SendErrorFilter` .然后，它在请求中设置特定的 `javax.servlet.error.*` 属性，并将请求转发到Spring Boot错误页面.

### 18.16.8 Zuul Eager应用程序上下文加载

Zuul内部使用Ribbon来调用远程URL.默认情况下，Spring Cloud在第一次调用时会延迟加载Ribbon客户端.可以使用以下配置更改Zuul的此行为，这会导致在应用程序启动时急切加载与子功能区相关的应用程序上下文.以下示例显示如何启用预先加载：

**application.yml.** 

```java
zuul:
ribbon:
eager-load:
enabled: true
```

