## 128.独立流应用程序

发送或接收消息从代理（例如RabbitMQ或Kafka），您可以使用 `spring-cloud-function-stream` 适配器.将适配器与Spring Cloud Stream中的相应Binders一起添加到类路径中.适配器将作为 `Processor` （输入和输出流）绑定到消息代理，除非用户使用 `spring.cloud.function.stream.{source,sink}.enabled=false` 显式禁用其中一个.

传入消息被路由到功能（或消费者）.如果只有一个，那么选择是显而易见的.如果有多个函数可以接受传入消息，则会检查消息以查看是否存在包含函数名称的 `stream_routekey` 标头.可以使用逗号或管道分隔的名称来组合路由标头或函数名称.Headers也会添加到供应商的外发邮件中.通过指定 `spring.cloud.function.stream.{processor,sink}.name` ，可以将没有路由密钥的消息专门路由到功能或消费者.如果无法识别单个函数来处理传入消息，则会出现错误，除非您设置 `spring.cloud.function.stream.shared=true` ，在这种情况下，此类消息将发送到所有兼容函数.可以使用 `spring.cloud.function.stream.source.name` 选择单个供应商来提供来自供应商的输出消息（如果有多个可用）.

如果消息代理不可用，并且函数目录包含在访问时立即生成消息的供应商，则
> someBinders将在启动时失败.您可以使用 `spring.cloud.function.strean.supplier.enabled=false` 标志在启动时关闭供应商的自动发布.

