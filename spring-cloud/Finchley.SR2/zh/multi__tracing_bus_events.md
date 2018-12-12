## 46.跟踪 Bus 活动

可以通过设置 `spring.cloud.bus.trace.enabled=true` 来跟踪总线事件（ `RemoteApplicationEvent` 的子类）.如果这样做，Spring Boot  `TraceRepository` （如果存在）显示发送的每个事件以及每个服务实例的所有ack.以下示例来自 `/trace` endpoints：

```java
{
"timestamp": "2015-11-26T10:24:44.411+0000",
"info": {
"signal": "spring.cloud.bus.ack",
"type": "RefreshRemoteApplicationEvent",
"id": "c4d374b7-58ea-4928-a312-31984def293b",
"origin": "stores:8081",
"destination": "*:**"
}
},
{
"timestamp": "2015-11-26T10:24:41.864+0000",
"info": {
"signal": "spring.cloud.bus.sent",
"type": "RefreshRemoteApplicationEvent",
"id": "c4d374b7-58ea-4928-a312-31984def293b",
"origin": "customers:9000",
"destination": "*:**"
}
},
{
"timestamp": "2015-11-26T10:24:41.862+0000",
"info": {
"signal": "spring.cloud.bus.ack",
"type": "RefreshRemoteApplicationEvent",
"id": "c4d374b7-58ea-4928-a312-31984def293b",
"origin": "customers:9000",
"destination": "*:**"
}
}
```

前面的跟踪显示 `RefreshRemoteApplicationEvent` 从 `customers:9000` 发送，广播到所有服务，并由 `customers:9000` 和 `stores:8081` 接收（确认）.

要自己处理ack信号，可以为 `AckRemoteApplicationEvent` 和 `SentApplicationEvent` 类型添加 `@EventListener` 到您的应用程序（并启用跟踪）.或者，您可以点击 `TraceRepository` 并从那里挖掘数据.

> 任何总线应用程序都可以跟踪ack.但是，有时，在中央服务中执行此操作非常有用，该服务可以对数据执行更复杂的查询或将其转发到专门的跟踪服务.

