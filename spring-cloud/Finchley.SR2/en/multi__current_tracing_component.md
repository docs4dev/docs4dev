## 53. Current Tracing Component

Brave supports a “current tracing component” concept, which should only be used when you have no other way to get a reference. This was made for JDBC connections, as they often initialize prior to the tracing component.

The most recent tracing component instantiated is available through  `Tracing.current()` . You can also use  `Tracing.currentTracer()`  to get only the tracer. If you use either of these methods, do not cache the result. Instead, look them up each time you need them.
