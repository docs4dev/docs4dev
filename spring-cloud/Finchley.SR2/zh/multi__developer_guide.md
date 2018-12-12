## 121.开发人员指南

TODO：编写自定义集成的概述

## 121.1编写自定义路线谓词工厂

TODO：撰写Custom Route Predicate Factories的文档

## 121.2编写自定义GatewayFilter工厂

为了编写GatewayFilter，你将会需要实现 `GatewayFilterFactory` .有一个名为 `AbstractGatewayFilterFactory` 的抽象类，您可以扩展它.

**PreGatewayFilterFactory.java.** 

```java
public class PreGatewayFilterFactory extends AbstractGatewayFilterFactory<PreGatewayFilterFactory.Config> {

	public PreGatewayFilterFactory() {
		super(Config.class);
	}

	@Override
	public GatewayFilter apply(Config config) {
		// grab configuration from Config object
		return (exchange, chain) -> {
//If you want to build a "pre" filter you need to manipulate the
//request before calling change.filter
ServerHttpRequest.Builder builder = exchange.getRequest().mutate();
//use builder to manipulate the request
return chain.filter(exchange.mutate().request(request).build());
		};
	}

	public static class Config {
//Put the configuration properties for your filter here
	}

}
```

**PostGatewayFilterFactory.java.** 

```java
public class PostGatewayFilterFactory extends AbstractGatewayFilterFactory<PostGatewayFilterFactory.Config> {

	public PostGatewayFilterFactory() {
		super(Config.class);
	}

	@Override
	public GatewayFilter apply(Config config) {
		// grab configuration from Config object
		return (exchange, chain) -> {
			return chain.filter(exchange).then(Mono.fromRunnable(() -> {
				ServerHttpResponse response = exchange.getResponse();
				//Manipulate the response in some way
			}));
		};
	}

	public static class Config {
//Put the configuration properties for your filter here
	}

}
```

## 121.3编写自定义全局过滤器

TODO：编写自定义全局过滤器的文档

## 121.4编写自定义路由定位器和写入器

TODO：编写自定义路由定位器和写入器的文档

