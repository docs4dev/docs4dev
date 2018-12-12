## 68.Consul重试

如果您希望应用程序启动时偶尔可能无法使用Consul代理，您可以要求它在失败后继续尝试.您需要将 `spring-retry` 和 `spring-boot-starter-aop` 添加到类路径中.默认行为是重试6次，初始退避间隔为1000毫秒，指数乘数为1.1，以便后续退避.您可以使用 `spring.cloud.consul.retry.*` 配置属性配置这些属性（和其他属性）.这适用于Spring Cloud Consul Config和Discovery注册.

> 完全控制重试添加 `@Bean` 类型 `RetryOperationsInterceptor` ，ID为"consulRetryInterceptor". Spring Retry有一个 `RetryInterceptorBuilder` ，可以很容易地创建一个.

