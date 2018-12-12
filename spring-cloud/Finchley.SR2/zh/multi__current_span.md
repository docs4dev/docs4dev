## 54.当前Span

勇敢支持代表飞行中操作的“当前Span”概念.您可以使用 `Tracer.currentSpan()` 将自定义标记添加到范围，并使用 `Tracer.nextSpan()` 创建任何正在运行的子项.

|图片/ important.png |重要|
| ---- | ---- |
|在Sleuth中，您可以通过 `tracer.currentSpan()` 方法自动装配 `Tracer`  bean以检索当前Span.要检索当前上下文，只需调用 `tracer.currentSpan().context()` .要将当前跟踪ID作为String获取，可以使用 `traceIdString()` 方法，如下所示： `tracer.currentSpan().context().traceIdString()` . |

## 54.1手动设置范围内的范围

编写新检测时，将在范围内创建的范围放在当前范围内非常重要.这样做不仅允许用户使用 `Tracer.currentSpan()` 访问它，而且还允许自定义（如SLF4J MDC）查看当前跟踪ID.

`Tracer.withSpanInScope(Span)` 促进了这一点，并且通过使用try-with-resources惯用法最方便地使用.每当调用外部代码（例如继续执行拦截器或其他操作）时，将span放在范围内，如以下示例所示：

```java
@Autowired Tracer tracer;

try (SpanInScope ws = tracer.withSpanInScope(span)) {
return inboundRequest.invoke();
} finally { // note the scope is independent of the span
span.finish();
}
```

在边缘情况下，您可能需要临时清除当前Span（例如，启动不应与当前请求关联的任务）.要执行tso，将null传递给 `withSpanInScope` ，如以下示例所示：

```java
@Autowired Tracer tracer;

try (SpanInScope cleared = tracer.withSpanInScope(null)) {
startBackgroundThread();
}
```

