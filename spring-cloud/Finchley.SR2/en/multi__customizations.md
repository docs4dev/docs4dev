## 59. Customizations

## 59.1 HTTP

If a customization of client / server parsing of the HTTP related spans is required, just register a bean of type  `brave.http.HttpClientParser`  or  `brave.http.HttpServerParser` . If client /server sampling is required, just register a bean of type  `brave.http.HttpSampler`  and name the bean  `sleuthClientSampler`  for client sampler and  `sleuthServerSampler`  for server sampler. For your convenience the  `@ClientSampler`  and  `@ServerSampler`  annotations can be used to inject the proper beans or to reference the bean names via their static String  `NAME`  fields.

Check out Brave’s code to see an example of how to make a path-based sampler [https://github.com/openzipkin/brave/tree/master/instrumentation/http#sampling-policy](https://github.com/openzipkin/brave/tree/master/instrumentation/http#sampling-policy)

If you want to completely rewrite the  `HttpTracing`  bean you can use the  `SkipPatternProvider`  interface to retrieve the URL  `Pattern`  for spans that should be not sampled. Below you can see an example of usage of  `SkipPatternProvider`  inside a server side,  `HttpSampler` .

```java
@Configuration
class Config {
@Bean(name = ServerSampler.NAME)
HttpSampler myHttpSampler(SkipPatternProvider provider) {
	Pattern pattern = provider.skipPattern();
	return new HttpSampler() {

		@Override public <Req> Boolean trySample(HttpAdapter<Req, ?> adapter, Req request) {
			String url = adapter.path(request);
			boolean shouldSkip = pattern.matcher(url).matches();
			if (shouldSkip) {
				return false;
			}
			return null;
		}
	};
}
}
```

## 59.2 TracingFilter

You can also modify the behavior of the  `TracingFilter` , which is the component that is responsible for processing the input HTTP request and adding tags basing on the HTTP response. You can customize the tags or modify the response headers by registering your own instance of the  `TracingFilter`  bean.

In the following example, we register the  `TracingFilter`  bean, add the  `ZIPKIN-TRACE-ID`  response header containing the current Span’s trace id, and add a tag with key  `custom`  and a value  `tag`  to the span.

```java
@Component
@Order(TraceWebServletAutoConfiguration.TRACING_FILTER_ORDER + 1)
class MyFilter extends GenericFilterBean {

	private final Tracer tracer;

	MyFilter(Tracer tracer) {
		this.tracer = tracer;
	}

	@Override public void doFilter(ServletRequest request, ServletResponse response,
			FilterChain chain) throws IOException, ServletException {
		Span currentSpan = this.tracer.currentSpan();
		if (currentSpan == null) {
			chain.doFilter(request, response);
			return;
		}
		// for readability we're returning trace id in a hex form
		((HttpServletResponse) response)
				.addHeader("ZIPKIN-TRACE-ID",
						currentSpan.context().traceIdString());
		// we can also add some custom tags
		currentSpan.tag("custom", "tag");
		chain.doFilter(request, response);
	}
}
```

## 59.3 Custom service name

By default, Sleuth assumes that, when you send a span to Zipkin, you want the span’s service name to be equal to the value of the  `spring.application.name`  property. That is not always the case, though. There are situations in which you want to explicitly provide a different service name for all spans coming from your application. To achieve that, you can pass the following property to your application to override that value (the example is for a service named  `myService` ):

```java
spring.zipkin.service.name: myService
```

## 59.4 Customization of Reported Spans

Before reporting spans (for example, to Zipkin) you may want to modify that span in some way. You can do so by using the  `SpanAdjuster`  interface.

In Sleuth, we generate spans with a fixed name. Some users want to modify the name depending on values of tags. You can implement the  `SpanAdjuster`  interface to alter that name.

The following example shows how to register two beans that implement  `SpanAdjuster` :

```java
@Bean SpanAdjuster adjusterOne() {
	return span -> span.toBuilder().name("foo").build();
}

@Bean SpanAdjuster adjusterTwo() {
	return span -> span.toBuilder().name(span.name() + " bar").build();
}
```

The preceding example results in changing the name of the reported span to  `foo bar` , just before it gets reported (for example, to Zipkin).

## 59.5 Host Locator

|images/important.png|Important|
|----|----|
|This section is about defining  **host**  from service discovery. It is  **NOT**  about finding Zipkin through service discovery. |

To define the host that corresponds to a particular span, we need to resolve the host name and port. The default approach is to take these values from server properties. If those are not set, we try to retrieve the host name from the network interfaces.

If you have the discovery client enabled and prefer to retrieve the host address from the registered instance in a service registry, you have to set the  `spring.zipkin.locator.discovery.enabled`  property (it is applicable for both HTTP-based and Stream-based span reporting), as follows:

```java
spring.zipkin.locator.discovery.enabled: true
```

