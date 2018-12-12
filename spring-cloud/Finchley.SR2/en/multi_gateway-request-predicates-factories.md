## 114. Route Predicate Factories

Spring Cloud Gateway matches routes as part of the Spring WebFlux  `HandlerMapping`  infrastructure. Spring Cloud Gateway includes many built-in Route Predicate Factories. All of these predicates match on different attributes of the HTTP request. Multiple Route Predicate Factories can be combined and are combined via logical  `and` .

## 114.1 After Route Predicate Factory

The After Route Predicate Factory takes one parameter, a datetime. This predicate matches requests that happen after the current datetime.

**application.yml.**  

```java
spring:
cloud:
gateway:
routes:
- id: after_route
uri: http://example.org
predicates:
- After=2017-01-20T17:42:47.789-07:00[America/Denver]
```

This route matches any request after Jan 20, 2017 17:42 Mountain Time (Denver).

## 114.2 Before Route Predicate Factory

The Before Route Predicate Factory takes one parameter, a datetime. This predicate matches requests that happen before the current datetime.

**application.yml.**  

```java
spring:
cloud:
gateway:
routes:
- id: before_route
uri: http://example.org
predicates:
- Before=2017-01-20T17:42:47.789-07:00[America/Denver]
```

This route matches any request before Jan 20, 2017 17:42 Mountain Time (Denver).

## 114.3 Between Route Predicate Factory

The Between Route Predicate Factory takes two parameters, datetime1 and datetime2. This predicate matches requests that happen after datetime1 and before datetime2. The datetime2 parameter must be after datetime1.

**application.yml.**  

```java
spring:
cloud:
gateway:
routes:
- id: between_route
uri: http://example.org
predicates:
- Between=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[America/Denver]
```

This route matches any request after Jan 20, 2017 17:42 Mountain Time (Denver) and before Jan 21, 2017 17:42 Mountain Time (Denver). This could be useful for maintenance windows.

## 114.4 Cookie Route Predicate Factory

The Cookie Route Predicate Factory takes two parameters, the cookie name and a regular expression. This predicate matches cookies that have the given name and the value matches the regular expression.

**application.yml.**  

```java
spring:
cloud:
gateway:
routes:
- id: cookie_route
uri: http://example.org
predicates:
- Cookie=chocolate, ch.p
```

This route matches the request has a cookie named  `chocolate`  who’s value matches the  `ch.p`  regular expression.

## 114.5 Header Route Predicate Factory

The Header Route Predicate Factory takes two parameters, the header name and a regular expression. This predicate matches with a header that has the given name and the value matches the regular expression.

**application.yml.**  

```java
spring:
cloud:
gateway:
routes:
- id: header_route
uri: http://example.org
predicates:
- Header=X-Request-Id, \d+
```

This route matches if the request has a header named  `X-Request-Id`  whos value matches the  `\d+`  regular expression (has a value of one or more digits).

## 114.6 Host Route Predicate Factory

The Host Route Predicate Factory takes one parameter: the host name pattern. The pattern is an Ant style pattern with  `.`  as the separator. This predicates matches the  `Host`  header that matches the pattern.

**application.yml.**  

```java
spring:
cloud:
gateway:
routes:
- id: host_route
uri: http://example.org
predicates:
- Host=**.somehost.org
```

This route would match if the request has a  `Host`  header has the value  `www.somehost.org`  or  `beta.somehost.org` .

## 114.7 Method Route Predicate Factory

The Method Route Predicate Factory takes one parameter: the HTTP method to match.

**application.yml.**  

```java
spring:
cloud:
gateway:
routes:
- id: method_route
uri: http://example.org
predicates:
- Method=GET
```

This route would match if the request method was a  `GET` .

## 114.8 Path Route Predicate Factory

The Path Route Predicate Factory takes one parameter: a Spring  `PathMatcher`  pattern.

**application.yml.**  

```java
spring:
cloud:
gateway:
routes:
- id: host_route
uri: http://example.org
predicates:
- Path=/foo/{segment}
```

This route would match if the request path was, for example:  `/foo/1`  or  `/foo/bar` .

This predicate extracts the URI template variables (like  `segment`  defined in the example above) as a map of names and values and places it in the  `ServerWebExchange.getAttributes()`  with a key defined in  `PathRoutePredicate.URL_PREDICATE_VARS_ATTR` . Those values are then available for use by [GatewayFilter Factories](multi_gateway-request-predicates-factories.html#gateway-route-filters)

## 114.9 Query Route Predicate Factory

The Query Route Predicate Factory takes two parameters: a required  `param`  and an optional  `regexp` .

**application.yml.**  

```java
spring:
cloud:
gateway:
routes:
- id: query_route
uri: http://example.org
predicates:
- Query=baz
```

This route would match if the request contained a  `baz`  query parameter.

**application.yml.**  

```java
spring:
cloud:
gateway:
routes:
- id: query_route
uri: http://example.org
predicates:
- Query=foo, ba.
```

This route would match if the request contained a  `foo`  query parameter whose value matched the  `ba.`  regexp, so  `bar`  and  `baz`  would match.

## 114.10 RemoteAddr Route Predicate Factory

The RemoteAddr Route Predicate Factory takes a list (min size 1) of CIDR-notation (IPv4 or IPv6) strings, e.g.  `192.168.0.1/16`  (where  `192.168.0.1`  is an IP address and  `16`  is a subnet mask).

**application.yml.**  

```java
spring:
cloud:
gateway:
routes:
- id: remoteaddr_route
uri: http://example.org
predicates:
- RemoteAddr=192.168.1.1/24
```

This route would match if the remote address of the request was, for example,  `192.168.1.10` .

### 114.10.1 Modifying the way remote addresses are resolved

By default the RemoteAddr Route Predicate Factory uses the remote address from the incoming request. This may not match the actual client IP address if Spring Cloud Gateway sits behind a proxy layer.

You can customize the way that the remote address is resolved by setting a custom  `RemoteAddressResolver` . Spring Cloud Gateway comes with one non-default remote address resolver which is based off of the [X-Forwarded-For header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-For),  `XForwardedRemoteAddressResolver` .

`XForwardedRemoteAddressResolver`  has two static constructor methods which take different approaches to security:

`XForwardedRemoteAddressResolver::trustAll`  returns a  `RemoteAddressResolver`  which always takes the first IP address found in the  `X-Forwarded-For`  header. This approach is vulnerable to spoofing, as a malicious client could set an initial value for the  `X-Forwarded-For`  which would be accepted by the resolver.

`XForwardedRemoteAddressResolver::maxTrustedIndex`  takes an index which correlates to the number of trusted infrastructure running in front of Spring Cloud Gateway. If Spring Cloud Gateway is, for example only accessible via HAProxy, then a value of 1 should be used. If two hops of trusted infrastructure are required before Spring Cloud Gateway is accessible, then a value of 2 should be used.

Given the following header value:

```java
X-Forwarded-For: 0.0.0.1, 0.0.0.2, 0.0.0.3
```

The  `maxTrustedIndex`  values below will yield the following remote addresses.

| `maxTrustedIndex` |result|
|----|----|
|[ `Integer.MIN_VALUE` ,0] |(invalid,  `IllegalArgumentException`  during initialization) |
|1 |0.0.0.3 |
|2 |0.0.0.2 |
|3 |0.0.0.1 |
|[4,  `Integer.MAX_VALUE` ] |0.0.0.1 |

Using Java config:

GatewayConfig.java

```java
RemoteAddressResolver resolver = XForwardedRemoteAddressResolver
.maxTrustedIndex(1);

...

.route("direct-route",
r -> r.remoteAddr("10.1.1.1", "10.10.1.1/24")
.uri("https://downstream1")
.route("proxied-route",
r -> r.remoteAddr(resolver,  "10.10.1.1", "10.10.1.1/24")
.uri("https://downstream2")
)
```

