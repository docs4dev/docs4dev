## 121. Developer Guide

TODO: overview of writing custom integrations

## 121.1 Writing Custom Route Predicate Factories

TODO: document writing Custom Route Predicate Factories

## 121.2 Writing Custom GatewayFilter Factories

In order to write a GatewayFilter you will need to implement  `GatewayFilterFactory` . There is an abstract class called  `AbstractGatewayFilterFactory`  which you can extend.

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

## 121.3 Writing Custom Global Filters

TODO: document writing Custom Global Filters

## 121.4 Writing Custom Route Locators and Writers

TODO: document writing Custom Route Locators and Writers

