## 51.采样

可以采用采样来减少收集和报告的数据.如果未对Span进行采样，则不会增加任何开销（noop）.

采样是一个前期决策，这意味着报告数据的决定是在跟踪中的第一个操作中做出的，并且该决策是向下游传播的.

默认情况下，全局采样器将单个速率应用于所有跟踪的操作.  `Tracer.Builder.sampler` 控制此设置，默认为跟踪每个请求.

## 51.1声明性抽样

某些应用程序需要根据java方法的类型或注释进行采样.

大多数用户使用框架拦截器来自动执行此类策略.以下示例显示了内部可能如何工作：

```java
@Autowired Tracer tracer;

// derives a sample rate from an annotation on a java method
DeclarativeSampler<Traced> sampler = DeclarativeSampler.create(Traced::sampleRate);

@Around("@annotation(traced)")
public Object traceThing(ProceedingJoinPoint pjp, Traced traced) throws Throwable {
// When there is no trace in progress, this decides using an annotation
Sampler decideUsingAnnotation = declarativeSampler.toSampler(traced);
Tracer tracer = tracer.withSampler(decideUsingAnnotation);

// This code looks the same as if there was no declarative override
ScopedSpan span = tracer.startScopedSpan(spanName(pjp));
try {
return pjp.proceed();
} catch (RuntimeException | Error e) {
span.error(e);
throw e;
} finally {
span.finish();
}
}
```

## 51.2自定义采样

根据操作的不同，您可能希望应用不同的策略.例如，您可能不希望跟踪对静态资源（如图像）的请求，或者您可能希望跟踪对新api的所有请求.

大多数用户使用框架拦截器来自动执行此类策略.以下示例显示了内部可能如何工作：

```java
@Autowired Tracer tracer;
@Autowired Sampler fallback;

Span nextSpan(final Request input) {
Sampler requestBased = Sampler() {
@Override public boolean isSampled(long traceId) {
if (input.url().startsWith("/experimental")) {
return true;
} else if (input.url().startsWith("/static")) {
return false;
}
return fallback.isSampled(traceId);
}
};
return tracer.withSampler(requestBased).nextSpan();
}
```

## 51.3采用Spring Cloud Sleuth进行采样

默认情况下，Spring Cloud Sleuth将所有Span设置为不可导出.这意味着跟踪显示在日志中，但不显示在任何远程存储中.对于测试，默认值通常就足够了，如果只使用日志（例如，使用ELK聚合器），它可能就是您所需要的.如果将Span数据导出到Zipkin，还有一个 `Sampler.ALWAYS_SAMPLE` 设置可以导出所有内容，而 `ProbabilityBasedSampler` 设置可以对固定的跨距进行采样.

> 如果使用 `spring-cloud-sleuth-zipkin` ， `ProbabilityBasedSampler` 是默认值.您可以通过设置 `spring.sleuth.sampler.probability` 来配置导出.传递的值必须是从 `0.0` 到 `1.0` 的两倍.

可以通过创建bean定义来安装采样器，如以下示例所示：

```java
@Bean
public Sampler defaultSampler() {
	return Sampler.ALWAYS_SAMPLE;
}
```

> 你可以设置HTTP标头 `X-B3-Flags` 到 `1` ，或者，在进行消息传递时，您可以将 `spanFlags` 标头设置为 `1` .这样做会强制当前Span可以导出，而不管采样决定如何.

