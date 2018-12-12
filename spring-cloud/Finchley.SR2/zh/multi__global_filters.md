## 116.全局过滤器

`GlobalFilter` 接口与 `GatewayFilter` 具有相同的签名.这些是有条件地应用于所有路线的特殊过滤器. （此界面和用法可能会在未来的里程碑中发生变化）.

## 116.1组合全局过滤器和GatewayFilter排序

当请求进入（并匹配路由）时，Filtering Web Handler会将 `GlobalFilter` 的所有实例和 `GatewayFilter` 的所有路由特定实例添加到过滤器链中.此组合过滤器链按 `org.springframework.core.Ordered` 接口排序，可通过实现 `getOrder()` 方法或使用 `@Order` 注释进行设置.

由于Spring Cloud Gateway区分了过滤器逻辑执行的“前”和“后”阶段（请参阅：工作原理），具有最高优先级的过滤器将是“前”阶段中的第一个和“后”中的最后一个“-相.

**ExampleConfiguration.java.** 

```java
@Bean
@Order(-1)
public GlobalFilter a() {
return (exchange, chain) -> {
log.info("first pre filter");
return chain.filter(exchange).then(Mono.fromRunnable(() -> {
log.info("third post filter");
}));
};
}

@Bean
@Order(0)
public GlobalFilter b() {
return (exchange, chain) -> {
log.info("second pre filter");
return chain.filter(exchange).then(Mono.fromRunnable(() -> {
log.info("second post filter");
}));
};
}

@Bean
@Order(1)
public GlobalFilter c() {
return (exchange, chain) -> {
log.info("third pre filter");
return chain.filter(exchange).then(Mono.fromRunnable(() -> {
log.info("first post filter");
}));
};
}
```

## 116.2前向路由过滤器

`ForwardRoutingFilter` 在交换属性 `ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR` 中查找URI.如果url具有 `forward` 方案（即 `forward:///localendpoint` ），它将使用Spring  `DispatcherHandler` 来处理请求.请求URL的路径部分将被转发URL中的路径覆盖.未修改的原始URL将附加到 `ServerWebExchangeUtils.GATEWAY_ORIGINAL_REQUEST_URL_ATTR` 属性中的列表中.

## 116.3 LoadBalancerClient过滤器

`LoadBalancerClientFilter` 在交换属性 `ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR` 中查找URI.如果url具有 `lb` 方案（即 `lb://myservice` ），它将使用Spring Cloud  `LoadBalancerClient` 将名称（前一示例中的 `myservice` ）解析为实际主机和端口，并替换相同属性中的URI.未修改的原始URL将附加到 `ServerWebExchangeUtils.GATEWAY_ORIGINAL_REQUEST_URL_ATTR` 属性中的列表中.过滤器还将查看 `ServerWebExchangeUtils.GATEWAY_SCHEME_PREFIX_ATTR` 属性以查看它是否等于 `lb` 然后应用相同的规则.

**application.yml.** 

```java
spring:
cloud:
gateway:
routes:
- id: myRoute
uri: lb://service
predicates:
- Path=/service/**
```

## 116.4网络路由过滤器

如果位于 `ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`  exchange属性中的URL具有 `http` 或 `https` 方案，则运行Netty路由过滤器.它使用Netty  `HttpClient` 来发出下游代理请求.响应将放在 `ServerWebExchangeUtils.CLIENT_RESPONSE_ATTR`  exchange属性中，以便在以后的过滤器中使用. （有一个实验 `WebClientHttpRoutingFilter` 执行相同的功能，但不需要netty）

## 116.5 Netty写响应过滤器

如果 `ServerWebExchangeUtils.CLIENT_RESPONSE_ATTR`  exchange属性中存在Netty  `HttpClientResponse` ， `NettyWriteResponseFilter` 将运行.它在所有其他过滤器完成后运行，并将代理响应写回网关客户端响应. （有一个实验 `WebClientWriteResponseFilter` 执行相同的功能，但不需要netty）

## 116.6 RouteToRequestUrl过滤器

如果 `ServerWebExchangeUtils.GATEWAY_ROUTE_ATTR`  exchange属性中有 `Route` 对象，则运行 `RouteToRequestUrlFilter` .它根据请求URI创建一个新URI，但使用 `Route` 对象的URI属性进行更新.新URI位于 `ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`  exchange属性`中.

如果URI具有方案前缀，例如 `lb:ws://serviceid` ，则 `lb` 方案将从URI中剥离并放置在 `ServerWebExchangeUtils.GATEWAY_SCHEME_PREFIX_ATTR` 中以供稍后使用过滤链.

## 116.7 Websocket路由过滤器

如果位于 `ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`  exchange属性中的URL具有 `ws` 或 `wss` 方案，则运行Websocket路由过滤器.它使用Spring Web Socket基础结构将Websocket请求转发到下游.

可以通过在URI前面添加 `lb` （例如 `lb:ws://serviceid` ）来对Websockets进行负载平衡.

> 如果您使用[SockJS](https://github.com/sockjs)作为普通http的后备，则应配置普通的HTTP路由以及Websocket路由.

**application.yml.** 

```java
spring:
cloud:
gateway:
routes:
# SockJS route
- id: websocket_sockjs_route
uri: http://localhost:3001
predicates:
- Path=/websocket/info/**
# Normwal Websocket route
- id: websocket_route
uri: ws://localhost:3001
predicates:
- Path=/websocket/**
```

## 116.8网关指标过滤器

要启用Gateway Metrics，请将spring-boot-starter-actuator添加为项目依赖项.然后，默认情况下，只要属性 `spring.cloud.gateway.metrics.enabled` 未设置为 `false` ，网关度量标准筛选器就会运行.此过滤器添加名为"gateway.requests"的计时器度量标准，其中包含以下标记：

-  `routeId` ：路线ID

-  `routeUri` ：API将路由到的URI

-  `outcome` ：按[HttpStatus.Series](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/HttpStatus.Series.html)分类的结果

-  `status` ：请求的Http状态返回给客户端

这些指标随后可以从 `/actuator/metrics/gateway.requests` 中删除，并且可以与Prometheus轻松集成以创建[Grafana](images/gateway-grafana-dashboard.jpeg) [dashboard](gateway-grafana-dashboard.json).

> 启用pometheusendpoints添加千分尺 - 注册表 -  prometheus作为项目依赖项.

## 116.9将交换路由为路由

在网关路由 `ServerWebExchange` 之后，它会通过将 `gatewayAlreadyRouted` 添加到交换机属性来将该交换标记为"routed".一旦请求被标记为路由，其他路由过滤器将不会再次路由请求，实质上是跳过过滤器.您可以使用便捷方法将交换标记为路由，或检查交换是否已路由.

-  `ServerWebExchangeUtils.isAlreadyRouted` 获取 `ServerWebExchange` 对象并检查它是否"routed"

-  `ServerWebExchangeUtils.setAlreadyRouted` 获取 `ServerWebExchange` 对象并将其标记为"routed"

