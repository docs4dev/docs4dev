## 59.自定义

## 59.1 HTTP

如果需要自定义客户端/服务器解析HTTP相关Span，只需注册 `brave.http.HttpClientParser` 或 `brave.http.HttpServerParser` 类型的bean.如果需要客户端/服务器采样，只需注册 `brave.http.HttpSampler` 类型的bean，并将bean  `sleuthClientSampler` 命名为客户端采样器，将 `sleuthServerSampler` 命名为server sampler.为方便起见， `@ClientSampler` 和 `@ServerSampler` 注释可用于注入正确的bean或通过其静态String  `NAME` 字段引用bean名称.

查看Brave的代码，查看如何制作基于路径的采样器的示例[https://github.com/openzipkin/brave/tree/master/instrumentation/http#sampling-policy](https://github.com/openzipkin/brave/tree/master/instrumentation/http#sampling-policy)

如果要完全重写 `HttpTracing`  bean，可以使用 `SkipPatternProvider` 接口检索URL  `Pattern` ，以查找不应采样的Span.您可以在下面看到服务器端 `SkipPatternProvider` 的使用示例 `HttpSampler` .

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

您还可以修改 `TracingFilter` 的行为，该组件负责处理输入HTTP请求并根据HTTP响应添加标记.您可以通过注册自己的 `TracingFilter`  bean实例来自定义标记或修改响应标头.

在下面的示例中，我们注册 `TracingFilter`  bean，添加包含当前Span的跟踪ID的 `ZIPKIN-TRACE-ID` 响应头，并向span添加带有键 `custom` 和值 `tag` 的标记.

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

## 59.3自定义服务名称

默认情况下，Sleuth假设，当您向Zipkin发送Span时，您希望span的服务名称等于 `spring.application.name` 属性的值.但情况并非总是如此.在某些情况下，您希望为来自应用程序的所有跨域显式提供不同的服务名称.为此，您可以将以下属性传递给应用程序以覆盖该值（该示例适用于名为 `myService` 的服务）：

```java
spring.zipkin.service.name: myService
```

## 59.4报告Span的自定义

在报告Span之前（例如，对Zipkin），您可能希望以某种方式修改该Span.您可以使用 `SpanAdjuster` 界面执行此操作.

在Sleuth中，我们生成具有固定名称的Span.某些用户希望根据标记的值修改名称.您可以实现 `SpanAdjuster` 接口来更改该名称.

以下示例显示如何注册实现 `SpanAdjuster` 的两个bean：

```java
@Bean SpanAdjuster adjusterOne() {
	return span -> span.toBuilder().name("foo").build();
}

@Bean SpanAdjuster adjusterTwo() {
	return span -> span.toBuilder().name(span.name() + " bar").build();
}
```

前面的示例导致在报告的范围报告之前将报告的范围的名称更改为 `foo bar` （例如，Zipkin）.

## 59.5主机定位器

|图片/ important.png |重要|
| ---- | ---- |
|本节介绍如何从服务发现中定义 **host** .关于通过服务发现找到Zipkin是 **NOT** . |

要定义与特定Span对应的主机，我们需要解析主机名和端口.默认方法是从服务器属性中获取这些值.如果未设置，我们尝试从网络接口检索主机名.

如果启用了发现客户端并且希望从服务注册表中的已注册实例检索主机地址，则必须设置 `spring.zipkin.locator.discovery.enabled` 属性（它适用于基于HTTP和基于流的Span报告），如下所示：

```java
spring.zipkin.locator.discovery.enabled: true
```

