## 114.路线谓词工厂

Spring Cloud Gateway将路由作为Spring WebFlux  `HandlerMapping` 基础结构的一部分进行匹配. Spring Cloud Gateway包含许多内置的Route Predicate Factories.所有这些谓词都匹配HTTP请求的不同属性.多路线谓词工厂可以组合并通过逻辑 `and` 组合.

## 114.1路线谓词工厂之后

After Route谓词工厂采用一个参数，一个日期时间.此谓词匹配当前日期时间之后发生的请求.

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

此路线与2017年1月20日17:42 Mountain Time（Denver）之后的所有要求相匹配.

## 114.2前路线谓词工厂

Before Route Predicate Factory采用一个参数，即日期时间.此谓词匹配在当前日期时间之前发生的请求.

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

此路线与2017年1月20日17:42 Mountain Time（Denver）之前的任何要求相匹配.

## 114.3在Route Predicate Factory之间

Between Route Predicate Factory有两个参数，datetime1和datetime2.此谓词匹配datetime1之后和datetime2之前发生的请求. datetime2参数必须在datetime1之后.

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

此路线与2017年1月20日17:42山地时间（丹佛）之后和2017年1月21日17:42山区时间（丹佛）之后的所有要求相匹配.这对维护窗口很有用.

## 114.4 Cookie Route Predicate Factory

Cookie Route Predicate Factory有两个参数，cookie名称和正则表达式.此谓词匹配具有给定名称且值与正则表达式匹配的cookie.

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

此路由匹配请求有一个名为 `chocolate` 的cookie，其值与 `ch.p` 正则表达式匹配.

## 114.5Headers路线谓词工厂

Header Route Predicate Factory有两个参数，Headers名称和正则表达式.此谓词与具有给定名称且值与正则表达式匹配的标头匹配.

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

如果请求具有名为 `X-Request-Id` 的标头，则该路由匹配，其值与 `\d+` 正则表达式匹配（具有一个或多个数字的值）.

## 114.6主机路线谓词工厂

Host Route Predicate Factory采用一个参数：主机名模式.该模式是一个Ant样式模式，其中 `.` 作为分隔符.此谓词匹配与模式匹配的 `Host` 标头.

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

如果请求的 `Host` 标头具有值 `www.somehost.org` 或 `beta.somehost.org` ，则此路由将匹配.

## 114.7方法路线谓词工厂

Method Route Predicate Factory采用一个参数：要匹配的HTTP方法.

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

如果请求方法是 `GET` ，则此路由将匹配.

## 114.8 Path Route Predicate Factory

Path Route Predicate Factory采用一个参数：Spring  `PathMatcher` 模式.

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

如果请求路径是，则此路由将匹配，例如： `/foo/1` 或 `/foo/bar` .

此谓词将URI模板变量（如上例中定义的 `segment` ）提取为名称和值的映射，并将其放在 `ServerWebExchange.getAttributes()` 中，并在 `PathRoutePredicate.URL_PREDICATE_VARS_ATTR` 中定义键.这些值随后可供[GatewayFilter Factories](multi_gateway-request-predicates-factories.html#gateway-route-filters)使用

## 114.9查询路线谓词工厂

Query Route Predicate Factory有两个参数：一个必需的 `param` 和一个可选的 `regexp` .

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

如果请求包含 `baz` 查询参数，则此路由将匹配.

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

如果请求包含 `foo` 查询参数且其值与 `ba.`  regexp匹配，则此路由将匹配，因此 `bar` 和 `baz` 将匹配.

## 114.10 RemoteAddr Route Predicate Factory

RemoteAddr Route Predicate Factory采用CIDR符号（IPv4或IPv6）字符串的列表（最小值为1），例如，  `192.168.0.1/16` （其中 `192.168.0.1` 是IP地址， `16` 是子网掩码）.

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

如果请求的远程地址是例如 `192.168.1.10` ，则此路由将匹配.

### 114.10.1修改解析远程地址的方式

默认情况下，RemoteAddrRoute Predicate Factory使用传入请求中的远程地址.如果Spring Cloud Gateway位于代理层后面，则可能与实际客户端IP地址不匹配.

您可以通过设置自定义 `RemoteAddressResolver` 来自定义解析远程地址的方式. Spring Cloud Gateway附带一个非默认的远程地址解析器，它基于[X-Forwarded-For header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-For)， `XForwardedRemoteAddressResolver` .

`XForwardedRemoteAddressResolver` 有两个静态构造函数方法，它们采用不同的安全方法：

`XForwardedRemoteAddressResolver::trustAll` 返回 `RemoteAddressResolver` ，它始终采用 `X-Forwarded-For` 标头中找到的第一个IP地址.这种方法容易受到欺骗，因为恶意客户端可以为解析器接受的 `X-Forwarded-For` 设置初始值.

`XForwardedRemoteAddressResolver::maxTrustedIndex` 采用与Spring Cloud Gateway前运行的可信基础架构数量相关的索引.例如，如果只能通过HAProxy访问Spring Cloud Gateway，则应使用值1.如果在可访问Spring Cloud Gateway之前需要两跳可信基础结构，则应使用值2.

给出以下标头值：

```java
X-Forwarded-For: 0.0.0.1, 0.0.0.2, 0.0.0.3
```

下面的 `maxTrustedIndex` 值将产生以下远程地址.

|  `maxTrustedIndex`  |结果|
| ---- | ---- |
| [ `Integer.MIN_VALUE` ，0] |（初始化期间无效， `IllegalArgumentException` ）|
| 1 | 0.0.0.3 |
| 2 | 0.0.0.2 |
| 3 | 0.0.0.1 |
| [4， `Integer.MAX_VALUE` ] | 0.0.0.1 |

使用Java配置：

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

