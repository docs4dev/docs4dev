## 62.整合

## 62.1 OpenTracing

Spring Cloud Sleuth与[OpenTracing](http://opentracing.io/)兼容.如果在类路径上有OpenTracing，我们会自动注册OpenTracing  `Tracer`  bean.如果要禁用此功能，请将 `spring.sleuth.opentracing.enabled` 设置为 `false` 

## 62.2可运行和可调用

如果将逻辑包装在 `Runnable` 或 `Callable` 中，则可以将这些类包装在其侦听代表中，如以下 `Runnable` 示例所示：

```java
Runnable runnable = new Runnable() {
	@Override
	public void run() {
		// do some work
	}

	@Override
	public String toString() {
		return "spanNameFromToStringMethod";
	}
};
// Manual `TraceRunnable` creation with explicit "calculateTax" Span name
Runnable traceRunnable = new TraceRunnable(tracing, spanNamer, runnable,
		"calculateTax");
// Wrapping `Runnable` with `Tracing`. That way the current span will be available
// in the thread of `Runnable`
Runnable traceRunnableFromTracer = tracing.currentTraceContext().wrap(runnable);
```

以下示例显示了如何为 `Callable` 执行此操作：

```java
Callable<String> callable = new Callable<String>() {
	@Override
	public String call() throws Exception {
		return someLogic();
	}

	@Override
	public String toString() {
		return "spanNameFromToStringMethod";
	}
};
// Manual `TraceCallable` creation with explicit "calculateTax" Span name
Callable<String> traceCallable = new TraceCallable<>(tracing, spanNamer, callable,
		"calculateTax");
// Wrapping `Callable` with `Tracing`. That way the current span will be available
// in the thread of `Callable`
Callable<String> traceCallableFromTracer = tracing.currentTraceContext().wrap(callable);
```

这样，您可以确保为每次执行创建并关闭新范围.

## 62.3 Hystrix

### 62.3.1自定义并发策略

我们注册了一个名为[HystrixConcurrencyStrategy](https://github.com/Netflix/Hystrix/wiki/Plugins#concurrencystrategy)的自定义 `TraceCallable` ，它包含了Sleuth代表中的所有 `Callable` 实例.策略要么开始要么继续Span，具体取决于在调用Hystrix命令之前是否已经进行了跟踪.要禁用自定义Hystrix并发策略，请将 `spring.sleuth.hystrix.strategy.enabled` 设置为 `false` .

### 62.3.2手动命令设置

假设您有以下 `HystrixCommand` ：

```java
HystrixCommand<String> hystrixCommand = new HystrixCommand<String>(setter) {
	@Override
	protected String run() throws Exception {
		return someLogic();
	}
};
```

要传递跟踪信息，您必须在 `HystrixCommand` 的Sleuth版本中包含相同的逻辑，该版本称为 `TraceCommand` ，如以下示例所示：

```java
TraceCommand<String> traceCommand = new TraceCommand<String>(tracer, setter) {
	@Override
	public String doRun() throws Exception {
		return someLogic();
	}
};
```

## 62.4 RxJava

我们注册了一个自定义[RxJavaSchedulersHook](https://github.com/ReactiveX/RxJava/wiki/Plugins#rxjavaschedulershook)，它将所有 `Action0` 实例包含在他们的Sleuth代表中，称为 `TraceAction` .挂钩可以启动或继续Span，具体取决于在计划操作之前是否已经进行了跟踪.要禁用自定义 `RxJavaSchedulersHook` ，请将 `spring.sleuth.rxjava.schedulers.hook.enabled` 设置为 `false` .

您可以为不希望创建Span的线程名称定义正则表达式列表.为此，请在 `spring.sleuth.rxjava.schedulers.ignoredthreads` 属性中提供以逗号分隔的正则表达式列表.

|图片/ important.png |重要|
| ---- | ---- |
|反应式编程和Sleuth的建议方法是使用Reactor支持. |

## 62.5 HTTP集成

通过将 `spring.sleuth.web.enabled` 属性设置为值等于 `false` ，可以禁用此部分中的功能.

### 62.5.1 HTTP过滤器

通过 `TracingFilter` ，所有采样的传入请求都会导致创建Span.该Span的名称是 `http:` 请求发送到的路径.例如，如果请求已发送到 `/this/that` ，则名称将为 `http:/this/that` .您可以通过设置 `spring.sleuth.web.skipPattern` 属性来配置要跳过的URI.如果在类路径上有 `ManagementServerProperties` ，则其 `contextPath` 的值将附加到提供的跳过模式.如果您想重用Sleuth的默认跳过模式并只是追加自己的跳过模式，请使用 `spring.sleuth.web.additionalSkipPattern` 传递这些模式.

要更改跟踪过滤器注册的顺序，请设置 `spring.sleuth.web.filter-order` 属性.

要禁用记录未捕获异常的过滤器，可以禁用 `spring.sleuth.web.exception-throwing-filter-enabled` 属性.

### 62.5.2 HandlerInterceptor

由于我们希望span名称是精确的，我们使用 `TraceHandlerInterceptor` 包装现有的 `HandlerInterceptor` 或直接添加到现有的 `HandlerInterceptors` 列表中.  `TraceHandlerInterceptor` 为给定的 `HttpServletRequest` 添加了一个特殊的请求属性.如果 `TracingFilter` 没有看到此属性，则会创建“回退”范围，这是在服务器端创建的附加范围，以便在UI中正确显示跟踪.如果发生这种情况，可能会缺少仪器.在这种情况下，请在Spring Cloud Sleuth中提出问题.

### 62.5.3 Async Servlet支持

如果您的控制器返回 `Callable` 或 `WebAsyncTask` ，则Spring Cloud Sleuth将继续现有的Span，而不是创建新的Span.

### 62.5.4 WebFlux支持

通过 `TraceWebFilter` ，所有采样的传入请求都会导致创建Span.该Span的名称是 `http:` 请求发送到的路径.例如，如果请求已发送到 `/this/that` ，则名称为 `http:/this/that` .您可以使用 `spring.sleuth.web.skipPattern` 属性配置要跳过的URI.如果类路径上有 `ManagementServerProperties` ，则其值 `contextPath` 会附加到提供的跳过模式.如果要重用Sleuth的默认跳过模式并附加自己的跳过模式，请使用 `spring.sleuth.web.additionalSkipPattern` 传递这些模式.

要更改跟踪过滤器注册的顺序，请设置 `spring.sleuth.web.filter-order` 属性.

### 62.5.5 Dubbo RPC支持

通过与Brave的集成，Spring Cloud Sleuth支持[Dubbo](http://dubbo.io/).添加 `brave-instrumentation-dubbo-rpc` 依赖项就足够了：

```xml
<dependency>
<groupId>io.zipkin.brave</groupId>
<artifactId>brave-instrumentation-dubbo-rpc</artifactId>
</dependency>
```

您还需要设置包含以下内容的 `dubbo.properties` 文件：

```java
dubbo.provider.filter=tracing
dubbo.consumer.filter=tracing
```

您可以阅读有关Brave  -  Dubbo整合[here](https://github.com/openzipkin/brave/tree/master/instrumentation/dubbo-rpc)的更多信息.可以找到Spring Cloud Sleuth和Dubbo的示例[here](https://github.com/openzipkin/sleuth-webmvc-example/compare/add-dubbo-tracing).

## 62.6 HTTP客户端集成

### 62.6.1同步静止模板

我们注入 `RestTemplate` 拦截器以确保将所有跟踪信息传递给请求.每次进行调用时，都会创建一个新的Span.收到回复后会关闭.要阻止同步 `RestTemplate` 功能，请将 `spring.sleuth.web.client.enabled` 设置为 `false` .

|图片/ important.png |重要|
| ---- | ---- |
|你必须将 `RestTemplate` 注册为bean，以便拦截器被注入.如果使用 `new` 关键字创建 `RestTemplate` 实例，则检测不起作用. |

### 62.6.2异步静止模板

|图片/ important.png |重要|
| ---- | ---- |
|从Sleuth  `2.0.0` 开始，我们不再注册 `AsyncRestTemplate` 类型的bean.由你来创建这样的bean.然后我们检测它. |

要阻止 `AsyncRestTemplate` 功能，请将 `spring.sleuth.web.async.client.enabled` 设置为 `false` .要禁用创建默认 `TraceAsyncClientHttpRequestFactoryWrapper` ，请将 `spring.sleuth.web.async.client.factory.enabled` 设置为 `false` .如果您根本不想创建 `AsyncRestClient` ，请将 `spring.sleuth.web.async.client.template.enabled` 设置为 `false` .

#### Multiple异步休息模板

有时您需要使用异步静态模板的多个实现.在以下代码段中，您可以看到如何设置此类自定义 `AsyncRestTemplate` 的示例：

```java
@Configuration
@EnableAutoConfiguration
static class Config {

	@Bean(name = "customAsyncRestTemplate")
	public AsyncRestTemplate traceAsyncRestTemplate() {
		return new AsyncRestTemplate(asyncClientFactory(), clientHttpRequestFactory());
	}

	private ClientHttpRequestFactory clientHttpRequestFactory() {
		ClientHttpRequestFactory clientHttpRequestFactory = new CustomClientHttpRequestFactory();
		//CUSTOMIZE HERE
		return clientHttpRequestFactory;
	}

	private AsyncClientHttpRequestFactory asyncClientFactory() {
		AsyncClientHttpRequestFactory factory = new CustomAsyncClientHttpRequestFactory();
		//CUSTOMIZE HERE
		return factory;
	}
}
```

### 62.6.3 WebClient

我们注入了一个 `ExchangeFilterFunction` 实现，它创建了一个span，并且通过on-success和on-error回调，负责关闭客户端Span.

要阻止此功能，请将 `spring.sleuth.web.client.enabled` 设置为 `false` .

|图片/ important.png |重要|
| ---- | ---- |
|您必须将 `WebClient` 注册为bean，以便应用跟踪工具.如果使用 `new` 关键字创建 `WebClient` 实例，则检测不起作用. |

### 62.6.4 Traverson

如果使用[Traverson](https://docs.spring.io/spring-hateoas/docs/current/reference/html/#client.traverson)库，则可以将 `RestTemplate` 作为bean注入到Traverson对象中.由于 `RestTemplate` 已被截获，因此您可以完全支持在您的客户端进行跟踪.以下伪代码显示了如何执行此操作：

```java
@Autowired RestTemplate restTemplate;

Traverson traverson = new Traverson(URI.create("http://some/address"),
MediaType.APPLICATION_JSON, MediaType.APPLICATION_JSON_UTF8).setRestOperations(restTemplate);
// use Traverson
```

### 62.6.5 Apache HttpClientBuilder和HttpAsyncClientBuilder

我们检测 `HttpClientBuilder` 和 `HttpAsyncClientBuilder` ，以便将跟踪上下文注入到已发送的请求中.

要阻止这些功能，请将 `spring.sleuth.web.client.enabled` 设置为 `false` .

### 62.6.6 Netty HttpClient

我们检测Netty的 `HttpClient` .

要阻止此功能，请将 `spring.sleuth.web.client.enabled` 设置为 `false` .

|图片/ important.png |重要|
| ---- | ---- |
|您必须将 `HttpClient` 注册为bean，以便进行检测.如果使用 `new` 关键字创建 `HttpClient` 实例，则检测不起作用. |

### 62.6.7 UserInfoRestTemplateCustomizer

我们测试Spring Security的 `UserInfoRestTemplateCustomizer` .

要阻止此功能，请将 `spring.sleuth.web.client.enabled` 设置为 `false` .

## 62.7假装

默认情况下，Spring Cloud Sleuth通过 `TraceFeignClientAutoConfiguration` 提供与Feign的集成.您可以通过将 `spring.sleuth.feign.enabled` 设置为 `false` 来完全禁用它.如果这样做，则不会发生与Feign相关的检测.

Feign仪器的一部分通过 `FeignBeanPostProcessor` 完成.您可以通过将 `spring.sleuth.feign.processor.enabled` 设置为 `false` 来禁用它.如果将其设置为 `false` ，则Spring Cloud Sleuth不会检测任何自定义Feign组件.但是，所有默认检测仍然存在.

## 62.8异步通信

### 62.8.1 @Async带注释的方法

在Spring Cloud Sleuth中，我们检测与异步相关的组件，以便在线程之间传递跟踪信息.您可以通过将 `spring.sleuth.async.enabled` 的值设置为 `false` 来禁用此行为.

如果使用 `@Async` 注释方法，我们会自动创建一个具有以下特征的新Span：

- 如果方法用 `@SpanName` 注释，则注释的值是Span的名称.

- 如果方法未使用 `@SpanName` 进行批注，则Span名称是带注释的方法名称.

-  span用方法的类名和方法名标记.

### 62.8.2 @Scheduled Annotated Methods

在Spring Cloud Sleuth中，我们检测预定的方法执行，以便在线程之间传递跟踪信息.您可以通过将 `spring.sleuth.scheduled.enabled` 的值设置为 `false` 来禁用此行为.

如果使用 `@Scheduled` 注释方法，我们会自动创建具有以下特征的新范围：

- 范围名称是带注释的方法名称.

-  span用方法的类名和方法名标记.

如果要跳过某些 `@Scheduled` 带注释类的Span创建，可以使用与 `@Scheduled` 带注释类的完全限定名称匹配的正则表达式设置 `spring.sleuth.scheduled.skipPattern` .如果同时使用 `spring-cloud-sleuth-stream` 和 `spring-cloud-netflix-hystrix-stream` ，则会为每个Hystrix指标创建一个范围并发送给Zipkin.这种行为可能很烦人.这就是为什么，默认情况下， `spring.sleuth.scheduled.skipPattern=org.springframework.cloud.netflix.hystrix.stream.HystrixStreamTask` .

### 62.8.3 Executor，ExecutorService和ScheduledExecutorService

我们提供 `LazyTraceExecutor` ， `TraceableExecutorService` 和 `TraceableScheduledExecutorService` .每次提交，调用或计划新任务时，这些实现都会创建Span.

以下示例显示了在使用 `CompletableFuture` 时如何使用 `TraceableExecutorService` 传递跟踪信息：

```java
CompletableFuture<Long> completableFuture = CompletableFuture.supplyAsync(() -> {
	// perform some logic
	return 1_000_000L;
}, new TraceableExecutorService(beanFactory, executorService,
		// 'calculateTax' explicitly names the span - this param is optional
		"calculateTax"));
```

|图片/ important.png |重要|
| ---- | ---- |
|Sleuth无法使用 `parallelStream()` 开箱即用.如果要通过流传播跟踪信息，您必须使用 `supplyAsync(…)` 的方法，如前所示. |

#### 执行者的自定义

有时，您需要设置 `AsyncExecutor` 的自定义实例.以下示例显示如何设置此类自定义 `Executor` ：

```java
@Configuration
@EnableAutoConfiguration
@EnableAsync
static class CustomExecutorConfig extends AsyncConfigurerSupport {

	@Autowired BeanFactory beanFactory;

	@Override public Executor getAsyncExecutor() {
		ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
		// CUSTOMIZE HERE
		executor.setCorePoolSize(7);
		executor.setMaxPoolSize(42);
		executor.setQueueCapacity(11);
		executor.setThreadNamePrefix("MyExecutor-");
		// DON'T FORGET TO INITIALIZE
		executor.initialize();
		return new LazyTraceExecutor(this.beanFactory, executor);
	}
}
```

## 62.9消息

通过将 `spring.sleuth.messaging.enabled` 属性设置为值等于 `false` ，可以禁用此部分中的功能.

### 62.9.1 Spring Integration和Spring Cloud Stream

Spring Cloud Sleuth与[Spring Integration](https://projects.spring.io/spring-integration/)集成.它为发布和订阅事件创建了Span.要禁用Spring Integration检测，请将 `spring.sleuth.integration.enabled` 设置为 `false` .

您可以提供 `spring.sleuth.integration.patterns` 模式以明确提供要包括在跟踪中的通道的名称.默认情况下，包括 `hystrixStreamOutput` 通道的所有通道.

|图片/ important.png |重要|
| ---- | ---- |
|使用 `Executor` 构建Spring Integration  `IntegrationFlow` 时，必须使用 `Executor` 的未跟踪版本.使用 `TraceableExecutorService` 装饰Spring Integration Executor Channel会导致Span不正确关闭. |

### 62.9.2 Spring RabbitMq

我们检测 `RabbitTemplate` ，以便将跟踪标头注入到消息中.

要阻止此功能，请将 `spring.sleuth.messaging.rabbit.enabled` 设置为 `false` .

### 62.9.3Spring天Kafka

我们检测Spring Kafka的 `ProducerFactory` 和 `ConsumerFactory` ，以便将跟踪标头注入创建的Spring Kafka的 `Producer` 和 `Consumer` 中.

要阻止此功能，请将 `spring.sleuth.messaging.kafka.enabled` 设置为 `false` .

> We不支持通过 `@KafkaListener` 注释进行上下文传播.检查[this issue for more information](https://github.com/spring-cloud/spring-cloud-sleuth/issues/1001).

## 62.10 Zuul

我们通过使用跟踪信息丰富Ribbon请求来检测Zuul Ribbon集成.要禁用Zuul支持，请将 `spring.sleuth.zuul.enabled` 属性设置为 `false` .

