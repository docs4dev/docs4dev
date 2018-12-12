## 68. Consul Retry

If you expect that the consul agent may occasionally be unavailable when your app starts, you can ask it to keep trying after a failure. You need to add  `spring-retry`  and  `spring-boot-starter-aop`  to your classpath. The default behaviour is to retry 6 times with an initial backoff interval of 1000ms and an exponential multiplier of 1.1 for subsequent backoffs. You can configure these properties (and others) using  `spring.cloud.consul.retry.*`  configuration properties. This works with both Spring Cloud Consul Config and Discovery registration.

> To take full control of the retry add a  `@Bean`  of type  `RetryOperationsInterceptor`  with id "consulRetryInterceptor". Spring Retry has a  `RetryInterceptorBuilder`  that makes it easy to create one.

