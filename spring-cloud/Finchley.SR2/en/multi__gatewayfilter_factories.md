## 115. GatewayFilter Factories

Route filters allow the modification of the incoming HTTP request or outgoing HTTP response in some manner. Route filters are scoped to a particular route. Spring Cloud Gateway includes many built-in GatewayFilter Factories.

NOTE For more detailed examples on how to use any of the following filters, take a look at the [unit tests](https://github.com/spring-cloud/spring-cloud-gateway/tree/master/spring-cloud-gateway-core/src/test/java/org/springframework/cloud/gateway/filter/factory).

## 115.1 AddRequestHeader GatewayFilter Factory

The AddRequestHeader GatewayFilter Factory takes a name and value parameter.

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

This will add  `X-Request-Foo:Bar`  header to the downstream request’s headers for all matching requests.

## 115.2 AddRequestParameter GatewayFilter Factory

The AddRequestParameter GatewayFilter Factory takes a name and value parameter.

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

This will add  `foo=bar`  to the downstream request’s query string for all matching requests.

## 115.3 AddResponseHeader GatewayFilter Factory

The AddResponseHeader GatewayFilter Factory takes a name and value parameter.

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

This will add  `X-Response-Foo:Bar`  header to the downstream response’s headers for all matching requests.

## 115.4 Hystrix GatewayFilter Factory

[Hystrix](https://github.com/Netflix/Hystrix) is a library from Netflix that implements the [circuit breaker pattern](https://martinfowler.com/bliki/CircuitBreaker.html). The Hystrix GatewayFilter allows you to introduce circuit breakers to your gateway routes, protecting your services from cascading failures and allowing you to provide fallback responses in the event of downstream failures.

To enable Hystrix GatewayFilters in your project, add a dependency on  `spring-cloud-starter-netflix-hystrix`  from [Spring Cloud Netflix](https://cloud.spring.io/spring-cloud-netflix/).

The Hystrix GatewayFilter Factory requires a single  `name`  parameter, which is the name of the  `HystrixCommand` .

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

This wraps the remaining filters in a  `HystrixCommand`  with command name  `myCommandName` .

The Hystrix filter can also accept an optional  `fallbackUri`  parameter. Currently, only  `forward:`  schemed URIs are supported. If the fallback is called, the request will be forwarded to the controller matched by the URI.

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

This will forward to the  `/incaseoffailureusethis`  URI when the Hystrix fallback is called. Note that this example also demonstrates (optional) Spring Cloud Netflix Ribbon load-balancing via the  `lb`  prefix on the destination URI.

Hystrix settings (such as timeouts) can be configured with global defaults or on a route by route basis using application properties as explained on the [Hystrix wiki](https://github.com/Netflix/Hystrix/wiki/Configuration).

To set a 5 second timeout for the example route above, the following configuration would be used:

**application.yml.**  

```java
hystrix.command.fallbackcmd.execution.isolation.thread.timeoutInMilliseconds: 5000
```

## 115.5 PrefixPath GatewayFilter Factory

The PrefixPath GatewayFilter Factory takes a single  `prefix`  parameter.

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

This will prefix  `/mypath`  to the path of all matching requests. So a request to  `/hello` , would be sent to  `/mypath/hello` .

## 115.6 PreserveHostHeader GatewayFilter Factory

The PreserveHostHeader GatewayFilter Factory has not parameters. This filter, sets a request attribute that the routing filter will inspect to determine if the original host header should be sent, rather than the host header determined by the http client.

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

The RequestRateLimiter GatewayFilter Factory is uses a  `RateLimiter`  implementation to determine if the current request is allowed to proceed. If it is not, a status of  `HTTP 429 - Too Many Requests`  (by default) is returned.

This filter takes an optional  `keyResolver`  parameter and parameters specific to the rate limiter (see below).

`keyResolver`  is a bean that implements the  `KeyResolver`  interface. In configuration, reference the bean by name using SpEL.  `#{@myKeyResolver}`  is a SpEL expression referencing a bean with the name  `myKeyResolver` .

**KeyResolver.java.**  

```java
public interface KeyResolver {
	Mono<String> resolve(ServerWebExchange exchange);
}
```

The  `KeyResolver`  interface allows pluggable strategies to derive the key for limiting requests. In future milestones, there will be some  `KeyResolver`  implementations.

The default implementation of  `KeyResolver`  is the  `PrincipalNameKeyResolver`  which retrieves the  `Principal`  from the  `ServerWebExchange`  and calls  `Principal.getName()` .

> The RequestRateLimiter is not configurable via the "shortcut" notation. The example below is invalid

**application.properties.**  

```java
# INVALID SHORTCUT CONFIGURATION
spring.cloud.gateway.routes[0].filters[0]=RequestRateLimiter=2, 2, #{@userkeyresolver}
```

### 115.7.1 Redis RateLimiter

The redis implementation is based off of work done at [Stripe](https://stripe.com/blog/rate-limiters). It requires the use of the  `spring-boot-starter-data-redis-reactive`  Spring Boot starter.

The algorithm used is the [Token Bucket Algorithm](https://en.wikipedia.org/wiki/Token_bucket).

The  `redis-rate-limiter.replenishRate`  is how many requests per second do you want a user to be allowed to do, without any dropped requests. This is the rate that the token bucket is filled.

The  `redis-rate-limiter.burstCapacity`  is the maximum number of requests a user is allowed to do in a single second. This is the number of tokens the token bucket can hold. Setting this value to zero will block all requests.

A steady rate is accomplished by setting the same value in  `replenishRate`  and  `burstCapacity` . Temporary bursts can be allowed by setting  `burstCapacity`  higher than  `replenishRate` . In this case, the rate limiter needs to be allowed some time between bursts (according to  `replenishRate` ), as 2 consecutive bursts will result in dropped requests ( `HTTP 429 - Too Many Requests` ).

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

This defines a request rate limit of 10 per user. A burst of 20 is allowed, but the next second only 10 requests will be available. The  `KeyResolver`  is a simple one that gets the  `user`  request parameter (note: this is not recommended for production).

A rate limiter can also be defined as a bean implementing the  `RateLimiter`  interface. In configuration, reference the bean by name using SpEL.  `#{@myRateLimiter}`  is a SpEL expression referencing a bean with the name  `myRateLimiter` .

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

The RedirectTo GatewayFilter Factory takes a  `status`  and a  `url`  parameter. The status should be a 300 series redirect http code, such as 301. The url should be a valid url. This will be the value of the  `Location`  header.

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

This will send a status 302 with a  `Location:http://acme.org`  header to perform a redirect.

## 115.9 RemoveNonProxyHeaders GatewayFilter Factory

The RemoveNonProxyHeaders GatewayFilter Factory removes headers from forwarded requests. The default list of headers that is removed comes from the [IETF](https://tools.ietf.org/html/draft-ietf-httpbis-p1-messaging-14#section-7.1.3).

**The default removed headers are:** 

- Connection

- Keep-Alive

- Proxy-Authenticate

- Proxy-Authorization

- TE

- Trailer

- Transfer-Encoding

- Upgrade

To change this, set the  `spring.cloud.gateway.filter.remove-non-proxy-headers.headers`  property to the list of header names to remove.

## 115.10 RemoveRequestHeader GatewayFilter Factory

The RemoveRequestHeader GatewayFilter Factory takes a  `name`  parameter. It is the name of the header to be removed.

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

This will remove the  `X-Request-Foo`  header before it is sent downstream.

## 115.11 RemoveResponseHeader GatewayFilter Factory

The RemoveResponseHeader GatewayFilter Factory takes a  `name`  parameter. It is the name of the header to be removed.

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

This will remove the  `X-Response-Foo`  header from the response before it is returned to the gateway client.

## 115.12 RewritePath GatewayFilter Factory

The RewritePath GatewayFilter Factory takes a path  `regexp`  parameter and a  `replacement`  parameter. This uses Java regular expressions for a flexible way to rewrite the request path.

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

For a request path of  `/foo/bar` , this will set the path to  `/bar`  before making the downstream request. Notice the  `$\`  which is replaced with  `$`  because of the YAML spec.

## 115.13 SaveSession GatewayFilter Factory

The SaveSession GatewayFilter Factory forces a  `WebSession::save`  operation before forwarding the call downstream. This is of particular use when using something like [Spring Session](https://projects.spring.io/spring-session/) with a lazy data store and need to ensure the session state has been saved before making the forwarded call.

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

If you are integrating [Spring Security](https://projects.spring.io/spring-security/) with Spring Session, and want to ensure security details have been forwarded to the remote process, this is critical.

## 115.14 SecureHeaders GatewayFilter Factory

The SecureHeaders GatewayFilter Factory adds a number of headers to the response at the reccomendation from [this blog post](https://blog.appcanary.com/2017/http-security-headers.html).

**The following headers are added (allong with default values):** 

-  `X-Xss-Protection:1; mode=block` 

-  `Strict-Transport-Security:max-age=631138519` 

-  `X-Frame-Options:DENY` 

-  `X-Content-Type-Options:nosniff` 

-  `Referrer-Policy:no-referrer` 

-  `Content-Security-Policy:default-src 'self' https:; font-src 'self' https: data:; img-src 'self' https: data:; object-src 'none'; script-src https:; style-src 'self' https: 'unsafe-inline'` 

-  `X-Download-Options:noopen` 

-  `X-Permitted-Cross-Domain-Policies:none` 

To change the default values set the appropriate property in the  `spring.cloud.gateway.filter.secure-headers`  namespace:

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

The SetPath GatewayFilter Factory takes a path  `template`  parameter. It offers a simple way to manipulate the request path by allowing templated segments of the path. This uses the uri templates from Spring Framework. Multiple matching segments are allowed.

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

For a request path of  `/foo/bar` , this will set the path to  `/bar`  before making the downstream request.

## 115.16 SetResponseHeader GatewayFilter Factory

The SetResponseHeader GatewayFilter Factory takes  `name`  and  `value`  parameters.

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

This GatewayFilter replaces all headers with the given name, rather than adding. So if the downstream server responded with a  `X-Response-Foo:1234` , this would be replaced with  `X-Response-Foo:Bar` , which is what the gateway client would receive.

## 115.17 SetStatus GatewayFilter Factory

The SetStatus GatewayFilter Factory takes a single  `status`  parameter. It must be a valid Spring  `HttpStatus` . It may be the integer value  `404`  or the string representation of the enumeration  `NOT_FOUND` .

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

In either case, the HTTP status of the response will be set to 401.

## 115.18 StripPrefix GatewayFilter Factory

The StripPrefix GatewayFilter Factory takes one paramter,  `parts` . The  `parts`  parameter indicated the number of parts in the path to strip from the request before sending it downstream.

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

When a request is made through the gateway to  `/name/bar/foo`  the request made to  `nameservice`  will look like  `http://nameservice/foo` .

## 115.19 Retry GatewayFilter Factory

The Retry GatewayFilter Factory takes  `retries` ,  `statuses` ,  `methods` , and  `series`  as parameters.

-  `retries` : the number of retries that should be attempted

-  `statuses` : the HTTP status codes that should be retried, represented using  `org.springframework.http.HttpStatus` 

-  `methods` : the HTTP methods that should be retried, represented using  `org.springframework.http.HttpMethod` 

-  `series` : the series of status codes to be retried, represented using  `org.springframework.http.HttpStatus.Series` 

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

> When using the retry filter with a  `forward:`  prefixed URL, the target endpoint should be written carefully so that in case of an error it does not do anything that could result in a response being sent to the client and committed. For example, if the target endpoint is an annotated controller, the target controller method should not return  `ResponseEntity`  with an error status code. Instead it should throw an  `Exception` , or signal an error, e.g. via a  `Mono.error(ex)`  return value, which the retry filter can be configured to handle by retrying.

