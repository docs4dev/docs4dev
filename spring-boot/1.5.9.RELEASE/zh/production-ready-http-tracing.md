## 59. HTTP跟踪

将自动为所有HTTP请求启用跟踪.您可以查看 `httptrace` endpoints并获取有关最近100次请求 - 响应交换的基本信息.

## 59.1自定义HTTP跟踪

要自定义每个跟踪中包含的项目，请使用 `management.trace.http.include` 配置属性.对于高级自定义，请考虑注册自己的 `HttpExchangeTracer` 实现.

默认情况下，使用存储最后100个请求 - 响应交换的跟踪的 `InMemoryHttpTraceRepository` .如果需要扩展容量，可以定义自己的 `InMemoryHttpTraceRepository`  bean实例.您还可以创建自己的替代 `HttpTraceRepository` 实现.

