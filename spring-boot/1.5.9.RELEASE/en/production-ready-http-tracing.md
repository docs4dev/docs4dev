## 59. HTTP Tracing

Tracing is automatically enabled for all HTTP requests. You can view the  `httptrace`  endpoint and obtain basic information about the last 100 request-response exchanges.

## 59.1 Custom HTTP tracing

To customize the items that are included in each trace, use the  `management.trace.http.include`  configuration property. For advanced customization, consider registering your own  `HttpExchangeTracer`  implementation.

By default, an  `InMemoryHttpTraceRepository`  that stores traces for the last 100 request-response exchanges is used. If you need to expand the capacity, you can define your own instance of the  `InMemoryHttpTraceRepository`  bean. You can also create your own alternative  `HttpTraceRepository`  implementation.

