## 116. Global Filters

The  `GlobalFilter`  interface has the same signature as  `GatewayFilter` . These are special filters that are conditionally applied to all routes. (This interface and usage are subject to change in future milestones).

## 116.1 Combined Global Filter and GatewayFilter Ordering

When a request comes in (and matches a Route) the Filtering Web Handler will add all instances of  `GlobalFilter`  and all route specific instances of  `GatewayFilter`  to a filter chain. This combined filter chain is sorted by the  `org.springframework.core.Ordered`  interface, which can be set by implementing the  `getOrder()`  method or by using the  `@Order`  annotation.

As Spring Cloud Gateway distinguishes between "pre" and "post" phases for filter logic execution (see: How It Works), the filter with the highest precedence will be the first in the "pre"-phase and the last in the "post"-phase.

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

## 116.2 Forward Routing Filter

The  `ForwardRoutingFilter`  looks for a URI in the exchange attribute  `ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR` . If the url has a  `forward`  scheme (ie  `forward:///localendpoint` ), it will use the Spring  `DispatcherHandler`  to handler the request. The path part of the request URL will be overridden with the path in the forward URL. The unmodified original url is appended to the list in the  `ServerWebExchangeUtils.GATEWAY_ORIGINAL_REQUEST_URL_ATTR`  attribute.

## 116.3 LoadBalancerClient Filter

The  `LoadBalancerClientFilter`  looks for a URI in the exchange attribute  `ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR` . If the url has a  `lb`  scheme (ie  `lb://myservice` ), it will use the Spring Cloud  `LoadBalancerClient`  to resolve the name ( `myservice`  in the previous example) to an actual host and port and replace the URI in the same attribute. The unmodified original url is appended to the list in the  `ServerWebExchangeUtils.GATEWAY_ORIGINAL_REQUEST_URL_ATTR`  attribute. The filter will also look in the  `ServerWebExchangeUtils.GATEWAY_SCHEME_PREFIX_ATTR`  attribute to see if it equals  `lb`  and then the same rules apply.

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

## 116.4 Netty Routing Filter

The Netty Routing Filter runs if the url located in the  `ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`  exchange attribute has a  `http`  or  `https`  scheme. It uses the Netty  `HttpClient`  to make the downstream proxy request. The response is put in the  `ServerWebExchangeUtils.CLIENT_RESPONSE_ATTR`  exchange attribute for use in a later filter. (There is an experimental  `WebClientHttpRoutingFilter`  that performs the same function, but does not require netty)

## 116.5 Netty Write Response Filter

The  `NettyWriteResponseFilter`  runs if there is a Netty  `HttpClientResponse`  in the  `ServerWebExchangeUtils.CLIENT_RESPONSE_ATTR`  exchange attribute. It is run after all other filters have completed and writes the proxy response back to the gateway client response. (There is an experimental  `WebClientWriteResponseFilter`  that performs the same function, but does not require netty)

## 116.6 RouteToRequestUrl Filter

The  `RouteToRequestUrlFilter`  runs if there is a  `Route`  object in the  `ServerWebExchangeUtils.GATEWAY_ROUTE_ATTR`  exchange attribute. It creates a new URI, based off of the request URI, but updated with the URI attribute of the  `Route`  object. The new URI is placed in the  `ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`  exchange attribute`.

If the URI has a scheme prefix, such as  `lb:ws://serviceid` , the  `lb`  scheme is stripped from the URI and placed in the  `ServerWebExchangeUtils.GATEWAY_SCHEME_PREFIX_ATTR`  for use later in the filter chain.

## 116.7 Websocket Routing Filter

The Websocket Routing Filter runs if the url located in the  `ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`  exchange attribute has a  `ws`  or  `wss`  scheme. It uses the Spring Web Socket infrastructure to forward the Websocket request downstream.

Websockets may be load-balanced by prefixing the URI with  `lb` , such as  `lb:ws://serviceid` .

> If you are using [SockJS](https://github.com/sockjs) as a fallback over normal http, you should configure a normal HTTP route as well as the Websocket Route.

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

## 116.8 Gateway Metrics Filter

To enable Gateway Metrics add spring-boot-starter-actuator as a project dependency. Then, by default, the Gateway Metrics Filter runs as long as the property  `spring.cloud.gateway.metrics.enabled`  is not set to  `false` . This filter adds a timer metric named "gateway.requests" with the following tags:

-  `routeId` : The route id

-  `routeUri` : The URI that the API will be routed to

-  `outcome` : Outcome as classified by [HttpStatus.Series](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/HttpStatus.Series.html)

-  `status` : Http Status of the request returned to the client

These metrics are then available to be scraped from  `/actuator/metrics/gateway.requests`  and can be easily integated with Prometheus to create a [Grafana](images/gateway-grafana-dashboard.jpeg) [dashboard](gateway-grafana-dashboard.json).

> To enable the pometheus endpoint add micrometer-registry-prometheus as a project dependency.

## 116.9 Making An Exchange As Routed

After the Gateway has routed a  `ServerWebExchange`  it will mark that exchange as "routed" by adding  `gatewayAlreadyRouted`  to the exchange attributes. Once a request has been marked as routed, other routing filters will not route the request again, essentially skipping the filter. There are convenience methods that you can use to mark an exchange as routed or check if an exchange has already been routed.

-  `ServerWebExchangeUtils.isAlreadyRouted`  takes a  `ServerWebExchange`  object and checks if it has been "routed"

-  `ServerWebExchangeUtils.setAlreadyRouted`  takes a  `ServerWebExchange`  object and marks it as "routed"

