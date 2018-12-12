## 55.仪表

Spring Cloud Sleuth会自动检测所有Spring应用程序，因此您无需执行任何操作即可激活它.根据可用的堆栈，使用各种技术添加仪器.例如，对于servlet Web应用程序，我们使用 `Filter` ，对于Spring Integration，我们使用 `ChannelInterceptors` .

您可以自定义span标记中使用的键.为了限制Span数据的量，默认情况下，HTTP请求仅使用少量元数据标记，例如状态代码，主机和URL.您可以通过配置 `spring.sleuth.keys.http.headers` （Headers名称列表）来添加请求标头.

仅当存在允许的 `Sampler` 时，才会收集和导出
> Tags.默认情况下，没有这样的 `Sampler` ，以确保在没有配置的情况下没有意外收集太多数据的危险）.

