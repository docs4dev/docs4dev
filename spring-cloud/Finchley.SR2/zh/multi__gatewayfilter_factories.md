## 115. GatewayFilter工厂

路由过滤器允许以某种方式修改传入的HTTP请求或传出的HTTP响应.路径过滤器的范围限定为特定路径. Spring Cloud Gateway包含许多内置的GatewayFilter工厂.

注意有关如何使用以下任何过滤器的更多详细示例，请查看[unit tests](https://github.com/spring-cloud/spring-cloud-gateway/tree/master/spring-cloud-gateway-core/src/test/java/org/springframework/cloud/gateway/filter/factory).

## 115.1 AddRequestHeader GatewayFilter Factory

AddRequestHeader GatewayFilter Factory采用名称和值参数.

**application.yml.** 

```java
spring:
cloud:
gateway:
routes:
- id: add_request_header_route
uri: http://example.org
filters:
- AddRequestHeader=X-Request-Foo, Bar
```

这将为所有匹配请求的下游请求标头添加 `X-Request-Foo:Bar` 标头.

## 115.2 AddRequestParameter GatewayFilter Factory

AddRequestParameter GatewayFilter Factory采用名称和值参数.

**application.yml.** 

```java
spring:
cloud:
gateway:
routes:
- id: add_request_parameter_route
uri: http://example.org
filters:
- AddRequestParameter=foo, bar
```

这会将 `foo=bar` 添加到下游请求的所有匹配请求的查询字符串中.

## 115.3 AddResponseHeader GatewayFilter Factory

AddResponseHeader GatewayFilter Factory采用名称和值参数.

**application.yml.** 

```java
spring:
cloud:
gateway:
routes:
- id: add_request_header_route
uri: http://example.org
filters:
- AddResponseHeader=X-Response-Foo, Bar
```

这会将 `X-Response-Foo:Bar` 标头添加到所有匹配请求的下游响应标头中.

## 115.4 Hystrix GatewayFilter Factory

[Hystrix](https://github.com/Netflix/Hystrix)是来自Netflix的库，用于实现[circuit breaker pattern](https://martinfowler.com/bliki/CircuitBreaker.html). Hystrix GatewayFilter允许您将断路器引入网关路由，保护您的服务免受级联故障的影响，并允许您在下游故障时提供回退响应.

要在项目中启用Hystrix GatewayFilters，请从[Spring Cloud Netflix](https://cloud.spring.io/spring-cloud-netflix/)添加 `spring-cloud-starter-netflix-hystrix` 的依赖项.

Hystrix GatewayFilter Factory需要一个 `name` 参数，该参数是 `HystrixCommand` 的名称.

**application.yml.** 

```java
spring:
cloud:
gateway:
routes:
- id: hystrix_route
uri: http://example.org
filters:
- Hystrix=myCommandName
```

这将使用命令名 `myCommandName` 将剩余的过滤器包装在 `HystrixCommand` 中.

Hystrix过滤器还可以接受可选的 `fallbackUri` 参数.目前，仅支持 `forward:` 计划的URI.如果调用了回退，请求将被转发到与URI匹配的控制器.

**application.yml.** 

```java
spring:
cloud:
gateway:
routes:
- id: hystrix_route
uri: lb://backing-service:8088
predicates:
- Path=/consumingserviceendpoint
filters:
- name: Hystrix
args:
name: fallbackcmd
fallbackUri: forward:/incaseoffailureusethis
- RewritePath=/consumingserviceendpoint, /backingserviceendpoint
```

当调用Hystrix后备时，这将转发到 `/incaseoffailureusethis`  URI.请注意，此示例还通过目标URI上的 `lb` 前缀演示（可选）Spring Cloud Netflix Ribbon负载平衡.

Hystrix设置（例如超时）可以使用全局默认值配置，也可以使用应用程序属性逐个路径配置，如[Hystrix wiki](https://github.com/Netflix/Hystrix/wiki/Configuration)所述.

要为上面的示例路由设置5秒超时，将使用以下配置：

**application.yml.** 

```java
hystrix.command.fallbackcmd.execution.isolation.thread.timeoutInMilliseconds: 5000
```

## 115.5 PrefixPath GatewayFilter Factory

PrefixPath GatewayFilter Factory采用单个 `prefix` 参数.

**application.yml.** 

```java
spring:
cloud:
gateway:
routes:
- id: prefixpath_route
uri: http://example.org
filters:
- PrefixPath=/mypath
```

这将 `/mypath` 前缀为所有匹配请求的路径.因此，对 `/hello` 的请求将被发送到 `/mypath/hello` .

## 115.6 PreserveHostHeader GatewayFilter Factory

PreserveHostHeader GatewayFilter Factory没有参数.此过滤器设置路由过滤器将检查的请求属性，以确定是否应发送原始主机头，而不是http客户端确定的主机头.

**application.yml.** 

```java
spring:
cloud:
gateway:
routes:
- id: preserve_host_route
uri: http://example.org
filters:
- PreserveHostHeader
```

## 115.7 RequestRateLimiter GatewayFilter Factory

RequestRateLimiter GatewayFilter Factory使用 `RateLimiter` 实现来确定是否允许当前请求继续.如果不是，则返回 `HTTP 429 - Too Many Requests` （默认情况下）的状态.

此过滤器采用可选的 `keyResolver` 参数和特定于速率限制器的参数（参见下文）.

`keyResolver` 是一个实现 `KeyResolver` 接口的bean.在配置中，使用SpEL按名称引用bean.  `#{@myKeyResolver}` 是一个引用名为 `myKeyResolver` 的bean的SpEL表达式.

**KeyResolver.java.** 
```java
public interface KeyResolver {
	Mono<String> resolve(ServerWebExchange exchange);
}
```

`KeyResolver` 接口允许可插拔策略派生用于限制请求的密钥.在未来的里程碑中，将会有一些 `KeyResolver` 实现.

`KeyResolver` 的默认实现是 `PrincipalNameKeyResolver` ，它从 `ServerWebExchange` 检索 `Principal` 并调用 `Principal.getName()` .

>  RequestRateLimiter无法通过"shortcut"表示法进行配置.以下示例无效

**application.properties.** 

```java
# INVALID SHORTCUT CONFIGURATION
spring.cloud.gateway.routes[0].filters[0]=RequestRateLimiter=2, 2, #{@userkeyresolver}
```

### 115.7.1 Redis RateLimiter

redis实现基于[Stripe](https://stripe.com/blog/rate-limiters)完成的工作.它需要使用 `spring-boot-starter-data-redis-reactive`  Spring Boot启动器.

使用的算法是[Token Bucket Algorithm](https://en.wikipedia.org/wiki/Token_bucket).

`redis-rate-limiter.replenishRate` 是您希望允许用户每秒执行多少请求，而不会丢弃任何请求.这是令牌桶填充的速率.

`redis-rate-limiter.burstCapacity` 是用户在一秒钟内允许执行的最大请求数.这是令牌桶可以容纳的令牌数.将此值设置为零将阻止所有请求.

通过在 `replenishRate` 和 `burstCapacity` 中设置相同的值来实现稳定的速率.通过将 `burstCapacity` 设置为高于 `replenishRate` ，可以允许临时突发.在这种情况下，需要在突发之间允许速率限制器一段时间（根据 `replenishRate` ），因为连续2次突发将导致请求被丢弃（ `HTTP 429 - Too Many Requests` ）.

**application.yml.** 

```java
spring:
cloud:
gateway:
routes:
- id: requestratelimiter_route
uri: http://example.org
filters:
- name: RequestRateLimiter
args:
redis-rate-limiter.replenishRate: 10
redis-rate-limiter.burstCapacity: 20
```

**Config.java.** 

```java
@Bean
KeyResolver userKeyResolver() {
return exchange -> Mono.just(exchange.getRequest().getQueryParams().getFirst("user"));
}
```

这定义了每个用户10的请求率限制.允许突发20，但下一秒只有10个请求可用.  `KeyResolver` 是一个简单的获取 `user` 请求参数（注意：这不建议用于生产环境）.

速率限制器也可以定义为实现 `RateLimiter` 接口的bean.在配置中，使用SpEL按名称引用bean.  `#{@myRateLimiter}` 是一个引用名为 `myRateLimiter` 的bean的SpEL表达式.

**application.yml.** 

```java
spring:
cloud:
gateway:
routes:
- id: requestratelimiter_route
uri: http://example.org
filters:
- name: RequestRateLimiter
args:
rate-limiter: "#{@myRateLimiter}"
key-resolver: "#{@userKeyResolver}"
```

## 115.8 RedirectTo GatewayFilter Factory

RedirectTo GatewayFilter Factory采用 `status` 和 `url` 参数.状态应该是300系列重定向http代码，例如301. url应该是有效的URL.这将是 `Location` 标头的值.

**application.yml.** 

```java
spring:
cloud:
gateway:
routes:
- id: prefixpath_route
uri: http://example.org
filters:
- RedirectTo=302, http://acme.org
```

这将发送带有 `Location:http://acme.org` 标头的状态302以执行重定向.

## 115.9 RemoveNonProxyHeaders GatewayFilter Factory

RemoveNonProxyHeaders GatewayFilter Factory从转发的请求中删除标头.删除的标头的默认列表来自[IETF](https://tools.ietf.org/html/draft-ietf-httpbis-p1-messaging-14#section-7.1.3).

**The default removed headers are:** 

- Connection

- Keep，活着

- Proxy进行身份验证

- Proxy的授权

- TE

- Trailer

- Transfer编码

- Upgrade

要更改此设置，请将 `spring.cloud.gateway.filter.remove-non-proxy-headers.headers` 属性设置为要删除的Headers名称列表.

## 115.10 RemoveRequestHeader GatewayFilter Factory

RemoveRequestHeader GatewayFilter Factory采用 `name` 参数.它是要删除的标头的名称.

**application.yml.** 

```java
spring:
cloud:
gateway:
routes:
- id: removerequestheader_route
uri: http://example.org
filters:
- RemoveRequestHeader=X-Request-Foo
```

这将在向下游发送之前删除 `X-Request-Foo` 标头.

## 115.11 RemoveResponseHeader GatewayFilter Factory

RemoveResponseHeader GatewayFilter Factory采用 `name` 参数.它是要删除的标头的名称.

**application.yml.** 

```java
spring:
cloud:
gateway:
routes:
- id: removeresponseheader_route
uri: http://example.org
filters:
- RemoveResponseHeader=X-Response-Foo
```

这将在响应返回到网关客户端之前从响应中删除 `X-Response-Foo` 标头.

## 115.12 RewritePath GatewayFilter Factory

RewritePath GatewayFilter Factory采用路径 `regexp` 参数和 `replacement` 参数.这使用Java正则表达式来灵活地重写请求路径.

**application.yml.** 

```xml
spring:
cloud:
gateway:
routes:
- id: rewritepath_route
uri: http://example.org
predicates:
- Path=/foo/**
filters:
- RewritePath=/foo/(?<segment>.*), /$\{segment}
```

对于 `/foo/bar` 的请求路径，这将在发出下游请求之前将路径设置为 `/bar` .请注意 `$\` 由于YAML规范而被 `$` 替换.

## 115.13 SaveSession GatewayFilter Factory

SaveSession GatewayFilter Factory在转发下游呼叫之前强制执行 `WebSession::save` 操作.当使用[Spring Session](https://projects.spring.io/spring-session/)之类的内容与惰性数据存储时，这是特别有用的，并且需要确保在转发调用之前已保存会话状态.

**application.yml.** 

```java
spring:
cloud:
gateway:
routes:
- id: save_session
uri: http://example.org
predicates:
- Path=/foo/**
filters:
- SaveSession
```

如果要将[Spring Security](https://projects.spring.io/spring-security/)与Spring Session集成，并且希望确保安全性详细信息已转发到远程进程，则这很关键.

## 115.14 SecureHeaders GatewayFilter Factory

SecureHeaders GatewayFilter Factory在[this blog post](https://blog.appcanary.com/2017/http-security-headers.html)的reccomendation中为响应添加了许多Headers.

**The following headers are added (allong with default values):** 

-  `X-Xss-Protection:1; mode=block` 

-  `Strict-Transport-Security:max-age=631138519` 

-  `X-Frame-Options:DENY` 

-  `X-Content-Type-Options:nosniff` 

-  `Referrer-Policy:no-referrer` 

-  `Content-Security-Policy:default-src 'self' https:; font-src 'self' https: data:; img-src 'self' https: data:; object-src 'none'; script-src https:; style-src 'self' https: 'unsafe-inline'` 

-  `X-Download-Options:noopen` 

-  `X-Permitted-Cross-Domain-Policies:none` 

要更改默认值，请在 `spring.cloud.gateway.filter.secure-headers` 命名空间中设置相应的属性：

**Property to change:** 

-  `xss-protection-header` 

-  `strict-transport-security` 

-  `frame-options` 

-  `content-type-options` 

-  `referrer-policy` 

-  `content-security-policy` 

-  `download-options` 

-  `permitted-cross-domain-policies` 

## 115.15 SetPath GatewayFilter Factory

SetPath GatewayFilter Factory采用路径 `template` 参数.它提供了一种通过允许模板化路径段来操作请求路径的简单方法.这使用了Spring Framework中的uri模板.允许多个匹配的段.

**application.yml.** 

```java
spring:
cloud:
gateway:
routes:
- id: setpath_route
uri: http://example.org
predicates:
- Path=/foo/{segment}
filters:
- SetPath=/{segment}
```

对于 `/foo/bar` 的请求路径，这将在创建下游之前将路径设置为 `/bar` 请求.

## 115.16 SetResponseHeader GatewayFilter Factory

SetResponseHeader GatewayFilter Factory采用 `name` 和 `value` 参数.

**application.yml.** 

```java
spring:
cloud:
gateway:
routes:
- id: setresponseheader_route
uri: http://example.org
filters:
- SetResponseHeader=X-Response-Foo, Bar
```

此GatewayFilter将替换具有给定名称的所有标头，而不是添加.因此，如果下游服务器使用 `X-Response-Foo:1234` 进行响应，则会将其替换为 `X-Response-Foo:Bar` ，这是网关客户端将接收的内容.

## 115.17 SetStatus GatewayFilter Factory

SetStatus GatewayFilter Factory采用单个 `status` 参数.它必须是有效的Spring  `HttpStatus` .它可以是整数值 `404` 或枚举 `NOT_FOUND` 的字符串表示形式.

**application.yml.** 

```java
spring:
cloud:
gateway:
routes:
- id: setstatusstring_route
uri: http://example.org
filters:
- SetStatus=BAD_REQUEST
- id: setstatusint_route
uri: http://example.org
filters:
- SetStatus=401
```

在任何一种情况下，响应的HTTP状态都将设置为401.

## 115.18 StripPrefix GatewayFilter Factory

StripPrefix GatewayFilter Factory采用一个参数 `parts` .  `parts` 参数指示在向下游发送之前从请求中剥离的路径中的部分数.

**application.yml.** 

```java
spring:
cloud:
gateway:
routes:
- id: nameRoot
uri: http://nameservice
predicates:
- Path=/name/**
filters:
- StripPrefix=2
```

当通过网关发出请求 `/name/bar/foo` 时，对 `nameservice` 的请求看起来像 `http://nameservice/foo` .

## 115.19重试GatewayFilter Factory

Retry GatewayFilter Factory将 `retries` ， `statuses` ， `methods` 和 `series` 作为参数.

-  `retries` ：应尝试的重试次数

-  `statuses` ：应重试的HTTP状态代码，使用 `org.springframework.http.HttpStatus` 表示

-  `methods` ：应该重试的HTTP方法，使用 `org.springframework.http.HttpMethod` 表示

-  `series` ：要重试的一系列状态代码，使用 `org.springframework.http.HttpStatus.Series` 表示

**application.yml.** 

```java
spring:
cloud:
gateway:
routes:
- id: retry_test
uri: http://localhost:8080/flakey
predicates:
- Host=*.retry.com
filters:
- name: Retry
args:
retries: 3
statuses: BAD_GATEWAY
```

> 当使用带有 `forward:` 前缀URL的重试过滤器时，应仔细编写目标endpoints，以便在出现错误时不会执行任何可能导致响应发送到客户端并提交的操作.例如，如果目标endpoints是带注释的控制器，则目标控制器方法不应返回带有错误状态代码的 `ResponseEntity` .相反，它应抛出 `Exception` ，或发出错误信号，例如通过 `Mono.error(ex)` 返回值，可以将重试过滤器配置为通过重试来处理.

