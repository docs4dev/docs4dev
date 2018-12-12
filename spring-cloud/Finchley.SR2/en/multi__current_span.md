## 54. Current Span

Brave supports a “current span” concept which represents the in-flight operation. You can use  `Tracer.currentSpan()`  to add custom tags to a span and  `Tracer.nextSpan()`  to create a child of whatever is in-flight.

|images/important.png|Important|
|----|----|
|In Sleuth, you can autowire the  `Tracer`  bean to retrieve the current span via  `tracer.currentSpan()`  method. To retrieve the current context just call  `tracer.currentSpan().context()` . To get the current trace id as String you can use the  `traceIdString()`  method like this:  `tracer.currentSpan().context().traceIdString()` . |

## 54.1 Setting a span in scope manually

When writing new instrumentation, it is important to place a span you created in scope as the current span. Not only does doing so let users access it with  `Tracer.currentSpan()` , but it also allows customizations such as SLF4J MDC to see the current trace IDs.

`Tracer.withSpanInScope(Span)`  facilitates this and is most conveniently employed by using the try-with-resources idiom. Whenever external code might be invoked (such as proceeding an interceptor or otherwise), place the span in scope, as shown in the following example:

```java
@Autowired Tracer tracer;

try (SpanInScope ws = tracer.withSpanInScope(span)) {
return inboundRequest.invoke();
} finally { // note the scope is independent of the span
span.finish();
}
```

In edge cases, you may need to clear the current span temporarily (for example, launching a task that should not be associated with the current request). To do tso, pass null to  `withSpanInScope` , as shown in the following example:

```java
@Autowired Tracer tracer;

try (SpanInScope cleared = tracer.withSpanInScope(null)) {
startBackgroundThread();
}
```

