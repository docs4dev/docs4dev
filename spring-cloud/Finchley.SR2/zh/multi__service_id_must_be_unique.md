## 44.服务ID必须是唯一的

总线尝试两次以消除处理事件 - 一次从原始 `ApplicationEvent` 和一次从队列.为此，它会根据当前服务ID检查发送服务ID.如果服务的多个实例具有相同的ID，则事件为没处理.在本地计算机上运行时，每个服务都位于不同的端口上，该端口是ID的一部分. Cloud Foundry提供了一个差异化指数.要确保该ID在Cloud Foundry外部是唯一的，请将 `spring.application.index` 设置为每个服务实例的唯一ID.
